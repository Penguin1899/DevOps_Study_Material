## L2 - Handling Slowness/Issues in a Linux System
This notes goes deep into debugging issues with linux systems. It covers questions like "what next?", "what was the probable cause for this?" and "how to ensure we do not see this again"

---
### High %us: User Processes Hogging CPU
What to Look For:
- The %us metric represents the percentage of CPU time spent on user-level processes (non-kernel processes). If this is high, it indicates that user applications or scripts are consuming the CPU.

Typical indicators:
- High load averages.
- A particular process consistently consuming a high percentage of CPU, even during low traffic times.
- Multiple processes contributing to h#igh overall CPU usage.

What to Do When You See High %us:
- Identify the offending process
    - Use top or htop to identify which processes are consuming the CPU.
    - Run ps aux --sort=-%cpu to get a more detailed process list ordered by CPU consumption.

Probable causes and their fixes :

1. High Network Traffic

Cause: Network-intensive applications, services, or processes that are consuming CPU resources while handling network traffic (e.g., sending/receiving large amounts of data, parsing network responses).

**What to Do Next: Action Plan for High Network Traffic**</br>

Monitor network usage:
- Use iftop, nload, or netstat -tulnp to identify which processes or services are generating high network traffic.
- Check if network traffic aligns with spikes in CPU usage.

Inspect the application:
- Review application logs for unusual patterns, high request/response volume, or timeouts.
- If it’s a web server or API, check for high request rates and evaluate if these requests are causing processing delays.

Use strace to trace system calls:
- Run strace -p <pid> on a process that is using both high CPU and network resources to see if there are excessive network-related system calls (e.g., recvfrom, sendto).

Evaluate traffic patterns:
- Look for any unnecessary or inefficient network communications. Check for synchronous blocking calls or excessive retries.
- If the traffic is high due to large data transfers (e.g., file uploads/downloads), consider using compression or chunking to minimize load.

Optimize network operations:
- Implement asynchronous I/O to prevent blocking during network transfers.
- Use load balancing if high traffic is expected to distribute the load evenly.

If database-related:
- Ensure the application is making optimized database queries. Use pagination or query optimization to reduce the amount of data transferred.

2. High Memory Consumption

Cause: Applications or processes consuming excessive memory may trigger increased CPU usage due to paging/swapping or garbage collection (in languages like Java, Python, etc.). If memory limits are reached, the system might spend CPU cycles managing memory.

**What to Do Next: Action Plan for High Memory Consumption**</br>

Check memory usage:
- Use top, htop, or free -m to get an overview of system memory consumption.
- Look for processes using excessive memory (check the %MEM column in ps aux or top).

Analyze swapping:
- High memory usage can lead to swapping, where data is written to disk (slow). Check vmstat or iostat to see if the system is swapping.
- If swapping is happening, this will lead to a significant increase in CPU usage.

Identify memory leaks:
- Use tools like valgrind or heap dumps to find memory leaks in the application.
- Look for unusually high memory usage over time (this is a sign that memory is not being properly freed).

Check for inefficient memory usage:
- If your app or service is not releasing memory (e.g., improperly handled buffers, unclosed database connections), this can increase CPU usage as the system tries to manage resources.
- Profile the application for inefficient memory allocation patterns.

Action Plan:
- Optimize the application’s memory usage by fixing memory leaks, releasing resources when done, and using better memory management techniques.
- If the system is under heavy load, consider scaling vertically (more memory) or scaling horizontally (multiple instances of the service).

3. Inefficient Code or Algorithm

Cause: Poorly written or inefficient code, such as infinite loops, redundant computations, or memory-heavy operations, can consume excessive CPU. These inefficiencies can be amplified when the system is under load.

**What to Do Next: Action Plan for Inefficient Code**</br>

Profile the application:
- Use perf, gprof, or strace to profile the application and identify the exact areas where CPU is being consumed.
- Focus on functions or methods that consume a large percentage of CPU time.

Analyze the code for infinite loops:
- Ensure there are no infinite loops or unoptimized recursive functions.
- Look for places in the code where the logic may be inefficient (e.g., redundant calculations in a loop).

Optimize algorithms:
- Replace inefficient algorithms with more efficient alternatives (e.g., use binary search instead of linear search for large datasets).
- Optimize data structures (e.g., using hash tables for frequent lookups instead of lists).

