# Compliance Reporting & IT-Revision — Referenzmodul

IT-Sicherheitsbericht, Datenschutz-Jahresbericht, Audit-Programm, KPI-Dashboard,
NIS2-Meldewesen, Datenpannen-Meldung, Governance-Berichte für Management.

---

## IT-Sicherheitsbericht (Vorlage für Geschäftsleitung)

```markdown
# IT-Sicherheitsbericht [Quartal/Jahr]
**Erstellt:** [Datum] | **Erstellt von:** [ISB/CISO] | **Klassifizierung:** Intern

---

## Executive Summary
[2-3 Sätze: Gesamtbewertung der Sicherheitslage, wichtigste Ereignisse,
dringendste Handlungsbedarfe]

Ampelstatus: 🟢 Grün / 🟡 Gelb / 🔴 Rot

---

## 1. Sicherheitsvorfälle

### 1.1 Vorfallstatistik
| Kategorie | Anzahl | Davon kritisch | Trend |
|-----------|--------|----------------|-------|
| Phishing-E-Mails (geblockt) | [X] | - | ↗ +15% |
| Malware-Erkennungen | [X] | [X] | ↘ -5% |
| Unbefugte Zugriffsversuche | [X] | [X] | → stabil |
| Datenpannen (gemeldet) | [X] | [X] | - |
| Sicherheitsvorfälle (intern) | [X] | [X] | - |

### 1.2 Bedeutende Vorfälle
[Beschreibung relevanter Vorfälle ohne sensible Details]

| Datum | Vorfall | Auswirkung | Status |
|-------|---------|------------|--------|
| [Datum] | [Kurzbeschreibung] | [Auswirkung] | Behoben / In Bearbeitung |

---

## 2. Schwachstellenmanagement

### 2.1 Patch-Status
| System-Kategorie | Kritische Patches | Davon offen | Ø Patch-Zeit |
|-----------------|-------------------|-------------|--------------|
| Server (Windows) | [X] | [X] | [X] Tage |
| Server (Linux) | [X] | [X] | [X] Tage |
| Workstations | [X] | [X] | [X] Tage |
| Netzwerkgeräte | [X] | [X] | [X] Tage |

### 2.2 Offene Schwachstellen
| CVE / Schwachstelle | CVSS | Betroffene Systeme | Fälligkeit |
|--------------------|------|-------------------|------------|
| [CVE-XXXX-XXXXX] | 9.8 | [System] | [Datum] |

---

## 3. Compliance-Status

### 3.1 DSGVO
- VVT vollständig und aktuell: [Ja/Nein]
- DSB bestellt: [Ja/Nein]
- Datenpannen-Meldungen: [Anzahl]
- Offene Betroffenenanfragen: [Anzahl / Fristwahrung OK?]
- DSFA-Status: [Alle erforderlichen DSFA durchgeführt: Ja/Nein]

### 3.2 BSI IT-Grundschutz / ISO 27001
- Umsetzungsgrad (letzter Check): [X]%
- Zertifizierungsstatus: [Zertifiziert bis / In Vorbereitung / Nicht angestrebt]
- Offene Maßnahmen: [X] (davon [X] überfällig)

### 3.3 NIS2 / KRITIS (falls zutreffend)
- Registrierung BSI: [Ja/Nein]
- Kontaktstelle 24/7: [Ja/Nein]
- Nachweis-Fälligkeit: [Datum]
- Letzter Nachweis eingereicht: [Datum]

---

## 4. Awareness und Schulungen

| Maßnahme | Zielgruppe | Teilnahme | Ergebnis |
|----------|-----------|-----------|---------|
| Phishing-Simulation | Alle MA | [X]% | [X]% Klickrate |
| DSGVO-Pflichtschulung | Alle MA | [X]% | Bestanden: [X]% |
| Admin-Schulung Notfallplan | IT-Team | [X]% | - |

---

## 5. Maßnahmenplan (Top 5 Prioritäten)

| # | Maßnahme | Priorität | Verantwortlich | Frist | Status |
|---|----------|-----------|----------------|-------|--------|
| 1 | [Maßnahme] | Kritisch | [Name] | [Datum] | 🔴 Offen |
| 2 | [Maßnahme] | Hoch | [Name] | [Datum] | 🟡 In Arbeit |
| 3 | [Maßnahme] | Mittel | [Name] | [Datum] | 🟢 Erledigt |

---

## 6. Budget und Ressourcen
[Übersicht IT-Sicherheitsbudget, Ausgaben, Planabweichungen]

---

**Zur Kenntnisnahme / Freigabe:**
Geschäftsleitung: _________________________ Datum: _______
ISB/CISO: _________________________ Datum: _______
```

---

## Datenpannen-Meldung (Art. 33/34 DSGVO)

