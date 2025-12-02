# Shell & Bash Cheatsheet - 2025 Edition

## Shells

### What is a Shell?
A **shell** is a command interpreter and programming language that sits between the user and the operating system kernel. It processes commands, manages environment variables, and executes scripts. 

### Common Shell Types

| Shell | Path | Key Characteristic | Best Use Case |
|-------|------|-------------------|---------------|
| **sh** (Bourne Shell) | `/bin/sh` | POSIX compliant, minimal | Legacy scripts, maximum portability  |
| **bash** | `/bin/bash` | Default on most Linux, feature-rich | General purpose, scripting  |
| **zsh** | `/bin/zsh` | Plugin ecosystem, advanced completion | Power users, macOS default  |
| **fish** | `/usr/bin/fish` | User-friendly, non-POSIX | Interactive daily use  |

## Identifying and Changing the Shell

### Identify Current Shell
```bash
echo $SHELL              # Shows login shell
echo $0                  # Shows current shell process
ps -p $$                 # Shows current shell with PID
cat /etc/passwd | grep $USER  # Shows default shell for user
```

### Check Available Shells
```bash
cat /etc/shells          # List all valid shells on system
chsh -l                  # Alternative command (if available)
```

### Change Default Shell
```bash
chsh -s /bin/bash        # Change to bash (prompts for password)
chsh -s /bin/zsh username  # Change for specific user (requires root)
usermod -s /bin/bash username  # Root alternative method
```

### Temporary Shell Change
```bash
exec /bin/bash           # Replace current shell session
/bin/zsh                 # Start new shell (exit to return)
```

## sh: Configuration Files

### Load Order (Login Shell)
1. `/etc/profile` - System-wide login initialization
2. `~/.profile` - User-specific login configuration
3. `~/.login` - Alternative user login file (CSH-style)

### Key Characteristics
- Minimalist, POSIX-compliant configuration 
- No fancy features like command history or tab completion
- Used for maximum script portability

### Basic `/etc/profile` Content
```bash
PATH=/usr/local/bin:/usr/bin:/bin
export PATH
umask 022
```

## sh: Script Execution

### Execution Methods
```bash
sh script.sh             # Execute with sh explicitly
./script.sh              # Direct execution (requires chmod +x)
sh -x script.sh          # Debug mode with trace
sh -n script.sh          # Syntax check without execution
sh -v script.sh          # Verbose mode (prints commands before execution)
```

### Script Shebang
```bash
#!/bin/sh                # Standard sh shebang
#!/usr/bin/env sh        # Portable shebang (finds sh in PATH)
```

### Security Best Practices
```bash
chmod 700 script.sh      # Owner only rwx (safest for production) [web:7]
chmod 755 script.sh      # Owner rwx, others rx (common for shared scripts)
```

## sh: Prompts

### Basic Prompt Variables
```bash
PS1='$ '                 # Primary prompt (default)
PS2='> '                 # Continuation prompt (multi-line commands)
PS3='#? '                # Select command prompt
PS4='+ '                 # Debug trace prompt (with set -x)
```

### Simple Custom Prompt
```bash
PS1='[\u@\h \W]\$ '      # [username@hostname directory]$
export PS1               # Make persistent in session
```

## bash: Bourne Again Shell

### Why Bash Dominates
- **Default on RHEL/CentOS/Ubuntu** - Universal Linux presence 
- **POSIX compatible** - Scripts run everywhere 
- **Rich scripting features** - Arrays, functions, advanced redirects
- **Backward compatible** - Understands sh scripts

### Bash-Specific Features
```bash
# Command substitution (modern)
today=$(date +%Y-%m-%d)

# Process substitution
diff <(ls dir1) <(ls dir2)

# Arrays
arr=(one two three)
echo ${arr }           # → two

# Associative arrays (bash 4+)
declare -A colors
colors[red]="#FF0000"

# Brace expansion
echo file{1..5}.txt      # → file1.txt file2.txt ... file5.txt
mkdir -p project/{src,bin,test}/{main,util}
```

## bash: Configuration Files

### Login vs Non-Login Shell Load Order

**Login Shell** (SSH, console login):
1. `/etc/profile` - System-wide
2. `~/.bash_profile` OR `~/.bash_login` OR `~/.profile` (first found wins)
3. `~/.bash_logout` - On exit

**Non-Login Shell** (new terminal window, tmux):
1. `/etc/bash.bashrc` OR `/etc/bashrc` - System-wide
2. `~/.bashrc` - User-specific

### Smart `.bash_profile` Pattern
```bash
# Source .bashrc if it exists
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi

# User-specific environment and startup programs
export PATH="$HOME/bin:$PATH"
```

### Essential `.bashrc` Structure
```bash
# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# History settings
HISTSIZE=10000
HISTFILESIZE=20000
HISTCONTROL=ignoreboth   # Ignore duplicates and space-prefixed

# Shell options
shopt -s histappend      # Append to history, don't overwrite
shopt -s checkwinsize    # Update window size after each command
shopt -s cdspell         # Auto-correct minor cd typos

# Aliases
alias ll='ls -lah --color=auto'
alias grep='grep --color=auto'

# Custom functions
mkcd() { mkdir -p "$1" && cd "$1"; }

# Prompt
PS1='\[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\$ '

# External tool initialization (place at end)
[ -f ~/.fzf.bash ] && source ~/.fzf.bash
```

