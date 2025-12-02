# **Shell Scripting Cheatsheet (2025 Edition)**

## **Shell Scripting Fundamentals**

Bash version in CentOS 9 Stream is typically 5.1+ with enhanced features like improved error handling and expanded pattern matching. Modern shell scripts require strict error handling and proper structure to be production-ready.   

### **Essential Script Header (2025 Best Practice)**

```bash
#!/bin/bash
# Script Name: script_name.sh
# Description: Brief description of what this script does
# Author: Your Name
# Date: 2025-12-02
# Version: 1.0
# Usage: ./script_name.sh [args]

# Strict mode - EXIT on errors immediately
set -euo pipefail

# IFS - Internal Field Separator (prevents word splitting issues)
IFS=$'\n\t'
```

### **Modern Shell Options Explained**

- **`set -e`** (errexit): Exit immediately if any command returns non-zero status  
- **`set -u`** (nounset): Exit if script tries to use undeclared variables  
- **`set -o pipefail`**: Return exit code of first failed command in pipeline, not just last  
- **`set -x`**: Debug mode - print each command before execution (disable with `set +x`) 

***

## **Example Shell Scripts**

### **Basic Script Structure**

```bash
#!/bin/bash
set -euo pipefail

# Variable declarations
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE }")" && pwd)"
readonly LOG_FILE="/var/log/myscript.log"

# Function definitions
log_message() {
    local level="$1"
    shift
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $*" | tee -a "$LOG_FILE"
}

check_root() {
    if [[ $EUID -ne 0 ]]; then
        log_message "ERROR" "This script must be run as root"
        exit 1
    fi
}

# Main execution
main() {
    log_message "INFO" "Script started"
    check_root
    # Your code here
    log_message "INFO" "Script completed successfully"
}

# Execute main function
main "$@"
```

### **Production-Ready Script Template**

```bash
#!/bin/bash
set -euo pipefail

# Trap for cleanup on EXIT (runs even if script fails)
cleanup() {
    local exit_code=$?
    echo "Cleaning up temporary files..."
    rm -f /tmp/tempfile_$$
    [[ $exit_code -ne 0 ]] && echo "Script failed with exit code: $exit_code" >&2
    exit "$exit_code"
}
trap cleanup EXIT INT TERM

# Global constants
readonly BACKUP_DIR="/backups"
readonly MAX_BACKUPS=7

# Your functions and logic here
```

***

## **Positional Parameters**

### **Accessing Arguments**

```bash
#!/bin/bash
# $0 = script name
# $1, $2, $3... = individual arguments
# $# = number of arguments
# $@ = all arguments as separate words
# $* = all arguments as single string
# $? = exit status of last command
# $$ = PID of current shell
# $! = PID of last background process

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Total arguments: $#"
echo "All arguments: $@"
echo "Process ID: $$"
```

### **Advanced Parameter Handling**

```bash
#!/bin/bash
set -euo pipefail

# Store all arguments in array
args=("$@")
echo "First arg (array): ${args }"
echo "Second arg (array): ${args }"

# Iterate through all arguments
for arg in "$@"; do
    echo "Processing: $arg"
done

# Shift parameters (removes $1, shifts others down)
echo "Before shift: $1"
shift
echo "After shift (was $2): $1"

# Check if arguments provided
if [[ $# -eq 0 ]]; then
    echo "Usage: $0 <arg1> <arg2> ..." >&2
    exit 1
fi
```

### **Parameter Expansion (Pro Techniques)**

```bash
# Default values
echo "${VAR:-default}"           # Use 'default' if VAR unset/empty
echo "${VAR:=default}"           # Assign 'default' if VAR unset/empty
echo "${VAR:?Error message}"     # Exit with error if VAR unset/empty

# String manipulation
file="report-2025.log"
echo "${file%.log}"              # Remove .log suffix: "report-2025"
echo "${file#report-}"           # Remove "report-" prefix: "2025.log"
echo "${file/2025/2026}"         # Replace: "report-2026.log"
echo "${file^^}"                 # Uppercase: "REPORT-2025.LOG"
echo "${file,,}"                 # Lowercase: "report-2025.log"

# Length
echo "${#file}"                  # String length: 16

# Substrings
echo "${file:0:6}"               # First 6 chars: "report"
```

***

## **Input & Output**

### **Reading User Input**

