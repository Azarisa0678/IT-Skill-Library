# NIS2 & KRITIS — Referenzmodul

NIS2-Umsetzungsgesetz, Mindestmaßnahmen Art. 21, Meldepflichten, Gap-Assessment, KRITIS-Dachgesetz.

---

## NIS2 — Geltungsbereich und Einstufung

### Betroffene Einrichtungen (NIS2UmsuCG)

```
WESENTLICHE EINRICHTUNGEN (Essential Entities):
  → Strengere Aufsicht, höhere Sanktionen (max. 10 Mio. € oder 2% Umsatz)

  Energie:            Strom, Gas, Öl, Wärme, Wasserstoff
  Verkehr:            Luft, Bahn, Wasser, Straße
  Bankwesen:          Kreditinstitute
  Finanzmarkt:        Handelsplätze, CCPs, Zahlungsdienstleister
  Gesundheit:         Krankenhäuser, Labore, Pharmahersteller, Blutspende
  Wasser:             Trinkwasser, Abwasser
  Digitale Infrastruktur: IXPs, DNS, TLD, Rechenzentren, CDN, Vertrauensdienste, Telko
  ICT-Dienstemanagement: MSPs, MSSPs (B2B)
  Öffentliche Verwaltung: Bund, Länder (neu!)
  Weltraum:           Satellitenbetreiber

WICHTIGE EINRICHTUNGEN (Important Entities):
  → Leichtere Aufsicht (reaktiv), niedrigere Sanktionen (max. 7 Mio. € oder 1,4% Umsatz)

  Post und Kurier
  Abfallwirtschaft
  Chemikalien
  Lebensmittel
  Verarbeitendes Gewerbe (Medizinprodukte, Elektro, Maschinenbau, Fahrzeuge)
  Digitale Dienste (Online-Marktplätze, Suchmaschinen, soziale Netzwerke)
  Forschung (öffentliche Einrichtungen)

SCHWELLENWERTE (Standard-Regel):
  Wesentlich: ≥ 250 MA ODER ≥ 50 Mio. € Umsatz UND ≥ 43 Mio. € Bilanzsumme
  Wichtig:    ≥ 50 MA ODER ≥ 10 Mio. € Umsatz UND ≥ 10 Mio. € Bilanzsumme

AUSNAHMEN (automatisch wesentlich, unabhängig von Größe):
  - Alle TLD-Registries und DNS-Resolver
  - TK-Anbieter (bestimmte)
  - Einziger Anbieter in einem Mitgliedstaat für kritischen Dienst
  - Systemrelevanz für kritischen Sektor
```

---

## NIS2 Mindestmaßnahmen (Art. 21)

### Vollständige Maßnahmenliste

