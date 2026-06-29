# Energy & Utility Security Reference

Spezialisierte Sicherheitsreferenz für Energieversorger, Netzbetreiber, Wasserwerke,
Gasversorgung und andere Versorgungsinfrastrukturen. Schwerpunkt: Deutschland/EU,
KRITIS-Regulierung, OT/IT-Konvergenz, sektorspezifische Standards.

---

## Regulatorische Landschaft Energie / Wasser (Deutschland / EU)

### Rechtsrahmen-Übersicht

| Rahmen | Geltungsbereich | Kernanforderungen |
|---|---|---|
| **KRITIS-Dachgesetz (BSIG § 8a ff.)** | KRITIS-Betreiber Energie, Wasser, Gas | Registrierung BSI, 24/7-Kontaktstelle, Nachweispflicht alle 2 Jahre, Meldepflicht |
| **IT-Sicherheitsgesetz 2.0 (2021)** | Erweitert KRITIS auf UBI (Unternehmen im besonderen Interesse) | BSI-Angriffserkennung Pflicht, Attack Detection Systems |
| **EnWG § 11 Abs. 1a** | Strom- und Gasnetzbetreiber | IT-Sicherheitskatalog der Bundesnetzagentur |
| **IT-Sicherheitskatalog BNetzA (Strom/Gas)** | Übertragungsnetzbetreiber, Verteilernetzbetreiber | ISO 27001 Zertifizierung, ISMS Pflicht |
| **IT-Sicherheitskatalog BNetzA (Wasser)** | Wasserversorgungsnetze | Analoge Anforderungen Strom/Gas |
| **NIS2 (EU 2022/2555)** | Energie als Essential Entity | Meldepflichten, Art. 21 Mindestmaßnahmen, Lieferkettenrisiko |
| **NIS2 Umsetzungsgesetz (NIS2UmsuCG)** | Deutsche Umsetzung, geplant 2024/2025 | Sektorale Erweiterung, strengere Sanktionen |
| **IEC 62351** | Cybersecurity für Energiesystemkommunikation | Protokollsicherheit: IEC 60870, IEC 61850, DNP3, ICCP |
| **IEC 62443** | Industrielle Automatisierungs- und Steuerungssysteme (IACS) | Defense-in-Depth, Zones & Conduits, Security Levels SL0-SL4 |
| **BDEW-Whitepaper** | Anforderungen an sichere Steuerungs- und Telekommunikationssysteme | Branchenstandard DE Energie, ergänzt IEC 62443 |
| **TAR (Technische Anschlussregeln)** | VDE-FNN, VDE-AR-N 4110/4120 | Anforderungen an Steuerungsinfrastruktur |

### KRITIS-Schwellenwerte Energie / Wasser

```
Sektor ENERGIE:
- Stromerzeugung:      ≥ 420 MW installierte Leistung → KRITIS
- Stromübertragung:    Übertragungsnetzbetreiber (ÜNB) → immer KRITIS
- Stromverteilung:     ≥ 3,7 Mio. Anschlüsse → KRITIS
- Gaserzeugung/-speich: ≥ bestimmte Durchleitungsmengen → KRITIS
- Gasverteilung:       ≥ 520.000 Anschlüsse → KRITIS
- Kraftstoff/Heizöl:   ≥ 3,2 Mio. t Jahresumschlag → KRITIS

Sektor WASSER:
- Trinkwasserversorgung: ≥ 500.000 Einwohner versorgt → KRITIS
- Abwasserentsorgung:    ≥ 500.000 Einwohner → KRITIS

UBI (Unternehmen im besonderen öffentlichen Interesse, IT-SiG 2.0):
- Rüstungsunternehmen, Unternehmen volkswirtschaftlicher Bedeutung
- Verpflichtung zur Systemen zur Angriffserkennung (SAK/IDS) → BSI § 8a Abs. 3
```

### IT-Sicherheitskatalog Bundesnetzagentur (Strom/Gas)

