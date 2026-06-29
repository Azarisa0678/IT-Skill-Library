# Exchange Hybrid & Exchange Online — Referenzmodul

Exchange Online Administration, Hybrid-Konfiguration, Migration, Mail-Flow, Sicherheit.

---

## Exchange Online PowerShell

```powershell
# Verbindung (Modern Auth / MFA-fähig)
Install-Module ExchangeOnlineManagement -Force
Connect-ExchangeOnline -UserPrincipalName admin@firma.de -ShowBanner:$false

# ─── POSTFÄCHER ────────────────────────────────────────────────────────────────
# Alle Postfächer mit Größe
Get-Mailbox -ResultSize Unlimited | Get-MailboxStatistics |
    Select-Object DisplayName, TotalItemSize, ItemCount |
    Sort-Object TotalItemSize -Descending | Format-Table -AutoSize

# Shared Mailboxes ohne Lizenz anzeigen
Get-Mailbox -RecipientTypeDetails SharedMailbox -ResultSize Unlimited |
    Select-Object DisplayName, PrimarySmtpAddress,
    @{N='Delegates'; E={(Get-MailboxPermission $_.Identity |
        Where-Object {$_.User -notlike "NT AUTHORITY*"}).User -join ", "}}

# Postfach-Berechtigungen setzen
Add-MailboxPermission -Identity "info@firma.de" -User "assistant@firma.de" `
    -AccessRights FullAccess -AutoMapping $true
Add-RecipientPermission "info@firma.de" -Trustee "assistant@firma.de" `
    -AccessRights SendAs -Confirm:$false

# Alias hinzufügen
Set-Mailbox -Identity "user@firma.de" -EmailAddresses `
    @{Add="alias@firma.de","alias2@firma-alt.de"}

# Postfach-Größenlimit
Set-Mailbox -Identity "ceo@firma.de" `
    -ProhibitSendQuota 100GB `
    -ProhibitSendReceiveQuota 110GB `
    -IssueWarningQuota 90GB

# ─── DISTRIBUTION GROUPS ──────────────────────────────────────────────────────
New-DistributionGroup -Name "IT-Team" -Alias "it-team" `
    -PrimarySmtpAddress "it-team@firma.de" -MemberJoinRestriction Closed

# Microsoft 365 Group erstellen
New-UnifiedGroup -DisplayName "Projektteam Alpha" -Alias "projekt-alpha" `
    -AccessType Private -AutoSubscribeNewMembers

# ─── MAIL-FLOW-REGELN (TRANSPORT RULES) ───────────────────────────────────────
# Externe E-Mail-Warnung hinzufügen
New-TransportRule -Name "Externe E-Mail Warnung" `
    -FromScope NotInOrganization `
    -PrependSubject "[EXTERN] " `
    -SetHeaderName "X-External-Source" `
    -SetHeaderValue "External"

# Disclaimer anhängen
New-TransportRule -Name "E-Mail Disclaimer" `
    -ApplyHtmlDisclaimerLocation Append `
    -ApplyHtmlDisclaimerText '<p style="font-size:9pt">Disclaimer Text...</p>' `
    -ApplyHtmlDisclaimerFallbackAction Wrap

# Alle Transport-Regeln anzeigen
Get-TransportRule | Select-Object Name, Priority, State, Description |
    Format-Table -AutoSize
```

---

## Exchange Hybrid — Konfiguration

```powershell
# ─── HYBRID-VORAUSSETZUNGEN ───────────────────────────────────────────────────
# 1. Exchange Server (2016/2019) on-premises
# 2. Azure AD Connect installiert und konfiguriert
# 3. Hybrid Configuration Wizard (HCW) ausgeführt
# 4. MX-Record und Autodiscover konfiguriert

# Hybrid-Status prüfen
Get-HybridConfiguration

# Connector-Status prüfen (On-Premises Exchange)
Get-SendConnector | Where-Object {$_.Name -like "*Outbound*"} |
    Select-Object Name, Enabled, SmartHosts, TlsAuthLevel

Get-ReceiveConnector | Where-Object {$_.Name -like "*Inbound*"} |
    Select-Object Name, Enabled, Bindings

# ─── MIGRATION BATCH ──────────────────────────────────────────────────────────
# Staged Migration: On-Premises → Exchange Online
New-MigrationBatch -Name "Migration-Wave1" `
    -SourceEndpoint (Get-MigrationEndpoint -Identity "On-Premises") `
    -CSVData ([System.IO.File]::ReadAllBytes("C:\migration-wave1.csv")) `
    -AutoStart -AutoComplete:$false `
    -NotificationEmails "it-admin@firma.de"

