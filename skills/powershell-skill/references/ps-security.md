# PowerShell Security — Referenzmodul

Sichere Skripting-Praktiken, Code-Signing, Logging, Constrained Language Mode, JEA.

---

## Sichere PowerShell-Grundlagen

```powershell
# ─── EXECUTION POLICY (Enterprise-Standard) ───────────────────────────────────
# AllSigned: Alle Skripte müssen signiert sein (empfohlen für Produktion)
Set-ExecutionPolicy AllSigned -Scope LocalMachine -Force

# RemoteSigned: Lokale Skripte unsigniert OK, Remote-Skripte müssen signiert sein
Set-ExecutionPolicy RemoteSigned -Scope LocalMachine -Force

# Execution Policy per GPO erzwingen (verhindert lokale Änderungen)
# Computer Configuration → Administrative Templates → Windows Components
# → Windows PowerShell → Turn on Script Execution → AllSigned

# ─── SECURE STRING (Passwörter nie als PlainText) ─────────────────────────────
# Schlecht:
$password = "GeheimesPasswort123!"

# Gut: SecureString im Skript
$password = Read-Host -Prompt "Passwort" -AsSecureString

# Gut: Aus verschlüsselter Datei (nur auf dem Schlüssel-PC entschlüsselbar)
$securePassword = Get-Content "C:\secure\password.enc" | ConvertTo-SecureString

# Besser: Aus Secrets Manager (Azure Key Vault)
$secret = Get-AzKeyVaultSecret -VaultName "kv-firma-prod" -Name "db-password"
$securePassword = $secret.SecretValue

# ─── CREDENTIAL-OBJEKT ────────────────────────────────────────────────────────
$cred = Get-Credential  # Interaktiv (für ineractive scripts)
$cred = New-Object System.Management.Automation.PSCredential(
    "FIRMA\svc-backup",
    (ConvertTo-SecureString "Passwort" -AsPlainText -Force)
)
# Hinweis: -AsPlainText nur in Notfällen — besser aus Key Vault

# ─── INPUT-VALIDIERUNG (SQL Injection / Command Injection verhindern) ──────────
function Invoke-SafeQuery {
    param(
        [Parameter(Mandatory)]
        [ValidatePattern('^[a-zA-Z0-9_-]{1,50}$')]  # Whitelist-Regex
        [string]$TableName,

        [Parameter(Mandatory)]
        [ValidateRange(1, 1000000)]
        [int]$UserId
    )
    # Parametrisierte Abfrage (nie String-Interpolation für SQL!)
    $query = "SELECT * FROM [$TableName] WHERE UserID = @UserID"
    $cmd = $conn.CreateCommand()
    $cmd.CommandText = $query
    $cmd.Parameters.AddWithValue("@UserID", $UserId) | Out-Null
    $cmd.ExecuteReader()
}
```

---

## PowerShell Logging (Enterprise)

