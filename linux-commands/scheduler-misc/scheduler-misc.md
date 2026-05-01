# Scheduler

Cron jobs, library inspection, background processes, and the `man` command.

***

## `crontab`

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

| Field | Range | Meaning                        |
| ----- | ----- | ------------------------------ |
| MIN   | 0–59  | Minute                         |
| HOUR  | 0–23  | Hour                           |
| DOM   | 1–31  | Day of month                   |
| MON   | 1–12  | Month                          |
| DOW   | 0–7   | Day of week (0 and 7 = Sunday) |

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
