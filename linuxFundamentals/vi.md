# Chapter 12 — A Gentle Introduction to vi (The Linux Command Line)

A quick-revision summary of surviving and working in the `vi`/`vim` text editor.

---

## 1. The Big Idea

`vi` is a small, fast, **always-available** text editor. Knowing it matters because:

- It's installed on virtually every Unix-like system, even minimal ones.
- It runs entirely in the terminal — no GUI, works over SSH.
- When something breaks and only a basic shell is available, `vi` is often the **only** editor present.

A little background: `vi` was written by **Bill Joy**. Most Linux systems ship **`vim`** ("vi improved"), an enhanced clone — often `vi` is just a link to `vim`.

> The hardest part of `vi` is that it has **modes**. A key does different things depending on the mode you're in. Master modes and the rest follows.

---

## 2. Starting and Stopping vi

```bash
vi filename     # open (or create) a file
vim filename    # the improved version, if installed
```

If you start with no filename, `vi` opens an empty buffer.

**Leaving `vi`** (type from command mode — press `Esc` first if unsure):

| Command | Effect |
|---------|--------|
| `:w`  | Write (save) |
| `:q`  | Quit (only if no unsaved changes) |
| `:q!` | Quit **discarding** changes (force) |
| `:wq` | Write **and** quit |
| `ZZ`  | Write and quit (shortcut, no colon) |

> Stuck? Press **`Esc`** a couple of times to get back to command mode, then `:q!` to bail out without saving.

---

## 3. The Three Modes

`vi` always sits in one of three modes:

| Mode | How you enter it | What it's for |
|------|------------------|----------------|
| **Command** (normal) | Default; press `Esc` to return | Move around, delete, copy, paste — keys are *commands* |
| **Insert** | `i` (and friends: `a`, `o`…) | Actually typing text |
| **Last-line / ex** | `:` | File-wide commands: save, quit, search-and-replace |

The golden rule: **`Esc` always returns you to command mode.**

Enter insert mode, type, then `Esc` back out:

```
i      → insert before the cursor
...type your text...
Esc    → back to command mode
```

---

## 4. Moving the Cursor (command mode)

| Key | Moves |
|-----|-------|
| `h` `j` `k` `l` | left, down, up, right (arrow keys also work) |
| `0` (zero) | Start of the line |
| `^` | First non-blank character of the line |
| `$` | End of the line |
| `w` | Forward to the start of the next word |
| `W` | Next word, treating punctuation as part of the word |
| `b` | Back to the start of the previous word |
| `G` | Last line of the file |
| `NG` | Line number `N` (e.g. `5G`) |
| `gg` | First line of the file (vim) |
| `Ctrl-F` / `Ctrl-B` | Page forward / backward |

> Many commands accept a **count** prefix — `5j` moves down 5 lines.

---

## 5. Basic Editing

### Entering insert mode in different places

| Key | Effect |
|-----|--------|
| `i` | Insert **before** the cursor |
| `a` | Append **after** the cursor |
| `A` | Append at the **end of the line** |
| `o` | Open a new line **below** and insert |
| `O` | Open a new line **above** and insert |

### Undo

| Key | Effect |
|-----|--------|
| `u` | Undo the last change (press repeatedly to keep undoing) |

### Joining lines

| Key | Effect |
|-----|--------|
| `J` | Join the next line onto the current one |

---

## 6. Deleting, Cutting, Copying, Pasting

In `vi`, **deleting also "cuts"** — the removed text goes into a buffer you can paste back.

### Delete (`d`) and `x`

| Command | Deletes |
|---------|---------|
| `x`  | The character under the cursor |
| `dd` | The current line |
| `Ndd` | `N` lines (e.g. `3dd`) |
| `dw` | From cursor to start of next word |
| `d$` | From cursor to end of line |
| `d0` | From cursor to start of line |
| `dG` | From cursor to end of file |
| `d1G` | From cursor to start of file |

### Yank (`y`) = copy

Same patterns as `d`, but copies instead of removing:

| Command | Copies |
|---------|--------|
| `yy` | The current line |
| `Nyy` | `N` lines |
| `yw` | A word |
| `y$` | To end of line |

### Paste

| Command | Effect |
|---------|--------|
| `p` | Paste **after** the cursor / **below** the current line |
| `P` | Paste **before** the cursor / **above** the current line |

> Workflow: `dd` (cut a line) → move → `p` (paste it) = move a line. `yy` → `p` = duplicate a line.

---

## 7. Search and Replace

### Searching

| Command | Effect |
|---------|--------|
| `/text` | Search **forward** for `text`, then Enter |
| `?text` | Search **backward** |
| `n` | Repeat the search in the same direction |
| `N` | Repeat in the opposite direction |
| `f c` | Within the current line, jump to next character `c` |

### Global search and replace (ex command)

```
:%s/old/new/g
```

| Piece | Meaning |
|-------|---------|
| `:`   | Enter last-line mode |
| `%`   | Apply to **every line** (omit it to act on the current line only) |
| `s`   | Substitute |
| `/old/new/` | Replace `old` with `new` |
| `g`   | **Global** — every match on each line, not just the first |
| `c`   | (add it: `…/gc`) ask for **confirmation** on each match |

You can target a **range** of lines instead of `%`:

```
:1,5s/old/new/g      # only lines 1 through 5
```

---

## 8. Editing Multiple Files

Open several at once:

```bash
vi file1 file2 file3
```

| Command | Effect |
|---------|--------|
| `:n` | Switch to the **next** file (must save or use `:n!`) |
| `:N` | Switch to the **previous** file |
| `:buffers` | List open buffers |
| `:buffer N` | Switch to buffer number `N` |
| `:e filename` | Open **another** file for editing |
| `:r filename` | **Read** (insert) a file's contents into the current one at the cursor |

> `:r` is handy for pulling boilerplate into a document without leaving `vi`.

---

## Quick Reference Cheat Sheet

```text
# Modes
Esc                back to command mode (your safety key)
i a A o O          enter insert mode (before/after/EOL/below/above)
:                  last-line (ex) command mode

# Quit / save
:w   :wq   ZZ      write / write+quit / write+quit
:q   :q!           quit / quit discarding changes

# Move
h j k l            left down up right
0  ^  $            line start / first non-blank / line end
w  b               next word / previous word
G  NG  gg          last line / line N / first line
Ctrl-F  Ctrl-B     page down / up

# Edit
u                  undo
x                  delete character
dd  dw  d$  dG     delete line / word / to EOL / to EOF
yy  yw  y$         yank (copy) line / word / to EOL
p   P              paste after/below  /  before/above
J                  join next line

# Search & replace
/text   ?text      search forward / backward
n   N              repeat search  same / opposite direction
:%s/old/new/g      replace everywhere
:%s/old/new/gc     replace everywhere, with confirmation
:1,5s/old/new/g    replace within lines 1-5

# Multiple files
vi f1 f2           open several
:n  :N             next / previous file
:e file            open another file
:r file            read a file into the current one
```

---

### Mental model to remember

> `vi` has **modes**: in **command mode** the keyboard is a set of commands; press `i` (or `a`/`o`) to start **typing**; press **`Esc`** to stop. Anything starting with **`:`** is a file-wide command (save, quit, search-and-replace).
> When in doubt: **`Esc`**, then `:q!` to escape, or `:wq` to save and leave.
