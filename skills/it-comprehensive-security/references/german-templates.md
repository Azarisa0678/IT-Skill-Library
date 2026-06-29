# Deutsche Sicherheits-Templates

## IT-Sicherheitsrichtlinien

### Vorlage: Passwortrichtlinie

```markdown
# IT-Sicherheitsrichtlinie: Passwörter und Zugangsdaten
**Version:** 1.0 | **Datum:** [Datum] | **Gültig ab:** [Datum]
**Verantwortlich:** IT-Sicherheitsbeauftragter | **Genehmigt von:** Geschäftsführung

## 1. Zweck
Diese Richtlinie legt die Anforderungen an die Erstellung, Verwaltung und den Schutz von Passwörtern und Zugangsdaten fest, um unbefugten Zugriff auf Systeme und Daten zu verhindern.

## 2. Geltungsbereich
Diese Richtlinie gilt für alle Mitarbeitenden, Auftragnehmer und Dienstleister, die Zugang zu IT-Systemen der [Unternehmensname] haben.

## 3. Anforderungen

### 3.1 Passwortlänge und -komplexität
- Mindestlänge: 12 Zeichen (empfohlen: 16+)
- Kombination aus: Groß- und Kleinbuchstaben, Ziffern, Sonderzeichen
- Keine Wörterbuchwörter oder persönliche Informationen
- Keine Wiederverwendung der letzten 12 Passwörter

### 3.2 Multi-Faktor-Authentifizierung (MFA)
- MFA ist für alle externen Zugänge (VPN, Cloud-Dienste, Remote Desktop) verpflichtend
- MFA ist für alle privilegierten Konten (Administratoren) verpflichtend
- Empfohlen: FIDO2/Passkeys oder Authenticator-App (TOTP)
- SMS-basiertes MFA ist nur als letztes Mittel zulässig

### 3.3 Passwortmanagement
- Verwendung eines genehmigten Passwortmanagers ist verpflichtend
- Passwörter dürfen nicht in Klartextform gespeichert werden
- Passwörter dürfen nicht per E-Mail oder Chat weitergegeben werden

### 3.4 Privilegierte Konten
- Separate Konten für administrative Tätigkeiten
- Just-in-Time-Zugang bevorzugt (PIM)
- Passwörter für Dienstkonten werden im Passwort-Vault gespeichert

## 4. Verantwortlichkeiten
- **Mitarbeitende:** Einhaltung der Richtlinie, sofortige Meldung bei Verdacht auf Kompromittierung
- **IT-Abteilung:** Technische Durchsetzung, Schulung, Monitoring
- **Führungskräfte:** Sicherstellung der Compliance im eigenen Team

## 5. Verstöße
Verstöße gegen diese Richtlinie können disziplinarische Maßnahmen zur Folge haben.

## 6. Überprüfung
Diese Richtlinie wird jährlich oder bei wesentlichen Änderungen der Bedrohungslage überprüft.
```

---

### Vorlage: Richtlinie für mobiles Arbeiten / Homeoffice

```markdown
# IT-Sicherheitsrichtlinie: Mobiles Arbeiten und Homeoffice
**Version:** 1.0 | **Datum:** [Datum]

## 1. Zweck
Regelung der sicheren Nutzung von IT-Ressourcen außerhalb der Unternehmensräumlichkeiten.

## 2. Anforderungen

### 2.1 Endgeräte
- Nur vom Unternehmen bereitgestellte oder genehmigte Geräte
- Geräte müssen vollständig verschlüsselt sein (BitLocker / FileVault)
- Aktuelle Betriebssystem-Updates müssen installiert sein
- Unternehmens-EDR/Antivirensoftware muss aktiv sein

### 2.2 Netzwerk
- VPN-Nutzung ist bei Arbeit außerhalb des Unternehmens verpflichtend
- Öffentliche WLAN-Netzwerke nur mit aktivem VPN
- Heimnetzwerk: WPA3 oder WPA2 mit starkem Passwort
- Keine Nutzung unverschlüsselter WLAN-Hotspots

### 2.3 Physische Sicherheit
- Bildschirm bei Abwesenheit sperren (automatisch nach 5 Minuten)
- Kein Zugriff auf vertrauliche Informationen in der Öffentlichkeit
- Sichtschutzfolie bei Arbeit in öffentlichen Bereichen empfohlen
- Geräte nicht unbeaufsichtigt lassen

### 2.4 Datenspeicherung
- Unternehmensdaten nur auf genehmigten Cloud-Speichern (SharePoint, OneDrive)
- Keine privaten USB-Sticks oder externe Festplatten für Unternehmensdaten
- Vertrauliche Dokumente nach Bearbeitung sicher löschen/archivieren

## 3. Vorfallmeldung
Bei Verlust oder Diebstahl eines Geräts: sofortige Meldung an IT-Helpdesk
(Kontakt: [Telefon / E-Mail] | 24/7-Notfallnummer: [Nummer])
```

