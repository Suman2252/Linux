# 🖥️ Chapter 29: Virtualization & KVM

<p align="center">
  <img src="https://img.shields.io/badge/Level-Expert-red?style=for-the-badge" alt="Expert">
  <img src="https://img.shields.io/badge/Chapter-29%20of%2034-blue?style=for-the-badge" alt="Chapter 29">
</p>

---

## 📑 Table of Contents

- [Virtualization Types](#virtualization-types)
- [KVM Setup](#kvm-setup)
- [Managing VMs with virsh](#managing-vms-with-virsh)
- [virt-manager (GUI)](#virt-manager-gui)
- [VM Templates & Cloning](#vm-templates--cloning)
- [Networking](#networking)
- [Storage](#storage)
- [Live Migration](#live-migration)
- [QEMU Tips](#qemu-tips)
- [Practice Exercises](#-practice-exercises)

---

## Virtualization Types

| Type | Description | Example |
|------|-------------|---------|
| **Type 1 (Bare-metal)** | Runs directly on hardware | KVM, Xen, VMware ESXi |
| **Type 2 (Hosted)** | Runs on host OS | VirtualBox, VMware Workstation |
| **Paravirtualization** | Guest OS modified for performance | Xen PV, virtio drivers |
| **Containers** | OS-level virtualization | Docker, LXC |

---

## KVM Setup

```bash
# Check hardware support
egrep -c '(vmx|svm)' /proc/cpuinfo    # > 0 means supported
lsmod | grep kvm

# Install KVM
sudo apt install qemu-kvm libvirt-daemon-system \
  libvirt-clients bridge-utils virtinst virt-manager

# Add user to groups
sudo usermod -aG libvirt,kvm $USER
newgrp libvirt

# Verify
virsh list --all
sudo systemctl status libvirtd
```

---

## Managing VMs with virsh

```bash
# Create VM from ISO
virt-install \
  --name ubuntu-server \
  --ram 2048 \
  --vcpus 2 \
  --disk size=20 \
  --os-variant ubuntu22.04 \
  --network bridge=virbr0 \
  --graphics vnc \
  --cdrom /path/to/ubuntu.iso

# List VMs
virsh list                            # Running VMs
virsh list --all                      # All VMs

# Start/Stop/Reboot
virsh start ubuntu-server
virsh shutdown ubuntu-server          # Graceful
virsh destroy ubuntu-server           # Force stop
virsh reboot ubuntu-server

# Autostart
virsh autostart ubuntu-server
virsh autostart --disable ubuntu-server

# Console access
virsh console ubuntu-server           # Serial console

# Info
virsh dominfo ubuntu-server
virsh vcpuinfo ubuntu-server
virsh domblklist ubuntu-server

# Snapshots
virsh snapshot-create-as ubuntu-server snap1 "Before update"
virsh snapshot-list ubuntu-server
virsh snapshot-revert ubuntu-server snap1
virsh snapshot-delete ubuntu-server snap1

# Edit VM config
virsh edit ubuntu-server              # Opens XML config

# Delete VM
virsh undefine ubuntu-server --remove-all-storage
```

---

## virt-manager (GUI)

```bash
# Launch GUI manager
virt-manager

# Remote management
virt-manager -c qemu+ssh://user@server/system
```

---

## VM Templates & Cloning

```bash
# Clone a VM
virt-clone --original ubuntu-server --name ubuntu-clone --auto-clone

# Create template (sysprep)
sudo apt install libguestfs-tools
sudo virt-sysprep -d ubuntu-server    # Remove unique data (SSH keys, etc.)

# Cloud-init for automation
cat > cloud-init.yaml << 'EOF'
#cloud-config
hostname: myvm
users:
  - name: sovon
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh_authorized_keys:
      - ssh-ed25519 AAAA... sovon@pc
package_update: true
packages:
  - nginx
  - htop
EOF
```

---

## Networking

```bash
# Default NAT network
virsh net-list --all
virsh net-info default

# Create bridge network (for direct LAN access)
# /etc/netplan/01-bridge.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
  bridges:
    br0:
      interfaces: [eth0]
      dhcp4: true

# Attach VM to bridge
virsh attach-interface ubuntu-server bridge br0 --model virtio --persistent

# Isolated network
virsh net-define isolated-net.xml
virsh net-start isolated-net
virsh net-autostart isolated-net
```

---

## Storage

```bash
# Storage pools
virsh pool-list --all
virsh pool-info default

# Create storage pool
virsh pool-define-as mypool dir - - - - /var/lib/libvirt/images/mypool
virsh pool-start mypool
virsh pool-autostart mypool

# Create volume
virsh vol-create-as default myvm.qcow2 20G --format qcow2

# Resize disk
virsh blockresize ubuntu-server /path/to/disk.qcow2 30G
# Then resize filesystem inside VM

# qcow2 operations
qemu-img create -f qcow2 disk.qcow2 20G
qemu-img info disk.qcow2
qemu-img resize disk.qcow2 +10G
qemu-img convert -f raw -O qcow2 disk.raw disk.qcow2
```

---

## Live Migration

```bash
# Prerequisites: shared storage, same CPU architecture
virsh migrate --live ubuntu-server qemu+ssh://target-host/system

# With storage migration
virsh migrate --live --copy-storage-all ubuntu-server qemu+ssh://target/system
```

---

## QEMU Tips

```bash
# Run VM directly with QEMU (without libvirt)
qemu-system-x86_64 \
  -enable-kvm \
  -m 2048 \
  -smp 2 \
  -hda disk.qcow2 \
  -cdrom ubuntu.iso \
  -boot d \
  -net nic -net user,hostfwd=tcp::2222-:22

# SSH into QEMU VM
ssh -p 2222 user@localhost
```

---

## 🏋️ Practice Exercises

1. **Install**: Set up KVM and verify with `virsh list`
2. **Create**: Install a VM from an ISO using `virt-install`
3. **Snapshot**: Take a snapshot, make changes, then revert
4. **Clone**: Clone an existing VM
5. **Network**: Create a bridged network for a VM
6. **Resize**: Expand a VM's disk and grow the filesystem
7. **Console**: Access a VM via `virsh console`
8. **Autostart**: Set a VM to auto-start on boot

---

<p align="center">
  <a href="../28-advanced-filesystems/README.md">← Previous: Advanced Filesystems</a> · <a href="../README.md">🏠 Home</a> · <a href="../30-ebpf-tracing/README.md">Next: eBPF & Tracing →</a>
</p>