# Migration-Status
Get-MigrationBatch | Select-Object Identity, Status, TotalCount, SyncedCount, FailedCount
Get-MigrationUser -BatchId "Migration-Wave1" |
    Where-Object {$_.Status -eq "Failed"} |
    Select-Object EmailAddress, Error

# Batch abschließen
Complete-MigrationBatch -Identity "Migration-Wave1"

# ─── FREE/BUSY KONFIGURATION (HYBRID) ────────────────────────────────────────
# Organization Relationship prüfen (Verfügbarkeit zwischen On-Prem und Cloud)
Get-OrganizationRelationship | Select-Object Name, DomainNames, FreeBusyAccessEnabled

# Organisation Relationship aktualisieren
Set-OrganizationRelationship -Identity "On Premises to Exchange Online" `
    -FreeBusyAccessEnabled $true `
    -FreeBusyAccessLevel LimitedDetails
```

---

## Anti-Spam / Anti-Phishing (Defender for Office 365)

```powershell
# ─── ANTI-SPAM RICHTLINIE ─────────────────────────────────────────────────────
# Standard-Richtlinie anzeigen
Get-HostedContentFilterPolicy -Identity Default |
    Select-Object SpamAction, HighConfidenceSpamAction, PhishSpamAction,
                  BulkThreshold, MarkAsSpamBulkMail

# Strenge Richtlinie für VIPs
New-HostedContentFilterPolicy -Name "Strict-VIP" `
    -SpamAction MoveToJmf `
    -HighConfidenceSpamAction Quarantine `
    -PhishSpamAction Quarantine `
    -HighConfidencePhishAction Quarantine `
    -BulkThreshold 4

New-HostedContentFilterRule -Name "Strict-VIP Rule" `
    -HostedContentFilterPolicy "Strict-VIP" `
    -SentToMemberOf "vip-gruppe@firma.de" `
    -Priority 0

# ─── ANTI-PHISHING ────────────────────────────────────────────────────────────
Set-AntiPhishPolicy -Identity Default `
    -EnableSpoofIntelligence $true `
    -EnableUnauthenticatedSender $true `
    -EnableFirstContactSafetyTips $true `
    -ImpersonationProtectionState Automatic

# ─── SPF / DKIM / DMARC PRÜFEN ───────────────────────────────────────────────
# DKIM-Signierung aktivieren
New-DkimSigningConfig -DomainName firma.de -Enabled $true
# → DNS-Einträge (CNAME) im Output beachten!

Get-DkimSigningConfig -Identity firma.de |
    Select-Object Domain, Enabled, Status, Selector1CNAME, Selector2CNAME

# DMARC-Eintrag prüfen (via DNS)
Resolve-DnsName -Name "_dmarc.firma.de" -Type TXT | Select-Object Strings

# DMARC-Empfehlung:
# v=DMARC1; p=quarantine; pct=100; rua=mailto:dmarc@firma.de; aspf=s; adkim=s
```

---

## Quarantäne-Management

```powershell
# Alle quarantänierten Nachrichten der letzten 7 Tage
Get-QuarantineMessage -StartReceivedDate (Get-Date).AddDays(-7) |
    Select-Object Subject, SenderAddress, RecipientAddress, QuarantineReason |
    Format-Table -AutoSize

# Nachricht freigeben
Release-QuarantineMessage -Identity <QuarantineMessageIdentity> `
    -ReleaseToAll

# Massenfreigabe (alle False Positives eines Absenders)
Get-QuarantineMessage -SenderAddress "partner@external.de" |
    Release-QuarantineMessage -ReleaseToAll
```

---

## Mail-Flow-Troubleshooting

```powershell
# Message Trace (letzte 10 Tage)
Get-MessageTrace -SenderAddress "absender@firma.de" `
    -RecipientAddress "empfaenger@kunde.de" `
    -StartDate (Get-Date).AddDays(-2) `
    -EndDate (Get-Date) |
    Select-Object Received, SenderAddress, RecipientAddress, Subject, Status, Size

# Detaillierter Trace (für Diagnose)
Get-MessageTrace -MessageId "<messageid@firma.de>" |
    Get-MessageTraceDetail |
    Select-Object Date, Event, Action, Detail

# NDR-Analyse (häufige Bounce-Ursachen)
Get-MessageTrace -Status Failed -StartDate (Get-Date).AddDays(-1) |
    Select-Object RecipientAddress, Subject,
    @{N='Fehler'; E={(Get-MessageTraceDetail $_.MessageTraceId |
        Where-Object {$_.Event -eq "Fail"}).Detail}} |
    Group-Object Fehler | Sort-Object Count -Descending

# Remotekonnektivitätstest
# → https://testconnectivity.microsoft.com (extern)
# → Test-OutboundConnector für On-Premises

Disconnect-ExchangeOnline -Confirm:$false
```
