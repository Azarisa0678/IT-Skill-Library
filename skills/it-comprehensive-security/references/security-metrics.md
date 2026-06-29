# Security Metrics & KPIs

## Why Metrics Matter

Security metrics translate technical work into business language, justify investments, and enable continuous improvement. Good metrics are **actionable, measurable, and tied to risk**.

---

## Core Operational Metrics

### Incident Response

| Metric | Formula | Target | Why it matters |
|--------|---------|--------|----------------|
| **MTTD** (Mean Time to Detect) | Avg time from breach to detection | < 24h | Shorter = less dwell time |
| **MTTR** (Mean Time to Respond) | Avg time from detection to containment | < 4h (critical) | Speed limits blast radius |
| **MTTC** (Mean Time to Contain) | Avg time to fully contain incident | < 8h (critical) | Stops lateral movement |
| **False Positive Rate** | FP alerts / total alerts | < 10% | SOC efficiency |
| **Alert-to-Ticket Ratio** | Tickets created / alerts fired | Baseline + trend | Noise reduction |
| **Incidents by Severity** | Count per P1/P2/P3/P4 | Trend downward | Overall posture |

### Vulnerability Management

| Metric | Formula | Target |
|--------|---------|--------|
| **Patch SLA Compliance** | Assets patched within SLA / total assets | > 95% |
| **Critical CVE Mean Time to Patch** | Days from CVE publish to patch | < 7 days |
| **Vulnerability Density** | Vulns per 1,000 assets | Trend downward |
| **Scan Coverage** | Assets scanned / total known assets | > 98% |
| **Risk Score Trend** | Aggregated CVSS score over time | Trending down |
| **Exploited CVEs Open** | CVEs on CISA KEV that are unpatched | 0 (immediate priority) |

### Access & Identity

| Metric | Formula | Target |
|--------|---------|--------|
| **MFA Adoption Rate** | Users with MFA / total users | 100% |
| **Privileged Account Count** | Number of admin accounts | Minimize; trend down |
| **Access Review Completion** | Reviews completed on time / scheduled | 100% |
| **Orphaned Account Rate** | Accounts for departed users / total | 0% |
| **Password Age Compliance** | Accounts within policy / total | 100% |
| **Service Account Inventory** | Documented SAs / total SAs | 100% |

### Awareness & Phishing

| Metric | Formula | Target |
|--------|---------|--------|
| **Phishing Simulation Click Rate** | Users who clicked / users targeted | < 5% |
| **Phishing Report Rate** | Users who reported / users targeted | > 70% |
| **Training Completion Rate** | Completed / assigned | > 95% |
| **Repeat Clickers** | Users who clicked in 2+ simulations | 0% (requires coaching) |

---

## Security Posture Metrics

### Asset Management

| Metric | Target |
|--------|--------|
| Asset Inventory Completeness | > 98% of known assets documented |
| Unmanaged Asset Discovery Rate | Trend toward 0 |
| EOL/EOS Software Inventory | 100% documented with risk acceptance |
| Shadow IT Detection Rate | Trend toward 0 |

### Cloud Security (CSPM)

| Metric | Target |
|--------|--------|
| CIS Benchmark Compliance Score | > 85% per cloud account |
| Public S3/Blob Buckets | 0 (unless intentional) |
| Unencrypted Storage Volumes | 0 |
| IAM Users with Console + No MFA | 0 |
| High-Severity CSPM Findings | Resolved within 7 days |

---

## Security KPI Dashboard Template

```markdown
## Security KPI Report — [Month/Quarter]
**Reporting Period:** [Start] – [End]
**Prepared by:** [Name/Team]

### Executive Summary
[2-3 sentences: overall posture, key improvements, key risks]

### Incident Response
| Metric | This Period | Last Period | Trend | Target |
|--------|------------|------------|-------|--------|
| MTTD (avg hours) | | | ↑/↓ | < 24h |
| MTTR (avg hours) | | | ↑/↓ | < 4h |
| P1 Incidents | | | ↑/↓ | 0 |
| P2 Incidents | | | ↑/↓ | < 2 |

### Vulnerability Management
| Metric | This Period | Last Period | Trend | Target |
|--------|------------|------------|-------|--------|
| Critical CVE MTTP (days) | | | ↑/↓ | < 7 |
| Patch SLA Compliance | | | ↑/↓ | > 95% |
| Scan Coverage | | | ↑/↓ | > 98% |

### Identity & Access
| Metric | Current | Last Period | Trend | Target |
|--------|---------|------------|-------|--------|
| MFA Adoption | | | ↑/↓ | 100% |
| Orphaned Accounts | | | ↑/↓ | 0 |
| Privileged Accounts | | | ↑/↓ | Minimize |

### Awareness
| Metric | This Period | Last Period | Trend | Target |
|--------|------------|------------|-------|--------|
| Phishing Click Rate | | | ↑/↓ | < 5% |
| Training Completion | | | ↑/↓ | > 95% |

### Top 3 Risks This Period
1. [Risk] — [Mitigation status]
2. [Risk] — [Mitigation status]
3. [Risk] — [Mitigation status]

### Actions Required from Leadership
- [Decision needed / resource required]
```

---

## Security Maturity Scoring (Simplified)

Score each domain 1–5:

| Domain | 1 (Initial) | 3 (Defined) | 5 (Optimizing) |
|--------|------------|------------|----------------|
| Asset Management | No inventory | Partial CMDB | Full automated inventory |
| Vuln Management | Ad hoc scans | Regular scans + SLA | Risk-based, automated prioritization |
| Identity | Shared accounts | MFA for some | PIM, FIDO2, Zero Trust |
| Incident Response | No playbooks | Basic playbooks | Automated + tested regularly |
| Security Awareness | No training | Annual training | Continuous, role-based, simulated |
| Cloud Security | No controls | CIS partial | Full CSPM + automated remediation |
| Threat Intelligence | No TI | Feed subscription | TI-driven hunting + sharing |

**Overall score:** Sum / (domains × 5) × 100 = Maturity %

---

## Reporting Cadence

| Audience | Frequency | Content |
|----------|-----------|---------|
| SOC / Security Team | Daily | Alert volumes, open incidents, patch status |
| IT Management | Weekly | KPI trends, open P1/P2, top vulnerabilities |
| CISO | Monthly | Full KPI dashboard, risk register update |
| Board / Executive | Quarterly | Strategic posture, business risk, investment needs |
| Regulators (NIS2/DORA) | As required | Incident reports, compliance status |

---

## PowerShell: Automated Metrics Collection

```powershell
# Collect basic security metrics from Active Directory
Import-Module ActiveDirectory

$report = [PSCustomObject]@{
    Date                  = Get-Date -Format 'yyyy-MM-dd'
    TotalUsers            = (Get-ADUser -Filter *).Count
    EnabledUsers          = (Get-ADUser -Filter {Enabled -eq $true}).Count
    UsersNoMFA            = 0  # Fill from Entra ID / MFA report
    PrivilegedAccounts    = (Get-ADGroupMember 'Domain Admins' -Recursive).Count
    StaleAccounts90Days   = (Get-ADUser -Filter {
        LastLogonDate -lt (Get-Date).AddDays(-90) -and Enabled -eq $true
    } -Properties LastLogonDate).Count
    PasswordNeverExpires  = (Get-ADUser -Filter {
        PasswordNeverExpires -eq $true -and Enabled -eq $true
    } -Properties PasswordNeverExpires).Count
}

$report | Export-Csv "C:\Reports\SecurityMetrics_$(Get-Date -f yyyyMMdd).csv" `
    -NoTypeInformation -Append
$report | Format-Table -AutoSize
```
