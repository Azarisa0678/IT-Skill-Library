---
name: backup-dr-skill
description: >
  Backup und Disaster Recovery fuer IT-Administratoren.
  3-2-1-1-0-Regel, RPO/RTO, Business Impact Analysis (BIA),
  Veeam Backup Replication v12, SOBR, Hardened Repository, Immutability,
  Azure Backup, Azure Site Recovery (ASR), AWS Backup, Vault Lock,
  Wasabi S3 Object Lock, Cloud-Backup DSGVO, restic, BorgBackup, rsync,
  Bacula, Bareos, systemd Timer, Backup-Monitoring,
  DR-Strategien, Cold Warm Hot Standby, Active-Active,
  DR-Testplanung, BCP, Notfallhandbuch, Ransomware-Schutz Backup,
  Immutable Storage, Air-Gap, Backup-Netzwerksegmentierung.
  Outputs: Backup-Konzepte (DE), PowerShell/Bash-Skripte, Checklisten, DR-Testprotokolle.
---

# Backup & Disaster Recovery Skill

## Guiding Principles

**Backup ist kein Backup ohne Restore-Test.** Jede Strategie enthält Testfrequenzen und Dokumentationspflichten.
**3-2-1-1-0 ist das Minimum.** 3 Kopien, 2 Medien, 1 offsite, 1 offline/air-gapped, 0 Fehler beim letzten Test.
**Ransomware-Resilienz ist Pflicht.** Immutable Storage oder Offline-Backup müssen immer Teil der Strategie sein.
**RPO und RTO definieren alles.** Ohne klare Ziele ist kein Backup-Konzept vollständig.
**Reihenfolge beim Restore:** Active Directory / DNS → Core-Infrastruktur → Applikationen → Dateiserver.

---

## Referenzmodule

Lade das passende Modul je nach Thema:

- `references/veeam.md` — Veeam B&R v12, PowerShell, SOBR, Hardened Repo, Troubleshooting
- `references/cloud-backup.md` — Azure Backup, AWS Backup, Wasabi S3, DSGVO-Checkliste
- `references/linux-backup.md` — restic, BorgBackup, rsync, Bareos, systemd Timer, Monitoring
- `references/dr-bcp.md` — DR-Strategien, BIA, ASR, DR-Testplan, BCP-Vorlage, Notfallhandbuch

Routing:
- Veeam / PowerShell-Automatisierung → `veeam.md`
- Azure Backup / AWS Backup / Wasabi / Cloud → `cloud-backup.md`
- Linux, restic, Borg, rsync, Bacula/Bareos → `linux-backup.md`
- DR-Test, BCP, RTO/RPO-Strategie, Notfallhandbuch, BIA → `dr-bcp.md`
- Ransomware-Schutz, Immutability → Inline unten + `veeam.md` + `cloud-backup.md`

---

## RPO/RTO-Tier-Klassifizierung

| Tier | Systeme | RPO | RTO | Backup-Typ | Mindesttest |
|------|---------|-----|-----|------------|-------------|
| **1 — Kritisch** | AD, ERP, Datenbank, E-Mail | ≤ 1 Std | ≤ 4 Std | Stündlich inkrementell | Quartalsweise VM-Restore |
| **2 — Wichtig** | Dateiserver, CRM, Webserver | ≤ 4 Std | ≤ 8 Std | 4x täglich inkrementell | Halbjährlich |
| **3 — Standard** | Dev-Systeme, Intranet | ≤ 24 Std | ≤ 48 Std | Täglich inkrementell | Jährlich |
| **4 — Archiv** | Historische Daten, Logs | Wöchentlich | ≤ 72 Std | Wöchentlich voll | Jährlich |

---

## 3-2-1-1-0-Regel — Architekturbeispiele

```
BEISPIEL 1: KMU mit Veeam + NAS + Cloud
  Kopie 1: Veeam → lokales Repository (NAS/Festplatte)
  Kopie 2: Veeam → Hardened Linux Repository (immutable, gleiches RZ)
  Kopie 3: Veeam SOBR Capacity-Tier → Wasabi S3 EU (Object Lock, offsite)
  Offline:  Monatlich: Tape oder USB-Rotation, Aufbewahrung extern
  0 Fehler: Tägliche Job-Prüfung + monatlicher Restore-Test

BEISPIEL 2: Azure-native
  Kopie 1: Azure Backup (Recovery Services Vault, GRS)
  Kopie 2: Azure Backup → zweite Region (Cross-Region Restore aktiviert)
  Kopie 3: Veeam Agent → on-premises NAS (für unabhängige Kopie)
  Offline:  Azure Soft Delete (14 Tage) + Immutable Vault
  0 Fehler: Monatlicher Test-Failover (Azure Site Recovery)

BEISPIEL 3: Linux-Server mit restic
  Kopie 1: restic → lokale externe Festplatte
  Kopie 2: restic → NAS (Netzwerk)
  Kopie 3: restic → Wasabi S3 EU (verschlüsselt, Object Lock)
  Offline:  Wöchentliche USB-Rotation, extern aufbewahrt
  0 Fehler: Wöchentlicher Einzeldatei-Restore-Test (automatisiert)
```

