# Finance Security Reference

Spezialisierte Sicherheitsreferenz für den Finanzsektor: Banken, Versicherungen, Zahlungsdienstleister,
FinTechs, Kapitalverwaltungsgesellschaften und Wertpapierfirmen. Regulatorischer Schwerpunkt auf
EU/DACH-Rahmenwerken mit Fokus auf DORA, BAIT, MaRisk und PCI DSS.

---

## Regulatorische Landschaft (Deutschland / EU)

### Primäre Rechtsrahmen

| Rahmen | Geltungsbereich | Kernanforderungen |
|---|---|---|
| **DORA (EU 2022/2554)** | Alle regulierten Finanzunternehmen ab 17.01.2025 | ICT-Risk, Incident Reporting, TLPT, Drittparteienmanagement |
| **MaRisk (BA 09/2017)** | Kreditinstitute (§ 25a KWG) | Risikomanagement, Auslagerungen, IT-Organisation |
| **BAIT (2021)** | Kreditinstitute | IT-Strategie, ISMS, Anwendungsentwicklung, Benutzerrechte |
| **VAIT (2021)** | Versicherungsunternehmen (VAG) | Analog BAIT für Versicherungen |
| **KAIT (2021)** | Kapitalverwaltungsgesellschaften (KAGB) | Analog BAIT für KVGs |
| **ZAIT (2021)** | Zahlungsdienstleister (ZAG) | Analog BAIT für Zahlungsdienstleister |
| **PCI DSS v4.0** | Alle mit Kartenzahlungsdaten | 12 Anforderungsbereiche, Penetrationstests |
| **PSD2 (EU 2015/2366)** | Zahlungsdienstleister | Strong Customer Authentication (SCA), Open Banking |
| **SWIFT CSCF** | SWIFT-Teilnehmer | Mandatory + Advisory Controls |
| **NIS2** | Banken + Finanzmarktinfrastrukturen | Meldepflichten, Mindestmaßnahmen |
| **DSGVO** | Alle Verarbeiter personenbezogener Daten | Datenschutz, Art. 32 TOM |

### DORA im Detail (ab 17. Januar 2025)

```
DORA gilt für:
✓ Kreditinstitute (§ 1 KWG)
✓ Zahlungsinstitute
✓ E-Geld-Institute
✓ Wertpapierfirmen
✓ Versicherungsunternehmen (EIOPA-Scope)
✓ Pensionskassen
✓ Kreditratingagenturen
✓ Kritische IKT-Drittdienstleister (von BaFin/ESA designiert)

5 Säulen DORA:

1. IKT-RISIKOMANAGEMENT (Art. 5-16)
   - ISMS mit vollständigem ICT Asset Register
   - Schutz, Erkennung, Eindämmung, Wiederherstellung
   - Business Continuity / DR mit RTO/RPO-Vorgaben

2. IKT-VORFALLSMANAGEMENT (Art. 17-23)
   - Klassifizierung: Erheblich vs. Nicht-erheblich
   - Meldung erheblicher Vorfälle:
     * Erstmeldung: 4 Stunden nach Klassifizierung (max. 24h nach Entdeckung)
     * Zwischenmeldung: 72 Stunden nach Erstmeldung
     * Abschlussbericht: 1 Monat nach Erstmeldung
   - Nationale Behörde: BaFin → ESMA/EBA/EIOPA

3. DIGITAL OPERATIONAL RESILIENCE TESTING (Art. 24-27)
   - Jährliche BCP/DR-Tests (alle Unternehmen)
   - Bedrohungsorientiertes Penetration Testing (TLPT):
     * Pflicht für bedeutende Institute (BaFin-Designation)
     * Alle 3 Jahre, Red Team nach TIBER-EU/TIBER-DE Framework
     * Führendes Finanzunternehmen + externer Red Team Provider

4. IKT-DRITTPARTEIENRISIKO (Art. 28-44)
   - Register aller IKT-Drittparteien (BaFin-Meldepflicht)
   - Vertragliche Mindestanforderungen für Dienstleister
   - Kritische Drittparteien: EU-Oversight-Rahmen (ESA-Designated)
   - Exit-Strategie für jeden kritischen Dienstleister

5. INFORMATIONSAUSTAUSCH (Art. 45)
   - Freiwilliger Austausch von Cyber-Bedrohungsinformationen
   - Vertrauensgruppen, sektorale ISACs
```

