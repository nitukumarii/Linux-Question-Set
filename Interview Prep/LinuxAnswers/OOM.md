# 🧠 What is the OOM (Out of Memory) Killer?

## Definition

The **OOM (Out of Memory) Killer** is a mechanism built into the **Linux kernel** that is triggered when the 
system runs out of available memory (RAM and usable swap).

Its job is to **prevent the entire system from hanging or crashing** by selecting and terminating one or more processes to free memory.

---

# Why is OOM Killer Needed?

Every running process requires memory.

When:

- RAM is completely utilized.
- Swap is also exhausted (or unavailable).
- No more memory can be allocated.

The Linux kernel cannot satisfy new memory requests.

Instead of allowing the system to freeze, the kernel activates the **OOM Killer**.

---

# How Does OOM Killer Work?

```text
Processes keep consuming memory
            │
            ▼
RAM becomes full
            │
            ▼
Swap becomes full (or unavailable)
            │
            ▼
Kernel cannot allocate more memory
            │
            ▼
OOM Killer is triggered
            │
            ▼
Kernel calculates OOM score
            │
            ▼
Kills the process with the highest score
            │
            ▼
Memory is released
            │
            ▼
System continues running
```

---

# How Does Linux Decide Which Process to Kill?

The kernel assigns every process an **OOM Score**.

The score depends on factors such as:

- Memory usage
- Process priority
- Whether the process is critical to the system
- OOM adjustment values (`oom_score_adj`)

Generally,

- Processes consuming **large amounts of memory** receive higher scores.
- Critical system processes receive lower scores to avoid terminating essential services.

---

# Check OOM Score

### View a process's OOM score

```bash
cat /proc/<PID>/oom_score
```

Example:

```bash
cat /proc/2456/oom_score
```

---

### Check OOM adjustment value

```bash
cat /proc/<PID>/oom_score_adj
```

Possible values:

```text
-1000   Never kill this process
0       Default
1000    Highest chance of being killed
```

---

# How to Check if OOM Killer Was Triggered?

Check kernel logs:

```bash
dmesg | grep -i oom
```

or

```bash
journalctl -k | grep -i oom
```

Example output:

```text
Out of memory: Killed process 2345 (java)
```

---

# Example Production Scenario

Suppose a Java application has a memory leak.

```text
Java App
      │
Consumes 20 GB RAM
      │
RAM becomes full
      │
Swap becomes full
      │
OOM Killer activates
      │
Kills Java process
      │
Memory becomes available
      │
Other services continue running
```

Instead of crashing the entire server, Linux sacrifices the problematic process.

---

# Does OOM Killer Always Kill the Largest Process?

**Not necessarily.**

It kills the process with the **highest OOM score**, which considers:

- Memory usage
- Process importance
- `oom_score_adj`
- Other kernel heuristics

A large process may survive if it has been protected with a low `oom_score_adj`.

---

# How Can You Reduce the Chances of OOM?

- Increase RAM.
- Add or increase swap space.
- Fix memory leaks.
- Set appropriate memory limits (e.g., containers, JVM, applications).
- Tune `oom_score_adj` for critical processes.
- Continuously monitor memory usage.

---

# Useful Commands

### Check memory usage

```bash
free -h
```

### Top memory-consuming processes

```bash
ps aux --sort=-%mem | head
```

or

```bash
top
```

or

```bash
htop
```

### Check OOM events

```bash
dmesg | grep -i oom
```

---

# Interview Answer

**Q: What is the OOM Killer?**

**Answer:**

The Out of Memory (OOM) Killer is a Linux kernel mechanism that protects the system when it runs out of available memory. When both RAM and usable swap are exhausted and the kernel cannot allocate additional memory, it calculates an OOM score for running processes and terminates the process with the highest score to free memory. This prevents the entire system from becoming unresponsive. The process selected is usually a memory-intensive, non-critical process, although the decision also considers factors such as process priority and `oom_score_adj`.

---

# Quick Memory Trick

```text
RAM Full
     ↓
Swap Full
     ↓
Kernel can't allocate memory
     ↓
OOM Killer
     ↓
Calculates OOM Score
     ↓
Kills one process
     ↓
Memory freed
     ↓
System survives
```

---

# Common Interview Follow-up Questions

### Q1. Does OOM Killer kill the entire system?
**No.** It kills one or more processes to keep the operating system running.

### Q2. Can OOM Killer kill Docker or Kubernetes containers?
**Yes.** If a process inside a container has the highest OOM score, it can be terminated. In Kubernetes, the container is typically restarted based on the pod's restart policy.

### Q3. Can we prevent a process from being killed?
**Yes.** By setting a lower `oom_score_adj` value (for example, `-1000`), although this should be done carefully because protecting one process may increase the likelihood that another process will be killed.
