# 🔨 Chapter 25: Kernel Compilation & Custom Modules

<p align="center">
  <img src="https://img.shields.io/badge/Level-Expert-red?style=for-the-badge" alt="Expert">
  <img src="https://img.shields.io/badge/Chapter-25%20of%2034-blue?style=for-the-badge" alt="Chapter 25">
</p>

---

## 📑 Table of Contents

- [Why Compile Your Own Kernel?](#why-compile-your-own-kernel)
- [Compiling the Linux Kernel](#compiling-the-linux-kernel)
- [Kernel Configuration](#kernel-configuration)
- [Writing a Kernel Module](#writing-a-kernel-module)
- [DKMS — Dynamic Kernel Module Support](#dkms--dynamic-kernel-module-support)
- [Practice Exercises](#-practice-exercises)

---

## Why Compile Your Own Kernel?

| Reason | Example |
|--------|---------|
| Enable specific features | Real-time patches, new filesystems |
| Remove bloat | Smaller kernel for embedded/VMs |
| Hardware support | Custom drivers not in mainline |
| Security | Remove unused features = smaller attack surface |
| Learning | Understand Linux internals deeply |

---

## Compiling the Linux Kernel

### Step 1: Get the Source

```bash
# Install dependencies
sudo apt install build-essential libncurses-dev bison flex libssl-dev \
  libelf-dev dwarves bc rsync

# Download kernel source
cd /usr/src
sudo wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.8.tar.xz
sudo tar -xJf linux-6.8.tar.xz
cd linux-6.8
```

### Step 2: Configure

```bash
# Start from current config
cp /boot/config-$(uname -r) .config
make olddefconfig                   # Accept defaults for new options

# Or interactive configuration
make menuconfig                     # TUI (recommended)
make xconfig                        # Qt GUI
make nconfig                        # Newer TUI
```

### Step 3: Compile

```bash
# Compile (use all CPU cores)
make -j$(nproc)

# Compile modules
make modules -j$(nproc)

# Install modules
sudo make modules_install

# Install kernel
sudo make install

# Update bootloader
sudo update-grub                    # Debian/Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # RHEL/Fedora

# Reboot to new kernel
sudo reboot
uname -r                           # Verify
```

---

## Kernel Configuration

### Key Options in menuconfig

```
General setup →
  Local version: -custom            # Append to kernel version
  Default hostname: myhost

Processor type and features →
  Processor family: (select yours)
  Preemption Model: (desktop vs server)

Enable block layer →
  IO Schedulers: (select needed ones)

File systems →
  Btrfs, XFS, ext4, etc.

Networking support →
  Networking options →
    TCP/IP networking
    Network packet filtering (Netfilter)

Device Drivers →
  (Enable/disable hardware support)

Security options →
  SELinux, AppArmor, etc.
```

### Quick Configuration Tips

```bash
# Enable only modules currently in use
make localmodconfig              # Uses lsmod to determine needed modules

# Minimal config
make tinyconfig                  # Absolute minimal kernel

# Compare configs
scripts/diffconfig .config.old .config
```

---

## Writing a Kernel Module

### Hello World Module

```c
// hello.c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Sovon");
MODULE_DESCRIPTION("Hello World Kernel Module");
MODULE_VERSION("1.0");

static int __init hello_init(void) {
    printk(KERN_INFO "Hello from kernel module!\n");
    return 0;
}

static void __exit hello_exit(void) {
    printk(KERN_INFO "Goodbye from kernel module!\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

### Makefile

```makefile
obj-m += hello.o

KDIR := /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)

all:
	make -C $(KDIR) M=$(PWD) modules

clean:
	make -C $(KDIR) M=$(PWD) clean
```

### Build & Load

```bash
make                              # Compile
sudo insmod hello.ko              # Load module
dmesg | tail -1                   # "Hello from kernel module!"
lsmod | grep hello                # Verify loaded
sudo rmmod hello                  # Unload
dmesg | tail -1                   # "Goodbye from kernel module!"
```

### Module with Parameters

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/moduleparam.h>

static int count = 1;
static char *name = "world";

module_param(count, int, 0644);
MODULE_PARM_DESC(count, "Number of greetings");
module_param(name, charp, 0644);
MODULE_PARM_DESC(name, "Name to greet");

static int __init hello_init(void) {
    int i;
    for (i = 0; i < count; i++)
        printk(KERN_INFO "Hello, %s!\n", name);
    return 0;
}

// ... exit function and module macros
```

```bash
sudo insmod hello.ko count=3 name="Linux"
```

---

## DKMS — Dynamic Kernel Module Support

DKMS automatically rebuilds modules when the kernel is upgraded.

```bash
sudo apt install dkms

# Create DKMS structure
sudo mkdir /usr/src/hello-1.0/
sudo cp hello.c Makefile /usr/src/hello-1.0/

# Create dkms.conf
sudo tee /usr/src/hello-1.0/dkms.conf << 'EOF'
PACKAGE_NAME="hello"
PACKAGE_VERSION="1.0"
BUILT_MODULE_NAME[0]="hello"
DEST_MODULE_LOCATION[0]="/updates/dkms"
AUTOINSTALL="yes"
MAKE[0]="make -C /lib/modules/${kernelver}/build M=${dkms_tree}/${PACKAGE_NAME}/${PACKAGE_VERSION}/build modules"
CLEAN="make -C /lib/modules/${kernelver}/build M=${dkms_tree}/${PACKAGE_NAME}/${PACKAGE_VERSION}/build clean"
EOF

# Register and build
sudo dkms add -m hello -v 1.0
sudo dkms build -m hello -v 1.0
sudo dkms install -m hello -v 1.0

# Status
dkms status
```

---

## 🏋️ Practice Exercises

1. **Config**: Copy your current kernel config and explore `make menuconfig`
2. **(VM)** Compile a custom kernel with a different `LOCALVERSION` string
3. **Module**: Build and load the hello world kernel module
4. **Params**: Modify the module to accept parameters
5. **dmesg**: Watch kernel messages while loading/unloading modules
6. **DKMS**: Package your module with DKMS

---

<p align="center">
  <a href="../24-lvm-raid/README.md">← Previous: LVM & RAID</a> · <a href="../README.md">🏠 Home</a> · <a href="../26-docker-containers/README.md">Next: Docker & Containers →</a>
</p>
