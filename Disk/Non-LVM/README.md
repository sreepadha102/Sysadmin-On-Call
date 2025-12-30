## Add and Mount a Fresh Disk (Non-LVM, XFS)

This module contains **standardized runbooks and SOPs** for provisioning new disks using **Non-LVM** and explains how to add a new disk, partition it with fdisk, format it using XFS, and mount it persistently on a Linux system without using LVM.

This procedure is suitable for:
- Bare metal or virtual machines
- Fresh, unused disks
- Persistent mounts across reboots

Scope:
- Disk state: New / empty
- Volume manager: None (No LVM)
- Filesystem: XFS
- Partitioning tool: fdisk
- Mount persistence: /etc/fstab

### High-Level Flow
```bash
Attach Disk
   ↓
Detect Disk
   ↓
Partition Disk (fdisk)
   ↓
Reload Partition Table
   ↓
Create XFS Filesystem
   ↓
Create Mount Point
   ↓
Mount Filesystem
   ↓
Persist in /etc/fstab
   ↓
Verify
```

## Who This Is For
- Linux system administrators
- SRE / DevOps engineers
- Platform and infrastructure teams

## Quick Start
If you want step-by-step commands:
➡️ `docs/SOP_Fresh_Disk.md`
➡️ `docs/SOP_Extend_Disk.md`

If you want architecture understanding:
➡️ `docs/ARCHITECTURE.md`

---

## Safety Notice
⚠️ These procedures assume the disk is **empty**.  
Running these commands on the wrong disk will result in **data loss**.

Always verify using:
```bash
lsblk
