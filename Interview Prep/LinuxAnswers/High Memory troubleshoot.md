


To troubleshoot high memory utilization, I first check overall memory using **free -h** to understand RAM and swap usage.

Look at:

available memory (important, not just "used")
swap usage


Then I identify top memory-consuming processes using **ps aux --sort=-%mem** or **top** or **htop**. 

Find which process is consuming most RAM (Java, DB, app, etc.)


After identifying the process,

When:

One process is high memory
Restart fixes issue temporarily
Problem keeps coming back

So now we check

pmap -x <PID> 
/proc/<PID>/status

**pmap -x <PID> ** - Tells where memory is growing 

It breaks memory into regions:

heap
stack
shared libs
anonymous memory

<img width="576" height="191" alt="image" src="https://github.com/user-attachments/assets/5a728cd6-c59d-4f03-8cfa-17e01680e2f5" />

👉 If you see heap growing continuously → Java memory leak
👉 If stack is large → too many threads
👉 If anon memory grows → unmanaged allocations

**cat /proc/1234/status | grep -i vm**

<img width="587" height="522" alt="image" src="https://github.com/user-attachments/assets/31a958bf-7504-4598-8289-2c8bee2cd298" />

<img width="647" height="577" alt="image" src="https://github.com/user-attachments/assets/393698d1-f507-46ae-8d68-25709c9e0818" />

VmRSS helps us understand actual physical memory consumption and OOM risk, while VmData confirms heap growth trends. So pmap helps identify where memory is growing, 
and /proc helps understand how critical the issue is from a system stability perspective.



<img width="912" height="597" alt="image" src="https://github.com/user-attachments/assets/440b6763-c2a3-4e0e-b0d3-20d67d7f324b" />


. I also check system-level trends using **vmstat** and look for any OOM events in **dmesg** or **journalctl**. Finally, I determine whether the
issue is due to memory leak, high workload, or misconfiguration, and take corrective actions such as restarting the service, tuning memory, or scaling the application.



