# Entra ID & Conditional Access

## Grundlagen

### Wichtige Konzepte
- **Entra ID** (früher Azure AD): Cloud-basierter Identitätsdienst
- **Tenant**: Dedizierte Instanz von Entra ID für eine Organisation
- **Primary Domain**: Standardmäßig `[tenant].onmicrosoft.com`
- **Verified Domain**: Eigene Domain (z.B. firma.de) — im Admin Center verifizieren

### Lizenzanforderungen
| Feature | Entra ID Free | P1 | P2 |
|---------|--------------|-----|-----|
| Basis-MFA | ✅ (Security Defaults) | ✅ | ✅ |
| Conditional Access | ❌ | ✅ | ✅ |
| PIM | ❌ | ❌ | ✅ |
| Identity Protection | ❌ | ❌ | ✅ |
| Access Reviews | ❌ | ✅ | ✅ |

---

## Benutzer & Gruppen

```powershell
# Verbinden mit Microsoft Graph
Connect-MgGraph -Scopes "User.ReadWrite.All","Group.ReadWrite.All","Directory.Read.All"

# Neuen Benutzer erstellen
$passwort = (ConvertTo-SecureString "TempPass123!" -AsPlainText -Force)
$params = @{
    DisplayName       = "Max Mustermann"
    UserPrincipalName = "m.mustermann@firma.de"
    MailNickname      = "m.mustermann"
    AccountEnabled    = $true
    PasswordProfile   = @{
        Password                      = "TempPass123!"
        ForceChangePasswordNextSignIn = $true
    }
    UsageLocation = "DE"
    Department    = "IT"
    JobTitle      = "IT-Administrator"
}
New-MgUser @params

# Benutzer in Gruppe aufnehmen
$user  = Get-MgUser -Filter "UserPrincipalName eq 'm.mustermann@firma.de'"
$group = Get-MgGroup -Filter "DisplayName eq 'IT-Abteilung'"
New-MgGroupMember -GroupId $group.Id -DirectoryObjectId $user.Id

# Alle Benutzer ohne Lizenz exportieren
Get-MgUser -All -Property DisplayName,UserPrincipalName,AssignedLicenses |
    Where-Object { $_.AssignedLicenses.Count -eq 0 -and $_.AccountEnabled } |
    Select-Object DisplayName, UserPrincipalName |
    Export-Csv "C:\Reports\BenutzerOhneLizenz.csv" -NoTypeInformation -Encoding UTF8

# Inaktive Benutzer (90+ Tage kein Login)
$cutoff = (Get-Date).AddDays(-90).ToString("yyyy-MM-ddTHH:mm:ssZ")
Get-MgUser -All -Filter "signInActivity/lastSignInDateTime le $cutoff" `
    -Property DisplayName,UserPrincipalName,SignInActivity |
    Select-Object DisplayName, UserPrincipalName,
        @{N='LetzterLogin';E={$_.SignInActivity.LastSignInDateTime}} |
    Export-Csv "C:\Reports\InaktiveBenutzer.csv" -NoTypeInformation -Encoding UTF8

# Benutzer deaktivieren
Update-MgUser -UserId "m.mustermann@firma.de" -AccountEnabled $false

# Benutzer löschen (soft delete — 30 Tage wiederherstellbar)
Remove-MgUser -UserId "m.mustermann@firma.de"
```

---

## MFA & Authentifizierungsmethoden

```powershell
# MFA-Registrierungsstatus aller Benutzer
Connect-MgGraph -Scopes "UserAuthenticationMethod.Read.All","Reports.Read.All"

Get-MgReportAuthenticationMethodUserRegistrationDetail -All |
    Select-Object UserPrincipalName, UserDisplayName,
        IsMfaRegistered, IsMfaCapable,
        IsPasswordlessCapable, IsSsprRegistered |
    Export-Csv "C:\Reports\MFA-Status.csv" -NoTypeInformation -Encoding UTF8

# Benutzer ohne MFA-Registrierung
Get-MgReportAuthenticationMethodUserRegistrationDetail -All |
    Where-Object { -not $_.IsMfaRegistered -and $_.IsEnabled } |
    Select-Object UserPrincipalName, UserDisplayName |
    Export-Csv "C:\Reports\BenutzerOhneMFA.csv" -NoTypeInformation -Encoding UTF8

# Authentifizierungsmethoden eines Benutzers anzeigen
$userId = (Get-MgUser -UserId "m.mustermann@firma.de").Id
Get-MgUserAuthenticationMethod -UserId $userId
```

---

## Conditional Access Policies

### Empfohlene Basis-Policies (Named Accounts)

```powershell
Connect-MgGraph -Scopes "Policy.ReadWrite.ConditionalAccess","Policy.Read.All"

# Policy 1: MFA für alle Benutzer (außer Notfallkonten)
$breakglassGroup = (Get-MgGroup -Filter "DisplayName eq 'CA-Ausnahme-Notfallkonten'").Id

$policy1 = @{
    DisplayName = "CA001 - MFA für alle Benutzer"
    State       = "enabled"
    Conditions  = @{
        Users = @{
            IncludeUsers  = @("All")
            ExcludeGroups = @($breakglassGroup)
        }
        Applications = @{ IncludeApplications = @("All") }
        ClientAppTypes = @("all")
    }
    GrantControls = @{
        Operator         = "OR"
        BuiltInControls  = @("mfa")
    }
}
New-MgIdentityConditionalAccessPolicy -BodyParameter $policy1