```bash
#!/bin/bash

# Basic read
echo "Enter your name:"
read name
echo "Hello, $name"

# Read with prompt (inline)
read -p "Enter filename: " filename

# Read with timeout (10 seconds)
read -t 10 -p "Enter value (10s timeout): " value

# Silent input (passwords)
read -sp "Enter password: " password
echo  # New line after password

# Read multiple values
read -p "Enter first and last name: " first last

# Read into array
echo "Enter multiple values (space-separated):"
read -a values
echo "First value: ${values }"

# Read from file descriptor
exec 10<&0                       # Save stdin to FD 10
exec < inputfile.txt             # Redirect file to stdin
while read line; do
    echo "Line: $line"
done
exec 0<&10 10<&-                # Restore stdin, close FD 10
```

### **Output Redirection**

```bash
# Standard output
echo "message" > file.txt        # Overwrite
echo "message" >> file.txt       # Append

# Standard error
echo "error" >&2                 # To stderr
echo "error" 2> error.log        # Stderr to file

# Both stdout and stderr
command > output.log 2>&1        # Both to same file
command &> output.log            # Same (newer syntax)
command > output.log 2> error.log # To separate files

# Discard output
command > /dev/null              # Discard stdout
command 2> /dev/null             # Discard stderr
command &> /dev/null             # Discard both

# Here documents
cat <<EOF > config.txt
Setting1=value1
Setting2=value2
EOF

# Here strings
grep "pattern" <<< "$variable"

# Tee (output to both file and stdout)
ls -la | tee output.txt
ls -la | tee -a output.txt       # Append mode
```

### **Advanced I/O with File Descriptors**

```bash
# Custom file descriptors
exec 3> custom.log               # Open FD 3 for writing
echo "Log entry" >&3             # Write to FD 3
exec 3>&-                        # Close FD 3

# Process substitution (treat command output as file)
diff <(ls dir1) <(ls dir2)
grep "ERROR" logfile | tee >(wc -l > count.txt)

# Command substitution (capture output)
current_date=$(date +%Y-%m-%d)   # Use $() - modern
current_dir=`pwd`                # Avoid `` - legacy

# Subshells (isolated environment)
(cd /tmp && ls)                  # CD doesn't affect parent
echo "Still in: $(pwd)"
```

***

## **Doing Math**

### **Arithmetic Expansion**

```bash
# Basic arithmetic with $(( ))
result=$((5 + 3))                # Addition: 8
result=$((10 - 4))               # Subtraction: 6
result=$((6 * 7))                # Multiplication: 42
result=$((20 / 3))               # Integer division: 6
result=$((20 % 3))               # Modulo: 2
result=$((2 ** 8))               # Exponentiation: 256

# With variables
a=10
b=5
sum=$((a + b))
echo "Sum: $sum"

# Increment/Decrement
count=0
((count++))                      # Increment
((count--))                      # Decrement
((count += 5))                   # Add 5
((count *= 2))                   # Multiply by 2

# Let command (alternative)
let "result = 5 + 3"
let result=5+3                   # Spaces optional without quotes
let result++
```

### **Floating Point Math (bc)**

```bash
# bc - binary calculator for precision math
result=$(echo "scale=2; 22/7" | bc)          # 3.14
result=$(echo "scale=4; sqrt(2)" | bc -l)    # 1.4142

# Complex calculations
result=$(bc -l <<< "scale=6; (10.5 * 3.2) / 2.1")

# Comparisons with bc (returns 1 for true, 0 for false)
if (( $(echo "$value > 3.5" | bc -l) )); then
    echo "Value is greater than 3.5"
fi
```

### **Common Math Operations**

```bash
# Random numbers
rand=$((RANDOM % 100))           # 0-99
rand=$((RANDOM % 100 + 1))       # 1-100

# Current timestamp
timestamp=$(date +%s)

# Math in conditionals
if ((value > 10 && value < 20)); then
    echo "Value between 10 and 20"
fi

# Ternary-like operation
result=$((value > 10 ? 100 : 0))
```

***

## **Comparisons with test**

### **File Tests**

