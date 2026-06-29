# Public Sector Security Reference

Spezialisierte Sicherheitsreferenz für Behörden, Kommunalverwaltungen, Landesbehörden,
Bundesbehörden, öffentliche Einrichtungen und IT-Dienstleister der öffentlichen Hand.
Schwerpunkt: Deutschland, BSI IT-Grundschutz, VS-NfD, EVB-IT, eGovernment-Sicherheit.

---

## Regulatorische Landschaft Öffentlicher Sektor (Deutschland)

### Rechtsrahmen-Übersicht

| Rahmen | Geltungsbereich | Kernanforderungen |
|---|---|---|
| **BSI IT-Grundschutz (200er-Reihe)** | Bundesbehörden (Pflicht), Länder/Kommunen (empfohlen) | IT-Grundschutz-Kompendium, Bausteine, Umsetzungshinweise |
| **BSI IT-Grundschutz-Zertifizierung** | Freiwillig, aber für Bundesbehörden zunehmend erwartet | Basis-, Standard-, Kern-Absicherung; ISO 27001 auf Basis IT-GS |
| **NIS2UmsuCG (geplant 2024/2025)** | Öffentliche Verwaltungen (Bund, Länder, ggf. Kommunen) | Meldepflichten, Art. 21 Mindestmaßnahmen, ISMS |
| **VS-NfD (Verschlusssachen-Anweisung)** | Bundesbehörden, beliehene Stellen, Auftragnehmer | VS-NfD-Behandlungsvorschriften, zugelassene IT-Systeme, SINA |
| **eIDAS-Verordnung (EU 910/2014)** | Elektronische Identifizierung und Vertrauensdienste | eID, qualifizierte elektronische Signatur, Siegelung |
| **OZG (Onlinezugangsgesetz)** | Bund und Länder | Digitalisierung Verwaltungsleistungen, Portalverbund |
| **DSGVO / BDSG** | Alle Behörden | Datenschutz, Verzeichnis Verarbeitungstätigkeiten, DSB-Pflicht |
| **EVB-IT** | Öffentliche Auftraggeber beim IT-Kauf | Vertragsklauseln, AGB-Ersatz für IT-Beschaffung |
| **UVgO / VgV** | IT-Vergabe | Vergabeverfahren öffentliche IT-Aufträge |
| **BayDiG / LandesdigitalisierungsG** | Länderspezifisch (Bayern, NRW etc.) | Digitale Verwaltung, IT-Sicherheitsvorgaben Länder |

---

## BSI IT-Grundschutz — Umsetzung

### IT-Grundschutz-Kompendium Struktur

