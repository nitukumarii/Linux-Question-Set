# Linux Boot Process (Interview Answer)

When we switch on the power button, the CPU starts executing the firmware stored on the motherboard, which is either **BIOS (Basic Input Output System)** or **UEFI (Unified Extensible Firmware Interface)**.

The firmware initiates the boot process by performing the **Power-On Self-Test (POST)**. During POST, it checks essential hardware components such as the **CPU, RAM, keyboard, display, storage controllers, and storage devices**. If a critical hardware issue is detected, the boot process is aborted.

After POST, the firmware initializes the storage controllers (such as SATA or NVMe controllers) so that it can access the bootable disk.

### BIOS Boot Process

In BIOS-based systems, the BIOS reads the **Master Boot Record (MBR)** located in the **first sector (Sector 0)** of the boot disk. The MBR is **512 bytes** in size and contains a small piece of boot code. BIOS loads this boot code into memory, and the boot code loads the **GRUB (Grand Unified Bootloader)**.

### UEFI Boot Process

In UEFI-based systems, UEFI does **not** read the MBR boot code. Instead, it looks for the **EFI System Partition (ESP)**, which is a dedicated FAT32 partition on the disk. Inside the ESP are executable **`.efi`** files, such as **`grubx64.efi`**. UEFI loads and executes this EFI bootloader, which is typically **GRUB**.

### GRUB (Grand Unified Bootloader)

GRUB is the Linux bootloader. Its primary responsibility is to locate and load the **Linux kernel** and the **initramfs (Initial RAM Filesystem)** into memory. GRUB can also display a boot menu, allow selection of different kernel versions, and pass boot parameters to the kernel.

Once the kernel and initramfs are loaded, GRUB's job is complete, and control is transferred to the Linux kernel.

### Linux Kernel

The Linux kernel initializes the operating system by:

* Initializing CPU scheduling
* Initializing memory management
* Loading built-in device drivers
* Detecting hardware
* Starting the **initramfs**

### initramfs (Initial RAM Filesystem)

The **initramfs** is a temporary root filesystem loaded into RAM. Its job is to load any additional drivers required to access the storage device, locate the real root filesystem, and mount it.

Once the real root filesystem (`/`) is mounted, initramfs hands over control back to the kernel.

### systemd (PID 1)

The kernel then starts **systemd**, which becomes **PID 1**, the first userspace process.

systemd is responsible for:

* Starting and managing system services (daemons)
* Starting services in the correct order based on dependencies
* Monitoring and restarting failed services when configured
* Managing the overall system state

It starts services such as:

* Networking
* SSH
* Logging (journald/rsyslog)
* Cron
* Docker
* Kubernetes services
* Application services

### systemd Targets

Finally, systemd reaches the configured target.

Common targets are:

* **multi-user.target** – Server mode (CLI), commonly used in production Linux servers.
* **graphical.target** – Desktop mode (GUI).

Once the target is reached:

* Networking is available.
* SSH service is running.
* Required applications and daemons are running.
* Users can log in locally or remotely.

At this point, the Linux boot process is complete.

---

## Boot Flow (Easy to Memorize)

```text
Power ON
    ↓
CPU starts
    ↓
BIOS / UEFI
    ↓
POST
    ↓
Initialize Storage Controller
    ↓
BIOS → MBR
or
UEFI → EFI System Partition (ESP)
    ↓
GRUB
    ↓
Kernel
    ↓
initramfs
    ↓
Mount Root Filesystem (/)
    ↓
systemd (PID 1)
    ↓
Start Services (Daemons)
    ↓
Reach Target
    ↓
System Ready for Login
```

