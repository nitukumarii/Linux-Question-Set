<img width="636" height="601" alt="image" src="https://github.com/user-attachments/assets/d74206aa-ae96-4c6b-84ca-42c02530c08f" />




# 🚀 FAANG SRE Advanced Question Bank (OS + Linux + Networking)
## Level 2 + Level 3 + Edge Cases + Production Scenarios

This repo contains deep-dive interview questions for FAANG SRE / Production Engineer / OCI + Cloud Infra roles.

---

# 🧠 OS (Operating Systems)

## 🔹 Level 2 (Deep Concepts)

- What happens internally from `fork()` to `exec()`?
- Explain user space vs kernel space with syscall flow
- How does context switching work internally (PCB, registers, scheduler queues)?
- What causes high load average but low CPU usage?
- What is thrashing in memory systems? How do you detect it in production?
- Difference between hardirq, softirq, and tasklets
- What happens when Linux OOM killer is triggered? How does it choose a process?

---

## 🔥 Level 3 (FAANG Depth + Real Debugging)

- A process is stuck in **D state forever** — how do you debug in production?
- CPU is 100% but no process appears to consume CPU — explain scenarios
- What is a fork bomb and how does Linux behave under it?
- Why does a process hang even when CPU and memory are normal?
- How does Linux handle memory fragmentation in long-running systems?
- What happens to a process if its parent dies? (orphan handling)
- What happens during kernel panic in distributed production systems?

---

## ⚠️ Edge Cases

- Disk is full but `df` shows space available → inode exhaustion
- `kill -9` not working → why?
- Zombie processes increasing continuously → root cause analysis
- High load average but low CPU usage → misinterpretation scenarios
- Process running but not responding → syscall blocking cases

---

# 🌐 NETWORKING (FAANG Critical)

## 🔹 Level 2 (Core Concepts)

- What happens step-by-step when you enter a URL in browser?
- TCP vs TLS handshake differences
- Why does TCP need a 3-way handshake?
- Latency vs bandwidth vs throughput differences
- How does DNS caching work (browser, OS, ISP)?
- What is MTU and what happens when packets exceed it?
- How does NAT work in cloud environments?

---

## 🔥 Level 3 (Real Production Scenarios)

- TCP connection established but request times out — possible failure points?
- Packet loss impact on Kafka / Redis / high-throughput systems
- Why does TCP slow down under intermittent packet loss?
- HTTP/1.1 vs HTTP/2 head-of-line blocking issues
- Service reachable via IP but not domain — deep debugging flow
- SYN flood attack — how systems detect and mitigate it?
- Congestion window collapse and retransmission behavior in TCP

---

## ⚠️ Edge Cases

- DNS resolves but service is unreachable → layered failure causes
- Ping works but curl fails → protocol-level debugging
- Intermittent 502/504 only under load
- TLS handshake fails only in production
- Cross-region API failure but local region works

---

# 🐧 LINUX + PRODUCTION DEBUGGING

## 🔹 Level 2

- How are file descriptors managed internally?
- What happens when `ulimit -n` is exceeded?
- How does `strace` help in debugging production issues?
- Hard link vs soft link differences in real systems
- What happens when `/var/log` disk fills up?

---

## 🔥 Level 3 (FAANG Debugging Depth)

- Production service slow but CPU, memory, network look normal — how do you debug?
- Why does restarting a service temporarily fix issues?
- How do you debug memory leaks in Kubernetes pods?
- Memory usage keeps increasing but logs show no errors — why?
- Container works in VM but fails in Kubernetes — root causes?
- Cascading failure in microservices — how do you stop propagation?

---

## ⚠️ Edge Cases

- Deleted file still consuming disk space (open file descriptor case)
- Container killed but port still shows “in use”
- Latency spikes during GC cycles only
- Works in load test but fails in production traffic patterns

---

# 🚨 PRODUCTION INCIDENT SCENARIOS (FAANG FINAL ROUND LEVEL)

## 🔥 Scenario 1
A service has **high latency only during peak traffic**, but CPU/memory are normal.

- What do you check first?
- How do you isolate DB vs service vs network?
- What metrics confirm root cause?

---

## 🔥 Scenario 2
Users report **intermittent 504 Gateway Timeout**.

- Why does it happen only intermittently?
- How do you correlate upstream/downstream logs?
- What role do retries play in amplifying this?

---

## 🔥 Scenario 3
A Kubernetes pod is **restarting randomly every few hours**.

- How do you debug crash loop?
- What logs/metrics matter?
- How do you differentiate OOM vs probe failure?

---

## 🔥 Scenario 4
DNS resolution is correct but service is unreachable from one region.

- Network vs routing vs firewall vs service issue?
- How do you isolate region-specific failure?

---

## 🔥 Scenario 5
CPU usage is normal but response time is increasing steadily.

- What hidden bottlenecks exist?
- Thread pool, GC, locks, IO waits analysis

---

## 🔥 Scenario 6
After deployment, error rate increases but rollback fixes it only partially.

- How do you determine partial rollback failure?
- What could persist after rollback?

---

# 🧠 FINAL NOTE (INTERVIEW REALITY)

## ✔ This covers:
- OS deep internals
- Linux production debugging
- Networking FAANG depth
- Edge cases
- Real incident thinking

## ⚠ But FAANG expects ALSO:
- System design failure analysis
- Communication under pressure
- Tradeoff reasoning
- Incident storytelling (STAR format)

---

# 🚀 If mastered, this = FAANG SRE ready level prep
