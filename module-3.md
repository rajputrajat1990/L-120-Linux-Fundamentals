# Linux Filesystem - CentOS 9 (2025 Edition)

## LINUX Filesystem Features

CentOS 9 uses **XFS as the default filesystem** (preferred for enterprise workloads and large files), built on Linux kernel 5.14.x with enhanced security and hardware support.   

### Key Filesystem Types & Features

**XFS (Default in CentOS 9)**
- High-performance 64-bit journaling filesystem
- Excellent for large files and parallel I/O operations
- Dynamic inode allocation, metadata journaling
- Commands: `xfs_info`, `xfs_repair`, `xfs_growfs`

**Ext4 (Legacy/Alternative)**
- Backward compatible, stable, widely supported
- Max file size: 16 TB, Max volume: 1 EB
- Journaling for crash recovery
- Commands: `tune2fs`, `dumpe2fs`, `e2fsck`

**Common Filesystem Commands**
```bash
df -hT              # Show filesystem types & disk usage
mount               # Display mounted filesystems
findmnt             # Tree view of mount points
lsblk -f            # Block devices with filesystem info
blkid               # UUID and filesystem type
```

### Advanced Features (CentOS 9)
- **SELinux Integration**: Mandatory access controls at filesystem level 
- **IMA (Integrity Measurement Architecture)**: File integrity verification 
- **ACLs**: `getfacl`, `setfacl` for granular permissions beyond traditional rwx

***

## Filesystem Hierarchy Standard (FHS 3.0)

The FHS defines the directory structure for Unix-like systems; latest version 3.0 republished November 2025. 

### Critical Root-Level Directories

| Directory | Purpose | Characteristics |
|-----------|---------|-----------------|
| `/` | Root - top of hierarchy | All filesystems branch from here  |
| `/boot` | Boot loader files, kernels | Static, required for system startup |
| `/etc` | System configuration files | Text-based configs, no binaries |
| `/home` | User home directories | User data and personal settings |
| `/root` | Root user's home | Separate from `/home` for security |
| `/var` | Variable data (logs, caches) | Grows over time, monitor with `du` |
| `/tmp` | Temporary files | Cleared on reboot (tmpfs) |
| `/usr` | User utilities & applications | Read-only shareable data  |
| `/opt` | Third-party software | Self-contained application packages |
| `/dev` | Device files | Hardware interfaces (disks, terminals) |
| `/proc` | Process & kernel info | Virtual filesystem, runtime data |
| `/sys` | Device & kernel interface | Virtual, sysfs for hardware info |

### Key `/usr` Subdirectories
```
/usr/bin        # User commands (systemwide binaries)
/usr/sbin       # System administration binaries
/usr/lib        # Shared libraries
/usr/lib64      # 64-bit libraries (CentOS 9)
/usr/local      # Locally installed software
/usr/share      # Architecture-independent data
```

### Key `/var` Subdirectories
```
/var/log        # System & application logs
/var/tmp        # Temp files (persists across reboots)
/var/spool      # Print queues, mail spools
/var/lib        # Application state data
```

***

## Navigating the Filesystem

### Essential Navigation Commands

**Location Awareness**
```bash
pwd                 # Print working directory (current location) [web:7][web:10]
pwd -P              # Physical path (resolve symlinks)
```

**Directory Listing**
```bash
ls                  # List files/directories [web:10]
ls -l               # Long format (permissions, owner, size, date)
ls -lh              # Human-readable sizes (K, M, G)
ls -la              # Include hidden files (. prefix)
ls -ltr             # Sort by time, reverse (oldest first)
ls -R               # Recursive listing
ls -i               # Show inode numbers
```

**Directory Navigation**
```bash
cd /path/to/dir     # Change to absolute path [web:10]
cd subdir           # Change to relative path
cd ..               # Parent directory
cd -                # Previous directory (toggle)
cd ~                # Home directory
cd                  # Home directory (shortcut)
```

**Path Types**
- **Absolute**: Start with `/` (e.g., `/etc/systemd/system`)
- **Relative**: From current location (e.g., `../config`)

