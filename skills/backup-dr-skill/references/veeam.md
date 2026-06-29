# Veeam Backup & Replication — Referenzmodul

Veeam B&R v12.x — Konfiguration, PowerShell-Automatisierung, Härtung, Troubleshooting.

---

## Architektur-Komponenten

```
┌─────────────────────────────────────────────────────┐
│  VEEAM BACKUP & REPLICATION SERVER                  │
│  (Orchestrierung, Jobsteuerung, Datenbank SQL)      │
├─────────────────────────────────────────────────────┤
│  VEEAM BACKUP PROXY                                 │
│  (Datenbewegung: VMware/Hyper-V → Repository)       │
│  Transportmodi: Direct SAN / Virtual Appliance /    │
│  Network (NBD) / Backup from Storage Snapshots      │
├─────────────────────────────────────────────────────┤
│  VEEAM BACKUP REPOSITORY                            │
│  Typen:                                             │
│  • Windows/Linux Direct-Attached Storage            │
│  • Scale-Out Backup Repository (SOBR)               │
│  • Deduplication Appliances (ExaGrid, Quantum)      │
│  • Object Storage (S3/Azure Blob/Wasabi)            │
│  • Hardened Linux Repository (Immutable)            │
├─────────────────────────────────────────────────────┤
│  VEEAM ENTERPRISE MANAGER (optional)                │
│  Self-Service Restore, Lizenzverwaltung, Reporting  │
└─────────────────────────────────────────────────────┘
```

---

## Hardened Linux Repository (Immutable Backups)

```bash
# Veeam Hardened Repository — Ubuntu 22.04 LTS Setup

# 1. Dedizierter lokaler Benutzer (KEIN sudo, KEIN SSH-Key-Login)
adduser --no-create-home --shell /bin/bash veeamrepo
passwd veeamrepo  # Starkes Passwort setzen

# 2. XFS-Dateisystem mit reflink (Veeam-Voraussetzung für Immutability)
mkfs.xfs -b size=4096 -m reflink=1,crc=1 /dev/sdb
mkdir -p /backup/repo1
echo '/dev/sdb /backup/repo1 xfs defaults 0 0' >> /etc/fstab
mount -a

# 3. Verzeichnisrechte: nur veeamrepo darf schreiben
chown veeamrepo:veeamrepo /backup/repo1
chmod 700 /backup/repo1

# 4. SSH-Härtung: nur Passwort-Auth für veeamrepo
# /etc/ssh/sshd_config.d/veeam-hardened.conf
cat << 'EOF' > /etc/ssh/sshd_config.d/veeam-hardened.conf
Match User veeamrepo
    PasswordAuthentication yes
    PubkeyAuthentication no
    AllowTcpForwarding no
    X11Forwarding no
EOF
systemctl reload sshd

# 5. Immutability-Zeitraum: in Veeam GUI einstellen
# Repository → Edit → Storage → "Make recent backups immutable for X days"
# Empfehlung: mind. 7 Tage (besser 14-30 Tage)

# 6. Single-Use Credentials: Veeam GUI
# Repository → Edit → Credentials → "Use single-use credentials for hardened repository"
```

---

## PowerShell-Automatisierung (Veeam v12)

