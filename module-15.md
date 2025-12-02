# Linux File Transfer & Web Utilities (2025 Edition)

## FTP, NcFTP, and lftp

### Critical Security Context
FTP transmits credentials and data in **plaintext** and is considered insecure in 2025. Modern alternatives include SFTP (SSH File Transfer Protocol) and FTPS (FTP over TLS/SSL) for production environments.  

### lftp - Advanced FTP Client (Recommended)

**Installation**
```bash
dnf -y install lftp
```

**Basic Connection & Authentication**
```bash
lftp -u username hostname              # Interactive password prompt
lftp -u username,password ftp.site.com # Inline credentials
lftp ftp://user:[email protected]      # URI format
```

**Essential Navigation & Operations**
```bash
pwd                    # Show remote directory
!pwd                   # Show local directory
ls                     # List remote files
!ls -l                 # List local files
cd directory           # Change remote directory
lcd /local/path        # Change local directory
```

**File Transfer Commands**
```bash
get file.txt                          # Download single file
mget *.txt                            # Download multiple files (pattern)
put localfile.txt                     # Upload single file
mput file1.txt file2.txt             # Upload multiple files
mirror /remote/dir /local/dir        # Mirror remote to local (recursive)
mirror -R /local/dir /remote/dir     # Reverse mirror (upload)
pget -n 5 largefile.iso              # Parallel download (5 connections)
```

**Advanced Features**
```bash
set xfer:clobber on                  # Overwrite existing files
set net:limit-rate 100000:50000      # Limit download:upload rate (bytes/s)
queue get file.tar.gz                # Queue downloads
queue start                          # Process queue
jobs                                 # Show running jobs
set ssl:verify-certificate no        # Skip SSL verification (insecure!)
```

**Bookmarks for Automation** 
```bash
bookmark add myserver ftp://user:[email protected]/path
bookmark list
open myserver && get file.txt        # Open bookmark and execute
```

**Scripting with lftp**
```bash
lftp -c 'open ftp.site.com; user username password; get file.txt; bye'
lftp -f script.txt                   # Execute commands from file
```

**Configuration File**: `~/.lftprc` or `~/.config/lftp/rc`
```bash
set ftp:passive-mode on
set net:timeout 30
set net:max-retries 5
bmk:save-passwords true              # Save passwords in bookmarks
```

### NcFTP - User-Friendly FTP Client

**Installation**
```bash
dnf install ncftp
```

**Key Features**  
```bash
ncftp ftp.site.com                   # Connect with prompt for credentials
ncftpget -R ftp://site.com/dir/      # Recursive download (-R flag)
ncftpput -R ftp://site.com/dest/ source/  # Recursive upload
ncftpls ftp://site.com/directory/    # List remote directory without login
```

**Automatic Reconnection**: NcFTP automatically resumes interrupted transfers and maintains connection stability. 

### Traditional FTP Client

**Basic Usage** (Avoid in production - use lftp instead)
```bash
ftp hostname
> user username
> password
> binary                             # Set binary transfer mode
> ascii                              # Set ASCII mode
> hash                               # Show progress indicators
> prompt off                         # Disable prompts for mget/mput
> mget *.zip                         # Download multiple files
> bye
```

***

## wget - Non-Interactive Network Downloader

**Installation** (usually pre-installed)
```bash
dnf install wget
```

### Basic Downloads
```bash
wget URL                                    # Download single file
wget -O custom_name.zip URL                # Save with custom name
wget -c URL                                # Resume interrupted download
wget -b URL                                # Background download
wget -i urls.txt                           # Download from URL list
```

### Advanced Download Options  
```bash
wget --limit-rate=200k URL                 # Limit speed to 200 KB/s
wget --tries=10 URL                        # Set retry attempts
wget --timeout=30 URL                      # Set timeout (seconds)
wget --wait=2 --random-wait URL            # Random wait between requests
wget --user-agent="Mozilla/5.0" URL        # Custom user agent
wget --header="Accept: text/html" URL      # Custom headers
```

### Authentication & Security
```bash
wget --user=username --password=pass URL   # HTTP authentication
wget --no-check-certificate URL            # Skip SSL verification (insecure!)
wget --secure-protocol=TLSv1_2 URL         # Force TLS version
wget --http-user=user --http-password=pass # HTTP basic auth
wget --ftp-user=user --ftp-password=pass   # FTP credentials
```

### Recursive Downloads & Mirroring   
```bash
# Basic recursive download
wget -r URL                                # Recursive download (default depth: 5)
wget -r -l 3 URL                           # Limit recursion depth to 3 levels
wget -r --no-parent URL                    # Don't ascend to parent directory

# Website mirroring (complete offline copy)
wget --mirror --convert-links --page-requisites --no-parent URL
# Breakdown:
#   --mirror: Enable recursive with infinite depth + timestamping
#   --convert-links: Convert links for offline viewing
#   --page-requisites: Download CSS, JS, images
#   --no-parent: Stay within directory

# Advanced mirroring with rate limiting
wget -mpEk --no-parent --random-wait -e robots=off URL
# -m: mirror, -p: page-requisites, -E: adjust extensions
# -k: convert links, robots=off: ignore robots.txt
```

### Domain & File Type Filtering 
```bash
wget -r --domains=example.com,example.org URL    # Restrict to domains
wget -r --exclude-domains=ads.example.com URL    # Exclude domains
wget -r -A pdf,zip URL                           # Accept only PDF and ZIP
wget -r -R jpg,png,gif URL                       # Reject image files
wget -r --accept-regex='.*\.pdf$' URL            # Accept by regex
wget -r --reject-regex='.*\.(jpg|gif)$' URL      # Reject by regex
```

### Directory & Output Control
```bash
wget -P /custom/path URL                   # Save to specific directory
wget -nc URL                               # No clobber (skip existing files)
wget -N URL                                # Timestamping (download only newer)
wget -nd -r URL                            # No directories (flatten)
wget --cut-dirs=2 -r URL                   # Skip N directory levels
```

### REST API Interaction 
```bash
wget --method=POST --body-data='{"key":"value"}' URL
wget --header="Content-Type: application/json" --post-data='{}' URL
wget -S URL                                # Show server response headers
wget --spider URL                          # Check if URL exists (no download)
```

