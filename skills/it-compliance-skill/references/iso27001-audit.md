# ISO 27001 & Audit — Referenzmodul

ISO 27001:2022, Annex A Controls, Auditplanung, interne Revision, Zertifizierungsvorbereitung.

---

## ISO 27001:2022 — Strukturübersicht

```
HAUPTKAPITEL (normativ):
  Kap. 4:  Kontext der Organisation (4.1-4.4)
  Kap. 5:  Führung (5.1-5.3)
  Kap. 6:  Planung (6.1-6.3)
  Kap. 7:  Unterstützung (7.1-7.5)
  Kap. 8:  Betrieb (8.1-8.3)
  Kap. 9:  Leistungsbewertung (9.1-9.3)
  Kap. 10: Verbesserung (10.1-10.2)

ANNEX A CONTROLS (2022: 93 Controls in 4 Themen):
  A.5  Organisatorische Controls   (37 Controls)
  A.6  Personenbezogene Controls   (8 Controls)
  A.7  Physische Controls          (14 Controls)
  A.8  Technologische Controls     (34 Controls)

NEU IN 2022 (gegenüber 2013):
  A.5.7  Bedrohungsinformationen (Threat Intelligence)
  A.5.23 Informationssicherheit bei Cloud-Nutzung
  A.5.30 IKT-Bereitschaft für Business Continuity
  A.7.4  Physische Sicherheitsüberwachung
  A.8.9  Konfigurationsmanagement
  A.8.10 Löschung von Informationen
  A.8.11 Datenmaskierung
  A.8.12 Verhinderung von Datenlecks (DLP)
  A.8.16 Überwachungsaktivitäten
  A.8.23 Web-Filterung
  A.8.28 Sichere Programmierung
```

---

## Statement of Applicability (SoA)

```markdown
## Statement of Applicability (SoA) — [Unternehmen GmbH]
**Version:** 1.2  |  **Datum:** TT.MM.JJJJ  |  **ISMS-Scope:** [Beschreibung]

| Control | Titel | Anwendbar | Begründung / Ausschlussgrund | Umsetzung |
|---------|-------|-----------|------------------------------|-----------|
| A.5.1  | Richtlinien für Informationssicherheit | Ja | Grundlage des ISMS | IT-Sicherheitsrichtlinie v2.1 |
| A.5.2  | Rollen und Verantwortlichkeiten | Ja | Kap. 5.3 ISO 27001 | ISB-Stellenbeschreibung, Org-Chart |
| A.5.7  | Bedrohungsinformationen | Ja | Awareness über aktuelle Bedrohungen nötig | BSI-Warnmeldungen, ACS-Mitgliedschaft |
| A.5.23 | Cloud-Nutzung | Ja | M365, Azure, AWS in Verwendung | Cloud-Sicherheitsrichtlinie, AVV |
| A.6.3  | Awareness, Schulung, Training | Ja | Mitarbeitersensibilisierung | Jährliche ISMS-Schulung, Phishing-Tests |
| A.7.1  | Physische Sicherheitsperimeter | Ja | RZ und Bürogebäude absichern | Zutrittskontrolle, Alarmanlage |
| A.8.2  | Privilegierte Zugriffsrechte | Ja | Admin-Konten besonders absichern | PAM-System, PIM, 4-Augen |
| A.8.5  | Sichere Authentifizierung | Ja | Identitätsschutz | MFA für alle extern+privilegiert |
| A.8.8  | Management technischer Schwachstellen | Ja | Patch-Management | Quartalsscan, Patch ≤ 1 Woche |
| A.8.15 | Protokollierung | Ja | Auditierbarkeit, IR-Unterstützung | SIEM, 90 Tage Aufbewahrung |
| A.8.24 | Kryptografie | Ja | Datenschutz und Vertraulichkeit | Kryptokonzept, AES-256, TLS 1.3 |
| A.5.10 | Zulässige Nutzung von Assets | Ja | Verhindern von Datenmissbrauch | Nutzungsrichtlinie, DLP |
| A.5.19 | Informationssicherheit in Lieferantenbeziehungen | Ja | Supply Chain Risiko | Lieferantenbewertung, AVV |
| A.8.13 | Informationssicherungs-Backup | Ja | Verfügbarkeit, DR | Backup-Konzept, DR-Test |
| A.7.9  | Sicherheit von Assets außerhalb der Räumlichkeiten | Ja | Homeoffice, mobile Geräte | MDM, Geräteverschlüsselung, VPN |
| A.8.32 | Änderungsmanagement | Ja | Kontrollierte Änderungen | Change-Management-Prozess |
```