```
Schichten und ausgewählte Bausteine (IT-Grundschutz-Kompendium Edition 2023):

SCHICHT ISMS:
  ISMS.1 Sicherheitsmanagement
  
SCHICHT ORP (Organisation und Personal):
  ORP.1  Organisation
  ORP.2  Personal
  ORP.3  Sensibilisierung und Schulung
  ORP.4  Identitäts- und Berechtigungsmanagement

SCHICHT CON (Konzepte):
  CON.1  Kryptokonzept
  CON.2  Datenschutz
  CON.3  Datensicherungskonzept
  CON.6  Löschen und Vernichten
  CON.7  Informationssicherheit auf Auslandsreisen
  CON.8  Software-Entwicklung
  CON.11 Geheimschutz VS-NfD (besondere Anforderungen)

SCHICHT OPS (Betrieb):
  OPS.1.1.2 Ordnungsgemäße IT-Administration
  OPS.1.1.3 Patch- und Änderungsmanagement
  OPS.1.1.4 Schutz vor Schadprogrammen
  OPS.1.1.5 Protokollierung
  OPS.1.1.6 Software-Tests und -Freigaben
  OPS.1.2.2 Archivierung
  OPS.2.2  Cloud-Nutzung
  OPS.2.3  Outsourcing für Kunden

SCHICHT DER (Detektion und Reaktion):
  DER.1  Detektion von sicherheitsrelevanten Ereignissen
  DER.2.1 Behandlung von Sicherheitsvorfällen
  DER.2.2 Vorsorge für die IT-forensische Untersuchung
  DER.3.1 Audits und Revisionen
  DER.3.2 Revisionen durch Dritte
  DER.4  Notfallmanagement

SCHICHT APP (Anwendungen):
  APP.1.1 Office-Produkte
  APP.1.2 Webbrowser
  APP.2.1 Allgemeiner Verzeichnisdienst (Active Directory)
  APP.3.1 Webanwendungen und Webservices
  APP.4.3 Relationale Datenbanken
  APP.4.4 Kubernetes
  APP.5.2 Microsoft Exchange und Outlook
  APP.5.3 Allgemeiner Email-Client und -Server
  APP.6  Allgemeine Software (Beschaffung)

SCHICHT SYS (IT-Systeme):
  SYS.1.1  Allgemeiner Server
  SYS.1.2.2 Windows Server
  SYS.1.3  Server unter Linux und Unix
  SYS.2.1  Allgemeiner Client
  SYS.2.2.3 Clients unter Windows
  SYS.2.3  Clients unter Linux und Unix
  SYS.3.2.1 Allgemeine Smartphones und Tablets
  SYS.4.1  Drucker, Kopierer und Multifunktionsgeräte

SCHICHT IND (Industrielle IT):
  IND.1   Bausteine der industriellen IT

SCHICHT NET (Netze und Kommunikation):
  NET.1.1  Netzarchitektur und -design
  NET.1.2  Netzmanagement
  NET.2.1  WLAN-Betrieb
  NET.3.1  Router und Switches
  NET.3.2  Firewall
  NET.3.3  VPN
  NET.4.1  TK-Anlagen
  NET.4.2  VoIP

SCHICHT INF (Infrastruktur):
  INF.1   Allgemeines Gebäude
  INF.2   Rechenzentrum und Serverraum
  INF.5   Raum sowie Schrank für technische Infrastruktur
  INF.7   Büroarbeitsplatz
  INF.8   Häuslicher Arbeitsplatz
```

### Absicherungsvarianten

```
1. BASIS-ABSICHERUNG (Einstieg):
   - Schnelle Grundabsicherung für kleine Behörden/Einrichtungen
   - Umsetzung aller Basis-Anforderungen der relevanten Bausteine
   - Keine Schutzbedarfsfeststellung erforderlich
   - Geeignet für: kleine Kommunen, kleine Fachbehörden

2. STANDARD-ABSICHERUNG (Regelfall):
   - Vollständige IT-Grundschutz-Methodik
   - Strukturanalyse + Schutzbedarfsfeststellung + Risikoanalyse
   - Umsetzung aller Basis- und Standard-Anforderungen
   - Optional: ISO 27001 Zertifizierung auf Basis IT-Grundschutz
   - Geeignet für: mittlere und große Behörden

3. KERN-ABSICHERUNG (Kronjuwelen):
   - Fokus auf die kritischsten Assets ("Kronjuwelen")
   - Nur ausgewählte, hochwertige Systeme in Scope
   - Alle Anforderungen (Basis + Standard + Erhöhter Schutzbedarf)
   - Geeignet für: Bundesbehörden mit besonders sensiblen Daten

WICHTIG: Für ISO 27001 auf Basis IT-Grundschutz:
- Standard-Absicherung ist Voraussetzung
- Zertifizierung durch BSI-lizenzierte Auditoren
- Gültigkeit: 3 Jahre, jährliche Überwachungsaudits
```

### IT-Grundschutz Workflow

```
Schritt 1: INITIIERUNG (ISMS.1)
  → Leitungsebene beauftragt ISB (Informationssicherheitsbeauftragter)
  → IT-Sicherheitsleitlinie erstellen und veröffentlichen
  → Ressourcen bereitstellen

Schritt 2: STRUKTURANALYSE
  → Informationsverbund definieren (Scope)
  → Erfassen: Geschäftsprozesse, IT-Systeme, Räume, Kommunikationsverbindungen
  → Netzplan erstellen
  Werkzeug: BSI-Tool VERINICE oder ISMS-Tool (z.B. HiScout)

Schritt 3: SCHUTZBEDARF
  → Je Zielobjekt: Vertraulichkeit / Integrität / Verfügbarkeit
  → Skala: Normal / Hoch / Sehr hoch
  → Maximumprinzip: höchster Schutzbedarf eines Prozesses zieht Systeme nach oben

Schritt 4: MODELLIERUNG
  → Bausteine aus IT-GS-Kompendium den Zielobjekten zuordnen

Schritt 5: IT-GRUNDSCHUTZ-CHECK
  → Anforderungen je Baustein prüfen: Ja / Nein / Entbehrlich
  → Dokumentation der Umsetzung

Schritt 6: RISIKOANALYSE (bei erhöhtem Schutzbedarf)
  → Für Zielobjekte mit hohem/sehr hohem Schutzbedarf
  → BSI-Methode: Gefährdungsübersicht, Eintrittshäufigkeit × Schadenshöhe

Schritt 7: RISIKOBEHANDLUNG
  → Maßnahmenkatalog, Umsetzungsplanung, Restrisiko

Schritt 8: KONSOLIDIERUNG + UMSETZUNG
  → Maßnahmenplan, Budgetierung, Controlling
```

