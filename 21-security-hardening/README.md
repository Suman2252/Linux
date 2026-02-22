# 🔒 Chapter 21: Security & Hardening

<p align="center">
  <img src="https://img.shields.io/badge/Level-Advanced-orange?style=for-the-badge" alt="Advanced">
  <img src="https://img.shields.io/badge/Chapter-21%20of%2034-blue?style=for-the-badge" alt="Chapter 21">
</p>

---

## 📑 Table of Contents

- [Security Principles](#security-principles)
- [SSH Hardening](#ssh-hardening)
- [Firewall Configuration](#firewall-configuration)
- [fail2ban — Intrusion Prevention](#fail2ban--intrusion-prevention)
- [User Security](#user-security)
- [File Integrity Monitoring](#file-integrity-monitoring)
- [System Auditing](#system-auditing)
- [CIS Benchmarks](#cis-benchmarks)
- [Security Scanning](#security-scanning)
- [Practice Exercises](#-practice-exercises)

---

## Security Principles

| Principle | Description |
|-----------|-------------|
| **Least Privilege** | Give minimum permissions required |
| **Defense in Depth** | Multiple layers of security |
| **Fail Secure** | On failure, deny access |
| **Keep Updated** | Patch regularly |
| **Minimize Attack Surface** | Remove unnecessary software/services |

---

## SSH Hardening

```bash
sudo vim /etc/ssh/sshd_config
```

```bash
# Essential hardening
Port 2222                             # Change default port
PermitRootLogin no                    # No root login
PasswordAuthentication no             # Keys only
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
MaxAuthTries 3
LoginGraceTime 30
PermitEmptyPasswords no
X11Forwarding no
AllowUsers sovon deploy               # Whitelist users
ClientAliveInterval 300
ClientAliveCountMax 2
Protocol 2

# Restrict ciphers and algorithms
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
```

```bash
sudo systemctl restart sshd
```

---

## Firewall Configuration

```bash
# Minimal server firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp comment 'SSH'
sudo ufw allow 80/tcp comment 'HTTP'
sudo ufw allow 443/tcp comment 'HTTPS'
sudo ufw enable
sudo ufw status verbose
```

---

## fail2ban — Intrusion Prevention

```bash
sudo apt install fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vim /etc/fail2ban/jail.local
```

```ini
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3
banaction = ufw

[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 86400
```

```bash
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd          # Check status
sudo fail2ban-client set sshd unbanip 1.2.3.4  # Unban IP
```

---

## User Security

```bash
# Password policies
sudo apt install libpam-pwquality
sudo vim /etc/security/pwquality.conf
# minlen = 12
# dcredit = -1    (require digit)
# ucredit = -1    (require uppercase)
# lcredit = -1    (require lowercase)
# ocredit = -1    (require special char)

# Password aging
sudo chage -M 90 -W 7 -m 1 username
# -M = max days, -W = warn days, -m = min days

# Lock inactive accounts
sudo useradd -f 30 username             # Lock after 30 days inactive

# Find users with empty passwords
sudo awk -F: '($2 == "" ) {print $1}' /etc/shadow

# Find SUID/SGID files (potential security risk)
find / -perm -4000 -type f 2>/dev/null   # SUID
find / -perm -2000 -type f 2>/dev/null   # SGID

# Find world-writable files
find / -perm -o+w -type f 2>/dev/null

# Find files with no owner
find / -nouser -o -nogroup 2>/dev/null
```

---

## File Integrity Monitoring

### AIDE (Advanced Intrusion Detection Environment)

```bash
sudo apt install aide

# Initialize database
sudo aideinit
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Check for changes
sudo aide --check

# Update database after known changes
sudo aide --update
```

---

## System Auditing

```bash
sudo apt install auditd
sudo systemctl enable --now auditd

# Watch a file for changes
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
sudo auditctl -w /etc/shadow -p wa -k shadow_changes
sudo auditctl -w /etc/ssh/sshd_config -p wa -k sshd_config

# Monitor command execution
sudo auditctl -a always,exit -F arch=b64 -S execve -k exec_commands

# View audit logs
sudo ausearch -k passwd_changes
sudo aureport --auth                    # Authentication report
sudo aureport --login                   # Login report
sudo aureport --failed                  # Failed events

# Permanent rules
sudo vim /etc/audit/rules.d/audit.rules
```

---

## CIS Benchmarks

```bash
# Install Lynis — security auditing tool
sudo apt install lynis

# Run system audit
sudo lynis audit system

# View suggestions
sudo cat /var/log/lynis-report.dat | grep suggestion

# Check specific category
sudo lynis audit system --tests-from-group "firewalls"
```

---

## Security Scanning

```bash
# Check for rootkits
sudo apt install rkhunter chkrootkit
sudo rkhunter --check
sudo chkrootkit

# Network vulnerability scanning
sudo apt install nmap
nmap -sV --script vuln localhost

# Check open ports
ss -tuln
sudo netstat -tlnp

# Check listening services
sudo lsof -i -P -n | grep LISTEN
```

### Automatic Security Updates

```bash
# Ubuntu/Debian
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades  # Enable automatic security updates

# Verify
cat /etc/apt/apt.conf.d/20auto-upgrades
```

---

## 🏋️ Practice Exercises

1. **SSH**: Harden sshd_config with at least 5 security changes
2. **fail2ban**: Set up fail2ban for SSH and test with wrong passwords
3. **Firewall**: Configure ufw with only necessary ports open
4. **Audit**: Set up auditd to monitor `/etc/passwd` changes
5. **Lynis**: Run a Lynis audit and fix the top 5 recommendations
6. **SUID**: Find all SUID files on your system and research each
7. **Updates**: Enable automatic security updates
8. **Scan**: Scan your system with rkhunter

---

<p align="center">
  <a href="../20-advanced-networking/README.md">← Previous: Advanced Networking</a> · <a href="../README.md">🏠 Home</a> · <a href="../22-selinux-apparmor/README.md">Next: SELinux & AppArmor →</a>
</p>
