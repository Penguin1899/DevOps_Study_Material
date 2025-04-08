## Understanding Processes
Everything running on a Linux system is a process, with a unique PID (Process ID).
The first process is init (or systemd in all modern systems), has PID 1.
Processes can be:
- Foreground / Background
- Parent / Child
- Interactive / Daemon

## What is "Process Management" in Linux?
In interviews, this refers to understanding, inspecting, controlling, and manipulating running processes on a Linux system.

It includes:

### Viewing Processes
- `ps aux` – Snapshot of all running processes.
- `ps -ef` – Another format with full command line.
- `top / htop` – Dynamic, real-time view (like Task Manager).
- `pgrep` – Find process IDs by name.
- `pidof` – Get PID of a specific command.
- `pstree` – Shows process hierarchy (like a family tree).

> Interview Note: Know the difference between ps and top, and when to use each.

### Understanding Process Attributes
Key columns in ps or top:
- PID – Process ID.
- PPID – Parent Process ID.
- USER – Who owns it.
- %CPU / %MEM – Resource usage.
- TTY – Terminal it’s attached to.
- STAT – Process state (see below).

### STAT values to know (Process States)
- R = Running
- S = Sleeping (waiting for input)
- D = Uninterruptible sleep (usually IO)
- Z = Zombie (terminated, but parent hasn’t cleaned it up)
- T = Stopped

### Killing Processes
- `kill PID` – Send signal (default is SIGTERM, 15).
- `kill -9 PID` – Force kill (SIGKILL).
- `pkill` – Kill by name.
- `killall` – Kill all of a certain process.

> Interview Tip: Know the difference between kill, kill -9, and pkill.

#### Bonus
- nice / renice – Adjust process priority.
- nohup – Run process immune to hangups (terminal close/ssh session close).
- disown – Remove from shell job table.

### Foreground & Background Jobs
- & – Run in background (simulates a fork() systemcall).
- jobs – List current shell jobs.
- fg – Bring job to foreground.
- bg – Resume a stopped job in background.
- Ctrl+Z – Pause a job (sends SIGSTOP).

---
### Signals
Processes can receive signals:
- SIGTERM (15): Graceful exit
- SIGKILL (9): Force quit
- SIGSTOP (19): Pause
- SIGCONT (18): Resume
- SIGHUP (1): Terminal hangup, often used to reload configs

Use:
```
kill -SIGSTOP 1234
kill -SIGCONT 1234
```

### Daemons and Init Systems
Daemon = background process, often system-related.
Managed via systemd (on most modern systems).
```
systemctl start nginx
systemctl status nginx
journalctl -xe # for logs
```


## Deep dive into concepts
<details>
<summary>zombie processes</summary>
A zombie is a process that has completed execution but still has an entry in the process table because its parent hasn’t acknowledged (reaped) its exit status.

Why this happens:
- When a child dies, the kernel keeps its exit code until the parent calls wait().
- If the parent forgets or crashes, the child becomes a zombie.

How to detect zombies:
```
ps aux | grep 'Z'
ps -el | grep Z
```
Look for STAT = Z (Zombie).

Example:
Make a quick zombie manually:
```
# Run a subshell to fork a background child that exits quickly
bash -c 'sleep 0.1 & echo "child PID: $!"; sleep 1000'
```

In another terminal:
```
ps -el | grep Z
```
You'll see the zombie briefly.

Fix?
- Usually, restart the parent process.
- Or the zombie is reaped by init (PID 1) if the parent dies.

Why are zombie processes bad?
- They consume process table entries.Every process in Linux takes up a slot in the kernel's process table.
- This table is finite—usually tens of thousands.
- If you have too many zombies, you can exhaust the table and block new processes from starting, even for root.

Symptoms in such a case:
- Can't open a new shell.
- fork() or exec() errors in apps.
- Daemons fail to restart.

They're signs of buggy parent processes
- A zombie means the parent didn’t call wait() properly.
- This often signals a programming bug in the parent process.
- For long-running services (e.g., daemons or web servers), this may indicate poor child process management, leading to eventual memory or resource leaks.

They clutter monitoring and logs
- When you're debugging system behavior, zombies can confuse monitoring tools, making it harder to get a clean view of running processes.
- Some automated alert systems (like Nagios, Zabbix) will throw warnings on excessive zombies.

In short:
- One or two zombies = fine, harmless.
- Dozens or hundreds = symptom of a deeper issue, can cause system instability.
</details>

<details>
<summary>nice / renice</summary>
These control the CPU scheduling priority of a process.

nice – Set priority when starting a process.
Range: -20 (highest priority) to 19 (lowest priority).

