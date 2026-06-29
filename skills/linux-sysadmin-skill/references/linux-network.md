# Linux Netzwerk — Referenzmodul

ip-Befehle, Bonding/VLAN, Troubleshooting, nftables, VPN, DNS.

---

## ip-Befehlsreferenz (iproute2)

```bash
# ─── INTERFACES ───────────────────────────────────────────────────────────────
ip link show                            # Alle Interfaces
ip link set eth0 up/down                # Interface aktivieren/deaktivieren
ip link set eth0 mtu 9000               # Jumbo Frames
ip -s link show eth0                    # Statistiken (Fehler, Drops)

# ─── IP-ADRESSEN ──────────────────────────────────────────────────────────────
ip addr show                            # Alle IP-Adressen
ip addr add 10.0.1.10/24 dev eth0       # IP hinzufügen
ip addr del 10.0.1.10/24 dev eth0       # IP entfernen
ip addr flush dev eth0                  # Alle IPs entfernen

# ─── ROUTING ──────────────────────────────────────────────────────────────────
ip route show                           # Routing-Tabelle
ip route add default via 10.0.1.1       # Default-Route
ip route add 192.168.0.0/16 via 10.0.2.1 dev eth1  # Statische Route
ip route del 192.168.0.0/16             # Route löschen
ip route get 8.8.8.8                    # Route für Ziel-IP

# ─── NEIGHBOURS (ARP) ─────────────────────────────────────────────────────────
ip neigh show                           # ARP-Tabelle
ip neigh flush dev eth0                 # ARP-Cache leeren

# ─── SOCKETS / VERBINDUNGEN ───────────────────────────────────────────────────
ss -tunaple                             # Alle Verbindungen + Prozesse
ss -tlnp                                # Lauschende TCP-Ports + Prozesse
ss -s                                   # Zusammenfassung
ss 'state established dport :443'       # Gefiltert
```

---

## Netzwerk-Konfiguration (Netplan — Ubuntu)

```yaml
# /etc/netplan/01-network.yaml
network:
  version: 2
  renderer: networkd

  ethernets:
    eth0:
      dhcp4: false
      dhcp6: false

    eth1:
      dhcp4: false
      dhcp6: false

  bonds:
    bond0:
      interfaces: [eth0, eth1]
      parameters:
        mode: active-backup        # oder 802.3ad für LACP
        primary: eth0
        mii-monitor-interval: 100
      addresses: [10.0.1.10/24]
      routes:
        - to: default
          via: 10.0.1.1
          metric: 100
      nameservers:
        addresses: [10.0.0.53, 10.0.0.54]
        search: [firma.local]

  vlans:
    vlan100:
      id: 100
      link: bond0
      addresses: [10.100.1.10/24]
    vlan200:
      id: 200
      link: bond0
      addresses: [10.200.1.10/24]
```

```bash
# Netplan anwenden
netplan try            # Testen (30s automatischer Rollback)
netplan apply          # Dauerhaft anwenden
netplan generate       # Konfiguration generieren (ohne anwenden)
```

---

## Troubleshooting-Werkzeuge

```bash
# ─── KONNEKTIVITÄT ────────────────────────────────────────────────────────────
ping -c 4 -W 1 8.8.8.8                 # ICMP-Test
traceroute -n 8.8.8.8                  # Route verfolgen
mtr -n --report 8.8.8.8               # MTR (kombiniert ping + traceroute)
curl -sv --max-time 10 https://api.firma.de  # HTTP-Test mit Details

# ─── DNS ──────────────────────────────────────────────────────────────────────
dig @10.0.0.53 firma.local A           # DNS-Abfrage gegen spezifischen Server
dig +trace www.google.com              # DNS-Auflösung verfolgen
dig -x 10.0.1.10                       # Reverse-DNS
resolvectl status                      # systemd-resolved Status
resolvectl query firma.local           # Via systemd-resolved

# ─── PORT-SCANS / ERREICHBARKEIT ─────────────────────────────────────────────
nc -zv 10.0.1.11 22                    # TCP-Port-Test
nc -zv -u 10.0.1.11 514               # UDP-Port-Test
nmap -sV -p 22,80,443 10.0.1.11       # Port-Scan mit Service-Erkennung
nmap -sn 10.0.1.0/24                  # Ping-Sweep (Host-Discovery)

# ─── PAKETANALYSE ─────────────────────────────────────────────────────────────
tcpdump -i eth0 -n 'host 10.0.1.11 and port 443'
tcpdump -i eth0 -n -w /tmp/capture.pcap 'port not 22'
tcpdump -r /tmp/capture.pcap 'tcp[tcpflags] & tcp-rst != 0'  # RST-Pakete

# ─── BANDBREITE ───────────────────────────────────────────────────────────────
iperf3 -s                              # Server starten
iperf3 -c 10.0.1.11 -P 4 -t 30       # Client-Test (4 Streams, 30 Sek)

# ─── HÄUFIGE PROBLEME ─────────────────────────────────────────────────────────
# "Connection refused" → Service nicht gestartet oder Port geblockt
systemctl status myapp && ss -tlnp | grep :3000

# "Connection timed out" → Firewall oder Route fehlt
traceroute -n 10.0.2.11
nft list ruleset | grep DROP

# "Name resolution failure" → DNS-Problem
systemd-resolve --status
cat /etc/resolv.conf
resolvectl flush-caches
```

---

## WireGuard VPN

```bash
# WireGuard-Keys generieren
wg genkey | tee /etc/wireguard/private.key | wg pubkey > /etc/wireguard/public.key
chmod 600 /etc/wireguard/private.key

# /etc/wireguard/wg0.conf — Server
cat << 'WG' > /etc/wireguard/wg0.conf
[Interface]
Address = 10.99.0.1/24
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY>

# NAT für Internet-Zugang (falls VPN-Gateway)
PostUp   = nft add rule ip nat postrouting oif eth0 masquerade
PostDown = nft delete rule ip nat postrouting oif eth0 masquerade

# Client 1: Admin-Laptop
[Peer]
PublicKey = <CLIENT1_PUBLIC_KEY>
AllowedIPs = 10.99.0.2/32

# Client 2: Remote-Server
[Peer]
PublicKey = <CLIENT2_PUBLIC_KEY>
AllowedIPs = 10.99.0.3/32, 192.168.100.0/24  # Subnet hinter Client
WG

# /etc/wireguard/wg0.conf — Client
cat << 'WGCLIENT' > /etc/wireguard/wg0.conf
[Interface]
Address = 10.99.0.2/24
PrivateKey = <CLIENT_PRIVATE_KEY>
DNS = 10.99.0.1

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = vpn.firma.de:51820
AllowedIPs = 10.0.0.0/8, 172.16.0.0/12    # Split-Tunnel (nur interner Traffic)
PersistentKeepalive = 25
WGCLIENT

systemctl enable --now wg-quick@wg0
wg show         # Status
wg show wg0 transfer  # Traffic-Statistiken
```

---

## Netzwerk-Performance-Tuning

```bash
# TCP-Tuning für hohen Durchsatz
cat << 'TUNE' >> /etc/sysctl.d/99-network-performance.conf
# Buffer-Größen
net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.core.netdev_max_backlog = 5000

# TCP-Algorithmus
net.ipv4.tcp_congestion_control = bbr
net.core.default_qdisc = fq

# Connection-Tracking
net.netfilter.nf_conntrack_max = 1048576
net.netfilter.nf_conntrack_tcp_timeout_established = 86400
TUNE
sysctl --system

# BBR aktivieren prüfen
sysctl net.ipv4.tcp_congestion_control
ss -tin | grep bbr
```
