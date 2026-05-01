# Navigation & Basic File Operations

Commands to move around the filesystem, inspect files, and perform everyday file management — the first commands you use every time you SSH into a server.

---

## 1. `ls`

```bash
ls                    # list files in current directory
ls -l                 # long format — permissions, owner, size, date
ls -a                 # show hidden files (dotfiles)
ls -al                # long format + hidden files
ls -ltr               # sort by time, oldest first (most useful for logs)
ls -lh                # human-readable file sizes
ls -lS                # sort by file size, largest first
ls /var/log           # list a specific directory
```

**`ls -ltr` is the most used in DevOps** — when you SSH into a server and want to see the most recently modified files at the bottom.

**Output columns (`ls -l`):**

| Column | Meaning |
|---|---|
| `-rw-r--r--` | Permissions (file type + owner/group/other) |
| `1` | Hard link count |
| `root root` | Owner and group |
| `1234` | File size in bytes |
| `Apr 28 10:30` | Last modified time |
| `file.txt` | Filename |

---

## 2. `pwd`

```bash
pwd
# /home/ubuntu/app
```

Print Working Directory — shows your exact current location in the filesystem. Use this when you're lost or before running destructive commands.

---

## 3. `mkdir`

```bash
mkdir mydir                        # create a directory
mkdir -p /opt/app/config/logs      # create nested directories (no error if exists)
mkdir -m 755 /opt/app              # create with specific permissions
```

> `-p` flag is essential in scripts and CI/CD pipelines — prevents failure if the directory already exists.

---

## 4. `cd`

```bash
cd /var/log             # absolute path
cd app/logs             # relative path
cd ..                   # go up one level
cd ../..                # go up two levels
cd ~                    # go to home directory
cd -                    # go back to previous directory (very useful)
```

**Interview tip:** `cd -` switches between your last two directories — useful when toggling between `/etc/nginx` and `/var/log/nginx` during debugging.

---

## 5. `touch`

```bash
touch file.txt                     # create empty file or update timestamp
touch app.log config.env           # create multiple files at once
touch -t 202501010000 file.txt     # set specific timestamp
```

Commonly used in scripts to create placeholder files or trigger file-watcher pipelines.

---

## 6. `cat`

```bash
cat file.txt                       # print file contents
cat -n file.txt                    # with line numbers
cat file1.txt file2.txt            # concatenate two files
cat /etc/os-release                # check OS version
cat /proc/cpuinfo                  # CPU details
cat /proc/meminfo                  # memory details
cat >> file.txt                    # append interactively (Ctrl+D to save)
```

**Interview tip:** For large files, `cat` is inefficient — use `less` or `tail` instead.

---

## 7. `head` / `tail`

```bash
head file.txt              # first 10 lines (default)
head -n 20 file.txt        # first 20 lines
head -n 1 /etc/passwd      # first line only

tail file.txt              # last 10 lines (default)
tail -n 50 app.log         # last 50 lines
tail -f /var/log/nginx/access.log    # follow live (real-time log monitoring)
tail -f app.log | grep ERROR         # follow + filter
```

**`tail -f` is one of the most-used commands in DevOps** — live log monitoring during deployments or incidents.

---

## 8. `more` / `less`

```bash
more file.txt              # paginate output — space to scroll, q to quit
less file.txt              # better than more — scroll up/down, search with /
less +F file.txt           # follow mode (like tail -f, but with scroll back)
```

**`less` shortcuts:**

| Key | Action |
|---|---|
| `Space` | Next page |
| `b` | Previous page |
| `/keyword` | Search forward |
| `n` | Next match |
| `G` | Jump to end |
| `g` | Jump to start |
| `q` | Quit |

Use `less` over `cat` for any file larger than a screen.

---

## 9. `cp`

```bash
cp file.txt backup.txt                    # copy file
cp -r /opt/app /opt/app-backup            # copy directory recursively
cp -p file.txt backup.txt                 # preserve permissions and timestamps
cp -i file.txt dest/                      # prompt before overwrite
cp *.log /tmp/log-backup/                 # copy multiple files with glob
```

> ⚠️ `cp` without `-i` silently overwrites. Always use `-i` when copying to a directory that may have existing files.

---

## 10. `mv`

```bash
mv oldname.txt newname.txt           # rename a file
mv file.txt /opt/app/               # move to another directory
mv -i file.txt /opt/                 # prompt before overwrite
mv *.log /var/log/archive/           # move multiple files
```