### Backup Before Modifications
```bash
cp ~/.bashrc ~/.bashrc.bak   # Always backup
cp ~/.bash_profile ~/.bash_profile.bak
```

### Reload Configuration
```bash
source ~/.bashrc         # Reload bashrc
. ~/.bash_profile        # Reload bash_profile
exec bash                # Restart shell (clean slate)
```

## bash: Command Line History, Editing and Completion

### History Configuration
```bash
# In ~/.bashrc
HISTSIZE=10000           # Commands in memory
HISTFILESIZE=20000       # Commands in ~/.bash_history file
HISTCONTROL=ignoreboth   # ignorespace:ignoredups
HISTIGNORE="ls:ps:history"  # Don't record these commands
HISTTIMEFORMAT="%F %T "  # Add timestamps to history

# Share history across sessions (advanced)
shopt -s histappend
PROMPT_COMMAND="history -a; history -c; history -r; $PROMPT_COMMAND"
```

### History Commands
```bash
history                  # Show all history
history 20               # Show last 20 commands
history -c               # Clear current session history
history -d 500           # Delete history entry 500
!500                     # Execute history command 500
!!                       # Repeat last command
!$                       # Last argument of previous command
!^                       # First argument of previous command
^old^new^                # Replace 'old' with 'new' in last command
```

### Command Line Editing (Emacs Mode - Default)
```bash
Ctrl+A                   # Move to beginning of line
Ctrl+E                   # Move to end of line
Ctrl+U                   # Delete from cursor to beginning
Ctrl+K                   # Delete from cursor to end
Ctrl+W                   # Delete word before cursor
Alt+D                    # Delete word after cursor
Ctrl+Y                   # Paste (yank) last deleted text
Ctrl+L                   # Clear screen
Ctrl+R                   # Reverse search history (repeat for next)
Ctrl+G                   # Escape from history search
Alt+.                    # Insert last argument of previous command [web:6]
Alt+Ctrl+E               # Shell-expand-line (expand variables/aliases) [web:6]
```

### Vi Mode (For Vi Users)
```bash
set -o vi                # Enable vi mode (in ~/.bashrc)
# Then use Esc to enter command mode, i for insert
```

### Tab Completion
```bash
# Enable programmable completion
if [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
fi

# Or on CentOS 9
if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
fi

# Usage examples:
cd /usr/lo<TAB>          # Completes to /usr/local/
systemctl sta<TAB>       # Shows: start, status, stop
git che<TAB>             # Completes subcommands
```

### Advanced Completion Behavior
```bash
# In ~/.bashrc
bind 'set show-all-if-ambiguous on'     # Show matches immediately
bind 'set completion-ignore-case on'     # Case-insensitive completion
bind 'set menu-complete-display-prefix on'  # Show prefix before cycling
```

## Bash: "shortcuts"

### Navigation Shortcuts
```bash
Ctrl+A / Home            # Start of line
Ctrl+E / End             # End of line
Alt+B                    # Back one word [web:9]
Alt+F                    # Forward one word [web:9]
Ctrl+XX                  # Toggle between start of line and current position
```

### Editing Shortcuts
```bash
Ctrl+D                   # Delete character under cursor (or exit if empty)
Alt+Backspace            # Delete word before cursor (respects word boundaries)
Alt+D                    # Delete word after cursor
Ctrl+T                   # Transpose characters (swap last two chars)
Alt+T                    # Transpose words
Alt+U                    # Uppercase word
Alt+L                    # Lowercase word
Alt+C                    # Capitalize word
```

### Process Control
```bash
Ctrl+C                   # Kill current foreground process (SIGINT)
Ctrl+Z                   # Suspend current process (send to background)
Ctrl+D                   # Exit shell or signal EOF
Ctrl+S                   # Stop terminal output (XOFF)
Ctrl+Q                   # Resume terminal output (XON)
```

### History Shortcuts (Advanced)
```bash
!!                       # Last command
!-2                      # Command 2 back
!string                  # Last command starting with 'string'
!?string                 # Last command containing 'string'
!$                       # Last argument of previous command
!^                       # First argument of previous command
!*                       # All arguments of previous command
^old^new                 # Quick substitution [web:6]
```

### Command Expansion
```bash
Alt+Ctrl+E               # Expand aliases and variables [web:6]
Ctrl+Alt+*               # Expand glob patterns [web:6]
Ctrl+_                   # Undo last edit [web:6]
```

### Bang (!) Command Modifiers
```bash
!!:p                     # Print last command without executing
!500:p                   # Print history command 500
!!:s/old/new/            # Substitute in last command
!!:gs/old/new/           # Global substitute
```

## bash: prompt

### PS1 Prompt Variables 

| Code | Description | Example Output |
|------|-------------|----------------|
| `\u` | Username | john |
| `\h` | Hostname (short) | server |
| `\H` | Hostname (full) | server.example.com |
| `\w` | Full working directory | /home/john/projects |
| `\W` | Basename of working directory | projects |
| `\$` | # for root, $ for user | $ or # |
| `\t` | Time 24-hour HH:MM:SS | 14:35:22 |
| `\T` | Time 12-hour HH:MM:SS | 02:35:22 |
| `\A` | Time 24-hour HH:MM | 14:35 |
| `\d` | Date (Weekday Month Date) | Mon Dec 02 |
| `\n` | Newline | |
| `\j` | Number of background jobs | 2 |
| `\!` | History number | 342 |
| `\#` | Command number | 25 |
| `\\` | Literal backslash | \ |
| `\[` | Begin non-printing sequence | |
| `\]` | End non-printing sequence | |

