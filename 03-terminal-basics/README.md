# 💻 Chapter 03: Terminal Basics

<p align="center">
  <img src="https://img.shields.io/badge/Level-Beginner-green?style=for-the-badge" alt="Beginner">
  <img src="https://img.shields.io/badge/Chapter-03%20of%2034-blue?style=for-the-badge" alt="Chapter 03">
</p>

---

## 📑 Table of Contents

- [Terminal, Shell, Console — What's the Difference?](#terminal-shell-console--whats-the-difference)
- [Opening the Terminal](#opening-the-terminal)
- [Anatomy of the Prompt](#anatomy-of-the-prompt)
- [Your First Commands](#your-first-commands)
- [Navigating the Filesystem](#navigating-the-filesystem)
- [Getting Help](#getting-help)
- [Command Structure](#command-structure)
- [Essential Keyboard Shortcuts](#essential-keyboard-shortcuts)
- [Input/Output Redirection](#inputoutput-redirection)
- [Pipes — Chaining Commands](#pipes--chaining-commands)
- [Command History](#command-history)
- [Tab Completion](#tab-completion)
- [Aliases](#aliases)
- [Practice Exercises](#-practice-exercises)

---

## Terminal, Shell, Console — What's the Difference?

| Term | What It Is |
|------|-----------|
| **Terminal** | The window/application where you type commands (e.g., GNOME Terminal, Konsole, iTerm2) |
| **Shell** | The program that *interprets* your commands (e.g., Bash, Zsh, Fish) |
| **Console** | A physical text terminal (historically a hardware device) |
| **CLI** | Command Line Interface — the text-based interface in general |

> 🏠 **Analogy**: The terminal is the window you look through. The shell is the person on the other side who understands what you say and does things for you.

### Common Shells

| Shell | Description |
|-------|-------------|
| **bash** | Bourne Again Shell — the most common default shell |
| **zsh** | Z Shell — default on macOS, feature-rich |
| **fish** | Friendly Interactive Shell — beginner-friendly with auto-suggestions |
| **sh** | The original Bourne Shell — minimal, POSIX-compliant |

```bash
# Check your current shell
echo $SHELL

# List all available shells
cat /etc/shells

# Change your default shell
chsh -s /bin/zsh
```

---

## Opening the Terminal

| Desktop Environment | Keyboard Shortcut |
|--------------------|-------------------|
| **Ubuntu (GNOME)** | `Ctrl + Alt + T` |
| **KDE Plasma** | `Ctrl + Alt + T` or search "Konsole" |
| **WSL** | Open from Start Menu or Windows Terminal |
| **macOS** | `Cmd + Space` → type "Terminal" |

---

## Anatomy of the Prompt

```
sovon@ubuntu:~/Documents$ _
  │      │       │       │
  │      │       │       └── $ = normal user (# = root)
  │      │       └── Current directory (~ = home)
  │      └── Hostname
  └── Username
```

### Root vs Regular User

```bash
user@host:~$       # Regular user prompt
root@host:~#       # Root (superuser) prompt — be careful!

# Become root temporarily
sudo su -           # Switch to root
exit                # Go back to normal user

# Run a single command as root
sudo apt update     # Prefix with 'sudo'
```

> ⚠️ **Warning**: Running as root can damage your system. Only use `sudo` when necessary, and never run `sudo rm -rf /` — it will **delete everything**.

---

## Your First Commands

```bash
# Who am I?
whoami              # Print your username

# What's today's date?
date                # Current date and time

# What's this machine?
hostname            # Computer's name
uname -a            # System information (kernel, architecture, etc.)

# How long has the system been running?
uptime              # System uptime

# Where am I?
pwd                 # Print Working Directory (your current location)

# What's in this directory?
ls                  # List files and directories

# Clear the screen
clear               # Or press Ctrl + L
```

---

## Navigating the Filesystem

### The `cd` Command (Change Directory)

```bash
cd /               # Go to root directory
cd ~               # Go to home directory (same as just 'cd')
cd ..              # Go up one level (parent directory)
cd ../..           # Go up two levels
cd -               # Go to the previous directory (toggle back)
cd /var/log        # Go to an absolute path
cd Documents       # Go to a relative path (inside current directory)
```

### The `ls` Command (List)

```bash
ls                 # Basic listing
ls -l              # Long format (permissions, size, date)
ls -a              # Show hidden files (starting with .)
ls -la             # Long format + hidden files
ls -lh             # Human-readable sizes (KB, MB, GB)
ls -lt             # Sort by modification time (newest first)
ls -lS             # Sort by size (largest first)
ls -R              # Recursive (show subdirectories too)
ls -ld /tmp        # Show directory info (not its contents)
```

### Understanding `ls -l` Output

```
drwxr-xr-x  2 sovon sovon 4096 Feb 22 10:00 Documents
│├──┤├──┤     │     │     │    │             │
││   │        │     │     │    │             └── Name
││   │        │     │     │    └── Modification date
││   │        │     │     └── Size in bytes
││   │        │     └── Group owner
││   │        └── File owner
││   └── Group permissions (r-x)
│└── Owner permissions (rwx)
└── d = directory, - = file, l = symlink
```

---

## Getting Help

### `man` — Manual Pages

```bash
man ls             # Open the manual for 'ls'
man -k "copy files"  # Search for manuals containing "copy files"
man 5 passwd       # Open section 5 (file formats) of passwd manual
```

**Navigating man pages:**
| Key | Action |
|-----|--------|
| `Space` | Next page |
| `b` | Previous page |
| `/pattern` | Search forward |
| `n` | Next search result |
| `q` | Quit |

### Other Help Methods

```bash
ls --help          # Quick help (most commands support this)
info ls            # More detailed than man (sometimes)
whatis ls          # One-line description
apropos "disk"     # Search for commands related to "disk"
type ls            # Show what 'ls' actually is (alias, builtin, etc.)
which ls           # Show the path to the command binary
```

---

## Command Structure

```
command  [options]  [arguments]
  │         │          │
  │         │          └── What to act on (file, directory, etc.)
  │         └── Modify behavior (flags like -l, -a, --verbose)
  └── The program to run
```

### Examples

```bash
ls -la /home          # command: ls, options: -la, argument: /home
cp -r dir1 dir2       # command: cp, options: -r, arguments: dir1 dir2
grep -i "hello" file  # command: grep, options: -i, arguments: "hello" file
```

### Short vs Long Options

```bash
ls -a                 # Short option (single dash, one letter)
ls --all              # Long option (double dash, full word)
ls -la                # Multiple short options combined
ls -l -a              # Same as above, written separately
```

---

## Essential Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl + C` | Cancel/kill the current command |
| `Ctrl + D` | Logout / End of input (EOF) |
| `Ctrl + L` | Clear the screen |
| `Ctrl + A` | Move cursor to start of line |
| `Ctrl + E` | Move cursor to end of line |
| `Ctrl + U` | Delete from cursor to start of line |
| `Ctrl + K` | Delete from cursor to end of line |
| `Ctrl + W` | Delete the previous word |
| `Ctrl + R` | Search command history (reverse search) |
| `Ctrl + Z` | Suspend current process (resume with `fg`) |
| `Tab` | Auto-complete commands and filenames |
| `Tab Tab` | Show all possible completions |
| `↑ / ↓` | Navigate command history |

---

## Input/Output Redirection

Every command has three data streams:

```
              ┌─────────┐
 stdin (0) ──▶│ Command │──▶ stdout (1)
              │         │──▶ stderr (2)
              └─────────┘
```

| Stream | Number | Default | Description |
|--------|--------|---------|-------------|
| **stdin** | 0 | Keyboard | Input to the command |
| **stdout** | 1 | Screen | Normal output |
| **stderr** | 2 | Screen | Error messages |

### Redirection Operators

```bash
# stdout to file (overwrite)
ls > files.txt

# stdout to file (append)
echo "new line" >> files.txt

# stderr to file
ls /nonexistent 2> errors.txt

# stdout and stderr to same file
ls /home /nonexistent &> all-output.txt

# stdin from file
sort < unsorted-list.txt

# Discard output (send to "black hole")
ls > /dev/null 2>&1
```

### The Here Document (heredoc)

```bash
cat << EOF
This is line 1
This is line 2
These lines are sent as input to cat
EOF
```

---

## Pipes — Chaining Commands

The **pipe** (`|`) sends the output of one command as input to the next.

```bash
# Count how many files are in /etc
ls /etc | wc -l

# Find all .conf files in ls output
ls /etc | grep ".conf"

# Sort processes by memory usage and show top 10
ps aux | sort -k4 -rn | head -10

# Chain multiple pipes
cat /var/log/syslog | grep "error" | sort | uniq -c | sort -rn | head -5
```

> 🏠 **Analogy**: Pipes are like an assembly line in a factory. Each worker (command) does one thing and passes the result to the next worker.

### Useful Commands for Pipes

| Command | Purpose |
|---------|---------|
| `grep` | Filter lines matching a pattern |
| `sort` | Sort lines |
| `uniq` | Remove duplicate lines |
| `wc` | Count lines, words, characters |
| `head` | Show first N lines |
| `tail` | Show last N lines |
| `cut` | Extract columns/fields |
| `tr` | Translate/replace characters |
| `tee` | Write to file AND stdout |

```bash
# tee — save output AND display it
ls -la | tee file-listing.txt

# wc — count lines, words, characters
cat myfile.txt | wc -l     # Count lines
cat myfile.txt | wc -w     # Count words
```

---

## Command History

```bash
history            # Show all command history
history 20         # Show last 20 commands
!42                # Run command number 42 from history
!!                 # Run the last command again
!ls                # Run the last command starting with "ls"
```

```bash
# Search history interactively
Ctrl + R           # Type to search, press Enter to run
                   # Press Ctrl + R again for older matches

# History file location
echo $HISTFILE     # Usually ~/.bash_history
echo $HISTSIZE     # Number of commands to keep in memory
```

> 💡 **Pro Tip**: Start a command with a space to prevent it from being saved in history (if `HISTCONTROL=ignorespace` is set).

---

## Tab Completion

Tab completion is your **best friend** — it saves typing and prevents typos.

```bash
cd /ho<Tab>         # Completes to /home/
cd /home/so<Tab>    # Completes to /home/sovon/

apt in<Tab><Tab>    # Shows: install, info, ...
```

### Install Enhanced Tab Completion

```bash
# Ubuntu/Debian
sudo apt install bash-completion

# Add to ~/.bashrc if not already there
if [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
fi
```

---

## Aliases

**Aliases** are shortcuts for commands you use frequently.

```bash
# Create temporary aliases (lost after logout)
alias ll='ls -la'
alias cls='clear'
alias ..='cd ..'
alias ...='cd ../..'
alias update='sudo apt update && sudo apt upgrade -y'

# Remove an alias
unalias ll

# Make aliases permanent — add to ~/.bashrc
echo "alias ll='ls -la'" >> ~/.bashrc
source ~/.bashrc    # Reload without logging out
```

### Useful Aliases to Add

```bash
# Add these to your ~/.bashrc
alias grep='grep --color=auto'
alias df='df -h'
alias du='du -h'
alias free='free -h'
alias mkdir='mkdir -pv'
alias ports='ss -tulanp'
alias myip='curl -s ifconfig.me'
```

---

## 🏋️ Practice Exercises

1. **Navigation**: Use `cd` to navigate to `/var/log`, then back to home
2. **Listing**: Run `ls -la /etc` and identify 3 hidden files
3. **Help**: Use `man grep` and find the option for case-insensitive search
4. **Pipes**: Count how many files are in `/usr/bin` using `ls | wc -l`
5. **Redirection**: Save the output of `ls -la ~` to a file called `home-listing.txt`
6. **History**: Use `Ctrl + R` to search your history for a previous command
7. **Alias**: Create an alias `myip` that runs `curl ifconfig.me`
8. **Chain**: Run `cat /etc/passwd | grep $USER | cut -d: -f7` — what does it show?

---

<p align="center">
  <a href="../02-installation-setup/README.md">← Previous: Installation & Setup</a> · <a href="../README.md">🏠 Home</a> · <a href="../04-filesystem-hierarchy/README.md">Next: Filesystem Hierarchy →</a>
</p>
