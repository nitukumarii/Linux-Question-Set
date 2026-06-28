
**A service crashes after running for several hours. How is this issue debugged?**

I first check the service status using systemctl status <service> to confirm whether it has stopped, failed, or is continuously restarting.

Step 2: Analyze logs

I review journalctl -u <service> -xe and the application logs to identify exceptions, crashes, or recurring error patterns.

Step 3: Correlate with system resources

Since the issue occurs only after several hours, I would suspect a memory leak or resource exhaustion. I would check memory, CPU, disk, swap, and OOM events using free -m, top, dmesg, and monitoring dashboards.

Step 4: Identify the root cause

I correlate the application logs with system metrics to determine whether the issue is caused by a memory leak, file descriptor exhaustion, dependency failures, or resource limits, and then implement the appropriate fix.