### Logging & Debugging
```bash
wget -o download.log URL                   # Log to file
wget -a download.log URL                   # Append to log
wget -d URL                                # Debug output
wget -v URL                                # Verbose
wget -q URL                                # Quiet mode
wget --progress=dot URL                    # Different progress indicator
```

**Configuration File**: `/etc/wgetrc` or `~/.wgetrc`

***

## lynx - Text-Based Web Browser

**Installation**
```bash
dnf install lynx
```

### Basic Usage 
```bash
lynx https://example.com               # Open website
lynx -dump URL                         # Dump formatted output to stdout
lynx -source URL                       # Dump raw HTML source
lynx -listonly -dump URL               # List all links only
lynx -listonly -nonumbers -dump URL    # Links without numbers
```

### Navigation Keys (Interactive Mode)
```
Arrow Keys      Navigate links and scroll
Enter           Follow selected link
← (Left)        Go back
→ (Right)       Follow link
g               Go to URL (prompt)
q               Quit
/               Search forward
n               Next search result
p               Print to file
d               Download selected link
h               Help
```

### Advanced Options  
```bash
lynx -accept_all_cookies URL           # Accept all cookies
lynx -cookie_file=cookies.txt URL      # Use cookie file
lynx -nolist URL                       # Don't display link list
lynx -width=132 URL                    # Set screen width
lynx -display_charset=utf-8 URL        # Set character encoding
lynx -useragent="Custom Agent" URL     # Custom user agent
```

### Dumping & Scraping 
```bash
# Extract all links from a page
lynx -dump -listonly https://example.com > links.txt

# Get readable text version
lynx -dump -nolist https://example.com > content.txt

# Extract internal and external links separately
lynx -dump -listonly URL | grep "^[[:space:]]*[0-9]"
```

### Configuration
**File**: `/etc/lynx.cfg` or `~/.lynxrc` 
```bash
# Common settings
CHARACTER_SET:utf-8
FORCE_SSL_COOKIES_SECURE:TRUE
ACCEPT_ALL_COOKIES:FALSE
```

***

## links - Enhanced Text Browser with Graphics Support

**Installation**
```bash
dnf install links
```

### Basic Usage 
```bash
links https://example.com              # Text mode
links -g https://example.com           # Graphics mode (X11 required)
links -dump URL                        # Dump to stdout
links -source URL                      # Dump HTML source
```

### Navigation (Interactive) 
```
Arrow Keys      Navigate
Enter           Follow link
Backspace       Go back
ESC             Menu
T               New tab
Ctrl+C          Close tab
Q               Quit (confirm)
/               Search
G               Go to URL
```

### Advanced Options 
```bash
links -driver svgalib https://example.com     # Use SVGAlib driver
links -mode 256 https://example.com           # 256 color mode
links -http-proxy proxy.example.com:8080 URL  # Use HTTP proxy
links -no-connect URL                         # Prevent connections (local only)
```

### Graphics Mode Features 
When compiled with `--enable-graphics`:
- Image rendering (PNG, JPEG, TIFF)
- Better formatting and layout
- Mouse support

**Configuration**: `/etc/links.cfg` or `~/.links/links.cfg` 

***

## Pro Tips for 2025

### Security Best Practices
1. **Never use plain FTP** for sensitive data - use SFTP or FTPS  
2. **Verify SSL certificates** in production; only skip for trusted internal networks
3. **Use SSH keys** instead of passwords for SFTP connections
4. **Audit credentials**: Never hardcode passwords in scripts; use environment variables or key management

### Automation Strategies
```bash
# lftp automation script
#!/bin/bash
lftp -c "
set ssl:verify-certificate no
open ftps://ftp.example.com
user $FTP_USER $FTP_PASS
mirror --reverse --delete --verbose /local/path /remote/path
bye
"

# wget retry and resume
wget --retry-connrefused --waitretry=1 --read-timeout=20 --timeout=15 -t 0 -c URL

# Combine wget with lynx for link extraction
lynx -dump -listonly -nonumbers URL | grep "^http" | wget -i -
```

### Performance Optimization
- **lftp**: Use `pget -n 10` for parallel segment downloads of large files
- **wget**: Use `--limit-rate` to avoid bandwidth throttling or blacklisting 
- **Mirror sites**: Always use `--random-wait` and rate limiting to avoid server bans 

# Linux Software Management (2025 Edition)

## Downloading Software

### Using curl for Downloads
```bash
# Download file with original name
curl -O https://example.com/file.tar.gz

# Download with custom name
curl -o custom-name.tar.gz https://example.com/file.tar.gz

# Resume interrupted download
curl -C - -O https://example.com/largefile.tar.gz

# Follow redirects (critical for many download links)
curl -L -O https://example.com/file.tar.gz

# Download with progress bar
curl -# -O https://example.com/file.tar.gz

# Download multiple files
curl -O https://site.com/file1.tar.gz -O https://site.com/file2.tar.gz

# Download with authentication
curl -u username:password https://example.com/file.tar.gz

# Silent mode with failure detection
curl -sSfL https://example.com/install.sh | bash
```

### Cloning Source Repositories
```bash
# Clone Git repository
git clone https://github.com/user/project.git

# Clone specific branch
git clone -b develop https://github.com/user/project.git

# Shallow clone (faster, single commit history)
git clone --depth 1 https://github.com/user/project.git

# Clone with submodules
git clone --recursive https://github.com/user/project.git
```

### Downloading Source Tarballs 
```bash
# Download and extract in one command
curl -L https://www.kernel.org/pub/linux/kernel/v6.x/linux-6.5.7.tar.xz | tar xJ

# Download kernel source
curl -L -o linux-6.5.7.tar.xz \
  https://www.kernel.org/pub/linux/kernel/v6.x/linux-6.5.7.tar.xz

# Extract compressed archives
tar xzf file.tar.gz        # gzip
tar xjf file.tar.bz2       # bzip2
tar xJf file.tar.xz        # xz (most common for source)
```

***

## Installing Software with DNF (Default for CentOS 9)

### Essential DNF Commands   

