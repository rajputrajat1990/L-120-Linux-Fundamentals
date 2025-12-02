# Linux Command Line Messaging Cheatsheet (CentOS 9 Stream - 2025 Edition)

## Command Line Messaging Fundamentals

### write - Direct User Messaging
The `write` command sends messages to specific logged-in users. **Note**: `write` and `mesg` commands have been removed from some recent Linux distributions (Debian/Ubuntu) but remain available in RHEL 9/CentOS 9 Stream.  

**Basic Syntax:**
```bash
write username [tty]          # Send message to specific user
write alice                   # Opens message prompt; Ctrl+D to send
write bob pts/0              # Send to specific terminal session
```

**Check Who's Logged In:**
```bash
who                          # List all logged-in users
w                            # Show who's logged in and what they're doing
users                        # Simple list of logged-in usernames
finger username              # Detailed user info (if installed)
```

**Practical Examples:**
```bash
# Send quick message
echo "Meeting in 5 minutes" | write alice

# Multi-line message
write bob << EOF
Code review scheduled at 3 PM
Conference Room B
EOF
```

### wall - Broadcast to All Users
Sends messages to all currently logged-in users' terminals. **Requires root/sudo for system-wide broadcasts**.  

**Syntax:**
```bash
wall [options] [message]
wall -n [message]            # Suppress banner (no "Broadcast message from...")
wall -t timeout [message]    # Timeout in seconds (default: 300)
```

**Practical Usage:**
```bash
# Simple broadcast
sudo wall "System reboot in 10 minutes"

# From file
sudo wall < /tmp/maintenance_notice.txt

# Without banner
sudo wall -n "Emergency maintenance starting now"

# With timeout (30 seconds)
sudo wall -t 30 "Quick system check in progress"

# Multi-line broadcast
sudo wall << 'END'
CRITICAL: Security patch deployment
Expected downtime: 15 minutes
Start time: 10:00 PM IST
END
```

### mesg - Control Message Reception
Controls whether other users can send messages to your terminal via `write` or `talk`. 

**Commands:**
```bash
mesg                         # Check current status (is y or n)
mesg y                       # Allow messages
mesg n                       # Block messages
mesg --verbose              # Show detailed status
```

**Security Best Practice (2025):**
```bash
# Add to ~/.bashrc for secure defaults
mesg n                       # Block messages by default

# Check terminal permissions
ls -l $(tty)                 # View current terminal permissions
# Example output: crw--w---- (group write enabled)
```

**Permission Details:**
- Default in CentOS 9: Mode `0620` (owner rw, group w) if tty group exists, else `0600` 
- `mesg y` enables group write permission
- `mesg n` disables group write permission
- Since util-linux 2.41: Only modifies group permissions, not "others"

## Legacy Tools (Historical Context)

### talk - Real-Time Chat Between Two Users
**Status in 2025**: Largely deprecated; rarely installed by default in modern distributions.

**Installation (if needed):**
```bash
sudo dnf install talk talk-server  # CentOS 9
```

**Basic Usage:**
```bash
talk username                # Talk to user on same machine
talk username@hostname       # Talk to remote user
talk alice pts/0             # Specify terminal
```

**Why It's Obsolete:**
- Split-screen terminal interface (impractical on modern systems)
- No encryption or security features
- Replaced by SSH + tmux/screen sharing or modern messaging apps

### ytalk - Enhanced Talk Alternative
**Status in 2025**: Essentially extinct; not maintained.

- Multi-user chat support (beyond 2 users)
- Multiple window support
- Not available in standard RHEL 9/CentOS 9 repos
- Use modern alternatives instead

## Internet Relay Chat (IRC)

IRC remains actively used in 2025 for open-source communities, tech support channels, and developer collaboration.  

### Best Terminal-Based IRC Clients

#### 1. WeeChat (Recommended for Power Users)
```bash
# Installation
sudo dnf install weechat

# Basic usage
weechat                      # Start WeeChat

# Inside WeeChat - essential commands
/server add liberachat irc.libera.chat/6697 -ssl
/connect liberachat
/join #centos
/join #linux
/nick your_nickname
/quit                        # Exit

# Configuration
/set irc.server.liberachat.autoconnect on
/set irc.server.liberachat.autojoin "#linux,#centos"
/save                        # Save config
```

