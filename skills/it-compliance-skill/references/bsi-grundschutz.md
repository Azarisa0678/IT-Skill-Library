# BSI IT-Grundschutz — Referenzmodul

Umsetzungshilfen, Bausteinauswahl, Schutzbedarfsfeststellung, Sicherheitskonzept, Zertifizierung.

---

## Schutzbedarfsfeststellung (SBF)

### SBF-Vorlage

```markdown
## Schutzbedarfsfeststellung — [Unternehmen] — [Datum]

### Schutzbedarfskategorien

| Kategorie | Beschreibung | Schaden |
|-----------|-------------|---------|
| **Normal** | Schadensauswirkung begrenzt und überschaubar | < 50.000 € |
| **Hoch** | Schadensauswirkung beträchtlich | 50.000 – 500.000 € |
| **Sehr hoch** | Schadensauswirkung existenzbedrohend oder katastrophal | > 500.000 € |

### Grundwerte: Vertraulichkeit / Integrität / Verfügbarkeit

| Asset | Prozess | Vertraulichkeit | Integrität | Verfügbarkeit | Gesamt |
|-------|---------|-----------------|------------|---------------|--------|
| Active Directory | Alle IT-Services | Sehr hoch | Sehr hoch | Sehr hoch | Sehr hoch |
| ERP-System (SAP) | Auftragsverarbeitung | Hoch | Sehr hoch | Hoch | Sehr hoch |
| Exchange/E-Mail | Kommunikation | Hoch | Hoch | Hoch | Hoch |
| Dateiserver | Dokumentenablage | Hoch | Hoch | Hoch | Hoch |
| Webserver (extern) | Website | Normal | Hoch | Hoch | Hoch |
| Entwicklungs-PC | Softwareentwicklung | Hoch | Normal | Normal | Hoch |
| Drucker | Drucken | Normal | Normal | Normal | Normal |

### Maximumprinzip
Schutzbedarf eines Systems = Maximum aller darauf laufenden Anwendungen
Beispiel: ERP-Server trägt SAP (sehr hoch) + Monitoring-Agent (normal) → sehr hoch
```

---

## Bausteinauswahl — Mapping

```markdown
## Bausteinauswahl für typische KMU-Infrastruktur

### ISMS-Bausteine (immer)
- ISMS.1    Sicherheitsmanagement
- ORP.1     Organisation
- ORP.2     Personal
- ORP.3     Sensibilisierung und Schulung zur Informationssicherheit
- ORP.4     Identitäts- und Berechtigungsmanagement
- CON.1     Kryptokonzept
- CON.2     Datenschutz
- CON.3     Datensicherungskonzept
- DER.1     Detektion von sicherheitsrelevanten Ereignissen
- DER.2.1   Behandlung von Sicherheitsvorfällen

### IT-Systeme
- SYS.1.1   Allgemeiner Server
- SYS.1.2.2 Windows Server 2016/2019/2022
- SYS.1.3   Server unter Linux und Unix
- SYS.2.1   Allgemeiner Client
- SYS.2.2.3 Clients unter Windows
- SYS.3.2.1 Allgemeine Smartphones und Tablets

### Anwendungen
- APP.2.1   Allgemeiner Verzeichnisdienst (Active Directory)
- APP.5.2   Microsoft Exchange und Outlook
- APP.3.1   Webanwendungen und Webservices
- APP.4.3   Relationale Datenbanken

### Netzwerk
- NET.1.1   Netzarchitektur und -design
- NET.3.2   Firewall
- NET.3.3   VPN
- NET.2.1   WLAN-Betrieb

### Infrastruktur
- INF.1     Allgemeines Gebäude
- INF.2     Rechenzentrum und Serverraum

### Betrieb
- OPS.1.1.2 Ordnungsgemäße IT-Administration
- OPS.1.1.3 Patch- und Änderungsmanagement
- OPS.1.1.4 Schutz vor Schadprogrammen
- OPS.1.1.5 Protokollierung
- OPS.2.2   Cloud-Nutzung
```

---

## Sicherheitskonzept — Dokumentstruktur