### Prompt Color Codes 
```bash
# Format: \e[STYLE;COLORm
# Style: 0=normal, 1=bold, 2=dim, 4=underline
# Color: 30-37 (foreground), 40-47 (background)

# Colors:
30=Black  31=Red     32=Green  33=Brown/Yellow
34=Blue   35=Purple  36=Cyan   37=Light Gray

# Bold versions (1;3X):
1;30=Dark Gray  1;31=Light Red  1;32=Light Green
1;33=Yellow     1;34=Light Blue  1;35=Light Purple
1;36=Light Cyan 1;37=White
```

### Practical Prompt Examples
```bash
# Simple colored prompt
PS1='\[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\$ '
# Output: user@host:~/dir $  (user@host in green, path in blue)

# With Git branch (requires git in PATH)
PS1='\[\e[32m\]\u\[\e[0m\]@\[\e[33m\]\h\[\e[0m\]:\[\e[34m\]\w\[\e[35m\]$(__git_ps1 " (%s)")\[\e[0m\]\$ '

# Two-line prompt with time
PS1='\[\e[36m\][\t]\[\e[0m\] \[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\n\$ '

# Root warning (red for root, green for user)
PS1='\[\e[$([[ $EUID == 0 ]] && echo "31" || echo "32")m\]\u@\h\[\e[0m\]:\w\$ '

# Minimalist with exit status
PS1='\[\e[31m\]$([ $? != 0 ] && echo "✗ ")\[\e[0m\]\w \$ '
```

### Modern Prompt: Starship (Cross-Shell) 
```bash
# Install Starship (https://starship.rs)
curl -sS https://starship.rs/install.sh | sh

# Add to ~/.bashrc
eval "$(starship init bash)"

# Configure via ~/.config/starship.toml
# Provides: Git status, language versions, execution time, etc.
```

### Make Prompt Permanent
```bash
# Add to ~/.bashrc
export PS1='\[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\$ '

# Then reload
source ~/.bashrc
```

### Testing Prompts Temporarily
```bash
# Try without modifying files
export PS1='Test> '      # Applies to current session only
```

***

## Advanced Pro Tips for 2025

### Script Security Hardening 
```bash
#!/bin/bash
set -euo pipefail        # Exit on error, undefined var, pipe failure [web:7]
# e: exit on error
# u: exit on undefined variable
# o pipefail: exit if any pipe command fails

# Use long options for readability [web:10]
rm --force --recursive /tmp/build  # Better than: rm -rf /tmp/build
```

### Modern Shell Enhancements 
```bash
# Install modern tools
sudo dnf install fzf bat exa ripgrep fd-find

# Add to ~/.bashrc
alias cat='bat --style=plain'
alias ls='exa --icons'
alias find='fd'
alias grep='rg'

# fzf for fuzzy history search (Ctrl+R replacement)
[ -f ~/.fzf.bash ] && source ~/.fzf.bash
```

### Environment Variable Management
```bash
# View all variables
printenv
env

# Set for current session
export EDITOR=vim

# Make permanent (in ~/.bashrc)
export EDITOR=vim
export PATH="$HOME/.local/bin:$PATH"

# Unset variable
unset VARNAME
```

### Shell Options (shopt) Cheatsheet
```bash
shopt -s histappend      # Append to history file
shopt -s checkwinsize    # Update LINES and COLUMNS after each command
shopt -s cdspell         # Auto-correct minor cd typos
shopt -s dirspell        # Auto-correct directory name typos
shopt -s autocd          # Type directory name to cd into it (bash 4+)
shopt -s globstar        # Enable ** recursive glob (bash 4+)
shopt -s dotglob         # Include hidden files in glob expansion
shopt -s nocaseglob      # Case-insensitive globbing

# View all options
shopt

# Disable option
shopt -u optionname
```

### Job Control Mastery
```bash
command &                # Run in background
jobs                     # List background jobs
fg %1                    # Bring job 1 to foreground
bg %1                    # Resume job 1 in background
kill %1                  # Kill job 1
disown %1                # Remove job from shell's job table
nohup command &          # Run immune to hangups
```

# CentOS 9 Shell Labs - 2025 Edition (Hands-On)

## Lab: Identify the Current Shell

### Comprehensive Shell Identification Methods
```bash
# Method 1: Show login shell path
echo $SHELL
# Output: /bin/bash (or /bin/zsh, etc.)

# Method 2: Show actively running shell
ps -p $$
# Output shows CMD column with current shell

# Method 3: Show shell process name
echo $0
# Output: bash, -bash (login), zsh, etc.

# Method 4: Detailed process tree
ps -f -p $$
# Shows PPID, CMD, and execution time

# Method 5: Check /proc filesystem
readlink /proc/$$/exe
# Output: Full path like /usr/bin/bash

# Method 6: Using shell-specific variables
echo $BASH_VERSION        # Only works in bash
echo $ZSH_VERSION         # Only works in zsh

# Method 7: Check current user's default shell
grep "^$USER:" /etc/passwd | cut -d: -f7
# Output: /bin/bash

# Method 8: View all logged-in users and their shells
w | awk '{print $1, $2}' && getent passwd | grep "$(w | tail -n +2 | awk '{print $1}')" | cut -d: -f1,7
```

