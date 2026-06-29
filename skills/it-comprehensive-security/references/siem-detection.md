# SIEM & Detection Engineering — Referenzmodul

Microsoft Sentinel, Splunk, QRadar, Sigma-Regeln, Detection-as-Code, Threat Hunting,
KQL/SPL-Abfragen, Use-Case-Bibliothek für DACH-relevante Angriffsmuster.

---

## Microsoft Sentinel

### Deployment-Architektur

```
Log Analytics Workspace (LAW)
    │
    ├── Sentinel aktiviert
    │
    ├── Data Connectors:
    │   ├── Microsoft 365 Defender (MDE, MDO, MDI, MDA) → nativer Connector
    │   ├── Entra ID (Sign-ins, Audit, Risk Events) → nativer Connector
    │   ├── Azure Activity / Defender for Cloud → nativer Connector
    │   ├── Syslog / CEF (Linux, Firewalls, Network Devices) → AMA Agent
    │   ├── Windows Security Events → AMA Agent
    │   └── Custom REST API Connector (beliebige Quellen)
    │
    ├── Analytics Rules (Erkennung)
    ├── Workbooks (Dashboards)
    ├── Automation (Logic Apps / Playbooks)
    └── Threat Intelligence (TI-Feeds)
```

### KQL — Produktions-Erkennungsregeln

