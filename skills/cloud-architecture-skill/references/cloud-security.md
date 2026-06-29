# Cloud Security — Referenzmodul

IAM, Zero Trust Cloud, CSPM, CWPP, CNAPP, Secrets Management, Network Security, Compliance.

---

## AWS IAM — Least Privilege

```json
// IAM Policy: Least Privilege für S3-Backup-Bucket
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowBackupWrite",
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject"],
      "Resource": "arn:aws:s3:::backup-firma-prod/*",
      "Condition": {
        "StringEquals": {"s3:prefix": ["backups/${aws:PrincipalTag/Team}/"]}
      }
    },
    {
      "Sid": "DenyDeleteAndACL",
      "Effect": "Deny",
      "Action": ["s3:DeleteObject", "s3:PutBucketAcl", "s3:PutObjectAcl"],
      "Resource": ["arn:aws:s3:::backup-firma-prod", "arn:aws:s3:::backup-firma-prod/*"]
    }
  ]
}
```

```python
# AWS IAM Access Analyzer — ungenutzte Berechtigungen finden
import boto3

analyzer = boto3.client('accessanalyzer', region_name='eu-central-1')
iam = boto3.client('iam')

# Analyzer für ungenutzte Berechtigungen (IAM Access Analyzer)
response = analyzer.list_findings(
    analyzerArn='arn:aws:access-analyzer:eu-central-1:123456789:analyzer/prod-analyzer',
    filter={
        'findingType': {'eq': ['UnusedPermission']},
        'resourceType': {'eq': ['AWS::IAM::Role']}
    }
)

for finding in response['findings']:
    print(f"Rolle: {finding['resource']} | Ungenutzte Aktion: {finding['findingDetails']}")
```

---

## Azure — Security Best Practices

```powershell
# ─── DEFENDER FOR CLOUD POLICIES HARDENING ───────────────────────────────────
Connect-AzAccount

# Secure Score für alle Subscriptions
Get-AzSubscription | ForEach-Object {
    Set-AzContext -SubscriptionId $_.SubscriptionId | Out-Null
    $score = Get-AzSecuritySecureScore -Name "ascScore"
    [PSCustomObject]@{
        Subscription = $_.Name
        Score        = "$($score.Score.Current)/$($score.Score.Max)"
        Prozent      = [math]::Round($score.Score.Percentage, 1)
    }
} | Sort-Object Prozent | Format-Table

# ─── STORAGE ACCOUNTS ABSICHERN ──────────────────────────────────────────────
# Alle Storage Accounts ohne HTTPS-Enforcement finden
Get-AzStorageAccount | Where-Object {-not $_.EnableHttpsTrafficOnly} |
    Select-Object StorageAccountName, ResourceGroupName |
    ForEach-Object {
        Set-AzStorageAccount -ResourceGroupName $_.ResourceGroupName `
            -Name $_.StorageAccountName `
            -EnableHttpsTrafficOnly $true
        Write-Host "Behoben: $($_.StorageAccountName)"
    }

# Alle Storage Accounts mit öffentlichem Blob-Zugang finden
Get-AzStorageAccount |
    Where-Object {$_.AllowBlobPublicAccess -ne $false} |
    ForEach-Object {
        Set-AzStorageAccount -ResourceGroupName $_.ResourceGroupName `
            -Name $_.StorageAccountName `
            -AllowBlobPublicAccess $false
        Write-Host "Public Access deaktiviert: $($_.StorageAccountName)"
    }

# ─── AZURE POLICY ENFORCEMENT ────────────────────────────────────────────────
# Policy: Nur erlaubte Regionen (DSGVO)
$allowedLocations = @("westeurope", "germanywestcentral", "germanynorth", "northeurope")
$policyDef = New-AzPolicyDefinition -Name "AllowedLocations-DSGVO" `
    -Policy '{
        "if": {
            "not": {
                "field": "location",
                "in": ["westeurope","germanywestcentral","germanynorth","northeurope","global"]
            }
        },
        "then": {"effect": "deny"}
    }'
New-AzPolicyAssignment -Name "DSGVO-Locations" `
    -PolicyDefinition $policyDef `
    -Scope "/subscriptions/$(Get-AzContext).Subscription.Id"
```

---

## CSPM — Cloud Security Posture Management

```yaml
# Checkov — IaC Security Scanning (Terraform)
# Installation: pip install checkov
# Ausführung:
# checkov -d ./terraform --framework terraform --output json > checkov-results.json

# GitHub Actions Integration
- name: Checkov IaC Scan
  uses: bridgecrewio/checkov-action@master
  with:
    directory: terraform/
    framework: terraform,cloudformation
    soft_fail: false
    output_format: sarif
    output_file_path: results.sarif
    skip_check: CKV_AWS_18,CKV_AWS_19  # Dokumentierte Ausnahmen

# Wichtige Checkov-Checks:
# CKV_AWS_18:  S3 Access Logging aktiviert
# CKV_AWS_53:  S3 Bucket öffentlicher Zugang gesperrt
# CKV_AWS_86:  CloudTrail Log-Datei-Validierung aktiv
# CKV_AZURE_3: Storage Account: HTTPS erzwingen
# CKV_AZURE_7: AKS: RBAC aktiviert
# CKV_AZURE_35: Key Vault: Soft Delete aktiviert
```

```python
# Prowler — AWS Security Assessment (Python)
# Installation: pip install prowler
# Ausführung:
# prowler aws --compliance cis_level2_aws_2.0
# prowler azure --compliance cis_azure_2.0
# prowler gcp --compliance cis_gcp_2.0

