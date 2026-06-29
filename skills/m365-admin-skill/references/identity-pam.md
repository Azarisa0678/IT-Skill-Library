# Identity & Privileged Access Management — Referenzmodul

CyberArk, BeyondTrust, Entra PIM, LAPS, Just-in-Time Access, Credential Vaulting,
Session Recording, Least Privilege — für DACH-Umgebungen mit Microsoft-Stack.

---

## Entra ID PIM (Privileged Identity Management)

### PIM-Grundkonzepte

```
ROLLEN-TYPEN in PIM:
  Eligible (Berechtigt):  Benutzer kann Rolle anfordern, ist aber NICHT aktiv
  Active (Aktiv):         Rolle ist dauerhaft zugewiesen (vermeiden!)
  Time-bound:             Rolle nur für definierten Zeitraum aktiv

AKTIVIERUNGSFLOW:
  1. Benutzer meldet sich an PIM an
  2. Begründung + MFA-Bestätigung
  3. Optionale Genehmigung durch Approver
  4. Rolle aktiv für max. 8h (konfigurierbar)
  5. Automatische Deaktivierung + Audit-Log-Eintrag

EMPFEHLUNG:
  Global Admin:       Eligible, max. 2 Break-Glass-Konten Active
  Exchange Admin:     Eligible, Genehmigungspflichtig
  Security Admin:     Eligible, Selbstaktivierung mit Begründung
  User Admin:         Eligible, Selbstaktivierung
  Helpdesk Admin:     Active (tägliche Nutzung)
```

### PIM via PowerShell (Graph)

```powershell
Connect-MgGraph -Scopes "RoleManagement.ReadWrite.Directory",
                         "PrivilegedAccess.ReadWrite.AzureAD"

# ─── AKTIVE ROLE ASSIGNMENTS PRÜFEN ──────────────────────────────────────────
# Wer hat dauerhaft Global Admin? (sollte fast leer sein)
$globalAdminRole = Get-MgDirectoryRole | Where-Object {$_.DisplayName -eq "Global Administrator"}
Get-MgDirectoryRoleMember -DirectoryRoleId $globalAdminRole.Id |
    Select-Object Id, @{N='DisplayName'; E={(Get-MgUser -UserId $_.Id).DisplayName}},
                      @{N='UPN';         E={(Get-MgUser -UserId $_.Id).UserPrincipalName}}

# ─── ELIGIBLE ASSIGNMENTS SETZEN ─────────────────────────────────────────────
# Benutzer als "Eligible" für Exchange Admin konfigurieren
$roleDefinition = Get-MgRoleManagementDirectoryRoleDefinition |
    Where-Object {$_.DisplayName -eq "Exchange Administrator"}

$params = @{
    Action           = "adminAssign"
    Justification    = "IT-Admin benötigt Exchange-Zugang für Routineaufgaben"
    RoleDefinitionId = $roleDefinition.Id
    PrincipalId      = (Get-MgUser -UserId "it-admin@firma.de").Id
    DirectoryScopeId = "/"
    ScheduleInfo     = @{
        StartDateTime = Get-Date
        Expiration    = @{
            Type        = "AfterDuration"
            Duration    = "P365D"    # 1 Jahr Eligible-Zuweisung
        }
    }
}
New-MgRoleManagementDirectoryRoleEligibilityScheduleRequest -BodyParameter $params

# ─── PIM-AKTIVIERUNGEN AUSWERTEN ─────────────────────────────────────────────
# Alle Aktivierungen der letzten 7 Tage
Get-MgAuditLogDirectoryAudit -Filter "activityDisplayName eq 'Add member to role completed (PIM activation)' and activityDateTime ge $((Get-Date).AddDays(-7).ToString('yyyy-MM-dd'))" |
    Select-Object ActivityDateTime,
        @{N='Benutzer';     E={$_.InitiatedBy.User.UserPrincipalName}},
        @{N='Rolle';        E={($_.TargetResources | Where-Object {$_.Type -eq "Role"}).DisplayName}},
        @{N='Begründung';   E={$_.AdditionalDetails | Where-Object {$_.Key -eq "justification"} | Select-Object -ExpandProperty Value}} |
    Format-Table -AutoSize

# ─── PIM ALERT-KONFIGURATION ─────────────────────────────────────────────────
# Empfohlene PIM-Alerts aktivieren (im Azure Portal):
# 1. "Rollen ohne MFA-Anforderung" → Sofort beheben
# 2. "Zu viele globale Administratoren" → Max. 5
# 3. "Veraltete privilegierte Konten" → > 180 Tage inaktiv
# 4. "Rollen werden direkt zugewiesen" → Statt über PIM
```

