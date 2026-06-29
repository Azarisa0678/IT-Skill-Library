# Eval-Testfälle: IT-Compliance & Datenschutz Skill

## Kategorie 1: Trigger-Tests

### T001 — VVT-Eintrag erstellen
**Input:** "Erstelle einen VVT-Eintrag für unser CRM-System"
**Kriterien:**
- ✅ Alle Art.-30-Pflichtfelder vorhanden
- ✅ Rechtsgrundlage explizit genannt (Art. 6 Abs. 1 lit. X)
- ✅ Kategorien Betroffener und Datenkategorien getrennt
- ✅ Löschfristen konkret (nicht "so lange nötig")
- ✅ TOM-Abschnitt vorhanden
- ✅ Drittlandübermittlung adressiert
- ✅ Vollständig auf Deutsch

### T002 — DSFA-Schwellwertprüfung
**Input:** "Wir führen Videoüberwachung im Büro ein. Brauchen wir eine DSFA?"
**Kriterien:**
- ✅ Ja/Nein-Entscheidung mit Begründung
- ✅ Art. 35 DSGVO referenziert
- ✅ Systematische Überwachung öffentl. Bereiche als Grund
- ✅ Konsultation DSB empfohlen
- ✅ Nächste Schritte konkret

### T003 — AVV prüfen
**Input:** "Wir nutzen Microsoft 365. Brauchen wir einen AVV?"
**Kriterien:**
- ✅ Ja, klar und begründet
- ✅ Art. 28 DSGVO referenziert
- ✅ Microsoft DPA (Datenschutzanhang) erwähnt
- ✅ Hinweis auf Standard-AVV im M365-Admin-Center
- ✅ Subunternehmer-Thema adressiert

### T004 — Datenpanne 72h
**Input:** "Wir hatten einen Phishing-Angriff und Mitarbeiterdaten wurden abgegriffen. Was müssen wir tun?"
**Kriterien:**
- ✅ 72-Stunden-Frist klar genannt
- ✅ Zuständige Behörde (Landesbehörde) erwähnt
- ✅ Interne Dokumentation Pflicht
- ✅ Betroffenen-Information prüfen (Art. 34)
- ✅ Sofortmaßnahmen zuerst (Eindämmung)
- ✅ Meldepflicht-Checkliste oder Protokoll

### T005 — Betroffenenrecht Auskunft
**Input:** "Ein Mitarbeiter fordert Auskunft über seine gespeicherten Daten. Wie gehe ich vor?"
**Kriterien:**
- ✅ 1-Monats-Frist genannt (Art. 15)
- ✅ Verlängerung auf 3 Monate möglich mit Begründung
- ✅ Identitätsprüfung des Antragstellers erwähnt
- ✅ Welche Daten mitzuteilen sind (vollständig)
- ✅ Musterantwort oder Template bereitgestellt
- ✅ Unentgeltlich

### T006 — BSI Grundschutz Einstieg
**Input:** "Wie starte ich mit der BSI IT-Grundschutz-Implementierung?"
**Kriterien:**
- ✅ 6-Phasen-Vorgehensweise erklärt
- ✅ Strukturanalyse als Basis
- ✅ Schutzbedarfsfeststellung erläutert
- ✅ Passende Bausteine (mind. 3 genannt)
- ✅ IT-Grundschutz-Kompendium erwähnt
- ✅ Realistischer Zeitplan

### T007 — TOM-Katalog
**Input:** "Erstelle einen TOM-Katalog für unser Unternehmen nach Art. 32 DSGVO"
**Kriterien:**
- ✅ Alle 7 Kontrollbereiche vorhanden (Zutritt, Zugang, Zugriff, Weitergabe, Eingabe, Verfügbarkeit, Trennung)
- ✅ Konkrete Maßnahmen je Bereich
- ✅ Status-Spalte (Umgesetzt/Teilweise/Offen)
- ✅ Auf Deutsch
- ✅ Tabellarisches Format

### T008 — Compliance Reporting
**Input:** "Erstelle eine Management-Summary über den DSGVO-Compliance-Status"
**Kriterien:**
- ✅ KPI-Tabelle vorhanden
- ✅ Gesamtstatus (Ampel) vergeben
- ✅ Offene Maßnahmen mit Verantwortlichem und Frist
- ✅ Nicht-technische Sprache für Geschäftsführung
- ✅ Vollständig auf Deutsch

## Kategorie 2: Qualitäts-Tests

### Q001 — Deutsche Ausgabe immer
**Alle Outputs müssen auf Deutsch sein** (außer explizit EN angefordert)
- ✅ Vorlagen auf Deutsch
- ✅ Gesetzesreferenzen (DSGVO, nicht GDPR)
- ✅ Deutsche Behörden genannt (BfDI, Landesdatenschutzbehörden)

### Q002 — Korrekte Fristen
- ✅ Auskunft: 1 Monat (verlängerbar auf 3)
- ✅ Datenpanne an Behörde: 72 Stunden
- ✅ Datenpanne an Betroffene: "unverzüglich"
- ✅ Löschung: unverzüglich nach Wegfall des Zwecks

## Kategorie 3: Negativ-Tests

### N001 — Technische Security-Frage
**Input:** "Führe einen Penetrationstest durch"
**Erwartetes Verhalten:**
- ❌ IT-Compliance Skill nicht primär zuständig
- ✅ IT Security Skill empfehlen

### N002 — M365 Admin
**Input:** "Erstelle einen Conditional Access Policy"
**Erwartetes Verhalten:**
- ❌ Compliance Skill nicht zuständig
- ✅ M365 Admin Skill empfehlen

## Bewertungsschema
**Mindest-Score:** 4/5 auf alle T-Tests
