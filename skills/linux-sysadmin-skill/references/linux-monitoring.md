# Linux Monitoring — Referenzmodul

Prometheus Node Exporter, Zabbix, Nagios/Icinga, Performance-Analyse, Log-Monitoring.

---

## Performance-Analyse-Werkzeuge

```bash
# ─── CPU ──────────────────────────────────────────────────────────────────────
top -b -n1 -o %CPU | head -20          # Top-Prozesse nach CPU (einmalig)
mpstat -P ALL 1 5                       # CPU-Auslastung je Core (5 Messungen)
pidstat -u 1 10                         # CPU je Prozess
sar -u 1 10                             # System-CPU-Statistik (sysstat)
perf top                                # Live Performance-Profiling (Kernel)

# ─── SPEICHER ─────────────────────────────────────────────────────────────────
free -h                                 # RAM + Swap übersicht
vmstat -SM 1 10                        # Virtual Memory Stats
cat /proc/meminfo                       # Detaillierter Speicherstatus
smem -rs pss | head -20                # Realer Speicher je Prozess (ohne Sharing)
pmap -x $(pgrep myapp) | tail -5       # Memory-Map eines Prozesses

# ─── I/O ──────────────────────────────────────────────────────────────────────
iostat -xz 1 5                         # Disk-I/O je Gerät (await, util%)
iotop -a -o                            # I/O je Prozess live
pidstat -d 1 10                         # Disk-I/O je Prozess
lsof +D /var/lib                       # Geöffnete Dateien in Verzeichnis

# ─── NETZWERK ─────────────────────────────────────────────────────────────────
ss -tunaple                            # Alle Verbindungen (Ersatz für netstat)
ss -s                                  # Verbindungsstatistiken
nethogs eth0                           # Netzwerk-Traffic je Prozess
iftop -i eth0                          # Live-Bandbreite je Verbindung
nload eth0                             # Bandwidth-Meter
tcpdump -i eth0 -n 'port 80' -w /tmp/capture.pcap  # Paket-Mitschnitt

# ─── SYSTEM LOAD ──────────────────────────────────────────────────────────────
uptime                                 # Load Average
w                                      # Eingeloggte User + Load
sar -q 1 10                            # Load-History

# ─── FESTPLATTE ───────────────────────────────────────────────────────────────
df -hT                                 # Dateisysteme und Typen
du -sh /var/* | sort -rh | head -20   # Größte Verzeichnisse
lsblk -f                               # Block-Geräte + Dateisystem
nvme smart-log /dev/nvme0             # NVMe-Status (nvme-cli)
smartctl -a /dev/sda                   # SMART-Status HDD/SSD

# ─── PROZESSE ─────────────────────────────────────────────────────────────────
ps auxf                                # Prozessbaum
pstree -p                              # Kompakter Prozessbaum
lsof -p $(pgrep myapp)                 # Offene Dateien/Sockets eines Prozesses
strace -p $(pgrep myapp) -e trace=network  # Syscalls mitschneiden
```

---

## Prometheus Node Exporter — Setup

```bash
# Installation (systemd)
VERSION="1.7.0"
wget "https://github.com/prometheus/node_exporter/releases/download/v${VERSION}/node_exporter-${VERSION}.linux-amd64.tar.gz"
tar xzf node_exporter-*.tar.gz
install -m 755 node_exporter-*/node_exporter /usr/local/bin/

useradd --system --no-create-home --shell /bin/false node_exporter

cat << 'UNIT' > /etc/systemd/system/node_exporter.service
[Unit]
Description=Prometheus Node Exporter
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter \
    --collector.systemd \
    --collector.processes \
    --collector.interrupts \
    --collector.tcpstat \
    --web.listen-address=:9100
Restart=on-failure
RestartSec=5s
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=yes

[Install]
WantedBy=multi-user.target
UNIT
systemctl daemon-reload && systemctl enable --now node_exporter
```

### Wichtige Node-Exporter-Metriken (PromQL)

```promql
# CPU-Auslastung (%)
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# RAM-Auslastung (%)
100 * (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)

# Disk-Auslastung (%)
100 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"} * 100)

# Disk-I/O (MB/s)
rate(node_disk_read_bytes_total{device="sda"}[5m]) / 1024 / 1024

# Netzwerk-Traffic (MB/s)
rate(node_network_receive_bytes_total{device="eth0"}[5m]) / 1024 / 1024

# Load Average
node_load1 / on(instance) count by(instance) (node_cpu_seconds_total{mode="idle"})

# Offene File-Descriptors (% des Limits)
node_filefd_allocated / node_filefd_maximum * 100
```

### Alert-Regeln Node-Level

