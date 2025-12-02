# SSH Mastery Cheatsheet (2025 Edition)

## Secure Shell (SSH) Fundamentals

### Modern SSH Configuration (`/etc/ssh/sshd_config`)

```bash
# Backup first - ALWAYS
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Test configuration syntax before applying
sudo sshd -t

# Apply changes
sudo systemctl restart sshd
```

### 2025 Security Hardening Essentials  

```bash
# /etc/ssh/sshd_config - Production-ready configuration

# Protocol & Port
Protocol 2                          # ONLY SSH-2 (v1 is obsolete/insecure)
Port 2222                          # Change from default 22 to reduce bot attacks
AddressFamily inet                 # IPv4 only (use 'any' for IPv4+IPv6)

# Authentication
PermitRootLogin no                 # NEVER allow direct root login
PasswordAuthentication no          # Key-only auth (disable after keys configured)
PubkeyAuthentication yes           
ChallengeResponseAuthentication no # Unless using 2FA
PermitEmptyPasswords no            # Obvious security risk
MaxAuthTries 3                     # Limit brute-force attempts
LoginGraceTime 30                  # 30 seconds to complete login

# User/Group Restrictions
AllowUsers alice bob               # Whitelist specific users
AllowGroups sshusers devops        # Or whitelist groups
DenyUsers guest testuser           # Blacklist users (processed before Allow)

# Session Management
ClientAliveInterval 300            # Send keepalive every 5 min
ClientAliveCountMax 0              # Disconnect after first timeout
MaxSessions 2                      # Limit concurrent sessions per connection
MaxStartups 10:30:60              # Connection rate limiting

# Cryptographic Hardening (2025 Standards) [web:3]
Ciphers [email protected],[email protected],aes256-ctr
MACs [email protected],[email protected]
KexAlgorithms curve25519-sha256,[email protected]
HostKeyAlgorithms ssh-ed25519,[email protected]

# Logging & Monitoring
LogLevel VERBOSE                   # Detailed logs for security auditing
SyslogFacility AUTHPRIV           # Separate auth logs

# Features
X11Forwarding no                   # Disable unless GUI forwarding needed
PermitTunnel no                    # Disable unless VPN/tunneling required
AllowAgentForwarding yes           # Enable for ssh-agent (be cautious)
AllowTcpForwarding no             # Disable unless port forwarding needed
PrintMotd no                       # Message of the day (handled by PAM)
Banner /etc/ssh/banner.txt         # Legal/warning message
```

### IP-Based Access Control  

```bash
# Method 1: sshd_config Match directive
Match Address 203.0.113.0/24
    AllowUsers adminuser
Match Address 192.168.1.100
    PasswordAuthentication yes     # Exception for specific IP

# Method 2: Firewall rules (PREFERRED for CentOS 9)
sudo firewall-cmd --permanent --remove-service=ssh  # Remove default
sudo firewall-cmd --permanent --add-port=2222/tcp
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="2222" accept'
sudo firewall-cmd --reload

# Method 3: TCP Wrappers (/etc/hosts.allow & /etc/hosts.deny)
# /etc/hosts.allow
sshd: 192.168.1.0/24 203.0.113.10

# /etc/hosts.deny
sshd: ALL
```

### SELinux Context for Custom SSH Port 

```bash
# Required when changing SSH port on CentOS 9
sudo semanage port -a -t ssh_port_t -p tcp 2222
sudo semanage port -l | grep ssh
```

***

## Accessing Remote Shells

### Basic Connection Patterns

```bash
# Standard connection
ssh username@hostname

# Custom port
ssh -p 2222 username@hostname

# Verbose debugging (troubleshooting) [web:2]
ssh -v username@hostname       # Basic verbosity
ssh -vv username@hostname      # More detail
ssh -vvv username@hostname     # Maximum verbosity (shows every step)

# Execute single command
ssh user@host 'df -h'
ssh user@host 'uptime && who'

# Interactive shell with command
ssh -t user@host 'sudo tail -f /var/log/messages'  # -t forces PTY allocation
```

### Advanced Connection Techniques

```bash
# Compression for slow connections [web:2]
ssh -C user@host               # Enable compression
ssh -C -c aes128-ctr user@host # Compression + faster cipher

# Background SSH and execute remote command
ssh -f user@host 'tail -f /var/log/app.log'  # -f backgrounds ssh

# Connection multiplexing (reuse connections - FAST) [web:10]
# ~/.ssh/config
Host *
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h-%p
    ControlPersist 10m

mkdir -p ~/.ssh/sockets
# First connection establishes socket, subsequent connections reuse it

# SSH Jump Host (Bastion/Proxy) [web:3]
ssh -J jump-host target-host
ssh -J user1@jump:port user2@target  # Custom port on jump host

# Multi-hop jump
ssh -J jump1,jump2,jump3 final-destination

# ProxyJump in config
Host target
    HostName 10.0.1.50
    ProxyJump jump-host
```

### SSH Client Configuration (`~/.ssh/config`)

```bash
# Per-user SSH client settings
# ~/.ssh/config (chmod 600)

Host myserver
    HostName 192.168.1.50
    User alice
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_work
    
Host prod-*
    User deploy
    IdentityFile ~/.ssh/id_ed25519_prod
    StrictHostKeyChecking yes
    UserKnownHostsFile ~/.ssh/known_hosts_prod

Host dev
    HostName dev.example.com
    User devuser
    ForwardAgent yes           # Forward ssh-agent (USE CAREFULLY)
    LocalForward 8080 localhost:80  # Local port forwarding
    RemoteForward 9000 localhost:3000  # Remote port forwarding
    
Host bastion
    HostName bastion.company.com
    User adminuser
    IdentityFile ~/.ssh/id_ed25519_bastion
    ServerAliveInterval 60     # Client-side keepalive
    ServerAliveCountMax 3
    
# Wildcard patterns
Host *.internal.company.com
    User sysadmin
    IdentityFile ~/.ssh/id_rsa_internal
    ProxyJump bastion
```

### Port Forwarding & Tunneling  

```bash
# Local Port Forwarding (access remote service locally)
ssh -L 8080:localhost:80 user@remote
# Now: localhost:8080 -> remote:80

ssh -L 5432:db-server:5432 user@jump-host
# Access db-server:5432 via localhost:5432 through jump-host

# Remote Port Forwarding (expose local service remotely)
ssh -R 9000:localhost:3000 user@remote
# remote:9000 -> your-machine:3000

# Dynamic Port Forwarding (SOCKS proxy)
ssh -D 1080 user@remote
# Configure browser to use SOCKS5 proxy localhost:1080
# All browser traffic routes through remote server

# Persistent tunnel (auto-reconnect)
autossh -M 0 -N -f -L 8080:localhost:80 user@remote
# -M 0: disable monitoring port
# -N: no command execution
# -f: background process
```

