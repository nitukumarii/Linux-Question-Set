** Server becomes unresponsive every day at a specific time. How would you investigate?**


When a server becomes unresponsive at a specific time or for a sustained period, I first treat it as a time-correlated incident rather than a sudden spike issue, which immediately leads me to suspect scheduled workloads such as cron jobs, batch processing, backups, log rotation, or ETL/report generation tasks. I start the investigation by confirming the exact time of impact, the duration of slowness, and correlating it with system resource usage during that window. For real-time validation, I use top to understand the current system state, checking user CPU, system CPU, idle CPU, I/O wait, and memory usage to determine whether the system is CPU-bound, memory-bound, or I/O-bound. To observe short-term behavior, I use vmstat 1 10, which provides per-second snapshots of CPU, run queue, memory, swap in/out, and I/O activity, helping me understand whether the issue is sustained or intermittent. For historical analysis, I rely on sar -q -f /var/log/sa/saXX and related SAR metrics, which help reconstruct past CPU, load, memory, and I/O patterns during the incident window. To understand the root cause beyond system metrics, I check journalctl and dmesg; if dmesg does not show any OOM kills, driver failures, or disk errors, it indicates that the kernel and hardware are stable, and the issue is likely at the application layer. In such cases, journalctl -u <service> and application logs help identify service crashes, traffic spikes, or abnormal behavior such as retries or failures. I further correlate this with traffic patterns using ss -tulnp or web server logs like nginx access logs to detect request surges or repeated authentication failures. Process-level analysis is done using ps aux --sort=-%cpu and top to identify CPU- or memory-intensive processes, while vmstat confirms overall system saturation. If the application is Java-based, I drill deeper using top -H -p <PID> to identify the specific thread consuming CPU and then use jstack to map that thread to the application code path. Finally, I correlate all findings to determine whether the issue is caused by scheduled jobs, application inefficiencies, traffic load, or configuration changes, and then apply the appropriate fix such as optimizing the job, rescheduling workloads, tuning the application, or restarting the affected service to restore stability.





When I run ss -tulnp, it shows all the network services currently running on the server along with the ports they are listening on and the process ID responsible for them. From this output, I can understand which services like SSH, Nginx, or database are active and whether any unexpected or unknown service is consuming a port. This helps me verify the system-side status of the server. On the other hand, when I run tail -f /var/log/nginx/access.log, I can see real-time user traffic coming to the web server. It shows which IP addresses are hitting the server, which API endpoints are being called, and how frequently requests are coming. If I notice repeated API calls or sudden spikes in requests from the same IP or endpoint, I can suspect issues like traffic surge, bot activity, or application retry loops. In production troubleshooting, I use ss -tulnp to understand what is running on the server and access.log to understand who is hitting the server and how the traffic pattern looks, then correlate both to identify whether the issue is system-level or traffic-related.


ss tulnp output

<img width="962" height="432" alt="image" src="https://github.com/user-attachments/assets/cfdd2fcb-4964-4b1f-a701-990e0bfc1627" />


🚨 Why we use this in CPU issue?

If server is slow:

Check if too many requests hitting nginx
Check if port 80/443 is overloaded
Identify unexpected services
