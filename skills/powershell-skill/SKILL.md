---
name: powershell-skill
description: >
  PowerShell Skill fuer IT-Administratoren und Security-Analysten.
  Sichere Scripting-Praktiken, SecureString, kein PlainText, Input-Validierung,
  Script Block Logging, Module Logging, Transcription Logging, GPO,
  Code-Signing, Authenticode, ADCS, Execution Policy AllSigned,
  JEA, Just Enough Administration, RoleCapabilities, SessionConfiguration,
  Constrained Language Mode, CLM, AppLocker,
  Active Directory, Bulk-Import, Offboarding, Stale Accounts, AD-Report,
  Entra ID, Microsoft Graph, Connect-MgGraph, MFA-Audit, CA-Policies, Sign-In-Logs,
  Automatisierung, Retry, Fehlerbehandlung, REST-API, ServiceNow, PagerDuty,
  Enterprise-Logging, HTML-Report, Send-MailMessage, Remoting, WinRM, Invoke-Command,
  Exchange Online, SharePoint, Intune, Azure, Az-Modul.
  Outputs: PowerShell-Skripte (.ps1), Module (.psm1), JEA-Dateien, HTML-Reports.
---

# PowerShell Skill

## Leitprinzipien

**Kein PlainText-Passwort.** SecureString, Azure Key Vault oder Get-Credential — niemals -AsPlainText in Produktion.
**set-StrictMode -Version Latest.** Fängt undefinierte Variablen und häufige Fehler frühzeitig ab.
**Fehlerbehandlung ist Pflicht.** Jedes Produktionsskript hat Try/Catch mit detailliertem Logging.
**Idempotenz.** Skripte müssen mehrfach ausführbar sein ohne ungewollte Nebeneffekte.
**Logging in Datei + Eventlog.** Jede Ausführung protokolliert, damit Audits möglich sind.
**Code-Signing in Produktion.** AllSigned Execution Policy + ADCS-Zertifikate für alle Skripte.

---

## Referenzmodule

- `references/ps-security.md` — Secure Scripting, Logging (GPO), Code-Signing, JEA, CLM, AppLocker
- `references/ps-ad-entra.md` — AD Bulk-Import, Offboarding, Stale Accounts, Graph MFA-Audit, CA-Policies
- `references/ps-automation.md` — Logging-Framework, Fehlerbehandlung, Retry, REST-API, HTML-Report, Remoting

Routing:
- Sichere Skript-Praktiken / Code-Signing / JEA / CLM → `ps-security.md`
- Active Directory / Entra ID / Microsoft Graph → `ps-ad-entra.md`
- Automatisierung / Logging / REST-API / Remoting / Reporting → `ps-automation.md`

---

## Skript-Template (Produktionsreif)

```powershell
#Requires -Version 5.1
#Requires -Modules ActiveDirectory
<#
.SYNOPSIS
    [Kurzbeschreibung was das Skript tut]
.DESCRIPTION
    [Ausführliche Beschreibung]
.PARAMETER Environment
    Zielumgebung: dev, staging, prod
.EXAMPLE
    .\deploy.ps1 -Environment prod -Verbose
#>
[CmdletBinding(SupportsShouldProcess)]
param(
    [Parameter(Mandatory)]
    [ValidateSet('dev','staging','prod')]
    [string]$Environment,

    [switch]$DryRun
)

Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'

# Konstanten
$script:LogFile  = "C:\Logs\$(Split-Path $PSCommandPath -Leaf)-$(Get-Date -Format 'yyyyMMdd').log"
$script:LockFile = "C:\Temp\$(Split-Path $PSCommandPath -Leaf).lock"

# Logging
function Write-Log {
    param([string]$Message, [ValidateSet('INFO','WARN','ERROR')][string]$Level = 'INFO')
    $entry = "[$(Get-Date -Format 'HH:mm:ss')] [$Level] $Message"
    $entry | Out-File $script:LogFile -Append -Encoding UTF8
    switch ($Level) {
        'INFO'  { Write-Verbose $entry }
        'WARN'  { Write-Warning $entry }
        'ERROR' { Write-Error   $entry }
    }
}

# Cleanup + Lock
function Initialize-Script {
    if (Test-Path $script:LockFile) {
        $pid = Get-Content $script:LockFile
        if (Get-Process -Id $pid -ErrorAction SilentlyContinue) {
            throw "Skript läuft bereits (PID: $pid)"
        }
    }
    $PID | Out-File $script:LockFile
    Register-EngineEvent -SourceIdentifier PowerShell.Exiting -Action {
        Remove-Item $script:LockFile -Force -ErrorAction SilentlyContinue
    } | Out-Null
}

# Hauptprogramm
try {
    Initialize-Script
    Write-Log "Gestartet — Umgebung: $Environment, DryRun: $DryRun"

    # ... Skript-Logik hier ...

    Write-Log "Erfolgreich abgeschlossen"
}
catch {
    Write-Log "KRITISCHER FEHLER: $_" -Level ERROR
    Write-Log "Stack: $($_.ScriptStackTrace)" -Level ERROR
    exit 1
}
finally {
    Remove-Item $script:LockFile -Force -ErrorAction SilentlyContinue
}
```

---

## Active Directory — Schnellreferenz

```powershell
# Benutzer anlegen
New-ADUser -Name "Max Mustermann" -SamAccountName "m.mustermann" `
    -UserPrincipalName "m.mustermann@firma.de" `
    -Path "OU=IT,OU=Benutzer,DC=firma,DC=local" `
    -AccountPassword (ConvertTo-SecureString "Temp!2024" -AsPlainText -Force) `
    -ChangePasswordAtLogon $true -Enabled $true

# Stale Accounts (> 90 Tage kein Login)
Get-ADUser -Filter {LastLogonDate -lt (Get-Date).AddDays(-90) -and Enabled -eq $true} `
    -Properties LastLogonDate, Department |
    Export-Csv "stale-accounts.csv" -Encoding UTF8 -NoTypeInformation

