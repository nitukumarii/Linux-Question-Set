### 1. What Linux flavor are you most comfortable with?

My primary experience has been with Red Hat Enterprise Linux (RHEL) and CentOS in production environments.
I've also worked with Ubuntu servers, particularly in cloud-native and containerized environments.

Most of my day-to-day operational work has involved Linux administration, troubleshooting, automation, 
performance monitoring, and Kubernetes workloads running on RHEL/CentOS-based systems.

Although distributions differ in package management and some system tools, 
the core Linux concepts such as process management, networking, storage, security, systemd, 
and performance troubleshooting remain largely the same. Because of this, I'm comfortable adapting to other distributions as required.

### 2. 
Difference between RHEL and Ubuntu?	                            yum/dnf vs apt, RPM vs DEB packages

Package manager used in RHEL?	                                  yum or dnf

Package manager used in Ubuntu?	                                apt

Check OS version?	                                              cat /etc/os-release

<img width="601" height="421" alt="image" src="https://github.com/user-attachments/assets/03f9541a-4c7c-4a47-accc-ad59063af79a" />


Check kernel version?                                          	uname -r

<img width="595" height="541" alt="image" src="https://github.com/user-attachments/assets/e7a78e83-ed4e-483a-a1f7-8b8c1e2d2074" />


Check architecture?	                                            uname -m

<img width="437" height="140" alt="image" src="https://github.com/user-attachments/assets/81ef3e8d-29ee-4d9a-bd45-e6216dd09ee2" />



### 3.
<img width="972" height="351" alt="image" src="https://github.com/user-attachments/assets/63e7326a-f157-4c8f-8aba-793aa11dfedf" />

### 4. 

# Common Linux Daemons Started During Boot

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

### systemd is a system and service manager that initializes the Linux system during boot and manages system services (daemons) and processes.
The first process (PID 1) that starts after the Linux kernel boots, and it then controls everything else in user space.



