---
name: it-comprehensive-security
description: >
  Full-spectrum IT security — pentesting, IR, hardening, forensics, malware, phishing, BEC,
  SIEM, Sigma-Regeln, Sentinel, Splunk, threat intelligence, MITRE ATT&CK, DevSecOps,
  SAST/DAST, container/Kubernetes, Zero Trust, ZTNA, CSPM, cloud AWS/Azure/GCP,
  PowerShell, AD/Entra ID, PAM, NIS2, DORA, IEC 62443, KRITIS, BSI Grundschutz, ISO 27001.
  Healthcare: Krankenhaus, KIS, PACS, DICOM, HL7, FHIR, Medizingeraet, IoMT, MDR, TI-Konnektor.
  Finance: DORA, BAIT, VAIT, PCI DSS, SWIFT, TIBER-DE, MaRisk, BaFin, Fraud, AML.
  Energy: Stadtwerke, Stromnetz, Wasserwerk, IEC 62351, BDEW, KRITIS Energie.
  Public Sector: Behoerde, BSI Grundschutz Kommunen, EVB-IT, VS-NfD, eGovernment.
  Pharma: GxP, CSV, GAMP 5, Annex 11, 21 CFR Part 11, Audit Trail, Validierung.
  Manufacturing: TISAX, VDA ISA, OPC UA, IEC 62443 Zones, Industrie 4.0.
  Always use for ANY security topic. Outputs: reports DE/EN, checklists, scripts, playbooks.
---

# IT Comprehensive Security Skill

A balanced, audience-aware security skill covering offensive, defensive, governance, OT/ICS,
forensics, threat intelligence, Zero Trust, and DevSecOps domains - in German and English.

## Guiding Principles

**Know your audience.** A sysadmin asking "how do I harden SSH?" needs a concrete config snippet. A CISO asking the same wants policy language and compliance mapping. Read context cues (role, vocabulary, tool mentions) and adjust depth, tone, and output format accordingly.

**Be actionable.** Avoid vague advice. Every recommendation should answer: *what exactly do I do, and why does it matter?* Prefer concrete commands, configs, and templates over abstract guidance.

**Be complete but scoped.** Don't dump every possible control — focus on what's relevant to the user's specific environment, threat model, and maturity level.

**Ethics always.** All offensive security guidance is for authorized testing, research, and education. Remind users of this briefly when relevant (e.g., for pentest scripts or exploit explanations). Never generate weaponized malware, working exploits targeting production systems, or tools designed for unauthorized access.

---

## Output Formats

Match the output format to the task:

| Task type | Default format |
|---|---|
| Pentest / audit report | See **Report Template** below |
| Hardening / configuration | Annotated code blocks + explanation |
| Incident response | Step-by-step numbered playbook |
| Compliance / policy | Structured markdown with control IDs |
| Threat model | STRIDE table or attack tree |
| Scripts / automation | Commented code with usage instructions |
| Checklists | Checkbox markdown `- [ ] item` |
| Risk assessment | Risk register table |

---

## Domain Guides

### 1. Penetration Testing & Red Team

When helping with pentests:
- Always clarify scope and authorization before producing tools or techniques
- Structure findings by severity: **Critical → High → Medium → Low → Informational**
- For each finding include: Description, Evidence/PoC, Impact, Remediation
- Reference CVEs, CWEs, or OWASP categories where applicable
- Use MITRE ATT&CK technique IDs (e.g., T1059.001) when relevant

**Common pentest phases to cover:**
1. Reconnaissance (OSINT, network scanning)
2. Enumeration (services, users, shares)
3. Exploitation
4. Post-exploitation / lateral movement
5. Reporting

For scripts and tools, prefer well-known open-source options (nmap, Metasploit, Burp Suite, Impacket, BloodHound, etc.) over writing novel exploits.

---

### 2. Defensive Security & Hardening

When hardening systems, structure output as:

```
## [System/Service] Hardening Guide

### Why this matters
<brief threat context>

### Configuration changes
<annotated config snippets>

### Verification
<how to test/confirm the control is in place>

### References
<CIS Benchmark, vendor docs, etc.>
```

Cover these layers when relevant: OS, network, application, identity, logging/monitoring.

**Key hardening topics by platform:**
- **Linux**: SSH config, sudo rules, auditd, AppArmor/SELinux, kernel parameters (sysctl)
- **Windows**: GPO, Defender, WEF, LAPS, PowerShell constrained language mode
- **Cloud (AWS/GCP/Azure)**: IAM least privilege, security groups, CloudTrail/logging, secrets management
- **Web apps**: CSP headers, input validation, auth controls, dependency scanning
- **Network**: firewall rules, VLAN segmentation, IDS/IPS tuning

---

### 3. Incident Response

For IR playbooks, use this structure:

```
## Incident: [Type]

### Detection signals
- IOCs / alert types that indicate this incident

### Immediate containment (first 30 min)
- [ ] Step 1
- [ ] Step 2

### Investigation
- [ ] Evidence to collect
- [ ] Key questions to answer
- [ ] Tools to use

### Eradication
- [ ] Remove threat actor persistence
- [ ] Patch / remediate root cause

### Recovery
- [ ] Restore affected systems
- [ ] Validate integrity

### Post-incident
- [ ] Timeline reconstruction
- [ ] Lessons learned
- [ ] Report to stakeholders
```

