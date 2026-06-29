# Firewall-Konfiguration — Referenzmodul

Palo Alto PAN-OS, Fortinet FortiGate, Cisco ASA/FTD, Zone-Based Firewall, NAT.

---

## Palo Alto PAN-OS

### Security Policy (Best Practice)

```
# Palo Alto: Zone-basierte Security Policy
# Zonen: UNTRUST (Internet), TRUST (LAN), DMZ, MGMT

# ─── INTRAZONE DEFAULT DENY ────────────────────────────────────────────────────
# (In PAN-OS: Intrazone ist standardmäßig ALLOW — explizit setzen!)
set rulebase security rules INTRAZONE-DENY-ALL from any
set rulebase security rules INTRAZONE-DENY-ALL to any
set rulebase security rules INTRAZONE-DENY-ALL source any
set rulebase security rules INTRAZONE-DENY-ALL destination any
set rulebase security rules INTRAZONE-DENY-ALL application any
set rulebase security rules INTRAZONE-DENY-ALL action deny
set rulebase security rules INTRAZONE-DENY-ALL log-setting SIEM-Log-Profile

# ─── BEISPIEL SECURITY RULES ──────────────────────────────────────────────────
# Regel 1: LAN → Internet (HTTP/HTTPS/DNS)
set rulebase security rules LAN-TO-INET from TRUST
set rulebase security rules LAN-TO-INET to UNTRUST
set rulebase security rules LAN-TO-INET source 10.0.0.0/8
set rulebase security rules LAN-TO-INET destination any
set rulebase security rules LAN-TO-INET application [ ssl web-browsing dns ]
set rulebase security rules LAN-TO-INET service application-default
set rulebase security rules LAN-TO-INET action allow
set rulebase security rules LAN-TO-INET profile-setting group BEST-PRACTICE-PROFILE
set rulebase security rules LAN-TO-INET log-setting SIEM-Log-Profile

# Regel 2: DMZ Webserver → Internet (Update-Traffic)
set rulebase security rules DMZ-UPDATES from DMZ
set rulebase security rules DMZ-UPDATES to UNTRUST
set rulebase security rules DMZ-UPDATES source DMZ-WEBSERVER-GRP
set rulebase security rules DMZ-UPDATES destination any
set rulebase security rules DMZ-UPDATES application [ ssl web-browsing ]
set rulebase security rules DMZ-UPDATES action allow

# Regel 3: Internet → DMZ Webserver (eingehend)
set rulebase security rules INET-TO-WEBSERVER from UNTRUST
set rulebase security rules INET-TO-WEBSERVER to DMZ
set rulebase security rules INET-TO-WEBSERVER source any
set rulebase security rules INET-TO-WEBSERVER destination DMZ-WEBSERVER-VIP
set rulebase security rules INET-TO-WEBSERVER application [ ssl http ]
set rulebase security rules INET-TO-WEBSERVER action allow
set rulebase security rules INET-TO-WEBSERVER profile-setting group STRICT-PROFILE
```

### PAN-OS NAT

```
# Source NAT (LAN → Internet via Firewall-IP)
set rulebase nat rules LAN-SNAT from TRUST
set rulebase nat rules LAN-SNAT to UNTRUST
set rulebase nat rules LAN-SNAT source 10.0.0.0/8
set rulebase nat rules LAN-SNAT destination any
set rulebase nat rules LAN-SNAT source-translation dynamic-ip-and-port interface-address interface ethernet1/1

# Destination NAT (Port-Forwarding: Internet → DMZ Webserver)
set rulebase nat rules DNAT-WEBSERVER from UNTRUST
set rulebase nat rules DNAT-WEBSERVER to UNTRUST
set rulebase nat rules DNAT-WEBSERVER destination 203.0.113.10    ! Externe IP
set rulebase nat rules DNAT-WEBSERVER service tcp-80-443
set rulebase nat rules DNAT-WEBSERVER destination-translation translated-address 10.50.0.10  ! DMZ-IP
```

### PAN-OS Troubleshooting (CLI)

```bash
# Packet-Capture (live)
debug dataplane packet-diag set filter match source 10.0.30.5 destination 8.8.8.8
debug dataplane packet-diag set capture stage firewall file /tmp/capture.pcap
debug dataplane packet-diag set capture on
# ... Traffic generieren ...
debug dataplane packet-diag set capture off
scp export filter-pcap from /tmp/capture.pcap to admin@10.0.0.100:/tmp/

# Security Policy testen (Test-Command)
test security-policy-match from TRUST to UNTRUST source 10.0.30.5 \
    destination 8.8.8.8 destination-port 443 protocol 6

# Session-Tabelle
show session all filter source 10.0.30.5
show session id 12345

# Routing
show routing route type unicast
test routing fib-lookup virtual-router default ip 8.8.8.8

# Counter / Statistiken
show counter global filter delta yes severity drop
```

---

## FortiGate (FortiOS)

### CLI-Konfiguration

