# FinOps & Cost Optimization — Referenzmodul

AWS Cost Management, Azure Cost Management, GCP Billing, Reserved Instances,
Savings Plans, Spot/Preemptible, Tagging-Strategie, FinOps Framework.

---

## FinOps Framework (Cloud Financial Management)

```
FINOPS PHASEN:
  INFORM:     Kosten verstehen, Transparenz schaffen, Tagging, Dashboards
  OPTIMIZE:   Idle Resources eliminieren, Right-Sizing, Commitment Discounts
  OPERATE:    Kontinuierliche Optimierung, Budgets, Anomalie-Alerts, Kultur

FINOPS PRINZIPIEN:
  ✓ Teams verantworten ihre eigenen Cloud-Kosten (Ownership)
  ✓ Cloud-Ausgaben sind Business-Entscheidungen (nicht IT-Entscheidungen)
  ✓ Zugängliche Kostendaten für alle Teams
  ✓ Optimierung kontinuierlich, nicht einmalig
  ✓ Cloud-Variabilität als Stärke nutzen (Scale up/down)

TAGGING-STRATEGIE (Pflicht für Cost Allocation):
  Pflicht-Tags:
    Environment:  dev | staging | prod
    Team:         platform | backend | frontend | data
    CostCenter:   CC-1234 (aus Finanzbuchhaltung)
    Project:      projektname (für Projektabrechnung)
    Owner:        email@firma.de (Verantwortlicher)

  Enforcement:
    AWS:   AWS Config Rule "required-tags" + SCP blockt ungetaggte Ressourcen
    Azure: Azure Policy "Require tag and its value"
    GCP:   Organization Policy + Budget Alerts
```

---

## AWS Cost Optimization

```python
#!/usr/bin/env python3
"""AWS Cost Optimization — Idle Resources finden"""
import boto3
from datetime import datetime, timedelta

ce     = boto3.client('ce',       region_name='eu-central-1')
ec2    = boto3.client('ec2',      region_name='eu-central-1')
cw     = boto3.client('cloudwatch', region_name='eu-central-1')

# ─── KOSTENÜBERSICHT LETZTE 30 TAGE ──────────────────────────────────────────
def get_monthly_costs():
    end   = datetime.today().strftime('%Y-%m-%d')
    start = (datetime.today() - timedelta(days=30)).strftime('%Y-%m-%d')

    response = ce.get_cost_and_usage(
        TimePeriod={'Start': start, 'End': end},
        Granularity='MONTHLY',
        Metrics=['UnblendedCost'],
        GroupBy=[{'Type': 'DIMENSION', 'Key': 'SERVICE'}]
    )
    print("\n=== AWS Kosten letzte 30 Tage ===")
    total = 0
    results = []
    for group in response['ResultsByTime'][0]['Groups']:
        service = group['Keys'][0]
        cost    = float(group['Metrics']['UnblendedCost']['Amount'])
        total  += cost
        results.append((service, cost))

    for service, cost in sorted(results, key=lambda x: x[1], reverse=True)[:10]:
        print(f"  {service:<45} ${cost:>10.2f}")
    print(f"  {'GESAMT':<45} ${total:>10.2f}")

# ─── IDLE EC2 INSTANZEN (CPU < 5% letzte 2 Wochen) ───────────────────────────
def find_idle_ec2():
    print("\n=== Idle EC2 Instanzen (CPU < 5%, 14 Tage) ===")
    instances = ec2.describe_instances(
        Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
    )

    for reservation in instances['Reservations']:
        for inst in reservation['Instances']:
            instance_id   = inst['InstanceId']
            instance_type = inst['InstanceType']
            name = next((t['Value'] for t in inst.get('Tags', [])
                        if t['Key'] == 'Name'), 'unnamed')

            # CloudWatch CPU Metrik
            metrics = cw.get_metric_statistics(
                Namespace='AWS/EC2',
                MetricName='CPUUtilization',
                Dimensions=[{'Name': 'InstanceId', 'Value': instance_id}],
                StartTime=datetime.utcnow() - timedelta(days=14),
                EndTime=datetime.utcnow(),
                Period=86400,
                Statistics=['Average']
            )

            if metrics['Datapoints']:
                avg_cpu = sum(d['Average'] for d in metrics['Datapoints']) / len(metrics['Datapoints'])
                if avg_cpu < 5.0:
                    print(f"  IDLE: {instance_id} ({instance_type}) {name} — Ø CPU: {avg_cpu:.1f}%")
                    print(f"    → Kandidat für Stop/Terminate oder Right-Sizing")

# ─── UNGENUTZTE RESSOURCEN ────────────────────────────────────────────────────
def find_unused_resources():
    print("\n=== Ungenutzte AWS Ressourcen ===")

    # Unattached EBS Volumes
    volumes = ec2.describe_volumes(Filters=[{'Name': 'status', 'Values': ['available']}])
    for vol in volumes['Volumes']:
        size = vol['Size']
        name = next((t['Value'] for t in vol.get('Tags', []) if t['Key'] == 'Name'), 'unnamed')
        print(f"  EBS unattached: {vol['VolumeId']} ({size}GB) {name} — Löschen möglich?")

    # Nicht zugeordnete Elastic IPs
    eips = ec2.describe_addresses(Filters=[{'Name': 'domain', 'Values': ['vpc']}])
    for eip in eips['Addresses']:
        if 'AssociationId' not in eip:
            print(f"  EIP ungenutzt: {eip.get('PublicIp')} — Kosten: ~$3.60/Monat → Freigeben!")

    # Old Snapshots (> 90 Tage)
    snapshots = ec2.describe_snapshots(OwnerIds=['self'])
    cutoff = datetime.utcnow() - timedelta(days=90)
    for snap in snapshots['Snapshots']:
        if snap['StartTime'].replace(tzinfo=None) < cutoff:
            name = next((t['Value'] for t in snap.get('Tags', []) if t['Key'] == 'Name'), '-')
            print(f"  Snapshot alt: {snap['SnapshotId']} ({snap['VolumeSize']}GB) {name} "
                  f"— {snap['StartTime'].strftime('%Y-%m-%d')}")

if __name__ == '__main__':
    get_monthly_costs()
    find_idle_ec2()
    find_unused_resources()
```

