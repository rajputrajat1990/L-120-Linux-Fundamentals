# Linux Text Processing

## Searching Inside Files

### grep - Core Pattern Matching
```bash
# Basic search
grep "pattern" file.txt                    # Find exact string
grep -i "pattern" file.txt                 # Case-insensitive search
grep -n "pattern" file.txt                 # Show line numbers
grep -c "pattern" file.txt                 # Count matches only

# Recursive & multi-file searches
grep -r "pattern" /path/to/dir            # Recursive through directories
grep -R "pattern" /etc                    # Follow symbolic links too
grep "pattern" *.txt                      # Search multiple files
grep -l "pattern" *.log                   # List only filenames with matches
grep -L "pattern" *.log                   # List files WITHOUT matches

# Context control (crucial for debugging)
grep -A 3 "error" app.log                 # Show 3 lines After match
grep -B 2 "error" app.log                 # Show 2 lines Before match
grep -C 2 "error" app.log                 # Show 2 lines of Context (before+after)

# Invert & exclude
grep -v "debug" app.log                   # Show lines NOT matching
grep -v -e "debug" -e "info" app.log      # Exclude multiple patterns

# Extended regex (use -E or egrep)
grep -E "error|warning|fatal" app.log     # OR operator
grep -E "^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" access.log  # IP addresses
grep -E "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b" data.txt # Email addresses

# Advanced patterns
grep -w "error" app.log                   # Match whole words only
grep "^Error" file.txt                    # Lines starting with "Error"
grep "end$" file.txt                      # Lines ending with "end"
grep -o "pattern" file.txt                # Show only matched text, not whole line

# Performance & compressed files
grep --include="*.java" -r "class" src/   # Search only specific file types
grep --exclude-dir=".git" -r "TODO" .     # Exclude directories
zgrep "error" /var/log/messages.*.gz      # Search compressed logs without extraction
```

 The `-r` flag enables recursive searching through directory trees, while `-E` activates extended regular expressions for advanced pattern matching. For compressed log analysis, `zgrep` operates on gzipped files without requiring decompression.     

## The Streaming Editor (sed)

### sed - In-Stream Text Transformation
```bash
# Substitution (s command) - most common operation
sed 's/old/new/' file.txt                 # Replace first occurrence per line
sed 's/old/new/g' file.txt                # Replace all occurrences (global)
sed 's/old/new/2' file.txt                # Replace only 2nd occurrence per line
sed 's/old/new/gi' file.txt               # Global + case insensitive

# In-place editing (DANGEROUS - always backup first)
sed -i.bak 's/old/new/g' file.txt         # Edit file directly, create .bak backup
sed -i 's/timeout=60/timeout=30/g' *.conf # Batch edit multiple files

# Line addressing
sed '5s/old/new/' file.txt                # Replace only on line 5
sed '10,20s/old/new/g' file.txt           # Replace on lines 10-20
sed '/pattern/s/old/new/g' file.txt       # Replace only on lines matching pattern
sed '1,/pattern/s/old/new/g' file.txt     # From line 1 until first pattern match

# Deletion (d command)
sed '1d' file.txt                         # Delete first line
sed '$d' file.txt                         # Delete last line
sed '5,10d' file.txt                      # Delete lines 5-10
sed '/^$/d' file.txt                      # Delete empty lines
sed '/debug/d' app.log                    # Delete lines containing "debug"
sed '/^#/d' config.conf                   # Delete comment lines

# Insertion & appending
sed '5i\New line here' file.txt           # Insert before line 5
sed '5a\New line here' file.txt           # Append after line 5
sed '$a\Footer text' file.txt             # Append to end of file
sed 's/^/Prefix: /' file.txt              # Add prefix to every line
sed 's/$/ [DONE]/' file.txt               # Add suffix to every line

# Extended regex (-r or -E)
sed -r 's/([0-9]{4})-([0-9]{2})-([0-9]{2})/\3\/\2\/\1/' dates.txt  # Date format conversion
sed -r 's/(error|warning|critical)/[\1]/gi' app.log                 # Wrap keywords in brackets
sed -r 's/^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9:]{8} //g' access.log    # Strip timestamps

# Multiple operations
sed -e 's/old/new/g' -e 's/foo/bar/g' file.txt    # Chain operations
sed '1d; s/old/new/g; /debug/d' file.txt           # Delete line 1, substitute, remove debug lines

# Print control (-n with p)
sed -n '10,20p' file.txt                  # Print only lines 10-20
sed -n '/error/p' app.log                 # Print only lines matching pattern (like grep)
sed -n '/start/,/end/p' file.txt          # Print between patterns (inclusive)

# Practical transformations
sed 's/\t/,/g' data.tsv                   # Convert tabs to commas (TSV to CSV)
sed 's/  */ /g' file.txt                  # Collapse multiple spaces to single space
sed 's/\r$//' dos_file.txt                # Remove Windows line endings (DOS to UNIX)
```

 The `-i` option enables direct file editing without creating intermediate files, while `-r` activates extended regular expressions for capturing groups and advanced patterns. The `sed` command excels at stream editing, reading input, transforming it based on specified rules, and writing output.  