**Search and Information**
```bash
dnf search keyword                    # Search for packages
dnf info package_name                 # Detailed package information
dnf list available                    # List all available packages
dnf list installed                    # List installed packages
dnf list updates                      # Show available updates
dnf repoquery --whatprovides /bin/ls  # Find package providing file
```

**Installation and Removal** 
```bash
dnf install package_name              # Install package
dnf install package1 package2         # Install multiple packages
dnf reinstall package_name            # Reinstall package
dnf remove package_name               # Remove package
dnf autoremove                        # Remove unused dependencies
dnf install -y package_name           # Auto-answer yes to prompts
```

**Updates and Upgrades**  
```bash
dnf check-update                      # Check for updates (no install)
dnf update                            # Update all packages
dnf update package_name               # Update specific package
dnf upgrade                           # Upgrade + remove obsolete packages
dnf upgrade --security                # Security updates only
```

**Group Installation** 
```bash
dnf group list                        # List available groups
dnf group info "Development Tools"    # Group details
dnf groupinstall "Development Tools"  # Install entire group
dnf groupremove "Development Tools"   # Remove group
```

**History and Rollback** 
```bash
dnf history                           # Show transaction history
dnf history info 15                   # Details of transaction #15
dnf history undo 15                   # Undo transaction #15
dnf history rollback 10               # Rollback to transaction #10
```

### Repository Management  

**List and Enable Repositories**
```bash
dnf repolist                          # Show enabled repos
dnf repolist --all                    # Show all repos (enabled/disabled)
dnf config-manager --enable repo_id   # Enable repository
dnf config-manager --disable repo_id  # Disable repository
```

**Add Third-Party Repositories**
```bash
# Add repository from URL
dnf config-manager --add-repo https://example.com/repo/rhel/9/x86_64/

# Install EPEL (Extra Packages for Enterprise Linux)
dnf install epel-release

# Install specific repo RPM
dnf install https://example.com/repo-release.rpm
```

**Repository Configuration Files**: `/etc/yum.repos.d/*.repo` 
```ini
[custom-repo]
name=Custom Repository
baseurl=https://example.com/centos/9/$basearch/
enabled=1
gpgcheck=1
gpgkey=https://example.com/RPM-GPG-KEY
```

### Download Packages Without Installing 
```bash
dnf download package_name             # Download RPM to current directory
dnf download --resolve package_name   # Download with dependencies
dnf install --downloadonly package_name  # Download to cache
```

### DNF Caching and Performance
```bash
dnf makecache                         # Update metadata cache
dnf clean all                         # Clean all cached data
dnf clean packages                    # Remove cached packages only
dnf clean metadata                    # Remove metadata only
```

### Advanced DNF Features 

**DNF Automatic (Unattended Updates)**
```bash
dnf install dnf-automatic
systemctl enable --now dnf-automatic.timer

# Configure: /etc/dnf/automatic.conf
[commands]
upgrade_type = security               # Only security updates
download_updates = yes
apply_updates = yes
emit_via = motd                       # Report via message of the day
```

***

## Installing Binary Packages with RPM

### Basic RPM Installation 

**Installation Commands**
```bash
rpm -ivh package.rpm                  # Install with verbose output + hash marks
rpm -Uvh package.rpm                  # Upgrade or install (recommended)
rpm -Fvh package.rpm                  # Freshen (upgrade only if installed)

# Force installation (dangerous - breaks dependencies)
rpm -ivh --force package.rpm
rpm -ivh --nodeps package.rpm         # Ignore dependency checking
```

**Package Options**
```bash
i = install
v = verbose output
h = hash marks (progress indicator)
U = upgrade (or install if not present)
F = freshen (upgrade only if older version installed)
```

### Installing from URLs
```bash
rpm -ivh https://example.com/package.rpm
rpm -Uvh ftp://ftp.example.com/pub/package.rpm
```

### Removal
```bash
rpm -e package_name                   # Erase/remove package
rpm -e --nodeps package_name          # Remove without dependency check
```

***

## Querying and Verifying with RPM

### Package Query Commands 

**Basic Queries**
```bash
rpm -q package_name                   # Check if package is installed
rpm -qa                               # Query all installed packages
rpm -qa | grep keyword                # Search installed packages
rpm -qf /path/to/file                 # Find package owning a file
rpm -qp package.rpm                   # Query uninstalled RPM file
```

**Detailed Information** 
```bash
rpm -qi package_name                  # Detailed package info (installed)
rpm -qip package.rpm                  # Info from RPM file
rpm -ql package_name                  # List all files in package
rpm -qlp package.rpm                  # List files in RPM file
rpm -qd package_name                  # List documentation files
rpm -qc package_name                  # List configuration files
rpm -q --scripts package_name         # Show install/uninstall scripts
rpm -q --changelog package_name       # Show package changelog
```

**Dependency Queries**
```bash
rpm -qR package_name                  # Show package dependencies (requires)
rpm -q --whatrequires package_name    # What packages depend on this
rpm -q --whatprovides /bin/bash       # What package provides this file
rpm -qpR package.rpm                  # Dependencies of uninstalled RPM
```

**Advanced Queries**
```bash
# Find recently installed packages
rpm -qa --last | head -20

# Find packages by size (largest first)
rpm -qa --queryformat '%{SIZE} %{NAME}\n' | sort -rn | head -20

# Custom query format
rpm -qa --queryformat '%{NAME}-%{VERSION}-%{RELEASE}.%{ARCH}\n'

# Find packages installed from specific vendor
rpm -qa --queryformat '%{NAME} %{VENDOR}\n' | grep -i "Red Hat"
```

### Package Verification and Integrity  

**Verify Installed Packages** 
```bash
rpm -V package_name                   # Verify single package
rpm -Va                               # Verify all installed packages
rpm -Vp package.rpm                   # Verify RPM file
```

**Verification Output Codes**
```
S = Size differs
M = Mode differs (permissions)
5 = MD5 checksum differs
D = Device major/minor mismatch
L = Symlink path differs
U = User ownership differs
G = Group ownership differs
T = Modification time differs
P = Capabilities differ
. = Test passed
? = Test could not be performed
```

### GPG Signature Verification  

**Import GPG Keys** 
```bash
# Import GPG key
rpm --import https://example.com/RPM-GPG-KEY

# Import from file
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-9

# List imported keys
rpm -qa gpg-pubkey*
rpm -qi gpg-pubkey-xxxxxxxx
```

