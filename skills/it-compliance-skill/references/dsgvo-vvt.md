# DSGVO — Verzeichnis der Verarbeitungstätigkeiten & Grundlagen

Vollständige Vorlagen für VVT, Datenpanne, Betroffenenrechte, TOM — auf Deutsch.

---

## Verzeichnis der Verarbeitungstätigkeiten (VVT)

### VVT-Vorlage (Art. 30 DSGVO)

```markdown
# Verzeichnis der Verarbeitungstätigkeiten
**Verantwortlicher:** [Unternehmen GmbH], [Adresse]
**Datenschutzbeauftragter:** [Name], [E-Mail], [Telefon]
**Stand:** [TT.MM.JJJJ]  |  **Version:** [X.Y]

---

## Verarbeitungstätigkeit Nr. 001: Personalverwaltung

| Feld | Inhalt |
|------|--------|
| **Bezeichnung** | Verwaltung von Beschäftigtendaten |
| **Zweck** | Erfüllung arbeitsrechtlicher Pflichten, Lohnabrechnung, Personalplanung |
| **Rechtsgrundlage** | Art. 6 Abs. 1 lit. b DSGVO (Vertragserfüllung), Art. 6 Abs. 1 lit. c DSGVO (rechtliche Verpflichtung), § 26 BDSG |
| **Kategorien betroffener Personen** | Mitarbeiter, ehemalige Mitarbeiter, Bewerber |
| **Kategorien personenbezogener Daten** | Stammdaten (Name, Adresse, Geburtsdatum), Vertragsdaten, Gehaltsabrechnungen, Krankmeldungen (Gesundheitsdaten!), Sozialversicherungsdaten |
| **Besondere Kategorien (Art. 9)** | Gesundheitsdaten (Krankmeldungen), ggf. Gewerkschaftszugehörigkeit |
| **Empfänger** | Steuerberater/Lohnbuchhaltung (AVV), Sozialversicherungsträger, Finanzamt, Berufsgenossenschaft |
| **Drittlandtransfer** | Nein (alle Empfänger in EU/EWR) |
| **Löschfrist** | 10 Jahre nach Beschäftigungsende (§ 257 HGB, § 147 AO); Bewerberdaten: 6 Monate nach Absage |
| **TOM** | Zugangskontrolle HR-System (Rollenkonzept), Verschlüsselung Personalakten, Sicherheitsüberprüfung HR-Mitarbeiter |
| **Auftragsverarbeiter** | DATEV eG (Lohnbuchhaltung) — AVV vorhanden |

---

## Verarbeitungstätigkeit Nr. 002: Kundenverwaltung / CRM

| Feld | Inhalt |
|------|--------|
| **Bezeichnung** | Verwaltung von Kundendaten und -beziehungen |
| **Zweck** | Vertragserfüllung, Angebotserstellung, Kundenkommunikation, After-Sales |
| **Rechtsgrundlage** | Art. 6 Abs. 1 lit. b (Vertragserfüllung), Art. 6 Abs. 1 lit. f (berechtigtes Interesse: Kundenpflege) |
| **Kategorien betroffener Personen** | Kunden (natürliche Personen), Ansprechpartner bei Firmenkunden |
| **Kategorien personenbezogener Daten** | Kontaktdaten, Vertragsdaten, Kommunikationshistorie, Zahlungsdaten |
| **Empfänger** | CRM-Dienstleister (AVV), Inkasso (bei Bedarf) |
| **Drittlandtransfer** | USA (Salesforce/HubSpot) — Standardvertragsklauseln + TIA |
| **Löschfrist** | 10 Jahre nach Vertragsende (HGB), Angebote ohne Auftrag: 3 Jahre |
| **Auftragsverarbeiter** | Salesforce Germany GmbH — AVV + EU-Standardvertragsklauseln |

---

## Verarbeitungstätigkeit Nr. 003: IT-Sicherheit / Logging

| Feld | Inhalt |
|------|--------|
| **Bezeichnung** | IT-Sicherheitsmonitoring, Protokollierung, SIEM |
| **Zweck** | Gewährleistung der IT-Sicherheit, Erkennung und Abwehr von Angriffen |
| **Rechtsgrundlage** | Art. 6 Abs. 1 lit. f DSGVO (berechtigtes Interesse: IT-Sicherheit), § 32 BDSG (Mitarbeiterüberwachung) |
| **Kategorien betroffener Personen** | Mitarbeiter, externe Nutzer, Angreifer |
| **Kategorien personenbezogener Daten** | IP-Adressen, Benutzernamen, Zugriffszeiten, aufgerufene URLs, E-Mail-Metadaten |
| **Empfänger** | SOC-Dienstleister (AVV), ggf. Strafverfolgungsbehörden (bei Vorfällen) |
| **Löschfrist** | 90 Tage (Standardlogs), 1 Jahr (Sicherheitsvorfälle), 6 Monate (BSI-Empfehlung) |
| **Mitbestimmung** | Betriebsvereinbarung IT-Monitoring erforderlich (BetrVG § 87 Abs. 1 Nr. 6)! |
| **TOM** | Zugriff auf Logs nur IT-Sicherheitsteam, keine Vollprotokollierung, Pseudonymisierung wo möglich |
```

