# Chapter 10 — Processes (The Linux Command Line)

A quick-revision summary of how Linux runs, views, and controls processes.

---

## 1. The Big Idea

A **process** is a running program. Linux is multitasking: it rapidly switches the CPU between many processes so they appear to run at once.

- At boot, the kernel starts a first process — historically **`init`** (PID **1**), today usually **`systemd`** — which launches everything else.
- Every process has a unique **PID** (process ID).
- Processes form a **tree**: each one (except the first) is spawned by a **parent**, and a process can spawn **children**. Each process inherits its parent's environment.
- Processes can be **interrupted, paused, resumed, moved between foreground/background, and killed**.

---

## 2. Viewing Processes

### `ps` — a snapshot

With no options, `ps` shows only processes tied to your **current terminal**:

```bash
ps
```

```
  PID TTY          TIME CMD
 5198 pts/0    00:00:00 bash
10208 pts/0    00:00:00 ps
```

| Column | Meaning |
|--------|---------|
| `PID`  | Process ID |
| `TTY`  | Controlling terminal (`?` = none) |
| `TIME` | CPU time consumed |
| `CMD`  | Command being run |

Useful invocations:

```bash
ps x        # all YOUR processes, regardless of terminal
ps aux      # ALL processes on the system, detailed (BSD style)
```

`ps aux` adds columns like:

| Column | Meaning |
|--------|---------|
| `USER` | Owner of the process |
| `%CPU` / `%MEM` | CPU and memory usage |
| `STAT` | Process state (see below) |
| `START`| When it started |

**Process states (`STAT`):**

| Code | State |
|------|-------|
| `R` | Running or runnable |
| `S` | Sleeping (waiting for an event) |
| `D` | Uninterruptible sleep (usually I/O) |
| `T` | Stopped (paused) |
| `Z` | Zombie — finished but not cleaned up by its parent |
| `<` | High priority |
| `N` | Low priority |
| `+` | In the foreground process group |

### `top` — a live, updating view

```bash
top
```

`ps` is a still photo; **`top` is a live video.** It refreshes every few seconds, showing a summary header (uptime, load average, tasks, CPU, memory) plus a sorted list of the busiest processes. Press **`q`** to quit, **`h`** for help, **`k`** to kill a process from inside.

---

## 3. Controlling Processes

### Running a program in the background — `&`

Append `&` to launch a program without tying up your shell:

```bash
xlogo &
```

The shell prints `[1] 28401` → **job number** `[1]` and **PID** `28401`, then returns the prompt.

### `jobs` — list jobs started from this shell

```bash
jobs
```

```
[1]+  Running    xlogo &
```

### `fg` — bring a job to the foreground

```bash
fg %1       # %1 = job number 1
```

### `bg` — resume a job in the background

Used after pausing a foreground job (see Ctrl-Z below):

```bash
bg %1
```

### Keyboard control of the foreground job

| Keys | Signal | Effect |
|------|--------|--------|
| **Ctrl-C** | INT  | Interrupt / terminate the program |
| **Ctrl-Z** | TSTP | **Stop (pause)** the program and return to the shell |

After **Ctrl-Z**, the job is paused — resume it with `fg` (foreground) or `bg` (background).

---

## 4. Signals

Processes are controlled by sending them **signals** — messages asking them to do something (often, to terminate).

### `kill` — send a signal to a process

Despite the name, `kill` *sends a signal*; it doesn't necessarily "kill."

```bash
kill 28401        # send default signal (TERM) to PID 28401
kill -1 28401     # send signal 1 (HUP) by number
kill -HUP 28401   # send by name
kill -9 28401     # send KILL (force, cannot be ignored)
kill %1           # signal job number 1
```

**Common signals:**

| # | Name | Meaning |
|---|------|---------|
| 1 | `HUP`  | Hangup — often makes daemons reload config |
| 2 | `INT`  | Interrupt — same as **Ctrl-C** |
| 9 | `KILL` | Force kill — **cannot be caught or ignored**; last resort |
| 15| `TERM` | Terminate — the **default**, polite "please exit" |
| 18| `CONT` | Continue a stopped process |
| 19| `STOP` | Pause — **cannot be caught or ignored** |
| 20| `TSTP` | Terminal stop — same as **Ctrl-Z** (can be ignored) |
| 3 | `QUIT` | Quit |

> Prefer `TERM` (the default) first so the program can clean up. Use `KILL` (`-9`) only when it won't respond.

List all signals with:

```bash
kill -l
```

### `killall` — signal processes by name

Kills **every** process matching a name (and optionally a user):

```bash
killall xlogo          # signal all processes named xlogo
killall -u bob         # signal all of bob's processes
killall -SIGKILL prog  # send a specific signal
```

---

## 5. A Few More Process-Related Commands

| Command | Purpose |
|---------|---------|
| `pstree` | Show processes as a **tree**, revealing parent/child relationships |
| `vmstat` | Snapshot of system resource use (memory, swap, I/O, CPU); `vmstat 5` repeats every 5s |
| `xload`  | Graph of system load over time (X program) |
| `tload`  | Like `xload`, but draws the graph in the terminal |

---

## Quick Reference Cheat Sheet

```text
# Viewing
ps                 # processes in this terminal
ps x               # all your processes
ps aux             # every process, detailed
top                # live, updating view (q to quit)
pstree             # process family tree

# Background / foreground
command &          # start in background
jobs               # list this shell's jobs
fg %1              # bring job 1 to foreground
bg %1              # resume job 1 in background
Ctrl-C             # interrupt foreground job (INT)
Ctrl-Z             # pause foreground job (TSTP)

# Signals
kill PID           # send TERM (default, polite)
kill -9 PID        # send KILL (force, last resort)
kill -HUP PID      # send by signal name
kill %1            # signal a job by number
kill -l            # list all signals
killall name       # signal all processes by name
```

---

### Mental model to remember

> `ps`/`top` **see** processes · `&`, `jobs`, `fg`, `bg`, Ctrl-Z **juggle** them between foreground and background · `kill`/`killall` **send signals** to them.
> Reach for `TERM` (polite) before `KILL` (`-9`, forceful) — and remember `KILL` and `STOP` are the two signals a process **cannot** ignore.
