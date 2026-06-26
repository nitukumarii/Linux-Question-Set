**How does Linux handle memory pressure?**


# 🧠 How does Linux handle memory pressure?

Memory pressure occurs when **available RAM becomes low** and the system needs to allocate memory for new or existing processes.

Linux handles memory pressure in the following order:

## 1. Reclaim Memory

The kernel first tries to free memory by reclaiming:

- Page cache
- Buffer cache
- Unused memory pages

This is why high "used" memory in Linux is not always a problem.

---

## 2. Use Swap

If reclaiming memory is not enough, the kernel moves inactive memory pages from **RAM to swap space**.

This frees RAM for active processes but is slower because swap resides on disk.

---

## 3. Memory Allocation Failure

If both RAM and swap become exhausted, the kernel cannot allocate additional memory for processes.

At this stage, the system may become slow or unresponsive.

---

## 4. OOM Killer

To prevent the system from hanging, the Linux kernel activates the **Out Of Memory (OOM) Killer**.

The OOM Killer:

- Calculates an OOM score for running processes.
- Selects the process with the highest OOM score (typically a high-memory, non-critical process).
- Terminates the selected process to free memory.

This allows the operating system to continue functioning instead of crashing.

---

## Troubleshooting Commands

Check memory:

```bash
free -h
```

Check swap:

```bash
swapon --show
```

Check memory statistics:

```bash
vmstat 1 5
```

Check OOM events:

```bash
dmesg | grep -i oom
```

---

# 🎯 Interview Answer

> When Linux experiences memory pressure, it first attempts to reclaim memory by freeing page cache and other reclaimable memory. If that is insufficient, it starts using swap space by moving inactive pages from RAM to disk. If both RAM and swap are exhausted, the kernel cannot allocate more memory and triggers the Out Of Memory (OOM) Killer. The OOM Killer calculates an OOM score for running processes and terminates the process with the highest score to free memory and keep the system stable.
