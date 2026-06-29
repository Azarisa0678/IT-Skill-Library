# Netzwerk-Design — Referenzmodul

Spine-Leaf, Three-Tier Campus, WAN-Design, SD-WAN, IPv6, QoS, Segmentierung.

---

## Spine-Leaf Architektur (Rechenzentrum)

```
              ┌─────────────┐    ┌─────────────┐
              │  SPINE-01   │    │  SPINE-02   │
              │  (Nexus 9K) │    │  (Nexus 9K) │
              └──┬───────┬──┘    └──┬───────┬──┘
                 │       │          │       │
         ┌───────┘   ┌───┘      ┌───┘   └───────┐
         │           │          │               │
    ┌────┴────┐ ┌────┴────┐ ┌───┴────┐ ┌───────┴─┐
    │ LEAF-01 │ │ LEAF-02 │ │LEAF-03 │ │ LEAF-04 │
    └────┬────┘ └────┬────┘ └────┬───┘ └────┬────┘
         │           │           │           │
    Server-Rack  Server-Rack  Server-Rack  Border-Leaf
                                           (→ WAN/Firewall)

EIGENSCHAFTEN:
- Jedes Leaf mit jedem Spine verbunden (Full-Mesh)
- Keine Leaf-zu-Leaf-Direktverbindungen
- Gleiche Hop-Anzahl von jedem Server zu jedem Server
- Horizontale Skalierung durch Hinzufügen von Leaf-Switches
- BGP EVPN/VXLAN für Layer-2-Overlay über Layer-3-Underlay
```

### VXLAN BGP EVPN (NX-OS)

```
! Underlay: OSPF für Loopback-Erreichbarkeit
feature ospf
feature bgp
feature vn-segment-vlan-based
feature nv overlay

! VTEP-Interface
interface nve1
 no shutdown
 source-interface loopback1
 host-reachability protocol bgp

! BGP EVPN Control Plane
router bgp 65001
 neighbor 10.255.255.1 remote-as 65001
 neighbor 10.255.255.1 update-source loopback0
 address-family l2vpn evpn
  neighbor 10.255.255.1 activate
  neighbor 10.255.255.1 send-community extended

! VNI-zu-VLAN-Mapping
vlan 10
 vn-segment 10010    ! VXLAN Network Identifier

evpn
 vni 10010 l2
  rd auto
  route-target import auto
  route-target export auto
```

---

## Three-Tier Campus (klassisch)

```
                    ┌─────────────────────────┐
                    │       CORE LAYER        │
                    │   (Catalyst 9500/9600)  │
                    │   L3, Routing, HSRP     │
                    └───────────┬─────────────┘
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
    ┌─────────┴───────┐ ┌───────┴───────┐ ┌──────┴────────┐
    │ DISTRIBUTION-01 │ │DISTRIBUTION-02│ │DISTRIBUTION-03│
    │  (Cat 9300/9400)│ │               │ │               │
    │  L3, SVI, ACLs  │ │               │ │               │
    └─────────┬───────┘ └───────┬───────┘ └──────┬────────┘
              │                 │                 │
    ┌─────────┴──┐         ┌────┴────┐       ┌───┴──────┐
    │ ACCESS-01  │         │ACCESS-02│       │ ACCESS-03│
    │(Cat 9200)  │         │         │       │          │
    │L2, PortFast│         │         │       │          │
    └────────────┘         └─────────┘       └──────────┘

IP-DESIGN:
  Core:          10.0.0.0/24       (P2P Links /30 oder /31)
  Distribution:  Loopbacks /32
  SVIs/VLANs:   10.x.0.0/24 je Gebäude/Etage
  MGMT:          10.255.0.0/24
```

### HSRP (High Availability Gateway)

```
! Distribution-01 (HSRP Active für VLAN 30)
interface Vlan30
 description CLIENTS-VLAN30
 ip address 10.30.0.2 255.255.255.0
 standby version 2
 standby 30 ip 10.30.0.1          ! Virtual IP (Default Gateway der Clients)
 standby 30 priority 110           ! Höhere Priorität = Active
 standby 30 preempt delay minimum 30
 standby 30 authentication md5 key-string HSRP-SECRET
 standby 30 track 1 decrement 30  ! Priority sinkt wenn Uplink weg

! Distribution-02 (HSRP Standby für VLAN 30)
interface Vlan30
 ip address 10.30.0.3 255.255.255.0
 standby version 2
 standby 30 ip 10.30.0.1
 standby 30 priority 100           ! Niedrigere Priorität = Standby
 standby 30 preempt delay minimum 30
 standby 30 authentication md5 key-string HSRP-SECRET
```

