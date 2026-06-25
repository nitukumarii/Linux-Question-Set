# Difference Between Process, Thread, and Daemon

## Process

A **process** is an independent running instance of a program with its own memory space and system resources.

### Characteristics

* Has its own Process ID (PID)
* Own memory space (virtual memory)
* Own file descriptors and resources
* Can contain one or more threads
* Communication between processes requires IPC mechanisms

### Examples

```bash
ps -ef
top
```

Processes:

* nginx
* java application
* mysql
* sshd

### Example

When you start a Java application:

```bash
java -jar app.jar
```

Linux creates a new process with its own PID.

---

## Thread

A **thread** is the smallest unit of execution within a process.

Multiple threads inside the same process share memory and resources.

### Characteristics

* Shares memory with other threads in the same process
* Has its own Thread ID (TID)
* Faster context switching than processes
* Lightweight compared to processes

### Example

A web server process may have:

```text
Nginx Process
 ├── Thread 1 (Request Handling)
 ├── Thread 2 (Request Handling)
 ├── Thread 3 (Logging)
 └── Thread 4 (Cache Management)
```

All threads share:

* Memory
* Open files
* Network connections

---

## Daemon

A **daemon** is a background process that runs continuously and provides services to the system.

It usually starts during system boot and runs without user interaction.

### Characteristics

* Runs in the background
* Usually managed by systemd
* No terminal attached
* Starts automatically during boot
* Provides system services

### Examples

```bash
systemctl list-units --type=service
```

Common daemons:

* sshd
* crond
* systemd-journald
* nginx
* docker
* kubelet

### Example

```bash
systemctl status sshd
```

The SSH daemon keeps running and waits for incoming SSH connections.

---

# Key Differences

| Feature            | Process                       | Thread                                   | Daemon                     |
| ------------------ | ----------------------------- | ---------------------------------------- | -------------------------- |
| Definition         | Running instance of a program | Smallest execution unit inside a process | Background service process |
| Memory             | Separate memory space         | Shared memory within process             | Separate process memory    |
| Resource Sharing   | No                            | Yes                                      | No                         |
| PID                | Yes                           | Belongs to a process PID                 | Yes                        |
| Runs in Background | Not necessarily               | Not necessarily                          | Yes                        |
| User Interaction   | Possible                      | Possible                                 | Usually none               |
| Created By         | OS                            | Process                                  | OS/Systemd                 |
| Example            | Java App, MySQL               | Worker Thread                            | sshd, crond, kubelet       |

---

# Interview Answer (2-Minute Version)

A process is an independent running program with its own memory space and system resources. For example, a Java application or MySQL server running on Linux

A thread is a lightweight execution unit within a process. Multiple threads share the same memory and resources, making communication faster and context switching cheaper than between processes.

A daemon is a special type of process that runs in the background and provides system services. Examples include sshd, crond, Docker, and kubelet. Daemons typically start during system boot and continue running without user interaction.



#### Common Linux Daemons Started During Boot

When a Linux system boots, `systemd` starts various daemons (background services) required for operating system functionality, networking, logging, security, and application workloads.

| Daemon                | Purpose                                                                         |
| --------------------- | ------------------------------------------------------------------------------- |
| systemd               | First userspace process (PID 1) responsible for starting and managing services. |
| systemd-journald      | Collects and stores system logs.                                                |
| systemd-logind        | Manages user logins and sessions.                                               |
| dbus-daemon           | Provides inter-process communication between applications and services.         |
| NetworkManager        | Manages network interfaces, IP addresses, DNS, and connectivity.                |
| systemd-networkd      | Alternative network management daemon often used on servers.                    |
| sshd                  | Accepts incoming SSH connections for remote administration.                     |
| crond / cron          | Executes scheduled tasks and jobs.                                              |
| rsyslogd              | Traditional syslog daemon for centralized log collection and forwarding.        |
| chronyd / ntpd        | Synchronizes system time with NTP servers.                                      |
| firewalld             | Manages firewall rules dynamically.                                             |
| auditd                | Records security-relevant system events for auditing and compliance.            |
| polkitd               | Provides authorization framework for privileged operations.                     |
| udevd (systemd-udevd) | Detects and manages hardware devices dynamically.                               |
| multipathd            | Manages multiple storage paths for high availability storage systems.           |
| tuned                 | Dynamically optimizes system performance profiles.                              |
| docker                | Manages Docker containers.                                                      |
| containerd            | Container runtime used by Docker and Kubernetes.                                |
| kubelet               | Kubernetes node agent responsible for managing pods and containers.             |
| nginx                 | Web server and reverse proxy service.                                           |
| httpd                 | Apache web server daemon.                                                       |
| mysqld                | MySQL database server process.                                                  |
| postgresql            | PostgreSQL database server process.                                             |
| redis-server          | In-memory key-value database service.                                           |
| elasticsearch         | Search and analytics engine service.                                            |
| prometheus            | Metrics collection and monitoring service.                                      |
| grafana-server        | Visualization and dashboard service.                                            |

