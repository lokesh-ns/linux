# Linux Interview Prep — 50 Real Interview Questions
### DevOps Engineer | Basic → Mid-level → Hard/Scenario

---

## 📋 Index

| Level | Questions | Topics |
|-------|-----------|--------|
| Basic | Q1–Q15 | File system, permissions, processes, SSH, cron, text processing |
| Mid-level | Q16–Q35 | Performance, networking, scripting, kernel, security, storage |
| Hard / Scenario | Q36–Q50 | Troubleshooting scenarios, forensics, kernel internals, advanced scripting |

---

## 🟢 BASIC (Q1–Q15)

---

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

---

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
- `setuid (4xxx)` — run as file owner (e.g. passwd command)
- `setgid (2xxx)` — run as group / new files inherit group
- `sticky bit (1xxx)` — only owner can delete (e.g. /tmp is 1777)

```bash
chmod 755 script.sh
chown user:group file
umask 022    # default permissions = 755 for dirs, 644 for files
```

---

### Q3. What is the difference between a process and a thread?

**Process:** independent program with its own memory space, PID, file descriptors. Isolated — if it dies, others are unaffected.

**Thread:** lightweight unit within a process. Shares memory/resources with sibling threads. Faster context switch but a crash in one thread can affect the whole process.

```bash
ps -eLf                                  # view threads
cat /proc/<PID>/status | grep Threads    # thread count for a process
```

**Real relevance:** Java app servers (Tomcat) are multi-threaded in one process. Node.js is single-threaded. Docker runs each container as a separate process (namespace-isolated).

---

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
- `SIGTERM (15)` — asks process to terminate, can be caught
- `SIGKILL (9)` — kernel kills it, cannot be caught or ignored
- `SIGHUP (1)` — reload config (nginx -s reload uses this)
- `SIGINT (2)` — Ctrl+C

---

### Q5. What is the difference between soft links and hard links?

**Hard link:** another name pointing to the same inode. Deleting original doesn't affect it. Same filesystem only. Cannot link directories.

**Soft link (symlink):** pointer to a path. Breaks if original is deleted. Can span filesystems. Can link directories.

```bash
ln /etc/hosts /tmp/hosts_hard               # hard link — same inode
ln -s /etc/nginx/nginx.conf /tmp/nginx.conf # symlink

ls -li    # shows inode numbers — hard links share the same inode
```

**Real use:** `/usr/bin/python3 → python3.11` (symlink for version management). nginx `sites-enabled` contains symlinks to `sites-available`.

---

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

---

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

---

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

---

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
- Is my app listening? → `ss -tulnp | grep 8080`
- Who's connected? → `ss -tnp state ESTABLISHED`
- Connection leak? → `ss -s` (check TIME_WAIT count)

---

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

---

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

---

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

**/etc/ssh/sshd_config hardening:**

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

---

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

---

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

---

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

---

## 🔵 MID-LEVEL (Q16–Q35)

---

### Q16. How do you troubleshoot high CPU usage on a Linux server? Walk me through step by step.

```bash
# Step 1: Get a snapshot
top -c              # -c shows full command path
htop                # tree mode with F5

# Step 2: Narrow down
ps aux --sort=-%cpu | head -15
pidstat -u 1 5     # per-PID CPU over 5 seconds

# Step 3: What is the process doing?
strace -p <PID> -c              # syscall summary (adds overhead)
perf top                         # kernel-level hot functions

# Step 4: JVM apps specifically
jstack <PID>                    # thread dump → find hot threads
jstat -gcutil <PID> 1000       # GC pressure?
```

**CPU breakdown in top:**
- `%us` = user-space CPU
- `%sy` = kernel/system CPU — high means kernel issue
- `%wa` = I/O wait — high means disk bottleneck, NOT CPU

**Common causes:** runaway loop, GC pressure, fork bomb, CPU-intensive cron job.

---

### Q17. A server is out of memory but you don't want to reboot. What do you do?

```bash
# Step 1: Assess
free -h
vmstat 1 5              # memory + swap trend (check si/so columns)

# Step 2: Find the consumer
ps aux --sort=-%mem | head -15

# Step 3: Investigate swap
swapon -s
# vmstat si/so > 0 = active swapping = severe degradation

# Step 4: Free up without rebooting
sync && echo 3 > /proc/sys/vm/drop_caches    # drop page cache (safe)
kill -15 <PID>                                # kill leaking process

# Step 5: Check OOM killer history
dmesg | grep -i "out of memory"
grep -i "oom" /var/log/syslog
```

---

### Q18. Explain load average in Linux. What does "2.5 4.0 3.8" mean?

Load average = average number of runnable + uninterruptible (I/O waiting) processes over **1, 5, 15 minutes**.

**Interpretation depends on CPU count:**

