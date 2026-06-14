# Basic File Commands

Essential commands for moving, copying, and managing files and directories in Linux.

---

## `cp` — Copy Files and Directories

```bash
cp source destination
```

| Option | What it does |
|--------|--------------|
| `-r` | Copy directories recursively (required for directories) |
| `-i` | Prompt before overwriting an existing file |
| `-n` | Never overwrite an existing file |
| `-u` | Copy only when the source is newer than the destination or the destination is missing |
| `-v` | Verbose — print each file as it's copied |
| `-p` | Preserve permissions, ownership, and timestamps |
| `-a` | Archive mode — equivalent to `-rpL`, preserves everything (links, permissions, etc.) |
| `-l` | Create hard links instead of copying |
| `-s` | Create symbolic links instead of copying |

Examples:

```bash
cp file.txt backup/                    # copy file into backup/
cp -r project/ project_backup/         # copy an entire directory
cp -iv *.pdf ~/Documents/              # copy all PDFs, verbose, ask before overwriting
cp -a /var/www/ /var/www_backup/       # full archive copy preserving everything
cp -u src/*.js dist/                   # only copy files that changed
```

---

## `mv` — Move or Rename Files

```bash
mv source destination
```

| Option | What it does |
|--------|--------------|
| `-i` | Prompt before overwriting |
| `-n` | Never overwrite |
| `-u` | Move only when the source is newer or the destination is missing |
| `-v` | Verbose — print what's being moved |
| `-f` | Force — don't prompt, even if overwriting (default on most systems) |

Examples:

```bash
mv old_name.txt new_name.txt           # rename a file
mv file.txt ~/Documents/              # move to another directory
mv -i *.jpg images/                   # move all JPGs, ask before overwriting
mv -v project/ ~/backups/             # move directory, show what happens
```

> **Tip:** `mv` works across filesystems — the shell copies then deletes when source and destination are on different partitions.

---

## `rm` — Remove Files and Directories

```bash
rm target
```

| Option | What it does |
|--------|--------------|
| `-r` | Remove directories and their contents recursively |
| `-i` | Prompt before every removal |
| `-I` | Prompt once if removing more than 3 files or using `-r` (less noisy than `-i`) |
| `-f` | Force — never prompt, ignore nonexistent files |
| `-v` | Verbose — print each file as it's removed |
| `-d` | Remove empty directories (same as `rmdir`) |

Examples:

```bash
rm file.txt                            # delete a file
rm -r old_project/                     # delete a directory and everything inside
rm -ri temp/                           # delete recursively, confirm each file
rm -I *.log                            # prompt once before bulk-deleting logs
```

> **Warning:** `rm` is permanent — there is no trash can. Double-check before using `-rf` on directories.

---

## `mkdir` — Create Directories

```bash
mkdir directory_name
```

| Option | What it does |
|--------|--------------|
| `-p` | Create parent directories as needed, no error if they exist |
| `-v` | Verbose — print each directory created |
| `-m` | Set permissions on creation (e.g., `-m 755`) |

Examples:

```bash
mkdir images                           # create a single directory
mkdir -p src/components/ui             # create the full path in one go
mkdir -pv project/{src,tests,docs}     # create multiple subdirectories at once
```

---

## `touch` — Create Files or Update Timestamps

```bash
touch filename
```

| Option | What it does |
|--------|--------------|
| `-a` | Change only the access time |
| `-m` | Change only the modification time |
| `-t` | Set a specific timestamp (format: `[[CC]YY]MMDDhhmm[.ss]`) |
| `-c` | Don't create the file if it doesn't exist |

Examples:

```bash
touch newfile.txt                      # create an empty file (or update its timestamp)
touch -c report.log                    # update timestamp only if the file exists
touch file1.txt file2.txt file3.txt    # create multiple files at once
```

---

## `ln` — Create Links

```bash
ln target link_name          # hard link
ln -s target link_name       # symbolic link
```

| Option | What it does |
|--------|--------------|
| `-s` | Create a symbolic (soft) link instead of a hard link |
| `-f` | Overwrite the destination if it already exists |
| `-v` | Verbose |
| `-r` | Create a relative symbolic link |

Examples:

```bash
ln -s /etc/nginx/nginx.conf ~/nginx.conf      # symlink to a config file
ln -sf /usr/bin/python3 /usr/bin/python        # force-update a symlink
ln -sr ../shared/utils.sh ./utils.sh           # relative symlink
```

> **Hard vs Soft links:** A hard link is another name for the same data on disk — it survives if the original is deleted. A symbolic link is a pointer to a path — it breaks if the target is moved or removed.

---

## Handy Patterns

```bash
# Dry-run a dangerous rm by listing first
ls target_pattern          # see what matches before deleting
rm -ri target_pattern      # then delete interactively

# Rename files in bulk (requires rename or a loop)
for f in *.jpeg; do mv "$f" "${f%.jpeg}.jpg"; done

# Copy structure without files
find src/ -type d -exec mkdir -p dest/{} \;

# Backup with a timestamp
cp -a project/ "project_backup_$(date +%F)/"
```
