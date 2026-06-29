# Firewall-Konfiguration

## Palo Alto – Zonen-Design
```
UNTRUST (Internet) → DMZ → TRUST (LAN: BENUTZER/SERVER/MGMT)
Default-Deny mit Logging auf alle Zonen
```
```bash
# Security Policy (CLI)
set rulebase security rules "Benutzer-Web" from TRUST to UNTRUST \
    source "Benutzer-Netz" destination any \
    application ["web-browsing","ssl","dns"] service application-default \
    action allow profile-setting profiles virus "Default" spyware "Default"
set rulebase security rules "Default-Deny" from any to any \
    source any destination any application any service any \
    action deny log-start yes log-end yes
commit
```

## FortiGate – IPsec VPN
```bash
config vpn ipsec phase1-interface
    edit "VPN-B"
        set interface "wan1"
        set ike-version 2
        set proposal aes256-sha256
        set dhgrp 14
        set remote-gw 203.0.113.50
        set psksecret "SICHERES_PSK"
    next
end
config vpn ipsec phase2-interface
    edit "VPN-B-P2"
        set phase1name "VPN-B"
        set proposal aes256-sha256
        set dhgrp 14
        set src-subnet 192.168.1.0 255.255.255.0
        set dst-subnet 192.168.2.0 255.255.255.0
    next
end
```

## Regelwerk-Dokumentation
```markdown
| Nr | Name | Von | Nach | Quelle | Ziel | Dienst | Aktion | Log |
|----|------|-----|------|--------|------|--------|--------|-----|
| 1 | Benutzer-Web | TRUST | UNTRUST | 192.168.10.0/24 | any | HTTPS | Allow | Ja |
| 99 | Default-Deny | any | any | any | any | any | Deny | Ja |
```

## Troubleshooting
```bash
# Palo Alto
test security-policy-match protocol 6 from TRUST to UNTRUST \
    source 192.168.1.100 destination 8.8.8.8 destination-port 443
# FortiGate
diagnose debug flow filter addr 192.168.1.100
diagnose debug flow trace start 100
diagnose debug enable
# Nach Test:
diagnose debug disable
```
