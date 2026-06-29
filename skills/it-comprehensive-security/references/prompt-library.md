# Security Prompt Library

Ready-to-use prompts for common IT security tasks. Copy, adapt, and use directly.

---

## Penetration Testing

**Pentest Report erstellen**
```
Erstelle einen vollständigen Penetrationstest-Bericht für folgende Situation:
- Ziel: [Applikation/System/Netzwerk]
- Scope: [Was war in scope?]
- Findings: [Liste der gefundenen Schwachstellen mit Severity]
- Methodik: [OWASP / PTES / intern]
Formatiere den Bericht professionell mit Executive Summary, Risk Summary Tabelle,
detaillierten Findings (CVSS, CWE, Remediation) und Remediation Roadmap.
```

**Einzelnen Finding dokumentieren**
```
Dokumentiere folgendes Pentest-Finding vollständig:
- Titel: [Name der Schwachstelle]
- Betroffene Komponente: [URL / Endpoint / System]
- Schwere: [Critical/High/Medium/Low]
- Beweis: [Screenshot-Beschreibung / Request-Response / PoC]
Erstelle eine strukturierte Finding-Beschreibung mit CVSS Score, CWE, Impact-Analyse
und konkreten Remediationsschritten.
```

**CVSS Score berechnen**
```
Berechne den CVSS 3.1 Score für folgende Schwachstelle und erkläre jeden Vektor:
- Schwachstelle: [Beschreibung]
- Angriffsmethode: [Wie wird sie ausgenutzt?]
- Voraussetzungen: [Authentifizierung nötig? Netzwerkzugang?]
- Auswirkung: [Was kann ein Angreifer erreichen?]
```

---

## Incident Response

**IR-Bericht erstellen**
```
Erstelle einen vollständigen Incident Response Bericht für folgenden Vorfall:
- Incident-Typ: [Ransomware / Datenleck / BEC / Phishing etc.]
- Betroffene Systeme: [Anzahl und Art]
- Zeitlicher Ablauf: [Wann entdeckt, wann eingedämmt, wann behoben]
- Einfallsvektor: [Wie gelangte der Angreifer ins System?]
Struktur: Executive Summary, Timeline, Root Cause, IOCs, Containment-Schritte,
Empfehlungen, Lessons Learned.
```

**IR Playbook für spezifischen Incident-Typ**
```
Erstelle ein detailliertes Incident Response Playbook für:
Incident-Typ: [Ransomware / Phishing / Credential Compromise / Insider Threat / DDoS]
Umgebung: [Windows/Linux/Cloud/Hybrid]
Teamgröße: [Solo IT / kleines Team / SOC]
Incluye: Detection signals, sofortige Containment-Schritte (erste 30 min),
Investigation-Schritte, Eradication, Recovery, Post-Incident.
```

**IOC-Liste analysieren**
```
Analysiere folgende IOCs und erstelle eine Übersicht mit:
- IOC-Typ (IP/Domain/Hash/Registry Key)
- Wahrscheinlicher Kontext / Malware-Familie
- Empfohlene Blocking-Maßnahmen (Firewall/DNS/EDR)
- MITRE ATT&CK Mapping

IOCs:
[Liste hier einfügen]
```

---

## Hardening & Konfiguration

**Hardening Guide für System**
```
Erstelle einen vollständigen Hardening Guide für:
System: [Ubuntu 22.04 / Windows Server 2022 / nginx / Apache / Docker etc.]
Umgebung: [Produktiv-Server / DMZ / Cloud / On-Prem]
Compliance: [CIS Benchmark / BSI Grundschutz / ISO 27001]
Includiere: Konkrete Konfigurationsbefehle, Verifizierungsschritte, und
Referenz auf den jeweiligen CIS Control oder Compliance-Anker.
```

**Firewall-Regelset erstellen**
```
Erstelle ein Firewall-Regelset für folgendes Szenario:
Umgebung: [iptables / nftables / Windows Firewall / Palo Alto / FortiGate]
Server-Rolle: [Webserver / DB-Server / Mailserver / Jump Host]
Netzwerk: [Interne IPs / DMZ / Internet-facing]
Prinzip: Default-Deny, nur explizit erlaubter Traffic.
Includiere Regeln mit Kommentaren und eine Verifizierungs-Checkliste.
```

**PowerShell Hardening Script**
```
Schreibe ein PowerShell-Script das folgendes System härtet:
System: [Windows Server 2022 / Windows 11 / AD Domain Controller]
Bereiche: [Audit Policy / Defender ASR / PowerShell Logging / SMB / RDP]
Anforderungen: Das Script soll idempotent sein, einen -WhatIf Parameter haben,
alle Änderungen loggen und gegen CIS Benchmark Level [1/2] ausgerichtet sein.
```

---

## Compliance & Governance

