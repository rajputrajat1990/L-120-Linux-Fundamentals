# Linux Regular Expressions

## Regular Expression Overview

Regular expressions (regex) are pattern-matching languages used to search, match, and manipulate text. CentOS 9 supports three main regex flavors: **Basic Regular Expressions (BRE)**, **Extended Regular Expressions (ERE)**, and **Perl-Compatible Regular Expressions (PCRE)**.   

### BRE vs ERE vs PCRE - Critical Differences

| Feature | BRE (default grep) | ERE (grep -E / egrep) | PCRE (grep -P) |
|---------|-------------------|----------------------|----------------|
| Metacharacters require escaping | `\+` `\?` `\|` `\{` `\(` | `+` `?` `|` `{` `(` | Same as ERE + advanced features |
| Alternation (OR) | `\|` | `|` | `|` |
| Grouping | `\( \)` | `( )` | `( )` |
| One or more | `\+` | `+` | `+` |
| Zero or one | `\?` | `?` | `?` |
| Quantifiers | `\{n,m\}` | `{n,m}` | `{n,m}` |
| Lookaheads/Lookbehinds | ❌ | ❌ | ✅ |
| Lazy matching | ❌ | ❌ | ✅ |

### Core Metacharacters (Universal)

```bash
.          # Any single character except newline
^          # Start of line (anchor)
$          # End of line (anchor)
*          # Zero or more of preceding element
[ ]        # Character class/set
[^ ]       # Negated character class
\          # Escape special character
```

### POSIX Character Classes (Standardized, Portable)

```bash
[[:alnum:]]   # Alphanumeric [A-Za-z0-9]
[[:alpha:]]   # Alphabetic [A-Za-z]
[[:digit:]]   # Digits [0-9]
[[:lower:]]   # Lowercase [a-z]
[[:upper:]]   # Uppercase [A-Z]
[[:space:]]   # Whitespace (space, tab, newline)
[[:blank:]]   # Space and tab only
[[:punct:]]   # Punctuation
[[:xdigit:]]  # Hexadecimal [0-9A-Fa-f]
[[:print:]]   # Printable characters
[[:graph:]]   # Visible characters (no space)
```

### Quantifiers Deep Dive

```bash
# BRE syntax (requires backslashes)
\{n\}      # Exactly n occurrences
\{n,\}     # At least n occurrences
\{n,m\}    # Between n and m occurrences
\+         # One or more (same as \{1,\})
\?         # Zero or one (same as \{0,1\})

# ERE syntax (no backslashes needed)
{n}        # Exactly n occurrences
{n,}       # At least n occurrences
{n,m}      # Between n and m occurrences
+          # One or more
?          # Zero or one
```

***

## grep - Pattern Searching Master Class

### Essential grep Flags

```bash
-i          # Case-insensitive
-v          # Invert match (show non-matching lines)
-w          # Match whole words only
-x          # Match whole lines only
-c          # Count matching lines
-n          # Show line numbers
-l          # Show only filenames with matches
-L          # Show only filenames WITHOUT matches
-r/-R       # Recursive search
-h          # Suppress filename in output
-H          # Force filename in output
-o          # Show only matched part
-A NUM      # Show NUM lines After match
-B NUM      # Show NUM lines Before match
-C NUM      # Show NUM lines of Context (before and after)
-E          # Use ERE (same as egrep)
-P          # Use PCRE (Perl regex)
-F          # Fixed strings (no regex, fastest)
--color=auto # Highlight matches
-q          # Quiet mode (exit status only)
-s          # Suppress error messages
```

### Anchors and Boundaries

```bash
# Line anchors
grep '^root' /etc/passwd              # Lines starting with 'root'
grep 'bash$' /etc/passwd              # Lines ending with 'bash'
grep '^$' file.txt                    # Empty lines
grep '^[[:space:]]*$' file.txt        # Blank lines (whitespace only)

# Word boundaries (use -w or manual)
grep -w 'cat' file.txt                # Match 'cat' but not 'concatenate'
grep '\bcat\b' file.txt               # Same using word boundary
grep '\<cat\>' file.txt               # Alternative word boundary syntax
```

### Character Classes and Ranges