**WeeChat Features:** 
- Multi-protocol (IRC, XMPP, Matrix via plugins)
- Highly scriptable (Python, Perl, Ruby, Lua)
- Active development and excellent documentation
- Lightweight and fast

#### 2. Irssi (Traditional CLI Client)
```bash
# Installation
sudo dnf install irssi

# Basic usage
irssi

# Inside Irssi - essential commands
/connect irc.libera.chat
/nick your_nickname
/join #channel_name
/msg nickname private_message
/part #channel              # Leave channel
/quit                       # Exit

# Auto-connect setup
/server add -auto -network Libera irc.libera.chat 6697
/channel add -auto #linux Libera
/save
```

**Irssi Features:**  
- Auto-logging capabilities
- Extensive Perl scripting API
- Theme and format customization
- Proxy support for bouncer setups

#### 3. HexChat (GUI IRC Client)
```bash
# Installation
sudo dnf install hexchat

# Launch
hexchat
```

**HexChat Features:** 
- User-friendly GUI
- Python and Perl scripting support
- SSL server support
- Multi-server/multi-channel in single window

### Popular IRC Networks (2025)
```bash
# Libera.Chat (largest FOSS network)
/connect irc.libera.chat 6697 -ssl

# OFTC (Debian, LLVM, other projects)
/connect irc.oftc.net 6697 -ssl

# Freenode (historical, less active post-2021)
/connect irc.freenode.net 6697 -ssl
```

### IRC Security Best Practices
```bash
# Always use SSL/TLS
/connect irc.libera.chat 6697 -ssl

# Register and authenticate nickname
/msg NickServ REGISTER password email@example.com
/msg NickServ IDENTIFY password

# Use SASL authentication (WeeChat example)
/set irc.server.liberachat.sasl_username your_nick
/set irc.server.liberachat.sasl_password your_password
/save
```

## Modern Instant Messaging (2025 Alternatives)

### Matrix Protocol - Decentralized Messaging

**Element (CLI via Matrix Commander):** 
```bash
# Install from PyPI
pip3 install matrix-commander

# Initial setup
matrix-commander --login
matrix-commander --store /path/to/credentials.json

# Send messages
matrix-commander -m "Hello from terminal"
matrix-commander -r '!roomid:matrix.org' -m "Message to specific room"

# Listen mode
matrix-commander --listen forever

# Use cases: Cron jobs, bots, notifications
```

**Element Desktop with Privacy (Nym Mixnet):** 
```bash
# Launch Element via SOCKS5 proxy
element-desktop --proxy-server=socks5://127.0.0.1:1080

# Create alias in ~/.bashrc
alias element="element-desktop --proxy-server=socks5://127.0.0.1:1080"
```

### Slack-Like Alternatives (Self-Hosted)

**Mattermost:** 
- Written in Golang + React
- Self-hosted option available
- LDAP/Active Directory integration
- Slack import functionality
- Private/public channels, 1-on-1 messaging

```bash
# CLI interaction (using mmctl)
sudo dnf install mmctl       # If packaged
mmctl auth login https://mattermost.example.com
mmctl channel list myteam
mmctl post create myteam:channel-name --message "Hello"
```

**Zulip:** 
- Apache licensed
- Thread-based conversations (organized by "streams")
- @-mentions, file uploads, rich media
- LDAP integration available

### Terminal-Based Modern Messaging

**Telegram CLI:**
```bash
# Installation (from source or COPR)
git clone --recursive https://github.com/vysheng/tg.git
cd tg
./configure
make

# Usage
telegram-cli
```

**Signal CLI:**
```bash
# Installation
sudo dnf install signal-cli  # May require EPEL or manual install

# Register
signal-cli -u +your_number register
signal-cli -u +your_number verify CODE

# Send message
signal-cli -u +your_number send -m "message" +recipient_number
```



### What's Actually Used in Production
- **`wall`**: Still actively used for system maintenance notifications on multi-user servers
- **`write`**: Rarely used; SSH + modern tools preferred
- **`talk`/`ytalk`**: Effectively dead; historical curiosity only
- **IRC**: Alive and well in open-source/tech communities (Libera.Chat, OFTC)
- **Modern alternatives**: Matrix, Mattermost, Zulip for team collaboration

### Security Considerations
```bash
# Disable legacy messaging on secure systems
mesg n                       # Block terminal write access
sudo chmod 600 $(tty)        # Remove group write permissions

# Monitor who's logged in (security auditing)
w                           # Current users and activity
last                        # Login history
lastlog                     # Last login per user
who -a                      # All login information
```