### X11 Forwarding (GUI Applications) 

```bash
# Enable on server: /etc/ssh/sshd_config
X11Forwarding yes
X11UseLocalhost yes

# Connect with X11 forwarding
ssh -X user@remote              # X11 forwarding
ssh -Y user@remote              # Trusted X11 (less secure checks)

# Launch GUI app
gedit &
firefox &
xterm &
```

***

## Transferring Files

### SCP (Secure Copy) - Traditional

```bash
# Local to remote
scp file.txt user@remote:/path/to/destination/
scp -P 2222 file.txt user@remote:/tmp/  # Custom port (-P uppercase)

# Remote to local
scp user@remote:/path/to/file.txt /local/path/
scp user@remote:/var/log/app.log .

# Recursive directory copy
scp -r /local/dir/ user@remote:/remote/dir/

# Preserve timestamps and permissions
scp -p file.txt user@remote:/tmp/

# Limit bandwidth (KB/s)
scp -l 1024 largefile.iso user@remote:/tmp/  # 1 MB/s limit

# Through jump host
scp -o "ProxyJump jump-host" file.txt user@target:/tmp/

# Copy between two remote hosts (through local)
scp -3 user1@host1:/file user2@host2:/destination  # -3 flag
```

### SFTP (SSH File Transfer Protocol) - Interactive

```bash
# Connect to SFTP server
sftp user@hostname
sftp -P 2222 user@hostname     # Custom port

# SFTP Commands (interactive session)
sftp> pwd                      # Remote working directory
sftp> lpwd                     # Local working directory
sftp> ls                       # List remote files
sftp> lls                      # List local files
sftp> cd /remote/path          # Change remote directory
sftp> lcd /local/path          # Change local directory

# Upload files
sftp> put localfile.txt        # Upload single file
sftp> put -r localdir/         # Upload directory recursively
sftp> mput *.txt               # Upload multiple files (wildcard)

# Download files
sftp> get remotefile.txt       # Download single file
sftp> get -r remotedir/        # Download directory recursively
sftp> mget *.log               # Download multiple files

# File operations
sftp> rm file.txt              # Delete remote file
sftp> rmdir emptydir/          # Delete empty remote directory
sftp> mkdir newdir             # Create remote directory
sftp> rename old.txt new.txt   # Rename remote file
sftp> chmod 644 file.txt       # Change remote permissions
sftp> chown 1000 file.txt      # Change remote ownership

# Progress and resume
sftp> reget largefile.iso      # Resume interrupted download
sftp> reput largefile.iso      # Resume interrupted upload

sftp> exit                     # Close session
```

### SFTP Batch Mode (Scripting)

```bash
# Execute SFTP commands from file
cat << 'EOF' > sftp_commands.txt
cd /var/www/html
lcd /local/website
put -r *
chmod 755 index.html
bye
EOF

sftp -b sftp_commands.txt user@remote

# One-liner SFTP
echo "get /remote/file.txt" | sftp user@remote

# SFTP with SSH key
sftp -i ~/.ssh/id_ed25519 -P 2222 user@remote
```

### Rsync over SSH (SUPERIOR for large transfers) 

```bash
# Basic rsync syntax
rsync -avz source/ user@remote:/destination/
# -a: archive mode (preserves permissions, timestamps, symlinks)
# -v: verbose
# -z: compression

# Rsync with progress and partial resume
rsync -avzP localdir/ user@remote:/remotedir/
# -P: combines --progress and --partial (resume)

# Custom SSH port
rsync -avz -e "ssh -p 2222" source/ user@remote:/dest/

# Delete extraneous files on destination (mirror)
rsync -avz --delete source/ user@remote:/dest/

# Dry run (test before actual transfer)
rsync -avzn source/ user@remote:/dest/  # -n or --dry-run

# Exclude patterns
rsync -avz --exclude='*.tmp' --exclude='cache/' source/ user@remote:/dest/

# Bandwidth limit
rsync -avz --bwlimit=1024 source/ user@remote:/dest/  # 1 MB/s

# Incremental backup strategy
rsync -avz --link-dest=/backup/previous /data/ user@remote:/backup/current/

# Rsync from remote to local
rsync -avz user@remote:/remote/data/ /local/backup/

# Advanced: SSH with specific key and options
rsync -avz -e "ssh -i ~/.ssh/backup_key -o StrictHostKeyChecking=yes" \
    source/ user@remote:/dest/
```

***

## Alternative SFTP Clients

### Command-Line Tools

```bash
# lftp (Advanced FTP/SFTP client with scripting)
sudo dnf install lftp

lftp sftp://user@hostname:2222
lftp user@hostname:~> mirror -R localdir remotedir  # Upload mirror
lftp user@hostname:~> mirror remotedir localdir     # Download mirror
lftp user@hostname:~> pget -n 4 largefile.iso       # Parallel download (4 connections)

# ncftp (Not for SFTP, but excellent for FTP)
# Note: Use sftp/rsync for secure transfers

# sshfs (Mount remote filesystem via SSH)
sudo dnf install fuse-sshfs

mkdir ~/remote-mount
sshfs user@remote:/remote/path ~/remote-mount
sshfs user@remote:/remote/path ~/remote-mount -p 2222 -o IdentityFile=~/.ssh/id_ed25519

# Access like local filesystem
ls ~/remote-mount
cp file.txt ~/remote-mount/

# Unmount
fusermount -u ~/remote-mount
```

### GUI SFTP Clients (mentioned for completeness)

```bash
# FileZilla (cross-platform)
sudo dnf install filezilla

# WinSCP (Windows - mentioned for reference)
# Cyberduck (macOS/Windows)
# Transmit (macOS)

# For CentOS 9 desktop environments:
# - GNOME Files (Nautilus): sftp://user@hostname:port/path
# - Dolphin (KDE): sftp://user@hostname:port/path
```

### Programmatic SFTP (Python example)

```python
# Using paramiko library
from paramiko import SSHClient, AutoAddPolicy

ssh = SSHClient()
ssh.set_missing_host_key_policy(AutoAddPolicy())
ssh.connect('hostname', username='user', key_filename='/home/user/.ssh/id_ed25519')

sftp = ssh.open_sftp()
sftp.put('/local/file.txt', '/remote/file.txt')
sftp.get('/remote/data.log', '/local/data.log')
sftp.close()
ssh.close()
```

***

## SSH Key Management

### Key Generation (2025 Best Practices)  

