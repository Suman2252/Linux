# 📖 Linux Glossary

<p align="center">
  <img src="https://img.shields.io/badge/Glossary-A%20to%20Z-FCC624?style=for-the-badge&logo=linux&logoColor=black" alt="Glossary">
</p>

Key Linux terms and definitions.

---

## A

| Term | Definition |
|------|-----------|
| **ACL** | Access Control List — fine-grained permissions beyond rwx |
| **AppArmor** | MAC security module that uses path-based profiles (Ubuntu/SUSE) |
| **APT** | Advanced Package Tool — Debian/Ubuntu package manager |

## B

| Term | Definition |
|------|-----------|
| **Bash** | Bourne Again Shell — the default shell on most Linux distros |
| **Block device** | Storage device that reads/writes data in fixed-size blocks (e.g., HDD, SSD) |
| **Boot loader** | Program that loads the kernel at startup (e.g., GRUB) |
| **Btrfs** | B-tree filesystem with snapshots, compression, and built-in RAID |

## C

| Term | Definition |
|------|-----------|
| **cgroups** | Control groups — limit and isolate resource usage per process group |
| **Chroot** | Change root — run a process with a different root directory |
| **CLI** | Command Line Interface — text-based terminal |
| **Container** | Lightweight isolation using namespaces and cgroups (e.g., Docker) |
| **Cron** | Time-based job scheduler daemon |

## D

| Term | Definition |
|------|-----------|
| **Daemon** | Background service process (usually ends with `d`, e.g., `sshd`) |
| **Device file** | Special file in `/dev/` representing hardware or virtual devices |
| **Distro** | Linux distribution — complete OS built around the Linux kernel |
| **DKMS** | Dynamic Kernel Module Support — auto-rebuilds modules on kernel upgrades |
| **DNS** | Domain Name System — translates hostnames to IP addresses |

## E

| Term | Definition |
|------|-----------|
| **eBPF** | Extended Berkeley Packet Filter — run sandboxed programs in kernel space |
| **Environment variable** | Named value available to processes (e.g., `$PATH`, `$HOME`) |
| **ext4** | Fourth extended filesystem — standard Linux filesystem |

## F

| Term | Definition |
|------|-----------|
| **FHS** | Filesystem Hierarchy Standard — defines directory structure (`/bin`, `/etc`, etc.) |
| **FOSS** | Free and Open Source Software |
| **fstab** | `/etc/fstab` — defines filesystems to mount at boot |

## G

| Term | Definition |
|------|-----------|
| **GNU** | GNU's Not Unix — the userspace tools that complement the Linux kernel |
| **GRUB** | GRand Unified Bootloader — the standard Linux bootloader |
| **GUI** | Graphical User Interface — desktop environment |

## H–I

| Term | Definition |
|------|-----------|
| **Hugepages** | Large memory pages (2MB/1GB) for performance-sensitive apps |
| **inode** | Data structure storing file metadata (permissions, size, block locations) |
| **initramfs** | Initial RAM filesystem — temporary root loaded during boot |
| **iptables** | Classic Linux firewall framework (being replaced by nftables) |

## J–K

| Term | Definition |
|------|-----------|
| **journalctl** | Tool to query systemd's journal (logs) |
| **Kernel** | Core of the OS — manages hardware, memory, processes |
| **KVM** | Kernel-based Virtual Machine — Type 1 hypervisor built into Linux |

## L

| Term | Definition |
|------|-----------|
| **LFS** | Linux From Scratch — build a complete Linux system from source |
| **LVM** | Logical Volume Manager — flexible disk management abstraction |

## M

| Term | Definition |
|------|-----------|
| **MAC** | Mandatory Access Control — system-enforced security (SELinux/AppArmor) |
| **Module** | Loadable kernel code (drivers, filesystems) — load with `modprobe` |
| **Mount** | Attach a filesystem to a directory in the file tree |

## N

| Term | Definition |
|------|-----------|
| **Namespace** | Kernel feature that isolates resources (PID, network, mount, etc.) |
| **NFS** | Network File System — share directories over the network |
| **nftables** | Modern replacement for iptables |

## O–P

| Term | Definition |
|------|-----------|
| **OOM Killer** | Out-of-Memory Killer — terminates processes when RAM is exhausted |
| **Package** | Software bundle with binaries, configs, and dependency info |
| **PID** | Process ID — unique number identifying a running process |
| **Pipe** | `|` — connects stdout of one command to stdin of another |
| **Process** | Running instance of a program |

## Q–R

| Term | Definition |
|------|-----------|
| **RAID** | Redundant Array of Independent Disks — combine disks for performance/redundancy |
| **Root** | The superuser (`UID 0`) or the top-level directory (`/`) |
| **rsync** | Smart file synchronization tool — only transfers changes |

## S

| Term | Definition |
|------|-----------|
| **SELinux** | Security-Enhanced Linux — MAC using security contexts (RHEL/Fedora) |
| **Shell** | Command interpreter — accepts and executes commands (bash, zsh, fish) |
| **Signal** | Software interrupt sent to a process (e.g., `SIGTERM`, `SIGKILL`) |
| **SSH** | Secure Shell — encrypted remote access protocol |
| **Swap** | Disk space used as overflow for RAM |
| **Symlink** | Symbolic link — pointer to another file or directory |
| **Systemd** | The init system (PID 1) on most modern distros |
| **sysctl** | Modify kernel parameters at runtime |

## T–U

| Term | Definition |
|------|-----------|
| **TTY** | TeleTYpewriter — terminal device or virtual console |
| **UEFI** | Unified Extensible Firmware Interface — modern BIOS replacement |
| **UID** | User ID — numeric identifier for users |

## V–W

| Term | Definition |
|------|-----------|
| **VFS** | Virtual File System — kernel abstraction layer over all filesystems |
| **WireGuard** | Modern, fast VPN protocol built into the Linux kernel |

## X–Z

| Term | Definition |
|------|-----------|
| **X11/Xorg** | Legacy display server (being replaced by Wayland) |
| **Wayland** | Modern display server protocol |
| **ZFS** | Advanced filesystem with built-in RAID, snapshots, and checksums |
| **Zombie** | Terminated process whose exit status hasn't been collected by its parent |

---

<p align="center">
  <a href="README.md">🏠 Back to Home</a>
</p>
