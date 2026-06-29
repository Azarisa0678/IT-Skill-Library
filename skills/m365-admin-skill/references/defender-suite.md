# Microsoft Defender Suite

## Überblick der Defender-Produkte

| Produkt | Schützt | Lizenz |
|---------|---------|--------|
| **Defender for Endpoint (MDE)** | Windows/Mac/Linux Endpoints | M365 E5, MDE P1/P2 |
| **Defender for Identity (MDI)** | Active Directory, On-Prem | M365 E5, MDI |
| **Defender for Cloud Apps (MDCA)** | SaaS-Apps, Shadow IT | M365 E5, MDCA |
| **Defender for Office 365 (MDO)** | E-Mail, Teams, SharePoint | M365 BP, E3+P1, E5 |
| **Defender Vulnerability Management** | Schwachstellenmanagement | MDE P2 |
| **Microsoft Sentinel** | SIEM/SOAR | Eigene Lizenz (pay-per-GB) |

**Admin-Portale:**
- **Microsoft Defender Portal:** security.microsoft.com (zentral für alle Defender-Produkte)
- **Microsoft Sentinel:** portal.azure.com → Sentinel

---

## Defender for Endpoint (MDE)

```powershell
# MDE in Intune integrieren (empfohlener Onboarding-Weg)
# Portal: Intune → Endpoint Security → Microsoft Defender for Endpoint → Verbinden

# Onboarding-Status prüfen
Connect-MgGraph -Scopes "DeviceManagementConfiguration.Read.All"
Get-MgDeviceManagementManagedDevice -All |
    Select-Object DeviceName, OperatingSystem,
        @{N='MDEOnboarded';E={$_.ManagedDeviceName -ne $null}} |
    Format-Table -AutoSize

# PowerShell: MDE API nutzen (Advanced Hunting)
$tenantId    = "your-tenant-id"
$clientId    = "your-app-id"
$clientSecret = "your-secret"

$body = @{
    client_id     = $clientId
    client_secret = $clientSecret
    grant_type    = "client_credentials"
    scope         = "https://api.securitycenter.microsoft.com/.default"
}
$token = (Invoke-RestMethod -Method Post `
    -Uri "https://login.microsoftonline.com/$tenantId/oauth2/v2.0/token" `
    -Body $body).access_token

# Advanced Hunting Query ausführen
$query = @"
DeviceEvents
| where Timestamp > ago(24h)
| where ActionType == "PowerShellCommand"
| where InitiatingProcessCommandLine contains "-enc"
| project Timestamp, DeviceName, InitiatingProcessCommandLine
| limit 100
"@

$result = Invoke-RestMethod `
    -Uri "https://api.securitycenter.microsoft.com/api/advancedhunting/run" `
    -Method Post `
    -Headers @{ Authorization = "Bearer $token"; "Content-Type" = "application/json" } `
    -Body (@{ Query = $query } | ConvertTo-Json)

$result.Results | Format-Table -AutoSize

# Offene Warnungen abrufen
$alerts = Invoke-RestMethod `
    -Uri "https://api.securitycenter.microsoft.com/api/alerts?`$filter=status ne 'Resolved'&`$top=50" `
    -Headers @{ Authorization = "Bearer $token" }
$alerts.value | Select-Object Title, Severity, Status, MachineId |
    Sort-Object Severity | Format-Table -AutoSize
```

### MDE Härtungseinstellungen

```powershell
# Attack Surface Reduction (ASR) Rules per Intune
# Portal: Intune → Endpoint Security → Attack Surface Reduction

# Empfohlene ASR-Rules (Block-Modus nach Test-Phase):
$asrRules = @{
    # Office-Makros
    "BE9BA2D9-53EA-4CDC-84E5-9B1EEEE46550" = "Block"  # Executable content from email
    "D4F940AB-401B-4EFC-AADC-AD5F3C50688A" = "Block"  # Office child processes
    "3B576869-A4EC-4529-8536-B80A7769E899" = "Block"  # Office executable content
    "92E97FA1-2EDF-4476-BDD6-9DD0B4DDDC7B" = "Block"  # Win32 API from Office macro
    # Credential Protection
    "9E6C4E1F-7D60-472F-BA1A-A39EF669E4B2" = "Block"  # Credential stealing from LSASS
    # Ransomware
    "C1DB55AB-C21A-4637-BB3F-A12568109D35" = "Block"  # Ransomware protection
    # Script-basierte Angriffe
    "5BEB7EFE-FD9A-4556-801D-275E5FFC04CC" = "Block"  # Obfuscated scripts
    "75668C1F-73B5-4CF0-BB93-3ECF5CB7CC84" = "Block"  # Office process injection
}

# Zuerst im Audit-Modus testen, dann auf Block wechseln!
```

