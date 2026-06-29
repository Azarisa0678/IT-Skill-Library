# Cloud Backup — Referenzmodul

Azure Backup, AWS Backup, Wasabi, S3-kompatible Lösungen — Konfiguration, Automatisierung,
Kostenoptimierung, DSGVO-Konformität.

---

## Azure Backup

### Architektur-Übersicht

```
Azure Backup Service
    │
    ├── Recovery Services Vault (klassisch, Blob-basiert)
    │   ├── Azure VM Backup (Snapshot + Blob-Transfer)
    │   ├── SQL Server in Azure VM
    │   ├── Azure Files (File Share Backup)
    │   └── On-Premises (MARS Agent, MABS, DPM)
    │
    └── Backup Vault (neuere Ressource, für Disks/Blobs/AKS)
        ├── Azure Disk Backup (inkrementell, Snapshot-basiert)
        ├── Azure Blob Backup (operational backup)
        └── AKS Backup (Kubernetes-Workloads)
```

### Azure VM Backup — PowerShell

```powershell
# Azure VM Backup — vollständige Einrichtung per PowerShell

# 1. Recovery Services Vault erstellen
$rg     = "rg-backup-prod"
$vault  = "rsv-backup-westeurope"
$location = "westeurope"

New-AzRecoveryServicesVault -Name $vault -ResourceGroupName $rg -Location $location

# Vault-Kontext setzen
$vaultObj = Get-AzRecoveryServicesVault -Name $vault -ResourceGroupName $rg
Set-AzRecoveryServicesVaultContext -Vault $vaultObj

# 2. Backup-Policy erstellen (täglich 23:00 Uhr, 30 Tage Retention)
$schemaVersion = "V2"
$retentionDays = 30
$backupTime    = "2000-01-01T23:00:00Z"  # UTC

$schedulePolicy = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType AzureVM -BackupManagementType AzureVM -PolicySubType Standard
$schedulePolicy.ScheduleRunTimes = @([datetime]$backupTime)
$schedulePolicy.ScheduleRunFrequency = "Daily"

$retentionPolicy = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType AzureVM
$retentionPolicy.DailySchedule.DurationCountInDays = $retentionDays
$retentionPolicy.IsWeeklyScheduleEnabled = $true
$retentionPolicy.WeeklySchedule.DurationCountInWeeks = 12
$retentionPolicy.IsMonthlyScheduleEnabled = $true
$retentionPolicy.MonthlySchedule.DurationCountInMonths = 12

$policy = New-AzRecoveryServicesBackupProtectionPolicy `
    -Name "Policy-Daily-30d" `
    -WorkloadType AzureVM `
    -RetentionPolicy $retentionPolicy `
    -SchedulePolicy $schedulePolicy

# 3. VMs schützen (alle VMs einer Ressourcengruppe)
$vms = Get-AzVM -ResourceGroupName "rg-prod"
foreach ($vm in $vms) {
    Enable-AzRecoveryServicesBackupProtection `
        -ResourceGroupName $vm.ResourceGroupName `
        -Name $vm.Name `
        -Policy $policy
    Write-Host "Backup aktiviert: $($vm.Name)" -ForegroundColor Green
}

# 4. Sofortigen Backup starten
$item = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -Name "vm-prod-01"
$job  = Backup-AzRecoveryServicesBackupItem -Item $item
Write-Host "Backup-Job gestartet: $($job.JobId)"

# 5. Backup-Status überwachen
Get-AzRecoveryServicesBackupJob -Status InProgress | Format-Table WorkloadName, Status, StartTime
```

### Azure Backup — Soft Delete und Immutability

```powershell
# Soft Delete: schützt vor versehentlichem/bösartigem Löschen
# Standard: 14 Tage, erweiterbar auf 180 Tage

# Soft Delete Status prüfen
$vault = Get-AzRecoveryServicesVault -Name "rsv-backup-westeurope"
$prop  = Get-AzRecoveryServicesVaultProperty -VaultId $vault.ID
$prop.SoftDeleteFeatureState  # Enabled/Disabled/AlwaysOn

