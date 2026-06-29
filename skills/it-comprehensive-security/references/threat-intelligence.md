# Threat Intelligence Reference

## Overview

Threat Intelligence (TI) is evidence-based knowledge about existing or emerging threats that enables informed decisions about response. It transforms raw data (IOCs, TTPs) into actionable context.

### Intelligence Types

| Type | Description | Consumer | Example |
|------|-------------|----------|---------|
| **Strategic** | High-level trends, threat actors, geopolitical context | CISO, Board | Nation-state activity targeting your sector |
| **Operational** | Active campaigns, attacker infrastructure, TTPs | SOC Lead, IR Team | LockBit targeting German manufacturers |
| **Tactical** | IOCs — IPs, domains, hashes, YARA rules | SOC Analysts, SIEM | C2 IP: 185.220.101.47 |
| **Technical** | Malware analysis, exploit details | Threat Hunters, Malware Analysts | Cobalt Strike config extracted from sample |

---

## Intelligence Lifecycle

```
1. Planning & Direction  → What do we need to know?
2. Collection           → Where do we get the data?
3. Processing           → Normalize, deduplicate, enrich
4. Analysis             → Context, relevance, confidence
5. Dissemination        → Right format to right audience
6. Feedback             → Was it actionable? Improve collection
```

---

## IOC Types and TTL

| IOC Type | Example | TTL (usefulness) |
|----------|---------|-----------------|
| IP address | 185.220.101.47 | Hours to days (rotates fast) |
| Domain | evil-c2.net | Days to weeks |
| URL | evil-c2.net/payload.ps1 | Days |
| File hash (SHA256) | a3f1c2...d9e4 | Weeks to months |
| Email subject | "Urgent: Invoice attached" | Days |
| YARA rule | Detects code pattern | Months to years |
| TTP (ATT&CK) | T1059.001 — PowerShell | Years |

**Rule of thumb:** The higher on the "Pyramid of Pain" (IOCs → TTPs), the more valuable and durable the intelligence.

---

## Key Platforms & Feeds

### Open Source / Free

| Platform | Type | URL |
|----------|------|-----|
| MISP | Sharing platform | misp-project.org |
| OpenCTI | CTI platform | opencti.io |
| AlienVault OTX | IOC feeds | otx.alienvault.com |
| Abuse.ch | Malware/botnet IOCs | abuse.ch |
| VirusTotal | File/URL analysis | virustotal.com |
| Shodan | Internet asset intelligence | shodan.io |
| CIRCL CVE Search | CVE database | cve.circl.lu |
| URLhaus | Malicious URL database | urlhaus.abuse.ch |
| MalwareBazaar | Malware samples | bazaar.abuse.ch |
| ThreatFox | IOC sharing | threatfox.abuse.ch |

### Commercial

| Platform | Strength |
|----------|---------|
| Recorded Future | Breadth, dark web, geopolitical |
| CrowdStrike Intel | Nation-state actor tracking |
| Mandiant Advantage | Incident response context |
| Palo Alto Unit 42 | Malware families, campaigns |
| ESET Threat Intelligence | European focus |

---

## STIX/TAXII — Intelligence Sharing Standards

**STIX 2.1** (Structured Threat Information eXpression): JSON-based format for describing threat intelligence objects.

```json
{
  "type": "indicator",
  "spec_version": "2.1",
  "id": "indicator--8e2e2d2b-17d4-4cbf-938f-98129998f9bb",
  "name": "Malicious IP C2",
  "pattern": "[ipv4-addr:value = '185.220.101.47']",
  "pattern_type": "stix",
  "valid_from": "2024-01-15T00:00:00Z",
  "labels": ["malicious-activity"],
  "confidence": 85,
  "relationship_type": "indicates",
  "description": "Cobalt Strike C2 server observed in LockBit campaign"
}
```

**TAXII 2.1** (Trusted Automated eXchange of Intelligence Information): API protocol for sharing STIX data.

