# Linux Process & Job Control Cheatsheet (CentOS 9 - 2025 Edition)

## What is a Process?

A process is an instance of a program in execution with its own memory space, file descriptors, security context, and system resources. In CentOS 9, processes are managed by the systemd init system (PID 1) and organized using cgroups (control groups) for resource management.  

**Key Process Attributes:**
- **PID (Process ID)**: Unique identifier assigned by kernel
- **PPID (Parent PID)**: PID of the process that created it
- **UID/GID**: User and group ownership
- **Nice value**: Priority (-20 highest to +19 lowest)
- **Memory space**: Virtual memory allocated (RSS, VSZ)
- **File descriptors**: stdin (0), stdout (1), stderr (2), and open files
- **SELinux context**: Security label on CentOS 9 (e.g., `system_u:system_r:httpd_t:s0`)

## Process Creation and States

**Process Creation Methods:**
```bash
# Fork-exec model (parent clones itself, then exec replaces with new program)
# systemd spawns most services via unit files
systemctl start httpd.service

# Manual process creation
./myprogram &              # Background process
nohup ./longrun.sh &       # Immune to HUP signal
setsid ./daemon.sh         # New session leader (detached from terminal)
```

**Process States:** 
| State | Code | Meaning |
|-------|------|---------|
| Running | `R` | Executing on CPU or runqueue |
| Sleeping | `S` | Interruptible sleep (waiting for event) |
| Uninterruptible Sleep | `D` | Cannot be interrupted (usually I/O wait) |
| Stopped | `T` | Suspended via SIGSTOP/SIGTSTP |
| Zombie | `Z` | Terminated but parent hasn't reaped exit status |
| Idle | `I` | Kernel idle thread (CentOS 9) |

**Check Process State:**
```bash
ps -eo pid,ppid,stat,cmd           # All processes with state
ps aux | awk '$8 == "D"'            # Find D-state processes (I/O bottleneck indicator)
cat /proc/<PID>/status | grep State # Detailed state from procfs
```

## Viewing Processes

**Essential Commands:**
```bash
# ps - Static snapshot
ps aux                    # BSD style: all users, detailed info
ps -ef                    # Unix style: full format listing [web:1]
ps -eLf                   # Show threads (LWP - Light Weight Process)
ps -u username            # Filter by user [web:1]
ps -C httpd               # Filter by command name
ps --ppid 1               # All direct systemd children
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem  # Custom format, sorted

# top - Real-time monitoring [web:1]
top -u username           # Filter by user
top -p 1234,5678          # Monitor specific PIDs
# Inside top:
# Press '1' - Show per-CPU stats
# Press 'c' - Show full command path
# Press 'k' - Kill process interactively
# Press 'M' - Sort by memory
# Press 'P' - Sort by CPU
# Press 'f' - Select display fields

# htop (install: dnf install htop)
htop -u username          # User filter, better UI
htop -t                   # Tree view (process hierarchy)

# pstree - Hierarchical view
pstree -p                 # Show PIDs
pstree -a                 # Show command arguments
pstree -u                 # Show UID transitions

# pgrep/pidof - Find PID by name
pgrep -u root httpd       # Find all root's httpd processes
pgrep -l java             # Show name with PID
pgrep -f "python.*flask"  # Match against full command line
pidof systemd             # Returns only PIDs (simpler)

# systemd-specific process viewing [web:10]
systemctl status httpd.service    # Service process tree
systemd-cgls                       # cgroup hierarchy view
systemd-cgtop                      # cgroup resource usage (like top)
```

**Advanced Process Inspection:**
```bash
# /proc filesystem - Direct kernel interface
ls -l /proc/<PID>/fd/              # Open file descriptors
cat /proc/<PID>/cmdline            # Full command line
cat /proc/<PID>/environ            # Environment variables
cat /proc/<PID>/limits             # Resource limits
cat /proc/<PID>/cgroup             # cgroup membership
cat /proc/<PID>/status             # Comprehensive state info

# lsof - List open files
lsof -p <PID>                      # All files/sockets for a process
lsof -u username                   # All files opened by user
lsof -i :80                        # Processes using port 80
lsof /var/log/messages             # What's accessing this file?

# strace - System call tracer
strace -p <PID>                    # Attach to running process
strace -c ./program                # Count syscalls (performance profiling)
strace -e open,read ./program      # Filter specific syscalls
```

## Signals

Signals are asynchronous notifications sent to processes. They interrupt normal execution to handle events (termination, suspend, user-defined actions).  