```kql
// ── BRUTE FORCE: Viele fehlgeschlagene Logins + dann Erfolg ────────────────
let threshold_failed = 10;
let lookback = 1h;
SigninLogs
| where TimeGenerated > ago(lookback)
| summarize
    FailedLogins = countif(ResultType != "0"),
    SuccessLogins = countif(ResultType == "0"),
    IPAddresses = make_set(IPAddress),
    Locations = make_set(Location)
  by UserPrincipalName, bin(TimeGenerated, 1h)
| where FailedLogins >= threshold_failed and SuccessLogins >= 1
| extend Severity = "High"
| project TimeGenerated, UserPrincipalName, FailedLogins, SuccessLogins,
          IPAddresses, Locations, Severity

// ── IMPOSSIBLE TRAVEL (Login aus zwei Ländern < 1h) ────────────────────────
let timedelta = 1h;
SigninLogs
| where ResultType == "0"
| where LocationDetails.countryOrRegion != ""
| project UserPrincipalName, TimeGenerated,
          Country = tostring(LocationDetails.countryOrRegion),
          IPAddress
| sort by UserPrincipalName asc, TimeGenerated asc
| serialize
| extend PrevCountry = prev(Country, 1),
         PrevTime    = prev(TimeGenerated, 1),
         PrevUser    = prev(UserPrincipalName, 1)
| where UserPrincipalName == PrevUser
| where Country != PrevCountry
| where TimeGenerated - PrevTime < timedelta
| project UserPrincipalName, TimeGenerated, Country, PrevCountry,
          TimeDiff = TimeGenerated - PrevTime, IPAddress

// ── PIM AKTIVIERUNG AUSSERHALB GESCHÄFTSZEITEN ──────────────────────────────
AuditLogs
| where TimeGenerated > ago(24h)
| where OperationName == "Add member to role completed (PIM activation)"
| extend
    Benutzer   = tostring(InitiatedBy.user.userPrincipalName),
    Rolle      = tostring(TargetResources[0].displayName),
    Stunde     = hourofday(TimeGenerated),
    Wochentag  = dayofweek(TimeGenerated)
| where Stunde < 7 or Stunde > 19    // Außerhalb 07:00-19:00
| where Wochentag in (0, 6)          // oder Wochenende
| project TimeGenerated, Benutzer, Rolle, Stunde, Wochentag, Result

// ── NEW GLOBAL ADMIN ASSIGNED ────────────────────────────────────────────────
AuditLogs
| where OperationName == "Add member to role"
| extend
    Ziel  = tostring(TargetResources[0].userPrincipalName),
    Rolle = tostring(TargetResources[1].displayName),
    Durch = tostring(InitiatedBy.user.userPrincipalName)
| where Rolle == "Global Administrator"
| project TimeGenerated, Ziel, Rolle, Durch, Result

// ── DEFENDER ALERT KORRELATION (Multi-Stage Attack) ─────────────────────────
SecurityAlert
| where TimeGenerated > ago(24h)
| summarize
    AlertCount    = count(),
    AlertTitles   = make_set(AlertName),
    Severity      = max(AlertSeverity),
    Entities      = make_set(Entities)
  by SystemAlertId, ProviderName,
     bin(TimeGenerated, 1h)
| where AlertCount >= 3
| order by AlertCount desc

// ── KERBEROASTING ERKENNUNG ──────────────────────────────────────────────────
SecurityEvent
| where EventID == 4769                          // Kerberos Service Ticket
| where TicketEncryptionType == "0x17"           // RC4 (schwach, Kerberoasting-Indikator)
| where ServiceName !endswith "$"                // Keine Computer-Accounts
| where ServiceName != "krbtgt"
| summarize
    Anzahl    = count(),
    Services  = make_set(ServiceName)
  by Account, IPAddress, bin(TimeGenerated, 1h)
| where Anzahl >= 5
| extend Severity = "High"

// ── LATERAL MOVEMENT: PASS-THE-HASH ─────────────────────────────────────────
SecurityEvent
| where EventID == 4624                          // Erfolgreicher Login
| where LogonType == 3                           // Netzwerk-Login
| where AuthenticationPackageName == "NTLM"     // NTLM statt Kerberos
| where WorkstationName != TargetDomainName      // Cross-Domain
| summarize count() by Account, IpAddress, WorkstationName, bin(TimeGenerated, 1h)
| where count_ >= 3

// ── RANSOMWARE PRECURSOR: VIELE DATEILÖSCHUNGEN ─────────────────────────────
// (FileCreationEvents aus MDE / Defender for Endpoint)
DeviceFileEvents
| where ActionType in ("FileDeleted", "FileRenamed")
| where FileName matches regex @".*\.(docx|xlsx|pdf|pptx|jpg|png)$"
| summarize
    DeletedFiles = count(),
    Directories  = make_set(FolderPath)
  by DeviceName, InitiatingProcessAccountName, bin(Timestamp, 10m)
| where DeletedFiles >= 100
| extend Severity = "Critical"

// ── DNS EXFILTRATION (lange Subdomains) ──────────────────────────────────────
DnsEvents
| where TimeGenerated > ago(1h)
| extend SubdomainLength = strlen(extract(@"^([^.]+)\.", 1, Name))
| where SubdomainLength > 50                     // Sehr lange Subdomains
| where Name !endswith ".microsoft.com"
| summarize
    QueryCount    = count(),
    UniqueQueries = dcount(Name)
  by ClientIP, Computer, bin(TimeGenerated, 10m)
| where UniqueQueries >= 20
```

---

## Sigma-Regeln (Detection-as-Code)

### Format und Konvertierung

```yaml
# Sigma-Grundformat
title: Kerberoasting Angriff erkannt
id: 3f5a9a4e-1234-5678-abcd-ef0123456789
status: stable
description: Erkennt typische Kerberoasting-Muster via RC4-Verschlüsselung
references:
  - https://attack.mitre.org/techniques/T1558/003/
author: Detection-Team Firma
date: 2024/01/15
modified: 2024/06/01
tags:
  - attack.credential_access
  - attack.t1558.003
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4769
    TicketEncryptionType: '0x17'
  filter_computers:
    ServiceName|endswith: '$'
  filter_krbtgt:
    ServiceName: 'krbtgt'
  condition: selection and not filter_computers and not filter_krbtgt
falsepositives:
  - Legacy-Systeme mit RC4-Anforderung
  - Service Accounts auf alten Systemen
level: high
```