```markdown
## NIS2 Art. 21 — Mindestmaßnahmen Umsetzungsplan

### Art. 21 Abs. 2 lit. a: Risikoanalyse und Sicherheitskonzepte
- [ ] ISMS implementiert (ISO 27001, BSI IT-Grundschutz, oder äquivalent)
- [ ] Jährliche Risikoanalyse durchgeführt und dokumentiert
- [ ] IT-Sicherheitsrichtlinie von Geschäftsleitung genehmigt
- [ ] Informationssicherheitsbeauftragter (ISB) benannt
- [ ] Sicherheitskonzept vorhanden und aktuell (< 1 Jahr)

### Art. 21 Abs. 2 lit. b: Bewältigung von Sicherheitsvorfällen
- [ ] Incident Response Plan vorhanden und getestet (jährlich)
- [ ] Klassifizierungsschema für Vorfälle definiert (erheblich vs. nicht-erheblich)
- [ ] Meldeprozess etabliert: Erstmeldung ≤ 24h, Zwischenmeldung ≤ 72h, Abschluss ≤ 1 Monat
- [ ] BSI als zuständige Behörde registriert / bekannt
- [ ] 24/7-Erreichbarkeit für Sicherheitsvorfälle sichergestellt

### Art. 21 Abs. 2 lit. c: Business Continuity
- [ ] Business Continuity Plan (BCP) vorhanden
- [ ] RPO und RTO für alle kritischen Systeme definiert
- [ ] Backup-Konzept: 3-2-1-1-0-Regel umgesetzt
- [ ] Disaster Recovery Plan (DR) vorhanden
- [ ] BCP/DR jährlich getestet, Ergebnisse dokumentiert
- [ ] Krisenmanagement-Plan vorhanden (Krisenstab, Kommunikation)

### Art. 21 Abs. 2 lit. d: Sicherheit der Lieferkette
- [ ] Inventar aller IKT-Dienstleister und Lieferanten
- [ ] Risikoanalyse kritischer Lieferanten (jährlich)
- [ ] Vertragliche Sicherheitsanforderungen für kritische Dienstleister
- [ ] Exit-Strategie für jeden kritischen Dienstleister
- [ ] Subauftragnehmer der Dienstleister bekannt und bewertet

### Art. 21 Abs. 2 lit. e: Sicherheit bei Erwerb, Entwicklung, Wartung
- [ ] Sicherheitsanforderungen bei IT-Beschaffung (EVB-IT, Sicherheitsklauseln)
- [ ] SSDLC-Prozess für eigene Softwareentwicklung
- [ ] Patch-Management: kritische Patches ≤ 1 Woche
- [ ] Schwachstellen-Scanning (mindestens quartalsweise)
- [ ] Penetrationstests für kritische Systeme (jährlich)

### Art. 21 Abs. 2 lit. f: Wirksamkeit von Sicherheitsmaßnahmen
- [ ] Regelmäßige Überprüfung der Sicherheitsmaßnahmen
- [ ] KPIs für IT-Sicherheit definiert und gemessen
- [ ] Internes oder externes Audit (jährlich)
- [ ] Ergebnisse an Geschäftsleitung berichtet

### Art. 21 Abs. 2 lit. g: Kryptografie und Verschlüsselung
- [ ] Kryptokonzept vorhanden (welche Daten, welche Algorithmen, Schlüsselmanagement)
- [ ] Datenverschlüsselung at rest: AES-256
- [ ] Datenverschlüsselung in transit: TLS 1.2+ (TLS 1.3 bevorzugt)
- [ ] E-Mail-Verschlüsselung für sensitive Kommunikation (S/MIME oder PGP)
- [ ] VPN für Remote-Zugriff (IKEv2/IPsec oder WireGuard)

### Art. 21 Abs. 2 lit. h: Personalmanagement, Zugangskontrolle, Asset-Management
- [ ] IT-Asset-Inventar vollständig und aktuell (HW, SW, Daten)
- [ ] Berechtigungskonzept: Need-to-Know, Least Privilege
- [ ] MFA für alle privilegierten Konten und Remote-Zugänge (Pflicht!)
- [ ] Onboarding/Offboarding-Prozess mit IT-Anbindung
- [ ] Sicherheitsschulung für alle Mitarbeiter (jährlich)
- [ ] Hintergrundüberprüfung für Mitarbeiter mit kritischem Zugang

### Art. 21 Abs. 2 lit. i: Multi-Faktor-Authentifizierung
- [ ] MFA für alle externen Zugänge (VPN, RDP, Cloud-Dienste)
- [ ] MFA für privilegierte Konten (Admin, IT-Admins)
- [ ] MFA für E-Mail-Dienste (Exchange Online, Gmail for Business)
- [ ] Phishing-resistente MFA für kritische Systeme (FIDO2/Hardware-Token bevorzugt)
- [ ] Single Sign-On (SSO) mit MFA für interne Anwendungen

### Art. 21 Abs. 2 lit. j: Sicherheit der Kommunikation
- [ ] Netzwerksegmentierung (DMZ, VLAN-Trennung)
- [ ] Sichere Videokonferenzlösungen (E2E-Verschlüsselung)
- [ ] Sicherer Dateitransfer (SFTP, HTTPS, kein FTP/HTTP)
- [ ] E-Mail-Sicherheit: SPF, DKIM, DMARC konfiguriert
```

---

## NIS2 Meldepflichten