Common IR scenarios: ransomware, phishing/BEC, data exfiltration, credential compromise, insider threat, DDoS, supply chain compromise.

---

### 4. Governance, Risk & Compliance (GRC)

When writing policies, controls, or compliance mappings:
- Use the user's framework (ISO 27001, NIST CSF, CIS Controls, SOC 2, GDPR, PCI-DSS, HIPAA)
- Map controls explicitly: "CIS Control 5.1 | ISO 27001 A.9.1.1"
- Write policies in plain language; avoid legal jargon unless the user is writing formal documentation
- Include: Purpose, Scope, Policy Statement, Responsibilities, Enforcement, Review cycle

**Risk Register template:**

| Risk ID | Description | Likelihood (1-5) | Impact (1-5) | Risk Score | Owner | Treatment | Status |
|---------|-------------|-----------------|--------------|------------|-------|-----------|--------|
| R-001 | ... | 3 | 4 | 12 | IT | Mitigate | Open |

---

### 5. DevSecOps & Secure Development

For DevSecOps tasks:
- Recommend security gates appropriate to the pipeline stage (pre-commit → CI → CD → runtime)
- Prefer tools that integrate natively (GitHub Actions, GitLab CI, Jenkins)
- Cover: SAST, DAST, SCA (dependency scanning), secrets detection, container scanning, IaC scanning

**Pipeline security checklist:**
- [ ] Secrets scanning (Gitleaks, truffleHog)
- [ ] SAST (Semgrep, Bandit, SonarQube)
- [ ] Dependency/SCA (Snyk, Dependabot, OWASP Dependency-Check)
- [ ] Container image scanning (Trivy, Grype)
- [ ] IaC scanning (Checkov, tfsec)
- [ ] DAST on staging (OWASP ZAP, Nuclei)
- [ ] Signed commits and artifact signing

For secure code review, flag issues by CWE category (e.g., CWE-89 SQL Injection, CWE-79 XSS).

---

## Pentest / Audit Report Template

Use this when producing formal security reports:

```markdown
# Security Assessment Report
**Target:** [System/Application/Network]
**Assessment Type:** [Internal/External Pentest | Vulnerability Assessment | Code Review]
**Date:** [Date]
**Assessor:** [Name/Team]
**Classification:** CONFIDENTIAL

---

## Executive Summary
[2-3 paragraph non-technical summary: what was tested, key risk posture, top priorities]

## Scope & Methodology
- **In scope:** ...
- **Out of scope:** ...
- **Methodology:** [PTES / OWASP Testing Guide / NIST 800-115]
- **Tools used:** ...

## Risk Summary

| Severity | Count |
|----------|-------|
| Critical | 0 |
| High | 0 |
| Medium | 0 |
| Low | 0 |
| Informational | 0 |

## Findings

### [CRIT-001] Finding Title
- **Severity:** Critical
- **CVSS Score:** 9.8
- **CWE/CVE:** CWE-xxx / CVE-xxxx-xxxx
- **Affected component:** ...
- **Description:** ...
- **Evidence:** [screenshot, log snippet, PoC command]
- **Impact:** ...
- **Remediation:** ...
- **References:** ...

---

## Remediation Roadmap
[Prioritized table of fixes with suggested timelines]

## Appendix
[Raw tool output, methodology details, glossary]
```

---

## Threat Modeling (STRIDE)

When threat modeling, produce a STRIDE table:

| Threat | Category | Component | Mitigation | Priority |
|--------|----------|-----------|------------|----------|
| Attacker intercepts API token | Information Disclosure | API Gateway | TLS + short-lived tokens | High |
| Unauthenticated user calls admin endpoint | Elevation of Privilege | Auth middleware | RBAC enforcement | Critical |

Follow up with an attack surface summary and top 3–5 prioritized mitigations.

---

## Scripts & Automation

When writing security scripts:
1. Add a header comment: purpose, usage, requirements, author
2. Include `--help` / argument parsing where applicable
3. Add error handling and safe defaults (e.g., read-only by default for recon scripts)
4. Note any permissions or elevated rights required
5. Include a brief usage example

Languages: default to Python for portability; use Bash for simple sysadmin tasks; PowerShell for Windows environments; Go/Rust if performance matters.

---

## Quick Reference: Common Tools by Category

| Category | Tools |
|----------|-------|
| Network scanning | nmap, masscan, netdiscover |
| Web app testing | Burp Suite, OWASP ZAP, ffuf, sqlmap, nikto |
| Password attacks | Hashcat, John, Hydra, CrackMapExec |
| AD / Windows | BloodHound, Impacket, Mimikatz (authorized use), Rubeus |
| Exploitation | Metasploit, searchsploit |
| Forensics | Volatility, Autopsy, The Sleuth Kit, FTK |
| OSINT | theHarvester, Shodan, Maltego, Recon-ng |
| Monitoring/SIEM | Splunk, Elastic SIEM, Wazuh, Graylog |
| Vulnerability mgmt | Nessus, OpenVAS, Qualys, Nuclei |
| Container/Cloud | Trivy, Prowler, ScoutSuite, Pacu |

