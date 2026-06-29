# Lizenzmanagement & Kostenoptimierung

## Microsoft 365 Lizenztypen

### Wichtigste Lizenz-SKUs für Unternehmen

| Lizenz | Zielgruppe | Wichtigste Inhalte | Preis/Monat* |
|--------|-----------|-------------------|-------------|
| **M365 Business Basic** | KMU bis 300 User | Exchange, Teams, SharePoint (Web-Apps) | ~6 € |
| **M365 Business Standard** | KMU bis 300 User | + Office Desktop Apps, Bookings | ~12 € |
| **M365 Business Premium** | KMU bis 300 User | + Intune, Defender, Entra ID P1 | ~22 € |
| **M365 E3** | Enterprise | Office Desktop, EMS E3, Win E3 | ~32 € |
| **M365 E5** | Enterprise | + Defender, Compliance, Power BI | ~57 € |
| **M365 F1** | Frontline Worker | Teams (keine Desktop-Apps), Exchange Kiosk | ~2 € |
| **M365 F3** | Frontline Worker | + Office Web Apps, Intune | ~8 € |
| **Entra ID P1** | Add-on | Conditional Access, PIM (eligible), SSPR | ~6 € |
| **Entra ID P2** | Add-on | + Identity Protection, PIM (vollständig) | ~9 € |

*Preise ca. Stand 2024, ohne Rabatte. Immer aktuell prüfen: https://www.microsoft.com/de-de/microsoft-365/business/compare-all-plans

---

## Lizenzaudit mit PowerShell

```powershell
Connect-MgGraph -Scopes "User.Read.All","Organization.Read.All","Directory.Read.All"

# Übersicht aller Lizenzen im Tenant
Get-MgSubscribedSku | Select-Object `
    SkuPartNumber,
    @{N='Gesamt';     E={$_.PrepaidUnits.Enabled}},
    @{N='Genutzt';    E={$_.ConsumedUnits}},
    @{N='Verfügbar';  E={$_.PrepaidUnits.Enabled - $_.ConsumedUnits}},
    @{N='Auslastung'; E={[math]::Round(($_.ConsumedUnits / $_.PrepaidUnits.Enabled) * 100, 1).ToString() + '%'}} |
    Format-Table -AutoSize

# Benutzer ohne zugewiesene Lizenzen (Optimierungspotenzial)
Get-MgUser -All -Property DisplayName,UserPrincipalName,AssignedLicenses,AccountEnabled |
    Where-Object { $_.AssignedLicenses.Count -eq 0 -and $_.AccountEnabled } |
    Select-Object DisplayName, UserPrincipalName |
    Export-Csv "C:\Reports\BenutzerOhneLizenz.csv" -NoTypeInformation -Encoding UTF8

# Deaktivierte Benutzer mit Lizenzen (sofort freigeben!)
Get-MgUser -All -Property DisplayName,UserPrincipalName,AssignedLicenses,AccountEnabled |
    Where-Object { $_.AssignedLicenses.Count -gt 0 -and -not $_.AccountEnabled } |
    Select-Object DisplayName, UserPrincipalName,
        @{N='Lizenzen';E={$_.AssignedLicenses.Count}} |
    Export-Csv "C:\Reports\DeaktivierteBenutzerMitLizenz.csv" -NoTypeInformation -Encoding UTF8

# Inaktive Benutzer mit Lizenzen (90+ Tage kein Login)
$cutoff = (Get-Date).AddDays(-90).ToString("yyyy-MM-ddTHH:mm:ssZ")
Get-MgUser -All -Filter "signInActivity/lastSignInDateTime le $cutoff" `
    -Property DisplayName,UserPrincipalName,AssignedLicenses,SignInActivity |
    Where-Object { $_.AssignedLicenses.Count -gt 0 } |
    Select-Object DisplayName, UserPrincipalName,
        @{N='LetzterLogin';E={$_.SignInActivity.LastSignInDateTime}},
        @{N='AnzahlLizenzen';E={$_.AssignedLicenses.Count}} |
    Export-Csv "C:\Reports\InaktiveBenutzerMitLizenz.csv" -NoTypeInformation -Encoding UTF8

# Lizenz von Benutzer entfernen (License Harvesting)
$userId = (Get-MgUser -UserId "ausgeschieden@firma.de").Id
$currentLicenses = (Get-MgUserLicenseDetail -UserId $userId).SkuId
Set-MgUserLicense -UserId $userId `
    -AddLicenses @() `
    -RemoveLicenses $currentLicenses
Write-Host "Lizenzen entfernt für $userId"
```

---

## Gruppenbasierte Lizenzzuweisung (empfohlen)