### AWS Savings Plans vs. Reserved Instances

```
SAVINGS PLANS (flexibler, empfohlen):
  Compute Savings Plans:   bis 66% Rabatt, gilt für EC2, Lambda, Fargate
                           Commitment: $/Stunde für 1 oder 3 Jahre
  EC2 Instance Savings:    bis 72% Rabatt, NUR für EC2 in einer Instanzfamilie
                           Commitment: spezifische Region + Instanzfamilie

RESERVED INSTANCES (älteres Modell):
  Standard RI:    bis 72% Rabatt, sehr unflexibel
  Convertible RI: bis 54% Rabatt, kann zu anderem Typ gewechselt werden
  Scheduled RI:   nur zu definierten Zeiten aktiv

ENTSCHEIDUNGSMATRIX:
  Stabile 24/7-Workload (EC2, RDS) → Savings Plans (1 Jahr, kein Upfront)
  Sehr stabiler, unveränderlicher Workload → RI (3 Jahre, All Upfront = max Rabatt)
  Variable Workloads / Batch / Fehlertolerante Jobs → Spot Instances (bis 90% Rabatt)
  Dev/Test-Umgebungen → Auto-Shutdown nach Bürostunden (Lambda + EventBridge)

FAUSTREGELN:
  ✓ Nie mehr als 70% der Baseline committen (Rest elastisch)
  ✓ 1-Jahres-Commitments bevorzugen (Flexibilität > maximaler Rabatt)
  ✓ Savings Plans Coverage-Report monatlich prüfen
  ✓ AWS Cost Explorer → Savings Plans Recommendations nutzen
```

---

## Azure Cost Management

```powershell
# ─── KOSTENÜBERSICHT PER POWERSHELL ──────────────────────────────────────────
Connect-AzAccount

# Kosten der letzten 30 Tage je Ressourcengruppe
$startDate = (Get-Date).AddDays(-30).ToString("yyyy-MM-dd")
$endDate   = (Get-Date).ToString("yyyy-MM-dd")

$costs = Get-AzConsumptionUsageDetail -StartDate $startDate -EndDate $endDate |
    Group-Object ResourceGroupName |
    Select-Object Name,
        @{N='KostenEUR'; E={[math]::Round(($_.Group | Measure-Object PretaxCost -Sum).Sum, 2)}} |
    Sort-Object KostenEUR -Descending

$costs | Format-Table -AutoSize
Write-Host "GESAMT: EUR $([math]::Round(($costs | Measure-Object KostenEUR -Sum).Sum, 2))"

# ─── AZURE ADVISOR COST RECOMMENDATIONS ──────────────────────────────────────
$recommendations = Get-AzAdvisorRecommendation -Category Cost
$recommendations | Select-Object ShortDescription,
    @{N='EinsparungEUR/Monat'; E={
        $impact = $_.ExtendedProperties['savingsAmount']
        if ($impact) { [math]::Round([decimal]$impact, 2) } else { 0 }
    }},
    @{N='Ressource'; E={$_.ResourceMetadata.ResourceId.Split('/')[-1]}} |
    Sort-Object 'EinsparungEUR/Monat' -Descending |
    Format-Table -AutoSize

# ─── IDLE VMs FINDEN (CPU < 5%, 14 Tage) ─────────────────────────────────────
$vms = Get-AzVM -Status | Where-Object {$_.PowerState -eq "VM running"}

foreach ($vm in $vms) {
    $metrics = Get-AzMetric -ResourceId $vm.Id `
        -MetricName "Percentage CPU" `
        -StartTime (Get-Date).AddDays(-14) `
        -EndTime (Get-Date) `
        -AggregationType Average `
        -TimeGrain ([TimeSpan]::FromDays(1))

    $avgCpu = ($metrics.Data | Measure-Object Average -Average).Average
    if ($avgCpu -and $avgCpu -lt 5) {
        Write-Host "IDLE: $($vm.Name) ($($vm.HardwareProfile.VmSize)) — Ø CPU: $([math]::Round($avgCpu,1))%"
    }
}

