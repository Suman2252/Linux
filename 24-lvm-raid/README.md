# 💽 Chapter 24: LVM & RAID

<p align="center">
  <img src="https://img.shields.io/badge/Level-Advanced-orange?style=for-the-badge" alt="Advanced">
  <img src="https://img.shields.io/badge/Chapter-24%20of%2034-blue?style=for-the-badge" alt="Chapter 24">
</p>

---

## 📑 Table of Contents

- [LVM (Logical Volume Manager)](#lvm-logical-volume-manager)
- [LVM Snapshots](#lvm-snapshots)
- [LVM Thin Provisioning](#lvm-thin-provisioning)
- [RAID with mdadm](#raid-with-mdadm)
- [RAID Levels Compared](#raid-levels-compared)
- [LVM on RAID](#lvm-on-raid)
- [Practice Exercises](#-practice-exercises)

---

## LVM (Logical Volume Manager)

LVM adds a flexible abstraction layer between physical disks and filesystems.

```
Physical Disks        PV (Physical Volume)        VG (Volume Group)        LV (Logical Volume)
┌──────┐             ┌──────┐                    ┌──────────────┐         ┌──────┐
│ sda  │────────────▶│ PV 1 │──────┐             │              │────────▶│ root │ → /
└──────┘             └──────┘      ├────────────▶│   vg_data    │         └──────┘
┌──────┐             ┌──────┐      │             │              │────────▶┌──────┐
│ sdb  │────────────▶│ PV 2 │──────┘             │              │         │ home │ → /home
└──────┘             └──────┘                    └──────────────┘         └──────┘
```

### Creating LVM

```bash
# Step 1: Create Physical Volumes
sudo pvcreate /dev/sdb /dev/sdc
sudo pvs                                 # List PVs
sudo pvdisplay                           # Detailed PV info

# Step 2: Create Volume Group
sudo vgcreate vg_data /dev/sdb /dev/sdc
sudo vgs                                 # List VGs
sudo vgdisplay                           # Detailed VG info

# Step 3: Create Logical Volumes
sudo lvcreate -L 50G -n lv_root vg_data
sudo lvcreate -L 100G -n lv_home vg_data
sudo lvcreate -l 100%FREE -n lv_data vg_data  # Use all remaining space
sudo lvs                                 # List LVs
sudo lvdisplay                           # Detailed LV info

# Step 4: Create filesystem and mount
sudo mkfs.ext4 /dev/vg_data/lv_root
sudo mkdir /mnt/mydata
sudo mount /dev/vg_data/lv_root /mnt/mydata
```

### Extending (Growing)

```bash
# Extend VG with new disk
sudo pvcreate /dev/sdd
sudo vgextend vg_data /dev/sdd

# Extend LV
sudo lvextend -L +50G /dev/vg_data/lv_home
sudo lvextend -l +100%FREE /dev/vg_data/lv_home  # All free space

# Resize filesystem (after extending LV)
sudo resize2fs /dev/vg_data/lv_home      # ext4
sudo xfs_growfs /mnt/home                 # XFS (specify mount point)

# One command (extend + resize)
sudo lvextend -r -L +50G /dev/vg_data/lv_home
```

### Shrinking (Dangerous!)

```bash
# Shrink ext4 (MUST unmount first! XFS cannot be shrunk!)
sudo umount /mnt/home
sudo e2fsck -f /dev/vg_data/lv_home
sudo resize2fs /dev/vg_data/lv_home 80G  # Shrink filesystem first!
sudo lvreduce -L 80G /dev/vg_data/lv_home  # Then shrink LV
sudo mount /dev/vg_data/lv_home /mnt/home
```

> ⚠️ **Always shrink the filesystem BEFORE the LV.** Doing it in wrong order = data loss.

### Removing LVM

```bash
sudo umount /mnt/mydata
sudo lvremove /dev/vg_data/lv_data
sudo vgremove vg_data
sudo pvremove /dev/sdb /dev/sdc
```

---

## LVM Snapshots

Instant point-in-time copies.

```bash
# Create snapshot (needs free space in VG)
sudo lvcreate -s -n snap_root -L 10G /dev/vg_data/lv_root

# Mount snapshot (read-only)
sudo mount -o ro /dev/vg_data/snap_root /mnt/snapshot

# Restore from snapshot
sudo lvconvert --merge /dev/vg_data/snap_root
sudo reboot  # If restoring root

# Remove snapshot
sudo lvremove /dev/vg_data/snap_root
```

---

## LVM Thin Provisioning

Allocate more space than physically available (overcommit).

```bash
# Create thin pool
sudo lvcreate --type thin-pool -L 100G -n thinpool vg_data

# Create thin volumes (can exceed pool size!)
sudo lvcreate --type thin -V 50G -n thin_vol1 vg_data/thinpool
sudo lvcreate --type thin -V 50G -n thin_vol2 vg_data/thinpool
sudo lvcreate --type thin -V 50G -n thin_vol3 vg_data/thinpool
# Total 150G allocated, but pool is only 100G!

# Monitor usage
sudo lvs -o+thin_data_percent
```

---

## RAID with mdadm

### RAID Levels Compared

| Level | Min Disks | Description | Redundancy | Speed | Usable |
|-------|-----------|-------------|------------|-------|--------|
| **RAID 0** | 2 | Striping | None | ⚡⚡ Fastest | 100% |
| **RAID 1** | 2 | Mirroring | 1 disk fail | ⚡ Read fast | 50% |
| **RAID 5** | 3 | Striping + parity | 1 disk fail | ⚡ Good read | (N-1)/N |
| **RAID 6** | 4 | Striping + dual parity | 2 disk fail | 🔄 Moderate | (N-2)/N |
| **RAID 10** | 4 | Mirror + Stripe | 1 per mirror | ⚡⚡ Fast | 50% |

### Creating RAID

```bash
sudo apt install mdadm

# RAID 1 (mirror)
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# RAID 5
sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd

# RAID 10
sudo mdadm --create /dev/md0 --level=10 --raid-devices=4 /dev/sdb /dev/sdc /dev/sdd /dev/sde

# Monitor build progress
cat /proc/mdstat
watch -n 1 cat /proc/mdstat

# Save configuration
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u

# Create filesystem and mount
sudo mkfs.ext4 /dev/md0
sudo mount /dev/md0 /mnt/raid
```

### Managing RAID

```bash
# View status
sudo mdadm --detail /dev/md0
cat /proc/mdstat

# Mark disk as failed
sudo mdadm --manage /dev/md0 --fail /dev/sdc

# Remove failed disk
sudo mdadm --manage /dev/md0 --remove /dev/sdc

# Add replacement disk
sudo mdadm --manage /dev/md0 --add /dev/sdf
# Rebuild starts automatically, monitor with cat /proc/mdstat

# Add spare disk
sudo mdadm --manage /dev/md0 --add-spare /dev/sdg

# Stop RAID
sudo mdadm --stop /dev/md0
```

---

## LVM on RAID

The best of both worlds — redundancy + flexibility.

```bash
# 1. Create RAID array
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# 2. Create LVM on RAID
sudo pvcreate /dev/md0
sudo vgcreate vg_raid /dev/md0
sudo lvcreate -L 50G -n lv_data vg_raid
sudo mkfs.ext4 /dev/vg_raid/lv_data
sudo mount /dev/vg_raid/lv_data /mnt/data
```

---

## 🏋️ Practice Exercises

1. **(VM)** Create two virtual disks, make PVs, a VG, and two LVs
2. **Extend**: Add a third disk to the VG and extend an LV
3. **Snapshot**: Create an LVM snapshot, modify files, then restore
4. **(VM)** Create a RAID 1 array with two disks
5. **Fail**: Simulate a disk failure and replace it in RAID 1
6. **Monitor**: Use `cat /proc/mdstat` to watch a RAID rebuild
7. **LVM on RAID**: Combine RAID + LVM for a redundant, flexible setup

---

<p align="center">
  <a href="../23-performance-tuning/README.md">← Previous: Performance Tuning</a> · <a href="../README.md">🏠 Home</a> · <a href="../25-kernel-compilation-modules/README.md">Next: Kernel Compilation & Modules →</a>
</p>
