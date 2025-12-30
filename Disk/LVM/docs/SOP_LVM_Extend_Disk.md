# SOP: Fresh LVM Disk Provisioning

## Purpose
Extend a disk using LVM.

## Preconditions
- Disk attached to OS and extended via cloud provider
- Root or sudo access

## Inputs
| Parameter | Example |
|-----------|---------|
| Disk      | /dev/sdb|
| VG        | vg_data |
| LV        | lv_data |
| Mount     | /data |
| FS        | ext4 |

---

## Procedure

### Step 1: Verify Disk
```bash
lsblk #lists block storage devices and their partitions showing mount points  and sizes.

[ec2-user@ip-xxx-xx-xx-xxx ~]$ lsblk
NAME              MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1           259:0    0   8G  0 disk 
├─nvme0n1p1       259:1    0   8G  0 part /
├─nvme0n1p127     259:2    0   1M  0 part 
└─nvme0n1p128     259:3    0  10M  0 part /boot/efi
nvme1n1           259:4    0  10G  0 disk 
└─vg_data-lv_data 253:0    0  10G  0 lvm  /data

# Extend a new disk/volume using cloud provider
[ec2-user@ip-xxx-xx-xx-xxx ~]$ lsblk
NAME              MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1           259:0    0   8G  0 disk 
├─nvme0n1p1       259:1    0   8G  0 part /
├─nvme0n1p127     259:2    0   1M  0 part 
└─nvme0n1p128     259:3    0  10M  0 part /boot/efi
nvme1n1           259:4    0  15G  0 disk 
└─vg_data-lv_data 253:0    0  10G  0 lvm  /data
```

### Step 2: Reize pv
```bash
pvresize /dev/sdb

# Validate resize
pvs
```

### Step 3: Extend LV
```bash
lvextend -l +100%FREE /dev/vg_data/lv_data

# Validate
lvs
```

### Step 4: Extend filesystem
```bash
xfs_growfs /data
or
resize2fs /dev/vg_data/lv_data

# Validate
df -h /data
```

Demo:

```
[ec2-user@ip-xxx-xx-xx-xxx ~]$ lsblk
NAME              MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1           259:0    0   8G  0 disk 
├─nvme0n1p1       259:1    0   8G  0 part /
├─nvme0n1p127     259:2    0   1M  0 part 
└─nvme0n1p128     259:3    0  10M  0 part /boot/efi
nvme1n1           259:4    0  10G  0 disk 
└─vg_data-lv_data 253:0    0  10G  0 lvm  /data

# Extend a new disk/volume using cloud provider
[ec2-user@ip-xxx-xx-xx-xxx ~]$ lsblk
NAME              MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1           259:0    0   8G  0 disk 
├─nvme0n1p1       259:1    0   8G  0 part /
├─nvme0n1p127     259:2    0   1M  0 part 
└─nvme0n1p128     259:3    0  10M  0 part /boot/efi
nvme1n1           259:4    0  15G  0 disk 
└─vg_data-lv_data 253:0    0  10G  0 lvm  /data

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo pvresize /dev/sdb
  Physical volume "/dev/sdb" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo pvs
  PV         VG      Fmt  Attr PSize   PFree
  /dev/sdb   vg_data lvm2 a--  <15.00g 5.00g

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo vgs
  VG      #PV #LV #SN Attr   VSize   VFree
  vg_data   1   1   0 wz--n- <15.00g 5.00g

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo lvs
  LV      VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_data vg_data -wi-ao---- <10.00g

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo lvextend -l +100%FREE /dev/vg_data/lv_data
  Size of logical volume vg_data/lv_data changed from <10.00 GiB (2559 extents) to <15.00 GiB (3839 extents).
  Logical volume vg_data/lv_data successfully resized.

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo lvs
  LV      VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_data vg_data -wi-ao---- <15.00g                                                    

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo xfs_growfs /data
meta-data=/dev/mapper/vg_data-lv_data isize=512    agcount=16, agsize=163776 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
         =                       exchange=0  
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=1      swidth=1 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2620416 to 3931136

[ec2-user@ip-xxx-xx-xx-xxx ~]$ df -h /data
Filesystem                   Size  Used Avail Use% Mounted on
/dev/mapper/vg_data-lv_data   15G  141M   15G   1% /data
```

Extend same vg using a new disk
```bash
pvcreate /dev/sdc
vgextend vg_data /dev/sdc
lvextend -l +100%FREE /dev/vg_data/lv_data
resize2fs /dev/vg_data/lv_data
or
xfs_growfs /dev/vg_data/lv_data
```