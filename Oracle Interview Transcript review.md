# SRE Interview Evaluation — Candidate: Nitu (Interviewer: Amit, OCI HPC Team)

---

## Q1: Day-to-day Linux performance troubleshooting — general approach

**Candidate Answer:**
Starts with `top` to check user CPU, system CPU, idle, IO wait, uptime. Then uses `ps aux --sort=-%cpu` to find the top resource-consuming process. For thread-level digging (mainly Java apps), uses `top -H -p <PID>` to get the thread ID. Then checks logs via `journalctl -u <service>`.

**Rating: 7/10**

**Interviewer Expectation:**
Testing whether the candidate has a structured, repeatable RCA methodology (not just tool recall) — starting broad (system-level) and narrowing to process/thread/log level. Mid-to-senior SRE standard expects mention of a "recovery-first, root-cause-second" mindset (which the candidate did state) plus correlation across CPU/memory/disk/network before diving into one dimension.

**Gaps:**
- Didn't mention checking `vmstat`, `dstat`, or `sar` for historical trend correlation.
- Didn't mention `htop` as a more readable alternative, or `perf top` for kernel-level profiling.
- No mention of checking recent deployments/changes (change correlation) before deep-diving into commands — a senior engineer usually asks "what changed?" first.
- Jumped to "thread analysis" assuming Java without first establishing that CPU (not something else) was the actual bottleneck.

**Ideal Answer (Senior-Level):**
"My first priority is service recovery, not RCA — so I check if a quick mitigation (restart, failover, scale-out) is viable while I investigate in parallel. For investigation: I start with `top`/`htop` for a live snapshot (user/sys/idle/iowait, load average), then `vmstat 1` and `sar` for historical trend to see if this is a spike or gradual degradation. I check `ps aux --sort=-%cpu` and `-%mem` to isolate the offending process, then drill into threads with `top -H -p <PID>` or `jstack`/`py-spy` for language-specific stack traces. I also check `journalctl -u <service> --since` and correlate with recent deployments via `git log`/CI-CD history, since most incidents follow a change."

**Key Learning:**
Always state "stabilize first, root-cause second" explicitly, and always ask "what changed recently?" — this is the #1 thing senior SREs check before touching commands.

---

## Q2: CPU is idle but load average is high — what do you check, and what's the difference between CPU utilization and load average?

**Candidate Answer:**
Explained that load average reflects processes waiting in the run queue, including those in uninterruptible sleep waiting on IO — so high load average with low CPU doesn't mean CPU is the bottleneck; it can mean IO is the bottleneck.

**Rating: 8/10**

**Interviewer Expectation:**
This is a classic Linux internals test — distinguishing "CPU-bound" vs "IO-bound" load. Senior-level answer should explicitly mention `R` (runnable) vs `D` (uninterruptible sleep, usually disk/network IO) process states from `ps`/`top`, and ideally mention that Linux load average includes D-state processes (unlike some other UNIX systems).

**Gaps:**
- Didn't mention process states (`R`, `D`, `Z`) explicitly — this is the core Linux internals concept being tested.
- Didn't mention `ps -eo state,pid,comm | grep D` as the direct way to find IO-blocked processes.

**Ideal Answer (Senior-Level):**
"CPU utilization tells you how busy the CPU cores currently are. Load average tells you the average number of processes that are either running or waiting for a resource (runnable, state `R`) over 1/5/15 minutes — and critically, on Linux, this also includes processes in uninterruptible sleep (state `D`), usually waiting on disk or network IO. So high load average with low CPU utilization usually means many processes are stuck in `D` state waiting on IO, not CPU. I'd confirm with `ps -eo state,pid,comm | grep ' D'` to list D-state processes, and cross-check with `iostat -x 1` for `%util` and `await` on the disk."

**Key Learning:**
Always tie load average back to process states (R/D/Z) — that's the internals-level answer interviewers are fishing for, not just "it could be IO."

---

## Q3: How do you dig deeper into the IO bottleneck / why are processes queued?

**Candidate Answer:**
Said the system needs disk space to process new requests; if disk isn't allowing writes or space is running low, load average rises. Recommended checking disk/space side.

**Rating: 5/10**

**Interviewer Expectation:**
Testing understanding of disk IO saturation vs disk space exhaustion — these are two very different problems, and the candidate conflated them.

**Gaps:**
- Confused "disk space full" with "disk IO saturated" — these are different failure modes. A disk can have plenty of free space and still be IO-saturated (high `%util`, high `await` in `iostat`).
- Should have introduced `iostat -x` metrics (`%util`, `await`, `svctm`, queue depth) at this stage rather than only at the next question.

