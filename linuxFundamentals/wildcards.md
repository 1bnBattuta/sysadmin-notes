# Wildcards (Globbing)

Wildcards — also called **glob patterns** — are expressions used by the shell to match filenames and paths. The shell expands them *before* passing arguments to a command, so they work with virtually any command (`ls`, `cp`, `rm`, `mv`, `find`, etc.).

> **Wildcards vs Regex:** Wildcards are not regular expressions. They share a few symbols (`*`, `?`, `[…]`) but the meaning differs. Commands like `grep`, `sed`, and `awk` expect regex, not globs. If you need a literal `*` or `?` in those contexts, quote or escape it.

---

## Basic Wildcards

| Pattern | Meaning | Example |
|---------|---------|---------|
| `*` | Any sequence of characters (including none) | `*.html` → all HTML files |
| `?` | Exactly one arbitrary character | `file?.txt` → `file1.txt`, `fileA.txt`, but not `file10.txt` |
| `\` | Escape character — treats the next character literally | `\*` → a literal `*`, `\\` → a literal `\` |

---

## Character Classes

Square brackets match **one** character from a set or range.

| Pattern | Meaning | Example |
|---------|---------|---------|
| `[abc]` | `a`, `b`, or `c` (no commas) like regex | `file[abc].txt` → `filea.txt`, `fileb.txt`, `filec.txt` |
| `[a-d]` | Any character in the range `a` to `d` | `[a-d]*` → everything starting with a, b, c, or d |
| `[!a-d]` or `[^a-d]` | Any character **not** in the range | `[!0-9]*` → files not starting with a digit |

---

## POSIX Character Classes

Used inside an extra pair of brackets: `[[:class:]]`.

| Class | Matches |
|-------|---------|
| `[[:alpha:]]` | Letters (a–z, A–Z) |
| `[[:digit:]]` | Digits (0–9) |
| `[[:alnum:]]` | Letters and digits |
| `[[:upper:]]` | Uppercase letters |
| `[[:lower:]]` | Lowercase letters |
| `[[:space:]]` | Whitespace (space, tab, newline, etc.) |
| `[[:punct:]]` | Punctuation characters |

Example: `ls [[:upper:]]*` lists files whose name starts with an uppercase letter.

---

## Quick Tips

- **Globbing is done by the shell**, not by the command. Running `ls *.txt` means the shell replaces `*.txt` with matching filenames, then passes that list to `ls`.
- **Hidden files** (starting with `.`) are not matched by `*` unless you set `shopt -s dotglob` in Bash.
- **No matches?** By default Bash passes the pattern as a literal string when nothing matches. Use `shopt -s nullglob` to get an empty result instead.
- **Extended globs:** enable with `shopt -s extglob` to get patterns like `?(…)`, `+(…)`, `*(…)`, `@(…)`, and `!(…)` for optional, one-or-more, zero-or-more, exact-one, and negation matching.
