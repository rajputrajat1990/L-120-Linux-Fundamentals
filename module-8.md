## **Linux Archive & Compression**

## **tar - Tape Archive**

### **Core Operations**
```bash
# Create archive (uncompressed)
tar -cvf archive.tar /path/to/dir

# Extract archive
tar -xvf archive.tar

# List contents without extracting
tar -tvf archive.tar

# Extract to specific directory
tar -xvf archive.tar -C /target/path

# Append files to existing archive
tar -rvf archive.tar newfile.txt

# Update archive (only newer files)
tar -uvf archive.tar file.txt

# Delete from archive
tar --delete -f archive.tar file.txt

# Extract specific files
tar -xvf archive.tar file1.txt dir/file2.txt
```

### **tar with Compression (Integrated)**
```bash
# GZIP compression (.tar.gz or .tgz)
tar -czvf archive.tar.gz /path/to/dir     # Create
tar -xzvf archive.tar.gz                  # Extract

# BZIP2 compression (.tar.bz2 or .tbz2)
tar -cjvf archive.tar.bz2 /path/to/dir    # Create
tar -xjvf archive.tar.bz2                 # Extract

# XZ compression (.tar.xz) - Better compression than bzip2
tar -cJvf archive.tar.xz /path/to/dir     # Create
tar -xJvf archive.tar.xz                  # Extract

# ZSTD compression (.tar.zst) - 2025 Modern: Fast + Good ratio
tar --zstd -cvf archive.tar.zst /path     # Create (requires tar 1.31+)
tar --zstd -xvf archive.tar.zst           # Extract
# Or: tar -I zstd -cvf archive.tar.zst /path

# LZ4 compression (.tar.lz4) - 2025 Modern: Fastest
tar --lz4 -cvf archive.tar.lz4 /path      # Create
tar --lz4 -xvf archive.tar.lz4            # Extract

# Auto-detect compression on extract
tar -xavf archive.tar.*                   # Works with any compression
```

### **Advanced tar Options**
```bash
# Preserve permissions (important for backups)
tar -cpvf archive.tar /etc                # -p preserves permissions

# Exclude files/directories
tar -czvf backup.tar.gz --exclude='*.log' --exclude='cache/*' /var/www

# Exclude from file
echo "*.tmp" > exclude.txt
echo "node_modules" >> exclude.txt
tar -czvf backup.tar.gz -X exclude.txt /project

# Include only specific types
find . -name "*.conf" | tar -czvf configs.tar.gz -T -

# Multi-volume archives (split large archives)
tar -czvf - /large/dir | split -b 1G - archive.tar.gz.part

# Compare archive with filesystem
tar -dvf archive.tar                      # Shows differences

# Incremental backups
tar -cvf full.tar --listed-incremental=snapshot.file /data
tar -cvf incr.tar --listed-incremental=snapshot.file /data  # Only changes

# Date-stamped backups
tar -czvf backup-$(date +%Y%m%d-%H%M%S).tar.gz /home

# Verify archive integrity
tar -tzf archive.tar.gz > /dev/null       # Exit 0 if OK
```

### **Piped Compression (Legacy/Alternative Method)**
```bash
# Manual gzip pipeline
tar -cvf - /path | gzip -9 > archive.tar.gz
gunzip -c archive.tar.gz | tar -xvf -

# Manual bzip2 pipeline
tar -cvf - /path | bzip2 -9 > archive.tar.bz2
bunzip2 -c archive.tar.bz2 | tar -xvf -

# Over network (SSH)
tar -czf - /data | ssh user@remote "cat > /backup/data.tar.gz"
ssh user@remote "cat /backup/data.tar.gz" | tar -xzf -
```

## **cpio - Copy In/Out**

### **Core Operations**
```bash
# Create archive (copy-out mode)
find . -depth -print | cpio -ov > archive.cpio
ls | cpio -ov > archive.cpio

# Extract archive (copy-in mode)
cpio -idv < archive.cpio                  # -d creates dirs

# List contents
cpio -tv < archive.cpio

# Copy-through mode (direct copy, no archive)
find /source -depth | cpio -pvdm /destination
```

### **cpio with Compression**
```bash
# Create compressed cpio
find . -type f | cpio -ov | gzip > archive.cpio.gz
find . -type f | cpio -ov | bzip2 > archive.cpio.bz2

# Extract compressed cpio
gunzip -c archive.cpio.gz | cpio -idv
bunzip2 -c archive.cpio.bz2 | cpio -idv

# Specific file types
find . -name "*.txt" | cpio -ov > texts.cpio
```

