# Intune Endpoint Management — Referenzmodul

Enrollment, Configuration Profiles, Compliance, App Deployment, Autopilot, Remediation.

---

## Enrollment-Strategien

```
WINDOWS:
  Autopilot (empfohlen für Neugeräte):
    1. Hardware-Hash bei Lieferant registrieren lassen (Dell/HP/Lenovo OEM)
    2. Gerät einschalten → OOBE startet automatisch
    3. AAD Join + Intune-Enrollment passiert ohne IT-Einsatz
    4. Autopilot Profile: User-Driven (Standard) oder Self-Deploying (Kiosk)

  Hybrid AAD Join (bestehende Umgebung):
    - GPO: Computer Configuration → Administrative Templates → MDM
    - Geräte sind On-Premises AD UND Entra ID beigetreten
    - Co-Management möglich (MECM + Intune)

  AAD Join (Cloud-Native):
    - Einstellungen → Zugriff auf Arbeit oder Schule → Verbinden
    - Dann: MDM-Enrollment automatisch wenn Conditional Access es verlangt

MOBILE (iOS / Android):
  iOS: Apple DEP (Device Enrollment Program) via Apple Business Manager
  Android: Android Enterprise (Work Profile / Fully Managed / Dedicated)
  BYOD: Work Profile (persönliche Daten getrennt von Firmendaten)
```

---

## Configuration Profiles (PowerShell + Graph API)

```powershell
# Microsoft Graph PowerShell — Intune
Connect-MgGraph -Scopes "DeviceManagementConfiguration.ReadWrite.All",
                         "DeviceManagementManagedDevices.ReadWrite.All"

# ─── ALLE GERÄTE ABRUFEN ──────────────────────────────────────────────────────
$devices = Get-MgDeviceManagementManagedDevice -All |
    Select-Object DeviceName, OperatingSystem, OsVersion,
                  ComplianceState, LastSyncDateTime, UserPrincipalName,
                  ManagementState, JoinType

$devices | Sort-Object ComplianceState | Format-Table -AutoSize

# ─── NICHT-KONFORME GERÄTE ────────────────────────────────────────────────────
$noncompliant = Get-MgDeviceManagementManagedDevice -All |
    Where-Object {$_.ComplianceState -eq "noncompliant"}

$noncompliant | ForEach-Object {
    $reasons = Get-MgDeviceManagementManagedDeviceCompliancePolicyState `
        -ManagedDeviceId $_.Id
    [PSCustomObject]@{
        Gerät   = $_.DeviceName
        Benutzer = $_.UserPrincipalName
        Gründe  = ($reasons | Where-Object {$_.State -eq "nonCompliant"}).DisplayName -join "; "
    }
} | Format-Table -AutoSize

# ─── REMOTE-AKTIONEN ──────────────────────────────────────────────────────────
# Remote-Wipe (ACHTUNG: löscht alle Daten!)
$device = Get-MgDeviceManagementManagedDevice -Filter "deviceName eq 'LAPTOP-001'"
Invoke-MgWipeDeviceManagementManagedDevice -ManagedDeviceId $device.Id

# Sync erzwingen
Invoke-MgSyncDeviceManagementManagedDevice -ManagedDeviceId $device.Id

# Lock
Invoke-MgRemoteLockDeviceManagementManagedDevice -ManagedDeviceId $device.Id
```

---

## Windows Compliance Policy (JSON-Vorlage)

```json
{
  "displayName": "Windows 11 Compliance",
  "description": "Mindestanforderungen für Windows 11 Unternehmensgeräte",
  "@odata.type": "#microsoft.graph.windows10CompliancePolicy",
  "passwordRequired": true,
  "passwordMinimumLength": 12,
  "passwordRequiredType": "deviceDefault",
  "passwordMinutesOfInactivityBeforeLock": 5,
  "requireHealthyDeviceReport": true,
  "osMinimumVersion": "10.0.22000",
  "bitLockerEnabled": true,
  "secureBootEnabled": true,
  "codeIntegrityEnabled": true,
  "storageRequireEncryption": true,
  "activeFirewallRequired": true,
  "defenderEnabled": true,
  "defenderVersion": "",
  "signatureOutOfDate": false,
  "rtpEnabled": true,
  "antivirusRequired": true,
  "antiSpywareRequired": true,
  "scheduledActionsForRule": [
    {
      "ruleName": "Noncompliance",
      "scheduledActionConfigurations": [
        {
          "actionType": "notification",
          "gracePeriodHours": 0,
          "notificationTemplateId": ""
        },
        {
          "actionType": "block",
          "gracePeriodHours": 72
        }
      ]
    }
  ]
}
```

---

## Windows Autopilot

```powershell
# Hardware-Hash aus bestehendem Gerät exportieren
Install-Script -Name Get-WindowsAutoPilotInfo
Get-WindowsAutoPilotInfo -OutputFile C:\autopilot-hash.csv

