# Pharma & Life Sciences Security — Referenzmodul

IT/OT-Sicherheit für Pharmaunternehmen, Medizinproduktehersteller, CROs, Labore.
Regulatorischer Fokus: EU GMP, EU Annex 11, CSV/GxP, FDA 21 CFR Part 11, GAMP 5, GDP.

---

## Regulatorische Landschaft Pharma (EU / DACH)

### Primäre Rechtsrahmen

| Rahmen | Geltungsbereich | IT-Kernanforderungen |
|---|---|---|
| **EU GMP (EudraLex Vol. 4)** | Arzneimittelhersteller EU | Qualitätssystem, Audit Trail, elektronische Aufzeichnungen |
| **Annex 11 (EU GMP)** | Computergestützte Systeme in GMP | Validierung, Audit Trail, Datensicherheit, Backup |
| **GAMP 5 (ISPE)** | Branchenstandard CSV | Risikobas. Validierung, Kategorie 1-5 Klassifizierung |
| **FDA 21 CFR Part 11** | US-Markt / FDA-regulierte Systeme | E-Signaturen, Audit Trails, System-Controls |
| **EU MDR (2017/745)** | Medizinprodukte inkl. Software (SaMD) | Cybersecurity als Produktsicherheit, SBOM, Post-Market |
| **EU IVDR (2017/746)** | In-vitro-Diagnostika | Gleiche Cybersecurity wie MDR |
| **GDP (Good Distribution Practice)** | Arzneimittelvertrieb | Datensicherheit in Lieferkette, Temperaturmonitoring |
| **NIS2** | Pharmahersteller als "Wichtige Einrichtung" | Meldepflichten, Mindestmaßnahmen Art. 21 |
| **DSGVO** | Klinische Studien, Patientendaten | Besondere Datenkategorien (Art. 9) |

### GAMP 5 Softwarekategorien

```
Kategorie 1: Infrastruktur-Software
  Beispiele: Betriebssysteme, Datenbanken, Virtualisierung
  Validierungsaufwand: Gering (Lieferant-Qualifizierung, IQ)
  IT-Sicherheit: Standard-Härtung, Patch-Management

Kategorie 3: Nicht-konfigurierte Standardsoftware
  Beispiele: Office, ERP (Standardfunktionen), LIMS (Out-of-the-Box)
  Validierungsaufwand: Mittel (IQ, OQ)
  IT-Sicherheit: Vendor-Sicherheitsbewertung, Zugangskontrolle

Kategorie 4: Konfigurierte Standardsoftware
  Beispiele: LIMS (konfiguriert), MES, QMS, SCADA
  Validierungsaufwand: Hoch (IQ, OQ, PQ)
  IT-Sicherheit: Konfigurationsmanagement, Change Control, Audit Trail

Kategorie 5: Individualsoftware (Custom)
  Beispiele: Eigenentwicklungen, Skripte in GMP-Prozessen
  Validierungsaufwand: Sehr hoch (vollständiger SDLC)
  IT-Sicherheit: SSDLC, Code-Review, Penetrationstest

WICHTIG: Kein Excel ohne CSV-Validierung in GMP-relevanten Prozessen!
```

---

## Computerized System Validation (CSV) / GxP

### Validierungsdokumentation (Annex 11)

```markdown
## Validierungsdokumentation — LIMS-System

### 1. User Requirements Specification (URS)
- Beschreibt WHAT das System tun soll (nicht HOW)
- GMP-relevante Anforderungen explizit kennzeichnen
- Basis für Risikoanalyse und Testplanung
- Beispiel-Anforderung: "Das System muss jeden Datenbankzugriff mit Benutzername,
  Zeitstempel und Aktion in einem unveränderbaren Audit Trail protokollieren"

### 2. Risikoanalyse (GAMP 5 / ICH Q9)
| Funktion | Auswirkung Ausfall | Wahrscheinlichkeit | Risiko | Maßnahme |
|----------|-------------------|-------------------|--------|---------|
| Probenregistrierung | Falsche Chargen-Zuordnung | Mittel | Hoch | PQ-Test, Doppelprüfung |
| Ergebnisfreigabe | Falsches Produkt freigegeben | Niedrig | Hoch | E-Signatur + 4-Augen |
| Backup/Restore | Datenverlust | Niedrig | Hoch | DR-Test, Backup-Monitoring |
| Audit Trail | Compliance-Verstoß | Niedrig | Kritisch | Permanente Überwachung |

### 3. Installationsqualifizierung (IQ)
- Hardware-Spezifikation erfüllt?
- Software-Version korrekt installiert?
- Betriebssystem-Einstellungen konform?
- Netzwerkkonfiguration dokumentiert?

### 4. Operationsqualifizierung (OQ)
- Funktionen gemäß URS getestet (positiv + negativ)
- Fehlermeldungen korrekt?
- Zugangskontrolle funktioniert?
- Audit Trail aktiv und unveränderbar?

### 5. Performance-Qualifizierung (PQ)
- Produktionsnahe Bedingungen
- Nachweis Systemperformance unter Last
- End-to-End Prozesse validiert
```