---

## Datenpanne-Meldung (Art. 33/34 DSGVO)

### Melde-Checkliste (72-Stunden-Frist)

```markdown
## Datenpanne-Protokoll — INTERN VERTRAULICH

**Datum Entdeckung:**    [TT.MM.JJJJ HH:MM]
**Datum Meldung DSB:**   [TT.MM.JJJJ HH:MM]  ← DSB sofort informieren!
**Incident-ID:**         [YYYY-MM-DD-XXX]
**Schweregrad:**         [ ] Gering  [ ] Mittel  [ ] Hoch  [ ] Kritisch

### Art der Verletzung (Art. 33 Abs. 3 lit. a)
- [ ] Vertraulichkeit: Unbefugter Zugriff/Offenbarung
- [ ] Integrität: Unbefugte Änderung
- [ ] Verfügbarkeit: Vernichtung/Verlust

### Betroffene Personen (Art. 33 Abs. 3 lit. c)
Kategorien: _____________________
Ungefähre Anzahl: _______________

### Betroffene Datensätze (Art. 33 Abs. 3 lit. c)
Kategorien: _____________________
Enthält Art.-9-Daten (besondere Kategorien)?: [ ] Ja  [ ] Nein

### Wahrscheinliche Folgen (Art. 33 Abs. 3 lit. d)
_______________________________________

### Ergriffene Maßnahmen (Art. 33 Abs. 3 lit. e)
_______________________________________

### Meldepflicht prüfen:
- [ ] Meldepflicht an Datenschutzbehörde (Art. 33): Risiko für Betroffene?
  - Ja: Meldung innerhalb 72h ab Kenntnis
  - Nein: Nur intern dokumentieren (Art. 33 Abs. 5)
- [ ] Benachrichtigungspflicht Betroffene (Art. 34): HOHES Risiko?
  - Ja: "Unverzüglich" benachrichtigen
  - Nein: Keine Benachrichtigung erforderlich

### Zuständige Datenschutzbehörde (DACH):
Bayern:          BayLDA — dsb@lda.bayern.de, +49 981 53-1300
Baden-Württemberg: LfDI — poststelle@lfdi.bwl.de
NRW:             LDI NRW — poststelle@ldi.nrw.de
Hessen:          HBDI — poststelle@datenschutz.hessen.de
Österreich:      DSB — dsb@dsb.gv.at
Schweiz:         EDÖB — info@edoeb.admin.ch
```

---

## Technische und Organisatorische Maßnahmen (TOM)

### TOM-Vorlage (Art. 32 DSGVO)

