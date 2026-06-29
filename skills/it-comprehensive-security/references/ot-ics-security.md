# OT/ICS Security Reference

## Overview

Operational Technology (OT) and Industrial Control Systems (ICS) security covers the protection of systems that monitor and control physical processes — manufacturing, energy, water, transport, and critical infrastructure.

### Key Difference: OT vs IT Security

| Aspect | IT Security | OT/ICS Security |
|--------|-------------|-----------------|
| **Priority** | Confidentiality → Integrity → Availability | Availability → Integrity → Confidentiality |
| **Patch cycles** | Days to weeks | Months to years (or never) |
| **Downtime tolerance** | Hours | Zero (safety-critical) |
| **System lifetime** | 3–5 years | 15–30+ years |
| **Protocols** | TCP/IP, HTTP, TLS | Modbus, DNP3, PROFINET, OPC-UA |
| **Testing** | Active scanning OK | Active scanning can crash systems |

---

## OT/ICS Architecture — Purdue Model

```
Level 5: Enterprise Network (IT)
         ↕ DMZ (Firewall + Data Diode)
Level 4: Site Business Planning (SCADA servers, historian)
         ↕ Firewall
Level 3: Site Operations (HMI, engineering workstations)
         ↕ Firewall
Level 2: Area Supervisory Control (DCS, SCADA)
Level 1: Basic Control (PLC, RTU, sensors)
Level 0: Physical Process (actuators, motors, valves)
```

**Key principle:** Each level boundary should have a firewall. IT and OT networks must be separated — ideally with a DMZ containing data historians and jump servers.

---

## Common OT/ICS Protocols

| Protocol | Used in | Port | Security concern |
|----------|---------|------|-----------------|
| Modbus TCP | Manufacturing, utilities | 502 | No authentication, no encryption |
| DNP3 | Utilities, water | 20000 | Limited auth (SAv5 optional) |
| PROFINET | Industrial automation | Various | No built-in security |
| OPC-UA | Modern industrial | 4840 | Has security modes (sign+encrypt) |
| IEC 61850 | Power systems | Various | MMS/GOOSE — limited security |
| BACnet | Building automation | 47808 | No authentication by default |

---

## Regulatory Frameworks

### IEC 62443 (Industrial Cybersecurity Standard)

| Part | Topic |
|------|-------|
| IEC 62443-2-1 | Security management system for operators |
| IEC 62443-2-4 | Security requirements for service providers |
| IEC 62443-3-2 | Security risk assessment for system design |
| IEC 62443-3-3 | System security requirements and security levels |
| IEC 62443-4-1 | Secure product development lifecycle |
| IEC 62443-4-2 | Technical security requirements for components |

**Security Levels (SL):**
- SL 1: Protection against casual/unintentional violation
- SL 2: Protection against intentional violation with simple means
- SL 3: Protection against sophisticated attacks with moderate resources
- SL 4: Protection against nation-state level attacks

### NERC CIP (Power Sector — North America)
Critical for electrical utilities. Key standards: CIP-002 (asset identification), CIP-005 (electronic security perimeter), CIP-007 (system security management).

### BSI ICS Security (Germany / KRITIS)
- BSI-CS 005: Industrial Control System Security
- BSI Grundschutz OT-Bausteine (IND.1 – IND.3)
- KRITIS-Betreiber: BSI-Gesetz § 8a

---

## OT Security Assessment

### Passive Network Discovery (Safe for OT)
```bash
# Passive capture only — never active scan OT networks without authorization
# Use dedicated OT monitoring tools:

# Zeek (passive protocol analysis)
zeek -i eth0 /opt/zeek/share/zeek/site/local.zeek

# Wireshark / tshark — capture and analyse OT protocols
tshark -i eth0 -Y "modbus or dnp3 or s7comm or enip" -w ot_capture.pcap

# Nozomi Networks / Claroty / Dragos (commercial passive OT monitoring)
# These fingerprint assets passively without sending any packets
```