```bash
# Ed25519 (RECOMMENDED - modern, fast, secure) [web:3]
ssh-keygen -t ed25519 -a 100 -C "user@workstation-2025"
# -a 100: KDF rounds (key derivation function)
# -C: comment for identification

# RSA-4096 (legacy compatibility)
ssh-keygen -t rsa -b 4096 -C "user@workstation-2025"

# ECDSA (not recommended - potential NSA backdoor concerns)
ssh-keygen -t ecdsa -b 521

# Key with custom filename
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_github -C "github-deploy-key"

# Key with passphrase (HIGHLY RECOMMENDED for security)
ssh-keygen -t ed25519 -a 100 -C "prod-server"
# Enter strong passphrase when prompted

# Generate without passphrase (automation only - RISK)
ssh-keygen -t ed25519 -N "" -f ~/.ssh/automation_key
```

### Key Deployment

```bash
# Copy public key to remote server (easiest method)
ssh-copy-id user@remote
ssh-copy-id -i ~/.ssh/id_ed25519_work.pub user@remote
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 user@remote

# Manual method (when ssh-copy-id unavailable)
cat ~/.ssh/id_ed25519.pub | ssh user@remote "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# Multiple keys to same server
ssh-copy-id -i ~/.ssh/id_ed25519_backup.pub user@remote  # Appends to authorized_keys

# Verify authorized_keys on remote
ssh user@remote "cat ~/.ssh/authorized_keys"
```

### Key Lifecycle Management 

```bash
# List all keys
ls -la ~/.ssh/
ssh-add -l                     # List keys in ssh-agent

# Key fingerprints (verification)
ssh-keygen -lf ~/.ssh/id_ed25519.pub        # SHA256 fingerprint (default)
ssh-keygen -lf ~/.ssh/id_ed25519.pub -E md5 # MD5 fingerprint

# Change key passphrase
ssh-keygen -p -f ~/.ssh/id_ed25519

# Remove old passphrase and add new
ssh-keygen -p -P "old-pass" -N "new-pass" -f ~/.ssh/id_ed25519

# Key rotation (CRITICAL security practice) [web:6]
# 1. Generate new key
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519_new -C "rotated-$(date +%Y%m%d)"

# 2. Deploy new key
ssh-copy-id -i ~/.ssh/id_ed25519_new.pub user@remote

# 3. Test new key
ssh -i ~/.ssh/id_ed25519_new user@remote "whoami"

# 4. Update ~/.ssh/config to use new key
# 5. Remove old key from remote
ssh user@remote "sed -i '/old-key-identifier/d' ~/.ssh/authorized_keys"

# 6. Archive old key (don't delete immediately)
mv ~/.ssh/id_ed25519 ~/.ssh/archive/id_ed25519_old_$(date +%Y%m%d)
```

### Enterprise Key Management  

```bash
# Unique keys per system (BEST PRACTICE) [web:6]
# Never reuse keys across multiple servers
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_prod_db -C "prod-database"
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_prod_web -C "prod-webserver"
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_dev -C "dev-environment"

# Key discovery and auditing
find /home -type f -name "authorized_keys" -exec grep -H "" {} \;
find /home -type f -name "id_*" ! -name "*.pub"  # Find private keys

# Certificate-based authentication (advanced) [web:9]
# Replaces static key management with time-limited certificates
# Requires CA setup - beyond basic scope but critical for large orgs

# Enforce key restrictions in authorized_keys
# ~/.ssh/authorized_keys on remote server
from="203.0.113.0/24",no-port-forwarding,no-agent-forwarding,no-X11-forwarding ssh-ed25519 AAAA...
command="/usr/local/bin/backup.sh" ssh-ed25519 AAAA...  # Forced command
```

### Security Best Practices 

```bash
# Proper permissions (CRITICAL)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519       # Private key
chmod 644 ~/.ssh/id_ed25519.pub   # Public key
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/known_hosts

# Protect private keys with Hardware Security Module (HSM) [web:6]
# Use YubiKey, Nitrokey, or other FIDO2 devices
# ssh-keygen -t ed25519-sk -O resident  # Requires OpenSSH 8.2+

# Audit key usage
grep "Accepted publickey" /var/log/secure
journalctl -u sshd | grep "Accepted publickey"

# Find unauthorized keys
diff <(ssh-keyscan hostname) <(cat ~/.ssh/known_hosts | grep hostname)
```

***

## ssh-agent Mastery

### Understanding ssh-agent  

```bash
# Check if ssh-agent is running
ps x | grep ssh-agent
echo $SSH_AUTH_SOCK       # Should show socket path
echo $SSH_AGENT_PID       # Should show process ID

# Start ssh-agent (if not running)
eval $(ssh-agent)
eval $(ssh-agent -s)      # Bourne shell syntax

# Start with specific lifetime
eval $(ssh-agent -t 3600)  # Agent terminates after 1 hour

# Kill running agent
eval $(ssh-agent -k)
ssh-agent -k
kill $SSH_AGENT_PID
```

### Managing Keys in ssh-agent 

```bash
# Add key to agent (with passphrase prompt)
ssh-add ~/.ssh/id_ed25519
ssh-add                    # Adds default keys (~/.ssh/id_*)

# Add key with timeout (key expires from agent)
ssh-add -t 3600 ~/.ssh/id_ed25519_prod  # 1 hour
ssh-add -t 7200 ~/.ssh/id_ed25519_work  # 2 hours

# Add all keys
ssh-add ~/.ssh/id_ed25519_*
ssh-add ~/.ssh/id_rsa_*

# List loaded keys
ssh-add -l                # SHA256 fingerprints
ssh-add -l -E md5         # MD5 fingerprints
ssh-add -L                # Show public keys

# Remove key from agent
ssh-add -d ~/.ssh/id_ed25519
ssh-add -D                # Remove ALL keys

# Lock/unlock agent (requires password)
ssh-add -x                # Lock agent
ssh-add -X                # Unlock agent
```

### Persistent ssh-agent Configuration