```bash
# File existence and type
[[ -e file ]]                    # Exists (any type)
[[ -f file ]]                    # Regular file exists
[[ -d dir ]]                     # Directory exists
[[ -L link ]]                    # Symbolic link exists
[[ -b device ]]                  # Block device
[[ -c device ]]                  # Character device
[[ -p pipe ]]                    # Named pipe (FIFO)
[[ -S socket ]]                  # Socket

# File permissions
[[ -r file ]]                    # Readable
[[ -w file ]]                    # Writable
[[ -x file ]]                    # Executable
[[ -u file ]]                    # SUID bit set
[[ -g file ]]                    # SGID bit set
[[ -k file ]]                    # Sticky bit set

# File attributes
[[ -s file ]]                    # Non-zero size
[[ -O file ]]                    # Owned by effective UID
[[ -G file ]]                    # Owned by effective GID
[[ -N file ]]                    # Modified since last read

# File comparisons
[[ file1 -nt file2 ]]           # file1 newer than file2
[[ file1 -ot file2 ]]           # file1 older than file2
[[ file1 -ef file2 ]]           # Same device and inode
```

### **String Comparisons**

```bash
# String tests with [[ ]] (modern, preferred)
[[ -z "$str" ]]                  # Zero length (empty)
[[ -n "$str" ]]                  # Non-zero length (not empty)
[[ "$str1" == "$str2" ]]         # Equal
[[ "$str1" != "$str2" ]]         # Not equal
[[ "$str1" < "$str2" ]]          # Lexicographic less than
[[ "$str1" > "$str2" ]]          # Lexicographic greater than

# Pattern matching (only with [[ ]])
[[ "$filename" == *.log ]]       # Glob pattern
[[ "$filename" == report-[0-9]*.txt ]]

# Regular expressions (only with [[ ]])
[[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]

# Case-insensitive comparison (bash 4+)
shopt -s nocasematch
[[ "$str1" == "$str2" ]]
shopt -u nocasematch
```

### **Integer Comparisons**

```bash
# Arithmetic comparisons with (( ))
((num1 == num2))                 # Equal
((num1 != num2))                 # Not equal
((num1 < num2))                  # Less than
((num1 <= num2))                 # Less than or equal
((num1 > num2))                  # Greater than
((num1 >= num2))                 # Greater than or equal

# Legacy [ ] syntax (avoid for new scripts)
[ "$num1" -eq "$num2" ]          # Equal
[ "$num1" -ne "$num2" ]          # Not equal
[ "$num1" -lt "$num2" ]          # Less than
[ "$num1" -le "$num2" ]          # Less than or equal
[ "$num1" -gt "$num2" ]          # Greater than
[ "$num1" -ge "$num2" ]          # Greater than or equal
```

### **Logical Operators**

```bash
# AND
[[ condition1 && condition2 ]]
[[ -f file ]] && [[ -r file ]]

# OR
[[ condition1 || condition2 ]]

# NOT
[[ ! condition ]]
[[ ! -f file ]]

# Combined
[[ -f "$file" && -r "$file" && -s "$file" ]]

# Multiple conditions with (( ))
((num > 0 && num < 100))
```

***

## **Conditional Statements**

### **if-elif-else**

```bash
#!/bin/bash

# Basic if
if [[ condition ]]; then
    commands
fi

# if-else
if [[ -f "$file" ]]; then
    echo "File exists"
else
    echo "File does not exist"
fi

# if-elif-else
if [[ $num -lt 0 ]]; then
    echo "Negative"
elif [[ $num -eq 0 ]]; then
    echo "Zero"
else
    echo "Positive"
fi

# Inline if (short-circuit)
[[ -d "$dir" ]] || mkdir -p "$dir"
[[ -f "$file" ]] && rm "$file"

# Command success check
if cp source dest; then
    echo "Copy successful"
else
    echo "Copy failed" >&2
    exit 1
fi
```

### **case Statement**

```bash
#!/bin/bash

# Basic case
case "$variable" in
    pattern1)
        commands
        ;;
    pattern2|pattern3)
        commands
        ;;
    *)
        default commands
        ;;
esac

# Practical example
case "$1" in
    start)
        echo "Starting service..."
        systemctl start myservice
        ;;
    stop)
        echo "Stopping service..."
        systemctl stop myservice
        ;;
    restart)
        echo "Restarting service..."
        systemctl restart myservice
        ;;
    status)
        systemctl status myservice
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status}" >&2
        exit 1
        ;;
esac

# Pattern matching in case
case "$filename" in
    *.txt)
        echo "Text file"
        ;;
    *.log)
        echo "Log file"
        ;;
    [Rr]eport-*)
        echo "Report file"
        ;;
    *)
        echo "Unknown file type"
        ;;
esac
```