| CPU Count | Load 2.5 | Meaning |
|-----------|----------|---------|
| 1 core | 2.5 | Overloaded — queue building |
| 4 cores | 2.5 | 62.5% — fine |
| 4 cores | 5.0 | Queue building up |

```bash
nproc                                    # check CPU count
grep -c ^processor /proc/cpuinfo
```

**Rule:** `load average / CPU count > 1.0` means processes are waiting.

**Pattern `2.5 → 4.0 → 3.8`:** spike happened ~5 minutes ago, system is now recovering.

**High load but low CPU%?** → likely I/O wait. Check `vmstat → wa column` or `iostat -x 1 5`.

---

### Q19. How do you troubleshoot network connectivity between two servers?

```bash
# L3 — Network
ping <target IP>
traceroute <target>
mtr <target>                  # live traceroute with packet loss %

# L4 — Transport (is the port open?)
nc -zv <host> <port>          # netcat — cleaner than telnet
curl -v http://host:port

# DNS
dig hostname
dig +trace hostname           # full resolver chain

# Firewall
iptables -L -n -v
firewall-cmd --list-all       # RHEL/CentOS firewalld

# Routing
ip route show
ip route get <target-IP>      # which route would be used?
```

**Diagnosis tips:**
- `ping` works but `nc` fails → firewall blocking the port
- `nc` works but HTTPS fails → TLS/cert issue or app not listening on TLS
- `dig` fails → DNS resolution problem, check `/etc/resolv.conf`

---

### Q20. Write a script to monitor a service and restart it if it goes down.

```bash
#!/bin/bash
SERVICE="nginx"
LOG="/var/log/service_monitor.log"
ALERT="ops@company.com"

timestamp() { date '+%Y-%m-%d %H:%M:%S'; }

if ! systemctl is-active --quiet "$SERVICE"; then
    echo "$(timestamp): $SERVICE is DOWN — attempting restart" >> "$LOG"
    systemctl restart "$SERVICE"
    sleep 5
    if systemctl is-active --quiet "$SERVICE"; then
        echo "$(timestamp): $SERVICE restarted successfully" >> "$LOG"
        echo "$SERVICE restarted at $(timestamp)" | mail -s "Service Recovery: $SERVICE" "$ALERT"
    else
        echo "$(timestamp): $SERVICE restart FAILED" >> "$LOG"
        echo "$SERVICE failed to restart" | mail -s "CRITICAL: $SERVICE down" "$ALERT"
    fi
fi

# Add to cron: */2 * * * * /usr/local/bin/monitor_service.sh
```

**Interview note:** "In production I'd use systemd's `Restart=on-failure` directive or Monit/supervisord — avoids cron race conditions and has built-in backoff."

---

### Q21. Explain inodes. What happens when a filesystem runs out of inodes even though there's disk space?

**Inode:** data structure storing file metadata — owner, permissions, timestamps, block pointers. NOT the filename. Each file/directory consumes exactly one inode. Inode count is fixed at filesystem creation.

**Symptom:** `df -h` shows space available but creating a file fails: *"No space left on device"*

```bash
# Diagnose
df -i                     # shows inode usage %
df -i / | tail -1

# Find inode hogs (dirs with millions of small files)
find / -xdev -printf '%h\n' | sort | uniq -c | sort -rn | head

# Fix — delete small temp/cache files
find /tmp -type f -delete
find /var/cache -name "*.cache" -delete
```

**Long-term fix:** recreate filesystem with more inodes: `mkfs.ext4 -N <inode_count> /dev/sdX`

**Real scenario:** mail servers, container builds, and session stores create millions of tiny files.

---

### Q22. How do you harden an SSH server in production?

```
# /etc/ssh/sshd_config

PermitRootLogin no
PasswordAuthentication no
ChallengeResponseAuthentication no
PubkeyAuthentication yes

AllowUsers deployuser jenkins
AllowGroups sshusers

ClientAliveInterval 300
ClientAliveCountMax 2

MaxAuthTries 3
LoginGraceTime 30

X11Forwarding no
AllowTcpForwarding no

# Strong ciphers only
KexAlgorithms curve25519-sha256
Ciphers aes256-gcm@openssh.com,chacha20-poly1305@openssh.com
MACs hmac-sha2-256-etm@openssh.com

Port 2222
```

```bash
systemctl restart sshd
systemctl enable fail2ban    # ban IPs with too many failed attempts
```

---

### Q23. How do you parse and process log files efficiently using bash?

```bash
# Count error types
grep "ERROR" app.log | awk '{print $4}' | sort | uniq -c | sort -rn | head -20

# Find IPs with > 100 requests
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | awk '$1 > 100 {print $2, $1}'

# Real-time monitoring
tail -f /var/log/app.log | grep --line-buffered "ERROR" | while read line; do
    echo "$(date): $line" >> /tmp/live_errors.log
done

# Process compressed rotated logs (never decompress to disk)
zcat /var/log/app.log.gz | grep "ERROR"
zgrep "ERROR" /var/log/app.log.gz
```

