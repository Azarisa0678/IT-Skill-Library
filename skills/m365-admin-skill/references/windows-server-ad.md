# Windows Server & Active Directory

## Active Directory Health Check

```powershell
# DC-Diagnose ausführen
dcdiag /test:all /v /logfile:C:\Logs\dcdiag.txt

# Replikationsstatus prüfen
repadmin /replsummary
repadmin /showrepl
repadmin /showrepl * /csv > C:\Logs\replication.csv

# Zeitdienst prüfen (W32TM)
w32tm /query /status
w32tm /query /peers

# SYSVOL-Replikation prüfen (DFSR)
dfsrdiag ReplicationState /Member:$env:COMPUTERNAME
Get-DfsReplicatedFolder -GroupName "Domain System Volume"

# AD-Gesamtstatus (PowerShell)
Import-Module ActiveDirectory
Get-ADDomain | Select-Object Name, DNSRoot, DomainMode, PDCEmulator,
    RIDMaster, InfrastructureMaster, SchemaMaster | Format-List

# Alle DCs auflisten
Get-ADDomainController -Filter * |
    Select-Object Name, Site, IPv4Address, IsGlobalCatalog,
        IsReadOnly, OperatingSystem | Format-Table -AutoSize
```

---

## AD-Benutzerverwaltung

```powershell
# Benutzer mit Ablaufdatum erstellen (z.B. Zeitarbeiter)
New-ADUser `
    -Name "Temp Mustermann" `
    -SamAccountName "temp.mustermann" `
    -UserPrincipalName "temp.mustermann@firma.de" `
    -GivenName "Temp" `
    -Surname "Mustermann" `
    -Department "Extern" `
    -AccountExpirationDate (Get-Date).AddMonths(3) `
    -AccountPassword (ConvertTo-SecureString "TempPass123!" -AsPlainText -Force) `
    -ChangePasswordAtLogon $true `
    -Enabled $true `
    -Path "OU=Zeitarbeiter,OU=Benutzer,DC=firma,DC=de"

# Massenerstellung aus CSV
Import-Csv "C:\benutzer.csv" -Delimiter ";" | ForEach-Object {
    New-ADUser `
        -Name "$($_.Vorname) $($_.Nachname)" `
        -SamAccountName $_.Benutzername `
        -UserPrincipalName "$($_.Benutzername)@firma.de" `
        -GivenName $_.Vorname `
        -Surname $_.Nachname `
        -Department $_.Abteilung `
        -AccountPassword (ConvertTo-SecureString $_.Passwort -AsPlainText -Force) `
        -ChangePasswordAtLogon $true `
        -Enabled $true `
        -Path "OU=$($_.Abteilung),OU=Benutzer,DC=firma,DC=de"
    Add-ADGroupMember -Identity $_.Gruppe -Members $_.Benutzername
    Write-Host "Erstellt: $($_.Benutzername)" -ForegroundColor Green
}

# Benutzer beim Ausscheiden deaktivieren und verschieben
function Disable-AusgeschiedenerBenutzer {
    param([string]$SamAccountName, [string]$Grund)
    $user = Get-ADUser $SamAccountName -Properties MemberOf
    # Aus allen Gruppen entfernen
    $user.MemberOf | ForEach-Object { Remove-ADGroupMember -Identity $_ -Members $SamAccountName -Confirm:$false }
    # Konto deaktivieren
    Disable-ADAccount -Identity $SamAccountName
    # Passwort randomisieren
    Set-ADAccountPassword -Identity $SamAccountName `
        -NewPassword (ConvertTo-SecureString ([System.Web.Security.Membership]::GeneratePassword(24,4)) -AsPlainText -Force)
    # Beschreibung setzen
    Set-ADUser -Identity $SamAccountName -Description "Deaktiviert: $(Get-Date -f 'yyyy-MM-dd') - Grund: $Grund"
    # In Deaktiviert-OU verschieben
    Move-ADObject -Identity (Get-ADUser $SamAccountName).DistinguishedName `
        -TargetPath "OU=Deaktiviert,DC=firma,DC=de"
    Write-Host "Benutzer $SamAccountName deaktiviert und verschoben." -ForegroundColor Yellow
}
```

---

## Gruppenrichtlinien (GPO)

```powershell
Import-Module GroupPolicy

# GPO-Übersicht aller Policies
Get-GPO -All | Select-Object DisplayName, GpoStatus, CreationTime, ModificationTime |
    Sort-Object ModificationTime -Descending | Format-Table -AutoSize

# GPO erstellen und verknüpfen
$gpo = New-GPO -Name "Firma - Windows Sicherheitseinstellungen" `
    -Comment "Basis-Sicherheitskonfiguration für alle Workstations"
New-GPLink -Name "Firma - Windows Sicherheitseinstellungen" `
    -Target "OU=Workstations,DC=firma,DC=de" `
    -LinkEnabled Yes

# Registry-Einstellung per GPO setzen
# Beispiel: Screensaver-Passwort erzwingen
Set-GPRegistryValue `
    -Name "Firma - Windows Sicherheitseinstellungen" `
    -Key "HKCU\Software\Policies\Microsoft\Windows\Control Panel\Desktop" `
    -ValueName "ScreenSaverIsSecure" `
    -Type String `
    -Value "1"