---

## PowerShell Security Scripting

PowerShell is the primary tool for Windows security work. Apply these standards to every script produced.

### Script Structure Template

```powershell
<#
.SYNOPSIS
    Short description of what the script does.
.DESCRIPTION
    Detailed description, use cases, assumptions.
.PARAMETER ParameterName
    Description of each parameter.
.EXAMPLE
    .\Script.ps1 -Parameter Value
.NOTES
    Author:      [Name]
    Version:     1.0
    Requires:    PowerShell 5.1+ / 7+; Admin rights if needed
    Authorized use only.
#>

[CmdletBinding()]
param(
    [Parameter(Mandatory)]
    [string]$Target,
    [switch]$WhatIf
)

Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'

function Write-Log {
    param([string]$Message, [string]$Level = 'INFO')
    $ts = Get-Date -Format 'yyyy-MM-dd HH:mm:ss'
    Write-Output "[$ts][$Level] $Message"
}

try {
    Write-Log "Starting script against $Target"
    # logic here
} catch {
    Write-Log "Error: $_" -Level 'ERROR'
    exit 1
}
```

### 1. Security & Hardening (GPO, Defender, Auditing)

```powershell
# Enable ASR rules in audit mode first, then switch to enforce after validation
$ASRRules = @(
    'BE9BA2D9-53EA-4CDC-84E5-9B1EEEE46550', # Block executable content from email
    'D4F940AB-401B-4EFC-AADC-AD5F3C50688A', # Block Office child processes
    '3B576869-A4EC-4529-8536-B80A7769E899', # Block Office executable content
    '92E97FA1-2EDF-4476-BDD6-9DD0B4DDDC7B'  # Block Win32 API calls from macros
)
$ASRRules | ForEach-Object {
    Add-MpPreference -AttackSurfaceReductionRules_Ids $_ `
                     -AttackSurfaceReductionRules_Actions AuditMode
}

# Check Defender status
Get-MpComputerStatus | Select-Object AMRunningMode, RealTimeProtectionEnabled,
    AntivirusSignatureLastUpdated, IsTamperProtected

# Enable key audit categories
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Management" /success:enable /failure:enable
auditpol /set /subcategory:"Process Creation" /success:enable

# GPO via PowerShell
Import-Module GroupPolicy
$GPO = New-GPO -Name "Security-Baseline-Servers"
New-GPLink -Name "Security-Baseline-Servers" -Target "OU=Servers,DC=corp,DC=local"
Set-GPRegistryValue -Name "Security-Baseline-Servers" `
    -Key "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" `
    -ValueName "RestrictAnonymous" -Type DWord -Value 2
```

### 2. Active Directory & Entra ID

```powershell
Import-Module ActiveDirectory

# Passwords never expire
Get-ADUser -Filter {PasswordNeverExpires -eq $true} -Properties PasswordNeverExpires |
    Select-Object Name, SamAccountName, Enabled |
    Export-Csv C:\Reports\NeverExpires.csv -NoTypeInformation

# Privileged group members
@('Domain Admins','Enterprise Admins','Schema Admins') | ForEach-Object {
    Write-Output "=== $_ ==="
    Get-ADGroupMember -Identity $_ -Recursive |
        Get-ADUser -Properties LastLogonDate, Enabled |
        Select-Object Name, SamAccountName, Enabled, LastLogonDate
}

# Stale accounts (90+ days)
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} `
    -Properties LastLogonDate |
    Select-Object Name, SamAccountName, LastLogonDate |
    Export-Csv C:\Reports\StaleAccounts.csv -NoTypeInformation

# Kerberoastable accounts
Get-ADUser -Filter {ServicePrincipalName -ne "$null" -and Enabled -eq $true} `
    -Properties ServicePrincipalName, PasswordLastSet |
    Select-Object Name, SamAccountName, ServicePrincipalName, PasswordLastSet

# Entra ID — users without MFA (Microsoft Graph)
Connect-MgGraph -Scopes "User.Read.All","Policy.Read.All"
Get-MgReportAuthenticationMethodUserRegistrationDetail |
    Where-Object { $_.IsMfaRegistered -eq $false } |
    Select-Object UserPrincipalName, UserDisplayName |
    Export-Csv C:\Reports\NoMFA.csv -NoTypeInformation
```

### 3. Incident Response & Forensics

```powershell
# Live triage snapshot (run as Admin)
$OutputDir = "C:\IR\$(Get-Date -f yyyyMMdd_HHmmss)"
New-Item -ItemType Directory -Path $OutputDir | Out-Null

# Processes with file hashes
Get-Process | ForEach-Object {
    $hash = if ($_.Path -and (Test-Path $_.Path)) {
        (Get-FileHash $_.Path -Algorithm SHA256).Hash
    } else { 'N/A' }
    $_ | Select-Object Id, Name, Path, CPU, StartTime,
        @{N='SHA256';E={$hash}}
} | Export-Csv "$OutputDir\processes.csv" -NoTypeInformation

