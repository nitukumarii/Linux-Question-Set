What happens when system memory becomes full?

When system memory becomes full, Linux first tries to reclaim memory using cache and buffer cleanup. 

**free -m**

If RAM is still insufficient, 
it starts using swap space. If both RAM and swap are exhausted, the system becomes slow and cannot allocate memory for new processes. 
At this stage, the Out Of Memory (OOM) Killer is triggered by the kernel, 
which calculates an OOM score for all processes and terminates the process with the highest score to free memory and keep the system stable.


OOM adjustable score which is ikely to be 


