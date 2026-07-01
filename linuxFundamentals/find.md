# Chapter 17 — Searching for Files (The Linux Command Line)

A quick-revision summary of finding files with `locate` and `find`.

---

## 1. The Big Idea

Two complementary tools:

| Tool | How it works | Best for |
|------|--------------|----------|
| **`locate`** | Searches a **prebuilt database** of pathnames — very fast | Finding files quickly **by name** |
| **`find`**   | Walks the directory tree **in real time** | Finding files by **any attribute** (type, size, time, permissions…) and **acting** on them |

Rule of thumb: `locate` when you just need to find something by name fast; `find` when you need power, precision, or to *do* something to the results.

---

## 2. `locate` — the Easy Way

```bash
locate bin/zip        # any path containing "bin/zip"
```

- Matches against a **database of pathnames**, so it returns instantly.
- Combine with `grep` to narrow results:

```bash
locate zip | grep bin
```

**The catch:** `locate` only knows files that existed when its database was last built. The database is created by the **`updatedb`** program, normally run **once a day** by a scheduled job. So brand-new files won't appear until then (run `updatedb` as root to refresh sooner).

> On many distributions the tool is `mlocate`; `locate` is a link to it.

---

## 3. `find` — the Basics

Give it one or more starting directories; it lists everything beneath them:

```bash
find ~                 # everything in your home directory
find ~ | wc -l         # count those items
```

Raw `find` is rarely useful alone — its power comes from combining **tests**, **operators**, and **actions**.

---

## 4. `find` — Tests (narrow *what* matches)

| Test | Matches |
|------|---------|
| `-name "pattern"` | Name matching a wildcard pattern (**quote it!**) |
| `-iname "pattern"` | Same, **case-insensitive** |
| `-type f` / `d` / `l` | Regular **file** / **directory** / symbolic **link** (also `b`, `c`) |
| `-size n` | File size (see units + `+`/`-` below) |
| `-empty` | Empty files and directories |
| `-mtime n` | Content **modified** `n` days ago |
| `-mmin n`  | Modified `n` minutes ago |
| `-newer file` | Modified more recently than `file` |
| `-perm mode` | Exact permissions (e.g. `-perm 0600`) |
| `-user name` / `-group name` | Owned by a user / group |
| `-nouser` / `-nogroup` | Owner/group no longer exists |

**Size units** (used with `-size`):

| Suffix | Unit |
|--------|------|
| `b` | 512-byte blocks (**default**) |
| `c` | bytes |
| `k` | kilobytes |
| `M` | megabytes |
| `G` | gigabytes |

Prefixes: `+` = **larger than**, `-` = **smaller than**, none = **exactly**.

```bash
find ~ -type f -name "*.jpg" -size +1M     # jpg files bigger than 1 MB
```

> For numeric tests (`-size`, `-mtime`…): `+n` means *more than n*, `-n` means *less than n*, `n` means *exactly n*.

---

## 5. `find` — Operators (combine tests logically)

| Operator | Meaning |
|----------|---------|
| *(space)* / `-and` | AND — **default** between tests |
| `-or` | OR |
| `-not` / `!` | NOT |
| `\( \)` | Grouping (parentheses must be **escaped** and space-separated) |

```bash
find ~ \( -type f -not -perm 0600 \) -or \( -type d -not -perm 0700 \)
```

> This finds files not set to `600` **or** directories not set to `700` — a quick security audit of a home directory.

---

## 6. `find` — Actions (do something with matches)

### Predefined actions

| Action | Effect |
|--------|--------|
| `-print` | Print the pathname (the **default** action) |
| `-ls` | List in `ls -dils` style |
| `-delete` | **Delete** matching files |
| `-quit` | Stop after the first match |

```bash
find ~ -type f -name "*.bak" -delete      # careful!
```

### User-defined actions with `-exec`

Run any command on each match; `{}` is replaced by the pathname, and the command ends with an **escaped** `\;`:

```bash
find ~ -type f -name "*.bak" -exec rm '{}' ';'
```

**Interactive** version — asks before each file:

```bash
find ~ -type f -name "*.bak" -ok rm '{}' ';'
```

### Efficiency: `\;` vs `+`

- Ending with `\;` runs the command **once per file** (slow for many files).
- Ending with `+` batches many pathnames into **one** command invocation:

```bash
find ~ -type f -name "*.bak" -exec ls -l '{}' +
```

---

## 7. `xargs` — Build Command Lines from Input

`xargs` reads items from standard input and appends them as arguments to a command — a classic partner for `find`:

```bash
find ~ -type f -name "*.bak" -print | xargs ls -l
```

**Filenames with spaces** break normal parsing. Use null separators on both sides:

```bash
find ~ -type f -name "*.bak" -print0 | xargs --null ls -l
```

(`-print0` on `find`, `--null`/`-0` on `xargs`.)

---

## 8. `find` — Options (control the *scope* of the search)

| Option | Effect |
|--------|--------|
| `-maxdepth n` | Descend at most `n` directory levels |
| `-mindepth n` | Ignore anything above level `n` |
| `-depth` | Process a directory's **contents before** the directory itself |
| `-mount` (`-xdev`) | Don't cross into other mounted filesystems |

---

## Quick Reference Cheat Sheet

```text
# locate (fast, name, from database)
locate bin/zip             # find by path fragment
locate zip | grep bin      # refine with grep
updatedb                   # (root) rebuild the database now

# find — basics
find ~                     # list everything under ~
find ~ | wc -l             # count items

# find — tests
-name "*.jpg"  -iname "*.JPG"     # by name (quote it)
-type f | d | l                   # file / dir / link
-size +1M   -size -100k           # bigger / smaller than
-mtime -1   -mmin -10             # modified recently
-empty  -perm 0600  -user bob     # more tests

# find — operators
-and (default)   -or   -not / !   \( ... \)   # group

# find — actions
-print (default)   -ls   -delete   -quit
-exec cmd {} \;     # run once per file
-exec cmd {} +      # batch into fewer runs
-ok  cmd {} \;      # ask before each

# xargs
find ... -print  | xargs cmd
find ... -print0 | xargs --null cmd   # handles spaces

# find — scope
-maxdepth n   -mindepth n   -depth   -mount
```

---

### Mental model to remember

> **`locate`** = fast lookup by name from a daily database (may miss brand-new files).
> **`find`** = live search you build from three kinds of pieces: **tests** (what to match) + **operators** (`-and`/`-or`/`-not`, grouped with `\( \)`) + **actions** (what to do, default `-print`).
> To act on matches, reach for **`-exec … {} \;`** (or `{} +` / **`xargs`** for speed) — and use `-print0 | xargs --null` when filenames contain spaces.
