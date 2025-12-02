# Linux Multi-User Concepts - 2025 Edition

## Multi-User System Architecture

Linux is a **true multi-user operating system** where multiple users can simultaneously access system resources (CPU, memory, disk, files) without interference. Each user has isolated home directories, process spaces, and permission-based access control. 

**Key Concepts to Master:**
- **UID (User ID)**: Numeric identifier (0 = root, 1-999 = system, 1000+ = regular users)
- **GID (Group ID)**: Group membership for shared resource access
- **Process Isolation**: Each user's processes run in separate security contexts
- **Resource Allocation**: Kernel manages fair CPU time and memory distribution 

***

## Root User (UID 0)

The **superuser** with unlimited system privileges - the most powerful and dangerous account.  

**Modern Best Practices (2025):**
- **Never use root for daily tasks** - Use regular account + sudo
- **Disable direct root SSH login** in production environments
- **Use sudo for granular control** - log all privileged actions  
- **Implement RBAC** (Role-Based Access Control) for team environments 

**Root Powers:**
- Install/remove software, modify system files
- Create/delete user accounts, change any password
- Access all files regardless of permissions
- Potentially destroy the entire system with one wrong command

***

## Switching User Contexts

### `su` (Substitute User)

| Command | Environment | Use Case | Security Level |
|---------|------------|----------|----------------|
| `su username` | **Keeps current env** | Quick user switch | Low - requires target password |
| `su - username` | **Full login shell** | Complete user context | Low - requires target password |
| `su` or `su -` | Switch to root | Emergency admin | **Dangerous - full root access** |

**Critical Difference - `su` vs `su -`:**
```bash
su alice       # Keeps YOUR $PATH, $HOME stays yours, PWD unchanged
su - alice     # Alice's $PATH, $HOME=/home/alice, PWD=/home/alice
               # Runs alice's .bash_profile and login scripts
```

**Lab Observation:** Use `env` or `pwd` after each command to see the differences. 

### `sudo` (SuperUser DO) - **Preferred in 2025**

**Why sudo is superior**:  
- Granular permission control (specific commands only)
- **Audit logging** - tracks who did what and when
- No need to share root password
- Time-limited elevated access (default 15 min timeout)
- Configurable via `/etc/sudoers` (use `visudo` to edit)

```bash
sudo command                    # Run single command as root
sudo -u alice command           # Run command as alice
sudo -i                         # Interactive root shell (like su -)
sudo -s                         # Root shell (keeps environment)
sudo -l                         # List your sudo privileges
```

**2025 Security Standard:** Configure sudo for specific commands only, never grant blanket access. 

***

## Gathering Login Session Info

### Essential Commands

| Command | Output | Best For |
|---------|--------|----------|
| `whoami` | Your current username | Quick identity check |
| `who` | All logged-in users + terminals + login time | Real-time monitoring   |
| `w` | Users + what they're doing + CPU usage | Detailed activity analysis  |
| `last` | Historical login/logout records | Security audits, forensics  |
| `id` | UID, GID, group memberships | Permission troubleshooting |
| `users` | Simple list of logged-in usernames | Quick count |

### Advanced Usage

```bash
# Who with detailed info
who -H                          # Add column headers
who -b                          # Last system boot time [web:10]
who -q                          # Quick count of users [web:10]
who -a                          # All information (comprehensive)

# W command - most informative
w                               # Full dashboard
w username                      # Filter by specific user

# Last command - historical
last                            # Recent logins
last -n 10                      # Last 10 entries
last username                   # Specific user history
last reboot                     # System reboot history
lastb                           # Failed login attempts (requires root)

# ID command variations
id                              # Your identity
id username                     # Another user's identity
id -u                           # Just UID
id -g                           # Just primary GID
id -G                           # All group IDs
```

***

## Getting Help

**Mastery Hierarchy** (from quick to comprehensive):

```bash
# Level 1: Quick Syntax
command --help                  # Brief usage summary (fastest)
command -h                      # Alternative for some commands

# Level 2: Manual Pages
man command                     # Full documentation
man -k keyword                  # Search across all man pages (apropos)
man 5 passwd                    # Specific section (1=commands, 5=files, 8=admin)

# Level 3: Info Pages (GNU-specific, more detailed)
info command                    # Hyperlinked documentation
info coreutils                  # Collection of core utilities

# Level 4: Documentation Files
ls /usr/share/doc/package-name  # README, examples, changelogs

# Level 5: Community Resources
tldr command                    # Simplified examples (install tldr first)
```

