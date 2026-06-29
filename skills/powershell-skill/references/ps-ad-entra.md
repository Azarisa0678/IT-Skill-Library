# PowerShell — Active Directory & Entra ID

AD-Administration, Entra ID Graph, Bulk-Operationen, Reporting, Automatisierung.

---

## Active Directory (RSAT)

```powershell
Import-Module ActiveDirectory

# ─── BENUTZER-MANAGEMENT ──────────────────────────────────────────────────────
# Benutzer erstellen
New-ADUser `
    -Name "Max Mustermann" `
    -GivenName "Max" `
    -Surname "Mustermann" `
    -SamAccountName "m.mustermann" `
    -UserPrincipalName "m.mustermann@firma.de" `
    -EmailAddress "m.mustermann@firma.de" `
    -Department "IT" `
    -Title "IT-Administrator" `
    -Manager (Get-ADUser -Filter {SamAccountName -eq "it-leiter"}) `
    -Path "OU=IT,OU=Benutzer,DC=firma,DC=local" `
    -AccountPassword (ConvertTo-SecureString "Temp-Pw!2024" -AsPlainText -Force) `
    -ChangePasswordAtLogon $true `
    -Enabled $true

# Benutzer aus CSV-Datei anlegen (Bulk-Import)
Import-Csv "neue-mitarbeiter.csv" -Delimiter ";" | ForEach-Object {
    try {
        $params = @{
            Name               = "$($_.Vorname) $($_.Nachname)"
            GivenName          = $_.Vorname
            Surname            = $_.Nachname
            SamAccountName     = "$($_.Vorname[0]).$($_.Nachname)".ToLower()
            UserPrincipalName  = "$($_.Vorname[0]).$($_.Nachname)@firma.de".ToLower()
            Department         = $_.Abteilung
            Path               = "OU=$($_.Abteilung),OU=Benutzer,DC=firma,DC=local"
            AccountPassword    = (ConvertTo-SecureString "Temp!2024" -AsPlainText -Force)
            ChangePasswordAtLogon = $true
            Enabled            = $true
        }
        New-ADUser @params
        Write-Host "OK: $($_.Vorname) $($_.Nachname)" -ForegroundColor Green
    } catch {
        Write-Host "FEHLER: $($_.Vorname) $($_.Nachname): $_" -ForegroundColor Red
    }
}