# Offboarding (Deaktivieren + Gruppe entfernen + OU verschieben)
Disable-ADAccount -Identity "m.mustermann"
(Get-ADUser "m.mustermann" -Properties MemberOf).MemberOf |
    ForEach-Object { Remove-ADGroupMember -Identity $_ -Members "m.mustermann" -Confirm:$false }
Move-ADObject -Identity (Get-ADUser "m.mustermann").DistinguishedName `
    -TargetPath "OU=Deaktiviert,DC=firma,DC=local"
```

→ Bulk-Import aus CSV, Passwort-Ablauf-Report, AD-Compliance-HTML-Report: `references/ps-ad-entra.md`

---

## Microsoft Graph — Schnellreferenz

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All","AuditLog.Read.All"

# Benutzer ohne MFA
Get-MgUser -All -Property Id,DisplayName,UserPrincipalName | ForEach-Object {
    if ((Get-MgUserAuthenticationMethod -UserId $_.Id).Count -le 1) {
        [PSCustomObject]@{ Name=$_.DisplayName; UPN=$_.UserPrincipalName; MFA="FEHLT" }
    }
} | Export-Csv "no-mfa.csv" -Encoding UTF8 -NoTypeInformation

# Fehlgeschlagene Logins letzte 24h
Get-MgAuditLogSignIn -Filter "createdDateTime ge $((Get-Date).AddDays(-1).ToString('yyyy-MM-ddTHH:mm:ssZ')) and status/errorCode ne 0" |
    Group-Object {$_.Status.FailureReason} | Sort-Object Count -Descending | Select-Object Count, Name

Disconnect-MgGraph
```

→ Gäste-Analyse, CA-Policy-Übersicht, Lizenz-Audit: `references/ps-ad-entra.md`

---

## Sicherheit — Schnellreferenz

```powershell
# Script Block Logging aktivieren (Registry)
$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
New-Item $path -Force | Out-Null
Set-ItemProperty $path -Name EnableScriptBlockLogging -Value 1

# Execution Policy
Set-ExecutionPolicy AllSigned -Scope LocalMachine -Force

# Skript signieren (mit ADCS-Zertifikat)
$cert = Get-ChildItem Cert:\CurrentUser\My -CodeSigningCert |
    Where-Object {$_.Issuer -like "*Firma-CA*"} | Select-Object -First 1
Set-AuthenticodeSignature -FilePath ".\deploy.ps1" -Certificate $cert `
    -TimestampServer "http://timestamp.digicert.com"

# JEA-Endpunkt (Helpdesk: nur Passwort-Reset)
Register-PSSessionConfiguration -Name "JEA-HelpDesk" `
    -Path "C:\PSSessionConfigurations\JEA-HelpDesk.pssc" -Force
```

→ JEA RoleCapabilities, CLM, AppLocker, Logging-Auswertung, Bypass-Erkennung: `references/ps-security.md`

---

## Automatisierung — Schnellreferenz

```powershell
# Retry-Pattern (3 Versuche, exponentielles Backoff)
function Invoke-WithRetry {
    param([scriptblock]$Action, [int]$MaxAttempts=3, [int]$DelaySeconds=5)
    for ($i=1; $i -le $MaxAttempts; $i++) {
        try { return (& $Action) }
        catch {
            if ($i -eq $MaxAttempts) { throw }
            Start-Sleep ($DelaySeconds * [Math]::Pow(2, $i-1))
        }
    }
}

# REST-API Aufruf
$headers = @{ Authorization = "Bearer $token"; "Content-Type" = "application/json" }
$result  = Invoke-RestMethod -Uri "https://api.firma.de/v1/users" `
    -Method GET -Headers $headers -TimeoutSec 30

# HTML-Report per E-Mail
$table = $data | ConvertTo-Html -Fragment
Send-MailMessage -SmtpServer "mail.firma.de" -Port 587 -UseSsl `
    -From "automation@firma.de" -To "it-ops@firma.de" `
    -Subject "Report $(Get-Date -Format 'dd.MM.yyyy')" `
    -Body "<html><body>$table</body></html>" -BodyAsHtml -Encoding UTF8

# Parallel auf mehreren Servern
Invoke-Command -ComputerName $servers -ThrottleLimit 10 -ScriptBlock {
    [PSCustomObject]@{
        Server = $env:COMPUTERNAME
        CPU    = (Get-CimInstance Win32_Processor).LoadPercentage
        FreeGB = [math]::Round((Get-CimInstance Win32_OperatingSystem).FreePhysicalMemory/1MB,2)
    }
} | Format-Table -AutoSize
```

→ Enterprise-Logging-Framework, ServiceNow/PagerDuty-Integration, WinRM-Konfiguration: `references/ps-automation.md`

---

## Output-Formate je Aufgabentyp

| Aufgabe | Format |
|---|---|
| Produktionsskript | Vollständig mit #Requires, CmdletBinding, param, StrictMode, Logging, Try/Catch |
| AD-Bulk-Operation | Skript mit CSV-Input, Fehlerprotokoll, Erfolgszähler |
| Graph/M365-Report | Export-Csv + HTML-Report, Disconnect am Ende |
| Sicherheits-Skript | JEA psrc/pssc Dateien vollständig |
| Automatisierungs-Task | Retry + Logging + Scheduled-Task-Registration |
| Kurzskript (<30 Zeilen) | Inline mit Kommentaren, kein volles Template nötig |
