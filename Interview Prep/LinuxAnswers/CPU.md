# 🧠 Interview Question: How do you troubleshoot High CPU Utilization?

## Production Scenario

In one of our production environments, users started reporting that the application was responding very slowly and API response times had increased significantly.

---

## Step 1: Verify CPU Utilization

I first logged into the server and checked the CPU utilization.


**top**


I observed:

- User CPU (us) = 72%
- System CPU (sy) = 18%
- Idle CPU (id) = 8%
- I/O Wait (wa) = 2%

This indicated that the CPU was heavily utilized and the issue was **CPU saturation**, not a disk I/O bottleneck.

---

## Step 2: Identify the CPU-intensive Process

Next, I identified the process consuming the highest CPU.


**ps aux --sort=-%cpu | head**


Output:

```text
USER   PID   %CPU  %MEM  COMMAND
root  12345  340   12.3  java
```

I found that a Java application was consuming around **340% CPU** on a multi-core server.

---

## Step 3: Check Whether It's a Temporary Spike

To verify whether it was a temporary spike or a continuous issue, I monitored the server using:


**vmstat 1 10**

I observed:

- High User CPU
- Very Low Idle CPU
- No Swap Activity
- Normal I/O Wait

This confirmed that the issue was sustained CPU utilization and not caused by memory pressure or disk latency.

---

## Step 4: Check Thread-Level CPU Usage

Since it was a Java application, I checked thread-level CPU usage.


**top -H -p 12345**


I found one thread consuming nearly **100% CPU** continuously.

---

## Step 5: Capture Thread Dump

I converted the thread ID into hexadecimal.


**printf "%x\n" <ThreadID>**


Then collected the Java thread dump.


**jstack 12345 > threaddump.txt**

Searching for the hexadecimal thread ID in the thread dump helped identify the exact Java thread responsible.

---

## Step 6: Root Cause Analysis

After reviewing the thread dump and application logs, we found that:

- A recent deployment introduced an inefficient loop.
- The application repeatedly processed the same records.
- One thread continuously executed without terminating, consuming excessive CPU.

---

## Step 7: Resolution

### Immediate Mitigation

- Restarted the Java service to restore application availability.

### Permanent Fix

The development team:

- Fixed the inefficient loop.
- Optimized the processing logic.
- Released a new version.

We also:

- Added Prometheus CPU alerts.
- Improved load testing before deployments.
- Added monitoring dashboards in Grafana.

The application CPU utilization dropped from **340%** to around **40–50%**, and response times returned to normal.

---

# Commands Used

```bash
top
```

```bash
ps aux --sort=-%cpu | head
```

```bash
vmstat 1 10
```

```bash
top -H -p <PID>
```

```bash
printf "%x\n" <ThreadID>
```

```bash
jstack <PID>
```

---

# 🎯 Final Interview Answer

> When users reported that the application was slow, I first verified CPU utilization using `top` and observed high user CPU with very low idle CPU, confirming CPU saturation. I then identified the top CPU-consuming process using `ps aux --sort=-%cpu` and found a Java process consuming most of the CPU. To determine whether it was a temporary spike or a sustained issue, I monitored the system using `vmstat`, which consistently showed high user CPU and low idle CPU with normal I/O wait. Since it was a Java application, I analyzed thread-level CPU usage using `top -H` and collected a thread dump using `jstack` to identify the thread responsible. The investigation revealed that a recently deployed feature had introduced an inefficient loop, causing one thread to consume excessive CPU. As an immediate mitigation, I restarted the service to restore availability. The development team then fixed the code, and we deployed the updated version. We also strengthened monitoring and alerting to detect similar issues proactively.