# Network connections with owning process
Get-NetTCPConnection -State Established |
    Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort,
        @{N='Process';E={(Get-Process -Id $_.OwningProcess -EA 0).Name}} |
    Export-Csv "$OutputDir\connections.csv" -NoTypeInformation

# Scheduled tasks
Get-ScheduledTask | Where-Object State -ne Disabled |
    Select-Object TaskName, TaskPath, State,
        @{N='Actions';E={($_.Actions.Execute) -join '; '}} |
    Export-Csv "$OutputDir\tasks.csv" -NoTypeInformation

# Suspicious: processes from temp/AppData locations
Get-Process | Where-Object {
    $_.Path -match '(Temp|AppData|Downloads|Public)'
} | Select-Object Id, Name, Path

# Base64 encoded command lines (common malware indicator)
Get-WmiObject Win32_Process |
    Where-Object { $_.CommandLine -match '-enc|-encodedcommand|FromBase64' } |
    Select-Object ProcessId, Name, CommandLine

# Key security event IDs (last 24h)
Get-WinEvent -FilterHashtable @{
    LogName   = 'Security','System'
    Id        = @(4624,4625,4648,4697,4698,4720,4732,7045,1102)
    StartTime = (Get-Date).AddHours(-24)
} -EA 0 | Select-Object TimeCreated, Id, Message |
    Export-Csv "$OutputDir\suspicious_events.csv" -NoTypeInformation

Write-Output "Triage complete: $OutputDir"
```

### 4. Automatisierung & DevOps

```powershell
# Secret scanning — check files for hardcoded credentials
$Patterns = @{
    'AWS Key'         = 'AKIA[0-9A-Z]{16}'
    'Private Key'     = '-----BEGIN (RSA|EC|OPENSSH) PRIVATE KEY-----'
    'Password in var' = '(?i)(password|passwd|pwd)\s*=\s*["\x27][^"\x27]{4,}'
    'Bearer Token'    = '(?i)bearer\s+[a-zA-Z0-9\-_\.]{20,}'
}

$Findings = @()
Get-ChildItem -Path "." -Recurse -File -Include *.ps1,*.json,*.yaml,*.yml,*.config,*.env |
    Where-Object { $_.FullName -notmatch '(\.git|node_modules)' } |
    ForEach-Object {
        $content = Get-Content $_.FullName -Raw -EA 0
        foreach ($p in $Patterns.GetEnumerator()) {
            if ($content -match $p.Value) {
                $Findings += [PSCustomObject]@{ File = $_.FullName; Pattern = $p.Key }
            }
        }
    }

if ($Findings) { $Findings | Format-Table -AutoSize; exit 1 }
else { Write-Output "No secrets detected." }

# Patch compliance check
$Session  = New-Object -ComObject Microsoft.Update.Session
$Missing  = $Session.CreateUpdateSearcher().Search("IsInstalled=0 and Type='Software'")
Write-Output "Missing updates: $($Missing.Updates.Count)"
$Missing.Updates | Select-Object Title, MsrcSeverity,
    @{N='KB';E={$_.KBArticleIDs}} | Sort-Object MsrcSeverity | Format-Table -AutoSize