**Performance tip:** for huge files, `awk` in a single pass is much faster than chained grep pipes.

---

### Q24. What are cgroups and namespaces? How do containers use them?

**Namespaces — isolation:**

| Namespace | What it isolates |
|-----------|-----------------|
| PID | Process IDs — container has its own PID 1 |
| NET | Network stack (interfaces, routes, iptables) |
| MNT | Filesystem mount points |
| UTS | Hostname |
| IPC | Inter-process communication |
| USER | UID/GID mapping (rootless containers) |

**cgroups — resource limits:**
- CPU: limit % of CPU time
- memory: hard limit, triggers OOM kill if exceeded
- blkio: disk I/O throttling
- pids: limit number of processes (fork bomb protection)

**Together:** namespaces provide isolation, cgroups provide resource constraints. Containers share the host kernel but are isolated via these primitives.

**Implication:** container escape = namespace bypass. Running as non-root matters. OpenShift SCCs enforce this at the pod level.

---

### Q25. Explain the difference between vmstat, iostat, and sar. When do you use each?

```bash
vmstat 1 5      # system-wide: CPU, memory, swap, I/O, process queue
                # r=runnable, b=blocked (I/O wait), wa=I/O wait %
                # si/so = swap in/out (>0 = memory pressure)

iostat -x 1 5   # per-disk I/O stats
                # %util = disk saturation (100% = bottleneck)
                # await = avg ms per I/O (high = slow disk)
                # r/s, w/s = reads/writes per second

sar -u 1 5      # CPU history
sar -r 1 5      # memory history
sar -d 1 5      # disk history (needs sysstat)
sar -n DEV 1 5  # network per interface
sar -f /var/log/sa/sa15   # historical data from 15th of month
```

**Decision tree:**
- High load + high `wa%` → `iostat` (disk bottleneck)
- High load + low `wa%` → `vmstat` → CPU or memory issue
- Need historical trend → `sar`

---

### Q26. What is LVM and how would you extend a logical volume online?

**LVM layers:**
```
PV (Physical Volume) → real disks/partitions
VG (Volume Group)    → pool of PVs
LV (Logical Volume)  → virtual partition from VG
```

**Extend online (no downtime):**

```bash
# 1. Check free space in VG
vgdisplay vgname

# 2. Extend LV by 10G
lvextend -L +10G /dev/vgname/lvname

# 3a. Resize ext4 filesystem
resize2fs /dev/vgname/lvname

# 3b. Resize xfs filesystem (RHEL default)
xfs_growfs /mount/point

# One-shot: extend LV AND resize filesystem
lvextend -r -L +10G /dev/vgname/lvname   # -r = auto resize-fs

# Verify
df -h
```

**Snapshots:**

```bash
lvcreate -L 5G -s -n snap /dev/vg/lv   # point-in-time backup without app downtime
```

---

### Q27. Explain iptables — how do you allow a port, block an IP, and view current rules?

```bash
# View rules with line numbers
iptables -L INPUT -n -v --line-numbers
iptables -L -n -v                      # all chains

# Allow port 8080
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT

# Allow from specific IP only
iptables -A INPUT -s 10.0.1.5 -p tcp --dport 5432 -j ACCEPT

# Block an IP
iptables -A INPUT -s 192.168.1.100 -j DROP

# Insert at top (before other rules)
iptables -I INPUT 1 -p tcp --dport 443 -j ACCEPT

# Allow established connections (stateful — critical!)
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Delete rule by line number
iptables -D INPUT 3

# Save rules
iptables-save > /etc/iptables/rules.v4
```

**Note:** modern systems use `nftables`. `firewalld` (RHEL) and `ufw` (Ubuntu) are higher-level wrappers.

---

### Q28. How do you handle errors in bash scripts? What is set -e, set -u, set -o pipefail?

```bash
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'      # safer word splitting

# set -e    → exit immediately if any command returns non-zero
# set -u    → treat unset variables as errors
# set -o pipefail → pipeline fails if ANY command in it fails
#   without pipefail: grep "x" file | wc -l → exits 0 even if grep fails!
```

**Trap for cleanup:**

```bash
cleanup() { rm -f /tmp/lockfile; echo "Cleaned up"; }
trap cleanup EXIT    # runs on exit (normal or error)
trap 'echo "Error on line $LINENO"' ERR
```

**Error handling pattern:**

```bash
die() { echo "Error: $1" >&2; exit 1; }

command || die "command failed"

if ! command; then
    echo "command failed" >&2
    exit 1
fi
```

---

### Q29. Explain SELinux/AppArmor — what are they and how do you troubleshoot a policy denial?

**SELinux (RHEL/CentOS):** Mandatory Access Control. Labels every process and file with a security context. Policy defines what label X can do to label Y.

