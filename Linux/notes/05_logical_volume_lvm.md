## LVM – Logical Volume Manager
What is LVM?

LVM is a flexible disk management layer that allows you to:
- Create resizable storage volumes
- Combine multiple physical disks into one logical volume
- Resize, extend, reduce storage without rebooting
- Take snapshots for backups or testing
- It sits between the filesystem and physical storage.

Why Use LVM?

| Traditional Disk      | LVM                         |
|-----------------------|-----------------------------|
| Fixed partitions      | Dynamic volumes             |
| Hard to resize        | Easy to resize on the fly   |
| One disk = one partition | Multiple disks = one volume |
| No snapshots          | Supports snapshots          |


## LVM Terminology
1. Physical Volume (PV)

- A raw disk or partition initialized for LVM
Example: /dev/sdb1
```
pvcreate /dev/sdb1                      # initialize with only one partition
pvcreate /dev/sdb1 /dev/sdc2 /dev/sdd1  # initialize with multiple partitions
```

- Verify Physical Volumes:
    - Check if the PVs have been created successfully:
```
pvdisplay
# or a more concise output:
pvs
```

---
2. Volume Group (VG)

- A pool of physical volumes
- You create logical volumes from this
```
vgcreate my_vg /dev/sdb1                     # create using only one partition
vgcreate my_vg /dev/sdb1 /dev/sdc2 /dev/sdd1 # create using multiple partitions
```

Once a VG is created, get its total size and other info via
```
vgdisplay my_vg
```
The output might look something like this:
```
--- Volume group ---
VG Name               my_vg
System ID
Format                lvm2
Metadata Areas        1
Metadata Sequence No  3
VG Access             read/write
VG Status             resizable
MAX LV                0
Cur LV                2
Open LV               2
Max PV                0
Cur PV                2
Act PV                2
VG Size                1.79 TiB
PE Size                4.00 MiB
Total PE              460543
Alloc PE / Size       459775 / 1.79 TiB
Free  PE / Size       768 / 3.00 GiB
VG UUID               a1b2c3-d4e5-f6g7-h8i9-j0k1-l2m3-n4o5p6
```
In this example, the total size of the Volume Group my_vg is 1.79 TiB.

Using vgs (more concise output):

Alternatively, you can use the vgs command for a more concise, table-like output:
```
vgs my_vg
```

Output:
```
VG    #PV #LV #SN Attr   VSize   VFree
my_vg   2   2   0 wz--n- 1.79t  3.00g
```

Here, "1.79t" indicates the total size of the my_vg Volume Group (1.79 Terabytes). "3.00g" shows the free space within the VG (3.00 Gigabytes).

> If you omit the VG name, it will show information for all VGs.

Both vgdisplay and vgs will give you the total size of your LVM Volume Group. vgdisplay provides more detailed information, while vgs offers a more compact overview

---
3. Logical Volume (LV)

- Think of this as a flexible partition
- It sits inside a VG and is formatted with a filesystem
```
# create a LV of size 10GB
lvcreate -L 10G -n my_lv my_vg

# create a fs with ext4 using above LVM
mkfs.ext4 /dev/my_vg/my_lv

# mount the fs to a dir
mount /dev/my_vg/my_lv /mnt/data
```

Diagram Overview
```
Disk (/dev/sdb)
└── Partition (/dev/sdb1)
     └── PV (Physical Volume)
          └── VG (Volume Group)
               └── LV (Logical Volume)
                    └── Filesystem (ext4/xfs)
```

---
### Common LVM Commands
- Create LVM from Scratch
```
pvcreate /dev/sdb1
vgcreate vg_data /dev/sdb1
lvcreate -L 5G -n lv_data vg_data
mkfs.ext4 /dev/vg_data/lv_data
mount /dev/vg_data/lv_data /mnt/data
```

- Extend a Logical Volume
```
lvextend -L +5G /dev/vg_data/lv_data
resize2fs /dev/vg_data/lv_data   # ext4
xfs_growfs /mnt/data             # xfs must be mounted
```

- Shrink a Logical Volume (careful!)
Unmount it
Check filesystem:
```
e2fsck -f /dev/vg_data/lv_data
resize2fs /dev/vg_data/lv_data 5G
lvreduce -L 5G /dev/vg_data/lv_data
mount /dev/vg_data/lv_data /mnt/data
```
> Never shrink an XFS filesystem—it doesn’t support it.

Take a Snapshot
```
lvcreate -s -L 1G -n snapshot_name /dev/vg_data/lv_data
```
- Use for backups or testing changes
- Can be merged back or removed

Remove LVM
```
umount /mnt/data
lvremove /dev/vg_data/lv_data
vgremove vg_data
pvremove /dev/sdb1
```

Check Status
```
pvs      # list physical volumes
vgs      # list volume groups
lvs      # list logical volumes
```

---
## Interview Examples:
- **Question**: Why use LVM over traditional partitions?

Flexibility. Resize volumes, combine multiple disks, snapshot capabilities.

---
- **Question**: How do you extend a logical volume?

Use lvextend + grow the filesystem with resize2fs or xfs_growfs.

---
- **Question**: Can you create LVM from multiple disks?

Yes. Just add more PVs to the VG:
```
vgextend my_vg /dev/sdc1
```

---
- **Question**: Can you take backups using LVM?

Yes, via snapshots. Fast, low-impact backups.