### Advanced Navigation & Search

**Finding Files**
```bash
find /path -name "pattern"          # Search by name [web:7]
find . -type f -mtime -7            # Files modified in last 7 days
find /etc -name "*.conf"            # Config files in /etc
find /home -size +100M              # Files larger than 100MB
locate filename                      # Fast search (updatedb database)
which command                        # Locate command binary
whereis command                      # Binary, source, manual locations
```

**Directory Operations**
```bash
mkdir dirname                        # Create directory [web:10]
mkdir -p path/to/nested/dir          # Create nested directories
rmdir dirname                        # Remove empty directory [web:10]
rm -rf dirname                       # Remove directory & contents (CAUTION)
```

**Disk Usage Analysis**
```bash
du -sh /path                         # Summarize directory size [web:7]
du -sh *                             # Size of each item in current dir
du -ah --max-depth=1 | sort -rh      # Sorted by size, largest first
df -h                                # Filesystem disk usage
```

**File Operations**
```bash
cp source dest                       # Copy file [web:10]
cp -r sourcedir destdir              # Copy directory recursively
cp -p file dest                      # Preserve permissions & timestamps
mv source dest                       # Move/rename [web:10]
touch filename                       # Create empty file or update timestamp
```

### Pro Tips for Tech Students

**Efficiency Shortcuts**
```bash
pushd /path          # Save current dir & change to /path
popd                 # Return to saved directory
dirs -v              # Show directory stack

tree /path           # Visual directory tree (install tree package)
tree -L 2            # Limit depth to 2 levels

ls -d */             # List only directories
ls -1                # One file per line (scripting)
```

**Wildcard Patterns**
```bash
*                    # Match any characters
?                    # Match single character
[abc]                # Match a, b, or c
[!abc]               # Match any except a, b, c
{jpg,png,gif}        # Match any extension

# Examples:
ls *.log             # All log files
rm file?.txt         # file1.txt, file2.txt, etc.
cp *.{conf,cfg} /backup/
```

**SELinux Context (CentOS 9 Specific)**
```bash
ls -Z                # Show SELinux context [web:4]
stat filename        # Detailed file info (size, inode, timestamps)
file filename        # Identify file type
```



# **Linux Directory & Disk Management** 
## CentOS 9 Stream | 2025 Edition

## **Displaying Directory Contents**

The `ls` command is fundamental for navigating and understanding filesystem contents. 

### Essential Commands

```bash
# Basic listing
ls                          # List files in current directory
ls -l                       # Long format (permissions, owner, size, date)
ls -a                       # Show hidden files (starting with .)
ls -lh                      # Long format with human-readable sizes
ls -lha                     # All files, long format, human-readable

# Pro-level options
ls -lt                      # Sort by modification time (newest first)
ls -ltr                     # Sort by time (oldest first) - reverse order
ls -lS                      # Sort by size (largest first)
ls -lSr                     # Sort by size (smallest first)
ls -R                       # Recursive - list subdirectories
ls -d */                    # List directories only
ls -F                       # Append indicators (/ for dirs, * for executables)
ls -i                       # Show inode numbers
ls -1                       # One file per line (useful for scripting)
```

### **Smart Combinations for Real Work**

```bash
# Find large files quickly
ls -lhS | head -n 10        # Top 10 largest files

# Recently modified files
ls -lht | head -n 10        # Last 10 modified files

# List with full timestamps
ls -l --time-style=full-iso

# Directories only with details
ls -ld */

# Color-coded output (usually default on CentOS 9)
ls --color=auto
```

## **Determining Disk Usage**

Understanding disk consumption is critical for system administration.  

### **df Command - Filesystem Level**

