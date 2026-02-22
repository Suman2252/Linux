# 🧙 Chapter 17: Advanced Shell Scripting

<p align="center">
  <img src="https://img.shields.io/badge/Level-Advanced-orange?style=for-the-badge" alt="Advanced">
  <img src="https://img.shields.io/badge/Chapter-17%20of%2034-blue?style=for-the-badge" alt="Chapter 17">
</p>

---

## 📑 Table of Contents

- [Regular Expressions (Regex)](#regular-expressions-regex)
- [grep — Pattern Matching](#grep--pattern-matching)
- [sed — Stream Editor](#sed--stream-editor)
- [awk — Pattern Processing](#awk--pattern-processing)
- [xargs — Build Commands from Input](#xargs--build-commands-from-input)
- [Process Substitution](#process-substitution)
- [Trap & Signal Handling](#trap--signal-handling)
- [Here Strings & Documents](#here-strings--documents)
- [Advanced Parameter Expansion](#advanced-parameter-expansion)
- [Practical Scripts](#practical-scripts)
- [Practice Exercises](#-practice-exercises)

---

## Regular Expressions (Regex)

| Pattern | Matches | Example |
|---------|---------|---------|
| `.` | Any single character | `h.t` → hat, hot, hit |
| `*` | Zero or more of previous | `ab*c` → ac, abc, abbc |
| `+` | One or more of previous | `ab+c` → abc, abbc (not ac) |
| `?` | Zero or one of previous | `colou?r` → color, colour |
| `^` | Start of line | `^Hello` |
| `$` | End of line | `world$` |
| `[abc]` | Any char in set | `[aeiou]` → vowels |
| `[^abc]` | Any char NOT in set | `[^0-9]` → non-digit |
| `[a-z]` | Range | `[A-Za-z]` → any letter |
| `\d` | Any digit | `\d+` → 123, 42 |
| `\w` | Word char (letter/digit/_) | `\w+` → hello_world |
| `\s` | Whitespace | `\s+` → spaces, tabs |
| `\b` | Word boundary | `\bword\b` |
| `(abc)` | Group | `(ha)+` → haha, hahaha |
| `a\|b` | Alternation (or) | `cat\|dog` |
| `{n}` | Exactly n times | `a{3}` → aaa |
| `{n,m}` | Between n and m times | `a{2,4}` → aa, aaa, aaaa |

---

## grep — Pattern Matching

```bash
# Basic grep
grep "error" /var/log/syslog           # Find lines with "error"
grep -i "error" file.txt               # Case-insensitive
grep -n "error" file.txt               # Show line numbers
grep -c "error" file.txt               # Count matches
grep -v "debug" file.txt               # Invert (lines WITHOUT "debug")
grep -r "TODO" /project/               # Recursive search
grep -l "error" *.log                  # List filenames only
grep -w "error" file.txt               # Match whole word only

# Extended regex (-E or egrep)
grep -E "error|warning|critical" file.txt
grep -E "^[0-9]{4}-[0-9]{2}" file.txt   # Lines starting with date

# Perl regex (-P) — most powerful
grep -P "\d+\.\d+\.\d+\.\d+" file.txt   # Match IP addresses
grep -oP "(?<=email: )\S+" file.txt      # Extract emails (lookbehind)

# Context lines
grep -B 3 "error" file.txt              # 3 lines Before match
grep -A 5 "error" file.txt              # 5 lines After match
grep -C 2 "error" file.txt              # 2 lines Context (before + after)

# Practical examples
grep -r --include="*.py" "import" /project/   # Search only .py files
grep -rn "TODO\|FIXME\|HACK" /project/        # Find all TODOs
ps aux | grep nginx                            # Find nginx processes
history | grep "docker"                        # Search command history
```

---

## sed — Stream Editor

```bash
# Substitute (replace)
sed 's/old/new/' file.txt              # Replace first on each line
sed 's/old/new/g' file.txt             # Replace ALL occurrences
sed 's/old/new/gi' file.txt            # Case-insensitive replace
sed -i 's/old/new/g' file.txt          # Edit file in-place
sed -i.bak 's/old/new/g' file.txt     # In-place with backup

# Delete lines
sed '5d' file.txt                      # Delete line 5
sed '1,10d' file.txt                   # Delete lines 1-10
sed '/pattern/d' file.txt             # Delete lines matching pattern
sed '/^$/d' file.txt                   # Delete empty lines
sed '/^#/d' file.txt                   # Delete comment lines

# Print specific lines
sed -n '5p' file.txt                   # Print only line 5
sed -n '10,20p' file.txt              # Print lines 10-20
sed -n '/start/,/end/p' file.txt      # Print between patterns

# Insert & Append
sed '3i\New line here' file.txt        # Insert before line 3
sed '3a\New line here' file.txt        # Append after line 3
sed '1i\#!/bin/bash' script.sh         # Add shebang at top

# Multiple operations
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt
sed 's/foo/bar/g; s/baz/qux/g' file.txt

# Practical examples
sed -i 's/\r$//' file.txt              # Fix Windows line endings
sed 's/^[ \t]*//' file.txt             # Remove leading whitespace
sed 's/[ \t]*$//' file.txt             # Remove trailing whitespace
sed -n '1~2p' file.txt                # Print odd-numbered lines
sed 'y/abcdef/ABCDEF/' file.txt       # Character translation
```

---

## awk — Pattern Processing

```bash
# Basic syntax: awk 'pattern { action }' file

# Print specific columns
awk '{print $1}' file.txt              # First column
awk '{print $1, $3}' file.txt          # First and third columns
awk '{print $NF}' file.txt            # Last column
awk -F: '{print $1}' /etc/passwd       # Custom delimiter (:)

# Patterns
awk '/error/ {print}' file.txt         # Lines matching "error"
awk '$3 > 100 {print $1, $3}' file.txt  # Where 3rd column > 100
awk 'NR==5 {print}' file.txt          # Only line 5
awk 'NR>=10 && NR<=20' file.txt        # Lines 10-20

# Built-in variables
awk '{print NR, $0}' file.txt         # NR = line number
awk '{print NF}' file.txt              # NF = number of fields
awk 'END {print NR}' file.txt         # Total lines (like wc -l)

# Calculations
awk '{sum += $3} END {print sum}' file.txt         # Sum column 3
awk '{sum += $3} END {print sum/NR}' file.txt      # Average
awk 'BEGIN {max=0} $3>max {max=$3} END {print max}' file.txt  # Maximum

# Formatting
awk '{printf "%-20s %10.2f\n", $1, $3}' file.txt  # Formatted output

# Multiple rules
awk '
  BEGIN { print "=== Report ===" }
  /error/ { errors++ }
  /warning/ { warnings++ }
  END { printf "Errors: %d, Warnings: %d\n", errors, warnings }
' /var/log/syslog

# Practical examples
# Disk usage above 80%
df -h | awk '$5+0 > 80 {print $6, $5}'

# Top memory users
ps aux | awk '{print $4, $11}' | sort -rn | head -10

# Sum file sizes in directory
ls -l | awk '{total += $5} END {print total/1024/1024 " MB"}'

# Parse Apache access log
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

---

## xargs — Build Commands from Input

```bash
# Basic usage
echo "file1 file2 file3" | xargs rm

# From find results
find . -name "*.tmp" | xargs rm
find . -name "*.log" -print0 | xargs -0 rm    # Handle spaces in names

# With placeholder
echo "1 2 3" | xargs -I{} echo "Number: {}"

# Parallel execution
find . -name "*.jpg" | xargs -P 4 -I{} convert {} -resize 50% {}

# Limit arguments per command
echo "1 2 3 4 5 6" | xargs -n 2 echo
# Output:
# 1 2
# 3 4
# 5 6

# Confirm before executing
echo "file1 file2" | xargs -p rm

# Practical examples
cat urls.txt | xargs -P 10 -I{} curl -sO {}       # Download 10 URLs in parallel
grep -rl "old_api" src/ | xargs sed -i 's/old_api/new_api/g'  # Replace across files
docker ps -q | xargs docker stop                    # Stop all Docker containers
```

---

## Process Substitution

```bash
# <() — treat command output as a file
diff <(ls dir1/) <(ls dir2/)            # Compare directory listings
diff <(sort file1.txt) <(sort file2.txt)  # Compare sorted files
paste <(cut -f1 file.txt) <(cut -f3 file.txt)  # Merge columns

# >() — treat command input as a file
tee >(gzip > output.gz) < input.txt    # Compress while viewing
command | tee >(process1) >(process2) > /dev/null  # Fan-out
```

---

## Trap & Signal Handling

```bash
#!/bin/bash

# Cleanup on exit
cleanup() {
    echo "Cleaning up temporary files..."
    rm -rf "$TMPDIR"
    echo "Done."
}
trap cleanup EXIT

# Handle Ctrl+C
trap 'echo "Caught SIGINT, exiting..."; exit 1' INT

# Handle multiple signals
trap 'echo "Signal received"' HUP INT TERM

# Ignore a signal
trap '' INT                    # Ignore Ctrl+C

# Reset trap to default
trap - INT

# Error trap with line numbers
trap 'echo "Error on line $LINENO. Exit code: $?"' ERR

# Create temp directory safely
TMPDIR=$(mktemp -d)
trap 'rm -rf "$TMPDIR"' EXIT
```

---

## Here Strings & Documents

```bash
# Here string — single-line input
grep "word" <<< "This is a string with word in it"
bc <<< "scale=2; 22/7"

# Here document — multi-line input
cat << 'EOF'
Variables are NOT expanded here: $HOME
Because we quoted EOF
EOF

cat << EOF
Variables ARE expanded: $HOME
Because EOF is unquoted
EOF

# Indent-friendly (with <<-)
cat <<-EOF
	This ignores leading tabs
	Useful in functions and if blocks
EOF
```

---

## Advanced Parameter Expansion

```bash
# Substring
str="Hello World"
echo ${str:0:5}        # Hello
echo ${str: -5}        # World (note the space)

# Search and replace
echo ${str/World/Linux}   # Hello Linux

# Default values
echo ${var:-default}       # Use default if unset
echo ${var:=default}       # Set AND use default if unset
echo ${var:+alt}           # Use alt if var IS set
echo ${var:?error msg}     # Print error and exit if unset

# Indirection
var_name="greeting"
greeting="Hello!"
echo ${!var_name}          # Hello! (indirect reference)

# Array operations
arr=(a b c d e)
echo ${arr[@]:2:3}         # c d e (slice)
echo ${#arr[@]}            # 5 (length)
echo ${arr[@]/#/prefix_}   # prefix_a prefix_b ... (prefix all)
```

---

## Practical Scripts

### Log Analyzer

```bash
#!/bin/bash
set -euo pipefail

LOG=${1:-/var/log/syslog}

echo "=== Log Analysis: $LOG ==="
echo "Total lines: $(wc -l < "$LOG")"
echo ""

echo "--- Messages by severity ---"
for level in emerg alert crit err warning notice info debug; do
    count=$(grep -ci "$level" "$LOG" 2>/dev/null || echo 0)
    printf "%-10s %d\n" "$level" "$count"
done

echo ""
echo "--- Top 10 sources ---"
awk '{print $5}' "$LOG" | cut -d: -f1 | sort | uniq -c | sort -rn | head -10
```

### Bulk File Renamer

```bash
#!/bin/bash
# Rename all .jpeg files to .jpg
for file in *.jpeg; do
    [[ -f "$file" ]] || continue
    mv -v "$file" "${file%.jpeg}.jpg"
done
```

---

## 🏋️ Practice Exercises

1. **grep**: Find all IP addresses in `/var/log/auth.log`
2. **sed**: Remove all blank lines and comments from a config file
3. **awk**: Parse `/etc/passwd` to list users with UID ≥ 1000
4. **xargs**: Find all `.log` files over 10MB and compress them
5. **Regex**: Write a regex to validate email addresses
6. **Pipeline**: Build a pipeline that extracts, sorts, and counts unique visitors from a web log
7. **Trap**: Write a script that cleans up temp files on exit
8. **awk script**: Write an awk script that generates a CSV report from log data

---

<p align="center">
  <a href="../16-archive-compression-backup/README.md">← Previous: Archive & Backup</a> · <a href="../README.md">🏠 Home</a> · <a href="../18-kernel-system-internals/README.md">Next: Kernel & System Internals →</a>
</p>
