# Scheduler & Misc

Cron jobs, library inspection, background processes, and the `man` command.

---

## 44. `crontab`

```bash
crontab -e          # edit cron jobs (opens $EDITOR)
crontab -l          # list current user's cron jobs
crontab -u root -l  # list another user's cron jobs
```

**Cron syntax:**

```
# MIN  HOUR  DOM  MON  DOW  COMMAND
  *    *     *    *    *    /path/to/script.sh
```

| Field | Range | Meaning |
|-------|-------|---------|
| MIN | 0–59 | Minute |
| HOUR | 0–23 | Hour |
| DOM | 1–31 | Day of month |
| MON | 1–12 | Month |
| DOW | 0–7 | Day of week (0 and 7 = Sunday) |

**Common examples:**

```bash
0    2  *  *  *   /scripts/backup.sh        # daily at 2:00 AM
*/5  *  *  *  *   /scripts/healthcheck.sh   # every 5 minutes
0    9  *  *  1   /scripts/report.sh        # every Monday at 9 AM
0    0  1  *  *   /scripts/monthly.sh       # first day of every month
```

> ⚠️ Cron uses a minimal PATH. Always use **absolute paths** in cron scripts (`/usr/bin/python3`, not `python3`).

**Debug a failing cron job:**

```bash
grep CRON /var/log/syslog        # Debian/Ubuntu
grep CRON /var/log/cron          # RHEL/CentOS
```

---

## 45. `ldd`

```bash
ldd /usr/bin/nginx       # list shared library dependencies
ldd /usr/bin/python3
```

Use when a binary fails with _"shared library not found"_ — `ldd` shows which dependency is missing.

---

## 46. `man`

```bash
man free        # full manual for the free command
man grep        # all options for grep
man tar
# Inside man: arrow keys to scroll, Q to quit
```

You don't need to memorise every flag. Use `man` at any time in the terminal.

---

## 47. Background processes

```bash
command &              # run in background, get PID
jobs                   # list background jobs
fg %1                  # bring job 1 to foreground
nohup command &        # run in background, survive logout
```

---

## 48–50. Destructive Commands — Handle with Extreme Caution

| Command | Risk | Safe Practice |
|---------|------|---------------|
| `rm -rf /path` | Permanent deletion — no recycle bin | Move to backup dir first |
| `kill -9 1` | Kills init/systemd — crashes the system | Never target PID 1 |
| `chmod 000 /bin/bash` | Breaks all shell access | Always verify the path |
| `> /dev/sda` | Wipes the entire disk | Never on production |
| `dd if=/dev/zero of=/dev/sda` | Same as above | Snapshot first |
