# Manufacturing & Industrie 4.0 Security — Referenzmodul

IT/OT-Sicherheit für produzierende Unternehmen, Smart Factory, Industrie 4.0,
vernetzte Produktion, MES, ERP-OT-Integration, digitale Lieferkette.

---

## Regulatorische Landschaft Produktion (DACH)

| Rahmen | Geltungsbereich | Relevanz |
|---|---|---|
| **IEC 62443** | Industrielle Automatisierung (IACS) | Kernstandard für OT-Sicherheit |
| **BDEW-Whitepaper** | Energieversorger (OT-Fokus) | Analog auf Produktion anwendbar |
| **NIS2 (Verarbeitendes Gewerbe)** | "Wichtige Einrichtung" ab 50 MA | Meldepflichten, Art. 21 |
| **KRITIS (falls Schwellenwerte)** | Kritische Infrastruktur | BSI-Registrierung, Nachweis |
| **ISO 27001** | Gesamt-ISMS inkl. OT | Zertifizierung empfohlen |
| **VDA ISA / TISAX** | Automobilindustrie | Assessment-basiertes Zertifikat |
| **DSGVO** | Mitarbeiterdaten, Kundendaten | Standard-Anforderungen |
| **Maschinenverordnung (EU 2023/1230)** | Maschinen mit eingebetteter Software | Cybersecurity als Produktsicherheit |

---

## Industrie 4.0 Referenzarchitektur (Security)

```
┌─────────────────────────────────────────────────────────────────┐
│  UNTERNEHMENSEBENE (ERP, MES, PLM, SCM)                        │
│  SAP/Oracle · Siemens Opcenter · PTC Windchill                 │
├─────────────────────────────────────────────────────────────────┤
│  INTEGRATION / IIOT PLATFORM (DMZ!)                             │
│  OPC UA Server · MQTT Broker · IIoT Gateway · Data Lake        │
│  → Datenaustausch zwischen IT und OT (kontrolliert, gefiltert) │
├─────────────────────────────────────────────────────────────────┤
│  STEUERUNGSEBENE (SCADA, DCS, MES-Anbindung)                   │
│  Siemens WinCC · Rockwell FactoryTalk · Wonderware             │
├─────────────────────────────────────────────────────────────────┤
│  FELDEBENE (SPS, Roboter, CNC, Sensoren, Aktoren)              │
│  Siemens S7 · Allen-Bradley · Fanuc · Kuka · B&R               │
└─────────────────────────────────────────────────────────────────┘

SICHERHEITSPRINZIPIEN:
  ✓ Strikte Trennung IT ↔ OT (kein direkter Zugang)
  ✓ DMZ als Puffer (Data Diode für unidirektionale Daten)
  ✓ OPC UA als sicheres Protokoll für IT/OT-Datenaustausch
  ✓ Keine direkten ERP→SPS-Verbindungen
  ✓ Fernwartung nur über gesicherte Jump-Hosts
```

---

## IEC 62443 in der Produktion

### Zones & Conduits Mapping (Fertigung)

```
ZONE-MODELL FERTIGUNGSANLAGE:

Zone 1: Enterprise (Büro-IT)
  Assets: ERP, E-Mail, Office-PCs
  Security Level: SL1
  Verbindung zur OT: NUR über Zone 5 (DMZ)

Zone 2: Operations (MES/SCADA)
  Assets: MES-Server, Historian, SCADA-Server
  Security Level: SL2
  Verbindung: Conduit C1 → Zone 1 (gefiltert, unidirektional wo möglich)

Zone 3: Control (SPS-Ebene)
  Assets: SPS, DCS, Sicherheits-SPS (Safety PLC)
  Security Level: SL2-SL3
  Verbindung: Conduit C2 → Zone 2 (streng kontrolliert)

Zone 4: Field (Sensoren, Aktoren, Roboter)
  Assets: Feldgeräte, Sensoren, Antriebe
  Security Level: SL1-SL2
  Verbindung: Feldbus (Profinet, EtherNet/IP)

Zone 5: DMZ (Datenaustausch IT↔OT)
  Assets: OPC UA Gateway, MQTT Broker, Datenpuffer
  Security Level: SL2
  Zweck: Daten zwischen Zone 1 und Zone 2 austauschen

Zone 6: Fernwartung
  Assets: Remote Access Server, Jump-Host
  Security Level: SL3
  Zugang: MFA + VPN + Session-Recording, zeitlich begrenzt

CONDUIT C1 (IT → OT):
  Erlaubt: OPC UA (lesend), HTTPS (API-Aufrufe an MES)
  Verboten: RDP direkt, SMB, Telnet, FTP
  Implementierung: Next-Gen Firewall mit Deep Packet Inspection

CONDUIT C2 (MES → SPS):
  Erlaubt: OPC UA, Profinet, EtherNet/IP (nur definierte Tags)
  Implementierung: Industrial Firewall (Fortinet FortiGate Rugged, Cisco IE)
```