# Bulk-Import (mehrere Geräte)
# Intune-Portal: Geräte → Windows → Registrierung → Geräte → Importieren

# Autopilot-Profil per Graph API erstellen
$profile = @{
    displayName            = "Autopilot-Profil-Standard"
    description            = "Standard-Profil für neue Windows-Geräte"
    deviceType             = "windowsPc"
    enableWhiteGlove       = $true   # Pre-provisioning (Techniker-Flow)
    extractHardwareHash    = $false
    language               = "de-DE"
    deviceNameTemplate     = "FI-%RAND:4%"   # z.B. FI-A3X9
    skipKeyboardSelectionPage = $true
    hidePrivacyPage        = $true
    hideEulaPage           = $true
    hideEscapeLink         = $true
    enrollmentStatusScreenSettings = @{
        hideInstallationProgress        = $false
        blockDeviceSetupRetryByUser     = $false
        allowDeviceResetOnInstallFailure = $true
        showInstallationProgressForFirstUser = $true
        blockDeviceSetupRetryByUser     = $true
        installProgressTimeoutInMinutes = 60
    }
}

# Deployment Profile über Graph API anlegen
New-MgDeviceManagementWindowsAutopilotDeploymentProfile -BodyParameter $profile
```

---

## App-Deployment

```powershell
# ─── WIN32-APP VERPACKEN ──────────────────────────────────────────────────────
# Microsoft Win32 Content Prep Tool
# Download: https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool
IntuneWinAppUtil.exe -c "C:\Apps\7zip" -s "7z2301-x64.exe" -o "C:\Output"
# → Erzeugt: 7z2301-x64.intunewin

# ─── INSTALL / UNINSTALL BEFEHLE ─────────────────────────────────────────────
# 7-Zip Beispiel:
# Install:   7z2301-x64.exe /S
# Uninstall: MsiExec.exe /X{23170F69-40C1-2702-2301-000001000000} /qn
# Detection: Datei existiert: C:\Program Files\7-Zip\7z.exe

# ─── POWERSHELL-REMEDIATION (Intune Remediations) ─────────────────────────────
# Detection-Skript (Exit 1 = Problem gefunden)
$setting = Get-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" `
    -Name NoAutoUpdate -ErrorAction SilentlyContinue
if ($setting.NoAutoUpdate -eq 1) { exit 1 } else { exit 0 }

# Remediation-Skript (behebt das Problem)
Set-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" `
    -Name NoAutoUpdate -Value 0 -Type DWord
```

---

## Intune Reporting (PowerShell)

```powershell
# ─── GERÄTEÜBERSICHT ──────────────────────────────────────────────────────────
function Get-IntuneDeviceReport {
    $devices = Get-MgDeviceManagementManagedDevice -All
    $report = $devices | Group-Object ComplianceState |
        Select-Object @{N='Status'; E={$_.Name}},
                      @{N='Anzahl'; E={$_.Count}}

    Write-Host "=== Intune Gerätestatus $(Get-Date -Format 'dd.MM.yyyy') ===" -ForegroundColor Cyan
    $report | Format-Table -AutoSize
    Write-Host "Gesamt: $($devices.Count) Geräte"

    # OS-Verteilung
    $devices | Group-Object OperatingSystem |
        Select-Object @{N='OS'; E={$_.Name}}, Count |
        Sort-Object Count -Descending | Format-Table
}

# ─── STALE DEVICES (> 30 Tage kein Sync) ──────────────────────────────────────
Get-MgDeviceManagementManagedDevice -All |
    Where-Object {$_.LastSyncDateTime -lt (Get-Date).AddDays(-30)} |
    Select-Object DeviceName, UserPrincipalName, LastSyncDateTime, ComplianceState |
    Sort-Object LastSyncDateTime |
    Export-Csv -Path "stale-devices.csv" -NoTypeInformation -Encoding UTF8

Disconnect-MgGraph
```