---

## Verschlusssachen (VS-NfD) IT-Sicherheit

### VS-NfD-Grundlagen

```
Geheimhaltungsgrade (Deutschland):
VS-NfD    — Nur für den Dienstgebrauch (niedrigste Stufe)
VS-V      — Verschlusssache Vertraulich
VS-G      — Verschlusssache Geheim
VS-StG    — Verschlusssache Streng Geheim (höchste Stufe)

VS-NfD — Relevanz für Kommunen und Behörden:
- Häufigste Stufe in Bundesbehörden und beliehenen Stellen
- Anforderungen definiert in: Allgemeine Verwaltungsvorschrift des BMI für VS (VSA)
- IT-System-Anforderungen: BSI TR-03104 (VS-NfD-IT-Systeme)

Typische VS-NfD-Informationen:
- Personenbezogene Sicherheitsüberprüfungen
- Polizeiliche Lageinformationen
- Bestimmte Sozialdaten, Steuerdaten
- Interna Behördenkommunikation bestimmter Vertraulichkeit

Zugelassene IT-Produkte für VS-NfD (BSI-Produktliste):
- SINA (Sichere Inter-Netzwerk-Architektur) — Secunet
- Trusted VPN (BSI-zugelassen)
- VS-NfD-zugelassene Thin Clients
- Kryptierte Mobiltelefone (z.B. SiMKo 3)
```

### VS-NfD IT-Anforderungen

```
Grundprinzipien für VS-NfD-IT:
1. Nur BSI-zugelassene oder -freigegebene Produkte
2. Strikte Zugangssteuerung (Sicherheitsüberprüfung der Nutzer, § 9 SÜG)
3. Räumliche Sicherheit: VS-Registratur, abschließbare Schränke
4. Protokollierung aller Zugriffe auf VS-Daten
5. Kein Cloud-Speicher für VS-Daten (Ausnahme: BSI-zugelassene Lösungen)
6. Vernichtung: Nur nach VSA-Vorgaben (Schredder, nicht Papierkorb)
7. Transport: nur in verschlossenem Umschlag, keine ungesicherte Email

Netzwerk für VS-NfD:
- Physisch getrenntes Netz oder kryptografisch getrenntes Netz (SINA)
- SINA Workstation: Software-Client für VS-NfD-Netze auf normalem Endgerät
- SINA L3-Box: Hardware-Verschlüsselung für VS-NfD-Netzübergänge

E-Mail für VS-NfD:
- Keine normale Email für VS-NfD-Inhalte
- Zugelassene Lösungen: BSI-zertifizierte Email-Gateway + SMIME mit VS-NfD-PKI
- DE-Mail: NICHT für VS-NfD geeignet
```

---

## EVB-IT — IT-Vertragsrecht öffentliche Hand

### Vertragstypen-Übersicht

