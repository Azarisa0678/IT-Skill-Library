# High Availability & Disaster Recovery — Referenzmodul

Multi-AZ, Multi-Region, RTO/RPO Cloud, Chaos Engineering, Azure Site Recovery, AWS DR Patterns.

---

## AWS DR-Strategien

```
BACKUP & RESTORE (RTO: Stunden | RPO: Stunden | Kosten: ★☆☆☆☆)
  - S3 Backups + AMI-Snapshots
  - Wiederherstellung: neue EC2 aus Snapshot starten
  - Geeignet für: Dev/Test, nicht-kritische Workloads

PILOT LIGHT (RTO: ~1h | RPO: Minuten | Kosten: ★★☆☆☆)
  - Minimale Kerninfrastruktur läuft in DR-Region (DB-Replikation aktiv)
  - Bei Disaster: Compute in DR-Region starten, DNS umschalten
  - Geeignet für: Wichtige Workloads mit moderaten RTO-Anforderungen

WARM STANDBY (RTO: ~15min | RPO: Sekunden | Kosten: ★★★☆☆)
  - DR-Region läuft im Reduced-Scale
  - Bei Disaster: Scale-Up + DNS-Failover
  - Geeignet für: Wichtige Business-Systeme

MULTI-SITE ACTIVE-ACTIVE (RTO: Sekunden | RPO: 0 | Kosten: ★★★★★)
  - Beide Regionen aktiv, Load Balancing über Route 53
  - Kein Failover nötig (automatisch)
  - Geeignet für: Mission-Critical, kein Ausfall tolerierbar
```

### AWS Multi-Region mit Route 53

```hcl
# Route 53 Failover Routing (Active-Passive)
resource "aws_route53_health_check" "primary" {
  fqdn              = "api-primary.firma.de"
  port              = 443
  type              = "HTTPS"
  resource_path     = "/healthz"
  failure_threshold = 3
  request_interval  = 10
  tags = { Name = "primary-health-check" }
}

resource "aws_route53_record" "api_primary" {
  zone_id         = aws_route53_zone.main.zone_id
  name            = "api.firma.de"
  type            = "A"
  set_identifier  = "primary"
  health_check_id = aws_route53_health_check.primary.id

  failover_routing_policy { type = "PRIMARY" }
  alias {
    name                   = aws_lb.primary.dns_name
    zone_id                = aws_lb.primary.zone_id
    evaluate_target_health = true
  }
}

resource "aws_route53_record" "api_secondary" {
  zone_id        = aws_route53_zone.main.zone_id
  name           = "api.firma.de"
  type           = "A"
  set_identifier = "secondary"
  provider       = aws.dr_region  # eu-west-1 als DR

  failover_routing_policy { type = "SECONDARY" }
  alias {
    name                   = aws_lb.secondary.dns_name
    zone_id                = aws_lb.secondary.zone_id
    evaluate_target_health = true
  }
}
```

---

## Azure High Availability

```hcl
# Terraform: AKS in Availability Zones + Application Gateway
resource "azurerm_kubernetes_cluster" "prod" {
  name                = "aks-prod-we"
  location            = "westeurope"
  resource_group_name = azurerm_resource_group.main.name
  dns_prefix          = "firma-prod"
  sku_tier            = "Standard"  # Für Production SLA!

  default_node_pool {
    name                = "system"
    node_count          = 3
    vm_size             = "Standard_D4s_v5"
    zones               = ["1", "2", "3"]  # Über alle AZs verteilen!
    type                = "VirtualMachineScaleSets"
    ultra_ssd_enabled   = false
    os_disk_type        = "Ephemeral"
  }

  # Zone-Redundant: mind. 3 Nodes über 3 AZs
  # SLA: 99.95% ohne AZ, 99.99% mit AZ!
}

# Azure Traffic Manager (Global Load Balancing)
resource "azurerm_traffic_manager_profile" "global" {
  name                = "tm-firma-global"
  resource_group_name = azurerm_resource_group.main.name
  traffic_routing_method = "Priority"  # oder Performance, Weighted, Geographic

  dns_config {
    relative_name = "firma-api"
    ttl           = 30  # Kurzes TTL für schnellen Failover!
  }

  monitor_config {
    protocol                     = "HTTPS"
    port                         = 443
    path                         = "/healthz"
    interval_in_seconds          = 10
    timeout_in_seconds           = 5
    tolerated_number_of_failures = 2
  }
}

resource "azurerm_traffic_manager_azure_endpoint" "primary" {
  name               = "primary-we"
  profile_id         = azurerm_traffic_manager_profile.global.id
  target_resource_id = azurerm_public_ip.primary.id
  weight             = 1
  priority           = 1
}

resource "azurerm_traffic_manager_azure_endpoint" "secondary" {
  name               = "secondary-ne"  # North Europe als DR
  profile_id         = azurerm_traffic_manager_profile.global.id
  target_resource_id = azurerm_public_ip.secondary.id
  weight             = 1
  priority           = 2  # Nur bei Primary-Ausfall aktiv
}
```