```
Pflichten für Strom-/Gasnetzbetreiber (§ 11 Abs. 1a EnWG):

1. ISMS nach ISO/IEC 27001:
   - Scope: Alle Systeme die für Netzsteuerung/Netzbetrieb relevant
   - Zertifizierung durch akkreditierte Stelle
   - Erstzertifizierung + Rezertifizierung alle 3 Jahre

2. Schutzprofilanpassung für Energie:
   - ISO 27001 + IEC 62443 + BDEW-Whitepaper als Referenz
   - Besondere Anforderungen: Leitsysteme, SCADA, Fernwirktechnik

3. Nachweis gegenüber BNetzA:
   - Zertifikat einreichen
   - Abweichungen begründen (Comply or Explain)

4. Meldepflichten:
   - Erhebliche Störungen → BSI (analog KRITIS)
   - Stromausfall-Relevanz → Bundesnetzagentur

Betroffene Systeme (typisch):
- Leitsystem / EMS (Energy Management System)
- SCADA / Fernwirksystem
- Umspannwerk-Steuerung
- Messdatensysteme (Smart Meter Gateway, SMGW)
- GIS (Geographisches Informationssystem Netz)
- Netzberechnungssysteme
```

---

## IEC 62351 — Sicherheit für Energieprotokolle

### Übersicht der Normenteile

```
IEC 62351 — Sicherheitsstandard für Energiesystem-Kommunikation:

Part 1:  Introduction and Security Issues
Part 2:  Glossary of Terms
Part 3:  Communication Network and System Security — TCP/IP Profiles
         → TLS-Anforderungen für TCP/IP-basierte Protokolle in der Energie
Part 4:  Profiles Including MMS (Manufacturing Message Specification)
Part 5:  Security for IEC 60870-5 and Derivatives
         → DNP3, IEC 60870-5-101/104: Authentifizierung, MAC
Part 6:  Security for IEC 61850 Profiles
         → GOOSE, Sampled Values (SV), MMS-Sicherheit
Part 7:  Network and System Management (NSM) Data Object Models
Part 8:  Role-Based Access Control for Power Systems
         → RBAC-Modell speziell für Energiesysteme
Part 9:  Cyber Security Key Management for Power Systems
Part 10: Security Architecture Guidelines
Part 11: Key Management Extensions
Part 14: Cyber Security for Systems Using Profiles of HTTPS
```

### Praktische Umsetzung IEC 62351

```
Protokoll          | IEC 62351 Teil | Maßnahmen
-------------------|----------------|------------------------------------------
IEC 60870-5-104    | Part 5         | Challenge-Response Auth, HMAC, T104-Sec
IEC 61850 MMS      | Part 4/6       | TLS 1.2+, Zertifikate, RBAC
IEC 61850 GOOSE    | Part 6         | HMAC-SHA256 Authentication Tag
DNP3               | Part 5         | DNP3 Secure Authentication v5 (SAv5)
ICCP / TASE.2      | Part 4         | TLS + Zertifikate
Modbus             | (nicht in Std) | VPN-Tunnel oder proprietäre Absicherung
OPC UA             | Part 14        | OPC UA Security Modes (TLS, Zertifikate)

WICHTIG:
- Viele Altsysteme unterstützen IEC 62351 NICHT
- Retrofit-Option: Kryptografische Appliances (Honeywell, Siemens, ABB)
- Alternative: Ende-zu-Ende-VPN als Kompensationskontrolle
- Protokollkonverter mit Sicherheitsfunktionen (Data Concentrator)
```

---

## BDEW-Whitepaper — Sicherheitsanforderungen für Steuersysteme

### Anforderungskategorien

