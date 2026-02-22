# 🆘 Chapter 34: Disaster Recovery & Incident Response

<p align="center">
  <img src="https://img.shields.io/badge/Level-Expert-red?style=for-the-badge" alt="Expert">
  <img src="https://img.shields.io/badge/Chapter-34%20of%2034-blue?style=for-the-badge" alt="Chapter 34">
</p>

---

## 📑 Table of Contents

- [Disaster Recovery Planning](#disaster-recovery-planning)
- [Boot Recovery](#boot-recovery)
- [Filesystem Recovery](#filesystem-recovery)
- [Data Recovery](#data-recovery)
- [Password Recovery](#password-recovery)
- [Network Troubleshooting Checklist](#network-troubleshooting-checklist)
- [Live Rescue Environments](#live-rescue-environments)
- [Incident Response Checklist](#incident-response-checklist)
- [Practice Exercises](#-practice-exercises)

---

## Disaster Recovery Planning

### DR Best Practices

| Practice | Description |
|----------|-------------|
| **Document everything** | Runbooks, architecture diagrams, configs |
| **Automate recovery** | Scripts for failover, restore, rebuild |
| **Test regularly** | Run DR drills at least quarterly |
| **3-2-1 backups** | 3 copies, 2 media types, 1 offsite |
| **RTO/RPO targets** | Define acceptable downtime and data loss |
| **Communication plan** | Who to contact, escalation paths |

---

## Boot Recovery

### GRUB Rescue

```bash
# If you see "grub rescue>"
ls                                    # List available partitions
ls (hd0,gpt2)/                       # Check for /boot
set root=(hd0,gpt2)
set prefix=(hd0,gpt2)/boot/grub
insmod normal
normal                                # Boot normally

# From a live USB — reinstall GRUB
sudo mount /dev/sda2 /mnt
sudo mount /dev/sda1 /mnt/boot/efi   # If UEFI
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt
grub-install /dev/sda                 # Or: grub-install --target=x86_64-efi
update-grub
exit
sudo umount -R /mnt
sudo reboot
```

### Kernel Panic Recovery

```bash
# At GRUB menu, press 'e' to edit boot entry
# Add to kernel line:
init=/bin/bash                        # Bypass init, get shell
# Or:
systemd.unit=emergency.target         # Emergency mode

# Once in shell:
mount -o remount,rw /                 # Remount root as read-write
# Fix the issue...
exec /sbin/init                       # Continue boot
```

### initramfs Issues

```bash
# From live USB or recovery mode
sudo mount /dev/sda2 /mnt
sudo chroot /mnt
update-initramfs -u -k all            # Rebuild all initramfs
# Or on RHEL:
dracut --force
exit
sudo reboot
```

---

## Filesystem Recovery

```bash
# Check filesystem (must be unmounted!)
sudo umount /dev/sda2
sudo fsck /dev/sda2                   # Interactive repair
sudo fsck -y /dev/sda2               # Automatic yes to repairs
sudo e2fsck -f /dev/sda2             # Force check ext4

# XFS repair
sudo xfs_repair /dev/sda2
sudo xfs_repair -L /dev/sda2         # Clear log (last resort)

# Btrfs repair
sudo btrfs check /dev/sda2
sudo btrfs rescue zero-log /dev/sda2
sudo btrfs rescue super-recover /dev/sda2

# If root filesystem is damaged, boot from live USB first!

# Check for bad blocks
sudo badblocks -sv /dev/sda           # Non-destructive scan
```

---

## Data Recovery

### Recover Deleted Files

```bash
# ext4 — extundelete
sudo apt install extundelete
sudo extundelete /dev/sda2 --restore-all
sudo extundelete /dev/sda2 --restore-file home/sovon/important.txt

# TestDisk — partition recovery
sudo apt install testdisk
sudo testdisk                          # Interactive TUI
# Can recover: deleted partitions, fix MBR, recover files

# PhotoRec — file carving (works on any filesystem)
sudo photorec /dev/sda2               # Recover by file type

# ddrescue — copy from failing disk
sudo apt install gddrescue
sudo ddrescue /dev/sdb /backup/disk.img /backup/logfile
# First pass: fast copy of readable areas
# Second pass: retry bad sectors
sudo ddrescue -r3 /dev/sdb /backup/disk.img /backup/logfile
```

### SMART Disk Monitoring

```bash
sudo apt install smartmontools

# Check disk health
sudo smartctl -a /dev/sda            # Full info
sudo smartctl -H /dev/sda            # Health status only
sudo smartctl -t short /dev/sda      # Run quick test
sudo smartctl -t long /dev/sda       # Run thorough test
sudo smartctl -l selftest /dev/sda   # View test results

# Enable auto-monitoring
sudo systemctl enable --now smartd
```

---

## Password Recovery

```bash
# Reset root password (single user mode)
# 1. Reboot, at GRUB press 'e'
# 2. Find the line starting with 'linux'
# 3. Add: init=/bin/bash
# 4. Press Ctrl+X to boot

mount -o remount,rw /
passwd root                           # Set new password
# Or: passwd username
exec /sbin/init

# Alternative: rd.break (RHEL/Fedora)
# Add to kernel line: rd.break
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
touch /.autorelabel                   # If SELinux is enabled
exit && exit
```

---

## Network Troubleshooting Checklist

```bash
# Layer 1: Physical
ip link show                          # Interface up?
ethtool eth0                          # Speed, duplex, link status

# Layer 2: Data Link
ip neighbor show                      # ARP table
arping 192.168.1.1                   # Test link-layer

# Layer 3: Network
ip addr show                          # IP configured?
ping -c 3 192.168.1.1               # Gateway reachable?
ping -c 3 8.8.8.8                   # Internet reachable?
ip route show                         # Routes correct?
traceroute 8.8.8.8                   # Where does it break?

# Layer 4: Transport
ss -tuln                              # Ports listening?
sudo nmap -sT localhost              # Scan local ports

# Layer 7: Application
curl -v http://localhost              # HTTP working?
dig example.com                       # DNS resolution?
nslookup example.com                  # DNS working?
cat /etc/resolv.conf                  # DNS config correct?

# Firewall
sudo iptables -L -n                   # Rules blocking?
sudo ufw status verbose
sudo nft list ruleset

# Quick diagnostic flow:
# 1. ip link → Is interface UP?
# 2. ip addr → Is IP assigned?
# 3. ping gateway → Can reach router?
# 4. ping 8.8.8.8 → Can reach internet?
# 5. ping example.com → Is DNS working?
# 6. curl URL → Is the app responding?
```

---

## Live Rescue Environments

```bash
# Boot from live USB, then:

# Mount the installed system
sudo mount /dev/sda2 /mnt
sudo mount /dev/sda1 /mnt/boot
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo mount --bind /run /mnt/run

# Chroot into installed system
sudo chroot /mnt /bin/bash

# Now you can:
# - Reinstall GRUB
# - Fix fstab
# - Reset passwords
# - Repair packages
apt --fix-broken install
dpkg --configure -a

# Exit and reboot
exit
sudo umount -R /mnt
sudo reboot
```

---

## Incident Response Checklist

### When Something Breaks

```
1. DON'T PANIC
2. Gather information
   - What changed recently?
   - What are the symptoms?
   - What do the logs say?

3. Check logs
   journalctl -xe                     # Recent errors
   journalctl -b -p err              # Errors this boot
   dmesg -T | tail -50               # Kernel messages
   tail -100 /var/log/syslog

4. Check resources
   df -h                             # Disk full?
   free -h                           # Memory exhausted?
   top                               # CPU overloaded?
   ss -tuln                          # Ports listening?

5. Check services
   systemctl --failed                # Failed services
   systemctl status <service>

6. Document everything
   - What you found
   - What you did to fix it
   - Timestamp of events

7. Post-mortem
   - Root cause analysis
   - Prevention measures
   - Update runbooks
```

---

## 🏋️ Practice Exercises

1. **(VM)** Break GRUB intentionally and recover from a live USB
2. **fsck**: Run a filesystem check on an unmounted partition
3. **Password**: Reset root password via single-user mode
4. **TestDisk**: Practice recovering deleted files
5. **SMART**: Check your disk's SMART status
6. **Network**: Practice the layer-by-layer troubleshooting checklist
7. **Chroot**: Boot from live USB and chroot into your installed system
8. **Incident**: Create an incident response runbook for your systems

---

<p align="center">
  <a href="../33-ha-clustering/README.md">← Previous: HA Clustering</a> · <a href="../README.md">🏠 Home</a>
</p>

---

## 🎉 Congratulations!

You've completed **The Ultimate Linux Guide** — 34 chapters covering Linux from absolute beginner to expert level. Keep practicing, keep breaking things (in VMs!), and keep learning. The Linux journey never truly ends.

> "Talk is cheap. Show me the code." — *Linus Torvalds*

<p align="center">
  <a href="../README.md">🏠 Back to Home</a> · <a href="../cheatsheets/README.md">📋 Cheatsheets</a> · <a href="../resources/README.md">📚 Resources</a>
</p>
