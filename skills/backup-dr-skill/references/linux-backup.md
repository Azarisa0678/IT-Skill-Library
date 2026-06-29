# Linux Backup — Referenzmodul

rsync, restic, BorgBackup, Bacula/Bareos — Skripte, Automatisierung, Monitoring.

---

## restic (empfohlen: modern, deduplizierend, verschlüsselt)

```bash
#!/usr/bin/env bash
# restic-backup.sh — Produktionsreifes Backup-Skript mit restic
set -euo pipefail

RESTIC_REPO="s3:https://s3.eu-central-2.wasabisys.com/backup-linux-prod"
export AWS_ACCESS_KEY_ID="WASABI_ACCESS_KEY"
export AWS_SECRET_ACCESS_KEY="WASABI_SECRET_KEY"
export RESTIC_PASSWORD_FILE="/etc/restic/password"  # chmod 400

BACKUP_PATHS="/etc /home /var/lib /opt/apps"
EXCLUDE_PATTERNS=("/proc" "/sys" "/dev" "/run" "/tmp" "/var/tmp" "/var/cache" "*.log" "*.tmp" "*.swap")
RETENTION_DAILY=7; RETENTION_WEEKLY=4; RETENTION_MONTHLY=12
LOG_FILE="/var/log/restic-backup.log"; NOTIFY_EMAIL="it-admin@firma.de"

exec >> "$LOG_FILE" 2>&1
echo "=== Backup Start: $(date '+%Y-%m-%d %H:%M:%S') ==="

if ! restic -r "$RESTIC_REPO" snapshots &>/dev/null; then
    restic -r "$RESTIC_REPO" init
fi

EXCLUDES=""; for p in "${EXCLUDE_PATTERNS[@]}"; do EXCLUDES="$EXCLUDES --exclude $p"; done

if restic -r "$RESTIC_REPO" backup $EXCLUDES --tag "$(hostname)" --tag "$(date +%Y-%m)" $BACKUP_PATHS; then
    echo "Backup erfolgreich."
else
    echo "FEHLER: Backup fehlgeschlagen!"
    echo "Backup FEHLER auf $(hostname) - $(date)" | mail -s "Backup-Fehler" "$NOTIFY_EMAIL"
    exit 1
fi

restic -r "$RESTIC_REPO" forget \
    --keep-daily "$RETENTION_DAILY" --keep-weekly "$RETENTION_WEEKLY" \
    --keep-monthly "$RETENTION_MONTHLY" --prune --tag "$(hostname)"

# Vollprüfung nur sonntags, sonst nur Metadaten
if [[ "$(date +%u)" == "7" ]]; then
    restic -r "$RESTIC_REPO" check --read-data
else
    restic -r "$RESTIC_REPO" check
fi

echo "=== Backup Ende: $(date '+%Y-%m-%d %H:%M:%S') ==="
```

```bash
# restic — Verwaltungsbefehle

restic -r "$RESTIC_REPO" snapshots                                          # Alle Snapshots
restic -r "$RESTIC_REPO" restore latest --target /tmp/restore \
    --include "/etc/nginx/nginx.conf"                                       # Einzeldatei restore
restic -r "$RESTIC_REPO" restore SNAPSHOT_ID --target /                     # Vollrestore
restic -r "$RESTIC_REPO" find "*.conf" --snapshot latest                    # Suchen
restic -r "$RESTIC_REPO" stats                                              # Statistiken
restic -r "$RESTIC_REPO" mount /mnt/restic-browse                           # FUSE-Mount
restic -r "$RESTIC_REPO" ls latest /etc                                     # Inhalt anzeigen
```

---

## BorgBackup (lokal/SSH, sehr effizient)

```bash
#!/usr/bin/env bash
set -euo pipefail
export BORG_REPO="ssh://backup-user@backup-nas.intern:22/backup/$(hostname)"
export BORG_PASSPHRASE="$(cat /etc/borg/passphrase)"

borg create --verbose --filter AME --list --stats --compression lz4 \
    --exclude-caches --exclude '/home/*/.cache' --exclude '/tmp' --exclude '/var/tmp' \
    "$BORG_REPO::$(hostname)-{now:%Y%m%d-%H%M%S}" \
    /etc /home /var/lib /opt

borg prune --list --keep-daily 7 --keep-weekly 4 --keep-monthly 12 "$BORG_REPO"
[[ "$(date +%d)" == "01" ]] && borg compact "$BORG_REPO"
```

