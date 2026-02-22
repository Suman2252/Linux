# 🧬 Chapter 18: Kernel & System Internals

<p align="center">
  <img src="https://img.shields.io/badge/Level-Advanced-orange?style=for-the-badge" alt="Advanced">
  <img src="https://img.shields.io/badge/Chapter-18%20of%2034-blue?style=for-the-badge" alt="Chapter 18">
</p>

---

## 📑 Table of Contents

- [Kernel Architecture](#kernel-architecture)
- [The Boot Process](#the-boot-process)
- [Kernel Modules](#kernel-modules)
- [sysctl — Runtime Kernel Parameters](#sysctl--runtime-kernel-parameters)
- [initramfs / initrd](#initramfs--initrd)
- [System Calls](#system-calls)
- [The init System (PID 1)](#the-init-system-pid-1)
- [Practice Exercises](#-practice-exercises)

---

## Kernel Architecture

The Linux kernel is **monolithic** — all core services run in kernel space as a single binary.

```
┌────────────────────────────────────────────────┐
│                USER SPACE                       │
│  Applications → Libraries → System Call API     │
├────────────────────────────────────────────────┤
│                KERNEL SPACE                     │
│  ┌──────────────────────────────────────────┐  │
│  │ System Call Interface                     │  │
│  ├────────┬────────┬────────┬───────────────┤  │
│  │ VFS    │Process │ Memory │ Network       │  │
│  │(Virtual│ Sched  │ Mgmt   │ Stack         │  │
│  │ FS)    │        │        │               │  │
│  ├────────┴────────┴────────┴───────────────┤  │
│  │         Device Drivers (modules)          │  │
│  ├──────────────────────────────────────────┤  │
│  │  Architecture-Dependent Code (x86, ARM)   │  │
│  └──────────────────────────────────────────┘  │
├────────────────────────────────────────────────┤
│                HARDWARE                         │
└────────────────────────────────────────────────┘
```

### Kernel Subsystems

| Subsystem | Role |
|-----------|------|
| **Process Scheduler** | Decides which process runs on which CPU core |
| **Memory Manager** | Virtual memory, paging, OOM killer |
| **VFS** | Abstraction layer over all filesystems |
| **Network Stack** | TCP/IP, sockets, netfilter |
| **Device Drivers** | Hardware communication (loaded as modules) |
| **IPC** | Inter-process communication (pipes, signals, sockets) |

```bash
# Kernel information
uname -r                        # Kernel version (e.g., 6.8.0-41-generic)
uname -a                        # All system info
cat /proc/version                # Kernel build details
cat /proc/cmdline                # Boot parameters
```

---

## The Boot Process

```
1. BIOS/UEFI
   └→ POST (Power-On Self-Test)
   └→ Find boot device

2. Bootloader (GRUB)
   └→ Load kernel + initramfs into memory
   └→ Pass boot parameters to kernel

3. Kernel
   └→ Decompress itself
   └→ Initialize hardware, memory, CPU
   └→ Mount initramfs as temporary root
   └→ Execute /init from initramfs

4. initramfs
   └→ Load necessary drivers (disk, filesystem)
   └→ Find and mount real root filesystem
   └→ pivot_root to real filesystem

5. Init System (systemd, PID 1)
   └→ Mount filesystems from /etc/fstab
   └→ Start services (networking, SSH, etc.)
   └→ Reach default target (multi-user or graphical)
   └→ Display login prompt
```

### GRUB Bootloader

```bash
# GRUB config
cat /boot/grub/grub.cfg              # Generated config (don't edit!)
cat /etc/default/grub                # User settings

# Edit GRUB settings
sudo vim /etc/default/grub
# GRUB_TIMEOUT=5
# GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
# GRUB_CMDLINE_LINUX=""

# Rebuild GRUB config
sudo update-grub                      # Debian/Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # RHEL/Fedora

# Reinstall GRUB (boot repair)
sudo grub-install /dev/sda
sudo update-grub
```

---

## Kernel Modules

Modules are pieces of kernel code loaded on demand (like drivers).

```bash
# List loaded modules
lsmod
lsmod | grep usb

# Module info
modinfo ext4
modinfo nvidia

# Load a module
sudo modprobe vfat               # Load FAT filesystem module
sudo modprobe -v snd-hda-intel   # Verbose load

# Unload a module
sudo modprobe -r vfat            # Remove module
sudo rmmod vfat                  # Alternative

# Load module at boot — add to:
echo "vfat" | sudo tee /etc/modules-load.d/vfat.conf

# Blacklist a module (prevent loading)
echo "blacklist nouveau" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
sudo update-initramfs -u         # Rebuild initramfs

# Module parameters
modinfo i915 | grep parm         # Show available parameters
echo "options i915 enable_guc=3" | sudo tee /etc/modprobe.d/i915.conf

# Module dependencies
modprobe --show-depends ext4
```

---

## sysctl — Runtime Kernel Parameters

```bash
# View all parameters
sysctl -a

# View specific parameter
sysctl net.ipv4.ip_forward
sysctl vm.swappiness

# Set temporarily (until reboot)
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -w vm.swappiness=10

# Set permanently
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p                   # Reload

# Or use drop-in files
echo "vm.swappiness = 10" | sudo tee /etc/sysctl.d/99-custom.conf
sudo sysctl --system              # Reload all
```

### Common sysctl Tweaks

| Parameter | Description | Default |
|-----------|------------|---------|
| `vm.swappiness` | How aggressively to use swap | 60 |
| `net.ipv4.ip_forward` | Enable IP forwarding (routing) | 0 |
| `net.core.somaxconn` | Max socket connections queue | 4096 |
| `fs.file-max` | Max open files system-wide | 9223372036854775807 |
| `fs.inotify.max_user_watches` | Max inotify watches | 8192 |

---

## initramfs / initrd

The **initramfs** is a temporary root filesystem loaded into RAM during boot, before the real root is mounted.

```bash
# View current initramfs
ls -la /boot/initrd.img*

# List contents
lsinitramfs /boot/initrd.img-$(uname -r)

# Rebuild initramfs
sudo update-initramfs -u              # Update current kernel
sudo update-initramfs -u -k all       # Update all kernels
sudo mkinitramfs -o /boot/initrd.img-$(uname -r)  # Manual rebuild

# Dracut (RHEL/Fedora)
sudo dracut --force
```

---

## System Calls

System calls are the API between user space and kernel space.

```bash
# Trace system calls of a command
strace ls /tmp                   # See all syscalls ls makes
strace -c ls /tmp                # Summary statistics
strace -e open,read,write ls     # Filter specific syscalls
strace -p 1234                   # Attach to running process
strace -f -e trace=network curl https://example.com  # Network calls

# Common system calls
# open()   — Open a file
# read()   — Read from file descriptor
# write()  — Write to file descriptor
# close()  — Close file descriptor
# fork()   — Create child process
# exec()   — Execute a program
# mmap()   — Map memory
# stat()   — Get file info
# socket() — Create network socket
```

---

## The init System (PID 1)

```bash
# Check your init system
ps -p 1 -o comm=                 # systemd, init, or others
stat /proc/1/exe                 # Binary path

# systemd (modern standard)
systemctl list-units              # All loaded units
systemctl list-unit-files         # All available units
systemctl get-default             # Default boot target
systemctl set-default multi-user.target   # Boot to CLI
systemctl set-default graphical.target    # Boot to GUI

# Boot targets (runlevels)
# multi-user.target = runlevel 3 (CLI)
# graphical.target  = runlevel 5 (GUI)
# rescue.target     = single user
# emergency.target  = minimal shell
```

---

## 🏋️ Practice Exercises

1. **Kernel**: Check your kernel version and when it was compiled
2. **Modules**: List all loaded modules and find one related to your network
3. **modinfo**: Get detailed info about the `ext4` module
4. **sysctl**: Check and change the swappiness value
5. **strace**: Trace the system calls of `cat /etc/hostname`
6. **Boot**: Read `/proc/cmdline` to see your boot parameters
7. **GRUB**: Look at `/etc/default/grub` settings
8. **initramfs**: List the contents of your current initramfs

---

<p align="center">
  <a href="../17-advanced-shell-scripting/README.md">← Previous: Advanced Shell Scripting</a> · <a href="../README.md">🏠 Home</a> · <a href="../19-systemd-service-management/README.md">Next: Systemd & Service Management →</a>
</p>