```powershell
# Lizenzen nie direkt Benutzern zuweisen — immer über Gruppen!
# Vorteil: Automatisch beim Onboarding/Offboarding

# Lizenz-Gruppe erstellen
New-MgGroup -DisplayName "LIC-M365-BusinessPremium" `
    -MailNickname "LIC-M365-BP" `
    -SecurityEnabled $true `
    -MailEnabled $false `
    -GroupTypes @()

# SKU-ID ermitteln
$skuId = (Get-MgSubscribedSku |
    Where-Object SkuPartNumber -eq "SPB").SkuId  # SPB = M365 Business Premium

# Lizenzzuweisung zur Gruppe (über Portal oder Graph)
# Im Portal: Entra ID → Gruppen → Gruppe wählen → Lizenzen
# Oder per Graph API:
$body = @{
    addLicenses    = @(@{ skuId = $skuId })
    removeLicenses = @()
}
Set-MgGroupLicense -GroupId $groupId -BodyParameter $body
```

---

## Azure Kostenoptimierung

```powershell
# Azure Cost Management per PowerShell
Connect-AzAccount

# Aktuelle Kosten der letzten 30 Tage
Get-AzConsumptionUsageDetail `
    -StartDate (Get-Date).AddDays(-30) `
    -EndDate (Get-Date) |
    Group-Object -Property ResourceGroupName |
    Select-Object Name, @{N='Kosten';E={($_.Group | Measure-Object PretaxCost -Sum).Sum}} |
    Sort-Object Kosten -Descending |
    Format-Table -AutoSize

# Nicht genutzte VMs identifizieren (CPU < 5% in letzten 7 Tagen)
# Über Azure Advisor (Portal) oder:
Get-AzAdvisorRecommendation -Category "Cost" |
    Where-Object { $_.ImpactedField -eq "Microsoft.Compute/virtualMachines" } |
    Select-Object ResourceMetadata, ShortDescription |
    Format-List

# Reserved Instances Empfehlung
# Regel: Wenn VM seit 6+ Monaten stabil läuft → 1-Jahr-Reservation = ~30% Ersparnis
# Über Azure Portal: Cost Management → Reservations → Empfehlungen
```

### Kostenoptimierungs-Checkliste

```
Sofortige Maßnahmen:
[ ] Deaktivierte Benutzer: Lizenzen entfernen
[ ] Inaktive Benutzer (90+ Tage): Lizenzen prüfen
[ ] Non-Prod VMs: Auto-Shutdown 18:00–08:00 Uhr
[ ] Ungenutzte Public IPs: löschen (kosten auch ohne VM)
[ ] Ungenutzte Managed Disks: löschen
[ ] Leere Resource Groups bereinigen

Mittelfristig:
[ ] Reserved Instances für stabile Prod-VMs (1-Jahr oder 3-Jahr)
[ ] Hybrid Benefit: Windows Server Lizenzen anrechnen lassen
[ ] Azure Dev/Test Subscription für Entwicklungsumgebungen
[ ] Budget-Alerts: Benachrichtigung bei 80% und 100% des Budgets
[ ] Azure Policy: Kostenpflichtige VM-SKUs einschränken

Lizenzoptimierung:
[ ] F1/F3 für Frontline Workers statt E3 (deutlich günstiger)
[ ] E3 + EMS E3 vs. M365 E3 — Bundlepreise vergleichen
[ ] Nicht genutzte Add-ons identifizieren und kündigen
[ ] Jahresvertrag statt Monat-für-Monat (Rabatt ~15%)
```

---

## Lizenz-Reporting Template (Deutsch)

```markdown
# Lizenz-Statusbericht — [Monat Jahr]
**Erstellt:** [Datum] | **Erstellt von:** [Name]

## Übersicht
| Lizenz | Gesamt | Genutzt | Verfügbar | Auslastung |
|--------|--------|---------|-----------|------------|
| M365 Business Premium | | | | |
| Entra ID P2 | | | | |
| [weitere] | | | | |

## Optimierungspotenzial
- Deaktivierte Benutzer mit Lizenz: **X** (Einsparung: X €/Monat)
- Inaktive Benutzer mit Lizenz: **X** (Einsparung: X €/Monat)
- Freie Lizenzen gesamt: **X**

## Empfehlungen
1. [Maßnahme] — Einsparung: X €/Monat
2. [Maßnahme] — Einsparung: X €/Monat

## Azure-Kosten (Top 5 Resource Groups)
| Resource Group | Kosten/Monat | Trend |
|---------------|-------------|-------|
| | | |
```

---

## Lizenz-Zuweisung via PowerShell (Graph)

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All","Directory.ReadWrite.All"

# Verfügbare Lizenzen im Tenant
Get-MgSubscribedSku | Select-Object SkuPartNumber,
    @{N='Verfügbar'; E={$_.PrepaidUnits.Enabled - $_.ConsumedUnits}},
    @{N='Gesamt';    E={$_.PrepaidUnits.Enabled}},
    @{N='Genutzt';   E={$_.ConsumedUnits}} |
    Format-Table -AutoSize

# Lizenz zuweisen (z.B. M365 Business Premium)
$sku = Get-MgSubscribedSku | Where-Object {$_.SkuPartNumber -eq "SPB"}
$params = @{
    AddLicenses    = @(@{ SkuId = $sku.SkuId })
    RemoveLicenses = @()
}
Set-MgUserLicense -UserId "user@firma.de" -BodyParameter $params

