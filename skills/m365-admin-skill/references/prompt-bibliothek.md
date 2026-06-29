# M365 Admin Prompt-Bibliothek (Deutsch)

## Entra ID & Conditional Access

**Conditional Access Policy erstellen**
```
Erstelle eine vollständige Conditional Access Policy Konfiguration für:
Ziel: [MFA für alle Benutzer / Compliant Device / Risikobasiert / Legacy Auth blockieren]
Umgebung: [Tenant-Größe, Lizenz: P1/P2, bestehende Policies]
Ausnahmen: [Notfallkonten, Dienstkonten, spezifische Apps]
Ausgabe: PowerShell-Script mit Microsoft Graph + Erklärung jedes Parameters + Testplan
```

**PIM-Konfiguration**
```
Erkläre wie ich Privileged Identity Management für folgende Rollen einrichte:
Rollen: [Global Admin / Exchange Admin / User Admin / Intune Admin]
Anforderungen: [Genehmigung nötig? MFA bei Aktivierung? Max. Dauer?]
Ausgabe: Schrittweise Anleitung + PowerShell-Script + Begründung für jede Einstellung
```

**Entra ID Audit-Report**
```
Erstelle ein PowerShell-Script das folgende Informationen aus Entra ID exportiert:
- Alle Benutzer ohne MFA-Registrierung
- Alle permanenten Admin-Rollenzuweisungen (ohne PIM)
- Inaktive Benutzer (90+ Tage kein Login) mit Lizenzen
- Gastbenutzer und deren letzte Aktivität
Format: CSV-Export nach C:\Reports\ mit Datum im Dateinamen
```

---

## Intune & Endpoint Management

**Autopilot Deployment planen**
```
Erstelle einen Autopilot-Deployment-Plan für:
Geräteanzahl: [Anzahl]
Gerätetypen: [Surface Pro / Dell Latitude / HP EliteBook]
Anforderungen: [Zero-Touch / Benutzer-gesteuert / Selbst-Deployment]
Besonderheiten: [Hybrid Join nötig? Lokale Admin-Rechte? Spezielle Apps?]
Ausgabe: Schritt-für-Schritt-Plan, Checkliste, benötigte PowerShell-Scripts
```

**Compliance Policy erstellen**
```
Erstelle eine Intune Compliance Policy für Windows 11 Geräte mit:
Mindestanforderungen: BitLocker, Defender aktiv, mindestens Windows 11 22H2
Strengere Anforderungen für: [Geschäftsführung / IT-Admins / Standardbenutzer]
Was soll bei Non-Compliance passieren: [E-Mail / Zugriff sperren / IT benachrichtigen]
Ausgabe: Policy-Konfiguration + PowerShell-Script + Monitoring-Abfrage
```

---

## Windows Server & Active Directory

**AD-Gesundheits-Bericht**
```
Erstelle ein PowerShell-Script für einen vollständigen AD-Gesundheitsbericht:
Prüfungen: DC-Replikation, Zeitdienst, SYSVOL, FSMO-Rollen, verwaiste Objekte
Sicherheitsaudit: Kerberoastable Accounts, Passwort nie ablaufend, inaktive Computer
Format: HTML-Report mit Ampelfarben (Grün/Gelb/Rot) + CSV-Exporte
Ausführung: Soll als geplante Aufgabe wöchentlich laufen
```

**GPO-Konzept erstellen**
```
Erstelle ein GPO-Konzept für folgende Anforderungen:
Umgebung: [OU-Struktur beschreiben]
Sicherheitsanforderungen: [Passwortrichtlinie, Bildschirmsperre, USB-Kontrolle, etc.]
Software-Verteilung: [Welche Software per GPO]
Ausgabe: GPO-Struktur-Empfehlung, Einstellungen mit Pfaden, PowerShell-Befehle
```

---

## Azure IaaS & Kosten

