# Linux Storage — Referenzmodul

LVM, RAID, NFS, Samba, ZFS, Festplatten-Diagnose, Performance-Tuning.

---

## LVM — Logical Volume Manager

```bash
# ─── ÜBERSICHT ────────────────────────────────────────────────────────────────
pvs                                # Physical Volumes
vgs                                # Volume Groups
lvs                                # Logical Volumes
pvdisplay /dev/sdb                 # Detaillierter PV-Status
vgdisplay vg_data                  # Detaillierter VG-Status
lvdisplay /dev/vg_data/lv_app      # Detaillierter LV-Status

# ─── ERSTELLEN ────────────────────────────────────────────────────────────────
# PV erstellen
pvcreate /dev/sdb /dev/sdc

# VG erstellen (PE-Größe 16MB für Snapshots)
vgcreate -s 16M vg_data /dev/sdb /dev/sdc

# LV erstellen (verschiedene Optionen)
lvcreate -L 50G  -n lv_app    vg_data   # Feste Größe
lvcreate -l 80%VG -n lv_data   vg_data   # 80% der VG
lvcreate -l 100%FREE -n lv_rest vg_data  # Restlichen Platz

# Dateisystem + Mount
mkfs.ext4 -L "app-data" /dev/vg_data/lv_app
mkdir -p /opt/app
echo '/dev/vg_data/lv_app /opt/app ext4 defaults,nofail 0 2' >> /etc/fstab
mount -a

# ─── VERGRÖSSERN (Live, kein Unmount nötig) ───────────────────────────────────
# 1. LV vergrößern
lvextend -L +20G /dev/vg_data/lv_app          # +20GB
lvextend -l +100%FREE /dev/vg_data/lv_app     # Ganzen freien Platz nutzen

# 2. Dateisystem vergrößern (live)
resize2fs /dev/vg_data/lv_app                 # ext4 (online)
xfs_growfs /opt/app                           # XFS (online, nur mit Mountpoint!)

# ─── SNAPSHOT (für konsistente Backups) ───────────────────────────────────────
# Snapshot erstellen (10GB Reserve für Änderungen)
lvcreate -L 10G -s -n lv_app_snap /dev/vg_data/lv_app

# Snapshot mounten (read-only)
mount -o ro /dev/vg_data/lv_app_snap /mnt/snapshot

# Backup vom Snapshot
rsync -avz /mnt/snapshot/ /backup/app/

# Snapshot entfernen
umount /mnt/snapshot
lvremove /dev/vg_data/lv_app_snap

# ─── VG ERWEITERN (neue Disk hinzufügen) ──────────────────────────────────────
pvcreate /dev/sdd
vgextend vg_data /dev/sdd

# ─── PV ENTFERNEN (Disk ersetzen ohne Downtime) ───────────────────────────────
pvmove /dev/sdb                    # Daten auf andere PVs verschieben
vgreduce vg_data /dev/sdb          # PV aus VG entfernen
pvremove /dev/sdb                  # PV-Label löschen
```

---

## Software-RAID (mdadm)

```bash
# RAID-1 erstellen (Spiegelung, 2 Disks)
mdadm --create /dev/md0 \
    --level=1 \
    --raid-devices=2 \
    /dev/sdb /dev/sdc

# RAID-5 erstellen (3 Disks + 1 Hotspare)
mdadm --create /dev/md1 \
    --level=5 \
    --raid-devices=3 \
    --spare-devices=1 \
    /dev/sdb /dev/sdc /dev/sdd /dev/sde

# Konfiguration sichern
mdadm --detail --scan >> /etc/mdadm/mdadm.conf
update-initramfs -u

# Status überwachen
mdadm --detail /dev/md0            # Detaillierter Status
cat /proc/mdstat                   # Schnellübersicht + Rebuild-Progress
watch -n1 cat /proc/mdstat         # Live-Monitor beim Rebuild

# ─── FEHLERFALL ───────────────────────────────────────────────────────────────
# Defekte Disk als fehlerhaft markieren
mdadm --manage /dev/md0 --fail /dev/sdb

# Defekte Disk entfernen
mdadm --manage /dev/md0 --remove /dev/sdb

# Neue Disk hinzufügen (Rebuild startet automatisch)
mdadm --manage /dev/md0 --add /dev/sde

# SMART-Test vor Einbau neuer Disk
smartctl -t short /dev/sde
sleep 120
smartctl -a /dev/sde | grep -A5 "SMART overall"
```

