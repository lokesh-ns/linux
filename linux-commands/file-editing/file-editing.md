# File Editing

Commands to create, view, and edit files directly on the server — essential for config changes, debugging, and on-the-fly fixes without a GUI.

***

## `vi` / `vim`

The standard editor on every Linux server. Always available — even on minimal installs where `nano` may not be.

```bash
vi file.txt          # open file (vi)
vim file.txt         # open file (vim — improved vi)
vim +50 file.txt     # open and jump to line 50
vim +/error file.txt # open and jump to first match of "error"
```

**vim has two primary modes:**

| Mode    | How to Enter           | Purpose                       |
| ------- | ---------------------- | ----------------------------- |
| Normal  | Default / press `Esc`  | Navigate, delete, copy, paste |
| Insert  | Press `i`, `a`, or `o` | Type and edit text            |
| Visual  | Press `v`              | Select text                   |
| Command | Press `:`              | Save, quit, search, replace   |

***

### Essential vim Commands

**Entering Insert Mode:**

| Key | Behaviour                 |
| --- | ------------------------- |
| `i` | Insert before cursor      |
| `a` | Insert after cursor       |
| `o` | New line below and insert |
| `O` | New line above and insert |

**Saving and Quitting (from Normal mode, press `:` first):**

| Command       | Action                                |
| ------------- | ------------------------------------- |
| `:w`          | Save (write)                          |
| `:q`          | Quit                                  |
| `:wq` or `:x` | Save and quit                         |
| `:q!`         | Quit without saving (force)           |
| `:wq!`        | Save and quit (force, even read-only) |

**Navigation:**

| Key      | Action                 |
| -------- | ---------------------- |
| `gg`     | Jump to top of file    |
| `G`      | Jump to bottom of file |
| `50G`    | Jump to line 50        |
| `Ctrl+f` | Page down              |
| `Ctrl+b` | Page up                |
| `0`      | Start of line          |
| `$`      | End of line            |

**Editing in Normal Mode:**

| Key      | Action                        |
| -------- | ----------------------------- |
| `dd`     | Delete (cut) current line     |
| `yy`     | Yank (copy) current line      |
| `p`      | Paste after cursor            |
| `u`      | Undo                          |
| `Ctrl+r` | Redo                          |
| `x`      | Delete character under cursor |
| `dw`     | Delete word                   |
| `5dd`    | Delete 5 lines                |

**Search and Replace:**

```bash
/keyword          # search forward
n                 # next match
N                 # previous match
:%s/old/new/g     # replace all occurrences in file
:%s/old/new/gc    # replace with confirmation per match
:10,20s/old/new/g # replace only in lines 10–20
```

**Interview tip:** The most common vim scenario in interviews&#x20;

"you SSH into a server and need to change a config value. How?" Answer: `vim /etc/nginx/nginx.conf`, navigate with `/keyword`, press `i` to edit, `Esc` then `:wq` to save.

***

## 2. `nano`

Simpler than vim — menu shortcuts shown at the bottom of the screen. Good for quick edits.

```bash
nano file.txt          # open file
nano +50 file.txt      # open at line 50
nano /etc/hosts        # edit hosts file
```

**nano shortcuts (shown at bottom of screen as `^` = Ctrl):**

| Shortcut | Action           |
| -------- | ---------------- |
| `Ctrl+O` | Save (Write Out) |
| `Ctrl+X` | Exit             |
| `Ctrl+W` | Search           |
| `Ctrl+K` | Cut line         |
| `Ctrl+U` | Paste line       |
| `Ctrl+G` | Help             |

**vim vs nano:**

|                      | vim                       | nano              |
| -------------------- | ------------------------- | ----------------- |
| Available everywhere | ✅ Yes                     | ❌ Not always      |
| Learning curve       | Steep                     | Low               |
| Best for             | Config editing, scripting | Quick small edits |
| DevOps interviews    | Asked frequently          | Rarely asked      |

> For interviews and real production work, know vim. Use nano when you just need to quickly change one line and don't want to think about modes.

***

## 3. Editing Files Without an Editor

For scripts and CI/CD pipelines where you can't open an interactive editor:

```bash
# Append a line to a file
echo "export PATH=$PATH:/usr/local/bin" >> ~/.bashrc

# Overwrite entire file content
echo "server_name localhost;" > /etc/nginx/conf.d/default.conf

# Append multiple lines using heredoc
cat >> /etc/hosts << EOF
192.168.1.10  app-server
192.168.1.11  db-server
EOF

# In-place replace using sed (no editor needed)
sed -i 's/LOG_LEVEL=debug/LOG_LEVEL=info/g' app.env
sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
```

**`>>` vs `>`:**

| Operator | Behaviour                         |
| -------- | --------------------------------- |
| `>`      | Overwrite — replaces file content |
| `>>`     | Append — adds to end of file      |

> ⚠️ `>` on a file that already exists will silently wipe its content. Use `>>` unless you intend to overwrite.

***

## 4. `touch` for File Creation

```bash
touch newfile.txt               # create empty file
touch file1.txt file2.txt       # create multiple files
touch -t 202501010000 file.txt  # set a specific timestamp
```

Used in scripts to create placeholder files, trigger file-watcher pipelines, or reset modification timestamps.

***

## Quick Reference — File Editing Cheat Sheet

```bash
# Open and edit a config file
vim /etc/nginx/nginx.conf

# Jump to a specific line
vim +142 /etc/nginx/nginx.conf

# Search and replace inside vim
:%s/worker_processes 1/worker_processes 4/g

# Quick non-interactive edit (CI/CD friendly)
sed -i 's/replicas: 1/replicas: 3/' deployment.yaml

# Append to a file without opening editor
echo "127.0.0.1 myapp.local" >> /etc/hosts

# View file with line numbers
cat -n /etc/nginx/nginx.conf | grep -n "server_name"
```

**Interview tip:** When asked about editing files on a production server — always mention checking if there's a backup first:

```bash
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
vim /etc/nginx/nginx.conf
nginx -t && systemctl reload nginx
```

Test the config (`nginx -t`) before reloading — never blind-reload on production.