**VM-Migration planen**
```
Plane die Migration folgender on-prem Server zu Azure:
Server: [Typ, Betriebssystem, RAM, CPU, Speicher, Workload]
Anforderungen: [Verfügbarkeit, Compliance, Backup, Monitoring]
Budget: [Monatliches Budget oder Kostenziel]
Ausgabe: VM-SKU-Empfehlung, Architekturskizze, monatliche Kostenschätzung,
         Migrationsstrategie (Lift & Shift / Replatform), ARM-Template/Bicep
```

**Kostenoptimierungs-Analyse**
```
Analysiere folgende Azure-Umgebung auf Einsparpotenzial:
Aktuelle VMs: [Liste oder beschreibe die Umgebung]
Nutzungsmuster: [24/7 Produktion / Bürozeiten / unregelmäßig]
Ausgabe: Konkrete Empfehlungen mit Einsparschätzung in €/Monat:
         Reserved Instances, Auto-Shutdown, Right-Sizing, Hybrid Benefit
```

---

## PowerShell Automatisierung

**Onboarding-Prozess automatisieren**
```
Erstelle ein vollständiges PowerShell-Onboarding-Script für neue Mitarbeiter:
Systeme: [Entra ID, AD on-prem, Exchange Online, Intune-Gruppe]
Eingabe: CSV-Datei mit Vorname, Nachname, Abteilung, Manager, Startdatum
Aktionen: Konto erstellen, Gruppen zuweisen, Lizenz vergeben, Willkommensmail senden
Besonderheiten: [Zeitarbeiter mit Ablaufdatum? Verschiedene Standorte?]
Standard: Idempotent, mit Log-Datei, WhatIf-Parameter, Fehlerbehandlung
```

**Monatlichen Admin-Report erstellen**
```
Erstelle ein PowerShell-Script das monatlich folgende Daten in einen HTML-Report sammelt:
M365: Lizenzen (genutzt/verfügbar), neue/deaktivierte Benutzer, Gäste, MFA-Status
AD: Neue Computer, inaktive Konten, Gruppenänderungen in privilegierten Gruppen
Azure: Kosten pro Resource Group, neue Ressourcen, VM-Status
Ausgabe: Formatierter HTML-Report per E-Mail an IT-Leitung
```

---

## Troubleshooting

**Anmelde-Problem diagnostizieren**
```
Ein Benutzer kann sich nicht anmelden. Hilf mir bei der systematischen Diagnose:
Fehlermeldung: [Exakter Fehlertext oder Code]
Benutzer: [Cloud-Only oder Hybrid?]
Betroffene App: [Alle Apps / Outlook / Teams / VPN]
Gerät: [Windows / Mac / Mobil / alle]
Ausgabe: Diagnose-Schritte mit PowerShell-Befehlen, mögliche Ursachen,
         Lösungsschritte für jede Ursache
```

**Exchange Mail-Flow Problem**
```
E-Mails kommen nicht an oder werden nicht zugestellt:
Richtung: [Extern → intern / intern → extern / intern → intern]
Symptom: [Bounced / Quarantäne / Kein Empfang / Verzögerung]
Absender-Domain: [domain.de]
Empfänger: [einzelner User / Gruppe / alle]
Ausgabe: Message Trace Befehle, Diagnose-Schritte, häufigste Ursachen
```

---

## Compliance & Governance

**M365 Sicherheits-Baseline prüfen**
```
Erstelle eine Checkliste und PowerShell-Auditscript für folgende Sicherheits-Baseline:
Frameworks: [CIS M365 Benchmark / Microsoft Secure Score Empfehlungen / intern]
Bereiche: Entra ID, Exchange Online, SharePoint, Teams, Defender
Ausgabe: Checkliste mit Ist/Soll, PowerShell-Script zum Auslesen der aktuellen Einstellungen,
         Priorisierung der Maßnahmen nach Risiko
```

**Gastbenutzer-Governance**
```
Erstelle ein Konzept und Script für die Verwaltung von Gastbenutzern:
Anforderungen: [Wer darf Gäste einladen? Max. Gültigkeitsdauer? Review-Prozess?]
Kontrollen: Quartalsweise Access Reviews, automatische Benachrichtigung bei Ablauf
Ausgabe: Governance-Konzept (DE), PowerShell-Script für regelmäßigen Gast-Audit,
         Konfigurationsanleitung für Access Reviews
```

