
# SOP: Fresh LVM Disk Provisioning

## Purpose
Provision a new disk using LVM and mount it persistently.

## Preconditions
- Disk attached to OS
- No existing data
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

[ec2-user@ip-xxxxxxx ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk 
├─nvme0n1p1   259:1    0   8G  0 part /
├─nvme0n1p127 259:2    0   1M  0 part 
└─nvme0n1p128 259:3    0  10M  0 part /boot/efi

# Attach a new disk/volume using cloud provider
[ec2-user@ip-xxx-xx-xx-xxx ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk 
├─nvme0n1p1   259:1    0   8G  0 part /
├─nvme0n1p127 259:2    0   1M  0 part 
└─nvme0n1p128 259:3    0  10M  0 part /boot/efi
nvme1n1       259:4    0  10G  0 disk

AWS uses NVMe storage, which is provided to the instance through AWS Nitro for fast disk performance.
Internally, the cloud platform (for example, AWS Nitro) still manages the disk like a traditional block device, but it presents it to the operating system as an NVMe device, so Linux shows nvme1n1 instead of sdb

# Verify block device:
ls -l /dev/sdb
lrwxrwxrwx. 1 root root 7 Dec 30 16:27 /dev/sdb -> nvme1n1
```

### Step 2: Create Physical Volume
```bash
pvcreate /dev/sdb
```

### Step 3: Create Volume Group
```bash
vgcreate vg_data /dev/sdb
```

### Step 4: Create Logical Volume
```bash
lvcreate -l 100%FREE -n lv_data vg_data

# size based:
lvcreate -L 5G -n lv_data vg_data
```

### Step 5: Create Filesystem
```bash
mkfs.ext4 /dev/vg_data/lv_data
or
mkfs.xfs /dev/vg_data/lv_data
```

### Step 6: Mount
```bash
mkdir -p /data
mount /dev/vg_data/lv_data /data
```

### Step 7: Persist Mount
```bash
blkid /dev/vg_data/lv_data
# add the UUID to /etc/fstab for persistent mount
vi /etc/fstab
UUID=daadfadsf-adfad-asdsd /data xfs defaults,nofail 0 2
```

### Step 8: Validate
```bash
df -h /data
```

Demo:
```bash

[ec2-user@ip-xxx-xx-xx-xxx ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk 
├─nvme0n1p1   259:1    0   8G  0 part /
├─nvme0n1p127 259:2    0   1M  0 part 
└─nvme0n1p128 259:3    0  10M  0 part /boot/efi
nvme1n1       259:4    0  10G  0 disk 

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo pvs
  PV         VG Fmt  Attr PSize  PFree 
  /dev/sdb      lvm2 ---  10.00g 10.00g

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo vgcreate vg_data /dev/sdb
  Volume group "vg_data" successfully created

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo vgs
  VG      #PV #LV #SN Attr   VSize   VFree  
  vg_data   1   0   0 wz--n- <10.00g <10.00g

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo lvcreate -l 100%FREE -n lv_data vg_data
  Logical volume "lv_data" created.

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo lvs
  LV      VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_data vg_data -wi-a----- <10.00g                                                    

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo mkdir /data
[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo mkfs.xfs /dev/vg_data/lv_data
meta-data=/dev/vg_data/lv_data   isize=512    agcount=16, agsize=163776 blks
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
[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo mount /dev/vg_data/lv_data /data
[ec2-user@ip-xxx-xx-xx-xxx ~]$ df -h /data
Filesystem                   Size  Used Avail Use% Mounted on
/dev/mapper/vg_data-lv_data   10G  104M  9.9G   2% /data

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo blkid /dev/vg_data/lv_data
/dev/vg_data/lv_data: UUID="edfc9070-8735-4f9b-9eff-9179f85ddb40" BLOCK_SIZE="512" TYPE="xfs"

[ec2-user@ip-xxx-xx-xx-xxx ~]$ cat /etc/fstab
#
UUID=170fd7a4-f127-46eb-a3be-8b380dc8d94a     /           xfs    defaults,noatime  1   1
UUID=7BBE-6739        /boot/efi       vfat    defaults,noatime,uid=0,gid=0,umask=0077,shortname=winnt,x-systemd.automount 0 2
UUID=edfc9070-8735-4f9b-9eff-9179f85ddb40 /data xfs defaults,nofail 0 2

[ec2-user@ip-xxx-xx-xx-xxx ~]$ df -h
Filesystem                   Size  Used Avail Use% Mounted on
devtmpfs                     4.0M     0  4.0M   0% /dev
tmpfs                        459M     0  459M   0% /dev/shm
tmpfs                        184M  452K  183M   1% /run
/dev/nvme0n1p1               8.0G  1.6G  6.4G  21% /
tmpfs                        459M     0  459M   0% /tmp
/dev/nvme0n1p128              10M  1.3M  8.7M  13% /boot/efi
tmpfs                         92M     0   92M   0% /run/user/1000
/dev/mapper/vg_data-lv_data   10G  104M  9.9G   2% /data
```
