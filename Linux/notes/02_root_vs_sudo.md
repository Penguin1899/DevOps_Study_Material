
## Root User vs Superuser
These terms are often used interchangeably, but here's the nuance:

Root User:
- The actual user account with UID 0.
- Full, unrestricted access to the entire system.
- Typically named root.
- Can do anything, including deleting system files, changing ownerships, killing any process, etc.

Superuser:
- A role or concept, not necessarily tied to the root user.
- Any user who has superuser privileges (often via sudo).
- So, when a normal user runs a command with sudo, they are temporarily acting as the superuser—even though they’re not root.

> [!NOTE]
> Every root user is a superuser, but not every superuser is the root user.
> </br>Superuser = role; root = account.

## Running a Command as Root vs. Using sudo
### Running as Root (Directly):
- You log in as root or switch to root (su -).
- Everything you do runs with root's full power.
- No need to type sudo, because you're already root.
- Can be dangerous, because there are no guardrails.

Example:
```
# Already root
rm -rf /etc
```
- This will immediately run, no warnings.

### Running with sudo:
- You run a single command as root or another user.
- You must be in the sudoers list.
- Logs are kept (in `/var/log/auth.log`), which is good for audit.
- Less risky—if you make a typo, it doesn’t affect your whole shell session.

Example:
```
sudo rm -rf /etc
```
- This does the same thing, but you had to deliberately choose to escalate just for this command.

## Using sudo -u (Run as Another User)
- You’re telling sudo to run a command as a user other than root.

Example:
```
sudo -u postgres psql
```
- Run the psql command as the postgres user, not as root.
- It’s helpful when you want to impersonate another service account (like www-data, postgres, etc.) without switching to them entirely.


## What is the sudoers file?
- Yes, the sudoers file is exactly what `visudo` edits.

Purpose:
- It defines who can use sudo, for what commands, and under what conditions.

Location:
- `/etc/sudoers`

Why use visudo instead of directly editing?
- `visudo` checks for syntax errors before saving.
- A bad sudoers file can lock you out from using sudo.
- `visudo` opens the file in a safe environment and warns you of issues.

Sample Contents:
```
# Allow root to run anything
root    ALL=(ALL:ALL) ALL

# Allow user 'john' to run all commands with sudo
john    ALL=(ALL) ALL

# Allow 'deploy' to restart nginx without password
deploy  ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx
```

