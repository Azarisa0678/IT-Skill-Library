# Troubleshooting-Guides Microsoft 365

## Diagnose-Prinzipien

1. **Eingrenzen:** Betrifft es einen Benutzer oder alle? Ein Gerät oder mehrere?
2. **Reproduzieren:** Genau den Fehler nachvollziehen — Screenshot, Fehlermeldung, Zeitpunkt
3. **Logs prüfen:** Entra ID Sign-in Logs, Message Trace, Intune-Gerätestatus
4. **Änderungen prüfen:** Was hat sich in den letzten 24-48h geändert? (GPO, CA-Policy, Update)

---

## Szenario 1: Benutzer kann sich nicht anmelden

### Schnelldiagnose
```powershell
Connect-MgGraph -Scopes "User.Read.All","AuditLog.Read.All"

# Konto-Status prüfen
$user = Get-MgUser -UserId "benutzer@firma.de" -Property AccountEnabled,
    SignInActivity, AssignedLicenses, UserPrincipalName
Write-Host "Konto aktiv: $($user.AccountEnabled)"
Write-Host "Letzter Login: $($user.SignInActivity.LastSignInDateTime)"
Write-Host "Lizenzen: $($user.AssignedLicenses.Count)"

# Sign-in Logs der letzten 24h abrufen
$signIns = Get-MgAuditLogSignIn -Filter "userPrincipalName eq 'benutzer@firma.de'" -Top 20 |
    Select-Object CreatedDateTime, AppDisplayName, Status,
        ConditionalAccessStatus, IPAddress, Location

$signIns | Format-Table -AutoSize

# Fehlercodes interpretieren
# 50126 = Falsches Passwort
# 50074 = MFA erforderlich
# 50076 = MFA abgelehnt
# 53003 = Conditional Access blockiert
# 50055 = Passwort abgelaufen
# 50057 = Konto deaktiviert
# 70011 = Ungültige Scope-Anfrage
```

### Häufige Ursachen & Lösungen

```powershell
# Ursache 1: Konto gesperrt (zu viele Fehlversuche)
# Lösung: Konto entsperren
Unlock-ADAccount -Identity "benutzername"
# Für Cloud-Only:
Update-MgUser -UserId "benutzer@firma.de" -AccountEnabled $true

# Ursache 2: Passwort abgelaufen
Set-MgUserPassword -UserId "benutzer@firma.de" -PasswordProfile @{
    Password = "NeuesTemp123!"
    ForceChangePasswordNextSignIn = $true
}

# Ursache 3: MFA-Methode nicht verfügbar (Handy verloren)
# MFA-Methoden zurücksetzen
$userId = (Get-MgUser -UserId "benutzer@firma.de").Id
# Temporärer Access Pass (TAP) erstellen
$tap = New-MgUserAuthenticationTemporaryAccessPassMethod -UserId $userId `
    -BodyParameter @{
        isUsableOnce   = $false
        lifetimeInMinutes = 480  # 8 Stunden
    }
Write-Host "Temporärer Zugangscode: $($tap.TemporaryAccessPass)"

# Ursache 4: Conditional Access blockiert
# Im Sign-in Log: ConditionalAccessStatus = "failure"
# Details in CA-Insights prüfen: Entra ID → Sicherheit → Conditional Access → Insights
```

---

## Szenario 2: MFA funktioniert nicht / Benutzer wird nicht nach MFA gefragt

```powershell
# Prüfen ob Security Defaults oder CA-Policies aktiv sind
Connect-MgGraph -Scopes "Policy.Read.All"

# Security Defaults Status
$secDefaults = Get-MgPolicyIdentitySecurityDefaultEnforcementPolicy
Write-Host "Security Defaults aktiv: $($secDefaults.IsEnabled)"

# Alle CA-Policies mit MFA-Anforderung
Get-MgIdentityConditionalAccessPolicy |
    Where-Object { $_.GrantControls.BuiltInControls -contains "mfa" } |
    Select-Object DisplayName, State | Format-Table

