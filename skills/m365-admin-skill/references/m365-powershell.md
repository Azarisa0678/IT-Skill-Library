# Microsoft 365 PowerShell Automatisierung

## Module & Verbindung

```powershell
# Benötigte Module installieren
Install-Module Microsoft.Graph -Scope CurrentUser -Force
Install-Module Az -Scope CurrentUser -Force
Install-Module ExchangeOnlineManagement -Scope CurrentUser -Force
Install-Module Microsoft.Online.SharePoint.PowerShell -Scope CurrentUser -Force

# Verbindungen
Connect-MgGraph -Scopes "User.ReadWrite.All","Group.ReadWrite.All","Mail.Read"
Connect-AzAccount -TenantId $tenantId
Connect-ExchangeOnline -UserPrincipalName "admin@firma.de"
Connect-SPOService -Url "https://firma-admin.sharepoint.com"

# Für Automation (Service Principal / App Registration)
# Einmalige App-Registrierung im Portal: Entra ID → App-Registrierungen
$tenantId = "your-tenant-id"
$clientId = "your-app-client-id"
$clientSecret = Get-Secret -Name "GraphAppSecret" -Vault "LocalStore"

Connect-MgGraph -TenantId $tenantId -ClientId $clientId -ClientSecret $clientSecret
```

---

## Onboarding-Automatisierung

```powershell
<#
.SYNOPSIS  Vollautomatisches Onboarding neuer Mitarbeiter
.EXAMPLE   .\New-Mitarbeiter.ps1 -Vorname "Max" -Nachname "Mustermann" -Abteilung "IT" -Manager "chef@firma.de"
#>
[CmdletBinding(SupportsShouldProcess)]
param(
    [Parameter(Mandatory)][string]$Vorname,
    [Parameter(Mandatory)][string]$Nachname,
    [Parameter(Mandatory)][string]$Abteilung,
    [Parameter(Mandatory)][string]$ManagerUPN,
    [string]$Startdatum = (Get-Date -f 'yyyy-MM-dd')
)

Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'

function Write-Log {
    param([string]$Msg, [string]$Level = 'INFO')
    Write-Host "[$Level] $Msg" -ForegroundColor $(if($Level -eq 'ERROR'){'Red'}elseif($Level -eq 'WARN'){'Yellow'}else{'Cyan'})
}

Connect-MgGraph -Scopes "User.ReadWrite.All","Group.ReadWrite.All" -NoWelcome

# UPN generieren
$benutzername = "$($Vorname.ToLower()[0]).$($Nachname.ToLower())"
$upn = "$benutzername@firma.de"
Write-Log "Erstelle Benutzer: $upn"

# Temporäres Passwort
$tempPasswort = "Start$(Get-Random -Min 1000 -Max 9999)!$(Get-Random -Min 10 -Max 99)"

# Benutzer erstellen
if ($PSCmdlet.ShouldProcess($upn, "Benutzer erstellen")) {
    $neuerUser = New-MgUser -DisplayName "$Vorname $Nachname" `
        -UserPrincipalName $upn `
        -MailNickname $benutzername `
        -GivenName $Vorname `
        -Surname $Nachname `
        -Department $Abteilung `
        -AccountEnabled $true `
        -UsageLocation "DE" `
        -PasswordProfile @{
            Password                      = $tempPasswort
            ForceChangePasswordNextSignIn = $true
        }
    Write-Log "Benutzer erstellt: $($neuerUser.Id)"

    # Manager zuweisen
    $manager = Get-MgUser -UserId $ManagerUPN
    Set-MgUserManagerByRef -UserId $neuerUser.Id `
        -BodyParameter @{"@odata.id" = "https://graph.microsoft.com/v1.0/users/$($manager.Id)"}

    # Gruppen zuweisen
    $abteilungsGruppen = @{
        "IT"         = @("IT-Abteilung","VPN-Benutzer","M365-E3")
        "HR"         = @("HR-Abteilung","M365-E3")
        "Buchhaltung"= @("Buchhaltung","Finance-Apps","M365-E3")
    }
    if ($abteilungsGruppen.ContainsKey($Abteilung)) {
        foreach ($gruppenName in $abteilungsGruppen[$Abteilung]) {
            $gruppe = Get-MgGroup -Filter "DisplayName eq '$gruppenName'" -Top 1
            if ($gruppe) {
                New-MgGroupMember -GroupId $gruppe.Id -DirectoryObjectId $neuerUser.Id
                Write-Log "Zu Gruppe hinzugefügt: $gruppenName"
            }
        }
    }

    # Willkommens-E-Mail an Manager senden
    $mailBody = @{
        Message = @{
            Subject = "Neuer Mitarbeiter: $Vorname $Nachname"
            Body    = @{
                ContentType = "HTML"
                Content     = @"
<h2>Willkommen, $Vorname $Nachname!</h2>
<p>Das Konto wurde erfolgreich erstellt.</p>
<table>
<tr><td><b>Benutzername:</b></td><td>$upn</td></tr>
<tr><td><b>Temporäres Passwort:</b></td><td>$tempPasswort</td></tr>
<tr><td><b>Abteilung:</b></td><td>$Abteilung</td></tr>
<tr><td><b>Startdatum:</b></td><td>$Startdatum</td></tr>
</table>
<p><i>Das Passwort muss beim ersten Login geändert werden.</i></p>
"@
            }
            ToRecipients = @(@{EmailAddress = @{Address = $ManagerUPN}})
        }
    }
    Send-MgUserMail -UserId "admin@firma.de" -BodyParameter $mailBody
    Write-Log "Willkommens-E-Mail gesendet an $ManagerUPN"
}
```

---

## Offboarding-Automatisierung

```powershell
<#
.SYNOPSIS  Sicheres Offboarding eines ausscheidenden Mitarbeiters
.EXAMPLE   .\Remove-Mitarbeiter.ps1 -UPN "m.mustermann@firma.de" -Grund "Kündigung"
#>
param(
    [Parameter(Mandatory)][string]$UPN,
    [Parameter(Mandatory)][string]$Grund,
    [string]$NachfolgerUPN = ""
)

