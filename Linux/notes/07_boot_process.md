## Boot Process & Services in Linux
This is a very commonly asked topic in Linux interviews, especially for roles involving system administration, DevOps, or troubleshooting.

### Overview of the Linux Boot Process
When a Linux system boots up, it follows several stages. Understanding each step helps you troubleshoot when something goes wrong.

---
1. BIOS/UEFI (UEFI is modern and secure, faster with more GUI options)
- Basic hardware initialization.
- Finds the bootable device and hands over control to the bootloader.

---
2. Bootloader (GRUB -> now GRUB2)
- GRUB (GRand Unified Bootloader) is the most common bootloader.
- It loads the Linux kernel into memory.
- You can choose between multiple kernels or OS versions here.

Config files:
- /etc/default/grub -> default config
    - /boot/grub/grub.cfg -> BIOS systems
    - / -> UEFI systems

---
3. Kernel
- Kernel is the core of the OS: manages hardware, memory, I/O, CPU.
- Loads essential drivers and mounts the root filesystem (/).

You can see the kernel boot logs via:
```
dmesg
journalctl -k
```

---
4. init / systemd
- The init system takes over after the kernel.
    - Systemd is the modern, default init system on most distros (replaces older SysVinit).
    - It brings up all user-space services and targets.
    - It boots into default target
        - use `systemctl get-default --type=target` to view default target
        - use `systemctl set-default` to change default target
> [!IMPORTANT]
> Systemd uses targets instead of runlevels.
> Targets = groups of services (e.g., graphical.target, multi-user.target).

5. Login Prompt (TTY or GUI)
- Once all services are up, system presents:
    - GUI Login (GDM/LightDM etc.)
    - or CLI prompt (TTY) based on runlevel/target

---
### Systemd: Managing Services and Boot Targets
Systemd controls everything from service startup to log management.

Check Boot Time & Breakdown
```
systemd-analyze
systemd-analyze blame      # See what delayed boot
```

Working with Services
Start service:

```
systemctl start nginx
```

Enable on boot:
```
systemctl enable nginx
```

Stop/Disable:

```
systemctl stop nginx
systemctl disable nginx
```

Status check:

```
systemctl status nginx
```

List All Services
```
systemctl list-units --type=service
```

View Logs (Journal)
```
journalctl -u nginx      # logs for a unit
journalctl -b            # logs since last boot
```

---
### Runlevels vs Targets
| SysV Runlevel | Systemd Target     | Description        |
|---------------|--------------------|--------------------|
| 0             | poweroff.target    | Shutdown           |
| 1             | rescue.target      | Single-user        |
| 3             | multi-user.target  | CLI, no GUI        |
| 5             | graphical.target   | GUI login          |
| 6             | reboot.target      | Reboot             |

- Switch target:
```
systemctl isolate multi-user.target
```

- Set default target:
```
systemctl get-default
systemctl set-default graphical.target
```
> More details of what this is will be clear after revising systemd services, units and Wants

---
### Troubleshooting Boot Issues
1. Drop to Emergency/Rescue Shell
Happens if system fails to mount root, or if fstab is misconfigured.

Use:
```
systemctl default        # Try to resume normal
journalctl -xb           # Check logs
```

2. Edit GRUB at Boot
Press e at GRUB screen.

Modify kernel options (e.g., add single for single-user mode).

Useful for:
- Resetting root password
- Skipping fsck
- Booting with different init

---
## Interview Examples:

- **Question**: Describe the Linux boot process.

BIOS → GRUB → Kernel → init/systemd → Login prompt.

---
- **Question**: What is systemd? Why is it used?

Modern init system that manages services, logging, targets, dependencies. Fast and parallel booting.

---
- **Question**: How to troubleshoot slow boot?
```
systemd-analyze
systemd-analyze blame
journalctl -b
```

---
- **Question**: Difference between rescue.target and emergency.target?

| Target           | rescue.target          | emergency.target       |
|------------------|------------------------|------------------------|
| Services         | Minimal services       | Bare minimum, no services |
| Filesystems      | Root is mounted        | Root may be read-only  |
| Use case         | Fix system issues      | Deep system rescue     |

---
- **Question**: What if a service fails at boot?
    - Check logs: journalctl -xe or journalctl -u service_name
    - Try restarting: systemctl restart
    - Mask it if it’s blocking boot: systemctl mask service_name