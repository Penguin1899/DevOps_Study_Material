## systemd & systemctl (Managing Services)

This is fundamental. Every modern Linux distribution (like RHEL 7+, Ubuntu 16.04+, CentOS 7+) uses systemd.

1. What is systemd?
systemd is the init system and service manager used in modern Linux. It is responsible for:
- Booting the system
- Starting/stopping services
- Managing dependencies
- Mounting filesystems
- Handling logging (via journald)
- It replaces older systems like SysVinit and Upstart.

2. Why is systemd Needed?
- Faster boot times through parallel service startup
- Unified management: services, mounts, timers, logging — all via systemctl
- Better dependency handling
- Used by default on most major distros now

3. Key systemd Components
- Unit files: define services, targets, sockets, devices, etc.
- systemctl: CLI to control systemd
- journald: centralized logging mechanism
- targets: like runlevels (e.g., multi-user.target)

4. Managing Services with systemctl
Start/Stop/Enable/Disable Services
```
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

sudo systemctl enable nginx    # Enable at boot
sudo systemctl disable nginx   # Don't start at boot
```

- Check Status
```
systemctl status nginx
```
- List All Services
```
systemctl list-units --type=service
```

5. Create a Custom Service

Create /etc/systemd/system/myapp.service:
```
[Unit]
Description=My App Service
After=network.target

[Service]
ExecStart=/usr/local/bin/myapp.sh
Restart=always
User=nobody

[Install]
WantedBy=multi-user.target
```

Enable & start:
```
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
```

6. Runlevels vs Targets
- Target / runlevel is a desired state your machine would reach. To reach this state/target, there are a bunch of services that need to come up
- Technically target is defined as - Targets in systemd are grouping mechanisms for other units
- There is a default target (or runlevel in older systems) set in any machine. This defines all services that will be started on boot to reach this target
- By default it is target 3/5 in modern systems
- 3=multi-user.target -> a state of the system in which multiple users can interact with it
- 5=gui target -> basically target3 + GUI services
    - To reach this target, the system will start a bunch of services on boot
    - Each custom service is defined in a unit file under `/etc/systemd/system/` and system defined service files are under `/lib/systemd/system`
    - Each unit file (system defined or custom services) mentions a set of pre-deps via `After` directive which basically tells the system to run this service "after" a particular stage has been done
    - Also, we mention a `Install` section with `WantedBy` directive in the unit file. This basically maps the service to a particular target. This basically implies that when the system reaches this particular target, this service would be running. (Not mentioning this will map the service to the default target in the system)
    - When we map a service to a target using the "Wants" directive, we are basically saying that this target "wants" this service to run.
    - Enabling such a custom service would create a symlink to the target's want directory. This is how the target knows to start this service. (Explained more in Q&A)

> [!IMPORTANT]
> Read in depth about creating custom service files in next section of this notes

| Traditional Runlevel | systemd Target      |
|----------------------|---------------------|
| 0                    | poweroff.target     |
| 1                    | rescue.target       |
| 3                    | multi-user.target   |
| 5                    | graphical.target    |
| 6                    | reboot.target       |

- Check default target:
```
systemctl get-default
```

Change it:
```
sudo systemctl set-default multi-user.target
```
> [!NOTE]
> by changing target is one way to enter single-user mode in linux

7. Viewing Boot Time Logs
```
journalctl -b          # logs from last boot
journalctl -xe         # recent errors
journalctl -u nginx    # logs for specific service
```

## Custom systemd Service File (Explained in Detail)
Creating your own systemd service gives you full control over when and how a script or binary runs in the background — especially on boot.

1. What is a systemd service file?

A unit file (with .service extension) is a configuration file that tells systemd:
- What program to start
- When to start it
- Under which user
- What to do if it fails
- What other services it depends on

It replaces older SysVinit-style shell scripts in `/etc/init.d/`

2. Where to Place It?

User-defined services should be placed in:
```
/etc/systemd/system/
```
> [!NOTE] 
> Never place your custom service directly under /lib/systemd/system/ — that's for distro-provided services.

3. Anatomy of a Service File

Here’s a working example of a custom service:
```
[Unit]
Description=My Custom App
After=network.target

[Service]
ExecStart=/usr/local/bin/myapp.sh
ExecStop=/usr/local/bin/stop_myapp.sh
Restart=always
User=nobody
Environment=APP_ENV=production

[Install]
WantedBy=multi-user.target
Section-by-Section Breakdown
```


[Unit] Section

| Directive   | Description                                                                    |
|-------------|--------------------------------------------------------------------------------|
| Description | Short text shown in systemctl                                                  |
| After       | Start only after certain target is reached -> network.target in this case (wait for network)                 |
| Requires    | Ensures other services are started too; if those fail, this does too          |
| Wants       | Like Requires but doesn't fail if dependent service fails                      |

[Service] Section

