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