**Verify Package Signatures**  
```bash
# Check package signature
rpm -K package.rpm                    # Short output
rpm --checksig package.rpm            # Same as -K

# Verbose signature check
rpm -Kv package.rpm

# Example output:
# package.rpm: digests signatures OK
# If not signed: package.rpm: digests OK
```

**Verify Signature Algorithm** 
```bash
# Check signature details
rpm -qp --qf '%|DSAHEADER?{%{DSAHEADER:pgpsig}}:{%|RSAHEADER?{%{RSAHEADER:pgpsig}}:{(none)}|}|\n' package.rpm

# Modern packages should show RSA/SHA256
```

**Configure GPG Check in DNF** 
```bash
# In /etc/yum.repos.d/*.repo
gpgcheck=1                            # Enable GPG checking
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-9
```

### Database Operations
```bash
rpm --rebuilddb                       # Rebuild RPM database (if corrupted)
rpm --initdb                          # Initialize new database
rpm --dbpath /custom/path -qa         # Query using custom database path
```

***

## Compiling and Installing from Source  

### Prerequisites for Compilation 
```bash
# Install development tools group
dnf groupinstall "Development Tools"

# Essential packages
dnf install gcc gcc-c++ make automake autoconf \
  kernel-devel kernel-headers \
  ncurses-devel openssl-devel \
  bison flex bc git wget
```

### Standard Build Process
```bash
# 1. Download and extract source
curl -L -O https://example.com/software-1.0.tar.gz
tar xzf software-1.0.tar.gz
cd software-1.0

# 2. Read documentation
cat README INSTALL

# 3. Configure build
./configure --prefix=/usr/local      # Standard location
./configure --help                   # See all options

# 4. Compile
make -j$(nproc)                      # Parallel build (use all CPU cores)

# 5. Optional: Run tests
make test
make check

# 6. Install (requires root)
sudo make install

# 7. Optional: Create uninstall script
sudo checkinstall                    # Creates RPM and installs (if available)
```

### Alternative Build Systems
```bash
# CMake-based projects
mkdir build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local
make -j$(nproc)
sudo make install

# Meson-based projects
meson setup build --prefix=/usr/local
cd build
ninja
sudo ninja install

# Python packages
python3 setup.py build
sudo python3 setup.py install
# OR modern approach
pip3 install .
```

### Managing Compiled Software
```bash
# Install to separate prefix for easy removal
./configure --prefix=/opt/software-1.0
sudo make install
sudo ln -s /opt/software-1.0 /opt/software  # Symlink for version management

# Uninstall (if Makefile supports it)
sudo make uninstall

# Or remove directory
sudo rm -rf /opt/software-1.0
```

***

## Modern Universal Package Formats

### Flatpak (Recommended for CentOS 9) 

**Installation and Setup**
```bash
# Install Flatpak
dnf install flatpak

# Add Flathub repository (largest Flatpak repo)
flatpak remote-add --if-not-exists flathub \
  https://flathub.org/repo/flathub.flatpakrepo
```

**Using Flatpak** 
```bash
flatpak search app_name               # Search applications
flatpak install flathub app.id        # Install from Flathub
flatpak list                          # List installed apps
flatpak run app.id                    # Run application
flatpak update                        # Update all Flatpaks
flatpak uninstall app.id              # Remove application
flatpak uninstall --unused            # Remove unused runtimes
```

### Snap Packages 

**Installation** (Not officially supported on CentOS 9, but possible)
```bash
# Enable EPEL
dnf install epel-release

# Install snapd
dnf install snapd
systemctl enable --now snapd.socket
ln -s /var/lib/snapd/snap /snap      # Create symbolic link
```

**Using Snap**
```bash
snap find app_name                    # Search
snap install app_name                 # Install
snap list                             # List installed
snap refresh                          # Update all
snap remove app_name                  # Remove
```

### AppImage 

**Using AppImage** (No installation required) 
```bash
# Download AppImage file
curl -L -O https://example.com/App.AppImage

# Make executable
chmod +x App.AppImage

# Run directly
./App.AppImage

# Optional: Integrate with desktop
./App.AppImage --appimage-extract    # Extract to squashfs-root/
# Then create desktop entry manually
```

***

## Installing Debian Packages on CentOS 9

### Using Alien Tool  

**⚠️ WARNING**: Installing Debian packages on CentOS is **NOT RECOMMENDED** for production systems. Debian and RPM-based systems have different: 
- Library versions and naming conventions
- Filesystem layout standards
- Init system configurations
- Dependency resolution

**Installation** 
```bash
# Download alien source (not in standard repos)
wget -c https://sourceforge.net/projects/alien-pkg-convert/files/release/alien_8.95.tar.xz
tar xf alien_8.95.tar.xz
cd alien

# Install Perl (required)
dnf install perl perl-devel

# Compile and install alien
perl Makefile.PL
make
sudo make install
```

**Converting and Installing DEB Packages**  
```bash
# Convert DEB to RPM
alien --to-rpm package.deb

# This creates package-version.rpm in current directory
# Then install with RPM
rpm -ivh package-version.rpm

# Or install directly (alien converts and installs)
alien -i package.deb

# Convert without scripts (safer)
alien --to-rpm --scripts package.deb
```

**Alternative: Extract and Manual Installation** 
```bash
# Extract DEB package manually
ar x package.deb
tar xzf data.tar.gz

# Then manually copy files to appropriate locations
# (Requires understanding package contents)
```

### Recommended Alternatives 
Instead of using alien:
1. **Find RPM equivalent**: Search CentOS/EPEL repositories first
2. **Build from source**: Compile software directly for your system
3. **Use Flatpak/Snap**: Distribution-agnostic packages
4. **Request RPM**: Contact software vendor for RPM version
5. **Create custom RPM**: Use `rpmbuild` to create proper RPM package

***

## Pro Tips for 2025

### Security Best Practices 

**Always Verify Package Signatures**
```bash
# Enable GPG checking globally
echo "gpgcheck=1" >> /etc/dnf/dnf.conf

# Never use --nogpgcheck in production
dnf install --nogpgcheck package  # DANGEROUS!

# Verify before installing
rpm -K package.rpm && rpm -ivh package.rpm
```

