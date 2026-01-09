# SOP: Extend LVM Disk

## Purpose
Extend a disk using LVM.

## Preconditions
- Disk attached to OS and extended via cloud provider
- Root or sudo access

## Inputs
| Parameter | Example |
|-----------|---------|
| Disk      | /dev/sdb|
| Mount     | /data |
| FS        | xfs |

---

## Procedure

### Step 1: Verify Disk
```bash
lsblk #lists block storage devices and their partitions showing mount points  and sizes.

[ec2-user@ip-xxx-xx-xx-xxx ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk 
├─nvme0n1p1   259:1    0   8G  0 part /
├─nvme0n1p127 259:2    0   1M  0 part 
└─nvme0n1p128 259:3    0  10M  0 part /boot/efi
nvme1n1       259:4    0  10G  0 disk 
└─nvme1n1p1   259:6    0  10G  0 part /data

# Extend a new disk/volume using cloud provider
[ec2-user@ip-xxx-xx-xx-xxx ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk 
├─nvme0n1p1   259:1    0   8G  0 part /
├─nvme0n1p127 259:2    0   1M  0 part 
└─nvme0n1p128 259:3    0  10M  0 part /boot/efi
nvme1n1       259:4    0  15G  0 disk 
└─nvme1n1p1   259:6    0  10G  0 part /data
```

### Step 2: Extend the Existing Partition (fdisk)
```bash
umount /data
fdisk /dev/sdc
p       → print partition table (note start sector of sdb1)
d       → delete partition 1   (DATA IS NOT DELETED)
n       → new partition
1       → partition number (same as before)
<enter> → SAME start sector as before
<enter> → use rest of disk
w       → write changes
# Validate resize
partprobe /dev/sdb
lsblk
```

### Step 3: mount file system
```bash
mount /data
```

### Step 4: Extend filesystem
```bash
xfs_growfs /data

# Validate
df -h /data
```

Demo:

```
lsblk #lists block storage devices and their partitions showing mount points  and sizes.

[ec2-user@ip-xxx-xx-xx-xxx ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk 
├─nvme0n1p1   259:1    0   8G  0 part /
├─nvme0n1p127 259:2    0   1M  0 part 
└─nvme0n1p128 259:3    0  10M  0 part /boot/efi
nvme1n1       259:4    0  10G  0 disk 
└─nvme1n1p1   259:6    0  10G  0 part /data

# Extend a new disk/volume using cloud provider
[ec2-user@ip-xxx-xx-xx-xxx ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk 
├─nvme0n1p1   259:1    0   8G  0 part /
├─nvme0n1p127 259:2    0   1M  0 part 
└─nvme0n1p128 259:3    0  10M  0 part /boot/efi
nvme1n1       259:4    0  15G  0 disk 
└─nvme1n1p1   259:6    0  10G  0 part /data

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo umount /data

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo fdisk /dev/sdc

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p
Disk /dev/sdc: 15 GiB, 16106127360 bytes, 31457280 sectors
Disk model: Amazon Elastic Block Store              
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0xaa495c7c

Device     Boot Start      End  Sectors Size Id Type
/dev/sdc1        2048 20971519 20969472  10G 83 Linux

Command (m for help): d
Selected partition 1
Partition 1 has been deleted.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-31457279, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-31457279, default 31457279): 

Created a new partition 1 of type 'Linux' and of size 15 GiB.
Partition #1 contains a xfs signature.

Do you want to remove the signature? [Y]es/[N]o: N

Command (m for help): w

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[ec2-user@ip-xxx-xx-xx-xxx ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk 
├─nvme0n1p1   259:1    0   8G  0 part /
├─nvme0n1p127 259:2    0   1M  0 part 
└─nvme0n1p128 259:3    0  10M  0 part /boot/efi
nvme1n1       259:4    0  15G  0 disk 
└─nvme1n1p1   259:7    0  15G  0 part 

[ec2-user@ip-xxx-xx-xx-xxx ~]$ xfs_growfs /data
xfs_growfs: /data is not a mounted XFS filesystem
[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo mount /dat

[ec2-user@ip-xxx-xx-xx-xxx ~]$ xfs_growfs /data
xfs_growfs: cannot open /dev/nvme1n1p1: Permission denied

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo xfs_growfs /data
meta-data=/dev/nvme1n1p1         isize=512    agcount=16, agsize=163824 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
         =                       exchange=0  
data     =                       bsize=4096   blocks=2621184, imaxpct=25
         =                       sunit=1      swidth=1 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2621184 to 3931904

[ec2-user@ip-xxx-xx-xx-xxx ~]$ df -h /data
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme1n1p1   15G  141M   15G   1% /data

