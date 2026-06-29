# Linux Hardening — Referenzmodul

SSH, sysctl, PAM, Firewall (nftables/ufw), SELinux/AppArmor, Audit, CIS Benchmark.

---

## SSH-Härtung

```bash
# /etc/ssh/sshd_config — Produktionshärtung
cat << 'SSHD' > /etc/ssh/sshd_config.d/99-hardening.conf
# Protokoll & Kryptographie
Protocol 2
HostKeyAlgorithms ssh-ed25519,rsa-sha2-512,rsa-sha2-256
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group16-sha512
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com

# Authentifizierung
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthenticationMethods publickey
MaxAuthTries 3
LoginGraceTime 30
MaxSessions 4
MaxStartups 10:30:60

# Verbindung
ClientAliveInterval 300
ClientAliveCountMax 2
TCPKeepAlive no
Compression no

# Features deaktivieren
X11Forwarding no
AllowTcpForwarding no
AllowAgentForwarding no
PermitTunnel no
GatewayPorts no

# Nur explizit erlaubte Benutzer/Gruppen
AllowGroups sshusers sudo
SSHD
systemctl reload sshd

# SSH-Keys generieren (Ed25519 bevorzugt)
ssh-keygen -t ed25519 -C "admin@$(hostname)-$(date +%Y%m%d)" -f ~/.ssh/id_ed25519
# RSA als Fallback (mind. 4096 Bit)
ssh-keygen -t rsa -b 4096 -C "admin@$(hostname)" -f ~/.ssh/id_rsa
```

---

## sysctl-Härtung (Kernel-Parameter)

```bash
cat << 'SYSCTL' > /etc/sysctl.d/99-hardening.conf
# ─── NETZWERK: IP-Spoofing-Schutz ─────────────────────────────────────────
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# ─── NETZWERK: ICMP-Härtung ───────────────────────────────────────────────
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# ─── NETZWERK: SYN-Flood-Schutz ───────────────────────────────────────────
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_max_syn_backlog = 2048

# ─── NETZWERK: IPv6 deaktivieren (falls nicht benötigt) ───────────────────
# net.ipv6.conf.all.disable_ipv6 = 1
# net.ipv6.conf.default.disable_ipv6 = 1

# ─── KERNEL: Schutzmaßnahmen ──────────────────────────────────────────────
kernel.randomize_va_space = 2          # ASLR vollständig aktiv
kernel.dmesg_restrict = 1             # dmesg nur für root
kernel.kptr_restrict = 2              # Kernel-Pointer verbergen
kernel.sysrq = 0                      # SysRq deaktivieren
kernel.core_uses_pid = 1              # Core-Dumps mit PID
kernel.perf_event_paranoid = 3        # Perf-Events einschränken
fs.suid_dumpable = 0                  # Kein Core-Dump für SUID-Binaries
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
fs.protected_fifos = 2
fs.protected_regular = 2

# ─── SPEICHER ─────────────────────────────────────────────────────────────
vm.swappiness = 10                    # Swap wenig nutzen
vm.mmap_min_addr = 65536              # Null-Pointer-Dereference verhindern
SYSCTL
sysctl --system
```

---

## nftables Firewall (modern, Ersatz für iptables)

```bash
# /etc/nftables.conf — Server-Grundkonfiguration
cat << 'NFT' > /etc/nftables.conf
#!/usr/sbin/nft -f
flush ruleset

define SSH_PORTS    = { 22 }
define WEB_PORTS    = { 80, 443 }
define MGMT_NET     = { 10.10.0.0/24 }   # Management-Netz

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;

        # Bestehende Verbindungen erlauben
        ct state established,related accept
        ct state invalid drop

        # Loopback
        iif lo accept

        # ICMP (limitiert)
        ip  protocol icmp  icmp  type { echo-request, echo-reply, destination-unreachable, time-exceeded } limit rate 10/second accept
        ip6 nexthdr icmpv6 icmpv6 type { echo-request, echo-reply, nd-neighbor-solicit, nd-neighbor-advert } accept

        # SSH: nur aus Management-Netz
        ip saddr $MGMT_NET tcp dport $SSH_PORTS accept

        # Web-Traffic
        tcp dport $WEB_PORTS accept

        # Rate-Limiting: Neue Verbindungen
        tcp flags syn ct state new limit rate 100/second burst 200 packets accept

        # Log + Drop Rest
        limit rate 5/minute log prefix "nft-drop: " flags all
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
    }

    chain output {
        type filter hook output priority 0; policy accept;

        # Optional: Egress einschränken
        # ct state established,related accept
        # ip daddr ... accept
    }
}

table inet nat {
    chain prerouting {
        type nat hook prerouting priority -100;
    }
    chain postrouting {
        type nat hook postrouting priority 100;
        # Masquerade für interne Netze:
        # ip saddr 192.168.0.0/24 oif eth0 masquerade
    }
}
NFT
systemctl enable --now nftables
```