**Gap Assessment Report**
```
Erstelle einen Gap Assessment Report für:
Framework: [ISO 27001 / NIS2 / SOC 2 / PCI-DSS / DORA]
Organisation: [KMU / Mittelstand / Enterprise / Behörde]
Branche: [Healthcare / Finance / Industrie / IT-Dienstleister]
Aktueller Reifegrad: [Anfänger / Grundlegend / Fortgeschritten]
Struktur: Executive Summary, Gap-Tabelle mit Priorisierung, Remediation Roadmap,
geschätzter Aufwand pro Maßnahme.
```

**Sicherheitsrichtlinie erstellen**
```
Erstelle eine professionelle IT-Sicherheitsrichtlinie für:
Thema: [Passwörter / Mobiles Arbeiten / BYOD / Datensicherung / Zugriffskontrolle / etc.]
Zielgruppe: [Alle Mitarbeiter / IT-Abteilung / Führungskräfte]
Unternehmenskontext: [Größe / Branche / Regulatorische Anforderungen]
Struktur: Zweck, Geltungsbereich, Richtlinienaussagen, Verantwortlichkeiten,
Ausnahmen, Verstöße, Überprüfungsintervall.
```

**Risikobewertung**
```
Führe eine Risikobewertung durch für:
Asset / Prozess: [Was soll bewertet werden?]
Bedrohungsszenarien: [Bekannte oder vermutete Bedrohungen]
Vorhandene Kontrollen: [Was ist bereits im Einsatz?]
Framework: [ISO 27005 / NIST RMF / qualitativ]
Output: Risiko-Register-Tabelle mit Likelihood, Impact, Risiko-Score,
Behandlungsoption und verantwortlicher Person.
```

---

## DevSecOps

**Security-Pipeline einrichten**
```
Entwirf eine DevSecOps-Pipeline für:
Plattform: [GitHub Actions / GitLab CI / Jenkins / Azure DevOps]
Sprache/Stack: [Python / Node.js / Java / .NET / Container-basiert]
Anforderungen: SAST, SCA, Secret Scanning, Container Scanning, DAST (optional)
Output: Vollständige Pipeline-Konfiguration (YAML) mit Erklärung jedes Security-Gates
und Empfehlung für Tools pro Stage.
```

**Secure Code Review**
```
Führe ein Security-focused Code Review durch für folgenden Code:
Sprache: [Python / JavaScript / Java / C# / etc.]
Kontext: [Was macht der Code? Welche Daten verarbeitet er?]

[CODE HIER EINFÜGEN]

Prüfe auf: Injection-Schwachstellen, Authentifizierungsprobleme, unsichere
Deserialisierung, Kryptographie-Fehler, Secrets im Code, gefährliche Funktionen.
Bewerte jeden Fund nach CWE und Severity, und schlage konkrete Fixes vor.
```

---

## Phishing & Awareness

**Phishing-Simulation planen**
```
Erstelle einen vollständigen Phishing-Simulationsplan für:
Organisation: [Größe / Branche]
Schwierigkeitsgrad: [Einfach / Mittel / Schwer]
Zielgruppe: [Alle Mitarbeiter / Finance / IT / Führungskräfte]
Tool: [GoPhish / KnowBe4 / Cofense / intern]
Includiere: Pretext-Szenarien, Metriken, Kommunikationsplan (vor/nach),
Follow-up Training, Eskalationspfad bei echten Meldungen.
```

**Awareness-Training Inhalt**
```
Erstelle Trainings-Content für ein Security Awareness Modul zu:
Thema: [Phishing / Passwörter / BEC / Social Engineering / Remote Work]
Zielgruppe: [Nicht-technische Mitarbeiter / Führungskräfte / Entwickler]
Format: [Präsentation / Merkblatt / Quiz / E-Learning-Skript]
Ton: [Formell / Locker / DACH-Kontext]
Länge: [5 min / 15 min / 30 min]
```

---

## Reporting & Kommunikation

**Executive Security Summary**
```
Erstelle eine Executive Summary für das Sicherheits-Reporting an die Geschäftsführung:
Zeitraum: [Quartal / Monat / Jahr]
Vorfälle: [Anzahl und Schwere]
KPIs: [Patch-Compliance, Phishing-Klickrate, offene Findings, etc.]
Ton: Nicht-technisch, risiko-orientiert, handlungsorientiert.
Maximal 1 Seite. Fokus auf Business Impact und benötigte Entscheidungen.
```

**Technisches Security Advisory**
```
Erstelle ein Security Advisory für folgende Schwachstelle:
CVE / Schwachstelle: [CVE-XXXX-XXXX oder Beschreibung]
Betroffene Systeme in unserer Umgebung: [Liste]
Verfügbare Patches: [Ja/Nein/Workaround]
Zielgruppe des Advisories: [IT-Team / Systemverantwortliche]
Struktur: Zusammenfassung, Betroffene Systeme, Risiko, Sofortmaßnahmen,
Patch-Anleitung, Verifizierung, Deadline.
```
