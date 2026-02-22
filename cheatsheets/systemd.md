# Systemd Cheatsheet

## Service Management
| Command | Description |
|---------|-------------|
| `systemctl start svc` | Start service |
| `systemctl stop svc` | Stop service |
| `systemctl restart svc` | Restart service |
| `systemctl reload svc` | Reload config |
| `systemctl enable svc` | Enable at boot |
| `systemctl disable svc` | Disable at boot |
| `systemctl enable --now svc` | Enable + start |
| `systemctl status svc` | Service status |
| `systemctl is-active svc` | Check if running |
| `systemctl is-enabled svc` | Check if enabled |
| `systemctl mask svc` | Prevent starting |
| `systemctl unmask svc` | Allow starting |

## Listing
| Command | Description |
|---------|-------------|
| `systemctl list-units --type=service` | Running services |
| `systemctl list-units --type=service --all` | All services |
| `systemctl list-unit-files` | All unit files |
| `systemctl --failed` | Failed services |
| `systemctl list-timers` | Active timers |

## Unit Files
| Command | Description |
|---------|-------------|
| `systemctl cat svc` | View unit file |
| `systemctl show svc` | All properties |
| `systemctl edit svc` | Create override |
| `systemctl edit --full svc` | Edit full file |
| `systemctl daemon-reload` | Reload unit files |

## Logs (journalctl)
| Command | Description |
|---------|-------------|
| `journalctl` | All logs |
| `journalctl -f` | Follow (live) |
| `journalctl -u svc` | Service logs |
| `journalctl -u svc -f` | Follow service |
| `journalctl -b` | Current boot |
| `journalctl -b -1` | Previous boot |
| `journalctl --since "1h ago"` | Last hour |
| `journalctl --since today` | Today |
| `journalctl -p err` | Errors only |
| `journalctl -k` | Kernel messages |
| `journalctl -xe` | Recent + explanations |
| `journalctl --disk-usage` | Log disk usage |
| `journalctl --vacuum-size=500M` | Trim logs |

## Targets
| Command | Description |
|---------|-------------|
| `systemctl get-default` | Default target |
| `systemctl set-default multi-user.target` | Boot to CLI |
| `systemctl set-default graphical.target` | Boot to GUI |
| `systemctl isolate rescue.target` | Rescue mode |

## System Control
| Command | Description |
|---------|-------------|
| `systemctl reboot` | Reboot |
| `systemctl poweroff` | Shutdown |
| `systemctl suspend` | Suspend |
| `systemctl hibernate` | Hibernate |
| `hostnamectl set-hostname name` | Set hostname |
| `timedatectl set-timezone Zone` | Set timezone |

## Resource Control
| Property | Description |
|----------|-------------|
| `CPUQuota=50%` | CPU limit |
| `MemoryMax=512M` | Memory limit |
| `TasksMax=100` | Process limit |
| `IOWeight=100` | I/O priority |
| `systemd-cgtop` | cgroup resource viewer |
