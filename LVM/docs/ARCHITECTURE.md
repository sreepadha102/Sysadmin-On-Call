flowchart TD
    A[Physical Disk<br>/dev/sdb] --> B[Physical Volume<br>(PV)]
    B --> C[Volume Group<br>vg_data]
    C --> D[Logical Volume<br>lv_data]
    D --> E[Filesystem<br>ext4 / xfs]
    E --> F[Mount Point<br>/data]


Multi-Disk Variant

flowchart TD
    A[/dev/sdb/] --> PV1[PV]
    B[/dev/sdc/] --> PV2[PV]
    PV1 --> VG[VG: vg_data]
    PV2 --> VG
    VG --> LV1[lv_data → /data]
    VG --> LV2[lv_logs → /logs]