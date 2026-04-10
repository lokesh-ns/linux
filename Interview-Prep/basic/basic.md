# Basic

## 📋 Index

| Level           | Questions | Topics                                                                     |
| --------------- | --------- | -------------------------------------------------------------------------- |
| Basic           | Q1–Q15    | File system, permissions, processes, SSH, cron, text processing            |
| Mid-level       | Q16–Q35   | Performance, networking, scripting, kernel, security, storage              |
| Hard / Scenario | Q36–Q50   | Troubleshooting scenarios, forensics, kernel internals, advanced scripting |

***

## 🟢 BASIC (Q1–Q15)

***

### Q1. What is the Linux directory structure? Explain key directories.

```
/        → root of everything
/etc     → system config files (sshd_config, fstab, cron)
/var     → variable data: logs (/var/log), spool, temp runtime
/proc    → virtual FS, kernel/process info (not on disk)
/sys     → virtual FS for hardware/kernel parameters
/tmp     → temporary files, cleared on reboot
/home    → user home directories
/usr     → user programs, libraries
/bin, /sbin → essential binaries (ls, cp, systemctl)
/opt     → optional/third-party software
/mnt, /media → mount points
```

**DevOps relevance:** app configs in `/etc`, logs in `/var/log`, container runtimes use `/proc` for cgroup/namespace info.

***

### Q2. Explain Linux file permissions — what does `-rwxr-xr--` mean?

Format: `[type][owner][group][others]`

```
-    = regular file (d=dir, l=symlink)
rwx  = owner: read(4) + write(2) + execute(1) = 7
r-x  = group: read + execute = 5
r--  = others: read only = 4
```

**Numeric:** `chmod 754 file`

**Special bits:**

* `setuid (4xxx)` — run as file owner (e.g. passwd command)
* `setgid (2xxx)` — run as group / new files inherit group
* `sticky bit (1xxx)` — only owner can delete (e.g. /tmp is 1777)

```bash
chmod 755 script.sh
chown user:group file
umask 022    # default permissions = 755 for dirs, 644 for files
```

***

### Q3. What is the difference between a process and a thread?

**Process:** independent program with its own memory space, PID, file descriptors. Isolated — if it dies, others are unaffected.

**Thread:** lightweight unit within a process. Shares memory/resources with sibling threads. Faster context switch but a crash in one thread can affect the whole process.

```bash
ps -eLf                                  # view threads
cat /proc/<PID>/status | grep Threads    # thread count for a process
```

**Real relevance:** Java app servers (Tomcat) are multi-threaded in one process. Node.js is single-threaded. Docker runs each container as a separate process (namespace-isolated).

***

### Q4. How do you find and kill a process using a specific port?

```bash
# Find process on port 8080
lsof -i :8080
ss -tulnp | grep 8080          # modern, preferred
netstat -tulnp | grep 8080     # older systems

# Kill it
kill -15 <PID>    # SIGTERM — graceful, try first
kill -9 <PID>     # SIGKILL — force, no cleanup

# One-liner
lsof -ti :8080 | xargs kill -9
```

**Signals:**

* `SIGTERM (15)` — asks process to terminate, can be caught
* `SIGKILL (9)` — kernel kills it, cannot be caught or ignored
* `SIGHUP (1)` — reload config (nginx -s reload uses this)
* `SIGINT (2)` — Ctrl+C

***

### Q5. What is the difference between soft links and hard links?

**Hard link:** another name pointing to the same inode. Deleting original doesn't affect it. Same filesystem only. Cannot link directories.

**Soft link (symlink):** pointer to a path. Breaks if original is deleted. Can span filesystems. Can link directories.

```bash
ln /etc/hosts /tmp/hosts_hard               # hard link — same inode
ln -s /etc/nginx/nginx.conf /tmp/nginx.conf # symlink

ls -li    # shows inode numbers — hard links share the same inode
```

**Real use:** `/usr/bin/python3 → python3.11` (symlink for version management). nginx `sites-enabled` contains symlinks to `sites-available`.

***

### Q6. How do you search for a pattern in files? Explain grep options you use daily.

```bash
grep "error" /var/log/app.log
grep -i "error" file           # case insensitive
grep -r "TODO" /src/           # recursive
grep -n "FAIL" file            # show line numbers
grep -v "DEBUG" file           # invert — exclude DEBUG lines
grep -E "error|fail|warn" file # extended regex (OR)
grep -A 3 -B 2 "exception" file # 3 lines after, 2 before context
grep -c "error" file           # count matches only

# Combine with pipes
cat /var/log/app.log | grep "ERROR" | awk '{print $1,$2}' | sort | uniq -c
```

***

### Q7. Explain awk and sed — when do you use each?

**awk** — field-based text processing (column processor):

```bash
awk '{print $1, $3}' file           # print columns 1 and 3
awk -F: '{print $1}' /etc/passwd    # use : as delimiter
awk '$3 > 80 {print $1}' file       # conditional filter
awk '{sum+=$2} END {print sum}' f   # aggregation
```

**sed** — stream editor (find/replace, delete lines):

```bash
sed 's/foo/bar/g' file              # replace all foo with bar
sed -i 's/old/new/g' file          # in-place edit
sed '/^#/d' file                   # delete comment lines
sed -n '10,20p' file               # print lines 10–20
sed 's/password=.*/password=REDACTED/g' config.env  # mask secrets
```