# Immutable Vault aktivieren (ACHTUNG: nicht rückgängig zu machen!)
# Locked = niemand kann Backups löschen (auch kein Global Admin)
Set-AzRecoveryServicesVaultProperty -VaultId $vault.ID `
    -ImmutabilityState Unlocked  # Erst Unlocked testen, dann Locked

# Vault-Lock für Compliance (Azure Policy):
# Verhindert, dass der Vault gelöscht wird
New-AzResourceLock -LockName "NoDelete-BackupVault" `
    -LockLevel CanNotDelete `
    -ResourceName "rsv-backup-westeurope" `
    -ResourceType "Microsoft.RecoveryServices/vaults" `
    -ResourceGroupName "rg-backup-prod"
```

### Azure Backup Monitoring mit Azure Monitor

```powershell
# Diagnose-Einstellungen: Backup-Logs → Log Analytics
$workspace = Get-AzOperationalInsightsWorkspace -Name "law-security-prod"
$vault     = Get-AzRecoveryServicesVault -Name "rsv-backup-westeurope"

Set-AzDiagnosticSetting -ResourceId $vault.ID `
    -WorkspaceId $workspace.ResourceId `
    -Enabled $true `
    -Category "AzureBackupReport","CoreAzureBackup","AddonAzureBackupAlerts"

# KQL-Query: Fehlgeschlagene Backup-Jobs (letzte 7 Tage)
# Im Log Analytics Workspace:
<#
AddonAzureBackupJobs
| where TimeGenerated > ago(7d)
| where JobStatus == "Failed"
| project TimeGenerated, BackupItemUniqueId, JobOperation, JobFailureCode, JobErrorMessage
| order by TimeGenerated desc
#>
```

---

## AWS Backup

### Backup Plan per AWS CLI / CloudFormation

```bash
# AWS Backup Plan erstellen (CLI)
aws backup create-backup-plan --backup-plan '{
  "BackupPlanName": "prod-backup-plan",
  "Rules": [
    {
      "RuleName": "daily-backup",
      "TargetBackupVaultName": "prod-backup-vault",
      "ScheduleExpression": "cron(0 23 * * ? *)",
      "StartWindowMinutes": 60,
      "CompletionWindowMinutes": 120,
      "Lifecycle": {
        "DeleteAfterDays": 35
      },
      "CopyActions": [
        {
          "DestinationBackupVaultArn": "arn:aws:backup:eu-west-1:123456789:backup-vault:dr-vault",
          "Lifecycle": { "DeleteAfterDays": 90 }
        }
      ]
    },
    {
      "RuleName": "monthly-backup-gfs",
      "TargetBackupVaultName": "prod-backup-vault",
      "ScheduleExpression": "cron(0 23 1 * ? *)",
      "Lifecycle": {
        "DeleteAfterDays": 365
      }
    }
  ]
}'

# Vault Lock (WORM) für Compliance — ACHTUNG: Governance vs. Compliance Mode
# Governance: Admins können Lock aufheben (für Testing)
# Compliance: NIEMAND kann löschen, auch nicht AWS Support!
aws backup put-backup-vault-lock-configuration \
    --backup-vault-name "prod-backup-vault" \
    --min-retention-days 7 \
    --max-retention-days 365
    # Ohne --changeable-for-days = sofort Compliance Mode (irreversibel!)
```

```python
# AWS Backup Status Report (Python/Boto3)
import boto3
from datetime import datetime, timedelta

backup = boto3.client('backup', region_name='eu-central-1')

# Fehlgeschlagene Jobs letzte 24h
response = backup.list_backup_jobs(
    ByState='FAILED',
    ByCreatedAfter=datetime.utcnow() - timedelta(hours=24)
)

for job in response['BackupJobs']:
    print(f"FEHLER: {job['ResourceArn']} | {job['StatusMessage']} | {job['CreationDate']}")
```

---

## Wasabi (S3-kompatibel, DSGVO-konform, EU-Region)

```
Wasabi-Vorteile für DSGVO-konforme Offsite-Backups:
- Rechenzentren: eu-central-1 (Frankfurt), eu-central-2 (Amsterdam)
- Kein Egress-Pricing (unbegrenzte Downloads kostenfrei)
- Object Lock (WORM) verfügbar
- ~80% günstiger als AWS S3

