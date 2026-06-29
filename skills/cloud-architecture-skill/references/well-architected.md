# Cloud Well-Architected — Referenzmodul

AWS Well-Architected Framework, Azure Well-Architected, Google Cloud Architecture Framework,
6 Säulen, Trade-offs, Review-Prozess, Workload-Analyse.

---

## Die 6 Säulen (AWS / Azure / GCP unified view)

```
SÄULE 1: OPERATIONAL EXCELLENCE
  Ziel: Systeme betreiben und überwachen, kontinuierlich verbessern
  Schlüsselpraktiken:
  ✓ Infrastructure as Code (IaC) für alle Ressourcen
  ✓ Runbooks und Playbooks für alle Betriebsvorgänge
  ✓ Kleine häufige Änderungen statt seltener Großänderungen
  ✓ Fehler antizipieren (Game Days, Chaos Engineering)
  ✓ Post-mortems nach Incidents (blameless)
  AWS:   CloudWatch, CloudTrail, AWS Config, Systems Manager
  Azure: Azure Monitor, Activity Log, Policy, Automation
  GCP:   Cloud Monitoring, Cloud Logging, Error Reporting

SÄULE 2: SECURITY
  Ziel: Systeme, Daten und Assets schützen
  Schlüsselpraktiken:
  ✓ Identität als primäre Sicherheitskontrolle (Zero Trust)
  ✓ Traceability: alle Aktionen loggen und auditieren
  ✓ Sicherheit auf allen Ebenen (Defense in Depth)
  ✓ Risiko-Management: automatisch reagieren
  ✓ Datenschutz at rest und in transit
  AWS:   IAM, CloudTrail, GuardDuty, Security Hub, KMS, Macie
  Azure: Entra ID, Defender for Cloud, Key Vault, Sentinel
  GCP:   Cloud IAM, Security Command Center, Cloud KMS

SÄULE 3: RELIABILITY
  Ziel: Korrekt und konsistent über Erwartungen hinaus funktionieren
  Schlüsselpraktiken:
  ✓ Foundations: Service Quotas, Netzwerk-Topologie planen
  ✓ Workload-Architektur: lose Kopplung, distributed design
  ✓ Change Management: Deployments mit Rollback-Fähigkeit
  ✓ Failure Management: Backups, DR, Chaos Testing
  AWS:   Route 53, ELB, Auto Scaling, Multi-AZ, S3 (99.999999999%)
  Azure: Traffic Manager, Load Balancer, Availability Zones, ASR
  GCP:   Cloud DNS, Load Balancing, Managed Instance Groups

SÄULE 4: PERFORMANCE EFFICIENCY
  Ziel: IT-Ressourcen effizient nutzen
  Schlüsselpraktiken:
  ✓ Richtige Ressourcentypen wählen (nicht immer der größte)
  ✓ Global ausrollen (Edge Locations, CDN)
  ✓ Serverless und managed Services nutzen
  ✓ Performance regelmäßig testen (Load Tests)
  AWS:   CloudFront, ElastiCache, RDS Read Replicas, Lambda
  Azure: Azure CDN, Cache for Redis, Cosmos DB, Functions
  GCP:   Cloud CDN, Memorystore, Spanner, Cloud Functions

SÄULE 5: COST OPTIMIZATION
  Ziel: Nur für das zahlen was tatsächlich genutzt wird
  Schlüsselpraktiken:
  ✓ Cloud Financial Management: FinOps-Praxis etablieren
  ✓ Verbrauch messen und optimieren
  ✓ Reserved Instances / Savings Plans für stabile Workloads
  ✓ Nicht benötigte Ressourcen löschen (Tag-Strategie!)
  AWS:   Cost Explorer, Trusted Advisor, Savings Plans, Spot
  Azure: Cost Management, Advisor, Reserved Instances, Spot VMs
  GCP:   Cloud Billing, Recommender, Committed Use Discounts

SÄULE 6: SUSTAINABILITY (neu, AWS 2021)
  Ziel: Umweltauswirkungen von Cloud-Workloads minimieren
  Schlüsselpraktiken:
  ✓ Region mit grüner Energie wählen
  ✓ Ressourceneffizienz maximieren
  ✓ Managed Services bevorzugen (effizienter als self-managed)
  ✓ Nicht genutzte Ressourcen eliminieren
```

---

## AWS Well-Architected Review

```python
#!/usr/bin/env python3
"""AWS Well-Architected Tool — Review automatisieren"""
import boto3

wa_client = boto3.client('wellarchitected', region_name='eu-west-1')

# Workload erstellen
workload = wa_client.create_workload(
    WorkloadName="Produktion-API-2024",
    Description="Haupt-API für Kundenportal",
    Environment="PRODUCTION",
    Lenses=["wellarchitected", "serverless"],
    AwsRegions=["eu-central-1", "eu-west-1"],
    ReviewOwner="cloud-team@firma.de",
    AccountIds=["123456789012"],
    Tags={"Environment": "prod", "Team": "platform"}
)
workload_id = workload['Workload']['WorkloadId']

# High-Risk Issues abrufen
answers = wa_client.list_answers(
    WorkloadId=workload_id,
    LensAlias="wellarchitected",
    PillarId="security"
)

# Risiken zusammenfassen
for answer in answers['AnswerSummaries']:
    if answer['Risk'] in ['HIGH', 'MEDIUM']:
        print(f"[{answer['Risk']}] {answer['QuestionTitle']}")
        print(f"  Verbesserungspläne: {len(answer.get('Reasons', []))}")
```