### **Ternary-like Operations**

```bash
# Using && and ||
[[ $status == "success" ]] && result="OK" || result="FAILED"

# Using arithmetic
result=$((value > 10 ? 100 : 0))

# Function-based
get_status() { [[ $1 -eq 0 ]] && echo "SUCCESS" || echo "FAILED"; }
status=$(get_status $exit_code)
```

***

## **The for Loop**

### **Classic for Loop**

```bash
# Iterate over list
for item in apple banana cherry; do
    echo "Fruit: $item"
done

# Iterate over command output
for file in *.txt; do
    echo "Processing: $file"
done

# Iterate over array
files=("file1.txt" "file2.txt" "file3.txt")
for file in "${files[@]}"; do
    echo "File: $file"
done

# Iterate over sequence
for i in {1..10}; do
    echo "Number: $i"
done

for i in {0..100..10}; do        # Start..End..Step
    echo "Value: $i"              # 0, 10, 20, ..., 100
done

# Iterate using seq
for i in $(seq 1 10); do
    echo "Count: $i"
done
```

### **C-style for Loop**

```bash
# C-style syntax
for ((i=0; i<10; i++)); do
    echo "Index: $i"
done

# Multiple variables
for ((i=0, j=10; i<10; i++, j--)); do
    echo "i=$i, j=$j"
done

# Nested loops
for ((i=1; i<=3; i++)); do
    for ((j=1; j<=3; j++)); do
        echo "i=$i, j=$j"
    done
done
```

### **Advanced for Loop Patterns**

```bash
# Loop over files safely (handles spaces)
while IFS= read -r -d '' file; do
    echo "File: $file"
done < <(find . -type f -name "*.log" -print0)

# Loop with continue and break
for i in {1..10}; do
    [[ $i -eq 5 ]] && continue    # Skip 5
    [[ $i -eq 8 ]] && break       # Stop at 8
    echo "$i"
done

# Parallel processing
for file in *.log; do
    process_file "$file" &        # Background process
done
wait                             # Wait for all to complete
```

***

## **The while Loop**

### **Basic while Loop**

```bash
# Basic while
count=1
while [[ $count -le 5 ]]; do
    echo "Count: $count"
    ((count++))
done

# While with arithmetic
i=0
while ((i < 10)); do
    echo "i = $i"
    ((i++))
done

# Infinite loop
while true; do
    echo "Running... (Ctrl+C to stop)"
    sleep 1
done

# Infinite loop alternative
while :; do
    commands
done
```

### **Reading Files with while**

```bash
# Read file line by line (BEST practice)
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt

# Read with custom delimiter
while IFS=: read -r user pass uid gid name home shell; do
    echo "User: $user, UID: $uid, Home: $home"
done < /etc/passwd

# Read command output
while read -r line; do
    echo "Output: $line"
done < <(some_command)

# Read with array
while IFS= read -r -d '' file; do
    echo "File: $file"
done < <(find . -type f -print0)
```

### **until Loop (opposite of while)**

```bash
# until loop (runs until condition is true)
count=0
until [[ $count -ge 5 ]]; do
    echo "Count: $count"
    ((count++))
done

# Wait until service is up
until systemctl is-active --quiet myservice; do
    echo "Waiting for service..."
    sleep 2
done
```

### **Loop Control**

```bash
# break - exit loop
for i in {1..10}; do
    [[ $i -eq 5 ]] && break
    echo "$i"
done

# continue - skip to next iteration
for i in {1..10}; do
    [[ $((i % 2)) -eq 0 ]] && continue  # Skip even numbers
    echo "$i"
done

# Loop with timeout
timeout=10
counter=0
while [[ $counter -lt $timeout ]]; do
    if check_condition; then
        break
    fi
    sleep 1
    ((counter++))
done
```

***

## **Lab: Create a Shell Script for Safe File Deletion**

### **Requirements Analysis**

A "safe delete" script should move files to a trash directory instead of permanently deleting them, with options for restoration and permanent cleanup. 

### **Implementation: safe_delete.sh**