### PIM-Einstellungen (Rollenrichtlinien)

```powershell
# Rollenrichtlinie für Global Admin abrufen und verschärfen
$policy = Get-MgPolicyRoleManagementPolicy -Filter "scopeId eq '/' and scopeType eq 'DirectoryRole'"
# Im Portal konfigurieren: Entra ID → PIM → Azure AD Rollen → Einstellungen

# Empfohlene Einstellungen für Global Admin:
<#
Aktivierungsdauer:        2 Stunden (nicht 8h)
MFA bei Aktivierung:      Ja (Pflicht)
Begründung:               Ja (Pflicht)
Ticket-Nummer:            Ja (empfohlen)
Genehmigung erforderlich: Ja (mind. 2 Genehmiger)
Genehmiger:               Sicherheitsteam-Gruppe
Benachrichtigungen:       Alle Aktivierungen → Security-Postfach
#>
```

---

## LAPS (Local Administrator Password Solution)

### Windows LAPS (integriert ab Windows Server 2022 / Win 11 22H2)

```powershell
# ─── LAPS VIA ENTRA ID (Cloud-native) ────────────────────────────────────────
# Voraussetzung: Intune, AAD-joined oder Hybrid-joined Geräte

# LAPS in Intune aktivieren:
# Endpoint Security → Account Protection → Local Admin Password Solution (Windows LAPS)

# Richtlinieneinstellungen:
<#
Backup-Ziel:                 Azure Active Directory
Passwortlänge:               14 Zeichen
Passwort-Komplexität:        Groß+Klein+Zahlen+Sonderzeichen
Passwort-Rotation:           Alle 30 Tage
Post-Authentifizierungs-Aktion: Passwort zurücksetzen + Abmelden (nach 24h)
#>

# LAPS-Passwort abrufen (Graph)
Connect-MgGraph -Scopes "DeviceLocalCredential.Read.All"

$device = Get-MgDevice -Filter "displayName eq 'LAPTOP-001'"
$cred = Get-MgDeviceLocalCredential -DeviceId $device.Id `
    -Property "credentials,refreshDateTime"

# Passwort anzeigen (Base64-dekodiert)
$password = [System.Text.Encoding]::UTF8.GetString(
    [System.Convert]::FromBase64String($cred.Credentials[0].PasswordBase64)
)
Write-Host "LAPS-Passwort für LAPTOP-001: $password" -ForegroundColor Yellow
Write-Host "Letzte Rotation: $($cred.RefreshDateTime)"

# ─── LAPS VIA ACTIVE DIRECTORY (on-premises) ─────────────────────────────────
# Schema-Erweiterung
Update-LapsADSchema

# OU-Berechtigung setzen (Computer dürfen ihr eigenes Passwort schreiben)
Set-LapsADComputerSelfPermission -Identity "OU=Workstations,DC=firma,DC=local"

# GPO: Computer Configuration → Administrative Templates → LAPS
<#
Passwort-Komplexität aktivieren:   Ja
Passwortlänge:                     14
Passwort-Alter (Tage):             30
Verzeichnisattribute verschlüsseln: Ja (Windows LAPS 2023+)
#>

