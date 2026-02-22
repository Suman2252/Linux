# 🔬 Chapter 30: eBPF & Advanced Tracing

<p align="center">
  <img src="https://img.shields.io/badge/Level-Expert-red?style=for-the-badge" alt="Expert">
  <img src="https://img.shields.io/badge/Chapter-30%20of%2034-blue?style=for-the-badge" alt="Chapter 30">
</p>

---

## 📑 Table of Contents

- [What is eBPF?](#what-is-ebpf)
- [BCC Tools](#bcc-tools)
- [bpftrace — One-Liners](#bpftrace--one-liners)
- [perf — Performance Counters](#perf--performance-counters)
- [ftrace — Kernel Tracer](#ftrace--kernel-tracer)
- [Flame Graphs](#flame-graphs)
- [Practice Exercises](#-practice-exercises)

---

## What is eBPF?

**eBPF** (extended Berkeley Packet Filter) lets you run sandboxed programs inside the Linux kernel — without modifying kernel code or loading modules.

```
User Space          Kernel Space
┌──────────┐       ┌───────────────────────┐
│          │       │                       │
│  Your    │──────▶│  eBPF Verifier        │
│  Program │       │       ↓               │
│          │       │  eBPF JIT Compiler    │
│          │       │       ↓               │
│          │◀──────│  eBPF Program runs    │
│  (maps)  │       │  in kernel safely     │
│          │       │                       │
└──────────┘       └───────────────────────┘
```

**Use cases**: Networking, security, observability, tracing, profiling.

---

## BCC Tools

BCC (BPF Compiler Collection) provides ready-made eBPF tools.

```bash
# Install
sudo apt install bpfcc-tools linux-headers-$(uname -r)

# Available tools (in /usr/sbin/ or /usr/share/bcc/tools/)
execsnoop-bpfcc          # Trace new process execution
opensnoop-bpfcc          # Trace file opens
biolatency-bpfcc         # Block I/O latency histogram
tcpconnect-bpfcc         # Trace TCP connections
tcplife-bpfcc            # TCP session lifetimes
cachestat-bpfcc          # Page cache hit/miss stats
filetop-bpfcc            # Top files by I/O
runqlat-bpfcc            # CPU run queue latency
bashreadline-bpfcc       # Sniff bash input (security auditing)
```

### Example Usage

```bash
# Watch all new process executions
sudo execsnoop-bpfcc

# Watch file opens
sudo opensnoop-bpfcc
sudo opensnoop-bpfcc -p 1234           # Specific PID

# Disk I/O latency histogram
sudo biolatency-bpfcc -D

# TCP connections
sudo tcpconnect-bpfcc
sudo tcplife-bpfcc                      # With duration

# File system I/O top
sudo filetop-bpfcc
```

---

## bpftrace — One-Liners

```bash
sudo apt install bpftrace

# Trace syscalls by process
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* { @[comm] = count(); }'

# Trace file opens
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args->filename)); }'

# Block I/O sizes
sudo bpftrace -e 'tracepoint:block:block_rq_issue { @bytes = hist(args->bytes); }'

# System call count by type
sudo bpftrace -e 'tracepoint:raw_syscalls:sys_enter { @[probe] = count(); }'

# Process CPU time
sudo bpftrace -e 'profile:hz:99 { @[comm] = count(); }'

# List available tracepoints
sudo bpftrace -l 'tracepoint:*' | head -50
sudo bpftrace -l 'kprobe:*tcp*'
```

---

## perf — Performance Counters

```bash
sudo apt install linux-tools-$(uname -r)

# CPU profiling
sudo perf stat ls                       # Basic event stats
sudo perf stat -d ./myprogram           # Detailed

# Record profile
sudo perf record -g ./myprogram         # With call graphs
sudo perf report                         # View results

# Top-like view
sudo perf top                            # Live CPU sampling

# System-wide for 10 seconds
sudo perf record -a -g -- sleep 10
sudo perf report

# Specific events
sudo perf stat -e cache-misses,cache-references ./myprogram
sudo perf stat -e cycles,instructions,branches,branch-misses ./myprogram

# Trace specific functions
sudo perf probe --add tcp_sendmsg
sudo perf record -e probe:tcp_sendmsg -a -- sleep 5
```

---

## ftrace — Kernel Tracer

```bash
# ftrace lives in /sys/kernel/debug/tracing/
cd /sys/kernel/debug/tracing

# Available tracers
cat available_tracers
# nop function function_graph

# Function tracer
echo function > current_tracer
echo 1 > tracing_on
cat trace | head -50
echo 0 > tracing_on

# Function graph (with call depth)
echo function_graph > current_tracer
echo 1 > tracing_on
cat trace | head -50
echo 0 > tracing_on

# Filter specific functions
echo 'tcp_*' > set_ftrace_filter
echo function > current_tracer
echo 1 > tracing_on

# Using trace-cmd (easier)
sudo apt install trace-cmd
sudo trace-cmd record -p function_graph -l 'tcp_*' sleep 5
sudo trace-cmd report | head -100
```

---

## Flame Graphs

Visualize performance profiles.

```bash
# Generate flame graph
git clone https://github.com/brendangregg/FlameGraph
sudo perf record -a -g -- sleep 30
sudo perf script > out.perf
./FlameGraph/stackcollapse-perf.pl out.perf > out.folded
./FlameGraph/flamegraph.pl out.folded > flamegraph.svg

# Open in browser
firefox flamegraph.svg
```

---

## 🏋️ Practice Exercises

1. **execsnoop**: Watch all new processes being created
2. **opensnoop**: Trace which files a specific program opens
3. **biolatency**: Generate a disk I/O latency histogram
4. **perf stat**: Profile a command and analyze cache misses
5. **perf top**: Identify the hottest functions on your system
6. **bpftrace**: Write a one-liner to count syscalls by process
7. **Flame graph**: Generate a flame graph of your system's CPU usage

---

<p align="center">
  <a href="../29-virtualization-kvm/README.md">← Previous: Virtualization & KVM</a> · <a href="../README.md">🏠 Home</a> · <a href="../31-linux-from-scratch/README.md">Next: Linux From Scratch →</a>
</p>