---

## Exchange & Mail-Flow

**Exchange Hybrid Migration planen**
```
Ich plane eine Exchange Hybrid Migration für [X] Benutzer von On-Premises Exchange [Version] zu Exchange Online.
Umgebung: [AD-Struktur, Anzahl Exchange-Server, Internetzugang, MX-Provider]
Anforderungen: [Koexistenz-Dauer, Free/Busy, Migration-Batches, Cut-Over-Zeitfenster]
Ausgabe: Schritt-für-Schritt-Plan, PowerShell-Befehle für jeden Schritt, Rollback-Plan
```

**Anti-Spam härten**
```
Unsere Organisation erhält zu viele Phishing-/Spam-Mails. Konfiguriere:
Lizenz: [Microsoft 365 Business Premium / Defender for Office 365 Plan 1/2]
Problem: [Phishing, BEC, Bulk-Spam, CEO-Fraud]
Ausgabe: Anti-Spam + Anti-Phishing Policy (PowerShell), DKIM/DMARC-Einrichtung, Awareness-Empfehlungen
```

---

## Intune & Endpoint

**Windows Autopilot einrichten**
```
Richte Windows Autopilot für unsere Umgebung ein:
Gerätetyp: [Physische PCs / Virtuelle Maschinen]
Deployment-Modus: [User-Driven / Self-Deploying / Pre-provisioning]
Anforderungen: [Gerätename-Schema, Sprache, Apps die sofort verfügbar sein müssen]
Ausgabe: Autopilot-Profil-Konfiguration, Enrollment Status Page Einstellungen, Test-Checkliste
```

**Compliance Policy erstellen**
```
Erstelle eine Intune Compliance Policy für [Windows 11 / iOS / Android]:
Sicherheitsanforderungen: [BitLocker, PIN-Länge, OS-Mindestversion, Jailbreak-Erkennung]
Konsequenz bei Nicht-Konformität: [E-Mail-Benachrichtigung, Zugangsblock nach X Tagen]
Ausgabe: JSON-Policy-Definition + PowerShell-Deployment + Erklärung jedes Parameters
```

---

## SharePoint & Teams

**SharePoint Governance einrichten**
```
Unsere SharePoint-Umgebung ist unkontrolliert gewachsen (viele Sites, externe Freigaben).
Aktueller Stand: [Anzahl Sites, externe User vorhanden? Compliance-Anforderungen?]
Ziel: [Externe Freigabe einschränken, Site Lifecycle Management, Berechtigungsaudit]
Ausgabe: PowerShell-Audit-Skript + Governance-Richtlinie (Vorlage) + Umsetzungsplan
```

**Teams-Governance und Guest Access**
```
Konfiguriere Teams-Governance für unsere Organisation:
Anforderungen: [Wer darf Teams erstellen? Externe Gäste erlaubt? Naming Convention?]
Lizenz: [M365 Business Premium / E3 / E5]
Ausgabe: PowerShell-Konfiguration + Teams-Richtlinie + Entra ID Group Lifecycle Policy
```

---

## Security & Compliance

**Zero Trust Roadmap**
```
Erstelle eine Zero Trust Roadmap für unsere Microsoft 365 Umgebung:
Ausgangslage: [Keine CA / Basic CA / Fortgeschritten]
Lizenz: [E3 + Entra P1 / E5 / Business Premium]
Prioritäten: [MFA zuerst / Device Compliance / Identity Protection]
Ausgabe: 90-Tage-Plan mit konkreten Maßnahmen + PowerShell-Skripte + Erfolgsmessung
```

**Sensitivity Labels konfigurieren**
```
Konfiguriere Microsoft Purview Sensitivity Labels:
Label-Kategorien: [Öffentlich / Intern / Vertraulich / Streng Vertraulich]
Anforderungen: [Automatische Klassifizierung? Verschlüsselung? Wasserzeichen?]
Ausgabe: Label-Konfiguration (Compliance Center) + PowerShell + Rollout-Plan + User-Guide
```