---

# Most Important Daemons for SRE Interviews

### systemd

Service manager that starts and monitors all services during boot.

### systemd-journald

Stores system logs that can be viewed using:

```bash
journalctl -xe
```

### sshd

Allows remote login and server administration.

### NetworkManager

Configures network interfaces and maintains connectivity.

### crond

Runs scheduled jobs such as backups and maintenance scripts.

### auditd

Captures security events and user activities.

### chronyd

Keeps system time synchronized across servers.

### docker/containerd

Runs and manages containers.

### kubelet

Communicates with Kubernetes control plane and ensures pods are running.

---

# Interview Question

**Q: Which daemons would you check first if a server is not functioning correctly after reboot?**

**Answer:**

1. systemd → Verify service startup.
2. NetworkManager/systemd-networkd → Check network connectivity.
3. sshd → Ensure remote access is available.
4. systemd-journald/rsyslog → Review logs.
5. application daemon (nginx, docker, kubelet, mysqld, etc.) → Verify application services.
6. chronyd → Confirm time synchronization.
7. auditd → Check for security-related issues.

Useful commands:

```bash
systemctl list-units --type=service
systemctl --failed
journalctl -b
systemctl status sshd
```

#### Daemon, Service Files, Config Files & daemon-reload

## What is a Daemon?

A daemon is a background process that runs continuously and provides system or application services.

Examples:

* sshd (SSH server)
* nginx (Web server)
* crond (Scheduler)
* docker (Container runtime)
* kubelet (Kubernetes node agent)

Most daemons are managed by `systemd`.

---

# Service File vs Configuration File

## Service File (.service)

Defines **how systemd should manage a service**.

Typical locations:

### RHEL/CentOS/Rocky

```bash
/usr/lib/systemd/system/
```

### Ubuntu/Debian

```bash
/lib/systemd/system/
```

### Custom Services

```bash
/etc/systemd/system/
```

Example:

```bash
/etc/systemd/system/myapp.service
```

Sample:

```ini
[Unit]
Description=My Application

[Service]
ExecStart=/opt/app/app.jar
Restart=always

[Install]
WantedBy=multi-user.target
```

---

## Configuration File

Defines **how the application behaves**.

Examples:

### Nginx

```bash
/etc/nginx/nginx.conf
```

### SSH

```bash
/etc/ssh/sshd_config
```

### Docker

```bash
/etc/docker/daemon.json
```

### MySQL

```bash
/etc/my.cnf
```

### Application

```bash
application.properties
application.yml
```

---

# What Does systemctl daemon-reload Do?

```bash
systemctl daemon-reload
```

It tells **systemd**:

> "Re-read all service (.service) files because they may have changed."

Important:

* Does NOT restart the application.
* Does NOT reread application config files.
* Only reloads systemd service definitions into memory.

---

# When to Use daemon-reload

## Service File Changed

Example:

```bash
/etc/systemd/system/myapp.service
```

Changed:

```ini
ExecStart
Environment
Restart
User
```

Run:

```bash
systemctl daemon-reload
systemctl restart myapp
```

✅ Correct

---

# When NOT to Use daemon-reload

## Application Config Changed

Example:

```bash
/etc/nginx/nginx.conf
```

Run:

```bash
nginx -t
systemctl reload nginx
```

❌ No daemon-reload needed

---

Example:

```bash
/etc/ssh/sshd_config
```

Run:

```bash
sshd -t
systemctl reload sshd
```

❌ No daemon-reload needed

---

# Reload vs Restart

Reload is preferred when an application supports re-reading configuration without stopping the process, as it avoids downtime. Restart stops the existing process and starts a new one, which may cause a brief interruption. For service file changes, I perform daemon-reload followed by a restart. For configuration changes, I use reload if the application supports it; otherwise, I perform a restart.

## Reload

```bash
systemctl reload nginx
```

* Service keeps running
* Reads new configuration
* Usually no downtime

## Restart

```bash
systemctl restart nginx
```

* Stops service
* Starts service again
* May cause brief downtime

---

# Interview Questions

### Q: I changed nginx.conf. What should I do?

```bash
nginx -t
systemctl reload nginx
```

Do NOT run:

```bash
systemctl daemon-reload
```

---

### Q: I changed myapp.service. What should I do?

```bash
systemctl daemon-reload
systemctl restart myapp
```

---

# Memory Trick

```text
.service file changed
    ↓
daemon-reload + restart

Application config changed
    ↓
reload/restart service
```

---

# One-Line Interview Answer

`systemctl daemon-reload` is used only when a systemd service file (.service) has been created or modified. It instructs systemd to reread service definitions. For application configuration changes, I use service reload or restart instead.
