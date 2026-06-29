---
name: ot-ics-skill
description: >
  OT/ICS Security Skill fuer OT-Sicherheitsbeauftragte und KRITIS-Betreiber.
  OT-Protokolle: Modbus, DNP3, IEC 60870-5-104, IEC 61850, GOOSE, MMS, OPC UA,
  Profinet, EtherNet/IP, MQTT, Sicherheitsanalyse, Kompensationsmassnamen,
  Passives OT-Monitoring, Nozomi, Claroty, Dragos, MS Defender for IoT,
  IEC 62443, CSMS, Zones Conduits, Security Levels SL0-SL4, Foundational Requirements,
  System Requirements SR, Assessment, 62443-2-1, 62443-2-3, 62443-3-2, 62443-3-3, 62443-4-1,
  Patch-Management IACS, Patch-Klassifizierung, Change Control OT,
  Fernwartungssicherheit, Jump-Host, Session-Recording, zeitlich begrenzte Zugaenge,
  IT/OT-Segmentierung, Purdue-Modell, DMZ, Data Diode,
  OT Incident Response, Ransomware-Schutz OT, OT-Forensik,
  SIEM-Integration, Sentinel KQL OT-Events, Nozomi API,
  KRITIS Energie Wasser, BSI ICS-Kompendium, BDEW-Whitepaper,
  SPS-Haertung, HMI, SCADA, Backup SPS-Programme.
  Outputs: Sicherheitskonzepte OT, Checklisten, IR-Playbooks, KQL-Queries, Python-Scripts.
---

# OT/ICS Security Skill

## Leitprinzipien

**Verfügbarkeit vor Vertraulichkeit.** In OT ist Produktionsausfall oft kritischer als Datenleck — Maßnahmen müssen Betrieb ermöglichen.
**Passiv vor aktiv.** Niemals aktive Scans (Nmap, Ping-Sweep) ohne Absprache mit OT-Betrieb — kann Anlagen zum Absturz bringen.
**Segmentierung ist die wichtigste Maßnahme.** IT und OT gehören getrennt — mit Firewall und DMZ dazwischen.
**Change Control für alles.** Kein Patch, keine Konfigurationsänderung ohne dokumentierte Genehmigung und Testplan.
**Backup offline.** SPS-Programme und SCADA-Konfigurationen müssen offline gesichert sein (Ransomware-Schutz).
**Lieber keine Verbindung als unsichere Verbindung.** Fernwartung nur über gesicherte Pfade — niemals direktes RDP ins OT-Netz.

---

## Referenzmodule

- `references/ot-protocols.md` — Modbus/DNP3/IEC 60870/IEC 61850/OPC UA Security-Analyse, Kompensationsmaßnahmen, Audit-Scripts
- `references/ot-monitoring.md` — Passives Monitoring, Nozomi API, SIEM, KQL, IR-Playbooks, Härtungscheckliste
- `references/iec62443.md` — CSMS, Zones & Conduits Design, SL-Assessment, SR-Anforderungskatalog, Patch-Management OT

Routing:
- OT-Protokoll / Sicherheitsanalyse / Kompensation → `ot-protocols.md`
- Monitoring / Nozomi / SIEM / Incident Response / Härtung → `ot-monitoring.md`
- IEC 62443 / Zones & Conduits / Security Level / Assessment → `iec62443.md`
- Patch-Management OT / Change Control / CSMS → `iec62443.md`
- IEC 62443 / Zones & Conduits → `ot-protocols.md` + `ot-monitoring.md`
- KRITIS Energie/Wasser → zusätzlich `it-comprehensive-security/references/energy-security.md`

---

## Protokoll-Sicherheit — Schnellreferenz

```
PROTOKOLL         AUTH  VERSCHLÜSSELUNG  MASSNAHME
─────────────────────────────────────────────────────
Modbus TCP        ❌     ❌              Firewall-Whitelist, VPN, Segmentierung
DNP3              ⚠️     ❌              SAv5 aktivieren oder VPN-Kompensation
IEC 60870-5-104   ⚠️     ❌              IEC 62351-5 oder TLS-Tunnel
IEC 61850 GOOSE   ⚠️     ❌              VLAN-Isolierung, IEC 62351-6, IDS
OPC UA            ✅     ✅              Basic256Sha256+, SignAndEncrypt, kein Anonymous
Profinet          ❌     ❌              Netz-Segmentierung, Port-Security
MQTT              ⚠️     ⚠️              TLS 1.3, Auth-Token, kein Anonymous
```