**Man Page Sections (Know These):**
- **1**: User commands (ls, cp, chmod)
- **5**: File formats (/etc/passwd, /etc/shadow)
- **8**: System administration (useradd, systemctl)

**Navigation in man/info:**
- `Space` = next page, `b` = previous page
- `/keyword` = search forward, `?keyword` = search backward
- `q` = quit

***

## Pro Tips

**Security Wisdom:**
1. **Always use `sudo` over `su`** - Better audit trail and principle of least privilege  
2. **Monitor failed logins** with `lastb` - Detect brute force attempts
3. **Check `w` for suspicious processes** - Users running unexpected commands
4. **Verify sudo logs** in `/var/log/auth.log` or `/var/log/secure`

**Troubleshooting Power Moves:**
```bash
# Check if user is in sudo group
groups username

# View sudo configuration
sudo cat /etc/sudoers

# Test what sudo access you have
sudo -l

# Switch user and verify environment loaded
su - alice -c 'echo $HOME && pwd'

# Find all logged-in sessions including SSH
who -a | grep -E 'pts|tty'
```

**Common Gotchas:**
- `su` without `-` doesn't source target user's environment → PATH issues
- Sudo timeout can cause unexpected permission denials → Use `sudo -v` to refresh
- `last` reads from `/var/log/wtmp` → Can be rotated, use `last -f /var/log/wtmp.1` for older data



# Linux User & Group Management - 2025 Edition

## Creating Users

### Basic User Creation
```bash
# Create a user (minimal)
sudo useradd username

# Create user with home directory and default shell (recommended)
sudo useradd -m -s /bin/bash username

# Create user with specific UID, home dir, and description
sudo useradd -u 1500 -m -d /home/username -s /bin/bash -c "Full Name" username

# Create system user (for services, no home dir, UID < 1000)
sudo useradd -r -s /usr/sbin/nologin serviceuser
```

**Pro Options**:  
- `-m` : Create home directory automatically
- `-s /bin/bash` : Set default shell
- `-c "comment"` : Add description/full name
- `-u UID` : Specify custom user ID
- `-g GROUP` : Set primary group during creation
- `-G GROUP1,GROUP2` : Add to supplementary groups
- `-e YYYY-MM-DD` : Set account expiration date

## Setting & Managing Passwords

```bash
# Set/change password for user
sudo passwd username

# Force password change on next login
sudo passwd -e username

# Lock user account (disable login)
sudo passwd -l username
sudo usermod -L username

# Unlock user account
sudo passwd -u username
sudo usermod -U username

# Set password aging (expires in 90 days)
sudo chage -M 90 username

# View password aging info
sudo chage -l username
```

**Security Best Practice**: Always use `passwd -e` for new users to force them to set their own password on first login. 

## Deleting Users

```bash
# Delete user only (keeps home directory and files)
sudo userdel username

# Delete user WITH home directory and mail spool (complete removal)
sudo userdel -r username

# Force delete even if user is logged in
sudo userdel -f username
```

**Smart Admin Tip**: Before deleting, check for running processes: `ps -u username` and find all user files: `sudo find / -user username` 

## Modifying Users

```bash
# Change username
sudo usermod -l newname oldname

# Change user ID (UID)
sudo usermod -u 2000 username

# Change user's home directory
sudo usermod -d /new/home/path -m username  # -m moves contents

# Change user's default shell
sudo usermod -s /bin/zsh username

# Change user's comment/description
sudo usermod -c "New Description" username

# Lock/unlock account
sudo usermod -L username  # Lock
sudo usermod -U username  # Unlock

# Set account expiration date
sudo usermod -e 2025-12-31 username
```

## Creating Groups

```bash
# Create a new group
sudo groupadd groupname

# Create group with specific GID
sudo groupadd -g 3000 groupname

# Create system group (GID < 1000)
sudo groupadd -r systemgroup
```

## Managing Groups

```bash
# Rename a group
sudo groupmod -n newname oldname

# Change group GID
sudo groupmod -g 4000 groupname

# View all groups
cat /etc/group
getent group

# View specific group details
getent group groupname
```

