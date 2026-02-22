# ⚙️ Chapter 10: Process Management

<p align="center">
  <img src="https://img.shields.io/badge/Level-Intermediate-yellow?style=for-the-badge" alt="Intermediate">
  <img src="https://img.shields.io/badge/Chapter-10%20of%2034-blue?style=for-the-badge" alt="Chapter 10">
</p>

---

## 📑 Table of Contents

- [What is a Process?](#what-is-a-process)
- [Viewing Processes](#viewing-processes)
- [Process States](#process-states)
- [Foreground & Background](#foreground--background)
- [Signals & Killing Processes](#signals--killing-processes)
- [Process Priority (nice & renice)](#process-priority-nice--renice)
- [The /proc Filesystem](#the-proc-filesystem)
- [System Resource Monitoring](#system-resource-monitoring)
- [Practice Exercises](#-practice-exercises)

---

## What is a Process?

A **process** is a running instance of a program. Every process has:

| Attribute | Description | Example |
|-----------|-------------|---------|
| **PID** | Process ID (unique number) | `1234` |
| **PPID** | Parent Process ID | `1` (init/systemd) |
| **UID** | User who owns the process | `1000` (sovon) |
| **State** | Current status | Running, Sleeping |
| **Priority** | CPU scheduling priority | `20` (normal) |
| **Memory** | RAM usage | `15.2 MB` |

```
┌────────────────────────────────────┐
│ PID 1: systemd (init)              │
├──────┬─────────┬───────────────────┤
│ PID  │ PID     │ PID               │
│ 200  │ 300     │ 400               │
│sshd  │ cron    │ login             │
│      │         ├──────┬────────────┤
│      │         │ PID  │ PID        │
│      │         │ 500  │ 600        │
│      │         │ bash │ bash       │
│      │         │      ├─────┬──────┤
│      │         │      │ vim │ grep │
└──────┴─────────┴──────┴─────┴──────┘
```

---

## Viewing Processes

### `ps` — Process Snapshot

```bash
ps                              # Your processes in current terminal
ps aux                          # ALL processes, all users, detailed
ps -ef                          # ALL processes, full format
ps aux --sort=-%mem | head -10  # Top 10 memory-consuming processes
ps aux --sort=-%cpu | head -10  # Top 10 CPU-consuming processes
ps -u sovon                     # Processes owned by user sovon
ps -p 1234                      # Info about specific PID
ps -C nginx                     # Processes named "nginx"
ps --forest                     # Process tree view
ps -eo pid,ppid,user,%cpu,%mem,cmd --sort=-%cpu  # Custom columns
```

### `top` — Real-Time Monitor

```bash
top
```

**Top keyboard shortcuts:**

| Key | Action |
|-----|--------|
| `q` | Quit |
| `k` | Kill a process (enter PID) |
| `r` | Renice a process |
| `M` | Sort by memory |
| `P` | Sort by CPU |
| `T` | Sort by time |
| `1` | Show individual CPU cores |
| `c` | Show full command path |
| `h` | Help |

### `htop` — Better top

```bash
sudo apt install htop
htop
```

`htop` provides a colorful, interactive process viewer with:
- CPU/memory bars
- Mouse support
- Tree view (F5)
- Filter (F4)
- Search (F3)
- Sort (F6)
- Kill (F9)

### `pgrep` & `pidof`

```bash
pgrep nginx                 # Find PID by name
pgrep -u sovon               # PIDs of user's processes
pgrep -a bash                # PID + full command line
pidof nginx                  # PID of exact program name
```

---

## Process States

| State | Symbol | Description |
|-------|--------|-------------|
| **Running** | `R` | Currently executing on CPU |
| **Sleeping** | `S` | Waiting for event (interruptible) |
| **Disk Sleep** | `D` | Waiting for I/O (uninterruptible) |
| **Stopped** | `T` | Suspended (Ctrl+Z or signal) |
| **Zombie** | `Z` | Finished but parent hasn't collected exit status |
| **Dead** | `X` | Should never be seen |

```bash
# Find zombie processes
ps aux | awk '$8 == "Z" {print}'

# Find and kill zombie parent
ps -eo pid,ppid,stat,cmd | grep "Z"
# Then kill the PARENT process (PPID) to clean zombies
```

---

## Foreground & Background

```bash
# Run in foreground (default)
sleep 100                    # Blocks your terminal

# Suspend a foreground process
Ctrl + Z                    # Sends SIGTSTP

# Resume in background
bg                           # Continue in background
bg %1                        # Resume job #1 in background

# Resume in foreground
fg                           # Bring to foreground
fg %2                        # Bring job #2 to foreground

# Start directly in background
sleep 100 &                  # The & runs it in background
echo $!                      # PID of last background process

# List background jobs
jobs                         # List all jobs
jobs -l                      # Include PIDs

# Disown a job (keeps running after logout)
sleep 1000 &
disown %1                    # Detach from terminal
disown -a                    # Disown all jobs

# nohup — survive terminal close
nohup long-running-command &
# Output goes to nohup.out
```

---

## Signals & Killing Processes

### Common Signals

| Signal | Number | Description | Default Action |
|--------|--------|-------------|---------------|
| `SIGHUP` | 1 | Terminal hangup | Terminate |
| `SIGINT` | 2 | Interrupt (Ctrl+C) | Terminate |
| `SIGQUIT` | 3 | Quit (Ctrl+\\) | Terminate + core dump |
| `SIGKILL` | 9 | Force kill | Cannot be caught! |
| `SIGTERM` | 15 | Graceful termination | Terminate |
| `SIGTSTP` | 20 | Stop (Ctrl+Z) | Suspend |
| `SIGCONT` | 18 | Continue | Resume |
| `SIGUSR1` | 10 | User-defined | Varies |

### Sending Signals

```bash
# kill (sends signal to process)
kill 1234                    # Send SIGTERM (graceful)
kill -9 1234                 # Send SIGKILL (force)
kill -SIGTERM 1234           # Same as kill 1234
kill -l                      # List all signals

# killall (by name)
killall nginx                # Kill all nginx processes
killall -9 firefox           # Force kill all firefox

# pkill (by pattern)
pkill -f "python server"     # Kill by command pattern
pkill -u alice               # Kill all of alice's processes
pkill -9 -t pts/0            # Kill all processes on terminal pts/0

# Best practice: always try graceful first
kill PID                     # Try SIGTERM first
sleep 5                      # Wait 5 seconds
kill -9 PID                  # Force kill if still running
```

### The Kill Escalation Path

```
1. kill PID          (SIGTERM — polite request to stop)
2. kill -2 PID       (SIGINT — like Ctrl+C)
3. kill -1 PID       (SIGHUP — reload or die)
4. kill -9 PID       (SIGKILL — instant death, cannot be caught)
```

> ⚠️ **SIGKILL (-9) should be a last resort.** It doesn't let the process clean up (flush buffers, close files, remove temp files).

---

## Process Priority (nice & renice)

Priority ranges from **-20 (highest)** to **19 (lowest)**. Default is **0**.

```bash
# Start a process with a specific priority
nice -n 10 ./heavy-computation.sh    # Lower priority (nice)
nice -n -10 ./important-task.sh      # Higher priority (need sudo for negative)
sudo nice -n -20 ./critical.sh       # Highest priority

# Change priority of running process
renice 15 -p 1234               # Set PID 1234 to priority 15
renice -5 -p 1234               # Set to -5 (need sudo for negative)
sudo renice -20 -u mysql        # Set all mysql processes to max priority

# View priorities
ps -eo pid,ni,cmd | head -20    # Show nice values
top                              # NI column shows nice value
```

---

## The /proc Filesystem

Every running process has a directory in `/proc`:

```bash
# Info about PID 1234
ls /proc/1234/
cat /proc/1234/status           # Process status
cat /proc/1234/cmdline          # Full command line
cat /proc/1234/environ          # Environment variables
ls -l /proc/1234/fd/            # Open file descriptors
ls -l /proc/1234/cwd            # Current working directory
cat /proc/1234/maps             # Memory mappings
cat /proc/1234/io               # I/O statistics
```

### System-Wide /proc Files

```bash
cat /proc/cpuinfo               # CPU information
cat /proc/meminfo               # Memory information
cat /proc/loadavg               # Load averages
cat /proc/uptime                # System uptime
cat /proc/version               # Kernel version
cat /proc/filesystems           # Supported filesystems
cat /proc/mounts                # Mounted filesystems
cat /proc/net/dev               # Network interface stats
```

---

## System Resource Monitoring

### Memory

```bash
free -h                          # Quick memory overview
# Output:
#               total    used    free   shared  buff/cache  available
# Mem:          16Gi    8.2Gi   2.1Gi   512Mi    5.7Gi      7.1Gi
# Swap:          4Gi    256Mi   3.7Gi

vmstat 1 5                       # Virtual memory stats (every 1s, 5 times)
```

### CPU

```bash
uptime                           # Load averages
# 10:00:00 up 5 days, 3:22, 2 users, load average: 0.52, 0.48, 0.45
# Load average: 1 min, 5 min, 15 min
# < number of CPUs = not overloaded

nproc                            # Number of CPU cores
lscpu                            # Detailed CPU info
mpstat 1 5                       # Per-CPU stats (need sysstat package)
```

### Disk I/O

```bash
iostat 1 5                       # Disk I/O stats
iotop                            # Top for disk I/O (needs root)
```

### Open Files & Connections

```bash
lsof                             # List all open files
lsof -u sovon                    # Open files by user
lsof -p 1234                     # Open files by PID
lsof -i :80                      # Who's using port 80?
lsof -i tcp                      # All TCP connections
lsof +D /var/log                 # Open files in directory
```

---

## 🏋️ Practice Exercises

1. **List** all running processes and sort by memory usage
2. **Background**: Start `sleep 300`, suspend it, move to background, then bring back
3. **Signals**: Start `sleep 1000 &`, then kill it gracefully with SIGTERM
4. **Priority**: Start a process with nice value 10, then renice to 5
5. **Monitor**: Use `htop` to find the most CPU-intensive process
6. **Proc**: Look at your current shell's `/proc/<PID>/` directory
7. **Find**: Use `lsof` to find which process is using port 22
8. **Zombie**: Write a script that creates a zombie process (hint: fork without wait)

---

<p align="center">
  <a href="../09-shell-scripting-fundamentals/README.md">← Previous: Shell Scripting</a> · <a href="../README.md">🏠 Home</a> · <a href="../11-disk-storage-management/README.md">Next: Disk & Storage Management →</a>
</p>
