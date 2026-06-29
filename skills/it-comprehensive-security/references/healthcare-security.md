# Healthcare Security Reference

Spezialisierte Sicherheitsreferenz für Gesundheitseinrichtungen: Krankenhäuser, MVZ, Reha-Kliniken,
Labore, Medizingerätehersteller und Krankenversicherungen. Alle regulatorischen Bezüge auf den
deutschen/europäischen Rechtsrahmen ausgerichtet.

---

## Regulatorische Landschaft (Deutschland / EU)

### Primäre Rechtsrahmen

| Rahmen | Scope | Kernpflichten |
|---|---|---|
| **DSGVO Art. 9** | Gesundheitsdaten = besondere Kategorie | Explizite Einwilligung, DSFA, DPO Pflicht |
| **DSGVO Art. 32** | Technische + organisatorische Maßnahmen | Pseudonymisierung, Verschlüsselung, Resilienz |
| **NIS2-Richtlinie** | Krankenhäuser ab 50 MA oder 10 Mio € | Meldepflicht 24h/72h, Mindestmaßnahmen Art. 21 |
| **SGB V § 75b** | KV-vertragsärztliche IT-Sicherheit | Technische Richtlinie BSI TR-03161 |
| **KHG / KHZG** | Krankenhausfinanzierung + Digitalisierung | Fördertatbestand 10: IT-Sicherheit verpflichtend |
| **KRITIS-Dachgesetz** | Kritische Infrastruktur Gesundheit | Meldepflichten, Sicherheitsstandards, Nachweispflichten |
| **MDR (EU 2017/745)** | Medizinprodukte inkl. Software (SaMD) | Cybersecurity als Produktsicherheit, Post-Market Surveillance |
| **IVDR (EU 2017/746)** | In-vitro-Diagnostika | Gleiche Cybersecurity-Anforderungen wie MDR |
| **EU AI Act** | KI in Medizinprodukten = High Risk | Konformitätsbewertung, Transparenz, menschl. Aufsicht |

### KRITIS-Schwellenwerte Gesundheit

```
Sektor Gesundheit (BSIG / KRITIS-Dachgesetz):
- Krankenhäuser: ≥ 30.000 vollstationäre Fälle/Jahr → KRITIS
- Labore: ≥ 1,5 Mio. Untersuchungen/Jahr → KRITIS
- Blutspendedienste: ≥ 100.000 Blutprodukte/Jahr → KRITIS
- Pharma: ≥ 50 Mio. definierte Tagesdosen/Jahr → KRITIS

Pflichten für KRITIS-Betreiber:
1. Registrierung beim BSI
2. Kontaktstelle 24/7
3. Nachweis angemessener Sicherheit alle 2 Jahre (Prüfung durch Dritte)
4. Meldepflicht erheblicher Störungen an BSI
```

### BSI-Orientierungshilfe Krankenhaus

Das BSI veröffentlicht spezifische Umsetzungshinweise:
- **B 1.0 Allgemeines**: Sicherheitsorganisation, ISMS-Aufbau
- **B 6.x Kommunikation**: Telematikinfrastruktur (TI), KIM, ePA
- **B 6.150 Krankenhausinformationssystem (KIS)**: HL7, DICOM, Schnittstellen
- Spezifische Bausteine für Medizingeräte, Laborsysteme, Radiologie (PACS/RIS)

---

## Kritische Systeme und Angriffsflächen

### Systemlandschaft typisches Krankenhaus

```
┌─────────────────────────────────────────────────────────────┐
│  KLINISCHE SYSTEME                                          │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌──────────────┐  │
│  │   KIS   │  │  PACS   │  │   RIS   │  │ Labor (LIS)  │  │
│  │ (SAP IS │  │(Dicom-  │  │(Radiol. │  │(HL7, LOINC)  │  │
│  │  -H/i.s.│  │ Server) │  │  Info.) │  │              │  │
│  └────┬────┘  └────┬────┘  └────┬────┘  └──────┬───────┘  │
│       └────────────┴────────────┴───────────────┘          │
│                    Klinisches Netz                          │
├─────────────────────────────────────────────────────────────┤
│  MEDIZINGERÄTE-NETZ (IoMT)                                  │
│  CT/MRT · Beatmung · Infusion · EKG · Monitoring           │
│  → Oft Legacy-OS (Win XP/7), kein Patching möglich         │
├─────────────────────────────────────────────────────────────┤
│  TELEMATIKINFRASTRUKTUR (TI)                                 │
│  Konnektor → ePA · eRezept · eAU · KIM · TI 2.0            │
├─────────────────────────────────────────────────────────────┤
│  OFFICE-NETZ / VERWALTUNG                                   │
│  ERP · Personalwesen · Einkauf · Email                      │
└─────────────────────────────────────────────────────────────┘
```