### BAIT-Anforderungen im Überblick

```
BAIT-Kapitel und IT-Sicherheitsbezug:

Kapitel 3 - IT-Governance:
  → IT-Strategie, IT-Organisationsstruktur
  → IT-Sicherheitsbeauftragter (CISO) als unabhängige Funktion
  → IT-Risikosteuerung integriert in Gesamtrisikomanagement

Kapitel 5 - Informationsrisikomanagement:
  → Schutzbedarfsfeststellung für alle Informationsassets
  → IT-Risikoanalyse (qualitativ + quantitativ)
  → Risikotragfähigkeit IT-Risiken

Kapitel 6 - Informationssicherheitsmanagement:
  → ISMS nach anerkanntem Standard (ISO 27001 oder BSI-Grundschutz)
  → Sicherheitsrichtlinien, Awareness-Programm
  → Security Operations (Monitoring, SIEM)
  → Penetrationstest-Programm

Kapitel 7 - Benutzerberechtigungsmanagement:
  → Minimalprinzip, Need-to-Know
  → Privilegierte Konten: PAM-System, 4-Augen-Prinzip
  → Rezertifizierung: mindestens jährlich
  → Sofortige Sperrung bei Ausscheiden / Rollenwechsel

Kapitel 8 - IT-Projekte / Anwendungsentwicklung:
  → SSDLC: Security-by-Design, Anforderungsanalyse
  → Code Reviews, SAST/DAST
  → Abnahmetest vor Produktivgang

Kapitel 9 - IT-Betrieb:
  → Patch-Management (kritische Patches ≤ 1 Woche)
  → Change Management, Konfigurationsmanagement
  → Kapazitätsmanagement

Kapitel 10 - Ausgliederungen:
  → Auslagerungsregister
  → Risikoanalyse vor Auslagerung
  → Vertragliche Mindestanforderungen, Exit-Planung
  → Prüfrechte, Kontrollpflichten
```

---

## Kritische Systeme Finance

### Systemlandschaft typische Bank / Finanzdienstleister

```
┌──────────────────────────────────────────────────────────────┐
│  CORE BANKING                                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐  │
│  │  Kernbank│  │ Zahlungs-│  │  Kredit- │  │  Wertpapier│  │
│  │  (Temenos│  │verkehr   │  │  verarbeit│  │  abwicklung│  │
│  │  Avaloq) │  │(SEPA/SWIFT│  │          │  │  (T2S,Xetra│  │
│  └─────┬────┘  └─────┬────┘  └─────┬────┘  └─────┬──────┘  │
│        └─────────────┴─────────────┴──────────────┘         │
│                    Banking-Backend-Netz                      │
├──────────────────────────────────────────────────────────────┤
│  ZAHLUNGSVERKEHR                                             │
│  SWIFT · SEPA · TARGET2 · Karten (Visa/MC) · POS-Netz       │
│  → HSM-basierte Kryptographie, SWIFT-CSCF Compliance        │
├──────────────────────────────────────────────────────────────┤
│  KUNDEN-CHANNELS                                             │
│  Online-Banking · Mobile Banking · API-Banking (PSD2)        │
│  → WAF, Strong Auth (SCA), OAuth2/OIDC, Fraud Detection     │
├──────────────────────────────────────────────────────────────┤
│  OFFICE / HANDEL                                             │
│  Trading Desks · Treasury · Compliance-Systeme · ERP        │
└──────────────────────────────────────────────────────────────┘
```

### Top-Angriffsvektoren Finance

