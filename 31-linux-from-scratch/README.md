# 🏗️ Chapter 31: Linux From Scratch (LFS)

<p align="center">
  <img src="https://img.shields.io/badge/Level-Expert-red?style=for-the-badge" alt="Expert">
  <img src="https://img.shields.io/badge/Chapter-31%20of%2034-blue?style=for-the-badge" alt="Chapter 31">
</p>

---

## 📑 Table of Contents

- [What is LFS?](#what-is-lfs)
- [Prerequisites](#prerequisites)
- [High-Level Build Process](#high-level-build-process)
- [Host System Preparation](#host-system-preparation)
- [Building the Toolchain](#building-the-toolchain)
- [Building the Base System](#building-the-base-system)
- [System Configuration](#system-configuration)
- [Making it Bootable](#making-it-bootable)
- [Beyond LFS (BLFS)](#beyond-lfs-blfs)
- [Practice Exercises](#-practice-exercises)

---

## What is LFS?

**Linux From Scratch** is building a complete Linux system entirely from source code. It's the deepest way to understand exactly how Linux works.

> 📘 Official guide: [linuxfromscratch.org](https://www.linuxfromscratch.org/)

### Why Build LFS?

| Reason | Description |
|--------|-------------|
| **Learning** | Understand every component |
| **Customization** | Include only what you need |
| **Security** | Know exactly what's installed |
| **Size** | Minimal system (can be < 100MB) |
| **Performance** | Compile for your exact hardware |

---

## Prerequisites

```bash
# You need:
# 1. A working Linux host system (Ubuntu, Fedora, etc.)
# 2. ~30 GB free disk space
# 3. Several hours of build time
# 4. Patience!

# Check host requirements
cat > version-check.sh << "EOF"
#!/bin/bash
export LC_ALL=C
bash --version | head -n1
echo "/bin/sh -> $(readlink -f /bin/sh)"
echo -n "Coreutils: "; ls --version | head -n1
echo -n "GCC: "; gcc --version | head -n1
echo -n "Make: "; make --version | head -n1
echo -n "Kernel: "; uname -r
EOF
bash version-check.sh
```

---

## High-Level Build Process

```
Phase 1: PREPARATION
  → Partition disk
  → Set up build environment
  → Download source packages

Phase 2: TEMPORARY TOOLCHAIN (Cross-compilation)
  → Build cross-compiler (binutils + GCC)
  → Build temporary system tools
  → Enter chroot environment

Phase 3: BASE SYSTEM
  → Build Glibc (C library)
  → Build final GCC
  → Build ~80 core packages
  → Configure system

Phase 4: BOOT
  → Build Linux kernel
  → Install GRUB
  → Reboot into your own system!
```

---

## Host System Preparation

```bash
# Create partition (assuming /dev/sdb)
sudo fdisk /dev/sdb
# Create: sdb1 (512M boot), sdb2 (20G+ root), sdb3 (2G swap)

# Format
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.ext4 /dev/sdb2
sudo mkswap /dev/sdb3

# Mount
export LFS=/mnt/lfs
sudo mkdir -pv $LFS
sudo mount /dev/sdb2 $LFS
sudo mkdir -pv $LFS/boot
sudo mount /dev/sdb1 $LFS/boot

# Create directories
sudo mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}
sudo mkdir -pv $LFS/tools

# Create LFS user
sudo groupadd lfs
sudo useradd -s /bin/bash -g lfs -m -k /dev/null lfs
sudo chown -v lfs $LFS/{usr{,/*},var,etc,tools}
```

---

## Building the Toolchain

```bash
# Download all source packages (~500MB)
mkdir -v $LFS/sources
chmod -v a+wt $LFS/sources
# Download package list from LFS book

# Build cross-compiler (key packages)
# 1. Binutils (pass 1) — assembler and linker
tar xf binutils-*.tar.xz && cd binutils-*
mkdir build && cd build
../configure --prefix=$LFS/tools --target=$LFS_TGT --with-sysroot=$LFS
make -j$(nproc)
make install

# 2. GCC (pass 1) — cross-compiler
tar xf gcc-*.tar.xz && cd gcc-*
./configure --prefix=$LFS/tools --target=$LFS_TGT ...
make -j$(nproc)
make install

# 3. Linux API headers
make mrproper
make headers
cp -rv usr/include/* $LFS/usr/include

# 4. Glibc — C library
# ... (follow LFS book exactly)

# Continue with remaining toolchain packages...
```

---

## Building the Base System

```bash
# Enter chroot (after toolchain is ready)
sudo chroot "$LFS" /usr/bin/env -i \
    HOME=/root \
    TERM="$TERM" \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/usr/bin:/usr/sbin \
    /bin/bash --login

# Inside chroot, build ~80 packages including:
# - Glibc (final)           - Bash
# - GCC (final)             - Coreutils
# - Binutils (final)        - Grep, Sed, Awk
# - Make                    - Systemd or SysVinit
# - Findutils               - Util-linux
# - E2fsprogs               - Procps-ng
# - Iproute2                - Man-pages
# ... and many more

# Each package follows the pattern:
tar xf package.tar.xz
cd package-*/
./configure --prefix=/usr
make -j$(nproc)
make check                   # Run tests (important!)
make install
cd ..
rm -rf package-*/
```

---

## System Configuration

```bash
# /etc/fstab
cat > /etc/fstab << "EOF"
/dev/sdb2  /       ext4  defaults        1 1
/dev/sdb1  /boot   ext4  defaults        0 2
/dev/sdb3  swap    swap  pri=1           0 0
proc       /proc   proc  nosuid,noexec   0 0
sysfs      /sys    sysfs nosuid,noexec   0 0
devpts     /dev/pts devpts gid=5,mode=620 0 0
tmpfs      /run    tmpfs defaults         0 0
EOF

# Hostname
echo "lfs" > /etc/hostname

# Network
cat > /etc/systemd/network/10-eth0.network << "EOF"
[Match]
Name=eth0
[Network]
DHCP=ipv4
EOF

# Timezone
ln -sf /usr/share/zoneinfo/Asia/Dhaka /etc/localtime

# Locale
cat > /etc/locale.conf << "EOF"
LANG=en_US.UTF-8
EOF
```

---

## Making it Bootable

```bash
# Build the Linux kernel
tar xf linux-*.tar.xz && cd linux-*
make mrproper
make defconfig                # Or: make menuconfig
make -j$(nproc)
make modules_install
cp arch/x86/boot/bzImage /boot/vmlinuz-6.8-lfs
cp System.map /boot/System.map-6.8-lfs
cp .config /boot/config-6.8-lfs

# Install GRUB
grub-install /dev/sdb

cat > /boot/grub/grub.cfg << "EOF"
set default=0
set timeout=5

menuentry "Linux From Scratch" {
    linux /boot/vmlinuz-6.8-lfs root=/dev/sdb2 ro
}
EOF

# Set root password
passwd

# Exit chroot and reboot!
exit
sudo reboot
# Select LFS from GRUB menu
```

---

## Beyond LFS (BLFS)

After LFS, you have a minimal system. BLFS adds:

| Category | Packages |
|----------|----------|
| **Networking** | wget, curl, openssh, NetworkManager |
| **Xorg/GUI** | Xorg, Mesa, Wayland |
| **Desktop** | GNOME, KDE, Xfce |
| **Development** | Git, Python, Node.js |
| **Multimedia** | FFmpeg, PulseAudio/PipeWire |
| **Servers** | Apache, Nginx, PostgreSQL |

---

## 🏋️ Practice Exercises

1. **Prepare**: Set up a VM and partition it for LFS
2. **Toolchain**: Build binutils and GCC cross-compiler
3. **Headers**: Install Linux API headers
4. **Chroot**: Enter the chroot environment
5. **Kernel**: Configure and build a minimal kernel
6. **Boot**: Install GRUB and boot your LFS system
7. **BLFS**: Add networking (wget, openssh) to your LFS

---

<p align="center">
  <a href="../30-ebpf-tracing/README.md">← Previous: eBPF & Tracing</a> · <a href="../README.md">🏠 Home</a> · <a href="../32-embedded-linux/README.md">Next: Embedded Linux →</a>
</p>
