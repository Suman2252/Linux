# ⏰ Chapter 14: Cron Jobs & Task Scheduling

<p align="center">
  <img src="https://img.shields.io/badge/Level-Intermediate-yellow?style=for-the-badge" alt="Intermediate">
  <img src="https://img.shields.io/badge/Chapter-14%20of%2034-blue?style=for-the-badge" alt="Chapter 14">
</p>

---

## 📑 Table of Contents

- [What is Cron?](#what-is-cron)
- [Crontab Syntax](#crontab-syntax)
- [Managing Crontabs](#managing-crontabs)
- [Practical Examples](#practical-examples)
- [System Cron Directories](#system-cron-directories)
- [The `at` Command — One-Time Tasks](#the-at-command--one-time-tasks)
- [Systemd Timers](#systemd-timers)
- [Anacron — For Desktops/Laptops](#anacron--for-desktopslaptops)
- [Practice Exercises](#-practice-exercises)

---

## What is Cron?

**Cron** is a time-based job scheduler. It runs commands or scripts at specified intervals.

> 🏠 **Analogy**: Cron is like setting an alarm clock — but instead of waking you up, it runs a command.

---

## Crontab Syntax

```
┌───────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌───────────── day of month (1-31)
│ │ │ ┌───────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0-7, 0 and 7 = Sunday)
│ │ │ │ │
* * * * * command_to_execute
```

### Special Characters

| Character | Meaning | Example |
|-----------|---------|---------|
| `*` | Every value | `* * * * *` = every minute |
| `,` | Multiple values | `1,15,30` = at 1, 15, 30 |
| `-` | Range | `1-5` = 1 through 5 |
| `/` | Step | `*/15` = every 15 |
| `@reboot` | Run once at startup | `@reboot /script.sh` |

### Quick Reference

| Schedule | Cron Expression | Description |
|----------|----------------|-------------|
| Every minute | `* * * * *` | Runs every minute |
| Every hour | `0 * * * *` | At minute 0 of every hour |
| Every day at midnight | `0 0 * * *` | 12:00 AM daily |
| Every day at 6 AM | `0 6 * * *` | 6:00 AM daily |
| Every Monday | `0 0 * * 1` | Monday at midnight |
| Every weekday | `0 9 * * 1-5` | Mon-Fri at 9 AM |
| Every 15 minutes | `*/15 * * * *` | :00, :15, :30, :45 |
| First of month | `0 0 1 * *` | Midnight, 1st day |
| Every 6 hours | `0 */6 * * *` | 00:00, 06:00, 12:00, 18:00 |

### Short Names

| Shortcut | Equivalent |
|----------|-----------|
| `@yearly` | `0 0 1 1 *` |
| `@monthly` | `0 0 1 * *` |
| `@weekly` | `0 0 * * 0` |
| `@daily` | `0 0 * * *` |
| `@hourly` | `0 * * * *` |
| `@reboot` | At system startup |

---

## Managing Crontabs

```bash
crontab -e                   # Edit your crontab
crontab -l                   # List your crontab
crontab -r                   # Remove your crontab (careful!)
crontab -u alice -l           # List alice's crontab (root only)
sudo crontab -e              # Edit root's crontab
```

---

## Practical Examples

```bash
# Edit crontab
crontab -e

# Backup home directory every day at 2 AM
0 2 * * * tar -czf /backup/home-$(date +\%Y\%m\%d).tar.gz /home/sovon

# Check disk space every hour, alert if > 90%
0 * * * * df -h | awk '$5 > 90 {print}' | mail -s "Disk Alert" admin@example.com

# Rotate logs every Sunday at 3 AM
0 3 * * 0 /usr/sbin/logrotate /etc/logrotate.conf

# Update packages every day at 4 AM
0 4 * * * apt update && apt upgrade -y >> /var/log/auto-update.log 2>&1

# Clean /tmp every day at midnight
0 0 * * * find /tmp -type f -mtime +7 -delete

# Send a report every Friday at 5 PM
0 17 * * 5 /home/sovon/scripts/weekly-report.sh

# Run at reboot
@reboot /home/sovon/scripts/startup.sh
```

### Logging Cron Output

```bash
# Log stdout and stderr to file
0 * * * * /script.sh >> /var/log/myjob.log 2>&1

# Discard all output
0 * * * * /script.sh > /dev/null 2>&1

# Send output as email (if mail is configured)
MAILTO="sovon@example.com"
0 6 * * * /script.sh
```

### Environment in Cron

```bash
# Cron runs with a minimal environment! Set paths explicitly:
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
SHELL=/bin/bash
HOME=/home/sovon
MAILTO=""

# Or use full paths in your commands
0 * * * * /usr/bin/python3 /home/sovon/scripts/myscript.py
```

> ⚠️ **Common Gotcha**: Cron doesn't load your `.bashrc`. Commands may fail because the `PATH` is different. Always use full paths!

---

## System Cron Directories

```bash
# Scripts placed here run automatically:
/etc/cron.d/           # Custom cron files
/etc/cron.hourly/      # Run every hour
/etc/cron.daily/       # Run every day
/etc/cron.weekly/      # Run every week
/etc/cron.monthly/     # Run every month

# Just drop an executable script in the directory:
sudo cp my-backup.sh /etc/cron.daily/
sudo chmod +x /etc/cron.daily/my-backup.sh

# System-wide crontab
cat /etc/crontab
```

---

## The `at` Command — One-Time Tasks

`at` schedules a command to run **once** at a specific time.

```bash
sudo apt install at

# Schedule for a specific time
at 10:00 PM
> /home/sovon/scripts/cleanup.sh
> <Ctrl+D>

# Schedule relative to now
at now + 30 minutes
> echo "Reminder: meeting!" | mail -s "Meeting" sovon
> <Ctrl+D>

# Schedule for a specific date
at 2:00 PM Feb 25
> /backup/run.sh
> <Ctrl+D>

# List pending jobs
atq

# Remove a job
atrm 3              # Remove job #3

# View job details
at -c 3
```

---

## Systemd Timers

Modern alternative to cron with better logging and dependency management.

### Create a Timer

```ini
# /etc/systemd/system/backup.service
[Unit]
Description=Daily Backup

[Service]
Type=oneshot
ExecStart=/home/sovon/scripts/backup.sh
User=sovon
```

```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Run backup daily at 2 AM

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
# Enable and start the timer
sudo systemctl daemon-reload
sudo systemctl enable backup.timer
sudo systemctl start backup.timer

# Check timer status
systemctl list-timers
systemctl status backup.timer

# Manually trigger the service
sudo systemctl start backup.service
```

### Timer vs Cron

| Feature | Cron | Systemd Timer |
|---------|------|--------------|
| Logging | Manual | journalctl |
| Dependencies | None | Full systemd deps |
| Missed runs | Lost | `Persistent=true` |
| Resource control | None | cgroups |
| Randomized delay | None | `RandomizedDelaySec` |

---

## Anacron — For Desktops/Laptops

Anacron ensures daily/weekly/monthly jobs run even if the machine was off at the scheduled time.

```bash
cat /etc/anacrontab
# period  delay  job-identifier   command
1          5      cron.daily       run-parts /etc/cron.daily
7         10      cron.weekly      run-parts /etc/cron.weekly
@monthly  15      cron.monthly     run-parts /etc/cron.monthly
```

---

## 🏋️ Practice Exercises

1. **Cron**: Create a cron job that logs the date to a file every 5 minutes
2. **Schedule**: Set up a cron job to clean `/tmp` files older than 7 days at midnight
3. **at**: Schedule a one-time command using `at` for 10 minutes from now
4. **Timer**: Create a systemd timer that runs a script every hour
5. **List**: View all scheduled cron jobs and systemd timers
6. **Log**: Set up a cron job with proper output redirection
7. **Verify**: Check cron logs to confirm your job ran: `grep CRON /var/log/syslog`

---

<p align="center">
  <a href="../13-ssh-remote-access/README.md">← Previous: SSH & Remote Access</a> · <a href="../README.md">🏠 Home</a> · <a href="../15-system-monitoring-logs/README.md">Next: System Monitoring & Logs →</a>
</p>
