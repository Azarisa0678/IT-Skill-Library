---
name: m365-admin-skill
description: >
  Microsoft 365 Admin Skill fuer IT-Administratoren. Entra ID, Conditional Access, MFA, PIM,
  Identity Protection, SSPR, LAPS, CyberArk, BeyondTrust, PAM, JIT VM Access,
  Intune, MDM, Autopilot, Compliance Policy, App Deployment, Remediations,
  Exchange Online, Hybrid Exchange, Anti-Spam, DKIM, DMARC, SPF, Mail-Flow, NDR,
  SharePoint, SharePoint Online, OneDrive, PnP, Teams, Guest Access,
  Defender, MDE, MDO, MDI, Defender for Endpoint, Defender for Office, Sentinel, Purview,
  Sensitivity Labels, DLP, eDiscovery, Compliance, Azure VM, Azure IaaS, Azure VNet,
  Azure Backup, Azure Site Recovery, Windows Server, Active Directory, GPO, AD Connect,
  Lizenz, M365 Lizenz, E3, E5, Business Premium, Kostenoptimierung,
  Microsoft Graph, Graph API, PowerShell, MSOnline, Connect-MgGraph.
  Outputs: PowerShell-Skripte (DE kommentiert), Step-by-Step-Anleitungen (DE), Checklisten.
---

# Microsoft 365 Admin Skill

## Guiding Principles

**Modern Auth first.** MSOnline und MSOL-Cmdlets sind deprecated — immer Microsoft Graph PowerShell nutzen.
**Least Privilege.** Kein Global Admin für Routineaufgaben — Exchange Admin, User Admin, etc. verwenden.
**Conditional Access vor Passwort-Komplexität.** MFA + CA schützt besser als Passwortregeln allein.
**Testen mit Report-Only.** Conditional Access Policies immer erst im Report-Only-Modus testen.
**Lizenz-Hygiene monatlich.** Inaktive Benutzer → Lizenz entziehen, spart erhebliche Kosten.
**Graph API ist die Zukunft.** Alle PowerShell-Skripte auf Microsoft.Graph-Modul aufbauen.

---

## Referenzmodule

- `references/entra-id.md` — Conditional Access, MFA, PIM, Identity Protection, SSPR, App Registrierungen
- `references/intune-endpoint.md` — Enrollment, Autopilot, Compliance Policies, App Deployment, Remediations, Reporting
- `references/hybrid-exchange.md` — Exchange Online Admin, Hybrid Migration, Anti-Spam/Phishing, DKIM/DMARC, Mail-Flow
- `references/sharepoint-onedrive.md` — SPO Administration, PnP PowerShell, Governance, Berechtigungen, OneDrive
- `references/defender-suite.md` — MDE, MDO, MDI, Defender for Cloud Apps, Incidents, Hunting, KQL
- `references/m365-powershell.md` — Graph PowerShell, Bulk-Operationen, Reporting, Automatisierung
- `references/windows-server-ad.md` — Active Directory, GPO, ADCS, AD Connect, Hybrid Identity
- `references/azure-iaas.md` — Azure VMs, VNets, Storage, Azure Backup, ASR, Cost Management
- `references/lizenz-kosten.md` — SKU-Übersicht, Lizenz-Zuweisung, Audit, Inaktive User, Optimierungs-Checkliste
- `references/troubleshooting.md` — Häufige Probleme, Diagnose-Befehle, Known Issues, Support-Eskalation
- `references/architekturen-templates.md` — Zero Trust, Security Baselines, Referenzarchitekturen, Templates
- `references/prompt-bibliothek.md` — Prompt-Templates für alle M365-Themenbereiche

Routing:
- Entra ID / Conditional Access / MFA / PIM → `entra-id.md`
- Intune / Autopilot / Compliance / App-Deployment → `intune-endpoint.md`
- Exchange / Hybrid / Anti-Spam / DKIM / Mail-Flow → `hybrid-exchange.md`
- SharePoint / OneDrive / PnP / Teams → `sharepoint-onedrive.md`
- Defender / MDE / MDO / Sentinel / KQL → `defender-suite.md`
- Graph PowerShell / Bulk-Ops / Reporting → `m365-powershell.md`
- Active Directory / GPO / AD Connect → `windows-server-ad.md`
- Azure VMs / VNets / IaaS / Cost → `azure-iaas.md`
- Lizenz / SKU / Kosten / inaktive User → `lizenz-kosten.md`
- Fehlermeldung / Diagnose / Troubleshooting → `troubleshooting.md`
- Zero Trust / Architektur / Templates → `architekturen-templates.md`
- Prompt-Vorlage für M365-Themen → `prompt-bibliothek.md`