### SystemD Integration (Modern Approach)
```bash
# Send wall messages via systemd
sudo systemctl status        # Shows system messages
journalctl -f                # Follow system logs
wall < <(systemctl status service_name)  # Broadcast service status
```

### Automation & Scripting
```bash
# Notify all users before reboot (cron/script)
sudo wall "System rebooting in 5 minutes for kernel updates" && sleep 300 && sudo reboot

# Conditional messaging
if who | grep -q alice; then
    echo "Build complete" | write alice
fi

# Integration with monitoring
tail -f /var/log/critical.log | while read line; do
    sudo wall "ALERT: $line"
done
```

### Network Messaging Beyond Local
```bash
# SSH-based messaging to remote users
ssh user@remote-host "wall 'Message from admin'"

# Send notifications via Matrix from scripts
matrix-commander -m "Backup completed on $(hostname) at $(date)"

# IRC bot for system notifications (using WeeChat or custom script)
echo "/msg #ops Server $(hostname) backup completed" | weechat -r -
```
# Linux Electronic Mail (CentOS 9 Stream - 2025 Edition)

## Mail Transfer Agent (MTA) Landscape

### Postfix - Default MTA in CentOS 9
**Postfix has replaced Sendmail as the default MTA in RHEL 9/CentOS 9 Stream**. It's more secure, faster, and easier to configure than Sendmail.  

**Installation & Basic Setup:**
```bash
# Install Postfix (usually pre-installed)
sudo dnf install postfix

# Start and enable Postfix
sudo systemctl start postfix
sudo systemctl enable postfix
sudo systemctl status postfix

# Check if Postfix is listening
sudo ss -tulpn | grep :25
netstat -an | grep :25
```

**Basic Configuration (`/etc/postfix/main.cf`):**
```bash
# Edit main configuration
sudo vi /etc/postfix/main.cf

# Key parameters for local mail only (default)
inet_interfaces = loopback-only     # Listen only on localhost
mydestination = $myhostname, localhost.$mydomain, localhost
mynetworks_style = host

# For accepting mail from network
inet_interfaces = all               # Listen on all interfaces
myhostname = mail.example.com      # FQDN of your server
mydomain = example.com
myorigin = $mydomain
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

# Relay configuration (null client - forward all mail)
relayhost = [smtp.gmail.com]:587   # External SMTP relay
mydestination =                     # Empty - not final destination
```

**SMTP Authentication (Gmail/Office365 relay):** 
```bash
# Create SASL password file
sudo vi /etc/postfix/sasl_passwd
# Add line: [smtp.gmail.com]:587 username@gmail.com:app_password

# Hash the password file
sudo postmap /etc/postfix/sasl_passwd

# Secure the files
sudo chmod 600 /etc/postfix/sasl_passwd
sudo chmod 600 /etc/postfix/sasl_passwd.db

# Add to /etc/postfix/main.cf
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_security_level = encrypt
smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt

# Reload Postfix
sudo systemctl reload postfix
```

**Firewall Configuration:**
```bash
# Allow SMTP
sudo firewall-cmd --permanent --add-service=smtp
sudo firewall-cmd --permanent --add-port=25/tcp

# For submission port (587)
sudo firewall-cmd --permanent --add-port=587/tcp
sudo firewall-cmd --reload
```

**Troubleshooting & Logs:**
```bash
# Real-time mail log monitoring
sudo tail -f /var/log/maillog
journalctl -u postfix -f

# Check mail queue
mailq
postqueue -p

# Flush mail queue (attempt delivery)
postqueue -f
sudo postfix flush

# Remove specific message from queue
sudo postsuper -d <queue_id>

# Remove all mail from queue
sudo postsuper -d ALL

# Test Postfix config
sudo postfix check
```

### Sendmail - Legacy MTA (Still Available)

**Installation:** 
```bash
# Install sendmail
sudo dnf install sendmail sendmail-cf

# Stop Postfix if running (conflict on port 25)
sudo systemctl stop postfix
sudo systemctl disable postfix

# Start Sendmail
sudo systemctl start sendmail
sudo systemctl enable sendmail
```

