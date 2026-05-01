# File & Directory

Commands to move around the filesystem, inspect files, and perform everyday file management — the first commands you use every time you SSH into a server.

***

## `ls`

```bash
ls                    # list files in current directory
ls -l                 # long format — permissions, owner, size, date
ls -a                 # show hidden files (dotfiles)
ls -al                # long format + hidden files
ls -ltr               # sort by time, oldest first (most useful for logs)
ls /var/log           # list a specific directory
```

**`ls -ltr` is the most used in DevOps** — when you SSH into a server and want to see the most recently modified files at the bottom.

**Output columns (`ls -l`):**

| Column         | Meaning                                     |
| -------------- | ------------------------------------------- |
| `-rw-r--r--`   | Permissions (file type + owner/group/other) |
| `1`            | Hard link count                             |
| `root root`    | Owner and group                             |
| `1234`         | File size in bytes                          |
| `Apr 28 10:30` | Last modified time                          |
| `file.txt`     | Filename                                    |

***

## `pwd`

```bash
pwd
# /home/ubuntu/app
```

Print Working Directory — shows your exact current location in the filesystem. Use this when you're lost or before running destructive commands.

***

## `mkdir`

```bash
mkdir mydir                        # create a directory
mkdir -p /opt/app/config/logs      # create nested directories (no error if exists)
```

> `-p` flag is essential in scripts and CI/CD pipelines — prevents failure if the directory already exists.

***

## `cd`

```bash
cd /var/log             # absolute path
cd app/logs             # relative path
cd ..                   # go up one level
cd ../..                # go up two levels
cd ~                    # go to home directory
cd -                    # go back to previous directory (very useful)
```

**Interview tip:** `cd -` switches between your last two directories — useful when toggling between `/etc/nginx` and `/var/log/nginx` during debugging.

***

## `rm`

```bash
rm file.txt                          # delete a file
rm -i file.txt                       # prompt before delete
rm -r /opt/old-app/                  # delete directory recursively
rm -rf /tmp/cache/                   # force delete, no prompts
rmdir emptydir/                      # delete empty directory only
```

> ⚠️ `rm -rf` is irreversible. Always double-check the path. A common horror story: `rm -rf /opt/ app/` (note the accidental space) deletes `/opt/` entirely.

**Safe delete habit in scripts:**

```bash
TARGET="/opt/old-app"
[ -d "$TARGET" ] && rm -rf "$TARGET"
```

***

## `clear`

```bash
clear        # clear terminal screen
Ctrl+L       # keyboard shortcut — same effect
```

***
