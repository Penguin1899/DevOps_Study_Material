## L1 - Handling Slowness/Issues in a Linux System
This is a top interview question, especially for senior roles. 

It tests your ability to diagnose, analyze, and fix system performance issues.

> [!NOTE]
> This notes only covers L1 debugging (commands overview)
> For more detailed triubleshooting, see L2 notes

---
### First Steps: Basic Health Check
1. System Uptime and Load Average
```
uptime
```
Output:
```
14:23:08 up 2 days,  4:32,  2 users,  load average: 1.32, 1.01, 0.76
```
Interpretation:
- Load average shows CPU load over 1, 5, and 15 mins.
- A load of 1.00 means 100% of the CPU being used in \<internal\>.
- For multicore systems, it's relative (e.g., 4.00 on 4 cores = fully utilized).

---
### Identify the Bottleneck
1. **CPU-Related Slowness**
Tools:
- `top` or `htop`: real-time view of CPU usage.
- `pidstat`: process-level breakdown.
- `mpstat`: per-core stats.

What to Look For:
- High %us: user processes hogging CPU.
- High %sy: system/kernel processes.
- High %wa (%iowait): CPU waiting for IO → may be disk issue.
- Check for zombie processes, or D (uninterruptible sleep) states.

Example Outputs:
- top
```
top - 17:13:15 up 1 day,  3:20,  2 users,  load average: 0.15, 0.21, 0.18
Tasks: 201 total,   1 running, 198 sleeping,   2 stopped,   0 zombie
%Cpu(s):  2.3 us,  0.7 sy,  0.0 ni, 96.8 id,  0.2 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7864.1 total,   3254.2 free,   2876.5 used,   1733.4 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.  4680.1 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 2345 user1     20   0  1567m  45676  23456 S   1.2  0.6   0:12.56 firefox
 1234 root      20   0   876m  12345   8765 S   0.5  0.2   0:05.89 systemd
 5678 user2     20   0   234m   5678   3456 S   0.1  0.1   0:01.23 gnome-terminal-
 9012 root      -2   0   123m   2345   1234 S   0.0  0.0   0:00.45 cron
 3456 user1     20   0  4567m  87654  34567 R  98.7  1.1   2:34.78 intensive_task
 7890 root      20   0   567m   1234    876 S   0.0  0.0   0:00.08 sshd
 ... (more processes)
```

Explanation of the Fields:
1. Top Line (System Uptime and Load Averages):
```
top - 17:13:15: Current system time (hours:minutes:seconds).
up 1 day, 3:20: System uptime, indicating how long the system has been running (1 day and 3 hours and 20 minutes in this case).
2 users: Number of users currently logged in to the system.

load average: 0.15, 0.21, 0.18: System load average over the last 1, 5, and 15 minutes, respectively. 
This indicates how busy the system has been. 
A load average of 1.0 means the system is fully utilized by the CPUs. Values above the number of CPUs might indicate a backlog of processes waiting for CPU time.
```

2. Tasks Line (Process Summary):
```
Tasks: 201 total: Total number of processes currently being managed by the kernel.
1 running: Number of processes currently in a runnable state (actively using the CPU or waiting for a CPU).
198 sleeping: Number of processes that are currently idle, waiting for an event to occur (e.g., I/O completion, signal).
2 stopped: Number of processes that have been stopped, usually by a signal from the user (e.g., using Ctrl+Z).
0 zombie: Number of zombie processes. These are processes that have finished execution but their parent process hasn't yet collected their exit status. A high number of zombie processes can indicate a problem with parent processes.
```

3. %Cpu(s) Line (CPU Utilization):
```
%Cpu(s):: Indicates the breakdown of CPU usage across different categories.
2.3 us: Percentage of CPU time spent running user-space processes (your applications).
0.7 sy: Percentage of CPU time spent running kernel-space processes (system calls, kernel tasks).
0.0 ni: Percentage of CPU time spent running user-space processes with a nice priority (manually adjusted priority). A positive nice value lowers priority, a negative value increases it.
96.8 id: Percentage of CPU time that is idle (not doing any work).
0.2 wa: Percentage of CPU time spent waiting for I/O operations to complete (e.g., disk access). High wa often indicates I/O bottlenecks.
0.0 hi: Percentage of CPU time spent servicing hardware interrupts.
0.0 si: Percentage of CPU time spent servicing software interrupts.
0.0 st: Percentage of CPU time stolen by the hypervisor from this virtual machine (if running in a virtualized environment).
```

4. MiB Mem Line (Memory Usage):
```
MiB Mem :: Indicates memory usage in megabytes.
7864.1 total: Total amount of physical RAM installed in the system.
3254.2 free: Amount of physical RAM that is currently unused.
2876.5 used: Amount of physical RAM that is currently being used by processes.
1733.4 buff/cache: Amount of physical RAM being used for buffers (for block devices) and cache (for frequently accessed files). This memory can be quickly reclaimed by applications if needed.
avail Mem: (Sometimes shown instead of buff/cache) An estimate of how much memory is available for starting new applications without the system having to resort to swapping. It takes into account free memory and reclaimable buff/cache.
```

5. MiB Swap Line (Swap Space Usage):
```
MiB Swap:: Indicates swap space usage in megabytes. Swap space is disk space used as virtual RAM when physical RAM is full.
2048.0 total: Total amount of swap space configured on the system.
2048.0 free: Amount of swap space that is currently unused.
0.0 used: Amount of swap space that is currently being used. High swap usage can indicate memory pressure and can significantly slow down the system.
```