```
BDEW-Whitepaper "Anforderungen an sichere Steuerungs- und Telekommunikationssysteme"
(Aktuelle Version: 2.0+, gemeinsam mit österreichischem E-CONTROL)

Kapitel-Übersicht und Kernanforderungen:

1. ORGANISATORISCHE ANFORDERUNGEN:
   - Informationssicherheits-Management (ISMS)
   - Sicherheitsbeauftragter (OT-Sicherheit separat von IT!)
   - Lieferanten-/Dienstleistermanagement
   - Schulung und Sensibilisierung (OT-spezifisch)

2. TECHNISCHE ANFORDERUNGEN — SYSTEMSICHERHEIT:
   - Härtung aller Steuerungskomponenten
   - Patchmanagement (OT-spezifisch, mit Herstellerfreigabe)
   - Whitelist-Anwendungssteuerung (statt Blacklisting)
   - Deaktivierung nicht benötigter Dienste und Ports
   - Physischer Schutz: Zugangsschutz Umspannwerke, Stationen

3. NETZWERKSICHERHEIT:
   - Segmentierung: Büronetz / OT-Netz / Fernwirknetz / Internet
   - Firewall an allen Übergängen
   - DMZ für Datenaustausch zwischen IT und OT
   - Keine direkte Verbindung OT-Netz zu Internet
   - Fernzugang: nur über gesicherte Verbindungen (VPN, Jump-Host)
   - Protokollfilterung: nur notwendige OT-Protokolle erlauben

4. IDENTITÄTSMANAGEMENT:
   - Individuelle Benutzerkonten (keine gemeinsamen Accounts)
   - MFA für Fernzugänge
   - Minimales Berechtigungsprinzip
   - Protokollierung aller administrativen Zugriffe

5. ÜBERWACHUNG UND DETEKTION:
   - Protokollierung aller sicherheitsrelevanten Ereignisse
   - OT-spezifisches Monitoring (passiv, nicht-intrusiv)
   - Angriffserkennung (IDS/IPS) an IT/OT-Übergängen
   - SOC-Integration oder OT-spezifisches NOC/SOC

6. NOTFALLMANAGEMENT:
   - Business Continuity Plan für Versorgungsunterbrechungen
   - Notbetriebskonzept: manuelle Steuerung möglich?
   - Backup und Wiederherstellung von Steuerungssystemen
   - Regelmäßige Übungen (inkl. Black-Start-Szenario)
```

---

## Architekturreferenz: IT/OT-Segmentierung Energieversorger

```
INTERNET / EXTERNE NETZE
        │
        ▼
[DMZ Extern: B2B-Kommunikation, Marktdaten, EDI]
        │
[Firewall 1 — Next-Gen, Deep Packet Inspection OT-Protokolle]
        │
CORPORATE IT NETZ (Büro, ERP, Email, HR)
        │
[Firewall 2 — Strenge Regeln: Default Deny von IT → OT]
        │
OT-NETZ / PROZESSNETZ
        │
        ├── LEITEBENE (Level 3):
        │   SCADA-Server, EMS, Historian, OT-SIEM
        │
        ├── STEUERUNGSEBENE (Level 2):
        │   RTU/IED-Kommunikation, Protokoll-Gateway
        │
        └── FELDEBENE (Level 1/0):
            RTU, IED, SPS, Sensoren, Aktoren

FERNWIRKNETZ (separat, verschlüsselt):
        │
        ├── Umspannwerke (UAW)
        ├── Trafostationen
        └── Schaltanlagen

MANAGEMENT-NETZ:
        └── Jump-Host für OT-Fernzugang, 2FA Pflicht, Session-Recording

Wichtige Prinzipien:
- Einbahnstraßen-Datenfluss bevorzugen: OT → IT (nicht umgekehrt)
- Data Diode für kritische unidirektionale Übertragungen (Historian → IT)
- Fernzugang IMMER über Bastion/Jump-Host mit Session-Recording
- Keine USB-Ports in OT-Umgebungen (physisch deaktivieren oder versiegeln)
```

---

## Smart Meter / SMGW-Sicherheit