---

## Azure Well-Architected Review

```powershell
# Azure Advisor Empfehlungen abrufen
Connect-AzAccount
$recommendations = Get-AzAdvisorRecommendation -Category Security |
    Sort-Object Impact -Descending

$recommendations | Select-Object ShortDescription, Impact,
    @{N='Ressource'; E={$_.ResourceMetadata.ResourceId.Split('/')[-1]}} |
    Format-Table -AutoSize

# Cost Recommendations
Get-AzAdvisorRecommendation -Category Cost |
    Select-Object ShortDescription, @{N='EinsparungUSD'; E={$_.ExtendedProperties.annualSavingsAmount}} |
    Sort-Object {[decimal]$_.EinsparungUSD} -Descending |
    Format-Table

# Defender for Cloud Secure Score
$secureScore = Get-AzSecuritySecureScore -Name "ascScore"
Write-Host "Secure Score: $($secureScore.Score.Current)/$($secureScore.Score.Max) = $([math]::Round($secureScore.Score.Percentage,1))%"
```

---

## Multi-Cloud Trade-off Entscheidungsmatrix

```
WORKLOAD → CLOUD-PROVIDER MATCHING:

Windows/.NET lastig:     Azure (beste Integration, Hybrid-AD, M365)
Linux/Open Source:       AWS oder GCP (größte Community, meiste Services)
KI/ML-Workloads:         GCP (Vertex AI, TPUs) oder Azure (OpenAI-Integration)
Kubernetes/Container:    Alle drei (EKS/AKS/GKE) — GKE historisch führend
SAP Workloads:           Azure (SAP-zertifiziert, direktes Partnernetz)
Serverless:              AWS Lambda (größtes Ökosystem) oder Azure Functions
Edge Computing:          AWS Outposts / Azure Stack Edge / GCP Distributed Cloud
DSGVO / EU-Datenhaltung: Azure DE (Frankfurt/Berlin Regionen), AWS EU (Frankfurt)
                         GCP EU (Frankfurt/Niederlande/Belgien)

KOSTEN-VERGLEICH (grobe Richtwerte, immer aktuell prüfen!):
Compute:  AWS EC2 ≈ Azure VMs ≈ GCP Compute Engine (±10-20%)
Storage:  GCP oft günstiger für Egress | Azure teurer für große Datenmengen
Database: AWS RDS gut | Azure SQL sehr gut für M365-Integration
Support:  AWS teurer | Azure Business Support günstiger | GCP kompetitiv

MULTI-CLOUD WANN?
  JA: Vendor-Lock-in vermeiden bei kritischen Komponenten
  JA: Compliance (bestimmte Daten nur in bestimmtem Cloud)
  JA: M&A (verschiedene Clouds zusammenführen)
  NEIN: Einfachere Workloads ohne starken Grund
  NEIN: Wenn Komplexität den Nutzen überwiegt
```

---

## Landing Zone Design

```
AWS LANDING ZONE (Control Tower):
  Management Account:
    - AWS Organizations
    - Service Control Policies (SCPs)
    - CloudTrail (organisationsweite Logs)
    - AWS Config (organisationsweite Rules)

  Security Account:
    - Security Hub
    - GuardDuty (alle Accounts aggregiert)
    - Macie
    - SIEM-Integration (Splunk/Sentinel)

  Log Archive Account:
    - S3 Buckets für alle zentralen Logs
    - Unveränderbar (S3 Object Lock)
    - 7 Jahre Aufbewahrung

  Network Account:
    - Transit Gateway (Hub für alle VPCs)
    - Shared VPC (gemeinsame Services)
    - Direct Connect / VPN Endpoints

  Workload Accounts (je Umgebung):
    - Dev Account
    - Staging Account
    - Prod Account (separate AWS-Accounts für Isolation!)

AZURE LANDING ZONE:
  Management Group Hierarchie:
    Tenant Root Group
    └── Firma (Management Group)
        ├── Platform (Management Group)
        │   ├── Management Subscription (Log Analytics, Automation)
        │   ├── Connectivity Subscription (Hub VNet, Firewall, VPN GW)
        │   └── Identity Subscription (AD DS, ADFS)
        └── Landing Zones (Management Group)
            ├── Corp (für on-premises verbundene Workloads)
            └── Online (für internet-facing Workloads)

  Azure Policies (an Management Group-Ebene):
    - "Require encryption at rest"
    - "Allowed regions: westeurope, germanywestcentral"
    - "Require tags: Environment, Owner, CostCenter"
    - "Deny public IP on VMs"
```