$timestamp = Get-Date -f 'yyyy-MM-dd HH:mm'
$user = Get-MgUser -UserId $UPN -Property Id,DisplayName,AssignedLicenses,MemberOf

Write-Host "Starte Offboarding für: $($user.DisplayName) ($UPN)" -ForegroundColor Yellow

# 1. Konto sofort deaktivieren
Update-MgUser -UserId $user.Id -AccountEnabled $false
Write-Host "✅ Konto deaktiviert"

# 2. Alle aktiven Sessions beenden
Revoke-MgUserSignInSession -UserId $user.Id
Write-Host "✅ Alle Sessions beendet"

# 3. MFA-Methoden entfernen (verhindert Wiederzugang)
Get-MgUserAuthenticationMethod -UserId $user.Id | ForEach-Object {
    # Methoden-spezifische Remove-Befehle
}

# 4. Aus allen Gruppen entfernen
$gruppen = Get-MgUserMemberOf -UserId $user.Id
foreach ($gruppe in $gruppen) {
    try {
        Remove-MgGroupMemberByRef -GroupId $gruppe.Id -DirectoryMemberId $user.Id
    } catch { }
}
Write-Host "✅ Aus $($gruppen.Count) Gruppen entfernt"

# 5. Passwort randomisieren
$neuesPasswort = [System.Web.Security.Membership]::GeneratePassword(32, 8)
Update-MgUser -UserId $user.Id -PasswordProfile @{
    Password                      = $neuesPasswort
    ForceChangePasswordNextSignIn = $true
}
Write-Host "✅ Passwort randomisiert"

# 6. Lizenzen entfernen
$lizenzen = (Get-MgUserLicenseDetail -UserId $user.Id).SkuId
Set-MgUserLicense -UserId $user.Id -AddLicenses @() -RemoveLicenses $lizenzen
Write-Host "✅ $($lizenzen.Count) Lizenz(en) entfernt"