```bash
# Show disk space of mounted filesystems
df                          # Basic output (1K blocks)
df -h                       # Human-readable (GB, MB)
df -H                       # Human-readable (powers of 1000)
df -T                       # Include filesystem type
df -i                       # Show inode usage instead of blocks

# Smart combinations
df -hT                      # Human-readable with filesystem types
df -h /home                 # Specific filesystem/mount point
df -h --output=source,avail,pcent,target  # Custom columns

# Check specific filesystem types
df -hT | grep xfs           # Show only XFS filesystems (default in CentOS 9)
```

### **du Command - Directory/File Level**

```bash
# Analyze disk usage by directory
du -h                       # Human-readable sizes
du -sh                      # Summary (total size only)
du -sh *                    # Size of each item in current directory
du -ah                      # All files including hidden ones
du -ch                      # Show grand total at the end

# Power user commands
du -sh * | sort -rh         # Sort directories by size (largest first)
du -ah /path | sort -rh | head -n 20  # Top 20 space consumers
du -d 1 -h                  # Depth limit = 1 level
du --max-depth=2 -h /var    # Limit depth to 2 levels

# Find largest directories
du -hx --max-depth=1 / 2>/dev/null | sort -rh | head -n 10

# Exclude specific directories
du -sh --exclude='*.log' /var/log
```

### **Pro Tips for 2025**

```bash
# Modern alternative: ncdu (install with: dnf install ncdu)
ncdu /                      # Interactive disk usage analyzer

# Quick filesystem summary
df -h && echo "---" && du -sh /*
```

## **Disk Usage with Quotas**

CentOS 9 uses XFS as the default filesystem, which has built-in quota support.  

### **Enable Quotas on XFS**

```bash
# 1. Enable quota on filesystem (requires remount)
umount /home
mount -o uquota,gquota /dev/sdb1 /home

# 2. Make it persistent - edit /etc/fstab
vi /etc/fstab
# Add: /dev/sdb1 /home xfs defaults,uquota,gquota 0 0

# 3. Verify quota is enabled
mount | grep /home
```

### **User Quota Management**

```bash
# Enter XFS quota tool (expert mode)
xfs_quota -x /home

# Check quota state
xfs_quota> state

# View current usage reports
xfs_quota> report -h                # Human-readable
xfs_quota> report -h -u             # User quotas
xfs_quota> report -h -g             # Group quotas

# Set user limits (soft=warning, hard=absolute limit)
xfs_quota> limit bsoft=4g bhard=5g username

# Non-interactive quota setting
xfs_quota -x -c 'limit bsoft=4g bhard=5g username' /home
```

### **Group Quota Management**

```bash
# Set group quotas
xfs_quota -x -c 'limit -g bsoft=10g bhard=12g developers' /home

# Check group quota usage
xfs_quota -x -c 'report -h -g' /home
```

### **Essential Quota Commands**

```bash
# Check your own quota
quota -s                    # Summary for current user
quota -vs                   # Verbose summary

# Admin: Check any user's quota
quota -u username -s

# Show quota warnings
repquota -a                 # All filesystems
repquota -aug               # All users and groups

# Set grace periods (expert mode)
xfs_quota -x /home
xfs_quota> timer -u -b 7days        # 7-day grace for users
xfs_quota> timer -g -i 14days       # 14-day grace for groups (inodes)
```

### **Monitor & Alert**

```bash
# Install quota warning tools
dnf install quota-warnquota

# Configure warnquota
vi /etc/quotatab
# Add: /dev/sdb1: Your Home Directory

# Send warning emails to users exceeding quotas
warnquota -s

# Automate with cron (run daily)
echo "0 2 * * * /usr/sbin/warnquota -s" | crontab -
```

### **Quick Reference: Quota Terminology**

- **Soft Limit**: Warning threshold - users can exceed temporarily during grace period 
- **Hard Limit**: Absolute maximum - cannot be exceeded under any circumstances 
- **Grace Period**: Time allowed to exceed soft limit (default: 7 days) 
- **Block Quota**: Limits disk space usage 
- **Inode Quota**: Limits number of files 

### **Troubleshooting Commands**