# Policy 2: Compliant Device für sensible Apps
$policy2 = @{
    DisplayName = "CA002 - Compliant Device für Exchange und SharePoint"
    State       = "enabledForReportingButNotEnforced"  # Erst im Report-Modus testen!
    Conditions  = @{
        Users        = @{ IncludeUsers = @("All"); ExcludeGroups = @($breakglassGroup) }
        Applications = @{
            IncludeApplications = @(
                "00000002-0000-0ff1-ce00-000000000000",  # Exchange Online
                "00000003-0000-0ff1-ce00-000000000000"   # SharePoint Online
            )
        }
    }
    GrantControls = @{
        Operator        = "AND"
        BuiltInControls = @("mfa","compliantDevice")
    }
}
New-MgIdentityConditionalAccessPolicy -BodyParameter $policy2

# Policy 3: Legacy-Authentifizierung blockieren
$policy3 = @{
    DisplayName = "CA003 - Legacy-Authentifizierung blockieren"
    State       = "enabled"
    Conditions  = @{
        Users        = @{ IncludeUsers = @("All") }
        Applications = @{ IncludeApplications = @("All") }
        ClientAppTypes = @("exchangeActiveSync","other")
    }
    GrantControls = @{
        Operator        = "OR"
        BuiltInControls = @("block")
    }
}
New-MgIdentityConditionalAccessPolicy -BodyParameter $policy3

# Alle CAPolicies anzeigen
Get-MgIdentityConditionalAccessPolicy |
    Select-Object DisplayName, State, CreatedDateTime |
    Sort-Object State | Format-Table -AutoSize
```

---

## Privileged Identity Management (PIM)

```powershell
# PIM erfordert Entra ID P2 Lizenz
Connect-MgGraph -Scopes "RoleManagement.ReadWrite.Directory","PrivilegedAccess.ReadWrite.AzureAD"

# Alle aktiven permanenten Admin-Zuweisungen finden (sollten eliminiert werden)
Get-MgRoleManagementDirectoryRoleAssignment -All |
    ForEach-Object {
        $role = Get-MgRoleManagementDirectoryRoleDefinition -UnifiedRoleDefinitionId $_.RoleDefinitionId
        $user = Get-MgUser -UserId $_.PrincipalId -EA 0
        [PSCustomObject]@{
            Benutzer   = $user.UserPrincipalName
            Rolle      = $role.DisplayName
            Typ        = "Permanent (Risiko!)"
        }
    } | Export-Csv "C:\Reports\PermanenteAdminRollen.csv" -NoTypeInformation -Encoding UTF8

# PIM-berechtigte Zuweisung erstellen (statt permanent)
$roleId = (Get-MgRoleManagementDirectoryRoleDefinition -Filter "DisplayName eq 'User Administrator'").Id
$userId = (Get-MgUser -UserId "m.mustermann@firma.de").Id

$params = @{
    Action           = "adminAssign"
    Justification    = "IT-Admin benötigt User Administrator für Routinetätigkeiten"
    RoleDefinitionId = $roleId
    DirectoryScopeId = "/"
    PrincipalId      = $userId
    ScheduleInfo     = @{
        StartDateTime = (Get-Date).ToUniversalTime().ToString("o")
        Expiration    = @{ Type = "noExpiration" }
    }
}
New-MgRoleManagementDirectoryRoleEligibilityScheduleRequest -BodyParameter $params
```

---

## Notfallkonten (Break-Glass Accounts)

```powershell
# Notfallkonten sollten:
# - NICHT in Conditional Access Policies eingeschlossen sein
# - Starkes, sicher verwahrtes Passwort haben (kein SSO, kein MFA-App)
# - Physisch verwahrte FIDO2-Keys für MFA
# - In einer eigenen Gruppe sein (CA-Ausnahme-Notfallkonten)
# - Regelmäßig auf Login-Aktivität überwacht werden

# Login-Aktivität der Notfallkonten überwachen
$notfallkonten = @("notfall1@firma.de","notfall2@firma.de")
foreach ($konto in $notfallkonten) {
    $user = Get-MgUser -UserId $konto -Property SignInActivity
    Write-Output "$konto - Letzter Login: $($user.SignInActivity.LastSignInDateTime)"
}
```

---

## Conditional Access — Checkliste

- [ ] Security Defaults deaktiviert (wenn eigene CA-Policies vorhanden)
- [ ] Mindestens 2 Notfallkonten erstellt und dokumentiert
- [ ] CA001: MFA für alle Benutzer aktiviert
- [ ] CA002: Compliant Device (zunächst Report-Modus, dann enforce)
- [ ] CA003: Legacy-Authentifizierung blockiert
- [ ] CA004: Risiko-basierte Policies (Identity Protection, P2 erforderlich)
- [ ] PIM für alle Global Admins und privilegierten Rollen
- [ ] Quartalsweise Access Reviews konfiguriert
- [ ] Named Locations für vertrauenswürdige Standorte definiert