```
EVB-IT (Ergänzende Vertragsbedingungen für die Beschaffung von IT-Leistungen):
Herausgegeben durch Bundesministerium des Innern (BMI) / KBSt

Vertragstypen und Anwendungsfälle:

EVB-IT System         — Kauf + Lieferung von IT-Systemen (Hard- + Software)
EVB-IT Dienstleistung — IT-Dienstleistungen (Betrieb, Wartung, Support)
EVB-IT Erstellung     — Entwicklung individueller Software
EVB-IT Pflege S       — Pflege von Standardsoftware (Updates, Fehlerbehebung)
EVB-IT Instandhaltung — Hardware-Wartung
EVB-IT Überlassung S  — Nutzungsrecht Standardsoftware (Lizenzierung)
EVB-IT Cloud          — Cloud-Dienste (SaaS, PaaS, IaaS) [seit 2021]
EVB-IT Systemlieferung— Lieferung, Montage, Installation

Wichtige IT-Sicherheitsklauseln in EVB-IT:

§ Datenschutz und Vertraulichkeit:
  → Auftragnehmer ist zur Vertraulichkeit verpflichtet
  → Daten dürfen nur für Vertragszwecke genutzt werden
  → Bei personenbezogenen Daten: AVV nach DSGVO Art. 28 erforderlich

§ Mängelhaftung / Gewährleistung:
  → Sicherheitsmängel: Nachbesserungspflicht
  → Wesentliche Sicherheitslücken: Rücktrittsrecht möglich

§ Unterauftragsvergabe:
  → Auftraggeber muss Unterauftragnehmer genehmigen
  → Sicherheitsanforderungen gelten auch für Unterauftragnehmer

§ Quellcode-Hinterlegung (EVB-IT Erstellung):
  → Pflicht zur Hinterlegung bei akkreditierter Stelle (z.B. Escrow)
  → Wichtig für langfristige IT-Systeme in Behörden
```

### IT-Beschaffung Sicherheitsanforderungen

```
Typische IT-Sicherheitsanforderungen in Vergabeverfahren (Leistungsverzeichnis):

TECHNISCH:
- BSI-Grundschutz-konforme Konfiguration
- Aktuelle Sicherheitsupdates bei Lieferung
- Keine End-of-Life-Software/Betriebssysteme
- Sicherheitsrelevante Konfigurationsparameter dokumentiert
- SAST/DAST-Nachweise für Individualsoftware (EVB-IT Erstellung)
- SBOM für alle Softwarekomponenten
- Penetrationstest-Nachweis für kritische Systeme

PROZESSUAL:
- Verpflichtung auf IT-Grundschutz oder ISO 27001
- Meldepflicht bei Sicherheitsvorfällen (< 24h)
- Kooperationspflicht bei forensischen Untersuchungen
- Auditrecht des Auftraggebers
- Zertifikatsnachweis (BSI IT-Grundschutz oder ISO 27001)

DATENSCHUTZ (DSGVO):
- Auftragsverarbeitungsvertrag (AVV) als Anlage zum EVB-IT
- Technische und organisatorische Maßnahmen (TOM) dokumentiert
- Löschkonzept für personenbezogene Daten am Vertragsende
- Sub-Auftragsverarbeiter müssen benannt und genehmigt sein

CLOUD (EVB-IT Cloud):
- Verarbeitungsort: nur EU/EWR (oder angemessenes Datenschutzniveau)
- C5-Testat oder SOC 2 Typ II bevorzugt
- BSI C5 (Cloud Computing Compliance Criteria Catalogue): Goldstandard DE
- Datenlöschung nach Vertragsende: zertifizierter Nachweis
```

---

## eGovernment-Sicherheit

### Online-Services und Portal-Sicherheit (OZG)

```
OZG-Sicherheitsanforderungen für Online-Dienste:

Authentifizierung Bürger:
- eID (Online-Ausweisfunktion): höchste Vertrauensstufe (eIDAS LoA high)
- BundID / MeinUnternehmenskonto: Bürger-/Unternehmensportal
- ELSTER-Zertifikat: Steuern, betriebliche Anwendungen
- Username/Passwort: nur für niedrige Vertrauensstufen

Verschlüsselung:
- HTTPS/TLS 1.2+: Pflicht für alle Online-Dienste
- Starke Cipher Suites (BSI TR-02102): ECDHE, AES-GCM
- BSI TR-03116 (eCard-API) für eID-Anwendungen
- HSTS, HPKP, CSP, X-Frame-Options: Pflicht-HTTP-Header

Sicherheitsarchitektur Bürgerportal:
- WAF (Web Application Firewall) vor allen öffentlichen Portalen
- DDoS-Schutz (Behörden sind beliebte DDoS-Ziele)
- Trennung: Fachverfahren-Backend darf nicht direkt erreichbar sein
- API-Gateway für Schnittstellen zu Fachverfahren
- Penetrationstest vor Go-Live (OWASP Testing Guide)

Barrierefreiheit + Sicherheit (BITV 2.0):
- CAPTCHA: barrierefrei gestalten (Audio-Alternative)
- Session-Timeouts: nicht zu kurz (Barrierefreiheit), nicht zu lang (Sicherheit)
```

