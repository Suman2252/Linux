# 🛡️ Chapter 22: SELinux & AppArmor

<p align="center">
  <img src="https://img.shields.io/badge/Level-Advanced-orange?style=for-the-badge" alt="Advanced">
  <img src="https://img.shields.io/badge/Chapter-22%20of%2034-blue?style=for-the-badge" alt="Chapter 22">
</p>

---

## 📑 Table of Contents

- [MAC vs DAC](#mac-vs-dac)
- [SELinux](#selinux)
- [AppArmor](#apparmor)
- [SELinux vs AppArmor](#selinux-vs-apparmor)
- [Practice Exercises](#-practice-exercises)

---

## MAC vs DAC

| Type | Description | Example |
|------|-------------|---------|
| **DAC** (Discretionary) | Owner decides permissions | chmod, chown |
| **MAC** (Mandatory) | System policy decides permissions | SELinux, AppArmor |

> DAC: "You can share your files with anyone." MAC: "The policy decides what you can access, regardless of file ownership."

---

## SELinux

**SELinux** (Security-Enhanced Linux) — used by RHEL, Fedora, CentOS.

### Modes

| Mode | Description |
|------|-------------|
| **Enforcing** | Policies enforced, violations blocked + logged |
| **Permissive** | Policies NOT enforced, violations only logged |
| **Disabled** | SELinux completely off |

```bash
# Check status
getenforce                               # Current mode
sestatus                                 # Detailed status

# Change mode (temporary)
sudo setenforce 0                        # Permissive
sudo setenforce 1                        # Enforcing

# Change mode (permanent)
sudo vim /etc/selinux/config
# SELINUX=enforcing
```

### Contexts & Labels

```bash
# View file context
ls -Z /var/www/html/
# -rw-r--r--. root root unconfined_u:object_r:httpd_sys_content_t:s0 index.html
#                        user:role:type:level

# View process context
ps -eZ | grep httpd
# system_u:system_r:httpd_t:s0   1234 ?    00:00:01 httpd

# Change file context
sudo chcon -t httpd_sys_content_t /var/www/html/newfile.html

# Restore default context
sudo restorecon -Rv /var/www/html/

# Set default context for a path
sudo semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
sudo restorecon -Rv /web/
```

### Booleans

```bash
# List all booleans
getsebool -a | grep httpd

# Enable/disable boolean
sudo setsebool -P httpd_can_network_connect on    # -P = permanent
sudo setsebool -P httpd_enable_homedirs on
```

### Troubleshooting

```bash
# View SELinux denials
sudo ausearch -m avc -ts recent
sudo sealert -a /var/log/audit/audit.log

# Generate policy fix
sudo audit2allow -a                      # Show what was blocked
sudo audit2allow -a -M mypolicy          # Generate policy module
sudo semodule -i mypolicy.pp             # Install policy

# Common port issues
sudo semanage port -l | grep http
sudo semanage port -a -t http_port_t -p tcp 8080  # Allow httpd on port 8080
```

---

## AppArmor

**AppArmor** — used by Ubuntu, Debian, SUSE.

### Modes

| Mode | Description |
|------|-------------|
| **Enforce** | Restrictions active, violations blocked + logged |
| **Complain** | Restrictions logged but NOT enforced |
| **Disabled** | Profile not loaded |

```bash
# Check status
sudo aa-status                           # All profiles and their modes
sudo apparmor_status                     # Same

# List loaded profiles
sudo aa-status | grep profiles
```

### Managing Profiles

```bash
# Install utilities
sudo apt install apparmor-utils

# Switch profile modes
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx    # Enable enforcement
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx   # Complain mode
sudo aa-disable /etc/apparmor.d/usr.sbin.nginx    # Disable

# Generate new profile
sudo aa-genprof /path/to/application    # Interactive profile generator
sudo aa-logprof                          # Update profile from logs

# Reload profiles
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx
sudo systemctl reload apparmor
```

### Profile Syntax

```
# /etc/apparmor.d/opt.myapp
#include <tunables/global>

/opt/myapp/bin/server {
  #include <abstractions/base>
  #include <abstractions/nameservice>

  /opt/myapp/** r,                # Read app files
  /opt/myapp/data/** rw,          # Read/write data
  /opt/myapp/logs/** w,           # Write logs
  /var/log/myapp.log w,           # Write system log
  /etc/myapp.conf r,              # Read config
  /tmp/myapp-* rw,                # Temp files
  network tcp,                     # Allow TCP
  deny /etc/shadow r,             # Explicitly deny
}
```

### Troubleshooting

```bash
# View denials
sudo dmesg | grep -i apparmor
sudo journalctl -k | grep apparmor
cat /var/log/syslog | grep apparmor

# Temporarily disable for debugging
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx
# Test your application
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx
```

---

## SELinux vs AppArmor

| Feature | SELinux | AppArmor |
|---------|--------|----------|
| **Approach** | Labels (contexts) | Path-based rules |
| **Complexity** | Higher | Lower |
| **Default on** | RHEL, Fedora | Ubuntu, SUSE |
| **Granularity** | Very fine-grained | Moderate |
| **Learning curve** | Steep | Easier |
| **Coverage** | All files by default | Only profiled apps |

---

## 🏋️ Practice Exercises

1. **Check**: Determine if SELinux or AppArmor is active on your system
2. **Status**: View all loaded profiles/policies
3. **AppArmor**: Put a profile in complain mode, test, then enforce
4. **SELinux**: Change a file's context and verify with `ls -Z`
5. **Troubleshoot**: Read denial logs and understand what was blocked
6. **Profile**: Create a basic AppArmor profile for a script
7. **Boolean**: Enable an SELinux boolean for a web server

---

<p align="center">
  <a href="../21-security-hardening/README.md">← Previous: Security & Hardening</a> · <a href="../README.md">🏠 Home</a> · <a href="../23-performance-tuning/README.md">Next: Performance Tuning →</a>
</p>