```bash
# Auto-start ssh-agent on login
# Add to ~/.bashrc or ~/.bash_profile

# Method 1: Simple auto-start
if [ -z "$SSH_AUTH_SOCK" ]; then
    eval $(ssh-agent -s)
    ssh-add ~/.ssh/id_ed25519
fi

# Method 2: Socket-based (prevents multiple agents)
SSH_ENV="$HOME/.ssh/agent-environment"

function start_agent {
    echo "Initializing new SSH agent..."
    /usr/bin/ssh-agent | sed 's/^echo/#echo/' > "${SSH_ENV}"
    chmod 600 "${SSH_ENV}"
    . "${SSH_ENV}" > /dev/null
    ssh-add ~/.ssh/id_ed25519
}

if [ -f "${SSH_ENV}" ]; then
    . "${SSH_ENV}" > /dev/null
    ps -ef | grep ${SSH_AGENT_PID} | grep ssh-agent$ > /dev/null || start_agent
else
    start_agent
fi

# Method 3: systemd user service (CentOS 9)
# ~/.config/systemd/user/ssh-agent.service
[Unit]
Description=SSH key agent

[Service]
Type=simple
Environment=SSH_AUTH_SOCK=%t/ssh-agent.socket
ExecStart=/usr/bin/ssh-agent -D -a $SSH_AUTH_SOCK

[Install]
WantedBy=default.target

# Enable and start
systemctl --user enable ssh-agent
systemctl --user start ssh-agent

# Add to ~/.bashrc
export SSH_AUTH_SOCK="$XDG_RUNTIME_DIR/ssh-agent.socket"
```

### Agent Forwarding (USE WITH CAUTION)  

```bash
# Enable agent forwarding
ssh -A user@jump-host
# Now can SSH to other hosts from jump-host using local keys

# ~/.ssh/config
Host jump-host
    ForwardAgent yes       # SECURITY RISK if jump-host compromised

# Server-side: /etc/ssh/sshd_config
AllowAgentForwarding yes

# Test agent forwarding
ssh -A user@jump-host
user@jump:~$ echo $SSH_AUTH_SOCK    # Should show socket
user@jump:~$ ssh-add -l             # Should list your local keys
user@jump:~$ ssh user@internal-server  # Uses forwarded agent

# SECURITY WARNING [web:10]
# Agent forwarding allows anyone with root on jump-host to use your keys
# BETTER: Use ProxyJump instead of agent forwarding
ssh -J jump-host internal-server    # No agent forwarding needed
```

### Advanced ssh-agent Patterns

```bash
# Different agents for different environments
ssh-agent -a ~/.ssh/agent.prod     # Production agent
SSH_AUTH_SOCK=~/.ssh/agent.prod ssh-add ~/.ssh/id_ed25519_prod

ssh-agent -a ~/.ssh/agent.dev      # Development agent  
SSH_AUTH_SOCK=~/.ssh/agent.dev ssh-add ~/.ssh/id_ed25519_dev

# Use specific agent
SSH_AUTH_SOCK=~/.ssh/agent.prod ssh user@prod-server

# Confirm before key usage (security measure)
ssh-add -c ~/.ssh/id_ed25519_prod
# Prompts for confirmation each time key is used

# Environment-specific agent wrapper script
#!/bin/bash
# ~/bin/ssh-prod
export SSH_AUTH_SOCK=~/.ssh/agent.prod
ssh "$@"

chmod +x ~/bin/ssh-prod
ssh-prod user@prod-server  # Uses production agent only
```

### Troubleshooting ssh-agent

```bash
# Debug agent issues
ssh -v user@host 2>&1 | grep -i agent
SSH_AUTH_SOCK=/tmp/invalid ssh -v user@host  # Test with invalid socket

# Check agent permissions
ls -la $SSH_AUTH_SOCK
ps aux | grep $SSH_AGENT_PID

# Verify key is loaded and working
ssh-add -l
ssh -vvv -i ~/.ssh/id_ed25519 user@host  # Force specific key

# Common issues
# 1. Could not open a connection to your authentication agent
#    → Agent not running: eval $(ssh-agent)
# 2. Permission denied (publickey)
#    → Key not in agent: ssh-add ~/.ssh/id_ed25519
# 3. Agent has no identities
#    → Add key: ssh-add ~/.ssh/id_ed25519
```

***

## Pro Tips

### Security Auditing

```bash
# Analyze SSH logs for attacks
sudo grep "Failed password" /var/log/secure | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn

# Active SSH sessions monitoring
who
w
last -a
loginctl list-sessions

# Kill suspicious session
sudo pkill -u suspicioususer
```

### Two-Factor Authentication (2FA)  

```bash
# Install Google Authenticator PAM
sudo dnf install google-authenticator

# Setup per user
google-authenticator

# /etc/pam.d/sshd
auth required pam_google_authenticator.so

# /etc/ssh/sshd_config
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
```

### Fail2ban Integration 

```bash
sudo dnf install fail2ban

# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = 2222
maxretry = 3
bantime = 3600
findtime = 600

sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```

# SSH Hands-On Lab Guide (CentOS 9 - 2025 Edition)

## LAB 1: Establish Secure SSH Session to Remote Host

### Lab Environment Setup

```bash
# Lab topology:
# • Local machine (client): your-workstation
# • Remote machine (server): lab-server (192.168.1.100)
# • Alternate server for testing: backup-server (192.168.1.101)

# Prerequisites verification (run on client)
which ssh                           # Verify SSH client installed
rpm -qa | grep openssh-clients      # Should show: openssh-clients-x.x.x

# Check SSH server status (run on remote server)
sudo systemctl status sshd
sudo ss -tlnp | grep :22            # Verify SSH listening on port 22
```

### Exercise 1.1: First Connection - Password Authentication  

```bash
# STEP 1: Test basic network connectivity
ping -c 4 192.168.1.100            # Verify host is reachable
nc -zv 192.168.1.100 22            # Test port 22 connectivity
telnet 192.168.1.100 22            # Alternative port test

# Expected output for nc:
# Connection to 192.168.1.100 22 port [tcp/ssh] succeeded!

# STEP 2: First SSH connection with verbose mode [web:16]
ssh -v student@192.168.1.100

# What you'll see [web:11]:
# 1. "The authenticity of host... can't be established"
# 2. "ECDSA key fingerprint is SHA256:..."
# 3. "Are you sure you want to continue connecting (yes/no)?"

# Type: yes (and press Enter)

# STEP 3: Verify host key was saved
cat ~/.ssh/known_hosts
# Should contain: 192.168.1.100 ecdsa-sha2-nistp256 AAAA...

# STEP 4: Successful login verification
whoami                              # Should show: student
hostname                            # Should show remote hostname
w                                   # Shows who is logged in
exit                                # Return to local machine
```

### Exercise 1.2: Connection Troubleshooting Scenarios  