---

## Veeam PowerShell — Schnellreferenz

```powershell
Import-Module Veeam.Backup.PowerShell -DisableNameChecking

# Job-Übersicht mit Status
Get-VBRJob | Select-Object Name, JobType,
    @{N='LetzterRun';  E={$_.FindLastSession().CreationTime}},
    @{N='Status';       E={$_.FindLastSession().Result}},
    @{N='DauerMin';     E={[math]::Round($_.FindLastSession().Progress.Duration.TotalMinutes,0)}},
    @{N='TransferGB';   E={[math]::Round($_.FindLastSession().Progress.TransferedSize/1GB,2)}} |
    Sort-Object Status | Format-Table -AutoSize

# Fehlgeschlagene Jobs → Email-Alert
$failed = Get-VBRJob | Where-Object { $_.FindLastSession().Result -in 'Failed','Warning' }
if ($failed) {
    $body = $failed | ForEach-Object {
        "Job: $($_.Name) | Status: $($_.FindLastSession().Result) | $($_.FindLastSession().CreationTime)"
    } | Out-String
    Send-MailMessage -SmtpServer "mail.firma.de" -To "it@firma.de" -From "veeam@firma.de" `
        -Subject "Veeam Backup-Fehler: $($failed.Count) Job(s)" -Body $body -Encoding UTF8
}

# Repository-Auslastung
Get-VBRBackupRepository | Select-Object Name,
    @{N='GesamtGB';E={[math]::Round($_.GetContainer().CachedTotalSpace.InGigabytes,1)}},
    @{N='FreiGB';  E={[math]::Round($_.GetContainer().CachedFreeSpace.InGigabytes,1)}},
    @{N='Belegt%'; E={[math]::Round((1-$_.GetContainer().CachedFreeSpace.InGigabytes/
        $_.GetContainer().CachedTotalSpace.InGigabytes)*100,1)}} | Format-Table -AutoSize

# Restore-Points für eine VM
Get-VBRRestorePoint -Name "vm-erp-01" | Sort-Object CreationTime -Descending |
    Select-Object -First 10 | Select-Object Name, CreationTime,
    @{N='GrößeGB'; E={[math]::Round($_.ApproxSize/1GB,2)}}, @{N='Typ'; E={$_.Algorithm}}
```

→ Für Hardened Repository, SOBR, Sandbox-Restore-Test: `references/veeam.md` lesen.

---

## Linux Backup — Schnellreferenz

```bash
# restic: Backup zu Wasabi S3
export RESTIC_REPOSITORY="s3:https://s3.eu-central-2.wasabisys.com/backup-prod"
export RESTIC_PASSWORD_FILE="/etc/restic/password"
restic backup /etc /home /var/lib --exclude /tmp --exclude /var/tmp
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune
restic check                         # Metadata-Integrität
restic snapshots                     # Übersicht
restic restore latest --target /tmp/restore --include "/etc/nginx/nginx.conf"

# BorgBackup: lokales/SSH-Remote Backup
export BORG_REPO="ssh://backup@nas:22/backup/$(hostname)"
borg create "$BORG_REPO::$(hostname)-{now:%Y%m%d}" /etc /home /var/lib \
    --compression lz4 --exclude '/tmp' --exclude '/var/cache'
borg prune --keep-daily 7 --keep-weekly 4 --keep-monthly 12 "$BORG_REPO"
borg check --verify-data "$BORG_REPO"
```

→ Für vollständige Skripte, systemd Timer, Bareos, Monitoring: `references/linux-backup.md` lesen.

---

## Ransomware-Schutz Backup — Maßnahmenkatalog

| Maßnahme | Schutzwirkung | Umsetzung |
|---|---|---|
| **Immutable Storage** | Backup kann nicht verschlüsselt/gelöscht werden | Veeam Hardened Repo / S3 Object Lock |
| **Air-Gap / Offline** | Physische Trennung, kein Netzwerkzugang möglich | Tape, USB-Rotation, extern aufbewahrt |
| **Backup-Netz segmentieren** | Ransomware erreicht Backup-Server nicht | Eigenes VLAN, Firewall Default Deny → Backup |
| **Separate Credentials** | Backup-Zugangsdaten nicht im AD | Lokales Konto, PAM-Vault, kein Domain-Join |
| **Backup-Monitoring** | Früherkennung wenn Jobs stillgelegt werden | Alert bei Job-Stillstand > X Stunden |
| **Retention: mind. 30 Tage** | Infektion vor Entdeckung → ältere Kopien nutzbar | GFS: Daily 30 / Weekly 12 / Monthly 12 |
| **Regelmäßige Restore-Tests** | Stellt sicher dass Backups tatsächlich nutzbar | Monatlich Einzeldatei, Quartalsweise VM |

```
NETZWERKARCHITEKTUR — Ransomware-resilienter Backup:

Produktionsnetz (VLAN 10)
    │
    │  [Firewall: VLAN 10 → VLAN 50 nur Backup-Port 9443]
    ▼
Backup-Netz (VLAN 50)          ← Backup-Proxy + Backup-Server hier
    │
    │  [Hardened Linux Repo: kein SSH-Key, nur Passwort, single-use credentials]
    ▼
Backup-Repository
    │
    └── Offsite: Wasabi S3 EU (Object Lock) oder Tape extern
```

---

## DR-Test-Protokoll — Kurzform

```markdown
# DR-Test-Protokoll
Test-Datum: ___________  System: ___________  Restore-Point: ___________
Testleiter: ___________  RPO-Ziel: ___________  RTO-Ziel: ___________

| Schritt | Soll | Ist | OK? |
|---------|------|-----|-----|
| Backup-Satz gefunden | < 5 Min | | [ ] |
| Restore gestartet | < 15 Min | | [ ] |
| VM/System gestartet | < 30 Min | | [ ] |
| Applikation verfügbar | < 60 Min | | [ ] |
| Datenverlust geprüft | Max RPO | | [ ] |
| Gesamtdauer | Max RTO | | [ ] |

Mängel: ___________  Nächster Test: ___________
Unterschrift: ___________  Datum: ___________
```

→ Vollständiges DR-Testprotokoll, BCP-Vorlage, Notfallhandbuch: `references/dr-bcp.md` lesen.

---

## Backup-Compliance-Checkliste (DE)

```markdown
## Backup & DR Compliance — [Unternehmen] — [Datum]

### Strategie & Dokumentation
- [ ] Backup-Konzept vorhanden und aktuell (< 1 Jahr)
- [ ] RPO/RTO für alle Systeme (Tier 1-4) definiert und dokumentiert
- [ ] 3-2-1-1-0-Regel umgesetzt und nachweisbar
- [ ] Backup-Konzept von IT-Leitung / Geschäftsführung genehmigt

### Technische Umsetzung
- [ ] Alle kritischen Systeme (Tier 1+2) täglich gesichert
- [ ] Immutable Backup vorhanden (Hardened Repo / S3 Object Lock)
- [ ] Offline/Air-Gap-Backup vorhanden (Tape oder USB-Rotation extern)
- [ ] Backup-Netz segmentiert (eigenes VLAN, Firewall-gesichert)
- [ ] Backup-Credentials: separate Konten, kein Domain-Join des Backup-Servers
- [ ] Verschlüsselung: Backup-Jobs verschlüsselt (Veeam-Job-Verschlüsselung oder restic)

### Monitoring & Alerting
- [ ] Tägliche automatische Prüfung aller Backup-Jobs
- [ ] Alert bei Backup-Fehler: E-Mail / SIEM-Ticket < 1 Stunde
- [ ] Repository-Auslastung überwacht (Alert bei > 80%)
- [ ] Backup-Stillstand-Alert (kein Job > 25h bei täglichem Backup)

### Restore-Tests
- [ ] Monatlich: Einzeldatei-Restore (mind. 1 Tier-1-System)
- [ ] Quartalsweise: VM/Server-Vollrestore (mind. 1 Tier-1-System)
- [ ] Jährlich: Vollständiger DR-Test (alle Tier-1-Systeme)
- [ ] Alle Tests protokolliert und aufbewahrt

### DR & BCP
- [ ] DR-Plan vorhanden und aktuell
- [ ] Business Continuity Plan (BCP) vorhanden
- [ ] Notfallhandbuch: gedruckt und physisch zugänglich (Notfallkoffer!)
- [ ] Wiederherstellungsreihenfolge dokumentiert (AD → Core → Apps)
- [ ] Notfall-Kontaktliste aktuell (intern + extern)
- [ ] BCP-Tabletop-Übung mindestens jährlich
```

---

## Output-Formate je Aufgabentyp

| Aufgabe | Format |
|---------|--------|
| Backup-Konzept KMU/Enterprise | Strukturiertes Dokument: Einleitung, Tier-Klassifizierung, Technologie, Monitoring, Testplan |
| Veeam PowerShell | Vollständiges .ps1 mit Set-StrictMode, Error-Handling, Kommentaren |
| Linux Backup-Skript | Vollständiges Bash mit set -euo pipefail, Logging, E-Mail-Alert |
| Azure/AWS Backup | PowerShell oder CLI-Befehle mit allen Schritten, Policy-Konfiguration |
| DR-Test-Protokoll | Ausfüllbare Markdown-Vorlage |
| Checkliste | Markdown-Checkboxen, gruppiert, DE |
| Notfallhandbuch | Vollstrukturiertes Dokument mit Kontakten, Eskalation, Restore-Prozeduren |