## Text Processing with Awk

### awk - Field-Based Data Processing
```bash
# Basic field extraction ($1, $2, $NF are field variables)
awk '{print $1}' file.txt                 # Print first field
awk '{print $2, $5}' data.txt             # Print 2nd and 5th fields
awk '{print $NF}' file.txt                # Print last field ($NF = Number of Fields)
awk '{print $(NF-1)}' file.txt            # Print second-to-last field
awk '{print $0}' file.txt                 # Print entire line

# Custom field separators (-F)
awk -F':' '{print $1, $3}' /etc/passwd    # Use colon as delimiter (users & UIDs)
awk -F',' '{print $2}' data.csv           # Parse CSV files
awk -F'=' '{print $2}' config.ini         # Parse key=value configs
awk -F'[,:]' '{print $1, $3}' mixed.txt   # Multiple delimiters (comma OR colon)

# Pattern matching (condition before action)
awk '/error/' file.txt                    # Print lines containing "error"
awk '/^[0-9]/' file.txt                   # Lines starting with digit
awk '!/debug/' app.log                    # Lines NOT containing "debug"
awk '/error|warning/' app.log             # Lines with "error" OR "warning"

# Conditional processing (if statements)
awk '$3 > 100' data.txt                   # Print lines where 3rd field > 100
awk '$1 == "ERROR"' app.log               # Exact match on first field
awk '$2 >= 50 && $2 <= 100' data.txt      # Range filtering
awk 'length($0) > 80' file.txt            # Lines longer than 80 characters
awk 'NF > 5' file.txt                     # Lines with more than 5 fields
awk 'NR > 1' file.txt                     # Skip header (row number > 1)

# Arithmetic & aggregation
awk '{sum += $3} END {print sum}' data.txt              # Sum 3rd column
awk '{sum += $2; count++} END {print sum/count}' data.txt  # Calculate average
awk '{total += $3} END {print "Total:", total}' sales.txt  # Running total
awk '$3 > max {max = $3} END {print max}' data.txt      # Find maximum value
awk '{min = (NR==1 || $2<min) ? $2 : min} END {print min}' data.txt  # Find minimum

# BEGIN and END blocks
awk 'BEGIN {print "Report Started"} {print $1} END {print "Report Finished"}' data.txt
awk 'BEGIN {FS=","; OFS="|"} {print $1, $2, $3}' data.csv  # Change input/output separators
awk 'BEGIN {print "Name,Count"} {print $1 "," $2}' data.txt  # Add CSV header

# Formatted output (printf)
awk '{printf "%-10s %5d\n", $1, $2}' data.txt           # Left-aligned string, right-aligned number
awk '{printf "Name: %s, Value: %d\n", $1, $2}' data.txt # Formatted strings
awk '{printf "%.2f\n", $3}' data.txt                    # Two decimal places

# Associative arrays (hash maps)
awk '{count[$1]++} END {for (key in count) print key, count[key]}' data.txt  # Frequency count
awk '{sum[$1] += $2} END {for (key in sum) print key, sum[key]}' data.txt    # Group and sum
awk '{ip[$NF]++} END {for (i in ip) print ip[i], i}' access.log | sort -nr   # Top IPs from logs

# Built-in variables
awk '{print NR, $0}' file.txt             # NR = record (line) number (add line numbers)
awk '{print NF, $0}' file.txt             # NF = number of fields per line
awk '{print FILENAME, $0}' *.txt          # FILENAME = current file being processed
awk 'BEGIN {print "Fields:", "Lines:"; FS=","} {fields+=NF; lines++} END {print fields, lines}' data.csv

# Practical real-world examples
awk -F':' '$3 >= 1000 {print $1}' /etc/passwd           # List non-system users
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10  # Top 10 IPs
awk '/ERROR/ {err++} /WARN/ {warn++} END {print "Errors:", err, "Warnings:", warn}' app.log
awk '{gsub(/[^0-9]/,""); print}' file.txt               # Extract only digits
```

 Awk excels at field-based processing, using variables like `$1`, `$2`, and `$NF` to access specific columns, while `NR` and `NF` provide line and field counts. The `-F` flag sets custom field separators for parsing structured data like CSV or configuration files.  