Default is 0.
```
# Run a command with lower priority
nice -n 10 ./long_script.sh
```

renice – Change priority of an existing process.
```
# Increase priority of process with PID 1234
sudo renice -n -5 -p 1234
```

You’ll need sudo to increase priority (set a lower nice value).

Interview Tip: Mention that this doesn’t "speed up" the process directly—it just gives it more CPU share when there’s competition.
</details>

<details>
<summary>nohup</summary>
Used to run a command immune to hangups, like closing a terminal.

Use Case:
You're running something long (like a backup) and want to log out.
```
nohup ./long_running.sh &
```
Keeps running even after logout.

Output goes to nohup.out by default.

Bonus Tip:
You can redirect output:
```
nohup ./script.sh > output.log 2>&1 &
```
Interview Angle: Mention it’s great for persistent scripts when systemd isn't used.
</details>

<details>
<summary>disown</summary>
Used to detach a job from the shell, so it won’t receive signals like SIGHUP.

Use Case:
You started a job without nohup, and now you want to log out without killing it.

```
./long.sh &
jobs            # See job ID
disown %1       # Remove job from shell
```
Now even if you close the terminal, it won't die.

You can also use:
```
disown -h %1
```
Which removes the job but leaves it on the process list (less common).
</details>

<details>
<summary>fg / bg</summary>
Control foreground and background jobs in your current shell session.

Common Flow:
```
./script.sh      # Running interactively
```

# Press Ctrl+Z to pause (SIGSTOP)

bg               # Resume in background
jobs             # List background jobs
fg %1            # Bring job #1 back to foreground
You can also start something directly in the background:

```
./heavy_task.sh &
```
</details>

<details>
<summary>System Calls, Process States & Signals</summary>
What are System Calls?

```
System calls are interfaces between a user-space application and the kernel. They allow programs to request services like file operations, memory allocation, process creation, etc.
```

Common system calls:
- fork() – to create a new process
- exec() – to replace a process with a new program
- open(), read(), write(), close() – for file I/O
- wait() – to wait for child processes
- exit() – to terminate a process

When a process starts:
- A new PID is created (via fork())
- The new process memory is prepared
- The binary is loaded with exec()
- Required file descriptors opened, etc.

Process States in Linux:
- R (Running): Actively using CPU
- S (Sleeping): Waiting for an event
- D (Uninterruptible sleep): Usually I/O wait
- T (Stopped): Halted via signal (e.g., SIGSTOP)
- Z (Zombie): Process exited but not reaped by parent
- X (Dead): Terminated (rarely seen)

Signals in Linux - Common Signals:
- SIGTERM (15): Graceful termination
- SIGKILL (9): Force kill (cannot be caught/ignored)
- SIGINT (2): Interrupt from keyboard (Ctrl+C)
- SIGSTOP (19): Stop process (cannot be caught/ignored)
- SIGCONT: Continue stopped process
- SIGHUP: Hang up, often used for reloading configs

Signals That Can Be Ignored:
Most can be caught/handled/ignored using trap in scripts or signal handlers in programs.
Cannot be ignored or caught:
- SIGKILL
- SIGSTOP
</details>

<details>
<summary>PID Table, ps Command, and /proc</summary>
How ps Works:

- Commands like `ps aux` or `ps -ef` pull information from the `/proc` virtual filesystem. This directory contains runtime process info for every PID (e.g., /proc/\<pid\>).


Details like command line, memory, CPU usage come from files like:
- /proc/\<pid\>/status
- /proc/\<pid\>/cmdline
- /proc/\<pid\>/stat

Where is the PID table stored?
- It's maintained in kernel memory, not user-accessible directly, but exposed via /proc.
</details>


---
## Interview Examples:
- **Question** "What is a zombie process? How do you find and deal with it?"
Answer:
```
Zombie = dead process not reaped by parent
Use ps aux | grep Z [or] ps -el | grep Z
Solution: kill parent, or ensure it calls wait()
```

---
- **Question** "How do you pause a running job and move it to background?"
```
Ctrl+Z       # Pause
bg           # Resume in background
```

---
- **Question** "How do you handle zombies?"
```
I would identify the zombie's parent process using ps -el or pstree, and check if it's stuck. If the parent isn't reaping its child, I can try restarting it. If the parent is defunct or misbehaving, I may need to kill it so init (PID 1) adopts and reaps the zombie.
```

