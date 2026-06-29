# systemd — Referenzmodul

Services, Timer, Journald, Targets, Troubleshooting, Hardening.

---

## Service-Units (produktionsreif)

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=MyApp Web Service
Documentation=https://docs.firma.de/myapp
After=network-online.target postgresql.service
Wants=network-online.target
Requires=postgresql.service
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=exec
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp

# Umgebungsvariablen
Environment=NODE_ENV=production
Environment=PORT=3000
EnvironmentFile=-/etc/myapp/environment   # - = optional (kein Fehler wenn nicht vorhanden)

# Ausführung
ExecStartPre=/usr/local/bin/myapp --check-config
ExecStart=/usr/local/bin/myapp server
ExecReload=/bin/kill -HUP $MAINPID
ExecStopPost=/usr/local/bin/myapp cleanup

# Neustart
Restart=on-failure
RestartSec=5s
TimeoutStartSec=30
TimeoutStopSec=30

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

# Ressourcen
LimitNOFILE=65536
LimitNPROC=4096
MemoryMax=2G
CPUQuota=200%           # Maximal 2 CPU-Kerne

# ── HARDENING ──────────────────────────────────────────────────────────────
NoNewPrivileges=yes
PrivateTmp=yes
PrivateDevices=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/lib/myapp /var/log/myapp
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectKernelLogs=yes
ProtectControlGroups=yes
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX
RestrictNamespaces=yes
RestrictRealtime=yes
RestrictSUIDSGID=yes
RemoveIPC=yes
CapabilityBoundingSet=CAP_NET_BIND_SERVICE   # Nur wenn Port < 1024
AmbientCapabilities=CAP_NET_BIND_SERVICE
SystemCallFilter=@system-service
SystemCallErrorNumber=EPERM
LockPersonality=yes
MemoryDenyWriteExecute=yes
ProtectHostname=yes
ProtectClock=yes

[Install]
WantedBy=multi-user.target
```

---

## systemd Timer (Cron-Ersatz)

```ini
# /etc/systemd/system/cleanup.service
[Unit]
Description=Cleanup alte Log-Dateien
After=local-fs.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/cleanup-logs.sh
User=root
StandardOutput=journal
```

```ini
# /etc/systemd/system/cleanup.timer
[Unit]
Description=Täglicher Cleanup um 03:00 Uhr

[Timer]
OnCalendar=*-*-* 03:00:00
RandomizedDelaySec=600    # Bis zu 10 Min zufällige Verschiebung
Persistent=true           # Nachgeholt wenn System offline war
AccuracySec=1min

[Install]
WantedBy=timers.target
```

```bash
# Timer aktivieren und verwalten
systemctl daemon-reload
systemctl enable --now cleanup.timer

# Status und nächste Ausführung
systemctl list-timers --all
systemctl status cleanup.timer

# Manuell auslösen (für Tests)
systemctl start cleanup.service
journalctl -u cleanup.service -n 50
```

---

## journald — Log-Verwaltung

```bash
# ─── BASIS-ABFRAGEN ───────────────────────────────────────────────────────────
journalctl -u nginx                        # Logs eines Services
journalctl -u nginx -f                     # Live-Follow
journalctl -u nginx --since "2h ago"       # Letzte 2 Stunden
journalctl -u nginx --since "2024-01-15 08:00" --until "2024-01-15 10:00"
journalctl -p err -n 100                   # Letzte 100 Fehler
journalctl -p err..crit                    # err, crit, alert, emerg
journalctl -k                              # Nur Kernel-Meldungen (dmesg)
journalctl _SYSTEMD_UNIT=nginx.service _COMM=nginx  # Kombinierte Filter

# ─── FORMAT & AUSGABE ────────────────────────────────────────────────────────
journalctl -u myapp -o json-pretty         # JSON-Ausgabe
journalctl -u myapp -o short-precise       # Mit Mikrosekunden
journalctl --no-pager -u myapp | grep -i error  # Pipen

# ─── SPEICHER-MANAGEMENT ─────────────────────────────────────────────────────
journalctl --disk-usage                    # Aktueller Speicherverbrauch
journalctl --vacuum-size=1G               # Auf 1GB begrenzen
journalctl --vacuum-time=30d              # Älter als 30 Tage löschen

# ─── JOURNALD-KONFIGURATION ──────────────────────────────────────────────────
cat << 'JOURNALD' > /etc/systemd/journald.conf.d/99-limits.conf
[Journal]
Storage=persistent
Compress=yes
SystemMaxUse=2G
SystemKeepFree=500M
MaxFileSec=1month
ForwardToSyslog=no
RateLimitInterval=30s
RateLimitBurst=10000
JOURNALD
systemctl restart systemd-journald
```

---

## Targets und Boot-Management

```bash
# ─── TARGETS ─────────────────────────────────────────────────────────────────
systemctl get-default                      # Aktuelles Default-Target
systemctl set-default multi-user.target    # Ohne GUI starten
systemctl set-default graphical.target     # Mit GUI starten
systemctl isolate rescue.target            # Rescue-Modus (sofort)

# ─── BOOT-ANALYSE ────────────────────────────────────────────────────────────
systemd-analyze                            # Gesamte Boot-Zeit
systemd-analyze blame                      # Langsamste Units
systemd-analyze critical-chain             # Kritischer Pfad
systemd-analyze plot > boot.svg            # Grafische Boot-Analyse

# ─── SERVICE-MANAGEMENT ──────────────────────────────────────────────────────
systemctl list-units --failed              # Fehlgeschlagene Units
systemctl list-units --state=activating    # Startende Units (hängend?)
systemctl list-dependencies nginx          # Abhängigkeitsbaum
systemctl show nginx --property=MainPID    # Einzelne Properties
systemctl cat nginx                        # Unit-Datei anzeigen
systemctl edit nginx                       # Drop-In erstellen (Override)
```

---

## Troubleshooting-Checkliste

```bash
# Service startet nicht?
systemctl status myapp.service -l          # Detaillierter Status
journalctl -u myapp.service -n 100 --no-pager  # Letzte Logs
journalctl -xe                             # Systemweite Fehler

# Häufige Ursachen:
# 1. Berechtigungen: ls -la /opt/myapp /var/lib/myapp
# 2. SELinux: ausearch -m AVC -ts recent | audit2why
# 3. Port belegt: ss -tlnp | grep :3000
# 4. Fehlende Abhängigkeiten: systemctl status postgresql
# 5. Config-Syntax: /usr/local/bin/myapp --check-config

# systemd-analyze security (Hardening-Score)
systemd-analyze security myapp.service
# Output: Zeigt Security-Score + fehlende Härtungsoptionen

# Resource-Limits prüfen
systemctl show myapp -p LimitNOFILE,MemoryMax,CPUQuota
cat /proc/$(systemctl show myapp -p MainPID --value)/limits
```

---

## Drop-In Overrides (Unit überschreiben ohne Original zu ändern)

```bash
# Interaktiv (empfohlen)
systemctl edit nginx.service
# Erstellt /etc/systemd/system/nginx.service.d/override.conf

# Manuell
mkdir -p /etc/systemd/system/nginx.service.d/
cat << 'OVERRIDE' > /etc/systemd/system/nginx.service.d/override.conf
[Service]
# Umgebungsvariable hinzufügen
Environment=NGINX_ENVID=prod

# Timeout erhöhen
TimeoutStartSec=60

# Zusätzliche Härtung
PrivateTmp=yes
NoNewPrivileges=yes
OVERRIDE
systemctl daemon-reload
systemctl restart nginx
```
