---
name: cloud-architecture-skill
description: >
  Cloud Architecture Skill fuer Cloud-Architekten und Platform-Engineers.
  AWS Azure GCP Well-Architected 6 Saeulen, Landing Zone, Control Tower, Management Groups,
  Multi-Cloud Trade-offs, CSPM, Checkov, Prowler, ScoutSuite, IAM Least Privilege,
  Secrets Management AWS Secrets Manager Azure Key Vault, DSGVO Cloud, NSG VPC VNet,
  High Availability Multi-AZ Multi-Region, Route 53 Failover, Azure Traffic Manager,
  Disaster Recovery Backup-Restore Pilot-Light Warm-Standby Active-Active,
  Chaos Engineering AWS FIS, SLA Composite, FinOps, Reserved Instances, Savings Plans,
  Spot Instances, Kubecost, Cost Optimization, idle EC2, Budget-Alerts,
  Serverless, Lambda, Azure Functions, Lambda Security, IAM Lambda, Code Signing Lambda,
  Kubernetes Security, RBAC, Pod Security Admission, PSA, restricted, Falco,
  Network Policy, External Secrets Operator ESO, Workload Identity, IRSA,
  EKS AKS GKE, Container Security, Image Signing Cosign, Trivy, Runtime Security.
  Outputs: Terraform HCL, Architektur-Diagramme, Entscheidungsmatrizen, Checklisten.
---

# Cloud Architecture Skill

## Leitprinzipien

**Security by Design.** Sicherheit ist nicht nachträglich — IAM, Encryption, Logging ab Tag 1.
**Least Privilege.** Kein Wildcard-IAM (*:*), keine Root/Global Admin für Routine-Aufgaben.
**Infrastructure as Code.** Jede Cloud-Ressource via Terraform/Bicep — kein Klicken in der Konsole.
**DSGVO-Datenhaltung.** Alle Produktionsdaten in EU/EWR-Regionen, Regionsverriegelung via Policy.
**FinOps von Anfang an.** Tagging-Strategie, Budget-Alerts, Reserved Instances für stabile Workloads.
**Composites SLA berechnen.** Mehrere Services × ihre SLAs = immer schlechter als der schwächste.

---

## Referenzmodule

- `references/well-architected.md` — 6 Säulen, AWS/Azure Review, Multi-Cloud Matching, Landing Zone Design
- `references/cloud-security.md` — IAM Least Privilege, CSPM, Checkov, Secrets Management, Network Security, DSGVO-Checkliste
- `references/ha-dr.md` — AWS DR-Strategien, Multi-Region Terraform, AKS Availability Zones, Traffic Manager, Chaos Engineering, SLA-Tabellen
- `references/finops.md` — FinOps Framework, AWS/Azure Cost Optimization, Savings Plans, Idle Resources, Kubecost, Budget-Alerts
- `references/serverless-k8s-security.md` — Lambda Security, Azure Functions, Kubernetes RBAC, PSA, Falco, Network Policy, ESO

Routing:
- Well-Architected Review / Landing Zone / Multi-Cloud → `well-architected.md`
- IAM / CSPM / Sicherheits-Scanning / DSGVO Cloud → `cloud-security.md`
- HA / DR / Multi-Region / Chaos Engineering / SLA → `ha-dr.md`
- FinOps / Cost Optimization / Savings Plans / Idle Resources → `finops.md`
- Lambda Security / Kubernetes RBAC / Falco / Network Policy → `serverless-k8s-security.md`

---

## Well-Architected — Schnellreferenz

```
6 SÄULEN:
  1. Operational Excellence  → IaC, Runbooks, Post-mortems, Game Days
  2. Security                → Identity-first, Defense in Depth, Encrypt everything
  3. Reliability             → Multi-AZ, loose coupling, Chaos Testing, Backups
  4. Performance Efficiency  → Richtige Instanztypen, Serverless, CDN, Caching
  5. Cost Optimization       → Reserved Instances, Spot, ungenutzte Ressourcen löschen
  6. Sustainability          → Grüne Regionen, Managed Services, Effizienz maximieren

COMPOSITE SLA:
  API GW (99.95%) × Lambda (99.95%) × DynamoDB (99.999%) = 99.899%
  → Immer SCHLECHTER als der schwächste Einzel-SLA!
```

→ AWS Well-Architected Tool Python-Script, Azure Advisor PowerShell, Landing Zone Architektur: `references/well-architected.md`

---

## Cloud Security — Schnellreferenz

```json
// IAM Policy: Least Privilege — kein *:* !
{
  "Effect": "Allow",
  "Action": ["s3:PutObject", "s3:GetObject"],
  "Resource": "arn:aws:s3:::backup-prod/backups/${aws:PrincipalTag/Team}/*"
}
```