| Vektor | Ziel | Gegenmaßnahme |
|---|---|---|
| Business Email Compromise (BEC) | SWIFT-Überweisungen, CEO-Fraud | DMARC/DKIM/SPF, Dual-Control für Zahlungen |
| Insider Threats | Trading-Daten, Kundendaten, Betrug | PAM, DLP, User Behaviour Analytics (UBA) |
| Ransomware | Betriebsunterbrechung | Segmentierung, EDR, Backup-Isolation |
| API-Angriffe (PSD2/Open Banking) | Kontodaten, Transaktionen | API-Gateway, OAuth2, Rate Limiting |
| ATM/POS-Malware | Karteninhaberdaten, Cash-Out | PCI DSS, Netzwerksegmentierung, Whitelisting |
| Supply Chain (Core Banking SW) | Systemkompromittierung | DORA Art. 28+, SBOM, Third-Party-Assessment |
| SWIFT-System-Kompromittierung | Betrügerische Überweisungen | SWIFT CSCF v2024, HSM, 4-Augen |

---

## PCI DSS v4.0

### 12 Anforderungsbereiche

```
NETZWERK AUFBAUEN UND SICHERN:
Req 1: Netzwerk-Sicherheitskontrollen (Firewall, Segmentierung CDE)
Req 2: Sichere Konfigurationen für alle Systemkomponenten

KONTODATEN SCHÜTZEN:
Req 3: Gespeicherte Kontodaten schützen (Verschlüsselung, Tokenisierung)
Req 4: Karteninhaber-Daten bei Übertragung schützen (TLS 1.2+)

SCHWACHSTELLEN MANAGEN:
Req 5: Schutz vor Malware (EDR, AV auf allen Systemen)
Req 6: Sichere Systeme und Software (Patch-Management, SAST/DAST)

ZUGANG KONTROLLIEREN:
Req 7: Zugang auf Notwendigkeit einschränken (Need-to-Know)
Req 8: Benutzer identifizieren und authentifizieren (MFA für alle CDE-Zugänge)
Req 9: Physischen Zugang zu Karteninhaberdaten einschränken

ÜBERWACHEN UND TESTEN:
Req 10: Zugriffe auf Systemkomponenten und Daten protokollieren
Req 11: Systeme und Netzwerke regelmäßig testen (Penetrationstests!)

SICHERHEITSRICHTLINIE:
Req 12: Informationssicherheitsrichtlinie dokumentieren und kommunizieren
```

### PCI DSS Pentest-Anforderungen (Req 11.4)

```
Pflichtumfang für PCI DSS v4.0:

Extern:
- Jährlicher Penetrationstest aus externer Perspektive
- Nach signifikanter Infrastrukturveränderung
- Umfang: Alle extern erreichbaren Systeme und Anwendungen in der CDE

Intern:
- Jährlicher Penetrationstest aus interner Perspektive
- Segmentierungstest: Verifikation dass CDE isoliert ist
  (mindestens jährlich ODER bei Segmentierungsänderungen)

Anforderungen an den Test:
- Durchführung durch qualifizierte interne oder externe Ressource
- Kein Interessenskonflikt mit dem zu testenden System
- Retesting bis alle Hochrisiko-Findings behoben

Testtiefe:
- Netzwerkebene (Infrastruktur)
- Anwendungsebene (Web-Apps, APIs)
- Manuelle Tests + automatisierte Scans
```

### CDE-Segmentierung (Cardholder Data Environment)

```
Ziel: Minimierung der CDE-Größe → weniger PCI-Scope, weniger Compliance-Aufwand

Segmentierungsoptionen:
1. Firewall-Segmentierung: CDE in eigenem VLAN, strikte ACLs
2. Tokenisierung: Kartennummern durch Tokens ersetzen (kein PAN mehr in Systemen)
3. P2PE (Point-to-Point Encryption): Verschlüsselung vom Terminal bis Zahlungsabwickler

Segmentierungstest-Checkliste:
- [ ] CDE in isoliertem VLAN/Netz-Segment
- [ ] Alle Firewall-Regeln documented und approved
- [ ] Out-of-scope-Systeme können CDE nicht direkt erreichen
- [ ] Jährlicher Segmentierungstest durch qualifizierte Person
- [ ] Test-Nachweis und -Dokumentation für QSA-Audit

Typische Segmentierungsfehler:
✗ Shared DNS-Server zwischen CDE und Non-CDE
✗ Shared AD-Domain-Controller in CDE und Non-CDE
✗ Management-VLAN übergreifend ohne Firewall
✗ Backup-System hat Zugang zu CDE und Non-CDE
```

