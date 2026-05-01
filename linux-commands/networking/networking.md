# Networking

Commands to check open ports, test connectivity, capture traffic, and debug DNS.

***

## `ping`

`ping` is used to **check network connectivity** between your machine and another host.

```bash
ping google.com
```

***

## `ss`

`ss` = **socket statistics**\
👉 It shows **network connections, listening ports, and socket info**

It’s the **modern replacement for `netstat`** (faster and more detailed).

```bash
ss -tulnp       # TCP+UDP listening ports with process names
ss -s           # socket statistics summary
```

**Flags:** `t`=TCP, `u`=UDP, `l`=listening, `n`=numeric ports, `p`=show process

Modern replacement for `netstat`. Faster and more detailed.

***

## `netstat` (legacy)

```bash
netstat -tulnp      # listening ports with processes
netstat -rn         # routing table
```

Still found on older systems. Use `ss` on modern Linux.

***

## `curl`

It’s used to **send HTTP/HTTPS requests** and interact with APIs or web servers from the command line.

```bash
curl https://example.com
```

**HTTP status codes:** `200`=OK, `301/302`=redirect, `4xx`=client error, `5xx`=server error.

***

## `wget`

It’s used to **download files from the internet** (HTTP, HTTPS, FTP)

```bash
wget https://example.com/file.zip
```

***

## `mtr`

Shows every network hop between you and the destination with live packet loss %. Pinpoints exactly where in the path traffic is dropping.

```bash
mtr google.com                          # live traceroute
mtr --report -c 10 google.com          # 10 pings per hop, generate report
```

***

## `nslookup`

```bash
nslookup google.com                     # resolve external DNS
nslookup myservice.internal             # check internal service DNS
```

If it returns `NXDOMAIN` → DNS is not resolving. Check `/etc/resolv.conf` and nameserver config.

***

## `ip addr`

```bash
ip addr show                # all interfaces and IPs
```

Replaces `ifconfig` on modern Linux.
