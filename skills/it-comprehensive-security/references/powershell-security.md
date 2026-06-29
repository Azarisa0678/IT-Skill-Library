# PowerShell Security Reference

## Logging & Visibility

### Script Block Logging (GPO)
Logs all PowerShell code executed, including obfuscated/decoded content.
```
HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
  EnableScriptBlockLogging = 1
  EnableScriptBlockInvocationLogging = 1  # Log start/stop of each block
```
Events land in: `Microsoft-Windows-PowerShell/Operational` (Event ID 4104)

### Module Logging
Logs pipeline execution details per module.
```
HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging
  EnableModuleLogging = 1
  ModuleNames = * (all modules)
```
Event ID: 4103

### Transcription Logging
Saves full session transcripts to a file.
```
HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription
  EnableTranscripting = 1
  OutputDirectory = \\fileserver\pslogs\
  EnableInvocationHeader = 1
```

### Over-the-Shoulder Transcription (PowerShell 5+)
```powershell
Start-Transcript -Path C:\Logs\PS_$(Get-Date -f yyyyMMdd_HHmmss).txt -Append
```

---

## Constrained Language Mode (CLM)

CLM restricts PowerShell to a safe subset — blocks .NET type access, COM objects, and many reflection techniques.

### Check current mode
```powershell
$ExecutionContext.SessionState.LanguageMode
# FullLanguage | ConstrainedLanguage | RestrictedLanguage | NoLanguage
```

### Enforce via AppLocker or WDAC
CLM is automatically enforced when AppLocker or Windows Defender Application Control (WDAC) is active with a code integrity policy. This is the recommended enterprise approach.

### Manually set (testing only)
```powershell
$ExecutionContext.SessionState.LanguageMode = "ConstrainedLanguage"
```

---

## AMSI (Antimalware Scan Interface)

AMSI allows antivirus engines to scan PowerShell scripts, WMI, Office macros, and more at runtime.

- Enabled by default in PowerShell 5+
- AV vendors hook into AMSI to scan script content before execution
- Bypass attempts (patching AMSI in memory) are a common malware technique and a detection signal

### Monitoring AMSI blocks
Event ID 1116 (Windows Defender) in `Microsoft-Windows-Windows Defender/Operational`

---

## Execution Policy Reference

| Policy | Behavior |
|--------|----------|
| `Restricted` | No scripts allowed (default on workstations) |
| `AllSigned` | Only signed scripts |
| `RemoteSigned` | Local scripts run; remote must be signed |
| `Unrestricted` | All scripts run (with warning for remote) |
| `Bypass` | Nothing blocked, no warnings |

> Note: Execution Policy is NOT a security boundary — it can be bypassed with `-ExecutionPolicy Bypass`. Use AppLocker/WDAC for real enforcement.

---

## Key Security Modules

| Module | Purpose | Install |
|--------|---------|---------|
| `ActiveDirectory` | AD user/group management | RSAT |
| `Microsoft.Graph` | Entra ID / M365 | `Install-Module Microsoft.Graph` |
| `Az` | Azure resource management | `Install-Module Az` |
| `PSScriptAnalyzer` | Static code analysis | `Install-Module PSScriptAnalyzer` |
| `Pester` | Testing framework | Built-in / `Install-Module Pester` |
| `PowerSploit` | Pentest (authorized use) | GitHub |
| `PowerShell Empire` | Red team C2 (authorized use) | GitHub |

---

## PSScriptAnalyzer — Security Rules

Run static analysis on scripts before deployment:
```powershell
Install-Module PSScriptAnalyzer -Force
Invoke-ScriptAnalyzer -Path .\MyScript.ps1 -Severity Warning,Error

# Include security-specific rules
Invoke-ScriptAnalyzer -Path .\MyScript.ps1 -Settings @{
    Rules = @{
        PSAvoidUsingPlainTextForPassword = @{ Enable = $true }
        PSAvoidUsingConvertToSecureStringWithPlainText = @{ Enable = $true }
        PSAvoidUsingInvokeExpression = @{ Enable = $true }
    }
}
```

---

## Credential Handling Best Practices

```powershell
# WRONG — plaintext password
$pass = "SuperSecret123"
$cred = New-Object PSCredential("user", $pass)

# RIGHT — SecureString from prompt
$cred = Get-Credential

# RIGHT — SecureString from encrypted file (per-user, per-machine)
$securePass = Read-Host -AsSecureString | ConvertFrom-SecureString
$securePass | Out-File C:\secure\pass.enc

# Later:
$securePass = Get-Content C:\secure\pass.enc | ConvertTo-SecureString
$cred = New-Object PSCredential("user", $securePass)

# RIGHT — from secrets vault (recommended for automation)
# Install-Module Microsoft.PowerShell.SecretManagement
# Install-Module Microsoft.PowerShell.SecretStore
Register-SecretVault -Name LocalStore -ModuleName Microsoft.PowerShell.SecretStore
Get-Secret -Name "SvcAccountPassword"
```

---

## Common Attack Techniques & Detections

| Technique | MITRE | Detection Signal |
|-----------|-------|-----------------|
| Encoded command (`-enc`) | T1059.001 | Event 4104 with base64 in CommandLine |
| AMSI bypass | T1562.001 | Patch of `amsi.dll` in memory; Event 1116 |
| Download cradle (`IEX (iwr ...)`) | T1105 | Network + Script Block Logging |
| Credential dumping via LSASS | T1003.001 | Event 4656/4663 on lsass.exe; Defender alert |
| PowerShell remoting lateral movement | T1021.006 | Event 4624 type 3 + WinRM events |
| Living off the land (LOLBins) | T1218 | Unusual parent-child process relationships |
| Scheduled task via PS | T1053.005 | Event 4698 (task created) |
