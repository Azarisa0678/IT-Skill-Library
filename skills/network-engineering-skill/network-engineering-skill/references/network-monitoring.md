# Netzwerk-Monitoring

## SNMP v3 (Cisco)
```cisco
snmp-server group MONITORING v3 priv
snmp-server user snmp-mon MONITORING v3 auth sha AUTHPASS priv aes 256 PRIVPASS
snmp-server host 192.168.1.200 version 3 priv snmp-mon
no snmp-server community public
```

## LibreNMS
```bash
php /opt/librenms/addhost.php 192.168.1.1 v3 \
    --authlevel authPriv --authname snmp-mon \
    --authpass AUTHPASS --cryptopass PRIVPASS
php /opt/librenms/discovery.php -h 192.168.1.1
```

## PRTG Sensoren je Gerät
```
- Ping (Verfügbarkeit)
- SNMP Traffic (Interface-Auslastung)
- SNMP CPU Load
- SNMP Memory
- SNMP Interface Errors
- BGP Neighbor State (Router)
```

## Troubleshooting-Prozess
```
1. PING-Test: ping -c4 ZIEL-IP
2. TRACEROUTE: traceroute ZIEL-IP
3. DNS: dig HOSTNAME @DNS-SERVER
4. Port: nc -zv HOST PORT
5. Capture: tcpdump -i eth0 host ZIEL-IP -nn
6. Interface-Stats:
   - Cisco: show interfaces counters errors
   - Linux: ip -s link show eth0
```
