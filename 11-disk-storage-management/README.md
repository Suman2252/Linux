# 💾 Chapter 11: Disk & Storage Management

<p align="center">
  <img src="https://img.shields.io/badge/Level-Intermediate-yellow?style=for-the-badge" alt="Intermediate">
  <img src="https://img.shields.io/badge/Chapter-11%20of%2034-blue?style=for-the-badge" alt="Chapter 11">
</p>

---

## 📑 Table of Contents

- [Storage Concepts](#storage-concepts)
- [Listing Disks & Partitions](#listing-disks--partitions)
- [Partitioning Disks](#partitioning-disks)
- [Filesystems](#filesystems)
- [Mounting & Unmounting](#mounting--unmounting)
- [The /etc/fstab File](#the-etcfstab-file)
- [Disk Usage & Space](#disk-usage--space)
- [Swap Space](#swap-space)
- [Practice Exercises](#-practice-exercises)

---

## Storage Concepts

```
┌──────────────────────────────────────┐
│          Physical Disk               │
│  (e.g., /dev/sda — 500 GB)          │
├──────────┬──────────┬────────────────┤
│ /dev/sda1│ /dev/sda2│ /dev/sda3      │
│  (boot)  │  (root)  │ (home)         │
│  512 MB  │  50 GB   │ 449.5 GB       │
│  ext4    │  ext4    │ ext4           │
│  /boot   │  /       │ /home          │
└──────────┴──────────┴────────────────┘
```

### Naming Convention

| Device | Path | Description |
|--------|------|-------------|
| First SATA/SCSI disk | `/dev/sda` | Entire disk |
| First partition | `/dev/sda1` | First partition on sda |
| Second partition | `/dev/sda2` | Second partition on sda |
| NVMe SSD | `/dev/nvme0n1` | First NVMe drive |
| NVMe partition | `/dev/nvme0n1p1` | First partition on NVMe |
| Virtual disk | `/dev/vda` | Virtio disk (VMs) |

---

## Listing Disks & Partitions

```bash
# List block devices (most readable)
lsblk
# NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
# sda      8:0    0   500G  0 disk
# ├─sda1   8:1    0   512M  0 part /boot
# ├─sda2   8:2    0    50G  0 part /
# └─sda3   8:3    0 449.5G  0 part /home

lsblk -f                        # Show filesystem types
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT  # Custom columns

# fdisk list (detailed)
sudo fdisk -l                    # All disks and partitions
sudo fdisk -l /dev/sda           # Specific disk

# Partition information
sudo parted -l                   # List all disk layouts
cat /proc/partitions             # Kernel's view of partitions

# Get UUIDs
sudo blkid                       # Block device IDs
ls -l /dev/disk/by-uuid/         # UUID symlinks
```

---

## Partitioning Disks

> ⚠️ **Warning**: Partitioning can destroy data. Always back up first and practice on VMs!

### Using `fdisk` (MBR)

```bash
sudo fdisk /dev/sdb              # Open fdisk for disk sdb

# fdisk commands (interactive):
# m → Help menu
# p → Print partition table
# n → New partition
# d → Delete partition
# t → Change partition type
# w → Write changes and exit
# q → Quit without saving
```

### Using `parted` (GPT — recommended for modern systems)

```bash
sudo parted /dev/sdb

# Interactive commands:
# print          → Show partition table
# mklabel gpt    → Create GPT partition table
# mkpart primary ext4 1MiB 100GiB  → Create partition
# rm 1           → Delete partition 1
# quit           → Exit

# Non-interactive examples:
sudo parted /dev/sdb mklabel gpt
sudo parted /dev/sdb mkpart primary ext4 1MiB 50GiB
sudo parted /dev/sdb mkpart primary ext4 50GiB 100%
```

### MBR vs GPT

| Feature | MBR | GPT |
|---------|-----|-----|
| Max disk size | 2 TB | 9.4 ZB |
| Max partitions | 4 primary (or 3 + extended) | 128 |
| Boot mode | BIOS (Legacy) | UEFI |
| Redundancy | No backup | Backup header at end of disk |
| Recommended | Old systems | Modern systems ✅ |

---

## Filesystems

### Creating Filesystems

```bash
# ext4 (most common for Linux)
sudo mkfs.ext4 /dev/sdb1

# XFS (RHEL default, great for large files)
sudo mkfs.xfs /dev/sdb1

# Btrfs (modern, CoW, snapshots)
sudo mkfs.btrfs /dev/sdb1

# FAT32 (USB drives, cross-platform)
sudo mkfs.vfat -F 32 /dev/sdb1

# exFAT (large files, cross-platform)
sudo mkfs.exfat /dev/sdb1
```

### Filesystem Comparison

| Filesystem | Max File | Max Volume | Journaling | Use Case |
|-----------|----------|-----------|------------|----------|
| **ext4** | 16 TB | 1 EB | ✅ | General purpose |
| **XFS** | 8 EB | 8 EB | ✅ | Large files, servers |
| **Btrfs** | 16 EB | 16 EB | ✅ CoW | Snapshots, NAS |
| **ZFS** | 16 EB | 256 ZB | ✅ CoW | Enterprise storage |
| **FAT32** | 4 GB | 2 TB | ❌ | USB, compatibility |
| **exFAT** | 128 PB | 128 PB | ❌ | Large USB/SD |

### Filesystem Check & Repair

```bash
# Check filesystem (MUST unmount first!)
sudo umount /dev/sdb1
sudo fsck /dev/sdb1              # Automatic detection
sudo fsck.ext4 /dev/sdb1         # Specific to ext4
sudo e2fsck -f /dev/sdb1         # Force check

# XFS check (can be done mounted, read-only)
sudo xfs_repair /dev/sdb1
```

---

## Mounting & Unmounting

### Mounting

```bash
# Create mount point
sudo mkdir -p /mnt/mydisk

# Mount a partition
sudo mount /dev/sdb1 /mnt/mydisk

# Mount with specific filesystem type
sudo mount -t ext4 /dev/sdb1 /mnt/mydisk

# Mount read-only
sudo mount -o ro /dev/sdb1 /mnt/mydisk

# Mount with options
sudo mount -o rw,noexec,nosuid /dev/sdb1 /mnt/mydisk

# Mount by UUID (preferred — doesn't change if disk order changes)
sudo mount UUID="xxxx-xxxx-xxxx" /mnt/mydisk

# Mount an ISO file
sudo mount -o loop image.iso /mnt/iso

# Mount a USB drive (usually auto-detected)
sudo mount /dev/sdb1 /mnt/usb

# Mount NFS share
sudo mount -t nfs server:/share /mnt/nfs

# Remount with different options (without unmounting)
sudo mount -o remount,rw /mnt/mydisk
```

### Unmounting

```bash
sudo umount /mnt/mydisk          # By mount point
sudo umount /dev/sdb1            # By device

# Force unmount (if busy)
sudo umount -f /mnt/mydisk

# Lazy unmount (detach now, cleanup when not busy)
sudo umount -l /mnt/mydisk

# Find what's using the mount
lsof +D /mnt/mydisk
fuser -vm /mnt/mydisk
```

### View Current Mounts

```bash
mount                            # All current mounts
mount | grep sdb                 # Filter for specific disk
findmnt                          # Tree view of mounts
findmnt -t ext4                  # Only ext4 mounts
df -hT                           # Mounts with filesystem type
```

---

## The /etc/fstab File

`/etc/fstab` defines filesystems to mount automatically at boot.

```bash
cat /etc/fstab
```

### Format

```
# <device>                              <mount>   <type>  <options>       <dump> <pass>
UUID=xxxx-xxxx-xxxx-xxxx               /          ext4    errors=remount-ro  0      1
UUID=yyyy-yyyy-yyyy-yyyy               /home      ext4    defaults           0      2
UUID=zzzz-zzzz-zzzz-zzzz              /boot      ext4    defaults           0      2
UUID=aaaa-aaaa-aaaa-aaaa              none       swap    sw                 0      0
tmpfs                                  /tmp       tmpfs   defaults,noatime   0      0
```

| Field | Description | Common Values |
|-------|-------------|---------------|
| **device** | UUID, LABEL, or device path | `UUID=...`, `/dev/sda1` |
| **mount** | Mount point | `/home`, `/mnt/data` |
| **type** | Filesystem type | `ext4`, `xfs`, `swap`, `nfs` |
| **options** | Mount options | `defaults`, `noatime`, `ro` |
| **dump** | Backup flag | `0` (no), `1` (yes) |
| **pass** | fsck order | `0` (skip), `1` (root), `2` (other) |

### Common Mount Options

| Option | Description |
|--------|------------|
| `defaults` | rw, suid, dev, exec, auto, nouser, async |
| `noatime` | Don't update access times (performance boost) |
| `noexec` | Prevent execution of binaries |
| `nosuid` | Ignore SUID/SGID bits |
| `ro` | Read-only |
| `rw` | Read-write |
| `auto` | Mount at boot |
| `noauto` | Don't mount at boot |
| `user` | Allow normal users to mount |
| `nofail` | Don't halt boot if device missing |

```bash
# Test fstab without rebooting
sudo mount -a                    # Mount everything in fstab

# Always test fstab before rebooting!
sudo findmnt --verify            # Verify fstab syntax
```

> 🚨 **A broken `/etc/fstab` can prevent your system from booting.** Always test with `mount -a` before rebooting.

---

## Disk Usage & Space

```bash
# Disk free space
df -h                            # Human-readable
df -hT                           # Include filesystem type
df -h /home                      # Specific mount point
df -i                            # Inode usage

# Directory sizes
du -sh /var/log                  # Total size of directory
du -sh /var/*                    # Size of each item in /var
du -sh * | sort -rh | head -10   # Top 10 largest items
du -ah . | sort -rh | head -20   # Top 20 largest files (recursive)

# ncdu — interactive disk analyzer
sudo apt install ncdu
ncdu /                           # Interactive usage viewer
```

---

## Swap Space

Swap is disk space used when RAM is full — acts as "overflow" memory.

### Check Current Swap

```bash
swapon --show                    # Active swap spaces
free -h                          # Memory + swap overview
cat /proc/swaps                  # Kernel's swap info
```

### Create Swap File

```bash
# Create a 4 GB swap file
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent — add to fstab
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Verify
swapon --show
free -h
```

### Adjust Swappiness

```bash
# Check current swappiness (0-100, lower = use swap less)
cat /proc/sys/vm/swappiness      # Default: 60

# Temporary change
sudo sysctl vm.swappiness=10

# Permanent change
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
```

---

## 🏋️ Practice Exercises

1. **List** all block devices with `lsblk -f`
2. **Check** disk usage of `/var` and find the largest subdirectory
3. **UUID**: Find the UUID of your root partition with `sudo blkid`
4. **fstab**: Read your `/etc/fstab` and identify each entry
5. **Swap**: Check your current swap space and swappiness value
6. **Mount**: Mount a USB drive or ISO file to `/mnt/test`
7. **ncdu**: Install `ncdu` and find the largest directory on your system
8. **(VM only)**: Create a new partition, format it as ext4, and mount it

---

<p align="center">
  <a href="../10-process-management/README.md">← Previous: Process Management</a> · <a href="../README.md">🏠 Home</a> · <a href="../12-networking-fundamentals/README.md">Next: Networking Fundamentals →</a>
</p>