**Configuration (`/etc/mail/sendmail.mc`):** 
```bash
# Edit macro configuration
sudo vi /etc/mail/sendmail.mc

# Key configurations to modify:
# DAEMON_OPTIONS(`Port=smtp,Addr=0.0.0.0, Name=MTA')dnl  # Listen on all IPs
# define(`SMART_HOST', `smtp.relay.com')dnl              # Set relay host
# define(`confAUTH_MECHANISMS', `LOGIN PLAIN')dnl       # Auth mechanisms

# Rebuild sendmail.cf from sendmail.mc
cd /etc/mail
sudo m4 sendmail.mc > sendmail.cf

# Restart Sendmail
sudo systemctl restart sendmail
```

**Why Postfix is Preferred (2025):**  
- Postfix is default in RHEL 9/CentOS 9
- Easier configuration (key-value pairs vs m4 macros)
- Better security track record
- Higher performance and scalability
- Modern TLS/encryption enabled by default

## Command-Line Email Tools

### s-nail - Modern Replacement for mail/mailx

**In CentOS 9/RHEL 9, the traditional `mailx` command has been replaced by `s-nail`**.  

**Installation:**
```bash
# Install s-nail
sudo dnf install s-nail

# Also need an MTA (Postfix)
sudo dnf install postfix
sudo systemctl start postfix
```

**Basic Usage:**
```bash
# Simple email
echo "Email body" | s-nail -s "Subject" recipient@example.com

# From file
s-nail -s "Subject" recipient@example.com < message.txt

# With custom sender
echo "Body" | s-nail -s "Subject" -r "sender@example.com" recipient@example.com

# Multiple recipients
echo "Body" | s-nail -s "Subject" user1@example.com user2@example.com

# CC and BCC
s-nail -s "Subject" -c "cc@example.com" -b "bcc@example.com" recipient@example.com

# With attachment
echo "See attached" | s-nail -s "Report" -a /path/to/file.pdf recipient@example.com

# HTML email
s-nail -s "HTML Test" -M "text/html" recipient@example.com < message.html
```

**Interactive Mode:**
```bash
# Start s-nail interactively
s-nail

# Inside s-nail:
?                           # Help
h                           # List headers
1                           # Read message 1
n                           # Next message
d 1                         # Delete message 1
d 1-5                       # Delete messages 1 through 5
u 1                         # Undelete message 1
q                           # Quit and save changes
x                           # Quit without saving changes
```

**Reading Local Mail:**
```bash
# Read your local mailbox
s-nail

# Read specific user's mail (as root)
s-nail -u username

# Read specific mailbox file
s-nail -f /var/spool/mail/username
```

**Advanced s-nail with Remote SMTP:** 
```bash
# Send via external SMTP (no local MTA required)
echo "Body" | s-nail -s "Subject" \
  -S mta=smtp://smtp.gmail.com:587 \
  -S smtp-use-starttls \
  -S smtp-auth=login \
  -S smtp-auth-user=user@gmail.com \
  -S smtp-auth-password='app_password' \
  -r "user@gmail.com" \
  recipient@example.com

# Create ~/.mailrc for persistent settings
vi ~/.mailrc
# Add:
set mta=smtp://smtp.gmail.com:587
set smtp-use-starttls
set smtp-auth=login
set smtp-auth-user=user@gmail.com
set smtp-auth-password='app_password'
set from="user@gmail.com"

# Then send simply:
echo "Body" | s-nail -s "Subject" recipient@example.com
```

**Scripting with s-nail:**
```bash
# Send system report
df -h | s-nail -s "Disk Usage Report - $(hostname)" admin@example.com

# Send with heredoc
s-nail -s "Server Status" admin@example.com << EOF
Server: $(hostname)
Uptime: $(uptime)
Load: $(cat /proc/loadavg)
Memory: $(free -h | grep Mem)
EOF

# Conditional email on failure
backup_script.sh || echo "Backup failed on $(hostname)" | \
  s-nail -s "ALERT: Backup Failure" admin@example.com

# Cron job notifications
0 2 * * * /usr/local/bin/backup.sh 2>&1 | s-nail -s "Nightly Backup Log" admin@example.com
```

### Legacy mail Command (Pre-EL9)
**Note:** On CentOS 9, use `s-nail` instead. The `mail` command is provided by the `s-nail` package as a compatibility wrapper. 

```bash
# If you see scripts using 'mail', they should work via s-nail compatibility
mail -s "Subject" recipient@example.com < body.txt
```

## Alpine Email Client (PINE Successor)

**PINE (Program for Internet News & Email) was discontinued in 2005. Alpine is its modern, open-source successor**.  

**Installation:**
```bash
# Install Alpine
sudo dnf install alpine