```bash
# Sigma nach verschiedenen Zielsystemen konvertieren
# Installation
pip install sigma-cli
sigma plugin install splunk sentinel qradar

# Konvertierung
sigma convert -t splunk  -p sysmon rules/kerberoasting.yml
sigma convert -t sentinel -p windows rules/kerberoasting.yml
sigma convert -t qradar  rules/kerberoasting.yml

# Bulk-Konvertierung (ganzes Regelverzeichnis)
sigma convert -t sentinel -p windows rules/windows/ \
    -o sentinel-rules-windows.json

# Mit Pipelining und Feldmapping
sigma convert -t splunk -p splunk_windows \
    --field-mapping "EventID=EventCode" \
    rules/windows/credential_access/
```

### Sigma-Regeln für DACH-relevante Angriffe

```yaml
# Regel 1: BEC / CEO-Fraud Indikator
title: Externe E-Mail-Weiterleitung zu unbekannter Domain
id: a1b2c3d4-5678-90ab-cdef-1234567890ab
status: experimental
description: Erkennt neu angelegte E-Mail-Weiterleitungsregeln zu externen Domains
tags:
  - attack.collection
  - attack.t1114.003
logsource:
  product: m365
  service: exchange
detection:
  selection:
    Workload: Exchange
    Operation:
      - New-InboxRule
      - Set-InboxRule
    Parameters|contains:
      - ForwardTo
      - RedirectTo
      - ForwardAsAttachmentTo
  filter_internal:
    Parameters|contains:
      - "@firma.de"
      - "@firma.com"
  condition: selection and not filter_internal
level: high
falsepositives:
  - Legitime Weiterleitungen zu bekannten Partnern
```

```yaml
# Regel 2: DCSync Angriff (Credential Dump via Replikation)
title: DCSync Angriff
id: b2c3d4e5-6789-01bc-def2-34567890bcde
status: stable
tags:
  - attack.credential_access
  - attack.t1003.006
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4662
    AccessMask: '0x100'    # Control Access
    Properties|contains:
      - '1131f6aa-9c07-11d1-f79f-00c04fc2dcd2'   # DS-Replication-Get-Changes
      - '1131f6ad-9c07-11d1-f79f-00c04fc2dcd2'   # DS-Replication-Get-Changes-All
  filter_dcs:
    SubjectUserName|endswith: '$'    # Domain Controller Accounts ausfiltern
  condition: selection and not filter_dcs
level: critical
```

---

## Splunk — Erkennungs-SPL

```splunk
| SPL: Brute Force Detection
index=wineventlog EventCode=4625
| bin _time span=5m
| stats count as FailedLogins dc(src_ip) as UniqueIPs by user, _time
| where FailedLogins > 20
| eval Severity=if(FailedLogins > 50, "Critical", "High")
| table _time, user, FailedLogins, UniqueIPs, Severity
```

```splunk
| SPL: Ransomware-Indikator (Viele Datei-Umbenennungen)
index=sysmon EventCode=11
| rex field=TargetFilename "\.(?<extension>[^.]+)$"
| where NOT (extension IN ("docx","xlsx","pdf","pptx","jpg","png","mp4","zip"))
| bin _time span=1m
| stats count as FileRenames values(TargetFilename) as Files
    by Computer, User, _time
| where FileRenames > 50
| eval Severity="Critical"
| table _time, Computer, User, FileRenames, Files, Severity
```

```splunk
| SPL: Admin-Konto außerhalb Geschäftszeiten
index=wineventlog EventCode=4624 LogonType=2 OR LogonType=10
| eval Hour=strftime(_time, "%H"), Wochentag=strftime(_time, "%u")
| where (Hour < 7 OR Hour > 19) OR Wochentag IN (6,7)
| where match(Account, "(?i)(admin|svc|service)")
| table _time, Account, Workstation_Name, src_ip, Hour, Wochentag
| sort - _time
```

---

## Threat Hunting Playbooks

### Hunt 1: Living-off-the-Land (LOLBins)

