# NIS2 & DORA Compliance Reference

## NIS2 — Network and Information Security Directive 2

### Overview
NIS2 (EU 2022/2555) replaces NIS1 and significantly expands scope. Effective from **October 18, 2024** (transposition deadline). Enforced by national authorities (Germany: BSI).

### Scope — Are You Covered?

**Essential Entities (EE)** — stricter supervision:
- Energy, Transport, Banking, Financial Market Infrastructure
- Health, Drinking Water, Wastewater, Digital Infrastructure
- Public Administration, Space
- Operators of Critical Infrastructure (KRITIS)

**Important Entities (IE)** — lighter supervision:
- Postal & Courier Services, Waste Management
- Chemicals, Food Production
- Manufacturing (medical devices, computers, motor vehicles)
- Digital Providers (search engines, social networks, online marketplaces)
- Research organizations

**Size thresholds** (both must apply):
- Medium: 50+ employees OR €10M+ turnover
- Large: 250+ employees OR €50M+ turnover

*Note: Some sectors have no size threshold (e.g. DNS, TLD registries, public administration)*

### NIS2 Core Requirements (Art. 21)

| Requirement | Description | Typical Controls |
|-------------|-------------|-----------------|
| Risk management policies | Systematic risk assessment & treatment | ISO 27005, ISMS |
| Incident handling | Detection, response, reporting procedures | IR playbook, SIEM |
| Business continuity | BCM, DR, crisis management | BCP/DRP, backups |
| Supply chain security | Assess & manage supplier risks | Vendor assessments, contracts |
| Network & system security | Secure architecture, vulnerability mgmt | Hardening, patching |
| Security in procurement | Security requirements for systems/services | Secure procurement policy |
| Cybersecurity hygiene | Awareness training, basic hygiene | Awareness program |
| Cryptography policies | Encryption standards and key management | Crypto policy |
| HR security & access control | Vetting, MFA, least privilege | IAM, PAM |
| Asset management | Inventory of systems and data | CMDB, asset register |
| Multi-factor authentication | MFA for all access where feasible | MFA rollout |
| Secure communications | Encrypted voice/video/text | E2E encryption |

### NIS2 Incident Reporting Obligations

| Timeline | Action |
|----------|--------|
| **24 hours** | Early warning to national CSIRT / competent authority |
| **72 hours** | Incident notification with initial assessment (severity, impact, indicators) |
| **1 month** | Final report with root cause, remediation, cross-border impact |

**Reportable incidents:** Significant incidents that cause or can cause severe operational disruption or financial loss, or affect other entities or persons.

**Severity threshold:** Significant disruption to the service, or affects 5%+ of users / turnover.

### NIS2 Penalties

| Entity Type | Maximum Fine |
|-------------|-------------|
| Essential Entities | €10 million or 2% of global turnover |
| Important Entities | €7 million or 1.4% of global turnover |

Personal liability: Management can be held personally liable for negligence.

### NIS2 Gap Assessment Template

```markdown
## NIS2 Gap Assessment — [Organisation]
Date: [Date] | Assessor: [Name] | Entity Type: [Essential/Important]

| Art. 21 Requirement | Current State | Gap | Priority | Remediation |
|--------------------|---------------|-----|----------|-------------|
| Risk management policies | Partial — no formal ISMS | High | Critical | Implement ISO 27001-aligned ISMS |
| Incident handling | IR playbook exists, not tested | Medium | High | Run tabletop exercise Q1 |
| Business continuity | BCP documented, DR untested | Medium | High | DR test within 6 months |
| Supply chain security | No formal vendor assessment | High | Critical | Vendor risk program |
| MFA | Deployed for O365, not VPN | Medium | High | MFA rollout to all systems |
| Awareness training | Annual generic training | Low | Medium | Role-specific training |
| Cryptography policy | No formal policy | Medium | High | Define and publish policy |
| Asset management | Partial CMDB | Medium | High | Complete asset inventory |
```

---

## DORA — Digital Operational Resilience Act

### Overview
DORA (EU 2022/2554) applies to financial entities and their ICT third-party providers. **Applicable from January 17, 2025.**

### Scope
Financial entities including:
- Banks, investment firms, credit institutions
- Insurance / reinsurance companies
- Payment institutions, e-money institutions
- Trading venues, central counterparties
- Crypto-asset service providers (under MiCA)
- ICT third-party service providers (direct obligations)

### DORA Five Pillars

