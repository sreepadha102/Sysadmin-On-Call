# LVM Fresh Disk Provisioning

This module contains **standardized runbooks and SOPs** for provisioning
new disks using **Linux Logical Volume Manager (LVM)**.

## What This module Covers
- Fresh disk provisioning from zero
- LVM concepts (PV, VG, LV)
- ext4 and XFS filesystems
- Persistent mounting using `/etc/fstab`
- Cloud-specific considerations (AWS, GCP, Azure)

## Who This Is For
- Linux system administrators
- SRE / DevOps engineers
- Platform and infrastructure teams

## Quick Start
If you want step-by-step commands:
➡️ `docs/SOP_LVM_Fresh_Disk.md`
➡️ `docs/SOP_LVM_Extend_Disk.md`

If you want architecture understanding:
➡️ `docs/ARCHITECTURE.md`

---

## Safety Notice
⚠️ These procedures assume the disk is **empty**.  
Running these commands on the wrong disk will result in **data loss**.

Always verify using:
```bash
lsblk