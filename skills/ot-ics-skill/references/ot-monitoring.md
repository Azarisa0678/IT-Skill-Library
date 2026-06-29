# OT/ICS Monitoring & Incident Response — Referenzmodul

Passive Monitoring, Anomalie-Erkennung, Claroty/Nozomi/Dragos, IR-Playbooks für OT.

---

## Passives OT-Monitoring (Grundprinzip)

```
WARUM PASSIV?
  Aktive Scans (Nmap, Ping) können OT-Geräte zum Absturz bringen!
  → Alte SPS-Firmware, zeitkritische Prozesse

PASSIVE METHODEN:
  ✓ Netzwerk-TAP (Test Access Point): Kopie des Traffics ohne Einfluss
  ✓ SPAN-Port am Switch (Kopie aller Pakete)
  ✓ Deep Packet Inspection (DPI) der OT-Protokolle
  ✓ Verhaltensbaseline: "Normal" lernen, Abweichungen alarmieren

TYPISCHE TOOLS:
  Claroty:  Asset Discovery, Anomalie-Erkennung, Netzwerk-Visualisierung
  Nozomi:   OT/IT Asset Inventory, Threat Detection, Integration mit SIEM
  Dragos:   ICS-spezifische Threat Intelligence, Incident Response
  Microsoft Defender for IoT (ehem. CyberX): Azure-integriert
  OpenSource: Zeek (Bro IDS), Suricata mit OT-Signaturen

DEPLOYMENT:
  ┌──────────────────────────────────────────────────────────┐
  │ SCADA-Netz (VLAN 50)                                     │
  │  SCADA-Server ──── Switch ──── SPS-1  SPS-2  SPS-3      │
  │                      │                                   │
  │                   SPAN-Port                              │
  │                      │                                   │
  │                  OT-Sensor (Nozomi/Claroty)             │
  │                      │                                   │
  └──────────────────────┼───────────────────────────────────┘
                         │ Nur Management-Traffic (keine OT-Daten!)
                    ┌────┴─────┐
                    │  SIEM   │
                    │(Sentinel)│
                    └──────────┘
```

---

## Nozomi Networks — Konfiguration und Integration

```python
#!/usr/bin/env python3
"""Nozomi Networks API — Asset und Alert Management"""
import requests
import json

class NozomiAPI:
    def __init__(self, host: str, user: str, password: str):
        self.base = f"https://{host}/api/open/v1"
        self.session = requests.Session()
        self.session.verify = True  # TLS-Validierung

        # Authentifizierung
        resp = self.session.post(f"{self.base}/auth/sign_in",
            json={"user": {"email": user, "password": password}})
        resp.raise_for_status()
        token = resp.headers.get("Authorization")
        self.session.headers.update({"Authorization": token})

    def get_assets(self, asset_type: str = None) -> list:
        """Alle OT-Assets abrufen"""
        params = {}
        if asset_type:
            params["type"] = asset_type  # plc, hmi, scada, sensor, etc.
        resp = self.session.get(f"{self.base}/assets", params=params)
        return resp.json().get("items", [])

    def get_alerts(self, severity: str = "high", days: int = 7) -> list:
        """Aktuelle Alerts abrufen"""
        resp = self.session.get(f"{self.base}/alerts", params={
            "severity": severity,
            "time_filter": f"last_{days}_days"
        })
        return resp.json().get("items", [])

    def get_network_flows(self, src_ip: str = None) -> list:
        """Netzwerk-Flows für Basis-Analyse"""
        params = {}
        if src_ip:
            params["src_ip"] = src_ip
        resp = self.session.get(f"{self.base}/network_flows", params=params)
        return resp.json().get("items", [])

# Nutzung:
nozomi = NozomiAPI("nozomi.firma.local", "admin@firma.de", "Passwort")

# Alle SPS im Netz
plcs = nozomi.get_assets(asset_type="plc")
print(f"Gefundene SPS: {len(plcs)}")
for plc in plcs:
    print(f"  {plc['ip']} | {plc['vendor']} {plc['model']} | FW: {plc.get('firmware_version','?')}")

# Kritische Alerts
alerts = nozomi.get_alerts(severity="critical")
for alert in alerts:
    print(f"[KRITISCH] {alert['time']}: {alert['name']} — {alert['description']}")
```

