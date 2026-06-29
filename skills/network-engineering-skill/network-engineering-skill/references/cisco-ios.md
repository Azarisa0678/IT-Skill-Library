# Cisco IOS/IOS-XE
## Grundabsicherung
```cisco
hostname SW-CORE-01
ip domain-name firma.local
crypto key generate rsa modulus 4096
ip ssh version 2
ip ssh authentication-retries 3
line vty 0 15
 transport input ssh
 login local
 exec-timeout 10 0
username admin privilege 15 secret 0 PASSWORT
enable secret 0 ENABLE_SECRET
no ip http server
no cdp run
logging buffered 65536 informational
logging host 192.168.1.100
snmp-server group MONITORING v3 priv
no snmp-server community public
spanning-tree mode rapid-pvst
```
## VLANs & Switching
```cisco
vlan 10
 name BENUTZER
vlan 20
 name SERVER
interface GigabitEthernet1/0/1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable
interface GigabitEthernet1/0/48
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20
ip dhcp snooping
ip dhcp snooping vlan 10,20
ip arp inspection vlan 10,20
```
## OSPF
```cisco
router ospf 1
 router-id 10.0.0.1
 area 0 authentication message-digest
 passive-interface default
 no passive-interface GigabitEthernet0/0
interface GigabitEthernet0/0
 ip ospf 1 area 0
 ip ospf message-digest-key 1 md5 OSPF_SECRET
 ip ospf network point-to-point
```
## Troubleshooting
```cisco
show interfaces status
show vlan brief
show spanning-tree vlan 10
show ip ospf neighbor
show ip route
ping 192.168.1.1 source GigabitEthernet0/0
traceroute 8.8.8.8
```