```powershell
# ─── SCRIPT BLOCK LOGGING AKTIVIEREN (GPO empfohlen) ─────────────────────────
# Loggt alle ausgeführten PowerShell-Code-Blöcke → Event 4104
# GPO: Computer Configuration → Administrative Templates → Windows Components
#      → Windows PowerShell → Turn on PowerShell Script Block Logging

# Via Registry (für schnelle Aktivierung):
$regPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
New-Item -Path $regPath -Force | Out-Null
Set-ItemProperty -Path $regPath -Name "EnableScriptBlockLogging" -Value 1
Set-ItemProperty -Path $regPath -Name "EnableScriptBlockInvocationLogging" -Value 1

# ─── MODULE LOGGING ───────────────────────────────────────────────────────────
$modPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging"
New-Item -Path $modPath -Force | Out-Null
Set-ItemProperty -Path $modPath -Name "EnableModuleLogging" -Value 1
New-Item -Path "$modPath\ModuleNames" -Force | Out-Null
Set-ItemProperty -Path "$modPath\ModuleNames" -Name "*" -Value "*"  # Alle Module

# ─── TRANSCRIPTION LOGGING ────────────────────────────────────────────────────
$transcriptPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription"
New-Item -Path $transcriptPath -Force | Out-Null
Set-ItemProperty -Path $transcriptPath -Name "EnableTranscripting" -Value 1
Set-ItemProperty -Path $transcriptPath -Name "EnableInvocationHeader" -Value 1
Set-ItemProperty -Path $transcriptPath -Name "OutputDirectory" -Value "\\fileserver\PSLogs\$env:COMPUTERNAME"

# ─── LOGS AUSWERTEN (SIEM-Vorbereitung) ──────────────────────────────────────
# Alle Script-Block-Logs des letzten Tages
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 1000 |
    Where-Object {$_.Id -eq 4104 -and $_.TimeCreated -gt (Get-Date).AddDays(-1)} |
    ForEach-Object {
        [PSCustomObject]@{
            Zeit    = $_.TimeCreated
            Benutzer = $_.Properties[5].Value
            Skript  = $_.Properties[0].Value.Substring(0, [Math]::Min(200, $_.Properties[0].Value.Length))
        }
    } | Format-Table -AutoSize

# Verdächtige Befehle suchen (rudimentäre Erkennung)
$suspiciousPatterns = @(
    'IEX', 'Invoke-Expression', 'DownloadString', 'WebClient',
    'FromBase64String', '-enc', '-EncodedCommand', 'bypass',
    'certutil', 'bitsadmin', 'Start-BitsTransfer'
)
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" |
    Where-Object {$_.Id -eq 4104} |
    Where-Object {
        $content = $_.Properties[0].Value
        $suspiciousPatterns | Where-Object {$content -ilike "*$_*"}
    } | Select-Object TimeCreated, @{N='Code'; E={$_.Properties[0].Value}} |
    Export-Csv "suspicious-ps-activity.csv" -Encoding UTF8 -NoTypeInformation
```

---

## Code-Signing

```powershell
# ─── SELF-SIGNED ZERTIFIKAT (NUR FÜR TESTS!) ─────────────────────────────────
$cert = New-SelfSignedCertificate `
    -DnsName "ps-signing.firma.local" `
    -CertStoreLocation "cert:\CurrentUser\My" `
    -Type CodeSigningCert `
    -KeyUsage DigitalSignature `
    -KeyLength 4096 `
    -HashAlgorithm SHA256

# ─── SKRIPT SIGNIEREN ─────────────────────────────────────────────────────────
# Mit Self-Signed (nur für Test/Dev):
Set-AuthenticodeSignature -FilePath "C:\Scripts\deploy.ps1" `
    -Certificate $cert `
    -HashAlgorithm SHA256

# Mit Enterprise-Zertifikat (aus PKI / ADCS):
$entCert = Get-ChildItem -Path "Cert:\CurrentUser\My" -CodeSigningCert |
    Where-Object {$_.Issuer -like "*Firma-CA*"} |
    Sort-Object NotAfter -Descending | Select-Object -First 1

Set-AuthenticodeSignature -FilePath "C:\Scripts\deploy.ps1" `
    -Certificate $entCert `
    -TimestampServer "http://timestamp.digicert.com" `
    -HashAlgorithm SHA256

# ─── SIGNATUR PRÜFEN ──────────────────────────────────────────────────────────
Get-AuthenticodeSignature -FilePath "C:\Scripts\deploy.ps1"
# Status: Valid = OK, NotSigned/Invalid = Problem!

# Alle Skripte in einem Verzeichnis prüfen
Get-ChildItem "C:\Scripts\" -Filter "*.ps1" -Recurse |
    Get-AuthenticodeSignature |
    Where-Object {$_.Status -ne "Valid"} |
    Select-Object Path, Status, StatusMessage
```

---

## Just Enough Administration (JEA)

```powershell
# JEA: Eingeschränkte PowerShell-Remoting-Endpunkte
# Ziel: Helpdesk darf nur spezifische Befehle ausführen (z.B. Passwort zurücksetzen)

# ─── ROLE CAPABILITY DATEI (Was darf die Rolle?) ─────────────────────────────
# Pfad: C:\Program Files\WindowsPowerShell\Modules\JEA\RoleCapabilities\HelpDesk.psrc