---

## SIEM-Integration (OT-Logs → Microsoft Sentinel)

```yaml
# Azure Monitor Agent Konfiguration für OT-Syslog
# (auf Nozomi/Claroty Management-Server installiert)

# /etc/opt/microsoft/azuremonagent/syslog.conf
[syslog]
  facility = local0
  filter = *
  destination = sentinel

# Sentinel Data Connector: Syslog
# Workspace: law-sentinel-prod
# Source: ot-sensor-nozomi
```

```kql
// KQL: OT-Anomalien in Microsoft Sentinel

// Neue Geräte im OT-Netz (wurden nie zuvor gesehen)
CommonSecurityLog
| where DeviceVendor == "Nozomi Networks"
| where Activity == "asset_discovered"
| where TimeGenerated > ago(24h)
| project TimeGenerated, SourceIP, DestinationIP,
    DeviceCustomString1,   // Asset-Typ (plc/hmi/sensor)
    DeviceCustomString2,   // Vendor
    AdditionalExtensions
| order by TimeGenerated desc

// Modbus-Write-Befehle (könnten SPS-Parameter ändern)
CommonSecurityLog
| where DeviceVendor == "Nozomi Networks"
| where Activity == "modbus_write"
| where TimeGenerated > ago(1h)
| summarize WriteCount=count() by SourceIP, DestinationIP, bin(TimeGenerated, 5m)
| where WriteCount > 10   // Mehr als 10 Schreibbefehle in 5 Min = Anomalie

// Fernwartungs-Session außerhalb Geschäftszeiten
CommonSecurityLog
| where DeviceVendor == "Nozomi Networks"
| where Activity == "remote_access"
| extend Hour = hourofday(TimeGenerated)
| where Hour < 7 or Hour > 19    // Außerhalb 07:00-19:00
| project TimeGenerated, SourceIP, DestinationIP, Hour, AdditionalExtensions

// Neue Protokoll-Verbindung (nie zuvor gesehen)
CommonSecurityLog
| where DeviceVendor == "Nozomi Networks"
| where Activity in ("new_connection", "protocol_violation")
| where TimeGenerated > ago(24h)
| project TimeGenerated, SourceIP, DestinationIP, Activity, Message
```

---

## OT Incident Response Playbooks

### Playbook 1: Ransomware in IT-Netz — OT-Schutz

```markdown
## IR-Playbook: Ransomware erkannt — OT-Isolation

### Auslöser
- EDR-Alert: Ransomware auf Windows-Server im IT-Netz
- SIEM-Alert: Verschlüsselungs-Muster erkannt
- OT-Sensor: Ungewöhnliche Verbindungsversuche aus IT → OT

### SOFORT (0-15 Minuten)

PRIORITÄT 1: OT SCHÜTZEN BEVOR ES ZU SPÄT IST
1. IT/OT-Firewall: Alle Verbindungen IT → OT SPERREN
   (Auch wenn OT noch nicht betroffen — prophylaktisch!)
   Cisco: interface vlan 20; shutdown
   Fortinet: set status disable (IT-zu-OT Policy)

2. OT-Segment prüfen:
   → OT-Sensor (Nozomi/Claroty): Gibt es Anomalien im OT-Netz?
   → SCADA: Läuft die Produktion noch normal?
   → Prozessleitung kontaktieren: Manuelle Überwachung aktivieren

3. Fernwartungszugänge sperren:
   → VPN für externe Lieferanten deaktivieren
   → Jump-Host: aktive Sessions terminieren

PRIORITÄT 2: IT ISOLIEREN
4. Kompromittierte Systeme identifizieren und isolieren
5. Kein Backup-Laufwerk von betroffenen Systemen aus anschließen
   (Backup könnte überschrieben werden!)

### KURZFRISTIG (15-60 Minuten)
6. BSI kontaktieren (falls KRITIS/NIS2-pflichtig): 0800 274 1000
7. OT-Hersteller-Hotline bereitstellen (Siemens, Rockwell, etc.)
8. Entscheidung: Produktion weiterlaufen lassen (OT isoliert) oder
   kontrolliertes Shutdown?
```