### KRITIS öffentliche Verwaltung

```
Öffentliche Verwaltung als KRITIS-Sektor:

Sektor "Staat und Verwaltung" (IT-SiG 2.0 § 2 Abs. 10 BSIG):
- Bundesministerien und nachgeordnete Behörden
- Kritische Bundesbehörden (z.B. BSI, BKA, BND, Bundeswehr-IT)
- Parlamentsverwaltung

Kommunen:
- Aktuell: keine direkte KRITIS-Einstufung für Kommunen
- ABER: zunehmend NIS2-Erweiterung auf "public administration" geplant
- BSI-Empfehlung: Kommunen sollten IT-Grundschutz freiwillig umsetzen
- Ländervorgaben variieren (Bayern: BAY-CISO, NRW: Kommunaler Masterplan)

Zuständigkeiten Bund/Länder/Kommunen:
- Bundesbehörden: BSI ist zentrale IT-Sicherheitsbehörde
- Landesbehörden: Landes-CIO/CISO, Landesrechenzentren
- Kommunen: Kommunale IT (z.B. AKDB in Bayern, Dataport in Norddeutschland)
  → Oft keine eigene IT-Sicherheitsorganisation → Dienstleistereinkauf
```

---

## Spezifische Bedrohungen öffentliche Verwaltung

### Aktuelle Angriffsmuster

```
TOP-ANGRIFFE auf Behörden (BSI Lagebericht, aktuelle Erkenntnisse):

1. RANSOMWARE gegen Kommunen (★★★★★):
   Bekannte Vorfälle: Landkreis Anhalt-Bitterfeld (2021), Potsdam (2022),
   Kaiserslautern (2024), Kerpen, Suhl...
   - Einfallstor: meist Phishing oder VPN-Schwachstelle
   - Folge: Monatelanger Ausfall von Bürgerservices
   - Besonderes Risiko: Kommunen oft keine IT-Sicherheits-Kompetenz vor Ort

2. SUPPLY CHAIN (IT-Dienstleister der öffentlichen Hand):
   - Angriff auf kommunalen IT-Dienstleister trifft viele Kommunen gleichzeitig
   - Beispiel: Angriff auf ITEOS (Baden-Württemberg)

3. SPIONAGE / APT (Bundesbehörden):
   - Ziel: Regierungsinformationen, VS-NfD-Daten, Verhandlungspositionen
   - Akteure: APT28 (Russland), APT40 (China), weitere staatliche Gruppen
   - BSI-Zielgruppe: Bundesministerien, Nachrichtendienste, Sicherheitsbehörden

4. DDoS auf Bürgerportale (★★★☆☆):
   - Hacktivisten-Angriffe auf öffentliche Online-Dienste
   - Oft politisch motiviert (Kriegszusammenhang)

5. PHISHING auf Verwaltungsmitarbeitende:
   - Ziel: VPN-Zugangsdaten, Office 365 / Webmail-Konten
   - Besonders: Täuschung mit BSI-/BKA-Branding ("Ihr System ist infiziert")
```

### IR-Playbook Ransomware Kommunalverwaltung

```
SOFORT (0-2 Stunden):
1. KRISENSTAB: Bürgermeister/Landrat + IT-Leitung + Datenschutzbeauftragter
2. ISOLATION: Betroffene Systeme vom Netz trennen
   → Finanzverfahren, Einwohnermeldeamt, Führerschein, Kita-Portal prioritär
3. NOTBETRIEB:
   → Papierbasierte Bearbeitung (Vordrucke vorhalten!)
   → Servicerufnummer für Bürger einrichten
   → Presse-Statement vorbereiten: "Technische Störung" (keine Details!)

MELDEKETTE:
4. BSI Lagezentrum: 0800 274 1000 (wenn KRITIS oder erhebliche Störung)
5. Landesdatenschutzbehörde (72h bei Datenpanne)
6. Landesamt für Verfassungsschutz (bei Staatsakteuren)
7. Polizei Cybercrime (LKA): Strafanzeige empfehlen
8. Kommunaler IT-Dienstleister (AKDB, Dataport etc.) einschalten
9. BSI-Unterstützung anfragen (kostenlos für Behörden!)
10. KFE (Kommunale IT-Forensik-Ersthelfer) — einige Länder bieten an

MITTELFRISTIG:
→ KEIN Lösegeld zahlen ohne Rücksprache mit BSI/BKA
→ Backup-Integrität prüfen BEVOR Restore
→ Root-Cause vor Wiederaufbau klären
→ Kommunikation: BSI bietet Unterstützung bei Pressearbeit an
```

