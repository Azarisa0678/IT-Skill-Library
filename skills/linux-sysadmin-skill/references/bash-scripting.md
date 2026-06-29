# Bash Scripting — Referenzmodul

Produktionsreife Skripte, Fehlerbehandlung, Logging, Argument-Parsing, Best Practices.

---

## Skript-Template (Produktionsreif)

```bash
#!/usr/bin/env bash
# Beschreibung: [Was macht dieses Skript?]
# Autor:        [Name]
# Version:      1.0.0
# Abhängigkeiten: curl, jq, systemctl
set -euo pipefail
IFS=$'\n\t'

# ─── KONSTANTEN ───────────────────────────────────────────────────────────────
readonly SCRIPT_NAME="$(basename "$0")"
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly LOG_FILE="/var/log/${SCRIPT_NAME%.sh}.log"
readonly LOCK_FILE="/var/run/${SCRIPT_NAME%.sh}.pid"
readonly TIMESTAMP=$(date '+%Y%m%d_%H%M%S')

# ─── FARBEN ───────────────────────────────────────────────────────────────────
RED='\033[0;31m'; YELLOW='\033[1;33m'; GREEN='\033[0;32m'
BLUE='\033[0;34m'; NC='\033[0m'
[[ ! -t 1 ]] && RED='' YELLOW='' GREEN='' BLUE='' NC=''  # Kein Farbe wenn kein TTY

# ─── LOGGING ──────────────────────────────────────────────────────────────────
log()  { echo -e "${GREEN}[$(date '+%H:%M:%S')] INFO${NC}  $*" | tee -a "$LOG_FILE"; }
warn() { echo -e "${YELLOW}[$(date '+%H:%M:%S')] WARN${NC}  $*" | tee -a "$LOG_FILE" >&2; }
err()  { echo -e "${RED}[$(date '+%H:%M:%S')] ERROR${NC} $*" | tee -a "$LOG_FILE" >&2; }
die()  { err "$*"; exit 1; }

# ─── CLEANUP ──────────────────────────────────────────────────────────────────
cleanup() {
    local exit_code=$?
    rm -f "$LOCK_FILE"
    [[ $exit_code -ne 0 ]] && err "Skript mit Exit-Code $exit_code beendet"
    exit $exit_code
}
trap cleanup EXIT
trap 'die "Signal empfangen, beende..."' INT TERM

# ─── LOCK ─────────────────────────────────────────────────────────────────────
acquire_lock() {
    if [[ -f "$LOCK_FILE" ]]; then
        local pid; pid=$(cat "$LOCK_FILE")
        kill -0 "$pid" 2>/dev/null && die "Skript läuft bereits (PID: $pid)"
        warn "Veraltete Lock-Datei gefunden, wird entfernt"
    fi
    echo $$ > "$LOCK_FILE"
}

# ─── ABHÄNGIGKEITEN PRÜFEN ────────────────────────────────────────────────────
check_dependencies() {
    local deps=("$@")
    local missing=()
    for dep in "${deps[@]}"; do
        command -v "$dep" &>/dev/null || missing+=("$dep")
    done
    [[ ${#missing[@]} -gt 0 ]] && die "Fehlende Tools: ${missing[*]}"
}

# ─── ARGUMENT-PARSING ─────────────────────────────────────────────────────────
usage() {
    cat << USAGE
Verwendung: $SCRIPT_NAME [OPTIONEN] <argument>

Optionen:
  -e, --env ENV       Umgebung (dev|staging|prod) [Standard: dev]
  -n, --dry-run       Nur simulieren, keine Änderungen
  -v, --verbose       Ausführliche Ausgabe
  -h, --help          Diese Hilfe anzeigen

Beispiel:
  $SCRIPT_NAME --env prod --verbose deploy
USAGE
}

parse_args() {
    ENV="dev"; DRY_RUN=false; VERBOSE=false; POSITIONAL=()
    while [[ $# -gt 0 ]]; do
        case $1 in
            -e|--env)      ENV="$2"; shift 2 ;;
            -n|--dry-run)  DRY_RUN=true; shift ;;
            -v|--verbose)  VERBOSE=true; shift ;;
            -h|--help)     usage; exit 0 ;;
            --)            shift; POSITIONAL+=("$@"); break ;;
            -*)            die "Unbekannte Option: $1" ;;
            *)             POSITIONAL+=("$1"); shift ;;
        esac
    done
    [[ ${#POSITIONAL[@]} -eq 0 ]] && { usage; die "Kein Argument angegeben"; }
    [[ "$ENV" =~ ^(dev|staging|prod)$ ]] || die "Ungültige Umgebung: $ENV"
}

# ─── HILFSFUNKTIONEN ──────────────────────────────────────────────────────────
run_cmd() {
    # Führt Befehl aus, respektiert dry-run
    if [[ "$DRY_RUN" == true ]]; then
        log "[DRY-RUN] $*"
    else
        "$@"
    fi
}

retry() {
    # Befehl mit automatischen Wiederholungsversuchen
    local attempts=${1}; local delay=${2}; shift 2
    local count=0
    until "$@"; do
        count=$((count + 1))
        [[ $count -ge $attempts ]] && die "Fehlgeschlagen nach $attempts Versuchen: $*"
        warn "Versuch $count/$attempts fehlgeschlagen, warte ${delay}s..."
        sleep "$delay"
    done
}

# ─── HAUPTPROGRAMM ────────────────────────────────────────────────────────────
main() {
    parse_args "$@"
    check_dependencies curl jq systemctl
    acquire_lock

    log "Starte $SCRIPT_NAME (Umgebung: $ENV)"
    [[ "$VERBOSE" == true ]] && set -x

    # Beispiel: Service-Status prüfen
    local service="${POSITIONAL[0]}"
    if systemctl is-active --quiet "$service"; then
        log "Service $service läuft"
    else
        warn "Service $service nicht aktiv"
        run_cmd systemctl start "$service"
    fi

    log "Abgeschlossen"
}

main "$@"
```