6. Process List (Information about Individual Processes):
```
Each row in this section represents a running process. The columns provide various details about each process:

PID: Process ID, a unique numerical identifier assigned to each running process.
USER: The username of the owner of the process.
PR: Priority of the process. Lower numbers generally indicate higher priority (real-time and negative values).
NI: Nice value of the process. This is a user-space priority adjustment. Negative values increase priority, positive values decrease it.
VIRT: Virtual memory size of the process, including all code, data, and shared libraries that the process has access to, even if it's not all in RAM.
RES: Resident memory size of the process, the actual amount of physical RAM the process is currently using.
SHR: Shared memory size, the amount of memory shared with other processes.
S: Process status:
S: Sleeping
R: Running
D: Uninterruptible sleep (usually waiting for I/O)
T: Stopped (e.g., by a signal) or traced
Z: Zombie
%CPU: Percentage of CPU time the process is currently using. This is an instantaneous value.
%MEM: Percentage of physical RAM the process is currently using.
TIME+: Total CPU time the process has accumulated since it started, in the format minutes:seconds.hundredths.
COMMAND: The command line used to start the process.
```

This detailed output provides a snapshot of the system's resource usage and the processes that are consuming those resources. It's a valuable tool for monitoring system performance and identifying potential bottlenecks.


---
2. **Memory Pressure**
Tools:
- free -h
- vmstat
- top, htop
- cat /proc/meminfo

Example Outputs:
- `free -m`
```
              total        used        free      shared  buff/cache   available
Mem:           7864        2900        3100         100        1864        4664
Swap:          2048           0        2048
```

Explanation of fields:

- The free -m command displays information about the system's memory usage in megabytes. Here's a breakdown of each column:
- Row 1: Mem (Physical Memory - RAM)
```
total: The total amount of physical RAM installed in the system (7864 MB in this example).

used: The amount of physical RAM that is currently being used by running processes and the operating system (2900 MB). This includes memory used by applications, kernel, and also the buff/cache.

free: The amount of physical RAM that is completely unused and readily available for allocation to new processes (3100 MB).

shared: The amount of memory used by the tmpfs file system, which is a temporary file system that resides in RAM. This memory can be shared between multiple processes (100 MB).

buff/cache: The amount of physical RAM being used for kernel buffers (to speed up access to block devices like disks) and page cache (to cache frequently accessed files from disk). This memory can be reclaimed by applications if needed, so it contributes to the available memory (1864 MB).

available: An estimate of how much memory is available for starting new applications without the system needing to resort to swap. It is calculated by adding the free memory to a portion of the buff/cache that the kernel deems can be easily reclaimed (4664 MB). This is often the most important value to look at to determine if your system is running low on memory.
```
- Row 2: Swap (Swap Space)
```
total: The total amount of swap space configured on the system (2048 MB). Swap space is disk space used as virtual RAM when the physical RAM is full.

used: The amount of swap space that is currently being used (0 MB). If this value is consistently high, it indicates that the system is under memory pressure and is swapping data to disk, which can significantly slow down performance.

free: The amount of swap space that is currently unused and available (2048 MB).
In summary, free -m provides a human-readable overview of your system's memory usage, helping you understand how much RAM is being used, how much is free, and how much is available for new applications. The available column is a good indicator of the actual memory available for new tasks.
```

Symptoms:
- Swap usage increasing.
- Low free memory.

High cache/buffer size: not bad – Linux uses free RAM for caching.
```
# Check swap usage
swapon --show
```

Clear PageCache (be careful):
```
sync; echo 3 > /proc/sys/vm/drop_caches
```

---
3. **Disk/IO Bottlenecks**
Tools:
- `iotop`
- `iostat`(from sysstat)
- `dstat`
- `df -hT`: disk usage and types
- `du -sh /path/*`: find large files (du -d2 -h | sort -h = more human readable with depth=1)

Symptoms:
- High IO wait times.
- Disk near full or full.
- Frequent read/write spikes.

---
4. **High Network Usage**
Tools:
- `iftop`
- `ip -s link`
- `ss -tuln (check open ports and listeners)`
- `netstat -i`
- `nload, bmon`

Symptoms:
- High bandwidth usage.
- Latency spikes.
- Network-facing services (like web servers) may slow down.

---
### Checking for Specific Resource Hogs
We can do this via :
- top/htop
    - Sort by %CPU, %MEM
    - Look for zombie (Z) or defunct processes.
    - Highlight processes stuck in D state (blocked IO).
- ps command for detailed view
```
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head
```

---
### Logs and Audit
Check syslog and dmesg
```
dmesg | tail
journalctl -p err..alert
```

Check for Kernel or Disk Errors
```
dmesg | grep -i error
journalctl -k
```

---
## Interview Examples:

- **Question**: What are the top tools you use to debug Linux performance issues?
    - CPU: top, htop, pidstat
    - Memory: free, vmstat
    - Disk: iotop, iostat, df
    - Network: iftop, ss, nload
    - Logs: dmesg, journalctl, /var/log/syslog

---
- **Question**: How do you interpret load average?

Load average is the number of processes waiting for CPU.
- 1.0 = 100% usage on single-core
- 2.0 = overloaded on single-core
- On multi-core, divide by number of cores

---
- **Question**: What if swap is being heavily used?

Indicates memory pressure.
- Check what's eating RAM: top, ps, smem
- Kill memory-heavy processes
- Add more RAM or tune swapiness

---
- **Question**: System is slow, high IO wait – what do you do?

Run iotop, check which processes are reading/writing heavily.
- Use iostat to check disk latency.
- If disk is full, use du to locate large files and clean up.
- Consider SSDs, or adding more IO bandwidth.