# ─── BENUTZER DEAKTIVIEREN (Offboarding) ─────────────────────────────────────
function Invoke-UserOffboarding {
    param([string]$SamAccountName, [string]$Reason = "Austritt")

    $user = Get-ADUser -Identity $SamAccountName -Properties MemberOf, Manager

    # 1. Konto deaktivieren
    Disable-ADAccount -Identity $SamAccountName

    # 2. Passwort zufällig setzen (verhindert weitere Nutzung)
    Set-ADAccountPassword -Identity $SamAccountName `
        -NewPassword (ConvertTo-SecureString (New-Guid).Guid -AsPlainText -Force) -Reset

    # 3. Aus allen Gruppen entfernen (außer "Domain Users")
    $user.MemberOf | ForEach-Object {
        Remove-ADGroupMember -Identity $_ -Members $SamAccountName -Confirm:$false
    }

    # 4. In Deaktiviert-OU verschieben
    Move-ADObject -Identity $user.DistinguishedName `
        -TargetPath "OU=Deaktiviert,DC=firma,DC=local"

    # 5. Beschreibung setzen
    Set-ADUser -Identity $SamAccountName `
        -Description "Deaktiviert: $(Get-Date -Format 'dd.MM.yyyy') | Grund: $Reason"

    Write-Host "Offboarding abgeschlossen: $SamAccountName" -ForegroundColor Yellow
    # Ggf. E-Mail an IT-Leitung senden
}

# ─── GRUPPEN-MANAGEMENT ───────────────────────────────────────────────────────
# Gruppe erstellen
New-ADGroup -Name "SG-VPN-Benutzer" `
    -GroupScope Global `
    -GroupCategory Security `
    -Path "OU=Gruppen,DC=firma,DC=local" `
    -Description "Benutzer mit VPN-Zugang"

# Mitglieder hinzufügen
Add-ADGroupMember -Identity "SG-VPN-Benutzer" `
    -Members "m.mustermann", "j.schmidt", "a.mueller"

# Verschachtelte Gruppen anzeigen (rekursiv)
function Get-ADGroupMembersRecursive {
    param([string]$GroupName)
    Get-ADGroupMember -Identity $GroupName -Recursive |
        Where-Object {$_.objectClass -eq "user"} |
        Select-Object Name, SamAccountName, DistinguishedName
}

# ─── STALE ACCOUNTS FINDEN ────────────────────────────────────────────────────
# Benutzer die sich > 90 Tage nicht angemeldet haben
$staleDate = (Get-Date).AddDays(-90)
Get-ADUser -Filter {
    LastLogonDate -lt $staleDate -and
    Enabled -eq $true -and
    PasswordNeverExpires -eq $false
} -Properties LastLogonDate, Department, Manager |
    Select-Object Name, SamAccountName, LastLogonDate, Department |
    Sort-Object LastLogonDate |
    Export-Csv "stale-accounts.csv" -Encoding UTF8 -NoTypeInformation

# Passwörter die bald ablaufen
$warnDate = (Get-Date).AddDays(14)
Get-ADUser -Filter {
    Enabled -eq $true -and
    PasswordNeverExpires -eq $false
} -Properties PasswordLastSet, PasswordExpired, EmailAddress |
    ForEach-Object {
        $maxAge = (Get-ADDefaultDomainPasswordPolicy).MaxPasswordAge.Days
        $expires = $_.PasswordLastSet.AddDays($maxAge)
        if ($expires -lt $warnDate -and -not $_.PasswordExpired) {
            [PSCustomObject]@{
                Name           = $_.Name
                UPN            = $_.UserPrincipalName
                PasswordExpires = $expires
                DaysLeft       = ($expires - (Get-Date)).Days
                Email          = $_.EmailAddress
            }
        }
    } | Sort-Object DaysLeft | Format-Table
```

---

## Entra ID — Microsoft Graph PowerShell

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All","Group.ReadWrite.All",
    "Directory.ReadWrite.All","AuditLog.Read.All"

# ─── BENUTZER-REPORTING ───────────────────────────────────────────────────────
# Alle Gäste (External Users) anzeigen
Get-MgUser -All -Filter "userType eq 'Guest'" |
    Select-Object DisplayName, UserPrincipalName, CreatedDateTime,
        @{N='LastLogin'; E={$_.SignInActivity.LastSignInDateTime}} |
    Sort-Object LastLogin | Format-Table

# Benutzer ohne MFA
$users = Get-MgUser -All -Property Id, DisplayName, UserPrincipalName
$users | ForEach-Object {
    $methods = Get-MgUserAuthenticationMethod -UserId $_.Id
    $hasMFA = $methods.Count -gt 1  # Mehr als nur Passwort
    if (-not $hasMFA) {
        [PSCustomObject]@{
            Name = $_.DisplayName
            UPN  = $_.UserPrincipalName
            MFA  = "FEHLT"
        }
    }
} | Export-Csv "users-without-mfa.csv" -Encoding UTF8 -NoTypeInformation

# ─── GRUPPEN-MANAGEMENT ───────────────────────────────────────────────────────
# M365 Group erstellen
$group = New-MgGroup -DisplayName "Projektteam Alpha" `
    -MailNickname "projektteam-alpha" `
    -GroupTypes @("Unified") `
    -MailEnabled:$true `
    -SecurityEnabled:$false `
    -Visibility "Private"

# Mitglied hinzufügen
$user = Get-MgUser -UserId "m.mustermann@firma.de"
New-MgGroupMember -GroupId $group.Id -DirectoryObjectId $user.Id

# ─── CONDITIONAL ACCESS AUSWERTEN ────────────────────────────────────────────
# Alle CA-Policies + Status
Get-MgIdentityConditionalAccessPolicy |
    Select-Object DisplayName, State,
        @{N='Benutzer'; E={$_.Conditions.Users.IncludeUsers -join ", "}},
        @{N='Apps';     E={$_.Conditions.Applications.IncludeApplications -join ", "}} |
    Format-Table -AutoSize -Wrap

# Policies in Report-Only-Modus finden (sollten irgendwann auf Enabled)
Get-MgIdentityConditionalAccessPolicy |
    Where-Object {$_.State -eq "enabledForReportingButNotEnforced"} |
    Select-Object DisplayName, ModifiedDateTime

# ─── SIGN-IN LOGS ANALYSIEREN ─────────────────────────────────────────────────
# Fehlgeschlagene Logins letzte 24h
Get-MgAuditLogSignIn -Filter "createdDateTime ge $((Get-Date).AddDays(-1).ToString('yyyy-MM-ddTHH:mm:ssZ')) and status/errorCode ne 0" |
    Group-Object {$_.Status.FailureReason} |
    Select-Object Count, Name | Sort-Object Count -Descending

Disconnect-MgGraph
```

---

## AD-Reporting (Schnellberichte)

```powershell
function Get-ADComplianceReport {
    param([string]$OutputPath = ".\AD-Compliance-$(Get-Date -Format yyyyMMdd).html")

    $report = @{
        StaleUsers      = (Get-ADUser -Filter {LastLogonDate -lt (Get-Date).AddDays(-90) -and Enabled -eq $true} -Properties LastLogonDate).Count
        NoPasswordExpiry = (Get-ADUser -Filter {PasswordNeverExpires -eq $true -and Enabled -eq $true}).Count
        AdminCount      = (Get-ADGroupMember "Domain Admins").Count
        DisabledInOU    = (Get-ADUser -Filter {Enabled -eq $false} -SearchBase "OU=Benutzer,DC=firma,DC=local").Count
    }

    $html = @"
<!DOCTYPE html><html><head><meta charset="UTF-8">
<style>body{font-family:Arial;} table{border-collapse:collapse;width:100%}
th{background:#003087;color:white;padding:8px} td{border:1px solid #ddd;padding:8px}
.warn{background:#fff3cd} .crit{background:#f8d7da}</style></head>
<body><h1>AD Compliance Report — $(Get-Date -Format "dd.MM.yyyy")</h1>
<table><tr><th>Metrik</th><th>Wert</th><th>Bewertung</th></tr>
<tr class='$(if($report.StaleUsers -gt 10){"crit"}else{""})'><td>Inaktive Benutzer (>90 Tage)</td><td>$($report.StaleUsers)</td><td>$(if($report.StaleUsers -gt 10){"⚠️ Prüfen"}else{"✅ OK"})</td></tr>
<tr class='$(if($report.NoPasswordExpiry -gt 5){"warn"}else{""})'><td>Passwort läuft nie ab</td><td>$($report.NoPasswordExpiry)</td><td>$(if($report.NoPasswordExpiry -gt 5){"⚠️ Prüfen"}else{"✅ OK"})</td></tr>
<tr class='$(if($report.AdminCount -gt 10){"crit"}else{""})'><td>Domain Admins</td><td>$($report.AdminCount)</td><td>$(if($report.AdminCount -gt 10){"⚠️ Zu viele"}else{"✅ OK"})</td></tr>
</table></body></html>
"@
    $html | Out-File -FilePath $OutputPath -Encoding UTF8
    Write-Host "Report erstellt: $OutputPath" -ForegroundColor Green
    Invoke-Item $OutputPath
}