Each line says:
- Who: john, deploy, etc.
- Where: ALL = any host (usually only 1 system, so it's fine).
- As Whom: (ALL) = can act as any user.
- Commands: What can they run, and optionally without a password.

<details>
<summary>How do you give a user sudo access only for specific commands?</summary>
</br>Let’s say we want to let user devops restart Nginx, but nothing else—not full root access.

Step-by-Step Answer (Interview Style)
1. Open the sudoers file safely using visudo:
```
sudo visudo
```
2. Add a rule for the user:
```
devops ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx
```
Explanation:
- devops – the username.
- ALL – can run from any host (normal for local systems).
- (ALL) – can run as any user (usually root).
- NOPASSWD: – doesn’t require password (optional).
- /bin/systemctl restart nginx – the only allowed command.

> Important Notes (For Interview):
> This restricts the user to exactly that command with exact arguments.
> For example, `systemctl stop nginx` or `systemctl restart apache2` will not work.

If the command has variables (like service names), you may need to be specific or use wildcards, but that can open risks.

Always use visudo to prevent misconfiguration.

How to test this:
Log in as devops and try:
```
sudo /bin/systemctl restart nginx   # Works
sudo /bin/systemctl stop nginx      # Fails
sudo su                             # Fails
```

> Bonus Tip (For Standout Points):
> You can also define command aliases and user groups for cleaner config:
```
Cmnd_Alias NGINX_CMDS = /bin/systemctl restart nginx
devops ALL=(ALL) NOPASSWD: NGINX_CMDS
```
</details>

---

<details>
<summary>How do you allow a user to run only a list of specific commands using sudo?</summary>
</br>
- Option 1: List commands directly
```
devuser ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx, /usr/bin/systemctl status nginx, /usr/bin/journalctl -u nginx
```
You just list each command, comma-separated.
This is fine for a short list.

- Option 2: Use a Cmnd_Alias (cleaner, preferred for many commands)

*Step 1*: Define the alias (near the top of sudoers):
```
Cmnd_Alias NGINX_TASKS = /usr/bin/systemctl restart nginx, /usr/bin/systemctl status nginx, /usr/bin/journalctl -u nginx
```
*Step 2*: Grant access:
```
devuser ALL=(ALL) NOPASSWD: NGINX_TASKS
```
This is much more readable, especially when multiple users are being configured.

**Bonus: Granular Control Over Arguments**
- sudoers doesn't just control the binary, it controls the full command line including arguments.

So this works:
```
/usr/bin/systemctl restart nginx
```
But this won't:
```
/usr/bin/systemctl restart apache2   # Not allowed
```
If you want to allow any argument to systemctl, you could do this (be careful):
```
Cmnd_Alias SYS_TASKS = /usr/bin/systemctl *
```
This is more flexible, but opens doors to misuse. Interviewers love it when you mention this trade-off.

**Pro Tip (Interview Gold)**:
Never allow unrestricted access like this unless necessary:
```
devuser ALL=(ALL) NOPASSWD: ALL
```
Because that’s just giving full root access. If you mention this danger in an interview, it shows maturity and awareness.
</details>

---
<details>
<summary>How do you let a user run only a specific script with sudo?</summary>
</br>Scenario:
You have a script like this: `/opt/scripts/deploy.sh`

```
#!/bin/bash
echo "Deploying app..."
# Some privileged commands here
systemctl restart myapp
```
You want user deployer to run only this script via sudo—nothing else.

Step-by-Step Guide
1. Make sure the script has correct permissions:
```
sudo chown root:root /opt/scripts/deploy.sh
sudo chmod 700 /opt/scripts/deploy.sh
```
- Owned by root
- Only root can modify it
- This ensures the user can’t tamper with it

2. Open sudoers via visudo:
```
sudo visudo
```

3. Add a rule:
```
deployer ALL=(ALL) NOPASSWD: /opt/scripts/deploy.sh
```
Now user deployer can run:
```
sudo /opt/scripts/deploy.sh
```
And only that script.

Best Practices (Interview Points):
- Always make sure scripts are owned by root and not writable by the user.
- If the script internally calls privileged commands, they’ll run fine, because the whole script is being executed as root.
- This is safer than allowing general sudo access to tools like systemctl.

Bonus: What If Script Needs Arguments?
If your script needs arguments like:

```
sudo /opt/scripts/deploy.sh staging
```

Then in the sudoers file, you must allow that specific command:
```
deployer ALL=(ALL) NOPASSWD: /opt/scripts/deploy.sh staging
```
Or allow any argument (less safe):
```
deployer ALL=(ALL) NOPASSWD: /opt/scripts/deploy.sh *
```
</details>


## What is in auth.log?
Location:
- Debian/Ubuntu: /var/log/auth.log
- RHEL/CentOS: /var/log/secure

What’s Logged:
- This file tracks authentication-related events, like:
- sudo usage (every command with timestamp and user)
- su command attempts
- SSH logins (successful and failed)
- PAM authentication failures
- Lockouts, login attempts, password changes

Sample log entries:
```
Apr 6 10:15:01 server sudo:   john : TTY=pts/0 ; PWD=/home/john ; USER=root ; COMMAND=/bin/systemctl restart nginx
Apr 6 10:17:05 server su: pam_unix(su:session): session opened for user root by john(uid=1001)
Apr 6 10:19:22 server sshd[1234]: Failed password for invalid user admin from 192.168.1.10 port 55123 ssh2
```

Why it’s useful:
- Helps in auditing who did what.
- Critical in security investigations.
- You can write scripts or use tools like fail2ban to read this and block brute-force attacks.