### Top-Angriffsvektoren Healthcare

| Vektor | Häufigkeit | Besonderheit |
|---|---|---|
| Ransomware via Phishing | ★★★★★ | Direkte Patientengefährdung möglich |
| VPN/RDP-Kompromittierung | ★★★★☆ | Telemedizin + Homeoffice vergrößern Fläche |
| Medizingeräte (IoMT) | ★★★☆☆ | Patchingverbot durch Hersteller, Legacy-OS |
| Supply Chain (SW-Lieferant) | ★★★☆☆ | Konzentrierte Angriffe auf KIS-Hersteller |
| Insider Threats | ★★★☆☆ | Neugier auf Prominentendaten, Datenschutz |
| TI-Konnektor | ★★☆☆☆ | Wachsende Fläche, gematik-Zulassung ≠ sicher |

---

## Medizingeräte-Sicherheit (IoMT)

### Inventarisierung

```bash
# Passives Nmap-Scanning (aktives Scanning kann Geräte beeinträchtigen!)
# Stattdessen: Passives Monitoring mit Claroty / Medigate / Nozomi

# Falls aktives Scanning nötig: NUR in Wartungsfenster, mit Gerätehersteller abgestimmt
nmap -sn 192.168.50.0/24 -oX iomt-inventory.xml  # Ping-Sweep only, keine Port-Scans!

# SNMP-basierte Inventarisierung (schonender)
snmpwalk -v2c -c public 192.168.50.x sysDescr
```

### Segmentierungsarchitektur IoMT

```
Empfehlung: Mikrosegmentierung nach Gerätetyp und Risikoprofil

ZONE 1 - High Risk Legacy (Win XP/7, nicht patchbar):
  → Isoliertes VLAN, kein Internet-Ausgang
  → Whitelisting via Next-Gen Firewall (nur benötigte Protokolle)
  → Virtual Patching via IPS-Signaturen

ZONE 2 - Klinische Geräte (patchbar, Herstellersupport):
  → Eigenes VLAN, KIS-Kommunikation via API-Gateway
  → Automatisches Patching im Wartungsfenster

ZONE 3 - Administrative Medizingeräte (z.B. Röntgen-Workstation):
  → Näher am Office-Netz, stärkere Kontrollen

Firewall-Regeln Grundprinzip:
- IoMT → KIS: Nur spezifische Ports (HL7 = 2575, DICOM = 104)
- KIS → IoMT: Nur für Auftrag/Rückgabe
- IoMT → Internet: VERBOTEN (außer explizit benötigte Hersteller-Updates)
- IoMT → IoMT: Default DENY
```

### SBOM und Patch-Management für Medizinprodukte

```
MDR/FDA-konforme SBOM-Anforderungen (ab 2024 verpflichtend):

1. Hersteller-Anforderungen:
   - Zyklische SBOM-Updates bei Komponentenänderungen
   - CVE-Monitoring für alle SBOM-Komponenten
   - Coordinated Vulnerability Disclosure (CVD) Prozess
   - Patchpflicht oder Kompensationsmaßnahmen dokumentieren

2. Betreiber-Pflichten:
   - SBOM vom Hersteller anfordern (vertraglich absichern)
   - Eigenes CVE-Monitoring gegen SBOM
   - Risikobasierte Entscheidung: Patchen vs. Isolieren vs. Abschalten
   - Dokumentation der Entscheidung (Nachweis gegenüber BSI/Behörden)

3. Tools:
   - SBOM-Formate: SPDX, CycloneDX
   - Monitoring: Dependency-Track, OWASP CycloneDX
   - CVE-Quellen: NVD, BSI-Advisories, ICS-CERT/CISA
```

---

## Telematikinfrastruktur (TI) Sicherheit

### TI-Architektur und Sicherheitsaspekte

```
Komponenten:
- Konnektor (gematik-zugelassen, in Einrichtung)
- eHealth-Kartenterminal (eHCT)
- Praxis-/Krankenhausverwaltungssystem (PVS/KIS) mit TI-Anbindung
- TI-Fachanwendungen: ePA, eRezept, eAU, KIM, NFDM, eMP

Sicherheitsmechanismen TI:
✓ Ende-zu-Ende-Verschlüsselung (PKI-basiert)
✓ Gegenseitige Authentifizierung aller Komponenten
✓ gematik-Zulassungsverfahren für Komponenten
✓ Separates, isoliertes Netz (kein Internet-Direktzugang)

Schwachstellen und Risiken:
⚠ Konnektor-Firmware (Updatepflicht beachten, Chaos Computer Club Analysen)
⚠ PVS/KIS-Integration: Schwachstellen in Schnittstellen
⚠ Phishing auf SMC-B-Karteninhaber (Praxisausweis)
⚠ TI 2.0 (Cloud-Konnektor): Neue Angriffsfläche durch Cloudanbindung
```

