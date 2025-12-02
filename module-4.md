# Directory Manipulation

### Navigation & Inspection
```bash
cd /path/to/dir          # Change directory
cd ~                     # Jump to home directory
cd -                     # Toggle to previous directory
cd ..                    # Move up one level
pwd                      # Print working directory (absolute path)
pwd -P                   # Show physical path (resolves symlinks)

ls -lah                  # Long format, all files, human-readable sizes
ls -lAh                  # Same but exclude . and ..
ls -ltr                  # Sort by modification time (oldest first)
ls -ltd */               # List only directories
ls -li                   # Show inode numbers
```

### Directory Creation & Removal
```bash
mkdir newdir             # Create single directory
mkdir -p path/to/nested  # Create nested directories in one go [web:5]
mkdir -m 755 mydir       # Create with specific permissions
mkdir -p {dir1,dir2,dir3}  # Create multiple directories

rmdir emptydir           # Remove empty directory only [web:5]
rm -r dirname/           # Remove directory and contents recursively [web:5]
rm -rf dirname/          # Force remove (no prompts) - USE WITH CAUTION
rm -ir dirname/          # Interactive removal with confirmation [web:5]
```

### Advanced Directory Operations
```bash
pushd /path/to/dir       # Push directory onto stack and cd to it
popd                     # Return to previous directory from stack
dirs -v                  # Show directory stack with indices

find . -type d -name "pattern"  # Find directories by name
find . -type d -empty    # Find empty directories
find . -maxdepth 2 -type d      # Limit search depth
```

## File Manipulation

### Viewing & Reading
```bash
cat file.txt             # Display entire file
cat -n file.txt          # Show with line numbers
cat -A file.txt          # Show all control characters

less file.txt            # Paginated view (q to quit, / to search)
more file.txt            # Simple paginated view

head file.txt            # First 10 lines [web:7]
head -n 20 file.txt      # First 20 lines
tail file.txt            # Last 10 lines [web:7]
tail -n 50 file.txt      # Last 50 lines
tail -f /var/log/app.log # Real-time log monitoring (Ctrl+C to exit) [web:7]
tail -F /var/log/app.log # Continue monitoring through log rotation [web:7]
```

### Copying & Moving
```bash
cp source.txt dest.txt   # Copy file [web:2]
cp -i file.txt dest/     # Interactive (prompt before overwrite)
cp -r sourcedir/ destdir/  # Recursive copy for directories [web:5]
cp -a source/ dest/      # Archive mode (preserves permissions, timestamps) [web:5]
cp -u file.txt dest/     # Copy only if source is newer
cp --reflink=auto file.txt dest/  # CoW copy on supporting filesystems

mv oldname newname       # Rename file [web:2][web:5]
mv file.txt /path/       # Move file to directory [web:2]
mv -i file.txt dest/     # Interactive move with confirmation
mv -n file.txt dest/     # No overwrite (skip if exists)
mv *.txt textfiles/      # Move multiple files by pattern
```

### Linking Files
```bash
ln target.txt hardlink   # Create hard link (same inode)
ln -s /path/to/target symlink  # Create symbolic link
readlink -f symlink      # Show absolute path of symlink target
ls -li file*             # Compare inode numbers (hard links share inodes)
```

### Searching & Comparing
```bash
find . -name "*.log"     # Find files by name pattern
find . -type f -mtime -7 # Files modified in last 7 days
find . -type f -size +100M  # Files larger than 100MB
find . -type f -exec ls -lh {} \; | sort -k5 -rh | head -10  # Top 10 largest files [web:7]

grep "pattern" file.txt  # Search content in file
grep -r "pattern" dir/   # Recursive search in directory
grep -i "pattern" file   # Case-insensitive search
grep -n "pattern" file   # Show line numbers
grep -v "pattern" file   # Invert match (show non-matching lines)

diff file1 file2         # Compare two files line by line [web:3]
diff -u file1 file2      # Unified format (easier to read)
```

### File Information & Analysis
```bash
file filename            # Detect file type (magic numbers) [web:7]
stat filename            # Detailed file metadata (inode, timestamps, size) [web:7]
wc -l file.txt           # Count lines [web:7]
wc -w file.txt           # Count words
wc -c file.txt           # Count bytes
du -sh /path/to/dir      # Directory size human-readable
du -sh */ | sort -h      # Size of all subdirectories sorted
```

## File Creation & Removal