**Critical Signals Table:**
| Signal | Number | Default Action | Can Block? | Use Case |
|--------|--------|----------------|------------|----------|
| SIGHUP | 1 | Terminate | Yes | Reload config (daemons)  |
| SIGINT | 2 | Terminate | Yes | Ctrl+C from terminal |
| SIGQUIT | 3 | Core dump | Yes | Ctrl+\ (quit with dump) |
| SIGKILL | 9 | Terminate | **NO** | Force kill   |
| SIGSEGV | 11 | Core dump | Yes | Segmentation fault |
| SIGTERM | 15 | Terminate | Yes | Graceful shutdown (default)  |
| SIGSTOP | 19 | Stop | **NO** | Pause process |
| SIGCONT | 18 | Continue | Yes | Resume stopped process |
| SIGTSTP | 20 | Stop | Yes | Ctrl+Z (job control) |
| SIGUSR1 | 10 | Terminate | Yes | User-defined |
| SIGUSR2 | 12 | Terminate | Yes | User-defined |

**Signal Handling Behavior:**
```bash
# Processes can:
# 1. Use default handler (terminate, ignore, stop, etc.)
# 2. Catch and handle signal with custom function
# 3. Ignore signal (SIG_IGN)
# SIGKILL and SIGSTOP cannot be caught, blocked, or ignored [web:5]

# List all signals
kill -l                            # Show signal names/numbers
trap -l                            # Bash built-in signal list

# Common signal patterns
# SIGTERM (15): "Please exit gracefully, cleanup resources"
# SIGKILL (9): "Die immediately, kernel handles it"
# SIGHUP (1): "Daemon: reload your config file" [web:5]
```

## Tools to Send Signals

```bash
# kill - Send signal by PID [web:1][web:2]
kill <PID>                         # Sends SIGTERM (15) by default
kill -15 <PID>                     # Explicit SIGTERM
kill -TERM <PID>                   # Named signal
kill -9 <PID>                      # SIGKILL - force kill [web:1][web:8]
kill -KILL <PID>                   # Same as -9
kill -1 <PID>                      # SIGHUP - reload config
kill -STOP <PID>                   # Pause process
kill -CONT <PID>                   # Resume process
kill -l                            # List all signals

# killall - Kill by process name [web:2]
killall httpd                      # Kill all httpd processes [web:2]
killall -9 httpd                   # Force kill all httpd [web:8]
killall -u username                # Kill all processes by user
killall -SIGHUP rsyslog            # Reload rsyslog
killall -r "python.*"              # Regex pattern matching

# pkill - Advanced pattern-based killing [web:2]
pkill -u username                  # Kill all user's processes [web:2]
pkill -TERM httpd                  # Graceful httpd termination
pkill -9 -f "java.*tomcat"         # Match full command line
pkill -P <PPID>                    # Kill all children of parent
pkill -t pts/0                     # Kill all processes on terminal pts/0
pkill --signal SIGUSR1 myapp       # Send custom signal

# systemctl kill - systemd-aware signaling [web:10]
systemctl kill httpd.service                    # Send SIGTERM to main process
systemctl kill --signal=SIGHUP httpd.service    # Reload daemon [web:10]
systemctl kill --kill-who=all httpd.service     # Signal all cgroup processes
systemctl kill --kill-who=control httpd.service # Signal only control process

# timeout - Send signal after duration
timeout -s SIGTERM 30s ./longrun.sh   # TERM after 30 seconds
timeout -s 9 1h ./backup.sh           # KILL after 1 hour

# Alternative: Direct signaling via /proc
echo "signal" > /proc/<PID>/comm              # (rarely used)
```

## Job Control Basics

Job control manages processes within a shell session, allowing foreground/background switching and suspension.

**Core Concepts:**
- **Foreground job**: Receives terminal input, blocks shell prompt
- **Background job**: Runs without terminal input, shell prompt available
- **Job ID**: Shell-assigned identifier (e.g., ` `, ` `)
- **Process group**: Collection of processes (pipeline = one job)
- **Session**: Group of process groups, typically bound to terminal

**Job Control Commands:**
```bash
# Starting jobs
./script.sh              # Foreground (shell waits)
./script.sh &            # Background (immediate prompt) [web:1]
nohup ./script.sh &      # Background + immune to SIGHUP

# Suspending jobs
# Ctrl+Z                 # Sends SIGTSTP, suspends foreground job

# Listing jobs
jobs                     # Show all jobs in current shell
jobs -l                  # Include PIDs
jobs -r                  # Only running jobs
jobs -s                  # Only stopped jobs

# Foreground/Background switching
fg                       # Bring most recent job to foreground
fg %1                    # Bring job #1 to foreground
fg %?script              # Foreground job matching "script"
bg                       # Resume most recent stopped job in background
bg %2                    # Resume job #2 in background

# Disowning jobs
disown                   # Remove most recent job from job table
disown %1                # Disown specific job
disown -a                # Disown all jobs
disown -r                # Disown running jobs

# Example workflow:
./compile.sh             # Start in foreground
# Press Ctrl+Z           # Suspend it
bg                       # Resume in background
jobs                     # Verify it's running
fg %1                    # Bring back to foreground
```

## Jobs

