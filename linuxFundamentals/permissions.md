# Chapter 9 — Permissions (The Linux Command Line)
---

## 1. The Big Idea

Linux is a **multiuser** system. To keep users from interfering with each other (and to keep the system safe), every file and directory has:

- an **owner** (a user),
- an associated **group**, and
- a set of **permissions** controlling who can do what.

Relevant system files:

| File | Holds |
|------|-------|
| `/etc/passwd` | User accounts (name, UID, GID, home, shell) |
| `/etc/group`  | Group definitions |
| `/etc/shadow` | Encrypted passwords |

Check your own identity with:

```bash
id
```

It shows your **uid** (user), **gid** (primary group), and the **groups** you belong to.

---

## 2. Reading File Attributes (`ls -l`)

```
-rw-r--r--  1  alice  users  8512  Jun 19 10:00  notes.txt
```

The first 10 characters are the key:

```
 -      rw-      r--      r--
type   owner    group    other (world)
```

### File type (first character)

| Char | Meaning |
|------|---------|
| `-`  | Regular file |
| `d`  | Directory |
| `l`  | Symbolic link (its permissions are ignored; the target's apply) |
| `c`  | Character device |
| `b`  | Block device |

### The three permission triads

Each triad is `r w x` for **owner**, then **group**, then **other**:

| Symbol | On a file | On a directory |
|--------|-----------|----------------|
| `r` (read) | Read/copy contents | List contents (`ls`) — *needs `x` too to be useful* |
| `w` (write) | Modify contents | Create/delete/rename files inside |
| `x` (execute) | Run as a program | Enter the directory (`cd`) and access files within |

> Note: To create or delete a file **inside** a directory, you need `w` **and** `x` on that directory — not write permission on the file itself.

---

## 3. `chmod` — Change Permissions

Only the **file's owner** or the **superuser** can change a file's mode.

### Octal notation

Each permission is a number; add them per triad:

| Permission | Value |
|------------|-------|
| `r` read   | 4 |
| `w` write  | 2 |
| `x` execute| 1 |

So one digit (0–7) = sum of permissions, and three digits = owner/group/other.

| Octal | Symbolic | Common use |
|-------|----------|------------|
| `777` | `rwxrwxrwx` | Everyone full access (rarely wise) |
| `755` | `rwxr-xr-x` | Programs & directories: owner edits, others run/read |
| `700` | `rwx------` | Private to owner only |
| `644` | `rw-r--r--` | Normal data file: owner edits, others read |
| `600` | `rw-------` | Private data file |

```bash
chmod 644 notes.txt
chmod 755 script.sh
```

### Symbolic notation

`chmod [who][operator][permission]`

- **Who:** `u` user/owner · `g` group · `o` other · `a` all
- **Operator:** `+` add · `-` remove · `=` set exactly
- **Permission:** `r` `w` `x`

```bash
chmod u+x script.sh      # give owner execute
chmod go-rw secret.txt   # remove read/write from group & other
chmod a+r file.txt       # everyone can read
chmod o= file.txt        # remove all "other" permissions
chmod u+x,g=rw,o-rwx f   # combine, comma-separated
```

> Symbolic is handy when you want to flip **one** bit without recomputing the whole octal number.

---

## 4. `umask` — Default Permissions

Controls which permission bits are **masked off** (removed) when new files are created.

```bash
umask        # shows current mask, e.g. 0022
```

- Files are normally created with base `666`, directories with `777`.
- The mask is **subtracted** from those bases.
- Default `0022` → files become `644`, directories `755`.

```bash
umask 0077   # new files private to owner (600 / 700)
```

> The leading `0` in `0022` is the **fourth (special-permissions) digit** — see the next section.

---

## 5. Some Special Permissions

Beyond the usual `r w x`, the octal mode has an optional **fourth digit** (placed in front) controlling three special bits. Values add up just like the others:

| Bit | Value | Applies to |
|-----|-------|------------|
| **setuid** | 4 | Executable files |
| **setgid** | 2 | Executable files **and** directories |
| **sticky** | 1 | Directories |

### setuid (`u+s`)

When set on an **executable**, the program runs with the privileges of the **file's owner** instead of the user who launched it. Commonly used to let ordinary users run a program as **root** for a specific task.

```bash
chmod 4755 program     # octal: leading 4 = setuid
chmod u+s program      # symbolic
```

In `ls -l`, the owner's `x` becomes **`s`**: `-rwsr-xr-x`.

### setgid (`g+s`)

- **On an executable:** the program runs with the privileges of the file's **group**.
- **On a directory:** new files created inside **inherit the directory's group** (rather than the creating user's group) — very useful for shared folders.

