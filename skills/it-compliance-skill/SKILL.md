---
name: it-compliance-skill
description: >
  IT-Compliance und Datenschutz fuer DACH-Unternehmen. Alle Outputs auf Deutsch.
  DSGVO, VVT, Verzeichnis der Verarbeitungstaetigkeiten, DSFA, AVV, Auftragsverarbeitung,
  Betroffenenrechte, Datenpanne, Datenschutzverletzung, 72 Stunden, DSB, Datenschutzbeauftragter,
  TOM, technische organisatorische Massnahmen, BDSG, Artikel 32, Artikel 33, Artikel 35,
  BSI Grundschutz, IT-Grundschutz, Schutzbedarfsfeststellung, Sicherheitskonzept, Baustein,
  ISO 27001, ISMS, Annex A, SoA, Statement of Applicability, Informationssicherheit,
  internes Audit, Management Review, Nonconformity, Zertifizierung, VERINICE,
  NIS2, NIS-2, Meldepflicht, erheblicher Vorfall, wesentliche Einrichtung,
  KRITIS, kritische Infrastruktur, Registrierung BSI, Angriffserkennung,
  IT-Revision, Compliance, Audit, Gap-Assessment, Risikoanalyse, Betriebsvereinbarung.
  Outputs: Deutsche Vorlagen, Checklisten, Mustervertraege, VVT-Eintraege, SoA-Tabellen.
---

# IT-Compliance & Datenschutz Skill

## Leitprinzipien

**Dokumentation ist Compliance.** Was nicht dokumentiert ist, gilt als nicht umgesetzt — im Audit und vor Behörden.
**Privacy by Design.** Datenschutz von Anfang an einbauen, nicht nachträglich.
**Verhältnismäßigkeit.** Maßnahmen müssen zum Risiko passen — kein Overengineering, aber kein Underengineering.
**Rechenschaftspflicht (Art. 5 Abs. 2 DSGVO).** Der Verantwortliche muss Compliance jederzeit nachweisen können.
**Kontinuierliche Verbesserung.** ISMS und Datenschutz sind kein Projekt, sondern ein dauerhafter Prozess.

---

## Referenzmodule

- `references/dsgvo-vvt.md` — VVT-Vorlagen, Datenpanne-Checkliste (72h), TOM-Dokumentation, Betroffenenrechte
- `references/bsi-grundschutz.md` — Schutzbedarfsfeststellung, Bausteinauswahl, Sicherheitskonzept-Struktur, Zertifizierungsprozess
- `references/iso27001-audit.md` — ISO 27001:2022 Struktur, SoA-Vorlage, Jahres-Auditplan, Checkbögen, Management-Review, Top-Findings
- `references/nis2-kritis.md`
- `references/evals.md` — Testfälle und Qualitätskriterien — Geltungsbereich, Art. 21 Maßnahmencheckliste, Meldekette, Gap-Assessment, KRITIS-Dachgesetz

Routing:
- DSGVO / VVT / Datenpanne / TOM / AVV → `dsgvo-vvt.md`
- BSI IT-Grundschutz / Schutzbedarfsfeststellung / Sicherheitskonzept → `bsi-grundschutz.md`
- ISO 27001 / ISMS / Audit / SoA / Management Review → `iso27001-audit.md`
- NIS2 / KRITIS / Meldepflicht / Gap-Assessment → `nis2-kritis.md`

---

## DSGVO — Schnellreferenz

```markdown
## Wichtigste Fristen im Überblick

| Pflicht | Frist | Basis |
|---------|-------|-------|
| Datenpanne → Datenschutzbehörde | 72 Stunden | Art. 33 DSGVO |
| Auskunftsanfrage beantworten | 1 Monat (verlängerbar auf 3) | Art. 12 DSGVO |
| Löschungsanfrage bearbeiten | Unverzüglich | Art. 17 DSGVO |
| AVV abschließen | Vor Beginn der Verarbeitung | Art. 28 DSGVO |
| DSFA durchführen | Vor Beginn der Verarbeitung | Art. 35 DSGVO |

## DSB-Pflicht (§ 38 BDSG)?
→ ≥ 20 Personen mit automatisierter Datenverarbeitung ODER
→ Umfangreiche Verarbeitung Art.-9-Daten ODER
→ Systematische Überwachung

## DSFA-Pflicht (Art. 35)?
→ Scoring/Profiling mit erheblichen Auswirkungen
→ Neue Technologien + hohes Risiko
→ DSK-Blacklist-Verarbeitung
```

