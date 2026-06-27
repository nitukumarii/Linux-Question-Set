# 🧠 How do you troubleshoot high CPU utilization?

## 1. Confirm CPU utilization

Check overall CPU usage:

```bash
top
```

or

```bash
mpstat -P ALL 1 5
```

Look for:
- User CPU (`us`)
- System CPU (`sy`)
- Idle CPU (`id`)
- I/O wait (`wa`)

---

## 2. Identify CPU-intensive processes

```bash
ps aux --sort=-%cpu | head
```

or

```bash
top
```

Find the process consuming the most CPU.

---

## 3. Analyze the process

If it's a Java process:

```bash
top -H -p <PID>
```

This shows thread-level CPU usage.

Then collect a thread dump (Java):

```bash
jstack <PID>
```

For non-Java applications:

- Check application logs
- Check process behavior
- Verify recent deployments or configuration changes

---

## 4. Check system-wide bottlenecks

```bash
vmstat 1 5
```

Look for:
- High `r` (run queue)
- Low `id` (idle CPU)
- High `sy` (system CPU)

---

## 5. Check recent changes

- Recent deployment
- Configuration changes
- Traffic increase
- Scheduled jobs
- Infinite loops or application bugs

---

## 6. Take corrective action

Depending on the root cause:

- Optimize application code
- Fix infinite loops
- Tune application configuration
- Scale horizontally/vertically
- Restart the service if immediate recovery is required

---

# 🎯 Interview Answer

> To troubleshoot high CPU utilization, I first verify CPU usage using `top` or `mpstat`. Then I identify the top CPU-consuming process using `top` or `ps aux --sort=-%cpu`. If required, I perform deeper analysis—for example, using `top -H` and `jstack` for Java applications. I also use `vmstat` to check whether the issue is CPU saturation or another system bottleneck. Finally, I identify the root cause, such as increased workload, inefficient code, or recent configuration changes, and take appropriate corrective actions like optimization, scaling, or restarting the service if necessary.

---

# 🧠 What are common causes of CPU spikes?

Common causes include:

- Sudden increase in application traffic
- Infinite loops or inefficient code
- High number of requests
- Too many running processes or threads
- High system calls (kernel activity)
- Scheduled jobs (cron jobs)
- Database-intensive queries
- Malware or runaway processes
- Recent deployment or configuration changes

---

# 🧠 How do you identify CPU-intensive processes?

Use:

```bash
top
```

or

```bash
ps aux --sort=-%cpu | head
```

For thread-level CPU usage:

```bash
top -H -p <PID>
```

If it's a Java application:

```bash
jstack <PID>
```

to identify the thread causing high CPU usage.

---

# 🧠 Explain CPU Load vs CPU Utilization

| CPU Load | CPU Utilization |
|----------|-----------------|
| Number of processes waiting to run or currently running on the CPU | Percentage of CPU currently being used |
| Indicates demand for CPU | Indicates actual CPU usage |
| Measured as Load Average | Measured as %CPU |
| Can be high even if CPU isn't fully utilized (e.g., processes waiting for I/O) | High utilization means CPU is actively executing work |

### Example

Suppose a server has **8 CPU cores**.

- **CPU Utilization = 90%**
  - CPUs are busy executing tasks.

- **Load Average = 12**
  - 12 processes are either running or waiting for CPU.
  - Since there are only 8 cores, some processes must wait.
  - This indicates CPU contention.

---

# 🎯 Interview Answer

> CPU utilization represents the percentage of CPU resources actively being used, whereas CPU load represents the number of processes that are running or waiting to run. High CPU utilization means the CPU is busy executing work. High load indicates demand for CPU resources and may also include processes waiting for CPU or uninterruptible resources such as disk I/O. Therefore, CPU utilization tells us how busy the CPU is, while CPU load tells us how much work is competing for CPU time.