```
Smart Meter Gateway (SMGW) — BSI-Anforderungen (BSI TR-03109):

Sicherheitsarchitektur:
- SMGW als zentrale Sicherheitskomponente
- BSI-Zertifizierung des SMGW (Common Criteria EAL4+)
- Separate Kommunikationskanäle: WAN (Lieferant/Netzbetreiber) / HAN (Haushalt) / LAN (Messsystem)

Schlüsselmanagement:
- PKI-basiert: Bundesamt für Sicherheit in der Informationstechnik als Root-CA
- Zertifikatsverwaltung über SMGW-Administrator (SMGW-Admin)
- Automatische Zertifikatserneuerung

Typische Sicherheitsrisiken:
- Firmware-Manipulation (nur BSI-signierte Firmware erlaubt)
- Kommunikationsabruch zur TSS (Trusted Service Shell)
- Fehlkonfiguration der Tarifanwendungsfälle
- Seitenkanalangriffe auf Kryptoprozessor

Härtung SMGW-Infrastruktur:
- SMGW-Admin-Server: gehärtet, ISO 27001 zertifiziert
- WAN-Kommunikation: TLS 1.3 + gegenseitige Zertifikatsauthentifizierung
- Monitoring: Kommunikationsanomalien, unerwartete Tarifänderungen
```

---

## Incident Response Energie / KRITIS

### Stromausfall durch Cyberangriff

```
SOFORT (0-1 Stunde):
1. KRISENSTAB aktivieren (Leitungsebene, OT-Betrieb, IT-Sicherheit, Kommunikation)
2. BETROFFENE SYSTEME identifizieren (Leitsystem, SCADA, Fernwirk)
3. ISOLATION: Kompromittierte OT-Systeme vom Netz trennen
   ACHTUNG: Physischen Notbetrieb sicherstellen BEVOR Isolation!
   → Manuelle Steuerung Umspannwerke aktivieren
   → Notfallkommunikation zu Schaltwartenteams aufbauen
4. MELDEKETTE:
   - BSI Lagezentrum: 0800 274 1000 (kostenlos, 24/7)
   - Bundesnetzagentur (bei netzbetriebsrelevanten Störungen)
   - Koordinierungsstelle Cyber (KSC) der Branche
   - BDEW Meldestelle (freiwillig, branchenintern)

KURZZEITIG (1-4 Stunden):
5. FORENSIK (parallel zu Notbetrieb):
   - Netzwerk-Logs sichern (Firewall, SCADA-Historian)
   - RAM-Dumps kompromittierter Systeme
   - IED/RTU-Ereignisprotokolle sichern
6. EXTERNE UNTERSTÜTZUNG:
   - BSI: Unterstützung auf Anfrage
   - Spezialisierte OT-Forensik-Anbieter (Claroty, Dragos, Nozomi)
   - Hersteller-Hotline (Siemens, ABB, Schneider Electric, GE)

MITTELFRISTIG (4-72 Stunden):
7. WIEDERHERSTELLUNG (Clean-Room):
   - Prüfe Backups auf Integrität vor Restore
   - Reihenfolge: Primäres Leitsystem → Fernwirksystem → Feldbusebene
   - Netzschutz vor Zuschaltung sicherstellen
   - Schrittweise Wiederinbetriebnahme mit Schaltplänen

BSI-MELDEPFLICHT (§ 8b BSIG):
- Erhebliche Störung = Beeinträchtigung der Versorgungssicherheit
- Erstmeldung: unverzüglich, spätestens 24h nach Kenntnis
- Abschlussbericht: nach Behebung
```

---

## Compliance-Checkliste Energie / KRITIS