## Replacing Text Characters

### tr - Character Translation & Deletion
```bash
# Case conversion
tr 'a-z' 'A-Z' < input.txt                # Convert lowercase to uppercase
tr 'A-Z' 'a-z' < input.txt                # Convert uppercase to lowercase
cat file.txt | tr '[:lower:]' '[:upper:]'  # POSIX character classes

# Character replacement
tr ' ' '_' < file.txt                     # Replace spaces with underscores
tr ':' ',' < data.txt                     # Replace colons with commas
tr '\t' ',' < data.tsv > data.csv         # Convert TSV to CSV

# Character deletion (-d)
tr -d '\r' < dos_file.txt > unix_file.txt  # Remove carriage returns
tr -d '[:punct:]' < file.txt              # Remove all punctuation
tr -d '0-9' < file.txt                    # Remove all digits
tr -d ' ' < file.txt                      # Remove all spaces

# Squeeze repeated characters (-s)
tr -s ' ' < file.txt                      # Collapse multiple spaces to single space
tr -s '\n' < file.txt                     # Remove empty lines
tr -s '/' < path.txt                      # Collapse repeated slashes in paths

# Complement (-c) - operate on everything EXCEPT specified
tr -cd '0-9\n' < file.txt                 # Keep only digits and newlines, delete everything else
tr -cd 'A-Za-z0-9 \n' < file.txt          # Keep only alphanumeric, space, newline

# Practical combinations
echo "Hello   World" | tr -s ' '          # Output: "Hello World"
cat file.txt | tr -d '\r' | tr -s '\n'    # DOS to UNIX + remove blank lines
echo "abc123xyz" | tr -cd '[:digit:]'     # Extract: "123"
cat data.txt | tr ',' '\t' | awk '{print $1}'  # CSV to TSV then extract first field
```

### Other character replacement tools
```bash
# expand/unexpand - tab/space conversion
expand -t 4 file.txt                      # Convert tabs to 4 spaces
unexpand -t 4 file.txt                    # Convert 4 spaces to tabs

# iconv - character encoding conversion
iconv -f UTF-8 -t ISO-8859-1 input.txt > output.txt     # UTF-8 to Latin1
iconv -f ISO-8859-1 -t UTF-8 input.txt > output.txt     # Latin1 to UTF-8

# dos2unix / unix2dos - line ending conversion (install via: yum install dos2unix)
dos2unix file.txt                         # Convert DOS/Windows to UNIX line endings
unix2dos file.txt                         # Convert UNIX to DOS/Windows line endings
```

 The `tr` command performs character-level transformations including case conversion, deletion, and squeezing repeated characters, while `sed` handles more complex string-level substitutions.  