| Directive        | Description                                                        |
|------------------|--------------------------------------------------------------------|
| ExecStart        | Full path to the command or script to start                       |
| ExecStop         | Optional — command to stop the service                           |
| Restart          | What to do if it crashes (no, on-failure, always)                |
| User             | Which Linux user to run it as (e.g., nobody, www-data)           |
| WorkingDirectory | Optional — run from this folder                                    |
| Environment      | Set environment variables                                          |
| Type             | Default is simple; can be forking, oneshot, notify, etc.          |

[Install] Section

| Directive | Description                                                              |
|-----------|--------------------------------------------------------------------------|
| WantedBy  | What target this service hooks into (multi-user is like runlevel 3) |


4. How to Register and Use the Service
```
# Reload to recognize new service
sudo systemctl daemon-reload

# Enable service to run at boot
sudo systemctl enable myapp.service

# Start service now
sudo systemctl start myapp.service

# View status and logs
systemctl status myapp.service
journalctl -u myapp.service
```

5. Debugging Common Issues
- Permission denied: Check ExecStart path and User
- Script fails silently: Add logging inside your script or set StandardOutput=journal
- Not starting on boot: Did you enable the service? Did you reload daemon?


---
## Interview Examples:

- **Question**: What is systemd, and how does it differ from SysVinit?

systemd is the modern init system. It starts services in parallel, uses unit files instead of init scripts, and has built-in logging. It replaces older, slower SysVinit scripts.

---
- **Question**: How do you write and enable a systemd service?

Create a unit file in /etc/systemd/system, define [Unit], [Service], and [Install] sections, run systemctl daemon-reload, then enable and start it.

---
- **Question**: How can you view logs of a service?
```
journalctl -u servicename
```

---
- **Question**: How does a target know what all service unit files to run when the system is reaching the target?

When you enable a service using systemctl enable <your_service.service>, and that service file has a WantedBy directive (e.g., WantedBy=multi-user.target), systemd does the following:

- Creates a .wants directory (if it doesn't already exist) within the directory associated with the target specified in WantedBy. For multi-user.target, this directory would be /etc/systemd/system/multi-user.target.wants/.
- Creates a symbolic link inside this .wants directory that points back to your service unit file. The name of the symlink is usually the name of your service file.

Example:

If your service file is named my_app.service and it contains 
```
WantedBy=multi-user.target
```
in the [Install] section, after you run sudo systemctl enable my_app.service, you will likely find a symlink like this:
- `etc/systemd/system/multi-user.target.wants/my_app.service -> /etc/systemd/system/my_app.service`
    - (The actual path on your system might vary slightly depending on where you placed your service file).

What the Symlink Does:
- This symbolic link tells systemd that when the multi-user.target is activated during the boot process (or manually), it should also start any services that have a symlink in its .wants directory. In essence, it creates a dependency: your service is "wanted by" the multi-user.target.

Disabling the Service:
- When you disable the service using systemctl disable <your_service.service>, systemd removes this symbolic link from the .wants directory. This means that when the target unit is activated in the future, your service will no longer be automatically started as it's no longer "wanted" by that target.

In Summary:
- The WantedBy directive doesn't directly start the service. Instead, it instructs systemctl enable to create a symbolic link in the .wants directory of the specified target. This symlink is what ultimately causes your service to be started when that target is reached during system startup or target activation. You are correct that it primarily functions through the creation and removal of these symbolic links. 


---
- **Question**: Where are logs for a custom systemd service stored?

By default, systemd captures all stdout and stderr from services and sends them to the journal, which is managed by journald.

So logs are not stored in a traditional log file like /var/log/myapp.log unless you explicitly redirect them.

---
- **Question**: How to View Logs for a Custom Service

Use journalctl:
```
journalctl -u myapp.service
```

To view live logs (like tail -f):
```
journalctl -u myapp.service -f
```

For specific boots or timestamps:
```
journalctl -u myapp.service --since "30 min ago"
```

---
- **Question**: How to Log to a File Instead (Optional)

If your service script writes logs to a file:
```
#!/bin/bash
echo "Starting myapp..." >> /var/log/myapp.log
```
> Make sure the service has permission to write to that path, especially if it runs as nobody or another limited user.

Redirect Output in the Service File (Optional)
If you want to redirect stdout/stderr to specific files, use:

```
[Service]
ExecStart=/usr/local/bin/myapp.sh
StandardOutput=append:/var/log/myapp.out
StandardError=append:/var/log/myapp.err
```

Other options:
journal (default)
null (discard)
tty (write to terminal)

---
- **Question**: Where Does journald Store Logs?

By default, In binary format under:
```
/var/log/journal/
```

To view or query: always use journalctl.
> If persistent logging isn’t enabled, logs will be stored in-memory and lost on reboot.

Enable persistent logging:
```
mkdir -p /var/log/journal
systemd-tmpfiles --create --prefix /var/log/journal
systemctl restart systemd-journald
```
