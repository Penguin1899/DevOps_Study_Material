## User Management
Important Files:
- `/etc/passwd`	 Stores user account details (username, UID, shell)
- `/etc/shadow`	 Stores encrypted passwords and aging info
- `/etc/group`	 Group definitions
- `/etc/sudoers` sudo access control (edit via visudo)

### User Commands
- Create user - `useradd john` or `adduser john`
- Delete user	- `userdel -r john`
- Modify user	- `usermod -aG docker john` (add to group)
- Change password -	`passwd john`
- Lock/unlock - `passwd -l john` / `passwd -u john`
- Change default shell of user - `chsh -s /bin/bash username`
    - before this, ensure the shell is part of `/etc/shells`
- View username - `id username`

### User Identifiers
UID and GID

- UID = User ID (stored in /etc/passwd)
    - UID 0 → root
    - UID 1-999 → system users
    - UID 1000+ → regular users
- GID = Group ID (stored in /etc/group)

### Default User Settings
Defaults set in:
- /etc/login.defs
- /etc/skel/ → copied to new users’ home dirs

### Locking & Disabling Users
- Lock a User (disable login)
```
passwd -l username
```
- Unlock a User
```
passwd -u username
```
- Expire a Password
```
chage -E 0 username
```


## Managing Groups

- Primary group: Defined in `/etc/passwd`
- Supplementary groups: Added with `usermod -aG`

#### Create a Group
```
groupadd developers
```
#### Add User to Multiple Groups
```
usermod -aG dev,qa,infra username
```
#### Change Primary Group
```
usermod -g groupname username
```
#### Key Commands
```
groups john
id john
groupadd devs
groupdel devs
```

## Permissions
Permission Model (rwx)
-Files have 3 levels of permissions:
```
Owner, Group, Others
```

Example:
```
-rwxr-x--x 1 root devs 1234 Apr 5 script.sh
```
Means:
- root can read/write/execute
- devs group can read/execute
- others can only execute

### `chmod` - Change Permissions
Change owner - `chown user file`
Change group - `chown :group file`
Both - `chown user:group file`
Change perms - `chmod 755 file` or `chmod u+x file`

#### Numeric chmod values:
Basically this is based on 3 binary bits
xxx - can be 000, 001, 010, 011....111
```
7	rwx (111) 
6	rw- (110)
5	r-x (101)
4	r-- (100)
0	--- (000)
```
So chmod 754 (111 101 100)= 
- Owner: rwx
- Group: r-x
- Others: r--

#### Symbolic Mode
```
chmod u+x script.sh      # Add execute to user
chmod g-w file.txt       # Remove write from group
chmod o=r file.txt       # Set read-only for others
```

#### Special Permission Bits
- SUID (Set UID) - Runs with file owner's privileges
- SGID (Set GID) - Runs with group privileges / inherited group in dirs
- Sticky Bit - Prevents users from deleting others' files in shared dirs (e.g., /tmp)
Set via chmod:
```
chmod u+s /usr/bin/somebin  # SUID
chmod g+s /some/dir         # SGID
chmod +t /tmp               # Sticky
```
View Special Bits
```
ls -l
# Look for 's' or 't' in the execute position:
# `s` in owner's execute position - SUID is set
# `s` in group's execute position - SGID is set
# -rwsr-xr-x = SUID
# drwxrwsr-x = SGID on dir
# drwxrwxrwt = sticky (like /tmp)
```

## `chown` - Change File Ownership
Syntax
```
chown [user][:group] file
```

Examples:
```
chown root:devs file.txt       # Change owner to root and group to devs
chown :devs file.txt           # Only change group

```
Recursive (-R) Use
```
chown -R user:group /dir
```
- Changes ownership of directory and all files/subdirs inside it.
> [!CAUTION]
> Interview Tip: Always use -R with caution—especially in /, /home, /var, etc.

#### umask – Default Permission Mask
What It Is:
- it is a way of "masking" default file/folder permissions
- umask subtracts permissions from default creation values to give us final default permission value

Default:
```
File: 666 (rw-rw-rw-)
Dir: 777 (rwxrwxrwx)
```
> [!IMPORTANT]
> Final permissions = Default – umask (do binary bit subrtraction or convert to decimal, subtract and find final bits)

Common umask Values:
umask	File Default	Dir Default
022	644	755
027	640	750
077	600	700
Check and Set
```
umask              # Show current
umask 027          # Set new umask
```

> [!IMPORTANT]
> Note: This is per session unless set in:
> ~/.bashrc or ~/.profile (user-specific)
> /etc/profile or /etc/bash.bashrc (global)

---
## Interview Example:
- **Question** "How would you ensure all newly created files are only accessible by the owner?"
```
umask 077
```

---
- **Question** "How do you secure a shared directory like /data?"
```
I'd use SGID to ensure group inheritance:

chgrp devteam /data
chmod 2770 /data
```

---
- **Question** "How to allow only owner to execute a script?"
```
chmod 700 script.sh
```

---
- **Question** "Prevent deletion of others’ files in /tmp?"
```
chmod +t /tmp  # sticky bit
```

---
- **Question** "Explain SUID with real example"
```
/usr/bin/passwd uses SUID so that normal users can update their password (write to /etc/shadow) securely.

ls -l /usr/bin/passwd
# -rwsr-xr-x
```