$rcParams = @{
    Author              = "IT-Security"
    CompanyName         = "Firma GmbH"
    Description         = "Helpdesk-Rolle: Nur Passwort-Reset und Account-Unlock"
    ModulesToImport     = @("ActiveDirectory")
    VisibleCmdlets      = @(
        "Set-ADAccountPassword",
        "Unlock-ADAccount",
        @{Name = "Get-ADUser"; Parameters = @{Name = "Identity"; ValidateSet = $null}},
        "Get-Help",
        "Exit-PSSession"
    )
    VisibleFunctions    = @("Reset-UserPassword")   # Eigene Hilfsfunktion
    VisibleExternalCommands = @()                    # Keine externen Befehle
    VisibleProviders    = @()                        # Kein Filesystem-Zugriff
    FunctionDefinitions = @{
        Name        = "Reset-UserPassword"
        ScriptBlock = {
            param([Parameter(Mandatory)][string]$Username)
            # Validierung
            if ($Username -notmatch '^[a-zA-Z0-9._-]{1,20}$') {
                throw "Ungültiger Benutzername"
            }
            $newPassword = ConvertTo-SecureString (New-Guid).Guid -AsPlainText -Force
            Set-ADAccountPassword -Identity $Username -NewPassword $newPassword -Reset
            Unlock-ADAccount -Identity $Username
            Write-Output "Passwort zurückgesetzt und Account entsperrt: $Username"
        }
    }
}
New-PSRoleCapabilityFile -Path "C:\PSRoleCapabilities\HelpDesk.psrc" @rcParams

# ─── SESSION CONFIGURATION (Wer darf den Endpunkt nutzen?) ───────────────────
# Pfad: C:\PSSessionConfigurations\JEA-HelpDesk.pssc

$sessionParams = @{
    Author              = "IT-Security"
    SessionType         = "RestrictedRemoteServer"
    Description         = "JEA Endpunkt für Helpdesk"
    RunAsVirtualAccount = $true          # Als lokales virtuelles Konto (kein echter Admin!)
    RoleDefinitions     = @{
        "FIRMA\Helpdesk-Gruppe" = @{RoleCapabilities = "HelpDesk"}
    }
    TranscriptDirectory = "\\fileserver\JEA-Logs"
    LanguageMode        = "NoLanguage"   # Kein vollständiges PowerShell
}
New-PSSessionConfigurationFile -Path "C:\PSSessionConfigurations\JEA-HelpDesk.pssc" @sessionParams

# Endpunkt registrieren
Register-PSSessionConfiguration -Name "JEA-HelpDesk" `
    -Path "C:\PSSessionConfigurations\JEA-HelpDesk.pssc" `
    -Force

# ─── NUTZUNG (Helpdesk-Mitarbeiter) ──────────────────────────────────────────
Enter-PSSession -ComputerName dc01 -ConfigurationName "JEA-HelpDesk"
# Nur erlaubte Befehle verfügbar
Reset-UserPassword -Username "max.mustermann"
```

---

## Constrained Language Mode (CLM)

```powershell
# CLM aktivieren (z.B. über AppLocker oder WDAC)
# AppLocker erzwingt CLM automatisch für nicht-signierte Skripte

# CLM prüfen
$ExecutionContext.SessionState.LanguageMode
# FullLanguage = kein CLM aktiv
# ConstrainedLanguage = CLM aktiv (Add-Type blockiert, COM-Objekte eingeschränkt)

# Bypass-Versuche erkennen (für Blue Team):
# ─ PowerShell 2.0 Downgrade: powershell -version 2
#   → Mitigierung: PS2 deaktivieren via Windows Feature
#   Remove-WindowsFeature PowerShell-v2

# ─ EncodedCommand Obfuskation:
#   powershell -enc <base64>
#   → Erkennung via Script Block Logging (Event 4104)

# ─── APPLOCKER FÜR POWERSHELL ────────────────────────────────────────────────
# GPO: Computer Configuration → Windows Settings → Security Settings → Application Control
# → AppLocker → Script Rules

# Nur signierte Skripte aus C:\Scripts\ erlauben:
# Condition: Publisher → signiert von Firma-CA
# Oder: Path → %PROGRAMFILES%\*, %WINDIR%\*
# Block: alle anderen Pfade (erzwingt automatisch CLM für nicht-erlaubte!)
```
