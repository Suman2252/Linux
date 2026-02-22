# ✏️ Chapter 08: Text Editors

<p align="center">
  <img src="https://img.shields.io/badge/Level-Beginner-green?style=for-the-badge" alt="Beginner">
  <img src="https://img.shields.io/badge/Chapter-08%20of%2034-blue?style=for-the-badge" alt="Chapter 08">
</p>

---

## 📑 Table of Contents

- [Choosing a Text Editor](#choosing-a-text-editor)
- [Nano — The Beginner-Friendly Editor](#nano--the-beginner-friendly-editor)
- [Vim — The Power Editor](#vim--the-power-editor)
- [Neovim — Modern Vim](#neovim--modern-vim)
- [Other Editors](#other-editors)
- [Practice Exercises](#-practice-exercises)

---

## Choosing a Text Editor

| Editor | Learning Curve | Speed | Best For |
|--------|---------------|-------|----------|
| **nano** | ⭐ Easy | Medium | Quick edits, beginners |
| **vim** | ⭐⭐⭐⭐ Hard | Fast | Power users, servers |
| **neovim** | ⭐⭐⭐⭐ Hard | Fastest | Modern vim experience |
| **emacs** | ⭐⭐⭐⭐⭐ Very Hard | Fast | Everything (it's practically an OS) |
| **VS Code** | ⭐⭐ Easy | Medium | GUI development |

> 💡 **Recommendation**: Learn **nano** for quick edits, and invest time in **vim** — it's available on every Linux server and will make you incredibly productive.

---

## Nano — The Beginner-Friendly Editor

Nano is pre-installed on most Linux systems and shows keyboard shortcuts at the bottom.

### Basic Usage

```bash
nano filename.txt                  # Open or create a file
nano +15 filename.txt              # Open at line 15
nano -l filename.txt               # Show line numbers
sudo nano /etc/hosts               # Edit system files (need root)
```

### Nano Keyboard Shortcuts

The `^` symbol means **Ctrl** and `M-` means **Alt**.

| Shortcut | Action |
|----------|--------|
| `Ctrl + O` | Save (Write Out) |
| `Ctrl + X` | Exit |
| `Ctrl + K` | Cut line |
| `Ctrl + U` | Paste line |
| `Ctrl + W` | Search |
| `Ctrl + \\` | Search & Replace |
| `Ctrl + G` | Help |
| `Ctrl + _` | Go to line number |
| `Alt + U` | Undo |
| `Alt + E` | Redo |
| `Ctrl + C` | Show current line number |
| `Alt + A` | Start selecting text |
| `Ctrl + ^` | Start selecting (alternative) |

### Configure Nano

```bash
# Create/edit nano config
nano ~/.nanorc
```

```ini
# ~/.nanorc — Useful nano settings
set tabsize 4              # Set tab width to 4 spaces
set tabstospaces           # Convert tabs to spaces
set linenumbers            # Always show line numbers
set autoindent             # Auto-indent new lines
set mouse                  # Enable mouse support
set smooth                 # Smooth scrolling
set constantshow           # Always show cursor position
set softwrap               # Soft wrap long lines

# Syntax highlighting (usually enabled by default)
include "/usr/share/nano/*.nanorc"
```

---

## Vim — The Power Editor

Vim (**Vi IMproved**) is the most popular terminal editor on servers. It has a steep learning curve but extraordinary power once mastered.

> 🏠 **Analogy**: Vim is like learning to touch-type. Slow and frustrating at first, but once you get it, you'll never go back.

### Install Vim

```bash
sudo apt install vim               # Debian/Ubuntu
sudo dnf install vim-enhanced      # Fedora
sudo pacman -S vim                 # Arch
```

### Understanding Vim Modes

Vim is a **modal editor** — keys do different things depending on which mode you're in:

```
                 ┌──────────────────┐
     i, a, o     │   INSERT MODE    │     Esc
  ┌─────────────▶│  Type text here  │──────────────┐
  │              └──────────────────┘               │
  │                                                 ▼
  │              ┌──────────────────┐         ┌────────────────┐
  │              │  COMMAND MODE    │         │  NORMAL MODE   │
  │              │  :w :q :wq :%s  │◀────────│  Navigate, edit│
  │              └──────────────────┘   :     │  copy, delete  │
  │                                           └───┬────────────┘
  │                                               │    v, V
  │              ┌──────────────────┐              │
  │              │  VISUAL MODE     │◀─────────────┘
  │              │  Select text     │
  │              └──────────────────┘
```

| Mode | Purpose | Enter | Leave |
|------|---------|-------|-------|
| **Normal** | Navigate, delete, copy | `Esc` | Press mode key |
| **Insert** | Type text | `i`, `a`, `o` | `Esc` |
| **Visual** | Select text | `v`, `V`, `Ctrl+v` | `Esc` |
| **Command** | Run commands | `:` | `Enter` or `Esc` |

### Surviving Vim (First 5 Minutes)

```
1. Open:      vim myfile.txt
2. Type text: Press 'i' (enters Insert mode), type your text
3. Stop typing: Press 'Esc' (back to Normal mode)
4. Save:      Type ':w' and press Enter
5. Quit:      Type ':q' and press Enter
6. Save+Quit: Type ':wq' and press Enter
7. Quit without saving: Type ':q!' and press Enter
```

> 🚨 **Stuck in Vim?** Press `Esc` then type `:q!` and press Enter. This quits without saving.

### Navigation (Normal Mode)

```
Character Movement:
h ← left    j ↓ down    k ↑ up    l → right

Word Movement:
w → next word start     b → previous word start
e → next word end       W → next WORD (space-separated)

Line Movement:
0 → start of line       $ → end of line
^ → first non-space     g_ → last non-space

Screen Movement:
gg → first line         G → last line
5G → go to line 5       Ctrl+d → half page down
Ctrl+u → half page up   H → top of screen
M → middle of screen    L → bottom of screen

Search:
/pattern → search forward    ?pattern → search backward
n → next match               N → previous match
* → search word under cursor
```

### Editing (Normal Mode)

```
Insert Text:
i → insert before cursor     a → insert after cursor
I → insert at line start     A → insert at line end
o → new line below           O → new line above
s → delete char + insert     S → delete line + insert

Delete:
x → delete character         X → delete char before cursor
dd → delete entire line      D → delete to end of line
dw → delete word             d$ → delete to line end
d0 → delete to line start    3dd → delete 3 lines

Copy (Yank) & Paste:
yy → copy line               yw → copy word
y$ → copy to end of line     p → paste after cursor
P → paste before cursor      3yy → copy 3 lines

Undo & Redo:
u → undo                     Ctrl+r → redo

Change (delete + enter insert mode):
cw → change word             cc → change line
c$ → change to line end      ci" → change inside quotes
```

### Command Mode Essentials

```vim
:w                          " Save
:q                          " Quit
:wq  or  :x  or  ZZ        " Save and quit
:q!                         " Quit without saving

:w newfile.txt              " Save as different file
:e otherfile.txt            " Open another file

:%s/old/new/g               " Replace all occurrences in file
:%s/old/new/gc              " Replace with confirmation
:s/old/new/g                " Replace in current line only

:set number                 " Show line numbers
:set nonumber               " Hide line numbers
:set paste                  " Paste mode (disable auto-indent)
:set nopaste                " Normal mode again

:! ls                       " Run external command
:r !date                    " Insert command output into file
:r filename                 " Insert contents of another file

:split file.txt             " Horizontal split
:vsplit file.txt            " Vertical split
Ctrl+w w                    " Switch between splits
Ctrl+w q                    " Close split
```

### Visual Mode (Selecting Text)

```
v → character-wise selection (like click+drag)
V → line-wise selection (select entire lines)
Ctrl+v → block selection (select a rectangle)

After selecting:
d → delete selection        y → copy selection
> → indent right            < → indent left
~ → toggle case             U → uppercase
u → lowercase               : → command on selection
```

### Vim Configuration

```bash
# Create ~/.vimrc for permanent settings
vim ~/.vimrc
```

```vim
" ~/.vimrc — Essential settings
set number                  " Show line numbers
set relativenumber          " Relative line numbers
set tabstop=4               " Tab width
set shiftwidth=4            " Indent width
set expandtab               " Tabs → spaces
set autoindent              " Auto-indent
set smartindent             " Smart indentation
set hlsearch                " Highlight search results
set incsearch               " Incremental search
set ignorecase              " Case-insensitive search
set smartcase               " ...unless uppercase used
set wildmenu                " Tab completion menu
set mouse=a                 " Enable mouse
set clipboard=unnamedplus   " Use system clipboard
set cursorline              " Highlight current line
syntax on                   " Syntax highlighting
set background=dark         " Dark background
colorscheme desert          " Color scheme
set showmatch               " Highlight matching brackets
set encoding=utf-8          " UTF-8 encoding
set scrolloff=8             " Keep 8 lines visible above/below cursor
```

---

## Neovim — Modern Vim

**Neovim** is a refactored, modern Vim with better defaults, Lua scripting, and LSP support.

```bash
# Install
sudo apt install neovim                 # Debian/Ubuntu
sudo dnf install neovim                 # Fedora
sudo pacman -S neovim                   # Arch

# Run
nvim filename.txt

# Config location
# ~/.config/nvim/init.vim    (Vim script)
# ~/.config/nvim/init.lua    (Lua — recommended)
```

### Key Differences from Vim

| Feature | Vim | Neovim |
|---------|-----|--------|
| Config file | `~/.vimrc` | `~/.config/nvim/init.lua` |
| Plugin system | Vimscript | Lua (faster) |
| Built-in LSP | ❌ No | ✅ Yes |
| Built-in terminal | Basic | Full |
| Async support | Limited | Full |

---

## Other Editors

### Command-Line Editors

```bash
# Emacs
sudo apt install emacs
emacs filename.txt

# micro — modern terminal editor
sudo apt install micro
micro filename.txt

# Visual Studio Code (from terminal)
code filename.txt
```

### Setting Your Default Editor

```bash
# Set default editor for the system
sudo update-alternatives --config editor

# Or set via environment variable
export EDITOR=vim
export VISUAL=vim
# Add to ~/.bashrc to make permanent
echo 'export EDITOR=vim' >> ~/.bashrc
```

---

## 🏋️ Practice Exercises

1. **Nano**: Create a file `practice.txt` with nano, add 5 lines, save and exit
2. **Vim Basics**: Open vim, enter insert mode, type "Hello Vim", save and quit
3. **Vim Navigation**: Open `/etc/passwd` in vim, navigate to line 10 using `10G`
4. **Vim Editing**: In vim, delete a line (`dd`), undo (`u`), copy a line (`yy`), paste (`p`)
5. **Vim Search**: Open a large file in vim and search for a word using `/word`
6. **Vim Replace**: Use `:%s/old/new/g` to replace text in a file
7. **Vim Config**: Create a `~/.vimrc` with at least 5 settings from above
8. **Editor Choice**: Set your preferred editor as the system default

---

<p align="center">
  <a href="../07-package-management/README.md">← Previous: Package Management</a> · <a href="../README.md">🏠 Home</a> · <a href="../09-shell-scripting-fundamentals/README.md">Next: Shell Scripting Fundamentals →</a>
</p>
