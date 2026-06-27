# 🧠 Kernel Panic & Linux Core Concepts (Interview Ready)

---

# 🔥 1. What causes a Kernel Panic?

A **Kernel Panic** happens when the Linux kernel detects a **fatal internal error** and cannot safely continue execution.

### Common causes:

- ❌ Hardware failure (RAM, disk, CPU issues)
- ❌ Corrupted filesystem
- ❌ Missing or corrupted kernel modules
- ❌ Bad driver installation or update
- ❌ Kernel bug after upgrade
- ❌ Disk full in critical partitions (/, /boot)
- ❌ RAID/storage controller failure
- ❌ Init process (PID 1) failure
- ❌ Out of memory in critical kernel path

---

# 🧠 2. How do you troubleshoot a Kernel Panic?

## 🔴 Step 1: Identify panic screen / reboot pattern
- Server suddenly rebooted
- Console shows "Kernel Panic - not syncing"

---

## 🔴 Step 2: Check previous boot logs

```bash
journalctl -b -1
```

👉 Shows logs from previous boot (VERY IMPORTANT)

---

## 🔴 Step 3: Check kernel messages

```bash
dmesg -T | tail -100
```

or after reboot:

```bash
journalctl -k
```

---

## 🔴 Step 4: Check system logs

```bash
journalctl -xe
```

or

```bash
/var/log/messages
/var/log/syslog
```

---

## 🔴 Step 5: Check crash dumps (if enabled)

```bash
ls /var/crash/
```

or kdump:

```bash
systemctl status kdump
```

---

## 🔴 Step 6: Check disk & filesystem health

```bash
df -h
lsblk
fsck -y /dev/sdX
```

---

## 🔴 Step 7: Check hardware issues

```bash
smartctl -a /dev/sda
```

---

# 🧠 3. What logs and tools are used after Kernel Panic?

## 📁 Logs:

- `/var/log/messages`
- `/var/log/syslog`
- `journalctl -k`
- `journalctl -b -1`
- `/var/crash/*`
- `dmesg`

---

## 🛠️ Tools:

- `journalctl`
- `dmesg`
- `kdump`
- `crash` utility (kernel dump analysis)
- `smartctl`
- `fsck`
- `vmcore` analysis tools

---

# 🧠 4. Difference: User Space vs Kernel Space

| Feature | User Space | Kernel Space |
|----------|------------|--------------|
| Access level | Restricted | Full access |
| Runs | Applications (nginx, java, ssh) | OS kernel |
| Stability impact | App crash only | System crash possible |
| Memory access | Limited | Full system memory |
| Privilege | Low | High |
| Examples | Chrome, Python, Java apps | Process scheduler, drivers, memory manager |

---

## 🔥 Simple explanation:

- **User Space = Applications layer**
- **Kernel Space = Core OS brain**

---

# 🎯 Interview Answer (Kernel Panic)

> A kernel panic occurs when the Linux kernel encounters a critical error from which it cannot recover safely, causing the system to halt or reboot. Common causes include hardware failures, corrupted filesystems, faulty drivers, kernel bugs after updates, or critical resource exhaustion. To troubleshoot, I first check previous boot logs using `journalctl -b -1`, analyze kernel messages using `dmesg` or `journalctl -k`, and review system logs under `/var/log`. I also check for crash dumps in `/var/crash` if kdump is enabled and validate disk and hardware health using tools like `fsck` and `smartctl`. Kernel panic analysis often involves distinguishing between hardware, filesystem, or kernel-level software issues.

---

# 🧠 One-line memory trick

```text
Kernel Panic = Kernel cannot recover → system stops completely
```