```bash
# Check if quota is enabled on filesystem
mount | grep quota

# View quota enforcement status
xfs_quota -x -c 'state' /home

# Disable quota enforcement (without removing accounting)
xfs_quota -x -c 'disable -ugv' /home

# Re-enable quota enforcement
xfs_quota -x -c 'enable -ugv' /home

# Remove all quotas for a user
xfs_quota -x -c 'limit -u bsoft=0 bhard=0 username' /home
```

# Linux File Permissions & Ownership Cheatsheet (CentOS 9 - 2025 Edition)

## File Ownership

Every file/directory has two owners: a **user** (owner) and a **group**.  

**View ownership:**
```bash
ls -l filename              # Long format shows owner and group
ls -ld directory/           # For directories
stat filename               # Detailed file info
```

**Change ownership:**
```bash
chown user filename                    # Change owner only
chown user:group filename              # Change owner and group
chown user: filename                   # Change owner + set group to user's login group
chown :group filename                  # Change group only (or use chgrp)
chown -R user:group directory/         # Recursive for directories
chown --reference=reffile targetfile   # Match ownership of another file
```

**Pro tip:** Only root can change file ownership; regular users can only change group if they're members of both current and target groups. 

## File and Directory Permissions

**Permission types:**
- **r (read)** - 4: View file contents / List directory contents
- **w (write)** - 2: Modify file / Create/delete files in directory
- **x (execute)** - 1: Run file as program / Enter directory (cd)

**Permission sets (in order):**
1. **User (u)** - File owner
2. **Group (g)** - Group owner
3. **Others (o)** - Everyone else

**Reading permissions:**
```bash
ls -l file.txt
# Output: -rwxr-xr--  1 user group 1234 Dec 02 18:00 file.txt
#         ├─┬─┼─┬─┼─┬─
#         │ │ │ │ │ └─ Others: r-- (4)
#         │ │ │ └───── Group: r-x (5)
#         │ └───────── User: rwx (7)
#         └─────────── File type (- = file, d = directory, l = link)
```

## Changing File Permissions

**Symbolic method:**
```bash
chmod u+x file              # Add execute for user
chmod g-w file              # Remove write from group
chmod o=r file              # Set others to read-only
chmod a+r file              # Add read for all (a = all)
chmod u+x,g+w,o-r file      # Multiple changes
chmod u=rwx,g=rx,o=r file   # Explicit set
```

**Numeric (octal) method:**
```bash
chmod 755 file              # rwxr-xr-x (common for executables)
chmod 644 file              # rw-r--r-- (common for files)
chmod 700 file              # rwx------ (private files)
chmod 775 directory/        # rwxrwxr-x (shared directories)
chmod 600 ~/.ssh/id_rsa     # Security best practice for SSH keys
```

**Recursive operations:**
```bash
chmod -R 755 directory/     # Apply to all files/subdirs
find /path -type f -exec chmod 644 {} \;  # Only files
find /path -type d -exec chmod 755 {} \;  # Only directories
```

## File Creation Permissions (umask)

**umask** defines default permissions by **subtracting** from base permissions (666 for files, 777 for directories). 

```bash
umask                       # View current umask (e.g., 0022)
umask 0002                  # Set new umask (more permissive for group)
umask 0077                  # Restrictive (files: 600, dirs: 700)
```

**Common umask values:**
- **0022** (default): Files=644 (rw-r--r--), Dirs=755 (rwxr-xr-x)
- **0002** (group-friendly): Files=664 (rw-rw-r--), Dirs=775 (rwxrwxr-x)
- **0077** (paranoid): Files=600 (rw-------), Dirs=700 (rwx------)

**Make permanent:**
```bash
echo "umask 0002" >> ~/.bashrc    # Per user
echo "umask 0002" >> /etc/profile # System-wide
```

## SUID and SGID on Files

**SUID (Set User ID) - Bit value: 4000**
- File executes with **owner's** privileges, not executor's  
- Example: `/usr/bin/passwd` (lets users change their password as root)