---

## Executive Security Report (Deutsch)

```markdown
# IT-Sicherheitsbericht — Q[X] [Jahr]
**Berichtszeitraum:** [Datum] – [Datum]
**Erstellt von:** [Name], IT-Sicherheit
**Klassifizierung:** Intern — Vertraulich

---

## Zusammenfassung für die Geschäftsführung

[2-3 Sätze zum Gesamtstatus: Reifegrad, wichtigste Fortschritte, wichtigste offene Risiken]

**Gesamtbewertung:** 🟢 Gut / 🟡 Verbesserungsbedarf / 🔴 Handlungsbedarf erforderlich

---

## Kennzahlen im Überblick

| Kennzahl | Ist-Wert | Zielwert | Trend |
|----------|----------|----------|-------|
| Sicherheitsvorfälle (P1/P2) | | 0 | ↑↓ |
| Ø Zeit bis zur Erkennung (MTTD) | | < 24h | ↑↓ |
| Patch-Compliance (kritisch) | | > 95% | ↑↓ |
| MFA-Abdeckung | | 100% | ↑↓ |
| Phishing-Klickrate | | < 5% | ↑↓ |
| Awareness-Schulung abgeschlossen | | > 95% | ↑↓ |

---

## Wesentliche Vorfälle im Berichtszeitraum

| Datum | Vorfall | Auswirkung | Status |
|-------|---------|------------|--------|
| | | | Behoben/Offen |

---

## Top-3-Risiken

### Risiko 1: [Titel]
- **Beschreibung:** [Was ist das Risiko?]
- **Mögliche Auswirkung:** [Finanziell, Reputational, Betrieblich]
- **Empfohlene Maßnahme:** [Was soll getan werden?]
- **Benötigte Ressourcen:** [Budget/Personal/Entscheidung]

---

## Compliance-Status

| Rahmenwerk | Status | Fälligkeit | Anmerkung |
|-----------|--------|-----------|-----------|
| NIS2 | 🟡 In Bearbeitung | [Datum] | Gap-Assessment läuft |
| ISO 27001 | 🟢 Konform | [Datum] | Nächstes Audit: [Datum] |
| DSGVO | 🟢 Konform | Laufend | |

---

## Entscheidungsbedarf der Geschäftsführung

1. **[Thema]:** [Kurze Beschreibung, Handlungsoptionen, Empfehlung]
2. **[Thema]:** [Kurze Beschreibung, Handlungsoptionen, Empfehlung]
```

---

## Pentest-Bericht (Deutsch)

```markdown
# Sicherheitsbewertung — Penetrationstest
**Auftraggeber:** [Unternehmensname]
**Prüfgegenstand:** [System/Applikation/Netzwerk]
**Prüfzeitraum:** [Datum] – [Datum]
**Erstellt von:** [Name/Team]
**Klassifizierung:** VERTRAULICH — Nur für interne Verwendung

---

## Zusammenfassung für die Geschäftsführung

[Nicht-technische Zusammenfassung: Was wurde geprüft, wie ist das Risikoniveau,
was sind die wichtigsten Sofortmaßnahmen]

**Gesamtrisikobewertung:** KRITISCH / HOCH / MITTEL / NIEDRIG

---

## Risikozusammenfassung

| Schweregrad | Anzahl |
|------------|--------|
| Kritisch | |
| Hoch | |
| Mittel | |
| Niedrig | |
| Informativ | |

---

## Befunde

### [KRIT-001] [Titel des Befunds]

**Schweregrad:** Kritisch
**CVSS-Score:** 9.8
**CWE/CVE:** CWE-xxx
**Betroffene Komponente:** [URL/System/Endpunkt]

**Beschreibung:**
[Was wurde gefunden und warum ist es gefährlich?]

**Nachweis:**
```
[Request, Response, Screenshot-Beschreibung, PoC-Befehl]
```

**Auswirkung:**
[Was kann ein Angreifer damit erreichen? Welcher Schaden ist möglich?]

**Empfehlung:**
1. [Sofortmaßnahme]
2. [Langfristige Lösung]
3. [Verifizierungsschritt]

**Referenzen:** [OWASP, BSI, Hersteller-Dokumentation]

---

## Maßnahmenplan

| Priorität | Befund | Aufwand | Zieldatum |
|-----------|--------|---------|-----------|
| Sofort | KRIT-001 | Mittel | Innerhalb 48h |
| Diese Woche | HOCH-001 | Gering | Innerhalb 1 Woche |

---

## Anhang

**A. Methodik:** [OWASP Testing Guide v4.2 / PTES / intern]
**B. Eingesetzte Tools:** [Burp Suite, nmap, etc.]
**C. Prüfumfang:** [Was war in scope / out of scope]
```