**Audit Installed Packages**
```bash
# Find packages with modified files
rpm -Va | grep '^..5'                 # Files with changed MD5

# Find packages not from official repos
rpm -qa --queryformat '%{NAME} %{VENDOR}\n' | grep -v "CentOS\|Red Hat"
```

### Performance Optimization 

**Speed Up DNF**
```bash
# In /etc/dnf/dnf.conf
max_parallel_downloads=10             # Parallel downloads
fastestmirror=True                    # Use fastest mirror
deltarpm=False                        # Disable delta RPMs (faster on fast connections)
```

### Development Workflow

**Create Local Repository**
```bash
# Install createrepo
dnf install createrepo_c

# Create repo from RPMs
mkdir -p /var/local-repo
cp *.rpm /var/local-repo/
createrepo /var/local-repo

# Configure DNF to use it
cat > /etc/yum.repos.d/local.repo <<EOF
[local-repo]
name=Local Repository
baseurl=file:///var/local-repo
enabled=1
gpgcheck=0
EOF
```

**Test Package Installation in Container**
```bash
# Test RPM installation safely
podman run -it --rm -v $(pwd):/rpms:z centos:stream9
dnf install /rpms/package.rpm
```

### Troubleshooting Common Issues

**Fix Broken Dependencies** 
```bash
dnf repoquery --requires package_name   # Check requirements
dnf deplist package_name                # Show all dependencies
dnf install --skip-broken               # Skip problematic packages
```

**Clean Corrupted Database**
```bash
# If RPM database is corrupted
rpm --rebuilddb
dnf clean all && dnf makecache
```

**Check What Changed After Update**
```bash
# Before update
rpm -qa > packages-before.txt

# After update
rpm -qa > packages-after.txt
diff packages-before.txt packages-after.txt
```
# Linux Source & Advanced Package Management Cheatsheet (CentOS 9 - 2025 Edition)

## Installing Source RPM Packages (SRPM)

### Understanding Source RPMs  

**What is an SRPM?**
Source RPMs (`.src.rpm`) contain:  
- Source code (usually as tarballs)
- SPEC file (build instructions)
- Patches to apply to source code
- Build metadata and dependencies

**Why Use SRPMs?**
- Rebuild packages with custom patches
- Recompile for performance optimizations
- Verify security fixes in source code
- Build for different architectures
- Debug package build issues

### RPM Build Environment Setup 

**Install Build Tools**
```bash
# Install development toolchain
dnf groupinstall "Development Tools"

# Install RPM build utilities
dnf install rpm-build rpm-devel rpmlint rpmdevtools

# Create RPM build tree (as regular user, NOT root)
rpmdev-setuptree
```

**Build Directory Structure** (created in `~/rpmbuild/`) 
```
~/rpmbuild/
├── BUILD/       # Build happens here (temporary)
├── BUILDROOT/   # Installation root during build
├── RPMS/        # Binary RPMs output
│   ├── x86_64/
│   ├── noarch/
├── SOURCES/     # Source tarballs and patches
├── SPECS/       # SPEC files
└── SRPMS/       # Source RPMs output
```

### Downloading Source RPMs  

**From CentOS Repositories** 
```bash
# Enable source repositories
dnf config-manager --set-enabled crb
dnf config-manager --set-enabled baseos-source
dnf config-manager --set-enabled appstream-source

# Download SRPM for installed package
dnf download --source bash

# Or use yumdownloader (if available)
yumdownloader --source bash
```

**From Vault or Mirror**
CentOS SRPM locations: 
- BaseOS: `https://vault.centos.org/centos/9-stream/BaseOS/Source/SPackages/`
- AppStream: `https://vault.centos.org/centos/9-stream/AppStream/Source/SPackages/`

```bash
# Download specific SRPM
wget https://vault.centos.org/centos/9-stream/BaseOS/Source/SPackages/bash-5.1.8-6.el9.src.rpm
```

**Inspect SRPM Contents** 
```bash
# List files inside SRPM
rpm -qpl bash-5.1.8-6.el9.src.rpm

# Show SRPM information
rpm -qpi bash-5.1.8-6.el9.src.rpm

# Extract SRPM without installing
rpm2cpio bash-5.1.8-6.el9.src.rpm | cpio -idmv
```

### Method 1: Quick Rebuild (One-Step)  

**Direct SRPM Rebuild**  
```bash
# Rebuild SRPM in one command (as regular user)
rpmbuild --rebuild bash-5.1.8-6.el9.src.rpm

# This process:
# 1. Installs SRPM contents to ~/rpmbuild/
# 2. Builds binary RPM
# 3. Cleans up source files
# 4. Places binary RPM in ~/rpmbuild/RPMS/x86_64/

# Find built RPM
ls -lh ~/rpmbuild/RPMS/x86_64/
```

**Common Build Options**
```bash
# Define custom macros
rpmbuild --rebuild --define 'debug_package %{nil}' package.src.rpm

# Skip tests during build (faster)
rpmbuild --rebuild --without check package.src.rpm

# Build only specific target
rpmbuild --rebuild --target=x86_64 package.src.rpm
```

### Method 2: Install SRPM and Build from SPEC  

**Installation and Build Process** 
```bash
# 1. Install SRPM (as regular user, extracts to ~/rpmbuild/)
rpm -ivh bash-5.1.8-6.el9.src.rpm
# OR (skip MD5 check if needed)
rpm --nomd5 -i bash-5.1.8-6.el9.src.rpm

# 2. Verify installation
ls ~/rpmbuild/SPECS/       # bash.spec
ls ~/rpmbuild/SOURCES/     # bash-5.1.8.tar.gz + patches

# 3. Build from SPEC file
cd ~/rpmbuild/SPECS
rpmbuild -ba bash.spec

# Build options:
# -ba : Build all (binary + source RPM)
# -bb : Build binary RPM only
# -bs : Build source RPM only
# -bi : Install built files (to BUILDROOT)
# -bl : Check file list
```

