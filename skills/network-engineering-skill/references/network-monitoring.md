# Netzwerk-Monitoring — Referenzmodul

SNMP, NetFlow, Syslog, PRTG, LibreNMS, Zabbix, Packet-Capture, Performance-Analyse.

---

## SNMP v3 Monitoring

```python
#!/usr/bin/env python3
"""SNMP v3 Polling — Bandbreite, CPU, Fehler je Interface"""
from pysnmp.hlapi import *
import time

DEVICES = [
    {"host": "sw-core-01.firma.local", "port": 161},
    {"host": "rtr-wan-01.firma.local", "port": 161},
]
SNMP_USER    = "snmp-monitor"
AUTH_KEY     = "Auth-Passwort-Sicher!"
PRIV_KEY     = "Priv-Passwort-Sicher!"

# Wichtige OIDs
OID_IFDESCR       = "1.3.6.1.2.1.2.2.1.2"    # Interface-Name
OID_IFINOCTETS    = "1.3.6.1.2.1.2.2.1.10"   # Eingehende Bytes
OID_IFOUTOCTETS   = "1.3.6.1.2.1.2.2.1.16"   # Ausgehende Bytes
OID_IFINDISCARDS  = "1.3.6.1.2.1.2.2.1.13"   # Eingehende Drops
OID_IFOUTDISCARDS = "1.3.6.1.2.1.2.2.1.19"   # Ausgehende Drops
OID_IFINERRORS    = "1.3.6.1.2.1.2.2.1.14"   # Eingehende Fehler
OID_CPU_CISCO     = "1.3.6.1.4.1.9.2.1.58.0" # Cisco CPU 5min
OID_MEM_CISCO     = "1.3.6.1.4.1.9.9.48.1.1.1.6.1" # Cisco Free Memory

def snmp_get(host, oid):
    iterator = getCmd(
        SnmpEngine(),
        UsmUserData(SNMP_USER,
            authKey=AUTH_KEY, privKey=PRIV_KEY,
            authProtocol=usmHMACSHAAuthProtocol,
            privProtocol=usmAesCfb128Protocol),
        UdpTransportTarget((host, 161), timeout=5, retries=2),
        ContextData(),
        ObjectType(ObjectIdentity(oid))
    )
    errorIndication, errorStatus, _, varBinds = next(iterator)
    if errorIndication or errorStatus:
        return None
    return int(varBinds[0][1])

def poll_interface_traffic(host, if_index, interval=60):
    """Bandbreite berechnen (bps)"""
    bytes1 = snmp_get(host, f"{OID_IFINOCTETS}.{if_index}")
    time.sleep(interval)
    bytes2 = snmp_get(host, f"{OID_IFINOCTETS}.{if_index}")
    if bytes1 and bytes2:
        bps = ((bytes2 - bytes1) * 8) / interval
        mbps = round(bps / 1_000_000, 2)
        print(f"{host} If{if_index}: {mbps} Mbps IN")
        return mbps
    return None

# CPU-Warnung
def check_cpu(host, threshold=80):
    cpu = snmp_get(host, OID_CPU_CISCO)
    if cpu and cpu > threshold:
        print(f"⚠️  WARNUNG: {host} CPU {cpu}% (Schwellenwert: {threshold}%)")
        # → Alert senden (E-Mail, Slack, PagerDuty)
```

---

## NetFlow / sFlow Analyse

```bash
# nfdump — NetFlow-Analyse (Linux)

# Top-10 Talker (größte Datenmengen)
nfdump -r /var/cache/nfdump/2024/01/15/nfcapd.202401150800 \
    -s record/bytes -n 10 \
    -o "fmt:%sa %da %sp %dp %byt %pkt"

# Traffic einer spezifischen IP
nfdump -r /var/cache/nfdump/ -R 2024/01/15/nfcapd.202401150800 \
    "host 10.0.30.5" \
    -o "fmt:%ts %sa %da %sp %dp %pr %byt"

# Verbindungen zu einer bestimmten Ziel-IP
nfdump -r /var/cache/nfdump/ -R . \
    "dst host 8.8.8.8" \
    -s record/flows -n 20

# Port-Scan Erkennung (viele Zielports von einer Source)
nfdump -r /var/cache/nfdump/ \
    "proto tcp and flags S and not flags ARSE" \
    -s dstport/flows -n 50

# Bandbreite je Interface (mit nfsen/ntopng)
# Oder direkt mit nfdump + aggregation
nfdump -r /var/cache/nfdump/ \
    -A srcip,dstip \
    -s bytes -n 20 \
    -o "fmt:%sa -> %da: %byt Bytes, %pkt Pakete"
```

### NetFlow Konfiguration (Cisco)