### Practical Lab Exercise
```bash
# Create a test script to identify shell
cat > identify_shell.sh << 'EOF'
#!/bin/bash
echo "=== Shell Identification Report ==="
echo "SHELL variable: $SHELL"
echo "Process name (\$0): $0"
echo "Bash version: ${BASH_VERSION:-Not Bash}"
echo "Zsh version: ${ZSH_VERSION:-Not Zsh}"
echo "Current PID: $$"
echo "Parent PID: $PPID"
echo "Shell executable: $(readlink /proc/$$/exe)"
EOF

chmod +x identify_shell.sh
./identify_shell.sh
```

## Lab: Examine Symbolic Links of Listed Shells

### Discovering Shell Symlinks
```bash
# List all valid shells
cat /etc/shells
# Output:
# /bin/sh
# /bin/bash
# /usr/bin/sh
# /usr/bin/bash

# Check if shells are symlinks
ls -l /bin/sh
ls -l /bin/bash
ls -l /usr/bin/sh
ls -l /usr/bin/bash

# Comprehensive symlink examination
for shell in $(cat /etc/shells); do
    echo "=== $shell ==="
    if [ -L "$shell" ]; then
        echo "  Type: Symbolic link"
        echo "  Points to: $(readlink -f $shell)"
    elif [ -f "$shell" ]; then
        echo "  Type: Regular file"
        echo "  Size: $(stat -c%s $shell) bytes"
    else
        echo "  Type: Not found or special file"
    fi
    [ -f "$shell" ] && file "$shell"
    echo
done
```

### CentOS 9 Specific Symlink Investigation
```bash
# Check if /bin/sh points to bash or dash
ls -lah /bin/sh
# Typical output: lrwxrwxrwx. 1 root root 4 Oct 15 12:34 /bin/sh -> bash

# Verify the symlink chain
readlink -f /bin/sh
# Output: /usr/bin/bash (shows final destination)

# Check usr-merge filesystem layout (CentOS 9 uses this)
ls -ld /bin
# Output: lrwxrwxrwx. 1 root root 7 Oct 15 12:34 /bin -> usr/bin

# Real shell binaries are in /usr/bin
ls -lh /usr/bin/bash /usr/bin/sh

# Advanced: Find all symlinks pointing to bash
find /bin /usr/bin -type l -exec sh -c 'readlink -f "$1" | grep -q bash && echo "$1 -> $(readlink -f "$1")"' _ {} \;
```

### Create Your Own Shell Symlink (Practice)
```bash
# Create a custom shell alias via symlink
sudo ln -s /bin/bash /usr/local/bin/mybash
ls -l /usr/local/bin/mybash

# Test it
/usr/local/bin/mybash
echo $SHELL
exit

# Cleanup
sudo rm /usr/local/bin/mybash
```

## Lab: Invoke Shell Directly and Change Login Shell

### Invoking Shells Directly
```bash
# Start a new bash subshell
bash
echo "Subshell level: $SHLVL"
ps --forest -o pid,ppid,cmd
exit

# Start bash as login shell (sources login configs)
bash --login
# or
bash -l

# Start bash without reading any config files
bash --norc --noprofile

# Start bash in POSIX mode
bash --posix

# Start bash in restricted mode (security sandbox)
bash --restricted
# or
rbash

# Interactive test: What's my shell depth?
echo $SHLVL              # Increments with each nested shell
bash
echo $SHLVL              # Should be +1 from previous
exit
```

### Changing Login Shell (Permanent)
```bash
# Method 1: Using chsh (interactive)
chsh
# Enter password
# Enter new shell: /bin/zsh
# Verification: cat /etc/passwd | grep $USER

# Method 2: Using chsh with -s flag (one-liner)
chsh -s /bin/bash
# Enter password when prompted

# Method 3: Using chsh for other users (requires root/sudo)
sudo chsh -s /bin/zsh username

# Method 4: Using usermod (requires root/sudo)
sudo usermod -s /bin/zsh username

# Verify change took effect
grep "^$USER:" /etc/passwd | cut -d: -f7

# IMPORTANT: Changes take effect on next login
# Test without logging out:
su - $USER
echo $SHELL
exit
```

### Practical Multi-Shell Lab Session
```bash
# Current state
echo "Current: $SHELL (Level $SHLVL)"

# Start sh (minimal POSIX shell)
sh
echo "Now in: $0 (Level $SHLVL)"
exit

# Start bash explicitly
bash
echo "Now in: $0 (Level $SHLVL)"
exit

# If zsh is installed, try it
zsh 2>/dev/null && echo "Zsh works! Version: $ZSH_VERSION" && exit

# Return to original shell
echo "Back to: $SHELL (Level $SHLVL)"
```

## Lab: Explore Functions Available Through Command Line History

