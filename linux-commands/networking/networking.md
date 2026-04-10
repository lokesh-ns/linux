# Networking

Commands to check open ports, test connectivity, capture traffic, and debug DNS.

---

## 25. `ss`

```bash
ss -tulnp       # TCP+UDP listening ports with process names
ss -s           # socket statistics summary
ss -anp         # all connections with PIDs
ss -tnp state ESTABLISHED   # active TCP connections only
```

**Flags:** `t`=TCP, `u`=UDP, `l`=listening, `n`=numeric ports, `p`=show process

Modern replacement for `netstat`. Faster and more detailed.

---

## 26. `netstat` (legacy)

```bash
netstat -tulnp      # listening ports with processes
netstat -rn         # routing table
```

Still found on older systems. Use `ss` on modern Linux.

---

## 27. `curl`

```bash
curl -i http://localhost                # include response headers
curl -s http://localhost/health         # silent output
curl -o /dev/null -w "%{http_code}" http://localhost   # print status code only
curl -v https://api.example.com         # verbose with TLS handshake details
curl -X POST -H "Content-Type: application/json" \
     -d '{"key":"val"}' http://api/endpoint
```

**HTTP status codes:** `200`=OK, `301/302`=redirect, `4xx`=client error, `5xx`=server error.

---

## 28. `wget`

```bash
wget http://localhost                   # download to disk
wget -q -O /dev/null http://localhost   # quiet health check
```

---

## 29. `mtr`

```bash
mtr google.com                          # live traceroute
mtr --report -c 10 google.com          # 10 pings per hop, generate report
```

Shows every network hop between you and the destination with live packet loss %. Pinpoints exactly where in the path traffic is dropping.

---

## 30. `tcpdump`

```bash
tcpdump -i eth0                          # capture all on eth0
tcpdump -i eth0 port 8080               # filter by port
tcpdump -i eth0 host 10.0.1.5          # filter by host
tcpdump -i eth0 -w capture.pcap        # save to file → open in Wireshark
tcpdump -i eth0 -c 20 port 80          # capture 20 packets then stop
tcpdump -i eth0 -nn                     # no DNS/port resolution (faster)
```

**Homework:** capture a `tcpdump` and analyse with [Wireshark](https://www.wireshark.org/) (free, open-source). Essential for production debugging.

---

## 31. `nmap`

```bash
nmap localhost                          # open ports on localhost
nmap -sV 10.0.1.5                      # detect service versions
nmap -p 1-65535 10.0.1.5              # scan all ports
```

Use for security audits and firewall verification — confirm which ports are actually reachable.

---

## 32. `nslookup`

```bash
nslookup google.com                     # resolve external DNS
nslookup myservice.internal             # check internal service DNS
```

If it returns `NXDOMAIN` → DNS is not resolving. Check `/etc/resolv.conf` and nameserver config.

---

## 33. `ip addr`

```bash
ip addr show                # all interfaces and IPs
ip addr show eth0           # specific interface
ip route show               # routing table
ip route get <target-IP>    # which route would be taken for this IP
```

Replaces `ifconfig` on modern Linux.
