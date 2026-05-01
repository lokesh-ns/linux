# System Health

Commands to check the overall health of a Linux server — the first things you run when you SSH into a slow or broken server.

***

## `uptime`

```bash
uptime
# 10:32:01 up 10 days, 2:15,  1 user,  load average: 0.52, 0.45, 0.40
```

**Output fields:**

| Field              | Meaning                                 |
| ------------------ | --------------------------------------- |
| `up 10 days`       | Server hasn't rebooted in 10 days       |
| `1 user`           | Number of logged-in sessions            |
| `0.52, 0.45, 0.40` | Load average: last 1 min, 5 min, 15 min |

**Load average rule:** if load > number of CPU cores → server is struggling.

```bash
nproc                             # check CPU core count
```

**Interview tip:** uptime also tells you if security patches have been applied — a server up for 1 year likely has unpatched vulnerabilities.

***

## `free`

```bash
free -h       # human readable (GiB)
free -m       # in MB
man free      # all options
```

**Output columns:**

| Column       | Meaning                              |
| ------------ | ------------------------------------ |
| `total`      | Total installed RAM                  |
| `used`       | Currently in use                     |
| `free`       | Completely unused                    |
| `shared`     | Used by tmpfs                        |
| `buff/cache` | Kernel cache — reclaimable           |
| `available`  | Actually available for new processes |

> Focus on `available`, not `free`. Swap being non-zero = memory pressure.

**Interview tip:** "If `vmstat` shows `si`/`so` > 0 consistently, the server is actively swapping — severe performance impact."

***

## `top`

```bash
top -c          # show full command path
```

**Shortcuts inside `top`:**

| Key | Action         |
| --- | -------------- |
| `P` | Sort by CPU    |
| `M` | Sort by memory |
| `Q` | Quit           |

**CPU line breakdown:**

| Field | Meaning                           |
| ----- | --------------------------------- |
| `us`  | User-space CPU                    |
| `sy`  | Kernel CPU                        |
| `wa`  | I/O wait — high = disk bottleneck |
| `id`  | Idle                              |

**Process table key columns:** `PID`, `USER`, `%CPU`, `%MEM`, `COMMAND`

***

## `htop`

```bash
htop
```

Same as `top` but color-coded and mouse-clickable. More readable for interactive troubleshooting.

**Shortcuts:**

| Key       | Action                               |
| --------- | ------------------------------------ |
| `Shift+P` | Sort by CPU                          |
| `Shift+M` | Sort by memory                       |
| `F5`      | Tree view (parent → child processes) |
| `Q`       | Quit                                 |

***

## `vmstat`

```bash
vmstat 1 5      # report every 1 second, 5 times
```

**Key columns:**

| Column      | Meaning                                         |
| ----------- | ----------------------------------------------- |
| `r`         | Runnable processes (CPU queue)                  |
| `b`         | Blocked — waiting for I/O                       |
| `si` / `so` | Swap in / swap out — non-zero = memory pressure |
| `wa`        | CPU % waiting on I/O                            |
| `us` / `sy` | User / kernel CPU %                             |

**Use:** single-shot view of CPU + memory + swap + I/O in one command. Great starting point when a server is slow.