---

## Nützliche Bash-Muster

```bash
# ─── ARRAYS ───────────────────────────────────────────────────────────────────
declare -a servers=("web01" "web02" "db01")
declare -A config=([host]="localhost" [port]="5432" [db]="prod")

for server in "${servers[@]}"; do
    echo "Verarbeite: $server"
done

# ─── STRING-OPERATIONEN ───────────────────────────────────────────────────────
filename="/var/log/app/error.log"
echo "${filename##*/}"          # error.log (Basename)
echo "${filename%/*}"           # /var/log/app (Dirname)
echo "${filename%.log}.bak"     # /var/log/app/error.bak
echo "${filename^^}"            # Grossbuchstaben
echo "${#filename}"             # Länge: 22

# ─── PROZESSSUBSTITUTION ──────────────────────────────────────────────────────
# Vergleich zweier Befehls-Ausgaben
diff <(ls /etc/nginx/sites-enabled/) <(ls /etc/nginx/sites-available/)

# ─── HERE-DOKUMENT ────────────────────────────────────────────────────────────
cat << 'SQL' | psql -U admin production
  SELECT count(*) FROM users WHERE created_at > NOW() - INTERVAL '24 hours';
SQL

# ─── PARALLELVERARBEITUNG ─────────────────────────────────────────────────────
process_server() {
    local host=$1
    ssh "$host" "uptime && df -h / && free -m" 2>&1 | sed "s/^/[$host] /"
}
export -f process_server
printf '%s\n' "${servers[@]}" | xargs -P4 -I{} bash -c 'process_server "$@"' _ {}

# ─── JSON MIT JQ ──────────────────────────────────────────────────────────────
# API-Response parsen
response=$(curl -sf "https://api.example.com/services" \
    -H "Authorization: Bearer $TOKEN")
mapfile -t service_names < <(echo "$response" | jq -r '.services[].name')
total=$(echo "$response" | jq '.total')
echo "Gefunden: $total Services"

# ─── DATUMSKALKULATION ────────────────────────────────────────────────────────
gestern=$(date -d "yesterday" '+%Y-%m-%d')
vor7tagen=$(date -d "7 days ago" '+%Y-%m-%d')
naechsterMontag=$(date -d "next monday" '+%Y-%m-%d')

# ─── TEMPORÄRE DATEIEN/VERZEICHNISSE ─────────────────────────────────────────
tmpdir=$(mktemp -d)
tmpfile=$(mktemp /tmp/myapp.XXXXXX)
trap 'rm -rf "$tmpdir" "$tmpfile"' EXIT

# ─── SEMAPHORE / RATE-LIMITING ────────────────────────────────────────────────
# Maximal 3 parallele Jobs
max_jobs=3
job_count=0
for item in "${items[@]}"; do
    process_item "$item" &
    job_count=$((job_count + 1))
    [[ $job_count -ge $max_jobs ]] && { wait -n; job_count=$((job_count - 1)); }
done
wait  # Auf alle verbleibenden Jobs warten
```