**Ideal Answer (Senior-Level):**
"IO wait doesn't necessarily mean the disk is full — it usually means the disk is saturated with too many read/write operations relative to its throughput capacity. I'd run `iostat -x 1` and look at `%util` (near 100% = saturated), `await` (time requests wait, including queue time), and `avgqu-sz` (queue depth). Separately, I'd check `df -h` just to rule out an actual space-exhaustion scenario, since a full disk can also block writes and cause processes to hang in D-state."

**Key Learning:**
Disk *space* exhaustion and disk *IO* saturation are distinct root causes — don't conflate them; check both independently.

---

## Q4: How do you check disk usage?

**Candidate Answer:** `df -h` for filesystem-level, `du -sh *` for directory-level.

**Rating: 8/10**

**Interviewer Expectation:** Basic Linux command literacy — junior-level baseline question.

**Gaps:** Minor — could have mentioned `du -sh * | sort -rh` for finding largest offenders quickly, or `df -i` for inode exhaustion (a very common "disk full but df -h shows space" trap).

**Ideal Answer (Senior-Level):**
"`df -h` for filesystem-level space usage, `du -sh /path/* | sort -rh | head` to rank the biggest space consumers. I'd also check `df -i` for inode exhaustion — a filesystem can report free space in `df -h` but still refuse writes if it's run out of inodes, which is a common gotcha."

**Key Learning:** Always mention inode exhaustion (`df -i`) alongside `df -h` — it's a frequent real-world trap interviewers like to probe for.

---

## Q5: How do you check reads/writes per second on a disk?

**Candidate Answer:** `iostat` gives real-time read/write throughput and IO wait.

**Rating: 7/10**

**Interviewer Expectation:** Testing familiarity with `iostat -x` and its key columns.

**Gaps:** Didn't name specific flags/columns (`-x` for extended stats, `%util`, `await`, `r/s`, `w/s`).

**Ideal Answer (Senior-Level):**
"`iostat -x 1` — the `-x` flag gives extended stats including `r/s`/`w/s` (reads/writes per second), `%util` (device saturation), and `await` (average wait time per IO). I'd also use `iotop` to attribute IO usage to specific processes."

**Key Learning:** Name exact flags and output columns — "I use iostat" alone reads as surface-level; naming `-x`, `%util`, `await` signals hands-on depth.

---

## Q6: Other reasons for high disk read/write besides space saturation (includes disk type follow-up: SSD/HDD/NVMe)

**Candidate Answer:**
Mentioned application-side memory leaks, processes not freeing memory/disk, and (after prompting) acknowledged SSD vs HDD vs NVMe speed differences, and that application write patterns (batching vs many small writes) matter.

**Rating: 5/10**

**Interviewer Expectation:**
Testing broad, multi-angle root-cause thinking (application behavior, hardware type, filesystem/journaling overhead, swapping) rather than a single explanation. The interviewer had to lead the candidate to most of the good answers rather than the candidate volunteering them.

**Gaps:**
- Candidate assumed "SSD by default" without verifying — the interviewer explicitly called this out as a bad habit ("don't assume, verify").
- Didn't proactively mention swapping (memory pressure causing swap IO) as a major cause of disk IO spikes — this connects directly to the RAM/swap discussion later in the interview and should have been raised here.
- Didn't mention filesystem journaling overhead, database checkpoint/flush behavior, or backup/cron jobs running concurrently as common causes.

**Ideal Answer (Senior-Level):**
"Several possibilities: (1) Application write pattern — small, frequent writes are far more expensive than batched sequential writes; (2) Memory pressure causing swapping, which shows up as disk IO; (3) Underlying storage type — HDD (mechanical, slow random IO) vs SSD vs NVMe, which I'd verify with `lsblk -d -o name,rota` rather than assume; (4) Filesystem/journaling overhead (e.g., ext4 journal, or database WAL/redo log flushing); (5) Concurrent batch jobs like backups, log rotation, or cron tasks competing for IO. I'd correlate `iostat` spikes with `dmesg`, cron logs, and app logs to pin down the actual cause rather than guessing."

**Key Learning:** Never assume hardware/config details ("it's probably SSD") — verify with a command (`lsblk -d -o rota`) before reasoning further; this was explicitly flagged by the interviewer as a red flag.

---

## Q7: Walk through what happens when you type google.com in a browser (DNS resolution, dig/nslookup, /etc/hosts, resolv.conf)