**Modes:**
- `enforcing` — policy enforced, violations blocked + logged
- `permissive` — violations logged but NOT blocked (use for testing)
- `disabled` — completely off

```bash
getenforce                              # check current mode
setenforce 0                            # switch to permissive (temporary)

# Troubleshoot a denial
ausearch -m AVC -ts recent
grep "avc: denied" /var/log/audit/audit.log

# Generate and apply a fix
ausearch -m AVC -ts recent | audit2allow -M mypolicy
semodule -i mypolicy.pp

# Fix file label
chcon -t httpd_content_t /var/www/html/file
restorecon -Rv /var/www/html
```

**AppArmor (Ubuntu):** path-based profiles. Simpler but less granular.

```bash
aa-status                       # active profiles
aa-complain /path/to/binary    # permissive mode for one app
```

---

### Q30. Walk me through the Linux boot process from power-on to login prompt.

1. **BIOS/UEFI** — hardware POST, finds bootable device
2. **Bootloader (GRUB2)** — loads kernel + initrd from `/boot`
3. **Kernel** — decompresses, initialises hardware, mounts initramfs (temp root FS)
4. **initramfs** — contains drivers needed to mount real root FS
5. **Kernel mounts real `/`**
6. **PID 1 starts** — systemd (modern) or init (legacy SysV)
7. **systemd** — reads targets, starts `default.target` (usually `multi-user.target`)
8. **Services start** in dependency order
9. **Login prompt** (getty on TTY or SSH daemon ready)

**Troubleshooting boot failures:**

```bash
journalctl -b -p err        # errors from current boot
journalctl -b -1 -k         # kernel messages from previous boot
# At GRUB: add "single" or "init=/bin/bash" to kernel args
```

**DevOps relevance:** EC2/cloud instances run `cloud-init` at boot for userdata. Understanding boot sequence helps debug cloud-init failures in AMIs/launch templates.

---

### Q31. How do you find and fix a memory leak in a running Linux process?

```bash
# Detect growth over time
watch -n 5 'ps -o pid,rss,vsz,comm -p <PID>'

# Detailed memory map
pmap -x <PID>
cat /proc/<PID>/smaps | grep -i pss | awk '{sum += $2} END {print sum/1024 " MB"}'

# JVM specific
jmap -heap <PID>                  # heap usage
jmap -histo <PID>                 # object histogram (leak candidate)
jstat -gcutil <PID> 1000 10      # frequent full GC = leak
```

**Mitigation without fixing the leak:**

```bash
# systemd — restart on OOM
MemoryMax=2G   # in unit file

# K8s — restart on OOM kill
resources:
  limits:
    memory: "2Gi"
```

**Root cause:** add heap dump on OOM for JVM: `-XX:+HeapDumpOnOutOfMemoryError`

---

### Q32. What is a VLAN, and how does Linux handle network bonding/teaming?

**VLAN:** logical network segmentation at L2 (802.1Q tagging). One physical interface can carry multiple VLANs.

```bash
# Create VLAN interface
ip link add link eth0 name eth0.100 type vlan id 100
ip addr add 192.168.100.1/24 dev eth0.100
ip link set eth0.100 up
```

**Bonding modes:**

| Mode | Name | Use case |
|------|------|----------|
| 0 | round-robin | load balance |
| 1 | active-backup | HA failover |
| 4 | 802.3ad LACP | active/active throughput + HA |

```bash
ip link add bond0 type bond
ip link set bond0 type bond mode active-backup
ip link set eth0 master bond0
ip link set eth1 master bond0
```

---

### Q33. How do you check and repair filesystem errors in Linux?

**Warning:** `fsck` must run on an unmounted filesystem.

```bash
# Check without repair
fsck -n /dev/sdb1

# Repair
umount /dev/sdb1
fsck -y /dev/sdb1          # auto-yes to all fixes

# XFS (modern RHEL default)
xfs_repair -n /dev/sdb1   # dry run
xfs_repair /dev/sdb1       # repair

# Schedule check at next boot (for root FS)
tune2fs -c 1 /dev/sda1    # force check after 1 mount

# Check current FS status
tune2fs -l /dev/sda1 | grep "Last checked"

# Disk health
smartctl -a /dev/sda
smartctl -t short /dev/sda
```

---

### Q34. How do you do string manipulation in bash? Common patterns.

