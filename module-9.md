# Linux Text Editors

Text editors are fundamental tools for system administration, development, and configuration management on Linux systems. This cheatsheet covers the three major terminal-based editors available on CentOS 9, from beginner-friendly to power-user tools.  

## GNU Nano: The Beginner-Friendly Editor

Nano is the most accessible text editor with intuitive shortcuts displayed at the bottom of the screen. It comes pre-installed on CentOS 9 and requires no learning curve.  

### Opening Files and Basic Operations
- `nano filename` - Open or create a file
- `nano -l filename` - Open with line numbers enabled
- `nano +LINE filename` - Open at specific line number
- `nano -B filename` - Create backup before saving
- `nano -m` - Enable mouse support
- `nano -i` - Auto-indent new lines

### Essential Nano Commands
- `Ctrl+G` (F1) - Display help text with all commands 
- `Ctrl+X` (F2) - Exit nano (prompts to save if modified) 
- `Ctrl+O` (F3) - Write (save) file to disk 
- `Ctrl+R` (F5) - Read/insert another file into current buffer 
- `Ctrl+C` - Display current cursor position (line/column) 

### Navigation Commands
- `Ctrl+A` - Move to beginning of line
- `Ctrl+E` - Move to end of line
- `Ctrl+Y` - Page up (scroll up one screen)
- `Ctrl+V` - Page down (scroll down one screen)
- `Ctrl+_` (Ctrl+Shift+-) or `Alt+G` - Go to specific line and column number 
- `Alt+\` - Go to first line
- `Alt+/` - Go to last line
- `Ctrl+Space` - Move forward one word
- `Alt+Space` - Move backward one word

### Editing Commands
- `Ctrl+K` (F9) - Cut current line into cut buffer 
- `Ctrl+U` (F10) - Paste (uncut) from cut buffer 
- `Alt+6` - Copy current line without cutting
- `Ctrl+J` (F4) - Justify (reflow) current paragraph 
- `Alt+U` - Undo last operation  
- `Alt+E` - Redo undone operation
- `Ctrl+D` - Delete character at cursor
- `Ctrl+H` - Delete character before cursor (backspace)
- `Alt+Delete` - Delete word to right
- `Ctrl+Shift+Delete` - Delete from cursor to end of line

### Search and Replace
- `Ctrl+W` (F6) - Search forward 
- `Alt+W` - Search next occurrence
- `Ctrl+Q` - Search backward
- `Ctrl+\` or `Alt+R` - Search and replace 
- `Alt+C` - Toggle case-sensitive search (during search prompt)
- `Alt+R` - Use regular expressions in search

### Selection and Block Operations
- `Alt+A` - Start selecting text (mark set)
- Move cursor to select text, then use cut/copy commands
- `Ctrl+K` after marking - Cut selection
- `Alt+6` after marking - Copy selection
- `Alt+^` - Copy from cursor to end of file

### Advanced Nano Features
- `Ctrl+T` (F12) - Invoke spell checker (if available)  
- `Alt+}` - Indent current line or marked text
- `Alt+{` - Unindent current line or marked text
- `Alt+3` - Comment/uncomment current line (language-aware)
- `Alt+N` - Toggle line numbers on/off
- `Alt+P` - Toggle whitespace visibility
- `Alt+X` - Toggle help bar at bottom

### Nano Configuration (~/.nanorc)
```bash
# Enable line numbers by default
set linenumbers

# Auto-indent new lines
set autoindent

# Convert tabs to spaces
set tabstospaces

# Set tab size to 4 spaces
set tabsize 4

# Enable mouse support
set mouse

# Enable smooth scrolling
set smooth

# Show filename in title bar
set titlecolor brightwhite,blue

# Enable syntax highlighting
include /usr/share/nano/*.nanorc
```

## Vi/Vim: The Power User's Choice

Vim is a modal editor that separates text insertion from text manipulation, making it extremely powerful for efficient editing. Understanding its philosophy is key: commands are composed semantically like sentences with verbs (operations) and objects (text objects).  

### Modal Editing Philosophy
Vim operates in distinct modes that separate navigation, editing, and command execution: 
- **Normal Mode**: Default mode for navigation and commands
- **Insert Mode**: For typing text
- **Visual Mode**: For selecting text
- **Command-Line Mode**: For executing ex commands

### Opening Files and Starting Vim
- `vim filename` - Open or create file
- `vim +LINE filename` - Open at specific line
- `vim +/pattern filename` - Open at first match of pattern
- `vim -R filename` or `view filename` - Open in read-only mode
- `vim -d file1 file2` - Open in diff mode
- `vim -p file1 file2 file3` - Open multiple files in tabs

### Mode Switching (Critical Foundation)
- `Esc` - Return to Normal mode from any mode
- `i` - Insert before cursor
- `I` - Insert at beginning of line
- `a` - Append after cursor
- `A` - Append at end of line
- `o` - Open new line below and insert
- `O` - Open new line above and insert
- `v` - Enter character-wise visual mode 
- `V` - Enter line-wise visual mode 
- `Ctrl+v` - Enter block-wise (column) visual mode 
- `:` - Enter command-line mode
- `R` - Enter replace mode (overwrite)

### Navigation in Normal Mode
**Character Movement:**
- `h` - Left
- `j` - Down
- `k` - Up
- `l` - Right

**Word Movement:**
- `w` - Forward to start of next word
- `W` - Forward to start of next WORD (whitespace-delimited)
- `e` - Forward to end of word
- `E` - Forward to end of WORD
- `b` - Backward to start of word
- `B` - Backward to start of WORD
- `ge` - Backward to end of previous word

**Line Movement:**
- `0` - Beginning of line (column 0)
- `^` - First non-whitespace character
- `$` - End of line
- `g_` - Last non-whitespace character
- `f{char}` - Find next occurrence of character on line
- `F{char}` - Find previous occurrence of character
- `t{char}` - Till (before) next occurrence
- `T{char}` - Till (before) previous occurrence
- `;` - Repeat last f/F/t/T movement
- `,` - Repeat last f/F/t/T movement in reverse

**Screen Movement:**
- `H` - Move to top of screen
- `M` - Move to middle of screen
- `L` - Move to bottom of screen
- `Ctrl+u` - Scroll up half screen
- `Ctrl+d` - Scroll down half screen
- `Ctrl+b` - Scroll up full screen (page up)
- `Ctrl+f` - Scroll down full screen (page down)
- `zt` - Scroll current line to top
- `zz` - Scroll current line to center
- `zb` - Scroll current line to bottom

**Document Movement:**
- `gg` - Go to first line
- `G` - Go to last line
- `{number}G` or `:{number}` - Go to line number
- `{` - Jump to previous paragraph
- `}` - Jump to next paragraph
- `%` - Jump to matching bracket/parenthesis/brace

### Text Object Philosophy (Game Changer)
Vim's text objects define "what" to operate on. They combine with operators to create powerful command chains: 

**Inner vs Around (i vs a):**
- `iw` - Inner word (word only)
- `aw` - A word (word + space)
- `iW` - Inner WORD
- `aW` - A WORD
- `is` - Inner sentence
- `as` - A sentence
- `ip` - Inner paragraph
- `ap` - A paragraph
- `i"` or `i'` or `` i` `` - Inside quotes
- `a"` or `a'` or `` a` `` - Around quotes (includes quotes)
- `i(` or `i)` or `ib` - Inside parentheses
- `a(` or `a)` or `ab` - Around parentheses
- `i{` or `i}` or `iB` - Inside braces
- `a{` or `a}` or `aB` - Around braces
- `i[` or `i]` - Inside brackets
- `a[` or `a]` - Around brackets
- `i<` or `i>` - Inside angle brackets
- `a<` or `a>` - Around angle brackets
- `it` - Inside HTML/XML tag
- `at` - Around tag

### Operators: The "Verbs" of Vim
Operators perform actions on text objects: 
- `d` - Delete (cut)
- `c` - Change (delete and enter insert mode)
- `y` - Yank (copy)
- `p` - Put (paste) after cursor
- `P` - Put before cursor
- `~` - Toggle case
- `gu` - Make lowercase
- `gU` - Make uppercase
- `>` - Indent right
- `<` - Indent left
- `=` - Auto-indent
- `!` - Filter through external command

### Power Command Combinations
Combine operators + motion/text-objects for semantic editing: 
- `dw` - Delete word 
- `d$` - Delete to end of line 
- `c3w` - Change next 3 words 
- `diw` - Delete inner word (cursor anywhere in word)
- `daw` - Delete a word (includes surrounding space)
- `ci"` - Change inside quotes
- `da{` - Delete around braces (includes braces)
- `yi}` - Yank inside braces
- `=ap` - Auto-indent a paragraph
- `gUiw` - Uppercase inner word
- `>ip` - Indent inner paragraph
- `dtx` - Delete till character 'x'
- `yf.` - Yank from cursor to next period

### Essential Editing Commands
- `x` - Delete character under cursor
- `X` - Delete character before cursor
- `r{char}` - Replace character with {char}
- `s` - Substitute character (delete and insert)
- `S` or `cc` - Substitute line
- `dd` - Delete line
- `yy` or `Y` - Yank line
- `D` - Delete to end of line
- `C` - Change to end of line
- `J` - Join line below with current line
- `gJ` - Join without adding space
- `.` - Repeat last change (extremely powerful)
- `u` - Undo
- `Ctrl+r` - Redo
- `U` - Undo all changes on current line

### Visual Mode Operations
After selecting text in visual mode:
- `d` - Delete selection
- `c` - Change selection
- `y` - Yank selection
- `>` - Indent selection
- `<` - Unindent selection
- `=` - Auto-indent selection
- `~` - Toggle case
- `u` - Make lowercase
- `U` - Make uppercase
- `o` - Move to other end of selection
- `O` - Move to other corner (block mode)

### Advanced Vi Commands

**Search and Replace:**
- `/pattern` - Search forward
- `?pattern` - Search backward
- `n` - Next match
- `N` - Previous match
- `*` - Search word under cursor forward
- `#` - Search word under cursor backward
- `:%s/old/new/g` - Replace all in file
- `:%s/old/new/gc` - Replace all with confirmation
- `:s/old/new/g` - Replace all in current line
- `:'<,'>s/old/new/g` - Replace in visual selection
- `:%s/\<old\>/new/g` - Replace whole word only

**Marks and Jumps:**
- `m{a-z}` - Set local mark
- `m{A-Z}` - Set global mark (across files)
- `'{mark}` - Jump to mark's line
- `` `{mark} `` - Jump to mark's exact position
- `''` - Jump back to previous location
- ``` `` ```
- `Ctrl+o` - Jump to older cursor position
- `Ctrl+i` - Jump to newer cursor position

**Macros:**
- `q{a-z}` - Start recording macro into register
- `q` - Stop recording
- `@{a-z}` - Execute macro
- `@@` - Repeat last macro
- `{number}@{a-z}` - Execute macro n times

**Registers:**
- `"{register}{operator}` - Use specific register
- `"ayy` - Yank line into register 'a'
- `"Ayy` - Append line to register 'a'
- `"ap` - Paste from register 'a'
- `"+y` - Yank to system clipboard
- `"+p` - Paste from system clipboard
- `"*y` - Yank to X11 primary selection
- `:reg` - View all registers

**Windows and Tabs:**
- `:split` or `Ctrl+w s` - Split horizontally
- `:vsplit` or `Ctrl+w v` - Split vertically
- `Ctrl+w w` - Switch windows
- `Ctrl+w h/j/k/l` - Navigate to window
- `Ctrl+w =` - Equal window sizes
- `Ctrl+w _` - Maximize height
- `Ctrl+w |` - Maximize width
- `Ctrl+w q` - Close window
- `:tabnew` - New tab
- `gt` - Next tab
- `gT` - Previous tab
- `{number}gt` - Go to tab number

**Folding:**
- `zf{motion}` - Create fold
- `zd` - Delete fold
- `zo` - Open fold
- `zc` - Close fold
- `za` - Toggle fold
- `zR` - Open all folds
- `zM` - Close all folds

**Command-Line Mode Commands:**
- `:w` - Write (save) file
- `:w filename` - Save as filename
- `:q` - Quit
- `:q!` - Quit without saving
- `:wq` or `:x` or `ZZ` - Save and quit
- `:e filename` - Edit file
- `:e!` - Reload file (discard changes)
- `:r filename` - Read file and insert below cursor
- `:r !command` - Read command output
- `:{range}d` - Delete range (e.g., `:10,20d`)
- `:{range}y` - Yank range
- `:{range}t{line}` - Copy range to line
- `:{range}m{line}` - Move range to line
- `:!command` - Execute shell command
- `:.!command` - Filter current line through command
- `:%!command` - Filter entire file through command

**Advanced Movement:**
- `[[` - Jump to previous section/function
- `]]` - Jump to next section/function
- `[{` - Jump to previous unmatched `{`
- `]}` - Jump to next unmatched `}`
- `[(` - Jump to previous unmatched `(`
- `])` - Jump to next unmatched `)`
- `gd` - Go to local declaration
- `gD` - Go to global declaration
- `gf` - Go to file under cursor

**Advanced Editing:**
- `Ctrl+a` - Increment number
- `Ctrl+x` - Decrement number
- `g~{motion}` - Toggle case
- `gwip` - Reformat paragraph
- `:{range}sort` - Sort lines
- `:{range}!sort` - Sort using external command
- `:retab` - Fix spacing/tabs

### Vim Configuration (~/.vimrc Essentials)
```vim
" Enable syntax highlighting
syntax on

" Show line numbers
set number
set relativenumber

" Enable mouse support
set mouse=a

" Tab settings
set tabstop=4
set shiftwidth=4
set expandtab
set autoindent
set smartindent

" Search settings
set ignorecase
set smartcase
set incsearch
set hlsearch

" Display settings
set showcmd
set showmatch
set ruler
set cursorline

" Enable filetype detection
filetype plugin indent on

" Better backspace behavior
set backspace=indent,eol,start

" Persistent undo
set undofile
set undodir=~/.vim/undodir

" Split behavior
set splitright
set splitbelow
```

## Emacs: The Extensible Environment

Emacs is not just a text editor but an extensible computing environment with built-in capabilities for file management, mail, IRC, and more. It uses a modifier-key based interface rather than modal editing.  

### Starting Emacs
- `emacs` - Start in GUI mode (if X11 available)
- `emacs -nw` - Start in terminal mode
- `emacs filename` - Open file
- `emacsclient filename` - Connect to Emacs daemon (faster)

### Emacs Notation
- `C-` means Control key (e.g., `C-x` = Ctrl+X)
- `M-` means Meta key (usually Alt or Esc)
- `C-M-` means Control+Meta together
- Commands are sequences: `C-x C-s` means Ctrl+X, then Ctrl+S

### Essential Emacs Commands

**Help System (Start Here!):**
- `C-h t` - Built-in tutorial
- `C-h k` - Describe key binding
- `C-h f` - Describe function
- `C-h v` - Describe variable
- `C-h a` - Command apropos (search commands)
- `C-h i` - Info documentation browser

**File Operations:**
- `C-x C-f` - Find (open) file
- `C-x C-s` - Save file
- `C-x C-w` - Write (save as)
- `C-x C-c` - Exit Emacs
- `C-x i` - Insert file at cursor

**Navigation:**
- `C-f` - Forward character
- `C-b` - Backward character
- `C-n` - Next line
- `C-p` - Previous line
- `M-f` - Forward word
- `M-b` - Backward word
- `C-a` - Beginning of line
- `C-e` - End of line
- `M-a` - Beginning of sentence
- `M-e` - End of sentence
- `M-{` - Backward paragraph
- `M-}` - Forward paragraph
- `C-v` - Page down
- `M-v` - Page up
- `M-<` - Beginning of buffer
- `M->` - End of buffer
- `M-g g` - Go to line number

**Basic Editing:**
- `C-d` - Delete character forward
- `DEL` or `Backspace` - Delete character backward
- `M-d` - Kill word forward
- `M-DEL` - Kill word backward
- `C-k` - Kill to end of line
- `C-S-Backspace` - Kill entire line
- `M-k` - Kill to end of sentence

**Kill and Yank (Cut/Copy/Paste):**
- `C-w` - Kill (cut) region
- `M-w` - Copy region (without killing)
- `C-y` - Yank (paste) most recent kill
- `M-y` - Cycle through kill ring (after C-y)
- `C-Space` or `C-@` - Set mark (start selection)
- `C-x C-x` - Exchange point and mark

**Undo and Redo:**
- `C-/` or `C-_` or `C-x u` - Undo
- `C-g` then undo again - Redo (undo the undos)

**Search:**
- `C-s` - Incremental search forward
- `C-r` - Incremental search backward
- `C-s C-s` - Search for last searched term
- `M-%` - Query replace
- During query replace:
  - `y` - Replace this match
  - `n` - Skip this match
  - `!` - Replace all remaining
  - `q` - Quit replace

**Buffer Management:**
- `C-x b` - Switch to buffer
- `C-x C-b` - List buffers
- `C-x k` - Kill buffer
- `C-x left` - Previous buffer
- `C-x right` - Next buffer

**Window Management:**
- `C-x 2` - Split horizontally
- `C-x 3` - Split vertically
- `C-x 1` - Close other windows (maximize current)
- `C-x 0` - Close current window
- `C-x o` - Switch to other window
- `C-M-v` - Scroll other window

### Advanced Emacs Commands

**Advanced Editing:**
- `C-t` - Transpose characters
- `M-t` - Transpose words
- `C-x C-t` - Transpose lines
- `M-u` - Uppercase word
- `M-l` - Lowercase word
- `M-c` - Capitalize word
- `C-x C-u` - Uppercase region
- `C-x C-l` - Lowercase region
- `M-q` - Fill (reflow) paragraph
- `C-x f` - Set fill column
- `M-^` - Join line with previous

**Advanced Kill/Yank:**
- `C-M-k` - Kill s-expression (programming)
- `M-z char` - Zap (kill) to character
- `C-x r k` - Kill rectangle
- `C-x r y` - Yank rectangle
- `C-x r t` - String rectangle (replace with text)

**Advanced Search and Replace:**
- `M-s w` - Word search
- `M-s .` - Search for symbol at point
- `M-x replace-string` - Simple replace all
- `M-x replace-regexp` - Replace with regex
- `C-M-s` - Regex incremental search forward
- `C-M-r` - Regex incremental search backward

**Keyboard Macros:**
- `C-x (` or `F3` - Start recording macro
- `C-x )` or `F4` - Stop recording macro
- `C-x e` or `F4` - Execute macro
- `e` (after C-x e) - Execute again
- `C-u {number} C-x e` - Execute n times
- `C-x C-k n` - Name last macro
- `M-x {name}` - Execute named macro

**Marks and Registers:**
- `C-x r Space {char}` - Save position to register
- `C-x r j {char}` - Jump to register
- `C-x r s {char}` - Save region to register
- `C-x r i {char}` - Insert register contents

**Multiple Cursors (iedit mode):**
- `C-;` - Edit all occurrences (if package installed)
- `M-x iedit-mode` - Toggle iedit mode

**Dired (Directory Editor):**
- `C-x d` - Open dired
- In dired mode:
  - `n` - Next line
  - `p` - Previous line
  - `RET` - Open file/directory
  - `d` - Mark for deletion
  - `x` - Execute deletions
  - `C` - Copy file
  - `R` - Rename/move file
  - `+` - Create directory
  - `g` - Refresh
  - `q` - Quit dired

**Shell and Compilation:**
- `M-x shell` - Start shell
- `M-x eshell` - Start Emacs shell
- `M-!` - Execute shell command
- `M-|` - Pipe region through command
- `M-x compile` - Run compilation command
- `M-x grep` - Run grep

**Programming Features:**
- `C-M-f` - Forward s-expression
- `C-M-b` - Backward s-expression
- `C-M-d` - Down into expression
- `C-M-u` - Up out of expression
- `C-M-a` - Beginning of function
- `C-M-e` - End of function
- `C-M-h` - Mark function
- `M-;` - Comment/uncomment region
- `C-c C-c` - Compile/evaluate (mode-dependent)
- `M-.` - Find definition
- `M-,` - Pop back from definition

**Advanced Navigation:**
- `C-u C-Space` - Pop mark (return to previous positions)
- `C-x C-Space` - Pop global mark
- `M-m` - Back to indentation
- `C-M-l` - Reposition window

### Emacs Configuration (~/.emacs or ~/.emacs.d/init.el)
```elisp
;; Disable startup message
(setq inhibit-startup-message t)

;; Show line numbers
(global-display-line-numbers-mode t)

;; Highlight current line
(global-hl-line-mode t)

;; Show matching parentheses
(show-paren-mode t)

;; Enable column number mode
(column-number-mode t)

;; Better defaults
(setq-default
 indent-tabs-mode nil   ; Use spaces instead of tabs
 tab-width 4            ; 4 spaces per tab
 fill-column 80)        ; Line length

;; Enable recent files
(recentf-mode t)
(setq recentf-max-saved-items 50)

;; Better scrolling
(setq scroll-conservatively 100)

;; Auto-refresh buffers when files change
(global-auto-revert-mode t)

;; Use evil-mode for Vim keybindings (optional)
;; (require 'evil)
;; (evil-mode 1)
```

## Comparison and Editor Selection

| Editor | Learning Curve | Speed | Best For | Modal | Extensibility |
|--------|---------------|-------|----------|-------|---------------|
| Nano | None | Instant | Quick edits, configs, beginners | No | Low |
| Vim | Steep | Very Fast | Programming, power users, SSH | Yes | High |
| Emacs | Steep | Fast | Programming, multi-tasking, customization | No | Very High |

## Lab Exercises

### Lab 1: Nano Mastery
1. Create file: `nano practice.txt`
2. Type several paragraphs of text
3. Navigate with Ctrl+A, Ctrl+E, Ctrl+Y, Ctrl+V
4. Cut three lines with Ctrl+K (three times)
5. Paste them elsewhere with Ctrl+U
6. Search for a word with Ctrl+W
7. Replace all occurrences with Ctrl+\
8. Enable line numbers with Alt+N
9. Go to specific line with Ctrl+_
10. Save with Ctrl+O, exit with Ctrl+X

### Lab 2: Vim Motion and Editing
1. Create file: `vim essay.txt`
2. Enter insert mode (i), type content, exit to normal mode (Esc)
3. Practice word motions: `w`, `b`, `e`, `ge`
4. Practice line motions: `0`, `^`, `$`, `f`, `t`
5. Delete operations: `dw`, `dd`, `d$`, `dtx`
6. Change operations: `cw`, `ci"`, `ca{`
7. Visual mode: Select with `v`, cut with `d`, paste with `p`
8. Search with `/pattern`, jump with `n` and `N`
9. Replace: `:%s/old/new/g`
10. Save and quit: `:wq`

### Lab 3: Advanced Vim Power User
1. Open file: `vim code.py`
2. Set marks: `ma` at line 10, `mb` at line 50
3. Jump between marks: `'a`, `'b`
4. Record macro: `qa` → make edits → `q`
5. Execute macro: `@a`, repeat with `10@a`
6. Split windows: `Ctrl+w v`, navigate with `Ctrl+w h/l`
7. Use text objects: `di"`, `ca(`, `=ap`
8. Block visual mode: `Ctrl+v`, select columns, insert with `I`
9. External filter: `:%!sort` to sort lines
10. Use registers: `"ayy` to yank to register a, `"ap` to paste

### Lab 4: Emacs Kill, Yank, and Search
1. Start Emacs: `emacs -nw document.txt`
2. Navigate: `C-f`, `C-b`, `M-f`, `M-b`, `C-n`, `C-p`
3. Set mark: `C-Space`, move to select region
4. Kill region: `C-w` (cut)
5. Copy region: `M-w`
6. Yank: `C-y`, cycle kill ring with `M-y`
7. Kill line: `C-k`, kill word: `M-d`
8. Incremental search: `C-s`, type pattern, `C-s` to next
9. Query replace: `M-%`, respond with y/n/!/q
10. Split window: `C-x 3`, switch: `C-x o`, close: `C-x 1`

### Lab 5: Emacs Advanced Operations
1. Open dired: `C-x d`
2. Navigate files, mark for deletion with `d`, execute with `x`
3. Start keyboard macro: `C-x (`
4. Perform repetitive edits
5. End macro: `C-x )`
6. Execute macro: `C-x e`, repeat with `e e e`
7. Open multiple files in buffers: `C-x C-f`
8. Switch buffers: `C-x b`, list buffers: `C-x C-b`
9. Rectangle operations: `C-x r k` (kill), `C-x r y` (yank)
10. Use shell: `M-x shell`, run commands

***

**Pro Tips**

1. **Vim's Dot Command (`.`)** - Master repeating changes. Make an edit once, repeat anywhere with `.`
2. **Text Objects** - Understanding `ci"`, `da{`, `=ap` puts you in the top 10% of Vim users 
3. **Semantic Commands** - Think in verb+noun: "delete a word" (`daw`), not "move and delete 5 times" 
4. **Macros** - Automate repetitive tasks in both Vim and Emacs
5. **Know Your Editor** - Choose one editor, master it deeply rather than knowing three superficially
6. **Configuration** - Customize your editor; the smartest users have personalized workflows
7. **Modal Thinking** - Understand why Vim's modal approach makes sense for editing vs writing
8. **Keyboard > Mouse** - Terminal editors force keyboard fluency, making you faster
9. **Registers/Clipboard** - Use multiple clipboards/registers for complex editing tasks
10. **Integration** - Learn how editors integrate with shell commands (`!`, `:r!`, etc.)