```bash
# Basic character classes
grep '[aeiou]' file.txt               # Any vowel
grep '[^aeiou]' file.txt              # Any non-vowel
grep '[0-9]' file.txt                 # Any digit
grep '[a-zA-Z]' file.txt              # Any letter
grep '[A-Z][a-z]*' file.txt           # Words starting with uppercase

# POSIX classes (recommended for portability)
grep '[[:digit:]]' file.txt           # Any digit
grep '[[:alnum:]]' file.txt           # Alphanumeric
grep '^[[:upper:]]' file.txt          # Lines starting with uppercase
grep '[[:space:]]' file.txt           # Whitespace

# Multiple ranges
grep '[0-9A-F]' file.txt              # Hex digit
grep '[a-zA-Z0-9_-]' file.txt         # Alphanumeric plus underscore/hyphen
```

### Quantifiers with grep (BRE)

```bash
# Asterisk (zero or more)
grep 'ab*c' file.txt                  # Matches: ac, abc, abbc, abbbc
grep '^[[:space:]]*' file.txt         # Leading whitespace
grep 'error.*critical' file.txt       # 'error' followed by 'critical'

# Plus (one or more) - BRE requires backslash
grep 'ab\+c' file.txt                 # Matches: abc, abbc, abbbc (NOT ac)
grep '[0-9]\+' file.txt               # One or more digits

# Question mark (zero or one) - BRE requires backslash
grep 'colou\?r' file.txt              # Matches: color, colour

# Specific counts
grep '^[0-9]\{3\}' file.txt           # Lines starting with exactly 3 digits
grep '[0-9]\{2,4\}' file.txt          # 2 to 4 digits
grep '^.\{80,\}' file.txt             # Lines with 80+ characters
grep -E '^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$' file.txt  # IP-like pattern
```

### Alternation (OR Logic) - BRE

```bash
# BRE requires escaped pipe
grep 'fatal\|error\|critical' /var/log/messages
grep '\(jpg\|png\|gif\)$' file.txt    # Image extensions

# Multiple pattern matching
grep -e 'pattern1' -e 'pattern2' file.txt
grep 'pattern1' file.txt | grep 'pattern2'  # AND logic via pipe
```

### Practical grep Patterns

```bash
# Email validation (basic)
grep -E '\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b' file.txt

# IPv4 address (simplified)
grep -E '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' file.txt

# Date patterns (YYYY-MM-DD)
grep -E '[0-9]{4}-[0-9]{2}-[0-9]{2}' file.txt

# URLs
grep -E 'https?://[a-zA-Z0-9./?=_%:-]*' file.txt

# MAC addresses
grep -E '([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}' file.txt

# Phone numbers (US format)
grep -E '\(?[0-9]{3}\)?[-. ]?[0-9]{3}[-. ]?[0-9]{4}' file.txt

# Credit card (generic pattern - 4 groups of 4 digits)
grep -E '[0-9]{4}[[:space:]-]?[0-9]{4}[[:space:]-]?[0-9]{4}[[:space:]-]?[0-9]{4}' file.txt

# Log levels
grep -E '(DEBUG|INFO|WARN|ERROR|FATAL)' app.log

# Extract function definitions (Python/Bash)
grep -E '^def [a-zA-Z_][a-zA-Z0-9_]*\(' file.py
grep -E '^function [a-zA-Z_][a-zA-Z0-9_]*' file.sh

# Find commented lines
grep '^[[:space:]]*#' script.sh
grep '^[[:space:]]*//' script.js
```

### Advanced grep Combinations

```bash
# Recursive search with file filtering
grep -r --include="*.conf" 'max_connections' /etc/

# Exclude directories
grep -r --exclude-dir={.git,node_modules} 'TODO' .

# Context display
grep -C 3 'error' /var/log/messages     # 3 lines before and after

# Count occurrences per file
grep -c 'error' *.log

# List files with/without match
grep -rl 'pattern' /path/               # Files with pattern
grep -rL 'pattern' /path/               # Files without pattern

# Show only matched portions
grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' access.log

# Case-insensitive whole-word match
grep -iw 'error' file.txt

# Multiple patterns with line numbers
grep -nE 'pattern1|pattern2|pattern3' file.txt

# Invert match to exclude patterns
grep -v '^#' /etc/ssh/sshd_config | grep -v '^$'  # Remove comments and blank lines
```

***

## egrep / grep -E - Extended Regular Expressions

The `egrep` command is deprecated but equivalent to `grep -E` [][]. ERE simplifies syntax by treating metacharacters as special by default (no escaping needed) [][].

### Key ERE Advantages

