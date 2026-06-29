# Netzwerk-Design

## Spine-Leaf (Datacenter)
```
Spine-01 ── Spine-02
  │  ╲  ╱  │
Leaf-01  Leaf-02  Leaf-03
Eigenschaften: Gleichmäßige Latenz (2 Hops), ECMP, VXLAN/EVPN
```

## Campus Three-Tier
```
Internet ─ Firewall ─ Core-Switch
                          │
               Distribution-Layer
              ╱           │         ╲
           Access-01  Access-02  Access-03
             │            │           │
            PCs          PCs      Drucker/IoT
```

## IP-Adressplanung
```markdown
# IP-Plan [Unternehmen]  Supernetz: 10.0.0.0/8

| Standort | Netz | Zweck |
|---------|------|-------|
| Hauptsitz Management | 10.1.1.0/24 | Netzwerkgeräte |
| Hauptsitz Server | 10.1.10.0/24 | Server |
| Hauptsitz Benutzer | 10.1.20.0/22 | 1022 Hosts |
| Hauptsitz VoIP | 10.1.30.0/24 | Telefonie |
| Filiale-A | 10.2.0.0/24 | Kleinstandort |
| Azure VNet | 10.100.0.0/16 | Cloud |
| VPN-Pool | 10.200.0.0/24 | Remote-Access |
| P2P-Links | 10.255.0.0/24 | /30 je WAN-Link |
```