### IEC 62443 Security Levels

```
SL0: Keine Schutzanforderungen
SL1: Schutz gegen zufällige/unbeabsichtigte Verletzungen
SL2: Schutz gegen einfache Angriffe (opportunistisch)
     → Standard für die meisten Produktionssysteme
SL3: Schutz gegen anspruchsvolle Angriffe (gezielt, motiviert)
     → Für kritische Steuerungsfunktionen (Safety-relevante SPS)
SL4: Schutz gegen staatlich gesponserte Angreifer
     → Nur für extrem kritische Infrastruktur

TARGET SL FESTLEGEN (IEC 62443-3-2):
  1. Konsequenzen bei Kompromittierung bestimmen
     (Personenschäden? Umweltschäden? Produktionsausfall? Datenverlust?)
  2. Bedrohungsakteure identifizieren
  3. Target SL je Zone festlegen
  4. Maßnahmen implementieren um Target SL zu erreichen
  5. Achieved SL dokumentieren (Assessment)
```

---

## OPC UA Security (IT/OT-Schnittstelle)

```xml
<!-- OPC UA Security Konfiguration (Beispiel) -->
<!-- Sicherheitspolitik: Basic256Sha256 (mindestens!) -->
<ApplicationConfiguration>
  <SecurityPolicies>
    <!-- VERBOTEN (Legacy, unsicher): -->
    <!-- <SecurityPolicy>None</SecurityPolicy> -->
    <!-- <SecurityPolicy>Basic128Rsa15</SecurityPolicy> -->

    <!-- ERLAUBT: -->
    <SecurityPolicy>Basic256Sha256</SecurityPolicy>
    <SecurityPolicy>Aes128Sha256RsaOaep</SecurityPolicy>
    <SecurityPolicy>Aes256Sha256RsaPss</SecurityPolicy>  <!-- Empfohlen ab 2023 -->
  </SecurityPolicies>

  <MessageSecurity>SignAndEncrypt</MessageSecurity>  <!-- Niemals None oder Sign-Only! -->

  <UserAuthentication>
    <!-- Nur Zertifikat oder Username/Passwort + TLS -->
    <!-- KEIN Anonymous Access! -->
    <Anonymous>false</Anonymous>
    <UserNamePassword>true</UserNamePassword>
    <Certificate>true</Certificate>
  </UserAuthentication>
</ApplicationConfiguration>
```

```bash
# OPC UA Sicherheitsprüfung mit UA-Expert oder python-opcua
from opcua import Client
from opcua.crypto import security_policies

client = Client("opc.tcp://plc-01.oem.local:4840")
client.set_security(
    security_policies.SecurityPolicyBasic256Sha256,
    certificate="client_cert.pem",
    private_key="client_key.pem",
    server_certificate="server_cert.pem",
    mode=ua.MessageSecurityMode.SignAndEncrypt
)
client.connect()
# Nodes lesen...
client.disconnect()
```

---

## Fernwartungssicherheit (Remote Maintenance)

```
ANFORDERUNGEN FÜR SICHERE FERNWARTUNG (Maschinen/Anlagen):

TECHNISCH:
  ✓ VPN mit MFA (kein RDP direkt ins OT-Netz!)
  ✓ Jump-Host in separater DMZ
  ✓ Session-Recording (Video + Keystroke-Logging)
  ✓ Zeitlich begrenzte Zugänge (max. 4h, verlängerbar)
  ✓ Nur auf freigegebene Anlagen/Systeme (kein Netz-Zugriff)
  ✓ Trennung: Hersteller A kann nur seine Anlagen sehen

ORGANISATORISCH:
  ✓ Anfrage-Genehmigungsprozess (Werksleiter/Betriebsleiter)
  ✓ Begleitung durch internen Mitarbeiter bei kritischen Zugriffen
  ✓ Zugang sofort sperren nach Wartungsende
  ✓ Audit-Log: wer hat was wann geändert (für Wartungsprotokoll)

LIEFERANTENANFORDERUNGEN:
  ✓ Keine eigenen VPN-Client-Installationen im OT-Netz
  ✓ Lieferanten nutzen VOM BETREIBER bereitgestellten Fernzugang
  ✓ NDA + IT-Sicherheitsanforderungen vertraglich fixiert
  ✓ Jährliche Bestätigung der Einhaltung

LÖSUNGEN:
  Secomea: Industrie-Fernwartungsplattform, DE-Server
  Tosibox: Plug-and-Play OT-Fernwartung
  Claroty xDome: OT-Sicherheitsplattform mit Fernwartungsmodul
  Fortinet ZTNA: Zero-Trust-Ansatz für Fernwartung
```

---

