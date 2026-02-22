# 📂 Chapter 05: File & Directory Operations

<p align="center">
  <img src="https://img.shields.io/badge/Level-Beginner-green?style=for-the-badge" alt="Beginner">
  <img src="https://img.shields.io/badge/Chapter-05%20of%2034-blue?style=for-the-badge" alt="Chapter 05">
</p>

---

## 📑 Table of Contents

- [Creating Files & Directories](#creating-files--directories)
- [Copying Files & Directories](#copying-files--directories)
- [Moving & Renaming](#moving--renaming)
- [Deleting Files & Directories](#deleting-files--directories)
- [Viewing File Contents](#viewing-file-contents)
- [Finding Files](#finding-files)
- [Links — Hard & Symbolic](#links--hard--symbolic)
- [Wildcards & Globbing](#wildcards--globbing)
- [File Metadata](#file-metadata)
- [Practice Exercises](#-practice-exercises)

---

## Creating Files & Directories

### Create Files

```bash
touch newfile.txt               # Create empty file (or update timestamp)
touch file1.txt file2.txt       # Create multiple files at once
touch /tmp/test-{1..10}.txt     # Create test-1.txt through test-10.txt

echo "Hello" > hello.txt        # Create file with content (overwrites)
echo "World" >> hello.txt       # Append to file

cat > notes.txt << EOF          # Multi-line content using heredoc
Line 1: This is my notes file
Line 2: Learning Linux is fun
EOF

printf "Name: %s\nAge: %d\n" "Sovon" 25 > info.txt
```

### Create Directories

```bash
mkdir mydir                     # Create a single directory
mkdir dir1 dir2 dir3            # Create multiple directories
mkdir -p parent/child/grandchild  # Create nested directories (-p = parents)
mkdir -pv project/{src,docs,tests}  # Create project structure (-v = verbose)
```

```bash
# Create a complete project structure in one command
mkdir -pv myproject/{src/{main,utils},docs,tests,config}
```

---

## Copying Files & Directories

### `cp` — Copy

```bash
cp file.txt backup.txt              # Copy file
cp file.txt /tmp/                   # Copy to another directory
cp file.txt /tmp/newname.txt        # Copy with new name
cp -i file.txt backup.txt           # Prompt before overwrite (-i = interactive)
cp -v file.txt backup.txt           # Show what's being done (-v = verbose)
cp -p file.txt backup.txt           # Preserve timestamps & permissions

# Copy directories (MUST use -r)
cp -r mydir/ mydir-backup/          # Copy directory recursively
cp -rv /etc/nginx/ /tmp/nginx-bak/  # Copy with verbose output
cp -a source/ dest/                 # Archive mode (preserves everything: -r + -p + links)
```

> 🚨 **Common Mistake**: Forgetting `-r` when copying directories will give an error.

---

## Moving & Renaming

### `mv` — Move and Rename

```bash
# Move files
mv file.txt /tmp/                   # Move to /tmp
mv file.txt /tmp/newname.txt        # Move and rename
mv *.log /var/log/                  # Move all .log files

# Rename (same directory = rename)
mv oldname.txt newname.txt          # Rename a file
mv old-dir/ new-dir/               # Rename a directory

# Safety options
mv -i source dest                   # Prompt before overwrite
mv -v source dest                   # Verbose output
mv -n source dest                   # Never overwrite (no-clobber)

# Move multiple files to a directory
mv file1.txt file2.txt file3.txt /backup/
```

> 💡 **Tip**: `mv` works on both files **and** directories — no `-r` flag needed (unlike `cp`).

---

## Deleting Files & Directories

### `rm` — Remove

```bash
rm file.txt                     # Delete a file (permanent!)
rm -i file.txt                  # Prompt before deleting
rm -v file.txt                  # Verbose
rm file1.txt file2.txt          # Delete multiple files
rm *.tmp                        # Delete all .tmp files

# Delete directories
rm -r mydir/                    # Delete directory and contents recursively
rm -ri mydir/                   # Recursive with confirmation for each file
rm -rf mydir/                   # Force delete (no prompts, no errors)
```

> 🚨 **DANGER ZONE**: `rm -rf /` will **destroy your entire system**. There is NO recycle bin, NO undo. Always double-check before using `rm -rf`.

### `rmdir` — Remove Empty Directories

```bash
rmdir emptydir                  # Only works if directory is empty
rmdir -p parent/child/grandchild  # Remove nested empty directories
```

### Safer Alternatives

```bash
# Use trash-cli instead of rm
sudo apt install trash-cli
trash-put file.txt              # Move to trash
trash-list                      # List trashed files
trash-restore                   # Restore a file
trash-empty                     # Empty the trash
```

---

## Viewing File Contents

| Command | Use Case | Memory Usage |
|---------|----------|-------------|
| `cat` | Small files | Loads entire file |
| `less` | Large files | Loads page by page |
| `more` | Large files | Older, less features |
| `head` | First N lines | Minimal |
| `tail` | Last N lines | Minimal |
| `nl` | With line numbers | Loads entire file |

```bash
# cat — concatenate and display
cat file.txt                    # Display entire file
cat -n file.txt                 # Show with line numbers
cat -A file.txt                 # Show hidden characters (tabs, line endings)
cat file1.txt file2.txt > merged.txt  # Concatenate files

# less — paginated viewing (most useful for large files)
less /var/log/syslog
# Navigation: Space=next page, b=back, /=search, q=quit, g=top, G=bottom

# head — first lines
head file.txt                   # First 10 lines (default)
head -n 5 file.txt              # First 5 lines
head -c 100 file.txt            # First 100 bytes

# tail — last lines
tail file.txt                   # Last 10 lines (default)
tail -n 20 file.txt             # Last 20 lines
tail -f /var/log/syslog         # Follow file in real-time (great for logs!)
tail -F /var/log/syslog         # Follow even if file is rotated

# nl — number lines
nl file.txt                     # Number non-empty lines
nl -ba file.txt                 # Number ALL lines

# Other viewers
tac file.txt                    # cat in reverse (last line first)
rev file.txt                    # Reverse each line
wc file.txt                     # Count lines, words, characters
wc -l file.txt                  # Count lines only
```

---

## Finding Files

### `find` — The Powerful Finder

```bash
# Basic syntax: find [path] [criteria] [action]

# Find by name
find / -name "passwd"                     # Exact name (case-sensitive)
find / -iname "readme.md"                 # Case-insensitive
find /home -name "*.txt"                  # All .txt files
find . -name "*.log" -o -name "*.tmp"     # .log OR .tmp files

# Find by type
find /var -type f                         # Files only
find /var -type d                         # Directories only
find /dev -type l                         # Symbolic links only

# Find by size
find / -size +100M                        # Larger than 100 MB
find / -size -1k                          # Smaller than 1 KB
find . -size 0                            # Empty files
find /var -type f -empty                  # Empty files

# Find by time
find / -mtime -7                          # Modified in last 7 days
find / -atime +30                         # Accessed more than 30 days ago
find / -mmin -60                          # Modified in last 60 minutes
find / -newer reference.txt              # Newer than reference file

# Find by permissions
find / -perm 777                          # Exact permission 777
find / -perm -u=x                         # User has execute permission
find /tmp -writable                       # Writable by current user

# Find by owner
find /home -user sovon                    # Owned by user
find / -group docker                      # Owned by group

# Actions on found files
find . -name "*.tmp" -delete              # Delete found files
find . -name "*.sh" -exec chmod +x {} \;  # Make all .sh files executable
find . -name "*.log" -exec gzip {} \;     # Compress all log files
find . -type f -name "*.txt" -exec grep -l "error" {} \;  # Find files containing "error"
```

### `locate` — Fast Filename Search

```bash
# Uses a pre-built database (faster than find, but not real-time)
sudo apt install mlocate
sudo updatedb                             # Update the database

locate passwd                             # Find all files with "passwd" in the path
locate -i readme                          # Case-insensitive search
locate -c "*.conf"                        # Count matches
locate --regex "\.conf$"                  # Use regex
```

### `which`, `whereis`, `type`

```bash
which python3          # Full path to executable
whereis ls             # Binary, source, and man page locations
type cd                # Show if built-in, alias, or binary
```

---

## Links — Hard & Symbolic

### Symbolic Links (Soft Links)

A **symlink** is a pointer to another file — like a shortcut.

```bash
ln -s /path/to/original /path/to/link

# Example
ln -s /var/log/syslog ~/syslog-shortcut
ls -l ~/syslog-shortcut
# lrwxrwxrwx 1 sovon sovon 15 Feb 22 10:00 syslog-shortcut -> /var/log/syslog
```

### Hard Links

A **hard link** is a direct reference to the same data on disk.

```bash
ln original.txt hardlink.txt

# Both point to the same inode (data on disk)
ls -li original.txt hardlink.txt
# 12345 -rw-r--r-- 2 sovon sovon 100 Feb 22 10:00 original.txt
# 12345 -rw-r--r-- 2 sovon sovon 100 Feb 22 10:00 hardlink.txt
```

### Symlink vs Hard Link

| Feature | Symbolic Link | Hard Link |
|---------|--------------|-----------|
| Can link to directories | ✅ Yes | ❌ No |
| Can cross filesystems | ✅ Yes | ❌ No |
| Breaks if target deleted | ✅ Yes (dangling link) | ❌ No (data remains) |
| Has its own inode | ✅ Yes | ❌ No (shares inode) |
| Shows as link in ls -l | ✅ `l` prefix | ❌ Regular `-` prefix |

---

## Wildcards & Globbing

**Wildcards** (globbing patterns) let you match multiple files:

| Pattern | Matches | Example |
|---------|---------|---------|
| `*` | Any characters (zero or more) | `*.txt` → all .txt files |
| `?` | Any single character | `file?.txt` → file1.txt, fileA.txt |
| `[abc]` | Any one of a, b, or c | `file[123].txt` → file1.txt, file2.txt |
| `[a-z]` | Any character in range | `file[a-z].txt` → filea.txt, fileb.txt |
| `[!abc]` | NOT a, b, or c | `file[!0-9].txt` → fileA.txt, not file1.txt |
| `{a,b,c}` | Brace expansion | `file.{txt,md}` → file.txt, file.md |

```bash
# Examples
ls *.conf                       # All .conf files
ls image?.png                   # image1.png, imageA.png, etc.
ls log-[0-9][0-9].txt           # log-01.txt through log-99.txt
cp *.{jpg,png,gif} /images/    # Copy all image files
rm temp*                        # Delete all files starting with "temp"
```

---

## File Metadata

```bash
# File type and info
file document.pdf               # PDF document, version 1.4
file script.sh                  # ASCII text, Bash script

# Detailed file status
stat myfile.txt
# Shows: size, blocks, inode, permissions, timestamps, etc.

# Disk usage
du -sh mydir/                   # Total size of directory
du -sh *                        # Size of each item in current directory
du -ah mydir/ | sort -rh | head -10  # Top 10 largest files

# Disk free space
df -h                           # Free space on all mounted filesystems
df -h /home                     # Free space on /home partition
```

---

## 🏋️ Practice Exercises

1. **Create** a directory structure: `project/{src,docs,tests,build}`
2. **Create** 5 files: `touch project/src/file{1..5}.py`
3. **Copy** the `src` directory to `src-backup`
4. **Rename** `project/docs` to `project/documentation`
5. **Find** all `.py` files in your home directory: `find ~ -name "*.py"`
6. **Symlink**: Create a link from `~/quick-access` to `project/`
7. **View**: Use `head`, `tail`, and `less` on `/etc/passwd`
8. **Wildcards**: List all files in `/etc` that end with `.conf`
9. **Size**: Find files larger than 10 MB in `/var`: `find /var -size +10M`
10. **Clean up**: Delete all `.tmp` files in `/tmp`

---

<p align="center">
  <a href="../04-filesystem-hierarchy/README.md">← Previous: Filesystem Hierarchy</a> · <a href="../README.md">🏠 Home</a> · <a href="../06-users-groups-permissions/README.md">Next: Users, Groups & Permissions →</a>
</p>
