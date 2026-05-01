# Text Processing

Commands to search, filter, transform, and extract data from files and command output.

***

## `grep`

```bash
grep "error" app.log
grep -i "error" app.log              # case insensitive
grep -r "password" /etc/             # recursive search
grep -n "FAIL" app.log               # show line numbers
grep -v "DEBUG" app.log              # exclude matching lines
grep -c "error" app.log              # count occurrences only
grep -E "error|fail|warn" app.log    # OR pattern (extended regex)
grep -A 3 -B 2 "exception" app.log   # 3 lines after, 2 lines before
grep -iE "oom|kill|fail" /var/log/syslog
```

***

## `awk`

Field/column processor — powerful for structured log data.

```bash
awk '{print $1, $3}' app.log          # print columns 1 and 3
awk -F: '{print $1}' /etc/passwd      # use : as delimiter
awk '$3 > 80 {print $1}' file         # conditional filter
awk '{print $NF}' app.log             # print last field
awk '{print $(NF-1)}' app.log         # print second-to-last field
awk '{sum+=$2} END {print sum}' file  # aggregate / sum a column
```

**Unique count of last field (e.g. error types):**

```bash
awk '{print $NF}' app.log | sort | uniq -c | sort -rn | head -10
```

***

## `sed`

Stream editor — find/replace and line operations.

```bash
sed 's/foo/bar/g' file               # replace all (stdout only)
sed -i 's/old/new/g' file           # in-place edit
sed '/^#/d' file                     # delete comment lines
sed -n '10,20p' file                 # print only lines 10–20
sed -i 's/password=.*/password=REDACTED/g' app.env   # mask secrets
sed -i 's/LOG_LEVEL=debug/LOG_LEVEL=info/g' app.env  # change log level
```

**Use in CI/CD:** patch config files in pipelines without opening an editor. Powerful for bulk replacements.

***

## `find`

```bash
find . -type f -name "*.log"                       # by name
find /var/log -mtime -7                            # modified last 7 days
find / -user root -perm -4000                      # setuid files (security audit)
find /tmp -type f -mmin +60 -delete               # delete files older than 60 mins
find / -type f -size +50M                          # files larger than 50MB
find /app -name "*.conf" -exec grep -l "password" {} \;  # search inside files
```

***

## `xargs`

Passes output of one command as arguments to another.

```bash
# Search error string inside all log files
find /var/log -type f | xargs grep "error" 2>/dev/null

# Delete old temp files found by find
find /tmp -mtime +7 -type f | xargs rm -f

# Grep inside all .log files
find . -name "*.log" | xargs grep -i "exception"
```

**Why xargs?** `find` gives filenames. `xargs` feeds them to the next command. More efficient than `-exec` for large numbers of files.

***

## Pipe Chaining — The Linux Superpower

Each `|` passes the output of one command as input to the next. You can chain as many as needed.

```bash
# Top 10 error types in a log file
cat app.log | grep "ERROR" | awk '{print $NF}' | sort | uniq -c | sort -rn | head -10

# Top IPs hitting your server
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -20

# Filesystems over 70% usage
df -h | awk 'NR>1 && $5+0 > 70 {print $5, $6}'
```