`mv` is also how you rename files in Linux — there is no separate `rename` command in most distros.

---

## 11. `rm`

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

---

## 12. `echo`

```bash
echo "Hello World"                        # print to stdout
echo $HOME                                # print environment variable
echo "export PATH=$PATH:/usr/local/bin" >> ~/.bashrc   # append to file
echo -e "line1\nline2"                    # interpret escape sequences
echo -n "no newline"                      # suppress trailing newline
```

Used heavily in shell scripts for output, debugging, and writing to files.

---

## 13. `wc`

```bash
wc file.txt               # lines, words, bytes
wc -l file.txt            # line count only
wc -w file.txt            # word count only
wc -c file.txt            # byte count only

# Real-world usage
grep "ERROR" app.log | wc -l            # count error lines
ls /var/log/*.log | wc -l              # count log files
cat /etc/passwd | wc -l                # number of users
```

**Interview tip:** `wc -l` combined with `grep` is your quick way to quantify errors — "how many 500 errors in the last hour?"

---

## 14. `ln` — Hard and Soft Links

```bash
# Soft (Symbolic) Link — like a shortcut, points to the path
ln -s /Users/lokeshns/shell-scripting/new.txt s-new
ln -s /opt/app/current /opt/app/live    # deployment pattern

# Hard Link — points directly to the inode
ln /Users/lokeshns/shell-scripting/new.txt h-new
```

**Soft vs Hard link:**

| | Soft Link | Hard Link |
|---|---|---|
| Works across filesystems | ✅ Yes | ❌ No |
| Works for directories | ✅ Yes | ❌ No |
| Breaks if original deleted | ✅ Yes (dangling) | ❌ No (file survives) |
| Use case | Deployment aliases, config shortcuts | Backup references |

**DevOps use case:** Symlinks are used in deployment strategies — `/opt/app/current -> /opt/app/releases/v1.2.3`. Rolling back = just updating the symlink.

---

## 15. `cut`

```bash
cut -b 4 test.txt              # extract 4th byte/character
cut -b 1-4 test.txt            # extract characters 1 to 4
cut -d: -f1 /etc/passwd        # delimiter : , extract field 1 (usernames)
cut -d, -f2 data.csv           # extract column 2 from CSV
cut -d' ' -f1 access.log       # extract first field (IP) from nginx log
```

**Use:** extracting specific columns from structured text — log files, CSVs, `/etc/passwd`.

---

## 16. `tee`

```bash
echo "hai lokesh" | tee test.txt          # print AND save to file
echo "hai lokesh" | tee -a test.txt       # print AND append to file
command 2>&1 | tee output.log             # capture both stdout and stderr
```

**Why tee?** Without `tee`, you can either print output OR save it — not both. `tee` does both simultaneously.

**Use in CI/CD:**
```bash
./deploy.sh 2>&1 | tee deploy-$(date +%Y%m%d).log
```
Saves deployment output to a dated log file while still showing it on screen.

---

## 17. `sort`

```bash
sort file.txt                    # alphabetical sort
sort -r file.txt                 # reverse order
sort -n file.txt                 # numeric sort
sort -k2 file.txt                # sort by column 2
sort -u file.txt                 # sort and remove duplicates
sort -rn file.txt | head -10     # top 10 largest numbers

# Real-world: top IPs by frequency
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

---

## 18. `diff`

```bash
diff file1.txt file2.txt         # compare two files
diff -u file1.txt file2.txt      # unified format — cleaner, shows context
diff -r dir1/ dir2/              # compare two directories recursively
```

**`diff -u` output:**
```
--- file1.txt   2025-01-01
+++ file2.txt   2025-01-02
@@ -3,4 +3,4 @@
 unchanged line
-old line
+new line
 unchanged line
```

Lines with `-` were removed, `+` were added. This is the same format used by `git diff`.

**Use:** comparing config files before and after a change, verifying deployment artifacts.

---

## 19. `clear`

```bash
clear        # clear terminal screen
Ctrl+L       # keyboard shortcut — same effect
```

---

## Quick Reference — Navigation Cheat Sheet

```bash
# Where am I and what's here?
pwd && ls -ltr

# Go somewhere fast
cd /var/log && ls -ltr | tail -5

# Find recently changed files
find /etc -mtime -1 -type f

# Count lines in a file
wc -l /var/log/syslog

# Watch a log live
tail -f /var/log/nginx/access.log | grep "500"

# Compare two config versions
diff -u /etc/nginx/nginx.conf.bak /etc/nginx/nginx.conf
```