**Real use:** `sed` to patch config files in CI pipelines without a full template engine.

***

### Q8. How do you check disk usage and identify what's filling up a filesystem?

```bash
df -h                              # disk usage per filesystem
du -sh /var/log/*                  # size per directory/file
du -sh * | sort -rh | head -20     # top 20 largest items

# Find large files
find / -type f -size +100M -exec ls -lh {} \;

# Find deleted files still held open (common disk leak!)
lsof +D /path
```

**Real scenario:** disk full on `/var/log` because a log file grew unchecked. `lsof` shows a deleted file still consuming space because a process has it open — restart the process or truncate: `> /var/log/app.log`

***

### Q9. What does netstat/ss show and how do you use it in troubleshooting?

`ss` is the modern replacement for `netstat`.

```bash
ss -tulnp          # TCP/UDP listening ports with process names
ss -s              # socket statistics summary
ss -anp            # all connections with PIDs
ss -tnp state ESTABLISHED  # active TCP connections only

netstat -tulnp     # same on older systems
ip route show      # routing table
ip addr show       # interface IPs (replaces ifconfig)
```

**Common checks:**

* Is my app listening? → `ss -tulnp | grep 8080`
* Who's connected? → `ss -tnp state ESTABLISHED`
* Connection leak? → `ss -s` (check TIME\_WAIT count)

***

### Q10. How do you manage services with systemd? Common systemctl commands.

```bash
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx       # reload config without full restart
systemctl enable nginx       # start on boot
systemctl disable nginx
systemctl status nginx       # status + last log lines
systemctl is-active nginx    # returns active/inactive

# View logs for a service
journalctl -u nginx           # all logs
journalctl -u nginx -f        # follow (tail -f equivalent)
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx -n 50    # last 50 lines

# Check boot failures
systemctl --failed
journalctl -p err -b          # errors since last boot
```

***

### Q11. How do you manage users and groups in Linux?

```bash
useradd -m -s /bin/bash username      # create user with home dir
passwd username                        # set password
usermod -aG docker username            # add to group
userdel -r username                    # delete user + home dir
id username                            # show UID, GID, groups

# Key files
/etc/passwd   # user accounts
/etc/shadow   # hashed passwords
/etc/group    # group definitions

# sudo access via visudo
username ALL=(ALL:ALL) NOPASSWD: /usr/bin/systemctl
```

**Real use:** service accounts for CI/CD agents — no shell (`useradd -s /sbin/nologin`), no interactive login, minimal group membership.

***

### Q12. How does SSH key-based authentication work? How do you set it up?

**Flow:**

1. Client has private key; server has public key in `~/.ssh/authorized_keys`
2. Server sends a challenge encrypted with the public key
3. Only the holder of the private key can decrypt and respond
4. No password is ever transmitted

```bash
ssh-keygen -t ed25519 -C "user@host"   # generate key pair
ssh-copy-id user@server                 # copy public key to server
chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys
```

**/etc/ssh/sshd\_config hardening:**

```
PermitRootLogin no
PasswordAuthentication no
AllowUsers deployuser
Port 2222
MaxAuthTries 3
```

```bash
# SSH tunneling (port forward through bastion)
ssh -L 8080:localhost:3306 user@jumphost
```

***

### Q13. How does cron work? Write a cron expression for "every day at 2:30 AM".

**Format:** `minute hour day-of-month month day-of-week command`

```
30 2 * * * /path/to/script.sh       # every day at 2:30 AM

*/5 * * * *          # every 5 minutes
0 9 * * 1            # every Monday 9 AM
0 0 1 * *            # first day of month midnight
0 2 * * 0,6          # Sat/Sun 2 AM
```

```bash
crontab -e           # edit current user's cron
crontab -l           # list current user's cron
/etc/cron.d/         # system-wide cron jobs
grep CRON /var/log/syslog   # check execution log
```

**Common trap:** cron runs with a minimal PATH — always use full binary paths in cron scripts.

***

### Q14. How do you find files in Linux? Common find command patterns.

```bash
find /path -name "*.log"              # by name
find /var/log -mtime -7               # modified in last 7 days
find / -user root -perm -4000         # setuid files (security audit)
find /tmp -type f -empty              # empty files
find /app -name "*.conf" -exec grep -l "password" {} \;  # search inside

# Delete large old logs
find /var/log -name "*.log" -size +50M -delete

# Combine with xargs (efficient for many files)
find /src -name "*.py" | xargs grep "TODO"

# -type f=file, d=dir, l=symlink
# -mtime=days, -mmin=minutes
# -newer file = modified more recently than 'file'
```

***

### Q15. How do you compress and archive files in Linux?

```bash
# tar
tar -czvf archive.tar.gz /path/       # create gzip-compressed
tar -xzvf archive.tar.gz             # extract
tar -tzvf archive.tar.gz             # list contents without extracting
tar -czvf backup.tar.gz /etc --exclude=/etc/ssl  # with exclude

# flags: c=create x=extract z=gzip j=bzip2 J=xz v=verbose f=file

# zip / unzip
zip -r archive.zip /path/
unzip archive.zip -d /destination/

# gzip (single files)
gzip file.log         # → file.log.gz
gunzip file.log.gz

# Timestamped backup
tar -czf backup-$(date +%Y%m%d).tar.gz /app/data
```
