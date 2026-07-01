# FAANG-Level OS & Networking — Progressive Depth Question Bank

Each topic escalates through 4 levels:
- **L1 (Baseline)** — junior-level recall, "what is X"
- **L2 (Applied)** — mid-level, "how do you use/check X"
- **L3 (Internals)** — senior-level, "how does X actually work under the hood"
- **L4 (Systems/Edge-case)** — staff/principal-level, "break it, scale it, or find the subtle failure mode"

Use these as follow-up chains in an interview: ask L1, and only go to L2/L3/L4 based on how confidently the candidate answers. Expected-answer notes are in *italics* under each question.

---

## 1. CPU, Load Average & Scheduling

**L1:** What does the `top` command show you?
*User/sys/idle/iowait CPU%, load average, memory, per-process CPU/mem.*

**L2:** Load average is high but CPU is idle — what's happening, and how do you confirm?
*Load average includes `D`-state (uninterruptible sleep) processes waiting on IO, not just runnable (`R`) ones. Confirm via `ps -eo state,pid,comm | grep ' D'` and `iostat -x 1`.*

**L3:** Explain exactly how Linux calculates load average, and why a load of "4" means something different on a 2-core vs 8-core machine.
*Load average is an exponentially-damped moving average of the run-queue length (nr_running + nr_uninterruptible), sampled every 5 seconds, over 1/5/15-minute windows. It's not normalized to core count — you must divide by `nproc` to get per-core utilization. A load of 4 on 2 cores = severely oversubscribed (200%); on 8 cores = comfortably under 50%.*