```powershell
#Requires -Modules Veeam.Backup.PowerShell
# Verbindung zu Veeam B&R Server
Connect-VBRServer -Server "veeam-srv01" -Credential (Get-Credential)

# ─── JOB-ÜBERSICHT ───────────────────────────────────────────────────────────
function Get-VBRJobReport {
    Get-VBRJob | ForEach-Object {
        $session = $_.FindLastSession()
        [PSCustomObject]@{
            Name        = $_.Name
            Typ         = $_.JobType
            Aktiviert   = $_.IsScheduleEnabled
            LetzterRun  = if ($session) { $session.CreationTime } else { "Nie" }
            Dauer       = if ($session) { $session.Progress.Duration } else { "-" }
            Status      = if ($session) { $session.Result } else { "Unbekannt" }
            Transferiert = if ($session) { 
                [math]::Round($session.Progress.TransferedSize / 1GB, 2).ToString() + " GB"
            } else { "-" }
        }
    } | Sort-Object Status, Name | Format-Table -AutoSize
}

# ─── FEHLERBERICHT + EMAIL-ALERT ─────────────────────────────────────────────
function Send-VBRFailureAlert {
    param(
        [string]$SmtpServer = "mail.contoso.de",
        [string]$To = "it-admin@contoso.de",
        [string]$From = "veeam-alert@contoso.de"
    )
    $failed = Get-VBRJob | Where-Object {
        $_.FindLastSession().Result -in @('Failed', 'Warning')
    }
    if ($failed) {
        $body = $failed | ForEach-Object {
            $s = $_.FindLastSession()
            "Job: $($_.Name) | Status: $($s.Result) | Zeit: $($s.CreationTime)"
        } | Out-String
        Send-MailMessage -SmtpServer $SmtpServer -To $To -From $From `
            -Subject "⚠️ Veeam Backup-Fehler: $($failed.Count) Job(s) fehlgeschlagen" `
            -Body $body -Encoding UTF8
    }
}

# ─── REPOSITORY-KAPAZITÄTSALERT ──────────────────────────────────────────────
function Get-VBRRepositoryCapacity {
    param([int]$WarnungProzent = 80)
    Get-VBRBackupRepository | ForEach-Object {
        $c = $_.GetContainer()
        $gesamt = $c.CachedTotalSpace.InGigabytes
        $frei   = $c.CachedFreeSpace.InGigabytes
        $belegt = if ($gesamt -gt 0) { [math]::Round((1 - $frei/$gesamt)*100, 1) } else { 0 }
        [PSCustomObject]@{
            Repository  = $_.Name
            GesamtGB    = [math]::Round($gesamt, 1)
            FreiGB      = [math]::Round($frei, 1)
            BelegtProzent = $belegt
            Status      = if ($belegt -ge $WarnungProzent) { "⚠️ WARNUNG" } else { "✅ OK" }
        }
    } | Format-Table -AutoSize
}

# ─── RESTORE-POINT-ÜBERSICHT ─────────────────────────────────────────────────
function Get-VBRRestorePoints {
    param([string]$VMName = "*", [int]$LetzteN = 10)
    Get-VBRRestorePoint -Name $VMName |
        Sort-Object CreationTime -Descending |
        Select-Object -First $LetzteN |
        Select-Object Name, CreationTime,
            @{N='GrößeGB'; E={[math]::Round($_.ApproxSize/1GB, 2)}},
            @{N='Typ'; E={$_.Algorithm}},
            @{N='Repository'; E={$_.FindChain().TargetRepository.Name}} |
        Format-Table -AutoSize
}

# ─── AUTOMATISCHER RESTORE-TEST (Sandbox / vSphere) ──────────────────────────
function Start-VBRSandboxRestoreTest {
    param(
        [Parameter(Mandatory)][string]$VMName,
        [string]$SandboxNetwork = "Sandbox-VLAN100"
    )
    $rp = Get-VBRRestorePoint -Name $VMName | Sort-Object CreationTime -Descending | Select-Object -First 1
    if (-not $rp) { Write-Error "Kein Restore-Point für '$VMName' gefunden"; return }
    
    Write-Host "Starte Sandbox-Restore: $VMName (Punkt: $($rp.CreationTime))" -ForegroundColor Cyan
    $session = Start-VBRRestoreVM -RestorePoint $rp -Reason "Automatischer Restore-Test" `
        -ToOriginalLocation:$false -RunAsync
    Write-Host "Session ID: $($session.Id) — Überwache in Veeam GUI" -ForegroundColor Yellow
}

