## Soft Links vs Hard Links in Linux
What is a Link?
- A link in Linux is a pointer to a file. There are two kinds:
    - Hard link: Points directly to the file's inode.
    - Soft link (symlink): Points to the file name/path (like a shortcut).

1. Hard Links : What is it?
- A hard link is another name for the same file.
- It shares the same inode number.
- Deleting one hard link does not delete the data as long as another link exists.

How to Create:
```
ln original.txt hardlink.txt
```
Key Properties:
- Works only within the same filesystem.
- Can’t be made for directories (unless using low-level tricks like -d and debugfs).
- If you delete the original file, the hard link still fully works.

Check Inode to Prove It:
```
ls -li original.txt hardlink.txt
```
They’ll show the same inode number.

---
2. Soft Links (Symbolic Links) : What is it?
- A soft link is like a shortcut or alias.
- It contains the path to the target file.
- If the original file is deleted, the soft link becomes broken (dangling).

How to Create:
```
ln -s original.txt softlink.txt
```
Key Properties:
- Can point to directories or files.
- Can span across filesystems.
- Broken easily if the target file is moved/deleted.

How to Identify:
```
ls -l
```
You’ll see:

```
softlink.txt -> original.txt
```
Comparison Table

| Feature	             | Hard Link     | Soft Link (Symlink) |
|------------------------|---------------|---------------------|
| Inode Shared           | Yes	         | No                  |
| Works across FS?       | No            | Yes                 |
| Can link dirs?	     | No (usually)	 | Yes                 |
| Breaks if target gone? |	No	         | Yes                 |
| Command	             | ln            |	ln -s              |

---
## Interview Examples:

- **Question**:  How do hard links avoid data duplication?

They point to the same inode. The content isn't duplicated, only the directory entries increase.

---
- **Question**: What happens to data if you delete the original file after creating a hard link?

Nothing. Data remains intact via the hard link.

---
- **Question**: Why are soft links useful for system-level paths?

Because you can redirect paths (e.g., /bin/sh pointing to /bin/bash) and even link across filesystems.