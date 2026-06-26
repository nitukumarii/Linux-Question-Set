flowchart TD
A[Power ON] --> B[CPU Starts]
B --> C[BIOS / UEFI]
C --> D[POST Hardware Check]
D --> E[Bootloader - GRUB]
E --> F[Linux Kernel]
F --> G[initramfs]
G --> H[Mount Root Filesystem /]
H --> I[systemd PID 1]
I --> J[Targets]
J --> K[Services Start]
K --> L[System Ready / Login]

### / (Root Filesystem)

This is the top-level directory of the entire Linux filesystem.



### /Boot Filesystem

<img width="407" height="517" alt="image" src="https://github.com/user-attachments/assets/529bb0ea-0747-402c-a680-2d1b5afb6b31" />




### /Root Filesystem
<img width="471" height="550" alt="image" src="https://github.com/user-attachments/assets/ef627326-40ee-459a-823c-64bbc9164d28" />





### /home
<img width="407" height="322" alt="image" src="https://github.com/user-attachments/assets/cc67c21c-aead-4de8-8476-78fb53415e46" />


### /etc
<img width="427" height="315" alt="image" src="https://github.com/user-attachments/assets/176a3159-75c2-4f5d-b079-913e4dcc2153" />



### /var

<img width="412" height="311" alt="image" src="https://github.com/user-attachments/assets/fd7343b2-d8d0-4a1e-b5db-956a08898078" />



### /usr

<img width="482" height="287" alt="image" src="https://github.com/user-attachments/assets/cf2d794d-4a49-4436-9234-d2d33de11cc3" />




### /proc
<img width="431" height="317" alt="image" src="https://github.com/user-attachments/assets/ae166996-cd5d-41d9-9708-c7901fc6d15b" />




### /sys

<img width="312" height="261" alt="image" src="https://github.com/user-attachments/assets/46a8bb22-515a-477a-a8c7-b45a72ab95d9" />




### /dev

<img width="292" height="302" alt="image" src="https://github.com/user-attachments/assets/3397ecb7-f069-46d4-a434-d7ada4507d4f" />