### Asset Inventory (OT-Safe Methods)
```python
# Parse existing network captures for asset discovery
# Never use active scanners like nmap in OT environments without explicit approval
import pyshark

cap = pyshark.FileCapture('ot_capture.pcap')
assets = {}
for pkt in cap:
    if hasattr(pkt, 'ip'):
        src = pkt.ip.src
        if src not in assets:
            assets[src] = {'protocols': set(), 'first_seen': pkt.sniff_time}
        if hasattr(pkt, 'modbus'):
            assets[src]['protocols'].add('Modbus')
        if hasattr(pkt, 'dnp3'):
            assets[src]['protocols'].add('DNP3')

for ip, data in assets.items():
    print(f"{ip}: {', '.join(data['protocols'])}")
```

---

## OT Hardening Checklist

### Network Segmentation
- [ ] IT and OT networks separated with firewall (not just VLAN)
- [ ] DMZ established between IT and OT with data historian
- [ ] No direct internet connectivity for OT systems
- [ ] Vendor remote access via dedicated jump server with MFA
- [ ] Data diodes for unidirectional data flow where possible

### System Hardening
- [ ] Disable unused services and ports on PLCs/RTUs/HMIs
- [ ] Change all default credentials (common issue: PLCs shipped with admin/admin)
- [ ] Remove unnecessary software from engineering workstations
- [ ] USB ports disabled or controlled where not needed
- [ ] Application whitelisting on HMI workstations

### Patch Management
- [ ] OT-specific patch management process (vendor approval required)
- [ ] Test patches in lab environment before production
- [ ] Compensating controls for unpatchable systems (network isolation, monitoring)
- [ ] Legacy system inventory with risk acceptance documentation

### Monitoring
- [ ] Passive network monitoring (Claroty, Nozomi, Dragos, or open-source)
- [ ] Baseline of normal process values established
- [ ] Alerts for unauthorized changes to PLC ladder logic
- [ ] Historian data integrity monitoring
- [ ] Physical access logging for control rooms

### Incident Response (OT-specific)
- [ ] IR playbook that accounts for safety requirements (no abrupt shutdown)
- [ ] Coordination with plant safety officer before any OT IR action
- [ ] Manual operation fallback procedure documented
- [ ] OT-specific forensics capability (Dragos, specialized tools)
- [ ] Regulatory notification requirements defined (BSI, sector regulator)

---

## Common OT Attack Vectors

| Vector | Description | Example |
|--------|-------------|---------|
| IT-to-OT lateral movement | Compromise IT, pivot to OT network | Industroyer, Triton |
| Malicious insider | Disgruntled employee with PLC access | Multiple water utility cases |
| Supply chain | Compromised vendor software/hardware | SolarWinds-style on OT |
| Remote access abuse | Vendor VPN credentials stolen | Colonial Pipeline |
| Unpatched vulnerabilities | CVEs in SCADA/HMI software | Eternal Blue on old Windows HMIs |
| USB/removable media | Air-gap bridging | Stuxnet |

---

## Notable OT/ICS Incidents

| Year | Incident | Impact |
|------|----------|--------|
| 2010 | Stuxnet | Iranian nuclear centrifuges destroyed |
| 2015/16 | Ukraine Power Grid | 225,000 customers without power |
| 2017 | Triton/TRISIS | Safety systems targeted (petrochemical) |
| 2021 | Oldsmar Water Plant | Attacker increased sodium hydroxide 111x |
| 2021 | Colonial Pipeline | 5,500 miles of pipeline shut down |
| 2022 | Industroyer2 | Ukraine power grid (thwarted) |

---

## MITRE ATT&CK for ICS

Key tactics specific to ICS:
- **T0883** — Internet Accessible Device (initial access)
- **T0865** — Spearphishing Attachment targeting OT engineers
- **T0845** — Program Upload (stealing PLC code)
- **T0836** — Modify Parameter (changing process setpoints)
- **T0813** — Denial of Control
- **T0828** — Loss of Productivity and Revenue
- **T0879** — Damage to Property

Reference: https://attack.mitre.org/matrices/ics/