**Check Dependencies Before Building** 
```bash
# List build dependencies
rpm -qp --requires bash-5.1.8-6.el9.src.rpm
rpmspec -q --buildrequires ~/rpmbuild/SPECS/bash.spec

# Install build dependencies automatically
dnf builddep ~/rpmbuild/SPECS/bash.spec
# OR from SRPM
dnf builddep bash-5.1.8-6.el9.src.rpm
```

### Method 3: Build Stages for Custom Modifications 

**Step-by-Step Build Process** 
```bash
cd ~/rpmbuild/SPECS

# Stage 1: Prepare source (extract and apply patches)
rpmbuild -bp bash.spec

# Stage 2: Compile source
rpmbuild -bc bash.spec

# Stage 3: Install to BUILDROOT
rpmbuild -bi bash.spec

# Stage 4: Create binary RPM
rpmbuild -bb bash.spec

# Create source RPM
rpmbuild -bs bash.spec
```

**Creating Custom Patches** 
```bash
# 1. Prepare source with patches applied
cd ~/rpmbuild/SPECS
rpmbuild -bp bash.spec

# 2. Create backup of source
cd ~/rpmbuild/BUILD/
cp -r bash-5.1.8 bash-5.1.8.orig

# 3. Make modifications
cd bash-5.1.8
vim some_file.c          # Make your changes

# 4. Generate patch file
cd ~/rpmbuild/BUILD/
diff -Npru bash-5.1.8.orig bash-5.1.8 > my_custom_fix.patch

# 5. Copy patch to SOURCES
cp my_custom_fix.patch ~/rpmbuild/SOURCES/

# 6. Edit SPEC file to include patch
cd ~/rpmbuild/SPECS
vim bash.spec
```

**Add Patch to SPEC File**
```spec
# Add after other patches
Patch100: my_custom_fix.patch

# In %prep section, add:
%patch100 -p1
```

**Rebuild with Custom Patch**
```bash
cd ~/rpmbuild/SPECS
rpmbuild -ba bash.spec
```

### Advanced: Building with Mock  

**Why Use Mock?** 
Mock provides:
- Clean, isolated build environment (chroot)
- No pollution of host system
- Build for different distributions/versions
- Automatic dependency installation
- More reproducible builds

**Setup Mock** 
```bash
# Install mock
dnf install mock

# Add user to mock group
sudo usermod -aG mock $USER

# Re-login or activate group
newgrp mock

# Verify mock configs
ls /etc/mock/
```

**Building with Mock** 
```bash
# Build SRPM in clean environment
mock -r centos-stream-9-x86_64 --rebuild bash-5.1.8-6.el9.src.rpm

# Results stored in:
ls /var/lib/mock/centos-stream-9-x86_64/result/

# Build for different architecture
mock -r centos-stream-9-aarch64 --rebuild bash-5.1.8-6.el9.src.rpm

# Interactive shell inside mock environment
mock -r centos-stream-9-x86_64 --shell

# Clean mock cache
mock -r centos-stream-9-x86_64 --clean
```

**Mock Configuration** 
```bash
# List available configurations
mock --list-configs | grep centos

# Copy and customize config
cp /etc/mock/centos-stream-9-x86_64.cfg ~/.config/mock/my-custom.cfg
vim ~/.config/mock/my-custom.cfg

# Use custom config
mock -r my-custom --rebuild package.src.rpm
```

### Creating Source RPMs from Scratch 

**Build SRPM from SPEC File** 
```bash
# Create SRPM (without building binary)
cd ~/rpmbuild/SPECS
rpmbuild -bs mypackage.spec

# Output in ~/rpmbuild/SRPMS/
ls ~/rpmbuild/SRPMS/
```

**Upload Source and Build**
```bash
# Place source tarball in SOURCES
cp myapp-1.0.tar.gz ~/rpmbuild/SOURCES/

# Create/edit SPEC file
vim ~/rpmbuild/SPECS/myapp.spec

# Build SRPM
rpmbuild -bs ~/rpmbuild/SPECS/myapp.spec

# Then rebuild
rpmbuild --rebuild ~/rpmbuild/SRPMS/myapp-1.0-1.el9.src.rpm
```

### Creating Local Repository from Built RPMs 

**Setup Local Repository** 
```bash
# Install createrepo
dnf install createrepo_c

# Create repository directory
sudo mkdir -p /var/local-repo

# Copy built RPMs
sudo cp ~/rpmbuild/RPMS/x86_64/*.rpm /var/local-repo/
# Or from mock
sudo cp /var/lib/mock/centos-stream-9-x86_64/result/*.rpm /var/local-repo/

# Initialize repository
sudo createrepo /var/local-repo

# Update repository (after adding new RPMs)
sudo createrepo --update /var/local-repo
```

**Configure DNF to Use Local Repo**
```bash
# Create repo file
sudo tee /etc/yum.repos.d/local.repo <<EOF
[local-repo]
name=Local Custom Repository
baseurl=file:///var/local-repo
enabled=1
gpgcheck=0
priority=1
EOF

# Refresh cache
dnf clean all && dnf makecache

# Install from local repo
dnf install your-custom-package
```

### Troubleshooting SRPM Builds 

**Common Issues and Solutions**

**Missing Build Dependencies**
```bash
# Error: BuildRequires: package-devel not found
# Solution:
dnf builddep ~/rpmbuild/SPECS/package.spec
# Or manually install missing packages
dnf install package-devel
```

**Build Fails During %check**
```bash
# Skip tests/checks
rpmbuild -ba --nocheck package.spec
# OR
rpmbuild -ba --without check package.spec
```

**Debuginfo Build Errors** 
```bash
# Disable debuginfo package generation
rpmbuild -ba --define 'debug_package %{nil}' package.spec
```

**Macro Errors**
```bash
# List all available macros
rpm --showrc | grep -i macro_name

# Evaluate macro
rpm --eval "%{_bindir}"
rpm --eval "%{_libdir}"

# Define custom macro
rpmbuild -ba --define 'custom_flag 1' package.spec
```

**Check SPEC File Syntax**
```bash
# Lint SPEC file
rpmlint ~/rpmbuild/SPECS/package.spec

# Parse and check
rpmspec -P ~/rpmbuild/SPECS/package.spec
```

***

## Lab: FTP, NcFTP, and wget Practice

### Lab Prerequisites

