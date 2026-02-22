# ⚡ Chapter 23: Performance Tuning

<p align="center">
  <img src="https://img.shields.io/badge/Level-Advanced-orange?style=for-the-badge" alt="Advanced">
  <img src="https://img.shields.io/badge/Chapter-23%20of%2034-blue?style=for-the-badge" alt="Chapter 23">
</p>

---

## 📑 Table of Contents

- [Performance Analysis Methodology](#performance-analysis-methodology)
- [CPU Tuning](#cpu-tuning)
- [Memory Tuning](#memory-tuning)
- [I/O Tuning](#io-tuning)
- [Network Tuning](#network-tuning)
- [cgroups v2 — Resource Limits](#cgroups-v2--resource-limits)
- [tuned — Automatic Profiles](#tuned--automatic-profiles)
- [Kernel Parameters for Performance](#kernel-parameters-for-performance)
- [Practice Exercises](#-practice-exercises)

---

## Performance Analysis Methodology

**USE Method** — for every resource, check:
- **U**tilization — How busy is it?
- **S**aturation — Are there queues? Is work waiting?
- **E**rrors — Are there errors?

| Resource | Utilization | Saturation | Errors |
|----------|------------|------------|--------|
| **CPU** | `top`, `mpstat` | `vmstat r > cores` | `dmesg`, `perf` |
| **Memory** | `free`, `vmstat` | `vmstat si/so`, OOM kills | `dmesg` |
| **Disk** | `iostat %util` | `iostat await` | `smartctl`, `dmesg` |
| **Network** | `sar -n DEV` | `ss -s`, drops | `ip -s link`, `dmesg` |

---

## CPU Tuning

### CPU Governors

```bash
# View current governor
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# List available governors
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors
# conservative ondemand userspace powersave performance schedutil

# Set governor
echo "performance" | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# Permanent with cpupower
sudo apt install linux-tools-common
sudo cpupower frequency-set -g performance
sudo cpupower frequency-info
```

| Governor | Behavior |
|----------|----------|
| `performance` | Always max frequency (best throughput) |
| `powersave` | Always min frequency (best battery) |
| `ondemand` | Scale up quickly on load |
| `schedutil` | Kernel scheduler decides (default) |

### CPU Affinity

```bash
# Pin process to specific CPUs
taskset -c 0,1 ./myprocess           # CPUs 0 and 1
taskset -cp 0,1 1234                  # Change running process
taskset -p 1234                       # View current affinity

# Using cgroups
sudo cgcreate -g cpu:myapp
sudo cgset -r cpuset.cpus=0-3 myapp
sudo cgexec -g cpu:myapp ./myprocess
```

---

## Memory Tuning

### Swappiness

```bash
# Lower = prefer RAM, higher = prefer swap
cat /proc/sys/vm/swappiness           # Default: 60
sudo sysctl vm.swappiness=10          # Set for SSD servers

# vm.vfs_cache_pressure
sudo sysctl vm.vfs_cache_pressure=50  # Default: 100
# Lower = keep dentries/inodes in cache longer
```

### Hugepages

```bash
# For databases and large-memory applications
cat /proc/meminfo | grep Huge

# Allocate 1024 hugepages (2MB each = 2GB)
echo 1024 | sudo tee /proc/sys/vm/nr_hugepages
# Permanent
echo "vm.nr_hugepages = 1024" | sudo tee -a /etc/sysctl.conf

# Transparent Hugepages (THP)
cat /sys/kernel/mm/transparent_hugepage/enabled
# Options: [always] madvise never
echo "madvise" | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
```

### OOM Killer Tuning

```bash
# View OOM score (higher = more likely to be killed)
cat /proc/$(pidof mysql)/oom_score
cat /proc/$(pidof mysql)/oom_score_adj

# Protect a process from OOM killer
echo -1000 | sudo tee /proc/$(pidof mysql)/oom_score_adj
```

---

## I/O Tuning

### I/O Schedulers

```bash
# View current scheduler
cat /sys/block/sda/queue/scheduler
# [mq-deadline] kyber bfq none

# Change scheduler
echo "kyber" | sudo tee /sys/block/sda/queue/scheduler
```

| Scheduler | Best For |
|-----------|----------|
| `mq-deadline` | General purpose, databases (default) |
| `kyber` | Fast NVMe SSDs |
| `bfq` | Desktop interactive, USB drives |
| `none` | NVMe with hardware queues |

### Read-Ahead

```bash
# View/set read-ahead (KB)
blockdev --getra /dev/sda             # Current value
sudo blockdev --setra 2048 /dev/sda   # Set to 2048 sectors
```

### Filesystem Mount Options

```bash
# /etc/fstab optimizations
UUID=xxx /     ext4  noatime,nodiratime,discard  0 1
#  noatime     = don't update access times (big performance gain)
#  nodiratime  = don't update directory access times
#  discard     = enable TRIM for SSD (or use fstrim.timer)

# Enable periodic TRIM
sudo systemctl enable fstrim.timer
```

---

## Network Tuning

```bash
# Key sysctl network parameters
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535
sudo sysctl -w net.core.rmem_max=16777216
sudo sysctl -w net.core.wmem_max=16777216
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 16777216"
sudo sysctl -w net.ipv4.tcp_wmem="4096 87380 16777216"
sudo sysctl -w net.ipv4.tcp_fin_timeout=15
sudo sysctl -w net.ipv4.tcp_tw_reuse=1
sudo sysctl -w net.core.netdev_max_backlog=5000
```

---

## cgroups v2 — Resource Limits

```bash
# Check if cgroups v2 is enabled
mount | grep cgroup2

# Create a cgroup
sudo mkdir /sys/fs/cgroup/mygroup

# Set limits
echo "500000 1000000" | sudo tee /sys/fs/cgroup/mygroup/cpu.max  # 50% CPU
echo "512M" | sudo tee /sys/fs/cgroup/mygroup/memory.max         # 512MB RAM
echo "100" | sudo tee /sys/fs/cgroup/mygroup/io.weight            # I/O weight

# Add process to cgroup
echo $PID | sudo tee /sys/fs/cgroup/mygroup/cgroup.procs

# With systemd (easier)
sudo systemctl set-property myservice CPUQuota=50%
sudo systemctl set-property myservice MemoryMax=512M
```

---

## tuned — Automatic Profiles

```bash
sudo apt install tuned
sudo systemctl enable --now tuned

# List profiles
tuned-adm list

# Apply profile
sudo tuned-adm profile throughput-performance

# Recommended
tuned-adm recommend
```

| Profile | Use Case |
|---------|----------|
| `balanced` | Default |
| `throughput-performance` | High throughput servers |
| `latency-performance` | Low latency |
| `network-latency` | Network-sensitive apps |
| `virtual-guest` | Running as VM |
| `desktop` | Desktop responsiveness |
| `powersave` | Laptop battery |

---

## Kernel Parameters for Performance

```bash
# /etc/sysctl.d/99-performance.conf
vm.swappiness = 10
vm.vfs_cache_pressure = 50
vm.dirty_ratio = 20
vm.dirty_background_ratio = 5
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
fs.file-max = 2097152
fs.inotify.max_user_watches = 524288

# Reload
sudo sysctl --system
```

### File Descriptor Limits

```bash
# View current limits
ulimit -a
ulimit -n                            # Open files limit

# Increase permanently
sudo vim /etc/security/limits.conf
# *    soft    nofile    65536
# *    hard    nofile    65536
```

---

## 🏋️ Practice Exercises

1. **CPU**: Change your CPU governor to `performance` and back
2. **Memory**: Tune swappiness to 10 and measure the effect
3. **I/O**: Check and change your disk scheduler
4. **cgroups**: Create a cgroup that limits a process to 50% CPU
5. **tuned**: Install tuned and apply `throughput-performance` profile
6. **Limits**: Increase the file descriptor limit to 65536
7. **noatime**: Add `noatime` to a mount and benchmark reads
8. **Monitor**: Use the USE method to analyze your system resources

---

<p align="center">
  <a href="../22-selinux-apparmor/README.md">← Previous: SELinux & AppArmor</a> · <a href="../README.md">🏠 Home</a> · <a href="../24-lvm-raid/README.md">Next: LVM & RAID →</a>
</p>