### TI-Härtungsmaßnahmen

```
1. Konnektor-Härtung:
   - Firmware immer aktuell halten (gematik-Updatepflicht)
   - Admin-Interface nur aus dediziertem Management-VLAN erreichbar
   - Standard-Passwörter SOFORT ändern
   - Logging aktivieren und SIEM-Integration

2. Netzwerk:
   - Konnektor in eigenem VLAN (kein direkter Zugang aus Office-Netz)
   - Firewall-Regel: Nur PVS/KIS darf mit Konnektor kommunizieren
   - Outbound: Nur TI-Netz (185.87.124.0/22 etc.) erlauben

3. Zugangssteuerung:
   - SMC-B und HBA immer in Kartenterminal, niemals in Konnektor-Webinterface
   - Zwei-Faktor für Konnektor-Administration
   - Auditlog für alle TI-Transaktionen
```

---

## Compliance-Checkliste Healthcare

### NIS2 + KRITIS Krankenhaus

```markdown
## NIS2 / KRITIS Krankenhaus — Compliance-Checkliste

### Governance & Organisation
- [ ] ISMS aufgebaut (ISO 27001 oder BSI-Grundschutz)
- [ ] Informationssicherheitsbeauftragter (ISB) benannt
- [ ] Datenschutzbeauftragter (DSB) benannt
- [ ] Sicherheitsrichtlinien genehmigt und kommuniziert
- [ ] Risikobewertung (jährlich) durchgeführt
- [ ] Geschäftsleitung über Cybersecurity-Risiken informiert (Haftung!)

### Technische Maßnahmen (NIS2 Art. 21)
- [ ] Multi-Faktor-Authentifizierung für alle kritischen Systeme
- [ ] Netzwerksegmentierung (klinisch / IoMT / Office / TI)
- [ ] Verschlüsselung Patientendaten at rest und in transit
- [ ] SIEM / Security Monitoring implementiert
- [ ] Vulnerability Management (quartalsweise Scan-Zyklen)
- [ ] Patch-Management-Prozess dokumentiert und aktiv
- [ ] Backup & DR: 3-2-1-1-0-Regel, regelmäßige Restore-Tests
- [ ] Endpoint Detection & Response (EDR) auf allen managebaren Systemen
- [ ] Email-Sicherheit (SPF, DKIM, DMARC, Anti-Phishing)

### Medizingeräte (IoMT)
- [ ] Vollständiges IoMT-Inventar vorhanden
- [ ] Netzwerksegmentierung IoMT implementiert
- [ ] SBOM von Herstellern angefordert
- [ ] Prozess für Patch/Kompensationsmaßnahmen definiert
- [ ] Passive Monitoring-Lösung (Claroty/Nozomi/Medigate) evaluiert

### Incident Response
- [ ] IR-Plan existiert und ist getestet (≥ jährliche Übung)
- [ ] 24/7-Kontaktstelle beim BSI registriert
- [ ] Meldeprozess: erhebliche Störung → BSI binnen 24h (Erstmeldung)
- [ ] Meldeprozess: Datenschutzverletzung → Datenschutzbehörde 72h
- [ ] Kommunikationsplan für Öffentlichkeit / Patienten vorhanden
- [ ] Forensik-Retainer mit externem Dienstleister abgeschlossen

### Telematikinfrastruktur
- [ ] Konnektor-Firmware aktuell
- [ ] TI-Netzwerksegmentierung implementiert
- [ ] TI-Zugang im Audit-Log erfasst
- [ ] SMC-B/HBA Zugangssteuerung dokumentiert

### Lieferkette
- [ ] Software-Lieferanten nach IT-Sicherheit bewertet
- [ ] AVV (Auftragsverarbeitungsvertrag) mit allen relevanten Dienstleistern
- [ ] Cloud-Dienste: DSGVO-konforme Verträge, EU-Hosting bevorzugt
- [ ] Notfallkonzept für Systemausfall kritischer Drittsoftware (KIS, PACS)
```

---

## Incident Response Healthcare

### Ransomware-Angriff auf Krankenhaus

