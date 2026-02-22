# 🔐 Chapter 13: SSH & Remote Access

<p align="center">
  <img src="https://img.shields.io/badge/Level-Intermediate-yellow?style=for-the-badge" alt="Intermediate">
  <img src="https://img.shields.io/badge/Chapter-13%20of%2034-blue?style=for-the-badge" alt="Chapter 13">
</p>

---

## 📑 Table of Contents

- [What is SSH?](#what-is-ssh)
- [Basic SSH Usage](#basic-ssh-usage)
- [SSH Key Authentication](#ssh-key-authentication)
- [SSH Config File](#ssh-config-file)
- [SSH Tunneling & Port Forwarding](#ssh-tunneling--port-forwarding)
- [SCP & File Transfer](#scp--file-transfer)
- [rsync — Smart Syncing](#rsync--smart-syncing)
- [SSH Security Best Practices](#ssh-security-best-practices)
- [Jump Hosts & ProxyJump](#jump-hosts--proxyjump)
- [SSH Agent](#ssh-agent)
- [Practice Exercises](#-practice-exercises)

---

## What is SSH?

**SSH** (Secure Shell) provides encrypted remote access to Linux systems. It replaces insecure tools like telnet and rlogin.

```
┌──────────┐     Encrypted Tunnel     ┌──────────┐
│  CLIENT  │ ════════════════════════ │  SERVER  │
│ (laptop) │     Port 22 (default)    │ (remote) │
└──────────┘                          └──────────┘
```

---

## Basic SSH Usage

```bash
# Connect to a remote server
ssh user@hostname
ssh sovon@192.168.1.100
ssh sovon@myserver.com

# Connect on a different port
ssh -p 2222 user@hostname

# Run a single command remotely
ssh user@host "uptime"
ssh user@host "df -h && free -h"

# Connect with verbose output (debugging)
ssh -v user@host                 # Verbose
ssh -vv user@host                # More verbose
ssh -vvv user@host               # Maximum verbosity
```

---

## SSH Key Authentication

Keys are more secure and convenient than passwords.

### Step 1: Generate a Key Pair

```bash
ssh-keygen -t ed25519 -C "sovon@mypc"
# -t ed25519 = algorithm (recommended, faster, more secure)
# -C = comment (usually your email or identifier)

# You'll be asked:
# 1. File location (press Enter for default: ~/.ssh/id_ed25519)
# 2. Passphrase (recommended — adds another layer of security)

# Alternative: RSA key (if ed25519 not supported)
ssh-keygen -t rsa -b 4096 -C "sovon@mypc"
```

### Step 2: Copy Public Key to Server

```bash
# Automatic method (easiest)
ssh-copy-id user@server

# Manual method
cat ~/.ssh/id_ed25519.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Or copy and paste manually
cat ~/.ssh/id_ed25519.pub
# Copy the output, then on the server:
echo "paste-the-key-here" >> ~/.ssh/authorized_keys
```

### Step 3: Connect Without Password

```bash
ssh user@server     # No password needed!
```

### Key File Permissions (Required!)

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519         # Private key
chmod 644 ~/.ssh/id_ed25519.pub     # Public key
chmod 600 ~/.ssh/authorized_keys    # On server
chmod 600 ~/.ssh/config             # Config file
```

> 🚨 **SSH will refuse to work if your private key has wrong permissions.** Always set `600`.

---

## SSH Config File

Create `~/.ssh/config` to save connection settings:

```bash
# ~/.ssh/config

# Default settings for all hosts
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    AddKeysToAgent yes

# Named host — now just type: ssh web
Host web
    HostName 192.168.1.100
    User sovon
    Port 22
    IdentityFile ~/.ssh/id_ed25519

# Production server
Host prod
    HostName prod.example.com
    User deploy
    Port 2222
    IdentityFile ~/.ssh/prod_key

# Jump through bastion to internal server
Host internal
    HostName 10.0.1.50
    User admin
    ProxyJump bastion

Host bastion
    HostName bastion.example.com
    User sovon
```

```bash
# Now connect with short names
ssh web                  # Instead of: ssh sovon@192.168.1.100
ssh prod                 # Instead of: ssh -p 2222 deploy@prod.example.com
ssh internal             # Automatically jumps through bastion
```

---

## SSH Tunneling & Port Forwarding

### Local Port Forwarding

Access a remote service through a local port.

```bash
# Forward local port 8080 → remote server's port 80
ssh -L 8080:localhost:80 user@server

# Access remote database through tunnel
ssh -L 3307:localhost:3306 user@dbserver
# Now connect to MySQL via localhost:3307

# Access a service on a private network
ssh -L 8080:internal-server:80 user@bastion
```

### Remote Port Forwarding

Make a local service accessible remotely.

```bash
# Make local port 3000 accessible on server's port 9090
ssh -R 9090:localhost:3000 user@server
```

### Dynamic Port Forwarding (SOCKS Proxy)

```bash
# Create a SOCKS proxy on port 1080
ssh -D 1080 user@server
# Configure your browser to use SOCKS proxy localhost:1080
```

### Keep Tunnels Alive

```bash
# Foreground tunnel
ssh -L 8080:localhost:80 -N user@server    # -N = no command, just tunnel

# Background tunnel
ssh -L 8080:localhost:80 -NfT user@server  # -f = background, -T = no terminal
```

---

## SCP & File Transfer

```bash
# Copy local → remote
scp file.txt user@server:/home/user/
scp file.txt user@server:~/Documents/

# Copy remote → local
scp user@server:/var/log/syslog ./

# Copy directory (recursive)
scp -r project/ user@server:~/

# With specific port
scp -P 2222 file.txt user@server:~/

# Preserve timestamps & permissions
scp -rp directory/ user@server:~/

# Using compression
scp -C large-file.tar user@server:~/
```

---

## rsync — Smart Syncing

`rsync` only transfers **changes** — much faster than scp for repeated transfers.

```bash
# Basic sync (local → remote)
rsync -avz source/ user@server:~/destination/
# -a = archive (recursive + preserves permissions/timestamps)
# -v = verbose
# -z = compress during transfer

# Remote → local
rsync -avz user@server:~/data/ ./local-data/

# Dry run (preview changes without doing anything)
rsync -avzn source/ user@server:~/destination/

# Delete files on destination that don't exist in source
rsync -avz --delete source/ user@server:~/destination/

# Exclude patterns
rsync -avz --exclude='*.log' --exclude='.git/' source/ dest/

# Progress bar
rsync -avz --progress source/ user@server:~/dest/

# Bandwidth limit (KB/s)
rsync -avz --bwlimit=1000 source/ user@server:~/dest/

# Over custom SSH port
rsync -avz -e "ssh -p 2222" source/ user@server:~/dest/
```

> 💡 **Tip**: Trailing slash on source (`source/`) syncs **contents**. No slash (`source`) syncs the directory itself.

---

## SSH Security Best Practices

### Harden `/etc/ssh/sshd_config`

```bash
sudo vim /etc/ssh/sshd_config
```

```bash
# Disable root login
PermitRootLogin no

# Disable password authentication (use keys only)
PasswordAuthentication no

# Change default port (security through obscurity)
Port 2222

# Limit login attempts
MaxAuthTries 3

# Allow specific users only
AllowUsers sovon alice

# Disable empty passwords
PermitEmptyPasswords no

# Use SSH Protocol 2 only (default on modern systems)
Protocol 2

# Idle timeout
ClientAliveInterval 300
ClientAliveCountMax 2
```

```bash
# Apply changes
sudo systemctl restart sshd
```

### Use fail2ban

```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Check banned IPs
sudo fail2ban-client status sshd
```

---

## Jump Hosts & ProxyJump

Access internal servers through a bastion/jump host.

```bash
# Direct method
ssh -J user@bastion user@internal-server

# Multiple jumps
ssh -J user@jump1,user@jump2 user@target

# In SSH config (preferred)
Host internal
    HostName 10.0.1.50
    User admin
    ProxyJump bastion
```

---

## SSH Agent

The SSH agent stores your decrypted keys in memory so you don't type the passphrase repeatedly.

```bash
# Start the agent
eval "$(ssh-agent -s)"

# Add your key
ssh-add ~/.ssh/id_ed25519

# List loaded keys
ssh-add -l

# Remove all keys
ssh-add -D

# Auto-start (add to ~/.bashrc)
if [ -z "$SSH_AUTH_SOCK" ]; then
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_ed25519
fi
```

---

## 🏋️ Practice Exercises

1. **Keys**: Generate an ed25519 SSH key pair
2. **Copy**: Use `ssh-copy-id` to add your key to a server (or VM)
3. **Config**: Create `~/.ssh/config` with at least two host entries
4. **SCP**: Copy a file to a remote server and back
5. **rsync**: Sync a directory to a remote server, modify a file, sync again (note partial transfer)
6. **Tunnel**: Create a local port forward to access a remote service
7. **Harden**: Update `sshd_config` to disable root login and password auth
8. **Agent**: Start ssh-agent and add your key

---

<p align="center">
  <a href="../12-networking-fundamentals/README.md">← Previous: Networking Fundamentals</a> · <a href="../README.md">🏠 Home</a> · <a href="../14-cron-jobs-scheduling/README.md">Next: Cron Jobs & Scheduling →</a>
</p>