## Adding Users to Groups

```bash
# Add user to supplementary groups (MOST IMPORTANT - preserves existing groups)
sudo usermod -aG group1,group2,group3 username

# Alternative method using gpasswd
sudo gpasswd -a username groupname

# Add user to multiple groups (overwrites existing - DANGEROUS)
sudo usermod -G group1,group2 username  # Missing -a removes other groups!
```

**Critical Warning**: Always use `-aG` (append to groups) with `usermod`. Using `-G` without `-a` will remove the user from all other supplementary groups!  

## Changing Primary Group

```bash
# Change user's primary group (GID in /etc/passwd)
sudo usermod -g newprimarygroup username

# Verify user's groups
groups username
id username
```

**Understanding Primary vs Supplementary**: 
- Primary group: Files created by user belong to this group (one only)
- Supplementary groups: Additional access permissions (can have many)

## Deleting Groups

```bash
# Delete a group
sudo groupdel groupname

# Cannot delete if it's a user's primary group - change primary group first
sudo usermod -g newgroup username
sudo groupdel oldgroup
```

## Essential Verification Commands

```bash
# Check user's groups
groups username
id username

# List all users
cat /etc/passwd
getent passwd

# List all groups  
cat /etc/group
getent group

# View detailed user info
sudo getent passwd username
sudo getent shadow username  # Password aging info

# Check who's logged in
who
w
last username

# Find all files owned by user
sudo find / -user username 2>/dev/null

# Find all files owned by group
sudo find / -group groupname 2>/dev/null
```

## Key Configuration Files

| File | Purpose | Permissions |
|------|---------|-------------|
| `/etc/passwd` | User account info | 644 (world-readable) |
| `/etc/shadow` | Encrypted passwords & aging | 000 (root only) |
| `/etc/group` | Group information | 644 (world-readable) |
| `/etc/gshadow` | Secure group info | 000 (root only) |
| `/etc/login.defs` | Default useradd values | 644 |
| `/etc/skel/` | Template for new home dirs | 755 |

## Modern 2025 Best Practices

**Security Essentials**:  
```bash
# 1. Find accounts with empty passwords (CRITICAL security risk)
sudo awk -F: '($2 == "") {print $1}' /etc/shadow

# 2. Enforce password expiration
sudo chage -M 90 -m 7 -W 14 username  # Max 90d, min 7d, warn 14d before

# 3. Find inactive accounts
sudo lastlog | grep -i never

# 4. Use strong password policies
sudo apt install libpam-pwquality  # Ubuntu/Debian
sudo dnf install libpwquality      # RHEL/Fedora
```

**Principle of Least Privilege**: 
- Create role-based groups: `developers`, `webadmins`, `dbusers`
- Grant sudo access per command, not full root: Edit via `sudo visudo`
- Regular audits: `sudo grep -i "NOPASSWD" /etc/sudoers.d/*`

**Automation-Friendly Commands**: 
```bash
# Batch create users from file
while IFS=, read -r user fullname; do
    sudo useradd -m -s /bin/bash -c "$fullname" "$user"
    sudo passwd -e "$user"
done < users.csv
```

## Quick Reference Card

| Task | Command |
|------|---------|
| Create user + home | `sudo useradd -m -s /bin/bash username` |
| Set password | `sudo passwd username` |
| Delete user only | `sudo userdel username` |
| Delete user + files | `sudo userdel -r username` |
| Add to groups (safe) | `sudo usermod -aG group1,group2 username` |
| Change primary group | `sudo usermod -g newgroup username` |
| Lock account | `sudo usermod -L username` |
| Create group | `sudo groupadd groupname` |
| Delete group | `sudo groupdel groupname` |
| Check user's groups | `id username` |

## Pro Tips

1. **Always use `-aG` not `-G`** - This single mistake can lock users out of critical system groups  

2. **Check before you delete** - Use `sudo find / -user username -ls` to see what you're about to orphan 

3. **Shadow file is your security friend** - Master `chage` command for password policies 

4. **System users are different** - Use `-r` flag for service accounts with UIDs < 1000 

5. **Audit regularly** - Run `sudo lastlog` and `getent passwd` to find dormant/suspicious accounts  