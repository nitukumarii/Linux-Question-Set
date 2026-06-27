How do you troubleshoot performance issues without restarting services?


I follow a layered approach for performance troubleshooting without restarting services.

Step 1: System level

First I check system metrics like CPU, memory, swap, and IO using top, free -h, vmstat, and iostat.

Step 2: Process identification

Then I identify the top resource-consuming process using ps -aux or top to find the PID.

Step 3: Deep process analysis

After that, I drill down into the process to understand thread-level behavior.

Step 4: Memory / thread inspection

I use tools like pmap -x for memory mapping, and if it’s a Java application, I use jstack for thread dumps.

Final step

The goal is always to fix or isolate the issue without restarting the service, and restart is only the last option.
