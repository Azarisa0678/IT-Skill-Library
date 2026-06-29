# Cisco IOS / IOS-XE / NX-OS — Referenzmodul

Konfigurationsreferenz, Hardening, Troubleshooting, VLANs, Trunking, EtherChannel, Spanning Tree.

---

## Grundkonfiguration (Hardening)

```
! Basis-Härtung für Cisco IOS / IOS-XE
! ─── HOSTNAME & DOMAIN ────────────────────────────────────────────────────────
hostname SW-CORE-01
ip domain-name firma.local
ip name-server 10.0.0.53 10.0.0.54

! ─── BENUTZER & AUTHENTIFIZIERUNG ────────────────────────────────────────────
username admin privilege 15 algorithm-type scrypt secret StarkesPasswort123!
enable algorithm-type scrypt secret EnableGeheim!
aaa new-model
aaa authentication login default group tacacs+ local
aaa authorization exec default group tacacs+ local
aaa accounting exec default start-stop group tacacs+

tacacs server TACACS-SRV
 address ipv4 10.0.0.100
 key 7 TACACS-SECRET

! ─── SSH (nur v2, kein Telnet) ────────────────────────────────────────────────
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
ip ssh source-interface Loopback0
crypto key generate rsa modulus 4096
no service telnet        ! Telnet global deaktivieren

line vty 0 15
 transport input ssh
 login authentication default
 exec-timeout 10 0
 logging synchronous

line con 0
 login authentication default
 exec-timeout 5 0
 logging synchronous

! ─── SERVICES DEAKTIVIEREN ───────────────────────────────────────────────────
no service pad
no service tcp-small-servers
no service udp-small-servers
no ip bootp server
no ip http server
no ip http secure-server    ! Nur deaktivieren wenn kein HTTPS-Management nötig
no ip finger
no ip source-route
no cdp run                  ! CDP nur intern, extern deaktivieren
no lldp run                 ! LLDP ebenso (falls nicht benötigt)
service password-encryption
service timestamps log datetime msec localtime show-timezone
service timestamps debug datetime msec localtime

! ─── LOGGING ──────────────────────────────────────────────────────────────────
logging buffered 16384 informational
logging trap informational
logging source-interface Loopback0
logging host 10.0.0.200
logging facility local6

! ─── NTP ──────────────────────────────────────────────────────────────────────
ntp server 10.0.0.10 prefer
ntp server 10.0.0.11
ntp authenticate
ntp authentication-key 1 md5 NTP-SECRET
ntp trusted-key 1
clock timezone CET 1
clock summer-time CEST recurring last Sun Mar 2:00 last Sun Oct 3:00

! ─── SNMP v3 (kein v1/v2c in Produktion) ─────────────────────────────────────
snmp-server group MGMT-GROUP v3 priv
snmp-server user snmp-admin MGMT-GROUP v3 auth sha AuthPasswd priv aes 128 PrivPasswd
snmp-server location "RZ Frankfurt, Schrank 3"
snmp-server contact it-ops@firma.de
no snmp-server community public    ! v1/v2c Communities entfernen
no snmp-server community private

! ─── BANNER ───────────────────────────────────────────────────────────────────
banner login ^
 *** ACHTUNG: Nur autorisierter Zugang! ***
 *** Alle Aktivitaeten werden protokolliert. ***
^
```

---

## VLANs und Trunking

```
! ─── VLAN-DEFINITION (Distribution/Core Switch) ──────────────────────────────
vlan 10
 name MGMT
vlan 20
 name SERVER
vlan 30
 name CLIENTS
vlan 40
 name VOIP
vlan 50
 name DMZ
vlan 100
 name NATIVE-TRUNK    ! Native VLAN von Default (1) trennen
vlan 999
 name PARKING         ! Ungenutzte Ports parken

! ─── TRUNK-PORT (zu Router/Core-Switch) ──────────────────────────────────────
interface GigabitEthernet1/0/1
 description TRUNK-zu-CORE-01
 switchport mode trunk
 switchport trunk native vlan 100      ! Nicht VLAN 1 als Native!
 switchport trunk allowed vlan 10,20,30,40,50
 switchport nonegotiate                ! DTP deaktivieren
 no shutdown

! ─── ACCESS-PORT (Client/Server) ─────────────────────────────────────────────
interface GigabitEthernet1/0/10
 description CLIENT-PC-BUERO-01
 switchport mode access
 switchport access vlan 30
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip dhcp snooping limit rate 15
 storm-control broadcast level 20.00
 storm-control action shutdown
 no cdp enable
 no lldp transmit
 no lldp receive
 no shutdown

! ─── VOICE VLAN ───────────────────────────────────────────────────────────────
interface GigabitEthernet1/0/20
 description IP-PHONE-BUERO-20
 switchport mode access
 switchport access vlan 30
 switchport voice vlan 40
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown

! ─── UNUSED PORTS DEAKTIVIEREN ───────────────────────────────────────────────
interface range GigabitEthernet1/0/40 - 48
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
```

