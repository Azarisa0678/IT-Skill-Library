# Datenschutz-Folgenabschätzung (DSFA) — Referenzmodul

DSFA-Pflicht prüfen, Schwellenwertanalyse, vollständige DSFA-Vorlage, Konsultationspflicht.

---

## DSFA-Pflicht (Art. 35 DSGVO)

```
DSFA ist PFLICHT wenn Verarbeitung "voraussichtlich hohes Risiko" für Rechte und
Freiheiten betroffener Personen birgt. Art. 35 Abs. 3 nennt Regelbeispiele:

1. Systematische und umfassende Bewertung persönlicher Aspekte durch automatisierte
   Verarbeitung (Profiling) mit Entscheidungen die erhebliche Wirkung haben
   → Kredit-Scoring, KI-Bewerberauswahl, Verhaltensanalyse

2. Umfangreiche Verarbeitung besonderer Kategorien (Art. 9/10) oder von
   strafrechtlich relevanten Daten
   → Gesundheits-Apps, biometrische Zeiterfassung, Krankenhausinformationssysteme

3. Systematische umfangreiche Überwachung öffentlich zugänglicher Bereiche
   → Videoüberwachung mit Gesichtserkennung, ANPR-Systeme

DSK-Positivliste (DSFA-Pflicht, Deutschland):
✓ Biometrische Verfahren zur eindeutigen Identifizierung
✓ Genetische Daten
✓ Scoring und Bonitätsprüfung
✓ Mitarbeiterbewertungssysteme (umfassend)
✓ Tracking von Bewegungsprofilen (umfangreich)
✓ Zentrale Speicherung von Kommunikationsdaten (umfangreich)
✓ Verarbeitung von Daten schutzbedürftiger Personen (Kinder, Patienten)
✓ KI-gestützte Entscheidungen mit erheblichen Auswirkungen
✓ Datenübermittlung in unsichere Drittländer (umfangreich)

KEINE DSFA ERFORDERLICH (Negativliste Art. 35 Abs. 5):
- Verarbeitung ist nicht voraussichtlich hoch riskant
- Verarbeitung nach Art. 6 Abs. 1 lit. c oder e (Rechtsgrundlage)
  UND Datenschutzrecht der EU/MS regelt dies bereits
- Verarbeitung war bereits in Liste der Aufsichtsbehörde (DSK-Negativliste)
```

---

## Schwellenwertanalyse (Vorprüfung)

```markdown
## Schwellenwertanalyse für: [Verarbeitungsbezeichnung]
Datum: [TT.MM.JJJJ]  |  Bearbeiter: [Name]  |  VVT-Referenz: [VVT-XXX]

### Schritt 1: Grundprinzipien-Check (Art. 5 DSGVO)
Sind alle Grundsätze erfüllt?
- [ ] Rechtmäßigkeit, Verarbeitung nach Treu und Glauben, Transparenz
- [ ] Zweckbindung (keine Zweckänderung ohne neue Rechtsgrundlage)
- [ ] Datensparsamkeit (nur notwendige Daten)
- [ ] Richtigkeit der Daten
- [ ] Speicherbegrenzung (Löschfristen definiert)
- [ ] Integrität und Vertraulichkeit

### Schritt 2: Risikofaktoren (je Ja = 1 Punkt)
| # | Kriterium | Ja/Nein |
|---|-----------|---------|
| 1 | Bewertung / Scoring / Profiling | |
| 2 | Automatisierte Entscheidung mit erheblicher Wirkung | |
| 3 | Systematische Überwachung | |
| 4 | Besondere Datenkategorien (Art. 9/10) | |
| 5 | Umfangreiche Datenmenge oder viele Betroffene (>10.000) | |
| 6 | Zusammenführung von Datensätzen aus verschiedenen Quellen | |
| 7 | Schutzbedürftige Personen (Kinder, Patienten, Mitarbeiter) | |
| 8 | Neue Technologie oder innovative Nutzung | |
| 9 | Drittlandübermittlung ohne Angemessenheitsbeschluss | |
| 10| Datenpannen können erhebliche Schäden verursachen | |

**Punkte gesamt: [X] / 10**

### Schritt 3: Bewertung
| Punkte | Ergebnis |
|--------|----------|
| 0-1 | Kein erhöhtes Risiko → Keine DSFA erforderlich |
| 2-3 | Erhöhtes Risiko → DSFA empfohlen, Einzelfallprüfung |
| ≥ 4 | Hohes Risiko → **DSFA erforderlich (Art. 35 DSGVO)** |
| ≥ 2 + DSK-Positivliste | **DSFA zwingend** |

**Ergebnis:** [ ] DSFA erforderlich  [ ] DSFA nicht erforderlich
**Begründung:** [Begründung dokumentieren!]
**Freigabe DSB:** _________________ Datum: _______
```

---

## DSFA-Vorlage (vollständig)

