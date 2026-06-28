A customer reports:

"Our application suddenly stopped working.

You investigate and find:

Disk Usage (/): 100%
CPU: Normal
Memory: Normal
Network: Normal

Users are receiving HTTP 500 Internal Server Error.

Questions
1. What is your first action?
2. How will you identify what is consuming the disk space?
3. Which Linux commands will you use?
4. How will you restore the service quickly?
5. After restoring the service, what preventive measures would you implement to avoid this issue in the future?


Since the application is returning HTTP 500 errors and the root filesystem is 100% full, my first priority is to restore the service. I would first identify which filesystem is full using df -h. Next, I would identify the directories consuming the most space using du -sh. If log files are consuming excessive space, I would safely archive, compress, or remove old logs after verification. If application or user files are responsible, I would coordinate with the respective teams before cleanup. I would also check inode usage using df -i to rule out inode exhaustion. Once sufficient space is freed and the application recovers, I would implement log rotation, monitoring, and disk usage alerts to prevent recurrence.