---

## PowerShell-Verbindungen (Schnellreferenz)

```powershell
# Microsoft Graph (moderne Standardverbindung)
Install-Module Microsoft.Graph -Force
Connect-MgGraph -Scopes "User.ReadWrite.All","Directory.ReadWrite.All",
    "DeviceManagementConfiguration.ReadWrite.All","Reports.Read.All"

# Exchange Online
Install-Module ExchangeOnlineManagement -Force
Connect-ExchangeOnline -UserPrincipalName admin@firma.de -ShowBanner:$false

# SharePoint Online + PnP
Install-Module Microsoft.Online.SharePoint.PowerShell, PnP.PowerShell -Force
Connect-SPOService -Url "https://firma-admin.sharepoint.com"
Connect-PnPOnline -Url "https://firma.sharepoint.com/sites/mysite" -Interactive

# Alle trennen
Disconnect-MgGraph
Disconnect-ExchangeOnline -Confirm:$false
Disconnect-SPOService
Disconnect-PnPOnline
```

---

## Entra ID — Schnellreferenz

```powershell
# Alle Benutzer (Graph)
Get-MgUser -All -Property DisplayName,UserPrincipalName,AccountEnabled,
    LastPasswordChangeDateTime,AssignedLicenses |
    Where-Object {$_.AccountEnabled} | Format-Table

# MFA-Status Bericht
$users = Get-MgUser -All -Property DisplayName,UserPrincipalName
$users | ForEach-Object {
    $methods = Get-MgUserAuthenticationMethod -UserId $_.Id
    [PSCustomObject]@{
        Name  = $_.DisplayName
        UPN   = $_.UserPrincipalName
        MFA   = ($methods.Count -gt 1)
        Types = ($methods.AdditionalProperties.Values | Select-Object -Unique) -join ", "
    }
} | Where-Object {-not $_.MFA} | Format-Table    # Nur ohne MFA

# Conditional Access Policies
Get-MgIdentityConditionalAccessPolicy |
    Select-Object DisplayName, State, @{N='Users'; E={$_.Conditions.Users.IncludeUsers}} |
    Format-Table -AutoSize
```

→ CA-Policy erstellen, PIM, Identity Protection, SSPR, App Registrierungen: `references/entra-id.md`

---

## Exchange Online — Schnellreferenz

```powershell
# Postfach-Übersicht
Get-Mailbox -ResultSize Unlimited |
    Get-MailboxStatistics |
    Select-Object DisplayName,TotalItemSize,ItemCount |
    Sort-Object TotalItemSize -Descending | Format-Table

# Mail-Flow-Trace
Get-MessageTrace -StartDate (Get-Date).AddDays(-1) -EndDate (Get-Date) |
    Where-Object {$_.Status -eq "Failed"} |
    Select-Object Received,SenderAddress,RecipientAddress,Subject,Status

# DKIM Status
Get-DkimSigningConfig -Identity firma.de |
    Select-Object Domain,Enabled,Status,Selector1CNAME
```

→ Hybrid Migration, Anti-Spam/Phishing, Transport Rules, Quarantäne: `references/hybrid-exchange.md`

---

## Intune — Schnellreferenz

```powershell
# Nicht-konforme Geräte
Get-MgDeviceManagementManagedDevice -All |
    Where-Object {$_.ComplianceState -eq "noncompliant"} |
    Select-Object DeviceName,UserPrincipalName,LastSyncDateTime,
        ComplianceState,OperatingSystem | Format-Table

# Stale Devices (> 30 Tage kein Sync)
Get-MgDeviceManagementManagedDevice -All |
    Where-Object {$_.LastSyncDateTime -lt (Get-Date).AddDays(-30)} |
    Select-Object DeviceName,UserPrincipalName,LastSyncDateTime
```

→ Autopilot, Compliance Policies JSON, App Deployment, Win32 Apps, Remediations: `references/intune-endpoint.md`

---

## SharePoint — Schnellreferenz

```powershell
# Sites nach Größe
Get-SPOSite -Limit All |
    Select-Object Url,Title,StorageUsageCurrent,SharingCapability |
    Sort-Object StorageUsageCurrent -Descending | Format-Table

# Externe Freigabe einschränken
Set-SPOTenant -SharingCapability ExistingExternalUserSharingOnly
Set-SPOSite -Identity "https://firma.sharepoint.com/sites/hr" -SharingCapability Disabled

# Ablaufdatum für Links
Set-SPOTenant -RequireAnonymousLinksExpireInDays 7
```

→ PnP PowerShell, Governance-Bericht, OneDrive Quota, Berechtigungen: `references/sharepoint-onedrive.md`

---

## Lizenz — Schnellreferenz

