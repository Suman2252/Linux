# 📦 Chapter 16: Archive, Compression & Backup

<p align="center">
  <img src="https://img.shields.io/badge/Level-Intermediate-yellow?style=for-the-badge" alt="Intermediate">
  <img src="https://img.shields.io/badge/Chapter-16%20of%2034-blue?style=for-the-badge" alt="Chapter 16">
</p>

---

## 📑 Table of Contents

- [Archiving with tar](#archiving-with-tar)
- [Compression Tools](#compression-tools)
- [zip & unzip](#zip--unzip)
- [Backup Strategies](#backup-strategies)
- [rsync for Backups](#rsync-for-backups)
- [Borgbackup — Deduplicated Backups](#borgbackup--deduplicated-backups)
- [dd — Disk Cloning](#dd--disk-cloning)
- [Practice Exercises](#-practice-exercises)

---

## Archiving with tar

**tar** (Tape Archive) bundles files into a single archive.

```bash
# Create archive
tar -cf archive.tar files/              # Create (no compression)
tar -czf archive.tar.gz files/          # Create + gzip
tar -cjf archive.tar.bz2 files/        # Create + bzip2
tar -cJf archive.tar.xz files/         # Create + xz (best ratio)

# Extract archive
tar -xf archive.tar                     # Extract
tar -xf archive.tar -C /destination/    # Extract to specific dir
tar -xzf archive.tar.gz                 # Extract gzipped
tar -xjf archive.tar.bz2               # Extract bzip2
tar -xJf archive.tar.xz                # Extract xz

# List contents (without extracting)
tar -tf archive.tar                     # List files
tar -tzf archive.tar.gz                 # List gzipped archive

# Verbose
tar -cvzf archive.tar.gz files/        # Show files being archived

# Add to existing archive
tar -rf archive.tar newfile.txt         # Append file

# Exclude files
tar -czf backup.tar.gz --exclude='*.log' --exclude='.git' project/

# Preserve permissions
tar -cpzf backup.tar.gz /etc/          # -p preserves permissions
```

### tar Flags Cheatsheet

| Flag | Meaning |
|------|---------|
| `-c` | Create archive |
| `-x` | Extract archive |
| `-t` | List contents |
| `-f` | Specify filename |
| `-v` | Verbose |
| `-z` | gzip compression |
| `-j` | bzip2 compression |
| `-J` | xz compression |
| `-C` | Change directory |
| `-p` | Preserve permissions |
| `-r` | Append to archive |
| `--exclude` | Exclude pattern |

---

## Compression Tools

### Comparison

| Tool | Extension | Speed | Ratio | Best For |
|------|-----------|-------|-------|----------|
| **gzip** | `.gz` | ⚡ Fast | Good | Daily use, web |
| **bzip2** | `.bz2` | 🐢 Slow | Better | Archiving |
| **xz** | `.xz` | 🐌 Slowest | Best | Distribution, long-term |
| **zstd** | `.zst` | ⚡⚡ Fastest | Great | Modern, balanced |
| **lz4** | `.lz4` | ⚡⚡⚡ Fastest | Lower | Real-time compression |

### gzip

```bash
gzip file.txt                   # Compress → file.txt.gz (original deleted)
gzip -k file.txt                # Keep original
gzip -d file.txt.gz             # Decompress
gunzip file.txt.gz              # Same as gzip -d
gzip -9 file.txt                # Maximum compression
gzip -1 file.txt                # Fastest compression
zcat file.txt.gz                # View without decompressing
```

### bzip2

```bash
bzip2 file.txt                  # Compress → file.txt.bz2
bzip2 -d file.txt.bz2           # Decompress
bunzip2 file.txt.bz2            # Same
bzip2 -k file.txt               # Keep original
bzcat file.txt.bz2              # View without decompressing
```

### xz

```bash
xz file.txt                     # Compress → file.txt.xz
xz -d file.txt.xz               # Decompress
unxz file.txt.xz                # Same
xz -k file.txt                  # Keep original
xzcat file.txt.xz               # View without decompressing
xz -9 file.txt                  # Maximum compression
xz -T 0 file.txt                # Use all CPU threads
```

### zstd (Modern — Recommended)

```bash
sudo apt install zstd
zstd file.txt                   # Compress → file.txt.zst
zstd -d file.txt.zst            # Decompress
zstd --rm file.txt              # Remove original after compress
zstd -19 file.txt               # Maximum compression
zstd -T0 file.txt               # Use all threads
```

---

## zip & unzip

Essential for cross-platform compatibility (Windows, Mac, Linux).

```bash
# Create zip
zip archive.zip file1.txt file2.txt
zip -r archive.zip directory/          # Recursive (directories)
zip -e secure.zip files/               # Encrypted (with password)
zip -9 max.zip file.txt               # Maximum compression

# Extract
unzip archive.zip
unzip archive.zip -d /destination/
unzip -l archive.zip                   # List contents
unzip -o archive.zip                   # Overwrite without prompt

# Update zip
zip -u archive.zip newfile.txt         # Add/update file in zip
```

---

## Backup Strategies

### The 3-2-1 Rule

| Rule | Description |
|------|-------------|
| **3** copies | Keep 3 copies of your data |
| **2** media types | Store on 2 different storage types |
| **1** offsite | Keep 1 copy in a different location |

### Types of Backups

| Type | Description | Speed | Space |
|------|-------------|-------|-------|
| **Full** | Copy everything | 🐢 Slow | 💾 Large |
| **Incremental** | Only changes since LAST backup | ⚡ Fast | 💾 Small |
| **Differential** | Changes since last FULL backup | 🔄 Medium | 💾 Medium |

---

## rsync for Backups

```bash
# Local backup
rsync -avz --delete /home/sovon/ /backup/home-sovon/

# Remote backup
rsync -avz --delete /home/sovon/ user@backup-server:~/backup/

# Incremental backup with date
rsync -avz /data/ /backup/data-$(date +%Y%m%d)/

# Backup with exclusions
rsync -avz \
  --exclude='.cache' \
  --exclude='node_modules' \
  --exclude='*.tmp' \
  /home/sovon/ /backup/home/

# Backup script
#!/bin/bash
SRC="/home/sovon/"
DST="/backup/home-$(date +%Y%m%d_%H%M)"
rsync -avz --delete "$SRC" "$DST" >> /var/log/backup.log 2>&1
echo "Backup completed: $DST"
```

---

## Borgbackup — Deduplicated Backups

**Borg** is a deduplicated, encrypted backup tool — the gold standard for Linux backups.

```bash
# Install
sudo apt install borgbackup

# Initialize repository
borg init --encryption=repokey /backup/borg-repo

# Create a backup
borg create /backup/borg-repo::backup-{now:%Y-%m-%d} \
  /home/sovon \
  --exclude '/home/sovon/.cache' \
  --exclude '/home/sovon/Downloads' \
  --progress --stats

# List archives
borg list /backup/borg-repo

# Restore
borg extract /backup/borg-repo::backup-2024-02-22

# Prune old backups
borg prune /backup/borg-repo \
  --keep-daily=7 \
  --keep-weekly=4 \
  --keep-monthly=6

# Check integrity
borg check /backup/borg-repo

# Borg info
borg info /backup/borg-repo
borg info /backup/borg-repo::backup-2024-02-22
```

---

## dd — Disk Cloning

**dd** copies data at the bit level — perfect for disk cloning and ISOs.

```bash
# Clone an entire disk
sudo dd if=/dev/sda of=/dev/sdb bs=64K status=progress

# Create disk image
sudo dd if=/dev/sda of=/backup/disk-image.img bs=4M status=progress

# Restore disk image
sudo dd if=/backup/disk-image.img of=/dev/sda bs=4M status=progress

# Create a bootable USB from ISO
sudo dd if=ubuntu.iso of=/dev/sdb bs=4M status=progress oflag=sync

# Wipe a disk (write zeros)
sudo dd if=/dev/zero of=/dev/sdb bs=4M status=progress

# Create a file of specific size (for testing)
dd if=/dev/zero of=testfile bs=1M count=100
```

> 🚨 **dd is called "disk destroyer" for a reason.** Double-check `if=` (input) and `of=` (output) before pressing Enter. A wrong `of=` can wipe your main drive!

---

## 🏋️ Practice Exercises

1. **tar**: Create a gzipped archive of your home directory
2. **Compare**: Compress a large file with gzip, bzip2, and xz — compare sizes
3. **zip**: Create an encrypted zip archive
4. **rsync**: Set up a backup script using rsync with exclusions
5. **Restore**: Practice extracting archives to specific directories
6. **dd**: Create a 100MB test file using `dd`
7. **Borg**: Set up a basic borgbackup repository and create your first backup
8. **cron + backup**: Schedule a daily backup with cron

---

<p align="center">
  <a href="../15-system-monitoring-logs/README.md">← Previous: System Monitoring & Logs</a> · <a href="../README.md">🏠 Home</a> · <a href="../17-advanced-shell-scripting/README.md">Next: Advanced Shell Scripting →</a>
</p>
