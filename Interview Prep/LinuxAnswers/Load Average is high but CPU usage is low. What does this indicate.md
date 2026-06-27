**Load Average is high but CPU usage is low. What does this indicate?**



**Load average represents:**

Processes running on CPU
Processes waiting for CPU (run queue)
Processes in uninterruptible sleep (I/O wait)


# 🧠 Load Average is high but CPU usage is low — What does it indicate?

## 🎯 Final Interview Answer (Combined + Correct Concept)

Load average represents the number of processes that are either **running on CPU or waiting for CPU or stuck in uninterruptible sleep (I/O wait state)**. It is not a direct measure of CPU utilization.

So when **load average is high but CPU usage is low**, it means that the system is not CPU-bound. Instead, processes are **waiting in the run queue or blocked on resources like I/O, locks, or network**, which makes the system slow even though CPU is not fully utilized.

For example, if a system has **4 CPU cores**, and **8 processes are ready to run**, only 4 can execute at a time while the remaining 4 wait in the queue. This increases load average. However, CPU usage may still look low if those running processes are not CPU-intensive or are frequently waiting on I/O.

---

# 📊 How to confirm this in production (Commands + What to look for)

## 1. Check Load Average
```bash
uptime
```

### Output:
```text
load average: 6.50, 5.80, 5.10
```

👉 Compare load with CPU cores:
```bash
nproc
```

If:
```
cores = 4
load = 6+
```

👉 System is overloaded in terms of demand

---

## 2. Check CPU usage
```bash
top
```

### Look for:
```text
%us = low
%id = high OR moderate
%wa = high OR normal
```

👉 Key observation:
- CPU is NOT fully busy
- But system still slow

---

## 3. Check run queue + system pressure
```bash
vmstat 1 5
```

### Output:
```text
r  b  swpd  free  buff  cache  us  sy  id  wa
5  2  0     ...   ...   ...    10  5   60  25
```

### What to check:

| Column | Meaning |
|--------|--------|
| **r** | Run queue (processes waiting for CPU) |
| **b** | Blocked processes (I/O wait) |
| **wa** | I/O wait percentage |

👉 If:
- `r > number of CPU cores` → CPU contention
- `b > 0` → I/O blocking issue
- `wa high` → disk/network bottleneck

---

## 4. Check CPU per core usage
```bash
mpstat -P ALL 1 5
```

### Look for:
- One core 100%? → CPU imbalance
- All low CPU but load high? → waiting issue

---

## 5. Check disk bottleneck (very important)
```bash
iostat -x 1 5
```

### Look for:
```text
%util ~ 100%
await high
```

👉 This confirms I/O bottleneck

---

## 6. Check process states
```bash
ps -eo state,pid,cmd | grep D
```

👉 If many processes in:
```
D state = uninterruptible sleep
```

✔ Means they are stuck waiting for disk/network

---

# 🚨 Final Interpretation

## Case: High Load + Low CPU

👉 Means:

- Processes are NOT actively computing
- They are WAITING

### Common real causes:
- Disk I/O latency
- Network latency (NFS, DB calls)
- Lock contention
- Too many processes waiting for CPU scheduling

---

# 🧠 Simple Mental Model

```text
Load = Demand (how many want CPU)
CPU = Supply (how much CPU is actually used)
```

---

# 🎯 Final Interview Answer (Short & Strong)

> When load average is high but CPU usage is low, it indicates that the system is not CPU-bound. Instead, processes are either waiting in the run queue or blocked on resources such as disk I/O, network latency, or locks. Load average includes both running and waiting processes, so even with low CPU utilization, the system can appear slow if processes are stuck waiting. I verify this using `vmstat` to check run queue and I/O wait, `iostat` for disk bottlenecks, and `ps` to identify processes in uninterruptible sleep state.

---

# 🧠 One-line Memory Trick

```text
High Load + Low CPU = WAITING problem (not CPU problem)
```