```bash
# Alternation (no backslash needed)
grep -E 'error|warning|critical' /var/log/messages
egrep 'jpg|png|gif|svg' filelist.txt

# Grouping (no backslash needed)
grep -E '(failed|error) login' /var/log/secure
grep -E '^(http|https|ftp)://' urls.txt

# Quantifiers (no backslash needed)
grep -E '[0-9]+' file.txt             # One or more digits
grep -E 'colou?r' file.txt            # Optional 'u'
grep -E '^.{50,}$' file.txt           # Lines with 50+ characters
```

### Advanced ERE Patterns

```bash
# Complex alternation with grouping
grep -E '(fatal|error|critical).*(database|connection|timeout)' app.log

# Multiple quantifiers
grep -E '^[A-Z][a-z]+[[:space:]]+[A-Z][a-z]+$' file.txt  # First Last name format

# Optional groups
grep -E 'Jan(uary)?' dates.txt        # Matches: Jan, January

# Nested groups
grep -E '((25[0-5]|2[0-4][0-9]| ?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]| ?[0-9][0-9]?)' ips.txt  # Accurate IP validation

# Username validation (8-20 chars, alphanumeric + underscore)
grep -E '^[a-zA-Z][a-zA-Z0-9_]{7,19}$' usernames.txt

# Password strength (min 8 chars, 1 upper, 1 lower, 1 digit)
grep -E '^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9]).{8,}$' passwords.txt  # Requires grep -P

# Extract all words
grep -oE '\b[a-zA-Z]+\b' file.txt

# Find duplicated words
grep -E '\b([a-zA-Z]+)[[:space:]]+\1\b' file.txt

# Version numbers
grep -E '[0-9]+\.[0-9]+\.[0-9]+' changelog.txt

# HTML/XML tags
grep -E '<[^>]+>' file.html

# Markdown headers
grep -E '^#{1,6}[[:space:]]' README.md
```

### Process Filtering with ERE

```bash
# Find Apache or Nginx processes
ps aux | grep -E 'apache|nginx|httpd'

# Filter by multiple users
ps aux | grep -E '^(root|apache|mysql)'

# Network connections
netstat -tulpn | grep -E ':(80|443|22|3306)[[:space:]]'
ss -tulpn | grep -E ':(80|443|22|3306)'

# System logs for multiple services
journalctl | grep -E 'sshd|systemd|kernel'

# Find large files (combine with find)
find /var -type f -size +100M | grep -E '\.(log|tar|zip)$'
```

### ERE One-Liners for Sysadmins

```bash
# Extract all IPv4 addresses from a file
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' /var/log/secure

# Find failed SSH login attempts
grep -E 'Failed password.*([0-9]{1,3}\.){3}[0-9]{1,3}' /var/log/secure

# Monitor multiple log levels in real-time
tail -f /var/log/app.log | grep -E --color '(ERROR|FATAL|CRITICAL)'

# Extract email addresses
grep -oE '\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b' contacts.txt

# Find uncommented config lines
grep -vE '^[[:space:]]*(#|;|$)' /etc/config.conf

# Validate date formats (YYYY-MM-DD)
grep -E '^(19|20)[0-9]{2}-(0[1-9]|1 )-(0[1-9]| [0-9]|3 )$' dates.txt

# Find lines with specific word count
grep -E '^([^[:space:]]+[[:space:]]+){5,}' file.txt  # Lines with 5+ words
```

***

## sed - Stream Editor with Regex

The `sed` command performs text transformations using regular expressions [][]. By default, `sed` uses BRE; use `-E` or `-r` for ERE [][].

### sed Syntax Fundamentals

```bash
# Basic syntax
sed 's/pattern/replacement/' file.txt          # Substitute first occurrence
sed 's/pattern/replacement/g' file.txt         # Global (all occurrences)
sed 's/pattern/replacement/gi' file.txt        # Global + case-insensitive
sed -i 's/pattern/replacement/g' file.txt      # In-place edit
sed -i.bak 's/pattern/replacement/g' file.txt  # In-place with backup

# Address ranges
sed '3s/old/new/' file.txt                     # Line 3 only
sed '1,5s/old/new/' file.txt                   # Lines 1-5
sed '10,$s/old/new/' file.txt                  # Line 10 to end
sed '/pattern/s/old/new/' file.txt             # Lines matching pattern
sed '/start/,/end/s/old/new/' file.txt         # Range between patterns
```

### sed Flags Explained

```bash
g    # Global - replace all occurrences on each line
i    # Case-insensitive matching
p    # Print matched lines (use with -n)
w    # Write matched lines to file
e    # Execute replacement as shell command
Ng   # Replace Nth occurrence (e.g., 2g for 2nd onwards)
```

### Search and Replace with Regex