### Playbook 2: Anomalie in OT-Netz

```markdown
## IR-Playbook: Unbekanntes Gerät / Anomalie im OT-Netz

### Auslöser
- Nozomi/Claroty-Alert: Neues Gerät entdeckt
- SIEM-Alert: Neues Protokoll oder unbekannte Verbindung
- Netzwerk-Admin: Verdächtiger Traffic

### ANALYSE (0-30 Minuten)
1. Gerät identifizieren:
   - IP/MAC-Adresse aus Alert
   - Switch-Port: show mac address-table | include [MAC]
   - Physical Location: show cdp neighbors [interface]
   
2. Traffic analysieren (PASSIV!):
   - Nozomi: welche Verbindungen hat das Gerät?
   - Welche Protokolle? (Modbus? ICMP? HTTP?)
   - Wohin kommuniziert es?

3. Legitimität prüfen:
   - Asset-Datenbank: Gerät bekannt?
   - Wartungsplan: Ist Fernwartung geplant?
   - Prozessleitung: Wurde neues Gerät installiert?

### EINDÄMMUNG (bei Verdacht)
4. Switch-Port deaktivieren (NICHT das Gerät selbst!):
   Cisco: interface GiX/X; shutdown
   (Gerät für forensische Analyse erhalten)

5. Netzwerk-Capture starten:
   tcpdump -i eth0 host [IP] -w /tmp/ot-evidence.pcap

6. Gerät physisch lokalisieren und sichern
   (Kein Reset, keine Änderungen am Gerät!)

### ESKALATION
7. OT-Sicherheitsbeauftragter informieren
8. Wenn Sabotage/Angriff vermutet: BSI + Polizei (ZAC)
9. Forensik-Experte für OT beauftragen (Dragos, Claroty PS)
```

---

## OT-Härtungscheckliste

```markdown
## OT/ICS Sicherheits-Baseline — Checkliste

### Asset Management
- [ ] Vollständiges OT-Asset-Inventar (Hersteller, Modell, FW-Version, IP)
- [ ] Bekannte Schwachstellen je Asset aus CISA ICS-CERT geprüft
- [ ] End-of-Support-Geräte identifiziert und Migrationsplan erstellt
- [ ] Netzwerkplan mit allen OT-Verbindungen aktuell

### Netzwerksicherheit
- [ ] IT/OT-Segmentierung: separate Netze mit Firewall dazwischen
- [ ] Default Deny zwischen IT und OT
- [ ] Kein direkter Internet-Zugang aus OT-Netz
- [ ] Fernwartung nur über Jump-Host mit Session-Recording
- [ ] Wireless: kein unverschlüsseltes WLAN im OT-Bereich

### Protokollsicherheit
- [ ] Modbus: Firewall-Whitelist, kein breiter Zugang
- [ ] OPC UA: Security Policy konfiguriert, kein Anonymous
- [ ] GOOSE: VLAN-Isolierung, Anomalie-Monitoring
- [ ] Nicht benötigte Protokolle deaktiviert

### Monitoring
- [ ] Passiver OT-Sensor (Nozomi/Claroty/MS Defender for IoT) aktiv
- [ ] OT-Alerts in SIEM integriert
- [ ] Baseline definiert (was ist "normal"?)
- [ ] Alert-Schwellen kalibriert (nicht zu viel, nicht zu wenig)

### Incident Response
- [ ] OT-spezifischer IR-Plan vorhanden
- [ ] Eskalationskontakte OT-Hersteller hinterlegt
- [ ] Backup alle SPS-Programme + HMI-Projekte (offline!)
- [ ] Notbetrieb-Konzept (manuelle Steuerung) dokumentiert und geübt
- [ ] BSI-Kontakt für KRITIS-Meldung bekannt
```