---
- **Question** "You're seeing a lot of zombie processes on a Linux server. Walk me through how you would identify and resolve the issue."
```
Step 1: Confirm Zombies Exist
Start by listing processes with STAT = Z:

ps -el | grep Z

Or more focused:

ps aux | awk '$8 ~ /Z/ { print $0 }'

This shows you zombie processes still in the process table.

Step 2: Identify the Parent Process (PPID)
Every zombie still has a parent process ID (PPID). To find it:

ps -o ppid= -p <zombie_pid>


Or use pstree to visually trace the hierarchy:

pstree -p | less

You’ll find something like:
systemd(1)─┬─sshd(1001)───bash(1222)───defunct(1355)

Here, bash(1222) is the parent of the zombie 1355.

Step 3: Inspect the Parent Process
Now check whether the parent is stuck or still working:

ps -p <ppid> -o pid,ppid,cmd,stat

If it’s an active service (like a custom script, nginx, Java app), it should be calling wait() on its children. If not, that’s your bug.

Step 4: Fix or Kill the Parent
Option A: If parent is hung or buggy:

kill -TERM <ppid>      # Graceful shutdown
# Or, if unresponsive
kill -9 <ppid>          # Force kill

Once it dies (it becomes a orphan process), init (PID 1) will adopt the zombie and reap it.

Option B: Restart the service If the zombie is from a daemon (e.g., Apache, cron job runner), restart it:

sudo systemctl restart apache2

That usually clears the zombie if the service starts fresh and reaps old children.

Bonus Step: Prevent It Long-Term
Check the code of the parent (cron job, bash script, C code) to ensure it calls wait() or handles signals like SIGCHLD.

Conclusion Summary (Interview Style):
I’d start by confirming the presence of zombies using ps or pstree. Then I’d trace the PPID to identify the parent process. If the parent is not cleaning up its children, I’d either restart or kill it, so init can reap the zombie. If this is happening often, I'd inspect the code or script for proper child process handling and add wait() logic where needed.
```

---
- **Question** "A process is using 100% CPU. How do you handle it?"
```
top → identify PID
renice to lower priority
strace or lsof to investigate
Kill if necessary
```

---
- **Question** "What is a zombie process and how do you identify it?"
```
ps aux | grep Z or look for [defunct]

Parent process is not reaping its children

Fix: kill/restart parent process
```

---
- **Question** "How to keep a script running even after logout?"
```
nohup ./deploy.sh &
disown %1
```

---
- **Question** "How to troubleshoot a hanging process?"
```
Use strace -p <PID> to see syscalls in real time
```

---
- **Question** "What’s the difference between a zombie and an orphan process?"
```
A zombie is a process that has completed execution but still has an entry in the process table because its parent hasn't read its exit status via wait(). It holds onto resources like PID. An orphan is a process whose parent has died; the OS reassigns it to init or systemd, which cleans it up. Zombies are problematic if many accumulate, while orphans are usually handled gracefully.
```

---
- **Question** "Implement fork process in bash"
```
In bash, you can fork a process using `&` to run it in the background (this is a way to simulate a fork).
You can use ps to view processes, and wait to handle completion.

Example script :
#!/bin/bash
echo "Starting parent process $$"
( 
  echo "Child process $$"
  sleep 5
) &
child_pid=$!
echo "Child PID: $child_pid"
wait $child_pid
echo "Child completed"
```

---
- **Question** "How would you fork a child process in bash and wait for it?"
```
I can fork a process using the & operator in bash. After that, I can capture its PID and use wait to handle its completion. For example:

( sleep 3; echo "Done" ) & pid=$!; wait $pid

This runs the child in background, waits for it, and ensures cleanup.
```

---
- **Question** "How to catch an Interrupt with trap in Bash?"
What is a signal?
```
A signal is a notification sent to a process to notify it of an event (e.g., SIGINT when you press Ctrl+C).

Interrupt is a SIGINT
```

What is trap?
- trap allows you to catch and handle signals in a Bash script.
- Useful for cleanup operations when a script is interrupted.
- Can run some cleanup stuff before exiting on being interrupted.

Example: Catch SIGINT (Ctrl+C)
```
#!/bin/bash
cleanup() {
    echo "Caught interrupt. Cleaning up..."
    exit 1
}

trap cleanup SIGINT

echo "Running... Press Ctrl+C to stop"
while true; do sleep 1; done
SIGINT = Signal 2 (Ctrl+C)
```

Other common signals: SIGTERM, SIGKILL, EXIT, ERR
Use Case:
- Prevent abrupt termination of a backup script or DB migration.
- Perform cleanup (remove temp files, close sockets).

> Interview Tip:
> Demonstrate how you’ve used trap for graceful shutdowns or cleanup, especially in cron jobs or long-running scripts.