### Creating Files
```bash
touch newfile.txt        # Create empty file or update timestamp
touch -t 202501011200 file  # Set specific timestamp (YYYYMMDDhhmm)
touch file{1..10}.txt    # Create multiple files (file1.txt to file10.txt)

> newfile.txt            # Create/truncate file to zero bytes
echo "content" > file    # Create file with content (overwrites)
echo "more" >> file      # Append content to file
cat > file.txt           # Type content, Ctrl+D to save
cat >> file.txt          # Append mode, Ctrl+D to save
```

### Removing Files
```bash
rm file.txt              # Remove file [web:2]
rm -i file.txt           # Interactive removal (prompts for confirmation) [web:5]
rm -f file.txt           # Force removal (no prompts)
rm *.tmp                 # Remove multiple files by pattern
rm -- -filename          # Remove files with special characters

shred -vfz -n 10 secret.txt  # Securely overwrite and delete
```

## Physical Unix File Structure

### Inode Architecture
```bash
ls -i filename           # Display inode number [web:6]
stat filename            # Show complete inode metadata including inode number [web:6]
df -i                    # Show inode usage per filesystem
df -i /                  # Check available inodes on root filesystem

# Key inode concepts [web:6][web:9]:
# - Inodes store file metadata: permissions, size, timestamps, owner, block pointers
# - Filenames are NOT stored in inodes; directory entries map names to inodes
# - Hard links share the same inode number
# - Each filesystem has a fixed inode count (set at creation)
```

### Filesystem Structure
```bash
# Standard Linux Filesystem Hierarchy
/              # Root directory
/bin           # Essential user binaries
/boot          # Boot loader files (kernel, initramfs)
/dev           # Device files (block and character devices)
/etc           # System configuration files
/home          # User home directories
/lib, /lib64   # Shared libraries
/mnt           # Temporary mount points
/opt           # Optional software packages
/proc          # Process and kernel information (virtual filesystem)
/root          # Root user's home directory
/run           # Runtime data (PIDs, sockets)
/sbin          # System binaries (admin commands)
/srv           # Service data
/sys           # Kernel and device information (virtual)
/tmp           # Temporary files (cleared on reboot)
/usr           # User utilities and applications
/var           # Variable data (logs, caches, spools)
```

### Block Structure & Direct Access
```bash
df -h                    # Show filesystem disk usage
df -T                    # Show filesystem types (xfs, ext4, etc.)
lsblk                    # List block devices
findmnt                  # Show all mounted filesystems

# Understanding file storage:
# - Data blocks: Actual file content stored in fixed-size blocks
# - Direct pointers: Inode points directly to data blocks (fast access)
# - Indirect pointers: For large files (single, double, triple indirect)
# - Directory = special file containing name→inode mappings [web:6][web:9]

dumpe2fs -h /dev/sda1 | grep -i "block size"  # Show block size (ext4)
xfs_info /dev/sda1       # Show XFS filesystem info (CentOS 9 default)
tune2fs -l /dev/sda1     # Display ext4 filesystem parameters
```

### Pro Tips
```bash
# Find files with duplicate inodes (hard links)
find /path -type f -links +1 -printf '%i %p\n' | sort -n

# Monitor real-time file system events
inotifywait -m -r /path/to/watch -e create,delete,modify

# Find broken symbolic links
find /path -xtype l

# Efficient log monitoring with filtering
tail -f /var/log/messages | grep --line-buffered "ERROR\|WARNING"

# Count files by type in directory
find . -type f | xargs file | awk -F: '{print $2}' | sort | uniq -c | sort -rn

# Show which files are using the most inodes
find /path -xdev -type f | cut -d "/" -f 2 | sort | uniq -c | sort -n

# Copy preserving all attributes including SELinux context
cp --preserve=all source dest
```



## Filesystem Links

### Hard Links Deep Dive
```bash
# Creating and analyzing hard links
ln original.txt hardlink.txt     # Create hard link [web:11]
ls -li original.txt hardlink.txt # Compare inode numbers (identical) [web:11]
stat original.txt                # Check link count in metadata

# Key hard link characteristics [web:11][web:12][web:13]:
# - Share the same inode number (point to same data on disk)
# - Survive original file deletion (data persists as long as one link exists)
# - CANNOT cross filesystem boundaries
# - CANNOT link to directories (except . and .. by system)
# - Must be on same partition/volume
# - Modification to any link affects all (same data blocks)

# Find all hard links to a specific file
find /path -samefile original.txt
find /path -inum 12345678        # Search by inode number

# Count hard links for a file
ls -l file.txt | awk '{print $2}' # Second column shows link count

# Find files with multiple hard links
find /home -type f -links +1     # Files with more than 1 link
find /home -type f -links +2 -ls # Show details of multiply-linked files
```