---

## NFS-Server und -Client

```bash
# ─── SERVER ───────────────────────────────────────────────────────────────────
apt-get install -y nfs-kernel-server

# /etc/exports
cat << 'EXPORTS' > /etc/exports
# Syntax: Verzeichnis Client(Optionen)
/data/shared    10.0.1.0/24(rw,sync,no_subtree_check,no_root_squash)
/data/backups   10.0.2.11(ro,sync,no_subtree_check)
/homes          *(rw,sync,no_subtree_check,root_squash)
EXPORTS

exportfs -rav                       # Exports neu laden + anzeigen
exportfs -v                         # Aktuelle Exports mit Optionen
systemctl enable --now nfs-server

# ─── CLIENT ───────────────────────────────────────────────────────────────────
apt-get install -y nfs-common

# Temporärer Mount
mount -t nfs4 -o rsize=1048576,wsize=1048576,hard,timeo=600 \
    nfs-server:/data/shared /mnt/shared

# Permanenter Mount (/etc/fstab)
echo 'nfs-server:/data/shared /mnt/shared nfs4 rw,hard,timeo=600,rsize=1048576,wsize=1048576,nofail,_netdev 0 0' >> /etc/fstab

# ─── TROUBLESHOOTING ──────────────────────────────────────────────────────────
showmount -e nfs-server            # Verfügbare Exports anzeigen
nfsstat -c                         # Client-Statistiken
nfsstat -s                         # Server-Statistiken
rpcinfo -p nfs-server              # RPC-Services
```

---

## ZFS (auf Ubuntu/Debian)

```bash
apt-get install -y zfsutils-linux

# ─── POOL ERSTELLEN ───────────────────────────────────────────────────────────
# Mirror (RAID-1)
zpool create tank mirror /dev/sdb /dev/sdc

# RAIDZ1 (RAID-5 ähnlich)
zpool create tank raidz /dev/sdb /dev/sdc /dev/sdd

# RAIDZ2 (doppelte Parität, 2 Disk-Ausfälle tolerierbar)
zpool create tank raidz2 /dev/sdb /dev/sdc /dev/sdd /dev/sde

# Hotspare hinzufügen
zpool add tank spare /dev/sdf

# ─── DATASETS ─────────────────────────────────────────────────────────────────
zfs create tank/data
zfs create -o compression=lz4 tank/backups
zfs create -o quota=100G tank/homes
zfs create -o mountpoint=/opt/app tank/app

# ─── SNAPSHOTS ────────────────────────────────────────────────────────────────
zfs snapshot tank/data@$(date +%Y%m%d)          # Snapshot erstellen
zfs list -t snapshot -r tank                     # Alle Snapshots
zfs rollback tank/data@20240115                  # Rollback
zfs clone tank/data@20240115 tank/data-restore   # Clone für Vergleich
zfs send tank/data@20240115 | zfs recv backup-pool/data  # Offsite-Replikation

# ─── MONITORING ───────────────────────────────────────────────────────────────
zpool status                               # Pool-Status (ONLINE/DEGRADED/FAULTED)
zpool iostat -v 5                          # I/O-Statistiken
zfs get all tank/data | grep -E "used|avail|compress"
```

---

## Festplatten-Diagnose

```bash
# SMART-Tests
smartctl -t short /dev/sda            # Kurzer Selbsttest starten
smartctl -t long  /dev/sda            # Langer Test (~1-2h für große HDDs)
smartctl -a /dev/sda                  # Vollständige SMART-Daten

# Kritische Werte überwachen:
smartctl -a /dev/sda | grep -E \
    "Reallocated_Sector|Current_Pending_Sector|Offline_Uncorrectable|\
    Reported_Uncorrect|UDMA_CRC_Error|Temperature_Celsius"

# Disk-Benchmark
hdparm -tT /dev/sda                   # Read-Speed (cached + direct)
dd if=/dev/zero of=/tmp/test bs=1M count=1024 oflag=direct  # Write-Speed
fio --name=randread --ioengine=libaio --iodepth=32 \
    --rw=randread --bs=4k --size=1G --numjobs=4 --time_based \
    --runtime=60 --filename=/dev/sda --direct=1  # Zufälliges Lesen

# Filesystem-Prüfung (nur unmounted!)
fsck -f /dev/vg_data/lv_app           # ext4 prüfen
xfs_repair /dev/vg_data/lv_xfs        # XFS reparieren (nur unmounted)
```
