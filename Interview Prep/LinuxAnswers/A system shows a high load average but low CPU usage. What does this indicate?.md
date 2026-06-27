**A system shows a high load average but low CPU usage. What does this indicate?**


If a system shows high load average but low CPU utilization, it usually indicates that the system is not CPU-bound.

Load average includes processes that are either running or waiting in the queue, including those waiting for CPU or I/O.

When CPU usage is low but load is high, it often means processes are stuck in uninterruptible sleep state, usually waiting for disk I/O.

In such cases, I check I/O wait using tools like iostat or vmstat.

So the bottleneck is typically disk or storage latency, not CPU.
