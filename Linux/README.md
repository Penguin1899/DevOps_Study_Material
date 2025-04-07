Core Topics (Must-Know for All Levels)
1. Users & Permissions
chmod, chown, umask

File permission types: rwx, sticky bit, SUID, SGID

/etc/passwd, /etc/shadow, /etc/group

2. Processes & Resource Management
ps, top, htop, nice/renice, kill, pgrep

Zombie, orphan, defunct processes

strace, lsof, iotop, vmstat, sar

3. Disk & Filesystem
df, du, mount, umount

lsblk, blkid, fdisk, parted

inodes, file descriptors

ext4, xfs, nfs, etc.

4. System Boot & Services
Boot stages: BIOS → GRUB → init/systemd

systemctl, journalctl

Service troubleshooting: failed, inactive, masked

5. Networking
ip a, ss, netstat, ping, traceroute

Ports, sockets, listening processes

DNS resolution, /etc/hosts, /etc/resolv.conf

Intermediate to Advanced Topics
6. Performance Troubleshooting (Slowness Issues)
CPU: top, pidstat, mpstat

RAM: free -m, vmstat, top

IO: iotop, iostat, dstat

Network: iftop, nload, ss -tulnp

Tools: perf, bpftrace, sar

How to triage: CPU bound? Memory leak? Disk IO? Network bottleneck?

7. Storage & Inodes
Inode exhaustion vs disk full

Check with df -i

Clean-up strategies when inode usage is high (e.g., lots of tiny files in /var/log)

8. cgroups & namespaces (Container-related)
Resource limiting (CPU, memory) using cgroups

Process isolation via namespaces

Basis for containers (Docker, Podman)

Expect high-level questions like:
"How does Docker isolate resources?" or "What's the role of cgroups in Kubernetes?"

9. Logs & Auditing
journalctl, /var/log/syslog, /var/log/messages

auth.log, secure, dmesg

auditd, SELinux/AppArmor logs

10. Crons & Scheduled Jobs
crontab -e, system-wide crons

at, anacron

Log location for cron jobs

Common cron issues (PATH, permissions)

11. Package Management
apt, yum, dnf, rpm

dpkg -l, rpm -qa, which, whereis

Repository configs

Advanced/Niche Topics (For Senior/DevOps Roles)
12. SELinux / AppArmor
Security context

Enforcing vs permissive

Troubleshooting denials

13. System Tuning & Limits
/etc/security/limits.conf

ulimit, sysctl, kernel parameters

swappiness, open files, TCP tuning

14. Kernel & Modules
View kernel version: uname -a

lsmod, modprobe, dmesg

Rebuilding/initramfs (rare, but for infra roles)

15. Automation Tools
Shell scripting basics

Basic knowledge of ansible, puppet, chef (interview bonus)

Extra Topics (If Time Permits)
Signal handling in shell

LVM (Logical Volume Management)

RAID concepts

SSH key management, ssh-agent

System hardening basics

Troubleshooting failed boots (rescue mode, init=/bin/bash)

Network mount issues (NFS, CIFS)

Backup strategies (rsync, tar, snapshots)