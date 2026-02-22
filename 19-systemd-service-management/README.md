# 🔧 Chapter 19: Systemd & Service Management

<p align="center">
  <img src="https://img.shields.io/badge/Level-Advanced-orange?style=for-the-badge" alt="Advanced">
  <img src="https://img.shields.io/badge/Chapter-19%20of%2034-blue?style=for-the-badge" alt="Chapter 19">
</p>

---

## 📑 Table of Contents

- [What is Systemd?](#what-is-systemd)
- [Managing Services](#managing-services)
- [Unit Files](#unit-files)
- [Creating Custom Services](#creating-custom-services)
- [Targets (Runlevels)](#targets-runlevels)
- [Journald (Logging)](#journald-logging)
- [Systemd Timers (Revisited)](#systemd-timers-revisited)
- [Socket Activation](#socket-activation)
- [Resource Control (cgroups)](#resource-control-cgroups)
- [Practice Exercises](#-practice-exercises)

---

## What is Systemd?

**systemd** is the init system (PID 1) on most modern Linux distros. It manages services, mounts, timers, devices, and more.

### systemd Components

| Component | Description |
|-----------|-------------|
| `systemd` | Init system and service manager |
| `systemctl` | Control systemd and services |
| `journald` | Logging daemon |
| `journalctl` | Query logs |
| `networkd` | Network management |
| `resolved` | DNS resolution |
| `logind` | Login management |
| `timedatectl` | Time and timezone |
| `hostnamectl` | Hostname management |

---

## Managing Services

```bash
# Start/Stop/Restart
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx         # Stop + Start
sudo systemctl reload nginx          # Reload config (no downtime)
sudo systemctl reload-or-restart nginx  # Prefer reload, fallback to restart

# Enable/Disable (auto-start at boot)
sudo systemctl enable nginx          # Start on boot
sudo systemctl disable nginx         # Don't start on boot
sudo systemctl enable --now nginx    # Enable AND start
sudo systemctl disable --now nginx   # Disable AND stop

# Status
systemctl status nginx               # Detailed status
systemctl is-active nginx            # Just "active" or "inactive"
systemctl is-enabled nginx           # "enabled" or "disabled"
systemctl is-failed nginx            # "failed" or "active"

# List services
systemctl list-units --type=service              # Running services
systemctl list-units --type=service --all        # All services
systemctl list-unit-files --type=service          # All service files
systemctl list-units --failed                     # Failed units

# Mask/Unmask (prevent starting entirely)
sudo systemctl mask nginx            # Cannot be started (even manually)
sudo systemctl unmask nginx          # Allow starting again
```

---

## Unit Files

Systemd uses **unit files** to define services, timers, mounts, and more.

```bash
# Find unit files
systemctl show -p FragmentPath nginx.service     # Location of unit file
systemctl cat nginx.service                       # View the unit file

# Unit file locations (priority order):
# /etc/systemd/system/        ← Admin customizations (highest priority)
# /run/systemd/system/        ← Runtime units
# /usr/lib/systemd/system/    ← Package-installed units
```

### Unit File Structure

```ini
[Unit]
Description=My Application Server
Documentation=https://docs.example.com
After=network.target postgresql.service
Wants=postgresql.service
Requires=network-online.target

[Service]
Type=simple
User=appuser
Group=appgroup
WorkingDirectory=/opt/myapp
Environment=NODE_ENV=production
EnvironmentFile=/opt/myapp/.env
ExecStartPre=/opt/myapp/pre-start.sh
ExecStart=/opt/myapp/server
ExecStartPost=/opt/myapp/post-start.sh
ExecReload=/bin/kill -HUP $MAINPID
ExecStop=/bin/kill -SIGTERM $MAINPID
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

### Key Directives

| Section | Directive | Purpose |
|---------|-----------|---------|
| `[Unit]` | `After` | Start after these units |
| `[Unit]` | `Requires` | Hard dependency (fail if dep fails) |
| `[Unit]` | `Wants` | Soft dependency (ok if dep fails) |
| `[Service]` | `Type` | simple, forking, oneshot, notify |
| `[Service]` | `ExecStart` | Command to run |
| `[Service]` | `Restart` | always, on-failure, on-abnormal |
| `[Service]` | `User/Group` | Run as this user/group |
| `[Install]` | `WantedBy` | Target to enable under |

### Service Types

| Type | Description |
|------|-------------|
| `simple` | Process stays in foreground (default) |
| `forking` | Process forks and parent exits (traditional daemons) |
| `oneshot` | Runs once and exits (scripts) |
| `notify` | Like simple but sends readiness notification |
| `idle` | Starts after all other jobs finish |

---

## Creating Custom Services

### Example: Node.js Application

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Node.js Application
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/node /opt/myapp/server.js
Restart=on-failure
RestartSec=10
Environment=PORT=3000
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

```bash
# Deploy
sudo systemctl daemon-reload         # Reload unit files
sudo systemctl enable --now myapp    # Enable and start
systemctl status myapp               # Check status
journalctl -u myapp -f               # Follow logs
```

### Example: Python Script Watchdog

```ini
# /etc/systemd/system/monitor.service
[Unit]
Description=System Monitor Script
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/python3 /opt/monitor/monitor.py
Restart=always
RestartSec=30
StandardOutput=append:/var/log/monitor.log
StandardError=append:/var/log/monitor-error.log

[Install]
WantedBy=multi-user.target
```

### Overriding Existing Services

```bash
# Create a drop-in override (don't edit original files!)
sudo systemctl edit nginx

# This creates: /etc/systemd/system/nginx.service.d/override.conf
# Add your overrides:
[Service]
LimitNOFILE=65536
```

---

## Targets (Runlevels)

Targets group units together — they define system states.

| Target | Old Runlevel | Description |
|--------|-------------|-------------|
| `poweroff.target` | 0 | Halt system |
| `rescue.target` | 1 | Single user mode |
| `multi-user.target` | 3 | CLI multi-user |
| `graphical.target` | 5 | GUI desktop |
| `reboot.target` | 6 | Reboot |
| `emergency.target` | — | Minimal shell |

```bash
systemctl get-default                         # Current default target
sudo systemctl set-default multi-user.target  # Boot to CLI
sudo systemctl set-default graphical.target   # Boot to GUI
sudo systemctl isolate rescue.target          # Switch to rescue NOW
```

---

## Journald (Logging)

```bash
# Configuration
sudo vim /etc/systemd/journald.conf

# Common settings:
[Journal]
Storage=persistent        # Store logs on disk (survives reboot)
SystemMaxUse=500M         # Max disk usage
MaxRetentionSec=1month    # Max retention
Compress=yes              # Compress logs

sudo systemctl restart systemd-journald
```

---

## Socket Activation

Start services **on-demand** when a connection arrives.

```ini
# /etc/systemd/system/myapp.socket
[Unit]
Description=My App Socket

[Socket]
ListenStream=8080

[Install]
WantedBy=sockets.target
```

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My App
Requires=myapp.socket

[Service]
ExecStart=/opt/myapp/server
```

```bash
sudo systemctl enable --now myapp.socket
# Service starts only when port 8080 receives a connection!
```

---

## Resource Control (cgroups)

Limit CPU, memory, and I/O per service.

```ini
# In the [Service] section:
[Service]
CPUQuota=50%               # Max 50% of one CPU
MemoryMax=512M             # Max 512 MB RAM
MemoryHigh=256M            # Throttle above 256 MB
IOWeight=100               # I/O priority (1-10000)
TasksMax=100               # Max 100 processes
```

```bash
# View resource usage per service
systemd-cgtop                # Top for cgroups

# View resource config
systemctl show nginx -p CPUQuota,MemoryMax,TasksMax

# Runtime override
sudo systemctl set-property nginx CPUQuota=25%
```

---

## 🏋️ Practice Exercises

1. **Create**: Write a custom service that runs a simple script on boot
2. **Status**: Check the status of sshd with `systemctl`
3. **Logs**: View the last 50 log entries for a specific service
4. **Target**: Check your current boot target and understand what it means
5. **Override**: Create a drop-in override for an existing service
6. **Resources**: Set memory limit on a service using cgroups
7. **Failed**: Find all failed services and investigate one
8. **Timer**: Create a timer that runs a cleanup script every 6 hours

---

<p align="center">
  <a href="../18-kernel-system-internals/README.md">← Previous: Kernel & System Internals</a> · <a href="../README.md">🏠 Home</a> · <a href="../20-advanced-networking/README.md">Next: Advanced Networking →</a>
</p>
