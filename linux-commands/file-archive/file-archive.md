# File & Archive

Commands to compress files, inspect open file handles, and manage permissions.

---

## 34. `tar`

```bash
tar -czvf archive.tar.gz /path/dir      # create gzip archive
tar -xzvf archive.tar.gz               # extract
tar -tzvf archive.tar.gz               # list contents without extracting
tar -czvf backup.tar.gz /etc --exclude=/etc/ssl   # with exclusion

# Timestamped backup
tar -czf backup-$(date +%Y%m%d).tar.gz /app/data
```

**Flags:**

| Flag | Meaning |
|------|---------|
| `c` | Create archive |
| `x` | Extract archive |
| `z` | gzip compression |
| `j` | bzip2 compression |
| `v` | Verbose output |
| `f` | Filename follows |

---

## 35. `lsof`

```bash
lsof -i :80               # which process is using port 80
lsof -p <PID>             # all files opened by a process
lsof +D /var/log          # all open files in a directory
lsof | grep deleted       # deleted files still held open
```

**Disk leak scenario:** `df -h` shows disk full but `du` shows less usage — a deleted file is still held open by a running process consuming space. Find it:

```bash
lsof | grep deleted
# then restart that process, or truncate the file:
> /var/log/app.log
```

---

## 36. `chmod`

```bash
chmod 755 script.sh       # rwx r-x r-x
chmod 644 config.conf     # rw- r-- r--
chmod 700 private.key     # rwx --- ---
chmod +x script.sh        # add execute for owner
```

**Numeric values:**

| Value | Permission |
|-------|------------|
| `4` | read (r) |
| `2` | write (w) |
| `1` | execute (x) |
| `7` | rwx |
| `6` | rw- |
| `5` | r-x |
| `4` | r-- |

**Special bits:**

| Bit | Value | Effect |
|-----|-------|--------|
| setuid | `4xxx` | Run as file owner |
| setgid | `2xxx` | Run as group |
| sticky | `1777` | Only owner can delete (used on `/tmp`) |

---

## 37. `chown`

```bash
chown user:group file.txt             # change owner and group
chown -R devuser:devteam /opt/app     # recursive
chown root:root /etc/passwd           # restore root ownership
```

---

## 38. `chroot`

```bash
chroot /mnt/recovery /bin/bash        # switch root to recovery mount
```

Used in rescue mode — boot from live USB, mount the broken filesystem, chroot into it to fix corrupted `/etc/fstab`, broken GRUB, or reset passwords.