### Audit Trail — Technische Anforderungen

```
ANNEX 11 / 21 CFR PART 11 ANFORDERUNGEN:

Pflichtinhalte Audit Trail:
  ✓ Datum und Uhrzeit (UTC oder mit Zeitzone)
  ✓ Benutzer-ID (eindeutig, nicht teilbar)
  ✓ Aktion (Erstellen / Ändern / Löschen)
  ✓ Original-Wert (bei Änderungen: Vorher-Wert)
  ✓ Neuer Wert
  ✓ Begründung (bei kritischen Änderungen: Pflicht)

Technische Anforderungen:
  ✓ Unveränderbar (kein Admin darf Audit Trail löschen/ändern)
  ✓ Zeitstempel vom sicheren Server (nicht Client-Zeit!)
  ✓ NTP-synchronisiert (Abweichung < 1 Minute)
  ✓ Auch Lesezugriffe auf kritische Daten loggen (Annex 11 §12)
  ✓ Regelmäßige Überprüfung (SOP erforderlich!)
  ✓ Aufbewahrung: mindestens Produktlebenszyklus + 1 Jahr

DATENBANK-BEISPIEL (PostgreSQL Audit):
  -- Unveränderlicher Audit Trail via Trigger
  CREATE TABLE audit_log (
      id          BIGSERIAL PRIMARY KEY,
      table_name  TEXT NOT NULL,
      action      TEXT NOT NULL,        -- INSERT/UPDATE/DELETE
      old_data    JSONB,
      new_data    JSONB,
      user_name   TEXT NOT NULL,
      changed_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
  );

  -- Trigger verhindert DELETE auf audit_log
  CREATE RULE no_delete_audit AS ON DELETE TO audit_log DO INSTEAD NOTHING;
  CREATE RULE no_update_audit AS ON UPDATE TO audit_log DO INSTEAD NOTHING;
```

---

## GxP-IT-Sicherheitsanforderungen

### Zugangskontrolle (Annex 11 §12, 21 CFR §11.10)

```powershell
# Active Directory GxP-konforme Passwortrichtlinie
# Fine-Grained Password Policy für GxP-Benutzer
New-ADFineGrainedPasswordPolicy `
    -Name "PSO-GxP-Systeme" `
    -Precedence 10 `
    -MinPasswordLength 12 `
    -PasswordHistoryCount 24 `
    -MaxPasswordAge "90.00:00:00" `
    -MinPasswordAge "1.00:00:00" `
    -LockoutThreshold 5 `
    -LockoutDuration "00:30:00" `
    -LockoutObservationWindow "00:30:00" `
    -ComplexityEnabled $true `
    -ReversibleEncryptionEnabled $false

# Auf GxP-Benutzergruppe anwenden
Add-ADFineGrainedPasswordPolicySubject `
    -Identity "PSO-GxP-Systeme" `
    -Subjects "GxP-Systembenutzer"

# Automatische Session-Timeouts in GxP-Systemen:
# Annex 11 §12: "Systeme sollen nach einer definierten Periode der Inaktivität
# den Benutzer automatisch abmelden"
# Empfehlung: 15 Minuten (kritische Systeme: 5 Minuten)
```

### Change Control für GxP-Systeme

```markdown
## Change Control Prozess — GxP IT-Systeme

### Wann ist Change Control erforderlich?
- Alle Änderungen an validierten Systemen
- Patch-Installation auf GxP-Servern
- Konfigurationsänderungen (auch "kleine")
- Hardware-Austausch
- Betriebssystem-Upgrades
- Datenbankschema-Änderungen

### Change Control Kategorien
MINOR Change:    Sicherheitspatch ohne Funktionsänderung
                 → Risikoanalyse + verkürzte Testung + QA-Freigabe
MAJOR Change:    Neue Funktionalität, Konfigurationsänderung
                 → Vollständige Re-Qualifizierung betroffener Funktionen
EMERGENCY Change: Sicherheitskritisch, kein Zeit für normalen Prozess
                 → Sofortmaßnahme + Nachvalidierung innerhalb 30 Tage

### Patch-Management in GxP (Annex 11 §4.8)
Problem: GxP-Systeme können nicht einfach gepatcht werden
         (Patches könnten validierte Funktionen ändern)

Lösung — Risikobasierter Ansatz:
  1. Patch-Bewertung: Ändert der Patch GxP-relevante Funktionen?
  2. Ja → Change Control + Re-Qualifizierung
  3. Nein → Vereinfachter Prozess + Dokumentation
  4. Hersteller-Statement einholen (schriftlich)
  5. Test in Qualifizierungsumgebung VOR Produktiveinsatz
  6. QA-Freigabe dokumentieren
