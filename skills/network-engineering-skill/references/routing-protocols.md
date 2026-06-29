# Routing-Protokolle — Referenzmodul

BGP, OSPF, EIGRP, statische Routen, Route Redistribution, Policy-Based Routing, VRF.

---

## OSPF (Open Shortest Path First)

```
! ─── OSPF SINGLE-AREA (Area 0) ────────────────────────────────────────────────
router ospf 1
 router-id 10.255.255.1          ! Immer manuell setzen!
 auto-cost reference-bandwidth 100000  ! 100 Gbps Referenz (für moderne Netze)
 passive-interface default        ! Alle Interfaces passiv...
 no passive-interface GigabitEthernet0/1  ! ...außer explizit aktiven
 no passive-interface GigabitEthernet0/2

interface GigabitEthernet0/1
 description LINK-zu-DIST-01
 ip ospf 1 area 0
 ip ospf cost 10
 ip ospf hello-interval 5        ! Schnellere Konvergenz (Standard: 10s)
 ip ospf dead-interval 15        ! 3x Hello (Standard: 40s)
 ip ospf network point-to-point  ! Kein DR/BDR auf P2P-Links
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 OSPF-AUTH-KEY

! ─── MULTI-AREA OSPF ──────────────────────────────────────────────────────────
router ospf 1
 router-id 10.255.255.2
 area 10 stub                    ! Stub-Area: keine External LSAs
 area 20 nssa                    ! NSSA: External LSAs als Type-7
 summary-address 10.10.0.0 255.255.0.0   ! Routen zusammenfassen (ABR)
 default-information originate   ! Default Route in OSPF propagieren

! ─── OSPF TROUBLESHOOTING ─────────────────────────────────────────────────────
! show ip ospf neighbor           → Nachbarn + Status (FULL = OK)
! show ip ospf database           → LSDB
! show ip ospf interface GiX/X   → Interface-Details, Hello/Dead Timer
! show ip route ospf              → OSPF-Routen in Routing-Tabelle
! debug ip ospf adj               → Adjacency-Events (VORSICHT in Produktion!)
```

---

## BGP (Border Gateway Protocol)

```
! ─── EBGP (zu ISP) ────────────────────────────────────────────────────────────
router bgp 65001                  ! AS-Nummer der eigenen Organisation
 bgp router-id 203.0.113.1
 bgp log-neighbor-changes
 no bgp default ipv4-unicast     ! IPv4 explizit aktivieren (Best Practice)

 neighbor 203.0.113.254 remote-as 1234    ! ISP AS
 neighbor 203.0.113.254 description ISP-TELEKOM
 neighbor 203.0.113.254 password BGP-SECRET-KEY
 neighbor 203.0.113.254 ttl-security hops 1    ! GTSM gegen BGP-Hijacking
 neighbor 203.0.113.254 update-source Loopback0

 address-family ipv4 unicast
  neighbor 203.0.113.254 activate
  neighbor 203.0.113.254 prefix-list PL-ISP-IN  in    ! Eingehende Routen filtern
  neighbor 203.0.113.254 prefix-list PL-ISP-OUT out   ! Eigene Prefixe nach außen
  neighbor 203.0.113.254 route-map RM-ISP-IN in
  network 203.0.113.0 mask 255.255.255.0               ! Eigenes Prefix ankündigen
 exit-address-family

! ─── IBGP (zwischen eigenen Routern) ─────────────────────────────────────────
router bgp 65001
 neighbor 10.255.255.2 remote-as 65001    ! Gleiches AS = IBGP
 neighbor 10.255.255.2 update-source Loopback0
 neighbor 10.255.255.2 next-hop-self      ! Next-Hop für IBGP-Peers setzen

! ─── ROUTE REFLECTOR (für große IBGP-Topologien) ─────────────────────────────
! (Vermeidet Full-Mesh IBGP)
router bgp 65001
 neighbor IBGP-CLIENTS peer-group
 neighbor IBGP-CLIENTS remote-as 65001
 neighbor IBGP-CLIENTS route-reflector-client
 neighbor 10.255.255.3 peer-group IBGP-CLIENTS
 neighbor 10.255.255.4 peer-group IBGP-CLIENTS

! ─── PREFIX LISTS (sicherer als ACLs für BGP) ────────────────────────────────
ip prefix-list PL-ISP-OUT seq 10 permit 203.0.113.0/24
ip prefix-list PL-ISP-OUT seq 20 deny 0.0.0.0/0 le 32

! Bogon-Filter (keine privaten/reservierten Prefixe von ISP akzeptieren)
ip prefix-list PL-BOGON-FILTER seq 5  deny 0.0.0.0/8 le 32
ip prefix-list PL-BOGON-FILTER seq 10 deny 10.0.0.0/8 le 32
ip prefix-list PL-BOGON-FILTER seq 15 deny 172.16.0.0/12 le 32
ip prefix-list PL-BOGON-FILTER seq 20 deny 192.168.0.0/16 le 32
ip prefix-list PL-BOGON-FILTER seq 25 deny 100.64.0.0/10 le 32
ip prefix-list PL-BOGON-FILTER seq 30 deny 127.0.0.0/8 le 32
ip prefix-list PL-BOGON-FILTER seq 35 permit 0.0.0.0/0 le 32

! ─── BGP ROUTE MAP (Attribute setzen) ────────────────────────────────────────
route-map RM-ISP-IN permit 10
 match ip address prefix-list PL-BOGON-FILTER
 set local-preference 100        ! Standard Local Preference

route-map RM-ISP-IN deny 65535  ! Implizites Deny am Ende

! ─── BGP TROUBLESHOOTING ──────────────────────────────────────────────────────
! show bgp summary                → Nachbarn + Prefixe
! show bgp neighbors X.X.X.X     → Detaillierter Nachbar-Status
! show bgp ipv4 unicast           → BGP-Tabelle
! show ip route bgp               → BGP-Routen in Routing-Tabelle
! show bgp ipv4 unicast X.X.X.X/Y → Specific Prefix (warum gewählt?)
! clear ip bgp X.X.X.X soft       → Soft-Reset (kein Session-Abbruch)
```