```
# ─── FIREWALL POLICY ─────────────────────────────────────────────────────────
config firewall policy
    edit 10
        set name "LAN-to-WAN"
        set srcintf "internal"
        set dstintf "wan1"
        set srcaddr "LAN-NET"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS"
        set ssl-ssh-profile "deep-inspection"
        set av-profile "AV-Profile-Default"
        set ips-sensor "IPS-Sensor-Default"
        set application-list "AppCtrl-Default"
        set logtraffic all
        set nat enable
    next
    edit 20
        set name "WAN-to-DMZ-HTTPS"
        set srcintf "wan1"
        set dstintf "dmz"
        set srcaddr "all"
        set dstaddr "Webserver-VIP"
        set action accept
        set schedule "always"
        set service "HTTPS"
        set ssl-ssh-profile "deep-inspection"
        set ips-sensor "IPS-Sensor-Strict"
        set logtraffic all
    next
end

# ─── VIP (Virtual IP = Destination NAT) ─────────────────────────────────────
config firewall vip
    edit "Webserver-VIP"
        set extip 203.0.113.10
        set mappedip 10.50.0.10
        set extintf "wan1"
        set portforward enable
        set extport 443
        set mappedport 443
    next
end

# ─── SD-WAN ──────────────────────────────────────────────────────────────────
config system sdwan
    set status enable
    config zone
        edit "virtual-wan-link"
        next
    end
    config members
        edit 1
            set interface "wan1"
            set gateway 203.0.113.1
            set priority 1
        next
        edit 2
            set interface "wan2"
            set gateway 198.51.100.1
            set priority 10
        next
    end
    config health-check
        edit "Google-DNS"
            set server "8.8.8.8"
            set members 1 2
        next
    end
    config service
        edit 1
            set name "CRITICAL-TRAFFIC"
            set src "all"
            set dst "all"
            set priority-members 1    ! Bevorzugt WAN1
        next
    end
end
```

### FortiGate Diagnose-Befehle

```bash
# Firewall Policy Match testen
diagnose firewall iprope lookup 10.0.30.5 8.8.8.8 6 1024 443 internal

# Session-Tabelle
diagnose sys session filter src 10.0.30.5
diagnose sys session list

# Routing
get router info routing-table all
diagnose ip route lookup 8.8.8.8

# Echtzeit-Logs (Sniffer)
diagnose sniffer packet any 'host 10.0.30.5 and port 443' 4 100 l

# IPS Events
diagnose ips anomaly list
get ips status
```

---

## Cisco Zone-Based Firewall (ZBF) — IOS

```
! Zonen definieren
zone security INSIDE
zone security DMZ
zone security OUTSIDE

! Zonen-Paare (Richtung der Policy)
zone-pair security INSIDE-TO-OUTSIDE source INSIDE destination OUTSIDE
zone-pair security OUTSIDE-TO-DMZ source OUTSIDE destination DMZ
zone-pair security INSIDE-TO-DMZ source INSIDE destination DMZ

! Class-Maps (Traffic-Klassifizierung)
class-map type inspect match-any CM-WEB
 match protocol http
 match protocol https
 match protocol dns

! Policy-Maps
policy-map type inspect PM-INSIDE-TO-OUTSIDE
 class type inspect CM-WEB
  inspect
 class class-default
  drop log

! Interfaces Zonen zuweisen
interface GigabitEthernet0/0
 description LAN
 zone-member security INSIDE

interface GigabitEthernet0/1
 description WAN
 zone-member security OUTSIDE

! Policy auf Zone-Pair anwenden
zone-pair security INSIDE-TO-OUTSIDE
 service-policy type inspect PM-INSIDE-TO-OUTSIDE
```

---

## Firewall-Audit-Checkliste

```markdown
## Firewall-Audit — Checkliste

### Regelwerk
- [ ] Default Deny (letztes Deny-All) vorhanden
- [ ] Keine "permit any any"-Regeln ohne Begründung
- [ ] Ungenutzte Regeln identifiziert und entfernt (Hit-Counter = 0)
- [ ] Regelwerk dokumentiert (Zweck jeder Regel)
- [ ] Regeln nach Least-Privilege-Prinzip
- [ ] Shadow-Rules (niemals erreichte Regeln) entfernt

### Logging
- [ ] Alle DENY-Events geloggt
- [ ] Logs an SIEM weitergeleitet
- [ ] Log-Aufbewahrung ≥ 90 Tage (BSI: 6 Monate empfohlen)
- [ ] Alert bei kritischen Events (Port-Scans, Brute Force)

### Management
- [ ] Management-Zugang nur aus dediziertem MGMT-VLAN
- [ ] SSH/HTTPS (kein Telnet/HTTP)
- [ ] MFA für Admin-Zugang
- [ ] Firmware aktuell (keine bekannten CVEs)
- [ ] Konfiguration regelmäßig gesichert (täglich)

### NAT & Services
- [ ] Kein unnötiger Port-Forward von außen nach innen
- [ ] VIPs/NAT-Einträge dokumentiert und regelmäßig überprüft
- [ ] SNMP v3 (kein v1/v2c)
- [ ] NTP synchronisiert (korrekte Zeitstempel für Logs)
```
