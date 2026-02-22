# Essential Linux Commands Cheatsheet

## Navigation
| Command | Description |
|---------|-------------|
| `pwd` | Print working directory |
| `cd /path` | Change directory |
| `cd ~` or `cd` | Go to home |
| `cd -` | Go to previous directory |
| `ls -la` | List all files with details |
| `tree -L 2` | Show directory tree |

## File Operations
| Command | Description |
|---------|-------------|
| `cp src dst` | Copy file |
| `cp -r src dst` | Copy directory |
| `mv src dst` | Move/rename |
| `rm file` | Delete file |
| `rm -rf dir` | Delete directory |
| `mkdir -p a/b/c` | Create nested dirs |
| `touch file` | Create empty file |
| `ln -s target link` | Symbolic link |

## Viewing Files
| Command | Description |
|---------|-------------|
| `cat file` | Print entire file |
| `less file` | Paginated viewing |
| `head -20 file` | First 20 lines |
| `tail -20 file` | Last 20 lines |
| `tail -f file` | Follow file (live) |
| `wc -l file` | Count lines |

## Searching
| Command | Description |
|---------|-------------|
| `find / -name "*.log"` | Find by name |
| `find / -size +100M` | Find by size |
| `find / -mtime -7` | Modified last 7 days |
| `grep "text" file` | Search in file |
| `grep -r "text" /dir` | Recursive search |
| `grep -i "text" file` | Case-insensitive |
| `which command` | Find command path |
| `locate filename` | Fast file search |

## Permissions
| Command | Description |
|---------|-------------|
| `chmod 755 file` | rwxr-xr-x |
| `chmod 644 file` | rw-r--r-- |
| `chmod +x file` | Add execute |
| `chown user:group file` | Change owner |
| `chown -R user dir` | Recursive owner |

## Disk & Storage
| Command | Description |
|---------|-------------|
| `df -h` | Disk usage (filesystems) |
| `du -sh /dir` | Directory size |
| `du -sh * \| sort -rh` | Sorted sizes |
| `lsblk` | Block devices |
| `mount \| column -t` | Mounted filesystems |
| `free -h` | Memory usage |

## Process Management
| Command | Description |
|---------|-------------|
| `ps aux` | All processes |
| `top` / `htop` | Interactive monitor |
| `kill PID` | Graceful kill |
| `kill -9 PID` | Force kill |
| `killall name` | Kill by name |
| `bg` / `fg` | Background / foreground |
| `jobs` | List background jobs |
| `nohup cmd &` | Run after logout |

## Networking
| Command | Description |
|---------|-------------|
| `ip addr show` | Show IP addresses |
| `ip route show` | Show routes |
| `ping host` | Test connectivity |
| `ss -tuln` | Listening ports |
| `curl -I url` | HTTP headers |
| `dig domain` | DNS lookup |
| `traceroute host` | Trace route |
| `wget url` | Download file |

## Archives
| Command | Description |
|---------|-------------|
| `tar -czf a.tar.gz dir/` | Create .tar.gz |
| `tar -xzf a.tar.gz` | Extract .tar.gz |
| `tar -xjf a.tar.bz2` | Extract .tar.bz2 |
| `zip -r a.zip dir/` | Create zip |
| `unzip a.zip` | Extract zip |

## System Info
| Command | Description |
|---------|-------------|
| `uname -a` | System info |
| `hostnamectl` | Hostname & OS |
| `uptime` | Uptime & load |
| `whoami` | Current user |
| `id` | User ID & groups |
| `date` | Current date/time |
| `last` | Recent logins |

## Package Management
| Distro | Install | Update | Remove | Search |
|--------|---------|--------|--------|--------|
| **Debian/Ubuntu** | `apt install pkg` | `apt update && apt upgrade` | `apt remove pkg` | `apt search pkg` |
| **Fedora/RHEL** | `dnf install pkg` | `dnf upgrade` | `dnf remove pkg` | `dnf search pkg` |
| **Arch** | `pacman -S pkg` | `pacman -Syu` | `pacman -R pkg` | `pacman -Ss pkg` |

## Pipes & Redirection
| Syntax | Description |
|--------|-------------|
| `cmd > file` | Stdout to file (overwrite) |
| `cmd >> file` | Stdout to file (append) |
| `cmd 2> file` | Stderr to file |
| `cmd &> file` | Both stdout+stderr |
| `cmd1 \| cmd2` | Pipe output to next |
| `cmd < file` | File as stdin |
| `cmd1 && cmd2` | Run cmd2 only if cmd1 succeeds |
| `cmd1 \|\| cmd2` | Run cmd2 only if cmd1 fails |