---

## EIGRP

```
! ─── EIGRP (Named Mode, modern) ──────────────────────────────────────────────
router eigrp FIRMA-NET           ! Named Mode statt AS-Nummer empfohlen
 !
 address-family ipv4 unicast autonomous-system 100
  !
  af-interface GigabitEthernet0/1
   authentication mode hmac-sha-256 EIGRP-SECRET
   passive-interface           ! Passiv wenn kein Nachbar erwartet
   no passive-interface        ! Aktiv für Nachbarn
   hello-interval 5
   hold-time 15
  exit-af-interface
  !
  topology base
   redistribute ospf 1 metric 1000000 0 255 1 1500   ! OSPF → EIGRP
   redistribute static                                 ! Statische Routen
  exit-af-topology
  !
  network 10.0.0.0 0.255.255.255  ! Welche Netze annoncieren
  eigrp router-id 10.255.255.1
 exit-address-family

! show eigrp neighbors           → Nachbarn
! show eigrp topology            → Topology-Tabelle (Successor/Feasible Successor)
! show ip route eigrp            → EIGRP-Routen
```

---

## VRF (Virtual Routing and Forwarding)

```
! ─── VRF-DEFINITION ──────────────────────────────────────────────────────────
! Anwendungsfall: Mandantentrennung, Management-VRF, MPLS-VPN

! Management-VRF (Out-of-Band-Management isoliert)
vrf definition MGMT
 description Out-of-Band Management
 rd 65001:1
 !
 address-family ipv4
 exit-address-family

interface GigabitEthernet0/0
 description OOB-MGMT-PORT
 vrf forwarding MGMT
 ip address 10.255.0.1 255.255.255.0

ip route vrf MGMT 0.0.0.0 0.0.0.0 10.255.0.254  ! Default Route im MGMT-VRF

! SSH/SNMP auf MGMT-VRF binden
ip ssh source-interface GigabitEthernet0/0
snmp-server trap-source GigabitEthernet0/0
logging source-interface GigabitEthernet0/0

! Ping/Traceroute im VRF:
! ping vrf MGMT 10.255.0.254
! show ip route vrf MGMT
```

---

## Policy-Based Routing (PBR)

```
! Anwendungsfall: Bestimmter Traffic über anderen ISP leiten
! z.B.: HTTP/HTTPS via ISP1, Management-Traffic via ISP2

ip access-list extended ACL-WEB-TRAFFIC
 permit tcp 10.0.30.0 0.0.0.255 any eq 80
 permit tcp 10.0.30.0 0.0.0.255 any eq 443

route-map RM-PBR permit 10
 match ip address ACL-WEB-TRAFFIC
 set ip next-hop 203.0.113.1     ! Via ISP1

route-map RM-PBR permit 20      ! Rest: normales Routing
 ! (kein set = normales Routing)

interface GigabitEthernet0/2
 description LAN-CLIENTS
 ip policy route-map RM-PBR

! Verifizierung:
! show route-map RM-PBR         → Hit-Counter
! debug ip policy               → Policy-Entscheidungen
```

---

## Route Redistribution

```
! ─── OSPF → BGP ───────────────────────────────────────────────────────────────
router bgp 65001
 address-family ipv4 unicast
  redistribute ospf 1 match internal external 1 external 2
  redistribute connected
  redistribute static

! ─── BGP → OSPF (VORSICHT: Routing-Schleifen möglich!) ──────────────────────
router ospf 1
 redistribute bgp 65001 subnets metric 100 metric-type 2
 distribute-list prefix-list PL-FILTER-REDISTRIBUTE in   ! Immer filtern!

! BEST PRACTICE:
! - Immer Route-Maps/Prefix-Lists bei Redistribution verwenden
! - Routing-Metriken sorgfältig setzen (verhindert Routing-Schleifen)
! - Tag-basiertes Filtering um Routen-Schleifen zu verhindern

route-map RM-OSPF-TO-BGP permit 10
 match ip address prefix-list PL-INTERN-ROUTEN
 set metric 100
 set tag 65001                  ! Tag setzen zum späteren Filtern
