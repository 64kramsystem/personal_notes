# Editors

- [Editors](#editors)
  - [Notes](#notes)
  - [Visual Studio Code](#visual-studio-code)
    - [Bindings](#bindings)
      - [Extensions](#extensions)
  - [Meld](#meld)
  - [Less](#less)
    - [bindings](#bindings-1)
  - [Anki](#anki)
  - [Vim](#vim)
    - [Misc](#misc)
    - [settings](#settings)
    - [multi-files](#multi-files)
    - [selection/search/sort](#selectionsearchsort)
      - [multi-line selection](#multi-line-selection)
    - [examples](#examples)
    - [history](#history)
    - [vimdiff](#vimdiff)
    - [modeline](#modeline)
  - [Other program shortcuts](#other-program-shortcuts)
  - [Windows](#windows)
  - [Laptops](#laptops)

## Notes

The document also includes shortcuts for non-editors.

The VIM section requires some cleanup.

## Visual Studio Code

### Bindings

- `Ctrl+B`				            Show/hide sidebar
- `Ctrl+J`				            Show/hide console
- ``Ctrl+` ``                 Show/hide terminal
- `Ctrl+Shift+V`              Open file preview in tab
- `Ctrl+K, V`                 Show file preview (side-by-side)
- `Ctrl+K, R`                 Open file directory in system file manager ("[R]eveal")
- `Ctrl+K, T`                 Open file directory in file explorer (custom)
- `Ctrl+\`                    Split editor vertically
- `Ctrl+K, Ctrl+\`            Split editor horizontally

- `Ctrl+R`                    Quick switch workspace
- `Ctrl+K, M`                 Select language mode

- `Ctrl+Shift+O`              Go to symbol in current file (includes Markdown headers)
- `Ctrl+T`                    Go to symbol in workspace
- `Alt+(←/→)`               Navigate through history
- `Ctrl+(↓/↑)`                 Scroll the viewport without moving the cursor
- `F8`                        Show next warning/error
- `Ctrl+@`                    Open link in browser (custom)

- `Ctrl+K, F`                 Go to parent fold (parent of current level) (custom)
- `Ctrl+K, G`                 Go to next folding range (next sibling of current level) (custom)
- `Ctrl+Shift` + `[`/`]`      Fold/unfold
- `Ctrl+K, Ctrl+0`            Fold all
- `Ctrl+K, Ctrl+J`            Unfold all (expand)

- `Alt+Shift+(↓/↑)`           Column (box) selection
- `Alt+C`                     Search box: Match case
- `Alt+R`                     Search box: Regex
- `Alt+L`                     Search box: Switch to replace inside selection (after `Ctrl+H`); it doesn't support block selection↵
                              See option `Auto Find in Selection` (can be annoying).
- `Alt+Shift+f`               Search in selected directory
- `Ctrl+Shift+f`              Search in workspace

- `Ctrl+Space`                Variables autocompletion (at least in Shell script syntax)
- `Ctrl+.`                    Quick fix(es) code
- `Ctrl+K, Ctrl+T`            Run test (custom)

- `Ctrl+Shift+C`              Copy file relative path (changed: binding, also work in editor)
- `Ctrl+Shift+Alt+C`          Copy file full path (changed: binding, also work in editor)
- `Ctrl+Alt+R`                Open folder of explorer's current file

- `Ctrl+K, S`                 Save without formatting

- `F10/F11`                   Step over/into
- `Shift+F11`                 Step out
- `F5`                        Start/Continue
- `Shift+F5`                  Stop
- `Ctrl+F5`                   Run without debug
- `F9`                        Toggle breakpoint

- `Ctrl+K, Ctrl+P`            Du(p)licate (copy) selection!! (custom)
- `Shift+Del`                 Delete line
- `Ctrl+Bksp`                 Delete last word (opt. with trailing whitespace)

#### Extensions

- `Ctrl+K, Ctrl+(L/U)`	      Change case to Upper/Lower
- `Ctrl+K, Ctrl+S`	          Change case to Snake

- `Ctrl+Alt+I`                Increment selection, inside column selection

- `Ctrl+K, Ctrl+X`		        Trim trailing whitespace

- `Ctrl+K, Ctrl+A`            Align by regex (modified)

- `Ctrl+Shift+G, B`           GitLens: Blame file
- `Ctrl+Shift+G, C`           GitLens: Show commit details (and operations)

## Meld

`Alt + PgUp/Down`           Switch panel
`Alt + Shift + ←/→`        Pull change from Left/Right
`Alt + ←/→`                Pust change to Left/Right
`Alt + Delete`              Delete
`Alt + [/]`                 Copy above Left/Right

## Less

### bindings

`Shift + F`					        Follow (like tail -f)

## Anki

`Ctrl + E`                  Add new card (it calls it "note"); WATCH OUT! The preselect deck is broken
`Ctrl + M, M`               Insert Matjax

## Vim

### Misc

`wq! <filename>`            Save to <filename> and quit
`:qa`                       Quit `a`ll
`:b(p|n)[#]`                Move `p`revious/`n`ext buffer [of `#` buffers]

`w/b`                       Move cursor words (forward/back)

`Ctrl + r`                  Redo
`Ctrl + p/n`                Auto-completion (search `p`revious/`n`ext)

`vim -u NONE`               Ignore configuration files

`:<params><command>`        Execute a command with the given params, eg. `:3,5bd` executes `bd` in the range `3,5`

Comment multiple lines: http://stackoverflow.com/questions/1883896/what-is-your-favorite-way-to-comment-several-lines-in-vim

### settings

`~/.vimrc`                  Startup cfg; store commands without ':'

`:set [no]ignorecase`
`:set [no]number`           Line numbers
`:set [no]wrap`             Search wrapping
`:syntax (on|off)`          Syntax hightlighting
`:set tabstop=x`            Tab size
`:set [no]expandtab`        Soft tabs
`:set shiftwidth=x`         Indent

### multi-files

`:Explore`                  File selection dialog
`:sav`                      Save as
`:buffers`                  Show all the buffers
`:bd bufNum`                Delete a buffer
`:3,5bd`                    Delete buffers in range

### selection/search/sort

`mc`                        Mark a line with the char “c”
`'c`                        Reference line c
`.`                         Current line
`%`                         Entire file
`$`                         Last line

`:itvs/patts/pattr/[c][i]`  replace, [c]onfirmation, [i]gnore case; see selection.examples for itv coding

`:itv sort sort lines for itv`

In order to type Ctrl chars, e.g. when searching/replacing, type Ctrl+V then the character, e.g.:

`:%s/<Ctrl-V><M>//g`

#### multi-line selection

Type `Ctrl+V` in order to enter "Visual mode"

Visual modes:

`v`                         Regular selection
`Ctrl+v`                    Rectangular selection

Search and replace. Vim will automatically fill the range, so that `%` must not be used:

`:s/search/replace`

Shift block of lines:

- `:set sw n`               Set the shift size as n
- (select the lines)
- `<` or `>`                Shift the lines


Add/remove characters
during change, only the first line will show thee difference; it will also take a small time after `Esc`

- (select the lines)
- type `Shift+I`
- make the change, then tap `Esc` to apply

### examples

`g'a`                       Goto line marked a
`:'a,$/ss/rr/g`             Replace ss with rr in the interval from line marked a to the end
`:%s/^M//g`                 Strip dos newline; to enter ^M, use Ctrl+V,Ctrl+M

### history

`q(:|/)`                    Visual history of commands/searches
`(:|/)[prefix]`             Clicking up/down goes through the history; if prefix is present, is used as filter

### vimdiff

`:set diffopt+=iwhite`      `i`gnore `white`space
`Ctrl+W,Ctrl+W`             Change frame


Remotely edit a file; the path after the port is relative to home, so the example file is in home:

`vim scp://user@myserver:port/restore_deleted_bookings.rb`

### modeline

Set in vimrc:

```
set modeline
set modelines=5
```

Format. put on top, or bottom of file:

`vim: setting=value{:setting=value}+`

## Other program shortcuts

Caja:

- `F6`                      Switch between side-to-side panels

Evince:

- `Ctrl+L`                  Go to page

Feh:

- `Ctrl+Del`                Delete image

Firefox:

- `Ctrl+Shift+O`            Bookmarks editor
- `Ctrl+., <number>`        Containers extension: display menu, and open new tab with given container

Google meet:

- `Ctrl+Alt+H`              Raise hand
- `Ctrl+D`                  Dis/Enable microphone

htop:

- `h`: help
- `H`: show/hide process threads

MATE:

- `Mod4 + Alt + ←/→`          Tile window top left/right
- `Mod4 + Alt + Shift + ←/→ ` Tile window bottom left/right
- `Alt + F8`                   Resize window

Meld:

- `Alt + ←/→`              Push diff block to the given direction
- `Alt + Shift + ←/→`      Pull diff block from the opposite of the given direction

Slack:

- `→`/`T`                  When on a thread message, open the thread panel
- `Ctrl+.`                  Open/close the right panel

Telnet:

- `Ctrl+]` => `q`           Terminate session

Thunderbird:

`Ctrl+Shift+C`             Add CC when composing an email

Tilix:

- `Ctrl+Shift+I`            Synchronize input
- `Alt+<direction>`         Move between terminals
- `Alt+Shift+<direction>`   Resize terminal

VMWare:

- `Ctrl+B`                 Start
- `Ctrl+J`                 Suspend
- `Cltr+E`                 Shutdown

## Windows

Desktop:

- `Win + Tab`              Window/desktop switcher
- `Win + Ctrl + ←/→`       Switch desktop

Windows Terminal:

- `Alt + Shift + +`        Split to right
- `Alt + Shift + -`        Split to bottom
- `Ctr + Shift + t`        New tab

## Laptops

| Laptop                | BIOS  | Boot  |
| --------------------- | :---: | :---: |
| Yoga 7 Gen 7 AMD      |  F2   |  F12  |
| Chuwi Miniboox X N100 |  F2   |   ?   |
| Odroid H2             |   ?   |  F7   |

Minibook S/N (old one): `ZMinBXHY6H231000191`