**Managing Multiple Jobs:**
```bash
# Start multiple background jobs
make all &               # Job  
rsync -avz /src /dst &   # Job  
./test.sh &              # Job  

# Job references
%1 or %make              # Job number or command prefix
%%                       # Current job (most recent)
%+                       # Current job (same as %%)
%-                       # Previous job
%?sync                   # Job with "sync" in command

# Practical examples
kill %2                  # Kill job #2
kill -9 %?rsync          # Force kill rsync job
wait %1                  # Wait for job #1 to complete
wait                     # Wait for all background jobs

# Job persistence across terminal close
# jobs are tied to shell session, lost on exit unless:
nohup ./script.sh &      # Redirects output to nohup.out
disown %1                # Detach from shell (but not terminal)
# Use screen/tmux for true persistence [web:6]

# Check job status
jobs -l
 + 12345 Running    make all &
 - 12346 Stopped    ./test.sh

# + indicates current job
# - indicates previous job
```

## Screen

GNU Screen is a terminal multiplexer that enables persistent sessions, detaching/reattaching, and multiple virtual terminals.  

**Installation & Basics:**
```bash
# Install (CentOS 9)
sudo dnf install screen

# Start new session
screen                           # Unnamed session
screen -S mysession              # Named session [web:9]

# Session management
screen -ls                       # List all sessions [web:9]
screen -r                        # Reattach to detached session [web:9]
screen -r mysession              # Reattach to named session [web:9]
screen -R                        # Reattach or create new [web:9]
screen -x mysession              # Multi-display attach (share session) [web:9]
screen -d mysession              # Detach session remotely
screen -X -S mysession quit      # Kill session remotely [web:9]

# Key concept: All screen commands start with Ctrl+a (release, then next key) [web:6]
```

## Using Screen

**Essential Screen Commands:** 
```bash
# Inside a screen session:

# Session control
Ctrl+a d                 # Detach from session (keeps running) [web:6]
Ctrl+a D D               # Detach and logout
Ctrl+a \                 # Kill all windows and terminate
Ctrl+a :quit             # Exit screen (kills all windows)

# Window management
Ctrl+a c                 # Create new window [web:6]
Ctrl+a n                 # Next window [web:6]
Ctrl+a p                 # Previous window [web:6]
Ctrl+a 0-9               # Switch to window 0-9
Ctrl+a "                 # Window list (interactive) [web:6]
Ctrl+a w                 # Window bar (shows all windows) [web:6]
Ctrl+a k                 # Kill current window [web:6]
Ctrl+a A                 # Rename current window [web:6]

# Window navigation
Ctrl+a '                 # Prompt for window number/name
Ctrl+a Ctrl+a            # Toggle between current and previous window [web:6]

# Copy mode (scrollback)
Ctrl+a [                 # Enter copy mode (scroll with vi/emacs keys)
Ctrl+a ]                 # Paste copied text
Ctrl+a Esc               # Enter copy mode (alternative)

# Help and info
Ctrl+a ?                 # Show key bindings
Ctrl+a :                 # Enter command mode [web:6]
```

**Practical Screen Workflows:**
```bash
# Remote session persistence (SSH disconnects)
ssh user@server
screen -S deployment
./deploy.sh              # Long-running deployment
# Press Ctrl+a d to detach
exit                     # Close SSH
# Later...
ssh user@server
screen -r deployment     # Resume exactly where you left off

# Run remote command in detached screen [web:6]
screen -dmS backup /usr/local/bin/backup.sh     # -d: start detached, -m: force new
screen -S backup -X stuff "cd /var/log\n"       # Send commands to session [web:6]

# Multi-window development environment
screen -S dev
# Window 0: vim
# Ctrl+a c
# Window 1: compilation
# Ctrl+a c
# Window 2: testing
# Ctrl+a " (to navigate between them)
```

## Advanced Screen

**Configuration File (~/.screenrc):** 
```bash
# Create configuration
nano ~/.screenrc

# Essential .screenrc settings [web:6]:

# Change escape key from Ctrl+a to Ctrl+j (avoid conflict with bash)
escape ^Jj

# Disable startup message
startup_message off

# Increase scrollback buffer
defscrollback 10000

# Custom status bar (bottom)
hardstatus alwayslastline
hardstatus string '%{= kG}[ %{G}%H %{g}][%= %{= kw}%?%-Lw%?%{r}(%{W}%n*%f%t%?(%u)%?%{r})%{w}%?%+Lw%?%?%= %{g}][%{B} %m-%d %{W}%c %{g}]'

# Start windows automatically on session creation
screen -t editor 0 vim
screen -t shell 1 bash
screen -t logs 2 tail -f /var/log/messages

# Custom key bindings
bind ^k kill                    # Ctrl+a Ctrl+k kills window
bind K kill                     # Uppercase K also kills
bind ^] paste .                 # Ctrl+a ] paste buffer
bind r source ~/.screenrc       # Ctrl+a r reload config

# Logging
deflog on                       # Enable logging for new windows
logfile /tmp/screen-%t.log      # Log file naming

# Encoding
defutf8 on                      # UTF-8 support [web:6]

# Window monitoring
activity "Activity in %n (%t)"  # Alert on activity

# Zombie mode (keep window after process exit)
zombie kr                       # k=destroy, r=resurrect on key press
```