### Pro-Level Command Pipelines
```bash
# Extract unique email domains from logs, count, and sort
grep -Eo "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b" app.log | awk -F'@' '{print $2}' | sort | uniq -c | sort -nr

# Parse Apache access log: extract IPs, count requests, show top 10
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10

# Clean CSV: remove quotes, convert to tab-delimited, extract columns
sed 's/"//g' data.csv | tr ',' '\t' | awk '{print $1, $3, $5}'

# Find and replace across multiple files with backup
find . -name "*.conf" -exec sed -i.bak 's/timeout=60/timeout=30/g' {} \;

# Extract failed SSH attempts with IP addresses
grep "Failed password" /var/log/secure | grep -Eo "([0-9]{1,3}\.){3}[0-9]{1,3}" | sort | uniq -c | sort -nr
```

## Text Sorting

### sort - Ordering & Arranging Data
```bash
# Basic sorting
sort file.txt                             # Alphabetical sort (default)
sort -r file.txt                          # Reverse order
sort -u file.txt                          # Sort and remove duplicates (like sort | uniq)
sort -o output.txt input.txt              # Write to file (can overwrite input safely)

# Numerical sorting (CRITICAL for numbers)
sort -n numbers.txt                       # Numeric sort (10 comes after 9, not after 1)
sort -h file.txt                          # Human-readable numbers (1K, 2M, 3G)
sort -g file.txt                          # General numeric (handles scientific notation)
sort -V versions.txt                      # Version number sort (v1.2.10 > v1.2.9)

# Column-based sorting (-k for key field)
sort -k2 file.txt                         # Sort by 2nd column (whitespace delimiter)
sort -k2,2 file.txt                       # Sort ONLY by 2nd column (ignore others)
sort -k2n file.txt                        # Sort by 2nd column numerically
sort -k2,2n -k3,3r file.txt               # Sort by 2nd col (numeric), then 3rd col (reverse)
sort -k1,1 -k2,2n data.txt                # Multi-key: alphabetic then numeric

# Custom delimiters
sort -t: -k3n /etc/passwd                 # Use colon delimiter, sort by 3rd field (UID)
sort -t',' -k2 data.csv                   # CSV: sort by 2nd column
sort -t$'\t' -k1 file.tsv                 # Tab-delimited files

# Character position sorting
sort -k1.5 file.txt                       # Sort starting from 5th character of 1st field
sort -k2.1,2.3 file.txt                   # Sort by characters 1-3 of 2nd field

# Case handling
sort -f file.txt                          # Case-insensitive (fold case)
sort -d file.txt                          # Dictionary order (ignore non-alphanumeric)
sort -i file.txt                          # Ignore non-printable characters

# Advanced options
sort -b file.txt                          # Ignore leading blanks
sort -s file.txt                          # Stable sort (preserve original order for equal keys)
sort -R file.txt                          # Random shuffle
sort --parallel=4 large_file.txt          # Use 4 CPU cores for large files
sort -S 2G large_file.txt                 # Use 2GB of memory buffer

# Check if already sorted
sort -c file.txt                          # Check if sorted, exit with error if not
sort -C file.txt                          # Silent check (only exit code)

# Real-world examples
sort -t: -k3n /etc/passwd | head -10      # Show users with lowest UIDs
sort -k5rn access.log | head -20          # Show largest file transfers (5th col, reverse numeric)
du -h | sort -hr | head -20               # Top 20 disk space users
ps aux | sort -nrk 3 | head -5            # Top 5 CPU-consuming processes
netstat -tuln | sort -k4                  # Sort network connections by address

# Merge sorted files (fast for pre-sorted data)
sort -m sorted1.txt sorted2.txt           # Merge already-sorted files (no re-sorting)

# Unique count workflow
sort file.txt | uniq -c | sort -rn        # Frequency count sorted by occurrence
```

 The `-k` option specifies sort keys by column position, with `-n` for numeric sorting to prevent lexical ordering issues where "10" would precede "2". The `-t` flag defines custom delimiters for structured data like CSV or `/etc/passwd` files.   

## Duplicate Removal Utility

