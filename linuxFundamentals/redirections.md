# Linux Command Line Redirections

Every process in a UNIX-based system is connected to three standard streams:

| Stream | Name | File Descriptor | Default Destination |
|--------|------|:-:|-----|
| Standard Input | `stdin` | 0 | Keyboard |
| Standard Output | `stdout` | 1 | Screen (terminal) |
| Standard Error | `stderr` | 2 | Screen (terminal) |

These are just files — everything in Linux is. By default, `stdin` reads from the keyboard, while `stdout` and `stderr` both print to the terminal. Although they share the same destination, `stdout` and `stderr` are separate streams: one carries results, the other carries error messages. This separation is what makes redirections powerful.

**Redirection** is the ability to re-wire these streams — sending output to a file instead of the screen, reading input from a file instead of the keyboard, or connecting commands together.

---

## Redirecting Standard Output (`>`, `>>`)

The `>` operator sends `stdout` to a file instead of the screen.

```bash
ls -l /usr/bin > ls-output.txt
```

This creates `ls-output.txt` (or **overwrites** it if it already exists) and writes the output of `ls` into it. Nothing appears on screen because `stdout` has been re-wired. If the command produces an error, that message still shows up in the terminal because `stderr` remains wired to the screen.

To **append** instead of overwriting, use `>>`:

```bash
echo "first line" > log.txt       # creates or overwrites
echo "second line" >> log.txt     # appends to the end
```

A useful trick: `> empty.txt` truncates an existing file to zero bytes (or creates an empty one). This is handy for clearing log files without deleting them.

---

## Redirecting Standard Error (`2>`, `2>>`)

There is no dedicated symbol for `stderr` redirection. Instead, the shell uses a general notation where you prefix `>` with the file descriptor number. Since `stderr` is file descriptor 2:

```bash
ls -l /nonexistent 2> errors.txt
```

This sends only error messages to `errors.txt`. Normal output (if any) still prints to the screen. Appending works the same way:

```bash
ls -l /nonexistent 2>> errors.txt
```

---

## Redirecting Both stdout and stderr

There are two common ways to send both streams to the same file.

**Method 1 — `&>` (preferred, Bash 4+):**

```bash
ls -l /usr/bin /nonexistent &> all-output.txt
```

The `&>` operator is shorthand for "redirect both file descriptors 1 and 2."

**Method 2 — classic syntax:**

```bash
ls -l /usr/bin /nonexistent > all-output.txt 2>&1
```

The `2>&1` part means "send file descriptor 2 (stderr) to wherever file descriptor 1 (stdout) is currently pointing." Order matters here — `stdout` must be redirected first, then `stderr` follows it. Reversing them won't work as expected.

Both methods also work with `>>` for appending:

```bash
ls -l /usr/bin /nonexistent &>> all-output.txt
ls -l /usr/bin /nonexistent >> all-output.txt 2>&1
```

---

## Redirecting stdout and stderr to Different Files

Each stream can go to its own file:

```bash
command > results.txt 2> errors.txt
```

This is useful when you want to process output and errors separately — for example, reviewing errors from a build while keeping the full log intact.

---

## Discarding Unwanted Output

`/dev/null` is a special file that accepts anything written to it and discards it silently. It's often called the "bit bucket."

```bash
ls -l /usr/bin 2> /dev/null           # suppress errors, keep output
ls -l /usr/bin > /dev/null            # suppress output, keep errors
ls -l /usr/bin &> /dev/null           # suppress everything
```

---

## Redirecting Standard Input (`<`)

The `<` operator wires `stdin` to a file instead of the keyboard.

```bash
sort < unsorted.txt
```

Here `sort` reads from `unsorted.txt` rather than waiting for keyboard input. Many commands accept a filename as an argument (`sort unsorted.txt` does the same thing), but `<` is useful when a command only reads from `stdin` or when you want to be explicit about the data flow.

### Here Documents (`<<`)

A **here document** lets you feed a block of text to `stdin` inline, without a separate file. Everything between the two delimiters (commonly `EOF`, but any word works) is treated as input:

```bash
cat << EOF
Hello, $USER.
Today is $(date +%A).
EOF
```

Variables and command substitutions are expanded inside a here document. To prevent expansion, quote the delimiter:

```bash
cat << 'EOF'
This $VARIABLE is printed literally.
EOF
```

### Here Strings (`<<<`)

A **here string** passes a single string to `stdin`:

```bash
wc -w <<< "count the words in this sentence"
```

This is a shorter alternative to `echo "..." | wc -w`.

---

## Pipelines (`|`)

A pipeline connects the `stdout` of one command directly to the `stdin` of the next, without any intermediate file.

```bash
ls -l /usr/bin | less
```

The output of `ls` flows into `less` for paginated viewing. Pipelines can chain multiple commands:

```bash
cat access.log | grep "404" | sort | uniq -c | sort -rn | head -10
```

This reads a log file, filters lines containing "404", sorts them, counts duplicates, sorts by count in descending order, and shows the top 10.

### Useful Commands for Pipelines

| Command | What it does |
|---------|--------------|
| `sort` | Sort lines alphabetically or numerically (`-n`) |
| `uniq` | Remove adjacent duplicate lines (use with `sort`); `-c` adds a count |
| `grep` | Filter lines matching a pattern |
| `wc` | Count lines (`-l`), words (`-w`), or characters (`-c`) |
| `head` | Show the first N lines (`-n N`, default 10) |
| `tail` | Show the last N lines; `-f` follows a growing file in real time |
| `cut` | Extract columns or fields (`-d` for delimiter, `-f` for field number) |
| `tr` | Translate or delete characters (e.g., `tr 'a-z' 'A-Z'` to uppercase) |
| `awk` | Pattern scanning and field processing |
| `sed` | Stream editor for search-and-replace and text transformations |
| `tee` | Split output: sends it both to `stdout` and to a file |

### Piping stderr

By default, only `stdout` passes through a pipe. To include `stderr`, redirect it to `stdout` first:

```bash
command 2>&1 | less
```

In Bash 4+, the shorthand `|&` does the same thing:

```bash
command |& less
```

---

## The `tee` Command

`tee` reads from `stdin` and writes to both `stdout` and one or more files at the same time. It's particularly useful for inspecting data mid-pipeline without breaking the chain:

```bash
ls -l /usr/bin | tee ls-output.txt | grep "zip"
```

This saves the full listing to `ls-output.txt` while simultaneously passing it to `grep`. Use `-a` to append instead of overwriting:

```bash
ls -l /usr/bin | tee -a ls-output.txt | grep "zip"
```

---

## Quick Reference

```text
command > file          stdout to file (overwrite)
command >> file         stdout to file (append)
command 2> file         stderr to file (overwrite)
command 2>> file        stderr to file (append)
command &> file         stdout + stderr to file (overwrite)
command &>> file        stdout + stderr to file (append)
command > f1 2> f2      stdout and stderr to separate files
command < file          stdin from file
command << DELIM        here document (inline stdin block)
command <<< "string"    here string (single-line stdin)
command1 | command2     pipe stdout of command1 into command2
command1 |& command2    pipe stdout + stderr into command2
command > /dev/null     discard stdout
command 2> /dev/null    discard stderr
command &> /dev/null    discard everything
```
