# 🐧 What is a Target in Linux (systemd)

## 📌 Definition

A **target** is a `systemd unit` that represents a **system state**.


---

## 📊 Types of Targets

| Target | Meaning | Use |
|--------|--------|-----|
| `multi-user.target`                  |                            CLI server mode | ⭐ Production servers |
| `graphical.target`                   |                             GUI desktop mode | Laptops / desktops |
| `rescue.target`                       |                                Single-user recovery mode | Troubleshooting |
| `emergency.target`                   |                             Minimal boot (root shell only) | Critical failure recovery |
| `poweroff.target`                       |                              Shutdown system | Power off |
| `reboot.target`                      |                                Restart system | Reboot |

---

## 🔥 Most Used in Production

👉 **multi-user.target**

Because production systems:

- No GUI required
- Only CLI + services

### It includes services like:

- SSH
- Networking
- Logging
- Cron
- Application services

✔ This is the default target for most Linux servers.

---

## 📁 Where are Targets defined?

Targets are systemd unit files located in:

```bash

/usr/lib/systemd/system/

or

/lib/systemd/system/

Example

/usr/lib/systemd/system/multi-user.target


/usr/lib/systemd/system/graphical.target


<img width="641" height="487" alt="image" src="https://github.com/user-attachments/assets/77c0c45a-2331-42b8-90af-2384a07a5f6d" />



How to check current target?


systemctl get-default


How to check active target?

systemctl list-units --type=target



<img width="485" height="635" alt="image" src="https://github.com/user-attachments/assets/e43f5f3d-ff69-4949-96e1-c0e31e153887" />



<img width="490" height="612" alt="image" src="https://github.com/user-attachments/assets/7a19b869-8fa7-4c0d-a037-f34c07ed35c9" />



<img width="757" height="637" alt="image" src="https://github.com/user-attachments/assets/65787af1-1298-4bfb-a103-b8f4b0225ffb" />