### uniq - Filtering Adjacent Duplicates
```bash
# CRITICAL: uniq only works on ADJACENT lines - always sort first!
sort file.txt | uniq                      # Remove adjacent duplicates
uniq file.txt                             # Remove duplicates (only if already sorted)

# Counting duplicates (-c)
sort file.txt | uniq -c                   # Count occurrences of each unique line
sort file.txt | uniq -c | sort -rn        # Most frequent items first
sort access.log | uniq -c | sort -rn | head -10  # Top 10 most common log entries

# Show only duplicates (-d) or uniques (-u)
sort file.txt | uniq -d                   # Show only lines that ARE duplicated
sort file.txt | uniq -u                   # Show only lines that appear ONCE
sort file.txt | uniq -D                   # Show ALL occurrences of duplicated lines

# Case-insensitive comparison
sort -f file.txt | uniq -i                # Ignore case when comparing

# Field and character selection
uniq -f 1 file.txt                        # Skip first field, compare from 2nd field onward
uniq -s 5 file.txt                        # Skip first 5 characters of each line
uniq -w 10 file.txt                       # Compare only first 10 characters

# Show duplicate groups with separation
sort file.txt | uniq --all-repeated=separate    # Group duplicates with blank line separator
sort file.txt | uniq --all-repeated=prepend     # Add blank line before each group

# Practical real-world examples
# Find duplicate IPs in access log
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20

# Find users with duplicate UIDs (security issue!)
sort -t: -k3 /etc/passwd | uniq -d -f 2

# Find duplicate email addresses
grep -Eo "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" emails.txt | sort -f | uniq -di

# Count unique error types in log
grep ERROR app.log | sort | uniq -c | sort -rn

# Find files with same content (using checksums)
find . -type f -exec md5sum {} \; | sort | uniq -w32 -D --all-repeated=separate
```

 The `uniq` command operates only on adjacent duplicate lines, requiring `sort` preprocessing for comprehensive duplicate detection. The `-c` option counts occurrences, `-d` shows only duplicates, and `-u` displays unique lines.   

## Extracting Columns of Text

### cut - Column & Character Extraction
```bash
# Field extraction (-f with delimiter)
cut -f1 file.txt                          # Extract 1st field (tab delimiter default)
cut -f2,5 file.txt                        # Extract 2nd and 5th fields
cut -f1-3 file.txt                        # Extract fields 1 through 3
cut -f3- file.txt                         # Extract from 3rd field to end
cut -f-4 file.txt                         # Extract first 4 fields

# Custom delimiters (-d)
cut -d: -f1,3 /etc/passwd                 # Extract username and UID (colon delimiter)
cut -d',' -f2,4 data.csv                  # Extract columns from CSV
cut -d' ' -f1 file.txt                    # Space delimiter
cut -d$'\t' -f2 file.tsv                  # Tab delimiter (explicit)

# Character/byte position extraction (-c or -b)
cut -c1-10 file.txt                       # First 10 characters of each line
cut -c1,5,9 file.txt                      # Characters at positions 1, 5, and 9
cut -c5- file.txt                         # From 5th character to end
cut -c-20 file.txt                        # First 20 characters
cut -b1-100 file.txt                      # First 100 bytes (for binary data)

# Output delimiter control (--output-delimiter)
cut -d: -f1,3 /etc/passwd --output-delimiter=','   # Change output delimiter to comma
cut -d'|' -f1,3,5 --output-delimiter=$'\t' data.txt # Pipe to tab conversion

# Complement (inverse selection)
cut --complement -f2 file.txt             # All fields EXCEPT 2nd
cut --complement -c1-5 file.txt           # All characters EXCEPT first 5

# Practical examples
# Extract usernames only
cut -d: -f1 /etc/passwd

# Extract IP addresses from Apache log (assuming space-delimited)
cut -d' ' -f1 access.log | sort | uniq -c | sort -rn

# Get file extensions
ls -1 | rev | cut -d. -f1 | rev | sort | uniq -c

# Extract specific columns from CSV
cut -d',' -f2,5,7 data.csv | column -t -s,

# Parse date from timestamp (YYYY-MM-DD from YYYY-MM-DD HH:MM:SS)
cut -d' ' -f1 dates.log

# Extract first 8 chars of commit hash
git log --oneline | cut -c1-8
```

 The `cut` command excels at simple field extraction with `-f` for fields and `-d` for delimiters, while being faster than `awk` for basic column operations. Character-based extraction uses `-c` for specific positions within lines.  