### Symbolic Links Deep Dive
```bash
# Creating symbolic links
ln -s /path/to/target symlink    # Create symlink [web:15]
ln -s target.txt link.txt        # Relative symlink
ln -s /absolute/path/file link   # Absolute path symlink (more reliable)
ln -sf newtarget existing_link   # Force overwrite existing symlink

# Symlink characteristics [web:11][web:13][web:15]:
# - CAN cross filesystem boundaries
# - CAN link to directories
# - Breaks if original is deleted (becomes "dangling" or "broken")
# - Acts as a pointer/shortcut to filename (not inode)
# - Takes minimal disk space regardless of target size
# - Shows different inode number than target

# Analyzing symlinks
readlink symlink                 # Show where symlink points
readlink -f symlink              # Show canonical absolute path (resolves all symlinks)
ls -l symlink                    # Shows: symlink -> target
namei -l /path/to/symlink        # Show all path components and permissions

# Finding and managing symlinks
find /path -type l               # Find all symlinks
find /path -xtype l              # Find broken/dangling symlinks
find /path -type l -ls           # List symlinks with details
find /path -lname "*pattern*"    # Find symlinks matching target pattern

# Replace broken symlinks
find . -xtype l -delete          # Remove all broken symlinks
find . -xtype l -exec rm {} \;   # Alternative removal method
```

### Advanced Link Operations
```bash
# Create symlinks for entire directory contents
for file in /source/*; do ln -s "$file" /dest/; done

# Check if path is a symlink in scripts
[ -L /path/to/check ] && echo "Is symlink" || echo "Not symlink"

# Copy symlink itself (not target)
cp -P symlink newlocation        # Preserve symlink
cp -d symlink newlocation        # Alternative flag

# Archive with symlinks preserved
tar -czf backup.tar.gz -h dir/   # Follow symlinks (archive target data)
tar -czf backup.tar.gz dir/      # Preserve symlinks as links

# Security consideration: Prevent symlink attacks
ls -l /tmp | grep "^l"           # Check for suspicious symlinks in /tmp
```

## File Extensions and Content

### Magic Numbers & True File Type Detection
```bash
file filename                    # Detect actual file type via magic numbers [web:19]
file -b filename                 # Brief mode (no filename prefix)
file -i filename                 # Show MIME type
file -b --mime-type filename     # MIME type only (parseable)
file -z compressed.gz            # Look inside compressed files
file -L symlink                  # Follow symlinks and check target

# Understanding magic numbers [web:16][web:19]:
# - First bytes of file that identify file type
# - Examples:
#   - JPEG: ff d8 ff e0
#   - PNG: 89 50 4e 47
#   - ZIP: 50 4b 03 04
#   - ELF (Linux executable): 7f 45 4c 46
#   - PDF: 25 50 44 46
```

### Inspecting File Contents vs Extensions
```bash
# Reveal true content (renamed files)
file malware.jpg                 # Might reveal: ELF executable, not JPEG
file document.pdf                # Could be: ASCII text, not PDF

# Batch check file types
find . -type f -exec file {} \; | grep -v "text"  # Find non-text files
for f in *; do echo "$f: $(file -b $f)"; done     # Check all files in directory

# Check character encoding
file -i textfile.txt             # Shows: text/plain; charset=utf-8
file -bi textfile.txt            # Brief mode with charset info

# Identify script types
file script.sh                   # Shows: Bourne-Again shell script, ASCII text
file *.py                        # Identify Python scripts
```

### MIME Type Operations
```bash
# Get MIME type for web applications
mimetype filename                # Alternative to file -i
xdg-mime query filetype file.pdf # Query system MIME associations

# Find files by MIME type
find . -type f -exec file --mime-type {} \; | grep "image/jpeg"
find . -type f -exec file --mime-type {} \; | grep "application/pdf"
```

## Displaying Files (Advanced)

### Context-Aware Display
```bash
# Display with line numbers and pagination
nl filename | less               # Number lines, then paginate
cat -n file | less               # Alternative with cat

# Side-by-side file comparison
pr -m -t file1.txt file2.txt     # Merge files side-by-side
diff -y file1.txt file2.txt      # Side-by-side diff with changes marked
diff --color file1.txt file2.txt # Colored diff output (modern systems)

# Display with context
grep -C 3 "pattern" file         # Show 3 lines before and after match [web:7]
grep -A 5 "pattern" file         # Show 5 lines after match
grep -B 5 "pattern" file         # Show 5 lines before match

# Column-based display
column -t data.txt               # Format into aligned columns
column -s: -t /etc/passwd        # Use : as delimiter and align
```

