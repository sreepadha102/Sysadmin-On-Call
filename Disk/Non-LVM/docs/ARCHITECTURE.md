Add a New Disk
```bash
┌──────────────────────────────┐
│ New disk attached            │
│ (Cloud / VM / Hardware)      │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ OS detects new disk          │
│ (lsblk shows /dev/sdc)       │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Create partition             │
│ (/dev/sdc1 using fdisk)      │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Create filesystem            │
│ (mkfs.xfs /dev/sdc1)         │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Create mount point           │
│ (mkdir /data)               │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Mount filesystem             │
│ (mount /dev/sdc1 /data)     │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Persist mount                │
│ (/etc/fstab via UUID)        │
└──────────────────────────────┘

```
Extend Existing Disk
```bash
┌──────────────────────────────┐
│ Disk size increased (Cloud / │
│ Hypervisor / Storage layer)  │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ OS sees larger disk (/sdc)   │
│ but partition is unchanged  │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Extend existing partition    │
│ (/dev/sdc1) using fdisk      │
│ - SAME start sector          │
│ - New end sector             │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Reload partition table       │
│ (partprobe / partx)          │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Filesystem still old size    │
│ (XFS does NOT auto-grow)     │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Grow XFS filesystem          │
│ xfs_growfs /data             │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ /data shows increased space  │
│ No remount, no data loss     │
└──────────────────────────────┘
```