```markdown
## Datenpannen-Meldeprozess

FRIST: 72 STUNDEN nach Kenntnis der Verletzung → Meldung an Aufsichtsbehörde!

ABLAUF:
T+0:  Entdeckung der Datenpanne
T+1:  DSB und Geschäftsführung informieren
T+4:  Erste Bewertung: Meldepflichtig?
T+24: Vorläufige Meldung an Aufsichtsbehörde (falls meldepflichtig)
T+72: Vollständige Meldung (72h-Frist)
Danach: Betroffene benachrichtigen (Art. 34) falls hohes Risiko

NICHT MELDEPFLICHTIG wenn:
- Kein Risiko für Rechte und Freiheiten betroffener Personen
- Daten waren verschlüsselt + Schlüssel nicht kompromittiert
- Betrifft nur den Verantwortlichen selbst (keine betroffenen Personen)

---

## Meldung an Aufsichtsbehörde (Art. 33 DSGVO)

Pflichtinhalt:
1. Art der Verletzung (Vertraulichkeit / Integrität / Verfügbarkeit)
2. Kategorien und ungefähre Zahl betroffener Personen
3. Kategorien und ungefähre Zahl betroffener Datensätze
4. Name und Kontaktdaten des DSB
5. Wahrscheinliche Folgen der Verletzung
6. Ergriffene und geplante Maßnahmen (inkl. Schadensbegrenzung)

Zuständige Behörden DACH:
Bayern:           BayLDA — www.lda.bayern.de — 0981 531300
Baden-Württemberg: LfDI BW — www.lfdi-bw.de
NRW:              LDI NRW — www.ldi.nrw.de
Berlin:           BlnBDI — www.datenschutz-berlin.de
Hessen:           HBDI — www.datenschutz.hessen.de
Österreich:       DSB — www.dsb.gv.at
Schweiz (EDÖB):   www.edoeb.admin.ch

---

## Meldevorlage Datenpanne

Betreff: Meldung einer Datenpanne gemäß Art. 33 DSGVO

Verantwortlicher:    [Firma, Adresse, Kontakt]
DSB:                 [Name, E-Mail, Tel.]
Meldedatum:          [TT.MM.JJJJ HH:MM]
Vorfallsdatum/-zeit: [TT.MM.JJJJ HH:MM] (Entdeckung: [TT.MM.JJJJ HH:MM])

Art der Verletzung:
[ ] Vertraulichkeit (unbefugter Zugriff/Offenlegung)
[ ] Integrität (unbefugte Veränderung)
[ ] Verfügbarkeit (Verlust/Vernichtung)

Beschreibung: [Sachverhalt, was ist passiert?]

Betroffene Personen:
Kategorien: [z.B. Kunden, Mitarbeiter]
Ungefähre Anzahl: [X Personen]

Betroffene Datenkategorien:
[z.B. Name, E-Mail, Kontodaten, Gesundheitsdaten]
Datensätze: ca. [X]

Wahrscheinliche Folgen:
[z.B. Identitätsdiebstahl, finanzielle Schäden, Reputationsschaden]

Ergriffene Maßnahmen:
[z.B. System isoliert, Passwörter zurückgesetzt, Strafanzeige erstattet]

Geplante Maßnahmen:
[z.B. Sicherheitsaudit, Betroffenenbenachrichtigung]

Benachrichtigung der Betroffenen (Art. 34):
[ ] Geplant am [Datum] — da hohes Risiko
[ ] Nicht erforderlich — Begründung: [...]
```

---

## NIS2 Meldewesen

```markdown
## NIS2-Meldepflichten (ab 17.01.2025 / NIS2UmsuCG)

ERHEBLICHER VORFALL — Definition (NIS2 Art. 23):
Ein Vorfall ist erheblich wenn er:
a) erhebliche Betriebsstörung oder finanzielle Verluste verursacht hat ODER
b) andere natürliche/juristische Personen erheblich beeinträchtigt hat

MELDEFRISTEN (gestaffelt):
T+24h: Frühwarnung an BSI (erhebliche Störung, Vermutung Angriff)
T+72h: Erstmeldung (aktualisierte Infos, erste Einschätzung)
T+1 Monat: Abschlussbericht (vollständige Analyse, Maßnahmen, Lessons Learned)

MELDESTELLE:
BSI Melde- und Informationsportal: meldungen.bsi.bund.de
BSI-Lagezentrum (24/7): 0800 274 1000 (kostenlos)

INHALT FRÜHWARNUNG (T+24h):
- Kurze Beschreibung des Vorfalls
- Art des Vorfalls (Cyberangriff / technischer Ausfall / sonstiges)
- Betroffene Dienste / Systeme
- Grobe Einschätzung Auswirkungen

INHALT ERSTMELDUNG (T+72h):
Alle Pflichtangaben aus DSGVO Art. 33 + zusätzlich:
- Bedrohungsindikator (IoCs falls bekannt)
- Grenzüberschreitende Auswirkungen (andere EU-Staaten?)
```

---

## Compliance-KPI-Dashboard

```markdown
## Compliance KPIs — Monatliche Messung

### DSGVO
| KPI | Ziel | Aktuell | Trend |
|-----|------|---------|-------|
| VVT-Aktualität (Einträge < 12 Monate alt) | 100% | [X]% | |
| Betroffenenanfragen fristgerecht beantwortet | 100% | [X]% | |
| Datenpannen ohne fristgerechte Meldung | 0 | [X] | |
| AVV mit allen Auftragsverarbeitern | 100% | [X]% | |
| Schulungsquote DSGVO | ≥ 95% | [X]% | |

### IT-Sicherheit
| KPI | Ziel | Aktuell | Trend |
|-----|------|---------|-------|
| Kritische Patches eingespielt (< 7 Tage) | ≥ 95% | [X]% | |
| MFA-Abdeckung (alle externen Zugänge) | 100% | [X]% | |
| Backup-Erfolgsrate | ≥ 99% | [X]% | |
| Restore-Test letzte 30 Tage | Ja | Ja/Nein | |
| MTTR (Mean Time to Resolve Incidents) | < 4h | [X]h | |
| Phishing-Klickrate (Simulation) | < 5% | [X]% | |

### BSI / NIS2 (falls zutreffend)
| KPI | Ziel | Aktuell | Trend |
|-----|------|---------|-------|
| IT-GS Umsetzungsgrad | ≥ 80% | [X]% | |
| Offene Hochrisiko-Maßnahmen | 0 | [X] | |
| Nachweis BSI fristgerecht | Ja | Ja/Nein | |
```
