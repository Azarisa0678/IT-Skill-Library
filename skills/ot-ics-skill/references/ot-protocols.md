# OT/ICS Protokolle — Referenzmodul

Modbus, DNP3, IEC 60870-5-104, IEC 61850, Profinet, EtherNet/IP, OPC UA — Sicherheitsanalyse.

---

## Protokollübersicht und Sicherheitseigenschaften

| Protokoll | Schicht | Authentifizierung | Verschlüsselung | Verbreitung |
|-----------|---------|-------------------|-----------------|-------------|
| Modbus TCP | TCP/502 | ❌ Keine | ❌ Keine | Sehr hoch (Legacy) |
| DNP3 | TCP/20000 | ⚠️ Optional (SAv5) | ❌ Keine (ohne VPN) | Hoch (Energie/Wasser) |
| IEC 60870-5-104 | TCP/2404 | ⚠️ Optional (IEC 62351-5) | ❌ Keine | Hoch (Europa Energie) |
| IEC 61850 MMS | TCP/102 | ⚠️ Optional (IEC 62351-4) | ⚠️ Optional TLS | Hoch (Schaltanlagen) |
| IEC 61850 GOOSE | Ethernet | ⚠️ Optional (IEC 62351-6) | ❌ Keine | Mittel |
| Profinet | Ethernet | ❌ Keine | ❌ Keine | Sehr hoch (DE Industrie) |
| EtherNet/IP | TCP/44818 | ❌ Keine | ❌ Keine | Hoch (US/global) |
| OPC UA | TCP/4840 | ✅ Zertifikat/User | ✅ TLS | Wachsend |
| MQTT (IIoT) | TCP/1883,8883 | ⚠️ Optional | ⚠️ Optional TLS | Wachsend IIoT |

---

## Modbus — Sicherheitsanalyse und Kompensation

```python
#!/usr/bin/env python3
"""Modbus TCP Sicherheits-Audit — passive Analyse"""
from pymodbus.client import ModbusTcpClient
import ipaddress, socket

def audit_modbus_device(host: str, port: int = 502):
    """Prüft ob Modbus TCP ohne Authentifizierung erreichbar ist"""
    print(f"\n[*] Modbus Audit: {host}:{port}")

    client = ModbusTcpClient(host, port=port, timeout=3)
    if not client.connect():
        print(f"  [-] Nicht erreichbar")
        return False

    print(f"  [!] WARNUNG: Modbus TCP ohne Auth erreichbar!")

    # Holding Register lesen (Unit ID 1, Register 0-9)
    result = client.read_holding_registers(0, 10, unit=1)
    if not result.isError():
        print(f"  [!] Register 0-9 lesbar: {result.registers}")
        print(f"      → Geräteparameter ohne Authentifizierung auslesbar!")

    # Coil lesen (digitale Ausgänge)
    coils = client.read_coils(0, 8, unit=1)
    if not coils.isError():
        print(f"  [!] Coils 0-7 lesbar: {coils.bits[:8]}")
        print(f"      → Schaltzustände ohne Auth auslesbar!")

    client.close()
    return True

# Netz-Scan auf Modbus (NUR in autorisierten Tests!)
def scan_modbus_network(network: str):
    net = ipaddress.ip_network(network, strict=False)
    vulnerable = []
    for ip in net.hosts():
        if audit_modbus_device(str(ip)):
            vulnerable.append(str(ip))
    print(f"\nErgebnis: {len(vulnerable)} Modbus-Geräte ohne Auth gefunden")
    return vulnerable
```

### Modbus-Kompensationsmaßnahmen

