# Disk Storage

Commands to check disk usage, find large files, manage storage, and repair filesystems.

***

## `df`

```bash
df -h           # human readable
df -hT          # include filesystem type
df -ih          # inode usage — CRITICAL
```

**Watch:** `Use%` column. Over 80% — investigate and clean up.

**Inode trap:** `df -h` shows space available but `touch newfile` fails with _"No space left on device"_?&#x20;

Run `df -ih`. You've exhausted inodes — common with mail servers, container builds, and session caches that create millions of tiny files.

### 📦 What is an inode?

An **inode** is:

> <mark style="color:$danger;">A data structure that stores metadata about a file (not the content)</mark>

Each file consumes:

* **1 inode**

***

### ⚠️ Important rule

> You need BOTH:

* Disk space
* AND free inodes

```
Filesystem      Inodes IUsed IFree IUse% Mounted on
/dev/root         896K  156K  741K   18% /
tmpfs             114K     2  114K    1% /dev/shm
```

***

## `du`

It tells you **how much space files and directories are actually using**

```bash
du -h
du -sh /var/log              # total size of a directory
du -sh *                     # size of each item in current dir
du -sh /opt/myapp            # specific folder
```

**Find top space consumers:**

```bash
du -sh /var/log/* | sort -rh | head -20
```

***

## `iostat` = I/O statistics

<pre class="language-bash"><code class="lang-bash">iostat
<strong>iostat -x -h 1 3        # extended, human readable, every 1s, 3 times
</strong></code></pre>

It shows **CPU usage + disk I/O performance**

Used to answer:

* Is disk slow?
* Is CPU waiting on disk?
* Which device is bottleneck?

**Key columns (with `-x`):**

| Column        | Meaning                                   |
| ------------- | ----------------------------------------- |
| `%util`       | Disk busy % — 100% = saturated/bottleneck |
| `await`       | Avg ms per I/O request — high = slow disk |
| `r/s` / `w/s` | Reads / writes per second                 |

**Use:** when `vmstat` shows high `wa` (I/O wait), run `iostat` to identify which disk device is the bottleneck.



***

## `fdisk`

```bash
fdisk -l        # list all disks and partitions
```

Shows partition table type, sector sizes, and disk geometry. Used before manual partitioning.