→ Python-Audit-Scripts (Modbus, GOOSE-Analyse), OPC UA sichere Konfiguration: `references/ot-protocols.md`

---

## IT/OT-Architektur (Zones & Conduits)

```
Zone 1: Enterprise (ERP, Office-IT)        SL1
    │
    │ Conduit C1: Nur OPC UA lesend + HTTPS API
    ▼
Zone 5: DMZ / IIoT Platform                SL2
    (OPC UA Gateway, MQTT Broker)
    │
    │ Conduit C2: Nur definierte Tags + Protokolle
    ▼
Zone 2: Operations (SCADA, MES, Historian) SL2
    │
    │ Conduit C3: Streng kontrolliert
    ▼
Zone 3: Control (SPS, DCS, Safety-SPS)    SL2-SL3
    │
Zone 4: Field (Sensoren, Aktoren)          SL1-SL2

Zone 6: Fernwartung (Jump-Host)            SL3
    → Session-Recording, MFA, zeitlich begrenzt
```

---

## OT Monitoring — Schnellreferenz

```python
# Nozomi API: Alle SPS im Netz
nozomi = NozomiAPI("nozomi.firma.local", user, password)
plcs = nozomi.get_assets(asset_type="plc")
for plc in plcs:
    print(f"{plc['ip']} | {plc['vendor']} {plc['model']} | FW: {plc.get('firmware_version','?')}")

# Kritische Alerts
for alert in nozomi.get_alerts(severity="critical"):
    print(f"[KRITISCH] {alert['time']}: {alert['name']}")
```

```kql
// Sentinel: Neue OT-Geräte (unbekannte Assets)
CommonSecurityLog
| where DeviceVendor == "Nozomi Networks"
| where Activity == "asset_discovered"
| where TimeGenerated > ago(24h)
| project TimeGenerated, SourceIP, DeviceCustomString1, DeviceCustomString2

// Modbus-Write-Anomalie (> 10 Schreibbefehle in 5 Min)
CommonSecurityLog
| where Activity == "modbus_write"
| summarize count() by SourceIP, DestinationIP, bin(TimeGenerated, 5m)
| where count_ > 10
```

→ Vollständige IR-Playbooks (Ransomware, Anomalie), OT-Härtungscheckliste: `references/ot-monitoring.md`

---

## Output-Formate

| Aufgabe | Format |
|---|---|
| OT-Sicherheitskonzept | Strukturiertes Dokument: Scope, Zones, Maßnahmen je Zone |
| Protokoll-Audit | Python-Skript + Befundliste + Kompensationsmaßnahmen |
| IR-Playbook OT | Schritt-für-Schritt mit Zeitvorgaben, Entscheidungsbäumen |
| SIEM KQL-Regeln | Fertige Queries mit Erklärung + Alert-Konfiguration |
| Härtungscheckliste | Markdown-Checkboxen, priorisiert nach Risiko |
| Netzwerk-Design | ASCII-Topologie mit Zonen, Conduits, Security Levels |
---

## IEC 62443 — Schnellreferenz

```
CSMS AUFBAUEN (62443-2-1):
  1. Risikoanalyse → Asset-Inventar + Bedrohungen + Schwachstellen
  2. Sicherheitsrichtlinie + Verfahren (Patch, IR, Change Control)
  3. Technische Maßnahmen (Segmentierung, Auth, Monitoring)
  4. Regelmäßige Audits + Management-Review

ZONES & CONDUITS (62443-3-2):
  Schritt 1: Alle IACS-Assets inventarisieren
  Schritt 2: Assets zu Zonen gruppieren (gleiche Funktion/Schutzbedarf)
  Schritt 3: Kommunikationsbeziehungen = Conduits
  Schritt 4: Target Security Level je Zone festlegen
  Schritt 5: Maßnahmen für Target SL implementieren

TARGET SL WÄHLEN:
  Personenschäden möglich  → SL3-SL4
  Produktionsausfall       → SL2-SL3
  Nur Datenverlust         → SL1-SL2

PATCH-MANAGEMENT OT (62443-2-3):
  Klasse A (CVSS ≥ 9.0):  30 Tage, Expedited Process
  Klasse B (sicherheitsrel.): 90 Tage, Standard Process
  Klasse C (Funktion):    Geplantes Wartungsfenster
  IMMER: Test in Testumgebung → Hersteller-Freigabe → Change Control → Backup VORHER
```

→ Vollständige Zone-Vorlage, Conduit-Matrix, SR-Anforderungskatalog (FR1-FR7), Assessment-Vorlage: `references/iec62443.md`
