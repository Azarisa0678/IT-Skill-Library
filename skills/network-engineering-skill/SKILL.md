---
name: network-engineering-skill
description: >
  Network Engineering Skill fuer Netzwerkadministratoren.
  Cisco IOS, IOS-XE, NX-OS, Hardening, VLANs, Trunking, EtherChannel, LACP, STP, RSTP,
  BGP, OSPF, EIGRP, VRF, PBR, Policy-Based Routing, Route Redistribution,
  Palo Alto, PAN-OS, FortiGate, FortiOS, Cisco ASA, ZBF, Firewall, NAT, DMZ,
  Spine-Leaf, Three-Tier, Campus, HSRP, VRRP, QoS, DSCP, IPv6, SD-WAN, VXLAN, EVPN,
  SNMP, NetFlow, sFlow, Syslog, LibreNMS, Zabbix, PRTG, nfdump,
  Netzwerk-Design, Netzwerkarchitektur, Segmentierung, Firewall-Audit,
  tcpdump, tshark, nmap, mtr, troubleshooting.
  Outputs: CLI-Konfigurationssnippets, ASCII-Diagramme, Troubleshooting-Guides.
---

# Network Engineering Skill

## Guiding Principles

**Defense in Depth.** Segmentierung + Firewall + IPS + NAC — niemals nur eine Schutzschicht.
**Least Privilege für Netzwerkzugriff.** ACLs und Firewall-Regeln nach Minimalprinzip — Default Deny.
**Kein Telnet, kein SNMP v1/v2c, kein HTTP-Management.** SSH v2, SNMP v3, HTTPS immer.
**Dokumentation ist Pflicht.** Jede Regel, jede Route, jeder VLAN hat einen dokumentierten Zweck.
**Change-Management vor jeder Änderung.** Rollback-Plan definieren bevor produktive Änderungen.
**Monitoring first.** Kein Gerät ohne SNMP, Syslog und NetFlow-Konfiguration in Produktion.

---

## Referenzmodule

- `references/cisco-ios.md` — IOS/IOS-XE/NX-OS Hardening, VLANs, Trunking, EtherChannel, STP, Troubleshooting
- `references/routing-protocols.md` — BGP, OSPF, EIGRP, VRF, PBR, Route Redistribution
- `references/firewall-config.md` — Palo Alto PAN-OS, FortiGate, Cisco ZBF, NAT, Audit-Checkliste
- `references/network-design.md` — Spine-Leaf, Three-Tier, HSRP, QoS, IPv6, SD-WAN, VXLAN
- `references/network-monitoring.md` — SNMP v3, NetFlow/nfdump, LibreNMS, Zabbix, tcpdump, tshark

Routing:
- Cisco IOS/NX-OS / Switch-Konfiguration → `cisco-ios.md`
- BGP / OSPF / EIGRP / Routing → `routing-protocols.md`
- Palo Alto / FortiGate / Firewall-Regeln → `firewall-config.md`
- Netzwerk-Design / Architektur / QoS → `network-design.md`
- Monitoring / NetFlow / Packet Capture → `network-monitoring.md`

---

## Cisco — Schnellreferenz

```
! Härtungs-Essentials (immer setzen)
hostname SW-CORE-01
no service telnet
ip ssh version 2
crypto key generate rsa modulus 4096
line vty 0 15
 transport input ssh
 login authentication default
 exec-timeout 10 0

! VLAN + Trunk
vlan 10,20,30,100,999
interface GigabitEthernet1/0/1
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 10,20,30
 switchport nonegotiate

! Access Port mit Härtung
interface GigabitEthernet1/0/10
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 spanning-tree bpduguard enable
 storm-control broadcast level 20.00
```

→ Vollständige Härtung (AAA/TACACS+, SNMP v3, NTP, Banner), EtherChannel, NX-OS: `references/cisco-ios.md`

---

## Routing — Schnellreferenz

```
! OSPF (Single-Area)
router ospf 1
 router-id 10.255.255.1
 auto-cost reference-bandwidth 100000
 passive-interface default
 no passive-interface GigabitEthernet0/1

interface GigabitEthernet0/1
 ip ospf 1 area 0
 ip ospf network point-to-point
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 OSPF-KEY

! BGP (zu ISP)
router bgp 65001
 neighbor 203.0.113.254 remote-as 1234
 neighbor 203.0.113.254 ttl-security hops 1
 neighbor 203.0.113.254 prefix-list PL-ISP-IN in
 neighbor 203.0.113.254 prefix-list PL-ISP-OUT out
```

→ BGP Route Reflector, EIGRP Named Mode, VRF, PBR, Route Redistribution: `references/routing-protocols.md`

---

## Firewall — Schnellreferenz

```bash
# Palo Alto: Policy testen
test security-policy-match from TRUST to UNTRUST \
    source 10.0.30.5 destination 8.8.8.8 destination-port 443 protocol 6

# Palo Alto: Live Packet Capture
debug dataplane packet-diag set filter match source 10.0.30.5
debug dataplane packet-diag set capture stage firewall file /tmp/cap.pcap
debug dataplane packet-diag set capture on

# FortiGate: Policy Match
diagnose firewall iprope lookup 10.0.30.5 8.8.8.8 6 1024 443 internal

# FortiGate: Live Sniffer
diagnose sniffer packet any 'host 10.0.30.5 and port 443' 4 100 l
```

→ Vollständige PAN-OS/FortiOS CLI-Konfiguration, NAT, SD-WAN, Firewall-Audit-Checkliste: `references/firewall-config.md`

---

## Monitoring — Schnellreferenz

```bash
# NetFlow Top-Talker
nfdump -r /var/cache/nfdump/ -s record/bytes -n 10 -o "fmt:%sa %da %byt"

# Packet Capture
tcpdump -i eth0 -n -w /tmp/cap.pcap 'host 10.0.30.5 and port 443'
tshark -i eth0 -Y 'http.request' -T fields -e ip.src -e http.host

# Nmap Host-Discovery
nmap -sn 10.0.0.0/24

# SNMP v3 Test
snmpget -v3 -u snmp-monitor -l authPriv \
    -a SHA -A "AuthPass" -x AES -X "PrivPass" \
    10.0.0.1 1.3.6.1.2.1.1.1.0
```

→ SNMP v3 Python-Poller, NetFlow-Konfiguration, LibreNMS/Zabbix-Setup: `references/network-monitoring.md`

---

## Output-Formate

| Aufgabe | Format |
|---|---|
| Switch/Router-Konfiguration | CLI-Konfigurationsblöcke mit Kommentaren |
| Firewall-Regeln | Kontext: Zone/Interface → Regel → Sicherheitsprofil |
| Netzwerk-Design | ASCII-Topologie + Erklärung der Designentscheidungen |
| Troubleshooting | Symptom → Diagnose-Befehle → Ursachen → Lösung |
| Firewall-Audit | Markdown-Checkliste mit Ja/Nein-Prüfpunkten |
| Routing-Konfiguration | IOS/NX-OS CLI mit Verifikationsbefehlen |