→ Vollständige VVT-Vorlage (3 Muster-Einträge), Datenpanne-Protokoll, TOM-Dokumentation: `references/dsgvo-vvt.md`

---

## BSI IT-Grundschutz — Schnellreferenz

```markdown
## Schutzbedarfskategorien
Normal    → Schaden < 50.000 €     → Standard-Maßnahmen
Hoch      → Schaden 50k–500k €     → Erhöhte Maßnahmen
Sehr hoch → Schaden > 500.000 €    → Maximale Maßnahmen

## Absicherungsvarianten
Basis-Absicherung:    Einstieg, kleine Behörden/Unternehmen
Standard-Absicherung: Regelfall, Voraussetzung für ISO 27001 auf IT-GS
Kern-Absicherung:     Nur für Kronjuwelen (hochsensible Assets)

## Pflicht-Bausteine (immer)
ISMS.1 | ORP.1 | ORP.2 | ORP.3 | ORP.4 | CON.1 | CON.2 | CON.3
DER.1  | DER.2.1 | NET.1.1 | NET.3.2
```

→ Bausteinauswahl KMU-Mapping, Sicherheitskonzept-Dokumentstruktur, Zertifizierungsprozess: `references/bsi-grundschutz.md`

---

## ISO 27001:2022 — Schnellreferenz

```markdown
## Pflichtdokumentation ISO 27001:2022
- Informationssicherheitsrichtlinie (Kap. 5.2)
- Statement of Applicability — SoA (Kap. 6.1.3)
- Risikobeurteilung und -behandlung (Kap. 6.1.2/6.1.3)
- Kompetenznachweise (Kap. 7.2)
- Interne Audit-Ergebnisse (Kap. 9.2)
- Management-Review-Protokoll (Kap. 9.3)
- Nichtkonformitäten + Korrekturmaßnahmen (Kap. 10.1)

## Neue Controls 2022 (gegenüber 2013)
A.5.7  Threat Intelligence | A.5.23 Cloud Security
A.8.9  Konfigurationsmanagement | A.8.12 DLP
A.8.16 Monitoring | A.8.28 Sichere Programmierung
```

→ SoA-Vorlage (15 Controls), Jahres-Auditplan, Audit-Checkbogen Zugangskontrolle, Management-Review-Agenda, Top-10-Findings: `references/iso27001-audit.md`

---

## NIS2 — Schnellreferenz

```markdown
## Bin ich betroffen?
Wesentlich: ≥ 250 MA oder ≥ 50 Mio. € Umsatz + Sektor (Energie, Gesundheit, Bank...)
Wichtig:    ≥ 50 MA oder ≥ 10 Mio. € Umsatz + erweiterter Sektor

## Meldepflichten (erheblicher Vorfall)
T+24h:  Erstmeldung an BSI (auch bei unvollständigen Infos!)
T+72h:  Zwischenmeldung
T+30d:  Abschlussbericht mit RCA

## Sofortmaßnahme (Priorität 1)
□ MFA für alle Remote-Zugänge (Art. 21 Abs. 2 lit. i)
□ Incident Response Plan erstellen + testen
□ BSI-Kontaktstelle registrieren
```

→ Vollständige Art.-21-Checkliste (10 Bereiche), Gap-Assessment-Vorlage, Meldekette: `references/nis2-kritis.md`

---

## Output-Formate je Aufgabentyp

| Aufgabe | Format |
|---|---|
| VVT-Eintrag erstellen | Markdown-Tabelle mit allen Art.-30-Pflichtfeldern |
| Datenpanne dokumentieren | Ausfüllbares Protokoll mit Meldepflicht-Check |
| TOM-Dokumentation | Strukturiertes Dokument nach 9 Schutzkategorien |
| BSI-Sicherheitskonzept | Vollständige Dokumentstruktur (10 Kapitel) |
| ISO 27001 SoA | Tabelle mit Control, Anwendbar/Nicht, Begründung, Umsetzung |
| Audit-Checkbogen | Prüfpunkte mit Methode + Ergebnis-Spalte |
| NIS2 Gap-Assessment | Bewertungstabelle 0-3 + Maßnahmen + Fristen |
| Richtlinie / Policy | Vollständiges Dokument: Zweck, Geltungsbereich, Anforderungen, Revision |