```
DA MODBUS KEINE NATIVE SICHERHEIT HAT:

Option 1: Netzwerksegmentierung (Minimum)
  - Modbus-Geräte in eigenem VLAN
  - Nur autorisierte HMI/SCADA darf auf Port 502 zugreifen
  - Industrial Firewall mit Deep Packet Inspection

Option 2: VPN-Tunnel
  - Modbus-Traffic über IPsec/WireGuard tunneln
  - Authentifizierung und Verschlüsselung auf Netzwerkebene

Option 3: Kryptografische Appliance
  - Hardware-Box vor Modbus-Gerät (z.B. Tofu Networks, Waterfall)
  - Transparente Verschlüsselung ohne Geräteänderung

Option 4: Protokoll-Migration
  - Langfristig: Migration auf OPC UA
  - OPC UA bietet native Auth + Verschlüsselung
  - Modbus-zu-OPC-UA-Gateway als Zwischenlösung

FIREWALL-REGELN (nftables Beispiel):
  # Nur SCADA-Server darf Modbus lesen
  nft add rule inet filter forward \
      ip daddr 192.168.50.10 tcp dport 502 \
      ip saddr 192.168.10.5 accept

  # Alle anderen Verbindungen zu Modbus blockieren
  nft add rule inet filter forward \
      tcp dport 502 drop
```

---

## DNP3 Secure Authentication v5 (SAv5)

```
DNP3 SAv5 — Challenge-Response Authentifizierung:

ABLAUF:
  1. Outstation (RTU) sendet Challenge an Master (SCADA)
  2. Master berechnet HMAC-SHA256 mit Key
  3. Outstation verifiziert HMAC → Authentifizierung OK
  4. Antwort wird mit MAC versehen

KONFIGURATION DNP3 SAv5 (Konzept):
  - Schlüsselmanagement: Update Key + Session Keys
  - Update Key: langfristiger Schlüssel (Jahre)
  - Session Keys: kurzfristig, regelmäßig rotiert
  - Schlüsselaustausch: Sicherer Kanal (physisch oder verschlüsselt)

GERÄTE MIT SAv5 UNTERSTÜTZUNG:
  - Schweitzer Engineering Laboratories (SEL)
  - General Electric Grid Solutions
  - Schneider Electric (bestimmte RTU-Modelle)
  - ABB (bestimmte RTU-Modelle)

WICHTIG: Viele Altgeräte unterstützen SAv5 NICHT
  → Kompensation: VPN + Netzwerksegmentierung
```

---

## IEC 61850 GOOSE-Sicherheit

```python
#!/usr/bin/env python3
"""IEC 61850 GOOSE Packet Analyse (mit Scapy)"""
from scapy.all import sniff, Ether
import struct

GOOSE_ETHERTYPE = 0x88B8

def analyze_goose_packet(pkt):
    if pkt.haslayer(Ether) and pkt[Ether].type == GOOSE_ETHERTYPE:
        payload = bytes(pkt[Ether].payload)

        # GOOSE PDU analysieren
        print(f"\n[GOOSE] Von: {pkt[Ether].src} → {pkt[Ether].dst}")

        # Prüfen ob Authentication Tag vorhanden (IEC 62351-6)
        # Tag 0x85 im GOOSE-PDU = Authentication Tag
        if b'\x85' not in payload:
            print(f"  [!] WARNUNG: Kein Authentication Tag!")
            print(f"      → GOOSE kann gefälscht werden (GOOSE Spoofing möglich!)")
        else:
            print(f"  [+] Authentication Tag vorhanden (IEC 62351-6)")

        # Status-Nummer extrahieren (St Num = Zustandswechsel-Zähler)
        # Erhöhung von St Num = Zustandswechsel (z.B. Schutzauslösung)
        print(f"  Payload-Länge: {len(payload)} Bytes")

# GOOSE-Traffic mitschneiden (NUR in autorisierten Tests!)
# sniff(iface="eth0", prn=analyze_goose_packet, filter="ether proto 0x88b8")
```

### GOOSE-Angriff: Spoofing-Risiko