**Advanced Screen Features:**
```bash
# Split screen (regions) [web:6]
Ctrl+a S                 # Horizontal split
Ctrl+a |                 # Vertical split (capital pipe)
Ctrl+a Tab               # Move to next region
Ctrl+a X                 # Close current region
Ctrl+a Q                 # Close all regions except current

# After splitting, switch to new region and open window:
# Ctrl+a Tab, then Ctrl+a c (or switch to existing window)

# Logging and monitoring
Ctrl+a H                 # Toggle logging for current window
Ctrl+a M                 # Monitor window for activity
Ctrl+a _                 # Monitor window for 30 sec silence

# Multiuser screen sharing (pair programming)
# User 1 (starts session):
screen -S shared
Ctrl+a :multiuser on
Ctrl+a :acladd user2     # Grant access to user2

# User 2 (joins session):
screen -x user1/shared   # Attach to user1's "shared" session

# Locking
Ctrl+a x                 # Lock screen (password protected)

# Executing commands in screen session from outside [web:9]
screen -S mysession -X stuff "ls -la\n"           # Send "ls -la" + Enter [web:9]
screen -S backup -X screen tail -f /var/log/backup.log  # Create new window with command

# Screen serial console (hardware debugging)
screen /dev/ttyUSB0 115200      # Connect to serial device at 115200 baud

# Screen as poor man's process manager
# .screenrc for production app:
screen -t webapp 0 /usr/local/bin/webapp
screen -t worker1 1 /usr/local/bin/worker --id 1
screen -t worker2 2 /usr/local/bin/worker --id 2
screen -t logs 3 tail -f /var/log/app.log

# Then start with: screen -c production.screenrc
```

**Screen vs tmux vs systemd (2025 Context):**
```bash
# Screen: Lightweight, universal, good for SSH persistence [web:6]
screen -dmS job ./script.sh

# tmux: Modern alternative, better defaults, active development
# (install: dnf install tmux)
tmux new -s job -d ./script.sh

# systemd: Proper service management (production workloads) [web:7][web:10]
# Create /etc/systemd/system/myapp.service:
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/myapp
Restart=always
User=appuser

[Install]
WantedBy=multi-user.target

# Then: systemctl enable --now myapp.service
```

**Pro Tips:**
- Use `screen -ls` regularly to clean up dead sessions
- For production services, prefer systemd over screen 
- Screen sessions survive SSH disconnects but not server reboots
- Combine screen with `nohup` for ultra-resilience (redundant but safe)
- On CentOS 9, check if tmux is more appropriate for your workflow
- Screen logs: `Ctrl+a H` then check `/tmp/screen-*.log`

# Linux Process & Job Control Lab Exercises (2025 Edition)

## Lab 1: Multi-tasking with Shell Jobs

**Exercise: Background Job Management Workflow**

```bash
# Step 1: Start multiple long-running tasks
dd if=/dev/zero of=/tmp/testfile1 bs=1M count=5000 &
# Output:  12345

dd if=/dev/zero of=/tmp/testfile2 bs=1M count=5000 &
# Output:  12346

find / -name "*.log" 2>/dev/null > /tmp/logfiles.txt &
# Output:  12347

# Step 2: Monitor all jobs
jobs -l
# Expected output:
#  12345 Running    dd if=/dev/zero of=/tmp/testfile1 bs=1M count=5000 &
#  - 12346 Running    dd if=/dev/zero of=/tmp/testfile2 bs=1M count=5000 &
#  + 12347 Running    find / -name "*.log" 2>/dev/null > /tmp/logfiles.txt &

# Step 3: Check which jobs are still running
jobs -r                   # Only running jobs
ps -p 12345,12346,12347   # Verify with ps

# Step 4: Wait for specific job to complete
wait %1                   # Blocks until dd job 1 finishes
echo "Job 1 completed with exit code: $?"

# Step 5: Clean up remaining jobs
kill %2 %3                # Terminate jobs 2 and 3
jobs                      # Verify they're terminated
```

**Exercise: Foreground/Background Juggling**

```bash
# Start a process in foreground
top

# Suspend it (press Ctrl+Z)
# Output:  + Stopped    top

# Resume in background
bg %1
jobs
# Output:  + Running    top &

# Bring back to foreground
fg %1

# Exit top (press 'q')

# Start multiple vim editors
vim file1.txt &
vim file2.txt &
vim file3.txt &

# List them
jobs
#  Stopped    vim file1.txt
#  - Stopped    vim file2.txt
#  + Stopped    vim file3.txt

# Bring specific vim to foreground
fg %?file2              # Match pattern "file2"

# Cleanup: kill all vim jobs
killall vim
```

