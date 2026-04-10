# Quick Troubleshooting Reference

Fast command sequences for the most common production scenarios.

---

## Server is slow

```bash
uptime              # check load average vs CPU count
free -h             # memory pressure?
vmstat 1 5          # swap, I/O wait, CPU breakdown
top -c              # which process is the culprit?
iostat -x 1 3       # disk bottleneck?
```

---

## High CPU

```bash
top -c                           # identify process
ps aux --sort=-%cpu | head -10
pidstat -u 1 5                   # per-PID CPU over time
strace -p <PID> -c               # what syscalls is it making?
```

---

## Disk full

```bash
df -h                            # which filesystem is full?
df -ih                           # inode exhaustion?
du -sh /var/log/* | sort -rh     # what's consuming space?
lsof | grep deleted              # deleted files still held open?
```

---

## Port not responding

```bash
ss -tulnp | grep <port>          # is the process even listening?
systemctl status <service>       # is the service running?
journalctl -u <service> -n 50   # any errors in the logs?
nc -zv localhost <port>          # can you connect locally?
```

---

## Network connectivity issue

```bash
ping <host>                      # basic reachability
mtr <host>                       # where is it dropping?
nc -zv <host> <port>             # is the port reachable?
tcpdump -i eth0 port <port> -c 20  # are packets arriving?
```

---

## Process crashed / not starting

```bash
dmesg -T | grep -i kill          # OOM killed?
journalctl -u <service> -n 100  # service logs
systemctl status <service>       # exit code and last error
```

---

## DNS not resolving

```bash
nslookup hostname                # basic check
dig hostname                     # detailed
dig +trace hostname              # full recursive trace
cat /etc/resolv.conf             # which nameserver is configured?
```

---

## Disk full but du shows less

```bash
lsof | grep deleted              # find deleted files held open by processes
# Restart the process holding the file, or truncate it:
> /var/log/app.log
```

---

## Common Command Reference

| Scenario | Commands |
|----------|----------|
| Check CPU | `top`, `htop`, `ps aux --sort=-%cpu`, `mpstat` |
| Check memory | `free -h`, `vmstat`, `cat /proc/meminfo` |
| Check disk | `df -h`, `df -ih`, `du -sh`, `iostat -x` |
| Check network | `ss -tulnp`, `ip addr`, `mtr`, `tcpdump` |
| Check processes | `ps -ef`, `pgrep`, `kill`, `strace` |
| Check services | `systemctl status`, `journalctl -u` |
| Search logs | `grep -i`, `awk`, `tail -f`, `journalctl -f` |
| Security audit | `find / -perm -4000`, `iptables -L`, `last`, `who` |
