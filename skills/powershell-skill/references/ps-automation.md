# PowerShell Automatisierung — Referenzmodul

Scheduled Tasks, Remoting, REST-APIs, E-Mail-Reports, Fehlerbehandlung, Logging-Framework.

---

## Enterprise Logging-Framework

```powershell
# ─── LOGGING-MODUL (in eigene .psm1 auslagern) ───────────────────────────────
enum LogLevel { DEBUG = 0; INFO = 1; WARNING = 2; ERROR = 3; CRITICAL = 4 }

function Write-Log {
    param(
        [Parameter(Mandatory)][string]$Message,
        [LogLevel]$Level = [LogLevel]::INFO,
        [string]$LogFile = $script:LogFile,
        [switch]$NoConsole
    )

    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss.fff"
    $caller    = (Get-PSCallStack)[1].Command
    $logEntry  = "[$timestamp] [$Level] [$caller] $Message"

    # Datei-Log
    if ($LogFile) {
        $logEntry | Out-File -FilePath $LogFile -Append -Encoding UTF8
    }

    # Konsole mit Farben
    if (-not $NoConsole) {
        $color = switch ($Level) {
            "DEBUG"    { "Gray" }
            "INFO"     { "White" }
            "WARNING"  { "Yellow" }
            "ERROR"    { "Red" }
            "CRITICAL" { "Magenta" }
        }
        Write-Host $logEntry -ForegroundColor $color
    }

    # Eventlog (für WARNING+)
    if ($Level -ge [LogLevel]::WARNING) {
        $eventType = if ($Level -ge [LogLevel]::ERROR) {"Error"} else {"Warning"}
        Write-EventLog -LogName Application -Source "PSAutomation" `
            -EventId 1000 -EntryType $eventType -Message $logEntry `
            -ErrorAction SilentlyContinue
    }
}

# Initialisierung in Hauptskript:
$script:LogFile = "C:\Logs\script-$(Get-Date -Format 'yyyyMMdd').log"

# Nutzung:
Write-Log "Skript gestartet"
Write-Log "Warnung: X Einträge übersprungen" -Level WARNING
Write-Log "Kritischer Fehler aufgetreten" -Level CRITICAL
```

---

## Fehlerbehandlung (Enterprise-Pattern)

```powershell
# ─── TRY/CATCH MIT DETAILLIERTEM FEHLERBERICHT ────────────────────────────────
function Invoke-WithErrorHandling {
    param(
        [Parameter(Mandatory)][scriptblock]$Action,
        [string]$Context = "Unbekannte Aktion",
        [switch]$ContinueOnError
    )
    try {
        & $Action
        return $true
    }
    catch [System.Net.WebException] {
        Write-Log "Netzwerkfehler in '$Context': $($_.Exception.Message)" -Level ERROR
        Write-Log "Status: $($_.Exception.Response?.StatusCode)" -Level ERROR
    }
    catch [System.UnauthorizedAccessException] {
        Write-Log "Zugriffsfehler in '$Context': $($_.Exception.Message)" -Level ERROR
        Write-Log "Pfad: $($_.TargetObject)" -Level ERROR
    }
    catch {
        Write-Log "Fehler in '$Context': $($_.Exception.Message)" -Level ERROR
        Write-Log "Typ: $($_.Exception.GetType().FullName)" -Level ERROR
        Write-Log "Stack: $($_.ScriptStackTrace)" -Level DEBUG
    }
    if (-not $ContinueOnError) { throw }
    return $false
}

# Nutzung:
Invoke-WithErrorHandling -Context "AD-Benutzer anlegen" -ContinueOnError -Action {
    New-ADUser -Name "Test" -SamAccountName "test.user" -Enabled $true
    Write-Log "Benutzer erfolgreich angelegt"
}

# ─── RETRY-MECHANISMUS ────────────────────────────────────────────────────────
function Invoke-WithRetry {
    param(
        [Parameter(Mandatory)][scriptblock]$Action,
        [int]$MaxAttempts = 3,
        [int]$DelaySeconds = 5,
        [string]$Context = "Aktion"
    )
    for ($i = 1; $i -le $MaxAttempts; $i++) {
        try {
            $result = & $Action
            Write-Log "Erfolgreich nach $i Versuch(en): $Context" -Level DEBUG
            return $result
        }
        catch {
            if ($i -eq $MaxAttempts) {
                Write-Log "Endgültig fehlgeschlagen nach $MaxAttempts Versuchen: $Context" -Level ERROR
                throw
            }
            Write-Log "Versuch $i/$MaxAttempts fehlgeschlagen, warte ${DelaySeconds}s: $Context" -Level WARNING
            Start-Sleep -Seconds $DelaySeconds
            $DelaySeconds *= 2  # Exponentielles Backoff
        }
    }
}

# Nutzung:
Invoke-WithRetry -Context "Exchange-Verbindung" -MaxAttempts 3 -DelaySeconds 10 -Action {
    Connect-ExchangeOnline -UserPrincipalName "admin@firma.de" -ShowBanner:$false
}
```