---

## Compliance-Checkliste Öffentliche Verwaltung

```markdown
## IT-Sicherheit Kommunalverwaltung — Checkliste

### Basismaßnahmen (sofort umsetzbar)
- [ ] ISB (Informationssicherheitsbeauftragter) benannt
- [ ] IT-Sicherheitsleitlinie vorhanden und bekannt
- [ ] Kritische Systeme inventarisiert
- [ ] Backup-Strategie 3-2-1, Restore-Tests quartalsweise
- [ ] Backup offline / air-gapped (Ransomware-Schutz)
- [ ] MFA für alle Fernzugänge (VPN, RDP, OWA)
- [ ] Patch-Management: kritische Patches ≤ 72h, Standard ≤ 4 Wochen
- [ ] EDR auf allen Endpunkten
- [ ] Email-Sicherheit: SPF, DKIM, DMARC, Anti-Phishing
- [ ] Notfallkonzept: wer entscheidet im Angriffsfall?

### BSI IT-Grundschutz
- [ ] Basis-Absicherung umgesetzt (Minimum für alle Kommunen)
- [ ] Standard-Absicherung gestartet (für Kommunen ab ~50.000 EW empfohlen)
- [ ] ISB-Schulung (BSI IT-Grundschutz-Praktiker oder höher)
- [ ] VERINICE oder gleichwertiges ISMS-Werkzeug im Einsatz
- [ ] Jährlicher IT-Sicherheitsbericht an Verwaltungsführung

### Bürger-Online-Dienste (OZG)
- [ ] HTTPS auf allen Online-Diensten (TLS 1.2 mind., 1.3 bevorzugt)
- [ ] HTTP-Sicherheits-Header gesetzt (HSTS, CSP, X-Frame-Options)
- [ ] Penetrationstest vor Go-Live jedes Online-Dienstes
- [ ] eID-Anbindung: BSI TR-03130 Konformität

### Datenschutz (DSGVO)
- [ ] DSB bestellt (intern oder extern)
- [ ] Verzeichnis der Verarbeitungstätigkeiten (VVT) vollständig
- [ ] AVV mit allen IT-Dienstleistern
- [ ] Meldeprozess Datenpanne: DSB → Behördenleitung → LDA 72h

### Beschaffung (EVB-IT)
- [ ] EVB-IT-Vertragstyp korrekt gewählt
- [ ] IT-Sicherheitsanforderungen im Leistungsverzeichnis
- [ ] Cloud-Dienste: nur mit C5-Testat oder BSI-Freigabe
- [ ] AVV als Vertragsanlage bei Auftragsverarbeitung
```

---

## Weiterführende Ressourcen

- **BSI**: [IT-Grundschutz-Kompendium](https://www.bsi.bund.de/grundschutz) (kostenlos, jährlich aktualisiert)
- **BSI**: [IT-Grundschutz-Profile für Kommunen](https://www.bsi.bund.de/kommunen)
- **BSI**: [Lagebericht IT-Sicherheit in Deutschland](https://www.bsi.bund.de/lagebericht) (jährlich)
- **BSI**: [Allianz für Cybersicherheit (ACS)](https://www.allianz-fuer-cybersicherheit.de)
- **VITAKO**: Bundes-Arbeitsgemeinschaft der Kommunalen IT-Dienstleister
- **KGSt**: Kommunale Gemeinschaftsstelle — IT-Sicherheit in Kommunen
- **BMI**: [EVB-IT Vertragswerke](https://www.cio.bund.de/evb-it)
- **BayLDA**: Datenschutz für bayerische Behörden
- **AKDB**: IT-Sicherheitslösungen für bayerische Kommunen
- **Dataport**: IT-Dienstleister norddeutsche Länder und Kommunen
- **kommunale-it.de**: Ressourcenpool IT-Sicherheit Kommunen