# Häufige Ursachen:
# 1. Benutzer ist in Ausnahmegruppe (CA-Ausnahme-Notfallkonten)
# 2. Named Location als vertrauenswürdig markiert → MFA übersprungen
# 3. CA-Policy im Report-Only-Modus → enforced nicht

# Named Locations prüfen
Get-MgIdentityConditionalAccessNamedLocation |
    Select-Object DisplayName, Id | Format-Table

# Benutzer-Gruppenmitgliedschaften prüfen (Ausnahmegruppen?)
Get-MgUserMemberOf -UserId "benutzer@firma.de" |
    Select-Object @{N='Gruppe';E={$_.AdditionalProperties.displayName}} |
    Format-Table
```

---

## Szenario 3: E-Mail wird nicht zugestellt

```powershell
Connect-ExchangeOnline -UserPrincipalName "admin@firma.de" -ShowBanner:$false

# Message Trace (letzten 48h)
Get-MessageTrace `
    -SenderAddress "absender@extern.de" `
    -RecipientAddress "empfaenger@firma.de" `
    -StartDate (Get-Date).AddHours(-48) `
    -EndDate (Get-Date) |
    Select-Object Received, SenderAddress, RecipientAddress, Subject, Status, ToIP |
    Format-Table -AutoSize

# Detaillierte Trace-Informationen
$trace = Get-MessageTrace -MessageId "<message-id@domain.de>"
Get-MessageTraceDetail -MessageTraceId $trace.MessageTraceId -RecipientAddress "empfaenger@firma.de" |
    Select-Object Date, Event, Action, Detail | Format-Table -AutoSize

# Mögliche Status-Werte:
# GettingStatus   = Noch in Verarbeitung
# Failed          = Zustellung fehlgeschlagen
# Delivered       = Erfolgreich zugestellt
# FilteredAsSpam  = Als Spam gefiltert
# Quarantined     = In Quarantäne

# Quarantäne prüfen
Get-QuarantineMessage -RecipientAddress "empfaenger@firma.de" -StartReceivedDate (Get-Date).AddDays(-7) |
    Select-Object ReceivedTime, SenderAddress, Subject, QuarantineTypes | Format-Table

# Transport Rules prüfen (blockiert eine Regel die E-Mail?)
Get-TransportRule | Where-Object { $_.State -eq "Enabled" } |
    Select-Object Name, Priority, State | Format-Table

# Anti-Spam Policy prüfen
Get-HostedContentFilterPolicy | Select-Object Name, SpamAction, BulkSpamAction,
    HighConfidenceSpamAction | Format-Table
```

---

## Szenario 4: Entra Connect Sync-Fehler

```powershell
Import-Module ADSync

# Letzte Sync-Läufe prüfen
Get-ADSyncRunStepResult | Select-Object ConnectorName, StepResult, StartDate,
    EndDate, CountImported, CountExported, CountSyncFailures |
    Sort-Object StartDate -Descending | Select-Object -First 10 | Format-Table -AutoSize

# Sync-Fehler detailliert
Get-ADSyncConnectorRunStatus | Format-List

# Objekte mit Sync-Fehlern
$connector = Get-ADSyncConnector | Where-Object { $_.Name -match "firma.de" }
Get-ADSyncCSObject -ConnectorIdentifier $connector.Identifier `
    -HasSyncError $true |
    Select-Object DistinguishedName, SyncExportError | Format-Table

# Häufige Sync-Fehler und Lösungen:
# AttributeValueMustBeUnique → Doppelter UPN oder ProxyAddress
# ObjectTypeMismatch → Objekt-Typ stimmt nicht überein
# InvalidSoftMatch → Soft Match auf falschem Objekt

# Doppelte UPNs in AD finden
Search-ADAccount -UsersOnly |
    Get-ADUser -Properties UserPrincipalName |
    Group-Object UserPrincipalName |
    Where-Object { $_.Count -gt 1 } |
    Select-Object Name, Count | Format-Table

# Sync manuell auslösen nach Fehlerbehebung
Start-ADSyncSyncCycle -PolicyType Delta
```