## Lab 2: Advanced Job Control

**Exercise: Pipeline Jobs**

```bash
# Pipelines are treated as a single job
cat /var/log/messages | grep error | wc -l &
# Output:  12350

jobs -l
#  + 12350 Running    cat /var/log/messages | grep error | wc -l &

# Check all PIDs in the job (pipeline has multiple processes)
ps -g $(ps -o pgid= -p 12350) -o pid,pgid,cmd
# Shows all processes in the process group

# Killing the job kills entire pipeline
kill %1
```

**Exercise: Disowning Jobs (Survive Shell Exit)**

```bash
# Start a long task
sleep 3600 &
# Output:  12360

# Check job
jobs -l
#  + 12360 Running    sleep 3600 &

# Disown it (remove from job table, no longer receives SIGHUP)
disown %1

jobs
# (empty - job removed from shell's job table)

# Process still running
ps -p 12360
# 12360 ?   00:00:00 sleep

# Test: exit shell and reconnect - process survives
exit
# SSH back in
ps aux | grep 12360        # Still running

# Cleanup
kill 12360
```

**Exercise: nohup vs disown vs screen**

```bash
# Method 1: nohup (redirects output to nohup.out) [web:11]
nohup ./long_script.sh &
# Output: nohup: ignoring input and appending output to 'nohup.out'
tail -f nohup.out          # Monitor output

# Method 2: disown (no output capture)
./long_script.sh > /tmp/output.log 2>&1 &
disown

# Method 3: Use subshell with setsid (completely detached)
(setsid ./long_script.sh > /tmp/output.log 2>&1 &)

# Verify all three are immune to terminal close
ps aux | grep long_script
exit
# SSH back and verify processes still running
```

## Lab 3: Fork Bomb (CONTROLLED ENVIRONMENT ONLY)

**⚠️ WARNING: Only run in a VM or isolated lab environment**  

**Pre-Lab: Set Protective Limits ** 

```bash
# Step 1: Check current process limits
ulimit -u
# Output: 12288 (example - yours may differ)

# Step 2: Count your current processes [web:12]
pgrep -wcu $USER
# Output: 145 (example)

# Step 3: Set a SAFE soft limit (current + buffer) [web:12]
ulimit -S -u 200
# Allows only 200 total processes for this shell session

# Step 4: Verify the limit is set
ulimit -u
# Output: 200

# Step 5: Open a SECOND terminal WITHOUT limits for cleanup
# Terminal 2: Keep this open to kill runaway processes
```

**Exercise: Controlled Fork Bomb Test ** 

```bash
# In Terminal 1 (with limits):

# Simple fork bomb (DO NOT RUN WITHOUT LIMITS)
# :(){ :|:& };:
# Translation: Function named ":" that calls itself twice, backgrounded

# Instead, use a LIMITED fork bomb for learning:
fork_test() {
    for i in {1..50}; do
        sleep 100 &
    done
}

# Run it
fork_test

# Observe process explosion
pgrep -u $USER sleep | wc -l
# Output: 50 (or near your limit)

# Try to create more processes - SHOULD FAIL
sleep 200 &
# Output: bash: fork: Resource temporarily unavailable [web:11][web:12]

# System protections worked!
```

**Lab: Cleanup Fork Bomb ** 

```bash
# In Terminal 2 (no limits):

# Method 1: pkill by name
pkill -9 -u $USER sleep    # Kill all sleep processes [web:12]

# Method 2: killall
killall -9 sleep

# Method 3: pgrep + xargs
pgrep -u $USER sleep | xargs kill -9

# Verify cleanup
pgrep -u $USER sleep
# (no output = success)

# In Terminal 1:
# Remove the limit
ulimit -S -u 12288         # Reset to original [web:12]
```

**Post-Lab: System-Wide Protection (Root Only)**

```bash
# Permanent limits via /etc/security/limits.conf [web:11][web:13]
sudo vim /etc/security/limits.conf

# Add these lines:
# username    soft    nproc    1000
# username    hard    nproc    2000
# @students   soft    nproc     500
# @students   hard    nproc    1000

# Apply limits for new logins
# Current sessions not affected - need to logout/login

# Verify limits for user
su - username
ulimit -u
# Output: 1000 (soft limit)

ulimit -Hu
# Output: 2000 (hard limit)
```

## Lab 4: Process Management Tools Examination

**Exercise: System State Analysis**

