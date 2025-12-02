# CentOS 9 Shell Mastery Cheatsheet (2025 Edition)

## Role of Command Shell

The shell is a command interpreter that sits between you and the kernel, translating commands into system calls. CentOS Stream 9 uses bash 5.1+ as the default shell, offering enhanced features for modern DevOps workflows.   

**Key Shell Types:**
- **Login shell**: Reads `/etc/profile`, `~/.bash_profile`, `~/.bash_login`, `~/.profile`
- **Non-login interactive**: Reads `~/.bashrc`
- **Non-interactive**: Used in scripts, inherits parent environment

**Critical Commands:**
- `echo $0` - Check current shell (`-bash` = login, `bash` = non-login)
- `$SHELL` - Default shell path
- `$$` - Current shell PID
- `$PPID` - Parent process ID

## Communication Channels (File Descriptors)

Every process has three standard streams with numeric file descriptors: 

| FD | Name | Symbol | Default | Purpose |
|---|---|---|---|---|
| 0 | stdin | `<` | keyboard | Input stream |
| 1 | stdout | `>` | terminal | Normal output |
| 2 | stderr | `2>` | terminal | Error messages |

**Advanced FD Operations:**
- `exec 3< file` - Open file on FD 3 for reading
- `exec 4> file` - Open file on FD 4 for writing
- `exec 3<&-` - Close FD 3
- `/dev/fd/N` - Access file descriptor N
- `&>` - Redirect both stdout and stderr (bash shorthand)

## File Redirection

**Output Redirection:**
- `cmd > file` - Overwrite file with stdout
- `cmd >> file` - Append stdout to file
- `cmd 2> file` - Redirect stderr only
- `cmd &> file` - Redirect both stdout and stderr (preferred modern syntax)
- `cmd > file 2>&1` - Redirect stderr to stdout's destination (POSIX-compliant)
- `cmd 2>&1 > file` - **WRONG ORDER** (stderr goes to terminal, stdout to file)

**Input Redirection:**
- `cmd < file` - Read input from file
- `cmd <<< "string"` - Here-string (feed string as input)
- `cmd << EOF` - Here-document (multi-line input until EOF)

**Advanced Patterns:**
- `cmd > output.txt 2> error.txt` - Split stdout and stderr
- `cmd 2>&1 | tee file` - Merge streams and duplicate to file
- `cmd &> /dev/null` - Discard all output (silence command)
- `cmd |& grep error` - Pipe both stdout and stderr (bash 4.0+)

**Pro Tip:** Order matters! `2>&1` must come *after* `>file` to capture stderr in the same destination. 

## Piping Commands Together

Pipes connect stdout of one command to stdin of another, creating powerful processing chains. 

**Basic Syntax:**
- `cmd1 | cmd2` - stdout of cmd1 → stdin of cmd2
- `cmd1 |& cmd2` - stdout AND stderr of cmd1 → stdin of cmd2

**Essential Pipe Patterns:**
- `ps aux | grep process | grep -v grep` - Filter processes (exclude grep itself)
- `cat file | tr '[:lower:]' '[:upper:]' | sort | uniq -c` - Transform, sort, count
- `find . -type f | wc -l` - Count files
- `history | tail -20 | cut -d' ' -f4-` - Last 20 commands (extract command only)
- `ls -l | tee listing.txt | grep "^d"` - Save output and filter directories

**Advanced Piping:**
- `cmd1 | command substitution is different` - See Nesting Commands section
- `cmd1 | xargs cmd2` - Convert stdin to command arguments
- `cmd1 | parallel cmd2` - GNU parallel for concurrent processing
- `set -o pipefail` - Pipeline fails if any command fails (critical for scripts)

## Wildcard Patterns/Globbing

Globbing expands patterns BEFORE command execution (shell operation, not command). 

**Core Wildcards:**
- `*` - Match 0+ characters (except leading dot)
- `?` - Match exactly 1 character
- `[abc]` - Match any one: a, b, or c
- `[!abc]` or `[^abc]` - Match any character EXCEPT a, b, or c
- `[a-z]` - Match any character in range
- `[[:alpha:]]` - POSIX character class (letters)

**POSIX Character Classes:**
- `[[:alnum:]]` - Alphanumeric (letters + digits)
- `[[:alpha:]]` - Letters only
- `[[:digit:]]` - Digits 0-9
- `[[:lower:]]` - Lowercase letters
- `[[:upper:]]` - Uppercase letters
- `[[:space:]]` - Whitespace (space, tab, newline)

**Extended Globbing** (`shopt -s extglob`):
- `?(pattern)` - Match 0 or 1 occurrence
- `*(pattern)` - Match 0+ occurrences
- `+(pattern)` - Match 1+ occurrences
- `@(pattern)` - Match exactly one
- `!(pattern)` - Match anything except pattern

**Examples:**
- `*.{txt,log}` - All .txt and .log files (brace expansion + globbing)
- `file?.txt` - file1.txt, fileA.txt (not file10.txt)
- `[!.]*` - All files not starting with dot
- `**/*.py` - All .py files recursively (`shopt -s globstar`)

## Brace Expansion

Brace expansion generates arbitrary strings BEFORE globbing.  

