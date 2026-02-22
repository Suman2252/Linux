# 📜 Chapter 09: Shell Scripting Fundamentals

<p align="center">
  <img src="https://img.shields.io/badge/Level-Intermediate-yellow?style=for-the-badge" alt="Intermediate">
  <img src="https://img.shields.io/badge/Chapter-09%20of%2034-blue?style=for-the-badge" alt="Chapter 09">
</p>

---

## 📑 Table of Contents

- [What is a Shell Script?](#what-is-a-shell-script)
- [Your First Script](#your-first-script)
- [Variables](#variables)
- [User Input](#user-input)
- [Conditionals (if/else)](#conditionals-ifelse)
- [Comparison Operators](#comparison-operators)
- [Case Statements](#case-statements)
- [Loops](#loops)
- [Functions](#functions)
- [Exit Codes & Error Handling](#exit-codes--error-handling)
- [Arrays](#arrays)
- [String Manipulation](#string-manipulation)
- [Arithmetic](#arithmetic)
- [Debugging Scripts](#debugging-scripts)
- [Practice Exercises](#-practice-exercises)

---

## What is a Shell Script?

A **shell script** is a text file containing a sequence of commands that the shell executes automatically.

> 🏠 **Analogy**: A recipe is a script for cooking — it lists ingredients (variables) and step-by-step instructions (commands) to produce a result.

### Why Learn Shell Scripting?

- 🔄 **Automate** repetitive tasks
- 🛠️ **System administration** — backups, monitoring, deployments
- ⚡ **DevOps** — CI/CD pipelines, infrastructure automation
- 📝 **Glue code** — connect different tools and programs

---

## Your First Script

### Step 1: Create the Script

```bash
#!/bin/bash
# My first shell script
# Filename: hello.sh

echo "Hello, World!"
echo "Today is: $(date)"
echo "You are: $(whoami)"
echo "Current directory: $(pwd)"
```

### Step 2: Make it Executable

```bash
chmod +x hello.sh
```

### Step 3: Run it

```bash
./hello.sh                    # Run from current directory
bash hello.sh                 # Run with bash explicitly
```

### The Shebang Line

```bash
#!/bin/bash                   # Use bash
#!/bin/sh                     # Use POSIX shell (more portable)
#!/usr/bin/env bash           # Find bash in PATH (most portable)
#!/usr/bin/env python3        # Works for other languages too
```

> 💡 **The shebang** (`#!`) tells the system which interpreter to use. Always include it as the first line.

---

## Variables

### Declaring Variables

```bash
# No spaces around the = sign!
name="Sovon"                  # String
age=25                        # Number (stored as string in bash)
today=$(date)                 # Command output
path=$HOME/Documents          # Using another variable
readonly PI=3.14159           # Constant (cannot be changed)
```

> 🚨 **Common Mistake**: `name = "Sovon"` (with spaces) will NOT work. No spaces around `=`.

### Using Variables

```bash
echo "Hello, $name"           # Hello, Sovon
echo "Hello, ${name}!"        # Hello, Sovon! (use braces for clarity)
echo "I am $age years old"
echo "Today: $today"
```

### Variable Scoping

```bash
# By default, variables are local to the script
my_var="local"

# Export to make available to child processes
export MY_VAR="global"

# Or inline for a single command
MY_VAR="temp" ./myscript.sh

# Unset a variable
unset my_var
```

### Special Variables

| Variable | Description |
|----------|-------------|
| `$0` | Script name |
| `$1` to `$9` | Positional arguments |
| `$#` | Number of arguments |
| `$@` | All arguments (as separate words) |
| `$*` | All arguments (as single string) |
| `$?` | Exit code of last command |
| `$$` | PID of current script |
| `$!` | PID of last background process |
| `$_` | Last argument of previous command |

```bash
#!/bin/bash
echo "Script: $0"
echo "First arg: $1"
echo "All args: $@"
echo "Number of args: $#"
```

### Quoting Rules

```bash
name="World"

echo "Hello, $name"           # Double quotes: variables ARE expanded
# Output: Hello, World

echo 'Hello, $name'           # Single quotes: variables NOT expanded
# Output: Hello, $name

echo "Today is $(date)"       # Command substitution works in double quotes
echo "The file has $(wc -l < file.txt) lines"
```

---

## User Input

```bash
#!/bin/bash

# Simple input
read -p "Enter your name: " username
echo "Hello, $username!"

# Silent input (for passwords)
read -sp "Enter password: " password
echo    # New line after hidden input
echo "Password received."

# Input with default value
read -p "Enter color [blue]: " color
color=${color:-blue}          # Default to "blue" if empty
echo "Color: $color"

# Read multiple values
read -p "Enter first and last name: " first last
echo "First: $first, Last: $last"

# Read with timeout
read -t 5 -p "Quick! Enter something (5s): " answer
```

---

## Conditionals (if/else)

```bash
#!/bin/bash

# Basic if
if [ "$1" = "hello" ]; then
    echo "Hi there!"
fi

# if-else
age=$1
if [ "$age" -ge 18 ]; then
    echo "You are an adult."
else
    echo "You are a minor."
fi

# if-elif-else
if [ "$age" -lt 13 ]; then
    echo "Child"
elif [ "$age" -lt 18 ]; then
    echo "Teenager"
elif [ "$age" -lt 65 ]; then
    echo "Adult"
else
    echo "Senior"
fi
```

### Modern Test Syntax `[[ ]]`

```bash
# [[ ]] is bash-specific but more powerful than [ ]
if [[ "$name" == "Sovon" ]]; then
    echo "Hello, Sovon!"
fi

# Supports regex matching
if [[ "$email" =~ ^[a-zA-Z]+@[a-zA-Z]+\.[a-zA-Z]+$ ]]; then
    echo "Valid email format"
fi

# Supports pattern matching
if [[ "$file" == *.txt ]]; then
    echo "It's a text file"
fi
```

---

## Comparison Operators

### Numeric Comparisons

| Operator | Meaning | Example |
|----------|---------|---------|
| `-eq` | Equal | `[ "$a" -eq "$b" ]` |
| `-ne` | Not equal | `[ "$a" -ne "$b" ]` |
| `-gt` | Greater than | `[ "$a" -gt "$b" ]` |
| `-ge` | Greater or equal | `[ "$a" -ge "$b" ]` |
| `-lt` | Less than | `[ "$a" -lt "$b" ]` |
| `-le` | Less or equal | `[ "$a" -le "$b" ]` |

### String Comparisons

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` or `==` | Equal | `[ "$a" = "$b" ]` |
| `!=` | Not equal | `[ "$a" != "$b" ]` |
| `-z` | Empty string | `[ -z "$a" ]` |
| `-n` | Not empty | `[ -n "$a" ]` |
| `<` | Less than (alpha) | `[[ "$a" < "$b" ]]` |
| `>` | Greater than (alpha) | `[[ "$a" > "$b" ]]` |

### File Tests

| Operator | True If... |
|----------|-----------|
| `-e file` | File exists |
| `-f file` | Is a regular file |
| `-d file` | Is a directory |
| `-r file` | Is readable |
| `-w file` | Is writable |
| `-x file` | Is executable |
| `-s file` | File size > 0 |
| `-L file` | Is a symbolic link |
| `f1 -nt f2` | f1 is newer than f2 |
| `f1 -ot f2` | f1 is older than f2 |

```bash
#!/bin/bash

file=$1

if [ -e "$file" ]; then
    echo "$file exists"
    if [ -f "$file" ]; then echo "  It's a regular file"; fi
    if [ -d "$file" ]; then echo "  It's a directory"; fi
    if [ -r "$file" ]; then echo "  It's readable"; fi
    if [ -w "$file" ]; then echo "  It's writable"; fi
    if [ -x "$file" ]; then echo "  It's executable"; fi
else
    echo "$file does not exist"
fi
```

### Logical Operators

```bash
# AND
if [ "$a" -gt 0 ] && [ "$a" -lt 100 ]; then echo "Between 1-99"; fi
if [[ "$a" -gt 0 && "$a" -lt 100 ]]; then echo "Between 1-99"; fi

# OR
if [ "$a" = "yes" ] || [ "$a" = "y" ]; then echo "Confirmed"; fi
if [[ "$a" = "yes" || "$a" = "y" ]]; then echo "Confirmed"; fi

# NOT
if [ ! -f "$file" ]; then echo "File does not exist"; fi
if ! grep -q "error" log.txt; then echo "No errors found"; fi
```

---

## Case Statements

```bash
#!/bin/bash

read -p "Enter a fruit: " fruit

case $fruit in
    apple|Apple)
        echo "🍎 Apples are red!"
        ;;
    banana|Banana)
        echo "🍌 Bananas are yellow!"
        ;;
    orange|Orange)
        echo "🍊 Oranges are orange!"
        ;;
    *)
        echo "🤷 Unknown fruit: $fruit"
        ;;
esac
```

### Practical Example: Service Manager

```bash
#!/bin/bash

case "$1" in
    start)
        echo "Starting service..."
        ;;
    stop)
        echo "Stopping service..."
        ;;
    restart)
        echo "Restarting service..."
        ;;
    status)
        echo "Checking status..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac
```

---

## Loops

### For Loop

```bash
# Basic for loop
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

# C-style for loop
for ((i=0; i<10; i++)); do
    echo "Count: $i"
done

# Range (brace expansion)
for i in {1..10}; do echo "$i"; done
for i in {0..100..5}; do echo "$i"; done    # Step by 5

# Loop over files
for file in *.txt; do
    echo "Processing: $file"
done

# Loop over command output
for user in $(cut -d: -f1 /etc/passwd); do
    echo "User: $user"
done

# Loop over arguments
for arg in "$@"; do
    echo "Argument: $arg"
done
```

### While Loop

```bash
# Basic while
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/passwd

# Infinite loop
while true; do
    echo "Press Ctrl+C to stop"
    sleep 1
done

# While with condition
while ! ping -c1 -W1 google.com &>/dev/null; do
    echo "Waiting for network..."
    sleep 5
done
echo "Network is up!"
```

### Until Loop

```bash
# Runs UNTIL condition is true (opposite of while)
count=1
until [ $count -gt 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

### Loop Control

```bash
# break — exit the loop
for i in {1..100}; do
    if [ $i -eq 5 ]; then break; fi
    echo "$i"
done
# Output: 1 2 3 4

# continue — skip to next iteration
for i in {1..10}; do
    if (( i % 2 == 0 )); then continue; fi
    echo "$i"
done
# Output: 1 3 5 7 9
```

---

## Functions

```bash
#!/bin/bash

# Define a function
greet() {
    echo "Hello, $1!"
}

# Call the function
greet "Sovon"     # Hello, Sovon!
greet "Alice"     # Hello, Alice!

# Function with return value
is_even() {
    if (( $1 % 2 == 0 )); then
        return 0    # True (success)
    else
        return 1    # False (failure)
    fi
}

if is_even 42; then
    echo "42 is even"
fi

# Function that outputs a value
get_hostname() {
    echo $(hostname)
}
my_host=$(get_hostname)

# Function with local variables
calculate() {
    local result=$(( $1 + $2 ))    # 'local' keeps it inside function
    echo $result
}
sum=$(calculate 10 20)
echo "Sum: $sum"    # 30
```

---

## Exit Codes & Error Handling

Every command returns an **exit code** (0 = success, 1-255 = failure).

```bash
# Check exit code
ls /tmp
echo $?     # 0 (success)

ls /nonexistent
echo $?     # 2 (failure)
```

### Error Handling

```bash
#!/bin/bash

# Exit on any error
set -e

# Exit on undefined variable
set -u

# Pipe fails if any command fails
set -o pipefail

# Common combination (put at the top of every script)
set -euo pipefail

# Custom error handling
trap 'echo "Error on line $LINENO"; exit 1' ERR

# Or handle cleanup on exit
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/mytemp.*
}
trap cleanup EXIT
```

### Conditional Execution

```bash
# && = run second command only if first succeeds
mkdir /tmp/test && echo "Directory created"

# || = run second command only if first fails
mkdir /tmp/test || echo "Failed to create directory"

# Combine for try-catch pattern
command_that_might_fail && echo "Success" || echo "Failed"
```

---

## Arrays

```bash
# Declare an array
fruits=("apple" "banana" "cherry" "date")

# Access elements (0-indexed)
echo ${fruits[0]}          # apple
echo ${fruits[2]}          # cherry

# All elements
echo ${fruits[@]}          # apple banana cherry date

# Number of elements
echo ${#fruits[@]}         # 4

# Add element
fruits+=("elderberry")

# Remove element
unset fruits[1]            # Remove banana

# Slice
echo ${fruits[@]:1:2}     # Elements 1-2

# Loop over array
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Associative arrays (bash 4+)
declare -A colors
colors[red]="#FF0000"
colors[green]="#00FF00"
colors[blue]="#0000FF"

echo ${colors[red]}        # #FF0000
echo ${!colors[@]}         # Keys: red green blue
echo ${colors[@]}          # Values: #FF0000 #00FF00 #0000FF
```

---

## String Manipulation

```bash
str="Hello, World!"

# Length
echo ${#str}               # 13

# Substring
echo ${str:7}              # World!
echo ${str:7:5}            # World

# Replace
echo ${str/World/Linux}    # Hello, Linux!
echo ${str//l/L}           # HeLLo, WorLd! (replace all)

# Remove prefix/suffix
file="document.tar.gz"
echo ${file#*.}            # tar.gz (remove shortest prefix match)
echo ${file##*.}           # gz (remove longest prefix match)
echo ${file%.*}            # document.tar (remove shortest suffix)
echo ${file%%.*}           # document (remove longest suffix)

# Default values
echo ${var:-"default"}     # Use "default" if var is unset
echo ${var:="default"}     # Set var to "default" if unset
echo ${var:+"alternative"} # Use "alternative" if var IS set
echo ${var:?"Error msg"}   # Print error and exit if unset

# Case conversion (bash 4+)
echo ${str^^}              # HELLO, WORLD! (uppercase)
echo ${str,,}              # hello, world! (lowercase)
echo ${str^}               # Hello, World! (capitalize first)
```

---

## Arithmetic

```bash
# Using $(( ))
echo $((5 + 3))       # 8
echo $((10 / 3))      # 3 (integer division)
echo $((10 % 3))      # 1 (modulo)
echo $((2 ** 10))     # 1024 (power)

# Increment/Decrement
((count++))
((count--))
((count += 5))

# Using let
let "result = 5 * 3"
echo $result           # 15

# For floating point, use bc
echo "scale=2; 10 / 3" | bc    # 3.33
echo "scale=4; 3.14 * 2" | bc  # 6.28
```

---

## Debugging Scripts

```bash
# Run with debug output
bash -x script.sh              # Print each command before executing

# Enable debug mode inside script
set -x                          # Start debug
set +x                          # Stop debug

# Trace with line numbers
export PS4='+(${BASH_SOURCE}:${LINENO}): '
set -x

# Syntax check without running
bash -n script.sh              # Check for syntax errors

# Use shellcheck for best practices
sudo apt install shellcheck
shellcheck script.sh           # Lint your script
```

---

## 🏋️ Practice Exercises

1. **Hello Script**: Write a script that greets the user by name (passed as argument)
2. **Calculator**: Write a script that takes two numbers and an operator (+, -, *, /) and prints the result
3. **File Checker**: Write a script that checks if a file exists, is readable, and shows its size
4. **Backup Script**: Write a script that copies a directory to a backup location with a timestamp
5. **User Creator**: Write a script that reads a list of usernames from a file and creates each user
6. **System Report**: Write a script that outputs hostname, uptime, disk usage, and memory usage
7. **Log Analyzer**: Write a script that counts error occurrences in `/var/log/syslog`
8. **Menu System**: Create an interactive menu using `select` or `case`

---

<p align="center">
  <a href="../08-text-editors/README.md">← Previous: Text Editors</a> · <a href="../README.md">🏠 Home</a> · <a href="../10-process-management/README.md">Next: Process Management →</a>
</p>