```markdown
## KRITIS Energie — Compliance-Checkliste (BSI / IT-SiG 2.0)

### KRITIS-Grundpflichten
- [ ] Registrierung beim BSI als KRITIS-Betreiber
- [ ] Benennung einer Kontaktstelle (24/7 erreichbar)
- [ ] Nachweis angemessener IT-Sicherheit alle 2 Jahre
  (Zertifizierung nach ISO 27001 oder Sicherheitsaudit durch Dritte)
- [ ] Meldeverfahren für erhebliche Störungen etabliert

### IT-Sicherheitskatalog BNetzA (Strom/Gas)
- [ ] ISMS nach ISO 27001 implementiert
- [ ] ISO 27001 Zertifizierung gültig und aktuell
- [ ] Scope ISMS deckt alle kritischen Systeme ab
- [ ] Rezertifizierung alle 3 Jahre geplant
- [ ] Abweichungen mit Maßnahmen dokumentiert (Comply or Explain)

### OT/ICS-Sicherheit
- [ ] OT-Asset-Inventar vollständig (IED, RTU, SPS, HMI, SCADA)
- [ ] IT/OT-Segmentierung implementiert (Firewall, DMZ)
- [ ] Kein direkter Internetzugang aus OT-Netz
- [ ] Fernzugang nur über gesicherte Verbindung + MFA
- [ ] Patchmanagement OT: Herstellerkoordination, Testprozess
- [ ] Passivscan / OT-Monitoring implementiert (Claroty/Nozomi/Dragos)
- [ ] USB-Ports in OT physisch deaktiviert oder kontrolliert
- [ ] Whitelist-Anwendungssteuerung auf OT-Systemen

### IEC 62443 Umsetzung
- [ ] Zones und Conduits definiert und dokumentiert
- [ ] Security Level (SL) je Zone festgelegt
- [ ] Risikoanalyse nach IEC 62443-3-2 durchgeführt
- [ ] Maßnahmen gegen Target SL implementiert

### Protokollsicherheit (IEC 62351)
- [ ] IEC 60870-5-104: Secure Authentication (SAv5) oder VPN-Kompensation
- [ ] IEC 61850 GOOSE: Authentication Tag aktiviert
- [ ] DNP3: Secure Authentication v5 aktiviert
- [ ] Veraltete Protokolle (Modbus/TCP ungesichert): Kompensationsmaßnahmen

### SAK / Angriffserkennung (IT-SiG 2.0 § 8a Abs. 3)
- [ ] System zur Angriffserkennung (SAK) implementiert
- [ ] SAK erkennt Angriffe auf IT UND OT
- [ ] Eignung des SAK gegenüber BSI nachgewiesen
- [ ] Protokollierung und Auswertung der SAK-Alarme

### Business Continuity
- [ ] BCP für Versorgungsunterbrechung durch Cyberangriff vorhanden
- [ ] Notbetriebskonzept: manuelle Steuerung möglich und geübt
- [ ] Backup-Strategie: OT-Konfigurationen, Leitsystem-Snapshots
- [ ] Jährliche BCP-Übung (inkl. Szenario Cyberangriff auf Leitsystem)
- [ ] Koordination mit anderen KRITIS-Betreibern (gegenseitige Abhängigkeiten)
```

---

## Weiterführende Ressourcen

- **BSI**: [Industrielle Steuerungs- und Automatisierungssysteme](https://www.bsi.bund.de/ics)
- **BSI ICS-Security Kompendium**: Vollständige Umsetzungshilfe OT-Sicherheit
- **BDEW**: [Whitepaper Steuerungs- und Telekommunikationssysteme](https://www.bdew.de)
- **Bundesnetzagentur**: [IT-Sicherheitskatalog](https://www.bundesnetzagentur.de)
- **IEC 62443**: Normreihe (kostenpflichtig via IEC/DIN)
- **IEC 62351**: Normreihe Energiesystem-Kommunikationssicherheit
- **CISA**: ICS-CERT Advisories — sektorspezifisch filterbar
- **Dragos**: OT-Threat Intelligence, ICS-spezifische Angriffskampagnen
- **Nozomi Networks**: OT/IoT Asset Discovery, Anomalieerkennung
- **Claroty**: Industrielle Netzwerküberwachung
- **E-ISAC**: Electricity Information Sharing and Analysis Center
