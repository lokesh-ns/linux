# Users & Security

Commands to manage users, groups, permissions, firewall rules, and SSH hardening.

---

## 39. `id` / `whoami`

```bash
id                        # current user — UID, GID, all groups
id devuser                # specific user details
whoami                    # just the current username
```

---

## 40. `usermod`

```bash
usermod -aG docker username           # add user to docker group
usermod -aG staging-access devuser    # add to another group
usermod -s /sbin/nologin svcuser      # disable interactive shell
```

> ⚠️ The `-a` flag (append) is critical. Without it, the user is removed from all other groups and only added to the new one.

**Service accounts best practice:**

```bash
useradd -s /sbin/nologin -M -r svcuser   # no shell, no home dir, system account
```

---

## 41. `groupmod`

```bash
groupmod -n newname oldname          # rename a group
```

---

## 42. SSH Hardening

```bash
# Check current values
grep -E "PasswordAuthentication|PermitRootLogin" /etc/ssh/sshd_config
```

**Recommended `/etc/ssh/sshd_config` settings:**

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers deployuser jenkins
MaxAuthTries 3
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 2
X11Forwarding no
AllowTcpForwarding no
KexAlgorithms curve25519-sha256
Ciphers aes256-gcm@openssh.com,chacha20-poly1305@openssh.com
Port 2222
```

```bash
systemctl restart sshd
systemctl enable fail2ban    # auto-ban IPs with repeated failed attempts
```

---

## 43. `iptables`

```bash
iptables -L -n -v --line-numbers                 # list all rules
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT  # allow port 8080
iptables -A INPUT -s 192.168.1.100 -j DROP       # block an IP
iptables -I INPUT 1 -p tcp --dport 443 -j ACCEPT # insert rule at top
iptables -D INPUT 3                              # delete rule #3
# Allow established connections — always add this
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
# Save rules
iptables-save > /etc/iptables/rules.v4
```
