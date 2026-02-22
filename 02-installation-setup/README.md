# 💿 Chapter 02: Installation & Setup

<p align="center">
  <img src="https://img.shields.io/badge/Level-Beginner-green?style=for-the-badge" alt="Beginner">
  <img src="https://img.shields.io/badge/Chapter-02%20of%2034-blue?style=for-the-badge" alt="Chapter 02">
</p>

---

## 📑 Table of Contents

- [Before You Install](#before-you-install)
- [Method 1: Virtual Machine (Recommended for Beginners)](#method-1-virtual-machine-recommended-for-beginners)
- [Method 2: WSL (Windows Subsystem for Linux)](#method-2-wsl-windows-subsystem-for-linux)
- [Method 3: Dual Boot](#method-3-dual-boot)
- [Method 4: Live USB (Try Without Installing)](#method-4-live-usb-try-without-installing)
- [Method 5: Cloud Instance](#method-5-cloud-instance)
- [Post-Installation Setup](#post-installation-setup)
- [Practice Exercises](#-practice-exercises)

---

## Before You Install

### System Requirements (Ubuntu 24.04 LTS)

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **CPU** | 2 GHz dual-core | 4+ cores |
| **RAM** | 4 GB | 8 GB+ |
| **Storage** | 25 GB | 50 GB+ |
| **Display** | 1024×768 | 1920×1080 |

### Download a Linux ISO

| Distro | Download Link | Notes |
|--------|--------------|-------|
| Ubuntu | [ubuntu.com/download](https://ubuntu.com/download) | LTS = Long Term Support (5 years) |
| Fedora | [fedoraproject.org](https://fedoraproject.org/) | Latest desktop tech |
| Linux Mint | [linuxmint.com](https://linuxmint.com/download.php) | Great for Windows users |

> 💡 **Tip**: Always verify your download with the provided SHA256 checksum to ensure integrity.

```bash
# Verify ISO integrity (on Linux/Mac)
sha256sum ubuntu-24.04-desktop-amd64.iso

# On Windows PowerShell
Get-FileHash .\ubuntu-24.04-desktop-amd64.iso -Algorithm SHA256
```

---

## Method 1: Virtual Machine (Recommended for Beginners)

A **Virtual Machine (VM)** runs Linux inside your current OS — no risk to your existing system.

### Using VirtualBox (Free)

**Step 1: Install VirtualBox**
- Download from [virtualbox.org](https://www.virtualbox.org/wiki/Downloads)
- Install with default settings

**Step 2: Create a New VM**
1. Click **"New"**
2. Name: `Ubuntu`
3. Type: `Linux`, Version: `Ubuntu (64-bit)`
4. RAM: `4096 MB` (4 GB minimum)
5. Create a virtual hard disk: `VDI`, `Dynamically allocated`, `50 GB`

**Step 3: Configure the VM**
```
Settings → System → Processor → 2+ CPUs
Settings → Display → Video Memory → 128 MB
Settings → Storage → Controller: IDE → Choose your ISO file
```

**Step 4: Install Linux**
1. Start the VM
2. Select **"Install Ubuntu"**
3. Follow the graphical installer
4. Choose **"Erase disk and install"** (this only erases the virtual disk!)
5. Set your username, password, and timezone
6. Reboot when prompted

**Step 5: Install Guest Additions** (for better performance)
```bash
# After booting into your new Linux install
sudo apt update
sudo apt install virtualbox-guest-utils virtualbox-guest-x11
sudo reboot
```

> 🚀 **Pro Tip**: Enable **Shared Clipboard** and **Drag & Drop** in VM Settings → General → Advanced.

### Using VMware Workstation Player (Free)
- Download from [vmware.com](https://www.vmware.com/products/workstation-player.html)
- Steps are similar to VirtualBox
- VMware often has better performance out of the box

---

## Method 2: WSL (Windows Subsystem for Linux)

**WSL** lets you run Linux directly on Windows 10/11 — no VM needed. Perfect for developers.

### Install WSL 2

```powershell
# Open PowerShell as Administrator and run:
wsl --install

# This installs WSL 2 with Ubuntu by default
# Restart your computer when prompted
```

### After Restart

```powershell
# Ubuntu will launch automatically
# Set your username and password when prompted

# To install a different distro:
wsl --list --online           # See available distros
wsl --install -d Debian       # Install Debian
wsl --install -d Fedora       # Install Fedora
```

### Useful WSL Commands

```powershell
wsl --list --verbose          # List installed distros
wsl --set-default Ubuntu      # Set default distro
wsl --shutdown                # Shut down all WSL instances
wsl --update                  # Update WSL kernel
```

### Access Files Between Windows and WSL

```bash
# From WSL, access Windows files:
cd /mnt/c/Users/YourName/Desktop

# From Windows, access WSL files:
# Open File Explorer and navigate to:  \\wsl$\Ubuntu\home\username
```

> ⚠️ **Warning**: Do NOT edit WSL Linux files from Windows using Windows tools — it can corrupt them. Use `code .` to open VS Code with WSL support.

---

## Method 3: Dual Boot

Run Linux **alongside** Windows on the same machine. You choose which to boot at startup.

### Prerequisites
- Back up all important data
- At least 50 GB of free disk space
- A USB drive (8 GB+)
- Disable Secure Boot in BIOS (sometimes needed)
- Disable Fast Startup in Windows

### Step 1: Create a Bootable USB

**Using Rufus (Windows):**
1. Download [Rufus](https://rufus.ie/)
2. Insert USB drive
3. Select your Linux ISO
4. Partition scheme: **GPT** (for UEFI) or **MBR** (for Legacy BIOS)
5. Click **Start**

**Using Etcher (Cross-platform):**
1. Download [balenaEtcher](https://etcher.balena.io/)
2. Select ISO → Select USB → Flash!

### Step 2: Shrink Windows Partition

```
Windows: Disk Management → Right-click C: → Shrink Volume → Enter 50000 MB (50 GB)
```

### Step 3: Boot from USB

1. Restart your computer
2. Press boot menu key during startup:
   - **Dell/Lenovo**: `F12`
   - **HP**: `F9`
   - **ASUS**: `F8` or `Esc`
   - **Acer**: `F12`
3. Select your USB drive

### Step 4: Install Linux

1. Choose **"Install alongside Windows"** (the installer will auto-detect)
2. Allocate space (50 GB minimum recommended)
3. The installer will set up GRUB bootloader automatically
4. On reboot, you'll see a menu to choose Windows or Linux

> 🛡️ **Important**: Always install Linux AFTER Windows. Linux's GRUB bootloader will detect Windows, but Windows' bootloader won't detect Linux.

---

## Method 4: Live USB (Try Without Installing)

Boot from USB to try Linux without changing anything on your hard drive.

1. Create a bootable USB (same as dual boot Step 1)
2. Boot from USB
3. Choose **"Try Ubuntu"** instead of "Install"
4. Explore! Changes are lost when you shut down.

> 💡 **Use Case**: Great for testing hardware compatibility, recovering data from a broken system, or just exploring.

---

## Method 5: Cloud Instance

Run Linux on a remote server — no local installation needed.

### Free Tier Options

| Provider | Free Offer |
|----------|-----------|
| **AWS** | 750 hours/month of t2.micro for 12 months |
| **Google Cloud** | e2-micro instance always free |
| **Oracle Cloud** | 2 AMD instances always free |
| **DigitalOcean** | $200 credit for 60 days (with GitHub Student Pack) |

### Quick Start with AWS

```bash
# After creating an AWS account and launching an EC2 instance:
ssh -i "your-key.pem" ubuntu@your-instance-ip

# You now have a full Linux terminal!
```

---

## Post-Installation Setup

Once you have Linux running, do these first:

### 1. Update Your System

```bash
# Debian/Ubuntu
sudo apt update && sudo apt upgrade -y

# Fedora
sudo dnf update -y

# Arch
sudo pacman -Syu
```

### 2. Install Essential Tools

```bash
# Ubuntu/Debian
sudo apt install -y curl wget git vim htop net-tools build-essential

# Fedora
sudo dnf install -y curl wget git vim htop net-tools gcc make
```

### 3. Set Your Timezone

```bash
# Check current timezone
timedatectl

# List available timezones
timedatectl list-timezones

# Set timezone
sudo timedatectl set-timezone Asia/Dhaka
```

### 4. Create Your First File

```bash
echo "Hello, Linux!" > ~/my-first-file.txt
cat ~/my-first-file.txt
# Output: Hello, Linux!
```

🎉 **Congratulations! You now have a working Linux system!**

---

## 🏋️ Practice Exercises

1. **Install Linux** using any method above (VM is recommended for beginners)
2. **Update your system** using the appropriate package manager
3. **Install `htop`** and run it: `htop`
4. **Check your kernel version**: `uname -r`
5. **Check your disk space**: `df -h`
6. **Create a file**: `echo "I'm learning Linux!" > ~/hello.txt`

---

<p align="center">
  <a href="../01-introduction/README.md">← Previous: Introduction</a> · <a href="../README.md">🏠 Home</a> · <a href="../03-terminal-basics/README.md">Next: Terminal Basics →</a>
</p>