```bash
#!/bin/bash
set -euo pipefail

# Configuration
readonly TRASH_DIR="$HOME/.local/trash"
readonly TRASH_INFO="$TRASH_DIR/.info"
readonly SCRIPT_NAME="$(basename "$0")"

# Color codes for output
readonly RED='\033[0;31m'
readonly GREEN='\033[0;32m'
readonly YELLOW='\033[1;33m'
readonly NC='\033[0m' # No Color

# Initialize trash directory
init_trash() {
    if [[ ! -d "$TRASH_DIR" ]]; then
        mkdir -p "$TRASH_DIR" "$TRASH_INFO"
        echo "Trash directory created at: $TRASH_DIR"
    fi
}

# Log deletion info
log_deletion() {
    local file="$1"
    local timestamp
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "$timestamp|$(basename "$file")|$(realpath "$file")" >> "$TRASH_INFO/deleted.log"
}

# Safe delete function
safe_delete() {
    local file="$1"
    
    # Check if file exists
    if [[ ! -e "$file" ]]; then
        echo -e "${RED}Error: File '$file' does not exist${NC}" >&2
        return 1
    fi
    
    # Generate unique trash name
    local basename
    basename=$(basename "$file")
    local trash_name="${basename}.$(date +%s)"
    local trash_path="$TRASH_DIR/$trash_name"
    
    # Move to trash
    if mv "$file" "$trash_path"; then
        log_deletion "$file"
        echo -e "${GREEN}Moved to trash: $file${NC}"
        echo "  Trash location: $trash_path"
        return 0
    else
        echo -e "${RED}Error: Failed to move '$file' to trash${NC}" >&2
        return 1
    fi
}

# List trash contents
list_trash() {
    echo -e "${YELLOW}Trash Contents:${NC}"
    if [[ -f "$TRASH_INFO/deleted.log" ]]; then
        cat "$TRASH_INFO/deleted.log" | tail -20 | while IFS='|' read -r timestamp filename original_path; do
            printf "%s - %s (from: %s)\n" "$timestamp" "$filename" "$original_path"
        done
    else
        echo "  (empty)"
    fi
}

# Empty trash permanently
empty_trash() {
    read -p "Are you sure you want to permanently delete all trash? (yes/no): " confirm
    if [[ "$confirm" == "yes" ]]; then
        rm -rf "${TRASH_DIR:?}"/*
        rm -f "$TRASH_INFO/deleted.log"
        echo -e "${GREEN}Trash emptied${NC}"
    else
        echo "Operation cancelled"
    fi
}

# Restore file from trash
restore_file() {
    local pattern="$1"
    local found=0
    
    if [[ ! -f "$TRASH_INFO/deleted.log" ]]; then
        echo "No files in trash"
        return 1
    fi
    
    while IFS='|' read -r timestamp filename original_path; do
        if [[ "$filename" == *"$pattern"* ]]; then
            local trash_file
            trash_file=$(find "$TRASH_DIR" -name "${filename}.*" | head -1)
            
            if [[ -f "$trash_file" ]]; then
                read -p "Restore '$filename' to '$original_path'? (y/n): " confirm
                if [[ "$confirm" == "y" ]]; then
                    mkdir -p "$(dirname "$original_path")"
                    if mv "$trash_file" "$original_path"; then
                        echo -e "${GREEN}Restored: $original_path${NC}"
                        found=1
                    else
                        echo -e "${RED}Failed to restore: $filename${NC}" >&2
                    fi
                fi
            fi
        fi
    done < "$TRASH_INFO/deleted.log"
    
    [[ $found -eq 0 ]] && echo "No matching files found in trash"
}

# Usage information
usage() {
    cat <<EOF
Usage: $SCRIPT_NAME [OPTION] [FILE...]

Safe file deletion utility that moves files to trash instead of permanent deletion.

Options:
    -d, --delete FILE(s)     Move file(s) to trash
    -l, --list               List trash contents
    -r, --restore PATTERN    Restore file matching pattern
    -e, --empty              Empty trash (permanent deletion)
    -h, --help               Show this help message

Examples:
    $SCRIPT_NAME -d file.txt
    $SCRIPT_NAME -d *.log
    $SCRIPT_NAME --list
    $SCRIPT_NAME --restore myfile
    $SCRIPT_NAME --empty

Trash location: $TRASH_DIR
EOF
}

# Main function
main() {
    init_trash
    
    # Check arguments
    if [[ $# -eq 0 ]]; then
        usage
        exit 1
    fi
    
    # Parse options
    case "$1" in
        -d|--delete)
            shift
            if [[ $# -eq 0 ]]; then
                echo "Error: No files specified for deletion" >&2
                exit 1
            fi
            for file in "$@"; do
                safe_delete "$file"
            done
            ;;
        -l|--list)
            list_trash
            ;;
        -r|--restore)
            if [[ -z "${2:-}" ]]; then
                echo "Error: No pattern specified for restoration" >&2
                exit 1
            fi
            restore_file "$2"
            ;;
        -e|--empty)
            empty_trash
            ;;
        -h|--help)
            usage
            ;;
        *)
            echo "Error: Unknown option: $1" >&2
            usage
            exit 1
            ;;
    esac
}

# Execute main
main "$@"
```

