# Eval-Testfälle: Linux Sysadmin Skill

## Kategorie 1: Trigger-Tests

### T001 — SSH-Härtung
**Input:** "Härte SSH auf einem Ubuntu 22.04 Server ab"
**Kriterien:**
- ✅ PermitRootLogin no gesetzt
- ✅ PasswordAuthentication no gesetzt
- ✅ Nur Key-Authentifizierung
- ✅ AllowGroups oder AllowUsers gesetzt
- ✅ sshd -t vor Neustart empfohlen
- ✅ Starke Kex-Algorithmen (curve25519)

### T002 — Bash-Script mit Fehlerbehandlung
**Input:** "Schreibe ein Bash-Script das alle inaktiven User (90+ Tage) deaktiviert"
**Kriterien:**
- ✅ set -euo pipefail am Anfang
- ✅ Logging-Funktion vorhanden
- ✅ Root-Check
- ✅ WhatIf/DryRun-Modus oder Bestätigung
- ✅ trap cleanup EXIT
- ✅ Keine Hardcoded-Pfade

### T003 — Systemd-Service erstellen
**Input:** "Erstelle einen Systemd-Service für eine Node.js-Anwendung"
**Kriterien:**
- ✅ [Unit], [Service], [Install] Sektionen vollständig
- ✅ User= gesetzt (kein root)
- ✅ Restart=always oder on-failure
- ✅ Security-Härtung: NoNewPrivileges, PrivateTmp, ProtectSystem
- ✅ Logging: StandardOutput=journal
- ✅ systemctl daemon-reload Hinweis

### T004 — LVM-Erweiterung
**Input:** "Mein /var Filesystem ist voll. Wie erweitere ich es mit LVM?"
**Kriterien:**
- ✅ pvdisplay/vgdisplay/lvdisplay zur Diagnose
- ✅ lvextend -L +Xg /dev/vg/lv
- ✅ resize2fs für ext4 oder xfs_growfs für XFS
- ✅ df -h zur Verifikation
- ✅ Kein Neustart nötig erwähnt

### T005 — Ansible Playbook
**Input:** "Erstelle ein Ansible Playbook das nginx auf allen Webservern installiert und konfiguriert"
**Kriterien:**
- ✅ Korrektes YAML-Format
- ✅ become: yes für Root-Operationen
- ✅ package-Modul (nicht apt direkt) für Distro-Unabhängigkeit
- ✅ Handler für nginx reload
- ✅ Idempotent (state: present, not latest)

### T006 — Performance-Diagnose
**Input:** "Mein Linux-Server ist langsam. Wie diagnostiziere ich das Problem?"
**Kriterien:**
- ✅ CPU: top/htop, sar
- ✅ Memory: free -h, vmstat
- ✅ Disk I/O: iostat -x, iotop
- ✅ Netzwerk: ss -tulnp, netstat
- ✅ Load Average interpretiert
- ✅ Systematischer Ansatz (nicht random Befehle)

### T007 — Firewall-Konfiguration (ufw)
**Input:** "Richte ufw auf einem Ubuntu-Server ein. Nur SSH und HTTPS erlauben"
**Kriterien:**
- ✅ ufw default deny incoming
- ✅ ufw default allow outgoing
- ✅ SSH nur von internem Netz
- ✅ ufw enable
- ✅ ufw status verbose zur Verifikation

### T008 — Distro-Unterschied
**Input:** "Wie installiere ich automatische Sicherheitsupdates auf RHEL 9?"
**Kriterien:**
- ✅ dnf-automatic (nicht unattended-upgrades)
- ✅ /etc/dnf/automatic.conf anpassen
- ✅ apply_updates = yes für Security-Updates
- ✅ systemctl enable --now dnf-automatic.timer
- ✅ Unterschied zu Ubuntu erwähnt

## Kategorie 2: Qualitäts-Tests

### Q001 — Script-Qualität
**Kriterien für jedes generierte Bash-Script:**
- ✅ #!/usr/bin/env bash
- ✅ set -euo pipefail
- ✅ Kommentare auf Deutsch oder Englisch
- ✅ Keine Passwörter hartcodiert
- ✅ Exit-Codes korrekt verwendet

### Q002 — Distro-Awareness
**Input:** "Installiere Docker"
**Kriterien:**
- ✅ Unterschied Ubuntu vs RHEL erklärt
- ✅ Kein universeller "apt install docker" ohne Kontext
- ✅ Offizielles Docker-Repository empfohlen

## Kategorie 3: Negativ-Tests

### N001 — Nicht-Linux-Thema
**Input:** "Konfiguriere einen Cisco Switch"
**Erwartetes Verhalten:**
- ❌ Linux Skill nicht primär zuständig
- ✅ Network Engineering Skill empfehlen

### N002 — Windows-Frage
**Input:** "Wie erstelle ich einen GPO in Active Directory?"
**Erwartetes Verhalten:**
- ❌ Linux Skill nicht zuständig
- ✅ M365 Admin Skill empfehlen

## Bewertungsschema
| Score | Bedeutung |
|-------|-----------|
| 5/5 | Alle Kriterien, exzellente Script-Qualität |
| 4/5 | Fast alle Kriterien, kleine Lücken |
| 3/5 | Grundanforderungen, Verbesserungsbedarf |
| 2/5 | Wesentliche Kriterien fehlen |
| 1/5 | Skill hat nicht korrekt ausgelöst |

**Mindest-Score:** 4/5 auf alle T-Tests