```bash
# SCENARIO A: "Connection refused" error [web:19]
ssh student@192.168.1.100
# Error: ssh: connect to host 192.168.1.100 port 22: Connection refused

# Diagnostic steps:
sudo systemctl status sshd          # Check if SSH service running
sudo systemctl start sshd           # Start if stopped
sudo systemctl enable sshd          # Enable on boot

# Verify listening ports [web:19]
sudo ss -tlnp | grep sshd
sudo lsof -i -n -P | grep LISTEN | grep ssh

# Check firewall
sudo firewall-cmd --list-all
sudo firewall-cmd --add-service=ssh --permanent
sudo firewall-cmd --reload

# SCENARIO B: "Permission denied" - wrong credentials [web:16]
ssh -v wronguser@192.168.1.100

# Debug output analysis:
# Look for: "debug1: Next authentication method: password"
# Means: Server rejected key auth, trying password

# SCENARIO C: "Broken pipe" during connection [web:16]
ssh -vvv student@192.168.1.100

# Diagnostic steps on server:
sudo tail -f /var/log/secure        # Watch authentication logs
sudo grep "sshd" /var/log/messages

# Check server config:
sudo grep -i "PasswordAuthentication" /etc/ssh/sshd_config
# Should show: PasswordAuthentication yes

# If set to "no", enable it:
sudo sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# SCENARIO D: Connection timeout
ssh -o ConnectTimeout=10 student@192.168.1.100

# Verify no blocking:
cat /etc/hosts.deny                 # Should NOT have: sshd: ALL
cat /etc/hosts.allow                # Check whitelist rules
```

### Exercise 1.3: Advanced Connection Methods

```bash
# Custom port connection
ssh -p 2222 student@192.168.1.100

# Execute remote command without login
ssh student@192.168.1.100 'uptime'
ssh student@192.168.1.100 'df -h | grep -v tmpfs'

# Validation: Store output locally
ssh student@192.168.1.100 'free -m' > remote_memory.txt
cat remote_memory.txt

# Interactive remote command requiring TTY
ssh -t student@192.168.1.100 'sudo systemctl status sshd'
# -t: Force pseudo-terminal allocation (required for sudo)

# Connection with compression (slow network simulation)
ssh -C student@192.168.1.100

# Test difference:
time ssh student@192.168.1.100 'cat /var/log/messages'
time ssh -C student@192.168.1.100 'cat /var/log/messages'
```

### Exercise 1.4: Multi-Host Jump Connection

```bash
# Direct connection blocked, must go through bastion
# Topology: workstation → bastion (192.168.1.50) → target (10.0.1.100)

# Single jump host
ssh -J student@192.168.1.50 admin@10.0.1.100

# Verification inside target:
echo $SSH_CLIENT                    # Shows bastion IP, not workstation
who                                 # Shows your connection

# Multi-hop (3 servers deep)
ssh -J student@jump1,admin@jump2 root@final-target

# Create persistent config:
cat >> ~/.ssh/config << 'EOF'
Host target
    HostName 10.0.1.100
    User admin
    ProxyJump bastion

Host bastion
    HostName 192.168.1.50
    User student
    Port 22
EOF

# Now simple connection:
ssh target                          # Automatically uses jump host
```

### Exercise 1.5: Connection Verification & Session Management

```bash
# Check active SSH connections (on remote server)
sudo ss -tnp | grep :22             # Active TCP connections
sudo netstat -tnpa | grep sshd      # Alternative method

# View logged-in users
who                                 # Current users
last -a                             # Login history
w                                   # Detailed current sessions

# Find your current session
who am i
echo $SSH_TTY                       # Your terminal
echo $SSH_CONNECTION                # Client IP, client port, server IP, server port

# Security audit: Find failed login attempts
sudo grep "Failed password" /var/log/secure | tail -20
sudo grep "Accepted" /var/log/secure | tail -10

# Kill stale SSH sessions (as root)
ps aux | grep sshd
sudo pkill -u suspicious_user       # Disconnect all sessions for user
```

***

## LAB 2: Secure File Transfer with SCP

### Exercise 2.1: Basic SCP Operations

```bash
# Setup test environment
mkdir -p ~/lab2_files
cd ~/lab2_files
echo "Test file $(date)" > local_file.txt
dd if=/dev/urandom of=largefile.bin bs=1M count=50  # 50MB test file

# UPLOAD: Local → Remote
scp local_file.txt student@192.168.1.100:/tmp/
scp -v local_file.txt student@192.168.1.100:/tmp/  # Verbose mode

# Verify upload succeeded:
ssh student@192.168.1.100 "ls -lh /tmp/local_file.txt"
ssh student@192.168.1.100 "cat /tmp/local_file.txt"

# DOWNLOAD: Remote → Local
scp student@192.168.1.100:/etc/hostname ./remote_hostname.txt
cat remote_hostname.txt             # Verify content

# Verify file integrity (checksum comparison)
md5sum local_file.txt
ssh student@192.168.1.100 "md5sum /tmp/local_file.txt"
# Both checksums MUST match
```

### Exercise 2.2: Advanced SCP Techniques

```bash
# Recursive directory copy
mkdir -p website/{css,js,images}
echo "<h1>Test</h1>" > website/index.html
echo "body { margin: 0; }" > website/css/style.css

scp -r website/ student@192.168.1.100:/var/www/html/

# Verify remote structure:
ssh student@192.168.1.100 "tree /var/www/html/website"

# Preserve timestamps and permissions
touch -t 202501010000 important.txt
chmod 640 important.txt
ls -l important.txt                 # Note timestamp and perms

scp -p important.txt student@192.168.1.100:/tmp/
ssh student@192.168.1.100 "ls -l /tmp/important.txt"
# Timestamp and permissions should be IDENTICAL

# Bandwidth limiting (simulate slow network)
time scp largefile.bin student@192.168.1.100:/tmp/test1.bin
time scp -l 1024 largefile.bin student@192.168.1.100:/tmp/test2.bin
# -l 1024 = 1 Mbps limit; second transfer should be slower

# Custom port
scp -P 2222 local_file.txt student@192.168.1.100:/tmp/
```

### Exercise 2.3: Real-World SCP Scenarios

```bash
# SCENARIO A: Backup multiple config files
scp student@192.168.1.100:/etc/{hostname,hosts,resolv.conf} ./backup/

# Verify all files copied:
ls -l backup/
md5sum backup/*

# SCENARIO B: Copy between two remote servers (3-way transfer)
scp -3 user1@server1:/data/file.txt user2@server2:/backup/
# -3: File transfers through local machine

# SCENARIO C: Through jump host
scp -o "ProxyJump student@bastion" file.txt admin@10.0.1.100:/tmp/

# SCENARIO D: Wildcard pattern copy
ssh student@192.168.1.100 "ls -1 /var/log/*.log"
scp student@192.168.1.100:'/var/log/*.log' ./logs/
# Note: Single quotes around remote path for wildcard expansion

# SCENARIO E: Progress monitoring for large files
scp largefile.bin student@192.168.1.100:/tmp/ &
# Get PID
SCP_PID=$!
watch -n 1 "ssh student@192.168.1.100 'ls -lh /tmp/largefile.bin'"
# Watch file growing on remote server
```

