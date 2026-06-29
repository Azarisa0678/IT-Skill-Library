# Compliance Frameworks Quick Reference

## ISO/IEC 27001:2022 — Control Domains

| Domain | Controls (A.x) | Key focus |
|--------|---------------|-----------|
| Organizational | A.5 | Policies, roles, threat intelligence, supplier security |
| People | A.6 | Screening, terms, awareness, remote work |
| Physical | A.7 | Physical entry, clear desk, equipment security |
| Technological | A.8 | Access control, cryptography, secure dev, vulnerability mgmt |

**Certification steps:** Gap assessment → ISMS design → Risk treatment → Internal audit → Stage 1 audit → Stage 2 audit → Surveillance audits

---

## NIST Cybersecurity Framework (CSF) 2.0

| Function | Description | Key categories |
|----------|-------------|----------------|
| **GOVERN** | Risk strategy, roles, policy | GV.OC, GV.RM, GV.RR |
| **IDENTIFY** | Asset & risk awareness | ID.AM, ID.RA, ID.IM |
| **PROTECT** | Safeguards | PR.AA, PR.AT, PR.DS, PR.PS |
| **DETECT** | Anomaly detection | DE.CM, DE.AE |
| **RESPOND** | Incident response | RS.MA, RS.AN, RS.CO |
| **RECOVER** | Restoration | RC.RP, RC.CO |

---

## CIS Controls v8

### Implementation Group 1 (IG1) — Essential (all orgs)
1. Inventory & Control of Enterprise Assets
2. Inventory & Control of Software Assets
3. Data Protection
4. Secure Configuration of Enterprise Assets & Software
5. Account Management
6. Access Control Management

### IG2 adds (mid-size orgs):
7. Continuous Vulnerability Management
8. Audit Log Management
9. Email & Web Browser Protections
10. Malware Defenses
11. Data Recovery

### IG3 adds (mature orgs):
12–18: Network monitoring, penetration testing, incident response, security awareness program

---

## SOC 2 — Trust Services Criteria

| Category | Criteria | Common controls |
|----------|----------|-----------------|
| Security (CC) | CC6–CC9 | MFA, encryption, access reviews, IDS |
| Availability (A) | A1 | Uptime monitoring, DR plan, capacity mgmt |
| Confidentiality (C) | C1 | Data classification, NDA, encryption at rest |
| Processing Integrity (PI) | PI1 | Input validation, error handling, monitoring |
| Privacy (P) | P1–P8 | DSAR process, consent, retention limits |

**Type I:** Controls designed adequately at a point in time.
**Type II:** Controls operating effectively over a period (typically 6–12 months).

---

## PCI-DSS v4.0 — 12 Requirements

| # | Requirement |
|---|-------------|
| 1 | Install & maintain network security controls |
| 2 | Apply secure configurations |
| 3 | Protect stored account data |
| 4 | Protect cardholder data in transit |
| 5 | Protect against malicious software |
| 6 | Develop & maintain secure systems |
| 7 | Restrict access by business need |
| 8 | Identify users & authenticate access |
| 9 | Restrict physical access |
| 10 | Log & monitor all access |
| 11 | Test security regularly |
| 12 | Support information security with policies |

---

## GDPR Key Articles for Security

| Article | Topic |
|---------|-------|
| Art. 5 | Data processing principles (integrity & confidentiality) |
| Art. 25 | Data protection by design and by default |
| Art. 32 | Technical & organisational security measures |
| Art. 33 | Breach notification to supervisory authority (72 hours) |
| Art. 34 | Breach notification to data subjects |
| Art. 35 | Data Protection Impact Assessment (DPIA) |

**Art. 32 measures typically include:** encryption, pseudonymisation, ongoing confidentiality/integrity/availability, regular testing.

---

## Cross-Framework Mapping (Sample)

| Topic | ISO 27001 | NIST CSF | CIS Control | SOC 2 |
|-------|-----------|----------|-------------|-------|
| Access control | A.8.3 | PR.AA | CIS 5, 6 | CC6.1 |
| Vulnerability management | A.8.8 | ID.RA | CIS 7 | CC7.1 |
| Logging & monitoring | A.8.15, A.8.16 | DE.CM | CIS 8 | CC7.2 |
| Incident response | A.5.26 | RS.MA | CIS 17 | CC7.3 |
| Encryption | A.8.24 | PR.DS | CIS 3 | C1.1 |
| Supplier / third-party | A.5.19–5.22 | GV.SC | CIS 15 | CC9.2 |