Consider multi-threading:
- If CPU consumption is high due to sequential processing, consider parallelizing the tasks using multi-threading or worker queues.

Test for concurrency issues:
- If high CPU usage occurs during simultaneous requests (e.g., in web servers), check for thread synchronization issues or deadlocks in the application.

4. I/O Bound Processes (Disk or Network I/O)

Cause: Processes that are heavily dependent on disk or network I/O (e.g., reading/writing large files, waiting for data from a remote server) can cause high CPU usage. This is especially true when the system is waiting on resources, leading to high %wa (I/O wait), but the application also processes data once the I/O operation completes.

**What to Do Next: Action Plan for I/O Bound Processes**</br>

Check disk I/O:
- Use iostat, iotop, or dstat to identify disk I/O bottlenecks.
- High I/O wait can increase CPU usage because processes have to wait for I/O operations to complete.

Analyze disk and network performance:
- For disk issues, check for disk errors (e.g., with smartctl) and assess if the disk is the bottleneck.
- If network-related, use netstat, ss, or iftop to check for any network congestion or latency.

Optimize I/O operations:
- Implement asynchronous I/O for tasks that depend on disk or network.
- Use I/O schedulers (e.g., noop, deadline) to reduce I/O bottlenecks, especially in virtualized environments.
- For applications that require frequent reads, consider caching frequently accessed data to reduce I/O operations.

File system and storage optimization:
- Ensure that file systems are optimized (e.g., use ext4, XFS, or ZFS) for your workload.
- If applicable, migrate to SSD storage to improve read/write speeds.

5. External Factors (e.g., DDoS Attacks or Network Storms)

Cause: A sudden surge in external traffic, such as during a Distributed Denial-of-Service (DDoS) attack or network storms (e.g., broadcast packets), can overwhelm the system, leading to high %usr CPU usage as processes try to handle or respond to the incoming traffic.

**What to Do Next: Action Plan for External Factors**</br>
Analyze traffic patterns:
- Use netstat, iftop, or tcpdump to analyze incoming network traffic.
- Look for abnormal traffic spikes, especially from unusual sources or an unusually high number of packets from specific IP addresses.

Check for DDoS or malicious activity:
- If you suspect a DDoS attack, implement traffic filtering or rate limiting at the firewall or load balancer level.
- Use fail2ban or similar security tools to block IPs exhibiting suspicious behavior.

Limit or block unwanted traffic:
- Implement rate limiting to control the number of requests per second for certain services.
- Geo-block or IP block sources of malicious traffic if necessary.

Scale infrastructure:
- If under a heavy, legitimate traffic load, consider scaling horizontally (more servers) or using Content Delivery Networks (CDNs) to offload traffic.

Summary Action Plan for High %usr CPU:
- Monitor and analyze network traffic with tools like iftop or netstat to identify the processes contributing to high CPU and network usage.
- Check for memory-related issues using top, ps aux, or free and consider optimizing memory usage or increasing system resources.
- Profile the application to identify inefficiencies in code or algorithm design that could be contributing to CPU consumption.
- Optimize I/O operations if disk or network I/O is the bottleneck, using asynchronous I/O or tuning disk schedulers.
- Investigate external factors such as DDoS attacks or network storms and mitigate using appropriate security measures.


---
### High %sy: System/Kernel Processes
What to Look For:
%sy measures the CPU time spent in system-level operations, such as I/O operations, system calls, or kernel processes.

High %sy often indicates excessive kernel work, such as disk I/O, network processing, or interrupts.

What to Do When You See High %sy:
Check for excessive I/O:

Use iostat or iotop to check if high I/O operations are causing the system to spend time handling disk read/writes.

If disk I/O is a problem, check disk health with smartctl or dmesg for any hardware issues or errors.

Examine network traffic:

Use netstat or ss to see if the system is spending a lot of time handling network-related system calls. Look for network traffic spikes that could be overwhelming the system.

Check for system call bottlenecks:

Use strace to trace system calls of a particular process. Look for processes that are making excessive system calls (e.g., file accesses, socket operations) and determine if they're behaving as expected.

Look for kernel bugs or misconfigurations:

Sometimes, kernel bugs or misconfigurations in system parameters (e.g., vm.swappiness, vm.dirty_background_ratio) can cause high %sy.

Ensure that your system is up-to-date with the latest kernel patches.

Examine interrupt handling:

Use cat /proc/interrupts to see if certain hardware devices are generating high interrupt rates, causing the kernel to spend too much time handling interrupts. Network interfaces or storage controllers are often culprits here.

Prevention:
Tune I/O schedulers: Adjust I/O scheduler settings (e.g., use deadline or noop instead of cfq) for better performance under heavy load.

Network Offload: Use TCP offload engines (TOE) or kernel bypass techniques (e.g., DPDK) to reduce kernel networking overhead.

Kernel Updates: Keep the system kernel updated to avoid bugs related to system calls or interrupts.

---
3. High %wa (I/O Wait)
What to Look For:
%wa indicates the percentage of time the CPU spends waiting for I/O operations to complete (i.e., disk read/write, network I/O).

High %wa suggests that your CPU is idle, but waiting on a slow disk or other I/O subsystem to return data.

What to Do When You See High %wa:
Use iostat or iotop to analyze disk usage:

Check if there’s a specific disk or volume with unusually high I/O wait times.

Use iostat -x to get extended stats on devices, looking at await times (the average wait time for I/O operations to complete).

Check for disk bottlenecks:

Look at disk usage and health using smartctl. If a disk is reporting errors or a high number of reallocated sectors, it could be failing and causing delays.

Consider using faster storage options like SSDs if using traditional HDDs, or even RAID arrays to improve throughput.

Check network file systems (NFS):

If using NFS or other network-based storage, check network latency or server load. Use tools like nfsstat to get an overview of NFS performance.

Check the /etc/fstab configuration and make sure NFS mounts are properly optimized.

Evaluate database performance:

If the I/O wait is related to database operations (e.g., MySQL, PostgreSQL), check if queries are causing heavy disk reads/writes. Use database logs to look for slow queries or use EXPLAIN to identify poorly optimized queries.

Optimize I/O throughput:

If possible, use RAID (0, 1, 10) or other volume managers like LVM to spread I/O load.

Consider tuning filesystem parameters (e.g., adjusting noatime, nodiratime flags) for lower disk write overhead.

Prevention:
Use faster storage solutions: If the application can tolerate it, upgrade to SSDs for faster disk access.

Optimize disk usage patterns: Minimize unnecessary disk writes and prioritize read-heavy or in-memory operations.

Regular disk health checks: Implement a proactive disk health monitoring system (e.g., using smartmontools).

---
4. Zombie Processes or D (Uninterruptible Sleep) States
What to Look For:
Zombie processes: These are processes that have completed execution but still have an entry in the process table, typically because the parent process hasn’t read their exit status (e.g., due to a bug in the parent process).

D state (uninterruptible sleep): Indicates that a process is stuck waiting for a resource, usually related to I/O (e.g., waiting on disk or network response).

What to Do When You See Zombies or D States:
Identify zombie processes:

Use ps aux | grep Z or top to spot processes in a Z (zombie) state. These processes don’t consume CPU but can pile up if not cleaned up by the parent process.

If you have a large number of zombies, check the parent process (PPID) to determine which parent is not reaping its children.

Investigate uninterruptible sleep (D state):

Check which processes are in a D state using top or ps -eo pid,stat,cmd | grep ^D. These are often stuck on I/O waits, particularly related to disk or network operations.

If these processes are related to disk access, consider checking disk health or filesystem status. Look at the I/O subsystem for hardware issues.

Force termination if necessary:

If the zombie process’s parent is not able to reap it, and the process is stuck indefinitely, kill the parent process (if safe), or reboot the machine as a last resort to clean up zombie processes.

For processes stuck in D state, determine if they can be safely killed using kill -9 <pid>. If the process can’t be killed, it may require a system reboot or more in-depth investigation into the kernel’s handling of I/O.

Prevention:
Proper parent-child handling: Ensure that applications properly handle child processes and clean up after them to avoid zombies.

Improve process responsiveness: If stuck processes are frequent, consider reviewing how the system handles blocking operations or adjust timeouts in I/O-bound services.

Summary of Actions:
High %us: Profile and optimize application code, or scale horizontally.

High %sy: Investigate kernel-level issues (e.g., excessive I/O), consider tuning system settings, or update the kernel.

High %wa: Investigate disk/network I/O bottlenecks and optimize storage or improve system resources.

Zombie/D State: Clean up zombie processes or investigate hardware issues causing uninterruptible sleep states.