---

## REST API Integration

```powershell
# ─── GENERIC REST-CLIENT MIT AUTH ─────────────────────────────────────────────
function Invoke-RestApiCall {
    param(
        [Parameter(Mandatory)][string]$Url,
        [string]$Method = "GET",
        [hashtable]$Headers = @{},
        [object]$Body = $null,
        [string]$BearerToken = $null,
        [System.Management.Automation.PSCredential]$Credential = $null,
        [int]$TimeoutSec = 30
    )

    if ($BearerToken) {
        $Headers["Authorization"] = "Bearer $BearerToken"
    }
    $Headers["Content-Type"] = "application/json"
    $Headers["Accept"]       = "application/json"

    $params = @{
        Uri             = $Url
        Method          = $Method
        Headers         = $Headers
        TimeoutSec      = $TimeoutSec
        UseBasicParsing = $true
    }
    if ($Body)       { $params["Body"] = ($Body | ConvertTo-Json -Depth 10) }
    if ($Credential) { $params["Credential"] = $Credential }

    try {
        $response = Invoke-RestMethod @params
        return $response
    }
    catch {
        $statusCode = $_.Exception.Response?.StatusCode.value__
        $errorBody  = $_.ErrorDetails?.Message
        throw "API-Fehler $statusCode für $Url : $errorBody"
    }
}

# ─── SERVICENOW INTEGRATION ───────────────────────────────────────────────────
function New-ServiceNowIncident {
    param(
        [string]$Title,
        [string]$Description,
        [ValidateSet("1","2","3")][string]$Priority = "3",
        [string]$AssignmentGroup = "IT-Infrastruktur"
    )

    $body = @{
        short_description  = $Title
        description        = $Description
        priority           = $Priority
        assignment_group   = $AssignmentGroup
        caller_id          = "automation-service"
        category           = "Software"
    }

    $cred = Get-Credential -Message "ServiceNow API-Credentials"
    $result = Invoke-RestApiCall `
        -Url "https://firma.service-now.com/api/now/table/incident" `
        -Method POST -Body $body -Credential $cred

    Write-Log "ServiceNow Incident erstellt: $($result.result.number)"
    return $result.result.number
}

# ─── PAGERDUTY ALERT ──────────────────────────────────────────────────────────
function Send-PagerDutyAlert {
    param(
        [string]$Summary,
        [ValidateSet("critical","error","warning","info")][string]$Severity = "error",
        [hashtable]$Details = @{}
    )

    $body = @{
        routing_key  = $env:PAGERDUTY_ROUTING_KEY
        event_action = "trigger"
        payload      = @{
            summary        = $Summary
            severity       = $Severity
            source         = $env:COMPUTERNAME
            custom_details = $Details
        }
    }
    Invoke-RestApiCall -Url "https://events.pagerduty.com/v2/enqueue" `
        -Method POST -Body $body
}
```

---

## E-Mail-Reporting (HTML)

```powershell
function Send-HTMLReport {
    param(
        [Parameter(Mandatory)][string]$Subject,
        [Parameter(Mandatory)][string]$HtmlBody,
        [string[]]$To = @("it-ops@firma.de"),
        [string]$From = "automation@firma.de",
        [string]$SmtpServer = "mail.firma.de",
        [string[]]$Attachments = @()
    )

    $mailParams = @{
        SmtpServer  = $SmtpServer
        Port        = 587
        UseSsl      = $true
        Credential  = (Get-Credential -Message "SMTP-Zugangsdaten")
        From        = $From
        To          = $To
        Subject     = $Subject
        Body        = $HtmlBody
        BodyAsHtml  = $true
        Encoding    = [System.Text.Encoding]::UTF8
    }
    if ($Attachments) {
        $mailParams["Attachments"] = $Attachments
    }
    Send-MailMessage @mailParams
    Write-Log "E-Mail gesendet: $Subject → $($To -join ', ')"
}