**Install Required Tools**
```bash
# Install all tools for lab
dnf install ftp ncftp lftp wget curl

# Verify installations
ftp --version
ncftp -v
lftp --version
wget --version
```

### Lab Exercise 1: Anonymous FTP Access

**Objective**: Connect to public FTP servers and download test files

**Public Test FTP Servers**   
- **Tele2 Speedtest**: `ftp://speedtest.tele2.net` (anonymous login)  
- **DLP Test**: `ftp.dlptest.com` (user: dlpuser, pass: rNrKYTX9g7z3RgJRmxWuGHbeu) 

**Task 1.1: Traditional FTP Client** 
```bash
# Connect to Tele2 FTP
ftp speedtest.tele2.net

# When prompted:
# Name: anonymous
# Password: (press Enter or use your email)

# FTP commands to practice:
ftp> ls                    # List files
ftp> cd upload             # Try changing directory
ftp> ls -la                # Detailed listing
ftp> binary                # Set binary mode (for non-text files)
ftp> get 1KB.zip           # Download 1KB test file
ftp> mget *.zip            # Download multiple files (with prompt)
ftp> prompt off            # Disable prompts
ftp> mget *.zip            # Download multiple without prompts
ftp> hash                  # Enable progress indicators
ftp> get 10MB.zip          # Download larger file with progress
ftp> bye                   # Disconnect

# Verify downloads
ls -lh *.zip
```

**Task 1.2: Using NcFTP** 
```bash
# Method 1: Interactive
ncftp speedtest.tele2.net
# Login as anonymous (automatic)

ncftp> ls
ncftp> get 1MB.zip
ncftp> mget 1*.zip         # Download files matching pattern
ncftp> bye

# Method 2: Command-line download (one-liner)
ncftpget -v speedtest.tele2.net . /1KB.zip

# Method 3: Recursive download
ncftpget -R -v speedtest.tele2.net . /upload

# Method 4: Background download job
ncftpget -bb speedtest.tele2.net . /10MB.zip
ncftpbatch -l                    # List queued jobs
ncftpbatch -d                    # Process jobs in background
```

**Task 1.3: Using lftp (Recommended)** 
```bash
# Method 1: Interactive session
lftp ftp://speedtest.tele2.net

lftp> ls
lftp> cd upload
lftp> get 1KB.zip
lftp> mget *.zip           # Download multiple
lftp> pget -n 4 10MB.zip   # Parallel download (4 connections)
lftp> mirror /              # Mirror entire directory
lftp> bye

# Method 2: One-liner command
lftp -c 'open ftp://speedtest.tele2.net; mget 1*.zip; bye'

# Method 3: Parallel download with progress
lftp -c 'pget -n 8 ftp://speedtest.tele2.net/10MB.zip'

# Method 4: Mirror with options
lftp -c 'open ftp://speedtest.tele2.net; mirror --verbose --parallel=2 / ./local-mirror; bye'
```

### Lab Exercise 2: Authenticated FTP 

**Objective**: Practice with credentials-based FTP access

**Using DLP Test Server** 
```bash
# Server: ftp.dlptest.com
# User: dlpuser
# Password: rNrKYTX9g7z3RgJRmxWuGHbeu

# Create test file for upload
echo "Test upload at $(date)" > test-upload.txt

# Task 2.1: lftp with credentials
lftp -u dlpuser,rNrKYTX9g7z3RgJRmxWuGHbeu ftp.dlptest.com

lftp> ls
lftp> put test-upload.txt
lftp> ls -l test-upload.txt
lftp> get test-upload.txt -o downloaded-test.txt
lftp> rm test-upload.txt   # Clean up
lftp> bye

# Task 2.2: NcFTP with credentials
ncftp -u dlpuser -p rNrKYTX9g7z3RgJRmxWuGHbeu ftp.dlptest.com
ncftp> put test-upload.txt
ncftp> ls
ncftp> bye

# Task 2.3: Upload using ncftpput
ncftpput -u dlpuser -p rNrKYTX9g7z3RgJRmxWuGHbeu \
  ftp.dlptest.com / test-upload.txt

# Task 2.4: Scripted upload/download
lftp -c "
  open -u dlpuser,rNrKYTX9g7z3RgJRmxWuGHbeu ftp.dlptest.com
  put test-upload.txt
  ls -l test-upload.txt
  bye
"
```

### Lab Exercise 3: wget Advanced Download Scenarios

**Objective**: Master wget for various download tasks

**Task 3.1: Basic Downloads**  
```bash
# Download single file
wget http://speedtest.tele2.net/1MB.zip

# Download with custom name
wget -O tele2-1mb.zip http://speedtest.tele2.net/1MB.zip

# Resume interrupted download
wget -c http://speedtest.tele2.net/100MB.zip
# Press Ctrl+C to interrupt, then run again to resume

# Download to specific directory
wget -P ~/downloads/ http://speedtest.tele2.net/1MB.zip
```

**Task 3.2: Multiple Downloads** 
```bash
# Create list of URLs
cat > download-list.txt <<EOF
http://speedtest.tele2.net/1KB.zip
http://speedtest.tele2.net/100KB.zip
http://speedtest.tele2.net/1MB.zip
EOF

# Download all from list
wget -i download-list.txt

# Download with rate limiting
wget --limit-rate=200k http://speedtest.tele2.net/10MB.zip

# Download with retry and timeout
wget --tries=10 --timeout=30 http://speedtest.tele2.net/10MB.zip
```

**Task 3.3: FTP Downloads with wget** 
```bash
# Download from FTP using wget
wget ftp://speedtest.tele2.net/1MB.zip

# FTP with authentication (if needed)
wget --ftp-user=username --ftp-password=password ftp://server.com/file.zip

# Recursive FTP download
wget -r ftp://speedtest.tele2.net/upload/

# Mirror FTP directory
wget -m ftp://speedtest.tele2.net/upload/
```

**Task 3.4: Website Mirroring**
```bash
# Mirror small website (use responsibly!)
wget --mirror --convert-links --page-requisites --no-parent \
  --wait=1 --limit-rate=200k \
  http://example.com/documentation/

# Explanation:
# --mirror: Infinite depth, timestamping
# --convert-links: Make links work offline
# --page-requisites: Download CSS, images, JS
# --no-parent: Don't go to parent directory
# --wait=1: Wait 1 second between requests (be polite!)
# --limit-rate: Don't overwhelm the server
```