```
ERHEBLICHER VORFALL — Definition (NIS2 Art. 23):
  → Mindestens eines der folgenden:
  - Schwere Betriebsstörungen (> X% der Nutzer, Dauer > Y Stunden)
  - Finanzieller Schaden > Schwellenwert
  - Andere erhebliche materielle oder immaterielle Schäden
  - Datendiebstahl mit erheblichen Auswirkungen

MELDEKETTE (Deutschland → BSI):
  T+0:   Entdeckung / Verdacht auf erheblichen Vorfall
  T+24h: ERSTMELDUNG an BSI (auch wenn Details noch fehlen!)
         → Inhalt: Datum, Art, betroffene Dienste/Systeme, erster Eindruck
  T+72h: ZWISCHENMELDUNG an BSI
         → Aktueller Status, Schadensbewertung, ergriffene Maßnahmen
  T+30d: ABSCHLUSSBERICHT
         → Vollständige RCA, endgültige Schadensbeurteilung, Lessons Learned

MELDUNG AN BSI:
  Online-Formular: https://www.bsi.bund.de/meldestelle
  Notfall-Hotline: +49 (0) 228 99 9582-444 (24/7 für KRITIS)
  E-Mail:          meldestelle@bsi.bund.de

PARALLELE MELDEPFLICHTEN:
  Datenschutzverletzung:     Datenschutzbehörde ≤ 72h (DSGVO Art. 33)
  Finanzsektor (DORA):       BaFin ≤ 4h Erstmeldung
  Telekommunikation:         BNetzA (eigene Pflichten)
```

---

## NIS2 Gap-Assessment Vorlage

```markdown
## NIS2 Gap-Assessment — [Unternehmen] — [Datum]

### Bewertungsskala
0 = Nicht vorhanden | 1 = Teilweise | 2 = Weitgehend | 3 = Vollständig umgesetzt

| Art. 21 Anforderung | Ist-Stand | Soll | Gap | Maßnahme | Frist |
|---------------------|-----------|------|-----|----------|-------|
| Risikoanalyse + ISMS | 1 | 3 | 2 | ISO 27001 Projekt starten | Q2/2025 |
| Incident Response Plan | 0 | 3 | 3 | IR-Plan erstellen + testen | Q1/2025 |
| Business Continuity | 2 | 3 | 1 | DR-Test durchführen | Q1/2025 |
| Lieferkettensicherheit | 0 | 3 | 3 | Lieferanten-Assessment | Q3/2025 |
| Beschaffungssicherheit | 1 | 2 | 1 | Sicherheitsklauseln in IT-Verträge | Q2/2025 |
| Wirksamkeitsprüfung | 0 | 3 | 3 | Audit beauftragen | Q4/2025 |
| Kryptokonzept | 1 | 3 | 2 | Kryptokonzept dokumentieren | Q2/2025 |
| Asset + Berechtigungen | 2 | 3 | 1 | PAM einführen | Q3/2025 |
| MFA (Pflicht!) | 1 | 3 | 2 | MFA für alle Remote-Zugänge | SOFORT |
| Kommunikationssicherheit | 2 | 3 | 1 | DMARC auf p=reject | Q1/2025 |

**Gesamtbewertung:** [X/30 Punkten] = [X%]
**Kritische Lücken (Gap > 2):** [Liste]
**Priorität 1 (sofort):** MFA, Incident Response Plan
```

---

## KRITIS-Dachgesetz — Überblick

```
KRITIS-DACHGESETZ (Umsetzung geplant 2024/2025):
  → Ergänzt NIS2 um physische Resilienz-Anforderungen
  → Gilt für KRITIS-Betreiber aller 11 Sektoren

WESENTLICHE NEUERUNGEN:
  1. Resilienzpläne: Physische Sicherheit, nicht nur IT
  2. Hintergrundüberprüfungen: Für kritisches Personal
  3. Vorfallmeldungen: Auch für physische Vorfälle (Einbruch, Sabotage)
  4. BSI-Koordinierung: BSI als zentrale Behörde auch für physische Sicherheit
  5. KRITIS-Koordinierungsstellen: Je Sektor

PFLICHTEN FÜR KRITIS-BETREIBER:
  □ Registrierung beim BSI (Kontaktstelle 24/7)
  □ ISMS mit IT- UND physischer Sicherheit
  □ Resilienzplan (BCP für physische Vorfälle)
  □ Nachweis angemessener Sicherheit alle 2 Jahre
  □ Systeme zur Angriffserkennung (SAK) — IDS/SIEM
  □ Meldepflicht bei erheblichen Störungen
```