---

## Defender for Office 365 (MDO)

```powershell
Connect-ExchangeOnline -UserPrincipalName "admin@firma.de" -ShowBanner:$false

# Safe Links Policy (URL-Scanning in E-Mails und Teams)
New-SafeLinksPolicy `
    -Name "Firma - Safe Links" `
    -EnableSafeLinksForEmail $true `
    -EnableSafeLinksForTeams $true `
    -EnableSafeLinksForOffice $true `
    -ScanUrls $true `
    -EnableForInternalSenders $true `
    -DeliverMessageAfterScan $true `
    -DisableUrlRewrite $false `
    -TrackClicks $true

New-SafeLinksRule `
    -Name "Firma - Safe Links Rule" `
    -SafeLinksPolicy "Firma - Safe Links" `
    -RecipientDomainIs "firma.de" `
    -Priority 0

# Safe Attachments Policy (Sandbox für Anhänge)
New-SafeAttachmentPolicy `
    -Name "Firma - Safe Attachments" `
    -Action Block `
    -Enable $true `
    -Redirect $false

New-SafeAttachmentRule `
    -Name "Firma - Safe Attachments Rule" `
    -SafeAttachmentPolicy "Firma - Safe Attachments" `
    -RecipientDomainIs "firma.de" `
    -Priority 0

# Anti-Phishing Policy (Impersonation Protection)
New-AntiPhishPolicy `
    -Name "Firma - Anti-Phishing" `
    -Enabled $true `
    -EnableSpoofIntelligence $true `
    -EnableUnauthenticatedSender $true `
    -EnableViaTag $true `
    -PhishThresholdLevel 2 `
    -EnableMailboxIntelligence $true `
    -EnableMailboxIntelligenceProtection $true `
    -MailboxIntelligenceProtectionAction MoveToJmf

# Aktuelle Bedrohungsberichte
Get-MailDetailATPReport -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) |
    Group-Object EventType |
    Select-Object Name, Count |
    Sort-Object Count -Descending | Format-Table -AutoSize
```

---

## Defender for Identity (MDI)

```powershell
# MDI überwacht Active Directory auf Angriffe wie:
# - Pass-the-Hash, Pass-the-Ticket
# - Golden Ticket, Silver Ticket
# - Kerberoasting, AS-REP Roasting
# - Lateral Movement
# - Reconnaissance

# MDI Sensor auf DC installieren
# 1. Download: security.microsoft.com → Einstellungen → Identitäten → Sensoren
# 2. Installation auf allen DCs (Domänencontroller)

# MDI API - Warnungen abrufen
$mdaToken = # Token wie bei MDE (andere Scope)
$mdaAlerts = Invoke-RestMethod `
    -Uri "https://api.aad.securitycenter.microsoft.com/api/alerts" `
    -Headers @{ Authorization = "Bearer $mdaToken" }

# Sensitive Gruppen in MDI konfigurieren (zusätzlich zu Standardgruppen)
# Portal: security.microsoft.com → Einstellungen → Identitäten → Entity Tags
# "Sensitive" markieren für: Domain Admins, Enterprise Admins, Schema Admins

# Honeytoken Account erstellen (lockt Angreifer)
New-ADUser -Name "svc-backup-admin" `
    -SamAccountName "svc-backup-admin" `
    -Description "DO NOT USE - Monitoring Account" `
    -Enabled $true `
    -AccountPassword (ConvertTo-SecureString (New-Guid).Guid -AsPlainText -Force)
# In MDI als Honeytoken markieren → Alert bei jedem Login
```

---

## Defender for Cloud Apps (MDCA)