# ─── BUDGET-ALERT EINRICHTEN ─────────────────────────────────────────────────
$budget = New-AzConsumptionBudget `
    -Name "Budget-IT-Prod-2024" `
    -Amount 5000 `
    -Category Cost `
    -StartDate "2024-01-01" `
    -EndDate "2024-12-31" `
    -TimeGrain Monthly `
    -ContactEmails @("finops@firma.de", "it-leiter@firma.de") `
    -Threshold 80    # Alert bei 80% des Budgets
```

---

## Kubernetes Cost Optimization (Kubecost)

```bash
# Kubecost installieren (Helm)
helm repo add kubecost https://kubecost.github.io/cost-analyzer/
helm install kubecost kubecost/cost-analyzer \
    --namespace kubecost --create-namespace \
    --set kubecostToken="demotoken" \
    --set global.prometheus.enabled=true

# Kubecost CLI (kubectl plugin)
kubectl cost namespace --window 30d --show-all-resources

# Namespace-Kosten letzte 30 Tage
# OUTPUT:
# NAMESPACE        CPU COST   MEM COST   GPU COST   PV COST   TOTAL
# production       $234.50    $89.20     $0.00      $45.00    $368.70
# staging          $45.20     $22.10     $0.00      $10.00    $77.30

# Pod ohne Resource-Limits (Kostenfalle!)
kubectl get pods --all-namespaces -o json | jq -r '
.items[] |
select(.spec.containers[].resources.limits == null) |
"\(.metadata.namespace)/\(.metadata.name)"'

# VPA (Vertical Pod Autoscaler) für Right-Sizing
kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler.yaml

# VPA-Empfehlungen anzeigen
kubectl get vpa --all-namespaces -o json | jq -r '
.items[] |
"\(.metadata.name): CPU request → \(.status.recommendation.containerRecommendations[0].target.cpu),
 MEM request → \(.status.recommendation.containerRecommendations[0].target.memory)"'
```

---

## FinOps Checkliste

```markdown
## FinOps Checkliste — [Unternehmen] — [Datum]

### Visibility (Inform)
- [ ] Tagging-Strategie definiert und dokumentiert
- [ ] Pflicht-Tags via Policy erzwungen (Environment, Team, CostCenter)
- [ ] Cost Dashboard eingerichtet (AWS Cost Explorer / Azure Cost Mgmt)
- [ ] Kosten aufgeteilt nach Team/Projekt/Environment
- [ ] Monatlicher Kostenbericht an Stakeholder

### Optimierung
- [ ] Idle Resources identifiziert (CPU < 5%, 14 Tage)
- [ ] Unattached Storage (EBS, Managed Disks) bereinigt
- [ ] Ungenutzte Elastic IPs / Public IPs freigegeben
- [ ] Snapshots älter als 90 Tage überprüft und bereinigt
- [ ] Dev/Test: Auto-Shutdown nach 19:00 Uhr konfiguriert
- [ ] Right-Sizing: Instanztypen dem tatsächlichen Bedarf angepasst

### Commitment Discounts
- [ ] Stabile Workloads identifiziert (> 80% Auslastung)
- [ ] Savings Plans / Reserved Instances evaluiert
- [ ] Coverage ≥ 70% der stabilen Baseline committet
- [ ] 1-Jahres-Commitments bevorzugt (Flexibilität)

### Governance
- [ ] Budget-Alerts: 80% + 100% des Monatsbudgets
- [ ] Anomalie-Erkennung aktiv (AWS Cost Anomaly Detection)
- [ ] FinOps-Review: monatlich (Kosten, Trends, Abweichungen)
- [ ] Shared Services korrekt aufgeteilt (z.B. NAT Gateway, Monitoring)
```