```bash
# Checkov: Terraform auf Sicherheit scannen
checkov -d ./terraform --framework terraform \
    --output sarif --output-file results.sarif

# Prowler: AWS Security Assessment
prowler aws --compliance cis_level2_aws_2.0

# Azure: Storage ohne HTTPS-Enforcement finden + reparieren
Get-AzStorageAccount | Where-Object {-not $_.EnableHttpsTrafficOnly} |
    ForEach-Object { Set-AzStorageAccount -Name $_.StorageAccountName -EnableHttpsTrafficOnly $true }
```

→ CSPM Tools (Prowler, ScoutSuite), Secrets Management Multi-Cloud, NSG/Security Groups Terraform, DSGVO-Checkliste: `references/cloud-security.md`

---

## HA & DR — Schnellreferenz

```hcl
# AKS mit Availability Zones (99.99% SLA)
resource "azurerm_kubernetes_cluster" "prod" {
  sku_tier = "Standard"  # Pflicht für SLA!
  default_node_pool {
    node_count = 3
    zones      = ["1", "2", "3"]  # Über alle AZs verteilen
  }
}

# Route 53 Failover (Primary/Secondary)
resource "aws_route53_record" "api_primary" {
  failover_routing_policy { type = "PRIMARY" }
  health_check_id = aws_route53_health_check.primary.id
}
resource "aws_route53_record" "api_secondary" {
  failover_routing_policy { type = "SECONDARY" }
}
```

```
DR-STRATEGIEN (Kosten vs. RTO):
  Backup & Restore: Stunden RTO | ★☆☆☆☆ Kosten
  Pilot Light:      ~1h RTO     | ★★☆☆☆ Kosten
  Warm Standby:     ~15min RTO  | ★★★☆☆ Kosten
  Active-Active:    Sekunden    | ★★★★★ Kosten
```

→ AWS FIS Chaos Engineering, Azure Traffic Manager, SLA-Tabellen AWS/Azure: `references/ha-dr.md`

---

## Multi-Cloud Entscheidungsmatrix

| Workload | Empfehlung | Begründung |
|---|---|---|
| Windows/.NET | Azure | Beste Integration, Hybrid-AD, M365 |
| KI/ML | GCP (Vertex AI) oder Azure (OpenAI) | TPUs, beste ML-Services |
| SAP | Azure | SAP-zertifiziert, Partnernetz |
| DSGVO-kritisch | Azure DE oder AWS EU | Regionen in Deutschland |
| Kubernetes | Alle drei | GKE historisch führend |
| Serverless | AWS Lambda | Größtes Ökosystem |

---

## Output-Formate

| Aufgabe | Format |
|---|---|
| Architektur-Review | Stärken/Schwächen je Säule + priorisierte Maßnahmen |
| Landing Zone Design | ASCII-Diagramm + Management Group / Account Struktur |
| IaC-Sicherheit | Terraform + Checkov-Konfiguration + GitHub Actions Pipeline |
| DR-Plan | RTO/RPO-Tabelle + Terraform-Konfiguration + Testplan |
| Cost-Review | Einsparpotenziale quantifiziert + Umsetzungsschritte |
---

## FinOps — Schnellreferenz

```python
# AWS: Idle EC2 finden (CPU < 5%, 14 Tage)
# → references/finops.md: vollständiges Script

# Azure: Advisor Cost Recommendations
Get-AzAdvisorRecommendation -Category Cost |
    Select-Object ShortDescription,
    @{N='EinsparungEUR'; E={$_.ExtendedProperties['savingsAmount']}} |
    Sort-Object EinsparungEUR -Descending | Format-Table
```

```
SAVINGS PLANS vs RESERVED INSTANCES:
  Savings Plans:   flexibel, EC2+Lambda+Fargate, empfohlen für die meisten Workloads
  Reserved Inst.:  max. Rabatt (72%), nur EC2, sehr unflexibel
  Spot/Preemptible: bis 90% Rabatt, nur fehlertolerante/unterbrechbare Workloads
  Faustregel: Nie mehr als 70% der Baseline committen
```

→ FinOps Framework, Tagging-Strategie, Kubecost, Budget-Alerts, Checkliste: `references/finops.md`

---

## Kubernetes & Serverless Security — Schnellreferenz

```yaml
# Pod Security Admission: Namespace auf "restricted" setzen
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted

# Jeder Container braucht:
securityContext:
  runAsNonRoot: true
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
```

```python
# Lambda: Secrets aus Secrets Manager (nie ENV-Variablen für Secrets!)
def get_secret(name): return boto3.client('secretsmanager').get_secret_value(SecretId=name)
db = get_secret(os.environ['DB_SECRET_NAME'])  # Nur Name in ENV, nicht Wert!
```

→ Falco-Regeln, Network Policies, ESO, RBAC, Lambda IAM Least Privilege, K8s Checkliste: `references/serverless-k8s-security.md`
