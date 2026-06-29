# Referenz-Architekturen & Templates

## Architektur 1: KMU mit M365 Business Premium (bis 300 User)

```
┌─────────────────────────────────────────────────────────────┐
│                    Microsoft 365 Cloud                       │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Entra ID    │  │  Exchange    │  │  SharePoint /    │  │
│  │  (Identität) │  │  Online      │  │  OneDrive        │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Intune      │  │  Teams       │  │  Defender        │  │
│  │  (MDM/MAM)   │  │              │  │  (MDE + MDO)     │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
         ↑ Conditional Access + MFA für alle Zugriffe ↑
         
┌─────────────────────────────────────────────────────────────┐
│                   Unternehmens-Netzwerk                      │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Windows 11  │  │  macOS       │  │  Mobile          │  │
│  │  (Entra Join)│  │  (Intune)    │  │  (iOS/Android)   │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Lokaler Fileserver (optional) + NAS                  │  │
│  │  → Migration zu SharePoint/OneDrive empfohlen         │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

Lizenz: M365 Business Premium (~22 €/User/Monat)
Enthält: Office, Exchange, Teams, SharePoint, Intune, Defender, Entra ID P1
```

### KMU Onboarding-Reihenfolge
```
Woche 1:  Tenant einrichten, Domains verifizieren, Admin-Konten
Woche 2:  Entra ID, MFA aktivieren, Conditional Access Basis
Woche 3:  Exchange Online, E-Mail-Migration, SPF/DKIM/DMARC
Woche 4:  Intune, Geräte enrollen, Compliance Policies
Woche 5:  SharePoint/OneDrive, Datenmigration
Woche 6:  Teams, Schulung, Go-Live
```

---

## Architektur 2: Enterprise Hybrid (E3/E5 mit On-Prem AD)

```
┌─────────────────────────────────────────────────────────────┐
│                    Microsoft 365 E5 Cloud                    │
│                                                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐  │
│  │Entra ID  │ │Exchange  │ │SharePoint│ │  Defender    │  │
│  │P2 + PIM  │ │Online    │ │Online    │ │  Suite       │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────┘  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐  │
│  │  Intune  │ │  Teams   │ │Sentinel  │ │  Compliance  │  │
│  │          │ │ + Telefon│ │  SIEM    │ │  Center      │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────┘  │
└─────────────────────────────────────────────────────────────┘
                         ↑ Sync ↑
                  Entra Connect (PHS)
                         ↑
┌─────────────────────────────────────────────────────────────┐
│                    On-Premises                               │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Active       │  │  Windows     │  │  Exchange        │  │
│  │ Directory    │  │  Server      │  │  Server          │  │
│  │ (Tier 0)     │  │  (Tier 1/2)  │  │  (Hybrid)        │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Azure VPN Gateway / ExpressRoute → Azure IaaS        │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## Architektur 3: Azure Landing Zone (Hybrid IaaS)

```
┌─────────────────────────────────────────────────────────────┐
│                    Azure Tenant                              │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Management Group: Firma                             │   │
│  │                                                      │   │
│  │  ┌──────────────┐    ┌──────────────────────────┐   │   │
│  │  │ Subscription │    │  Subscription            │   │   │
│  │  │ Produktion   │    │  Entwicklung             │   │   │
│  │  │              │    │  (Dev/Test Preise)        │   │   │
│  │  │ rg-hub-we    │    │  rg-dev-we               │   │   │
│  │  │ ┌──────────┐ │    │  ┌──────────────────────┐│   │   │
│  │  │ │ VNet Hub │ │    │  │ VNet Dev             ││   │   │
│  │  │ │ Bastion  │ │    │  │ (Peering zu Hub)     ││   │   │
│  │  │ │ Firewall │ │    └──┴──────────────────────┘│   │   │
│  │  │ │ VPN GW   │ │                               │   │   │
│  │  │ └──────────┘ │    ┌──────────────────────────┐   │   │
│  │  │              │    │  Subscription            │   │   │
│  │  │ rg-prod-we   │    │  Identität               │   │   │
│  │  │ ┌──────────┐ │    │  (Entra Connect, ADFS)   │   │   │
│  │  │ │ VNet     │ │    └──────────────────────────┘   │   │
│  │  │ │ Spoke    │ │                                   │   │
│  │  │ │(Peering) │ │                                   │   │
│  │  │ └──────────┘ │                                   │   │
│  │  └──────────────┘                                   │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                         ↕ ExpressRoute / VPN
┌──────────────────────────────────────┐
│  On-Premises Rechenzentrum           │
│  AD, Exchange, File Server           │
└──────────────────────────────────────┘
```

---

## Vorlage: M365 Betriebshandbuch

```markdown
# Microsoft 365 Betriebshandbuch
**Organisation:** [Unternehmensname]
**Version:** 1.0 | **Datum:** [Datum]
**Verantwortlich:** [Name], IT-Administration
**Klassifizierung:** Intern — Vertraulich

