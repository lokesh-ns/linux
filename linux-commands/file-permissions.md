# File Permissions

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