# Lizenz entfernen
$params = @{
    AddLicenses    = @()
    RemoveLicenses = @($sku.SkuId)
}
Set-MgUserLicense -UserId "user@firma.de" -BodyParameter $params

# Massenweise Lizenzen zuweisen (aus CSV)
# CSV-Format: UserPrincipalName
Import-Csv "neue-mitarbeiter.csv" | ForEach-Object {
    try {
        Set-MgUserLicense -UserId $_.UserPrincipalName `
            -BodyParameter @{
                AddLicenses    = @(@{ SkuId = $sku.SkuId })
                RemoveLicenses = @()
            }
        Write-Host "OK: $($_.UserPrincipalName)" -ForegroundColor Green
    } catch {
        Write-Host "FEHLER: $($_.UserPrincipalName) — $_" -ForegroundColor Red
    }
}
```

---

## Lizenz-Audit und Kostenoptimierung

```powershell
# ─── UNLIZENZIERTE BENUTZER MIT POSTFACH ─────────────────────────────────────
Connect-ExchangeOnline -ShowBanner:$false
$mailboxes = Get-Mailbox -ResultSize Unlimited |
    Where-Object {$_.RecipientTypeDetails -eq "UserMailbox"}

$unlicensed = $mailboxes | ForEach-Object {
    $user = Get-MgUser -UserId $_.UserPrincipalName -Property AssignedLicenses
    if ($user.AssignedLicenses.Count -eq 0) { $_.PrimarySmtpAddress }
}
Write-Host "Unlizenzierte Postfächer: $($unlicensed.Count)"

# ─── INAKTIVE LIZENZEN (kein Login > 90 Tage) ────────────────────────────────
$inactiveUsers = Get-MgUser -All -Property DisplayName,UserPrincipalName,
    SignInActivity,AssignedLicenses |
    Where-Object {
        $_.AssignedLicenses.Count -gt 0 -and
        ($_.SignInActivity.LastSignInDateTime -lt (Get-Date).AddDays(-90) -or
         $null -eq $_.SignInActivity.LastSignInDateTime)
    }

$inactiveUsers | Select-Object DisplayName, UserPrincipalName,
    @{N='LetzterLogin'; E={$_.SignInActivity.LastSignInDateTime}},
    @{N='Lizenzen';     E={$_.AssignedLicenses.Count}} |
    Sort-Object LetzterLogin |
    Export-Csv "inaktive-lizenzen.csv" -NoTypeInformation -Encoding UTF8

Write-Host "Potenziell einsparbare Lizenzen: $($inactiveUsers.Count)"

# ─── LIZENZ-OVERLAP ANALYSE ──────────────────────────────────────────────────
# Benutzer mit doppelter Lizenzierung (E3 + Add-ons die in E5 enthalten)
$e3Users = Get-MgUser -All -Property AssignedLicenses |
    Where-Object {$_.AssignedLicenses.SkuId -contains $e3SkuId}
$e3WithDefender = $e3Users | Where-Object {
    $_.AssignedLicenses.SkuId -contains $defenderSkuId
}
Write-Host "E3 + Defender Add-on (→ E5 günstiger?): $($e3WithDefender.Count)"

Disconnect-MgGraph
Disconnect-ExchangeOnline -Confirm:$false
```

---

## Microsoft 365 Kostenoptimierungs-Checkliste

```markdown
## M365 Lizenz-Review Checkliste

### Monatlich
- [ ] Inaktive Benutzer (> 30 Tage kein Login) identifizieren
- [ ] Neue Mitarbeiter: Richtige Lizenz zugewiesen? (nicht pauschal E3)
- [ ] Ausgetretene Mitarbeiter: Lizenz entzogen?

### Quartalsweise
- [ ] Inaktive Lizenzen (> 90 Tage kein Login) → Kündigung prüfen
- [ ] Shared Mailboxes: Brauchen sie weiterhin Volllizenzen?
- [ ] Teams Rooms / gemeinsame Geräte: Richtige Raumlizenz?
- [ ] Lizenz-Overlap: E3 + Defender Add-ons vs. direktes E5?

### Jährlich
- [ ] Gesamte Lizenzanzahl vs. tatsächliche Nutzer (Concurrent Use?)
- [ ] Frontline Worker (F1/F3 statt E3 für Mitarbeiter ohne Desktop)?
- [ ] Power Platform: Pro-Lizenzen aktiv genutzt?
- [ ] Add-ons: Tatsächlich in Verwendung (Visio, Project, Power BI Pro)?
- [ ] Microsoft-Volume-Discount: Enterprise Agreement aktuell verhandelt?

### Tools
- Microsoft 365 Admin Center → Berichte → Nutzung
- Microsoft Productivity Score
- Azure Cost Management (für Azure-Komponenten)
- Drittanbieter: Blueshift365, Alchemy Technology (Lizenzoptimierung)
```