### Built-in History Functions
```bash
# View history with line numbers
history

# Search history interactively
# Press Ctrl+R, then start typing command
# Press Ctrl+R again to cycle through matches
# Press Enter to execute, or Right Arrow to edit

# Get last command
!!
# Example usage:
ls /root
sudo !!              # Runs: sudo ls /root

# Get specific argument from last command
echo one two three
echo !$              # Prints: three (last arg)
echo !^              # Prints: one (first arg)
echo !*              # Prints: one two three (all args)

# Search history by pattern
!?search?            # Run last command containing "search"
!ssh                 # Run last command starting with "ssh"

# Substitute in last command
^old^new             # Replace first 'old' with 'new' in last command
# Example:
cat /etc/pass
^pass^passwd         # Executes: cat /etc/passwd
```

### Advanced History Functions
```bash
# Access history ranges
!120                 # Execute history command #120
!-3                  # Execute command 3 back in history
!!:p                 # Print last command without executing
!ssh:p               # Print last ssh command without executing

# Word designators
!!:0                 # Command name from last command
!!:1                 # First argument
!!:2                 # Second argument
!!:$                 # Last argument
!!:*                 # All arguments
!!:1-3               # Arguments 1 through 3

# Modifiers
!!:s/old/new/        # Substitute first occurrence
!!:gs/old/new/       # Global substitution (all occurrences)
!!:t                 # Tail of pathname (basename)
!!:h                 # Head of pathname (dirname)
!!:r                 # Remove trailing suffix

# Practical examples
ls /var/log/messages
cd !!:2:h            # cd to /var/log
cat !-2:$:r.txt      # Complex history manipulation
```

### Custom History Functions
```bash
# Add to ~/.bashrc

# Function: Search history by keyword
hsearch() {
    history | grep "$1"
}

# Function: Execute command from history by number
hrun() {
    eval "$(history | grep "^ *$1" | sed 's/^ *[0-9]* *//')"
}

# Function: Show history statistics
hstats() {
    history | awk '{CMD[$2]++;count++;}END {for (a in CMD)print CMD[a] " " CMD[a]/count*100 "% " a;}' | grep -v "./" | column -c3 -s " " -t | sort -nr | nl | head -n20
}

# Function: Clear duplicate history entries
hdedupe() {
    nl ~/.bash_history | sort -k2 -k1,1nr | uniq -f1 | sort -n | cut -f2 > ~/.bash_history.tmp
    mv ~/.bash_history.tmp ~/.bash_history
}

# Reload and test
source ~/.bashrc
hsearch "sudo"
hstats
```

## Lab: Display All Aliases, Create, and Remove Aliases

### Display Existing Aliases
```bash
# Show all aliases
alias

# Show specific alias
alias ls
alias ll

# Show alias with full command expansion
type ll
type -a ls           # Show all definitions (alias, function, file)

# Check if a command is aliased
which ls             # Shows alias or binary path
command -v ls        # POSIX-compliant alternative
```

### Create New Aliases (Session-Only)
```bash
# Basic syntax: alias name='command'
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -alF'

# Aliases with options
alias grep='grep --color=auto'
alias mkdir='mkdir -pv'
alias rm='rm -i'            # Interactive (ask before delete)
alias cp='cp -i'            # Interactive copy
alias mv='mv -i'            # Interactive move

# Navigation aliases
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias cd..='cd ..'           # Typo protection

# System monitoring
alias psg='ps aux | grep -v grep | grep -i -e VSZ -e'
alias ports='netstat -tulanp'
alias meminfo='free -m -l -t'
alias df='df -H'

# Networking
alias myip='curl ifconfig.me'
alias pingg='ping -c 5 google.com'

# Development
alias gc='git commit -m'
alias gs='git status'
alias gp='git push'

# Safety aliases (advanced)
alias rm='rm -i --preserve-root'
alias chown='chown --preserve-root'
alias chmod='chmod --preserve-root'
alias chgrp='chgrp --preserve-root'
```

### Remove Aliases
```bash
# Remove specific alias
unalias ll

# Remove all aliases
unalias -a

# Temporarily bypass an alias (run original command)
\ls                  # Run actual ls binary, not alias
command ls           # Same effect, more explicit
/bin/ls              # Direct path to binary

# Check if alias is gone
alias ll             # Should show: -bash: alias: ll: not found
```

### Practical Lab Exercise: Test Alias Behavior
```bash
# Create test alias
alias echo='echo TEST:'
echo hello           # Output: TEST: hello

# Bypass alias
\echo hello          # Output: hello
command echo hello   # Output: hello

# Remove test alias
unalias echo
echo hello           # Output: hello
```

## Lab: Add Aliases to .bashrc for Persistence