Konfiguration mit Veeam:
1. Wasabi-Bucket erstellen: eu-central-2.wasabisys.com
2. IAM-User mit MinimalPolicy:
   s3:PutObject, s3:GetObject, s3:DeleteObject (für Ablauf),
   s3:ListBucket, s3:GetBucketLocation
3. Object Lock am Bucket aktivieren (nur bei Bucket-Erstellung möglich!)
4. In Veeam: Backup Infrastructure → Object Storage Repositories → Add
   → S3 Compatible, URL: s3.eu-central-2.wasabisys.com
5. SOBR Capacity-Tier → Wasabi-Repository auswählen
```

```bash
# Wasabi Bucket-Verwaltung mit AWS CLI (kompatibel)
export AWS_ACCESS_KEY_ID="wasabi-access-key"
export AWS_SECRET_ACCESS_KEY="wasabi-secret-key"

# Bucket erstellen (eu-central-2 = Amsterdam)
aws s3api create-bucket \
    --bucket backup-offsite-firma \
    --endpoint-url https://s3.eu-central-2.wasabisys.com \
    --create-bucket-configuration LocationConstraint=eu-central-2

# Object Lock aktivieren (MUSS beim Erstellen passieren)
aws s3api put-object-lock-configuration \
    --bucket backup-offsite-firma \
    --endpoint-url https://s3.eu-central-2.wasabisys.com \
    --object-lock-configuration '{
        "ObjectLockEnabled": "Enabled",
        "Rule": {
            "DefaultRetention": {
                "Mode": "COMPLIANCE",
                "Days": 14
            }
        }
    }'

# Bucket-Inhalt und Größe prüfen
aws s3 ls s3://backup-offsite-firma \
    --endpoint-url https://s3.eu-central-2.wasabisys.com \
    --recursive --human-readable --summarize
```

---

## DSGVO-Checkliste Cloud-Backup

```markdown
## DSGVO-Compliance Cloud-Backup

### Speicherort
- [ ] Verarbeitungsort: EU/EWR (Frankfurt, Amsterdam, Dublin, Irland)
- [ ] KEIN automatischer Datentransfer in Drittländer (USA, Asien)
- [ ] Regionsverriegelung aktiviert (Azure: keine geo-redundante Replikation außerhalb EU)

### Vertragliches
- [ ] Auftragsverarbeitungsvertrag (AVV / DPA) mit Cloud-Anbieter abgeschlossen
  → Azure: OST (Online Services Terms) + DPA automatisch
  → AWS: AWS DPA auf aws.amazon.com/de/agreement/
  → Wasabi: DPA auf wasabi.com/legal/
- [ ] Subauftragsverarbeiter des Cloud-Anbieters bekannt und akzeptiert

### Verschlüsselung
- [ ] Encryption at rest: AES-256 (Standard bei Azure/AWS/Wasabi)
- [ ] Encryption in transit: TLS 1.2+ (Standard)
- [ ] Kundenseitige Schlüsselverwaltung (BYOK): evaluiert?
  → Azure: Customer-Managed Keys (CMK) via Key Vault
  → AWS: AWS KMS mit Customer-Managed Key
  → Veeam: Backup-Job-Verschlüsselung VOR Upload (empfohlen für Wasabi)

### Zugriffskontrolle
- [ ] IAM-Rollen mit minimalen Rechten (kein full S3-Access)
- [ ] MFA für Cloud-Console-Zugang
- [ ] Backup-Daten nur für Backup-Accounts zugänglich (kein shared access)
- [ ] Zugriffslogs aktiviert (Azure Storage Logs / AWS CloudTrail / Wasabi Access Logs)

### Löschung
- [ ] Löschkonzept dokumentiert: wann werden Backups gelöscht?
- [ ] Sicherstellung dass nach Ablauf Retention-Zeit tatsächlich gelöscht wird
- [ ] Bei Vertragsende: Datenlöschnachweis vom Anbieter anfordern
```