# 7. Postfach weiterleiten (optional)
if ($NachfolgerUPN) {
    Connect-ExchangeOnline -UserPrincipalName "admin@firma.de" -ShowBanner:$false
    Set-Mailbox -Identity $UPN -ForwardingSmtpAddress $NachfolgerUPN -DeliverToMailboxAndForward $false
    Write-Host "✅ E-Mails weitergeleitet an $NachfolgerUPN"
}

# 8. Beschreibung aktualisieren
Update-MgUser -UserId $user.Id -Description "Offboarding: $timestamp | Grund: $Grund | Bearbeiter: $env:USERNAME"
Write-Host "✅ Offboarding abgeschlossen für $UPN" -ForegroundColor Green
```

---

## Exchange Online Verwaltung

```powershell
Connect-ExchangeOnline -UserPrincipalName "admin@firma.de" -ShowBanner:$false

# Postfachgröße aller Benutzer
Get-Mailbox -ResultSize Unlimited |
    Get-MailboxStatistics |
    Select-Object DisplayName, TotalItemSize, ItemCount,
        @{N='GrößeMB';E={[math]::Round($_.TotalItemSize.Value.ToMB(),1)}} |
    Sort-Object GrößeMB -Descending |
    Export-Csv "C:\Reports\Postfachgroessen.csv" -NoTypeInformation -Encoding UTF8

# Shared Mailbox erstellen
New-Mailbox -Shared -Name "Info" -DisplayName "Info Postfach" `
    -PrimarySmtpAddress "info@firma.de"
Add-MailboxPermission -Identity "info@firma.de" `
    -User "m.mustermann@firma.de" -AccessRights FullAccess `
    -InheritanceType All -AutoMapping $true

# Anti-Spam und Mail-Flow Regeln prüfen
Get-TransportRule | Select-Object Name, State, Priority |
    Sort-Object Priority | Format-Table -AutoSize

# DMARC/DKIM Status
Get-DkimSigningConfig | Select-Object Domain, Enabled, Status | Format-Table
```

---

## Teams Verwaltung

```powershell
# Teams-Module
Install-Module MicrosoftTeams -Force
Connect-MmicrosoftTeams

# Alle Teams auflisten mit Mitgliederzahl
Get-Team | ForEach-Object {
    $members = Get-TeamUser -GroupId $_.GroupId
    [PSCustomObject]@{
        Name      = $_.DisplayName
        Mitglieder = $members.Count
        Archiviert = $_.Archived
        Sichtbarkeit = $_.Visibility
    }
} | Sort-Object Mitglieder -Descending |
    Export-Csv "C:\Reports\Teams.csv" -NoTypeInformation -Encoding UTF8

# Inaktive Teams identifizieren (keine Aktivität in 90 Tagen)
# Über Microsoft 365 Admin Center → Reports → Teams Activity
# Oder über Graph API Reports

# Gast-Benutzer in Teams finden
Get-AzureADUser -Filter "userType eq 'Guest'" |
    Select-Object DisplayName, UserPrincipalName, CreationType |
    Export-Csv "C:\Reports\GastBenutzer.csv" -NoTypeInformation -Encoding UTF8
```

---

## M365 Admin Checkliste: PowerShell-Automatisierung

- [ ] Microsoft Graph SDK installiert und aktuell
- [ ] App-Registrierung für unattended Scripts erstellt (kein interaktiver Login)
- [ ] Client Secret in Azure Key Vault (nicht in Script-Datei)
- [ ] Onboarding-Script getestet und dokumentiert
- [ ] Offboarding-Script getestet und freigegeben
- [ ] Wöchentlicher Lizenz-Audit automatisiert
- [ ] Inaktive Benutzer Report (monatlich per Aufgabenplanung)
- [ ] Exchange Postfachgröße Report (monatlich)
- [ ] Scripts in Git-Repository versioniert
- [ ] Log-Dateien für alle Automation-Scripts