**Task 3.5: Background Downloads** 
```bash
# Download in background
wget -b http://speedtest.tele2.net/100MB.zip

# Check progress
tail -f wget-log

# Multiple background downloads with logging
wget -b -o download1.log http://speedtest.tele2.net/10MB.zip
wget -b -o download2.log http://speedtest.tele2.net/100MB.zip

# Monitor both
tail -f download*.log
```

### Lab Exercise 4: Combining Tools

**Objective**: Create practical automation scripts

**Task 4.1: Download and Verify Script**
```bash
#!/bin/bash
# download-and-verify.sh

URL="ftp://speedtest.tele2.net/1MB.zip"
FILE="1MB.zip"
MD5_EXPECTED="example_md5_hash"

# Download with wget
echo "Downloading ${FILE}..."
wget -O "${FILE}" "${URL}"

if [ $? -eq 0 ]; then
    echo "Download successful"
    
    # Verify file exists and has size
    if [ -s "${FILE}" ]; then
        SIZE=$(stat -f%z "${FILE}" 2>/dev/null || stat -c%s "${FILE}")
        echo "File size: ${SIZE} bytes"
    fi
else
    echo "Download failed"
    exit 1
fi
```

**Task 4.2: Mirror FTP Directory Script**
```bash
#!/bin/bash
# ftp-mirror.sh

FTP_HOST="speedtest.tele2.net"
REMOTE_DIR="/upload"
LOCAL_DIR="./ftp-backup"
LOG_FILE="ftp-mirror-$(date +%Y%m%d-%H%M%S).log"

echo "Starting FTP mirror at $(date)" | tee -a "${LOG_FILE}"

lftp -c "
  open ${FTP_HOST}
  mirror --verbose --delete --parallel=2 ${REMOTE_DIR} ${LOCAL_DIR}
  bye
" 2>&1 | tee -a "${LOG_FILE}"

echo "Mirror completed at $(date)" | tee -a "${LOG_FILE}"
```

**Task 4.3: Batch Download with Progress Report**
```bash
#!/bin/bash
# batch-download.sh

declare -a FILES=(
    "ftp://speedtest.tele2.net/1KB.zip"
    "ftp://speedtest.tele2.net/100KB.zip"
    "ftp://speedtest.tele2.net/1MB.zip"
)

TOTAL=${#FILES[@]}
CURRENT=0

for url in "${FILES[@]}"; do
    ((CURRENT++))
    filename=$(basename "${url}")
    echo "[$CURRENT/$TOTAL] Downloading: ${filename}"
    
    wget -q --show-progress "${url}"
    
    if [ $? -eq 0 ]; then
        echo "✓ ${filename} - Success"
    else
        echo "✗ ${filename} - Failed"
    fi
done

echo "Download batch complete: $CURRENT/$TOTAL files"
```

### Lab Exercise 5: Performance Comparison

**Objective**: Compare tool performance and features

**Task 5.1: Speed Test**  
```bash
#!/bin/bash
# speed-comparison.sh

TEST_URL="ftp://speedtest.tele2.net/10MB.zip"

echo "=== Speed Comparison Test ==="

# Test 1: wget
echo -e "\n1. Testing wget..."
rm -f 10MB.zip
time wget -q "${TEST_URL}"

# Test 2: curl
echo -e "\n2. Testing curl..."
rm -f 10MB.zip
time curl -s -O "${TEST_URL}"

# Test 3: lftp (single connection)
echo -e "\n3. Testing lftp (single)..."
rm -f 10MB.zip
time lftp -c "get ${TEST_URL}"

# Test 4: lftp (parallel)
echo -e "\n4. Testing lftp (parallel 4 segments)..."
rm -f 10MB.zip
time lftp -c "pget -n 4 ${TEST_URL}"

echo -e "\n=== Test Complete ==="
```

**Task 5.2: Feature Matrix Exercise**
Create a comparison table of what you've learned:

| Feature | ftp | ncftp | lftp | wget |
|---------|-----|-------|------|------|
| Interactive | ✓ | ✓ | ✓ | ✗ |
| Resume Downloads | ✗ | ✓ | ✓ | ✓ |
| Parallel Segments | ✗ | ✗ | ✓ | ✗ |
| Background Jobs | ✗ | ✓ | ✓ | ✓ |
| Recursive Download | ✗ | ✓ | ✓ | ✓ |
| Scripting-Friendly | ✗ | ✓ | ✓ | ✓ |
| HTTP Support | ✗ | ✗ | ✓ | ✓ |

### Lab Cleanup

**Remove Downloaded Files**
```bash
# Clean up lab files
rm -f *.zip test-upload.txt download-list.txt
rm -f wget-log download*.log ftp-mirror*.log

# Remove practice scripts
rm -f download-and-verify.sh ftp-mirror.sh batch-download.sh speed-comparison.sh
```

***

## Pro Tips for Source RPMs and Labs

### SRPM Best Practices  

**Always Work as Regular User**
```bash
# NEVER build RPMs as root - security risk
# Use mock or regular user account
```

**Version Control Your SPEC Files**
```bash
# Track spec file changes
cd ~/rpmbuild/SPECS
git init
git add *.spec
git commit -m "Initial SPEC file"
```

**Document Build Requirements**
```bash
# Create build documentation
cat > ~/rpmbuild/BUILD-NOTES.md <<EOF
# Build Requirements
- CentOS Stream 9
- Development Tools group
- Special flags: --without check

# Build command
rpmbuild -ba --without check mypackage.spec
EOF
```

### Lab Safety Tips  

**Use Test Files Only**
- Never upload sensitive data to public FTP servers 
- DLP Test server deletes files after 10 minutes 
- Public anonymous FTP is read-only 

**Bandwidth Courtesy** 
- Use `--limit-rate` when downloading large files
- Add `--wait` between recursive requests
- Don't hammer servers with parallel downloads

**Security Awareness** 
- FTP transmits passwords in plaintext
- Only use for testing and non-sensitive data
- Prefer SFTP/FTPS in production environments