**Candidate Answer:**
Explained DNS resolution happens before TCP three-way handshake — browser queries nearest DNS server for IP, then TCP handshake begins. Mentioned A records generically but couldn't fully explain the resolution chain (recursive/iterative lookup). Correctly identified `dig` and `nslookup` as tools to query DNS records when prompted. Correctly explained `/etc/hosts` (local hostname mappings) and `/etc/resolv.conf` (DNS server configuration) after being walked through it, and correctly explained that for local testing before DNS is configured, entries can be added to `/etc/hosts` to bypass DNS resolution.

**Rating: 5/10**

**Interviewer Expectation:**
A classic "walk me through the request lifecycle" systems-design question. Senior-level answer expects the full flow: browser cache → OS resolver cache → `/etc/hosts` → `/etc/resolv.conf`/DNS resolver → recursive resolver → root/TLD/authoritative DNS servers → IP returned → TCP three-way handshake → (TLS handshake if HTTPS) → HTTP request/response. The candidate had the right instinct but lost the thread multiple times, required heavy interviewer prompting, and admitted "I'm losing this part" twice.

**Gaps:**
- Could not articulate the recursive resolver → root → TLD → authoritative server chain.
- Needed prompting to arrive at `dig`/`nslookup` and the `/etc/hosts` local-override use case — these should be immediate recall for someone claiming Linux administration experience.
- No mention of DNS caching (OS-level, browser-level, resolver-level) or TTL.
- No mention of TLS handshake as a distinct step from TCP handshake when HTTPS is involved.

**Ideal Answer (Senior-Level):**
"First, the browser checks its own cache, then the OS resolver cache, then `/etc/hosts` for a static mapping. If unresolved, the OS consults `/etc/resolv.conf` to find the configured DNS resolver and sends a query. The resolver checks its cache; if not cached, it performs a recursive lookup: root server → TLD server (`.com`) → authoritative nameserver for `google.com`, which returns the A/AAAA record. Once the IP is known, the OS initiates a TCP three-way handshake (SYN, SYN-ACK, ACK) with the server on port 443. If HTTPS, a TLS handshake follows (certificate exchange, key negotiation) before the actual HTTP GET request is sent. To manually inspect any step, `dig google.com` or `nslookup google.com` shows DNS records directly; adding an entry in `/etc/hosts` lets you test a site locally before DNS is publicly configured, since `/etc/hosts` is checked before `/etc/resolv.conf`/DNS by default (per `nsswitch.conf`)."

**Key Learning:** The DNS resolution chain (cache → hosts → resolver → root/TLD/authoritative) is foundational networking knowledge for any SRE role — this needs to be fluent, not reconstructed under prompting.

---

## Q8: If ping to a server fails (no ICMP reply), does that mean the server is down?

**Candidate Answer:**
Said if ICMP itself isn't working, the server might be unhealthy or not accepting new requests, and referenced Kubernetes readiness/liveness probes as an analogous concept.

**Rating: 6/10**

**Interviewer Expectation:**
Testing whether the candidate knows ICMP can be blocked by firewalls/security groups independent of actual server health — a very common false-negative trap.

**Gaps:**
- Didn't explicitly state that ICMP is frequently blocked by firewalls/security groups/NACLs even when the server and application are perfectly healthy — this is the key insight being tested, and the candidate only implied it indirectly via the Kubernetes tangent.

**Ideal Answer (Senior-Level):**
"No — a failed ping is not conclusive proof the server is down. ICMP is commonly blocked by firewalls, security groups, or NACLs as a hardening measure, even when the host and application are fully healthy. I'd follow up with a port-level check like `nc -zv <ip> <port>` or `curl` on the actual service port, and check `traceroute`/`mtr` to see where packets are actually being dropped, before concluding the server itself is unreachable."

**Key Learning:** Never conclude "server is down" from ICMP failure alone — always distinguish "ICMP blocked" from "host/service actually down" using port-level checks.

---

## Q9: Host A cannot reach Host B — full troubleshooting methodology (ping, traceroute, curl, tcpdump, firewall/NACL/SG)

**Candidate Answer:**
Listed: `ping` (check reachability), `traceroute` (find where packet drops), `curl` (check HTTP response codes), `tcpdump` (packet capture), then after prompting added checking firewall (outbound on A), NACL (subnet level), and security groups.

**Rating: 7/10**