---

## SWIFT CSCF (Customer Security Controls Framework)

### Mandatory Controls v2024

```
SICHERN UND SCHÜTZEN:
1.1 SWIFT-Umgebung einschränken (isoliertes Netz, keine Internet-Verbindung)
1.2 Betriebssystem-Sicherheit auf SWIFT-Systemen
1.3 Virtuelle Umgebungssicherheit (wenn zutreffend)
1.4 Einschränkung des Internetzugangs
2.1 Interne Datensicherheitsfunktionen
2.2 Sicherheitsupdate-Strategie
2.3 Systemhärtung

ERKENNEN UND REAGIEREN:
6.1 Malwareschutz auf SWIFT-Systemen
6.2 Software-Integrität
6.3 Datenbankintegrität
7.1 Cyber-Vorfallreaktionsplanung
7.2 Sicherheitstraining

ZUGRIFF KONTROLLIEREN:
4.1 Passwortrichtlinie
4.2 Multi-Faktor-Authentifizierung für SWIFT-Interface
5.1 Logische Zugriffskontrolle
5.2 Token-Management
5.3 Personal-Integrität

Advisory Controls (empfohlen, nicht mandatory):
2.4-2.9: Erweiterte Härtung
3.1: Physische Sicherheit
6.4-6.5: Transaktionsanomalien

Assessment: Jährliche Selbsteinschätzung + Nachweis an Korrespondenzbanken
```

---

## Compliance-Checkliste Finance

### DORA Gap-Assessment-Vorlage

```markdown
## DORA Gap-Assessment — [Unternehmen] — [Datum]

### Säule 1: IKT-Risikomanagement
- [ ] IKT-Risikomanagement-Rahmen dokumentiert und genehmigt
- [ ] ICT Asset Register vollständig (HW, SW, Daten, Dienstleister)
- [ ] Schutzbedarfsfeststellung für alle IKT-Assets
- [ ] Business-Impact-Analyse (BIA) für kritische Funktionen
- [ ] RPO/RTO-Ziele definiert und getestet
- [ ] Backup-Strategie: 3-2-1-1-0, DR-Tests dokumentiert
- [ ] Lebenszyklusmanagement IKT-Assets (inkl. Ausmusterung)

### Säule 2: IKT-Incident-Management
- [ ] Incident-Klassifizierungsschema nach DORA-Kriterien implementiert
- [ ] Meldeprozess für erhebliche Vorfälle: 4h / 72h / 1 Monat
- [ ] BaFin als zuständige Behörde im Meldeprozess hinterlegt
- [ ] IR-Team mit klaren Rollen und Eskalationspfaden
- [ ] IR-Plan jährlich getestet
- [ ] Major Incidents der letzten 12 Monate überprüft

### Säule 3: Digital Operational Resilience Testing
- [ ] Jährliches BCP/DR-Testprogramm implementiert
- [ ] Jährliche Penetrationstests (alle Systeme, kritische Funktionen)
- [ ] TLPT-Pflicht geprüft: Ist Unternehmen von BaFin als bedeutend eingestuft?
- [ ] Falls TLPT pflichtig: TIBER-DE-Anbieter evaluiert, Testplan erstellt
- [ ] Test-Ergebnisse und Maßnahmen dokumentiert

### Säule 4: IKT-Drittparteienrisiko
- [ ] Register aller IKT-Drittdienstleister vollständig
- [ ] Kritische Drittdienstleister identifiziert (nach DORA-Kriterien)
- [ ] Risikoanalyse für alle kritischen Dienstleister durchgeführt
- [ ] Verträge auf DORA-Mindestklauseln (Art. 30) geprüft
- [ ] Exit-Strategie für jeden kritischen Dienstleister vorhanden
- [ ] BaFin-Meldung des IKT-Dienstleisterregisters (jährlich)
- [ ] Subauslagerungen bekannt und bewertet

### Säule 5: Informationsaustausch
- [ ] Teilnahme an sektoralem ISAC (FS-ISAC, Allianz für Cybersicherheit) geprüft
- [ ] Prozess für Weitergabe von Bedrohungsinformationen definiert
```

