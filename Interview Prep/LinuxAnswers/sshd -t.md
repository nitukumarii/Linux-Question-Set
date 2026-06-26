# SSHD -t Interview Question & Answer

## Q1: What is `sshd -t`?

**Answer:**

`sshd -t` is a command used to validate the SSH daemon configuration file (`/etc/ssh/sshd_config`) without restarting or reloading the SSH service. It checks the syntax and configuration correctness.

If the configuration is correct, the command produces no output. If there is an error, it displays the line number and issue.

---
<img width="1017" height="432" alt="image" src="https://github.com/user-attachments/assets/a65b3c1d-32e1-4efe-8706-8556618e900d" />

## Q2: Why do we use `sshd -t` before restarting SSH?

**Answer:**

We use `sshd -t` to ensure that the SSH configuration file is valid before applying changes. This is critical in production because an incorrect SSH configuration can prevent the SSH service from starting, which may lock us out of the server.

---

## Q3: What happens if `sshd -t` fails?

**Answer:**

If `sshd -t` fails, it means there is a syntax or configuration error in `/etc/ssh/sshd_config`. In this case:

* The SSH service should NOT be restarted or reloaded.
* The configuration error must be fixed first.
* After correction, `sshd -t` should be run again to validate.

---

## Q4: What is the correct production workflow when updating SSH configuration?

**Answer:**

The safe production workflow is:

1. Edit SSH configuration file:

   ```bash
   vi /etc/ssh/sshd_config
   ```

2. Validate configuration:

   ```bash
   sshd -t
   ```

3. If validation is successful, apply changes:

   ```bash
   systemctl reload sshd
   ```

   or

   ```bash
   systemctl restart sshd
   ```

---

## Q5: What is the risk of skipping `sshd -t`?

**Answer:**

Skipping `sshd -t` is risky because:

* A syntax error may prevent SSH daemon from starting.
* This can cause loss of remote access to the server.
* In production environments, it can lead to downtime and require physical or console access to fix the issue.

---

## Q6: How is `sshd -t` different from `systemctl restart sshd`?

**Answer:**

| Command                  | Purpose                                                 |
| ------------------------ | ------------------------------------------------------- |
| `sshd -t`                | Validates SSH configuration without applying changes    |
| `systemctl restart sshd` | Stops and starts the SSH service with new configuration |

---

## Q7: What is the best practice before reloading SSH in production?

**Answer:**

Best practice is:

> Always validate the configuration using `sshd -t` before reloading or restarting the SSH service to avoid service failure and potential loss of access.

---

## One-line Interview Answer:

`sshd -t` is used to validate the SSH daemon configuration file for syntax errors before restarting or reloading the service, ensuring safe changes without risking SSH downtime or lockout.
