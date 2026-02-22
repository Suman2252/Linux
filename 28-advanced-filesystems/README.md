# 📀 Chapter 28: Advanced Filesystems (Btrfs & ZFS)

<p align="center">
  <img src="https://img.shields.io/badge/Level-Expert-red?style=for-the-badge" alt="Expert">
  <img src="https://img.shields.io/badge/Chapter-28%20of%2034-blue?style=for-the-badge" alt="Chapter 28">
</p>

---

## 📑 Table of Contents

- [Filesystem Comparison](#filesystem-comparison)
- [Btrfs](#btrfs)
- [ZFS](#zfs)
- [Btrfs vs ZFS](#btrfs-vs-zfs)
- [Practice Exercises](#-practice-exercises)

---

## Filesystem Comparison

| Feature | ext4 | XFS | Btrfs | ZFS |
|---------|------|-----|-------|-----|
| Max file size | 16 TiB | 8 EiB | 16 EiB | 16 EiB |
| Snapshots | ❌ | ❌ | ✅ | ✅ |
| Checksums | ❌ | ❌ | ✅ | ✅ |
| Compression | ❌ | ❌ | ✅ | ✅ |
| RAID built-in | ❌ | ❌ | ✅ | ✅ |
| Copy-on-Write | ❌ | ❌ | ✅ | ✅ |
| Deduplication | ❌ | ❌ | ❌ | ✅ |
| Self-healing | ❌ | ❌ | ✅ | ✅ |

---

## Btrfs

### Creating & Managing

```bash
# Create Btrfs filesystem
sudo mkfs.btrfs /dev/sdb
sudo mkfs.btrfs -L mydata /dev/sdb     # With label

# Mount
sudo mount /dev/sdb /mnt/data

# Multi-device (built-in RAID)
sudo mkfs.btrfs -d raid1 -m raid1 /dev/sdb /dev/sdc
# -d = data profile, -m = metadata profile
# Options: single, raid0, raid1, raid5, raid6, raid10

# Add device
sudo btrfs device add /dev/sdd /mnt/data
sudo btrfs balance start /mnt/data      # Rebalance data

# Remove device
sudo btrfs device remove /dev/sdb /mnt/data
```

### Subvolumes

```bash
# Create subvolumes (like lightweight partitions)
sudo btrfs subvolume create /mnt/data/@home
sudo btrfs subvolume create /mnt/data/@snapshots
sudo btrfs subvolume create /mnt/data/@var

# List subvolumes
sudo btrfs subvolume list /mnt/data

# Mount specific subvolume
sudo mount -o subvol=@home /dev/sdb /home

# Delete subvolume
sudo btrfs subvolume delete /mnt/data/@old
```

### Snapshots

```bash
# Create snapshot (instant!)
sudo btrfs subvolume snapshot /mnt/data/@home /mnt/data/@snapshots/home-$(date +%Y%m%d)

# Read-only snapshot
sudo btrfs subvolume snapshot -r /mnt/data/@home /mnt/data/@snapshots/home-readonly

# Restore from snapshot
sudo btrfs subvolume delete /mnt/data/@home
sudo btrfs subvolume snapshot /mnt/data/@snapshots/home-20240222 /mnt/data/@home
```

### Compression

```bash
# Enable compression
sudo mount -o compress=zstd /dev/sdb /mnt/data
# Options: zlib, lzo, zstd (recommended), zstd:3 (level 1-15)

# Compress existing files
sudo btrfs filesystem defragment -r -czstd /mnt/data/

# /etc/fstab
UUID=xxx /mnt/data btrfs defaults,compress=zstd,noatime 0 0
```

### Maintenance

```bash
# Filesystem info
sudo btrfs filesystem show /mnt/data
sudo btrfs filesystem usage /mnt/data
sudo btrfs filesystem df /mnt/data

# Scrub (verify data integrity)
sudo btrfs scrub start /mnt/data
sudo btrfs scrub status /mnt/data

# Balance (redistribute data)
sudo btrfs balance start /mnt/data

# Defragment
sudo btrfs filesystem defragment -r /mnt/data
```

---

## ZFS

### Installation

```bash
# Ubuntu
sudo apt install zfsutils-linux

# Verify
zfs --version
```

### Pools (zpools)

```bash
# Create pool
sudo zpool create mypool /dev/sdb                    # Single disk
sudo zpool create mypool mirror /dev/sdb /dev/sdc    # Mirror (RAID 1)
sudo zpool create mypool raidz /dev/sdb /dev/sdc /dev/sdd   # RAIDZ (RAID 5)
sudo zpool create mypool raidz2 /dev/sd{b,c,d,e}    # RAIDZ2 (RAID 6)

# Pool status
sudo zpool status
sudo zpool list
sudo zpool iostat -v mypool 1           # I/O stats

# Add devices
sudo zpool add mypool /dev/sdd          # Add disk (stripe)
sudo zpool add mypool cache /dev/sde    # Add L2ARC cache (SSD)
sudo zpool add mypool log /dev/sdf      # Add ZIL (write log)

# Replace failed disk
sudo zpool replace mypool /dev/sdb /dev/sdg

# Scrub (verify integrity)
sudo zpool scrub mypool
```

### Datasets

```bash
# Create datasets (like subvolumes)
sudo zfs create mypool/data
sudo zfs create mypool/home
sudo zfs create mypool/backup

# Set properties
sudo zfs set compression=lz4 mypool/data
sudo zfs set quota=100G mypool/home
sudo zfs set reservation=50G mypool/data
sudo zfs set mountpoint=/home mypool/home
sudo zfs set atime=off mypool/data

# View properties
zfs get all mypool/data
zfs get compression,quota mypool/data

# List datasets
zfs list
```

### Snapshots

```bash
# Create snapshot
sudo zfs snapshot mypool/data@before-upgrade
sudo zfs snapshot -r mypool@daily-$(date +%Y%m%d)   # Recursive

# List snapshots
zfs list -t snapshot

# Rollback
sudo zfs rollback mypool/data@before-upgrade

# Clone a snapshot (writable copy)
sudo zfs clone mypool/data@before-upgrade mypool/data-clone

# Send/Receive (backup/replication)
sudo zfs send mypool/data@snap1 | ssh remote sudo zfs receive backup/data
sudo zfs send -i @snap1 mypool/data@snap2 | ssh remote sudo zfs receive backup/data
```

### Automatic Snapshots

```bash
sudo apt install zfs-auto-snapshot
# Creates automatic snapshots:
# - Every 15 minutes (4 kept)
# - Hourly (24 kept)
# - Daily (31 kept)
# - Weekly (8 kept)
# - Monthly (12 kept)
```

---

## Btrfs vs ZFS

| Feature | Btrfs | ZFS |
|---------|-------|-----|
| **License** | GPL (in-kernel) | CDDL (module) |
| **Maturity** | Maturing | Very mature |
| **Deduplication** | Limited | Full inline |
| **RAID** | 0,1,5,6,10 | mirror, raidz1/2/3 |
| **Memory usage** | Low | High (ARC cache) |
| **Best for** | Desktop, flex storage | Servers, data integrity |

---

## 🏋️ Practice Exercises

1. **(VM)** Create a Btrfs filesystem with two disks in RAID 1
2. **Subvolumes**: Create subvolumes for @home and @data
3. **Snapshot**: Take a Btrfs snapshot, modify files, then restore
4. **Compression**: Enable zstd compression and check space savings
5. **(VM)** Create a ZFS pool with mirrors
6. **ZFS Snapshot**: Create, list, and rollback a ZFS snapshot
7. **Properties**: Set compression and quotas on ZFS datasets
8. **Scrub**: Run a scrub on both Btrfs and ZFS and read the output

---

<p align="center">
  <a href="../27-kubernetes-orchestration/README.md">← Previous: Kubernetes</a> · <a href="../README.md">🏠 Home</a> · <a href="../29-virtualization-kvm/README.md">Next: Virtualization & KVM →</a>
</p>