```python
from taxii2client.v21 import Server

# Connect to a TAXII server
server = Server('https://cti.example.com/taxii/',
                user='analyst', password='secret')

# List collections
for collection in server.api_roots[0].collections:
    print(f"{collection.id}: {collection.title}")

# Get indicators from collection
from stix2 import parse
collection = server.api_roots[0].get_collection('your-collection-id')
for obj in collection.get_objects()['objects']:
    parsed = parse(obj, allow_custom=True)
    if parsed.type == 'indicator':
        print(f"IOC: {parsed.pattern}")
```

---

## MISP — Threat Intelligence Platform

```bash
# Install MISP (Docker)
git clone https://github.com/MISP/misp-docker
cd misp-docker
cp template.env .env
docker-compose up -d

# Import feeds (via UI or API)
curl -H "Authorization: YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -X POST https://misp.yourorg.com/feeds/fetchFromFeed/1

# Search for IOC via API
curl -H "Authorization: YOUR_API_KEY" \
     -H "Accept: application/json" \
     "https://misp.yourorg.com/attributes/restSearch/value:185.220.101.47"
```

---

## OpenCTI — CTI Platform

```bash
# Deploy with Docker Compose
git clone https://github.com/OpenCTI-Platform/docker
cd docker
cp .env.sample .env
# Edit .env with your settings
docker-compose up -d

# Import MITRE ATT&CK
# Via UI: Settings → Connectors → MITRE ATT&CK → Enable

# Python client
from pycti import OpenCTIApiClient
api = OpenCTIApiClient('https://opencti.yourorg.com', 'YOUR_TOKEN')

# Search for indicators
indicators = api.indicator.list(filters={
    'mode': 'and',
    'filters': [{'key': 'value', 'values': ['185.220.101.47']}]
})
```

---

## Threat Hunting with Intelligence

### Hunt Hypothesis Example
```
Intelligence: Cobalt Strike beacons observed using port 443 with
              self-signed certificates (CN=*) and 60s jitter

Hunt query (Splunk):
index=network dest_port=443
| eval cert_cn=ssl_subject_cn
| where match(cert_cn, "^\*$")
| stats count, values(dest_ip), dc(src_ip) by cert_cn
| where count > 10

Hunt query (KQL / Defender):
DeviceNetworkEvents
| where RemotePort == 443
| join kind=inner (
    DeviceTvmCertificateInfo
    | where SubjectCN == "*"
) on DeviceId
| summarize count() by RemoteIP, DeviceName
```

---

## Threat Actor Tracking

### Key Groups (Sample)
| Group | Origin | Primary Targets | Key TTPs |
|-------|--------|-----------------|----------|
| APT28 (Fancy Bear) | Russia | Government, Defence, Energy | Spearphishing, credential theft |
| APT41 | China | Technology, Healthcare, Finance | Supply chain, zero-days |
| Lazarus Group | North Korea | Finance, Crypto, Defence | SWIFT fraud, ransomware |
| Sandworm | Russia | Critical Infrastructure, OT | Destructive malware, ICS attacks |
| Scattered Spider | Criminal (EN-speaking) | Telecom, Finance | Social engineering, MFA bypass |
| LockBit | Criminal (RaaS) | All sectors | Ransomware, double extortion |

---

## Intelligence-Driven Security Checklist

- [ ] Threat intelligence feeds integrated into SIEM (automated IOC blocking)
- [ ] MISP or OpenCTI deployed for internal sharing
- [ ] Sector-specific ISAC membership (financial, energy, health)
- [ ] Regular threat landscape briefings for SOC and leadership
- [ ] ATT&CK framework used to map detections to threat actor TTPs
- [ ] Vulnerability intelligence: CVE feeds prioritized by threat actor exploitation
- [ ] Dark web monitoring for leaked credentials and company mentions
- [ ] Threat hunting program driven by fresh intelligence
- [ ] Intelligence requirements defined by stakeholders (not ad hoc)
