# 🛡️ Chapter 20: Advanced Networking

<p align="center">
  <img src="https://img.shields.io/badge/Level-Advanced-orange?style=for-the-badge" alt="Advanced">
  <img src="https://img.shields.io/badge/Chapter-20%20of%2034-blue?style=for-the-badge" alt="Chapter 20">
</p>

---

## 📑 Table of Contents

- [iptables — The Classic Firewall](#iptables--the-classic-firewall)
- [nftables — Modern Replacement](#nftables--modern-replacement)
- [Advanced Routing](#advanced-routing)
- [VLANs](#vlans)
- [Network Bonding/Teaming](#network-bondingteaming)
- [Traffic Control (tc)](#traffic-control-tc)
- [WireGuard VPN](#wireguard-vpn)
- [Network Namespaces](#network-namespaces)
- [Practice Exercises](#-practice-exercises)

---

## iptables — The Classic Firewall

### Chains and Tables

```
                    ┌─────────────────┐
   Incoming ───────▶│   PREROUTING    │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │    ROUTING      │
                    │   DECISION      │
                    └──┬───────────┬──┘
                       │           │
              ┌────────▼──┐  ┌────▼────────┐
              │   INPUT   │  │   FORWARD   │
              └────────┬──┘  └────┬────────┘
                       │          │
              ┌────────▼──┐  ┌────▼────────┐
              │  LOCAL     │  │ POSTROUTING │───▶ Outgoing
              │  PROCESS   │  └─────────────┘
              └────────┬──┘
              ┌────────▼──┐
              │  OUTPUT   │
              └────────┬──┘
              ┌────────▼──────┐
              │  POSTROUTING  │───▶ Outgoing
              └───────────────┘
```

### Basic iptables Commands

```bash
# View rules
sudo iptables -L -v -n                   # List all rules
sudo iptables -L INPUT -v -n --line-numbers  # With line numbers
sudo iptables -t nat -L -v -n            # NAT table

# Default policies
sudo iptables -P INPUT DROP              # Drop all incoming by default
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT           # Allow all outgoing

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow ping
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Block specific IP
sudo iptables -A INPUT -s 192.168.1.50 -j DROP

# Allow from subnet only
sudo iptables -A INPUT -p tcp --dport 3306 -s 10.0.0.0/8 -j ACCEPT

# Rate limiting (anti-brute-force)
sudo iptables -A INPUT -p tcp --dport 22 -m recent --set --name ssh
sudo iptables -A INPUT -p tcp --dport 22 -m recent --update --seconds 60 --hitcount 4 --name ssh -j DROP

# Delete a rule
sudo iptables -D INPUT 3                 # Delete rule #3
sudo iptables -F                          # Flush all rules

# NAT / Port forwarding
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Save rules (so they persist after reboot)
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

---

## nftables — Modern Replacement

```bash
# nftables uses a cleaner syntax and replaces iptables

# List rules
sudo nft list ruleset

# Create a table
sudo nft add table inet filter

# Create chains
sudo nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
sudo nft add chain inet filter forward { type filter hook forward priority 0 \; policy drop \; }
sudo nft add chain inet filter output { type filter hook output priority 0 \; policy accept \; }

# Add rules
sudo nft add rule inet filter input ct state established,related accept
sudo nft add rule inet filter input iif lo accept
sudo nft add rule inet filter input tcp dport 22 accept
sudo nft add rule inet filter input tcp dport {80, 443} accept
sudo nft add rule inet filter input ip saddr 192.168.1.0/24 accept
sudo nft add rule inet filter input icmp type echo-request accept

# Save
sudo nft list ruleset > /etc/nftables.conf
sudo systemctl enable nftables
```

---

## Advanced Routing

```bash
# Multiple routing tables
echo "100 custom" | sudo tee -a /etc/iproute2/rt_tables
sudo ip route add default via 10.0.0.1 table custom
sudo ip rule add from 10.0.0.0/24 table custom

# Policy-based routing
sudo ip rule add from 192.168.1.100 lookup custom
sudo ip rule show

# Static routes
sudo ip route add 10.10.0.0/16 via 192.168.1.1 dev eth0
sudo ip route add 172.16.0.0/12 via 192.168.1.254 metric 100

# IP forwarding (router mode)
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
```

---

## VLANs

```bash
# Create VLAN interface
sudo ip link add link eth0 name eth0.100 type vlan id 100
sudo ip addr add 192.168.100.1/24 dev eth0.100
sudo ip link set eth0.100 up

# Permanent (Netplan)
# /etc/netplan/01-vlans.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
  vlans:
    eth0.100:
      id: 100
      link: eth0
      addresses: [192.168.100.1/24]
```

---

## Network Bonding/Teaming

Combine multiple interfaces for redundancy or throughput.

```bash
# Load bonding module
sudo modprobe bonding

# Create bond
sudo ip link add bond0 type bond mode active-backup
sudo ip link set eth0 master bond0
sudo ip link set eth1 master bond0
sudo ip addr add 192.168.1.10/24 dev bond0
sudo ip link set bond0 up

# Bond modes:
# 0 = balance-rr (round robin)
# 1 = active-backup
# 2 = balance-xor
# 3 = broadcast
# 4 = 802.3ad (LACP)
# 5 = balance-tlb
# 6 = balance-alb
```

---

## Traffic Control (tc)

Simulate network conditions or shape traffic.

```bash
# Add 100ms latency
sudo tc qdisc add dev eth0 root netem delay 100ms

# Add packet loss (1%)
sudo tc qdisc add dev eth0 root netem loss 1%

# Limit bandwidth to 1 Mbit
sudo tc qdisc add dev eth0 root tbf rate 1mbit burst 32kbit latency 400ms

# Remove all rules
sudo tc qdisc del dev eth0 root

# View current rules
tc qdisc show dev eth0
```

---

## WireGuard VPN

Modern, fast, simple VPN.

```bash
# Install
sudo apt install wireguard

# Generate keys
wg genkey | tee privatekey | wg pubkey > publickey

# Server config (/etc/wireguard/wg0.conf)
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <server-private-key>
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <client-public-key>
AllowedIPs = 10.0.0.2/32

# Start
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0

# Status
sudo wg show
```

---

## Network Namespaces

Isolate network stacks — the foundation of container networking.

```bash
# Create namespace
sudo ip netns add ns1
sudo ip netns add ns2

# List namespaces
ip netns list

# Run command in namespace
sudo ip netns exec ns1 ip addr show

# Create virtual ethernet pair
sudo ip link add veth0 type veth peer name veth1

# Assign to namespaces
sudo ip link set veth0 netns ns1
sudo ip link set veth1 netns ns2

# Configure
sudo ip netns exec ns1 ip addr add 10.0.0.1/24 dev veth0
sudo ip netns exec ns1 ip link set veth0 up
sudo ip netns exec ns2 ip addr add 10.0.0.2/24 dev veth1
sudo ip netns exec ns2 ip link set veth1 up

# Test connectivity
sudo ip netns exec ns1 ping 10.0.0.2

# Delete
sudo ip netns del ns1
```

---

## 🏋️ Practice Exercises

1. **iptables**: Set up a basic firewall that allows SSH, HTTP, and HTTPS
2. **NAT**: Enable IP masquerading for internet sharing
3. **nftables**: Convert your iptables rules to nftables
4. **WireGuard**: Set up a WireGuard VPN between two machines (or VMs)
5. **tc**: Add 200ms latency to an interface and test with ping
6. **Namespaces**: Create two network namespaces and make them communicate
7. **Routing**: Add a static route to reach a different subnet
8. **Monitor**: Use `ss` and `iftop` to analyze live traffic

---

<p align="center">
  <a href="../19-systemd-service-management/README.md">← Previous: Systemd & Service Management</a> · <a href="../README.md">🏠 Home</a> · <a href="../21-security-hardening/README.md">Next: Security & Hardening →</a>
</p>
