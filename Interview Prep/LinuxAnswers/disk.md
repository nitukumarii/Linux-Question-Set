# 🧠 How do you troubleshoot a Disk Full issue?

## 1. Check disk usage

```bash
df -h
```

This shows:

- Filesystem
- Total size
- Used space
- Available space
- Mount point

Identify which filesystem is full.

---

## 2. Identify directories consuming space

```bash
du -sh /* 2>/dev/null
```

If `/var` is large:

```bash
du -sh /var/* 2>/dev/null
```

Keep drilling down until you find the culprit.

---

## 3. Identify large files

```bash
find / -type f -size +500M 2>/dev/null
```

or

```bash
find /var -type f -size +500M
```

---

## 4. Check inode usage

Sometimes disk space is available but no new files can be created.

```bash
df -i
```

If inode usage is **100%**, too many small files are consuming all inodes.

---

## 5. Check deleted but open files

Sometimes logs are deleted but still held open by a running process.

```bash
lsof | grep deleted
```

Restart the process or service to release the space.

---

## 6. Take corrective action

Depending on the cause:

- Delete unnecessary files
- Rotate or compress logs
- Archive old backups
- Increase disk capacity if required

---

# 🎯 Interview Answer

> To troubleshoot a disk full issue, I first check filesystem usage using `df -h` to identify which partition is full. Then I use `du` to locate directories consuming the most space and `find` to identify large files. I also check inode usage with `df -i`, as a filesystem can run out of inodes even when disk space is available. Finally, I check for deleted but open files using `lsof | grep deleted` and take corrective actions such as cleaning up files, rotating logs, archiving data, or increasing disk capacity if necessary.

---

# 🧠 How do you troubleshoot High Disk I/O?

## 1. Check disk utilization

```bash
iostat -x 1 5
```

Look for:

- `%util`
- `await`
- `r/s`
- `w/s`

---

## 2. Identify processes causing I/O

```bash
iotop
```

or

```bash
pidstat -d 1
```

---

## 3. Check CPU I/O wait

```bash
vmstat 1 5
```

Look at:

```text
wa
```

High `wa` means CPU is waiting for disk operations.

---

## 4. Check application logs

Look for:

- excessive logging
- backup jobs
- database activity
- file copy operations

---

## 5. Corrective actions

- Optimize application I/O
- Reduce unnecessary logging
- Tune database queries
- Move workload to faster storage (SSD/NVMe)
- Scale storage if required

---

# 🎯 Interview Answer

> To troubleshoot high disk I/O, I first use `iostat -x` to analyze disk utilization, latency, and throughput. Then I identify the processes generating the most disk activity using `iotop` or `pidstat -d`. I also check CPU I/O wait using `vmstat`, as high wait time indicates disk bottlenecks. Finally, I determine whether the issue is caused by heavy application activity, database operations, logging, or backup jobs and take appropriate corrective actions.

---

# 🧠 What are common causes of Disk Latency?

Common causes include:

- Heavy read/write workload
- Slow HDD storage
- Database-intensive operations
- Backup or restore jobs
- Large file transfers
- Excessive logging
- Disk nearing full capacity
- RAID rebuild or storage issues
- Virtualized storage contention
- Network-attached storage latency

---

# 🧠 How do you identify Large Files consuming disk space?

### Find files larger than 500 MB

```bash
find / -type f -size +500M 2>/dev/null
```

### Find files larger than 1 GB

```bash
find / -type f -size +1G 2>/dev/null
```

### Find the largest directories

```bash
du -sh /* 2>/dev/null
```

Sort directories by size:

```bash
du -sh /var/* | sort -hr
```

---

# 🎯 Interview Answer

> I identify large files using the `find` command with the `-size` option and locate large directories using `du`. I first identify the filesystem that is full using `df -h`, then drill down into directories to find the files consuming the most disk space. This helps determine whether the issue is caused by logs, backups, application data, or temporary files.