### **Advanced cpio Usage**
```bash
# With tar format output
find / -name "*.c" | cpio -o -H tar -F c-files.tar

# Preserving file attributes
find . -depth | cpio -pdm --preserve-modification-time /backup

# Pattern matching during extract
cpio -idv "*.conf" < archive.cpio         # Extract only .conf files

# Append to existing archive
find newfiles/ | cpio -oA -F archive.cpio

# Reset ownership on extract
cpio -idv --no-preserve-owner < archive.cpio
```

### **Key Differences: tar vs cpio**
- **tar**: Archive files directly from command line args, self-contained format  
- **cpio**: Reads filenames from stdin (requires `find`/`ls`), more flexible for piping 
- **Modern practice**: Use tar for general archiving; cpio mainly for RPM packages and special cases 

## **compress / uncompress (Legacy)**

The `compress` utility uses LZW algorithm and produces `.Z` files. It's largely obsolete but still present on Unix systems for backward compatibility. 

```bash
# Compress file
compress filename                         # Creates filename.Z, removes original
compress -c filename > filename.Z         # Keep original

# Uncompress file
uncompress filename.Z                     # Creates filename, removes .Z
uncompress -c filename.Z > filename       # Keep .Z file

# View compressed file
zcat filename.Z
zmore filename.Z

# Force compression/overwrite
compress -f filename

# Compression statistics
compress -v filename
```

**Note**: `compress` is NOT available by default on modern CentOS/RHEL. Install with: `dnf install ncompress` 

**Why deprecated**: Poor compression ratio, patent issues (expired), replaced by gzip 

## **gzip / gunzip**

Most widely used compression on Linux - fast, reliable, moderate compression.  

### **Basic Operations**
```bash
# Compress (removes original)
gzip filename                             # Creates filename.gz

# Keep original
gzip -c filename > filename.gz
gzip -k filename                          # -k keeps original (gzip 1.6+)

# Decompress
gunzip filename.gz
gzip -d filename.gz

# Compression levels (1=fast/large, 9=slow/small)
gzip -1 filename                          # Fastest
gzip -9 filename                          # Best compression (default: -6)

# Recursive compression
gzip -r directory/                        # Compresses all files in place

# View compressed content
zcat filename.gz                          # Output to stdout
zmore filename.gz                         # Page through content
zless filename.gz                         # Better pager
zgrep "pattern" file.gz                   # Search in compressed file
zdiff file1.gz file2.gz                   # Compare compressed files
```

### **Advanced gzip**
```bash
# Force overwrite
gzip -f filename

# Test integrity
gzip -t filename.gz

# Verbose output
gzip -v filename

# Multiple files
gzip file1 file2 file3

# Parallel gzip (multi-core, faster)
pigz -9 largefile                         # Install: dnf install pigz
pigz -p 4 filename                        # Use 4 cores
unpigz filename.gz
```

## **bzip2 / bunzip2**

Better compression than gzip, especially for large text files; slower speed.  

### **Basic Operations**
```bash
# Compress (removes original)
bzip2 filename                            # Creates filename.bz2

# Keep original
bzip2 -c filename > filename.bz2
bzip2 -k filename                         # Keep original

# Decompress
bunzip2 filename.bz2
bzip2 -d filename.bz2

# Compression levels (1-9)
bzip2 -1 filename                         # Fastest
bzip2 -9 filename                         # Best (default: -9)

# View compressed content
bzcat filename.bz2
bzmore filename.bz2
bzless filename.bz2
bzgrep "pattern" file.bz2
bzdiff file1.bz2 file2.bz2
```

### **Advanced bzip2**
```bash
# Force overwrite
bzip2 -f filename

# Test integrity
bzip2 -t filename.bz2

# Verbose output
bzip2 -v filename

# Small mode (less memory, slower)
bzip2 -s filename

# Parallel bzip2 (multi-core)
pbzip2 -9 largefile                       # Install: dnf install pbzip2
pbzip2 -p4 filename                       # Use 4 processors
pbunzip2 filename.bz2
```

## **xz / unxz (Modern Alternative)**

Best compression ratio, widely used for software distribution and archival.  

```bash
# Compress
xz filename                               # Creates filename.xz
xz -k filename                            # Keep original
xz -9e filename                           # Extreme compression

# Decompress
unxz filename.xz
xz -d filename.xz

# View compressed content
xzcat filename.xz
xzless filename.xz
xzgrep "pattern" file.xz

# Parallel xz
pxz -9e largefile                         # Install: dnf install pxz
```

## **Compression Comparison (2025)**