```bash
# Borg — Verwaltungsbefehle
borg list "$BORG_REPO"                                              # Archive auflisten
borg list "$BORG_REPO::archivname"                                 # Archiv-Inhalt
cd /tmp/restore && borg extract "$BORG_REPO::archivname" etc/nginx/nginx.conf
borg check --verify-data "$BORG_REPO"                              # Integritätsprüfung
borg info "$BORG_REPO"                                             # Statistiken
```

---

## rsync — Time Machine-Stil (Hardlinks)

```bash
#!/usr/bin/env bash
set -euo pipefail
SRC="/data /etc /home"
DEST_BASE="/mnt/backup-nas/$(hostname)"
DATE=$(date '+%Y%m%d_%H%M%S')
DEST="$DEST_BASE/$DATE"; LATEST="$DEST_BASE/latest"

for path in $SRC; do
    safe="${path//\//_}"
    rsync -avz --delete --link-dest="$LATEST/$safe" \
        --exclude={/proc,/sys,/dev,/tmp,'*.tmp','*.swap'} \
        "$path/" "$DEST/$safe/"
done

ln -sfn "$DEST" "$LATEST"
find "$DEST_BASE" -maxdepth 1 -type d -name "20*" -mtime +30 -exec rm -rf {} \; 2>/dev/null || true
```

---

## systemd Timer für automatisches Backup

```ini
# /etc/systemd/system/restic-backup.service
[Unit]
Description=restic Backup
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/restic-backup.sh
TimeoutStartSec=4h
Restart=no
```

```ini
# /etc/systemd/system/restic-backup.timer
[Unit]
Description=Tägliches restic Backup 02:30 Uhr

[Timer]
OnCalendar=*-*-* 02:30:00
RandomizedDelaySec=600
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
systemctl daemon-reload && systemctl enable --now restic-backup.timer
systemctl list-timers restic-backup.timer
journalctl -u restic-backup.service --since "24h ago"
```

---

## Backup-Monitoring Skript

```bash
#!/usr/bin/env bash
# Täglicher Health-Check: restic + Borg + Speicherplatz
set -euo pipefail
REPORT="/tmp/backup-health-$(date +%Y%m%d).txt"; ALERT=0

echo "=== Backup Health Check $(date) ===" > "$REPORT"

# restic: letzter Snapshot
LAST=$(restic -r "$RESTIC_REPO" snapshots --json 2>/dev/null | \
    python3 -c "import json,sys; s=json.load(sys.stdin); print(s[-1]['time'][:19])" 2>/dev/null || echo "FEHLER")
echo "restic letzter Snapshot: $LAST" >> "$REPORT"
[[ "$LAST" == "FEHLER" ]] && { echo "WARNUNG: Repo nicht erreichbar!" >> "$REPORT"; ALERT=1; }

# Speicherplatz Backup-NAS
df -h /mnt/backup-nas 2>/dev/null >> "$REPORT" || true

# systemd Service-Status
systemctl is-failed restic-backup.service 2>/dev/null && {
    echo "WARNUNG: restic-backup.service fehlgeschlagen!" >> "$REPORT"; ALERT=1
}

cat "$REPORT"
[[ $ALERT -eq 1 ]] && mail -s "Backup-Problem auf $(hostname)" it-admin@firma.de < "$REPORT"
```

---

## Bareos (Community-Fork Bacula) — Schnellreferenz

```bash
# bconsole-Schnellbefehle
bconsole -c "status director"          # Director-Status
bconsole -c "list jobs days=1"         # Jobs letzte 24h
bconsole -c "list jobs status=E days=7" # Fehler letzte 7 Tage
bconsole -c "list volumes"             # Volume-Übersicht
bconsole -c "run job=BackupClient1 yes" # Job sofort starten
```