### Exercise 2.4: SCP Error Handling & Troubleshooting

```bash
# ERROR: Permission denied
scp test.txt student@192.168.1.100:/root/
# Fix: Use writable directory or sudo
scp test.txt student@192.168.1.100:/tmp/
ssh student@192.168.1.100 "sudo mv /tmp/test.txt /root/"

# ERROR: No such file or directory
scp test.txt student@192.168.1.100:/nonexistent/path/
# Fix: Create directory first
ssh student@192.168.1.100 "mkdir -p /tmp/newdir"
scp test.txt student@192.168.1.100:/tmp/newdir/

# ERROR: Disk space issues
ssh student@192.168.1.100 "df -h /tmp"  # Check before copying
# If space low, clean up first:
ssh student@192.168.1.100 "sudo rm -rf /tmp/old_data"

# Verify successful transfer with checksums
LOCAL_MD5=$(md5sum largefile.bin | awk '{print $1}')
REMOTE_MD5=$(ssh student@192.168.1.100 "md5sum /tmp/largefile.bin" | awk '{print $1}')

if [ "$LOCAL_MD5" = "$REMOTE_MD5" ]; then
    echo "✓ Transfer verified successfully"
else
    echo "✗ Transfer corrupted!"
fi
```

***

## LAB 3: Generate and Use RSA and DSA Keys

### Exercise 3.1: RSA Key Generation (Modern Standard)

```bash
# Lab setup
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cd ~/.ssh

# CRITICAL: Check for existing keys first
ls -la ~/.ssh/id_*
# If keys exist, backup before generating new ones:
[ -f ~/.ssh/id_rsa ] && cp ~/.ssh/id_rsa ~/.ssh/id_rsa.backup.$(date +%Y%m%d)

# Generate RSA-4096 key with passphrase [web:16]
ssh-keygen -t rsa -b 4096 -C "student-lab-$(date +%Y%m%d)"

# Prompts you'll see:
# Enter file in which to save the key (/home/student/.ssh/id_rsa): [Press Enter]
# Enter passphrase (empty for no passphrase): [Type: MyStr0ngP@ss!]
# Enter same passphrase again: [Type: MyStr0ngP@ss!]

# Verify key creation
ls -la ~/.ssh/id_rsa*
# Should show:
# -rw------- 1 student student 3389 Dec  2 19:30 id_rsa          (private)
# -rw-r--r-- 1 student student  743 Dec  2 19:30 id_rsa.pub      (public)

# CRITICAL: Verify permissions [web:16]
stat -c "%a %n" ~/.ssh/id_rsa       # Must show: 600
stat -c "%a %n" ~/.ssh/id_rsa.pub   # Must show: 644

# Fix permissions if incorrect:
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

### Exercise 3.2: View and Analyze RSA Keys

```bash
# Display public key
cat ~/.ssh/id_rsa.pub
# Format: ssh-rsa AAAAB3NzaC1yc2EAAA... student-lab-20251202

# Extract key fingerprint (SHA256 - default) [web:16]
ssh-keygen -lf ~/.ssh/id_rsa.pub
# Output: 4096 SHA256:XyZ... student-lab-20251202 (RSA)

# MD5 fingerprint (legacy compatibility)
ssh-keygen -lf ~/.ssh/id_rsa.pub -E md5
# Output: 4096 MD5:a1:b2:c3... student-lab-20251202 (RSA)

# View private key header (DO NOT share this)
head -n 1 ~/.ssh/id_rsa
# Should show: -----BEGIN OPENSSH PRIVATE KEY-----

# Get key details
ssh-keygen -l -v -f ~/.ssh/id_rsa.pub
# Shows visual ASCII art fingerprint

# Test key is valid
ssh-keygen -y -f ~/.ssh/id_rsa > /tmp/reconstructed_pub.key
diff ~/.ssh/id_rsa.pub /tmp/reconstructed_pub.key
# No output = keys match perfectly
```

### Exercise 3.3: DSA Key Generation (Legacy - Educational Only)

```bash
# WARNING: DSA is DEPRECATED and insecure for 2025 [web:3]
# This is for educational/legacy system compatibility only

# Generate DSA key (1024-bit - only supported size)
ssh-keygen -t dsa -f ~/.ssh/id_dsa_legacy -C "dsa-legacy-test"

# Compare key sizes
ls -lh ~/.ssh/id_rsa ~/.ssh/id_dsa_legacy
# RSA: ~3.3KB, DSA: ~668 bytes (much smaller, less secure)

# View differences
head -n 1 ~/.ssh/id_rsa
# -----BEGIN OPENSSH PRIVATE KEY-----

head -n 1 ~/.ssh/id_dsa_legacy  
# -----BEGIN OPENSSH PRIVATE KEY-----

file ~/.ssh/id_rsa ~/.ssh/id_dsa_legacy
# Shows different key types

# REALITY CHECK: Modern server rejection
ssh -i ~/.ssh/id_dsa_legacy student@192.168.1.100
# Likely ERROR: no mutual signature algorithm
# Many 2025 servers REJECT DSA entirely
```

### Exercise 3.4: Deploy Keys to Remote Server  

```bash
# Method 1: ssh-copy-id (EASIEST) [web:11]
ssh-copy-id student@192.168.1.100

# You'll see:
# /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s)
# Number of key(s) added: 1
# Now try logging into the machine...

# Verify key was added:
ssh student@192.168.1.100 "cat ~/.ssh/authorized_keys"

# Method 2: ssh-copy-id with specific key
ssh-copy-id -i ~/.ssh/id_rsa.pub student@192.168.1.100

# Method 3: Manual deployment (when ssh-copy-id unavailable) [web:16]
cat ~/.ssh/id_rsa.pub | ssh student@192.168.1.100 \
  "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# Method 4: Multiple keys to same server
ssh-copy-id -i ~/.ssh/id_rsa_work.pub student@192.168.1.100
ssh-copy-id -i ~/.ssh/id_rsa_personal.pub student@192.168.1.100
# Both keys now in authorized_keys

# Verify remote authorized_keys permissions [web:16]
ssh student@192.168.1.100 "stat -c '%a %n' ~/.ssh/authorized_keys"
# Must show: 600 /home/student/.ssh/authorized_keys
```

### Exercise 3.5: Test Key-Based Authentication

```bash
# First connection with new key (requires passphrase)
ssh student@192.168.1.100
# Enter passphrase for key '/home/student/.ssh/id_rsa': [Type passphrase]