```bash
chmod 2755 shared_dir  # octal: leading 2 = setgid
chmod g+s shared_dir   # symbolic
```

In `ls -l`, the group's `x` becomes **`s`**: `drwxr-sr-x`.

### Sticky bit (`+t`)

A legacy of old Unix, now used mainly on **shared directories**. When set, only a file's **owner** (or the directory's owner, or root) may delete or rename files inside it — even if others have write access. This is why everyone can write to `/tmp` but can't delete each other's files.

```bash
chmod 1777 /shared/tmp  # octal: leading 1 = sticky
chmod +t /shared/tmp    # symbolic
```

In `ls -l`, the "other" `x` becomes **`t`**: `drwxrwxrwt`.

> **Capital vs lowercase letter:** an uppercase `S` or `T` in the listing means the special bit is set but the underlying `x` is **not** — usually a mistake worth fixing.

---

## 6. Changing Identities

Sometimes you need to act as another user (often the superuser/root).

### `su` — substitute user

```bash
su              # become root (prompts for root's password)
su -            # become root WITH a full login environment
su -l user      # become another user with login environment
su -c 'command' # run a single command as another user
```

### `sudo` — execute one command as another user

```bash
sudo command    # run command as root (prompts for YOUR password)
sudo -i         # start an interactive root session
sudo -l         # list what commands you're allowed to run
```

Key differences:

| | `su` | `sudo` |
|---|------|--------|
| Password asked | **Target** user's (root's) | **Your own** |
| Scope | Full shell as the other user | Usually a single, **pre-authorized** command |
| Config | — | Admin-defined in `/etc/sudoers` |

---

## 7. `chown` — Change Owner and/or Group

Requires **superuser** privileges. Syntax: `chown owner:group file`.

```bash
chown bob file.txt          # change owner to bob
chown bob:admins file.txt   # change owner AND group
chown :admins file.txt      # change group only
chown bob: file.txt         # owner bob, group = bob's login group
```

---

## 8. `chgrp` — Change Group (older command)

Changes only the group. Useful when you own the file and belong to the target group.

```bash
chgrp admins file.txt
```

---

## 9. `passwd` — Change a Password

```bash
passwd          # change your own password
passwd user     # change another user's password (superuser only)
```

Good passwords are long, mixed-case, with numbers and symbols, and not dictionary words. `passwd` may reject weak ones.

---

## Quick Reference Cheat Sheet

```text
r=4  w=2  x=1            # octal values
755 = rwxr-xr-x          # typical for scripts/dirs
644 = rw-r--r--          # typical for data files
600 = rw-------          # private files

chmod 755 file           # set by octal
chmod u+x file           # add owner execute (symbolic)
chmod go-w file          # remove write from group & other

umask 0022               # default mask (files 644, dirs 755)

# Special bits (leading 4th octal digit): setuid=4 setgid=2 sticky=1
chmod 4755 prog          # setuid  -> -rwsr-xr-x
chmod 2755 dir           # setgid  -> drwxr-sr-x (group inherited in dir)
chmod 1777 dir           # sticky  -> drwxrwxrwt (only owner deletes)

su -                     # full root shell
sudo cmd                 # one command as root (your password)

chown user:group file    # change ownership (root)
chgrp group file         # change group
id                       # who am I / my groups
```

---
