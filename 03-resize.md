---
title: 3. Resizing Volumes
nav_order: 4
---

# Resizing Volumes

This is LVM's killer feature. Growing a logical volume takes 30 seconds and works while the filesystem is mounted and in use.

---

## Growing a logical volume (online, no downtime)

### Scenario: `/var` is nearly full

```bash
df -h /var
```
```
Filesystem                   Size  Used Avail Use% Mounted on
/dev/mapper/vg_tank-lv_var   50G   47G   3G   94% /var
```

Check that the VG has free space:
```bash
vgs vg_tank
```
```
  VG      #PV #LV #SN Attr   VSize  VFree
  vg_tank   2   2   0 wz--n- 1.49t  400.00g
```

Good — 400 GB free in the pool.

### Step 1 — Extend the LV

Add 100 GB:
```bash
lvextend -L +100G /dev/vg_tank/lv_var
```

Or extend to a specific total size:
```bash
lvextend -L 150G /dev/vg_tank/lv_var
```

Or use all remaining free space:
```bash
lvextend -l +100%FREE /dev/vg_tank/lv_var
```

### Step 2 — Resize the filesystem

The LV is now bigger, but the filesystem doesn't know yet. Tell it:

**ext4:**
```bash
resize2fs /dev/vg_tank/lv_var
```

**xfs** (xfs can only grow, not shrink):
```bash
xfs_growfs /var
```

{: .tip }
For ext4, you can do both steps in one command with `-r`:
```bash
lvextend -L +100G -r /dev/vg_tank/lv_var
```
The `-r` flag runs `resize2fs` automatically after extending.

Verify:
```bash
df -h /var
```
```
Filesystem                   Size  Used Avail Use% Mounted on
/dev/mapper/vg_tank-lv_var   150G  47G  103G  31% /var
```

Done. Zero downtime. The filesystem was mounted the entire time.

---

## What's happening visually

```
BEFORE lvextend
┌──────────────────────────────────────────────┐
│                   vg_tank                    │
│  ┌──────────────┐  ┌───────────────────────┐ │
│  │    lv_var    │  │      FREE SPACE       │ │
│  │    50 GB     │  │      400 GB           │ │
│  └──────────────┘  └───────────────────────┘ │
└──────────────────────────────────────────────┘

AFTER lvextend -L +100G
┌──────────────────────────────────────────────┐
│                   vg_tank                    │
│  ┌────────────────────────┐  ┌─────────────┐ │
│  │        lv_var          │  │  FREE SPACE │ │
│  │       150 GB           │  │   300 GB    │ │
│  └────────────────────────┘  └─────────────┘ │
└──────────────────────────────────────────────┘
```

---

## Shrinking a logical volume (offline, requires care)

{: .warning }
**Shrinking is risky.** You must unmount the filesystem first, check it for errors, shrink the filesystem, *then* shrink the LV — in that exact order. Getting the order wrong corrupts data. XFS cannot be shrunk at all.

### Scenario: shrink `lv_home` from 500 GB to 300 GB

**Step 1 — Unmount**
```bash
umount /home
```

**Step 2 — Check the filesystem for errors**
```bash
e2fsck -f /dev/vg_tank/lv_home
```

**Step 3 — Shrink the filesystem first**
```bash
resize2fs /dev/vg_tank/lv_home 300G
```

**Step 4 — Then shrink the LV** (must be equal to or larger than the filesystem size)
```bash
lvreduce -L 300G /dev/vg_tank/lv_home
```

**Step 5 — Remount**
```bash
mount /dev/vg_tank/lv_home /home
```

{: .important }
Always shrink the **filesystem before the LV**. If you shrink the LV first, you cut off data that the filesystem thinks still exists — instant corruption.

---

## Adding a new disk to an existing VG

Your `vg_tank` is nearly full. You add a new 2 TB drive at `/dev/sdd`.

```bash
# Step 1: Create a PV from the new disk
pvcreate /dev/sdd

# Step 2: Add it to the existing VG
vgextend vg_tank /dev/sdd

# Step 3: VG now has 2 TB more
vgs vg_tank
```
```
  VG      #PV #LV #SN Attr   VSize  VFree
  vg_tank   3   2   0 wz--n- 3.49t  2.30t
```

No data was moved. No volumes were disrupted. The extra space is immediately available for new LVs or to extend existing ones.

---

## Moving data from one disk to another (pvmove)

You want to remove `/dev/sdb` from `vg_tank` — maybe you're replacing it with a faster drive. LVM can live-migrate the data.

```bash
# Move all data off /dev/sdb to other PVs in the VG
pvmove /dev/sdb

# This can take a while for large volumes — you can watch progress
pvmove -v /dev/sdb

# Once complete, remove it from the VG
vgreduce vg_tank /dev/sdb

# Remove LVM from the disk entirely
pvremove /dev/sdb
```

The data was moved **while the filesystem was still mounted and in use**. LVM handles this completely transparently.

---

## Quick reference

| Task | Command |
|---|---|
| Grow LV by 100 GB | `lvextend -L +100G /dev/vg/lv` |
| Grow LV to exactly 500 GB | `lvextend -L 500G /dev/vg/lv` |
| Use all free space | `lvextend -l +100%FREE /dev/vg/lv` |
| Grow LV + resize ext4 in one step | `lvextend -L +100G -r /dev/vg/lv` |
| Resize ext4 after growing | `resize2fs /dev/vg/lv` |
| Grow xfs after growing LV | `xfs_growfs /mountpoint` |
| Add new disk to VG | `pvcreate /dev/sdX && vgextend vg /dev/sdX` |
| Move data off a disk | `pvmove /dev/sdX` |
| Remove disk from VG | `vgreduce vg /dev/sdX` |