---

## QoS (Quality of Service)

```
! ─── DSCP MARKING (am Access-Switch) ──────────────────────────────────────────
! Traffic klassifizieren und markieren (edge: trust nothing from endpoints)

class-map match-all CM-VOIP-RTP
 match ip dscp ef                   ! EF = Expedited Forwarding (VoIP)

class-map match-all CM-SIGNALING
 match ip dscp cs3 af31             ! CS3/AF31 = VoIP-Signalisierung

class-map match-all CM-CRITICAL-DATA
 match ip dscp af21 af22 af23      ! AF2x = wichtige Daten

class-map match-all CM-SCAVENGER
 match ip dscp cs1                  ! CS1 = Best-Effort/Scavenger

! Policy auf Uplink-Interface (Queuing)
policy-map PM-QOS-UPLINK
 class CM-VOIP-RTP
  priority percent 20               ! LLQ: Garantierte Bandbreite
 class CM-SIGNALING
  bandwidth percent 5
 class CM-CRITICAL-DATA
  bandwidth percent 40
  fair-queue
 class class-default
  bandwidth percent 35
  fair-queue

interface GigabitEthernet1/1
 description UPLINK-zu-DIST-01
 service-policy output PM-QOS-UPLINK

! DSCP Trust am Access-Port für IP-Phones
interface GigabitEthernet1/0/10
 mls qos trust dscp               ! Trust DSCP vom IP-Phone
```

---

## IPv6 Dual-Stack

```
! ─── IPv6 GRUNDKONFIGURATION ─────────────────────────────────────────────────
ipv6 unicast-routing
ipv6 cef

interface GigabitEthernet0/1
 description WAN-LINK
 ipv6 address 2001:db8:1::/64 eui-64    ! EUI-64 (aus MAC generiert)
 ipv6 address 2001:db8:1::1/64           ! Manuell (bevorzugt)
 ipv6 nd ra-interval 30
 ipv6 nd ra-lifetime 90
 ipv6 ospf 1 area 0

! OSPFv3 für IPv6
ipv6 router ospf 1
 router-id 10.255.255.1
 area 0 range 2001:db8::/32

! BGP für IPv6
router bgp 65001
 neighbor 2001:db8::1 remote-as 1234
 address-family ipv6 unicast
  neighbor 2001:db8::1 activate
  network 2001:db8:firma::/48

! ─── IPv6 SECURITY ────────────────────────────────────────────────────────────
! RA Guard (verhindert Rogue Router Advertisements)
ipv6 nd raguard policy CLIENTS
 device-role host

interface GigabitEthernet1/0/10
 ipv6 nd raguard attach-policy CLIENTS

! DHCPv6 Snooping
ipv6 dhcp guard policy DHCP-CLIENTS
 device-role client

! IPv6 Source Guard
ipv6 source-guard attach-policy default
```

---

## SD-WAN Design (Überblick)

```
KLASSISCHES WAN:                    SD-WAN:
  HQ ────MPLS──── Branch             HQ ────Controller──── Branch
                                          Internet (IPsec)
                                          LTE/5G (Backup)
                                          MPLS (Premium)

VORTEILE SD-WAN:
  ✓ Application-Aware Routing
  ✓ Zero-Touch Provisioning (ZTP)
  ✓ Zentrales Policy-Management
  ✓ WAN-Optimierung
  ✓ Kostenreduktion vs. reinem MPLS

LÖSUNGEN DACH-MARKT:
  Cisco Viptela (SD-WAN)
  Fortinet Secure SD-WAN
  VMware SD-WAN (VeloCloud)
  Palo Alto Prisma SD-WAN
  Meraki SD-WAN

DESIGN-PRINZIPIEN:
  1. Underlay: Internet + LTE als primäre Links, MPLS optional
  2. Overlay: IPsec-Tunnel zwischen allen Standorten (Hub-Spoke oder Full-Mesh)
  3. Policy: Kritische Apps (SAP, Teams) via MPLS; Best-Effort via Internet
  4. Security: NGFW am Edge (lokal oder Cloud-basiert SSE/SASE)
  5. Monitoring: Zentrales Dashboard mit Application Performance Metrics
```