---

## Incident Response Meldung (NIS2-konform)

```markdown
# Sicherheitsvorfallmeldung
**Meldungstyp:** ☐ Erstmeldung (≤ 24h) ☐ Zwischenmeldung (≤ 72h) ☐ Abschlussbericht (≤ 1 Monat)
**Vorfalls-ID:** IR-[Jahr]-[Nummer]
**Meldendes Unternehmen:** [Name, Rechtsform, Adresse]
**Meldedatum/-zeit:** [Datum, Uhrzeit, Zeitzone]
**Ansprechpartner:** [Name, Funktion, E-Mail, Telefon]

---

## Vorfallsbeschreibung

**Art des Vorfalls:** ☐ Ransomware ☐ Datenleck ☐ DDoS ☐ Unbefugter Zugriff ☐ Sonstiges: ___

**Erste Erkennung:** [Datum, Uhrzeit]
**Betroffene Systeme/Dienste:** [Beschreibung]
**Geschätzte Anzahl betroffener Nutzer/Kunden:** [Zahl oder Schätzung]
**Geografische Auswirkung:** ☐ Lokal ☐ National ☐ Grenzüberschreitend

---

## Auswirkungen

**Betriebliche Auswirkungen:**
[Beschreibung der Dienstunterbrechungen, Ausfallzeiten]

**Wirtschaftliche Auswirkungen (Schätzung):**
[Direkte Kosten, Umsatzausfall, Wiederherstellungskosten]

**Auswirkungen auf andere Einrichtungen:**
[Bekannte oder mögliche Auswirkungen auf Lieferkette, Kunden, Partner]

---

## Technische Details (soweit bekannt)

**Angriffsvektor:** [Phishing / Schwachstelle / Insider / unbekannt]
**Indicators of Compromise (IOCs):**
- [IP-Adressen, Domains, Datei-Hashes]

---

## Ergriffene Maßnahmen

**Eindämmungsmaßnahmen:**
- [Liste der Maßnahmen]

**Wiederherstellungsstatus:**
☐ Eingedämmt ☐ In Wiederherstellung ☐ Vollständig wiederhergestellt

---

## Behördliche Meldepflichten
☐ BSI gemeldet (§ 8b BSIG) | Datum: ___
☐ Datenschutzbehörde (Art. 33 DSGVO) | Datum: ___
☐ Sektorspezifische Behörde | Datum: ___
```

---

## Awareness-Training Inhalte (Deutsch)

### Merkblatt: Phishing erkennen

```
SO ERKENNST DU PHISHING-E-MAILS
════════════════════════════════

✅ IMMER PRÜFEN:
  → Absender-Domain: acme-support.com ≠ acme.de
  → URL vor dem Klicken: Maus über Link halten
  → Unerwartete Anhänge: Makro-aktivierte Dateien (.docm, .xlsm)

⚠️  WARNSIGNALE:
  → Zeitdruck: "Sofort handeln", "24 Stunden"
  → Geheimhaltung: "Bitte nicht mit Kollegen besprechen"
  → Passwort- oder TAN-Anfragen per E-Mail
  → Unerwartete Gewinner, Rückerstattungen oder Drohungen

🆘 WENN DU GEKLICKT HAST:
  → Sofort IT-Helpdesk anrufen: [NUMMER]
  → Passwort ändern
  → Gerät nicht weiter nutzen

📧 PHISHING MELDEN:
  → Outlook: "Phishing melden"-Button
  → E-Mail an: security@[unternehmen].de
```