```powershell
# Verfügbare Lizenzen
Get-MgSubscribedSku | Select-Object SkuPartNumber,
    @{N='Verfügbar'; E={$_.PrepaidUnits.Enabled - $_.ConsumedUnits}},
    @{N='Genutzt';   E={$_.ConsumedUnits}} | Format-Table

# Inaktive Benutzer mit Lizenz (> 90 Tage kein Login)
Get-MgUser -All -Property DisplayName,UserPrincipalName,SignInActivity,AssignedLicenses |
    Where-Object {
        $_.AssignedLicenses.Count -gt 0 -and
        $_.SignInActivity.LastSignInDateTime -lt (Get-Date).AddDays(-90)
    } | Select-Object DisplayName,UserPrincipalName,
        @{N='LetzterLogin'; E={$_.SignInActivity.LastSignInDateTime}} |
    Export-Csv "inaktive-lizenzen.csv" -Encoding UTF8 -NoTypeInformation
```

→ Lizenz-Zuweisung (Einzel + Bulk), Overlap-Analyse, Optimierungs-Checkliste: `references/lizenz-kosten.md`

---

## Defender — Schnellreferenz

```kql
// MDE: Geräte mit fehlenden kritischen Patches
DeviceTvmSoftwareVulnerabilities
| where VulnerabilitySeverityLevel == "Critical"
| where RecommendedSecurityUpdate == ""
| summarize Lücken=count() by DeviceName
| order by Lücken desc

// MDO: Phishing-Mails der letzten 24h
EmailEvents
| where Timestamp > ago(24h)
| where ThreatTypes has "Phish"
| summarize count() by SenderFromAddress, RecipientEmailAddress
| order by count_ desc
```

→ MDE Onboarding, MDO Policies, Incidents, Advanced Hunting, KQL-Queries: `references/defender-suite.md`

---

## Output-Formate je Aufgabentyp

| Aufgabe | Format |
|---|---|
| PowerShell-Skript | Kommentierte Blöcke, Connect/Disconnect, Error-Handling, Export-CSV |
| Conditional Access Policy | Schritt-für-Schritt GUI + PowerShell/Graph-API-Äquivalent |
| Compliance Policy (Intune) | JSON-Definition + Deployment-Befehl + Erklärung je Parameter |
| Migration (Exchange/SharePoint) | Phasenplan: Pre-Migration → Migration → Validation → Cutover |
| Troubleshooting | Symptom → Diagnose-Befehle → Ursachen → Lösung |
| Checkliste | Markdown-Checkboxen, gruppiert, auf Deutsch |
| Architektur | ASCII-Diagramm + Komponentenbeschreibung + Sicherheitshinweise |

---

## Identity & PAM — Schnellreferenz

```powershell
# Wer hat Global Admin dauerhaft (sollte fast leer sein)?
$role = Get-MgDirectoryRole | Where-Object {$_.DisplayName -eq "Global Administrator"}
Get-MgDirectoryRoleMember -DirectoryRoleId $role.Id |
    ForEach-Object { Get-MgUser -UserId $_.Id | Select-Object DisplayName, UserPrincipalName }

# PIM-Aktivierungen der letzten 7 Tage
Get-MgAuditLogDirectoryAudit -Filter "activityDisplayName eq 'Add member to role completed (PIM activation)' and activityDateTime ge $((Get-Date).AddDays(-7).ToString('yyyy-MM-dd'))" |
    Select-Object ActivityDateTime,
        @{N='User';  E={$_.InitiatedBy.User.UserPrincipalName}},
        @{N='Rolle'; E={($_.TargetResources | Where-Object {$_.Type -eq "Role"}).DisplayName}}

# LAPS-Passwort abrufen
Connect-MgGraph -Scopes "DeviceLocalCredential.Read.All"
$device = Get-MgDevice -Filter "displayName eq 'LAPTOP-001'"
$cred = Get-MgDeviceLocalCredential -DeviceId $device.Id -Property "credentials"
[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($cred.Credentials[0].PasswordBase64))

# JIT VM Access anfordern (Azure)
Start-AzJitNetworkAccessPolicy -ResourceGroupName "rg-prod" `
    -Location "westeurope" -Name "JIT-vm-db01" `
    -VirtualMachine @(@{id=$vmId; ports=@(@{number=3389; allowedSourceAddressPrefix=$myIp; endTimeUtc=(Get-Date).AddHours(1).ToUniversalTime().ToString("o")})})
```

→ Vollständige PIM-Konfiguration, CyberArk REST API, BeyondTrust Password Safe, LAPS on-prem, JIT-Patterns, PAM-Compliance-Checkliste: `references/identity-pam.md`