**Basic Syntax:**
- `{A,B,C}` → `A B C`
- `file{1,2,3}.txt` → `file1.txt file2.txt file3.txt`
- `{A,B}.{js,py}` → `A.js A.py B.js B.py` (cartesian product)

**Sequence Generation:**
- `{1..5}` → `1 2 3 4 5`
- `{a..z}` → `a b c ... z`
- `{01..10}` → `01 02 03 ... 10` (zero-padded)
- `{1..10..2}` → `1 3 5 7 9` (step by 2)
- `{10..1}` → `10 9 8 ... 1` (reverse)

**Nested Braces:**
- `{{A..C},{1..3}}` → `A B C 1 2 3`
- `{A,B{1,2}}` → `A B1 B2`

**Practical Use Cases:**
- `mkdir -p project/{src,test,docs}/{js,py}` - Create complex directory structure
- `cp file.txt{,.bak}` → `cp file.txt file.txt.bak` (quick backup)
- `mv config.{old,new}` → `mv config.old config.new`
- `touch test{1..100}.log` - Create 100 test files
- `echo {1..3}{A..C}` → `1A 1B 1C 2A 2B 2C 3A 3B 3C`

## Shell/Environment Variables

**Shell Variables** (local to current shell):
- `VARNAME=value` - Create/set (NO spaces around `=`)
- `echo $VARNAME` - Access value
- `unset VARNAME` - Delete variable

**Environment Variables** (inherited by child processes):
- `export VARNAME=value` - Create and export
- `export VARNAME` - Export existing shell variable
- `env` - Show all environment variables
- `printenv VARNAME` - Show specific variable

**Critical Environment Variables:**
- `PATH` - Colon-separated command search paths
- `HOME` - User's home directory
- `USER` / `LOGNAME` - Current username
- `SHELL` - Default shell
- `PWD` - Current working directory
- `OLDPWD` - Previous directory (used by `cd -`)
- `PS1` - Primary prompt string
- `IFS` - Internal Field Separator (default: space, tab, newline)

**Special Variables:**
- `$?` - Exit status of last command (0 = success)
- `$$` - Current shell PID
- `$!` - PID of last background process
- `$0` - Script name or shell name
- `$1, $2, ... $9, ${10}` - Positional parameters
- `$#` - Number of positional parameters
- `$@` - All parameters as separate words
- `$*` - All parameters as single word

**Variable Operations:**
- `${VAR:-default}` - Use default if VAR unset/null
- `${VAR:=default}` - Assign default if VAR unset/null
- `${VAR:?error}` - Exit with error if VAR unset/null
- `${VAR:+value}` - Use value if VAR is set
- `${#VAR}` - Length of VAR
- `${VAR:offset:length}` - Substring extraction
- `${VAR//pattern/replacement}` - Replace all occurrences

## General Quoting Rules

Quoting controls shell interpretation of special characters.

**Single Quotes (`'...'`):**
- Preserve EVERYTHING literally (strongest quoting)
- Cannot escape single quote inside single quotes
- `echo '$(pwd) $HOME *'` → `$(pwd) $HOME *` (no expansion)

**Double Quotes (`"..."`):**
- Allow variable expansion, command substitution, arithmetic
- Prevent word splitting and globbing
- Preserve whitespace
- `echo "$(pwd) $HOME *"` → `/current/path /home/user *` (expands variables/commands)

**Backslash (`\`):**
- Escape next single character
- `echo \$HOME` → `$HOME` (literal dollar sign)
- Line continuation: `\` at end of line

**Special Cases:**
- `$'...'` - ANSI-C quoting: `$'\n'` = newline, `$'\t'` = tab
- No quotes - Subject to word splitting, globbing, expansion

**Critical Patterns:**
- `"$@"` - Preserve individual arguments (ALWAYS quote in scripts)
- `"$*"` - All arguments as single string
- `"${array[@]}"` - Preserve array elements
- `rm "$file"` - ALWAYS quote variables containing paths

**Pro Rule:** When in doubt, use double quotes around variables to prevent word splitting bugs.

## Nesting Commands

Command substitution executes commands and replaces them with output. 

**Modern Syntax (Preferred):**
- `$(command)` - Nestable, readable, POSIX
- `echo "Today is $(date +%Y-%m-%d)"` - Inline substitution
- `files=$(ls -1 | wc -l)` - Assign output to variable

**Legacy Syntax (Backticks):**
- `` `command` `` - Cannot nest easily
- ``files=`ls -1 | wc -l` `` - Same as above but less readable

**Nested Substitution:**
- `echo "User $(whoami) in $(basename $(pwd))"` - Multiple levels
- `$(command1 $(command2))` - Inner executes first

**Process Substitution (Advanced):**
- `<(command)` - Creates temporary file with command output
- `>(command)` - Creates temporary file for command input
- `diff <(ls dir1) <(ls dir2)` - Compare command outputs
- `command > >(tee file1) 2> >(tee file2 >&2)` - Split and save streams

**Arithmetic Expansion:**
- `$((expression))` - Integer arithmetic
- `echo $((5 + 3 * 2))` → `11`
- `i=$((i + 1))` - Increment variable
- Supports: `+ - * / % **` (exponent), bitwise operators

**Combined Power Pattern:**
```bash
for file in $(find . -type f -mtime -1); do
    echo "Modified: $(stat -c %y "$file") - $(basename "$file")"
done
```