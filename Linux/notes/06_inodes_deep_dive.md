## Inodes in Linux

1. What is an Inode?

An inode (index node) is a data structure used by Linux filesystems (like ext4) to store metadata about files and directories. It does not store file names or actual data — just metadata.

---
2. Why is it Needed?

Every file/directory in Linux is represented by an inode. Without it, the system wouldn’t know:
- Where a file’s data is on disk
- Who owns it
- What permissions are set
- Timestamps (created, modified, accessed)
- Type (file, directory, symlink, etc.)

---
3. What Does an Inode Contain?
- File type (regular file, dir, symlink, etc.)
- Owner UID and GID
- File size
- Permissions (rwx bits)
- Timestamps: creation, modification, access
- Number of hard links
- Pointers to data blocks (location on disk)

---
4. What It Does Not Contain
- File name
- Directory path
> Names are stored in directory entries, which map names to inode numbers.

---
5. How to View Inode Info?
- Check inode number of a file
```
ls -i filename
```

- View inode info in detail
```
stat filename
```

- Check inode usage per partition
```
df -i
```

- Check inode usage in a folder
```
find . -xdev -printf '%i\n' | sort | uniq -c | sort -n
```

---
## Interview Examples:

- **Question**: Inode Exhaustion (Very Important Interview Point)
What Happens When Inodes Run Out?

Disk may have free space left, but you can’t create new files.
Especially common when:
- Millions of small files (like logs, cache, temp files)
- Hosting environments with many mailboxes

How to Check for It?
```
df -i
```
Solution
- Delete unnecessary files (especially tiny ones)
- Reformat with larger inode count (last-resort)
- NOTE: This will "re-format" the disk
```
mkfs.ext4 -N <inode-count> /dev/sdX
```
- Monitor inode usage on production systems!

---
- **Question**: What is an inode, and why is it important?

It’s a data structure that holds metadata about a file. Without it, Linux can’t track file ownership, permissions, or where its data is stored.

---
- **Question**: Can a disk be full even if space is free?

Yes — if all inodes are used, no new files can be created even if blocks are free.

---
- **Question**: How do you monitor inode usage?

Use df -i to check inode stats per mounted filesystem.