#### 1. ICT Risk Management (Art. 5–16)
- Documented ICT risk management framework
- Asset inventory covering all ICT assets supporting business functions
- Business impact analysis (BIA) for critical functions
- ICT-related incident management procedures
- Backup, recovery, and restoration procedures
- Testing of ICT continuity plans

**Key deliverable:** ICT Risk Management Framework document

#### 2. ICT Incident Management & Reporting (Art. 17–23)

| Classification | Criteria |
|----------------|----------|
| Major incident | High number of clients affected; significant financial losses; long duration; wide geographic spread; data integrity impact |
| Non-major | Everything below major threshold |

**Reporting timelines for Major ICT Incidents:**

| Timeline | Report | Content |
|----------|--------|---------|
| 4 hours | Initial notification | Detection time, nature, initial impact |
| 72 hours | Intermediate report | Updated impact, preliminary root cause, mitigation |
| 1 month | Final report | Root cause, full impact, permanent fix, lessons learned |

#### 3. Digital Operational Resilience Testing (Art. 24–27)

| Test Type | Frequency | Who |
|-----------|-----------|-----|
| Basic ICT testing (vulnerability scans, gap analysis) | Annual | All in-scope entities |
| Threat-Led Penetration Testing (TLPT) | Every 3 years | Significant entities only |

**TLPT** is based on the TIBER-EU framework — red team testing based on real threat intelligence.

#### 4. ICT Third-Party Risk Management (Art. 28–44)
- Maintain a register of all ICT third-party providers
- Classify providers by criticality
- Contractual requirements (Art. 30):
  - Service level descriptions
  - Data location and portability
  - Audit rights
  - Exit strategies
  - Incident notification obligations
- Report Critical ICT Third-Party Providers to competent authority

**ICT Third-Party Register columns (minimum):**
- Provider name, country, service description
- Criticality classification
- Contract start/end date
- Data categories processed
- Sub-contractors (if applicable)
- Last audit date

#### 5. Information Sharing (Art. 45)
Voluntary sharing of cyber threat intelligence between financial entities (within trust communities). Encouraged but not mandatory.

### DORA vs. NIS2 Relationship

| Aspect | NIS2 | DORA |
|--------|------|------|
| Sector | Cross-sector | Financial sector only |
| Focus | General cybersecurity | Operational resilience + ICT risk |
| Lex specialis | General law | Overrides NIS2 for financial entities |
| TLPT | Not required | Required (significant entities) |
| Third-party register | Not mandated | Mandatory |
| Incident reporting | 24h/72h/1 month | 4h/72h/1 month |

### DORA Implementation Roadmap

```
Phase 1 — Foundation (Months 1–3):
[ ] Establish ICT Risk Management Framework
[ ] Complete ICT asset inventory
[ ] Classify critical business functions
[ ] Gap assessment against DORA requirements

Phase 2 — Processes (Months 3–6):
[ ] Document ICT incident management procedure
[ ] Build ICT third-party register
[ ] Draft/update contracts with Art. 30 requirements
[ ] Define backup and recovery procedures

Phase 3 — Testing (Months 6–9):
[ ] Run basic ICT resilience tests (vulnerability scans, BCP test)
[ ] Conduct tabletop exercise for major ICT incident
[ ] Test backup/recovery procedures

Phase 4 — Advanced (Months 9–12):
[ ] Prepare for TLPT (if significant entity)
[ ] Establish threat intelligence sharing connections
[ ] Internal audit against DORA requirements
[ ] Board-level reporting on ICT risk posture
```

---

## Cross-Framework: NIS2 + DORA + ISO 27001

| Topic | NIS2 Art. | DORA Art. | ISO 27001 |
|-------|-----------|-----------|-----------|
| Risk management | 21(2)(a) | 6–9 | A.6.1, Clause 6 |
| Asset management | 21(2)(i) | 8 | A.8.1 |
| Access control / MFA | 21(2)(j) | 9 | A.8.3 |
| Incident management | 21(2)(b), 23 | 17–19 | A.5.26 |
| Business continuity | 21(2)(c) | 11–12 | A.5.30 |
| Supply chain / third party | 21(2)(d) | 28–44 | A.5.19–5.22 |
| Security awareness | 21(2)(g) | 13 | A.6.3 |
| Cryptography | 21(2)(h) | 9 | A.8.24 |
| Vulnerability management | 21(2)(e) | 10 | A.8.8 |
| Penetration testing | — | 24–27 | A.8.8 |