### Advanced Filtering and Processing
```bash
# Display specific columns
awk '{print $1,$3}' file.txt     # Print columns 1 and 3
cut -d: -f1,3 /etc/passwd        # Extract fields 1 and 3 with : delimiter

# Display with custom separators
paste -d, file1 file2            # Combine files with comma separator
paste -s -d+ numbers.txt         # Join lines with + (useful for math)

# Display file metadata and content
(echo "=== File: $file ==="; stat $file; echo; cat $file) # Show everything

# Numbered display with pattern highlighting
grep -n --color "pattern" file   # Line numbers with color highlighting
```

### Modern Display Alternatives
```bash
# exa - modern ls replacement [web:26][web:28][web:30]
exa -l                           # Long listing with colors
exa -la                          # Include hidden files
exa --tree                       # Tree view of directory structure
exa -l --git                     # Show git status alongside files
exa -lh --sort=size              # Sort by size, human-readable
exa --icons                      # Display file type icons (requires font support)

# Installation:
# dnf install exa  # or: cargo install exa
```

## Previewing Files

### Binary File Inspection
```bash
# Hexadecimal dumps
xxd filename                     # Create hex dump with ASCII representation [web:22][web:23]
xxd -l 256 filename              # First 256 bytes only [web:23]
xxd -g 1 filename                # Group bytes individually
xxd -b filename                  # Binary dump instead of hex [web:23]
xxd -u filename                  # Uppercase hex characters
xxd -c 32 filename               # 32 bytes per line (default is 16)

# View magic numbers (file signature)
xxd -l 16 file.jpg               # First 16 bytes to see file signature [web:19]
hexdump -C file | head           # Alternative with canonical hex+ASCII [web:25]
od -A x -t x1z -v file | head    # Octal dump with hex display

# Reverse hex dump to binary
xxd -r hexdump.txt original.bin  # Convert hex dump back to binary [web:23]

# Alternative hex viewers
hexdump -C filename              # Canonical hex+ASCII dump [web:25]
hd filename                      # Shorthand for hexdump -C
od -Ax -tx1 filename             # Octal dump in hex format
```

### Extracting Readable Content from Binaries
```bash
# Extract printable strings from binary files
strings binary_file              # Show all ASCII strings (4+ chars)
strings -n 10 binary_file        # Minimum string length of 10
strings -t x binary_file         # Show offset in hex
strings -a binary_file           # Scan entire file (not just data sections)
strings binary | grep -i "password"  # Search for specific strings

# Useful for:
# - Finding embedded text in executables
# - Extracting URLs from binaries
# - Finding hardcoded credentials (security audit)
# - Analyzing malware samples
```

### Quick Content Previews
```bash
# Smart preview (first + last)
(head -n 20 file.txt; echo "..."; tail -n 20 file.txt) # Preview both ends

# Preview directory of files
for f in *.txt; do echo "=== $f ==="; head -3 "$f"; echo; done

# Preview with file info
file file.txt && echo && head -20 file.txt

# Preview images in terminal (if supported)
catimg image.jpg                 # Display image in terminal (requires catimg)
timg image.png                   # Terminal image viewer (requires timg)

# Preview PDFs as text
pdftotext file.pdf -             # Extract text to stdout
pdfinfo file.pdf                 # Show PDF metadata
```

### JSON and Structured Data Previewing
```bash
# Pretty-print JSON
jq . data.json                   # Format and colorize JSON
jq '.key' data.json              # Extract specific key
python3 -m json.tool file.json   # Alternative JSON formatter

# XML preview
xmllint --format file.xml        # Format XML
xmllint --xpath '//node' file.xml  # XPath queries

# CSV/TSV preview
column -t -s, data.csv | head    # Align CSV columns
csvlook data.csv                 # Pretty CSV viewer (requires csvkit)
```

## Searching the Filesystem

### Database-Backed File Location
```bash
# locate / plocate - fastest file search [web:20]
locate filename                  # Search entire filesystem via database
locate -i filename               # Case-insensitive search
locate -c pattern                # Count matches
locate -r "\.txt$"               # Regex search (files ending in .txt)
locate -e filename               # Only show existing files (check if still present)
plocate filename                 # Modern, faster locate replacement [web:20]

# Update locate database
updatedb                         # Rebuild database (run as root)
updatedb -U /specific/path       # Update specific path only

# locate limitations:
# - Database may be outdated (updated daily via cron)
# - Won't find very recently created files
# - Only searches filenames, not content [web:20]
```