### Step-by-Step: Making Aliases Persistent
```bash
# Step 1: Backup your .bashrc
cp ~/.bashrc ~/.bashrc.backup.$(date +%Y%m%d_%H%M%S)

# Step 2: Open .bashrc in editor
vim ~/.bashrc
# or
nano ~/.bashrc

# Step 3: Find the alias section (usually near line 90-110)
# Look for existing aliases like:
# alias ll='ls -alF'
# alias la='ls -A'

# Step 4: Add your custom aliases in a dedicated section
cat >> ~/.bashrc << 'EOF'

# ============================================
# Custom Aliases - Added $(date +%Y-%m-%d)
# ============================================

# Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias home='cd ~'
alias projects='cd ~/projects'

# Safety
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias ln='ln -i'

# List operations
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -alFh'
alias lt='ls -alFht'         # Sort by time
alias lsize='ls -alFhS'      # Sort by size

# System
alias update='sudo dnf update'
alias install='sudo dnf install'
alias remove='sudo dnf remove'
alias ports='sudo netstat -tulanp'
alias listen='sudo lsof -i -P -n | grep LISTEN'

# Development
alias gs='git status'
alias ga='git add'
alias gc='git commit -m'
alias gp='git push'
alias gl='git log --oneline --graph --decorate'
alias python='python3'
alias pip='pip3'

# Docker (if used)
alias dps='docker ps'
alias dpsa='docker ps -a'
alias di='docker images'

# Kubernetes (if used)
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'

# Custom functions as aliases
alias mkcd='mkdir_and_cd'

EOF

# Step 5: Add supporting functions
cat >> ~/.bashrc << 'EOF'

# ============================================
# Custom Functions
# ============================================

# Create directory and cd into it
mkdir_and_cd() {
    mkdir -p "$1" && cd "$1"
}

# Extract any archive
extract() {
    if [ -f "$1" ]; then
        case "$1" in
            *.tar.bz2)   tar xjf "$1"     ;;
            *.tar.gz)    tar xzf "$1"     ;;
            *.bz2)       bunzip2 "$1"     ;;
            *.rar)       unrar e "$1"     ;;
            *.gz)        gunzip "$1"      ;;
            *.tar)       tar xf "$1"      ;;
            *.tbz2)      tar xjf "$1"     ;;
            *.tgz)       tar xzf "$1"     ;;
            *.zip)       unzip "$1"       ;;
            *.Z)         uncompress "$1"  ;;
            *.7z)        7z x "$1"        ;;
            *)           echo "'$1' cannot be extracted" ;;
        esac
    else
        echo "'$1' is not a valid file"
    fi
}

EOF

# Step 6: Test without reloading (in new terminal or subshell)
bash
alias ll

# Step 7: Reload configuration in current session
source ~/.bashrc

# Step 8: Verify aliases are loaded
alias | grep "^alias ll="
alias | head -5
```

### Troubleshooting Persistent Aliases
```bash
# Check if .bashrc is sourced on login
grep "bashrc" ~/.bash_profile

# If not found, add this to ~/.bash_profile
cat >> ~/.bash_profile << 'EOF'
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
EOF

# Test in fresh login shell
bash --login
alias ll

# Verify aliases survive reboot (without actually rebooting)
su - $USER
alias ll
exit
```

## Lab: Customise the Bash Shell

### Visual Customization Lab
```bash
# Step 1: Create customization test environment
mkdir -p ~/bash_custom_lab
cd ~/bash_custom_lab

# Step 2: Export current PS1 for backup
echo "export PS1='$PS1'" > ps1_backup.sh

# Step 3: Test different prompts (temporary)
# Minimalist
PS1='$ '

# Classic with user@host
PS1='\u@\h:\w\$ '

# Colored user@host with path
PS1='\[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\$ '

# Two-line prompt with time
PS1='\[\e[36m\][\t]\[\e[0m\] \[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\n\$ '

# With Git branch (requires git-prompt)
PS1='\[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[35m\]$(__git_ps1 " (%s)")\[\e[0m\]\$ '

# Step 4: Create advanced customized .bashrc section
cat > custom_bashrc_snippet.sh << 'EOF'
# ============================================
# Advanced Bash Customization
# ============================================

# Set 256-color terminal
export TERM=xterm-256color

# History customization
export HISTSIZE=50000
export HISTFILESIZE=100000
export HISTCONTROL=ignoreboth:erasedups
export HISTIGNORE="ls:ps:history:clear:exit"
export HISTTIMEFORMAT="%F %T "

# Shell options
shopt -s histappend          # Append to history
shopt -s checkwinsize        # Check window size after each command
shopt -s cdspell             # Autocorrect cd typos
shopt -s dirspell            # Autocorrect directory name typos
shopt -s autocd              # Type directory name to cd
shopt -s globstar            # Enable ** recursive glob
shopt -s dotglob             # Include hidden files in glob
shopt -s nocaseglob          # Case-insensitive globbing

# Custom colored man pages
export LESS_TERMCAP_mb=$'\e[1;32m'
export LESS_TERMCAP_md=$'\e[1;32m'
export LESS_TERMCAP_me=$'\e[0m'
export LESS_TERMCAP_se=$'\e[0m'
export LESS_TERMCAP_so=$'\e[01;33m'
export LESS_TERMCAP_ue=$'\e[0m'
export LESS_TERMCAP_us=$'\e[1;4;31m'

# Custom prompt with exit status and Git
__prompt_command() {
    local EXIT="$?"
    PS1=""
    
    # Show red X if last command failed
    if [ $EXIT != 0 ]; then
        PS1+="\[\e[31m\]✗ "
    else
        PS1+="\[\e[32m\]✓ "
    fi
    
    # User@host in green
    PS1+="\[\e[32m\]\u@\h\[\e[0m\]:"
    
    # Path in blue
    PS1+="\[\e[34m\]\w\[\e[0m\]"
    
    # Git branch in purple (if in git repo)
    if type __git_ps1 &>/dev/null; then
        PS1+="\[\e[35m\]$(__git_ps1 ' (%s)')\[\e[0m\]"
    fi
    
    PS1+="\$ "
}

PROMPT_COMMAND=__prompt_command

# Custom window title
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;\u@\h: \w\a\]$PS1"
    ;;
esac

EOF

# Step 5: Apply and test
source custom_bashrc_snippet.sh
echo "Test prompt customization"
false                    # Should show red X
true                     # Should show green checkmark
```