# Prowler Output in JSON → SIEM
# prowler aws -M json -o /output/prowler-results.json
# Sentinal Workbook für Prowler verfügbar

# ScoutSuite — Multi-Cloud Security Auditing
# Installation: pip install scoutsuite
# AWS: scout aws --report-dir ./scoutsuite-report
# Azure: scout azure --tenant <tenant-id> --report-dir ./scoutsuite-report
```

---

## Secrets Management (Multi-Cloud)

```python
#!/usr/bin/env python3
"""Unified Secrets Management — AWS Secrets Manager + Azure Key Vault + HashiCorp Vault"""

import boto3
import json
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

class SecretsManager:
    """Abstraktionsschicht für Multi-Cloud Secrets"""

    def __init__(self, provider: str, **kwargs):
        self.provider = provider
        if provider == "aws":
            self.client = boto3.client('secretsmanager',
                region_name=kwargs.get('region', 'eu-central-1'))
        elif provider == "azure":
            credential = DefaultAzureCredential()
            self.client = SecretClient(
                vault_url=f"https://{kwargs['vault_name']}.vault.azure.net",
                credential=credential
            )

    def get_secret(self, secret_name: str) -> dict:
        if self.provider == "aws":
            response = self.client.get_secret_value(SecretId=secret_name)
            return json.loads(response['SecretString'])
        elif self.provider == "azure":
            secret = self.client.get_secret(secret_name)
            return {"value": secret.value}

    def rotate_secret(self, secret_name: str, new_value: str):
        if self.provider == "aws":
            self.client.put_secret_value(
                SecretId=secret_name,
                SecretString=new_value
            )
        elif self.provider == "azure":
            self.client.set_secret(secret_name, new_value)

# Nutzung:
aws_secrets   = SecretsManager("aws", region="eu-central-1")
azure_secrets = SecretsManager("azure", vault_name="kv-firma-prod")

db_creds = aws_secrets.get_secret("prod/database/credentials")
api_key  = azure_secrets.get_secret("api-key-production")
```

---

## Network Security (Cloud)

```hcl
# Terraform: AWS VPC mit Security Groups (Least Privilege)
resource "aws_security_group" "app_tier" {
  name        = "sg-app-tier-prod"
  description = "App-Tier: nur von Load Balancer"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]  # Nur vom ALB!
    description     = "HTTP vom Application Load Balancer"
  }

  egress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.db_tier.id]
    description     = "PostgreSQL zur Datenbank"
  }

  egress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTPS für externe APIs"
  }

  tags = merge(local.common_tags, {Name = "sg-app-tier-prod"})
}

# Azure: Network Security Group
resource "azurerm_network_security_group" "app" {
  name                = "nsg-app-prod"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  security_rule {
    name                       = "Allow-LB-Inbound"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "8080"
    source_address_prefix      = "AzureLoadBalancer"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "Deny-All-Inbound"
    priority                   = 4096
    direction                  = "Inbound"
    access                     = "Deny"
    protocol                   = "*"
    source_port_range          = "*"
    destination_port_range     = "*"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}
```

---

## Cloud Compliance (DSGVO / ISO 27001)

```markdown
## Cloud-DSGVO Checkliste

### Datenresidenz
- [ ] Alle Produktionsdaten in EU/EWR (AWS eu-central-1, Azure westeurope/germanywestcentral)
- [ ] Keine unbeabsichtigte Datenreplikation in Drittländer
- [ ] Regionsverriegelung via Policy erzwungen (Azure Policy / AWS SCP)
- [ ] DR-Region ebenfalls in EU (z.B. AWS eu-west-1 Irland)

### Auftragsverarbeitung
- [ ] DPA/AVV mit AWS/Azure/GCP abgeschlossen (automatisch via Nutzungsbedingungen)
- [ ] Subauftragsverarbeiter des Cloud-Anbieters bekannt (Websites verfügbar)
- [ ] Standard-Vertragsklauseln für US-basierte Dienste vorhanden

### Technische Maßnahmen
- [ ] Verschlüsselung at rest: Customer-Managed Keys (CMK) für hochsensible Daten
- [ ] Verschlüsselung in transit: TLS 1.2+ erzwungen
- [ ] Key Vault / KMS: HSM-gesichert (FIPS 140-2 Level 3 für sehr sensible Daten)
- [ ] Backup-Verschlüsselung: CMK oder Service-Managed Keys

### Zugangskontrolle
- [ ] MFA für alle Cloud-Konsolen (kein Root/Global Admin ohne MFA!)
- [ ] IAM: Least Privilege, keine Wildcard-Policies (*:*)
- [ ] Privileged Identity Management (PIM/AWS SSO) für Admin-Zugang
- [ ] Service Accounts: Managed Identity statt Passwörter (kein Hardcoding!)

### Logging & Monitoring
- [ ] CloudTrail / Azure Activity Log / GCP Audit Logs: in allen Regionen aktiv
- [ ] Logs: 90 Tage online, 1 Jahr Archiv, unveränderlich (S3 Object Lock / Azure Immutable)
- [ ] SIEM-Integration: Logs in Sentinel/Splunk für Alerting
- [ ] Alert: Root/Global Admin Login, Policy-Änderungen, Public Access aktiviert
```