# Passwort abfragen (on-prem)
Get-LapsADPassword -Identity "LAPTOP-001" -AsPlainText
```

---

## CyberArk PAM

### Architektur-Überblick

```
CyberArk-Komponenten:

  Digital Vault (EPV):      Zentraler, gehärteter Tresor für Credentials
  PVWA (Web Interface):     Admin-UI für Vault-Verwaltung, Session-Start
  CPM (Password Manager):   Automatische Passwort-Rotation (timed/on-use)
  PSM (Session Manager):    RDP/SSH-Proxy + Session Recording
  PSMP (SSH Proxy):         Linux/Unix SSH-Sessions via PSM
  PTA (Threat Analytics):   Anomalie-Erkennung bei privilegierten Zugriffen
  AAM (App Access Mgr):     Credentials für Anwendungen (kein Hardcoding)
  Conjur (Cloud-native):    Secrets für DevOps/Kubernetes (CyberArk-Produkt)

TYPISCHER ZUGANGSFLOW:
  1. Admin öffnet PVWA im Browser
  2. Authentifizierung: LDAP + MFA (RADIUS/TOTP)
  3. Account aus Safe auswählen (z.B. "srv-web01-local-admin")
  4. PSM öffnet RDP-Session (Transparent: Admin sieht Passwort nie)
  5. Session wird aufgezeichnet (Video + Keystroke-Log)
  6. Nach Session: CPM rotiert Passwort automatisch
```

### CyberArk REST API (PowerShell)

```powershell
$pvwaUrl = "https://cyberark.firma.de"

# ─── AUTHENTIFIZIERUNG ────────────────────────────────────────────────────────
$authBody = @{
    username = "api-service-account"
    password = $env:CYBERARK_PASSWORD
} | ConvertTo-Json

$session = Invoke-RestMethod -Uri "$pvwaUrl/PasswordVault/API/auth/LDAP/Logon" `
    -Method POST -Body $authBody -ContentType "application/json"
$token = $session  # Bearer Token

$headers = @{ Authorization = $token }

# ─── ACCOUNTS SUCHEN ──────────────────────────────────────────────────────────
$accounts = Invoke-RestMethod -Uri "$pvwaUrl/PasswordVault/API/Accounts?search=srv-web&limit=100" `
    -Headers $headers -Method GET

$accounts.value | Select-Object name, address, userName, platformId, safeName |
    Format-Table -AutoSize

# ─── PASSWORT ABRUFEN (mit Begründung / Ticket) ───────────────────────────────
$accountId = $accounts.value[0].id

$retrieveBody = @{
    reason  = "Notfall-Wartung JIRA-4711"
    TicketingSystemName = "Jira"
    TicketId = "OPS-4711"
} | ConvertTo-Json

$password = Invoke-RestMethod `
    -Uri "$pvwaUrl/PasswordVault/API/Accounts/$accountId/Password/Retrieve" `
    -Headers $headers -Method POST -Body $retrieveBody -ContentType "application/json"

# ─── PASSWORT SOFORT ROTIEREN ─────────────────────────────────────────────────
Invoke-RestMethod -Uri "$pvwaUrl/PasswordVault/API/Accounts/$accountId/Change" `
    -Headers $headers -Method POST

# ─── SESSION VERIFIZIEREN (PSM) ───────────────────────────────────────────────
$sessions = Invoke-RestMethod -Uri "$pvwaUrl/PasswordVault/API/LiveSessions" `
    -Headers $headers -Method GET
$sessions.LiveSessions | Select-Object SessionID, User, RemoteAddress, StartTime, Status

# ─── LOGOUT ──────────────────────────────────────────────────────────────────
Invoke-RestMethod -Uri "$pvwaUrl/PasswordVault/API/Auth/Logoff" `
    -Headers $headers -Method POST
```

---

## BeyondTrust PAM

### Password Safe (PowerShell + REST)

