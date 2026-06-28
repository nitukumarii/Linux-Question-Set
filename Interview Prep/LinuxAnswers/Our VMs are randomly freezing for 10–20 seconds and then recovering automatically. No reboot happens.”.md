**Our VMs are randomly freezing for 10–20 seconds and then recovering automatically. No reboot happens.”**


You already checked:

CPU → normal
Memory → normal
Disk → normal
Network → no obvious packet loss

But the issue still continues across a few VMs on the same host.


**1. What are your top 3 likely root causes? (Think beyond VM-level issues)
2. What hidden metric or signal would you check next? (hint: something not visible in basic top)
3. How do you differentiate between: VM issue Host/hypervisor issue Network fabric issue
4. What is your immediate mitigation step to protect customers?**


1. Top 3 checks are as below

1. Hypervisor check
   Inside the VM, you usually cannot directly prove a hypervisor problem. You gather evidence, then use cloud infrastructure monitoring or escalate to the infrastructure team.

  ** Top command**
   Can check - CPU steal time that st
   Steal time  means The VM is ready to run, but the hypervisor isn't giving it CPU time.

   

   
