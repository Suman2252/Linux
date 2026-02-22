# 👥 Chapter 06: Users, Groups & Permissions

<p align="center">
  <img src="https://img.shields.io/badge/Level-Beginner-green?style=for-the-badge" alt="Beginner">
  <img src="https://img.shields.io/badge/Chapter-06%20of%2034-blue?style=for-the-badge" alt="Chapter 06">
</p>

---

## 📑 Table of Contents

- [Understanding Users](#understanding-users)
- [Managing Users](#managing-users)
- [Understanding Groups](#understanding-groups)
- [Managing Groups](#managing-groups)
- [File Permissions Explained](#file-permissions-explained)
- [Changing Permissions — chmod](#changing-permissions--chmod)
- [Changing Ownership — chown & chgrp](#changing-ownership--chown--chgrp)
- [Special Permissions (SUID, SGID, Sticky Bit)](#special-permissions-suid-sgid-sticky-bit)
- [Access Control Lists (ACLs)](#access-control-lists-acls)
- [The sudo System](#the-sudo-system)
- [Practice Exercises](#-practice-exercises)

---

## Understanding Users

Linux is a **multi-user** system. Every user has:

| Attribute | Description | Example |
|-----------|-------------|---------|
| **Username** | Human-readable name | `sovon` |
| **UID** | Unique numeric identifier | `1000` |
| **Home Directory** | Personal file space | `/home/sovon` |
| **Default Shell** | Login shell | `/bin/bash` |
| **Password** | Authentication (stored in `/etc/shadow`) | `*` (hashed) |

### Types of Users

| Type | UID Range | Examples | Purpose |
|------|-----------|----------|---------|
| **Root** | 0 | `root` | Superuser — unlimited power |
| **System** | 1–999 | `www-data`, `mysql`, `nobody` | Services and daemons |
| **Regular** | 1000+ | `sovon`, `alice` | Normal human users |

### Key Files

```bash
# User accounts
cat /etc/passwd
# Format: username:password:UID:GID:comment:home:shell
# sovon:x:1000:1000:Sovon:/home/sovon:/bin/bash

# Encrypted passwords (only root can read)
sudo cat /etc/shadow
# sovon:$6$abc123...:19200:0:99999:7:::

# Group definitions
cat /etc/group
# sovon:x:1000:
# docker:x:998:sovon
```

---

## Managing Users

### Create Users

```bash
# Create a user with defaults
sudo useradd alice

# Create with all options (recommended)
sudo useradd -m -s /bin/bash -c "Alice Smith" -G sudo,docker alice
#  -m  → Create home directory
#  -s  → Set login shell
#  -c  → Comment/full name
#  -G  → Additional groups

# Interactive user creation (Debian/Ubuntu)
sudo adduser bob          # Walks you through the process
```

### Modify Users

```bash
sudo usermod -aG docker alice     # Add to group (-a = append, -G = groups)
sudo usermod -l newname oldname   # Rename user
sudo usermod -s /bin/zsh alice    # Change shell
sudo usermod -L alice             # Lock account (disable login)
sudo usermod -U alice             # Unlock account
sudo usermod -d /home/newhome alice  # Change home directory
```

### Delete Users

```bash
sudo userdel alice               # Delete user (keep home directory)
sudo userdel -r alice            # Delete user AND home directory
```

### Set/Change Passwords

```bash
passwd                           # Change your own password
sudo passwd alice                # Change another user's password (root only)
sudo passwd -l alice             # Lock password
sudo passwd -u alice             # Unlock password
sudo passwd -e alice             # Force password change at next login
sudo chage -l alice              # View password aging info
sudo chage -M 90 alice           # Max password age = 90 days
```

### Switch Users

```bash
su - alice                       # Switch to alice (need alice's password)
su -                             # Switch to root
sudo -i                          # Root shell via sudo
sudo -u alice whoami             # Run command as alice via sudo
whoami                           # Who am I right now?
id                               # Detailed identity info (UID, GID, groups)
```

---

## Understanding Groups

**Groups** allow you to manage permissions for multiple users at once.

> 🏠 **Analogy**: Groups are like departments in a company. Everyone in the "Engineering" group can access engineering files.

Every user has:
- **Primary group** (one) — matches their username by default
- **Secondary groups** (many) — additional access rights

---

## Managing Groups

```bash
# Create a group
sudo groupadd developers

# Delete a group
sudo groupdel developers

# Add user to group
sudo usermod -aG developers sovon    # -a = append (important!)
sudo gpasswd -a sovon developers     # Alternative method

# Remove user from group
sudo gpasswd -d sovon developers

# View groups
groups                   # Your groups
groups alice             # Alice's groups
id sovon                 # Detailed group info

# List all groups
cat /etc/group

# Change primary group
sudo usermod -g developers sovon
```

> ⚠️ **Important**: Always use `-aG` (with the `-a` flag) when adding groups. Without `-a`, you'll **replace** all existing groups!

---

## File Permissions Explained

```bash
ls -l myfile.txt
# -rw-r--r-- 1 sovon developers 4096 Feb 22 10:00 myfile.txt
```

### Breaking Down Permissions

```
- rw- r-- r--
│ │   │   │
│ │   │   └── Others (everyone else)
│ │   └── Group (users in the file's group)
│ └── Owner/User (the file's owner)
└── File type (- = file, d = directory, l = link)
```

### Permission Meanings

| Symbol | Numeric | For Files | For Directories |
|--------|---------|-----------|-----------------|
| `r` | 4 | Read contents | List contents (`ls`) |
| `w` | 2 | Modify contents | Add/delete files inside |
| `x` | 1 | Execute as program | Enter the directory (`cd`) |
| `-` | 0 | No permission | No permission |

### Numeric (Octal) Representation

```
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
--- = 0+0+0 = 0
```

### Common Permission Patterns

| Numeric | Symbolic | Meaning | Use Case |
|---------|----------|---------|----------|
| `755` | `rwxr-xr-x` | Owner: full, Others: read+execute | Scripts, directories |
| `644` | `rw-r--r--` | Owner: read+write, Others: read | Regular files |
| `700` | `rwx------` | Owner only | Private directories |
| `600` | `rw-------` | Owner only | Private files, SSH keys |
| `777` | `rwxrwxrwx` | Everyone: full access | ⚠️ Insecure! Avoid! |

---

## Changing Permissions — chmod

### Symbolic Mode

```bash
# Format: chmod [who][+/-/=][permissions] file

# Who: u=user/owner, g=group, o=others, a=all
# Operators: +=add, -=remove, ==set exactly

chmod u+x script.sh             # Add execute for owner
chmod g+rw file.txt             # Add read+write for group
chmod o-rwx secret.txt          # Remove all permissions for others
chmod a+r public.txt            # Add read for everyone
chmod u=rwx,g=rx,o=r file.txt   # Set exact permissions
chmod ug+x script.sh            # Add execute for user and group
```

### Numeric (Octal) Mode

```bash
chmod 755 script.sh             # rwxr-xr-x
chmod 644 file.txt              # rw-r--r--
chmod 600 ~/.ssh/id_rsa         # rw-------  (SSH keys MUST be 600)
chmod 700 ~/private/            # rwx------
chmod 777 /tmp/shared           # rwxrwxrwx  (avoid in production!)
```

### Recursive Permissions

```bash
chmod -R 755 project/           # Apply to directory and ALL contents
chmod -R u+rwX project/         # X = execute only for directories (smart!)
```

> 💡 **Pro Tip**: Capital `X` sets execute permission only on directories and files that already have execute for some user. This is perfect for setting directory permissions without making all files executable.

---

## Changing Ownership — chown & chgrp

```bash
# Change owner
sudo chown alice file.txt                 # Change owner to alice
sudo chown alice:developers file.txt      # Change owner AND group
sudo chown :developers file.txt           # Change group only

# Change recursively
sudo chown -R alice:developers project/   # All files and subdirectories

# chgrp — change group only
sudo chgrp developers file.txt
sudo chgrp -R www-data /var/www/
```

---

## Special Permissions (SUID, SGID, Sticky Bit)

### SUID (Set User ID) — Numeric: `4`

When set on a file, it **executes as the file's owner**, not the user running it.

```bash
# Example: passwd command needs to modify /etc/shadow (root-owned)
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root 68208 Feb 22 10:00 /usr/bin/passwd
#    ^
#    s = SUID is set (executes as root)

# Set SUID
chmod u+s /usr/local/bin/myprogram
chmod 4755 /usr/local/bin/myprogram
```

### SGID (Set Group ID) — Numeric: `2`

On **files**: executes with the group's permissions.
On **directories**: new files inherit the directory's group.

```bash
# Set SGID on a shared project directory
chmod g+s /shared/project/
chmod 2775 /shared/project/

# Now any file created inside inherits the "project" group
ls -ld /shared/project/
# drwxrwsr-x 2 root project 4096 Feb 22 10:00 /shared/project/
#       ^
#       s = SGID is set
```

### Sticky Bit — Numeric: `1`

On directories: only the **file owner** can delete their own files (even if others have write access).

```bash
# /tmp uses sticky bit so users can't delete each other's files
ls -ld /tmp
# drwxrwxrwt 15 root root 4096 Feb 22 10:00 /tmp
#          ^
#          t = sticky bit is set

# Set sticky bit
chmod +t /shared/
chmod 1777 /shared/
```

### Summary Table

| Permission | Numeric | On Files | On Directories |
|-----------|---------|----------|----------------|
| **SUID** | `4xxx` | Run as file owner | (no effect) |
| **SGID** | `2xxx` | Run as file group | New files inherit group |
| **Sticky** | `1xxx` | (rare) | Only owner can delete files |

---

## Access Control Lists (ACLs)

Standard permissions only allow one user and one group. **ACLs** provide fine-grained control.

```bash
# Install ACL support
sudo apt install acl

# View ACLs
getfacl file.txt

# Grant read+write to a specific user
setfacl -m u:alice:rw file.txt

# Grant read to a specific group
setfacl -m g:developers:r file.txt

# Remove an ACL entry
setfacl -x u:alice file.txt

# Remove ALL ACLs
setfacl -b file.txt

# Set default ACLs for a directory (new files inherit)
setfacl -d -m g:developers:rwx /shared/project/

# An ACL-enabled file shows a + in ls -l
ls -l file.txt
# -rw-rw-r--+ 1 sovon sovon 100 Feb 22 10:00 file.txt
#           ^
#           + indicates ACL is set
```

---

## The sudo System

**sudo** = "superuser do" — run a command as root (or another user).

```bash
# Run a command as root
sudo apt update

# Open a root shell
sudo -i                          # Login shell
sudo -s                          # Non-login shell

# Run as another user
sudo -u alice cat /home/alice/file.txt

# Edit files safely as root
sudo -e /etc/hosts               # Opens in $EDITOR with sudo
sudoedit /etc/hosts              # Same as above (safer than sudo vim)

# List your sudo privileges
sudo -l
```

### Configuring sudo — `/etc/sudoers`

```bash
# ALWAYS use visudo to edit sudoers (it checks syntax!)
sudo visudo

# Grant full sudo access to a user
sovon ALL=(ALL:ALL) ALL

# Allow sudo without password
sovon ALL=(ALL) NOPASSWD: ALL

# Allow specific commands only
alice ALL=(ALL) /usr/bin/apt update, /usr/bin/apt upgrade

# Allow a group
%developers ALL=(ALL) ALL
```

> ⚠️ **Never edit `/etc/sudoers` directly with a text editor.** Always use `visudo` — it prevents syntax errors that could lock you out.

---

## 🏋️ Practice Exercises

1. **Create a user** called `testuser` with a home directory and bash shell
2. **Create a group** called `devteam` and add your user to it
3. **Create a file** and set permissions to `640` — who can do what?
4. **SUID**: Find all SUID files on your system: `find / -perm -u=s -type f 2>/dev/null`
5. **Sticky Bit**: Verify `/tmp` has the sticky bit set
6. **ACL**: Grant read access to `testuser` on a specific file using ACLs
7. **Ownership**: Create a directory owned by `devteam` group with SGID set
8. **sudo**: Check your sudo privileges with `sudo -l`

---

<p align="center">
  <a href="../05-file-directory-operations/README.md">← Previous: File & Directory Operations</a> · <a href="../README.md">🏠 Home</a> · <a href="../07-package-management/README.md">Next: Package Management →</a>
</p>