```markdown
# Datenschutz-Folgenabschätzung
## [Titel der Verarbeitung]

**Erstellt:** [Datum]
**Version:** [1.0]
**Erstellt von:** [Name, Funktion]
**Geprüft von (DSB):** [Name]
**Freigegeben von (GL):** [Name, Datum]
**Nächste Überprüfung:** [Datum]

---

## 1. Beschreibung der Verarbeitungsvorgänge

### 1.1 Verarbeitungskontext
[Beschreibung der Ausgangssituation, des Projekts, der eingesetzten Technologie]

### 1.2 Art der Verarbeitung
Beschreibung: [Was wird mit welchen Daten gemacht?]
Rechtsgrundlage: Art. 6 Abs. 1 lit. [b/c/f] DSGVO
Falls Art. 9: Art. 9 Abs. 2 lit. [___] DSGVO

### 1.3 Kategorien betroffener Personen
- [Kategorie 1: ca. X Personen]
- [Kategorie 2: ca. X Personen]

### 1.4 Kategorien personenbezogener Daten
| Datenkategorie | Sensitivität | Umfang |
|----------------|--------------|--------|
| [z.B. Name, E-Mail] | Normal | [Anzahl] |
| [z.B. Gesundheitsdaten] | Art. 9 (hoch) | [Anzahl] |

### 1.5 Empfänger
[Auflistung aller Empfänger inkl. Drittländer]

### 1.6 Eingesetzte Technologien
[Software, Cloud-Dienste, Algorithmen, KI-Komponenten]

---

## 2. Notwendigkeit und Verhältnismäßigkeit

### 2.1 Zwecke und Rechtsgrundlagen
[Darlegung warum die Verarbeitung notwendig ist und auf welcher Rechtsgrundlage]

### 2.2 Datensparsamkeit
[Begründung warum nicht weniger Daten ausreichen]

### 2.3 Speicherbegrenzung
[Löschfristen und deren Begründung]

### 2.4 Betroffenenrechte
Umsetzung folgender Rechte:
- Auskunftsrecht (Art. 15): [Wie wird es gewährt?]
- Berichtigungsrecht (Art. 16): [Prozess]
- Löschung (Art. 17): [Prozess, Ausnahmen]
- Einschränkung (Art. 18): [Prozess]
- Datenübertragbarkeit (Art. 20): [falls zutreffend]
- Widerspruch (Art. 21): [falls zutreffend]

---

## 3. Risikoidentifikation und -bewertung

### 3.1 Identifizierte Risiken

| # | Risiko | Eintritts-W'keit | Schwere | Risikostufe |
|---|--------|-----------------|---------|-------------|
| 1 | Unbefugter Zugriff auf Kundendaten durch externen Angreifer | Mittel | Hoch | **Hoch** |
| 2 | Versehentliche Weitergabe an falschen Empfänger | Niedrig | Mittel | **Mittel** |
| 3 | Datenverlust durch technischen Defekt | Niedrig | Hoch | **Mittel** |
| 4 | Diskriminierung durch KI-Algorithmus | Mittel | Sehr hoch | **Sehr hoch** |

Risikomatrix:
```
Schwere →      | Niedrig | Mittel | Hoch | Sehr hoch
Eintritts-W. ↓ |         |        |      |
Niedrig        | Gering  | Gering | Mittel | Hoch
Mittel         | Gering  | Mittel | Hoch   | Sehr hoch
Hoch           | Mittel  | Hoch   | Sehr h.| Kritisch
```

### 3.2 Quellen der Risiken
[Beschreibung möglicher Bedrohungsquellen: externe Angreifer, interne Fehler, technische Ausfälle, etc.]

---

## 4. Maßnahmen zur Risikobehandlung

### 4.1 Technische Maßnahmen (TOM nach Art. 32)
| Risiko | Maßnahme | Verantwortlich | Frist | Status |
|--------|----------|----------------|-------|--------|
| 1 | Verschlüsselung AES-256 at rest + TLS 1.3 in transit | IT-Leiter | [Datum] | ✅ |
| 1 | MFA für alle Zugänge | IT-Leiter | [Datum] | ✅ |
| 2 | Vier-Augen-Prinzip bei Datenexporten | Fachbereich | [Datum] | 🔄 |
| 3 | Tägliches Backup, getestet | IT-Leiter | [Datum] | ✅ |
| 4 | Regelmäßige Algorithmus-Prüfung, menschliche Kontrolle | Projektleiter | [Datum] | 🔄 |

### 4.2 Organisatorische Maßnahmen
| Maßnahme | Verantwortlich | Frist | Status |
|----------|----------------|-------|--------|
| Schulung der Mitarbeiter | HR/DSB | [Datum] | ✅ |
| Vertraulichkeitsverpflichtung | HR | [Datum] | ✅ |
| Zugriffskonzept mit Need-to-Know | IT-Leiter | [Datum] | ✅ |
| Incident Response Plan | IT-Sicherheit | [Datum] | 🔄 |

### 4.3 Restrisiken nach Maßnahmen
| Risiko | Restrisiko | Akzeptiert? | Begründung |
|--------|-----------|-------------|------------|
| 1 | Gering | ✅ Ja | TOM ausreichend |
| 4 | Mittel | ⚠️ Bedingt | Kontinuierliches Monitoring |

**Gesamtbewertung:** Das Restrisiko ist [akzeptabel / nicht akzeptabel].

---

## 5. Ergebnis und Freigabe

Ergebnis:
- [ ] Verarbeitung kann ohne Konsultation der Aufsichtsbehörde stattfinden
- [ ] Verarbeitung kann nach Umsetzung der Maßnahmen stattfinden
- [ ] Konsultation der Aufsichtsbehörde erforderlich (Restrisiko bleibt hoch)
- [ ] Verarbeitung darf NICHT stattfinden

Begründung: [Begründung]

**DSB:** _________________________________ Datum: _______
**Geschäftsführung:** ____________________ Datum: _______

---

## 6. Konsultation der Aufsichtsbehörde (Art. 36 DSGVO)
[Falls Restrisiko hoch bleibt: Pflicht zur vorherigen Konsultation]

Zuständige Behörde: [z.B. BayLDA, LfDI BW, LDI NRW]
Einreichungsdatum: [Datum]
Antwort erhalten: [Datum]
Ergebnis: [Beschreibung]
```
