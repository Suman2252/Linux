# 📁 Chapter 04: Filesystem Hierarchy

<p align="center">
  <img src="https://img.shields.io/badge/Level-Beginner-green?style=for-the-badge" alt="Beginner">
  <img src="https://img.shields.io/badge/Chapter-04%20of%2034-blue?style=for-the-badge" alt="Chapter 04">
</p>

---

## 📑 Table of Contents

- [Everything is a File](#everything-is-a-file)
- [The FHS (Filesystem Hierarchy Standard)](#the-fhs-filesystem-hierarchy-standard)
- [Root Directory /](#root-directory-)
- [Directory Deep Dive](#directory-deep-dive)
- [Special Filesystems](#special-filesystems)
- [File Types in Linux](#file-types-in-linux)
- [Absolute vs Relative Paths](#absolute-vs-relative-paths)
- [Practice Exercises](#-practice-exercises)

---

## Everything is a File

One of the most important Linux philosophies:

> **"On a Linux system, everything is a file. If something is not a file, it is a process."**

This means:
- Regular files → files
- Directories → special files
- Hard drives → files (`/dev/sda`)
- USB devices → files (`/dev/usb`)
- Network sockets → files
- Running processes → files (in `/proc`)
- Even your keyboard and screen → files (`/dev/stdin`, `/dev/stdout`)

---

## The FHS (Filesystem Hierarchy Standard)

The **FHS** defines the standard directory structure for Linux systems.

```
/                       ← Root of everything
├── bin/                ← Essential user binaries
├── boot/               ← Boot loader files, kernel
├── dev/                ← Device files
├── etc/                ← System configuration files
├── home/               ← User home directories
│   ├── sovon/
│   └── alice/
├── lib/                ← Shared libraries
├── lib64/              ← 64-bit shared libraries
├── media/              ← Mount points for removable media
├── mnt/                ← Temporary mount points
├── opt/                ← Optional/third-party software
├── proc/               ← Virtual filesystem for processes
├── root/               ← Root user's home directory
├── run/                ← Runtime data
├── sbin/               ← System binaries (admin commands)
├── srv/                ← Service data
├── sys/                ← Virtual filesystem for hardware
├── tmp/                ← Temporary files (cleared on reboot)
├── usr/                ← User programs & data (read-only)
│   ├── bin/
│   ├── lib/
│   ├── local/
│   └── share/
└── var/                ← Variable data (logs, mail, spool)
    ├── log/
    ├── cache/
    └── tmp/
```

---

## Root Directory /

The `/` (slash) is the top-level directory. **Everything** in Linux exists under `/`.

> 🏠 **Analogy**: If your Linux system were a building, `/` would be the ground floor lobby. Every room (directory) and every item (file) in the building is accessed from here.

> ⚠️ **Warning**: `/` (root directory) ≠ `/root` (root user's home directory). Don't confuse them!

---

## Directory Deep Dive

### `/bin` — Essential User Binaries

Contains commands that **all users** need, even in single-user mode.

```bash
ls /bin
# Contains: ls, cp, mv, rm, cat, grep, bash, echo, mount ...
```

> 📝 On modern systems, `/bin` is often a symlink to `/usr/bin`.

---

### `/boot` — Boot Files

Contains everything needed to **start the system**.

```bash
ls /boot
# vmlinuz-6.8.0-xx    ← The Linux kernel
# initrd.img-6.8.0-xx ← Initial ramdisk
# grub/               ← GRUB bootloader config
```

> ⚠️ **Do NOT** delete files in `/boot` unless you know exactly what you're doing.

---

### `/dev` — Device Files

Everything is a file — even hardware devices.

```bash
ls /dev
# sda         ← First hard drive
# sda1        ← First partition of sda
# sdb         ← Second hard drive (or USB)
# null        ← Black hole (discards all data)
# zero        ← Infinite stream of zeros
# random      ← Random number generator
# urandom     ← Faster random generator
# tty         ← Terminal devices
# stdin       ← Standard input (keyboard)
# stdout      ← Standard output (screen)
# stderr      ← Standard error (screen)
```

```bash
# Useful /dev tricks
echo "Gone forever" > /dev/null     # Discard output
dd if=/dev/zero bs=1M count=100 of=testfile  # Create a 100MB file of zeros
cat /dev/urandom | head -c 32 | base64       # Generate random string
```

---

### `/etc` — Configuration Files

**All system-wide configuration** lives here. This is one of the most important directories.

```bash
ls /etc
# passwd       ← User account information
# shadow       ← Encrypted passwords
# group        ← Group information
# hostname     ← System hostname
# hosts        ← Static hostname to IP mapping
# fstab        ← Filesystem mount table
# sudoers      ← Sudo permissions
# ssh/         ← SSH configuration
# apt/         ← APT package manager config
# cron.d/      ← Cron job configurations
# systemd/     ← Systemd configuration
# nginx/       ← Nginx web server config (if installed)
```

```bash
# Peek at common config files
cat /etc/hostname       # Your computer's name
cat /etc/os-release     # Distro information
head -5 /etc/passwd     # First 5 user accounts
```

> 📝 **"etc"** originally stood for "et cetera" but is now treated as "Editable Text Configuration."

---

### `/home` — User Home Directories

Each user gets their own directory under `/home`.

```bash
/home/
├── sovon/          ← Your stuff
│   ├── Desktop/
│   ├── Documents/
│   ├── Downloads/
│   ├── .bashrc     ← Bash configuration (hidden)
│   ├── .ssh/       ← SSH keys (hidden)
│   └── .config/    ← App settings (hidden)
└── alice/          ← Another user's stuff
```

```bash
# The ~ shortcut always refers to YOUR home directory
echo ~              # /home/sovon
echo ~alice         # /home/alice

# Important dotfiles in your home
cat ~/.bashrc       # Bash shell configuration
cat ~/.profile      # Login shell configuration
ls ~/.ssh/          # SSH keys and config
```

---

### `/proc` — Process Information (Virtual)

A **virtual filesystem** — files here don't exist on disk. The kernel generates them on-the-fly.

```bash
ls /proc
# 1/         ← Process with PID 1 (systemd/init)
# 1234/      ← Process with PID 1234
# cpuinfo    ← CPU information
# meminfo    ← Memory information
# version    ← Kernel version
# uptime     ← System uptime
# loadavg    ← System load averages

# Useful examples
cat /proc/cpuinfo       # Detailed CPU info
cat /proc/meminfo       # Memory details
cat /proc/version       # Kernel version string
cat /proc/uptime        # Uptime in seconds
cat /proc/loadavg       # Load averages
cat /proc/partitions    # Disk partitions
```

---

### `/var` — Variable Data

Data that **changes frequently** during normal operation.

```bash
/var/
├── log/        ← System logs
│   ├── syslog
│   ├── auth.log
│   ├── kern.log
│   └── apt/
├── cache/      ← Application cache
│   └── apt/    ← Downloaded .deb packages
├── mail/       ← User mail
├── spool/      ← Print and mail queues
├── tmp/        ← Temporary files (survives reboot)
└── www/        ← Web server files (Apache/Nginx)
```

```bash
# Check recent system logs
sudo tail -20 /var/log/syslog

# See authentication logs
sudo tail -20 /var/log/auth.log

# Check disk usage of /var
du -sh /var/*
```

---

### `/tmp` — Temporary Files

```bash
# Anyone can write here. Files are deleted on reboot.
echo "temporary data" > /tmp/mytemp.txt
ls -la /tmp

# Check the sticky bit (the 't' at the end)
ls -ld /tmp
# drwxrwxrwt  ← 't' means only the owner can delete their own files
```

---

### `/usr` — User Programs

Contains the majority of user-facing programs and data. Treated as **read-only**.

```bash
/usr/
├── bin/        ← Most user commands (ls, grep, vim, python, etc.)
├── sbin/       ← Non-essential system commands
├── lib/        ← Libraries for /usr/bin and /usr/sbin
├── local/      ← Locally installed software (compiled from source)
│   ├── bin/
│   ├── lib/
│   └── share/
├── share/      ← Architecture-independent data (man pages, icons)
│   ├── man/    ← Manual pages
│   ├── doc/    ← Documentation
│   └── icons/  ← System icons
└── include/    ← C/C++ header files
```

---

### `/opt` — Optional Software

Third-party software that doesn't follow the standard `/usr` layout.

```bash
/opt/
├── google/chrome/      ← Google Chrome
├── discord/            ← Discord
└── lampp/              ← XAMPP web stack
```

---

## Special Filesystems

### `/proc` and `/sys` — Virtual Filesystems

These don't exist on disk — the kernel creates them in memory.

| Filesystem | Purpose | Example |
|-----------|---------|---------|
| `/proc` | Process and kernel info | `/proc/cpuinfo`, `/proc/1/status` |
| `/sys` | Hardware and driver info | `/sys/class/net/`, `/sys/block/sda/` |

```bash
# Find your network interfaces
ls /sys/class/net/

# Check battery status (laptops)
cat /sys/class/power_supply/BAT0/capacity

# Check disk scheduler
cat /sys/block/sda/queue/scheduler
```

---

## File Types in Linux

Linux has 7 file types, identified by the first character in `ls -l` output:

| Symbol | Type | Description |
|--------|------|-------------|
| `-` | Regular file | Text files, binaries, images |
| `d` | Directory | Contains other files |
| `l` | Symbolic link | Pointer to another file |
| `c` | Character device | Serial data (keyboard, terminal) |
| `b` | Block device | Block data (hard drives) |
| `s` | Socket | Inter-process communication |
| `p` | Named pipe (FIFO) | Inter-process communication |

```bash
# Identify file type
file /bin/bash          # ELF 64-bit executable
file /etc/passwd        # ASCII text
file /dev/sda           # block special

# Find all symlinks in /usr/bin
find /usr/bin -type l | head -10
```

---

## Absolute vs Relative Paths

| Type | Starts With | Example | Meaning |
|------|------------|---------|---------|
| **Absolute** | `/` | `/home/sovon/Documents` | Full path from root |
| **Relative** | Anything else | `Documents` or `./Documents` | Relative to current directory |

### Special Path Symbols

| Symbol | Meaning |
|--------|---------|
| `/` | Root directory |
| `~` | Home directory |
| `.` | Current directory |
| `..` | Parent directory |
| `-` | Previous directory (for `cd`) |

```bash
# These are equivalent (if you're in /home/sovon):
cd /home/sovon/Documents    # Absolute
cd ~/Documents              # Using ~
cd Documents                # Relative
cd ./Documents              # Relative (explicit)
```

---

## 🏋️ Practice Exercises

1. **Explore**: Run `ls /` and identify each directory's purpose
2. **Config Files**: Read `/etc/os-release` — what distro are you running?
3. **Processes**: Run `ls /proc` and find your shell's PID with `echo $$`, then `ls /proc/$$`
4. **Hardware**: Read `/proc/cpuinfo` — how many CPU cores do you have?
5. **Memory**: Read `/proc/meminfo` — how much total RAM do you have?
6. **Devices**: List `/dev` — can you find your hard drive device?
7. **Logs**: Run `sudo tail -10 /var/log/syslog` — what's the latest log entry?
8. **File Types**: Run `file /bin/bash /etc/passwd /dev/null` — what types are they?

---

<p align="center">
  <a href="../03-terminal-basics/README.md">← Previous: Terminal Basics</a> · <a href="../README.md">🏠 Home</a> · <a href="../05-file-directory-operations/README.md">Next: File & Directory Operations →</a>
</p>
