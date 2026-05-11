---
title: 4. Snapshots
nav_order: 5
---

# Snapshots

A snapshot is an instant point-in-time copy of a logical volume. Creating one takes milliseconds, regardless of how large the volume is.

---

## How snapshots work

Snapshots use a technique called **copy-on-write (CoW)**. When you create a snapshot:

1. LVM creates a new, small LV linked to the original
2. **Nothing is copied yet** — the snapshot initially contains zero data
3. As long as the original data doesn't change, the snapshot just reads from the original
4. The moment a block on the original is about to be *modified*, LVM first copies the **original** version of that block into the snapshot, then lets the write proceed

This means:
- Creating a snapshot is near-instant, no matter the source size
- The snapshot stays small if the source data doesn't change much
- You always have the original state preserved

```
T=0: Create snapshot
─────────────────────────────────────────────
  lv_home (origin)    │  snap_home (snapshot)
  ┌───────────────┐   │  ┌─────────────────┐
  │ A  B  C  D  E │   │  │  [empty]        │
  └───────────────┘   │  │  reads: origin  │
                      │  └─────────────────┘

T=1: Block B is modified on origin
─────────────────────────────────────────────
  lv_home (origin)    │  snap_home (snapshot)
  ┌───────────────┐   │  ┌─────────────────┐
  │ A  B' C  D  E │   │  │  [old B saved]  │
  └───────────────┘   │  │  reads: A from origin
  (B is now B')       │  │          B from snapshot
                      │  │          C,D,E from origin
                      │  └─────────────────┘
```

The snapshot presents the original view of the data (with old B), even though the origin has moved on.

---

## Creating a snapshot

```bash
lvcreate -L 20G -s -n snap_home /dev/vg_tank/lv_home
```

| Flag | Meaning |
|---|---|
| `-L 20G` | Reserve 20 GB for changed blocks |
| `-s` | This is a snapshot |
| `-n snap_home` | Name of the snapshot LV |
| `/dev/vg_tank/lv_home` | The source (origin) LV |

{: .tip }
The snapshot size (`20G` here) doesn't need to match the source. It only needs to hold the *changes* that will accumulate before you delete the snapshot. For a brief backup snapshot, 10–20% of the source is usually enough. If the snapshot fills up, it becomes invalid.

Verify:
```bash
lvs
```
```
  LV        VG      Attr       LSize  Origin   Snap%  ...
  lv_home   vg_tank owi-aos--- 200.00g                ...
  snap_home vg_tank swi-a-s--- 20.00g lv_home  0.00   ...
```

`Snap%` shows how full the snapshot's reserved space is. Watch this — if it hits 100%, the snapshot is lost.

---

## Using a snapshot for backup

Mount the snapshot (read-only is safest):

```bash
mkdir /mnt/snap
mount -o ro /dev/vg_tank/snap_home /mnt/snap
```

Now back it up:
```bash
tar -czf /backup/home_backup.tar.gz -C /mnt/snap .
rsync -a /mnt/snap/ /backup/home/
```

The origin (`/home`) is still live and in use the entire time. Your backup is from a frozen point in time.

When done:
```bash
umount /mnt/snap
lvremove /dev/vg_tank/snap_home
```

---

## Restoring from a snapshot

If something went wrong and you need to roll back to the snapshot state:

```bash
# Unmount the origin
umount /home

# Merge the snapshot back into the origin
lvconvert --merge /dev/vg_tank/snap_home
```

LVM will merge on the next activation (usually on reboot if the LV was mounted). After the merge, the origin is back to the snapshot's point in time, and the snapshot LV disappears.

{: .note }
If the LV is not mounted, the merge happens immediately. If it was mounted, LVM marks it for merge on next mount.

---

## Thin provisioning (advanced)

Regular snapshots allocate their space from the parent VG. **Thin provisioning** is a more powerful model:

- Create a thin pool inside the VG
- Create thin LVs inside the pool
- LVs only use actual disk space when data is written
- Snapshots of thin LVs are extremely efficient

```bash
# Create a 500 GB thin pool
lvcreate -L 500G --thinpool tp_pool vg_tank

# Create thin LVs (these can sum to more than 500 GB — overprovisioning)
lvcreate -V 200G --thin -n lv_thin1 vg_tank/tp_pool
lvcreate -V 200G --thin -n lv_thin2 vg_tank/tp_pool

# Snapshot a thin LV (instant, very cheap)
lvcreate -s -n snap_thin1 vg_tank/lv_thin1
```

{: .warning }
Thin provisioning allows you to create more virtual space than physical space (overprovisioning). This is useful but risky — if real usage exceeds the pool size, the pool fills up and all thin LVs in it go offline. Monitor thin pool usage carefully with `lvs`.

---

## Snapshot monitoring

```bash
# Watch snapshot fill percentage
watch -n 5 "lvs | grep snap"

# Get detailed snapshot info
lvdisplay /dev/vg_tank/snap_home
```

Set up automatic snapshot extension (LVM 2.02.99+):
```bash
# In /etc/lvm/lvm.conf, set:
snapshot_autoextend_threshold = 70   # extend when 70% full
snapshot_autoextend_percent = 20     # extend by 20%
```

---

## Summary

| Operation | Command |
|---|---|
| Create snapshot (20G reserve) | `lvcreate -L 20G -s -n snap_name /dev/vg/lv_name` |
| Mount snapshot read-only | `mount -o ro /dev/vg/snap_name /mnt/snap` |
| Check snapshot usage | `lvs` (watch `Snap%` column) |
| Delete snapshot | `lvremove /dev/vg/snap_name` |
| Restore (merge back) | `lvconvert --merge /dev/vg/snap_name` |