```

### PowerShell Security Best Practices

| Practice | Why it matters |
|----------|----------------|
| `Set-StrictMode -Version Latest` | Catches undefined variables and bad syntax |
| `$ErrorActionPreference = 'Stop'` | Errors are never silently swallowed |
| Sign scripts in production | Prevents tampered script execution |
| Use `-WhatIf` for destructive ops | Safe testing without side effects |
| Avoid `Invoke-Expression` | Common code injection vector |
| Use `SecureString` for credentials | Avoids plaintext passwords in memory |
| Enable Script Block Logging | GPO: `HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging` |
| Constrained Language Mode | Restrict untrusted scripts enterprise-wide |

---

## Cloud Security

When handling cloud security tasks, read `references/cloud-security.md` for CLI commands and checklists. Key principles:

- **Shared Responsibility**: Always clarify what the cloud provider secures vs. the customer — this shapes every recommendation.
- **IAM is the perimeter**: In cloud environments, identity is the new network boundary. Default to least-privilege roles, no long-lived access keys, MFA everywhere.
- **Output structure** for cloud security tasks:
  1. Audit commands (CLI/PowerShell) to assess current state
  2. Hardening steps with concrete commands
  3. Verification commands to confirm the fix
  4. Checklist for the platform (AWS/Azure/GCP)

**Platform-specific defaults:**
- **AWS**: Use AWS CLI + Prowler for audits; GuardDuty + Security Hub for detection; Secrets Manager for credentials
- **Azure**: Use Az CLI + PowerShell (Microsoft.Graph); Defender for Cloud; Sentinel for SIEM
- **GCP**: Use gcloud CLI; Security Command Center; Workload Identity instead of service account keys

---

## NIS2 & DORA Compliance

When handling NIS2 or DORA topics, read `references/nis2-dora.md` for full requirements. Key guidance:

- **Scope check first**: Always establish whether the organization is an Essential Entity, Important Entity, or financial entity under DORA before diving into requirements.
- **NIS2 incident reporting**: 24h early warning → 72h notification → 1 month final report. Always include these timelines when advising on IR procedures.
- **DORA incident reporting**: 4h initial → 72h intermediate → 1 month final. Applies to financial entities only.
- **Gap assessments**: Use the gap assessment template in the reference file; map findings to Art. 21 (NIS2) or the five DORA pillars.
- **DORA third-party register**: Mandatory for all in-scope financial entities — always recommend this as a priority deliverable.
- **Lex specialis**: For financial entities, DORA takes precedence over NIS2 — note this when both could apply.

---

## Container & Kubernetes Security

When handling container/K8s security tasks, read `references/container-kubernetes.md`. Key principles:

- **Layered security**: Apply security at every layer — Dockerfile, image registry, runtime, cluster, network, and supply chain.
- **Least privilege by default**: Non-root containers, dropped capabilities, read-only filesystems, minimal RBAC.
- **Output structure** for container security tasks:
  1. Dockerfile / manifest changes with explanation
  2. Scanning commands (Trivy, Hadolint, kube-bench)
  3. Runtime controls (Falco, admission controllers)
  4. Checklist for Docker or Kubernetes

**Key tool defaults:**
- Image scanning: **Trivy** (covers CVEs, misconfigs, SBOM, IaC)
- K8s auditing: **kube-bench** (CIS benchmark) + **Polaris** (best practices)
- Runtime detection: **Falco**
- Policy enforcement: **Kyverno** or **OPA/Gatekeeper**
- Secrets: **External Secrets Operator** with HashiCorp Vault or cloud KMS

---

## Phishing & Social Engineering

When handling phishing/social engineering topics, read `references/phishing-social-engineering.md` for full guidance. Key outputs:

- **Simulation plans**: Include pretext scenarios, difficulty levels, metrics (click rate, report rate), communication plan, and follow-up training.
- **Awareness content**: Adapt tone and depth to audience (non-technical staff vs. developers vs. executives). DACH context by default for German-language requests.
- **BEC prevention**: Always cover both technical controls (DMARC, external email banners) and process controls (dual-authorization, callback verification).
- **IR for phishing victims**: Use the phishing victim IR checklist — immediate isolation, credential reset, OAuth app audit, email gateway search.

---

## Secure Code Review

When reviewing code for security issues, read `references/secure-code-review.md` for language-specific checklists. General approach:

1. Start with the **General checklist** (auth, authz, input validation, secrets, crypto)
2. Apply the **language-specific checklist** (Python, JavaScript, Java, C#)
3. Flag dangerous functions explicitly with the risk they carry
4. Rate each finding by CWE and severity (Critical → Low)
5. Provide a concrete fix alongside every finding — not just a description of the problem

When producing code review output, use this structure per finding:
```
**[SEVERITY] Finding Title**
- CWE: CWE-xxx
- Location: file.py, line N
- Issue: [What is wrong and why it is dangerous]
- Evidence: [The vulnerable code snippet]
- Fix: [The corrected code snippet]
```

---

## MITRE ATT&CK Integration

When producing security content, map findings and recommendations to ATT&CK where relevant. Read `references/mitre-attack.md` for the full technique reference.

- **Pentest reports**: Add ATT&CK technique IDs (e.g. T1566.001) to each finding
- **IR reports**: Map attacker actions to tactics/techniques in the timeline
- **Hardening guides**: Reference which techniques the control mitigates
- **Threat modeling**: Use ATT&CK tactics as the threat enumeration structure

Always reference the tactic (TA00xx) alongside the technique (Txxxx) for clarity.

---

## Zero Trust Architecture

When handling Zero Trust topics, read `references/zero-trust.md`. Key guidance:

- **Identity first**: Start every ZT implementation with strong MFA (phishing-resistant FIDO2) and Conditional Access — this delivers the most impact per effort.
- **Maturity model**: Use the CISA ZT Maturity Model (Traditional → Initial → Advanced → Optimal) to frame current state and roadmap.
- **Output structure** for Zero Trust tasks: Current state assessment → Gap to desired maturity → Prioritized roadmap by pillar (Identity → Devices → Network → Applications → Data).
- **Platform defaults**: Microsoft (Entra + Intune + Defender) for Microsoft environments; Cloudflare Access/Zscaler ZPA for ZTNA; HashiCorp Vault for workload identity.

---

## OT/ICS Security

When handling OT/ICS/SCADA topics, read `references/ot-ics-security.md`. Critical guidance:

- **Safety before security**: OT environments prioritize availability and safety. Never recommend actions that could disrupt production without explicit authorization and change management.
- **Passive-only scanning**: Active network scanning can crash PLCs and HMIs. Always recommend passive monitoring tools (Claroty, Nozomi, Zeek) for OT asset discovery.
- **Purdue model**: Use as the reference architecture for segmentation recommendations.
- **Standards**: IEC 62443 for industrial systems; BSI Grundschutz OT-Bausteine and KRITIS-Anforderungen for German operators.
- **Air gap vs. connectivity**: Always address the IT/OT boundary explicitly — firewall + DMZ as minimum, data diode for highest-risk unidirectional flows.

---

## Threat Intelligence

When handling threat intelligence topics, read `references/threat-intelligence.md`. Key guidance:

- **Intelligence types**: Match output to consumer — tactical IOCs for SOC analysts, strategic briefings for CISO/board.
- **Platform defaults**: MISP for open-source sharing; OpenCTI for analyst workflow; AlienVault OTX and Abuse.ch feeds for free IOC data.
- **ATT&CK integration**: Always map threat actor TTPs to MITRE ATT&CK techniques when producing intelligence reports.
- **Pyramid of Pain**: Remind users that IP/domain IOCs rotate quickly — TTPs and behavioral detections are more durable.
- **STIX/TAXII**: Use for structured sharing; provide working Python examples when integration is the goal.

---

## Security Metrics & KPIs

When handling security metrics, KPI dashboards, or executive reporting, read `references/security-metrics.md`. Key guidance:

- **Audience-first**: SOC gets operational metrics (MTTD, MTTR, alert volumes); management gets trends and compliance; board gets business risk language.
- **Metric defaults**: MTTD, MTTR, Patch SLA Compliance, MFA Adoption Rate, Phishing Click Rate — these five cover the most ground for most organizations.
- **Reporting template**: Use the structured KPI Report template from the reference for executive reports.
- **Maturity scoring**: Use the simplified 1–5 domain scoring to give a quick posture assessment.
- **PowerShell automation**: Provide the AD metrics collection script when Windows environments are involved.

---

## Digital Forensics & Malware Analysis

When handling forensics or malware analysis topics, read `references/forensics-malware.md`. Key guidance:

- **Chain of custody first**: Always mention evidence integrity (write blockers, hash verification) before diving into analysis steps.
- **Order of volatility**: Structure collection guidance from most volatile (RAM, network state) to least volatile (disk).
- **Tool defaults**: Volatility 3 for memory forensics; Autopsy/Sleuth Kit for disk; YARA for pattern matching; Any.run or CAPE for sandboxing.
- **Static before dynamic**: For malware analysis, always recommend static analysis (strings, hashes, PE structure) before running samples.
- **Safety**: Malware analysis must always be performed in isolated, non-production environments (VM snapshots, air-gapped sandbox).

---

## Healthcare Security

When handling healthcare, hospital, medical device, or clinical IT security topics, read `references/healthcare-security.md`. Key guidance:

- **Regulatorik DACH**: NIS2 für Krankenhäuser ab 30.000 Fällen/Jahr, KRITIS-Dachgesetz, DSGVO Art. 9 für Gesundheitsdaten (besondere Kategorie!), MDR/IVDR für Medizinprodukte, KHZG Fördertatbestand 10
- **IoMT-Sicherheit**: Passives Scanning bevorzugen (aktives Scanning kann Geräte beeinträchtigen), Mikrosegmentierung nach Risikoklasse, SBOM von Herstellern anfordern, Virtual Patching für Legacy-Systeme
- **TI-Sicherheit**: Konnektor immer aktuell halten, eigenes VLAN für TI-Netz, nur PVS/KIS darf mit Konnektor kommunizieren
- **KIS/PACS/HL7**: DICOM-TLS auf Port 2762 statt 104, FHIR R4 mit SMART on FHIR, DB-Verschlüsselung aktivieren
- **Ransomware-IR**: Notbetrieb sofort aktivieren, BSI binnen 24h melden (KRITIS), Datenschutzbehörde 72h, Forensik vor Abschalten
- **Awareness**: Schwerpunkte Phishing auf TI-Dienste, keine Patientendaten per WhatsApp, Zugangsdaten nie teilen

---

## Finance Security

When handling finance, banking, insurance, payment, or FinTech security topics, read `references/finance-security.md`. Key guidance:

- **DORA (ab 17.01.2025)**: 5 Säulen — ICT-Risk, Incident Reporting, TLPT, Drittparteienrisiko, Informationsaustausch; Meldung erheblicher Vorfälle: 4h Erstmeldung / 72h Zwischenmeldung / 30 Tage Abschluss an BaFin
- **BAIT/VAIT/ZAIT/KAIT**: CISO als unabhängige Funktion, ISMS (ISO 27001 oder BSI-Grundschutz), PAM für privilegierte Konten, Rezertifizierung jährlich, kritische Patches ≤ 1 Woche
- **PCI DSS v4.0**: 12 Anforderungsbereiche, CDE-Segmentierung minimieren (Tokenisierung / P2PE), jährliche Penetrationstests inkl. Segmentierungstest, QSA-Audit
- **SWIFT CSCF**: Isoliertes SWIFT-Netz (kein Internet-Direktausgang), MFA für Alliance-Administration, Integrity-Check vor SW-Installation
- **TIBER-DE**: Bedrohungsorientiertes Red-Team-Testing, koordiniert durch Bundesbank, Phasen: Prep → Threat Intel → Red Team → Closure
- **Fraud/AML**: Real-Time-Erkennung < 100ms, Sanctions Screening, AML nach § 25h KWG; Audit-Logging unveränderlich

---

## Energy & Utility Security

When handling energy, utility, water, gas, or KRITIS-Energie topics, read `references/energy-security.md`. Key guidance:

- **KRITIS-Schwellen**: Strom ≥ 420 MW / Stromverteilung ≥ 3,7 Mio. Anschlüsse / Wasser ≥ 500.000 Einwohner → KRITIS-Pflichten (BSI-Registrierung, 24/7-Kontaktstelle, Nachweis alle 2 Jahre)
- **IT-Sicherheitskatalog BNetzA**: ISO 27001 Zertifizierung Pflicht für Strom-/Gasnetzbetreiber (§ 11 Abs. 1a EnWG) — Scope umfasst alle Systeme für Netzsteuerung
- **IEC 62351**: Sicherheitsstandard für Energieprotokolle — Part 5 für IEC 60870-5-104/DNP3, Part 6 für IEC 61850 GOOSE/MMS; Retrofit-Option: Kryptoappliances oder VPN als Kompensation
- **BDEW-Whitepaper**: Branchenstandard DE Energie — Segmentierung IT/OT/Fernwirk, Whitelisting, Fernzugang nur über Jump-Host mit MFA, passive OT-Überwachung
- **Angriffserkennung (§ 8a Abs. 3 IT-SiG 2.0)**: SAK-Pflicht für UBI/KRITIS, muss IT und OT abdecken
- **IR Energiesektor**: Physischen Notbetrieb sichern BEVOR OT-Isolation; Meldung BSI + BNetzA; Hersteller-Hotlines (Siemens/ABB/Schneider) einbinden

---

## Public Sector Security

When handling public administration, municipality, government agency, or Behörden IT security topics, read `references/public-sector.md`. Key guidance:

- **BSI IT-Grundschutz**: Bundesbehörden: Pflicht; Kommunen: dringend empfohlen — Drei Absicherungsvarianten: Basis (Einstieg) → Standard (Regelfall, Voraussetzung ISO 27001 auf Basis IT-GS) → Kern (Kronjuwelen)
- **VS-NfD**: Nur BSI-zugelassene Produkte (SINA, VS-NfD-PKI); keine normale Cloud, keine normale Email für VS-NfD-Inhalte; Nutzer brauchen Sicherheitsüberprüfung (§ 9 SÜG)
- **EVB-IT**: Richtiger Vertragstyp wählen (System / Dienstleistung / Erstellung / Cloud / Pflege S); IT-Sicherheitsanforderungen ins Leistungsverzeichnis; AVV als Pflichtanlage bei Auftragsverarbeitung
- **Kommunen-Ransomware**: Häufigstes Angriffsszenario; BSI-Unterstützung kostenlos anfragen; kein Lösegeld ohne BSI/BKA-Rücksprache; Notbetrieb mit Papier vorbereiten
- **OZG-Sicherheit**: TLS 1.3, BSI TR-02102 Cipher Suites, HSTS/CSP/X-Frame-Options Pflicht, Penetrationstest vor Go-Live, eID-Anbindung nach BSI TR-03130
- **Beschaffung**: C5-Testat (BSI Cloud Computing Compliance Criteria) für Cloud-Dienste; SBOM-Anforderung in Ausschreibung; Quellcode-Escrow bei Individualsoftware (EVB-IT Erstellung)

---

## German Language Output

When the user communicates in German or requests German-language deliverables, read `references/german-templates.md` and apply:

- Use the German policy templates for Passwortrichtlinie, Homeoffice-Richtlinie, and similar documents
- Use the German Pentest-Bericht template for formal assessment reports
- Use the NIS2-konforme Vorfallmeldung template for incident notifications
- Use the Executive Security Report template for Geschäftsführungs-Berichte
- Awareness content defaults to German (Phishing-Merkblatt, Schulungsinhalte)
- Compliance references use German regulatory context (BSI, BayLDA, BfDI, KRITIS)

---

## References (read as needed)

- `references/compliance-frameworks.md` — ISO 27001, NIST CSF, CIS Controls, SOC 2, PCI-DSS, GDPR mappings
- `references/owasp-top10.md` — OWASP Top 10 (Web + API) with CWE mappings and mitigations
- `references/powershell-security.md` — PowerShell logging, CLM, AMSI, credential handling, key modules
- `references/mitre-attack.md` — Full ATT&CK tactic/technique reference with detection signals
- `references/secure-code-review.md` — Language-specific checklists: Python, JavaScript, Java, C#
- `references/phishing-social-engineering.md` — Simulation planning, BEC prevention, awareness training, IR
- `references/example-reports.md` — Full example pentest report and IR report to use as output templates
- `references/prompt-library.md` — Ready-to-use prompts for common security tasks (DE/EN)
- `references/cloud-security.md` — AWS, Azure, GCP audit commands, checklists, and CSPM tools
- `references/nis2-dora.md` — NIS2 Art. 21 requirements, DORA five pillars, incident timelines, gap templates
- `references/container-kubernetes.md` — Docker hardening, K8s RBAC/PSS/NetworkPolicy, scanning tools
- `references/zero-trust.md` — ZT principles, CISA maturity model, implementation roadmap, Microsoft/Cloudflare ZT
- `references/ot-ics-security.md` — Purdue model, IEC 62443, OT protocols, passive monitoring, KRITIS
- `references/threat-intelligence.md` — STIX/TAXII, MISP, OpenCTI, IOC types, threat actor tracking
- `references/security-metrics.md` — KPIs, MTTD/MTTR, dashboards, maturity scoring, PowerShell collection
- `references/forensics-malware.md` — Volatility, Autopsy, YARA, static/dynamic analysis, chain of custody
- `references/german-templates.md` — DE policy templates, pentest reports, IR Meldung, executive reports
- `references/healthcare-security.md` — KRITIS Krankenhaus, IoMT/Medizingeräte, KIS/PACS/HL7/DICOM, TI-Konnektor, NIS2/DSGVO/MDR, Ransomware-IR Healthcare
- `references/finance-security.md` — DORA, BAIT/VAIT/ZAIT, PCI DSS v4.0, SWIFT CSCF, TIBER-DE, MaRisk, Fraud/AML, Online-Banking/SCA/PSD2
- `references/energy-security.md` — KRITIS Energie/Wasser/Gas, IEC 62351, IEC 62443, BDEW-Whitepaper, IT-Sicherheitskatalog BNetzA, SAK/Angriffserkennung, Smart Meter/SMGW
- `references/public-sector.md` — BSI IT-Grundschutz, VS-NfD/SINA, EVB-IT, OZG-Sicherheit, Kommunen-Ransomware-IR, eGovernment, BSI C5 Cloud

---

## SIEM & Detection Engineering

When handling SIEM, detection rules, threat hunting, Sigma, Splunk, or Sentinel topics, read `references/siem-detection.md`. Key guidance:

- **Sentinel KQL**: Brute Force, Impossible Travel, PIM-Aktivierungen außerhalb Geschäftszeiten, DCSync, Kerberoasting, Ransomware-Indikator, DNS-Exfiltration — fertige Produktions-Queries
- **Sigma-Format**: Detection-as-Code Ansatz — Regeln vendor-neutral schreiben, dann mit `sigma convert` nach Splunk/Sentinel/QRadar konvertieren
- **Detection Maturity Level (DML)**: Ziel DML-5/6 (verhaltensbasiert + TTP-basiert); Einstieg DML-2/3 realistisch
- **Tuning-Prozess**: Baseline 30 Tage → FP-Rate messen → Whitelist dokumentiert → Threshold anpassen → max. 5% FP-Rate vor Produktion
- **Detection-as-Code Pipeline**: Sigma in Git → PR-Review → sigma validate → sigma convert → automatisch in Sentinel/Splunk deployen
- **Threat Hunting**: LOLBins-Hunt (certutil/bitsadmin/mshta mit HTTP/Base64), Persistence via Run Keys, Lateral Movement Patterns

---

## Supply Chain Security (SIEM-Perspektive)

- **SBOM-basierte Erkennung**: Neue CVEs in deployed SBOMs → Sentinel-Alert innerhalb 24h
- **Image-Signatur-Verletzung**: Alert wenn unsigniertes Image in Produktion deployt
- **Dependency Confusion**: DNS/Network Monitoring auf unerwartete npm/pypi Registry-Requests

---

## Pharma & Life Sciences Security

When handling pharma, GxP, CSV, Annex 11, GAMP 5, or life sciences topics, read `references/pharma-security.md`. Key guidance:

- **Regulatorik**: EU GMP Annex 11, GAMP 5 Kategorien 1-5, 21 CFR Part 11, MDR/IVDR, NIS2 als "Wichtige Einrichtung"
- **CSV/Validierung**: URS → Risikoanalyse → IQ/OQ/PQ; kein Excel ohne CSV in GxP; GAMP 5 bestimmt Validierungstiefe
- **Audit Trail**: Unveränderbar, NTP-synchronisiert, auch Lesezugriffe loggen, mind. Produktlebenszyklus + 1 Jahr aufbewahren
- **Change Control**: Jeder Patch = Change Control; Hersteller-Statement einholen; Testumgebung vor Produktiv
- **Zugangskontrolle**: Individuelle Accounts (keine Gruppenlogins!), Session-Timeout 15 Min (kritisch: 5 Min), E-Signaturen nach 21 CFR Part 11

---

## Manufacturing & Industrie 4.0 Security

When handling manufacturing, smart factory, MES, IIoT, or Industrie 4.0 topics, read `references/manufacturing-security.md`. Key guidance:

- **IEC 62443 Zones & Conduits**: Zone 1 Enterprise → Zone 5 DMZ → Zone 2 SCADA → Zone 3 SPS/DCS → Zone 4 Feld; Security Levels SL1-SL4 je Zone festlegen
- **OPC UA Pflicht**: Einziges OT-Protokoll mit nativer Sicherheit (Auth + Verschlüsselung) — alle anderen brauchen Kompensationsmaßnahmen
- **Fernwartung**: Immer über Betreiber-Jump-Host; Session-Recording; zeitlich begrenzt; Lieferanten haben KEINEN eigenen VPN-Zugang
- **TISAX (Automobilindustrie)**: VDA ISA Fragebogen, Assessment durch akkreditierten Auditor, keine 0/1 in Pflicht-Anforderungen, 3 Jahre Gültigkeit
- **NIS2**: Verarbeitendes Gewerbe ab 50 MA = "Wichtige Einrichtung"; Meldepflichten Art. 21 gelten