```bash
# Basic substitution
sed 's/foo/bar/' file.txt                      # Replace first 'foo' per line
sed 's/foo/bar/g' file.txt                     # Replace all 'foo'
sed 's/foo/bar/2' file.txt                     # Replace 2nd occurrence only
sed 's/foo/bar/2g' file.txt                    # Replace 2nd occurrence onwards

# Case-insensitive
sed 's/error/ERROR/gi' file.txt

# Using delimiters (when pattern contains /)
sed 's|/usr/local|/opt|g' file.txt
sed 's#/var/log#/logs#g' file.txt
sed 's@http://old.com@https://new.com@g' file.txt

# Multiple substitutions
sed 's/foo/bar/g; s/baz/qux/g' file.txt
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt
```

### Back-References and Capture Groups

```bash
# BRE back-references (require \( \) and \1, \2, etc.)
sed 's/\([0-9]\{3\}\)-\([0-9]\{4\}\)/(\1) \2/g' file.txt  # 555-1234 → (555) 1234

# ERE back-references (use -E or -r)
sed -E 's/([0-9]{3})-([0-9]{4})/(\1) \2/g' file.txt

# Swap two words
sed -E 's/^([a-zA-Z]+)[[:space:]]+([a-zA-Z]+)$/\2 \1/' names.txt  # First Last → Last First

# Duplicate detection
sed -E 's/\b([a-zA-Z]+)[[:space:]]+\1\b/\1/g' file.txt  # Remove duplicate words

# Extract and rearrange
sed -E 's/^([0-9]+),([a-zA-Z]+),(.*)$/\2-\1: \3/' data.csv

# Add quotes around numbers
sed -E 's/([0-9]+)/"&"/g' file.txt                # & represents entire match

# Wrap URLs in HTML
sed -E 's|(https?://[^[:space:]]+)|<a href="\1">\1</a>|g' file.txt
```

### Advanced sed Patterns

```bash
# Delete operations
sed '/pattern/d' file.txt                       # Delete lines matching pattern
sed '/^$/d' file.txt                            # Delete empty lines
sed '/^[[:space:]]*$/d' file.txt                # Delete blank lines
sed '1,10d' file.txt                            # Delete lines 1-10
sed '/start/,/end/d' file.txt                   # Delete range between patterns

# Insertion and appending
sed '3i\New line before line 3' file.txt        # Insert before line 3
sed '3a\New line after line 3' file.txt         # Append after line 3
sed '/pattern/i\Inserted text' file.txt         # Insert before pattern
sed '/pattern/a\Appended text' file.txt         # Append after pattern

# Change (replace) entire line
sed '/pattern/c\Replacement line' file.txt      # Replace matching lines

# Print specific lines
sed -n '5p' file.txt                            # Print line 5 only
sed -n '10,20p' file.txt                        # Print lines 10-20
sed -n '/pattern/p' file.txt                    # Print matching lines (like grep)

# Transform characters
sed 'y/abc/ABC/' file.txt                       # Transliterate (like tr)
```

### Real-World sed Examples

```bash
# Remove comments and blank lines
sed -e 's/#.*//' -e '/^$/d' config.conf

# Add prefix to all lines
sed 's/^/PREFIX: /' file.txt

# Add suffix to all lines
sed 's/$/ SUFFIX/' file.txt

# Number non-empty lines
sed '/./=' file.txt | sed 'N; s/\n/. /'

# Double-space a file
sed G file.txt

# Remove leading whitespace
sed 's/^[[:space:]]*//' file.txt

# Remove trailing whitespace
sed 's/[[:space:]]*$//' file.txt

# Remove both leading and trailing whitespace
sed 's/^[[:space:]]*//; s/[[:space:]]*$//' file.txt

# Convert DOS to Unix line endings
sed 's/\r$//' dos_file.txt > unix_file.txt

# Extract specific fields from CSV
sed -E 's/^([^,]*),([^,]*),.*$/\1 - \2/' data.csv

# Comment out lines matching pattern
sed '/pattern/s/^/# /' file.txt

# Uncomment lines
sed 's/^[[:space:]]*#//' file.txt

# Replace multiple spaces with single space
sed 's/[[:space:]]\+/ /g' file.txt

# Capitalize first letter of each word
sed -E 's/\b([a-z])/\U\1/g' file.txt

# Convert to lowercase
sed 's/.*/\L&/' file.txt

# Convert to uppercase
sed 's/.*/\U&/' file.txt

# Extract IP addresses
sed -En 's/.*([0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}).*/\1/p' file.txt

# Log file timestamp extraction
sed -En 's/^\[([0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2})\].*/\1/p' app.log

# HTML tag stripping
sed -E 's/<[^>]+>//g' file.html

# URL parameter extraction
sed -En 's/.*[?&]id=([0-9]+).*/\1/p' urls.txt
```