---

## Chaos Engineering

```python
#!/usr/bin/env python3
"""AWS Fault Injection Simulator (FIS) — Chaos Experiments"""
import boto3
import time

fis = boto3.client('fis', region_name='eu-central-1')

# Experiment: EC2-Instanzen in einer AZ terminieren
experiment = fis.create_experiment_template(
    description="Simulate AZ-Outage: Terminate 50% of EC2 in eu-central-1a",
    targets={
        "EC2Instances": {
            "resourceType": "aws:ec2:instance",
            "resourceTags": {"Environment": "staging"},
            "filters": [{"path": "Placement.AvailabilityZone",
                        "values": ["eu-central-1a"]}],
            "selectionMode": "PERCENT(50)"
        }
    },
    actions={
        "terminateInstances": {
            "actionId": "aws:ec2:terminate-instances",
            "targets": {"Instances": "EC2Instances"},
            "parameters": {}
        }
    },
    stopConditions=[{
        "source": "aws:cloudwatch:alarm",
        "value": "arn:aws:cloudwatch:eu-central-1:123456789:alarm:ErrorRateCritical"
    }],
    roleArn="arn:aws:iam::123456789:role/FISRole",
    tags={"Name": "AZ-Outage-Experiment"}
)

template_id = experiment['experimentTemplate']['id']
print(f"Experiment Template erstellt: {template_id}")

# Experiment starten (nur in Staging!)
exp = fis.start_experiment(experimentTemplateId=template_id)
exp_id = exp['experiment']['id']

# Status überwachen
while True:
    status = fis.get_experiment(id=exp_id)['experiment']['state']['status']
    print(f"Status: {status}")
    if status in ['completed', 'failed', 'stopped']:
        break
    time.sleep(10)
```

---

## RTO/RPO Berechnung und SLA

```markdown
## Cloud SLA Zusammenfassung

### AWS
| Service | SLA |
|---------|-----|
| EC2 (Single AZ) | 99.5% |
| EC2 (Multi-AZ) | 99.99% |
| RDS (Single-AZ) | 99.95% |
| RDS Multi-AZ | 99.95% |
| S3 | 99.9% (Standard), 99.99% (Standard-IA) |
| Lambda | 99.95% |
| EKS | 99.95% |
| Route 53 | 100% |

### Azure
| Service | SLA |
|---------|-----|
| VMs (Single) | 99.9% |
| VMs (Availability Set) | 99.95% |
| VMs (Availability Zones) | 99.99% |
| AKS | 99.9% (Free), 99.95% (Standard), 99.99% (Premium) |
| Azure SQL | 99.99% |
| Storage (LRS) | 99.9% |
| Storage (ZRS/GRS) | 99.99% |
| Functions | 99.95% |

### Verfügbarkeit Berechnung
99.9%  = max. 8.76h Ausfall/Jahr (526 Min/Jahr)
99.95% = max. 4.38h Ausfall/Jahr (263 Min/Jahr)
99.99% = max. 52.6 Min Ausfall/Jahr
99.999% = max. 5.26 Min Ausfall/Jahr

COMPOSITE SLA (mehrere Services):
  API Gateway (99.95%) × Lambda (99.95%) × DynamoDB (99.999%)
  = 0.9995 × 0.9995 × 0.99999 = 99.899% ≈ 99.9%
  → Composite SLA ist immer schlechter als schlechtester Einzel-SLA!
```