# Launch Alpine
alpine
```

**First-Time Setup:** 
```bash
# At startup, press 'E' to exit license
# Press 'S' for Setup, then 'C' for Config

# Essential configurations:
Personal Name = Your Full Name
User Domain = example.com
SMTP Server (for sending) = smtp.gmail.com:587/tls/user=username@gmail.com
IMAP Server = imap.gmail.com:993/ssl/user=username@gmail.com

# Press 'E' to exit setup
# Press 'Y' to commit changes
```

**Advanced Alpine Configuration:**
```bash
# Direct config file editing (optional)
vi ~/.pinerc

# Key settings in .pinerc:
user-domain=example.com
smtp-server=smtp.gmail.com:587/tls/user=username@gmail.com/novalidate-cert
inbox-path={imap.gmail.com:993/ssl/user=username@gmail.com}INBOX
folder-collections="Mail" {imap.gmail.com:993/ssl/user=username@gmail.com}[]

# Enable useful features (in Setup > Config > Enable Features):
- enable-background-sending
- enable-verbose-smtp-posting
- warn-if-blank-subject
- combined-folder-display
- enable-alternate-editor-cmd    # Use vim/nano for composition
```

**Navigation & Usage:** 
```bash
# Main Menu:
M       # Main Menu
L       # Folder List
I       # Message Index (Inbox)
C       # Compose new message
A       # Address Book
S       # Setup
Q       # Quit Alpine

# In Message Index:
Enter   # Read selected message
N       # Next message
P       # Previous message
D       # Delete message
U       # Undelete message
R       # Reply
F       # Forward
S       # Save message to folder
/       # Search
;       # Select current message (for batch operations)
A       # Apply command to selected messages

# In Compose mode:
Ctrl+X  # Send message
Ctrl+C  # Cancel
Ctrl+R  # Read file into message
Ctrl+J  # Attach file
Ctrl+T  # To Address Book
```

**Sending Email with Alpine:**
```bash
# Method 1: Launch and compose
alpine
# Press 'C' for Compose
# Fill: To, Cc, Attachmnt, Subject, Message
# Ctrl+X to send

# Method 2: Command-line email
alpine -attach /path/to/file.pdf recipient@example.com

# Method 3: Batch mode (scripting)
alpine -I XS,1,2,3,4          # Execute commands (not commonly used)
```

**Alpine for Gmail/Modern Providers (2025):** 
```bash
# IMPORTANT: Use App Passwords, not 2FA passwords
# Gmail: Generate at https://myaccount.google.com/apppasswords

# ~/.pinerc settings for Gmail:
user-domain=gmail.com
smtp-server=smtp.gmail.com:587/tls/user=yourusername@gmail.com
inbox-path={imap.gmail.com:993/ssl/user=yourusername@gmail.com}INBOX

# Features to enable:
- enable-background-sending
- quell-ssl-largeblob-warning  # Gmail has large certs
```

**Alpine Folder Management:**
```bash
# In Folder List (press 'L'):
A       # Add folder
D       # Delete folder
R       # Rename folder
Enter   # Open folder

# Subscribe to IMAP folders
S       # Subscribe
U       # Unsubscribe
```

**Smart Alpine Tips (2025):**
- Alpine excels at IMAP folder management (better than Thunderbird for complex hierarchies) 
- Lightweight and fast for SSH/remote access scenarios
- Excellent for text-only email workflows
- Not ideal for HTML-heavy emails or attachments-heavy workflows
- Consider NeoMutt if you need more customization 

## Evolution - GNOME Email Client

**Evolution is the default integrated email/calendar/contacts client for GNOME desktops**.   

**Installation:**
```bash
# Install Evolution
sudo dnf install evolution evolution-ews  # ews = Exchange Web Services

# Launch
evolution
```

**Initial Setup (GUI Wizard):** 
```bash
# Evolution walks through:
1. Identity: Name, email address
2. Receiving Email: 
   - IMAP (recommended for Gmail/modern providers)
   - POP3 (downloads and removes from server)
   - Exchange Web Services (for Office365/Exchange)