Disconnect-VBRServer
```

---

## Scale-Out Backup Repository (SOBR) Konfiguration

```powershell
# SOBR mit Performance-Tier (lokal) + Capacity-Tier (S3/Azure)

# 1. Extent-Repositories definieren (bereits in Veeam angelegt)
$extents = @(
    Get-VBRBackupRepository -Name "Repo-Local-01",
    Get-VBRBackupRepository -Name "Repo-Local-02"
)

# 2. Object Storage (Capacity-Tier) konfigurieren
# In Veeam GUI: Backup Infrastructure → Object Storage Repositories → Add
# Typ: S3 Compatible, Azure Blob, oder AWS S3
# Für Wasabi (DE-Region):
# URL: s3.eu-central-2.wasabisys.com
# Bucket: backup-offsite-[firma]

# 3. SOBR erstellen (PowerShell)
$capacityTier = Get-VBRObjectStorageRepository -Name "Wasabi-EU-Offsite"
Add-VBRScaleOutBackupRepository `
    -Name "SOBR-Produktion" `
    -Extent $extents `
    -PolicyType DataLocality `
    -EnableCapacityTier `
    -CapacityTierRepository $capacityTier `
    -OffloadAfterDays 14 `
    -EncryptCapacityTier
```

---

## Immutability-Matrix (Übersicht)

| Storage-Typ | Immutability-Methode | Veeam-Unterstützung | Empfehlung |
|---|---|---|---|
| Hardened Linux Repo | `chattr +i` / Veeam-intern | ✅ Nativ | ★★★★★ Ransomware-sicher |
| AWS S3 | Object Lock (WORM) | ✅ Nativ | ★★★★★ |
| Azure Blob | Immutability Policy | ✅ Nativ | ★★★★★ |
| Wasabi S3 | Object Lock (WORM) | ✅ Nativ | ★★★★☆ (günstiger als AWS) |
| ExaGrid | ShieldStore (retention lock) | ✅ Nativ | ★★★★☆ |
| Quantum DXi | Deduplizierung + Lock | ✅ Nativ | ★★★★☆ |
| Windows Repo | Keine native Immutability | ❌ | ✗ nicht für Immutable nutzen |
| SMB/NAS Share | Keine native Immutability | ❌ | ✗ nicht für Immutable nutzen |

---

## Troubleshooting häufige Veeam-Fehler

```
FEHLER: "Failed to connect to host" (VMware Transport)
→ Prüfe: Veeam Proxy Service läuft? (services.msc)
→ Prüfe: Firewall Port 902 (VMware) und 6162 (Veeam) offen?
→ Prüfe: vCenter-Credentials aktuell?
→ Fix: Backup Infrastructure → Managed Servers → Rescan

FEHLER: "Repository is out of space"
→ Sofortmaßnahme: Redundante Restore-Points löschen (Get-VBRRestorePoint | Sort CreationTime | Select -First N | Remove-VBRRestorePoint)
→ Langfristig: SOBR mit Capacity-Tier einrichten

FEHLER: "Unable to truncate SQL Server transaction logs"
→ Ursache: SQL-Agent-Konto ohne sysadmin-Rechte ODER VSS-Fehler
→ Fix: SQL-Job-Credentials prüfen, VSS-Writer-Status: vssadmin list writers

FEHLER: Backup-Job läuft, aber Immutability greift nicht
→ Prüfe: XFS mit reflink=1 formatiert? (xfs_info /backup/repo1 | grep reflink)
→ Prüfe: Veeam-Transport-Service läuft auf Linux-Repo?
→ Prüfe: Immutability-Zeitraum in Repository-Einstellungen gesetzt?

FEHLER: "GFS (Grandfather-Father-Son) Restore-Points fehlen"
→ Prüfe GFS-Konfiguration im Job: Weekly/Monthly/Yearly Restore Points aktiviert?
→ SOBR: GFS läuft nur auf Capacity-Tier (Object Storage), nicht auf Performance-Tier
```