```

---

## Cybersecurity in klinischen Studien

```markdown
## Sicherheitsanforderungen Klinische Studien IT

### Elektronische Trial Master File (eTMF)
- 21 CFR Part 11 / EU Annex 11 konforme Systeme
- E-Signaturen: Eindeutig einem Benutzer zuordenbar + MFA
- Audit Trail: Alle Zugriffe und Änderungen
- Datenverschlüsselung at rest und in transit

### Remote Data Entry (EDC-Systeme)
- Multi-Faktor-Authentifizierung für alle Prüfärzte
- Rollenbasierte Zugangskontrolle (CRA, Investigator, Sponsor)
- Datentransfer nur verschlüsselt (TLS 1.3)
- Query-Management-Prozess mit Audit Trail

### Biomarker / Genomdaten
- Besondere Schutzanforderungen (DSGVO Art. 9: genetische Daten)
- Pseudonymisierung zwingend (Trennung: klinische Daten ↔ Genomdaten)
- Sichere Übertragung an Zentrallabore (SFTP/AS2 mit Verschlüsselung)
- Aufbewahrung: Studienspezifisch + Regulatory Requirement

### Ransomware-Schutz für Studiendaten
- Kritisch: Rohdaten aus klinischen Studien sind nicht reproduzierbar!
- Backup-Strategie: 3-2-1-1-0, Immutable Backups
- Offline-Kopie der Patientensicherheitsdaten (SAE-Reports)
- Notfallplan: Papierbasierte Datenerfassung wenn EDC nicht verfügbar
```

---

## Compliance-Checkliste Pharma / GxP

```markdown
## GxP IT-Compliance Checkliste — [Unternehmen] — [Datum]

### Systemvalidierung
- [ ] Alle GxP-relevanten Systeme im CSV-Inventar erfasst
- [ ] Risikoklassifizierung (GAMP 5 Kategorien) für alle Systeme
- [ ] Validierungsdokumentation (URS, RA, IQ, OQ, PQ) vollständig
- [ ] Validierungsstatus aktuell (letzte Re-Qualifizierung < 3 Jahre)
- [ ] Change Control für alle Systemänderungen dokumentiert

### Audit Trail
- [ ] Audit Trail in allen GxP-Systemen aktiv
- [ ] Audit Trail unveränderbar (technisch gesichert)
- [ ] Zeitstempel NTP-synchronisiert (Abweichung < 1 Min)
- [ ] Regelmäßige Audit-Trail-Überprüfung (SOP vorhanden, Nachweise)
- [ ] Aufbewahrungsdauer: mind. Produktlebenszyklus + 1 Jahr

### Zugangskontrolle
- [ ] Individuelle Benutzerkonten (keine Gruppenlogins in GxP-Systemen!)
- [ ] Passwortrichtlinie: 12 Zeichen, 90-Tage-Wechsel, 24er-History
- [ ] Session-Timeout: 15 Minuten (kritisch: 5 Minuten)
- [ ] MFA für alle Remote-Zugänge auf GxP-Systeme
- [ ] E-Signaturen: 21 CFR Part 11 / Annex 11 konform
- [ ] Berechtigungen: SOP für Anlage/Änderung/Sperrung, 4-Augen

### Datensicherheit
- [ ] Backup: täglich, mindestens 10 Jahre Aufbewahrung (Arzneimittel)
- [ ] Restore-Test: quartalsweise dokumentiert
- [ ] Datenverschlüsselung at rest + in transit
- [ ] DR-Plan für GxP-Systeme vorhanden und getestet

### Infrastruktur
- [ ] GxP-Server physisch oder logisch getrennt von Office-IT
- [ ] Kein Internetzugang für GxP-Systeme (nur wenn zwingend nötig)
- [ ] Patch-Management: Change-Control-Prozess für GxP-Patches
- [ ] Antivirus mit Whitelist-Ansatz (kein aggressives Blacklisting)

### Lieferanten
- [ ] Audit der Systemhersteller (GMP-Audit oder Fragebogen)
- [ ] Quality Technical Agreement (QTA) mit Softwarelieferanten
- [ ] Subprozessoren in Cloud-Diensten bekannt und auditiert
```

---

## Weiterführende Ressourcen

- **EMA**: [EU GMP Annex 11](https://www.ema.europa.eu/annex11)
- **ISPE**: GAMP 5 (Good Automated Manufacturing Practice)
- **FDA**: 21 CFR Part 11 Guidance
- **ICH**: Q9 Quality Risk Management, Q10 Pharmaceutical Quality System
- **PIC/S**: PI 011-3 Good Practices for Computerised Systems in GxP
- **BfArM**: Nationale Zulassungsbehörde DE
- **BSI**: Branchenspezifischer Sicherheitsstandard (B3S) Gesundheit (auch Pharma-relevant)
