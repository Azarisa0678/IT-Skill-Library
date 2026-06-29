# SharePoint Online & OneDrive — Referenzmodul

SPO-Administration, Berechtigungen, PnP PowerShell, OneDrive Sync, Governance.

---

## SharePoint Online PowerShell (PnP + SPO)

```powershell
# ─── VERBINDUNG ───────────────────────────────────────────────────────────────
Install-Module PnP.PowerShell -Force
Install-Module Microsoft.Online.SharePoint.PowerShell -Force

# PnP (für Site-Administration)
Connect-PnPOnline -Url "https://firma.sharepoint.com" `
    -Interactive    # oder -ClientId für App-Auth

# SPO Admin Center
Connect-SPOService -Url "https://firma-admin.sharepoint.com"

# ─── SITE-MANAGEMENT ──────────────────────────────────────────────────────────
# Alle Sites anzeigen
Get-SPOSite -Limit All |
    Select-Object Url, Title, StorageUsageCurrent, SharingCapability |
    Sort-Object StorageUsageCurrent -Descending |
    Format-Table -AutoSize

# Neue Team-Site erstellen
New-PnPSite -Type TeamSite -Title "Projektteam Alpha" `
    -Alias "projektteam-alpha" `
    -IsPublic:$false `
    -Owners "it-admin@firma.de" `
    -Members "projektteam-alpha@firma.de"

# Site-Größe und Storage Quota setzen
Set-SPOSite -Identity "https://firma.sharepoint.com/sites/projektteam-alpha" `
    -StorageQuota 10240 `        # 10 GB in MB
    -StorageQuotaWarningLevel 8192

# ─── BERECHTIGUNGEN ───────────────────────────────────────────────────────────
# Externe Freigabe deaktivieren (für sensible Sites)
Set-SPOSite -Identity "https://firma.sharepoint.com/sites/hr" `
    -SharingCapability Disabled

# Site-Besitzer hinzufügen
Add-SPOUser -Site "https://firma.sharepoint.com/sites/projektteam-alpha" `
    -LoginName "user@firma.de" `
    -Group "Projektteam Alpha Owners"

# Alle Benutzer einer Site anzeigen
Get-SPOUser -Site "https://firma.sharepoint.com/sites/projektteam-alpha" |
    Where-Object {$_.IsSiteAdmin -eq $false} |
    Select-Object LoginName, DisplayName, Groups

# ─── PnP: LISTEN UND BIBLIOTHEKEN ────────────────────────────────────────────
Connect-PnPOnline -Url "https://firma.sharepoint.com/sites/projektteam-alpha" -Interactive

# Alle Listen
Get-PnPList | Select-Object Title, ItemCount, LastItemModifiedDate | Format-Table

# Elemente aus Liste abrufen
$items = Get-PnPListItem -List "Dokumente" -PageSize 500
$items | ForEach-Object {
    [PSCustomObject]@{
        Name    = $_["FileLeafRef"]
        Autor   = $_["Author"].LookupValue
        Geändert = $_["Modified"]
        Größe   = $_["File_x0020_Size"]
    }
} | Format-Table -AutoSize

# ─── DATEIVERWALTUNG ──────────────────────────────────────────────────────────
# Datei hochladen
Add-PnPFile -Path "C:\Dokumente\bericht.pdf" `
    -Folder "Freigegebene Dokumente/Berichte"

# Ordner erstellen
Add-PnPFolder -Name "Archiv 2024" -Folder "Freigegebene Dokumente"

# Berechtigungen brechen (unique permissions)
Set-PnPListItemPermission -List "Verträge" -Identity 42 `
    -AddRole "Lesen" -User "partner@extern.de"
```

---

## SharePoint Governance