# HTML-Tabelle aus Objekten generieren
function ConvertTo-HTMLTable {
    param(
        [Parameter(Mandatory, ValueFromPipeline)][object[]]$InputObject,
        [string]$TableClass = "report-table"
    )
    begin { $rows = @() }
    process { $rows += $InputObject }
    end {
        if ($rows.Count -eq 0) { return "<p>Keine Daten</p>" }
        $headers = $rows[0].PSObject.Properties.Name
        $html = "<table class='$TableClass'><thead><tr>"
        $html += ($headers | ForEach-Object { "<th>$_</th>" }) -join ""
        $html += "</tr></thead><tbody>"
        foreach ($row in $rows) {
            $html += "<tr>"
            foreach ($h in $headers) {
                $val = $row.$h
                $html += "<td>$([System.Web.HttpUtility]::HtmlEncode($val))</td>"
            }
            $html += "</tr>"
        }
        $html += "</tbody></table>"
        return $html
    }
}

# Vollständiger Report:
$staleUsersTable = Get-ADUser -Filter {LastLogonDate -lt (Get-Date).AddDays(-90) -and Enabled -eq $true} `
    -Properties LastLogonDate, Department |
    Select-Object Name, SamAccountName, LastLogonDate, Department |
    ConvertTo-HTMLTable

$body = @"
<!DOCTYPE html><html><head><meta charset="UTF-8">
<style>
  body{font-family:Arial,sans-serif;font-size:13px}
  h1{color:#003087} h2{color:#555;border-bottom:1px solid #ddd}
  .report-table{border-collapse:collapse;width:100%;margin:10px 0}
  .report-table th{background:#003087;color:white;padding:6px 10px;text-align:left}
  .report-table td{border:1px solid #ddd;padding:5px 10px}
  .report-table tr:nth-child(even){background:#f9f9f9}
</style></head><body>
<h1>IT-Security Weekly Report — $(Get-Date -Format "dd.MM.yyyy")</h1>
<h2>Inaktive Benutzerkonten (> 90 Tage)</h2>
$staleUsersTable
<p><small>Automatisch generiert am $(Get-Date -Format "dd.MM.yyyy HH:mm") von $env:COMPUTERNAME</small></p>
</body></html>
"@

Send-HTMLReport -Subject "IT-Security Report $(Get-Date -Format 'KW W, yyyy')" -HtmlBody $body
```

---

## PowerShell Remoting

```powershell
# ─── REMOTE-BEFEHLE ───────────────────────────────────────────────────────────
# Einzelner Server
Invoke-Command -ComputerName "srv-web01" -ScriptBlock {
    Get-Service | Where-Object {$_.Status -eq "Stopped" -and $_.StartType -eq "Automatic"}
}

# Mehrere Server parallel (ThrottleLimit = Gleichzeitige Verbindungen)
$servers = @("srv-web01", "srv-web02", "srv-db01", "srv-app01")
$results = Invoke-Command -ComputerName $servers -ThrottleLimit 10 -ScriptBlock {
    [PSCustomObject]@{
        Server     = $env:COMPUTERNAME
        CPU        = (Get-CimInstance Win32_Processor).LoadPercentage
        FreeRAMGB  = [math]::Round((Get-CimInstance Win32_OperatingSystem).FreePhysicalMemory/1MB, 2)
        DiskFreeGB = [math]::Round((Get-PSDrive C).Free/1GB, 2)
        Uptime     = (Get-Date) - (gcim Win32_OperatingSystem).LastBootUpTime
    }
}
$results | Sort-Object CPU -Descending | Format-Table -AutoSize

# ─── PERSISTENT SESSION ───────────────────────────────────────────────────────
$session = New-PSSession -ComputerName "srv-db01" -Credential (Get-Credential)
Invoke-Command -Session $session -ScriptBlock { Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 }
Copy-Item -Path "C:\scripts\update.sql" -Destination "C:\temp\" -ToSession $session
Remove-PSSession $session

# ─── WINRM KONFIGURATION (Ziel-Server) ───────────────────────────────────────
# Auf Ziel-Server ausführen:
Enable-PSRemoting -Force
Set-Item WSMan:\localhost\Service\Auth\Basic -Value $false      # Basic-Auth deaktivieren
Set-Item WSMan:\localhost\Service\Auth\Negotiate -Value $true   # Kerberos/NTLM
Set-Item WSMan:\localhost\Service\AllowUnencrypted -Value $false # Verschlüsselung erzwingen

# Nur bestimmten IPs erlauben:
Set-Item WSMan:\localhost\Service\IPv4Filter -Value "10.0.0.*,10.255.255.*"
```
