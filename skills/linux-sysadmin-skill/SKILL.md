---
name: linux-sysadmin-skill
description: >
  Linux Sysadmin Skill fuer IT-Administratoren und DevOps-Engineers.
  Bash-Scripting, Skript-Templates, Argument-Parsing, Logging, Retry,
  SSH-Haertung, sysctl, PAM, nftables, AppArmor, SELinux, auditd, CIS Benchmark,
  systemd Services, Timer, Journald, Drop-In Overrides, Troubleshooting,
  Performance-Analyse, cpu, iostat, vmstat, ss, perf, strace,
  Netzwerk, ip-Befehle, Netplan, Bonding, VLAN, WireGuard VPN, tcpdump, DNS,
  Storage, LVM, mdadm RAID, NFS, Samba, ZFS, Snapshots, SMART,
  Monitoring, Prometheus Node Exporter, Grafana, Loki, Promtail, Alert-Regeln,
  Ansible, Playbooks, Roles, Inventory, Vault, AWX, Rolling Updates,
  Backup, restic, BorgBackup, Borg, rsync, Bacula, Bareos,
  Ubuntu, Debian, RHEL, CentOS, Rocky, AlmaLinux.
  Outputs: Bash-Skripte (set -euo pipefail), Ansible-Playbooks, Config-Dateien, Checklisten.
---

# Linux Sysadmin Skill

## Guiding Principles

**set -euo pipefail ist Pflicht.** Jedes produktive Bash-Skript beginnt damit.
**Idempotenz.** Skripte und Playbooks müssen mehrfach ausführbar sein ohne Nebenwirkungen.
**Logging gehört dazu.** Jedes Skript loggt nach /var/log/ und sendet Alerts bei Fehlern.
**Härtung vor Inbetriebnahme.** SSH, sysctl, PAM und Firewall konfigurieren bevor ein System in Produktion geht.
**systemd-Timer statt cron.** Besseres Logging, Abhängigkeiten, Persistent-Flag.
**Distributionsunterschiede beachten.** apt (Ubuntu/Debian) vs. dnf (RHEL/Rocky), firewalld vs. nftables.

---

## Referenzmodule

- `references/bash-scripting.md` — Skript-Templates, Argument-Parsing, Logging, Arrays, Parallelverarbeitung, Health-Check
- `references/linux-hardening.md` — SSH, sysctl, nftables, PAM, AppArmor, SELinux, auditd, CIS Auto-Check
- `references/systemd.md` — Service-Units, Timer, Journald, Targets, Hardening, Drop-In, Troubleshooting
- `references/linux-monitoring.md` — Node Exporter, PromQL, Alert-Regeln, Loki/Promtail, Nagios-Check-Skript
- `references/linux-network.md` — ip-Befehle, Netplan, Bonding/VLAN, WireGuard, tcpdump, NFS-Troubleshooting
- `references/linux-storage.md` — LVM (Snapshot, Resize), mdadm RAID, NFS, ZFS, SMART-Diagnose
- `references/ansible.md` — Projektstruktur, Playbooks, Roles, Vault, Rolling Updates, AWX
- `references/evals.md` — Eval-Testfälle und Qualitätskriterien

Routing:
- Bash-Skript schreiben / Template → `bash-scripting.md`
- SSH / sysctl / Firewall / Härtung / CIS → `linux-hardening.md`
- systemd Service / Timer / journalctl → `systemd.md`
- Prometheus / Grafana / Monitoring / Alerts → `linux-monitoring.md`
- ip / Route / VPN / Netzwerk / tcpdump → `linux-network.md`
- LVM / RAID / NFS / ZFS / Disk → `linux-storage.md`
- Ansible / Playbook / Role / Vault → `ansible.md`

---

## Bash — Schnellreferenz

```bash
# Pflicht-Header
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'

# Logging
log()  { echo "[$(date '+%H:%M:%S')] INFO  $*" | tee -a /var/log/script.log; }
warn() { echo "[$(date '+%H:%M:%S')] WARN  $*" | tee -a /var/log/script.log >&2; }
err()  { echo "[$(date '+%H:%M:%S')] ERROR $*" | tee -a /var/log/script.log >&2; }
die()  { err "$*"; exit 1; }

# Cleanup immer registrieren
trap 'echo "Exit: $?"' EXIT

# Abhängigkeiten prüfen
for cmd in curl jq systemctl; do
    command -v "$cmd" &>/dev/null || die "Fehlendes Tool: $cmd"
done

# Retry-Pattern
retry() { local n=$1 d=$2; shift 2
    for ((i=1; i<=n; i++)); do "$@" && return 0
        (( i < n )) && { warn "Versuch $i/$n fehlgeschlagen, warte ${d}s"; sleep $d; }
    done; die "Fehlgeschlagen nach $n Versuchen: $*"; }
```

→ Vollständiges Template mit Lock, Argument-Parsing, Arrays, Parallelverarbeitung: `references/bash-scripting.md`

---

## Härtung — Schnellreferenz

```bash
# SSH: Passwort-Auth deaktivieren (Pflicht!)
echo "PasswordAuthentication no" >> /etc/ssh/sshd_config.d/99-hardening.conf
echo "PermitRootLogin no"        >> /etc/ssh/sshd_config.d/99-hardening.conf
echo "AllowGroups sshusers"      >> /etc/ssh/sshd_config.d/99-hardening.conf
systemctl reload sshd

# sysctl: Wichtigste Kernel-Härtungen
sysctl -w kernel.randomize_va_space=2      # ASLR
sysctl -w net.ipv4.tcp_syncookies=1        # SYN-Flood-Schutz
sysctl -w net.ipv4.conf.all.rp_filter=1    # IP-Spoofing-Schutz
sysctl -w kernel.dmesg_restrict=1          # dmesg nur für root

# nftables: Einfache Server-Grundregel
nft add table inet filter
nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'
nft add rule inet filter input ct state established,related accept
nft add rule inet filter input iif lo accept
nft add rule inet filter input tcp dport 22 accept
```

