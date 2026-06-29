# Routing-Protokolle

## BGP
```cisco
router bgp 65001
 bgp router-id 10.0.0.1
 no bgp default ipv4-unicast
 neighbor 203.0.113.2 remote-as 65002
 neighbor 203.0.113.2 description ISP-Upstream
 neighbor 203.0.113.2 password BGP_SECRET
 address-family ipv4
  neighbor 203.0.113.2 activate
  neighbor 203.0.113.2 prefix-list FILTER-IN in
  neighbor 203.0.113.2 prefix-list FILTER-OUT out
  network 192.168.0.0 mask 255.255.0.0
 exit-address-family
ip prefix-list FILTER-OUT seq 10 permit 192.168.0.0/16
show ip bgp summary
show ip bgp neighbors 203.0.113.2
```

## OSPF Multi-Area
```cisco
router ospf 1
 router-id 10.0.0.1
 area 0 authentication message-digest
 area 10 stub
 passive-interface default
 no passive-interface GigabitEthernet0/0
interface GigabitEthernet0/0
 ip ospf 1 area 0
 ip ospf message-digest-key 1 md5 OSPF_SECRET
 ip ospf network point-to-point
show ip ospf neighbor detail
show ip ospf database
```

## Policy-Based Routing
```cisco
ip access-list extended ACL-VOIP
 permit udp 192.168.10.0 0.0.0.255 any range 16384 32767
route-map PBR-VOIP permit 10
 match ip address ACL-VOIP
 set ip next-hop 10.0.0.2
interface GigabitEthernet0/1
 ip policy route-map PBR-VOIP
```