```markdown
# IT-Sicherheitskonzept [Unternehmen GmbH]
**Version:** 2.1  |  **Datum:** TT.MM.JJJJ  |  **Klassifizierung:** Intern
**Erstellt von:** [ISB]  |  **Genehmigt von:** [Geschäftsführung]

---

## 1. Einleitung und Geltungsbereich

### 1.1 Zweck
Dieses Dokument definiert die Sicherheitsmaßnahmen für die IT-Infrastruktur
der [Unternehmen GmbH] gemäß BSI IT-Grundschutz (Standard-Absicherung).

### 1.2 Geltungsbereich
Alle IT-Systeme, Anwendungen und Daten am Standort [Standort] sowie
Cloud-Dienste und Homeoffice-Arbeitsplätze der Mitarbeiter.

### 1.3 Abgrenzung
Nicht erfasst: Produktionssteuerung (OT), separate OT-Sicherheitskonzept vorhanden.

---

## 2. Sicherheitsziele

Basierend auf der Schutzbedarfsfeststellung gelten folgende Sicherheitsziele:

| Grundwert | Ziel | Maßnahmen |
|-----------|------|-----------|
| Vertraulichkeit | Schutz vor unbefugtem Zugriff | Verschlüsselung, Zugangskontrolle, Need-to-Know |
| Integrität | Schutz vor unbefugter Änderung | Digitale Signaturen, Checksummen, Change-Management |
| Verfügbarkeit | Sicherstellung der Betriebsfähigkeit | Redundanz, Backup, DR-Plan |

---

## 3. Strukturanalyse

### 3.1 Geschäftsprozesse
[Liste der Geschäftsprozesse aus Schutzbedarfsfeststellung]

### 3.2 IT-Systeme
[Asset-Inventar: Server, Clients, Netzwerkgeräte, Cloud-Dienste]

### 3.3 Netzplan
[Netzwerk-Topologie-Diagramm einfügen]

---

## 4. Schutzbedarfsfeststellung
[Verweis auf SBF-Dokument oder hier eingebettet]

---

## 5. Modellierung
[Zuordnung Bausteine zu Assets]

---

## 6. IT-Grundschutz-Check
[Tabelle: Anforderung → Umsetzung → Verantwortlicher → Status]

---

## 7. Risikoanalyse (nur bei erhöhtem Schutzbedarf)
[Für Assets mit hohem/sehr hohem Schutzbedarf: Gefährdungsübersicht, Eintrittshäufigkeit, Schadenspotenzial]

---

## 8. Maßnahmenplan

| Maßnahme | Priorität | Verantwortlich | Frist | Status | Kosten (ca.) |
|----------|-----------|----------------|-------|--------|--------------|
| MFA für alle Benutzer | Hoch | IT-Admin | 31.03.2024 | In Umsetzung | 2.000 € |
| EDR auf allen Endpunkten | Hoch | IT-Admin | 30.04.2024 | Geplant | 5.000 €/Jahr |
| Patch-Management-Tool | Mittel | IT-Admin | 30.06.2024 | Offen | 3.000 €/Jahr |
| ISB-Schulung | Niedrig | ISB | 31.12.2024 | Offen | 1.500 € |

---

## 9. Revision und Pflege
Dieses Konzept wird jährlich oder bei wesentlichen Änderungen überarbeitet.
Nächste Revision: [Datum]
```

---

## IT-Grundschutz-Zertifizierung — Prozess

```
ZERTIFIZIERUNGSTYPEN:
  Basis-Zertifikat:        Basis-Absicherung, vereinfachter Nachweis
  ISO 27001 auf IT-GS:    Standard-Absicherung + Zertifizierungsaudit durch BSI-Auditor

PROZESS ISO 27001 auf IT-GS:
  1. VORBEREITUNG (3-6 Monate):
     - Informationsverbund definieren
     - Sicherheitskonzept erstellen (Strukturanalyse, SBF, Modellierung)
     - IT-GS-Check durchführen (alle Anforderungen)
     - Maßnahmen umsetzen, Risikoanalyse

  2. PRÜFVORBEREITUNG (1-2 Monate):
     - Auditor beauftragen (BSI-lizenziert, Liste auf bsi.bund.de)
     - Dokumentation finalisieren
     - Vorgespräch mit Auditor

  3. AUDIT (1-2 Wochen):
     - Dokumentenprüfung (remote/vor Ort)
     - Interviews mit ISB, IT-Admin, Geschäftsleitung
     - Stichproben-Kontrollen (Systeme, Konfigurationen)
     - Abschlussgespräch + Findings

  4. ZERTIFIZIERUNG:
     - Bei erfolgreicher Prüfung: Zertifikat (3 Jahre Gültigkeit)
     - Jährliche Überwachungsaudits
     - Re-Zertifizierung nach 3 Jahren

TOOLS:
  VERINICE:  Open-Source IT-GS-Tool (empfohlen)
  HiScout:   Kommerzielle Lösung
  Digicomp:  SaaS-Lösung für KMU
  BSI-Tool:  Kostenloser IT-GS-Check (einfacher)
```

---

## BSI-Grundschutz-Kompendium Schnellzugriff

```
WICHTIGSTE ANFORDERUNGEN JE BAUSTEIN (Auszug):

ORP.4 (Berechtigungsmanagement):
  BASIS: Benutzerkonten eindeutig zuordenbar, Trennung Admin/User
  STANDARD: Prozess Anlage/Änderung/Sperrung, Rezertifizierung jährlich
  ERHÖHT: PAM-System für privilegierte Konten

OPS.1.1.3 (Patch-Management):
  BASIS: Systeminventar, Patch-Prozess definiert
  STANDARD: Kritische Patches ≤ 1 Woche, regelmäßige Schwachstellenscans
  ERHÖHT: CVSS-basierte Priorisierung, automatisiertes Patching

OPS.1.1.4 (Schutz vor Schadprogrammen):
  BASIS: AV auf Clients und Servern, regelmäßige Updates
  STANDARD: EDR statt reinem AV, zentrales Management
  ERHÖHT: Verhaltensbasierte Erkennung, SIEM-Integration

DER.2.1 (Sicherheitsvorfälle):
  BASIS: Meldewege für Mitarbeiter definiert
  STANDARD: IR-Plan, Klassifizierungsschema, Eskalationspfade
  ERHÖHT: CERT/SOC, Forensik-Capability, externe Experten-Retainer
```
