# Disk & Storage

Commands to check disk usage, find large files, manage storage, and repair filesystems.

---

## 6. `df`

```bash
df -h           # human readable
df -hT          # include filesystem type
df -ih          # inode usage — CRITICAL
```

**Watch:** `Use%` column. Over 80% — investigate and clean up.

**Inode trap:** `df -h` shows space available but `touch newfile` fails with _"No space left on device"_? Run `df -ih`. You've exhausted inodes — common with mail servers, container builds, and session caches that create millions of tiny files.

---

## 7. `du`

```bash
du -sh /var/log              # total size of a directory
du -sh *                     # size of each item in current dir
du -sh /opt/myapp            # specific folder
```

**Find top space consumers:**

```bash
du -sh /var/log/* | sort -rh | head -20
```

---

## 8. `iostat`

```bash
iostat
iostat -x -h 1 3        # extended, human readable, every 1s, 3 times
```

**Key columns (with `-x`):**

| Column | Meaning |
|--------|---------|
| `%util` | Disk busy % — 100% = saturated/bottleneck |
| `await` | Avg ms per I/O request — high = slow disk |
| `r/s` / `w/s` | Reads / writes per second |

**Use:** when `vmstat` shows high `wa` (I/O wait), run `iostat` to identify which disk device is the bottleneck.

---

## 9. `lsblk`

```bash
lsblk
```

Tree view of all disks, partitions, sizes, and mount points. Run before partitioning or extending volumes to understand the storage layout.

---

## 10. `fdisk`

```bash
fdisk -l        # list all disks and partitions
```

Shows partition table type, sector sizes, and disk geometry. Used before manual partitioning.