### column - Pretty-Print Columns
```bash
# Basic table formatting
column -t file.txt                        # Auto-detect columns and align
mount | column -t                         # Make mount output readable
cat /etc/passwd | column -t -s:           # Format passwd file as table

# Custom delimiters
column -t -s',' data.csv                  # Format CSV as aligned table
column -t -s$'\t' data.tsv                # Format TSV as aligned table
column -t -s: -o ' | ' /etc/passwd        # Input : delimiter, output | separator

# Hide empty columns
column -t -N name,age,city -H city file.txt  # Hide "city" column

# Specify column names
column -t -N "User,UID,GID,Home" -s: /etc/passwd

# JSON output
column -t -J -N id,name,value file.txt    # Output as JSON

# Truncate columns for width control
column -t -T 2,3 file.txt                 # Truncate 2nd and 3rd columns if too wide

# Fill rows before columns (classic column behavior)
ls | column                               # Multi-column listing
ls | column -c 80                         # Set width to 80 characters
```

 The `column` command transforms unstructured text into aligned tabular format, with `-t` creating tables and `-s` specifying input delimiters. The `-N` option assigns header names to columns for better readability.  

## Merging Multiple Files

### paste - Horizontal File Merging
```bash
# Basic horizontal merge (side-by-side)
paste file1.txt file2.txt                 # Merge files with tab separator
paste file1.txt file2.txt file3.txt       # Merge three files

# Custom delimiter
paste -d',' file1.txt file2.txt           # Use comma as separator (create CSV)
paste -d'|' file1.txt file2.txt           # Use pipe separator
paste -d' ' file1.txt file2.txt           # Use space separator

# Multiple delimiters (cycle through)
paste -d',;:' f1.txt f2.txt f3.txt        # Use , then ; then : then repeat

# Serial mode (-s) - transpose file (rows to columns)
paste -s file.txt                         # Merge all lines into one (tab-separated)
paste -s -d',' file.txt                   # Merge all lines with comma
paste -s -d'\n\t' file1.txt file2.txt     # Separate files by newline, fields by tab

# Reading from stdin
cut -d: -f1 /etc/passwd | paste -d' ' - /etc/group  # Merge stdin with file
cut -d: -f1,3 /etc/passwd | paste -d' ' - -         # Read from stdin twice

# Practical examples
# Create CSV from separate column files
paste -d',' names.txt ages.txt cities.txt > people.csv

# Add line numbers
nl file.txt | paste -d'' - file.txt       # Duplicate with numbers

# Combine multiple cut operations
paste <(cut -d: -f1 /etc/passwd) <(cut -d: -f3 /etc/passwd) <(cut -d: -f5 /etc/passwd)

# Create key-value pairs
paste -d'=' keys.txt values.txt > config.ini

# Transpose data (rows ↔ columns)
paste -s -d'\t' data.txt                  # All rows → single row
```

 The `paste` command performs horizontal merging by outputting lines from multiple files side-by-side with tab delimiters by default. Serial mode (`-s`) merges all lines from a single file into one line.  