### BAIT-Compliance-Checkliste

```markdown
## BAIT Compliance-Status — [Institut] — [Datum]

### IT-Governance (BAIT Kap. 3)
- [ ] IT-Strategie vorhanden, aktuell, von Geschäftsleitung genehmigt
- [ ] CISO als unabhängige Funktion besetzt
- [ ] IT-Risiken im Gesamtrisiko-Reporting integriert
- [ ] IT-Ausschuss oder gleichwertige Governance-Struktur

### Informationsrisikomanagement (BAIT Kap. 5)
- [ ] Schutzbedarfsfeststellung für alle Informationsassets
- [ ] IT-Risikoanalyse aktuell (≤ 1 Jahr)
- [ ] Risiken dokumentiert, Maßnahmen verfolgt

### ISMS (BAIT Kap. 6)
- [ ] ISMS implementiert (ISO 27001 oder BSI-Grundschutz)
- [ ] Sicherheitsrichtlinien (Passwort, Clean Desk, Mobile, Remote)
- [ ] Security Awareness Programm aktiv
- [ ] SIEM/SOC in Betrieb, Alerting-Schwellen definiert
- [ ] Jährliche Penetrationstests, Ergebnisse nachverfolgt
- [ ] Schwachstellen-Scanning (mindestens quartalsweise)

### Benutzerberechtigungsmanagement (BAIT Kap. 7)
- [ ] Prozess für Benutzeranlage / -änderung / -sperrung
- [ ] Privilegierte Accounts in PAM-System (CyberArk/BeyondTrust)
- [ ] 4-Augen-Prinzip für privilegierte Aktionen
- [ ] Jährliche Rezertifizierung aller Benutzerrechte
- [ ] Sofortige Sperrung bei Ausscheiden (SLA ≤ 24h)

### Anwendungsentwicklung (BAIT Kap. 8)
- [ ] SSDLC-Prozess implementiert
- [ ] SAST in CI/CD-Pipeline integriert
- [ ] DAST für alle produktiven Web-Anwendungen
- [ ] Abnahmetest vor Produktivgang (Security-Kriterien)

### Patch-Management (BAIT Kap. 9)
- [ ] Kritische Patches: Deployment ≤ 1 Woche nach Verfügbarkeit
- [ ] Alle Patches: ≤ 4 Wochen
- [ ] Ausnahmen dokumentiert und risikoadjustiert genehmigt
- [ ] Patchstand-Reporting an CISO / Geschäftsleitung

### Auslagerungen (BAIT Kap. 10 / MaRisk AT 9)
- [ ] Auslagerungsregister vollständig und aktuell
- [ ] Risikoanalyse vor jeder wesentlichen Auslagerung
- [ ] Verträge mit Mindestklauseln (Prüfrechte, SLA, Exit)
- [ ] Jährliche Kontrolle aller wesentlichen Auslagerungen
- [ ] Exit-Strategie für jeden wesentlichen Dienstleister
```

---

## Fraud Detection & Anti-Money Laundering (AML) IT

### Technische Anforderungen Fraud-Systeme

```
Transaktions-Monitoring:
- Real-Time Fraud Detection: Latenz < 100ms für Zahlungsautorisierung
- ML-Modelle: Verhaltensbasierte Anomalieerkennung
  * Velocity Checks: Ungewöhnliche Transaktionsfrequenz
  * Amount Profiling: Abweichung vom historischen Betragsmuster
  * Geo-Velocity: Physisch unmögliche Standortwechsel
  * Device Fingerprinting: Neues Gerät + neue IP + hohes Volumen

AML-Systeme (Geldwäscheprävention):
- Transaktionsmonitoring nach § 25h KWG
- Sanctions Screening: Echtzeit gegen OFAC/EU/UN-Listen
- PEP-Screening (Politically Exposed Persons)
- CTR (Currency Transaction Reports) Automatisierung

Regulatorische Anforderungen:
- BaFin: Rundschreiben 05/2023 (GW) — AML-Anforderungen
- 6. EU-Geldwäscherichtlinie (AMLD6)
- AMLA (neue EU Anti-Money Laundering Authority) ab 2025

Security der Fraud/AML-Systeme:
- Audit-Logging: Unveränderliche Protokollierung aller Entscheidungen
- Zugriffskontrolle: Nur Compliance/Fraud-Team, 4-Augen für Systemänderungen
- Modell-Governance: Änderungen an Scoring-Modellen über Change-Control
- Backtesting: Regelmäßige Überprüfung der Erkennungsleistung
```