```
SOFORT (0-2 Stunden):
1. ISOLATION: Betroffene Systeme vom Netz trennen (nicht ausschalten!)
   - KIS, PACS, Laborsysteme isolieren sobald Infektion bestätigt
   - Konnektor isolieren (TI schützen)
   
2. NOTBETRIEB aktivieren:
   - Papierbasierte Fallaufnahme
   - OP-Planung manuell
   - Medikamentengabe aus physischen Verordnungen
   - Rettungsstelle: Abmelden aus IVENA (kein Aufnahmestopp ohne Rücksprache mit Leitung)
   
3. MELDEKETTE:
   - Krankenhausleitung / Bereitschaft
   - BSI (0800 274 1000) — Erstmeldung innerhalb 24h Pflicht bei KRITIS
   - Landesdatenschutzbehörde (wenn Patientendaten betroffen) — 72h
   - Krankenversicherungen informieren (Abrechnungsunterbrechung)
   - Polizei (BKA / LKA Cybercrime) — Anzeige empfehlen

KURZZEITIG (2-24 Stunden):
4. FORENSIK:
   - RAM-Dumps vor Abschalten (Volatility)
   - Netzwerk-Logs sichern (Firewall, Proxy, DNS)
   - Externe Forensiker kontaktieren (Retainer nutzen)
   - KEINE Lösegeldzahlung ohne Rücksprache mit BSI/Behörden

5. KOMMUNIKATION:
   - Interne Kommunikation: Messengerdienst außerhalb des Krankenhaus-Netzes (Handy)
   - Pressemitteilung vorbereiten (Vorlage!)
   - Patienteninformation bei Datenpanne

MITTELFRISTIG (24-72 Stunden):
6. RESTORE:
   - Backups auf Infektion prüfen (Backup-Daten können befallen sein!)
   - Clean-Room-Wiederherstellung
   - Priorität: Intensivmedizin → OP → Notaufnahme → Station → Verwaltung

LESSONS LEARNED (nach Wiederherstellung):
7. Root-Cause-Analyse dokumentieren
8. BSI-Abschlussbericht (Pflicht bei KRITIS-Meldung)
9. ISMS aktualisieren
```

### DSGVO-Datenpanne Meldung (Healthcare)

```
Frist: 72 Stunden nach Kenntnis der Verletzung

Meldepflicht wenn: unbefugter Zugriff / Verlust / Vernichtung von Gesundheitsdaten

Inhalt der Meldung (Art. 33 DSGVO):
1. Art der Verletzung (Vertraulichkeit / Integrität / Verfügbarkeit)
2. Kategorien und ungefähre Zahl der betroffenen Personen
3. Kategorien der betroffenen Datensätze
4. Wahrscheinliche Folgen der Verletzung
5. Ergriffene und geplante Maßnahmen

Zuständige Behörden (DACH):
- Bayern: BayLDA (Ansbach)
- Baden-Württemberg: LfDI BW (Stuttgart)
- NRW: LDI NRW (Düsseldorf)
- Österreich: Datenschutzbehörde (Wien)
- Schweiz: EDÖB (Bern)

Betroffenenbenachrichtigung (Art. 34 DSGVO):
→ Pflicht wenn "voraussichtlich hohes Risiko für persönliche Rechte und Freiheiten"
→ Bei Gesundheitsdaten: Regel ist hohe Risikoeinschätzung → Benachrichtigung fast immer nötig
→ Ausnahme: Daten waren verschlüsselt und Schlüssel nicht kompromittiert
```

---

## Härtungsmaßnahmen KIS / PACS / HL7

### Krankenhausinformationssystem (KIS)

```
Häufig eingesetzte Systeme: SAP IS-H, i.s.h.med, Dedalus (Orbis), Nexus, Medico

Härtungsmaßnahmen allgemein:
1. Netzwerkisolierung: KIS-Server in eigenem VLAN
2. Firewall: Nur autorisierte Clients dürfen auf KIS-Ports zugreifen
3. Datenbankverschlüsselung: Oracle TDE / SQL Server TDE aktivieren
4. Audit-Logging: Alle Datenbankzugriffe auf Patientendaten loggen
5. Benutzerkonzept: Minimale Rechte, Rollenmodell (Arzt / Pflege / Verwaltung)
6. Sitzungsmanagement: Auto-Logout nach 5-10 Minuten Inaktivität
7. API-Sicherheit: HL7-FHIR-Endpunkte mit OAuth 2.0 absichern

HL7-spezifisch:
- HL7 v2: Mirth Connect / Rhapsody als Gateway mit TLS
- FHIR R4: SMART on FHIR für OAuth2/OIDC-basierte Zugriffskontrolle
- Payload-Validierung: Schema-Validierung für alle eingehenden HL7-Nachrichten
```