```
! IOS-XE: Flexible NetFlow
flow record NETFLOW-RECORD
 match ipv4 source address
 match ipv4 destination address
 match transport source-port
 match transport destination-port
 match ipv4 protocol
 match ip tos
 match interface input
 collect transport tcp flags
 collect counter bytes
 collect counter packets
 collect timestamp sys-uptime first
 collect timestamp sys-uptime last

flow exporter NETFLOW-EXPORTER
 destination 10.0.0.200
 source Loopback0
 transport udp 2055
 version 9

flow monitor NETFLOW-MONITOR
 record NETFLOW-RECORD
 exporter NETFLOW-EXPORTER
 cache timeout active 60
 cache timeout inactive 15

interface GigabitEthernet0/1
 ip flow monitor NETFLOW-MONITOR input
 ip flow monitor NETFLOW-MONITOR output
```

---

## LibreNMS (Alerting + Monitoring)

```php
# LibreNMS Alert-Regeln (SQL-ähnliche Syntax)

# Regel 1: Interface Down
%macros.if_down = "1" AND %devices.disabled = "0"

# Regel 2: Hohe Bandbreite (> 80% eines Links)
%ports.ifInOctets_rate > (%ports.ifSpeed * 0.8 / 8)
OR %ports.ifOutOctets_rate > (%ports.ifSpeed * 0.8 / 8)

# Regel 3: BGP-Peer Down
%bgpPeers.bgpPeerState != "established"

# Regel 4: OSPF-Neighbor Down
%ospf_nbrs.ospfNbrState != "full"

# Regel 5: Gerät nicht erreichbar
%devices.status = "0" AND %devices.disabled = "0"
```

```bash
# LibreNMS CLI-Befehle
# Gerät hinzufügen
lnms device:add sw-core-01.firma.local --v3 --authlevel authPriv \
    --authname snmp-monitor --authpass "Auth-Pass" \
    --cryptopass "Priv-Pass" --authprotocol SHA --cryptoprotocol AES

# Discovery manuell auslösen
php /opt/librenms/discovery.php -h sw-core-01.firma.local

# Polling manuell
php /opt/librenms/poller.php -h sw-core-01.firma.local -r

# Alle Alerts anzeigen
lnms alerts:list

# Alerts testen
lnms alert:test --device-id 5 --rule-id 12
```

---

## Zabbix (Enterprise-Monitoring)

```yaml
# Zabbix Template via API (Python)
import requests

ZABBIX_URL = "https://zabbix.firma.de/api_jsonrpc.php"

def zabbix_login():
    resp = requests.post(ZABBIX_URL, json={
        "jsonrpc": "2.0", "method": "user.login",
        "params": {"username": "Admin", "password": "GeheimPass"},
        "id": 1
    })
    return resp.json()["result"]

def create_host(token, hostname, ip, template_ids, group_ids):
    resp = requests.post(ZABBIX_URL, json={
        "jsonrpc": "2.0", "method": "host.create",
        "auth": token,
        "params": {
            "host": hostname,
            "interfaces": [{"type": 2, "main": 1, "useip": 1,
                           "ip": ip, "dns": "", "port": "161",
                           "details": {
                               "version": 3, "securityname": "snmp-monitor",
                               "authprotocol": 1, "authpassphrase": "Auth-Pass",
                               "privprotocol": 1, "privpassphrase": "Priv-Pass",
                               "securitylevel": 2
                           }}],
            "groups": [{"groupid": gid} for gid in group_ids],
            "templates": [{"templateid": tid} for tid in template_ids],
        }, "id": 1
    })
    return resp.json()
```

---

## Packet Capture & Analyse

```bash
# ─── TCPDUMP ──────────────────────────────────────────────────────────────────
# Capture auf Interface
tcpdump -i eth0 -n -s 0 -w /tmp/capture.pcap 'host 10.0.30.5'

# Verbindungen zu Port 443 (ohne SSH-Traffic)
tcpdump -i eth0 -n 'port 443 and not port 22'

# ICMP-Pakete
tcpdump -i eth0 -n 'icmp'

# Roher Hex-Output (für Protokoll-Debug)
tcpdump -i eth0 -XX -s 0 'host 10.0.30.5 and port 80'

# ─── TSHARK (Wireshark CLI) ────────────────────────────────────────────────────
# Live-Anzeige HTTP-Requests
tshark -i eth0 -Y 'http.request' -T fields \
    -e ip.src -e http.host -e http.request.uri

# DNS-Abfragen analysieren
tshark -i eth0 -Y 'dns.qry.type == 1' -T fields \
    -e ip.src -e dns.qry.name

# TLS-Zertifikate anzeigen
tshark -i eth0 -Y 'tls.handshake.type == 11' -T fields \
    -e ip.src -e ip.dst -e tls.handshake.certificate

# PCAP analysieren (ohne Live-Capture)
tshark -r /tmp/capture.pcap -z io,stat,1 -q    # Traffic-Statistik pro Sekunde
tshark -r /tmp/capture.pcap -z conv,tcp -q      # TCP-Konversationen

# ─── NMAP NETZWERK-SCAN ────────────────────────────────────────────────────────
# Host-Discovery
nmap -sn 10.0.0.0/24 -oG hosts-alive.txt

# Port-Scan + Service-Erkennung
nmap -sV -p 22,80,443,8080,8443 10.0.30.0/24

# OS-Erkennung
nmap -O 10.0.30.5

# Vulnerability-Scan (NSE-Scripts)
nmap --script=vuln 10.0.30.5