# GPO-Backup (regelmäßig ausführen!)
$backupPath = "C:\GPO-Backup\$(Get-Date -f 'yyyyMMdd')"
New-Item -ItemType Directory -Path $backupPath -Force | Out-Null
Backup-GPO -All -Path $backupPath
Write-Host "GPO-Backup erstellt: $backupPath"

# GPO-Vererbung für OU prüfen
Get-GPInheritance -Target "OU=Workstations,DC=firma,DC=de" |
    Select-Object -ExpandProperty InheritedGpoLinks |
    Select-Object DisplayName, GpoId, Order, Enabled | Format-Table -AutoSize
```

---

## Windows Server Hardening

```powershell
# Windows Firewall aktivieren und konfigurieren
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True
Set-NetFirewallProfile -Profile Domain -DefaultInboundAction Block -DefaultOutboundAction Allow

# RDP-Zugriff absichern
# NLA erzwingen
Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `
    -Name "UserAuthentication" -Value 1

# RDP nur aus internem Netz erlauben
New-NetFirewallRule -DisplayName "RDP - Nur intern" `
    -Direction Inbound -Protocol TCP -LocalPort 3389 `
    -RemoteAddress "192.168.0.0/16" -Action Allow

# SMBv1 deaktivieren (kritische Sicherheitslücke!)
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force

# PowerShell Logging aktivieren
$regPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell"
@("ScriptBlockLogging","ModuleLogging","Transcription") | ForEach-Object {
    $fullPath = "$regPath\$_"
    New-Item -Path $fullPath -Force | Out-Null
    Set-ItemProperty -Path $fullPath -Name "Enable$_" -Value 1
}

# Windows Defender Einstellungen
Set-MpPreference -DisableRealtimeMonitoring $false
Set-MpPreference -CloudBlockLevel High
Set-MpPreference -MAPSReporting Advanced
Set-MpPreference -SubmitSamplesConsent SendAllSamples

# LAPS (Local Administrator Password Solution) einrichten
# Windows LAPS ist ab Windows Server 2019 / Windows 10 20H2 eingebaut
# Konfiguration per Intune oder GPO
```

---

## AD-Sicherheitsaudit

```powershell
# Kerberoastable Accounts
Get-ADUser -Filter {ServicePrincipalName -ne "$null" -and Enabled -eq $true} `
    -Properties ServicePrincipalName, PasswordLastSet, LastLogonDate |
    Select-Object Name, SamAccountName, ServicePrincipalName, PasswordLastSet |
    Export-Csv "C:\Reports\Kerberoastable.csv" -NoTypeInformation -Encoding UTF8

# AS-REP Roastable Accounts (PreAuth deaktiviert)
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true -and Enabled -eq $true} `
    -Properties DoesNotRequirePreAuth |
    Select-Object Name, SamAccountName |
    Export-Csv "C:\Reports\ASREPRoastable.csv" -NoTypeInformation -Encoding UTF8

# Passwörter nie ablaufend
Get-ADUser -Filter {PasswordNeverExpires -eq $true -and Enabled -eq $true} `
    -Properties PasswordNeverExpires, PasswordLastSet |
    Select-Object Name, SamAccountName, PasswordLastSet |
    Export-Csv "C:\Reports\PasswortNieAblaufend.csv" -NoTypeInformation -Encoding UTF8

# Domain Admins auflisten
Get-ADGroupMember "Domain Admins" -Recursive |
    Get-ADUser -Properties LastLogonDate, PasswordLastSet |
    Select-Object Name, SamAccountName, Enabled, LastLogonDate, PasswordLastSet |
    Export-Csv "C:\Reports\DomainAdmins.csv" -NoTypeInformation -Encoding UTF8

# Computerkonten ohne Login (90+ Tage)
$cutoff = (Get-Date).AddDays(-90)
Get-ADComputer -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} `
    -Properties LastLogonDate, OperatingSystem |
    Select-Object Name, LastLogonDate, OperatingSystem |
    Export-Csv "C:\Reports\InaktiveComputer.csv" -NoTypeInformation -Encoding UTF8
```

---

## Windows Server Checkliste

- [ ] Alle DCs: dcdiag ohne Fehler
- [ ] Replikation: repadmin /replsummary ohne Fehler
- [ ] SMBv1 deaktiviert auf allen Systemen
- [ ] RDP: NLA erzwungen, nur aus internem Netz
- [ ] Windows Defender: Echtzeitschutz aktiv, Cloud-Schutz aktiv
- [ ] LAPS auf allen Workstations und Member-Servern
- [ ] PowerShell Script Block Logging aktiviert (GPO)
- [ ] GPO-Backup: wöchentlich automatisiert
- [ ] AD-Papierkorb aktiviert (Get-ADOptionalFeature)
- [ ] Fine-Grained Password Policies für Admin-Konten
- [ ] Tiered Administration Model implementiert oder geplant