---

## PAM-Härtung (Passwortrichtlinie)

```bash
# Passwort-Qualität mit pam_pwquality (Ubuntu/Debian)
apt-get install -y libpam-pwquality

cat << 'PAM' > /etc/security/pwquality.conf
minlen = 14          # Mindestlänge
dcredit = -1         # Mind. 1 Ziffer
ucredit = -1         # Mind. 1 Großbuchstabe
ocredit = -1         # Mind. 1 Sonderzeichen
lcredit = -1         # Mind. 1 Kleinbuchstabe
maxrepeat = 3        # Max. 3 gleiche Zeichen hintereinander
maxclassrepeat = 4   # Max. 4 Zeichen gleicher Klasse
gecoscheck = 1       # Benutzerinformationen prüfen
dictcheck = 1        # Wörterbuchprüfung
PAM

# Kontosperrung (pam_faillock)
cat << 'FAILLOCK' >> /etc/pam.d/common-auth
auth required pam_faillock.so preauth silent audit deny=5 unlock_time=900
FAILLOCK

# faillock-Konfiguration
cat << 'FL' > /etc/security/faillock.conf
deny = 5
unlock_time = 900
fail_interval = 900
FL

# Passwort-Aging (chage)
chage -M 90 -m 1 -W 14 username   # Max 90 Tage, min 1 Tag, 14 Tage Warnung
```

---

## AppArmor (Ubuntu-Standard)

```bash
# Status prüfen
aa-status

# Profil für eigene Anwendung erstellen
aa-genprof /usr/local/bin/myapp  # Interaktiver Assistent

# Manuelles Profil
cat << 'PROFILE' > /etc/apparmor.d/usr.local.bin.myapp
#include <tunables/global>

/usr/local/bin/myapp {
  #include <abstractions/base>
  #include <abstractions/nameservice>

  /usr/local/bin/myapp  mr,          # Ausführen
  /etc/myapp/**         r,           # Konfiguration lesen
  /var/lib/myapp/**     rw,          # Daten lesen/schreiben
  /var/log/myapp/*.log  w,           # Logs schreiben
  /tmp/myapp-*          rw,          # Temporäre Dateien
  /proc/sys/kernel/hostname r,       # Hostname lesen

  # Netzwerk
  network inet stream,               # TCP IPv4
  network inet6 stream,              # TCP IPv6

  # Deny
  deny /etc/shadow r,
  deny /root/** rw,
}
PROFILE
apparmor_parser -r /etc/apparmor.d/usr.local.bin.myapp
```

---

## SELinux (RHEL/CentOS/AlmaLinux)

```bash
# Status
getenforce           # Enforcing / Permissive / Disabled
sestatus             # Detaillierter Status

# Temporär auf Permissive (für Troubleshooting)
setenforce 0
# Permanent (ACHTUNG: Neustart nötig)
sed -i 's/SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config

# Contexts verwalten
ls -Z /var/www/html/              # SELinux-Context anzeigen
chcon -t httpd_sys_content_t /var/www/html/myfile.html
restorecon -Rv /var/www/html/     # Defaults wiederherstellen

# Boolean-Switches (Zugriffe erlauben/verweigern)
getsebool -a | grep httpd         # Alle HTTP-Booleans anzeigen
setsebool -P httpd_can_network_connect on  # Apache darf Netzwerk nutzen

# AVC-Denials analysieren (wichtigstes SELinux-Tool)
ausearch -m AVC -ts recent        # Aktuelle Denials
audit2why < /var/log/audit/audit.log  # Erklärung der Denials
audit2allow -M mymodule < /var/log/audit/audit.log  # Modul erstellen
semodule -i mymodule.pp           # Modul installieren
```