```bash
VAR="Hello World"

# Length
echo ${#VAR}                     # 11

# Substring
echo ${VAR:0:5}                  # Hello
echo ${VAR:6}                    # World

# Remove prefix/suffix
FILE="deploy_20241201.tar.gz"
echo ${FILE#deploy_}             # 20241201.tar.gz
echo ${FILE##*.}                 # gz (extension)
echo ${FILE%.tar.gz}             # deploy_20241201

# Replace
echo ${FILE/20241201/20241202}   # replace first occurrence
echo ${FILE//2024/2025}          # replace all

# Uppercase / lowercase (bash 4+)
echo ${VAR^^}    # HELLO WORLD
echo ${VAR,,}    # hello world

# Default values
echo ${VAR:-default}             # use default if VAR unset
echo ${VAR:?"Error: not set"}    # exit with error if unset

# Real use — extract version from git tag
TAG="v1.2.3-rc1"
VERSION=${TAG#v}                 # 1.2.3-rc1
MAJOR=${VERSION%%.*}             # 1
```

---

### Q35. Explain DNS resolution — what happens when you type "curl google.com"?

1. Process checks `/etc/hosts` → no match
2. Checks local DNS cache (`systemd-resolved`)
3. Queries resolver in `/etc/resolv.conf` (e.g. `8.8.8.8`)
4. If not cached → recursive lookup: root servers → `.com` TLD → `google.com` authoritative NS → returns A record (IP)
5. `curl` opens TCP connection to the IP on port 80/443
6. TLS handshake (if HTTPS)
7. HTTP request sent

```bash
dig google.com                   # detailed lookup
dig +trace google.com           # full recursive trace
dig google.com @8.8.8.8         # query specific resolver
getent hosts google.com         # respects /etc/hosts too
```

**Container DNS:** In Kubernetes, each pod's `/etc/resolv.conf` points to CoreDNS. Service discovery: `<service>.<namespace>.svc.cluster.local`

---

## 🔴 HARD / SCENARIO (Q36–Q50)

---

### Q36. A production server is responding slowly. Users report timeouts. You have SSH access. What is your exact diagnostic process?

```bash
# Minute 0–2: Baseline
uptime                    # load average vs CPU count
free -h                   # memory pressure?
df -h                     # disk full?
top -c                    # quick overview

# Minute 2–5: Find the bottleneck

# CPU bound?
ps aux --sort=-%cpu | head -10
mpstat -P ALL 1 5

# Memory bound?
vmstat 1 5                # check si/so (swap), r and b columns
cat /proc/meminfo | grep MemAvailable

# I/O bound?
iostat -x 1 5             # %util per disk → saturation?
iotop -o                  # which process is hammering disk?

# Network bound?
sar -n DEV 1 5
ss -s                     # TIME_WAIT buildup = connection leak

# Application level
tail -100 /var/log/app.log | grep -E "ERROR|SLOW|timeout"
journalctl -u app -n 100 --no-pager

# Minute 5+: Correlate
# Recent deployments?
# Cron jobs started? crontab -l + /var/log/cron
# System changes? last, /var/log/auth.log
```

**Communication:** update stakeholders every 5–10 minutes while investigating.

---

### Q37. You accidentally deleted /etc/passwd. How do you recover?

```bash
# If you still have active SSH sessions open (processes keep file descriptors)
# Recover minimal passwd immediately
echo "root:x:0:0:root:/root:/bin/bash" > /etc/passwd
echo "daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin" >> /etc/passwd
echo "youruser:x:1000:1000::/home/youruser:/bin/bash" >> /etc/passwd

# Restore from Debian auto-backup
cp /var/backups/passwd /etc/passwd

# Boot to recovery mode
# grub → add "single" to kernel args → mount -o remount,rw / → restore file

# From config management (Ansible/Puppet/Chef)
ansible-playbook site.yml --tags users
```

**Prevention:** `chattr +i /etc/passwd` (make immutable). Regular snapshots. Manage users via Ansible/Puppet.

---

### Q38. chmod 000 /bin/bash was run on a server. How do you fix it without rebooting?

**Key insight:** existing running processes still work. You cannot use bash to fix bash — use another interpreter.

```bash
# Option 1: use sh (dash)
sh -c 'chmod 755 /bin/bash'

# Option 2: use Perl
perl -e 'chmod 0755, "/bin/bash"'

# Option 3: use Python
python3 -c "import os; os.chmod('/bin/bash', 0o755)"

# Option 4: boot to recovery/rescue mode
# mount filesystem → chmod 755 /bin/bash

# Option 5: copy from another system
scp user@goodserver:/bin/bash /bin/bash
```

**Why this works:** permissions are checked at `exec()` time. Running bash processes already loaded into memory are unaffected.

---

### Q39. Explain the Linux kernel's OOM killer — how does it choose which process to kill?

The OOM killer triggers when the kernel cannot allocate memory and swap is exhausted.

**Selection:** each process gets an `oom_score` (0–1000). Higher = more likely to be killed.

**Factors:**
- Memory consumption (RSS) — biggest contributor
- Duration (long-running processes score lower — protecting systemd)
- Number of children
- Process priority (nice value)

