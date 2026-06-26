A **Zombie Process** (also called a **defunct process**) is a process that has **finished execution but still has an entry in the process table**
because its parent process has **not yet collected its exit status**.
\

## Does a Zombie Process Slow Down the System?

### Normally: **No**

A zombie process:

- ❌ Does not consume CPU
- ❌ Does not consume RAM
- ❌ Does not execute any code

It only occupies:

- A PID
- An entry in the process table

## How to Identify a Zombie Process?
ps -el

or

ps aux

Look for

STAT = Z

## How to Remove (Kill) a Zombie Process?

### ❌ You cannot kill a zombie process directly.

Since the process has already exited, there is nothing left to terminate.

Instead, you must deal with the **parent process**.

### Option 1 (Preferred)

If the parent process is healthy, it should call `wait()` automatically and clean up the zombie.

---

### Option 2

Restart or terminate the parent process.

```bash
kill -TERM <parent_pid>
```

or if necessary:

```bash
kill -9 <parent_pid>
```

When the parent exits, the zombie is adopted by **systemd (PID 1)**, which collects its exit status and removes it.