---

## Auditd-Konfiguration (BSI/CIS-konform)

```bash
# /etc/audit/rules.d/99-hardening.rules
cat << 'AUDIT' > /etc/audit/rules.d/99-hardening.rules
# Audit-Buffer und Fehlerbehandlung
-b 8192
-f 2  # Kernel Panic bei Audit-Fehler (höchste Sicherheit)

# Privilegierte Befehle überwachen
-a always,exit -F arch=b64 -S execve -F euid=0 -k privileged
-a always,exit -F path=/usr/bin/sudo -F perm=x -k sudo_use
-a always,exit -F path=/bin/su -F perm=x -k su_use

# Authentifizierung
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/sudoers -p wa -k sudoers
-w /etc/sudoers.d/ -p wa -k sudoers

# SSH
-w /etc/ssh/sshd_config -p wa -k sshd_config
-w /root/.ssh -p wa -k root_ssh

# Systemstart und Module
-w /sbin/insmod -p x -k modules
-w /sbin/rmmod -p x -k modules
-a always,exit -F arch=b64 -S init_module,delete_module -k modules

# Netzwerk-Konfiguration
-a always,exit -F arch=b64 -S sethostname -S setdomainname -k network_config
-w /etc/hosts -p wa -k network_config
-w /etc/network/ -p wa -k network_config
-w /etc/sysconfig/network -p wa -k network_config

# Cron
-w /etc/cron.allow -p wa -k cron
-w /etc/cron.deny -p wa -k cron
-w /etc/crontab -p wa -k cron
-w /var/spool/cron/ -p wa -k cron
-w /etc/cron.d/ -p wa -k cron

# Dateizugriffe (selektiv — nicht alles loggen!)
-a always,exit -F arch=b64 -S open,openat -F dir=/etc -F success=0 -k config_access_failure

# Konfiguration unänderbar machen (ACHTUNG: Reboot zum Rückgängigmachen)
-e 2
AUDIT
auditctl -R /etc/audit/rules.d/99-hardening.rules
augenrules --load
```

---

## CIS Benchmark Auto-Check (Bash)

```bash
#!/usr/bin/env bash
# Kurzcheck der wichtigsten CIS-Benchmarks
PASS=0; FAIL=0; WARN=0

check() {
    local desc=$1; local cmd=$2; local expected=$3
    local result; result=$(eval "$cmd" 2>/dev/null)
    if echo "$result" | grep -q "$expected"; then
        echo "✅ PASS: $desc"; ((PASS++))
    else
        echo "❌ FAIL: $desc (Ist: ${result:0:60})"; ((FAIL++))
    fi
}

# SSH
check "SSH PermitRootLogin no"     "sshd -T | grep permitrootlogin"       "permitrootlogin no"
check "SSH PasswordAuth no"        "sshd -T | grep passwordauthentication" "passwordauthentication no"
check "SSH Protocol 2"             "sshd -T | grep protocol"               "protocol 2"

# Kernel
check "ASLR aktiv"                 "sysctl kernel.randomize_va_space"       "= 2"
check "SYN Cookies aktiv"          "sysctl net.ipv4.tcp_syncookies"         "= 1"
check "RP Filter aktiv"            "sysctl net.ipv4.conf.all.rp_filter"     "= 1"

# Dateisystem
check "nodev auf /tmp"             "findmnt /tmp"                           "nodev"
check "nosuid auf /tmp"            "findmnt /tmp"                           "nosuid"
check "noexec auf /tmp"            "findmnt /tmp"                           "noexec"

# Services
check "auditd aktiv"              "systemctl is-active auditd"             "active"
check "cron aktiv"                "systemctl is-active cron"               "active"
check "at deaktiviert"            "systemctl is-enabled atd 2>/dev/null"   "disabled\|masked"

echo ""
echo "Ergebnis: ✅ $PASS PASS | ❌ $FAIL FAIL | ⚠️  $WARN WARN"
```
