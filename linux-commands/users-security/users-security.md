# User Management

Commands to manage users, groups, permissions, firewall rules, and SSH hardening.

***

## `id` / `whoami`

```bash
id                        # current user — UID, GID, all groups
id devuser                # specific user details
whoami                    # just the current username
```

***

## `useradd`

```bash
sudo useradd loki
```

***

## `userdel`

```bash
sudo userdel loki 
```

***

## `passwd`

```bash
sudo userpasswd loki
```

***

## su

```bash
su loki
```

***

## `sudo`

```bash
sudo <command>   # command for root privelage
```

## `usermod`

```bash
usermod -aG docker username           # add user to docker group
usermod -aG staging-access devuser    # add to another group

```

> ⚠️ The `-a` flag (append) is critical. Without it, the user is removed from all other groups and only added to the new one.

***

## `groupmod`

```bash
groupmod -n newname oldname          # rename a group
```
