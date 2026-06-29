# Eval-Testfälle: Network Engineering Skill

## Kategorie 1: Trigger-Tests

### T001 — VLAN-Setup Cisco
**Input:** "Konfiguriere VLANs auf einem Cisco Switch für Benutzer, Server und Management"
**Kriterien:**
- ✅ VLANs mit sinnvollen Namen
- ✅ Access-Ports mit portfast und bpduguard
- ✅ Trunk-Port mit native VLAN 99
- ✅ DHCP Snooping aktiviert
- ✅ Dynamic ARP Inspection
- ✅ Management-Interface konfiguriert

### T002 — IPsec VPN FortiGate
**Input:** "Konfiguriere einen Site-to-Site IPsec VPN zwischen zwei FortiGate Firewalls"
**Kriterien:**
- ✅ Phase 1 (IKEv2, AES256-SHA256, DH14)
- ✅ Phase 2 mit korrekten Subnetzen
- ✅ Statische Route zum Remote-Netz
- ✅ Firewall Policy für VPN-Traffic
- ✅ Kein schwacher Cipher (kein DES/3DES)
- ✅ PSK oder Zertifikat-Empfehlung

### T003 — OSPF-Konfiguration
**Input:** "Konfiguriere OSPF zwischen drei Cisco Routern in Area 0"
**Kriterien:**
- ✅ router-id explizit gesetzt
- ✅ area 0 authentication message-digest
- ✅ passive-interface default + no passive für WAN
- ✅ auto-cost reference-bandwidth 10000 (10Gbps)
- ✅ point-to-point für P2P-Links
- ✅ show ip ospf neighbor zur Verifikation

### T004 — BGP mit Filterung
**Input:** "Konfiguriere eBGP zu unserem ISP mit Prefix-Listen zur Filterung"
**Kriterien:**
- ✅ AS-Nummer korrekt verwendet
- ✅ Prefix-Listen für IN und OUT
- ✅ Route-Map für Local Preference
- ✅ no bgp default ipv4-unicast
- ✅ password für MD5-Authentifizierung
- ✅ show ip bgp summary zur Verifikation

### T005 — Firewall Regelwerk (Palo Alto)
**Input:** "Erstelle ein Sicherheitskonzept für die Palo Alto Firewall-Regeln"
**Kriterien:**
- ✅ Zonen-Design beschrieben (UNTRUST/DMZ/TRUST)
- ✅ Default-Deny mit Logging am Ende
- ✅ App-ID statt Port-basiert empfohlen
- ✅ Security Profiles auf Allow-Regeln
- ✅ Regelwerk-Dokumentationstabelle
- ✅ Regelmäßige Review empfohlen

### T006 — Netzwerk-Design Spine-Leaf
**Input:** "Entwirf eine Spine-Leaf-Architektur für unser neues Rechenzentrum"
**Kriterien:**
- ✅ Spine-Leaf-Topologie erklärt
- ✅ Vorteile vs. Three-Tier genannt
- ✅ ECMP erwähnt
- ✅ VXLAN/EVPN für L2-Overlay
- ✅ Überabonnierungsverhältnis adressiert
- ✅ Skalierungslogik beschrieben

### T007 — Netzwerk-Troubleshooting
**Input:** "Ein Server kann keine Verbindung zum Internet aufbauen. Wie gehe ich vor?"
**Kriterien:**
- ✅ Systematischer Layer-by-Layer-Ansatz (1→7)
- ✅ Ping-Test lokal, Gateway, DNS, Internet
- ✅ Routing-Tabelle prüfen
- ✅ Firewall-Regeln prüfen
- ✅ DNS-Auflösung testen
- ✅ Konkrete Befehle für jeden Schritt

### T008 — SNMP v3 konfigurieren
**Input:** "Konfiguriere SNMP v3 auf einem Cisco Switch für unser Monitoring-System"
**Kriterien:**
- ✅ SNMPv3 (nicht v1/v2c!)
- ✅ snmp-server group mit priv-Level
- ✅ snmp-server user mit auth sha und priv aes
- ✅ Community-Strings public/private entfernt
- ✅ snmp-server host konfiguriert
- ✅ Verifizierungsbefehl

## Kategorie 2: Qualitäts-Tests

### Q001 — Sicherheit by Default
**Jede Netzwerkkonfiguration:**
- ✅ Kein SNMPv1/v2c empfohlen
- ✅ SSH statt Telnet
- ✅ Starke Cipher-Suites (kein DES/3DES)
- ✅ Management-Zugriff eingeschränkt

### Q002 — Dokumentation
**Jede Konfigurationsempfehlung:**
- ✅ Verifikationsbefehle am Ende
- ✅ Rollback-Hinweis bei kritischen Änderungen
- ✅ Change Request erwähnt

## Kategorie 3: Negativ-Tests

### N001 — Linux-Frage
**Input:** "Wie installiere ich nginx auf Ubuntu?"
**Erwartetes Verhalten:**
- ❌ Network Skill nicht zuständig
- ✅ Linux Sysadmin Skill empfehlen

### N002 — Security Pentest
**Input:** "Führe einen Netzwerk-Pentest durch"
**Erwartetes Verhalten:**
- ✅ Basis-Netzwerkinfos OK
- ✅ IT Security Skill für Pentest-Details empfehlen

## Bewertungsschema
**Mindest-Score:** 4/5 auf alle T-Tests
