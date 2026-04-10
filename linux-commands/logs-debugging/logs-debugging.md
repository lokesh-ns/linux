# Logs & Debugging

Commands to read system logs, service logs, and historical performance data.

---

## 16. `dmesg`

```bash
dmesg                                    # kernel ring buffer output
dmesg | tail -30                         # last 30 messages
dmesg -T                                 # human-readable timestamps
dmesg -T | grep -i error                 # filter errors
dmesg -T | grep -iE "oom|kill|fail"      # OOM kills and failures
```

First place to check after a crash, kernel panic, or OOM kill event. Hardware errors, driver issues, and kernel-level events all appear here.

---

## 17. `journalctl`

```bash
journalctl -u nginx                       # logs for a specific service
journalctl -u nginx -f                    # follow live (like tail -f)
journalctl -u nginx -n 100               # last 100 lines
journalctl -u nginx --since "1 hour ago"
journalctl -u myapp | grep -i failed     # filter failures
journalctl -p err -b                      # errors from current boot
journalctl -b -1 -k                       # kernel messages from previous boot
```

Master log for every systemd service. Use this when you need app-level log detail that `dmesg` doesn't show.

---

## 18. `sar`

```bash
sar -u 1 3          # CPU utilization
sar -r 1 3          # memory utilization
sar -d 1 3          # disk I/O
sar -n DEV 1 3      # network per interface
sar -f /var/log/sa/sa15   # historical data from the 15th of the month
```

**Use:** "What was CPU/memory/network at 3 AM last night?" — `sar` keeps historical data via the `sysstat` cron job. Essential for post-incident analysis.

**Format:** `sar -u 1 3` → report every 1 second, 3 times.

---

## 19. `systemctl`

```bash
systemctl status nginx
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx        # reload config — no downtime (sends SIGHUP)
systemctl enable nginx        # start on boot
systemctl disable nginx
systemctl --failed            # list all failed services
```

**`reload` vs `restart`:**

| Command | Behaviour | Downtime |
|---------|-----------|----------|
| `reload` | Sends SIGHUP, app re-reads config | No |
| `restart` | Full stop + start | Brief |

Use `reload` for nginx config changes. Use `restart` when the binary itself changes.
