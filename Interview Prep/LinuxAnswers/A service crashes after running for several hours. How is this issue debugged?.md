
**A service crashes after running for several hours. How is this issue debugged?**

Step 1: Service status check

I first check the service state using systemctl status <service> to confirm whether it is stopped, failed, or restarting.

Step 2: Log analysis

Then I analyze logs using journalctl -u <service> -xe and also check application-specific log files to identify error patterns or crash reasons.

Step 3: System resource correlation

Since the issue happens after several hours, I check system metrics like memory usage, CPU spikes, disk space, and possible OOM kills using dmesg, top, and monitoring dashboards.

Step 4: Root cause isolation

I then correlate logs with system behavior to identify causes like memory leaks, file descriptor exhaustion, dependency failures, or resource limits, and take corrective action accordingly.