```yaml
groups:
  - name: linux-nodes
    rules:
      - alert: NodeHighCPU
        expr: 100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
        for: 10m
        labels: { severity: warning }
        annotations:
          summary: "CPU hoch: {{ $labels.instance }} ({{ $value | humanize }}%)"

      - alert: NodeMemoryLow
        expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 < 10
        for: 5m
        labels: { severity: critical }
        annotations:
          summary: "RAM kritisch: {{ $labels.instance }} ({{ $value | humanize }}% frei)"

      - alert: NodeDiskFilling
        expr: |
          (node_filesystem_avail_bytes{fstype!~"tmpfs|fuse.lxcfs"})
          / node_filesystem_size_bytes * 100 < 15
        for: 10m
        labels: { severity: warning }
        annotations:
          summary: "Disk füllt sich: {{ $labels.instance }}:{{ $labels.mountpoint }}"

      - alert: NodeDiskIOWait
        expr: avg by(instance)(rate(node_cpu_seconds_total{mode="iowait"}[5m])) * 100 > 20
        for: 10m
        labels: { severity: warning }
        annotations:
          summary: "I/O-Wait hoch: {{ $labels.instance }} ({{ $value | humanize }}%)"

      - alert: NodeSystemdUnitFailed
        expr: node_systemd_unit_state{state="failed"} == 1
        for: 1m
        labels: { severity: critical }
        annotations:
          summary: "systemd Unit fehlgeschlagen: {{ $labels.instance }} / {{ $labels.name }}"
```

---

## Log-Monitoring mit Loki + Promtail

```yaml
# /etc/promtail/config.yml
server:
  http_listen_port: 9080

positions:
  filename: /var/lib/promtail/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system-logs
    static_configs:
      - targets: [localhost]
        labels:
          job: syslog
          host: "{{ HOSTNAME }}"
          __path__: /var/log/syslog

  - job_name: application-logs
    static_configs:
      - targets: [localhost]
        labels:
          job: myapp
          host: "{{ HOSTNAME }}"
          __path__: /var/log/myapp/*.log
    pipeline_stages:
      - json:
          expressions:
            level: level
            msg: message
            ts: timestamp
      - labels:
          level:
      - timestamp:
          source: ts
          format: RFC3339
```

---

## Monitoring-Skript (All-in-One Check)

```bash
#!/usr/bin/env bash
# linux-health-check.sh — Nagios/Icinga-kompatibler Exit-Code
set -euo pipefail

WARN_CPU=80; CRIT_CPU=95
WARN_MEM=85; CRIT_MEM=95
WARN_DISK=80; CRIT_DISK=90
WARN_LOAD=4; CRIT_LOAD=8

STATUS=0; OUTPUT=(); PERF=()

# CPU
cpu=$(top -bn1 | grep "Cpu(s)" | awk '{print int($2)}')
PERF+=("cpu=${cpu}%;${WARN_CPU};${CRIT_CPU}")
if   [[ $cpu -ge $CRIT_CPU ]]; then OUTPUT+=("CRIT CPU:${cpu}%"); STATUS=2
elif [[ $cpu -ge $WARN_CPU ]]; then OUTPUT+=("WARN CPU:${cpu}%"); [[ $STATUS -lt 1 ]] && STATUS=1
fi

# Speicher
mem_free=$(awk '/MemAvailable/{print $2}' /proc/meminfo)
mem_total=$(awk '/MemTotal/{print $2}' /proc/meminfo)
mem_pct=$(( (mem_total - mem_free) * 100 / mem_total ))
PERF+=("memory=${mem_pct}%;${WARN_MEM};${CRIT_MEM}")
if   [[ $mem_pct -ge $CRIT_MEM ]]; then OUTPUT+=("CRIT MEM:${mem_pct}%"); STATUS=2
elif [[ $mem_pct -ge $WARN_MEM ]]; then OUTPUT+=("WARN MEM:${mem_pct}%"); [[ $STATUS -lt 1 ]] && STATUS=1
fi

# Disk (alle Mounts > 1GB)
while read -r pct mp; do
    pct=${pct%\%}
    PERF+=("disk_${mp//\//_}=${pct}%;${WARN_DISK};${CRIT_DISK}")
    if   [[ $pct -ge $CRIT_DISK ]]; then OUTPUT+=("CRIT DISK $mp:${pct}%"); STATUS=2
    elif [[ $pct -ge $WARN_DISK ]]; then OUTPUT+=("WARN DISK $mp:${pct}%"); [[ $STATUS -lt 1 ]] && STATUS=1
    fi
done < <(df -h --output=pcent,target | awk 'NR>1 && $1~/[0-9]/ {gsub(/%/,"",$1); print $1, $2}')

# Load
load=$(awk '{print int($1)}' /proc/loadavg)
PERF+=("load1=${load};${WARN_LOAD};${CRIT_LOAD}")
if   [[ $load -ge $CRIT_LOAD ]]; then OUTPUT+=("CRIT LOAD:${load}"); STATUS=2
elif [[ $load -ge $WARN_LOAD ]]; then OUTPUT+=("WARN LOAD:${load}"); [[ $STATUS -lt 1 ]] && STATUS=1
fi

# Ausgabe (Nagios-Format)
label=("OK" "WARNING" "CRITICAL" "UNKNOWN")
echo "${label[$STATUS]}: ${OUTPUT[*]:-Alles OK} | $(IFS=' '; echo "${PERF[*]}")"
exit $STATUS
```
