# Bash Scripting Cheatsheet

## Basics
```bash
#!/bin/bash              # Shebang
set -euo pipefail        # Strict mode
```

## Variables
```bash
name="Sovon"             # No spaces around =
echo "$name"             # Use double quotes
echo "${name}_suffix"    # Braces for clarity
readonly PI=3.14         # Constant
unset name               # Delete variable
```

## Special Variables
| Variable | Meaning |
|----------|---------|
| `$0` | Script name |
| `$1-$9` | Arguments |
| `$#` | Number of arguments |
| `$@` | All arguments (separate) |
| `$*` | All arguments (single string) |
| `$?` | Last exit code |
| `$$` | Script PID |
| `$!` | Last background PID |

## Conditionals
```bash
if [[ condition ]]; then
    commands
elif [[ condition ]]; then
    commands
else
    commands
fi
```

### Test Operators
| Operator | Description |
|----------|-------------|
| `-z "$s"` | String is empty |
| `-n "$s"` | String is not empty |
| `"$a" == "$b"` | Strings equal |
| `"$a" != "$b"` | Strings not equal |
| `$a -eq $b` | Numbers equal |
| `$a -ne $b` | Not equal |
| `$a -lt $b` | Less than |
| `$a -gt $b` | Greater than |
| `-f file` | File exists |
| `-d dir` | Directory exists |
| `-r file` | Readable |
| `-w file` | Writable |
| `-x file` | Executable |

## Loops
```bash
# For loop
for i in 1 2 3; do echo $i; done
for i in {1..10}; do echo $i; done
for file in *.txt; do echo "$file"; done
for ((i=0; i<10; i++)); do echo $i; done

# While loop
while [[ condition ]]; do commands; done
while read -r line; do echo "$line"; done < file.txt

# Until loop
until [[ condition ]]; do commands; done
```

## Functions
```bash
greet() {
    local name=$1        # Local variable
    echo "Hello, $name!"
    return 0
}
greet "Sovon"
result=$(greet "World") # Capture output
```

## Arrays
```bash
arr=(a b c d)
echo ${arr[0]}           # First element
echo ${arr[@]}           # All elements
echo ${#arr[@]}          # Length
arr+=(e)                 # Append
unset arr[1]             # Delete
for item in "${arr[@]}"; do echo "$item"; done
```

## String Operations
```bash
str="Hello World"
echo ${#str}             # Length: 11
echo ${str:0:5}          # Substring: Hello
echo ${str/World/Linux}  # Replace: Hello Linux
echo ${str,,}            # Lowercase: hello world
echo ${str^^}            # Uppercase: HELLO WORLD
```

## File/Path Operations
```bash
path="/home/sovon/file.tar.gz"
echo ${path##*/}         # Basename: file.tar.gz
echo ${path%/*}          # Directory: /home/sovon
echo ${path##*.}         # Extension: gz
echo ${path%.tar.gz}     # Remove ext: /home/sovon/file
```

## Input/Output
```bash
read -p "Name: " name    # Prompt
read -s -p "Pass: " pass # Silent (passwords)
echo "text" > file       # Overwrite
echo "text" >> file      # Append
cmd 2>&1                 # Stderr to stdout
cmd &>/dev/null          # Silence all output
```

## Common Patterns
```bash
# Default values
${var:-default}          # Use default if unset
${var:=default}          # Set and use default

# Command substitution
result=$(command)
today=$(date +%Y%m%d)

# Arithmetic
((count++))
result=$((a + b))

# Ternary
[[ $a -gt $b ]] && echo "yes" || echo "no"
```