| Tool | Ratio | Speed | Decompress | Memory | Modern Use |
|------|-------|-------|------------|--------|-----------|
| **gzip** | 2-3:1 | Fast | Fastest | Low | Logs, web, general   |
| **bzip2** | 3-4:1 | Moderate | Fast | Moderate | Large text, archives  |
| **xz** | 4-6:1 | Slow | Moderate | High | Software dist, long-term storage  |
| **zstd** | 2.5-4:1 | Very Fast | Very Fast | Moderate | **2025 recommended** - best balance   |
| **lz4** | 1.8-2.5:1 | Fastest | Fastest | Low | Real-time, streaming data  |
| **compress** | 1.5-2:1 | Fast | Fast | Low | **Obsolete** - don't use  |

### **When to Use What (2025 Best Practices)**
- **zstd**: Default choice for backups, deployments - fast and efficient  
- **lz4**: Real-time compression, streaming, speed critical 
- **xz**: Maximum compression for distribution packages, archives 
- **gzip**: Compatibility required, web servers, widespread support 
- **bzip2**: Legacy systems, moderate compression needs 

## **Practical Lab Examples**

### **Lab 1: System Backup with tar**
```bash
# Full system backup excluding volatile data
sudo tar -czpvf /backup/system-$(date +%F).tar.gz \
  --exclude=/proc \
  --exclude=/sys \
  --exclude=/dev \
  --exclude=/tmp \
  --exclude=/backup \
  /

# Restore system
sudo tar -xzpvf /backup/system-2025-12-02.tar.gz -C /
```

### **Lab 2: Log File Management**
```bash
# Archive and compress old logs
find /var/log -name "*.log" -mtime +30 | tar -czf oldlogs-$(date +%Y%m).tar.gz -T -

# Search in compressed logs
zgrep "ERROR" /var/log/messages*.gz

# Compare current vs archived logs
zdiff /var/log/secure /var/log/secure-20241201.gz
```

### **Lab 3: Software Deployment**
```bash
# Create deployment package with modern compression
tar --zstd -cvf app-v2.5.tar.zst \
  --exclude='.git' \
  --exclude='node_modules' \
  --exclude='*.log' \
  /opt/myapp

# Deploy over network
tar --zstd -cvf - /opt/myapp | ssh prod@server "tar --zstd -xvf - -C /opt"
```

### **Lab 4: Database Backup**
```bash
# Backup and compress MySQL dump
mysqldump -u root -p mydb | gzip -9 > mydb-$(date +%F).sql.gz

# Backup with xz for maximum compression
pg_dump mydb | xz -9e > mydb-$(date +%F).sql.xz

# Verify and restore
xzcat mydb-2025-12-02.sql.xz | psql mydb
```

### **Lab 5: Incremental Backups**
```bash
# Day 0: Full backup
tar -czpvf /backup/full-$(date +%F).tar.gz \
  --listed-incremental=/backup/snapshot.file \
  /home

# Day 1+: Incremental (only changes)
tar -czpvf /backup/incr-$(date +%F).tar.gz \
  --listed-incremental=/backup/snapshot.file \
  /home

# Restore: Extract full first, then incrementals in order
tar -xzpvf /backup/full-2025-12-01.tar.gz -C /restore
tar -xzpvf /backup/incr-2025-12-02.tar.gz -C /restore
```

### **Lab 6: cpio for RPM Extraction**
```bash
# Extract RPM contents without installing
rpm2cpio package.rpm | cpio -idmv

# Create initramfs (common cpio use)
find . | cpio -o -H newc | gzip > initramfs.img
```

### **Lab 7: Compression Benchmark**
```bash
# Test different methods on same data
cp /var/log/messages testfile

time gzip -9 -c testfile > testfile.gz
time bzip2 -9 -c testfile > testfile.bz2
time xz -9 -c testfile > testfile.xz
time zstd -19 -c testfile > testfile.zst

# Compare results
ls -lh testfile.*
```

### **Lab 8: Automated Backup Script**
```bash
#!/bin/bash
# /usr/local/bin/daily-backup.sh

BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d-%H%M)
SOURCE="/home /etc /var/www"
RETENTION_DAYS=30

# Create compressed backup
tar --zstd -cpvf ${BACKUP_DIR}/backup-${DATE}.tar.zst \
  --exclude='*.tmp' \
  --exclude='cache/*' \
  ${SOURCE}

# Verify integrity
tar --zstd -tvf ${BACKUP_DIR}/backup-${DATE}.tar.zst > /dev/null
if [ $? -eq 0 ]; then
  echo "Backup successful: backup-${DATE}.tar.zst"
else
  echo "Backup verification failed!" | mail -s "Backup Error" admin@example.com
fi

# Cleanup old backups
find ${BACKUP_DIR} -name "backup-*.tar.zst" -mtime +${RETENTION_DAYS} -delete

# Run daily at 2 AM: crontab -e
# 0 2 * * * /usr/local/bin/daily-backup.sh
```