### Shell Behavior Customization
```bash
# Add to ~/.bashrc

# Auto-correction for cd
shopt -s cdspell

# Test: Try misspelled directory
mkdir testdir
cd testdri           # Auto-corrects to testdir
cd ..

# Enable recursive globbing
shopt -s globstar

# Test: Find all .conf files recursively
echo **/*.conf

# Case-insensitive completion
bind 'set completion-ignore-case on'

# Show all matches immediately (no double-tab needed)
bind 'set show-all-if-ambiguous on'

# Enable menu-complete (cycle through matches with TAB)
bind 'TAB:menu-complete'
bind 'set show-all-if-unmodified on'
bind 'set menu-complete-display-prefix on'
```

## Lab: Run the Z Shell

### Installing Zsh on CentOS 9
```bash
# Step 1: Install zsh
sudo dnf install -y zsh [web:15]

# Step 2: Verify installation
zsh --version
which zsh

# Step 3: Check if zsh is in valid shells list
cat /etc/shells | grep zsh

# If not present, add it:
echo "/bin/zsh" | sudo tee -a /etc/shells
```

### First-Time Zsh Configuration 
```bash
# Step 1: Start zsh for first time
zsh

# You'll see the configuration wizard (zsh-newuser-install)
# Choose option (2) to populate with recommended defaults
# Or option (0) to create config with comments
# Or option (1) to customize each setting

# Step 2: Manual configuration alternative
cat > ~/.zshrc << 'EOF'
# Basic Zsh configuration
autoload -Uz compinit
compinit

# History
HISTFILE=~/.zsh_history
HISTSIZE=10000
SAVEHIST=10000
setopt SHARE_HISTORY
setopt HIST_IGNORE_DUPS
setopt HIST_IGNORE_SPACE

# Prompt
PROMPT='%F{green}%n@%m%f:%F{blue}%~%f%# '

# Key bindings (emacs mode)
bindkey -e

# Aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
EOF

# Step 3: Reload configuration
source ~/.zshrc
```

### Installing Oh My Zsh Framework  
```bash
# Step 1: Install prerequisites
sudo dnf install -y git curl

# Step 2: Install Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Alternative with wget:
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Step 3: Configuration file created at ~/.zshrc
# Edit theme and plugins
vim ~/.zshrc

# Change theme (line ~11):
ZSH_THEME="agnoster"     # Or: robbyrussell, powerlevel10k, refined, etc.

# Enable plugins (line ~70):
plugins=(
    git
    sudo
    docker
    kubectl
    colored-man-pages
    command-not-found
    zsh-autosuggestions
    zsh-syntax-highlighting
)

# Step 4: Install additional plugins
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# Step 5: Reload
source ~/.zshrc
```

### Zsh vs Bash Feature Comparison Lab
```bash
# Start zsh
zsh

# Test 1: Advanced tab completion
cd /u/s/b<TAB>           # Completes to /usr/share/bash-completion
# Zsh is smarter with path completion!

# Test 2: Spelling correction
ls /ect/passwd           # Zsh asks: "correct 'ect' to 'etc' [nyae]?"

# Test 3: Glob qualifiers (zsh-specific)
ls -l *(.x)              # Only executable regular files
ls -l *(m-7)             # Files modified in last 7 days
ls -l *(L0)              # Empty files

# Test 4: Advanced array handling
arr=(one two three)
echo $arr # In zsh: one (1-indexed!)
echo $arr[-1]            # Last element: three

# Exit zsh
exit
```

### Make Zsh Default Shell  
```bash
# Method 1: Using chsh
chsh -s $(which zsh)
# Enter password

# Method 2: Direct path (if chsh doesn't work)
chsh -s /bin/zsh

# Method 3: For root or other users
sudo chsh -s /bin/zsh username

# Verify change
grep "^$USER:" /etc/passwd

# Log out and log back in to see changes
# Or test with:
su - $USER
echo $SHELL              # Should show /bin/zsh
```

## Lab: Explore Prompt Options Including Right Hand Prompt

### Bash: Advanced Prompt Engineering
```bash
# Custom PS1 with all possible elements
PS1='\n\[\e[33m\][\d \t]\[\e[0m\] \[\e[32m\]\u\[\e[0m\]@\[\e[36m\]\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\] \[\e[35m\][\j jobs]\[\e[0m\]\n\[\e[31m\][\!]\[\e[0m\] \$ '

# Breakdown:
# \n               - Newline
# [\d \t]          - [Date Time] in yellow
# \u               - Username in green
# @\h              - @hostname in cyan
# :\w              - :full-path in blue
# [\j jobs]        - [background jobs] in purple
# \n               - Newline
# [\!]             - [history number] in red
# \$               - $ or # for root

# Prompt with Git branch and status
cat > ~/.bash_git_prompt << 'EOF'
__git_prompt() {
    local git_status="$(git status 2>/dev/null)"
    local branch="$(git branch 2>/dev/null | grep '^*' | colrm 1 2)"
    
    if [ -n "$branch" ]; then
        local status_color="\[\e[32m\]"  # Green by default
        
        if echo "$git_status" | grep -q "Changes not staged\|Untracked files"; then
            status_color="\[\e[31m\]"     # Red if dirty
        elif echo "$git_status" | grep -q "Changes to be committed"; then
            status_color="\[\e[33m\]"     # Yellow if staged
        fi
        
        echo -e "${status_color}(${branch})\[\e[0m\]"
    fi
}

PS1='\[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]$(__git_prompt) \$ '
EOF

source ~/.bash_git_prompt
```

