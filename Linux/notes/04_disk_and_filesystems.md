## Disks, Partitions, and Filesystems in Linux
1. What is a Block Device?
A block device is a storage device that transfers data in fixed-size blocks. Examples:
- Physical disks: /dev/sda, /dev/nvme0n1
- Partitions: /dev/sda1, /dev/sdb2
- Virtual devices: loop devices (/dev/loop0), LVM volumes (/dev/mapper/vg-lv)

> [!NOTE]
> A block device does not contain a filesystem by default. It’s just raw space until we partition and format it.

2. Do Blocks Come with a Filesystem by Default?
- No, a new disk (or block device) is empty. You must:
- Partition it (optional, but common).
- Format it with a filesystem (mandatory before use).
- Mount it to a directory to access.

3. Step-by-Step: Add a New Disk
A. Identify the Disk
```
lsblk
fdisk -l
```
Look for unpartitioned disks like /dev/sdb with no children.

B. Partition the Disk
```
fdisk /dev/sdb
```
Inside fdisk:
- n – create new partition
- p – primary
- 1 – partition number
- Accept default start/end blocks
- w – write and exit
- Creates /dev/sdb1

> [!NOTE]
> For GPT partitions (especially on large disks or NVMe), use gdisk or parted.

C. Format the Partition (Create Filesystem)
```
mkfs.ext4 /dev/sdb1
# or
mkfs.xfs /dev/sdb1
```
This creates the actual filesystem on the partition. Options include:

- ext4 – common, reliable
- xfs – great for large files, scalable
- btrfs, f2fs, vfat, ntfs – depending on need

> [!NOTE]
> You can only have one filesystem per partition.

D. Mount the Filesystem
```
mkdir /mnt/data
mount /dev/sdb1 /mnt/data
```
Now the contents of /mnt/data map to your new filesystem.

E. Persistent Mount (survive reboot)
Edit /etc/fstab:

```
/dev/sdb1  /mnt/data  ext4  defaults  0 0
```

Or use UUID (safer):

```
blkid /dev/sdb1
# Then
UUID=xxxxxxxx /mnt/data ext4 defaults 0 0
```

4. Check Disk & Filesystem Info
Show all disks/partitions:

```
lsblk
```

Filesystem type and UUIDs:

```
blkid
```

Filesystem-specific details:

```
tune2fs -l /dev/sdb1   # For ext filesystems
xfs_info /mnt/data     # For XFS
```

5. Filesystem Usage
Disk usage:

```
df -hT
```
Directory usage:

```
du -sh *
du -d1 -h | sort -h # (sorted one level view of sizes)
```

See inode usage:

```
df -i
```
> [!NOTE]
> inodes are created when a filesystem is created and not when a partition in made

6. Delete a Filesystem
There is no rmfs command. To delete:

Unmount it:

```
umount /mnt/data
```
Optionally, delete partition using fdisk, parted, or wipefs:

```
wipefs -a /dev/sdb1     # Wipes filesystem signature
fdisk /dev/sdb          # Delete the partition
```

##############################################

7. Resize/Repair/Check a Filesystem
Resize (after changing partition size):

```
```
resize2fs /dev/sdb1     # ext4
xfs_growfs /mnt/data    # xfs (must be mounted)
Check filesystem:

```
```
fsck /dev/sdb1
Caution: Never run fsck on a mounted FS. Use in single-user mode or unmount first.

8. Inodes Explained
Every file has an inode, which stores metadata:

File type, permissions

Owner, group

Timestamps

Pointers to data blocks

But inode count is fixed when formatting. If you run out of inodes:

```
```
df -i
You cannot create new files—even if space exists!

9. Creating a Filesystem with Custom Inodes
```
```
mkfs.ext4 -N 500000 /dev/sdb1
-N sets number of inodes

-T small optimizes for lots of tiny files

10. Interview-Focused Questions and Answers
Q: What happens when you attach a new disk to Linux?
It appears as /dev/sdX or /dev/nvmeXn1

You need to partition it (fdisk)

Then format it (mkfs.ext4)

Then mount it (mount)

Make it persistent (fstab)

Q: Can you have multiple filesystems on the same disk?
Yes, by creating multiple partitions:

```
```
/dev/sdb1 – ext4
/dev/sdb2 – xfs
Q: What’s the difference between a partition and a filesystem?
Partition: Block of space on a disk

Filesystem: Structure that organizes data in that space

Q: What’s the risk if inode count is exhausted?
You can’t create new files—even if space is available. Use df -i to monitor.

Q: How do you delete a filesystem?
Unmount it → wipefs or fdisk → re-partition or reformat as needed