→ Vollständige nftables-Config, PAM, AppArmor-Profile, auditd-Regeln, CIS-Check: `references/linux-hardening.md`

---

## systemd — Schnellreferenz

```bash
# Service-Management
systemctl status|start|stop|restart|reload|enable|disable myapp
systemctl list-units --failed                  # Fehlgeschlagene Units
systemctl list-timers                          # Timer-Übersicht
journalctl -u myapp -f                         # Live-Logs
journalctl -u myapp --since "1h ago" -p err    # Fehler letzte Stunde
systemd-analyze blame                          # Langsamste Boot-Units
systemd-analyze security myapp.service         # Hardening-Score
```

```ini
# Minimales gehärtetes Service-Template
[Service]
User=myapp
ExecStart=/usr/local/bin/myapp
Restart=on-failure
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ReadWritePaths=/var/lib/myapp
```

→ Vollständiges gehärtetes Unit mit allen Options, Timer, Drop-In, Journald-Config: `references/systemd.md`

---

## Netzwerk — Schnellreferenz

```bash
ip addr show                                   # IPs
ip route show                                  # Routing-Tabelle
ss -tlnp                                       # Lauschende Ports + Prozesse
ss -tunaple                                    # Alle Verbindungen
dig @10.0.0.53 firma.local A                   # DNS-Test
mtr -n --report 8.8.8.8                       # Netzwerk-Pfad analysieren
tcpdump -i eth0 -n 'port 443' -w /tmp/cap.pcap
resolvectl status                              # DNS-Resolver-Status
```

→ Netplan Bonding/VLAN, WireGuard VPN, nftables, Performance-Tuning (BBR): `references/linux-network.md`

---

## Storage — Schnellreferenz

```bash
# LVM: LV vergrößern (live, kein Unmount nötig)
lvextend -L +20G /dev/vg_data/lv_app
resize2fs /dev/vg_data/lv_app     # ext4
# xfs_growfs /mountpoint          # XFS

# LVM-Snapshot für Backup
lvcreate -L 10G -s -n snap /dev/vg_data/lv_app
mount -o ro /dev/vg_data/snap /mnt/snapshot
# Backup von /mnt/snapshot...
lvremove /dev/vg_data/snap

# RAID-Status
cat /proc/mdstat
mdadm --detail /dev/md0

# SMART-Schnelltest
smartctl -t short /dev/sda
smartctl -a /dev/sda | grep -E "Reallocated|Pending|Uncorrectable"
```

→ Vollständige LVM-Workflows, mdadm RAID, NFS-Server/Client, ZFS, Benchmarking: `references/linux-storage.md`

---

## Ansible — Schnellreferenz

```bash
# Ad-hoc
ansible webservers -m ping
ansible webservers -m shell -a "uptime && df -h /"
ansible webservers -m service -a "name=nginx state=reloaded" --become

# Playbook-Workflow
ansible-playbook --syntax-check playbooks/site.yml
ansible-playbook --check --diff  playbooks/site.yml   # Dry-Run
ansible-playbook                 playbooks/site.yml \
    --limit "webservers:!web02"  \
    --tags "nginx"               \
    --vault-password-file ~/.vault_password
```

→ Vollständige Projektstruktur, Roles, Vault, Rolling Updates, AWX: `references/ansible.md`

---

## Monitoring — Schnellreferenz

```promql
# CPU-Auslastung (%)
100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# RAM frei (%)
node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100

# Disk-Auslastung (%)
100 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes * 100)
```

→ Node Exporter Setup, Alert-Regeln, Loki/Promtail, Nagios-Check-Skript: `references/linux-monitoring.md`

---

## Distributions-Unterschiede (Schnellreferenz)

| Aufgabe | Ubuntu/Debian | RHEL/Rocky/AlmaLinux |
|---|---|---|
| Paket installieren | `apt install nginx` | `dnf install nginx` |
| Service aktivieren | `systemctl enable --now nginx` | identisch |
| Firewall | `ufw` oder `nftables` | `firewalld` oder `nftables` |
| MAC-Framework | AppArmor | SELinux |
| Log-Rotation | `/etc/logrotate.d/` | identisch |
| Netzwerk-Config | Netplan | NetworkManager / nmcli |
| Python-Pfad | `/usr/bin/python3` | `/usr/bin/python3` |
| Paketverwaltung Low-Level | `dpkg` | `rpm` |
| Repo hinzufügen | `add-apt-repository` | `dnf config-manager --add-repo` |

---

## Output-Formate je Aufgabentyp

| Aufgabe | Format |
|---|---|
| Bash-Skript | set -euo pipefail, Logging, Cleanup-Trap, Argument-Parsing |
| systemd Unit | Vollständige [Unit]/[Service]/[Install] mit Hardening-Options |
| Ansible Playbook | Vollständig mit pre/post_tasks, handlers, become |
| Ansible Role | tasks/main.yml + defaults/main.yml + handlers/main.yml |
| Härtungsskript | Idempotent, mit Kommentaren pro Maßnahme, Begründung |
| Monitoring Alert | YAML-Alert mit expr, for, labels, annotations, runbook_url |
| Checkliste | Markdown-Checkboxen, gruppiert, auf Deutsch |