```powershell
$btUrl = "https://beyondtrust.firma.de/BeyondTrust/api/public/v3"

# ─── AUTHENTIFIZIERUNG ────────────────────────────────────────────────────────
$authHeader = [Convert]::ToBase64String(
    [Text.Encoding]::ASCII.GetBytes("$($env:BT_USER):$($env:BT_PASSWORD)")
)

$signinBody = @{ ApplicationID = "PowerShell-Script" } | ConvertTo-Json
$session = Invoke-RestMethod -Uri "$btUrl/Auth/SignAppin" `
    -Method POST -Body $signinBody -ContentType "application/json" `
    -Headers @{ Authorization = "PS-Auth key=$($env:BT_APIKEY); runas=$($env:BT_USER);" }

$headers = @{ "Authorization" = "Bearer $($session.Token)" }

# ─── MANAGED ACCOUNTS ABRUFEN ─────────────────────────────────────────────────
$accounts = Invoke-RestMethod -Uri "$btUrl/ManagedAccounts?systemName=srv-web01" `
    -Headers $headers

# ─── REQUEST CREDENTIAL (Checkout) ───────────────────────────────────────────
$requestBody = @{
    SystemID  = $accounts[0].ManagedSystemID
    AccountID = $accounts[0].ManagedAccountID
    DurationMinutes = 60
    Reason    = "Patch-Deployment CHANGE-001"
    ConflictOption = "reuse"
} | ConvertTo-Json

$request = Invoke-RestMethod -Uri "$btUrl/Requests" `
    -Method POST -Body $requestBody -ContentType "application/json" `
    -Headers $headers

# Passwort abrufen
$cred = Invoke-RestMethod -Uri "$btUrl/Credentials/$($request.RequestID)" `
    -Headers $headers

Write-Host "Passwort für srv-web01: $($cred.Password)"

# ─── CHECK-IN (Credential zurückgeben) ───────────────────────────────────────
Invoke-RestMethod -Uri "$btUrl/Requests/$($request.RequestID)/Checkin" `
    -Method PUT -Headers $headers -ContentType "application/json" `
    -Body '"Patch erfolgreich abgeschlossen"'

# Abmelden
Invoke-RestMethod -Uri "$btUrl/Auth/Signout" -Method POST -Headers $headers
```

---

## Just-in-Time (JIT) Access Patterns

```
PATTERN 1: Entra PIM (Cloud-Ressourcen)
  → Eligible Assignment + Aktivierung auf Anfrage
  → Azure-Rollen und Entra ID-Rollen
  → Integriert in Microsoft 365 / Azure

PATTERN 2: CyberArk / BeyondTrust (on-premises + hybrid)
  → Credential-Checkout: Passwort nur für Session-Dauer sichtbar
  → PSM/Proxy: Admin sieht Passwort nie (Transparent Login)
  → Automatische Rotation nach Session

PATTERN 3: Azure Bastion + JIT VM Access (Azure VMs)
  → VM-Ports standardmäßig geschlossen
  → JIT-Anfrage: Port 3389/22 für IP + Zeitraum öffnen
  → Kein öffentliches RDP/SSH offen

PATTERN 4: Ephemeral Credentials (DevOps)
  → OIDC statt statischer Secrets (GitHub Actions, GitLab)
  → Azure Managed Identity (kein Passwort überhaupt)
  → AWS IAM Roles Anywhere
```

### Azure JIT VM Access (PowerShell)