---

## Internes Audit — Planung und Durchführung

### Jahres-Auditplan

```markdown
## ISMS Interner Audit-Jahresplan [Jahr]

### Auditziele
- Wirksamkeit des ISMS gemäß ISO 27001:2022 überprüfen
- Umsetzung der Sicherheitsmaßnahmen verifizieren
- Basis für Management-Review liefern

### Auditprogramm

| Quartal | Auditbereich | Schwerpunkt Controls | Auditor | Geplant | Status |
|---------|-------------|---------------------|---------|---------|--------|
| Q1 | Zugangskontrolle + Berechtigungen | A.5.15, A.5.16, A.8.2, A.8.5 | [Intern/Extern] | März | - |
| Q1 | Incident Management | A.5.24, A.5.25, A.5.26 | [Intern] | März | - |
| Q2 | Patch + Schwachstellen-Mgmt | A.8.8, A.8.19 | [Intern] | Juni | - |
| Q2 | Backup + Business Continuity | A.8.13, A.5.29, A.5.30 | [Intern] | Juni | - |
| Q3 | Physische Sicherheit + Infrastruktur | A.7.1-A.7.13 | [Intern] | Sept | - |
| Q3 | Lieferantenmanagement | A.5.19-A.5.22 | [Intern] | Sept | - |
| Q4 | Kryptografie + Datenschutz | A.8.24, DSGVO | [Extern] | Nov | - |
| Q4 | Awareness + Personal | A.6.3, A.6.4 | [Intern] | Nov | - |

### Vor Zertifizierungsaudit (alle Bereiche)
- Vollständiges Programm 3 Monate vor Zertifizierungsaudit
- Alle Findings schließen oder Maßnahmenplan vorlegen
```

### Audit-Checkbogen — Zugangskontrolle (Beispiel)

```markdown
## Audit-Checkliste: Zugangskontrolle (A.5.15, A.5.16, A.8.2, A.8.5)
Auditor: ___________  Datum: ___________  Abteilung: IT

| Nr. | Prüfpunkt | Methode | Ergebnis | Befund |
|-----|-----------|---------|----------|--------|
| 1 | Berechtigungskonzept vorhanden und aktuell (< 1 Jahr)? | Dokumentenprüfung | Ja / Nein | |
| 2 | Ist das Minimalprinzip (Need-to-Know) implementiert? | Stichprobe: 5 Benutzerkonten | | |
| 3 | Werden Berechtigungen bei Rollenwechsel angepasst (< 24h)? | Prozessinterview + Ticketsystem | | |
| 4 | Werden Berechtigungen bei Austritt sofort gesperrt? | Stichprobe: letzte 3 Austritte | | |
| 5 | Gibt es eine jährliche Rezertifizierung? | Nachweis vorlegen | | |
| 6 | Sind Admin-Konten von normalen Konten getrennt? | Stichprobe AD-Gruppen | | |
| 7 | Ist MFA für alle Remote-Zugänge aktiv? | Log-Auszug / Screenshot | | |
| 8 | Sind Service-Accounts mit Minimalrechten konfiguriert? | Stichprobe: 3 Service-Accounts | | |
| 9 | Werden fehlgeschlagene Logins protokolliert + alarmiert? | SIEM-Demo | | |
| 10 | Gibt es ein PAM-System für privilegierte Konten? | System-Demo | | |

**Gesamtbewertung:**
  Conformance: [X/10]
  Kritische Findings (sofortiger Handlungsbedarf): ___
  Major Nonconformity: ___  Minor Nonconformity: ___  Observations: ___
```

---

## Management-Review (Kap. 9.3)