```powershell
# MDCA = Cloud App Security / Shadow IT Discovery
# Admin-Portal: security.microsoft.com → Cloud Apps

# Shadow IT Discovery via Defender for Endpoint (automatisch wenn MDE aktiv)
# Alternativ: Firewall/Proxy-Logs hochladen

# App-Governance via PowerShell (Graph)
Connect-MgGraph -Scopes "Application.Read.All"

# Alle OAuth-Apps im Tenant auflisten
Get-MgServicePrincipal -All |
    Where-Object { $_.AppRoles.Count -gt 0 -or $_.Oauth2PermissionScopes.Count -gt 0 } |
    Select-Object DisplayName, AppId, CreatedDateTime |
    Export-Csv "C:\Reports\OAuthApps.csv" -NoTypeInformation -Encoding UTF8

# Apps mit hohen Berechtigungen identifizieren
Get-MgServicePrincipal -All -Property DisplayName,AppId,AppRoles |
    Where-Object {
        $_.AppRoles | Where-Object { $_.Value -match "Mail.ReadWrite|Files.ReadWrite.All|Directory.ReadWrite" }
    } |
    Select-Object DisplayName, AppId | Format-Table -AutoSize

# MCAS Session Policies können:
# - Downloads aus Cloud-Apps auf nicht-verwalteten Geräten blockieren
# - Malware-Scan vor Upload erzwingen
# - Copy/Paste aus sensitiven Apps blockieren
```

---

## Microsoft Sentinel

```powershell
# Sentinel Workspace erstellen
New-AzOperationalInsightsWorkspace `
    -ResourceGroupName "rg-sentinel" `
    -Name "law-sentinel-firma" `
    -Location "westeurope" `
    -Sku "PerGB2018"

# Sentinel auf Workspace aktivieren (Az CLI)
az security insights create `
    --resource-group "rg-sentinel" `
    --workspace-name "law-sentinel-firma"

# Datenquellen verbinden (Connectors)
# Portal: Sentinel → Datenkonnektoren
# Wichtigste Connectors:
# - Microsoft 365 Defender (E-Mail, Endpoint, Identity, Cloud Apps)
# - Azure Activity
# - Entra ID (Sign-in Logs, Audit Logs)
# - Microsoft Defender for Cloud
# - Windows Security Events via AMA

# Analytikregeln via PowerShell (ARM-Template oder Bicep)
# Vordefinierte Templates: Sentinel → Analytik → Regelvorlagen

# KQL-Abfrage für verdächtige Anmeldungen
$kqlQuery = @"
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType != 0
| where RiskLevelDuringSignIn in ('high', 'medium')
| project TimeGenerated, UserPrincipalName, AppDisplayName,
          IPAddress, Location, RiskLevelDuringSignIn, ResultDescription
| order by TimeGenerated desc
"@

# Abfrage über Log Analytics API ausführen
Invoke-AzOperationalInsightsQuery `
    -WorkspaceId "your-workspace-id" `
    -Query $kqlQuery |
    Select-Object -ExpandProperty Results |
    Format-Table -AutoSize
```

---

## Defender Suite Checkliste

- [ ] MDE: alle Endpoints ongeboardet (Windows, Mac, Linux)
- [ ] MDE: ASR Rules im Audit-Modus, nach Test auf Block
- [ ] MDO: Safe Links Policy aktiv für E-Mail, Teams, Office
- [ ] MDO: Safe Attachments Policy aktiv
- [ ] MDO: Anti-Phishing Policy mit Impersonation Protection
- [ ] MDI: Sensor auf allen Domain Controllern installiert
- [ ] MDI: Sensitive Groups konfiguriert
- [ ] MDI: Honeytoken Accounts erstellt
- [ ] MDCA: Shadow IT Discovery aktiv (via MDE oder Log-Upload)
- [ ] MDCA: OAuth-App-Governance — riskante Apps blockiert
- [ ] Sentinel: Workspace erstellt, M365 Defender Connector aktiv
- [ ] Sentinel: Analytikregeln für kritische Szenarien (Impossible Travel, MFA Fatigue)
- [ ] Wöchentlicher Threat-Report aus security.microsoft.com