**L4:** You have a load average of 200 on a 32-core box, but `top` shows almost no CPU usage and no obvious IO-heavy processes in `D` state. What are the possible causes, and how do you find the culprit?
*Possible causes: (a) fork bomb / runaway process creation inflating `nr_running` transiently; (b) processes stuck on uninterruptible NFS/network-mount IO (classic "D-state on stale NFS handle" — doesn't always show as expected IO); (c) massive thread count from a single misbehaving process (check `ps -eLf | wc -l`); (d) a kernel scheduler/cgroup throttling issue (check `/sys/fs/cgroup/.../cpu.stat` for `nr_throttled`); (e) load average calculation bug/hung task detector — check `dmesg` for "hung task" warnings, which indicate real, sustained D-state blocking even if the specific process log spam is misleading. Diagnostic path: `ps -eo pid,stat,wchan:32,comm | grep D`, `dmesg -T | tail -100`, and `/proc/loadavg` combined with `/proc/stat`.*

---

## 2. Virtual Memory, Swap & OOM

**L1:** What's the difference between RAM and swap?
*RAM = physical volatile memory; swap = disk-backed overflow for inactive pages.*

**L2:** What does `vm.swappiness` control, and what's a sane value for a database server?
*0–100, controls kernel's preference for reclaiming page cache vs swapping anonymous pages. Default 60; DB servers often tuned to 1–10 to avoid swapping hot data.*

**L3:** Walk through exactly what happens when the kernel decides to reclaim memory — page cache eviction, `kswapd`, direct reclaim, and how swap fits in.
*Kernel maintains active/inactive LRU lists for anonymous and file-backed pages. `kswapd` runs asynchronously in the background reclaiming pages once free memory drops below the low watermark. If allocation can't wait, the allocating process itself performs **direct reclaim** (synchronous, adds latency — visible as `allocstall` in `/proc/vmstat`). Clean file-backed pages are dropped for free; dirty ones are written back; anonymous pages (heap/stack, not backed by a file) can only be reclaimed by writing to swap.*

**L4:** A latency-sensitive service shows periodic p99 latency spikes correlating with low but non-zero swap usage, even though `free -h` shows plenty of RAM available. Explain the likely mechanism and how you'd confirm/fix it.
*Likely cause: even small amounts of swapped-out pages that get touched again cause a page fault → synchronous disk read (major fault) — a few ms each, but enough to blow p99 latency on a hot path. Also possible: NUMA effects — if the process's memory was allocated on a remote NUMA node or reclaimed/swapped due to per-node pressure while aggregate free memory looks fine. Confirm via `sar -B` (majflt/s), `vmstat 1` (si/so columns), and `/proc/<pid>/status` (VmSwap). Fixes: set `vm.swappiness=1` or disable swap for latency-critical workloads, use `mlock()`/`mlockall()` to pin critical memory, check NUMA binding with `numactl --hardware` and `numastat`.*

---

## 3. OOM Killer

**L1:** What is the OOM killer?
*Kernel mechanism that kills a process to free memory when the system is critically low on memory.*

**L2:** How does the kernel decide which process to kill?
*`oom_score` per process (based on memory usage %), adjustable via `oom_score_adj` (-1000 to +1000); highest score gets killed.*

**L3:** Explain the full OOM invocation path — what triggers it, what `panic_on_oom` and `overcommit_memory` do, and where to find evidence after the fact.
*Triggered when a memory allocation request can't be satisfied even after reclaim/swap attempts. `vm.overcommit_memory` (0=heuristic, 1=always allow overcommit, 2=strict no-overcommit) determines whether allocations are even allowed to "succeed" optimistically before physical backing is needed — this is why a process can be OOM-killed for touching memory it "successfully" allocated earlier. `vm.panic_on_oom` (0/1/2) controls whether the kernel panics instead of invoking the killer — used in HA clusters where a fast, deterministic crash+failover is preferable to an unpredictable process kill. Evidence: `dmesg | grep -i "killed process"`, `/var/log/messages`, `journalctl -k`.*

**L4:** In a Kubernetes cluster, a pod is being OOM-killed even though its container's memory usage (per `docker stats`/cgroup) is well under its configured limit. What are the possible causes?
*(a) cgroup memory limit includes page cache attributed to the container — under memory pressure at the node level, kernel accounting can attribute reclaimable cache against the limit; (b) the container is hitting the **node's** memory pressure (system OOM), not its own cgroup limit — check `kubectl describe node` for `MemoryPressure` condition and node-level `dmesg`; (c) sidecar/init container memory not accounted in the app's own limit but sharing the pod's cgroup; (d) a mismatch between `requests` and `limits` causing overcommit at the node level — many pods with low requests/high limits scheduled onto the same node, and aggregate usage exceeds node capacity even though no single pod exceeds its own limit; (e) kernel-level slab/cgroup memory accounting lag. Diagnosis: `kubectl top pod`, cgroup v2 `memory.stat` (check `pgmajfault`, `oom_kill` counter directly in `memory.events`), node-level `dmesg`.*

---

## 4. Disk I/O

**L1:** How do you check disk usage?
*`df -h` (filesystem), `du -sh` (directory).*

**L2:** How do you check IOPS/throughput per disk, and what do `%util` and `await` mean in `iostat -x`?
*`%util` = percentage of time device had an I/O request outstanding (near 100% = saturated); `await` = average time (ms) an I/O request waits (queue + service time).*

**L3:** `%util` is at 100% but throughput (`r/s`+`w/s`) looks low — is the disk actually saturated? Explain why this can be misleading, especially on SSD/NVMe.
*On a single-spindle HDD, 100% `%util` reliably means saturation since it can only serve one request at a time. On SSDs/NVMe with internal parallelism (multiple channels/queues), 100% `%util` just means "busy at least some of the time" — it doesn't capture actual queue depth or whether the device could handle more concurrent requests. Better metrics for modern storage: `avgqu-sz`/queue depth, and comparing against the device's rated IOPS/queue depth (e.g., NVMe supports 64K queues). Use `fio` for controlled benchmarking rather than trusting `%util` alone on NVMe.*

**L4:** A database on EBS shows periodic latency spikes exactly every hour, though `iostat` looks normal most of the time. What EBS-specific and Linux-specific factors would you investigate?
*EBS-specific: burst credit exhaustion for gp2 volumes (burst bucket depletes, falls back to baseline IOPS = 3 IOPS/GB) — check CloudWatch `BurstBalance`; gp3 doesn't have this issue since IOPS/throughput are provisioned independently of size. Also check if it's a `io1`/`io2` volume hitting a co-located "noisy neighbor" issue on shared EBS infrastructure (rare but real), or EBS-optimized instance bandwidth cap being hit (check instance's dedicated EBS bandwidth limit vs actual traffic). Linux-specific: cron jobs firing hourly (logrotate, backup scripts) creating a write burst; filesystem journal checkpoint intervals; `dirty_ratio`/`dirty_background_ratio` causing a flush storm once dirty page cache crosses the threshold — check `/proc/sys/vm/dirty_ratio` and correlate flush timing with `sar -r` dirty page stats.*

---

## 5. DNS Resolution

**L1:** What happens when you type a URL into a browser? (high level)
*DNS resolution → TCP handshake → (TLS handshake) → HTTP request/response.*

**L2:** Walk through the full DNS resolution chain, and what `/etc/hosts` vs `/etc/resolv.conf` vs `nsswitch.conf` each do.
*Browser cache → OS resolver cache → `/etc/hosts` (checked per `nsswitch.conf` order, usually before DNS) → `/etc/resolv.conf` (configured DNS resolver) → recursive resolver → root → TLD → authoritative server → A/AAAA record returned.*

**L3:** Explain the difference between a recursive resolver and an authoritative server, what a "glue record" is, and why DNSSEC exists.
*Recursive resolver does the legwork of walking root→TLD→authoritative on behalf of the client and caches results per TTL. Authoritative server holds the actual zone records for a domain and gives a definitive answer for its zone. Glue records: when a nameserver for a domain is itself a subdomain of that domain (e.g., `ns1.example.com` is authoritative for `example.com`), you get a circular dependency — the parent zone (TLD) includes the NS's IP directly ("glue") alongside the NS record so the resolver doesn't need to resolve `ns1.example.com` to find `ns1.example.com`. DNSSEC adds cryptographic signing (RRSIG records, chain of trust from root) to prevent cache poisoning/spoofing, since plain DNS over UDP has no authentication.*

**L4:** Your service's DNS TTL is set to 300s, but during a failover you observe some clients taking 20+ minutes to pick up the new IP. What are the possible causes?
*(a) Resolvers/ISPs ignoring TTL and caching longer than specified (common with some corporate/ISP resolvers); (b) application-level DNS caching in the JVM (default infinite cache unless `networkaddress.cache.ttl` is set) or other language runtimes caching resolved IPs beyond the OS/DNS TTL; (c) client-side connection pooling — existing long-lived TCP connections don't re-resolve DNS at all until the connection drops, regardless of TTL; (d) negative caching TTL (SOA minimum) being much longer if it was a NXDOMAIN→valid transition; (e) `systemd-resolved` or local caching resolver (`dnsmasq`) not honoring TTL correctly. Mitigation: use a load balancer/VIP with a stable IP instead of DNS-based failover for anything requiring sub-minute failover, or explicitly tune application-level DNS cache TTLs.*

---

## 6. TCP/IP & Connection Troubleshooting

**L1:** What is a TCP three-way handshake?
*SYN → SYN-ACK → ACK.*

**L2:** Given host A can't reach host B, what's your troubleshooting order?
*`ping` → `traceroute`/`mtr` → check SG/NACL/firewall (both ends) → `nc -zv`/`telnet` on the specific port → `curl -v` for app-layer → `tcpdump` for packet capture.*

**L3:** Explain the states a TCP connection goes through, and what a large number of connections stuck in `TIME_WAIT` or `CLOSE_WAIT` tells you.
*States: `LISTEN → SYN_SENT/SYN_RECV → ESTABLISHED → FIN_WAIT1/2 → TIME_WAIT → CLOSED` (active closer side), or `CLOSE_WAIT → LAST_ACK` (passive closer side). Excess `TIME_WAIT`: high connection churn (short-lived connections, e.g., no keep-alive/connection pooling) — usually harmless unless it exhausts the ephemeral port range, tunable via `net.ipv4.tcp_tw_reuse` and increasing the ephemeral port range. Excess `CLOSE_WAIT`: the **application** isn't calling `close()` after the peer closed its side — this is an application-level file-descriptor/socket leak bug, not a network issue, and will eventually exhaust FDs.*

**L4:** Two microservices in the same VPC intermittently fail to connect, but `ping` and `traceroute` both succeed consistently. `tcpdump` on the server shows the SYN arriving but no SYN-ACK is sent. What would you investigate?
*(a) Server-side `SYN backlog` full — `net.core.somaxconn` and application's `listen()` backlog too small for the connection rate; check `netstat -s | grep -i "listen"` for overflow counters, or `ss -lnt` for `Recv-Q` exceeding backlog; (b) `net.ipv4.tcp_syncookies` behavior under SYN flood-like load masking the real issue; (c) conntrack table exhaustion on a stateful firewall/NAT gateway in between (`nf_conntrack: table full`) — check `dmesg` and `/proc/sys/net/netfilter/nf_conntrack_count` vs `nf_conntrack_max`; (d) security group / iptables rule silently dropping (not rejecting) the SYN-ACK on the return path due to asymmetric routing; (e) application thread pool exhaustion — process accepted the connection at the kernel level but is too busy to call `accept()`, causing the backlog to fill. This is a classic "ping/traceroute lie because ICMP ≠ TCP" trap — L3 reachability doesn't guarantee L4 service health.*

---

## 7. Firewalls, Security Groups & NACLs (AWS-specific)

**L1:** What's the difference between a Security Group and a NACL?
*SG = stateful, instance-level; NACL = stateless, subnet-level.*

**L2:** Given SG allows the traffic but the connection still fails, what else could block it?
*NACL inbound/outbound rules (both directions needed, since stateless), host-based firewall (`iptables`/`firewalld`), route table missing a route, or the destination's own SG/NACL/host firewall (need to check the return path too).*

**L3:** Why does a stateless NACL require explicit outbound rules for a response, while a stateful SG doesn't — and how could a misconfigured NACL cause a connection to work in one direction but fail in the other (e.g., SSH connects but hangs)?
*SG automatically allows return traffic for an established connection because it tracks connection state; NACL doesn't track state, so it evaluates every packet against its rule list independently in each direction. A common failure: NACL inbound allows port 22, but outbound rules only allow port 22 (not ephemeral ports 1024-65535) — the SYN gets in, but the server's response (from an ephemeral source port back to the client) gets blocked, so the TCP handshake completes at L3 visibility (SYN seen) but never fully establishes, leading to "SSH hangs after connecting."*

**L4:** You need to add a NACL rule to block a single malicious IP mid-incident without disrupting existing traffic. Walk through the risk considerations (rule ordering/evaluation, existing established connections, and blast radius).
*NACL rules are evaluated in numerical order, first match wins — so a DENY rule for the malicious IP must have a lower rule number than any broader ALLOW rule that would otherwise match it first. Because NACLs are stateless, adding a DENY doesn't retroactively kill already-established connections at the OS/kernel level of the instances themselves (NACL just stops new packets matching that IP from getting through — but note: unlike SGs, this can partially/fully disrupt an in-progress connection at the next packet since there's no state tracking to "grandfather" it in). Blast radius: NACL applies at the subnet level, affecting every resource in that subnet, not just the target service — a broad block could be safer/faster applied at the SG or host `iptables` level (`DROP`/`REJECT` for the specific IP) if you want to scope the blast radius tighter and preserve statefulness for legitimate existing sessions.*

---

## 8. TLS/SSL

**L1:** What ports does HTTP/HTTPS use?
*80 / 443.*

**L2:** What causes a browser to show "insecure connection"?
*Expired cert, hostname/SAN mismatch, untrusted/self-signed CA, missing intermediate cert, deprecated TLS version, mixed content.*

**L3:** Walk through the full TLS handshake (1.2 vs 1.3 differences) and explain what asymmetric vs symmetric crypto is used for at each stage.
*TLS 1.2: ClientHello → ServerHello + certificate + ServerKeyExchange → ClientKeyExchange → Finished (2 round trips before app data). Asymmetric crypto (RSA/ECDHE) is used only to securely establish a shared symmetric session key (since asymmetric is too slow for bulk data); actual data is then encrypted with a fast symmetric cipher (AES-GCM). TLS 1.3 reduces this to 1 round trip (combines key exchange into the initial hello using ephemeral Diffie-Hellman by default, removes weak ciphers/RSA key exchange, and enables 0-RTT resumption for repeat connections at the cost of some replay-attack considerations for 0-RTT data.*

**L4:** A service behind an ALB terminates TLS at the load balancer, but internal traffic to backend instances is also required to be encrypted (mTLS) for compliance. Design considerations and failure modes to watch for?
*Design: ALB terminates client TLS, then re-encrypts to backend using a separate cert (backend target group HTTPS listener) — or pass through TLS entirely to the instance (NLB, since ALB can't do mTLS passthrough to clients easily) if client cert validation is required end-to-end. For mTLS specifically: certificate rotation/expiry now needs to be managed on both ends (client cert too, not just server cert) — a common outage cause is backend service cert rotation not being coordinated with the ALB's trust store update, or clock skew on instances causing premature cert-invalid errors. Also watch for: increased latency/CPU from double TLS termination (ALB + backend), and cert chain validation failures if the backend uses an internal CA that isn't in the ALB's trusted CA bundle.*

---

## 9. Load Balancing (ALB/NLB/ELB)

**L1:** ALB vs NLB — what's the difference?
*ALB = L7 (HTTP/HTTPS aware, content-based routing); NLB = L4 (raw TCP/UDP, static IP, ultra-low latency).*

**L2:** Why would you choose NLB over ALB for a database or gRPC workload?
*Database wire protocols and often gRPC (HTTP/2 with long-lived streams) aren't well-served by L7 HTTP-aware routing; NLB passes TCP through with minimal overhead/latency and preserves source IP.*

**L3:** Explain how ALB/NLB perform health checks differently, and a scenario where a target could be marked "healthy" by the LB but still be functionally broken for real traffic.
*ALB health checks are HTTP-based (configurable path/status code expected). NLB health checks (in TCP mode) just check port connectivity, not application health, unless you configure an HTTP health check via a target group. Failure scenario: an app is listening on the port (TCP handshake succeeds, NLB marks healthy) but is deadlocked/hung and never responds to actual requests, or a specific downstream dependency (DB connection pool) is exhausted — the shallow health check passes while real traffic times out. This is why health checks should hit an actual app-level `/health` endpoint that validates critical dependencies, not just "process is listening."*

**L4:** During a deploy, ALB target group health checks pass immediately after new instances register, but a spike of 5xx errors occurs for ~30 seconds after each deploy. Diagnose the likely causes and the fix.
*Likely cause: connection draining/deregistration delay misconfigured — old targets are deregistered and stop receiving new connections, but in-flight requests aren't given enough time to complete (`deregistration_delay` too short), causing dropped connections. Or the opposite: new targets pass the health check (port open) but the application hasn't fully warmed up (JIT warmup, connection pool not yet established, cache cold) — traffic gets routed before the app can actually serve requests correctly. Fixes: (a) increase health check interval/threshold or add a readiness-gate that only reports healthy after warmup completes; (b) enable/tune `deregistration_delay` to allow graceful connection draining; (c) use ALB's "slow start" mode to ramp traffic gradually to newly healthy targets instead of sending full traffic immediately.*

---

## 10. Filesystems & Storage Internals

**L1:** How do you check if a disk is running out of space?
*`df -h`.*

**L2:** `df -h` shows plenty of free space but you still get "No space left on device" — why?
*Inode exhaustion — check `df -i`.*

**L3:** Explain the difference between a hard link and a symbolic link, and why deleting a file doesn't always free disk space immediately.
*Hard link: another directory entry pointing to the same inode (same data blocks) — file data is only freed when the link count (`nlink`) reaches 0. Symlink: a separate file containing a path string, pointing to another path (can cross filesystems, can dangle). "Deleted but space not freed": a process still has the file open (file descriptor holds a reference even after `unlink()` removes the directory entry) — the actual blocks aren't freed until the last open FD is closed too. Diagnose with `lsof | grep deleted`.*

**L4:** An application logs heavily to a file that was deleted (via `rm`) while still being written to, but disk usage isn't dropping and `df -h` still shows the space consumed even though `ls` shows the file gone. How do you reclaim the space without restarting the application?
*Find the deleted-but-open file: `lsof | grep deleted` (or `ls -l /proc/<pid>/fd/` for a known PID) — it'll show the inode with size still growing. To reclaim space without killing the process: truncate the underlying open file descriptor directly via `> /proc/<pid>/fd/<fd_number>` (this works because the process still holds an open FD referencing the inode; truncating it frees blocks immediately while the process can continue writing from its current file offset). Longer-term fix: use `logrotate` with `copytruncate`, or ensure the app reopens its log file (`SIGHUP`-triggered reopen) instead of relying on external `rm`.*

---

## 11. Process/Container Isolation (cgroups & namespaces)

**L1:** What's the difference between a container and a VM?
*Container shares the host kernel (isolated via namespaces/cgroups); VM virtualizes hardware and runs a full separate kernel.*

**L2:** What are cgroups used for?
*Resource limiting/accounting (CPU, memory, IO, PIDs) per process group.*

**L3:** Explain what Linux namespaces provide isolation for, and what's NOT isolated by default (a common container security gotcha).
*Namespaces: PID (process tree), mount (filesystem view), network (interfaces/routing), UTS (hostname), IPC, user (UID/GID mapping), cgroup. NOT isolated by default unless explicitly configured: the kernel itself (shared — a kernel exploit affects all containers on the host), time namespace (older kernels), and critically, without a user namespace, root inside a container maps to root on the host (UID 0 = UID 0) — a container escape gives host root. This is why rootless containers / user namespace remapping matters for security-sensitive environments.*

**L4:** A container is being OOM-killed even though the pod's memory limit looks generous, and `cgroup` memory usage reported by Kubernetes seems inconsistent with what the app reports internally (e.g., JVM heap). Explain the discrepancy and how to resolve it.
*JVM (and similar runtimes) by default detect "available memory" from the host's total, not the cgroup limit, unless container-awareness is enabled (`-XX:+UseContainerSupport`, on by default in modern JVMs, but older versions or misconfigured JVM flags can miscalculate heap sizing far beyond the actual cgroup limit). Also: cgroup memory accounting includes page cache/buffers used by the process (e.g., memory-mapped files), which count against the cgroup limit but aren't part of the JVM's heap — so total cgroup usage (heap + off-heap + page cache + native libraries) can exceed what `-Xmx` alone would suggest, causing OOM kills that look surprising from the app's internal metrics alone. Fix: explicitly set `-Xmx` well below the cgroup limit (leaving headroom for off-heap/native/metaspace/page cache), verify JVM version has proper cgroup v2 awareness, and monitor `memory.stat`/`memory.current` from the cgroup directly rather than trusting only app-level heap metrics.*

---

## How to Use This as an Interviewer

- Start every topic at **L1**. If answered fluently and fast, skip straight to **L3**.
- **L2→L3 transition** is where most "good but not senior" candidates plateau — they know commands but not mechanisms.
- **L4 questions have no single "correct" answer** — they're meant to see if the candidate can reason from first principles about failure modes they may not have seen before. Grade on reasoning process, not just the "right" answer.
- If a candidate says "I don't know, but here's how I'd investigate/find out," **that's a good answer at any level** — reward diagnostic instinct over memorized trivia.
