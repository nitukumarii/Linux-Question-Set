
## Scenario

You are the **On-Call Cloud Operations Engineer**.

A **P1 (Critical)** incident is raised.

### Customer Complaint

> "Our production application has become extremely slow immediately after today's deployment."

### Application Architecture

```
Users
   │
Load Balancer
   │
Web Servers
   │
Application Servers
   │
Database
```

### Deployment Information

* Deployment completed **15 minutes ago**
* Customers immediately started reporting slowness

### Current Monitoring

```
CPU        : 20%
Memory     : 45%
Disk       : Normal
Network    : Normal
Application Response Time

Before Deployment : 200 ms
After Deployment  : 6 seconds
```

---

# Interview Questions

## Question 1

As the On-Call Engineer, what will you do in the **first five minutes**?

---

## Question 2

How will you isolate the problem?

How will you determine whether the issue is in:

* Load Balancer
* Web Server
* Application Server
* Database

---

## Question 3

Would you immediately rollback today's deployment?

Explain your reasoning.

---

## Question 4

The developers tell you:

> "Deployment was successful from our side."

How will you continue your investigation?

---

# How an SRE Thinks (Most Important Part)

Most candidates immediately start saying:

```
top
ps
journalctl
vmstat
```

This is usually **not** what interviewers are looking for.

A good SRE follows this mindset:

```
Observe
        ↓
Form Hypothesis
        ↓
Isolate the Fault
        ↓
Mitigate Customer Impact
        ↓
Perform Root Cause Analysis (RCA)
```

---

# Step 1 — Observe

First, don't run Linux commands.

Ask yourself:

### What changed?

The deployment finished only 15 minutes ago.

The application became slow immediately afterwards.

This creates a strong correlation.

First hypothesis:

> The deployment introduced a regression.

---

# Step 2 — Don't Assume

Never assume the application is the problem.

Instead ask:

Which layer is slow?

```
Users
   │
Load Balancer
   │
Web
   │
Application
   │
Database
```

Your goal is to isolate the faulty layer.

---

# Step 3 — Isolate Each Layer

### Load Balancer

Questions:

* Is the Load Balancer healthy?
* Are health checks passing?
* Is traffic evenly distributed?
* Are backend servers healthy?

If yes

↓

Move to Web Tier.

---

### Web Server

Questions:

* Is the web server responding normally?
* Is only one web server affected?
* Are all web servers slow?

If web servers are healthy

↓

Move to Application Tier.

---

### Application Server

Questions:

* Are there application exceptions?
* Did the deployment introduce a bug?
* Infinite loop?
* Thread deadlock?
* Connection pool exhausted?

If application appears healthy

↓

Move to Database.

---

### Database

Questions:

* Are slow queries occurring?
* Database locks?
* Long-running transactions?
* Connection pool exhaustion?
* New indexes missing?
* Recent schema change?

---

# Step 4 — Mitigate

Remember:

As an SRE

Your first responsibility is NOT fixing the bug.

Your responsibility is reducing customer impact.

Possible mitigations:

* Roll back deployment
* Route traffic to previous version
* Remove unhealthy instances
* Scale healthy instances
* Enable failover if available

---

# Step 5 — Root Cause Analysis

After customers are stable

Investigate:

* Deployment logs
* Application logs
* Infrastructure metrics
* Database metrics
* Monitoring dashboards
* Configuration changes

---

# Sample Interview Answer

> Since the issue started immediately after today's deployment, my first hypothesis is that the deployment introduced a regression. Before running Linux commands, I would compare application performance before and after deployment and isolate which layer is slow. I would verify whether the Load Balancer, Web Server, Application Server, or Database is responsible. If customer impact is severe and evidence points to the deployment, I would coordinate with the development team to perform a rollback or redirect traffic to the previous stable version. Once service is restored, I would continue the Root Cause Analysis using application logs, deployment logs, monitoring dashboards, and database metrics.

---

# Golden Rule for SRE Interviews

Don't think:

```
Which Linux command should I run first?
```

Think:

```
What changed?

↓

Which layer is affected?

↓

How do I reduce customer impact?

↓

Which commands will prove my hypothesis?
```

Interviewers hire engineers who can **reason through incidents**, not engineers who only remember Linux commands.
