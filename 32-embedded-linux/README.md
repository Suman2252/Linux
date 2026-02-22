# 🔌 Chapter 32: Embedded Linux

<p align="center">
  <img src="https://img.shields.io/badge/Level-Expert-red?style=for-the-badge" alt="Expert">
  <img src="https://img.shields.io/badge/Chapter-32%20of%2034-blue?style=for-the-badge" alt="Chapter 32">
</p>

---

## 📑 Table of Contents

- [What is Embedded Linux?](#what-is-embedded-linux)
- [Build Systems](#build-systems)
- [Yocto Project](#yocto-project)
- [Buildroot](#buildroot)
- [Cross-Compilation](#cross-compilation)
- [BusyBox — Swiss Army Knife](#busybox--swiss-army-knife)
- [U-Boot Bootloader](#u-boot-bootloader)
- [Device Trees](#device-trees)
- [Root Filesystem Types](#root-filesystem-types)
- [Practice Exercises](#-practice-exercises)

---

## What is Embedded Linux?

Embedded Linux runs on specialized hardware: IoT devices, routers, smart TVs, cars, industrial controllers, Raspberry Pi, etc.

```
Typical Embedded Stack:
┌─────────────────────┐
│    Application      │  ← Your code
├─────────────────────┤
│   Root Filesystem   │  ← BusyBox, libraries
├─────────────────────┤
│   Linux Kernel      │  ← Custom-configured
├─────────────────────┤
│   Bootloader        │  ← U-Boot
├─────────────────────┤
│   Hardware (SoC)    │  ← ARM, MIPS, RISC-V
└─────────────────────┘
```

---

## Build Systems

| Tool | Description | Complexity |
|------|-------------|------------|
| **Yocto** | Full-featured, layer-based, industry standard | High |
| **Buildroot** | Simple, fast, menu-driven | Medium |
| **OpenWrt** | Router/networking focused | Medium |
| **Manual** | Cross-compile everything yourself | Very High |

---

## Yocto Project

```bash
# Install dependencies
sudo apt install gawk wget git diffstat unzip texinfo \
  gcc build-essential chrpath socat cpio python3 python3-pip \
  python3-pexpect xz-utils debianutils iputils-ping \
  python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev \
  pylint xterm python3-subunit mesa-common-dev zstd liblz4-tool

# Clone Yocto
git clone git://git.yoctoproject.org/poky
cd poky
git checkout kirkstone          # LTS branch

# Initialize build environment
source oe-init-build-env build

# Configure
# Edit conf/local.conf:
# MACHINE = "qemux86-64"        # Target machine
# DISTRO_FEATURES:append = " systemd"

# Build minimal image
bitbake core-image-minimal      # ~1-2 hours first time

# Test with QEMU
runqemu qemux86-64
```

### Yocto Layers

```bash
# Add community layers
git clone git://git.openembedded.org/meta-openembedded
bitbake-layers add-layer ../meta-openembedded/meta-oe

# Create your own layer
bitbake-layers create-layer ../meta-mylayer
bitbake-layers add-layer ../meta-mylayer
```

---

## Buildroot

```bash
# Download
git clone https://github.com/buildroot/buildroot.git
cd buildroot

# Configure
make menuconfig
# Target options → Architecture → ARM
# Build options → Download dir, output dir
# Toolchain → External toolchain or Buildroot toolchain
# System → Init system, /dev management
# Target packages → Select packages needed
# Filesystem → ext4, squashfs, etc.
# Kernel → Enable, select version
# Bootloader → U-Boot

# Build
make -j$(nproc)

# Output in output/images/
ls output/images/
# rootfs.ext4, zImage, dtbs, etc.

# Quick configs for boards
make raspberrypi3_64_defconfig
make qemu_x86_64_defconfig
make beaglebone_defconfig
make
```

---

## Cross-Compilation

```bash
# Install ARM cross-compiler
sudo apt install gcc-aarch64-linux-gnu

# Cross-compile a program
aarch64-linux-gnu-gcc -o hello hello.c

# Or set environment variables
export CROSS_COMPILE=aarch64-linux-gnu-
export ARCH=arm64

# Cross-compile the kernel
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
```

---

## BusyBox — Swiss Army Knife

BusyBox combines 300+ Linux utilities into a single binary (<1MB).

```bash
# Download
wget https://busybox.net/downloads/busybox-1.36.1.tar.bz2
tar xjf busybox-*.tar.bz2 && cd busybox-*

# Configure
make menuconfig                 # Select which applets to include
make defconfig                  # All applets

# Build
make CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)

# Install
make install                    # Creates _install/ directory

# Result: single binary with symlinks
ls -la _install/bin/
# busybox   (the binary)
# ls -> busybox
# cat -> busybox
# sh -> busybox
# ...
```

---

## U-Boot Bootloader

```bash
# Download
git clone https://github.com/u-boot/u-boot.git
cd u-boot

# Configure for target board
make CROSS_COMPILE=aarch64-linux-gnu- rpi_3_defconfig

# Build
make CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)

# Key U-Boot commands (at U-Boot prompt)
# printenv          — Show environment variables
# setenv bootargs "console=ttyS0,115200 root=/dev/mmcblk0p2"
# saveenv           — Save to flash
# boot              — Boot the kernel
# tftp              — Load image over network
# fatload           — Load from FAT partition
```

---

## Device Trees

Device Trees describe hardware to the kernel (instead of hardcoding).

```dts
// example.dts
/dts-v1/;

/ {
    compatible = "myboard";
    model = "My Custom Board";

    memory@80000000 {
        device_type = "memory";
        reg = <0x80000000 0x40000000>;  /* 1GB at 0x80000000 */
    };

    leds {
        compatible = "gpio-leds";
        led0 {
            label = "status";
            gpios = <&gpio1 5 0>;       /* GPIO1 pin 5 */
        };
    };
};
```

```bash
# Compile device tree
dtc -I dts -O dtb -o board.dtb board.dts

# Decompile
dtc -I dtb -O dts -o board.dts board.dtb

# Kernel uses DTB at boot
# U-Boot loads: kernel + DTB → boots Linux
```

---

## Root Filesystem Types

| Type | Description | Use Case |
|------|-------------|----------|
| **ext4** | Standard read-write | General purpose |
| **squashfs** | Read-only, compressed | Firmware, LiveUSB |
| **initramfs** | In-memory, loaded at boot | Tiny systems, rescue |
| **UBIFS** | For raw NAND flash | Flash storage |
| **overlayfs** | Read-only base + writable overlay | Router firmware |

```bash
# Create initramfs
find _install | cpio -o -H newc | gzip > initramfs.cpio.gz

# Create squashfs
mksquashfs rootfs/ rootfs.squashfs -comp xz
```

---

## 🏋️ Practice Exercises

1. **Buildroot**: Build a minimal system for QEMU and boot it
2. **BusyBox**: Configure and build BusyBox with a custom set of utilities
3. **Cross-compile**: Cross-compile a C program for ARM
4. **Yocto**: Build `core-image-minimal` for `qemux86-64`
5. **Device Tree**: Decompile a DTB and explore the hardware description
6. **initramfs**: Create a minimal initramfs with BusyBox
7. **Raspberry Pi**: Build a custom Linux image for Raspberry Pi with Buildroot

---

<p align="center">
  <a href="../31-linux-from-scratch/README.md">← Previous: Linux From Scratch</a> · <a href="../README.md">🏠 Home</a> · <a href="../33-ha-clustering/README.md">Next: HA Clustering →</a>
</p>
