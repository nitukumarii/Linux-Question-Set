# Linux Interview Questions (Highest Probability)

## Linux Fundamentals

* Difference between Process, Thread, and Daemon - **Done**
* Explain the Linux Boot Process - **Done**
* What happens when a Linux server boots?  **Done**
* What is `systemd`? - **Done**
* What is a Zombie Process? **Done**
* What is an Orphan Process?
* What is the OOM (Out of Memory) Killer? **Done**
* Difference between Soft Link (Symbolic Link) and Hard Link **Done**

## Memory Management

* What happens when system memory becomes full? **Done**
* How do you troubleshoot high memory utilization? **Done**
* How does Linux handle memory pressure?
* What is Swap Memory and when is it used?

## CPU Troubleshooting

* How do you troubleshoot high CPU utilization?
* What are common causes of CPU spikes?
* How do you identify CPU-intensive processes?
* Explain CPU Load vs CPU Utilization

## Disk & Storage Troubleshooting

* How do you troubleshoot a disk full issue?
* How do you troubleshoot high disk I/O?
* What are common causes of disk latency?
* How do you identify large files consuming disk space?

## Performance & Monitoring

* What is Load Average?
* How do you interpret Load Average values?
* A system has 16 CPUs and a Load Average of 40. What does it indicate?
* How do you determine if a server is CPU-bound or I/O-bound?

## Kernel & System Stability

* What causes a Kernel Panic? **done**
* How do you troubleshoot a Kernel Panic?**done*
* What logs and tools would you use after a Kernel Panic?**done**
* Difference between User Space and Kernel Space **done**

## Linux Performance Tools

### vmstat

* What is `vmstat`?
* How do you interpret `vmstat` output?
* Which columns are important during troubleshooting?

### iostat

* What is `iostat`?
* How do you analyze disk performance using `iostat`?
* What do `%util`, `await`, and `svctm` indicate?

### sar

* What is `sar`?
* How do you use `sar` for historical performance analysis?
* What metrics can be collected using `sar`?

## Practical Scenario-Based Questions

* Production server CPU is consistently above 90%. How would you troubleshoot?
* Application is slow but CPU usage is low. What would you check?
* Memory utilization is 100%. What steps would you take?
* Disk usage suddenly reaches 100%. How would you investigate?
* Load Average is high but CPU usage is low. What does this indicate?
* Users report slowness after reboot. How would you approach troubleshooting?
* A process is stuck in an uninterruptible state (D state). What could be the cause?
* Server becomes unresponsive every day at a specific time. How would you investigate?