### join - Relational File Joining (Like SQL JOIN)
```bash
# CRITICAL: Both files MUST be sorted by join field first!

# Basic join (default: join on 1st field)
join file1.txt file2.txt                  # Inner join on first field

# Specify join fields
join -1 2 -2 1 file1.txt file2.txt        # Join file1's 2nd col with file2's 1st col
join -1 1 -2 3 employees.txt salaries.txt # Join employees col 1 with salaries col 3

# Custom delimiter
join -t: file1.txt file2.txt              # Use colon delimiter (both input and output)
join -t',' -o auto file1.csv file2.csv    # Join CSV files

# Outer joins (include unmatched lines)
join -a 1 file1.txt file2.txt             # Left outer join (all from file1)
join -a 2 file1.txt file2.txt             # Right outer join (all from file2)
join -a 1 -a 2 file1.txt file2.txt        # Full outer join (all from both)

# Replace empty fields
join -a 1 -a 2 -e 'NULL' -o auto file1.txt file2.txt  # Use 'NULL' for missing values

# Control output format (-o)
join -o 1.1,1.2,2.3 file1.txt file2.txt   # Output: file1 col1, file1 col2, file2 col3
join -o auto file1.txt file2.txt          # Auto format (all fields intelligently)

# Case-insensitive join
join -i file1.txt file2.txt               # Ignore case when matching

# Check for unpaired lines only
join -v 1 file1.txt file2.txt             # Show only lines from file1 with no match
join -v 2 file1.txt file2.txt             # Show only lines from file2 with no match
join -v 1 -v 2 file1.txt file2.txt        # Show unmatched from both (XOR)

# Practical examples
# Join employee names with salaries
sort -k1 employees.txt > emp_sorted.txt
sort -k1 salaries.txt > sal_sorted.txt
join -t',' -1 1 -2 1 emp_sorted.txt sal_sorted.txt

# Find users in both password and shadow files
join -t: -1 1 -2 1 <(sort /etc/passwd) <(sort /etc/shadow)

# Compare two lists and find common items
comm -12 <(sort list1.txt) <(sort list2.txt)  # Alternative to join

# Database-style join with header
(echo "ID,Name,Department,Salary"; join -t',' -o auto emp.csv sal.csv) > combined.csv
```

 The `join` command performs relational joins similar to SQL, matching lines from two files based on a common field, but requires both input files to be pre-sorted. The `-a` options enable outer joins to include unmatched lines from either or both files.     

### cat & Related - Vertical File Merging
```bash
# Concatenate files vertically
cat file1.txt file2.txt file3.txt         # Merge files end-to-end
cat *.log > combined.log                  # Combine all logs

# Add line numbers
cat -n file.txt                           # Number all lines
cat -b file.txt                           # Number only non-blank lines

# Show non-printing characters
cat -v file.txt                           # Show non-printing chars
cat -A file.txt                           # Show all (tabs as ^I, line ends as $)
cat -e file.txt                           # Show line endings only
cat -t file.txt                           # Show tabs only

# Suppress repeated blank lines
cat -s file.txt                           # Squeeze multiple blank lines to one

# Practical combinations
# Merge with headers
(echo "=== File1 ==="; cat file1.txt; echo "=== File2 ==="; cat file2.txt) > merged.txt

# Combine logs with timestamps preserved
cat /var/log/app.log.{1,2,3} | sort       # Merge and sort chronologically
```

### comm - Compare Sorted Files
```bash
# Compare two sorted files (3-column output)
comm file1.txt file2.txt                  # Col1: only in file1, Col2: only in file2, Col3: in both

# Show specific columns only
comm -12 file1.txt file2.txt              # Only lines in BOTH files (intersection)
comm -23 file1.txt file2.txt              # Only in file1, not in file2 (difference)
comm -13 file1.txt file2.txt              # Only in file2, not in file1 (difference)

# Practical examples
# Find common users between systems
comm -12 <(cut -d: -f1 /etc/passwd | sort) <(cut -d: -f1 backup_passwd | sort)

# Find packages installed on system1 but not system2
comm -23 <(rpm -qa | sort) <(ssh system2 'rpm -qa' | sort)
```

### Advanced Multi-File Operations
```bash
# Process multiple files with headers
awk 'FNR==1 && NR!=1{next} 1' *.csv > combined.csv  # Skip headers except first

# Merge files with filename column
for f in *.log; do awk -v fname="$f" '{print fname, $0}' "$f"; done > combined.log

# Parallel processing with paste
paste <(cut -f1 file1.txt | sort | uniq -c) <(cut -f1 file2.txt | sort | uniq -c)

# Split and merge workflow
split -l 1000 large.txt chunk_            # Split into 1000-line files
# Process chunk_* files individually
cat chunk_* > processed.txt               # Recombine

# Merge and deduplicate across files
sort -u file1.txt file2.txt file3.txt > unique_combined.txt
```