---

## System-Health-Check-Skript

```bash
#!/usr/bin/env bash
set -euo pipefail

# Schwellenwerte
CPU_WARN=80; CPU_CRIT=95
MEM_WARN=85; MEM_CRIT=95
DISK_WARN=80; DISK_CRIT=90
LOAD_WARN=2.0

HOSTNAME=$(hostname -f)
ISSUES=(); CRITS=()

check_cpu() {
    local cpu
    cpu=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d. -f1)
    if   [[ $cpu -ge $CPU_CRIT ]]; then CRITS+=("CPU: ${cpu}%")
    elif [[ $cpu -ge $CPU_WARN ]]; then ISSUES+=("CPU: ${cpu}%")
    fi
}

check_memory() {
    local used total pct
    read -r total used <<< "$(free -m | awk '/^Mem:/{print $2, $3}')"
    pct=$(( used * 100 / total ))
    if   [[ $pct -ge $MEM_CRIT ]]; then CRITS+=("RAM: ${pct}%")
    elif [[ $pct -ge $MEM_WARN ]]; then ISSUES+=("RAM: ${pct}%")
    fi
}

check_disk() {
    while IFS= read -r line; do
        local pct mp
        pct=$(echo "$line" | awk '{print $5}' | tr -d '%')
        mp=$(echo "$line" | awk '{print $6}')
        if   [[ $pct -ge $DISK_CRIT ]]; then CRITS+=("Disk $mp: ${pct}%")
        elif [[ $pct -ge $DISK_WARN ]]; then ISSUES+=("Disk $mp: ${pct}%")
        fi
    done < <(df -h | awk 'NR>1 && $6 !~ /^(\/dev|\/sys|\/proc|\/run)/ {print}')
}

check_load() {
    local load1
    load1=$(awk '{print $1}' /proc/loadavg)
    if (( $(echo "$load1 > $LOAD_WARN" | bc -l) )); then
        ISSUES+=("Load: $load1")
    fi
}

check_services() {
    local services=("nginx" "postgresql" "redis" "ssh")
    for svc in "${services[@]}"; do
        systemctl is-active --quiet "$svc" || CRITS+=("Service DOWN: $svc")
    done
}

main() {
    check_cpu; check_memory; check_disk; check_load; check_services

    if [[ ${#CRITS[@]} -gt 0 ]]; then
        echo "CRITICAL [$HOSTNAME]: ${CRITS[*]} | ${ISSUES[*]}"
        exit 2
    elif [[ ${#ISSUES[@]} -gt 0 ]]; then
        echo "WARNING [$HOSTNAME]: ${ISSUES[*]}"
        exit 1
    else
        echo "OK [$HOSTNAME]: Alle Werte im Normalbereich"
        exit 0
    fi
}
main
```
