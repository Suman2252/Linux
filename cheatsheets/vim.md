# Vim Cheatsheet

## Modes
| Key | Mode | Description |
|-----|------|-------------|
| `Esc` | Normal | Navigate & command |
| `i` | Insert | Type text (before cursor) |
| `a` | Insert | Type text (after cursor) |
| `o` | Insert | New line below |
| `O` | Insert | New line above |
| `v` | Visual | Select characters |
| `V` | Visual Line | Select lines |
| `Ctrl+v` | Visual Block | Select block |
| `:` | Command | Execute commands |

## Navigation
| Key | Action |
|-----|--------|
| `h j k l` | Left, Down, Up, Right |
| `w` / `b` | Next / previous word |
| `0` / `$` | Start / end of line |
| `gg` / `G` | Start / end of file |
| `Ctrl+d/u` | Half page down / up |
| `Ctrl+f/b` | Full page down / up |
| `{` / `}` | Previous / next paragraph |
| `%` | Matching bracket |
| `:42` | Go to line 42 |

## Editing
| Key | Action |
|-----|--------|
| `x` | Delete character |
| `dd` | Delete line |
| `dw` | Delete word |
| `d$` or `D` | Delete to end of line |
| `yy` | Copy (yank) line |
| `yw` | Copy word |
| `p` / `P` | Paste after / before |
| `u` | Undo |
| `Ctrl+r` | Redo |
| `r` | Replace character |
| `cw` | Change word |
| `cc` | Change line |
| `ci"` | Change inside quotes |
| `.` | Repeat last action |
| `>>` / `<<` | Indent / unindent |

## Search & Replace
| Command | Action |
|---------|--------|
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` / `N` | Next / previous match |
| `*` | Search word under cursor |
| `:%s/old/new/g` | Replace all in file |
| `:%s/old/new/gc` | Replace with confirm |
| `:s/old/new/g` | Replace in current line |

## File Operations
| Command | Action |
|---------|--------|
| `:w` | Save |
| `:q` | Quit |
| `:wq` or `:x` | Save + quit |
| `:q!` | Quit without saving |
| `:e file` | Open file |
| `:w file` | Save as |

## Windows & Tabs
| Command | Action |
|---------|--------|
| `:split` / `:sp` | Horizontal split |
| `:vsplit` / `:vs` | Vertical split |
| `Ctrl+w w` | Switch windows |
| `Ctrl+w hjkl` | Navigate windows |
| `:tabnew` | New tab |
| `gt` / `gT` | Next / previous tab |

## Tips
| Command | Action |
|---------|--------|
| `ciw` | Change inner word |
| `ci(` | Change inside parentheses |
| `di"` | Delete inside quotes |
| `da"` | Delete around quotes |
| `yap` | Yank paragraph |
| `=G` | Auto-indent to end |
| `ggVG` | Select entire file |
| `:set number` | Show line numbers |
| `:noh` | Clear search highlight |