### sed Multi-Line Patterns

```bash
# Join lines with specific pattern
sed ':a;N;$!ba;s/\\\n//g' file.txt              # Join lines ending with backslash

# Replace newlines with spaces
sed ':a;N;$!ba;s/\n/ /g' file.txt

# Delete blank lines following a pattern
sed '/pattern/{N;s/\n$//}' file.txt

# Insert blank line before pattern
sed '/pattern/i\\' file.txt

# Insert blank line after pattern
sed '/pattern/a\\' file.txt
```

### sed In-Place Editing Best Practices

```bash
# Always create backup first
sed -i.orig 's/old/new/g' file.txt

# Test before in-place edit
sed 's/old/new/g' file.txt | less
sed 's/old/new/g' file.txt > test_output.txt

# Batch processing with find
find /etc -name "*.conf" -exec sed -i.bak 's/old/new/g' {} \;

# Safe in-place with GNU sed
sed -i'.bak' 's/old/new/g' file.txt             # GNU syntax
sed -i .bak 's/old/new/g' file.txt              # BSD syntax (note space)
```

### sed Configuration Management Examples

```bash
# Update configuration values
sed -i 's/^max_connections.*/max_connections = 500/' /etc/config.conf

# Enable/disable options
sed -i 's/^#\(PermitRootLogin\)/\1/' /etc/ssh/sshd_config  # Uncomment
sed -i 's/^\(PermitRootLogin\)/#\1/' /etc/ssh/sshd_config  # Comment

# Replace variables in template
sed "s/{{HOSTNAME}}/$HOSTNAME/g; s/{{IP}}/$IP/g" template.conf > app.conf

# Version bumping
sed -i -E 's/(version[[:space:]]*=[[:space:]]*")([0-9]+\.[0-9]+\.)([0-9]+)"/echo "\1\2$((\3+1))"/ge' version.txt

# Log rotation simulation (keep last N lines)
sed -i -e :a -e '$q;N;11,$D;ba' logfile.txt     # Keep last 10 lines

# Sanitize logs (remove sensitive data)
sed -E 's/password=[^&[:space:]]*/password=REDACTED/gi' access.log
```

***

## Pro Tips and Performance

### Regex Performance Optimization

```bash
# Use fixed strings when possible (100x faster)
grep -F 'literal_string' file.txt              # No regex processing
fgrep 'literal_string' file.txt                # Same as grep -F

# Anchor patterns when possible
grep '^pattern' file.txt                       # Faster than 'pattern' alone

# Use simpler patterns
grep 'error' file.txt                          # Better than grep '.*error.*'

# Avoid greedy quantifiers when unnecessary
sed 's/start.*end/REPLACE/'                    # Greedy (matches longest)
sed -E 's/start[^e]*end/REPLACE/'              # More specific

# Combine operations
sed 's/foo/bar/g; s/baz/qux/g' file.txt        # Better than two sed commands
```

### Debugging Regex

```bash
# Test patterns interactively
echo "test string" | grep -E 'pattern'

# Show only matched portions
grep -oE 'pattern' file.txt

# Use verbose PCRE mode
grep -P '(?x)  # Comments allowed
  ^            # Start of line
  [0-9]{3}     # Three digits
  -            # Dash
  [0-9]{4}     # Four digits
  $            # End of line
' file.txt

# Highlight matches
grep --color=always 'pattern' file.txt | less -R
```

### Common Pitfalls to Avoid

```bash
# WRONG: Forgetting to escape in BRE
grep 'test+' file.txt                          # Matches 'test+'
grep 'test\+' file.txt                         # Matches 'testt', 'testtt'

# WRONG: Unescaped dots
grep '192.168.1.1' file.txt                    # Matches '192x168x1x1'
grep '192\.168\.1\.1' file.txt                 # Correct

# WRONG: Greedy matching
sed 's/<.*>/REMOVED/'                          # Removes from first < to last >
sed 's/<[^>]*>/REMOVED/g'                      # Removes each tag individually

# WRONG: Not testing before in-place edit
sed -i 's/old/new/' important.txt              # Dangerous
sed 's/old/new/' important.txt > /dev/null && sed -i 's/old/new/' important.txt  # Safer
```