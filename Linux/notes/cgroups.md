cgroups (Control Groups)
A high-impact interview topic, especially in DevOps, SRE, and cloud-native roles (Docker, Kubernetes, etc.). You may get asked directly or be expected to understand resource limits behind containers or Linux servers.

1. What is a cgroup?
Control Groups (cgroups) is a Linux kernel feature used to control, monitor, and limit the resource usage (CPU, memory, disk I/O, network, etc.) of processes or groups of processes.

You can:

Limit how much memory a process uses

Restrict CPU cores or percentage

Monitor usage per group

Freeze, restart, or isolate processes

2. Why is it needed?
Prevent any one process from hogging all system resources

Improve system stability

Used by Docker and Kubernetes to isolate container resources

Manage multi-user systems or apps with fine control

3. cgroups vs nice/renice
Feature	nice/renice	cgroups
Controls	CPU priority only	CPU, Memory, IO, Network
Scope	Per-process	Groups of processes
Granularity	Coarse	Fine-grained and hierarchical
Persistent	No	Yes
4. cgroups Versions
cgroups v1: Older, widely supported.

cgroups v2: Unified hierarchy, cleaner, used by systemd, Docker, and modern Linux.

Check version:

bash
Copy
Edit
mount | grep cgroup
Or:

bash
Copy
Edit
cat /proc/filesystems | grep cgroup
5. How to Use cgroups (v1 – Manual)
Step 1: Create cgroup directory
bash
Copy
Edit
sudo mkdir /sys/fs/cgroup/memory/mygroup
Step 2: Set memory limit (e.g., 200MB)
bash
Copy
Edit
echo $((200*1024*1024)) > /sys/fs/cgroup/memory/mygroup/memory.limit_in_bytes
Step 3: Add a process (e.g., PID 1234)
bash
Copy
Edit
echo 1234 > /sys/fs/cgroup/memory/mygroup/tasks
6. systemd-based cgroups (Modern method)
With systemd, you can define resource limits in service units.

Example: Limit memory for a custom service
ini
Copy
Edit
[Service]
MemoryMax=200M
CPUQuota=50%
Place this in a custom unit (/etc/systemd/system/myapp.service) and run:

bash
Copy
Edit
systemctl daemon-reexec
systemctl start myapp
7. Monitor and Inspect
bash
Copy
Edit
cat /sys/fs/cgroup/memory/mygroup/memory.usage_in_bytes
bash
Copy
Edit
systemctl status myapp
Use tools:

systemd-cgls: tree of control groups

systemd-cgtop: live resource usage per cgroup

8. Interview Q&A
Q: What are cgroups and where are they used?

Kernel feature to control resource usage of processes. Used by Docker, Kubernetes, and systemd to isolate and limit resource usage.

Q: How are cgroups better than renice?

nice only sets CPU priority. cgroups can control memory, CPU %, I/O, etc., and apply it to groups of processes.

Q: How do you create a cgroup manually to limit memory?

Create a cgroup directory, set memory.limit_in_bytes, and echo the PID to tasks.