---

## Incident Response Finance

### DORA-konformer Meldeprozess

```
ERHEBLICHER IKT-VORFALL — Klassifizierungskriterien (DORA Art. 18):

Ein Vorfall ist ERHEBLICH wenn mindestens eines zutrifft:
✓ Anzahl betroffener Kunden > Schwellenwert (RTS wird noch festgelegt)
✓ Geografische Ausbreitung > ein Mitgliedstaat
✓ Ausfallzeit kritischer Dienste > Schwellenwert
✓ Wirtschaftlicher Schaden > Schwellenwert
✓ Reputationsschaden wahrscheinlich
✓ Kritikalität der betroffenen Systeme

MELDEKETTE:
T+0: Entdeckung des Vorfalls
T+4h: Erstmeldung an BaFin (sobald als erheblich klassifiziert)
     → Auch wenn Auswirkungen noch unklar!
     → Meldung über BaFin-Melde- und Informationsportal (MVP)
T+72h: Zwischenmeldung — Aktueller Status, Ausmaß, laufende Maßnahmen
T+30 Tage: Abschlussbericht — Root Cause, Maßnahmen, Lessons Learned

MELDEINHALTE (Erstmeldung):
- Zeitstempel Entdeckung und Klassifizierung
- Betroffene Systeme und Dienste
- Ersteinschätzung Ausmaß und Auswirkungen
- Sofortmaßnahmen eingeleitet

PARALLELE MELDEPFLICHTEN:
- Datenschutzverletzung → Datenschutzbehörde 72h (Art. 33 DSGVO)
- Schwere Zahlungssicherheitsvorfälle → EZB und Heimatbehörde (PSD2)
- Operationelle Verluste → Bundesbank (OpRisk-Meldung § 25p KWG)
```

### Business Continuity Finance

```
Kritische Recovery-Ziele (Beispiel Großbank):

System                | RTO        | RPO
----------------------|------------|--------
SEPA-Zahlungsverkehr  | 2 Stunden  | 0 (sync)
Online-Banking        | 4 Stunden  | 15 min
Core Banking          | 4 Stunden  | 0 (sync)
SWIFT                 | 2 Stunden  | 0 (sync)
Wertpapierabwicklung  | 4 Stunden  | 0 (sync)
Interne IT (Email)    | 24 Stunden | 4 Stunden
Filialsysteme         | 4 Stunden  | 1 Stunde

BCP-Testanforderungen DORA:
- Jährliche Vollübung (Failover-Test produktiver Systeme)
- Mindestens Tabletop-Übungen für alle kritischen Szenarien
- Dokumentation aller Tests, Ergebnisse und Verbesserungsmaßnahmen
- Einbeziehung kritischer IKT-Drittdienstleister in Tests
```

---

## TIBER-DE (Bedrohungsorientiertes Penetration Testing)

### Framework-Überblick

```
TIBER-DE = Deutsche Umsetzung des TIBER-EU Frameworks
Koordiniert durch: Deutsche Bundesbank
Zielgruppe: Bedeutende Finanzinstitute (von BaFin designated)

Phasen:
1. VORBEREITUNG (3-4 Monate)
   - Scope-Festlegung mit Bundesbank/BaFin
   - Auswahl externer Red-Team-Provider (TIBER-zertifiziert)
   - Threat Intelligence Provider auswählen
   
2. THREAT INTELLIGENCE (2-3 Monate)
   - Gezielte Bedrohungsanalyse für das Institut
   - Identifikation relevanter Bedrohungsakteure
   - Angriffsziele und -methoden (TTPs)
   - Lieferung: Generic Threat Intelligence Report + Targeted Threat Intelligence Report

3. RED TEAM TEST (3-4 Monate)
   - Angriffssimulation basierend auf TTI
   - Ohne Wissen der Blue Team-Mitarbeiter
   - Ziel: Production-nahe kritische Systeme, nicht nur IT
   - Unbegrenzte Kreativität (Social Engineering, Physical Intrusion erlaubt)

4. CLOSURE (1-2 Monate)
   - Purple Team Exercise: Red + Blue Team zusammen
   - Bericht: Findings, Detektionsnachweis
   - Remediation-Plan
   - Attestierung an Regulatoren (ohne Details — nur Pass/Fail)

Besonderheit: Ergebnis wird unter Regulatoren geteilt (gegenseitige Anerkennung)
→ TIBER-Test in DE kann auch NL, FR, etc. Anforderungen erfüllen
```

