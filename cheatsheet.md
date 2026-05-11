---
title: Cheat Sheet
nav_order: 10
---

# LVM Cheat Sheet

Everything in one place. Bookmark this page.

---

## The three-step creation flow

```bash
# 1. Tag disks as Physical Volumes
pvcreate /dev/sdb /dev/sdc

# 2. Pool them into a Volume Group
vgcreate vg_name /dev/sdb /dev/sdc

# 3. Carve out a Logical Volume
lvcreate -L 100G -n lv_name vg_name

# 4. Format
mkfs.ext4 /dev/vg_name/lv_name

# 5. Mount
mount /dev/vg_name/lv_name /mnt/point
```

---

## Physical Volumes (PV)

| Command | What it does |
|---|---|
| `pvcreate /dev/sdX` | Create a PV from a disk |
| `pvs` | Quick list of all PVs |
| `pvdisplay` | Full details for all PVs |
| `pvdisplay /dev/sdX` | Details for one PV |
| `pvscan` | Re-scan and find all PVs |
| `pvremove /dev/sdX` | Remove LVM from a disk |
| `pvmove /dev/sdX` | Move all data off this PV to others in VG |
| `pvmove /dev/sdX /dev/sdY` | Move data from sdX specifically to sdY |

---

## Volume Groups (VG)

| Command | What it does |
|---|---|
| `vgcreate vg_name /dev/sdX /dev/sdY` | Create VG from one or more PVs |
| `vgs` | Quick list of all VGs |
| `vgdisplay` | Full details for all VGs |
| `vgdisplay vg_name` | Details for one VG |
| `vgscan` | Re-scan for VGs |
| `vgextend vg_name /dev/sdZ` | Add a new PV to an existing VG |
| `vgreduce vg_name /dev/sdX` | Remove a PV from a VG (PV must be empty) |
| `vgremove vg_name` | Delete a VG (destroys all LVs in it) |
| `vgrename old_name new_name` | Rename a VG |
| `vgchange -ay vg_name` | Activate a VG (makes LVs accessible) |
| `vgchange -an vg_name` | Deactivate a VG |
| `vgexport vg_name` | Export VG (for moving disks to another machine) |
| `vgimport vg_name` | Import a VG from another machine |

---

## Logical Volumes (LV)

| Command | What it does |
|---|---|
| `lvcreate -L 100G -n lv_name vg_name` | Create 100 GB LV |
| `lvcreate -l 100%FREE -n lv_name vg_name` | Create LV using all free space |
| `lvcreate -l 50%VG -n lv_name vg_name` | Create LV using 50% of VG |
| `lvs` | Quick list of all LVs |
| `lvdisplay` | Full details for all LVs |
| `lvdisplay /dev/vg_name/lv_name` | Details for one LV |
| `lvscan` | Re-scan for LVs |
| `lvremove /dev/vg_name/lv_name` | Delete an LV (destroys data) |
| `lvrename vg_name old_lv new_lv` | Rename an LV |

---

## Resizing

| Command | What it does |
|---|---|
| `lvextend -L +50G /dev/vg/lv` | Grow LV by 50 GB |
| `lvextend -L 200G /dev/vg/lv` | Grow LV to exactly 200 GB |
| `lvextend -l +100%FREE /dev/vg/lv` | Grow LV using all free space |
| `lvextend -L +50G -r /dev/vg/lv` | Grow LV + resize ext4/xfs automatically |
| `resize2fs /dev/vg/lv` | Resize ext4 filesystem to match LV |
| `xfs_growfs /mountpoint` | Grow xfs filesystem (use mountpoint, not device) |
| `lvreduce -L 100G /dev/vg/lv` | Shrink LV to 100 GB (unmount + resize fs first!) |
| `lvresize -L 100G /dev/vg/lv` | Resize LV to exactly 100 GB (grow or shrink) |

---

## Snapshots

| Command | What it does |
|---|---|
| `lvcreate -L 10G -s -n snap /dev/vg/lv` | Create 10 GB snapshot of lv |
| `mount -o ro /dev/vg/snap /mnt/snap` | Mount snapshot read-only |
| `lvconvert --merge /dev/vg/snap` | Restore origin from snapshot |
| `lvremove /dev/vg/snap` | Delete snapshot |

---

## Status and inspection

```bash
pvs && vgs && lvs          # Quick overview of everything
lsblk                      # Block device tree (shows LVM structure)
lvmdiskscan                # Show all disks LVM can see
dmsetup ls                 # List device mapper devices
df -h                      # Filesystem usage (after mounting)
```

---

## Device paths

An LV named `lv_home` in `vg_tank` appears at:

```
/dev/vg_tank/lv_home          ← symlink (easy to use)
/dev/mapper/vg_tank-lv_home   ← actual device node
```

Both work identically.

---

## fstab entry

```
# Device                        Mountpoint  FS     Options    Dump Pass
/dev/vg_tank/lv_home            /home       ext4   defaults   0    2
UUID=abc123...                  /data       xfs    defaults   0    2
```

{: .tip }
Using UUID in fstab is more reliable than device paths — device names like `/dev/sdb` can change between boots, but UUIDs don't. Get UUIDs with `blkid`.

---

## Common error messages

| Error | Cause | Fix |
|---|---|---|
| `Couldn't find device with uuid` | LVM can't find a PV | Run `pvscan`, check if disk is connected |
| `Volume group not found` | VG not activated | Run `vgchange -ay vg_name` |
| `Insufficient free space` | Not enough space in VG | Check `vgs`, add a disk with `vgextend` |
| `Snapshot has been invalidated` | Snapshot filled up | Allocate more space next time with `-L` |
| `resize2fs: New size smaller than minimum` | Tried to shrink too small | Check how much data is actually used with `df` |
| `Can't remove open logical volume` | LV is still mounted | `umount` first |

---

## Quick mental model

```
Disks      →    PVs       →    VG         →    LVs        →    Filesystems
/dev/sdb        tag it         pool them        carve them       use them
/dev/sdc        pvcreate       vgcreate         lvcreate         mkfs + mount
```

One sentence: **PVs feed into a VG pool; LVs are scooped out of that pool.**
