# 🌐 Chapter 12: Networking Fundamentals

<p align="center">
  <img src="https://img.shields.io/badge/Level-Intermediate-yellow?style=for-the-badge" alt="Intermediate">
  <img src="https://img.shields.io/badge/Chapter-12%20of%2034-blue?style=for-the-badge" alt="Chapter 12">
</p>

---

## 📑 Table of Contents

- [Networking Concepts](#networking-concepts)
- [Network Interfaces](#network-interfaces)
- [IP Addressing](#ip-addressing)
- [DNS (Domain Name System)](#dns-domain-name-system)
- [Network Diagnostic Tools](#network-diagnostic-tools)
- [Network Configuration](#network-configuration)
- [Downloading & Transferring](#downloading--transferring)
- [Firewall Basics (ufw)](#firewall-basics-ufw)
- [Practice Exercises](#-practice-exercises)

---

## Networking Concepts

### The OSI Model (Simplified)

```
Layer 7 │ Application  │ HTTP, SSH, DNS, FTP       │ curl, wget
Layer 4 │ Transport    │ TCP, UDP                   │ ss, netstat
Layer 3 │ Network      │ IP, ICMP, routing          │ ip, ping, traceroute
Layer 2 │ Data Link    │ Ethernet, ARP, MAC address │ arp, bridge
Layer 1 │ Physical     │ Cables, WiFi signals       │ ethtool
```

### Key Terms

| Term | Description |
|------|-------------|
| **IP Address** | Unique network identifier (e.g., `192.168.1.100`) |
| **Subnet** | A network segment (e.g., `192.168.1.0/24`) |
| **Gateway** | Router that connects to other networks |
| **DNS** | Translates domain names → IP addresses |
| **DHCP** | Auto-assigns IP addresses |
| **Port** | Endpoint for communication (0–65535) |
| **MAC Address** | Hardware address (`aa:bb:cc:dd:ee:ff`) |

### Common Ports

| Port | Service | Protocol |
|------|---------|----------|
| 22 | SSH | TCP |
| 80 | HTTP | TCP |
| 443 | HTTPS | TCP |
| 53 | DNS | TCP/UDP |
| 25 | SMTP | TCP |
| 3306 | MySQL | TCP |
| 5432 | PostgreSQL | TCP |
| 6379 | Redis | TCP |

---

## Network Interfaces

```bash
# List interfaces with ip command (modern)
ip link show
ip -brief link show          # Concise output
ip -c addr show              # With colors

# List interfaces (legacy)
ifconfig                     # May need: sudo apt install net-tools

# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down
```

---

## IP Addressing

```bash
# Show IP addresses
ip addr show                 # All interfaces
ip -4 addr show              # IPv4 only
ip -6 addr show              # IPv6 only
ip addr show eth0            # Specific interface
hostname -I                  # Quick — just the IPs

# Add/Remove IP address (temporary)
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr del 192.168.1.100/24 dev eth0

# Get your public IP
curl -s ifconfig.me
curl -s icanhazip.com
curl -s ipinfo.io/ip
```

### Routing

```bash
# Show routing table
ip route show                # Or: ip r
route -n                     # Legacy format

# Default gateway
ip route | grep default

# Add a route
sudo ip route add 10.0.0.0/8 via 192.168.1.1 dev eth0

# Delete a route
sudo ip route del 10.0.0.0/8

# Trace the route to a host
traceroute google.com        # Shows every hop
tracepath google.com         # Similar, no root needed
mtr google.com               # Real-time traceroute (interactive)
```

---

## DNS (Domain Name System)

```bash
# Check DNS settings
cat /etc/resolv.conf
resolvectl status            # systemd-resolved status

# Resolve hostnames
nslookup google.com          # Simple lookup
dig google.com               # Detailed lookup
dig +short google.com        # Just the IP
dig -x 8.8.8.8               # Reverse DNS lookup
host google.com              # Simple resolution

# /etc/hosts — local DNS override
cat /etc/hosts
# 127.0.0.1    localhost
# 192.168.1.50 myserver.local

# Test resolution order
getent hosts google.com
```

---

## Network Diagnostic Tools

### `ping` — Test Connectivity

```bash
ping google.com              # Continuous ping (Ctrl+C to stop)
ping -c 5 google.com         # Send 5 packets
ping -i 0.5 google.com       # 0.5 second interval
ping -s 1024 google.com      # Custom packet size
```

### `ss` — Socket Statistics (Modern netstat)

```bash
ss                           # All connections
ss -t                        # TCP connections
ss -u                        # UDP connections
ss -l                        # Listening sockets
ss -tuln                     # TCP/UDP listening, numeric ports
ss -tulnp                    # Include process name (need sudo)
ss -s                        # Summary statistics
ss state established         # Only established connections
ss -t dst 192.168.1.1        # Connections to specific host
```

### `netstat` — Legacy (Still Useful)

```bash
sudo apt install net-tools
netstat -tuln                # TCP/UDP listening, numeric
netstat -an                  # All connections, numeric
netstat -rn                  # Routing table
```

### `curl` & `wget` — HTTP Tools

```bash
# curl — transfer data
curl https://example.com                  # GET request
curl -I https://example.com               # Headers only
curl -o file.html https://example.com     # Save to file
curl -O https://example.com/file.zip      # Save with original name
curl -L https://short.url                 # Follow redirects
curl -s https://api.github.com/users/sovon | jq .  # JSON API

# wget — download files
wget https://example.com/file.zip         # Download file
wget -c https://example.com/large.zip     # Resume download
wget -r https://example.com               # Recursive download
wget -q -O - https://example.com          # Output to stdout
```

### Other Tools

```bash
# ARP table
ip neighbor show             # Modern
arp -a                       # Legacy

# Network statistics
ss -s                        # Socket statistics
cat /proc/net/dev            # Interface statistics
nstat                        # Kernel network statistics

# Bandwidth test
sudo apt install speedtest-cli
speedtest                    # Test internet speed

# Port scanning (with nmap)
sudo apt install nmap
nmap localhost               # Scan localhost
nmap 192.168.1.0/24          # Scan entire subnet
nmap -sV 192.168.1.1         # Detect service versions
```

---

## Network Configuration

### NetworkManager (Most Desktops)

```bash
# Using nmcli (NetworkManager CLI)
nmcli device status          # List network devices
nmcli connection show        # List connections
nmcli device wifi list       # List available WiFi networks

# Connect to WiFi
nmcli device wifi connect "MyNetwork" password "mypassword"

# Set static IP
nmcli connection modify "Wired connection 1" \
  ipv4.method manual \
  ipv4.addresses "192.168.1.100/24" \
  ipv4.gateway "192.168.1.1" \
  ipv4.dns "8.8.8.8,8.8.4.4"

# Apply changes
nmcli connection up "Wired connection 1"
```

### Netplan (Ubuntu Servers)

```yaml
# /etc/netplan/01-config.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

```bash
sudo netplan apply            # Apply netplan config
sudo netplan try              # Test (auto-reverts in 120s)
```

---

## Firewall Basics (ufw)

**ufw** (Uncomplicated Firewall) is the easy frontend for iptables.

```bash
# Install & enable
sudo apt install ufw
sudo ufw enable               # Enable firewall
sudo ufw disable              # Disable firewall
sudo ufw status verbose       # Show current rules

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow/Deny rules
sudo ufw allow 22/tcp         # Allow SSH
sudo ufw allow 80/tcp         # Allow HTTP
sudo ufw allow 443/tcp        # Allow HTTPS
sudo ufw allow from 192.168.1.0/24  # Allow entire subnet
sudo ufw deny 3306/tcp        # Deny MySQL from outside

# App profiles
sudo ufw app list             # List available profiles
sudo ufw allow 'Nginx Full'   # Allow HTTP + HTTPS

# Delete rules
sudo ufw status numbered      # Show rules with numbers
sudo ufw delete 3             # Delete rule #3

# Reset everything
sudo ufw reset
```

---

## 🏋️ Practice Exercises

1. **IP**: Find your IP address with `ip addr show` and `hostname -I`
2. **DNS**: Use `dig` to find the IP of `github.com`
3. **Ping**: Ping `8.8.8.8` with 5 packets and note the latency
4. **Ports**: List all listening ports with `ss -tuln`
5. **Route**: Find your default gateway with `ip route`
6. **curl**: Download a web page and save it to a file
7. **Firewall**: Enable ufw and allow only SSH (port 22)
8. **Public IP**: Find your public IP using `curl`

---

<p align="center">
  <a href="../11-disk-storage-management/README.md">← Previous: Disk & Storage Management</a> · <a href="../README.md">🏠 Home</a> · <a href="../13-ssh-remote-access/README.md">Next: SSH & Remote Access →</a>
</p>