```bash
# Check scores
cat /proc/<PID>/oom_score
cat /proc/<PID>/oom_score_adj   # manual: -1000 to +1000

# Protect a critical process (never kill)
echo -1000 > /proc/<PID>/oom_score_adj

# In systemd unit
OOMScoreAdjust=-900

# View OOM history
dmesg | grep -i "oom"
journalctl -k | grep -i "out of memory"
```

**Kubernetes:** OOM kill → pod restarts → `CrashLoopBackOff`. Fix: set proper `resources.limits.memory` so the right pod is killed and not a neighbour.

---

### Q40. How does a TCP connection get established and torn down? What is TIME_WAIT and why does it matter?

**3-way handshake (establish):**
```
Client → SYN → Server
Client ← SYN-ACK ← Server
Client → ACK → Server   → ESTABLISHED
```

**4-way teardown:**
```
Initiator → FIN → Peer
Initiator ← ACK ← Peer
Initiator ← FIN ← Peer
Initiator → ACK → Peer  → TIME_WAIT on initiator
```

**TIME_WAIT:** socket stays in this state for 2×MSL (~60–120 seconds) to ensure delayed packets from the old connection don't confuse a new connection on the same port.

**Problem at scale:** a busy load balancer initiating thousands of short connections can exhaust ephemeral ports while sockets are in TIME_WAIT.

```bash
ss -s | grep TIME-WAIT
ss -tan state time-wait | wc -l

# Fixes
sysctl net.ipv4.tcp_tw_reuse=1
sysctl net.ipv4.tcp_fin_timeout=15
sysctl net.ipv4.ip_local_port_range="10000 65535"
```

---

### Q41. You have a 50GB log file. How do you efficiently search and extract data without loading it all into memory?

```bash
# NEVER: cat 50gb.log | grep (loads into pipe)

# Search efficiently
grep "ERROR" app.log                  # streams file, doesn't load all
grep -m 100 "ERROR" app.log           # stop after 100 matches (faster!)

# Get specific line range without reading everything
sed -n '10000,10100p' app.log
awk 'NR==10000,NR==10100' app.log     # stops at line 10100

# Count lines
wc -l app.log

# Tail without reading all
tail -n 1000 app.log
tail -c 100M app.log                  # last 100MB

# Compressed logs — never decompress to disk!
zcat app.log.gz | grep "ERROR"
zgrep "ERROR" app.log.gz

# Parallel search across multiple compressed logs
parallel zgrep "ERROR" ::: logs/*.gz

# Split for repeated operations
split -l 1000000 app.log chunk_
```

**Real answer:** "For repeated analysis at this scale, I'd push it to Elasticsearch or use Splunk's indexer — which is what we use at ANZ for production log analysis."

---

### Q42. How do you audit and investigate a potentially compromised Linux server?

**Golden rule:** don't trust the compromised server's tools (attacker may have replaced `ls`, `ps`, `netstat`).

```bash
# Immediate containment
iptables -I INPUT -j DROP      # isolate — block ingress
# Take snapshot/image BEFORE touching anything

# Investigation
who; w; last; lastb            # who logged in recently

# Unusual processes (use /proc — harder to fake)
ls -la /proc/*/exe             # actual binaries for each process
cat /proc/net/tcp              # raw kernel network data

# Recent file changes
find / -mtime -1 -type f -not -path "/proc/*"
find / -perm -4000 -type f    # setuid files (backdoor check)

# Check for persistence
crontab -l
cat /etc/cron.d/*
systemctl list-units --type=service
cat /etc/rc.local

# Auth logs
grep "Accepted" /var/log/auth.log   # successful SSH logins
grep "Failed" /var/log/auth.log     # brute force attempts

# Rootkit check (from clean media)
chkrootkit
rkhunter --check
```

---

### Q43. Explain the difference between swap space and RAM. When does swapping become a problem?

**RAM:** fast, volatile. Active data and code live here.
**Swap:** disk-backed extension of memory. 100x–1000x slower even on SSD.

```bash
# Control swappiness
# 0  → avoid swap, prefer reclaim page cache (good for DBs)
# 10 → swap only under pressure (typical for databases)
# 60 → default, balanced

sysctl vm.swappiness=10
echo "vm.swappiness=10" >> /etc/sysctl.conf

# Diagnose
vmstat 1                     # si/so columns = swap in/out
sar -W 1 5                   # swap statistics
swapon -s                    # swap devices and usage
```

**When it's a problem:**
- Active swapping (`si`/`so` > 0) = severe performance degradation
- DB process getting swapped = catastrophic latency
- JVM: GC pause + swap pages = long stop-the-world pause

**Containers:** Kubernetes containers have no swap access by default when `memory-swap` equals `memory`. Always size memory limits correctly.

---

### Q44. How do you use tcpdump to capture and analyse network traffic?