### Bash: Simulating Right-Hand Prompt (Workaround)
```bash
# Bash doesn't have native RPROMPT, but we can hack it
__right_prompt() {
    local right_info="$(date +%H:%M:%S)"
    local save="\e7"        # Save cursor position
    local restore="\e8"     # Restore cursor position
    local move_right="\e[${COLUMNS}C"  # Move to right edge
    local move_left="\e[${#right_info}D"  # Move left by string length
    
    echo -ne "${save}${move_right}${move_left}${right_info}${restore}"
}

PROMPT_COMMAND='__right_prompt'
PS1='\[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\$ '

# Test: Your prompt should now show time on the right
# Left: user@host:path $
# Right: 18:45:23
```

### Zsh: Native Right-Hand Prompt  
```bash
# Start zsh
zsh

# Basic RPROMPT
RPROMPT='%t'             # Time on right side [web:19]

# Full path on right
RPROMPT='%~' [web:19]

# Combined left and right prompts [web:19]
PROMPT='%n@%m[%h] '      # user@host[history]
RPROMPT='%~'              # path on right

# Color in right prompt
RPROMPT='%F{cyan}%t%f'   # Cyan time

# Multi-element right prompt
RPROMPT='%F{yellow}%D{%L:%M:%S}%f %F{green}%~%f'
# Output format: HH:MM:SS path

# Conditional right prompt (disappears if line is long)
setopt transient_rprompt
RPROMPT='%F{242}[%D{%L:%M:%S}]%f'

# Git-aware right prompt
autoload -Uz vcs_info
precmd() { vcs_info }
setopt prompt_subst
RPROMPT='${vcs_info_msg_0_}'
zstyle ':vcs_info:git:*' formats '%F{magenta}(%b)%f'
```

### Zsh: Advanced Prompt Lab 
```bash
# In zsh, create comprehensive prompt configuration

# Left prompt with status indicator
PROMPT='%(?.%F{green}✓.%F{red}✗%?)%f %F{blue}%~%f %# '
# Explanation:
# %(?. ... . ... )  - Ternary: if exit=0 then ... else ...
# %F{green}         - Green foreground
# %f                - Reset foreground
# %~                - Path with ~ for home
# %#                - % or # for root

# Right prompt with time and host
RPROMPT='%F{cyan}%n@%m%f %F{yellow}%D{%H:%M:%S}%f'

# Test with successful and failed commands
true
echo "Last command succeeded - see green checkmark on left"

false
echo "Last command failed - see red X and exit code on left"

# Dynamic right prompt that shows load average
RPROMPT='%F{cyan}[%D{%H:%M}]%f %F{yellow}$(uptime | awk -F"load average:" '\''{print $2}'\''|cut -d"," -f1)%f'
```

### Zsh: Prompt Escape Sequences Reference
```bash
# In zsh, create cheat sheet

cat << 'EOF'
Zsh Prompt Sequences:
=====================
%n - Username
%m - Hostname (short)
%M - Hostname (full)
%~ - Current path with ~ for home
%/ or %d - Current absolute path
%c or %. - Current directory (basename)
%# - # for root, % for normal user
%h or %! - Current history number
%j - Number of jobs
%D - Date (yy-mm-dd)
%T - Time (24-hour format)
%t or %@ - Time (12-hour am/pm)
%* - Time (24-hour with seconds)
%w - Date (day dd)
%W - Date (mm/dd/yy)
%D{format} - Custom strftime format

Colors:
%F{color} - Set foreground color
%f - Reset foreground
%K{color} - Set background color
%k - Reset background

Colors: black, red, green, yellow, blue, magenta, cyan, white
Or use numbers: 0-255 for 256-color support

Conditional:
%(?.true-text.false-text) - Ternary based on exit status
%(?..✗ ) - Show ✗ only if command failed
EOF
```

### Comparing Bash vs Zsh Prompts Side-by-Side
```bash
# Create comparison script
cat > prompt_compare.sh << 'EOF'
#!/bin/bash
echo "=== BASH PROMPT CONFIGURATION ==="
echo "PS1='\\[\\e[32m\\]\\u@\\h\\[\\e[0m\\]:\\[\\e[34m\\]\\w\\[\\e[0m\\]\\$ '"
echo ""
echo "=== ZSH EQUIVALENT ==="
echo "PROMPT='%F{green}%n@%m%f:%F{blue}%~%f%# '"
echo "RPROMPT='%F{cyan}[%D{%H:%M:%S}]%f'"
echo ""
echo "Zsh advantage: Native right-hand prompt (RPROMPT)"
EOF

chmod +x prompt_compare.sh
./prompt_compare.sh
```