---

## Härtungsmaßnahmen Finance-Systeme

### Online-Banking / Mobile Banking

```
Authentifizierung (PSD2 SCA-Anforderungen):
- Zwei voneinander unabhängige Elemente aus: Wissen / Besitz / Inhärenz
- Dynamische Verknüpfung: SCA-Code an Transaktionsbetrag und Empfänger gebunden
- Ausnahmen (Transaktionsrisikoanalyse — TRA) nur bei sehr niedrigen Betrügerraten

Technische Maßnahmen:
1. WAF: OWASP CRS, SQL-Injection, XSS, CSRF-Schutz
2. API-Gateway (PSD2 Open Banking): OAuth 2.0 + PKCE, Rate Limiting, API-Key-Rotation
3. Session-Management: Token-Invalidierung nach Transaktion, kurze Sitzungszeiten
4. Device Fingerprinting: Neue Geräte → verstärkte Authentifizierung
5. Anomalie-Erkennung: ML-basierte Verhaltensmuster
6. Certificate Pinning: Mobile Apps pinnen auf definierte Zertifikatsketten

JWS-Signierung (PSD2 eIDAS):
- Alle Nachrichten im Open-Banking-API mit qualifizierter Signatur
- eIDAS-Zertifikat: QSealC (Siegel) oder QWAC (Website-Authentifizierung)
- Validierung der Gegenseite zwingend
```

### SWIFT-Umgebungshärtung (CSCF 1.1)

```bash
# SWIFT-Netzwerk-Segmentierung
# Dedicated VLAN für SWIFT-Komponenten
# Kein direkter Internetausgang für SWIFT-Systeme

# Firewall-Regeln (Beispiel iptables/nftables):
# Nur SWIFT-Service Bureau oder Alliance Gateway Outbound erlauben
iptables -A OUTPUT -d <SWIFT_SB_IP> -p tcp --dport 443 -j ACCEPT
iptables -A OUTPUT -d <SWIFT_SB_IP> -p tcp --dport 8443 -j ACCEPT
iptables -A OUTPUT -j DROP  # Alles andere verwerfen

# MFA für Alliance Access / Alliance Gateway Administration
# (kein Passwort-Only für SWIFT-Systeme!)

# Integrity Check Alliance Software:
# SWIFT stellt Hashwerte für alle Softwareversionen bereit
# Vor Installation verifizieren!
sha256sum SWIFTAlliance_<version>.tar.gz
# Vergleich mit SWIFT-Referenzwerten auf support.swift.com
```

---

## Weiterführende Ressourcen

- **BaFin**: [BAIT](https://www.bafin.de/bait), [DORA-Seite](https://www.bafin.de/dora), [MaRisk](https://www.bafin.de/marisk)
- **Deutsche Bundesbank**: [TIBER-DE Framework](https://www.bundesbank.de/tiber-de)
- **EBA**: Guidelines on ICT and Security Risk Management
- **SWIFT**: Customer Security Controls Framework — [security.swift.com](https://security.swift.com)
- **PCI SSC**: PCI DSS v4.0 — [pcisecuritystandards.org](https://www.pcisecuritystandards.org)
- **FS-ISAC**: Financial Services Information Sharing and Analysis Center
- **ENISA**: DORA Implementation Guidance
- **ECB**: TIBER-EU Framework (Basis für TIBER-DE)