```bash
tcpdump -i eth0                          # capture all on eth0
tcpdump -i any                           # all interfaces
tcpdump -i eth0 -w capture.pcap         # write to file (open in Wireshark)

# Filters
tcpdump -i eth0 port 8080               # specific port
tcpdump -i eth0 host 10.0.1.5          # specific host
tcpdump -i eth0 'tcp port 443 and host 10.0.1.5'
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'   # SYN packets only

# Useful flags
tcpdump -nn        # no DNS or port name resolution (faster)
tcpdump -A         # print ASCII payload (HTTP debugging)
tcpdump -s 0       # capture full packet (default truncates)
tcpdump -c 100     # capture only 100 packets

# Real scenarios
# Is my service receiving requests?
tcpdump -i eth0 -nn port 8080 -c 20

# Capture and send to analyst
tcpdump -i eth0 -w /tmp/cap.pcap -s 0 -c 10000
scp /tmp/cap.pcap analyst@jump:/tmp/
```

---

### Q45. Write a script to rotate logs — keep last 7 days, compress files older than 1 day.

```bash
#!/bin/bash
set -euo pipefail
LOG_DIR="/var/log/myapp"
DAYS_KEEP=7
DAYS_COMPRESS=1

# Compress logs older than 1 day (not already compressed)
find "$LOG_DIR" -name "*.log" -mtime +"$DAYS_COMPRESS" ! -name "*.gz" -exec gzip -9 {} \;

# Delete compressed logs older than 7 days
find "$LOG_DIR" -name "*.log.gz" -mtime +"$DAYS_KEEP" -delete

# Rotate active log and signal app to reopen
if [ -f "$LOG_DIR/app.log" ]; then
    mv "$LOG_DIR/app.log" "$LOG_DIR/app.log.$(date +%Y%m%d_%H%M%S)"
    [ -f /var/run/myapp.pid ] && kill -HUP $(cat /var/run/myapp.pid)
fi

remaining=$(find "$LOG_DIR" -type f | wc -l)
echo "$(date '+%Y-%m-%d %H:%M:%S') rotation done. $remaining files remain." >> "$LOG_DIR/rotation.log"

# Cron: 0 1 * * * /usr/local/bin/log_rotate.sh
```

**Interview note:** "In production I'd use the built-in `logrotate` with `/etc/logrotate.d/` configs — it handles `copytruncate` for apps that can't reopen logs and is battle-tested."

---

### Q46. What is a kernel panic? How do you diagnose and prevent it in production?

**Kernel panic:** fatal error the kernel cannot recover from. System halts or reboots.

**Common causes:** faulty RAM, buggy kernel module/driver, filesystem corruption, stack overflow, memory corruption.

```bash
# Configure kdump BEFORE a panic happens (RHEL)
yum install kexec-tools
systemctl enable kdump
# /etc/kdump.conf → set crash dump path
# Add crashkernel= to GRUB to reserve memory for dump kernel

# After crash — check logs
dmesg | tail -50
journalctl -b -1 -k           # kernel messages from previous boot

# Analyse crash dump
crash /usr/lib/debug/lib/modules/$(uname -r)/vmlinux /var/crash/vmcore
> bt       # backtrace
> log      # kernel log buffer
> ps       # running processes at crash time

# Hardware diagnostics
memtest86+           # boot-time memory test
smartctl -a /dev/sda
mcelog               # CPU/memory hardware error log
```

---

### Q47. Explain network namespaces and how Docker uses them for container networking.

```bash
# Create and use a network namespace manually
ip netns add myns
ip netns exec myns ip link show
ip netns exec myns bash
```

**Docker bridge mode — what happens per container:**

1. Docker creates `docker0` bridge on host
2. Creates a `veth` pair (virtual ethernet cable):
   - `veth0` → in host namespace, attached to `docker0` bridge
   - `veth1` → in container's NET namespace (appears as `eth0`)
3. Container gets IP from `docker0` subnet (`172.17.0.0/16` default)
4. NAT (`iptables MASQUERADE`) translates outbound traffic to host IP

**Port mapping (`-p 8080:80`):**

```bash
iptables -A PREROUTING -p tcp --dport 8080 -j DNAT --to 172.17.0.2:80
```

**Kubernetes CNI (Calico, Flannel, Cilium):** same pattern but CNI plugin handles IP assignment, veth setup, and cross-node routing/overlays.

```bash
nsenter -t <PID> -n ip addr    # enter container's network namespace from host
```

---

### Q48. Your application has intermittent latency spikes. The system looks healthy. How do you find the cause?

**Approach: capture data DURING the spike.**

