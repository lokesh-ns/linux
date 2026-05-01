# Process Management

Commands to find, monitor, prioritise, and kill processes.

***

## `ps`

```bash
ps -ef                            # all processes, full format
ps aux                            # BSD style, all users
ps aux --sort=-%cpu | head -15    # top CPU consumers
ps aux --sort=-%mem | head -15    # top memory consumers
ps -ef | grep nginx               # find a specific process
```

**Key columns:**

| Column          | Meaning             |
| --------------- | ------------------- |
| `PID`           | Process ID          |
| `PPID`          | Parent Process ID   |
| `%CPU` / `%MEM` | Resource usage      |
| `COMMAND`       | Full command string |

***

## `kill`

```bash
kill -15 <PID>      # SIGTERM — graceful shutdown, try first
kill -9 <PID>       # SIGKILL — force kill, no cleanup
kill -1 <PID>       # SIGHUP — reload config

# Find PID first
ps -ef | grep nginx
pgrep nginx
```

**Signal reference:**

| Signal  | Number | Behaviour                             |
| ------- | ------ | ------------------------------------- |
| SIGTERM | 15     | Graceful — app can catch and clean up |
| SIGKILL | 9      | Force — cannot be caught or ignored   |
| SIGHUP  | 1      | Reload config without restart         |
| SIGINT  | 2      | Ctrl+C                                |

> ⚠️ Always try SIGTERM (15) before SIGKILL (9). SIGKILL gives the process no chance to clean up open files or connections.

***

## `pkill`

```bash
pkill nginx           # kill by process name
pkill -9 java         # force kill by name
pkill -u www-data     # kill all processes of a user
```

Use when you know the name but not the PID.

***

## `nice` / `renice`

```bash
nice -n 10 ./script.sh          # start with lower priority (10)
nice -n -20 ./critical.sh       # start with highest priority (-20)
renice -n 5 -p <PID>            # change priority of a running process
```

**Priority scale:** -20 (highest) to +19 (lowest). Default is 0.

**Use:** when a server is resource-starved, ensure critical processes (DB, API gateway) get CPU time first.

***

## `strace`

```bash
strace -p <PID>                            # attach to running process
strace -p <PID> -e trace=open,read,write   # filter specific syscalls
strace -p <PID> -T                         # time spent per syscall
strace -p <PID> -c                         # syscall summary (adds overhead)
```

**Use:** when a process misbehaves but logs show nothing. See exactly what system calls it's making — file opens, network reads, locks. The `-T` flag is especially useful to find which syscall is slow.

> ⚠️ `strace` adds overhead to the traced process. Use `-c` for summary, not full trace, on production.
