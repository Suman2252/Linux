# 📊 Chapter 15: System Monitoring & Logs

<p align="center">
  <img src="https://img.shields.io/badge/Level-Intermediate-yellow?style=for-the-badge" alt="Intermediate">
  <img src="https://img.shields.io/badge/Chapter-15%20of%2034-blue?style=for-the-badge" alt="Chapter 15">
</p>

---

## 📑 Table of Contents

- [System Logging Architecture](#system-logging-architecture)
- [journalctl — Systemd Journal](#journalctl--systemd-journal)
- [Traditional Log Files](#traditional-log-files)
- [Real-Time Monitoring Tools](#real-time-monitoring-tools)
- [System Information Commands](#system-information-commands)
- [Log Rotation](#log-rotation)
- [Practice Exercises](#-practice-exercises)

---

## System Logging Architecture

```
Applications → systemd-journald → /run/log/journal/ (volatile)
                    ↓                    ↓
               rsyslog/syslog-ng → /var/log/ (persistent)
```

---

## journalctl — Systemd Journal

```bash
# View all logs
journalctl                           # All logs (oldest first)
journalctl -r                        # Reverse (newest first)
journalctl -f                        # Follow in real-time (like tail -f)

# Filter by time
journalctl --since "2024-01-01"
journalctl --since "1 hour ago"
journalctl --since "today"
journalctl --since "09:00" --until "10:00"
journalctl -b                        # Current boot only
journalctl -b -1                     # Previous boot

# Filter by unit/service
journalctl -u nginx                  # Nginx logs
journalctl -u sshd --since today     # SSH logs today
journalctl -u cron -f                # Follow cron logs

# Filter by priority
journalctl -p err                    # Errors and above
journalctl -p warning                # Warnings and above
# Priorities: emerg, alert, crit, err, warning, notice, info, debug

# Filter by process
journalctl _PID=1234
journalctl _UID=1000
journalctl _COMM=bash

# Kernel messages
journalctl -k                        # Same as dmesg
journalctl -k -b -1                  # Kernel logs from last boot

# Output formats
journalctl -o json-pretty            # JSON format
journalctl -o short-iso              # ISO timestamps
journalctl --no-pager                # Don't paginate

# Disk usage
journalctl --disk-usage              # How much space logs use
sudo journalctl --vacuum-size=500M   # Trim to 500 MB
sudo journalctl --vacuum-time=7d     # Keep only 7 days
```

---

## Traditional Log Files

```bash
# Key log files in /var/log/
/var/log/syslog            # General system log (Debian/Ubuntu)
/var/log/messages          # General system log (RHEL/CentOS)
/var/log/auth.log          # Authentication (Debian/Ubuntu)
/var/log/secure            # Authentication (RHEL/CentOS)
/var/log/kern.log          # Kernel messages
/var/log/dmesg             # Boot messages
/var/log/boot.log          # Boot process
/var/log/dpkg.log          # Package manager (Debian)
/var/log/apt/history.log   # APT history
/var/log/cron              # Cron job logs
/var/log/faillog           # Failed logins
/var/log/wtmp              # Login records (binary — use 'last')
/var/log/btmp              # Bad login attempts (binary — use 'lastb')

# Common commands
sudo tail -f /var/log/syslog          # Follow system log
sudo tail -50 /var/log/auth.log       # Last 50 auth entries
sudo grep "Failed" /var/log/auth.log  # Find failed logins
last                                   # Recent logins
lastb                                  # Failed login attempts
sudo dmesg                            # Kernel ring buffer
sudo dmesg -T                         # With human-readable timestamps
sudo dmesg -l err,warn                # Only errors and warnings
```

---

## Real-Time Monitoring Tools

### `htop` — Interactive Process Viewer

```bash
sudo apt install htop
htop
# F1=Help F2=Setup F3=Search F4=Filter F5=Tree F6=Sort F9=Kill F10=Quit
```

### `vmstat` — Virtual Memory Stats

```bash
vmstat 1 10                  # Every 1 second, 10 times
# Output: procs, memory, swap, io, system, cpu
```

### `iostat` — Disk I/O

```bash
sudo apt install sysstat
iostat -xh 1 5               # Extended, human-readable, every 1s
```

### `sar` — System Activity Reporter

```bash
sar -u 1 5                   # CPU usage (every 1s, 5 samples)
sar -r 1 5                   # Memory usage
sar -d 1 5                   # Disk I/O
sar -n DEV 1 5               # Network traffic
sar -q 1 5                   # Load averages
```

### `iotop` — Disk I/O Per Process

```bash
sudo apt install iotop
sudo iotop                   # Interactive I/O monitor
sudo iotop -ao               # Accumulated, only show active
```

### `nload` / `iftop` — Network Monitoring

```bash
sudo apt install nload iftop
nload                        # Network bandwidth per interface
sudo iftop                   # Network connections bandwidth
```

### `glances` — All-in-One

```bash
sudo apt install glances
glances                      # CPU, memory, disk, network, processes
```

---

## System Information Commands

```bash
# System overview
uname -a                     # Kernel info
hostnamectl                  # Hostname & OS info
lsb_release -a               # Distribution info
cat /etc/os-release          # OS release info

# Hardware
lscpu                        # CPU details
lsmem                        # Memory layout
lsblk                        # Block devices
lsusb                        # USB devices
lspci                        # PCI devices
lshw -short                  # All hardware summary (need sudo)

# Uptime & load
uptime                       # Uptime + load
w                            # Who's logged in + what they're doing
who                          # Who's logged in

# Temperature (if supported)
sudo apt install lm-sensors
sudo sensors-detect          # First-time setup
sensors                      # Show temperatures
```

---

## Log Rotation

**logrotate** prevents logs from filling up your disk.

```bash
# Main config
cat /etc/logrotate.conf

# Per-service configs
ls /etc/logrotate.d/
cat /etc/logrotate.d/nginx
```

### Example Config

```
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily                    # Rotate daily
    missingok                # Don't error if log missing
    rotate 14                # Keep 14 rotations
    compress                 # gzip old logs
    delaycompress            # Compress on next rotation
    notifempty               # Don't rotate empty logs
    create 0640 root adm     # Create new log with permissions
    postrotate               # Command after rotating
        systemctl reload myapp
    endscript
}
```

```bash
# Test logrotate
sudo logrotate -d /etc/logrotate.d/myapp    # Dry run (debug)
sudo logrotate -f /etc/logrotate.d/myapp    # Force rotation
```

---

## 🏋️ Practice Exercises

1. **journalctl**: View all logs from the current boot
2. **Filter**: Find SSH authentication failures from the last 24 hours
3. **Follow**: Monitor syslog in real-time with `journalctl -f`
4. **Priority**: Show only errors and critical messages
5. **htop**: Identify the top 3 memory-consuming processes
6. **sar**: Record CPU usage for 10 seconds and analyze
7. **Last**: Check the last 5 user logins with `last -5`
8. **logrotate**: Create a logrotate config for a custom log file

---

<p align="center">
  <a href="../14-cron-jobs-scheduling/README.md">← Previous: Cron Jobs & Scheduling</a> · <a href="../README.md">🏠 Home</a> · <a href="../16-archive-compression-backup/README.md">Next: Archive, Compression & Backup →</a>
</p>
