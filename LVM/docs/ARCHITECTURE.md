```
+--------------------+
|   Physical Disk    |
|    /dev/sdb        |
+---------+----------+
          |
          v
+--------------------+
| Physical Volume    |
|       (PV)         |
+---------+----------+
          |
          v
+--------------------+
| Volume Group       |
|   vg_data          |
| (Storage Pool)     |
+---------+----------+
          |
          v
+--------------------+
| Logical Volume     |
|   lv_data          |
+---------+----------+
          |
          v
+--------------------+
| Filesystem         |
|   ext4 / xfs       |
+---------+----------+
          |
          v
+--------------------+
| Mount Point        |
|   /data            |
+--------------------+
```

### Multi-Disk LVM Architecture
```bash
/dev/sdb        /dev/sdc
   |               |
   v               v
+------+       +------+
| PV   |       | PV   |
+------+       +------+
     \           /
      \         /
       v       v
   +------------------+
   |  Volume Group    |
   |    vg_data       |
   +------------------+
        |        |
        v        v
  +----------+  +----------+
  | lv_data  |  | lv_logs  |
  +----------+  +----------+
       |               |
       v               v
     /data           /logs
```
### Extend Flow Architecture

```
Cloud Resize / New Disk
        |
        v
+------------------+
| Disk Capacity    |
| Increased        |
+--------+---------+
         |
         v
+------------------+
| pvresize         |
| (PV grows)       |
+--------+---------+
         |
         v
+------------------+
| vg_data          |
| Free Space       |
+--------+---------+
         |
         v
+------------------+
| lvextend         |
| lv_data grows    |
+--------+---------+
         |
         v
+------------------+
| resize2fs /      |
| xfs_growfs       |
+------------------+
```