```bash
# Scenario: System feels slow, investigate

# Step 1: Overall system snapshot
uptime
# Output: 14:35:12 up 7 days,  3:22,  5 users,  load average: 2.45, 1.89, 1.42

# Step 2: Identify top CPU consumers
ps aux --sort=-%cpu | head -10

# Step 3: Identify top memory consumers
ps aux --sort=-%mem | head -10

# Step 4: Check for D-state processes (I/O bottleneck indicator)
ps aux | awk '$8 == "D" {print $0}'
# D-state = uninterruptible sleep (usually disk I/O)

# Step 5: Find zombie processes
ps aux | awk '$8 == "Z" {print $0}'
# Zombies indicate parent not reaping children

# Step 6: Inspect specific suspicious process
PID=12345
ls -l /proc/$PID/fd/          # Open file descriptors
cat /proc/$PID/limits         # Resource limits
cat /proc/$PID/status         # Detailed state
lsof -p $PID                  # All open files/sockets

# Step 7: Real-time monitoring with top
top -d 1 -n 10 > /tmp/top_snapshot.txt    # Capture 10 iterations
# Press 'M' to sort by memory
# Press 'P' to sort by CPU
# Press 'c' to show full command
# Press 'k' and enter PID to kill

# Step 8: Tree view of process hierarchy
pstree -aup                   # Full tree with args, users, PIDs
pstree -p $PID                # Subtree from specific process

# Step 9: systemd service examination
systemctl list-units --type=service --state=running
systemd-cgtop                 # Real-time cgroup resource usage
systemd-cgls                  # cgroup hierarchy tree
```

**Exercise: Detective Work - Find Resource Hogs**

```bash
# Create test scenario
stress-ng --cpu 2 --timeout 600s &
stress-ng --vm 1 --vm-bytes 512M --timeout 600s &

# Detective work begins:

# 1. What's using all the CPU?
top -b -n 1 | grep stress-ng

# 2. Get the PIDs
pgrep -a stress-ng
# Output:
# 12400 stress-ng --cpu 2 --timeout 600s
# 12401 stress-ng --vm 1 --vm-bytes 512M --timeout 600s

# 3. Detailed inspection
ps -o pid,ppid,cmd,%cpu,%mem,stat,etime,user -p 12400,12401

# 4. What files are they accessing?
lsof -p 12400

# 5. System call activity
strace -c -p 12400    # Attach and count syscalls (Ctrl+C to stop)

# 6. Check CPU affinity
taskset -cp 12400
# Output: pid 12400's current affinity list: 0,1,2,3

# 7. Examine memory maps
cat /proc/12401/maps | head -20

# 8. Check cgroup membership
cat /proc/12400/cgroup

# Cleanup
killall stress-ng
```

## Lab 5: GUI Process Management Tools (CentOS 9)

**Exercise: GNOME System Monitor **  

```bash
# Installation
sudo dnf install gnome-system-monitor

# Launch from terminal
gnome-system-monitor &

# OR: Launch from GUI
# Activities -> System Monitor

# Tasks to perform:
# 1. Click "Processes" tab
# 2. Sort by CPU (click "CPU %" column)
# 3. Right-click a process -> Properties (see detailed info)
# 4. Right-click a process -> Memory Maps (see memory layout)
# 5. Right-click a process -> Kill Process
# 6. Click "Resources" tab -> View CPU/Memory/Network graphs

# Command-line equivalent for each GUI action:
# Sort by CPU: ps aux --sort=-%cpu
# Properties: cat /proc/<PID>/status
# Memory Maps: cat /proc/<PID>/maps
# Kill: kill <PID>
# Resources: top / htop
```

**Exercise: KDE System Guard (Plasma Desktop) **  

```bash
# Installation (if using KDE)
sudo dnf install plasma-systemmonitor ksysguard

# Launch
ksysguard &

# OR modern version:
plasma-systemmonitor &

# Tasks [web:15]:
# 1. Tab: System Load -> View CPU, Memory, Swap graphs
# 2. Tab: Process Table -> Kill processes
# 3. Create custom monitoring tab:
#    - Click "File" -> "New Tab"
#    - Drag sensors from left panel
#    - Add: CPU usage, Disk I/O, Network throughput
#    - Create multi-graph dashboard

# Advanced: Monitor specific disk I/O [web:15]
# - Sensor Browser -> Disk -> sda -> Read/Write rates
# - Drag to custom tab -> Choose "Line Graph" display

# Command-line equivalents:
# Disk I/O: iostat -x 1
# Network: iftop or nethogs
# Custom dashboard: Use tmux with multiple terminals running specific tools
```

## Lab 6: Cleanup Operations - The Kill Suite

**Exercise: Selective Process Termination**

```bash
# Setup: Create multiple test processes
for i in {1..10}; do
    bash -c "while true; do echo worker_$i; sleep 5; done" &
done

# Verify creation
pgrep -a bash | grep worker
# Output: Multiple bash processes

# Cleanup Method 1: kill specific PID
PID=$(pgrep -f worker_1)
kill $PID
ps -p $PID        # Verify it's gone

# Cleanup Method 2: killall by name
killall -9 bash   # WARNING: Kills ALL bash processes
# Use with caution!

# Safer: killall with user filter
killall -u $USER -9 bash

# Cleanup Method 3: pkill with pattern [web:17]
for i in {1..10}; do
    bash -c "while true; do echo test_worker_$i; sleep 5; done" &
done

pkill -f "test_worker"      # Pattern match on full command
pgrep -f "test_worker"      # Verify cleanup (no output)

# Cleanup Method 4: pkill by parent PID
sleep 1000 &
PARENT=$!
bash -c "sleep 500" &
bash -c "sleep 500" &

pkill -P $PARENT           # Kill all children of PARENT
ps --ppid $PARENT          # Verify (no output)

# Cleanup Method 5: pkill by terminal
tty                        # Find your terminal
# Output: /dev/pts/0

pkill -t pts/0             # Kills all processes on pts/0
# WARNING: Kills your current shell too!
```