3. Server settings: hostname, port, encryption
4. Sending Email: SMTP server configuration
5. Account summary and completion
```

**Evolution Account Configuration Examples:**

**Gmail:**
```
Receiving (IMAP):
  Server: imap.gmail.com
  Port: 993
  Security: SSL/TLS
  Username: yourname@gmail.com
  
Sending (SMTP):
  Server: smtp.gmail.com
  Port: 587
  Security: STARTTLS
  Authentication required: Yes
  Username: yourname@gmail.com
```

**Office365/Exchange:**  
```
Account Type: Exchange Web Services
Email: yourname@company.com
Server: outlook.office365.com
Username: yourname@company.com
Authentication: OAuth2 (Modern Auth)
```

**Evolution Features:**  
- **Integrated PIM**: Email, calendar, contacts, tasks, memos in one application
- **Microsoft Exchange support**: Best Linux client for Exchange/Office365  
- **GPG & S/MIME encryption**: Built-in security
- **Spam filtering**: SpamAssassin integration
- **Out-of-office replies**: Auto-reply feature (Thunderbird lacks this) 
- **IMAP folder sync**: Superior to Thunderbird for Outlook-created folders 
- **Read status sync**: Remembers read status across clients/devices 

**Evolution Command-Line Operations:**
```bash
# Launch Evolution
evolution

# Compose new email from command line
evolution mailto:recipient@example.com

# With subject and body
evolution "mailto:recipient@example.com?subject=Test&body=Hello"

# With attachment (URL-encoded)
evolution "mailto:test@example.com?attach=/path/to/file.pdf"

# Reset Evolution (troubleshooting)
evolution --force-shutdown
mv ~/.local/share/evolution ~/.local/share/evolution.bak
mv ~/.config/evolution ~/.config/evolution.bak

# Evolution backup
evolution --backup=/path/to/backup.tar.gz

# Evolution restore
evolution --restore=/path/to/backup.tar.gz
```

**GNOME Online Accounts Integration:** 
```bash
# Settings > Online Accounts
# Add Google, Microsoft, or other accounts
# Evolution automatically detects and uses these accounts

gnome-control-center online-accounts  # Open from terminal
```

**Evolution vs Thunderbird (2025 Context):**   

| Feature | Evolution | Thunderbird |
|---------|-----------|-------------|
| Exchange/Office365 | Native EWS support  | Requires add-ons/IMAP workaround |
| GNOME Integration | Excellent (native)  | Good but not native |
| Calendar Sync | Built-in, works with Google/Exchange | Requires Lightning add-on (now built-in TB 115+) |
| Cross-platform | Linux-only | Windows/Mac/Linux |
| Memory Usage | Lower on GNOME  | Higher |
| Configuration | GUI-focused | GUI + config files |
| Out-of-office | Yes  | No (need Outlook)  |

## Alternative Modern Email Clients (2025)

### NeoMutt - Modern Mutt Fork
**For power users who want maximum control and keyboard-driven workflows**.  

```bash
# Installation
sudo dnf install neomutt

# Minimal ~/.neomuttrc
set from = "you@example.com"
set realname = "Your Name"
set smtp_url = "smtps://you@smtp.gmail.com:465/"
set smtp_pass = "app_password"
set folder = "imaps://imap.gmail.com:993"
set spoolfile = "+INBOX"
set record = "+Sent"
set postponed = "+Drafts"
set trash = "+Trash"

# Launch
neomutt
```

**NeoMutt + mbsync + mu Setup (Advanced 2025 Stack):** 
- `mbsync`: IMAP sync with OAuth2
- `mu`: Fast email indexing/search
- `neomutt`: Email client
- `msmtp`: SMTP sending
- Best for: Offline access, fast search, complete control

### Thunderbird
```bash
# Installation
sudo dnf install thunderbird

# Launch
thunderbird
```

**Thunderbird Advantages:**
- Cross-platform (Windows/Mac/Linux)
- Large add-on ecosystem
- Active development (Mozilla)
- Familiar interface for Outlook users

## Practical Labs

### Lab 1: User Communication (mesg/write/talk)

**Quick Lab Steps:**
```bash
# Terminal 1 (User alice):
who              # Check logged-in users
mesg y           # Allow messages
write bob        # Send message to bob

# Terminal 2 (User bob):
mesg y           # Allow messages
# Receives message from alice
write alice      # Reply to alice
```

### Lab 2: Send Mail Using s-nail and Alpine

**Lab 2A: s-nail Command-Line Email**
```bash
# Setup local Postfix first
sudo dnf install postfix s-nail
sudo systemctl start postfix