## TISAX (Automobilindustrie)

```
TISAX = Trusted Information Security Assessment Exchange
Herausgeber: ENX Association (automotive Industrieverbund)
Basis: VDA ISA (Information Security Assessment)

PFLICHT FÜR:
  - Alle Lieferanten von Automobilherstellern (OEMs)
  - Wenn sensible Informationen vom OEM empfangen werden
  - Bei Prototypenentwicklung, Konstruktionsdaten, Produktionspläne

ASSESSMENT-LABELS:
  Info high:          Vertrauliche Unternehmensinformationen
  Info very high:     Streng vertraulich (OEM-Geheimnisse)
  Prototype:          Fahrzeugprototypen, Camouflage-Anforderungen
  Test vehicle:       Versuchsfahrzeuge

BEWERTUNGSSKALA:
  0 = Kein Prozess vorhanden
  1 = Informell (kein dokumentierter Prozess)
  2 = Dokumentiert
  3 = Genehmigt + kommuniziert
  4 = Quantitativ geführt
  5 = Optimierend

MINDEST-ANFORDERUNG für Zertifizierung: Ø 3 (keine 0 oder 1!)

ASSESSMENT-PROZESS:
  1. Selbstbewertung (VDA ISA Fragebogen)
  2. Anbieterauswahl (ENX-akkreditierter Auditor)
  3. Remote- oder Vor-Ort-Assessment
  4. Ergebnis in TISAX-Plattform geteilt (nur mit Berechtigten)
  5. Gültigkeit: 3 Jahre
```

---

## Manufacturing Security Checkliste

```markdown
## IT/OT-Security Checkliste Produktion — [Unternehmen] — [Datum]

### Netzwerksegmentierung
- [ ] IT und OT in getrennten Netzen (physisch oder VLAN)
- [ ] DMZ zwischen IT und OT implementiert
- [ ] Firewall-Regeln: Default Deny zwischen Zonen
- [ ] Keine direkten Verbindungen ERP → SPS
- [ ] Fernwartungszugang nur über gesicherten Jump-Host

### OT Asset Management
- [ ] Vollständiges Inventar aller OT-Assets (SPS, HMI, SCADA, Sensoren)
- [ ] Firmware-Versionen dokumentiert
- [ ] End-of-Life-Systeme identifiziert und Risikobehandlung geplant
- [ ] Passives Monitoring-Tool im Einsatz (Claroty, Nozomi, Dragos)

### Protokollsicherheit
- [ ] OPC UA: Security Policy Basic256Sha256 oder höher
- [ ] OPC UA: Anonymous Access deaktiviert
- [ ] Modbus/TCP: VPN-Tunnel oder Netzwerksegmentierung als Kompensation
- [ ] Profinet/EtherNet/IP: Segmentiert, keine WAN-Verbindung

### Patch-Management OT
- [ ] Patch-Management-Prozess für OT dokumentiert (Change Control)
- [ ] Hersteller-Freigabe für Patches eingeholt
- [ ] Test in separater Testumgebung vor Produktiveinsatz
- [ ] Wartungsfenster für Patches geplant (Produktionsstillstand)

### Fernwartung
- [ ] Zentraler Fernwartungsserver (kein direktes RDP ins OT)
- [ ] Session-Recording aktiv
- [ ] Zeitlich begrenzte Zugangsfreundschaften
- [ ] Lieferantenanforderungen vertraglich fixiert

### Incident Response OT
- [ ] IR-Plan enthält OT-Szenarien (Produktionsstillstand, Ransomware)
- [ ] Notbetrieb: manuelle Steuerung möglich und geübt
- [ ] Kontakte OT-Hersteller (Siemens, Rockwell, B&R) hinterlegt
- [ ] Backup aller SPS-Programme, HMI-Projekte, SCADA-Konfigurationen
- [ ] Backup-Restore für OT getestet

### TISAX (falls Automobilindustrie)
- [ ] VDA ISA Selbstbewertung durchgeführt
- [ ] Ergebnis: keine 0 oder 1 in Pflicht-Anforderungen
- [ ] TISAX-Zertifizierung gültig (< 3 Jahre)
```

---

## Weiterführende Ressourcen

- **IEC 62443**: Normreihe (kostenpflichtig via IEC/DIN)
- **BSI**: ICS-Security Kompendium (kostenlos)
- **ENISA**: Guidelines on Cybersecurity for Manufacturing
- **VDA**: VDA ISA Fragebogen (Basis für TISAX)
- **ENX**: TISAX-Portal — tisax.de
- **Claroty**: OT-Asset-Inventar, Monitoring
- **Nozomi Networks**: OT/IIoT Anomalie-Erkennung
- **Dragos**: ICS/OT Threat Intelligence
- **CISA**: ICS-CERT Advisories (kostenlos, englisch)
