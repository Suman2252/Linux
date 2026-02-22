<p align="center">
  <img src="https://img.shields.io/badge/OS-Linux-informational?style=for-the-badge&logo=linux&logoColor=white&color=FCC624" alt="Linux">
  <img src="https://img.shields.io/badge/Shell-Bash-informational?style=for-the-badge&logo=gnu-bash&logoColor=white&color=4EAA25" alt="Bash">
  <img src="https://img.shields.io/badge/License-MIT-informational?style=for-the-badge&color=blue" alt="MIT License">
  <img src="https://img.shields.io/badge/PRs-Welcome-informational?style=for-the-badge&color=brightgreen" alt="PRs Welcome">
</p>

<h1 align="center">🐧 The Ultimate Linux Guide</h1>

<p align="center">
  <strong>From Zero to Kernel Hacker — The most comprehensive Linux resource on GitHub.</strong>
</p>

<p align="center">
  <em>34 in-depth chapters · 6 cheatsheets · 200+ commands · Beginner → Expert</em>
</p>

---

## 🗺️ Roadmap

```
 YOU ARE HERE
      │
      ▼
┌─────────────┐     ┌──────────────────┐     ┌─────────────────┐     ┌──────────────────┐
│  🌱 BEGINNER │────▶│  🌿 INTERMEDIATE  │────▶│  🔥 ADVANCED     │────▶│  🧠 EXPERT        │
│  Ch. 01 – 08 │     │  Ch. 09 – 16      │     │  Ch. 17 – 24     │     │  Ch. 25 – 34      │
└─────────────┘     └──────────────────┘     └─────────────────┘     └──────────────────┘
 Terminal, files,     Scripting, SSH,         Kernel, security,       Containers, K8s,
 permissions,         networking, cron,       systemd, firewalls,     eBPF, LFS,
 packages, editors    processes, backups      performance, LVM        virtualization
```

---

## 📖 Table of Contents

### 🌱 Beginner

| # | Topic | What You'll Learn |
|---|-------|-------------------|
| 01 | [Introduction to Linux](./01-introduction/README.md) | History, distros, open-source philosophy |
| 02 | [Installation & Setup](./02-installation-setup/README.md) | Dual boot, VMs, WSL, cloud instances |
| 03 | [Terminal Basics](./03-terminal-basics/README.md) | Shell, prompt, navigation, man pages |
| 04 | [Filesystem Hierarchy](./04-filesystem-hierarchy/README.md) | FHS, key directories, everything is a file |
| 05 | [File & Directory Operations](./05-file-directory-operations/README.md) | cp, mv, rm, find, links, wildcards |
| 06 | [Users, Groups & Permissions](./06-users-groups-permissions/README.md) | chmod, chown, SUID/SGID, ACLs |
| 07 | [Package Management](./07-package-management/README.md) | apt, dnf, pacman, snap, flatpak, source |
| 08 | [Text Editors](./08-text-editors/README.md) | nano, vim, neovim essentials |

### 🌿 Intermediate

| # | Topic | What You'll Learn |
|---|-------|-------------------|
| 09 | [Shell Scripting Fundamentals](./09-shell-scripting-fundamentals/README.md) | Variables, loops, functions, exit codes |
| 10 | [Process Management](./10-process-management/README.md) | ps, top, kill, signals, bg/fg, /proc |
| 11 | [Disk & Storage Management](./11-disk-storage-management/README.md) | fdisk, parted, mkfs, mount, fstab |
| 12 | [Networking Fundamentals](./12-networking-fundamentals/README.md) | ip, ss, ping, DNS, NetworkManager |
| 13 | [SSH & Remote Access](./13-ssh-remote-access/README.md) | Keys, config, tunnels, SCP, rsync |
| 14 | [Cron Jobs & Scheduling](./14-cron-jobs-scheduling/README.md) | cron, crontab, at, systemd timers |
| 15 | [System Monitoring & Logs](./15-system-monitoring-logs/README.md) | journalctl, syslog, dmesg, sar |
| 16 | [Archive, Compression & Backup](./16-archive-compression-backup/README.md) | tar, gzip, xz, rsync, borgbackup |

### 🔥 Advanced