# Test 1: Local mail
echo "This is a test from s-nail" | s-nail -s "Test Subject" $USER

# Test 2: Check local mail
s-nail
# Press '1' to read first message, 'q' to quit

# Test 3: Send to external email (requires relay)
echo "External test" | s-nail -s "Test" your.email@gmail.com

# Test 4: Send with attachment
echo "Report attached" | s-nail -s "Daily Report" \
  -a /var/log/messages \
  admin@example.com

# Test 5: Multiple recipients
s-nail -s "Team Update" team@example.com << EOF
Project Status:
- Phase 1: Complete
- Phase 2: In progress
- Phase 3: Planning
EOF
```

**Lab 2B: Alpine Email Client**
```bash
# Install and configure Alpine
sudo dnf install alpine
alpine

# First-time setup:
# 1. Press 'S' (Setup) > 'C' (Config)
# 2. Set SMTP Server: smtp.gmail.com:587/tls/user=you@gmail.com
# 3. Set IMAP path (optional for receiving)
# 4. Exit and commit changes

# Compose and send email:
# 1. Press 'C' to compose
# 2. Fill To: recipient@example.com
# 3. Fill Subject: Test from Alpine
# 4. Type message body
# 5. Ctrl+X to send

# Read mail:
# 1. Press 'I' for Inbox
# 2. Arrow keys to navigate
# 3. Enter to read
# 4. 'R' to reply, 'D' to delete

# Attach file:
# 1. Press 'C' to compose
# 2. Tab to "Attchmt:" field
# 3. Ctrl+J to attach
# 4. Enter filename: /path/to/file
# 5. Complete and send with Ctrl+X
```

**Lab 2C: Production Mail Relay Setup**
```bash
# Gmail relay configuration (realistic scenario)
sudo vi /etc/postfix/sasl_passwd
# Add: [smtp.gmail.com]:587 username@gmail.com:app_password

sudo postmap /etc/postfix/sasl_passwd
sudo chmod 600 /etc/postfix/sasl_passwd*

sudo vi /etc/postfix/main.cf
# Add relay configuration (see Postfix section above)

sudo systemctl reload postfix

# Test relay
echo "Production relay test" | s-nail -s "Relay Test" recipient@example.com

# Monitor logs
sudo tail -f /var/log/maillog
```

## Smart-in-the-Room Knowledge (2025)

### What's Actually Used in Production
- **Postfix**: Industry standard MTA (default in RHEL/CentOS 9)  
- **s-nail**: Replaced mail/mailx in EL9 distributions 
- **Sendmail**: Legacy, still available but avoid for new deployments 
- **Alpine**: Used by sysadmins for SSH email access scenarios  
- **Evolution**: GNOME default, best for Exchange/Office365 on Linux  
- **NeoMutt/Mutt**: Power user terminal email workflows 

### Automation & Monitoring
```bash
# Email disk usage alerts
df -h | awk '$5 > 80 {print}' | s-nail -s "DISK ALERT" admin@example.com

# Database backup notification
pg_dump database | gzip > backup.sql.gz && \
  echo "Backup completed at $(date)" | s-nail -s "DB Backup OK" dba@example.com

# System health report (cron daily)
cat << EOF | s-nail -s "Daily Health: $(hostname)" admin@example.com
Uptime: $(uptime)
Disk: $(df -h / | tail -1)
Memory: $(free -h | grep Mem)
Failed Services: $(systemctl --failed)
EOF

# Failed systemd service alerts
systemctl list-units --state=failed --no-pager | \
  s-nail -s "ALERT: Failed Services on $(hostname)" admin@example.com
```

### Security Best Practices (2025)
```bash
# Always use TLS/encryption
# Postfix: smtp_tls_security_level = encrypt
# s-nail: smtp-use-starttls
# Alpine: /tls or /ssl in server config

# Use app-specific passwords (never account passwords)
# Gmail: https://myaccount.google.com/apppasswords
# Office365: App passwords or OAuth2

# Protect credentials
sudo chmod 600 /etc/postfix/sasl_passwd
chmod 600 ~/.mailrc ~/.pinerc

# Monitor mail logs for anomalies
sudo tail -f /var/log/maillog | grep -i "warning\|error\|reject"

# Implement SPF/DKIM/DMARC for sent mail
# Check DNS records: dig txt example.com
```