```bash
chmod u+s file              # Symbolic: Add SUID
chmod 4755 file             # Numeric: SUID + rwxr-xr-x
ls -l file                  # Shows: -rwsr-xr-x (s in owner's execute position)
find / -perm -4000 2>/dev/null  # Find all SUID files (security audit)
```

**SGID (Set Group ID) on files - Bit value: 2000**
- File executes with **group's** privileges  
- Less common than SUID but useful for group-based access

```bash
chmod g+s file              # Symbolic: Add SGID
chmod 2755 file             # Numeric: SGID + rwxr-xr-x
ls -l file                  # Shows: -rwxr-sr-x (s in group's execute position)
```

**Security warning:** SUID/SGID on writable files or scripts = **major security risk**. Audit regularly.  

## SGID and Sticky Bit on Directories

**SGID on directories - Bit value: 2000**
- New files inherit the **directory's group**, not creator's primary group  
- Perfect for collaborative project directories

```bash
mkdir /shared/project
chgrp developers /shared/project
chmod g+s /shared/project           # Add SGID
chmod 2775 /shared/project          # Numeric: SGID + rwxrwxr-x
ls -ld /shared/project              # Shows: drwxrwsr-x (s in group position)
```

**Sticky Bit on directories - Bit value: 1000**
- Users can only delete/rename their **own files**, even with write access  
- Default on `/tmp` to prevent users from deleting each other's files

```bash
chmod +t directory/                 # Symbolic: Add sticky bit
chmod 1777 directory/               # Numeric: Sticky + rwxrwxrwx
ls -ld directory/                   # Shows: drwxrwxrwt (t in others' execute position)
# Example: /tmp has drwxrwxrwt
```

**Combined special bits:**
```bash
chmod 6755 directory/               # SUID + SGID (rare on directories)
chmod 3775 directory/               # SGID + Sticky (collaborative + protected)
```

## User Private Group (UPG) Scheme

**Concept:** In RHEL/CentOS 9, each user automatically gets a **private group** with the same name as their username.  

**Why it matters:**
- Default umask 0002 becomes safer (group-writable OK since only user is in their private group)
- Enables flexible collaboration without compromising security
- Files created with mode 664 (group-writable) are still private by default

**UPG in action:**
```bash
useradd john                # Creates user 'john' AND group 'john'
id john                     # uid=1001(john) gid=1001(john) groups=1001(john)
ls -l ~john/newfile         # Shows: -rw-rw-r-- 1 john john
```

**Creating users with/without UPG:**
```bash
useradd -U alice            # Create with UPG (default behavior)
useradd -N bob              # Create WITHOUT UPG
useradd -g shared charlie   # Use existing group instead of creating UPG
```

**Collaboration pattern with UPG:**
```bash
# Add users to shared group while keeping their UPGs
usermod -aG developers john
usermod -aG developers alice
# Project directory with SGID ensures group ownership
mkdir /projects/webapp
chgrp developers /projects/webapp
chmod 2775 /projects/webapp    # SGID + group-writable
```

***

## Quick Reference Table

| Permission | Numeric | Symbolic | Files | Directories |
|------------|---------|----------|-------|-------------|
| Read | 4 | r | View content | List contents (ls) |
| Write | 2 | w | Modify content | Create/delete files |
| Execute | 1 | x | Run as program | Enter directory (cd) |
| SUID | 4000 | u+s | Run as owner | (no effect) |
| SGID | 2000 | g+s | Run as group | Inherit directory group |
| Sticky | 1000 | +t | (no effect) | Delete only own files |

## Essential Security Commands

```bash
# Audit special permissions (SUID/SGID/Sticky)
find / -perm /6000 -type f 2>/dev/null      # SUID or SGID files
find /tmp -type f ! -perm 1000 2>/dev/null  # Files in /tmp without sticky parent

# Lock down sensitive files
chmod 600 ~/.ssh/id_rsa ~/.ssh/config
chmod 644 ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys

# Find world-writable files (security risk)
find / -perm -002 -type f 2>/dev/null

# Check for files with no owner (orphaned)
find / -nouser -o -nogroup 2>/dev/null
```