# Verify key was used (not password)
ssh -v student@192.168.1.100 2>&1 | grep "Offering public key"
# Should show: debug1: Offering public key: /home/student/.ssh/id_rsa RSA SHA256:...
# Should show: debug1: Server accepts key: /home/student/.ssh/id_rsa RSA SHA256:...
# Should show: debug1: Authentication succeeded (publickey)

# Disable password authentication on server (security hardening)
ssh student@192.168.1.100
sudo sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
exit

# Test: Password auth should now be rejected
ssh -o PubkeyAuthentication=no student@192.168.1.100
# Should fail: Permission denied (publickey)

# Verify key auth still works
ssh student@192.168.1.100
# Should succeed with key
```

### Exercise 3.6: Key Lifecycle - Rotation & Management

```bash
# Generate new key for rotation
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_new -C "rotated-$(date +%Y%m%d)"

# Deploy new key alongside old key
ssh-copy-id -i ~/.ssh/id_rsa_new.pub student@192.168.1.100

# Verify BOTH keys present
ssh student@192.168.1.100 "cat ~/.ssh/authorized_keys | wc -l"
# Should show: 2 (or more)

# Test new key works
ssh -i ~/.ssh/id_rsa_new student@192.168.1.100 "hostname"

# Update config to use new key
cat >> ~/.ssh/config << EOF
Host lab-server
    HostName 192.168.1.100
    User student
    IdentityFile ~/.ssh/id_rsa_new
EOF

# Test config-based connection
ssh lab-server

# Remove old key from server (after confirming new key works)
ssh student@192.168.1.100
# On remote server:
OLD_KEY_FINGERPRINT="ssh-rsa AAAAB3... student-lab-20241115"
sed -i "/$OLD_KEY_FINGERPRINT/d" ~/.ssh/authorized_keys
exit

# Archive old key locally (don't delete immediately)
mkdir -p ~/.ssh/archive
mv ~/.ssh/id_rsa ~/.ssh/archive/id_rsa.old.$(date +%Y%m%d)
mv ~/.ssh/id_rsa.pub ~/.ssh/archive/id_rsa.pub.old.$(date +%Y%m%d)
```

### Exercise 3.7: Multi-Key Management Strategy

```bash
# Generate purpose-specific keys
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_github -C "github-deploy"
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_prod -C "production-servers"
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_dev -C "development-env"

# Configure SSH to use correct key per host
cat > ~/.ssh/config << 'EOF'
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_github
    IdentitiesOnly yes

Host prod-*
    User admin
    IdentityFile ~/.ssh/id_rsa_prod
    IdentitiesOnly yes
    StrictHostKeyChecking yes

Host dev-*
    User developer  
    IdentityFile ~/.ssh/id_rsa_dev
    IdentitiesOnly yes
EOF

chmod 600 ~/.ssh/config

# Test key selection
ssh -T git@github.com               # Uses id_rsa_github
ssh prod-web1                       # Uses id_rsa_prod
ssh dev-db1                         # Uses id_rsa_dev
```

***

## LAB 4: Use ssh-agent to Cache Decrypted Private Keys

### Exercise 4.1: Understand ssh-agent Basics  

```bash
# Check if ssh-agent already running
ps aux | grep ssh-agent
echo $SSH_AUTH_SOCK                 # Socket path (if running)
echo $SSH_AGENT_PID                 # Process ID (if running)

# If not running, start ssh-agent [web:17]
eval $(ssh-agent)
# Output:
# Agent pid 12345

# Verify environment variables set
echo $SSH_AUTH_SOCK
# Output: /tmp/ssh-XXXXXXXX/agent.12345

echo $SSH_AGENT_PID  
# Output: 12345

# Test agent is responsive
ssh-add -l
# Output: The agent has no identities.
```

### Exercise 4.2: Load Keys into ssh-agent  

```bash
# Add default RSA key to agent [web:17]
ssh-add ~/.ssh/id_rsa
# Prompts: Enter passphrase for /home/student/.ssh/id_rsa: [Type passphrase]
# Output: Identity added: /home/student/.ssh/id_rsa (student-lab-20251202)

# Verify key loaded [web:17]
ssh-add -l
# Output: 4096 SHA256:XyZ... student-lab-20251202 (RSA)

# List all loaded keys with full public key [web:17]
ssh-add -L
# Shows complete public key: ssh-rsa AAAAB3NzaC1yc2EAAA...

# Add multiple keys
ssh-add ~/.ssh/id_rsa_work
ssh-add ~/.ssh/id_rsa_github

ssh-add -l
# Should now show all 3 keys with fingerprints
```

### Exercise 4.3: Test Passwordless Authentication

```bash
# WITHOUT ssh-agent (requires passphrase each time)
ssh-add -D                          # Remove all keys from agent
ssh student@192.168.1.100
# Prompts for passphrase
exit

ssh student@192.168.1.100  
# Prompts for passphrase AGAIN (annoying!)
exit

# WITH ssh-agent (passphrase cached) [web:7][web:17]
ssh-add ~/.ssh/id_rsa
# Enter passphrase: [Type once]

ssh student@192.168.1.100
# NO passphrase prompt - seamless login
exit

ssh student@192.168.1.100
# STILL no passphrase - key cached in agent
exit

# Demonstrate performance improvement
time ssh student@192.168.1.100 "hostname"
# Should be < 1 second with agent
```

### Exercise 4.4: Advanced ssh-agent Key Management 

```bash
# Add key with timeout (expires after 1 hour) [web:17]
ssh-add -t 3600 ~/.ssh/id_rsa_prod
ssh-add -l
# Output shows: 4096 SHA256:... student-lab-20251202 (RSA) (3600s)

# Verify expiration countdown
sleep 10
ssh-add -l                          # Time remaining decreases

# Remove specific key from agent [web:17]
ssh-add -d ~/.ssh/id_rsa_work
ssh-add -l                          # Key should be gone

# Remove ALL keys [web:17]
ssh-add -D
ssh-add -l
# Output: The agent has no identities.

# Re-add keys for testing
ssh-add ~/.ssh/id_rsa

# Lock the agent (requires unlock password) [web:7]
ssh-add -x
# Enter lock password: [Type password]
# Output: Agent locked.

# Test: SSH should fail now
ssh student@192.168.1.100
# Error: sign_and_send_pubkey: signing failed: agent refused operation

# Unlock agent [web:7]
ssh-add -X
# Enter lock password: [Type same password]
# Output: Agent unlocked.

# Test: SSH works again
ssh student@192.168.1.100 "hostname"
```

### Exercise 4.5: Persistent ssh-agent Configuration 

```bash
# Method 1: Auto-start in ~/.bashrc [web:17]
cat >> ~/.bashrc << 'EOF'