**Interviewer Expectation:**
A comprehensive network troubleshooting methodology question — the interviewer explicitly tested whether the candidate would jump to conclusions (assuming "it was working before, so it's not firewall") — a legitimate senior-level trap about not assuming root cause based on history.

**Gaps:**
- Initially assumed firewall wasn't the issue because "it was working previously" — the interviewer directly corrected this ("don't assume anything... someone might have done something wrong, or there's a maintenance window").
- Didn't organize the answer using the OSI-layer mental model (L1 cable/NIC → L2 ARP → L3 IP/routing/firewall → L4 TCP port → L7 application), which is the structured approach senior SREs use to avoid missing a layer.
- Only mentioned firewall/NACL/SG after being prompted, not proactively.

**Ideal Answer (Senior-Level):**
"I'd work through the OSI layers systematically rather than assuming based on history: (1) L1/L2 — check `ip a`/`ip link` on both hosts for interface state; (2) L3 — `ping`, then `traceroute`/`mtr` to localize where packets are dropped; (3) Security — check security groups and NACLs on both source and destination (NACLs are stateless, so both inbound and outbound rules matter; SGs are stateful), and host-based firewall (`iptables`/`firewalld`) on both ends; (4) L4 — `nc -zv` or `telnet` to the specific port to isolate TCP-level reachability; (5) L7 — `curl -v` to see the actual HTTP response/error code; (6) Packet capture — `tcpdump` on both ends if still unresolved. Critically, I wouldn't assume 'it worked before, so it's not firewall' — configuration drift, maintenance windows, or unrelated changes can break previously-working connectivity at any point."

**Key Learning:** Structure network troubleshooting by OSI layer, and never rule out a cause just because "it worked before" — always verify, don't assume.

---

## Q10: SSL/TLS — why does a browser show "insecure connection," and on which ports does SSL happen?

**Candidate Answer:**
Mentioned CA certificate trust issues, then (with prompting) added expired certificates, and referenced a "Triple S service" being out of date (unclear/incorrect terminology — likely meant SSL/TLS library or `openssl`). Correctly identified port 22 for SSH but was unsure of the SSL/TLS port; interviewer supplied 443 (HTTPS) and 80 (HTTP).

**Rating: 4/10**

**Interviewer Expectation:**
Fundamental web/networking knowledge expected at any SRE level — ports 443/80 should be immediate recall, not looked up mid-interview. The "Triple S service" answer indicates a shaky, possibly fabricated recollection under pressure rather than solid knowledge.

**Gaps:**
- Did not know port 443 for HTTPS — a baseline fact for any infrastructure engineer.
- "Triple S service" is not a standard, recognized term and reads as guessing/filling space rather than a grounded answer — candidates should say "I'm not sure" rather than offer unclear terminology.
- Missed other common causes: outdated TLS version/cipher suite, hostname mismatch (SAN/CN not matching the domain, e.g., missing `www.` in SAN), mixed content (HTTP resources on an HTTPS page), self-signed certificates, or intermediate certificate chain issues.

**Ideal Answer (Senior-Level):**
"Common causes of 'insecure connection': (1) expired certificate; (2) hostname mismatch — the certificate's CN/SAN doesn't match the domain being accessed (e.g., cert issued for `example.com` but site accessed as `www.example.com`); (3) self-signed or untrusted CA — browser doesn't trust the issuing authority; (4) missing intermediate certificate in the chain; (5) outdated/deprecated TLS version (e.g., TLS 1.0/1.1 deprecated by modern browsers); (6) mixed content — an HTTPS page loading HTTP resources. SSL/TLS operates over port 443 for HTTPS (port 80 is plain HTTP, often redirected to 443). I'd verify with `openssl s_client -connect host:443` or `curl -vI https://host` to inspect the actual certificate chain, expiry, and negotiated TLS version."

**Key Learning:** Port 443 (HTTPS) is baseline knowledge — never guess or invent terminology under pressure; saying "I don't recall, but I'd verify with `openssl s_client`" is far stronger than guessing a made-up service name.

---

## Q11: ALB vs NLB — which would you recommend for a database workload?

**Candidate Answer:**
Initially recommended ALB for "simple user request" workloads and NLB for "extensively heavy load." When pushed further on a database-specific scenario with heavy, continuous queries, correctly switched to recommending NLB.

**Rating: 6/10**

**Interviewer Expectation:**
Testing understanding of L4 vs L7 load balancing and matching the right tool to workload type — especially that ALB (L7, HTTP/HTTPS-aware) is not well-suited for raw TCP database connections, while NLB (L4, low-latency, high-throughput, static IP support) is the correct choice for databases.