```kql
// Verdächtige Verwendung von Windows-Bordmitteln für Malware-Ausführung
DeviceProcessEvents
| where FileName in~ (
    "certutil.exe", "bitsadmin.exe", "wscript.exe", "cscript.exe",
    "mshta.exe", "regsvr32.exe", "rundll32.exe", "msiexec.exe",
    "wmic.exe", "powershell.exe", "cmd.exe"
)
| where ProcessCommandLine has_any (
    "http://", "https://", "\\\\", "base64", "bypass",
    "downloadstring", "webclient", "iwr", "invoke-expression"
)
| where InitiatingProcessFileName !in~ ("explorer.exe", "svchost.exe")
| project Timestamp, DeviceName, InitiatingProcessFileName,
          FileName, ProcessCommandLine, AccountName
| order by Timestamp desc
```

### Hunt 2: Persistence via Registry Run Keys

```kql
DeviceRegistryEvents
| where ActionType in ("RegistryValueSet", "RegistryKeyCreated")
| where RegistryKey has_any (
    @"Software\Microsoft\Windows\CurrentVersion\Run",
    @"Software\Microsoft\Windows\CurrentVersion\RunOnce",
    @"SYSTEM\CurrentControlSet\Services"
)
| where InitiatingProcessFileName !in~ (
    "msiexec.exe", "setup.exe", "install.exe", "svchost.exe"
)
| project Timestamp, DeviceName, AccountName, RegistryKey,
          RegistryValueName, RegistryValueData, InitiatingProcessFileName
| order by Timestamp desc
```

---

## SIEM-Tuning und False-Positive-Reduktion

```
FRAMEWORK: Detection Maturity Level (DML)

DML-0: Keine Erkennung
DML-1: Hash/Signatur-basiert (AV)
DML-2: IP/Domain-Blocklisten
DML-3: Netzwerkbasiert (Anomalien)
DML-4: Host-basiert (Prozesse, Registry)
DML-5: Verhaltensbasiert (UEBA)
DML-6: TTP-basiert (MITRE ATT&CK)

TUNING-PROZESS:
1. Baseline aufnehmen (30 Tage ohne Alerts einschalten)
2. Regel aktivieren, False-Positive-Rate messen
3. Whitelist-Einträge dokumentieren (Begründung!)
4. Threshold anpassen (nicht zu niedrig, nicht zu hoch)
5. Regel in Produktion bringen wenn FP-Rate < 5%
6. Monatliches Review: Threshold noch korrekt?

WHITELIST-MANAGEMENT (KQL-Beispiel):
let known_admin_ips = dynamic(["10.10.0.10", "10.10.0.11"]);
let known_service_accounts = dynamic(["svc-backup", "svc-monitoring"]);
SecurityEvent
| where EventID == 4625
| where not (IpAddress in (known_admin_ips))
| where not (Account in (known_service_accounts))
```

---

## Detection-as-Code Pipeline

```yaml
# .github/workflows/sigma-pipeline.yml
name: Sigma Detection Pipeline
on:
  push:
    paths: ['detections/**/*.yml']
  pull_request:
    paths: ['detections/**/*.yml']

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install sigma-cli
      - name: Sigma Syntax Validation
        run: sigma check detections/
      - name: Convert to Sentinel
        run: |
          sigma convert -t sentinel -p windows \
              detections/windows/ \
              -o output/sentinel-analytics-rules.json
      - name: Convert to Splunk
        run: |
          sigma convert -t splunk -p splunk_windows \
              detections/windows/ \
              -o output/splunk-searches.conf

  deploy-sentinel:
    needs: validate
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - name: Deploy Analytics Rules to Sentinel
        run: |
          az sentinel alert-rule create \
              --resource-group rg-sentinel \
              --workspace-name law-sentinel-prod \
              --rule-id $(uuidgen) \
              --scheduled-query-rule @output/sentinel-analytics-rules.json
```
