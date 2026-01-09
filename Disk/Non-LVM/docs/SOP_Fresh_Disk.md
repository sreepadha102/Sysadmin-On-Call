# SOP: Fresh Non LVM Disk Provisioning

## Purpose
Provision a new disk on a Linux system without using LVM, format it with XFS, mount it, and ensure persistence across reboots.

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
ls -l /dev/sdc
lrwxrwxrwx. 1 root root 7 Dec 30 16:27 /dev/sdc -> nvme1n1
```

### Step 2: Partition the Disk (fdisk)
```bash
fdisk /dev/sdc
enter: n - for new partition
accept default for partition - p
accept default for partition number - 1
accept default for first sector
accept default for last sector

w to save

lsblk to validate
[ec2-user@ip-xxx-xx-xx-xxx ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk 
├─nvme0n1p1   259:1    0   8G  0 part /
├─nvme0n1p127 259:2    0   1M  0 part 
└─nvme0n1p128 259:3    0  10M  0 part /boot/efi
nvme1n1       259:4    0  10G  0 disk 
└─nvme1n1p1   259:6    0  10G  0 part 
```

### Step 3: Create XFS Filesystem
```bash
sudo mkfs.xfs -s size=4096 /dev/sdc1
```

### Step 4: Mount file system
```bash
mkdir /data
mount /dev/sdc1 /data
```

### Step 5: Persist Mount
```bash
blkid /dev/sdc1
# add the UUID to /etc/fstab for persistent mount
vi /etc/fstab
UUID=daadfadsf-adfad-asdsd /data xfs defaults,nofail 0 2
```

### Step 6: Validate
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

#Attach a new disk
[ec2-user@ip-xxx-xx-xx-xxx ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk 
├─nvme0n1p1   259:1    0   8G  0 part /
├─nvme0n1p127 259:2    0   1M  0 part 
└─nvme0n1p128 259:3    0  10M  0 part /boot/efi
nvme1n1       259:4    0  10G  0 disk

[ec2-user@ip-xxx-xx-xx-xxx ~]$ ls -l /dev/sdc
lrwxrwxrwx. 1 root root 7 Dec 30 17:41 /dev/sdc -> nvme1n1

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo fdisk /dev/nvme1n1

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xaa495c7c.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (1-4, default 1): 
First sector (2048-20971519, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-20971519, default 20971519): 

Created a new partition 1 of type 'Linux' and of size 10 GiB.

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
nvme1n1       259:4    0  10G  0 disk 
└─nvme1n1p1   259:6    0  10G  0 part

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo blkid /dev/sdc1
/dev/sdc1: UUID="cb75231f-d15b-4f3b-a16f-520210edcf85" BLOCK_SIZE="4096" TYPE="xfs" PARTUUID="aa495c7c-01"

[ec2-user@ip-xxx-xx-xx-xxx ~]$ df -h
Filesystem        Size  Used Avail Use% Mounted on
devtmpfs          4.0M     0  4.0M   0% /dev
tmpfs             459M     0  459M   0% /dev/shm
tmpfs             184M  448K  183M   1% /run
/dev/nvme0n1p1    8.0G  1.6G  6.4G  20% /
tmpfs             459M     0  459M   0% /tmp
/dev/nvme0n1p128   10M  1.3M  8.7M  13% /boot/efi
tmpfs              92M     0   92M   0% /run/user/1000

[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo mkdir /data
[ec2-user@ip-xxx-xx-xx-xxx ~]$ sudo mount /dev/sdc1 /data
[ec2-user@ip-xxx-xx-xx-xxx ~]$ df -h /data
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme1n1p1   10G  105M  9.9G   2% /data
[ec2-user@ip-xxx-xx-xx-xxx ~]$ df -h
Filesystem        Size  Used Avail Use% Mounted on
devtmpfs          4.0M     0  4.0M   0% /dev
tmpfs             459M     0  459M   0% /dev/shm
tmpfs             184M  448K  183M   1% /run
/dev/nvme0n1p1    8.0G  1.6G  6.4G  20% /
tmpfs             459M     0  459M   0% /tmp
/dev/nvme0n1p128   10M  1.3M  8.7M  13% /boot/efi
tmpfs              92M     0   92M   0% /run/user/1000
/dev/nvme1n1p1     10G  105M  9.9G   2% /data

[ec2-user@ip-xxx-xx-xx-xxx ~]$ cat /etc/fstab
#
UUID=170fd7a4-f127-46eb-a3be-8b380dc8d94a     /           xfs    defaults,noatime  1   1
UUID=7BBE-6739        /boot/efi       vfat    defaults,noatime,uid=0,gid=0,umask=0077,shortname=winnt,x-systemd.automount 0 2
UUID=cb75231f-d15b-4f3b-a16f-520210edcf85 /data xfs defaults,noatime,nofail 0 2
```