```powershell
# JIT-Policy für VM aktivieren (via Defender for Cloud)
$vm = Get-AzVM -ResourceGroupName "rg-prod" -Name "vm-db01"

$jitPolicy = @{
    kind = "Basic"
    properties = @{
        virtualMachines = @(
            @{
                id    = $vm.Id
                ports = @(
                    @{
                        number           = 3389
                        protocol         = "TCP"
                        allowedSourceAddressPrefix = "*"
                        maxRequestAccessDuration   = "PT3H"   # Max. 3 Stunden
                    },
                    @{
                        number           = 22
                        protocol         = "TCP"
                        allowedSourceAddressPrefix = "*"
                        maxRequestAccessDuration   = "PT3H"
                    }
                )
            }
        )
    }
}

Set-AzJitNetworkAccessPolicy -ResourceGroupName "rg-prod" `
    -Location $vm.Location -Name "JIT-vm-db01" -Kind "Basic" `
    -VirtualMachine $jitPolicy.properties.virtualMachines

# JIT-Zugang anfordern (für eigene IP, 1 Stunde)
$myIp = (Invoke-RestMethod -Uri "https://api.ipify.org").Trim()

$request = @{
    virtualMachines = @(
        @{
            id    = $vm.Id
            ports = @(
                @{
                    number                    = 3389
                    allowedSourceAddressPrefix = $myIp
                    endTimeUtc                = (Get-Date).AddHours(1).ToUniversalTime().ToString("o")
                }
            )
        }
    )
}

Start-AzJitNetworkAccessPolicy -ResourceGroupName "rg-prod" `
    -Location $vm.Location -Name "JIT-vm-db01" -VirtualMachine $request.virtualMachines
Write-Host "JIT-Zugang für $myIp → vm-db01:3389 aktiv für 1 Stunde"
```

---

## PAM-Compliance-Checkliste (DACH)

```markdown
## PAM Compliance-Checkliste — [Unternehmen] — [Datum]

### Identifizierung privilegierter Konten
- [ ] Inventar aller privilegierten Konten vorhanden (AD, lokale Admins, Service Accounts, Cloud)
- [ ] Klassifizierung: Tier 0 (DC/PKI), Tier 1 (Server), Tier 2 (Workstations)
- [ ] Shared Accounts (z.B. "Administrator") eliminiert oder im PAM verwaltet
- [ ] Service Accounts: Minimale Rechte, kein interaktiver Login möglich

### PIM / Just-in-Time
- [ ] Entra PIM aktiviert für alle Azure AD Premium P2 Lizenzen
- [ ] Kein Benutzer hat Global Admin dauerhaft (max. 2 Break-Glass-Konten)
- [ ] Alle privilegierten Rollen als "Eligible" konfiguriert
- [ ] Aktivierungs-MFA + Begründungspflicht für kritische Rollen
- [ ] PIM-Alerts konfiguriert und an Security-Postfach gesendet
- [ ] Quartalsweise Access Review für alle Eligible Assignments

### Credential Vaulting (CyberArk / BeyondTrust)
- [ ] Alle Tier-0 und Tier-1 Passwörter im PAM-Vault gespeichert
- [ ] Automatische Passwort-Rotation aktiv (≤ 30 Tage)
- [ ] PSM/Session-Proxy für alle RDP/SSH-Zugriffe auf kritische Systeme
- [ ] Session Recording aktiviert und Aufbewahrung ≥ 90 Tage
- [ ] Kein direkter RDP/SSH-Zugang ohne PAM (Firewall-Regel)

### LAPS
- [ ] Windows LAPS auf allen Workstations und Member-Servern aktiviert
- [ ] LAPS-Passwort-Länge ≥ 14 Zeichen
- [ ] Rotation nach jeder Verwendung (Post-Auth-Action) konfiguriert
- [ ] LAPS-Lesezugriff nur für Helpdesk/IT-Admins (keine normalen User)

### Monitoring & Audit
- [ ] Alle privilegierten Anmeldungen in SIEM geloggt
- [ ] Alert bei: Außerhalb Geschäftszeiten, unbekannte IP, zu viele Fehlversuche
- [ ] Monatlicher Report: Wer hat welche privilegierten Rollen aktiviert?
- [ ] Jährliche PAM-Auditierung durch Dritte (ISO 27001 / DORA-Anforderung)
```