| # | Topic | What You'll Learn |
|---|-------|-------------------|
| 17 | [Advanced Shell Scripting](./17-advanced-shell-scripting/README.md) | Regex, sed, awk, xargs, traps |
| 18 | [Kernel & System Internals](./18-kernel-system-internals/README.md) | Kernel architecture, boot, initramfs |
| 19 | [Systemd & Service Management](./19-systemd-service-management/README.md) | Units, targets, journald, custom services |
| 20 | [Advanced Networking](./20-advanced-networking/README.md) | iptables, nftables, VLANs, WireGuard |
| 21 | [Security & Hardening](./21-security-hardening/README.md) | Firewalls, fail2ban, CIS benchmarks |
| 22 | [SELinux & AppArmor](./22-selinux-apparmor/README.md) | MAC policies, troubleshooting |
| 23 | [Performance Tuning](./23-performance-tuning/README.md) | CPU governors, cgroups, hugepages |
| 24 | [LVM & RAID](./24-lvm-raid/README.md) | Logical volumes, snapshots, mdadm |

### 🧠 Expert

| # | Topic | What You'll Learn |
|---|-------|-------------------|
| 25 | [Kernel Compilation & Modules](./25-kernel-compilation-modules/README.md) | Build kernels, DKMS, write modules |
| 26 | [Containers & Docker](./26-containers-docker/README.md) | Namespaces, cgroups, Docker, Podman |
| 27 | [Kubernetes on Linux](./27-kubernetes-on-linux/README.md) | kubeadm, pods, services, Helm |
| 28 | [Advanced Filesystems](./28-advanced-filesystems/README.md) | Btrfs, ZFS, ext4 internals, XFS |
| 29 | [Virtualization](./29-virtualization/README.md) | KVM, QEMU, libvirt, cloud-init |
| 30 | [eBPF & System Tracing](./30-ebpf-tracing/README.md) | bpftrace, BCC, perf, ftrace, strace |
| 31 | [Linux From Scratch](./31-linux-from-scratch/README.md) | Build your own Linux from source |
| 32 | [Embedded Linux](./32-embedded-linux/README.md) | Yocto, Buildroot, device trees |
| 33 | [High Availability & Clustering](./33-high-availability-clustering/README.md) | Pacemaker, HAProxy, keepalived |
| 34 | [Troubleshooting & Disaster Recovery](./34-troubleshooting-disaster-recovery/README.md) | Boot rescue, fsck, grub repair |

### 📋 Quick References

| Resource | Description |
|----------|-------------|
| [📝 Commands Cheatsheet](./cheatsheets/commands.md) | Essential commands at a glance |
| [⌨️ Vim Cheatsheet](./cheatsheets/vim.md) | Vim modes, motions, and commands |
| [🔤 Bash Shortcuts](./cheatsheets/bash-shortcuts.md) | Terminal keyboard shortcuts |
| [🔍 Regex Reference](./cheatsheets/regex.md) | Regular expressions explained |
| [🌐 Networking Cheatsheet](./cheatsheets/networking.md) | Networking commands reference |
| [🔒 Permissions Cheatsheet](./cheatsheets/permissions.md) | File permissions quick guide |
| [📚 Resources & Learning](./resources/README.md) | Books, courses, certifications |
| [📖 Glossary](./glossary/README.md) | A–Z Linux terminology |

---

## 🚀 How to Use This Repository

1. **Complete Beginners** — Start at [Chapter 01](./01-introduction/README.md) and work through sequentially.
2. **Some Experience** — Jump to the topic you need. Each chapter is self-contained.
3. **Quick Reference** — Use the [cheatsheets](./cheatsheets/) for fast lookups.
4. **Interview Prep** — Focus on chapters 06, 10, 12, 17, 18, 21 for common interview topics.

> 💡 **Tip**: Star ⭐ this repo so you can find it later. Share it with anyone learning Linux!

---

## 🤝 Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](./CONTRIBUTING.md) before submitting a pull request.

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](./LICENSE) file for details.

---

<p align="center">
  Made with ❤️ for the Linux community.<br>
  <strong>If this helped you, please ⭐ star this repository!</strong>
</p>