```
GOOSE SPOOFING — Angriffsszenario:

1. Angreifer im Netz (kompromittiertes Gerät oder physischer Zugang)
2. GOOSE-Paket sniffen → GoID, DataSetRef, StNum extrahieren
3. Gefälschtes GOOSE-Paket mit höherem StNum senden
4. Schutzgerät (IED) interpretiert als legitimes Signal
5. Mögliche Folge: Ungewollte Schutzauslösung / Leistungsschalter öffnet

GEGENMASSNAHMEN:
  ✓ IEC 62351-6: HMAC-SHA256 Authentication Tag aktivieren
  ✓ Netzwerk-Isolierung: GOOSE-VLAN, nur IEDs darin
  ✓ Port-Security auf Switches (nur bekannte MACs)
  ✓ IDS/IPS für IEC 61850 (Claroty, Nozomi: GOOSE-Anomalie-Erkennung)
  ✓ GOOSE-Traffic nicht über unsichere WAN-Strecken
```

---

## OPC UA Security-Konfiguration

```python
#!/usr/bin/env python3
"""OPC UA Client mit korrekter Sicherheitskonfiguration"""
from opcua import Client
from opcua.crypto import security_policies

# UNSICHER (niemals in Produktion!):
# client = Client("opc.tcp://plc-01:4840")  # Anonymous, kein TLS

# SICHER (Zertifikat-basiert):
client = Client("opc.tcp://plc-01.oem.local:4840")

# Sicherheitspolitik: Basic256Sha256 (Minimum ab 2024)
# Besser: Aes256Sha256RsaPss
client.set_security(
    security_policies.SecurityPolicyBasic256Sha256,
    certificate="/etc/opcua/client_cert.pem",
    private_key="/etc/opcua/client_key.pem",
    server_certificate="/etc/opcua/server_cert.pem"
)

# Benutzername + Passwort (zusätzlich zum Zertifikat)
client.set_user("opcua-scada")
client.set_password("Sicheres-OPC-Passwort!")

try:
    client.connect()
    print(f"Verbunden: {client.get_server_node().nodeid}")

    # Namespace-Index ermitteln
    idx = client.get_namespace_index("urn:firma:PLC01:DataModel")

    # Variable lesen
    temp_node = client.get_node(f"ns={idx};s=Kessel.Temperatur")
    temperature = temp_node.get_value()
    print(f"Temperatur: {temperature}°C")

    # Subscription für Änderungen
    sub = client.create_subscription(500, handler)
    handle = sub.subscribe_data_change(temp_node)

finally:
    client.disconnect()
```

---

## Protokoll-Sicherheits-Checkliste

```markdown
## OT-Protokoll Security-Audit — Checkliste

### Inventarisierung
- [ ] Alle OT-Protokolle im Netz dokumentiert
- [ ] Protokoll-Version je Gerät bekannt
- [ ] Protokoll-Port-Nutzung in Netzwerk-Diagram eingetragen

### Modbus TCP
- [ ] Kein Modbus direkt aus Internet erreichbar
- [ ] Modbus nur zwischen autorisierten Systemen erlaubt (Firewall)
- [ ] Kompensationsmaßnahme dokumentiert (VPN / Segmentierung)

### DNP3
- [ ] DNP3 Secure Authentication v5 aktiviert (wo unterstützt)
- [ ] Für Altgeräte ohne SAv5: VPN-Tunnel dokumentiert
- [ ] DNP3-Traffic nicht über ungesicherte WAN-Strecken

### IEC 60870-5-104
- [ ] IEC 62351-5 (SAv5-äquivalent) aktiviert (wo unterstützt)
- [ ] TLS-Tunnel als Kompensation (wo 62351 nicht möglich)

### IEC 61850 GOOSE
- [ ] GOOSE-VLAN isoliert (nur IEDs im Segment)
- [ ] IEC 62351-6 Authentication Tag aktiviert (wo unterstützt)
- [ ] IDS/IPS überwacht GOOSE-Anomalien

### OPC UA
- [ ] Kein Anonymous Access
- [ ] Security Policy: Basic256Sha256 oder stärker
- [ ] Message Security: SignAndEncrypt (niemals None!)
- [ ] Zertifikate: PKI-verwaltete Zertifikate (kein Self-Signed in Prod)

### Allgemein
- [ ] Protokoll-Traffic wird von OT-IDS überwacht
- [ ] Anomalie-Erkennung für ungewöhnliche Befehle/Werte konfiguriert
- [ ] Regelmäßiger Protokoll-Audit (halbjährlich)
```