```markdown
## ISMS Management-Review — [Unternehmen] — [Datum]

**Teilnehmer:** Geschäftsführung, ISB, IT-Leitung, ggf. DSB
**Frequenz:** Mindestens jährlich (ISO 27001 Pflicht)

### Tagesordnung (Mindestinhalt nach ISO 27001 Kap. 9.3):

#### 1. Status aus vorigen Management-Reviews
Offene Maßnahmen aus letztem Review:
| Maßnahme | Verantwortlich | Frist | Status |
|----------|---------------|-------|--------|
| MFA einführen | IT-Admin | 31.03. | ✅ Abgeschlossen |
| Notfallübung durchführen | ISB | 30.06. | ⚠️ Verschoben auf Q3 |

#### 2. Änderungen externer und interner Themen (Kap. 4.1)
- Neue regulatorische Anforderungen (NIS2, DSGVO-Änderungen)
- Neue Bedrohungen (BSI Lagebericht, aktuelle Angriffswellen)
- Organisatorische Änderungen (neue Standorte, Fusionen, Personalwechsel)

#### 3. Informationssicherheits-Performance (Kap. 9.1)
| KPI | Ziel | Ist | Trend |
|-----|------|-----|-------|
| Kritische Patches ≤ 1 Woche | 100% | 94% | ↗ |
| Sicherheitsvorfälle (schwer) | ≤ 2/Jahr | 1 | ✅ |
| Mitarbeiter-Awareness-Score | ≥ 80% | 87% | ✅ |
| Phishing-Klickrate | ≤ 5% | 3,2% | ✅ |
| Backup-Tests erfolgreich | 100% | 100% | ✅ |

#### 4. Audit-Ergebnisse
- Ergebnisse interner Audits Q1-Q4
- Externe Zertifizierungsaudit-Findings (falls vorhanden)
- Schließungsstatus der Nonconformities

#### 5. Risikobehandlung
- Neue Risiken identifiziert
- Bestehende Risiken eskaliert/reduziert
- Risk Appetite noch aktuell?

#### 6. Ressourcenbedarf
- Personalressourcen IT-Sicherheit ausreichend?
- Budget-Anforderungen für nächstes Jahr
- Tool-Bedarf (SIEM, EDR, PAM etc.)

#### 7. Verbesserungsmöglichkeiten
- Quick Wins (sofort umsetzbar)
- Mittelfristige Maßnahmen (6-12 Monate)
- Strategische Projekte (> 1 Jahr)

### Beschlüsse:
[Dokumentation der Entscheidungen und neuen Maßnahmen]

### Nächster Management-Review:
Datum: ___________
```

---

## Häufige Audit-Findings und Abhilfemaßnahmen

```markdown
## Top-10 ISO 27001 Audit-Findings (DACH)

| # | Finding | Häufigkeit | Typische Ursache | Abhilfe |
|---|---------|------------|-----------------|---------|
| 1 | Kein aktuelles Asset-Inventar | ★★★★★ | Manuelle Pflege, vergessen | CMDB einführen, automatische Discovery |
| 2 | Berechtigungen nach Austritt nicht gesperrt | ★★★★★ | Kein Offboarding-Prozess | HR-IT-Schnittstelle, automatische Sperrung |
| 3 | Kein dokumentierter IR-Plan | ★★★★☆ | "Haben wir im Kopf" | IR-Plan schreiben + jährlich testen |
| 4 | MFA nicht für alle Remote-Zugänge | ★★★★☆ | Legacy-Systeme, Widerstände | Conditional Access, Migration-Plan |
| 5 | Backup ohne Restore-Test | ★★★★☆ | "Läuft ja" | Quartalsweise Restore-Tests protokolliert |
| 6 | Kein Lieferanten-Assessment | ★★★☆☆ | Unbekannte Supply Chain | Lieferanten-Risikoanalyse + AVV |
| 7 | Veraltete Sicherheitsrichtlinien | ★★★☆☆ | Einmalig erstellt, nie überprüft | Jährliche Review-Pflicht in Richtlinie |
| 8 | Schwachstellenscans fehlen | ★★★☆☆ | Tool fehlt, Aufwand | Tenable/Qualys/OpenVAS quartalsweise |
| 9 | Kein Kryptokonzept | ★★★☆☆ | TOM vorhanden, aber nicht dokumentiert | Kryptokonzept nach BSI TR-02102 |
| 10 | Awareness-Schulung nicht nachweisbar | ★★★☆☆ | Mündlich durchgeführt | E-Learning mit Bestätigung, Phishing-Test |
```
