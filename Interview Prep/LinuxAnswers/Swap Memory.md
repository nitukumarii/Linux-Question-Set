# 🧠 What is Swap Memory and when is it used?

## What is Swap Memory?

**Swap memory** is a reserved area on disk (either a **swap partition** or a **swap file**) that Linux uses as **virtual memory** when physical RAM becomes insufficient.

Since swap resides on disk, it is **much slower than RAM**.

---

## When is Swap Used?

Linux uses swap when:

- Available RAM becomes low.
- The kernel moves **inactive or less frequently used memory pages** from RAM to swap.
- This frees up RAM for active processes.

Swap helps prevent the system from running out of memory immediately.

---

## What happens if Swap also becomes full?

If both **RAM and swap are exhausted**:

- New memory allocations fail.
- The system becomes slow or unresponsive.
- The Linux kernel activates the **OOM (Out Of Memory) Killer**, which terminates one or more processes to free memory.

---

## Check Swap Usage

View active swap:

```bash
swapon --show
```

Check RAM and swap usage:

```bash
free -h
```

---

## Interview Answer

> Swap memory is disk space used by Linux as virtual memory when available RAM becomes low. The kernel moves inactive memory pages from RAM to swap, freeing RAM for active processes. Although swap helps prevent immediate out-of-memory situations, it is much slower than RAM because it resides on disk. If both RAM and swap become exhausted, the Linux kernel triggers the OOM Killer to terminate one or more processes and free memory.