**Exercise: Advanced pgrep/pkill Patterns**

```bash
# Setup various processes
firefox &
firefox --private-window &
google-chrome &
python3 -m http.server 8000 &
python3 manage.py runserver &

# pgrep examples
pgrep -a firefox             # All firefox processes with commands
pgrep -c firefox             # Count only
pgrep -n firefox             # Newest only
pgrep -o firefox             # Oldest only
pgrep -l -u $USER            # All user's processes with names

# pkill with signals
pkill -STOP firefox          # Pause all firefox processes
ps aux | grep firefox        # Check status (should show 'T')
pkill -CONT firefox          # Resume them

# pkill specific Python app
pkill -f "http.server"       # Kill Python HTTP server only
pkill -f "manage.py"         # Kill Django dev server only

# Cleanup all browsers
pkill -9 "firefox|chrome"    # Regex pattern (force kill)

# Verify cleanup
pgrep -a "firefox|chrome|python3"
```

## Lab 7: Screen Session Creation & Management

**Exercise: Basic Screen Session ** 

```bash
# Step 1: Start named session
screen -S lab_session

# You're now inside screen (notice command prompt might change)

# Step 2: Create multiple windows
# Press Ctrl+a c (creates window 0)
echo "Window 0: Main shell"

# Press Ctrl+a c (creates window 1)
top

# Press Ctrl+a c (creates window 2)
tail -f /var/log/messages

# Step 3: Navigate windows
# Press Ctrl+a " (shows window list)
# Use arrow keys to select, press Enter

# OR:
# Press Ctrl+a 0 (switch to window 0)
# Press Ctrl+a 1 (switch to window 1)
# Press Ctrl+a n (next window)
# Press Ctrl+a p (previous window)

# Step 4: Rename windows for clarity
# Press Ctrl+a A
# Type: "Shell"
# Press Enter

# Switch to window 1
# Press Ctrl+a 1
# Press Ctrl+a A
# Type: "CPU Monitor"

# Step 5: View window bar
# Press Ctrl+a w
# Output at bottom: 0$ Shell  1$ CPU Monitor  2$ Logs

# Step 6: Detach from session
# Press Ctrl+a d
# Output: [detached from 12345.lab_session]
```

**Exercise: Reattaching & Session Persistence**

```bash
# List all screen sessions
screen -ls
# Output:
# There is a screen on:
#     12345.lab_session    (Detached)
# 1 Socket in /run/screen/S-username.

# Reattach to session
screen -r lab_session

# You're back! All windows still running

# Detach again
# Press Ctrl+a d

# Start a long-running process in detached screen
screen -dmS backup_job
screen -S backup_job -X stuff "rsync -avz /data /backup\n"

# Check it's running
screen -ls
ps aux | grep rsync

# Monitor the session
screen -r backup_job
# Watch rsync progress
# Press Ctrl+a d when done observing

# Kill session remotely (without attaching)
screen -X -S backup_job quit
screen -ls                     # Verify it's gone
```

## Lab 8: Screen Session Sharing (Multi-User)

**⚠️ Requires two user accounts or SSH sessions ** 

**User 1: Session Owner**

```bash
# Step 1: Start shared session
screen -S shared_lab

# Step 2: Enable multiuser mode [web:20]
# Press Ctrl+a :
# Type: multiuser on
# Press Enter

# Step 3: Grant access to user2 [web:20]
# Press Ctrl+a :
# Type: acladd user2
# Press Enter

# Step 4: Verify permissions
# Press Ctrl+a :
# Type: aclchg user2 +rwx "#?"
# Press Enter
# (Grants read, write, execute on all windows)

# Step 5: Share session name with user2
echo "Session ready: user1/shared_lab"

# Do some work
vim shared_document.txt
# Type something...
```

**User 2: Joining Session**

```bash
# Step 1: Check available sessions
screen -ls user1/
# Output: There is a screen on:
#     12345.shared_lab    (Multi, attached)

# Step 2: Attach to user1's session [web:20]
screen -x user1/shared_lab

# You now see EXACTLY what user1 sees!
# Both users can type simultaneously

# Step 3: Test collaboration
# User1 types in vim: "User 1 says hello"
# User2 sees it in real-time
# User2 types: "User 2 says hello back"
# User1 sees it in real-time

# Step 4: Create independent window while sharing
# Press Ctrl+a c
# User2's new window (user1 won't see this until they switch to it)

# Step 5: Detach (user1's session continues)
# Press Ctrl+a d

# User1 remains connected
```