# SSH Agent auto-start
if ! pgrep -u "$USER" ssh-agent > /dev/null; then
    eval $(ssh-agent -s)
    ssh-add ~/.ssh/id_rsa
fi
EOF

source ~/.bashrc                    # Test configuration

# Verify it works
logout                              # Close terminal
# Open new terminal
echo $SSH_AUTH_SOCK                 # Should be set
ssh-add -l                          # Should show keys

# Method 2: systemd user service (CentOS 9 preferred)
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/ssh-agent.service << 'EOF'
[Unit]
Description=SSH key agent

[Service]
Type=simple
Environment=SSH_AUTH_SOCK=%t/ssh-agent.socket
ExecStart=/usr/bin/ssh-agent -D -a $SSH_AUTH_SOCK

[Install]
WantedBy=default.target
EOF

# Enable and start
systemctl --user enable ssh-agent
systemctl --user start ssh-agent
systemctl --user status ssh-agent

# Add to shell environment
echo 'export SSH_AUTH_SOCK="$XDG_RUNTIME_DIR/ssh-agent.socket"' >> ~/.bashrc
source ~/.bashrc

# Add keys
ssh-add ~/.ssh/id_rsa

# Verify persistent across logins
logout
# Login again
ssh-add -l                          # Keys should still be loaded
```

### Exercise 4.6: Agent Forwarding (Advanced - Use with Caution) 

```bash
# SCENARIO: Need to access server2 from server1, but keys are on workstation
# Topology: workstation → server1 → server2

# Enable agent forwarding
ssh -A student@192.168.1.100       # -A enables forwarding

# On server1, verify agent is forwarded [web:7]
echo $SSH_AUTH_SOCK
# Output: /tmp/ssh-XXXXXX/agent.XXXX (forwarded socket)

ssh-add -l
# Shows YOUR LOCAL keys (from workstation)

# Connect to server2 using forwarded keys
ssh admin@192.168.1.101 "hostname"
# Works without keys on server1!

exit                                # Back to workstation

# Make forwarding persistent in config
cat >> ~/.ssh/config << 'EOF'
Host jump-server
    HostName 192.168.1.100
    User student
    ForwardAgent yes
EOF

ssh jump-server                     # Agent auto-forwarded
ssh-add -l                          # Shows your keys
```

### Exercise 4.7: Troubleshooting ssh-agent Issues

```bash
# PROBLEM 1: "Could not open a connection to your authentication agent"
ssh-add ~/.ssh/id_rsa
# Error: Could not open a connection to your authentication agent.

# FIX: Agent not running
eval $(ssh-agent)
ssh-add ~/.ssh/id_rsa               # Now works

# PROBLEM 2: "Permission denied (publickey)" despite ssh-agent
ssh -v student@192.168.1.100 2>&1 | grep -i "offer\|accept"
# debug1: Offering public key: ...
# debug1: Server accepts key: ...
# But still fails

# FIX: Key not in agent
ssh-add -l                          # Check loaded keys
ssh-add ~/.ssh/id_rsa               # Add if missing

# PROBLEM 3: Agent has stale socket
echo $SSH_AUTH_SOCK
ls -l $SSH_AUTH_SOCK
# Error: No such file or directory

# FIX: Kill old agent and start new
killall ssh-agent
eval $(ssh-agent)
ssh-add ~/.ssh/id_rsa

# PROBLEM 4: Multiple agents running
ps aux | grep ssh-agent | grep -v grep
# Shows multiple processes

# FIX: Kill all and start one
pkill ssh-agent
eval $(ssh-agent)

# PROBLEM 5: Verify which agent SSH is using
ssh -v student@192.168.1.100 2>&1 | grep -i agent
# debug1: Connecting to authentication agent.
# debug1: Trying private key: /home/student/.ssh/id_rsa
```

### Exercise 4.8: Security Best Practices Lab

```bash
# Create confirmation-required key
ssh-add -c ~/.ssh/id_rsa_prod
# -c: Requires confirmation for each use

# Test: Every SSH connection prompts for confirmation
ssh student@192.168.1.100
# Dialog: Allow use of key /home/student/.ssh/id_rsa_prod? (yes/no)

# Environment-specific agents (production isolation)
# Terminal 1 - Production agent
ssh-agent -a ~/.ssh/agent.prod > /tmp/prod_env.sh
source /tmp/prod_env.sh
ssh-add ~/.ssh/id_rsa_prod
ssh-add -l                          # Only prod key

# Terminal 2 - Development agent
ssh-agent -a ~/.ssh/agent.dev > /tmp/dev_env.sh
source /tmp/dev_env.sh
ssh-add ~/.ssh/id_rsa_dev
ssh-add -l                          # Only dev key

# Use specific agent
SSH_AUTH_SOCK=~/.ssh/agent.prod ssh prod-server
SSH_AUTH_SOCK=~/.ssh/agent.dev ssh dev-server

# Clean up agents when done
SSH_AUTH_SOCK=~/.ssh/agent.prod ssh-agent -k
SSH_AUTH_SOCK=~/.ssh/agent.dev ssh-agent -k
```

### Exercise 4.9: Complete Workflow Validation

```bash
# FINAL COMPREHENSIVE TEST

# 1. Start fresh
ssh-add -D                          # Clear agent
killall ssh-agent
eval $(ssh-agent)

# 2. Add keys with different lifetimes
ssh-add -t 7200 ~/.ssh/id_rsa       # 2 hours
ssh-add -t 3600 ~/.ssh/id_rsa_work  # 1 hour

# 3. Verify loaded
ssh-add -l
# Should show both keys with remaining time

# 4. Test connections
ssh student@192.168.1.100 "echo Connection 1 successful"
ssh student@192.168.1.100 "echo Connection 2 successful"
# No passphrase prompts = SUCCESS

# 5. Simulate long-running session
sleep 3601                          # Wait for work key to expire
ssh-add -l
# Should show only id_rsa (work key expired)

# 6. Verify expired key doesn't work
ssh -i ~/.ssh/id_rsa_work student@192.168.1.100
# Prompts for passphrase (not in agent anymore)

# 7. Lock agent before stepping away
ssh-add -x
# Agent locked - keys protected even if compromised

# 8. Unlock when back
ssh-add -X
ssh student@192.168.1.100 "echo Unlocked and working"

# SUCCESS CRITERIA:
# ✓ No passphrase prompts for cached keys
# ✓ Expired keys removed automatically  
# ✓ Lock/unlock works properly
# ✓ Multiple keys managed simultaneously
```

This lab guide provides complete hands-on exercises with validation steps, troubleshooting scenarios, and real-world patterns for mastering SSH operations on CentOS 9 in 2025.    