### **Testing the Safe Delete Script**

```bash
# Test 1: Delete a file
echo "test content" > testfile.txt
./safe_delete.sh -d testfile.txt

# Test 2: Delete multiple files
touch file1.txt file2.txt file3.txt
./safe_delete.sh -d *.txt

# Test 3: List trash
./safe_delete.sh --list

# Test 4: Restore a file
./safe_delete.sh --restore testfile

# Test 5: Empty trash
./safe_delete.sh --empty
```

***

## **Lab: Install New Shell Script**

### **Installation Methods**

Installing a shell script system-wide requires proper permissions, location, and executable flags. 

### **Method 1: System-Wide Installation (Requires Root)**

```bash
#!/bin/bash
# install_script.sh

set -euo pipefail

readonly SCRIPT_NAME="safe_delete.sh"
readonly INSTALL_DIR="/usr/local/bin"
readonly LINK_NAME="trash"

# Check if running as root
if [[ $EUID -ne 0 ]]; then
    echo "Error: This script must be run as root" >&2
    exit 1
fi

# Check if source script exists
if [[ ! -f "$SCRIPT_NAME" ]]; then
    echo "Error: $SCRIPT_NAME not found in current directory" >&2
    exit 1
fi

# Install script
echo "Installing $SCRIPT_NAME to $INSTALL_DIR..."

# Copy script
cp "$SCRIPT_NAME" "$INSTALL_DIR/$SCRIPT_NAME"

# Set permissions
chmod 755 "$INSTALL_DIR/$SCRIPT_NAME"

# Create symbolic link
if [[ -L "$INSTALL_DIR/$LINK_NAME" ]]; then
    rm "$INSTALL_DIR/$LINK_NAME"
fi
ln -s "$INSTALL_DIR/$SCRIPT_NAME" "$INSTALL_DIR/$LINK_NAME"

echo "Installation complete!"
echo "You can now use: $LINK_NAME or $SCRIPT_NAME"
echo "Example: trash -d file.txt"

# Verify installation
if command -v "$LINK_NAME" >/dev/null 2>&1; then
    echo "✓ Command '$LINK_NAME' is available system-wide"
else
    echo "✗ Warning: Command not found in PATH" >&2
fi
```

### **Method 2: User-Local Installation (No Root)**

```bash
#!/bin/bash
# install_local.sh

set -euo pipefail

readonly SCRIPT_NAME="safe_delete.sh"
readonly INSTALL_DIR="$HOME/.local/bin"
readonly LINK_NAME="trash"

# Create local bin directory if needed
mkdir -p "$INSTALL_DIR"

# Check if source script exists
if [[ ! -f "$SCRIPT_NAME" ]]; then
    echo "Error: $SCRIPT_NAME not found" >&2
    exit 1
fi

# Copy and set permissions
cp "$SCRIPT_NAME" "$INSTALL_DIR/$SCRIPT_NAME"
chmod +x "$INSTALL_DIR/$SCRIPT_NAME"

# Create symbolic link
ln -sf "$INSTALL_DIR/$SCRIPT_NAME" "$INSTALL_DIR/$LINK_NAME"

echo "Installed to: $INSTALL_DIR"

# Check if directory is in PATH
if [[ ":$PATH:" != *":$INSTALL_DIR:"* ]]; then
    echo ""
    echo "⚠️  $INSTALL_DIR is not in your PATH"
    echo "Add this line to ~/.bashrc or ~/.bash_profile:"
    echo "    export PATH=\"\$HOME/.local/bin:\$PATH\""
    echo ""
    echo "Then run: source ~/.bashrc"
else
    echo "✓ Installation complete!"
    echo "You can now use: $LINK_NAME"
fi
```