---

## 1. Umgebungsübersicht

### Tenant-Informationen
| Parameter | Wert |
|-----------|------|
| Tenant-ID | |
| Primäre Domain | firma.de |
| Admin-Portal | admin.microsoft.com |
| Lizenzmodell | M365 Business Premium |
| Benutzeranzahl | |
| Entra Connect Server | |

### Kritische Admin-Konten
| Konto | Funktion | MFA | Letzter Test |
|-------|---------|-----|-------------|
| admin@firma.de | Globaler Admin | FIDO2 | |
| notfall1@firma.de | Break-Glass | Physischer Key | |
| notfall2@firma.de | Break-Glass | Physischer Key | |

---

## 2. Regelmäßige Wartungsaufgaben

### Täglich
- [ ] Service Health prüfen (admin.microsoft.com → Integrität)
- [ ] Security Alerts in security.microsoft.com prüfen
- [ ] Entra Connect Sync-Status prüfen

### Wöchentlich
- [ ] Non-Compliant Geräte-Report prüfen
- [ ] MFA-Registrierungsstatus prüfen
- [ ] Neue Gastbenutzer prüfen und genehmigen

### Monatlich
- [ ] Lizenz-Audit: ungenutzte Lizenzen freigeben
- [ ] Inaktive Benutzer (90 Tage): deaktivieren oder lizenzentziehen
- [ ] Notfallkonto-Login testen und dokumentieren
- [ ] GPO-Backup erstellen und testen
- [ ] Conditional Access Policies reviewen

### Quartalsweise
- [ ] Access Reviews durchführen (privilegierte Rollen)
- [ ] PIM-Zuweisungen reviewen
- [ ] Externe Benutzer und Gastkonten bereinigen
- [ ] Lizenzplanung für nächstes Quartal

---

## 3. Onboarding-Prozess

### Neue Mitarbeiter
1. HR meldet neuen Mitarbeiter (Name, Abteilung, Startdatum, Manager)
2. IT erstellt Konto mit Onboarding-Script (.\New-Mitarbeiter.ps1)
3. Lizenz wird über Gruppenbasierung automatisch zugewiesen
4. Manager erhält E-Mail mit temporärem Passwort
5. Gerät wird über Autopilot eingerichtet (Starttag)
6. Einführung in M365-Tools (15-Minuten-Quickstart-Guide)

### Ausgeschiedene Mitarbeiter
1. HR meldet Ausscheiden (Name, letzter Arbeitstag, Nachfolger)
2. Am letzten Arbeitstag: Offboarding-Script (.\Remove-Mitarbeiter.ps1)
3. Konto wird 30 Tage im deaktivierten Zustand behalten
4. Nach 30 Tagen: endgültige Löschung (Papierkorb weitere 30 Tage)

---

## 4. Notfall-Kontakte

| Rolle | Name | Telefon | E-Mail |
|-------|------|---------|-------|
| IT-Leitung | | | |
| Microsoft Support | | 0800 284 2244 | support.microsoft.com |
| IT-Dienstleister | | | |

### Eskalationspfad
1. IT-Helpdesk → Ticket erstellen
2. IT-Administrator → Direkte Behandlung
3. Microsoft Support → Bei Plattformproblemen
4. IT-Leitung → Bei kritischen Ausfällen (P1)

---

## 5. Passwort-Tresor (Referenz)

Zugangsdaten werden ausschließlich im genehmigten Passwort-Manager gespeichert.
Vault-Name: [Name des Passwort-Managers]
Verantwortlicher: [Name]
```

---

## Vorlage: Change Request M365

```markdown
# Change Request: [Titel]
**CR-Nr.:** CR-[Jahr]-[Nr.] | **Datum:** [Datum]
**Antragsteller:** [Name] | **Genehmiger:** [Name]

## Beschreibung der Änderung
[Was soll geändert werden?]

## Begründung
[Warum ist die Änderung notwendig?]

## Betroffene Systeme/Benutzer
[Welche Systeme und wie viele Benutzer sind betroffen?]

## Risikobewertung
☐ Gering — keine Auswirkung auf Produktion
☐ Mittel — kurze Unterbrechung möglich
☐ Hoch — Produktionsauswirkung erwartet

## Zeitplan
**Geplantes Durchführungsfenster:** [Datum, Uhrzeit – Uhrzeit]
**Rollback-Frist:** [Wann muss entschieden werden?]

## Durchführungsplan
1. [Schritt 1]
2. [Schritt 2]

## Rollback-Plan
1. [Schritt 1 wenn Rollback nötig]

## Testergebnis
☐ Getestet in Test-/Dev-Umgebung
☐ Genehmigt durch [Name]

## Status
☐ Beantragt ☐ Genehmigt ☐ Abgelehnt ☐ Durchgeführt ☐ Abgebrochen
```
