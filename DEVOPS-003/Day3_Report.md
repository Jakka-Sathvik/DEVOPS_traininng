# Day 3 Report — DEVOPS-003: Linux Networking & Shell Scripting Fundamentals

**Sprint:** Sprint 9 (29/6 – 12/7)
**Ticket:** DEVOPS-003 — Linux Networking & Shell Scripting Fundamentals
**Date:** Day 3
**Environment:** Ubuntu 24.04 (Linux sandbox/VM)

---

## 1. Networking Concepts Learned

- **ICMP (ping):** Echo Request/Reply packets are used to test basic host reachability and measure round-trip time (RTT). A host can fail to respond either because it's genuinely down, or because a firewall/security policy is silently dropping ICMP traffic — the two look identical from the client side.
- **Traceroute / hop-by-hop path discovery:** `traceroute` reveals every router (hop) a packet passes through en route to a destination, and the latency to each one. This is how you localize *where* in a network path a slowdown or block is happening, rather than just knowing the final destination is unreachable.
- **DNS resolution (nslookup / dig):** Domain names are resolved to IP addresses via a resolver (e.g. `8.8.8.8`). Large services publish multiple A/AAAA records for the same hostname (round-robin DNS) so traffic is load-balanced across many front-end servers. `dig` exposes low-level protocol detail (flags, TTL, record counts, query time) that `nslookup` summarizes away.
- **Application-layer connectivity (curl/wget) vs. network-layer connectivity (ping):** A host can be fully reachable at the network layer but still refuse a specific web request — for example a firewall/proxy blocking HTTP(S) to a specific domain while ICMP and DNS both work fine. Troubleshooting "the network is down" really means checking **three independent layers**: DNS → ICMP/routing → application (HTTP) — each can fail on its own.
- **SSH and remote administration:** SSH (Secure Shell) provides encrypted remote login and command execution. A server identifies itself to clients using host keys (RSA/ECDSA/ED25519), generated once at install time, which clients then verify against on every subsequent connection (this is what a `known_hosts` fingerprint warning is protecting against).
- **Service management (systemd vs. SysV):** Modern Ubuntu systems use `systemd` (`systemctl`) to manage background services (daemons). Some minimal/containerized environments don't run systemd as PID 1, and fall back to the legacy SysV `service` command instead — both ultimately control the same underlying daemon process.
- **Listening ports and sockets:** A running service isn't proof that it's reachable — confirming it is actually bound to and *listening* on its port (e.g. TCP/22 for SSH) is the real verification step, checked with a tool like `ss -tlnp`.

---

## 2. Commands Practiced

### Task 3 — Network Troubleshooting

| Command | Purpose |
|---|---|
| `ping -c 4 <host>` | Test basic reachability and measure round-trip latency |
| `traceroute <host>` | Show the hop-by-hop path and per-hop latency to a destination |
| `nslookup <host>` | Resolve a hostname to its IP address(es) via a DNS server |
| `dig <host>` | Detailed DNS query/answer inspection (flags, TTL, query time) |
| `curl -sI <url>` | Fetch HTTP response headers only, to check a web service without downloading the body |
| `curl -s -o /dev/null -w '%{time_total}' <url>` | Measure detailed request timing (DNS/connect/TLS/total) |
| `wget -S <url>` | Download a resource while printing the server's response headers |

### Task 4 — OpenSSH Install & Configure

| Command | Purpose |
|---|---|
| `apt update && apt install openssh-server` | Install the SSH daemon (`sshd`) and its dependencies |
| `systemctl enable ssh` | Register `sshd` to start automatically on boot |
| `systemctl start ssh` / `service ssh start` | Start the SSH daemon now |
| `systemctl status ssh` / `service ssh status` | Check whether the daemon is currently running |
| `ss -tlnp \| grep :22` | Confirm `sshd` is actually bound to and listening on TCP port 22 |
| `sshd -V` | Print the installed OpenSSH server version |
| `ssh <user>@<host>` | Open an interactive, encrypted remote shell session |
| `logout` | Cleanly end an SSH session from inside the remote shell |

---

## 3. Problems Faced

1. **`ping`/`traceroute`/`curl` to external domains (e.g. `google.com`) failed with 100% packet loss / HTTP 403.**
   *Cause:* The working environment's outbound network is filtered by an egress proxy that only allows a fixed list of domains (package registries like `github.com`, `pypi.org`, etc.). ICMP and HTTP(S) to any other host is blocked at the proxy.
   *Resolution:* Confirmed this was a policy block, not a real outage, by checking the `x-deny-reason: host_not_allowed` header on the curl response and by successfully running the same commands against an allow-listed domain (`pypi.org`) and `localhost`, both of which worked instantly. DNS (`nslookup`/`dig`) was unaffected since UDP/53 traffic is permitted.

2. **`systemctl status/start/enable ssh` failed with "System has not been booted with systemd as init system (PID 1). Can't operate."**
   *Cause:* The container/sandbox used for this exercise runs a lightweight init process instead of full `systemd` as PID 1, which is common in minimal containerized Linux environments.
   *Resolution:* Used the equivalent legacy SysV command, `service ssh start` / `service ssh status`, which controls the exact same `sshd` daemon. On a standard Ubuntu VM or physical machine (which does boot with `systemd`), the `systemctl` commands from the task sheet work exactly as written — this was purely an environment difference, not a configuration error.

3. **SSH login initially had no non-root test account to log in as.**
   *Resolution:* Created a dedicated local user (`devuser`) with a password, and enabled `PasswordAuthentication yes` in `/etc/ssh/sshd_config` (restarting `sshd` to apply it) so an interactive login could be tested end-to-end, in line with the standard best practice of not routinely SSH-ing in as root.

---

## 4. Summary of Learning

Day 3 focused on the fundamentals of diagnosing network connectivity and enabling remote administration on Linux:

- Troubleshooting connectivity issues is a **layered process** — DNS resolution, network/ICMP reachability, and application-layer (HTTP) access are independent and must be checked separately, since a failure in one doesn't necessarily mean the others are broken.
- Command-line tools like `ping`, `traceroute`, `nslookup`, `dig`, `curl`, and `wget` each surface a different layer of that stack, and together give a complete picture of *where* a connection problem actually lives.
- Setting up SSH access involves more than installing the package: it requires starting and enabling the service, confirming it's actually listening on its port, and (for password-based access) explicitly enabling password authentication and creating an appropriate non-root account.
- Environment differences matter — a command failing (e.g. `systemctl` in a non-systemd container) doesn't always mean the underlying task failed; recognizing *why* a tool behaves differently in a given environment, and finding the equivalent working command, is itself a core troubleshooting skill.
- Verifying a service is genuinely working means checking multiple signals together (process running + port listening + a successful end-to-end client connection), not trusting a single "status: running" message in isolation.