---

## EtherChannel (LACP)

```
! ─── LACP ETHERCHANNEL (802.3ad) ──────────────────────────────────────────────
! Beide Seiten konfigurieren!

! Switch A:
interface range GigabitEthernet1/0/1 - 2
 description LACP-zu-SWITCH-B
 channel-group 1 mode active    ! active = LACP initiiert
 channel-protocol lacp
 no shutdown

interface Port-channel1
 description PC-BUNDLE-zu-SWITCH-B
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 10,20,30,40,50

! Switch B:
interface range GigabitEthernet1/0/1 - 2
 channel-group 1 mode passive   ! passive = LACP antwortet
 channel-protocol lacp
 no shutdown

! Verifizierung:
! show etherchannel summary
! show lacp neighbor
! show interface port-channel 1
```

---

## Spanning Tree (RSTP / MSTP)

```
! ─── RAPID PVST+ (Standard) ──────────────────────────────────────────────────
spanning-tree mode rapid-pvst
spanning-tree extend system-id

! Root Bridge definieren (Core Switch)
spanning-tree vlan 10,20,30 priority 4096      ! Primary Root
spanning-tree vlan 40,50    priority 8192      ! Secondary Root

! Auf Distribution/Access Switches:
spanning-tree vlan 10,20,30 priority 8192      ! Secondary für VLANs 10-30
spanning-tree vlan 40,50    priority 4096      ! Primary für VLANs 40-50

! ─── SICHERHEITSFUNKTIONEN ────────────────────────────────────────────────────
! Global:
spanning-tree portfast default           ! PortFast auf allen Access Ports
spanning-tree portfast bpduguard default ! BPDU Guard auf PortFast Ports

! Root Guard (verhindert dass ein Port Root wird):
interface GigabitEthernet1/0/24
 spanning-tree guard root

! BPDU Filter (vorsichtig verwenden!):
! spanning-tree portfast bpdufilter default  ! Nur in speziellen Fällen

! Verifizierung:
! show spanning-tree vlan 30
! show spanning-tree summary
! show spanning-tree inconsistentports
```

---

## NX-OS (Nexus) — Wesentliche Unterschiede

```
! NX-OS Feature-Aktivierung (anders als IOS!)
feature ospf
feature bgp
feature vpc
feature lacp
feature lldp
feature tacacs+

! vPC (Virtual Port Channel) — Dual-Homing ohne STP-Blockierung
vpc domain 10
 peer-keepalive destination 10.0.0.2 source 10.0.0.1 vrf management
 peer-switch
 auto-recovery

interface port-channel10
 vpc peer-link       ! vPC Peer-Link

interface port-channel20
 vpc 20              ! vPC Member

! NX-OS VRF
vrf context MGMT
 ip route 0.0.0.0/0 10.0.0.1

! NX-OS VLAN (anders als IOS: vlan database)
vlan 10-50,100,999
```

---

## Troubleshooting-Referenz

```
KONNEKTIVITÄTSPROBLEME:
show ip interface brief          → Interface-Status
show interfaces GiX/X/X         → Fehler, CRC, Input/Output-Drops
show mac address-table           → MAC-Tabelle
show arp                         → ARP-Tabelle
ping X.X.X.X source Loopback0   → Konnektivitätstest mit Source-IP
traceroute X.X.X.X               → Pfad-Verfolgung

VLAN-PROBLEME:
show vlan brief                  → VLAN-Status
show interfaces trunk            → Trunk-Konfiguration + erlaubte VLANs
show interfaces GiX switchport   → Switchport-Details

SPANNING TREE:
show spanning-tree vlan X        → STP-Status je VLAN
show spanning-tree blockedports  → Blockierte Ports
debug spanning-tree events       → STP-Events (VORSICHT in Produktion!)

PERFORMANCE:
show interfaces | include (rate|error|drop)
show processes cpu sorted        → CPU-Auslastung je Prozess
show memory summary              → Speichernutzung
show platform resources          → Hardware-Ressourcen (NX-OS)

AUTHENTIFIZIERUNG:
debug aaa authentication         → AAA Debug
test aaa group tacacs+ user pass legacy  → TACACS+ Test
show tacacs                      → TACACS+ Status
```