### **Method 3: Automated Installation Script**

```bash
#!/bin/bash
# smart_install.sh - Detects privileges and installs accordingly

set -euo pipefail

readonly SCRIPT_NAME="safe_delete.sh"

install_system_wide() {
    echo "Installing system-wide (requires sudo)..."
    sudo cp "$SCRIPT_NAME" /usr/local/bin/
    sudo chmod 755 "/usr/local/bin/$SCRIPT_NAME"
    sudo ln -sf "/usr/local/bin/$SCRIPT_NAME" /usr/local/bin/trash
    echo "✓ Installed to /usr/local/bin/"
}

install_user_local() {
    echo "Installing for current user only..."
    mkdir -p "$HOME/.local/bin"
    cp "$SCRIPT_NAME" "$HOME/.local/bin/"
    chmod +x "$HOME/.local/bin/$SCRIPT_NAME"
    ln -sf "$HOME/.local/bin/$SCRIPT_NAME" "$HOME/.local/bin/trash"
    
    # Add to PATH if needed
    if [[ ":$PATH:" != *":$HOME/.local/bin:"* ]]; then
        echo 'export PATH="$HOME/.local/bin:$PATH"' >> "$HOME/.bashrc"
        echo "✓ Added $HOME/.local/bin to PATH in ~/.bashrc"
        echo "  Run: source ~/.bashrc"
    fi
    echo "✓ Installed to $HOME/.local/bin/"
}

main() {
    if [[ ! -f "$SCRIPT_NAME" ]]; then
        echo "Error: $SCRIPT_NAME not found" >&2
        exit 1
    fi
    
    echo "Select installation method:"
    echo "1) System-wide (all users) - requires sudo"
    echo "2) User-local (current user only)"
    read -p "Choice [1-2]: " choice
    
    case "$choice" in
        1)
            install_system_wide
            ;;
        2)
            install_user_local
            ;;
        *)
            echo "Invalid choice" >&2
            exit 1
            ;;
    esac
}

main
```

### **Verification and Testing**

```bash
# Check installation
which trash
type trash
command -v trash

# Test installed script
trash --help
trash -d testfile.txt
trash --list

# Check permissions
ls -l $(which trash)

# View script location
readlink -f $(which trash)
```

### **Uninstallation Script**

```bash
#!/bin/bash
# uninstall.sh

set -euo pipefail

echo "Uninstalling safe_delete script..."

# System-wide removal
if [[ -f "/usr/local/bin/safe_delete.sh" ]]; then
    sudo rm -f /usr/local/bin/safe_delete.sh
    sudo rm -f /usr/local/bin/trash
    echo "✓ Removed from /usr/local/bin/"
fi

# User-local removal
if [[ -f "$HOME/.local/bin/safe_delete.sh" ]]; then
    rm -f "$HOME/.local/bin/safe_delete.sh"
    rm -f "$HOME/.local/bin/trash"
    echo "✓ Removed from $HOME/.local/bin/"
fi

echo "Uninstallation complete!"
```

***

## **Pro Tips & Best Practices (2025)**

### **Security Best Practices**

- Always validate and sanitize user input 
- Use `readonly` for constants to prevent modification
- Quote all variables to prevent word splitting: `"$var"` not `$var`  
- Use `set -euo pipefail` for strict error handling  
- Avoid using `eval` - major security risk
- Check file permissions before operations

### **Performance Optimization**

- Use built-in bash features instead of external commands 
- Prefer `[[ ]]` over `[ ]` for better performance
- Use `$(command)` instead of backticks for command substitution 
- Implement parallelization with `xargs -P` or background processes `&` 
- Minimize subshells when possible

### **Code Quality**

- Use ShellCheck for static analysis: `shellcheck script.sh` 
- Write functions for reusability and modularity  
- Use meaningful variable names 
- Add comprehensive comments and headers 
- Implement proper logging mechanisms  
- Use local variables in functions to limit scope 

### **Modern Features (Bash 5.1+ in CentOS 9)**

- Associative arrays for key-value data structures 
- Native regex support with `=~` operator 
- Process substitution for complex pipelines 
- Built-in string manipulation to avoid external tools