**Gaps:**
- Initial framing ("ALB for simple requests, NLB for heavy load") is incorrect — the distinction isn't about load volume, it's about protocol/layer. ALB operates at L7 (HTTP/HTTPS, content-based routing); NLB operates at L4 (raw TCP/UDP, ultra-low latency, static IP). A database connection is fundamentally a TCP protocol, not HTTP, so NLB (or no load balancer at all, depending on the DB engine's own HA mechanism) is the correct choice regardless of load volume.
- Didn't mention that many databases (e.g., RDS Multi-AZ, Aurora) have native failover/endpoint mechanisms that often make an external load balancer unnecessary.

**Ideal Answer (Senior-Level):**
"For a database workload, I'd recommend NLB, not because of load volume, but because of protocol: ALB operates at Layer 7 and understands HTTP/HTTPS semantics (host/path-based routing, SSL termination), which doesn't apply to raw database protocols like MySQL/PostgreSQL wire protocol. NLB operates at Layer 4, passing TCP connections through with minimal latency and preserving client IP, which is what a database connection needs. That said, for managed databases like RDS or Aurora, I'd first check if the engine's built-in endpoint/failover mechanism (writer/reader endpoints, Multi-AZ failover) already covers the need before introducing an NLB."

**Key Learning:** ALB vs NLB choice is about protocol layer (L7 vs L4), not load volume — database traffic is TCP, so NLB is the default answer for load-balanced DB access.

---

## Q12: RAM vs Swap — overview, VM swappiness, and whether high swap usage is good or bad

**Candidate Answer:**
Explained RAM as physical memory and swap as disk-backed overflow space used when RAM is exhausted, moving "infrequently used" processes to swap to improve performance. Correctly said high swap usage (visible via `free -m`) is not good and can be an early warning sign of an impending OOM condition. Did not recall the term "vm.swappiness" until prompted.

**Rating: 6/10**

**Interviewer Expectation:**
Testing kernel-level memory management understanding — specifically the `vm.swappiness` tunable (0-100, controls kernel's tendency to swap vs reclaim page cache) and the mechanics of what actually gets swapped (inactive anonymous pages, not "processes").

**Gaps:**
- Said "infrequent processing requests" are sent to swap — imprecise; it's inactive **memory pages** (anonymous, not backed by a file) that get swapped, not "requests" or whole processes.
- Needed prompting to recall `vm.swappiness`, despite it being a very standard SRE/Linux tuning parameter (`sysctl vm.swappiness`, default 60, range 0-100).
- Didn't distinguish swap from page cache reclaim — the kernel actually prefers reclaiming clean page cache pages before swapping, and `vm.swappiness` tunes that preference.

**Ideal Answer (Senior-Level):**
"RAM is physical volatile memory for active data. Swap is disk-backed space the kernel uses to page out inactive anonymous memory pages (memory not backed by a file, like heap/stack data) when RAM is under pressure, freeing physical RAM for active processes. This is controlled by `vm.swappiness` (0-100, default 60 on most distros) — a lower value makes the kernel prefer reclaiming page cache over swapping, while a higher value makes it swap more readily. High swap utilization (`free -h`) is generally a bad sign for latency-sensitive workloads — swap IO is orders of magnitude slower than RAM, so heavy swapping usually indicates memory pressure and can be an early warning sign of an impending OOM kill or severe latency degradation. For databases and low-latency services, `vm.swappiness` is often tuned down to 1 or 10 to avoid this."

**Key Learning:** `vm.swappiness` should be immediate recall for any SRE — it's one of the most commonly tuned kernel parameters in production, especially for databases.

---

## Q13: What triggers OOM, and what does the system actually do when it enters OOM state (OOM killer mechanics)?

**Candidate Answer:**
Explained OOM is triggered when the system is heavily relying on swap, has exhausted buffer/cache, and has very low remaining RAM — describing it as the kernel trying to "preserve the system from complete failure." When pushed on what action the system actually takes, initially misunderstood the question (thought it was about a "blue screen"/crash), then correctly recovered to explain the OOM killer evaluates an "adjustment score" (oom_score) to decide which process to kill, avoiding critical processes.

**Rating: 6/10**

**Interviewer Expectation:**
Testing knowledge of the Linux OOM killer mechanism specifically — `oom_score`/`oom_score_adj`, and that the kernel actively selects and kills a process (not just logs an event) to reclaim memory, rather than crashing the whole system (unlike Windows' BSOD, which the candidate briefly and mistakenly gravitated toward).

**Gaps:**
- Initial confusion with a "blue screen" concept (Windows-specific, not applicable to Linux) shows some conceptual gap between OS models, though self-corrected.
- Didn't mention `dmesg`/`/var/log/messages` showing "Out of memory: Killed process" as the way to confirm an OOM kill occurred.
- Didn't mention `oom_score_adj` as the tunable admins use to protect critical processes (e.g., setting `-1000` to exempt a process from OOM killing).

**Ideal Answer (Senior-Level):**
"When available memory (RAM + swap) drops critically low and the kernel cannot reclaim enough via cache eviction or swapping, it invokes the OOM killer. The OOM killer calculates an `oom_score` for each process (based on memory usage, adjusted by `oom_score_adj`) and kills the process with the highest score — typically the largest memory consumer — to free up memory and keep the system alive, rather than letting the entire system crash. Admins can protect critical processes by setting `oom_score_adj` to a very low value (e.g., -1000, effectively exempting it). After an OOM kill, you'll see 'Out of memory: Killed process' entries in `dmesg` or `/var/log/messages`, which is the first place I'd check to confirm this happened and identify which process was killed."

**Key Learning:** The OOM killer actively terminates a specific process based on `oom_score` — it's not a passive "event," it's an active kernel intervention; know where to find evidence of it (`dmesg`).

---

## Q14: Recent RCA / production incident — walk through a real example

**Candidate Answer:**
Described an incident where a deployment caused SSH (port 22) to stop working for some users on a financial-critical application. The team initially investigated application logs, considered rolling back Kubernetes to an older version, and eventually found the root cause was a firewall change disabling port 22 at the node-group level.

**Rating: 6/10**

**Interviewer Expectation:**
Testing real incident-response experience — structured triage, correct prioritization (P1 handling), and clear articulation of root cause and resolution. Senior-level answers typically use a structured format (impact → detection → investigation → root cause → fix → prevention).

**Gaps:**
- Answer was somewhat disorganized/rambling rather than following a clear incident narrative structure.
- Didn't mention what the actual fix/remediation was (re-enabling the port 22 rule) or any follow-up prevention step (e.g., adding this to a pre-deployment checklist, change review process, or monitoring/alerting for SSH connectivity).
- No mention of communication/escalation process during the P1 (stakeholder updates, timeline).

**Ideal Answer (Senior-Level):**
"During a deployment last week, SSH access broke for a subset of users on a financial-critical application — we declared a P1 immediately given the criticality. Initial investigation focused on application logs since the deployment 'looked' successful, and we briefly considered rolling back the Kubernetes deployment. However, further investigation at the node-group/security-group level revealed that the deployment had inadvertently modified the firewall rules, closing port 22. We restored the rule, verified SSH access across affected nodes, and closed the incident. As a follow-up, we added a pre-deployment validation step to catch firewall/security-group drift before it reaches production, and added connectivity monitoring for critical ports."

**Key Learning:** Structure incident narratives clearly — impact, investigation, root cause, fix, and (critically) the prevention/follow-up action; interviewers weight the "what did you change afterward" part heavily.

---

## Q15: Shell scripting — check/update package versions across 50 servers

**Candidate Answer:**
Described writing a list of IPs to a file, opening it with `with open()` in Python, iterating with a `for` loop, and running the update command per host. Also mentioned prior experience with plain bash loops for backups, monitoring, and log cleanup, and noted this is typically done via Ansible in practice.

**Rating: 6/10**

**Interviewer Expectation:**
A basic scripting/automation literacy check — the interviewer just wanted the core idea (loop over a host list and execute remotely), which the candidate did eventually convey, though the explanation was somewhat roundabout and mixed Python/bash without being asked.

**Gaps:**
- Didn't mention the actual remote-execution mechanism (`ssh`, `pssh`/`parallel-ssh`, or Ansible ad-hoc `ansible all -m shell -a "yum update -y"`), which is the missing piece — looping is trivial, but *how* the command reaches 50 remote hosts is the real crux of the question.
- No mention of parallelism (sequential SSH to 50 hosts is slow; tools like `pssh`, `ansible -f`, or `xargs -P` enable concurrency) or error handling/logging per host.

**Ideal Answer (Senior-Level):**
"Simplest approach: a bash loop reading a hosts file and running the command over SSH: `while read host; do ssh $host 'sudo yum update -y' ; done < hosts.txt`. For efficiency at 50+ hosts, I'd parallelize using `pssh -h hosts.txt -i 'sudo yum update -y'` or GNU `parallel`, and log per-host output/exit codes to a file for auditing. In practice, though, I'd use Ansible ad-hoc: `ansible all -i hosts.txt -m yum -a 'name=* state=latest' -b`, which handles parallelism, idempotency, and error reporting out of the box — this is the standard tool for this exact use case."

**Key Learning:** For "run X across N servers," the key answer is the remote-execution + parallelism mechanism (SSH loop / pssh / Ansible), not just "I'd use a for loop" — the loop is the trivial part.

---

## Q16 (Behavioral): Working with a difficult, uncollaborative teammate

**Candidate Answer:**
Shared a real example of joining a team and needing knowledge transfer from a senior colleague who was reluctant to share documentation for certificate management across 2,200+ servers. Handled it by being polite, flexible around his schedule, asking for concise 10-minute check-ins or existing documentation/runbooks rather than demanding hand-holding, and framing the ask around risk to him (if she failed at the task, it would escalate back to him).

**Rating: 8/10**

**Interviewer Expectation:**
Standard behavioral/collaboration question testing emotional intelligence, professionalism, and self-sufficiency when a colleague is uncooperative. Senior-level answers are expected to show initiative, empathy for the other person's constraints, and a concrete resolution — which the candidate delivered well with a specific, credible example.

**Gaps:**
- Minor — could have closed the loop by mentioning how the relationship evolved afterward (did it improve? Did they collaborate well going forward?) to fully demonstrate long-term relationship management, not just the initial approach.

**Ideal Answer (Senior-Level):**
Largely matches what the candidate gave — well-structured STAR-style example with a clear, mature strategy (polite persistence, respecting the colleague's time, framing around shared risk/incentive rather than demanding). Adding a brief note on the eventual outcome/relationship trajectory would elevate it further.

**Key Learning:** This was a strong answer — concrete example, mature framing (aligning incentives rather than escalating or complaining), good tone. Keep using specific real examples like this rather than generic statements.

---

## Q17 (Behavioral): Teammate's mistake causes production downtime — how do you explain it to your manager?

**Candidate Answer:**
Stated she would not blame a specific individual to the manager, would focus on team accountability rather than pointing fingers, and would prioritize restoring the service first before discussing root cause/blame.

**Rating: 8/10**

**Interviewer Expectation:**
Testing blameless post-incident culture and ownership/leadership maturity — a core SRE value. Senior-level answers should reflect the "blameless postmortem" principle common in SRE culture (Google SRE book), focusing on systemic/process causes rather than individual blame.

**Gaps:**
- Didn't explicitly use or reference the "blameless postmortem" framing/terminology, which would strongly signal SRE-culture fluency to an interviewer, even though the underlying behavior described was correct.
- Could have added that she'd still give the manager an honest, factual account of what happened (without naming/blaming), since transparency and no-blame aren't mutually exclusive.

**Ideal Answer (Senior-Level):**
"I'd follow a blameless postmortem approach — my priority is restoring service first, then giving my manager an honest, factual account of what happened and why, focused on the process/system gap (e.g., missing review step, insufficient testing) rather than pointing at an individual. Blame discourages people from surfacing mistakes early, which ultimately hurts reliability more than the original incident. I'd make sure the postmortem produces concrete action items to prevent recurrence, regardless of who was 'at fault.'"

**Key Learning:** Explicitly naming "blameless postmortem" as a principle (not just practicing it) signals stronger SRE-culture fluency to interviewers — know the term, not just the behavior.

---

## Q18: NIC not delivering expected bandwidth/throughput — hardware/network troubleshooting (interviewer-led)

**Candidate Answer:**
Candidate mostly listened and confirmed the interviewer's points (interface misconfiguration, faulty cable/NIC, outdated firmware/driver) rather than proactively generating the full list herself.

**Rating: 4/10**

**Interviewer Expectation:**
This is directly relevant to the OCI HPC/GPU role (NIC/RDMA/InfiniBand troubleshooting for GPU clusters) and expected a structured hardware-to-software escalation path: interface config → routing → IP assignment → physical layer (cable/NIC reseat) → firmware/driver — largely supplied by the interviewer rather than the candidate.

**Gaps:**
- Candidate didn't proactively drive this answer — mostly agreed with interviewer-supplied points ("yes," "true") rather than generating the systematic checklist herself.
- No mention of practical commands: `ethtool <iface>` for link speed/duplex/negotiation, `ip -s link show <iface>` for error/drop counters, `dmesg | grep <iface>` for driver/link events, or `lspci` to confirm the NIC is detected at the PCIe level.
- Given the target role, deeper concepts like RDMA/RoCE, NCCL, or InfiniBand-specific tools (`ibstat`, `ibping`) would strengthen this significantly — the candidate has no HPC networking hands-on experience, which is an honest and reasonable gap for this stage but worth flagging.

**Ideal Answer (Senior-Level):**
"I'd escalate from configuration to hardware: (1) confirm the interface is up and correctly configured — `ip a`, `ethtool <iface>` for negotiated speed/duplex (a common cause of 'low throughput' is a mismatched or auto-negotiated-down link); (2) check error/drop counters with `ip -s link show <iface>` or `ethtool -S <iface>` for CRC errors, indicating a physical-layer problem; (3) verify routing table and IP assignment are correct; (4) check `dmesg`/`journalctl` for link flap or driver errors; (5) if all software-level checks pass, physically reseat the NIC/cable, and if the issue persists, check firmware/driver version against vendor-recommended versions (`ethtool -i <iface>` shows driver/firmware version). For GPU/HPC-specific networking (RDMA/InfiniBand), I'd use `ibstat`/`ibping` to check link state and NCCL diagnostics for GPU-to-GPU throughput issues."

**Key Learning:** For hardware-layer troubleshooting, drive the answer proactively with specific commands (`ethtool`, `ip -s link`) rather than agreeing with interviewer-supplied points — and it's fine to honestly flag "I haven't worked with RDMA/InfiniBand yet, but here's how I'd approach it" for a role requiring HPC-specific hardware knowledge.

---

# Overall Summary

| # | Topic | Rating |
|---|-------|--------|
| 1 | General Linux perf troubleshooting | 7/10 |
| 2 | CPU idle vs high load average | 8/10 |
| 3 | IO bottleneck deep-dive | 5/10 |
| 4 | Disk usage commands | 8/10 |
| 5 | Disk read/write per second | 7/10 |
| 6 | Other causes of high disk IO | 5/10 |
| 7 | DNS/browser request lifecycle | 5/10 |
| 8 | Ping failure ≠ server down | 6/10 |
| 9 | Host-to-host unreachable troubleshooting | 7/10 |
| 10 | SSL/insecure connection causes | 4/10 |
| 11 | ALB vs NLB | 6/10 |
| 12 | RAM vs swap / vm.swappiness | 6/10 |
| 13 | OOM killer mechanics | 6/10 |
| 14 | Real incident RCA | 6/10 |
| 15 | Shell scripting across servers | 6/10 |
| 16 | Difficult teammate (behavioral) | 8/10 |
| 17 | Teammate's mistake / manager conversation (behavioral) | 8/10 |
| 18 | NIC throughput hardware troubleshooting | 4/10 |

**Average: ~6.2/10**

### Overall Pattern
- **Strengths:** Solid foundational Linux troubleshooting instincts (top, ps, iostat, df/du), correct behavioral/collaboration judgment (blameless, non-defensive, mature conflict handling), understands general AWS networking constructs (SG/NACL/ALB/NLB) at a working level.
- **Weaknesses:** Repeatedly needed interviewer prompting to reach the "expected" answer rather than volunteering it (DNS chain, vm.swappiness, port 443, NIC troubleshooting) — this pattern suggests knowledge exists but isn't fully internalized/fluent, which is a real risk under live production pressure. Occasional imprecise or fabricated-sounding terminology ("Triple S service") is a bigger red flag than not knowing the answer — admitting uncertainty is safer than guessing.
- **Fit for HPC/GPU-hardware-heavy role:** Currently weak on hardware/physical-layer and HPC-networking specifics (RDMA, InfiniBand, NIC firmware) — expected given her background, but a genuine gap for this specific team's core function, and worth being upfront about in follow-up rounds.

### Top 3 Coaching Priorities
1. **Stop guessing under pressure** — "I'm not 100% sure, but here's how I'd verify" beats inventing plausible-sounding but incorrect terms (e.g., "Triple S service").
2. **Internalize the DNS resolution chain and core networking constants (port 443/80, OSI-layer troubleshooting)** — these should be zero-latency recall for any SRE candidate.
3. **Proactively drive hardware-layer answers** with specific commands (`ethtool`, `ip -s link`, `lsblk -o rota`) rather than waiting for the interviewer to supply the checklist — this is the single biggest gap for this particular HPC-hardware-focused role.
