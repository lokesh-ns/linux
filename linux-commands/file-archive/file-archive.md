# File Archive

Commands to compress files, inspect open file handles, and manage permissions.

***

## `tar`

```bash
tar -czvf archive.tar.gz /path/dir      # create gzip archive
tar -xzvf archive.tar.gz               # extract

tar -tzvf archive.tar.gz               # list contents without extracting
tar -czvf backup.tar.gz /etc --exclude=/etc/ssl   # with exclusion
```

**Flags:**

| Flag | Meaning           |
| ---- | ----------------- |
| `c`  | Create archive    |
| `x`  | Extract archive   |
| `z`  | gzip compression  |
| `j`  | bzip2 compression |
| `v`  | Verbose output    |
| `f`  | Filename follows  |

***

## &#x20;`lsof`

`lsof` = **List Open Files**

👉 In Linux/Unix:

> **Everything is a file** (regular files, sockets, ports, devices)

So `lsof` shows:

* Which files are open
* Which process opened them
* Which ports are being used

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

***

## `chmod`

```bash
chmod 755 script.sh       # rwx r-x r-x
chmod 644 config.conf     # rw- r-- r--
chmod 700 private.key     # rwx --- ---
chmod +x script.sh        # add execute for owner
```

**Numeric values:**

| Value | Permission  |
| ----- | ----------- |
| `4`   | read (r)    |
| `2`   | write (w)   |
| `1`   | execute (x) |
| `7`   | rwx         |
| `6`   | rw-         |
| `5`   | r-x         |
| `4`   | r--         |

**Special bits:**

| Bit    | Value  | Effect                                 |
| ------ | ------ | -------------------------------------- |
| setuid | `4xxx` | Run as file owner                      |
| setgid | `2xxx` | Run as group                           |
| sticky | `1777` | Only owner can delete (used on `/tmp`) |

***

## `chown`

`chown` = **change owner**\
\
It changes **who owns a file/directory (user)** and optionally its **group**

```bash
chown user:group file.txt             # change owner and group
chown -R devuser:devteam /opt/app     # recursive
chown root:root /etc/passwd           # restore root ownership
```
