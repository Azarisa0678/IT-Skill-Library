---
name: network-engineering-skill
description: >
  Network Engineering Skill for network administrators and IT engineers.
  Covers: Cisco IOS IOS-XE NX-OS, Palo Alto PAN-OS, Fortinet FortiGate, HP Aruba,
  routing protocols BGP OSPF EIGRP, switching VLANs STP VPC LACP port-channel,
  SD-WAN Cisco Viptela Fortinet Meraki, network security firewall rules NAT VPN IPsec SSL,
  network monitoring SNMP NetFlow Syslog PRTG LibreNMS Zabbix,
  network design spine-leaf three-tier campus WAN, IPv6, QoS, troubleshooting.
  Triggers: Cisco, BGP, OSPF, EIGRP, VLAN, STP, spanning-tree, SD-WAN, Palo Alto,
  FortiGate, firewall, NAT, IPsec, VPN, LACP, port-channel, network design,
  spine-leaf, QoS, SNMP, NetFlow, Syslog, network troubleshooting, routing, switching.
  Outputs: CLI configuration snippets, network diagrams text-based, troubleshooting guides.
---

# Network Engineering Skill

## Guiding Principles

**Defense in Depth.** Segmentierung ist die erste Verteidigungslinie.
**Dokumentation vor Konfiguration.** Design zuerst, dann implementieren. Change Request Pflicht.
**Least Privilege.** Default-Deny an Firewalls und ACLs.
**Redundanz ohne Komplexitaet.** HA einbauen, Komplexitaet minimieren.

## Cisco IOS - Grundkonfiguration

```cisco
hostname SW-CORE-01
!
ip domain-name firma.local
crypto key generate rsa modulus 4096
ip ssh version 2
ip ssh authentication-retries 3
!
line vty 0 15
 transport input ssh
 login local
 exec-timeout 10 0
!
username admin privilege 15 secret 0 [PASSWORT]
enable secret 0 [ENABLE_SECRET]
!
no service pad
no ip http server
no ip http secure-server
no cdp run
!
logging buffered 65536 informational
logging host 192.168.1.100
!
snmp-server group MONITORING v3 priv
no snmp-server community public
no snmp-server community private
!
spanning-tree mode rapid-pvst
```

## VLAN-Setup (Access Switch)

```cisco
vlan 10
 name BENUTZER
vlan 20
 name SERVER
vlan 99
 name NATIVE-UNUSED
!
interface GigabitEthernet1/0/1
 description PC-Buero-101
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable
 storm-control broadcast level 20.00
!
interface GigabitEthernet1/0/48
 description UPLINK-CORE
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20
!
ip dhcp snooping
ip dhcp snooping vlan 10,20
ip arp inspection vlan 10,20
```

## Palo Alto Firewall

```
Zonen:
UNTRUST (Internet) -> DMZ -> TRUST (LAN) -> MANAGEMENT
Default-Deny mit Logging auf alle Zonen

Security Policy Best Practices:
- App-ID statt Port-basiert
- Security Profiles auf alle Allow-Regeln (AV, IPS, URL-Filter)
- Spezifischste Regeln zuerst
- Default-Deny am Ende mit Logging
```

## FortiGate IPsec VPN

```bash
config vpn ipsec phase1-interface
    edit "VPN-STANDORT-B"
        set interface "wan1"
        set ike-version 2
        set proposal aes256-sha256
        set dhgrp 14
        set remote-gw 203.0.113.50
        set psksecret "SICHERES_PSK"
    next
end

config vpn ipsec phase2-interface
    edit "VPN-STANDORT-B-P2"
        set phase1name "VPN-STANDORT-B"
        set proposal aes256-sha256
        set pfs enable
        set dhgrp 14
        set src-subnet 192.168.1.0 255.255.255.0
        set dst-subnet 192.168.2.0 255.255.255.0
    next
end
```

## Troubleshooting

```cisco
! Allgemein
show interfaces status
show ip interface brief
show vlan brief
show mac address-table
show spanning-tree vlan 10
show port-channel summary

! Routing
show ip route
show ip ospf neighbor
show ip bgp summary

! Diagnose
ping 192.168.1.1 source GigabitEthernet0/0
traceroute 8.8.8.8
show logging | include %LINK
```

## Bereiche

- references/cisco-ios.md - IOS/IOS-XE/NX-OS, VLANs, OSPF, BGP, ACLs
- references/firewall-config.md - Palo Alto, FortiGate, Zonen, NAT, VPN
- references/routing-protocols.md - BGP, OSPF, Route-Maps, Prefix-Listen
- references/network-design.md - Spine-Leaf, Campus, WAN, IP-Planung
- references/network-monitoring.md - SNMP, NetFlow, PRTG, LibreNMS