**Exercise: Screen Pair Programming Setup**

```bash
# User 1 (Instructor):
screen -S coding_session
# Press Ctrl+a :
# multiuser on
# Press Ctrl+a :
# acladd student1
# Press Ctrl+a :
# acladd student2

# Create windows for different tasks
# Window 0: Code editor
vim main.c

# Press Ctrl+a c
# Window 1: Compilation
gcc main.c -o program

# Press Ctrl+a c
# Window 2: Testing
./program

# Students join:
# student1: screen -x instructor/coding_session
# student2: screen -x instructor/coding_session

# All three see the same view, can collaborate in real-time

# Instructor controls which window everyone sees
# Press Ctrl+a 0  (everyone sees code)
# Press Ctrl+a 1  (everyone sees compilation)
```

## Lab 9: Advanced Split Screen Sessions

**Exercise: Vertical & Horizontal Splits**

```bash
# Start screen session
screen -S split_demo

# Step 1: Horizontal split
# Press Ctrl+a S (capital S)
# Screen splits into two regions (top empty, bottom active)

# Step 2: Move to new region
# Press Ctrl+a Tab

# Step 3: Open window in new region
# Press Ctrl+a c
# OR switch to existing window:
# Press Ctrl+a 0

# Now you have two views!

# Step 4: Create vertical split (in either region)
# Press Ctrl+a | (vertical pipe)
# Creates side-by-side split

# Step 5: Navigate regions
# Press Ctrl+a Tab (cycles through regions)

# Step 6: Populate each region
# Region 1: top
# Press Ctrl+a Tab
# Region 2: htop
# Press Ctrl+a Tab
# Region 3: tail -f /var/log/messages

# Step 7: Resize regions
# Press Ctrl+a :
# Type: resize 20    (20 lines for current region)
# Press Enter

# Step 8: Close specific region
# Navigate to region you want to close
# Press Ctrl+a X (capital X)

# Step 9: Close all regions except current
# Press Ctrl+a Q (capital Q)
```

**Exercise: Multi-Panel Monitoring Dashboard**

```bash
# Create comprehensive monitoring layout
screen -S monitoring

# Create 4-panel dashboard:

# Panel 1 (top-left): CPU monitoring
# Press Ctrl+a S (split horizontal)
# Press Ctrl+a Tab (move to new region)
# Press Ctrl+a c
top
# Press Ctrl+a A (rename window)
# Type: CPU

# Panel 2 (top-right): Network monitoring
# Press Ctrl+a | (split vertical)
# Press Ctrl+a Tab
# Press Ctrl+a c
watch -n 1 'netstat -i'
# Press Ctrl+a A
# Type: Network

# Panel 3 (bottom-left): Disk I/O
# Press Ctrl+a Tab (back to top-left)
# Press Ctrl+a S (split horizontal again)
# Press Ctrl+a Tab (move down)
# Press Ctrl+a c
watch -n 1 'iostat -x 1 2'
# Press Ctrl+a A
# Type: Disk

# Panel 4 (bottom-right): System logs
# Press Ctrl+a | (split vertical)
# Press Ctrl+a Tab
# Press Ctrl+a c
tail -f /var/log/messages
# Press Ctrl+a A
# Type: Logs

# Result: 4-panel dashboard showing CPU, network, disk, and logs simultaneously

# Save this layout (add to ~/.screenrc for automatic loading):
# Press Ctrl+a :
# Type: layout save monitoring
# Press Enter

# To reload layout later:
# Press Ctrl+a :
# Type: layout load monitoring
```

**Exercise: Split Screen + Session Sharing**

```bash
# User 1: Create shared split screen [web:20]
screen -S shared_split
# Press Ctrl+a S (split)
# Press Ctrl+a Tab
# Press Ctrl+a c
# Set up your layout

# Enable sharing
# Press Ctrl+a :
# multiuser on
# Press Ctrl+a :
# acladd user2

# User 2: Join and see the same split layout
screen -x user1/shared_split

# Both users see identical split screen layout
# Can navigate independently if needed:
# User2 can press Ctrl+a Tab to move between regions
# Changes affect only their view, not user1's focus
```

**Pro Lab Tips:**
- Use `screen -S $(date +%Y%m%d)_task` for date-named sessions
- Create `~/.screenrc` with `defscrollback 50000` for large scrollback 
- Use `screen -ls | grep Detached | cut -d. -f1 | xargs -I {} screen -X -S {} quit` to cleanup all detached sessions
- Combine screen with `tmux` command: `alias better_screen='tmux'` (tmux has better defaults for 2025)
- For production monitoring, use `systemd` services instead of screen sessions

These labs cover everything from basic job control through advanced screen multiplexing and collaborative terminal sessions.       