```markdown
## TOM-Dokumentation — [Unternehmen GmbH]
Stand: [TT.MM.JJJJ]

### 1. Zutrittskontrolle (physisch)
- Serverraum: Zugang nur per Schlüssel/Chipkarte, Besucherprotokoll
- Bürogebäude: Alarmanlage, Videoüberwachung (separates VVT!)
- Aktenvernichtung: Schredder Sicherheitsstufe P-4 (DIN 66399)

### 2. Zugangskontrolle (IT-Systeme)
- Passwortrichtlinie: mind. 12 Zeichen, Komplexitätsvorgaben
- MFA: für alle Remote-Zugänge und Cloud-Dienste Pflicht
- Automatische Sperrung nach 5 Fehleingaben, Entsperrung nach 15 Min.
- Screensaver mit Passwortschutz nach 5 Min. Inaktivität

### 3. Zugriffskontrolle (Berechtigungen)
- Rollenbasiertes Berechtigungskonzept (RBAC) dokumentiert
- Need-to-Know-Prinzip: Zugriff nur auf notwendige Daten
- Privilegierte Konten: separates Admin-Konto, PAM-System
- Rezertifizierung: halbjährlich
- Protokollierung aller privilegierten Zugriffe

### 4. Trennungskontrolle
- Mandantentrennung in Systemen (technisch sichergestellt)
- Test/Produktionssystem getrennt
- Keine Produktivdaten in Testsystemen

### 5. Pseudonymisierung / Verschlüsselung
- Datenverschlüsselung at rest: AES-256 (Festplatten, Backups)
- Datenverschlüsselung in transit: TLS 1.2+ für alle Übertragungen
- E-Mail-Verschlüsselung: S/MIME für sensitive Kommunikation
- Pseudonymisierung: Kundennummern statt Klarnamen in Logs

### 6. Verfügbarkeitskontrolle
- Backup: täglich, 3-2-1-1-0-Regel, monatlicher Restore-Test
- USV: 30 Min. Überbrückung im Serverraum
- Redundante Internetanbindung (zwei ISPs)
- DR-Plan vorhanden, jährlich getestet

### 7. Vertraulichkeit
- NDA mit allen Mitarbeitern und Dienstleistern
- Datenschutzschulung: jährlich für alle Mitarbeiter
- Clean-Desk-Policy: keine sensiblen Daten offen auf Schreibtischen
- Mobile Device Management (MDM) für dienstliche Geräte

### 8. Integrität
- Patch-Management: kritische Patches ≤ 1 Woche
- Change-Management-Prozess dokumentiert
- Code-Reviews vor Produktiveinsatz
- Checksummen bei Dateiübertragungen

### 9. Auftragskontrolle (AVV)
- Schriftliche AVV mit allen Auftragsverarbeitern (Art. 28 DSGVO)
- Regelmäßige Überprüfung der Dienstleister (jährlich)
- Weisungsrecht dokumentiert
```

---

## DSGVO-Pflichten Schnellübersicht

```
WANN IST EIN DSB PFLICHT? (§ 38 BDSG / Art. 37 DSGVO)
  ✓ Regelmäßige und systematische Überwachung von Personen (z.B. Videoüberwachung + > 20 MA)
  ✓ Umfangreiche Verarbeitung besonderer Kategorien (Art. 9: Gesundheit, Religion, etc.)
  ✓ Kreditauskunfteien, Inkassobüros, Auskunfteien
  ✓ Mind. 20 Personen beschäftigt mit automatisierter Verarbeitung (BDSG)

WANN DSFA PFLICHT? (Art. 35 + DSK-Blacklist)
  ✓ Scoring, Profiling mit erheblichen Auswirkungen
  ✓ Verarbeitung besonderer Datenkategorien im großen Umfang
  ✓ Systematische Überwachung öffentlicher Bereiche
  ✓ Neue Technologien mit hohem Risiko
  → Orientierung: DSK-Liste "muss DSFA" + "Blacklist"

BETROFFENENRECHTE (Art. 15-22 DSGVO) — Fristen:
  Auskunft (Art. 15):         1 Monat (verlängerbar auf 3 Monate)
  Berichtigung (Art. 16):     Unverzüglich
  Löschung (Art. 17):         Unverzüglich ("Recht auf Vergessenwerden")
  Einschränkung (Art. 18):    Unverzüglich
  Datenportabilität (Art. 20): 1 Monat
  Widerspruch (Art. 21):      Unverzüglich + Bearbeitung
```