### Command and Binary Location
```bash
# which - shows executable path in $PATH [web:27]
which python                     # Find which python binary will execute
which -a python                  # Show all matching executables in $PATH
which ls                         # Shows: /usr/bin/ls (or alias)

# whereis - finds binary, source, and man pages [web:27]
whereis ls                       # Shows: /usr/bin/ls /usr/share/man/man1/ls.1.gz
whereis -b python                # Binary location only
whereis -m bash                  # Man page location only
whereis -s gcc                   # Source code location only

# type - shows how command will be interpreted
type ls                          # May show: ls is aliased to `ls --color=auto`
type -a python                   # Show all definitions (alias, function, binary)
type -t cd                       # Show type only: builtin

# command - bypass aliases and functions
command -v ls                    # Show actual command path
command ls                       # Execute actual ls, not alias
```

### Modern Search Tools (2025 Best Practices)

```bash
# fd - modern find alternative [web:17]
# Installation: dnf install fd-find  # or: cargo install fd-find
fd pattern                       # Simple filename search (respects .gitignore)
fd pattern /path                 # Search in specific directory
fd -H pattern                    # Include hidden files
fd -I pattern                    # Don't respect .gitignore
fd -e txt pattern                # Search by extension
fd -t f pattern                  # Files only (-t d for directories)
fd -x ls -lh {}                  # Execute command on results

# Why fd > find [web:17]:
# - Faster (parallel execution)
# - Simpler syntax
# - Colored output
# - Respects .gitignore by default
# - Smart case sensitivity
```

### Content Search Tools
```bash
# ripgrep (rg) - blazing fast grep alternative [web:17][web:20]
# Installation: dnf install ripgrep  # or: cargo install ripgrep
rg "pattern"                     # Search content in current directory [web:20]
rg -i "pattern"                  # Case-insensitive
rg -l "pattern"                  # List files with matches only
rg -n "pattern"                  # Show line numbers
rg -C 3 "pattern"                # 3 lines of context
rg --type py "pattern"           # Search Python files only
rg --type-list                   # Show supported file types
rg "pattern" --hidden            # Include hidden files
rg "pattern" --no-ignore         # Don't respect .gitignore

# Why ripgrep > grep [web:20]:
# - 10-100x faster than grep
# - Respects .gitignore automatically
# - Recursive by default
# - Smart case sensitivity
# - Compressed file support
# - Colored output by default

# ripgrep searches FILE CONTENT, plocate searches FILE NAMES [web:20]
```

### Advanced find Operations (Beyond Basics)
```bash
# Time-based searches with precision
find . -type f -newermt "2025-01-01" ! -newermt "2025-02-01"  # Date range
find . -type f -mmin -60             # Modified in last 60 minutes
find . -type f -amin -30             # Accessed in last 30 minutes
find . -type f -cmin +120            # Status changed more than 2 hours ago

# Permission-based searches
find . -type f -perm 0777            # Exact permission match
find . -type f -perm -0644           # At least these permissions
find . -type f ! -perm -0400         # Files not readable by owner
find . -type f -perm /u+w,g+w        # Writable by owner OR group

# Owner-based searches
find . -type f -user username        # By username
find . -type f -uid 1000             # By UID
find . -type f -nouser               # Files with no valid user
find . -type f -nogroup              # Files with no valid group

# Complex logical operations
find . \( -name "*.log" -o -name "*.tmp" \) -mtime +30  # OR condition
find . -type f -size +10M ! -name "*.iso"  # Large files excluding ISOs
find . -type f -name "*.sh" -exec chmod +x {} \;  # Make scripts executable

# Performance optimization
find . -type f -name "*.txt" -quit   # Stop after first match
find /path -maxdepth 2 -name pattern # Limit recursion depth
```

### Pro Search Combinations
```bash
# Find largest files using modern tools
fd -t f -x du -h {} | sort -rh | head -20

# Search filename AND content
fd -e txt -x rg -l "pattern" {}

# Find files modified today
find . -type f -mtime 0 -ls

# Find and count file types
fd -t f | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Search with exclusions
rg "pattern" -g '!*.log' -g '!node_modules'  # Exclude patterns
fd -E node_modules -E "*.log" pattern        # Exclude with fd

# Find duplicate filenames (different paths)
fd -t f | xargs -n1 basename | sort | uniq -d
```