---

## Szenario 5: Intune Gerät nicht compliant

```powershell
Connect-MgGraph -Scopes "DeviceManagementManagedDevices.Read.All"

# Gerät suchen und Status prüfen
$geraet = Get-MgDeviceManagementManagedDevice -Filter "deviceName eq 'LAPTOP-XYZ123'"
$geraet | Select-Object DeviceName, ComplianceState, LastSyncDateTime,
    OperatingSystem, OsVersion, UserPrincipalName | Format-List

# Compliance-Details
Get-MgDeviceManagementManagedDeviceCompliancePolicyState `
    -ManagedDeviceId $geraet.Id |
    Select-Object DisplayName, State, SettingCount | Format-Table

# Häufige Ursachen Non-Compliance:
# BitLocker nicht aktiv → Gerät neu verschlüsseln
# Defender nicht aktiv → Dienst prüfen
# OS-Version veraltet → Windows Update erzwingen
# Langer Sync-Ausfall → Gerät in Intune: "Sync" auslösen

# Sync aus Intune-Portal: Gerät → Sync
# Oder auf Gerät selbst:
# Einstellungen → Konten → Auf Arbeits- oder Schulkonto zugreifen → [Konto] → Info → Sync

# Gerät zurücksetzen (letztes Mittel)
Invoke-MgDeviceManagementManagedDeviceWipe -ManagedDeviceId $geraet.Id
```

---

## Szenario 6: Teams-Anruf Probleme

```powershell
# Teams Call Quality Dashboard nutzen
# Portal: admin.teams.microsoft.com → Analyse & Berichte → Anrufqualität

# Netzwerk-Anforderungen prüfen
# Microsoft 365 Network Assessment Tool herunterladen und ausführen
# https://connectivity.office.com

# Häufige Ursachen schlechter Anrufqualität:
# - Netzwerklatenz > 150ms
# - Paketverlust > 1%
# - Jitter > 30ms
# - VPN ohne Split Tunneling (Teams-Traffic über VPN)

# VPN Split Tunneling konfigurieren (Empfehlung)
# Teams-IPs und Ports vom VPN ausschließen:
# IPs: 13.107.64.0/18, 52.112.0.0/14, 52.122.0.0/15
# Ports: UDP 3478-3481

# Diagnose-Informationen aus Teams sammeln
# Im Teams-Client: Strg+Alt+Shift+1 → Diagnosedateien erstellen
# Pfad: %AppData%\Microsoft\Teams\logs.txt
```

---

## Diagnose-Checkliste bei Eskalation

```
Welche Informationen beim Ticket sammeln:

□ Betroffene Benutzer: Einzelperson oder mehrere? Welche?
□ Betroffene Geräte: Windows, Mac, mobil, alle?
□ Fehlermeldung: Exakter Text, Screenshot, Fehlercode
□ Zeitpunkt: Wann begann das Problem? Plötzlich oder schleichend?
□ Änderungen: Was hat sich geändert? Update, neue Policy, Passwortänderung?
□ Reproduzierbar: Tritt der Fehler immer auf oder gelegentlich?
□ Umgehung vorhanden: Gibt es einen Workaround?
□ Logs: Sign-in Log URL, Message Trace ID, Gerätename in Intune
□ Auswirkung: Kritisch (Produktionsausfall) oder eingeschränkt?

Eskalations-Ressourcen:
- Microsoft Support: admin.microsoft.com → Support → Neue Serviceanfrage
- Microsoft Tech Community: techcommunity.microsoft.com
- Microsoft Docs: learn.microsoft.com
- Service Health: admin.microsoft.com → Integrität → Dienstintegrität
```