```powershell
# ─── TENANT-WEITE EINSTELLUNGEN ───────────────────────────────────────────────
# Externe Freigabe global einschränken
Set-SPOTenant -SharingCapability ExistingExternalUserSharingOnly
    # Optionen: Disabled, ExistingExternalUserSharingOnly,
    #           ExternalUserSharingOnly, ExternalUserAndGuestSharing

# Gastzugang nur für verifizierte Domains
Set-SPOTenant -AllowedDomainGuidsForSyncClient @()
Set-SPOTenant -SharingAllowedDomainList "partner.de trusted-customer.com"
Set-SPOTenant -SharingBlockedDomainList "competitor.de"

# Ablaufdatum für Gastbenutzer
Set-SPOTenant -ExternalUserExpirationRequired $true
Set-SPOTenant -ExternalUserExpireInDays 180

# Ablaufdatum für Freigabe-Links
Set-SPOTenant -RequireAnonymousLinksExpireInDays 7

# ─── SITE TEMPLATES ───────────────────────────────────────────────────────────
# Template aus bestehender Site exportieren
Get-PnPSiteTemplate -Out "projekt-template.xml" -Handlers Lists,Fields,ContentTypes

# Template auf neue Site anwenden
Invoke-PnPSiteTemplate -Path "projekt-template.xml"

# ─── GOVERNANCE-BERICHT ───────────────────────────────────────────────────────
function Get-SPOGovernanceReport {
    $sites = Get-SPOSite -Limit All

    # Sites ohne Besitzer
    $sitesWithoutOwner = $sites | Where-Object {
        (Get-SPOUser -Site $_.Url | Where-Object {$_.IsSiteAdmin}).Count -eq 0
    }

    # Externe Freigabe aktiv
    $externalSharing = $sites | Where-Object {
        $_.SharingCapability -ne "Disabled"
    }

    Write-Host "Sites ohne Besitzer: $($sitesWithoutOwner.Count)" -ForegroundColor Red
    Write-Host "Sites mit externer Freigabe: $($externalSharing.Count)" -ForegroundColor Yellow

    $externalSharing |
        Select-Object Url, Title, SharingCapability |
        Export-Csv "external-sharing-report.csv" -NoTypeInformation -Encoding UTF8
}
```

---

## OneDrive Administration

```powershell
# ─── ONEDRIVE STORAGE ─────────────────────────────────────────────────────────
# Standard-Quota setzen (1 TB)
Set-SPOTenant -OneDriveStorageQuota 1048576   # in MB

# Einzelnen Benutzer-Quota erhöhen
Set-SPOSite -Identity "https://firma-my.sharepoint.com/personal/user_firma_de" `
    -StorageQuota 5242880 `         # 5 TB
    -StorageQuotaWarningLevel 4194304

# OneDrive-Speichernutzung aller Benutzer
Get-SPOSite -IncludePersonalSite $true -Limit All -Filter "Url -like '*-my.sharepoint.com/personal*'" |
    Select-Object Owner, Url,
        @{N='GenutztGB'; E={[math]::Round($_.StorageUsageCurrent/1024,2)}},
        @{N='QuotaGB';   E={[math]::Round($_.StorageQuota/1024,2)}} |
    Sort-Object GenutztGB -Descending |
    Format-Table -AutoSize

# ─── ONEDRIVE BEI AUSSCHEIDEN ────────────────────────────────────────────────
# Zugang für Manager freischalten (30 Tage nach Kündigung)
Set-SPOUser -Site "https://firma-my.sharepoint.com/personal/ex_user_firma_de" `
    -LoginName "manager@firma.de" `
    -IsSiteCollectionAdmin $true

# OneDrive-Daten sichern bevor Konto gelöscht wird
# → Intune/Microsoft 365 Admin Center: Geräte → OneDrive-Dateien aufbewahren (24 Monate)

# ─── SYNC-CLIENT STEUERUNG ────────────────────────────────────────────────────
# Nur Firmen-PCs dürfen synchronisieren (via Entra Device-ID)
Set-SPOTenant -AllowedDomainGuidsForSyncClient @("tenant-guid")

# Bestimmte Dateitypen vom Sync ausschließen
Set-SPOTenant -ExcludedFileExtensionsForSyncClient @("exe","bat","cmd","vbs","js")
```

---

## SharePoint-Suche und Content-Management

```powershell
# ─── VERWALTETE EIGENSCHAFTEN ─────────────────────────────────────────────────
# Verwaltete Eigenschaft anzeigen
Get-PnPSearchConfiguration -Scope Tenant

# ─── VERSIONSVERLAUF BEREINIGEN ───────────────────────────────────────────────
# Alle Bibliotheken einer Site: Versionen auf 10 begrenzen
$lists = Get-PnPList | Where-Object {$_.BaseType -eq "DocumentLibrary"}
foreach ($list in $lists) {
    Set-PnPList -Identity $list.Id `
        -MajorVersions 10 `
        -EnableVersioning $true
}

# ─── CROSS-SITE SCRIPTING SCHUTZ ─────────────────────────────────────────────
# Custom Scripts deaktivieren (für alle Sites außer Entwicklung)
Set-SPOSite -Identity "https://firma.sharepoint.com/sites/production" `
    -DenyAddAndCustomizePages 1

# Tenant-weit
Set-SPOTenant -DelayDenyAddAndCustomizePagesEnforcement $false

Disconnect-PnPOnline
Disconnect-SPOService
```