### PACS / DICOM

```
Typische Schwachstellen:
- DICOM-Port 104 oft offen ins Netz (Shodan: "DICOM" findet weltweit offene Server!)
- Keine Authentifizierung in altem DICOM-Standard (AE-Title ≠ Authentifizierung)
- DICOM-Metadaten enthalten oft Klarnamen, GBD, Diagnosen

Härtung:
1. DICOM-TLS (DICOM über TLS / DIMSE-TLS): Pflicht!
   Port 2762 statt 104 für TLS-DICOM
2. DICOMweb (WADO-RS/STOW-RS): HTTPS + OAuth2
3. Firewall: Port 104 nur zwischen bekannten AE-Titles erlauben
4. Pseudonymisierung: Patientendaten aus DICOM-Headern für Forschung entfernen
   Tools: DicomCleaner, Pixelmed, DICOM Anonymization Service

Pentest-Ansätze für DICOM:
- findscu / storescu (DCMTK-Toolset) für Enumeration
- Shodan: port:104 DICOM
- Überprüfung: C-FIND ohne Authentifizierung möglich?
```

---

## Awareness-Inhalte Healthcare (Deutsch)

### Schulungsthemen für Klinikmitarbeiter

```
Modul 1 - Phishing im Gesundheitswesen (30 min):
- Täuschende Absender (KIM-Nachrichten, "BSI-Warnung", "AOK-Abrechnung")
- Gefälschte eRezept-Nachrichten
- Was tun bei Verdacht? → IT-Hotline, nicht klicken, nicht löschen

Modul 2 - Datenschutz im Alltag (20 min):
- Keine Patientendaten per WhatsApp/privater Email
- Bildschirm sperren bei Verlassen des Arbeitsplatzes (Win+L)
- Patientenlisten nicht als PDF per Email ohne Verschlüsselung
- "Neugier" auf Promidaten = Datenpanne (Arbeitgeberkontrollen, Abmahnung)

Modul 3 - Passwörter und Zugänge (15 min):
- Keine Zugangsdaten weitergeben (auch nicht an Kollegin "die mal schnell schauen muss")
- Warum MFA im Krankenhaus?
- Was tun wenn Login nicht funktioniert → IT-Helpdesk, kein Passwort-Reset per Telefon

Modul 4 - Meldewege kennen (10 min):
- Wer ist der IT-Sicherheitsbeauftragte?
- Wie melde ich einen Vorfall? (Telefon / Ticketsystem)
- Lieber einmal zu viel melden als zu wenig
```

---

## Referenzarchitektur Healthcare Security

```
Internet
    │
    ▼
[WAF + DDoS-Schutz]
    │
    ▼
[Next-Gen Firewall Zone 1]
    │
    ├── DMZ: Webportal Patientenportal, VPN-Gateway, Email-Gateway
    │
[Next-Gen Firewall Zone 2]
    │
    ├── Office-Netz: Verwaltung, HR, Buchhaltung
    │   └── EDR + SIEM-Agent auf allen Endpunkten
    │
    ├── Klinisches Netz: KIS, PACS, RIS, LIS
    │   └── Striktes Benutzerrollenmodell, DB-Verschlüsselung
    │
    ├── IoMT-Netz: Medizingeräte (segmentiert nach Risikoklasse)
    │   └── Passives Monitoring (Claroty/Nozomi), Virtual Patching
    │
    ├── TI-Netz: Konnektor, eHCT
    │   └── Isoliert, nur PVS/KIS-Zugriff
    │
    └── Management-Netz: Server, Switches, Firewalls
        └── Jumphost-Pflicht, MFA, PAM (CyberArk/BeyondTrust)

SIEM: Alle Zonen loggen zentral → SOC / SIEM (Splunk/Sentinel/QRadar)
Backup: Isoliertes Backup-Netz, Air-Gap für kritische Backups
```

---

## Weiterführende Ressourcen

- **BSI**: [Ransomware und Gesundheitssektor](https://www.bsi.bund.de/krankenhaus)
- **gematik**: Technische Spezifikationen TI, Konnektor-Handbücher
- **KHG / KHZG**: Fördertatbestand 10 — IT-Sicherheit
- **HIMSS**: Healthcare Information Security Frameworks
- **ENISA**: Healthcare Threat Landscape Report (jährlich)
- **FDA Guidance**: Cybersecurity in Medical Devices (US-Referenz für MDR-Analogien)
- **IEC 80001**: Risikomanagement IT-Netzwerke mit Medizinprodukten
- **HL7 FHIR Security**: https://www.hl7.org/fhir/security.html