```bash
# Pre-set continuous monitoring
sar -A 1 > /tmp/sar_capture.txt &
perf stat -a sleep 60 &

# During spike: syscall timing
strace -p <PID> -T -e trace=network,file    # -T shows time per syscall

# Common hidden causes + how to check
# GC pause (JVM): check GC logs → stop-the-world events
# Lock contention: strace shows long futex() waits
# DNS slowness: tcpdump port 53 during spike
# NFS/NAS latency: iostat shows high await on network mount
# CPU steal: top → %st column (hypervisor stealing CPU from your VM)
# NUMA mismatch: numastat → memory crossing NUMA nodes

# Modern low-overhead profiling with eBPF
bpftrace -e 'tracepoint:syscalls:sys_exit_read { @[comm] = hist(args->ret); }'

# Flame graph (find hot code paths)
perf record -F 99 -p <PID> -g -- sleep 30
perf script | flamegraph.pl > flame.svg
```

**Real answer:** "I'd first add p99 latency tracking at the application level, then correlate with system metrics. At ANZ, we use Dynatrace's distributed traces to pinpoint which microservice or DB call is causing spikes."

---

### Q49. How do you write a production-grade bash function library?

```bash
#!/bin/bash
# lib/common.sh — sourced by all scripts

# Guard against double-sourcing
[[ -n "${_LIB_COMMON_LOADED:-}" ]] && return 0
readonly _LIB_COMMON_LOADED=1

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Logging with timestamps
log()   { echo "[$(date '+%Y-%m-%dT%H:%M:%S%z')] [$1] ${*:2}" >&2; }
info()  { log INFO "$@"; }
warn()  { log WARN "$@"; }
error() { log ERROR "$@"; }
die()   { error "$@"; exit 1; }

# Retry with exponential backoff
retry() {
    local retries="$1" delay="$2"; shift 2
    local count=0
    until "$@"; do
        ((count++))
        [[ $count -ge $retries ]] && die "Failed after $retries attempts: $*"
        warn "Attempt $count failed. Retrying in ${delay}s..."
        sleep "$delay"
        delay=$((delay * 2))
    done
}

# Verify required tools are installed
require_cmds() {
    for cmd in "$@"; do
        command -v "$cmd" &>/dev/null || die "Required command not found: $cmd"
    done
}

# Usage in any script:
# source /opt/lib/common.sh
# require_cmds aws kubectl helm
# retry 3 5 kubectl apply -f manifest.yaml
```

---

### Q50. Explain how you would set up zero-downtime deployment using only Linux tools (no Kubernetes).

**Architecture:** HAProxy/Nginx → App1 + App2 (active-active)

**HAProxy config with health checks:**

```
backend app_servers
    balance roundrobin
    option httpchk GET /health
    server app1 10.0.1.1:8080 check
    server app2 10.0.1.2:8080 check
```

**Rolling restart script:**

```bash
#!/bin/bash
set -euo pipefail
HAPROXY_SOCK="/var/run/haproxy.sock"
SERVERS=(app1 app2)
BACKEND="app_servers"

for SERVER in "${SERVERS[@]}"; do
    echo "Deploying to $SERVER..."

    # Remove from load balancer
    echo "disable server $BACKEND/$SERVER" | socat stdio "$HAPROXY_SOCK"
    sleep 5   # drain existing connections

    # Deploy new version
    ssh "$SERVER" "systemctl stop myapp && \
                   cp /deploy/myapp.jar /opt/myapp/ && \
                   systemctl start myapp"

    # Wait for health check to pass
    for i in {1..30}; do
        curl -sf "http://$SERVER:8080/health" && break
        sleep 2
    done

    # Re-enable in load balancer
    echo "enable server $BACKEND/$SERVER" | socat stdio "$HAPROXY_SOCK"

    # Verify before moving to next server
    sleep 10
    curl -sf http://loadbalancer/health || { echo "LB unhealthy after $SERVER deploy"; exit 1; }
    echo "$SERVER done."
done
echo "Deployment complete."
```

**Rollback:** reverse the process with the old binary. For instant rollback, use blue/green: two full app sets, flip the LB all at once.

---

## 📌 Quick Reference — Commands to Know Cold

| Category | Commands |
|----------|----------|
| CPU | `top`, `htop`, `ps aux`, `pidstat`, `mpstat`, `perf top` |
| Memory | `free`, `vmstat`, `smem`, `cat /proc/meminfo` |
| Disk | `df -h`, `du -sh`, `iostat -x`, `lsof`, `find` |
| Network | `ss -tulnp`, `ip addr`, `ip route`, `tcpdump`, `nc`, `dig`, `mtr` |
| Process | `kill`, `strace`, `lsof -p`, `cat /proc/<PID>/` |
| Services | `systemctl`, `journalctl` |
| Files | `grep`, `awk`, `sed`, `find`, `tar`, `rsync` |
| Security | `iptables`, `ausearch`, `chcon`, `fail2ban` |
| Storage | `fdisk`, `lvcreate`, `lvextend`, `fsck`, `smartctl` |
| Scripting | `set -euo pipefail`, `trap`, `${VAR##*/}`, `getopts` |

---

*Next topic: CI/CD & Git — 50 questions*