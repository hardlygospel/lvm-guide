---
title: 2. Your First LVM Setup
nav_order: 3
---

# Your First LVM Setup

A complete walkthrough: two blank disks → one pool → two logical volumes → formatted and mounted.

---

## Prerequisites

Install the LVM tools if not already present:

```bash
# Debian / Ubuntu
apt install lvm2

# RHEL / Rocky / AlmaLinux / Fedora
dnf install lvm2

# Arch
pacman -S lvm2
```

Confirm the kernel module is loaded:
```bash
modprobe dm_mod
```

---

## The scenario

You have two blank disks:
- `/dev/sdb` — 500 GB
- `/dev/sdc` — 1 TB

You want to create:
- `lv_data` — 1 TB for application data at `/mnt/data`
- `lv_backup` — 500 GB for backups at `/mnt/backup`

Everything in a VG called `vg_tank`.

---

## Step 1 — Create Physical Volumes

```bash
pvcreate /dev/sdb /dev/sdc
```

Expected output:
```
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
```

Verify:
```bash
pvs
```
```
  PV         VG  Fmt  Attr PSize   PFree
  /dev/sdb       lvm2 ---  500.00g 500.00g
  /dev/sdc       lvm2 ---    1.00t   1.00t
```

The `VG` column is blank — these PVs haven't been added to a group yet.

{: .note }
`pvcreate` on a disk that already has a filesystem will warn you. Always double-check the disk path with `lsblk` before running it.

---

## Step 2 — Create the Volume Group

```bash
vgcreate vg_tank /dev/sdb /dev/sdc
```

Expected output:
```
  Volume group "vg_tank" successfully created
```

Verify:
```bash
vgs
```
```
  VG      #PV #LV #SN Attr   VSize  VFree
  vg_tank   2   0   0 wz--n- 1.49t  1.49t
```

`1.49t` — that's the combined 500 GB + 1 TB, minus small overhead. The VG is now one unified pool.

---

## Step 3 — Create Logical Volumes

Create `lv_data` at 1 TB:
```bash
lvcreate -L 1T -n lv_data vg_tank
```

Create `lv_backup` with the remaining space:
```bash
lvcreate -l 100%FREE -n lv_backup vg_tank
```

{: .tip }
`-L 1T` means exactly 1 TB. `-l 100%FREE` means "use whatever's left." The lowercase `-l` takes logical extents or percentages; uppercase `-L` takes human-readable sizes.

Verify:
```bash
lvs
```
```
  LV        VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log
  lv_backup vg_tank -wi-a----- 500.00g
  lv_data   vg_tank -wi-a-----   1.00t
```

Your LVs now exist at:
- `/dev/vg_tank/lv_data`
- `/dev/vg_tank/lv_backup`

---

## Step 4 — Format with a filesystem

```bash
mkfs.ext4 /dev/vg_tank/lv_data
mkfs.ext4 /dev/vg_tank/lv_backup
```

You can also use `xfs` (better for very large files), `btrfs`, or anything else — LVM doesn't care what filesystem sits on top.

---

## Step 5 — Mount

```bash
mkdir -p /mnt/data /mnt/backup
mount /dev/vg_tank/lv_data    /mnt/data
mount /dev/vg_tank/lv_backup  /mnt/backup
```

Verify:
```bash
df -h /mnt/data /mnt/backup
```
```
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/vg_tank-lv_data   1.0T   28K  1.0T   1% /mnt/data
/dev/mapper/vg_tank-lv_backup 492G   28K  492G   1% /mnt/backup
```

---

## Step 6 — Make it permanent (fstab)

Get the UUIDs:
```bash
blkid /dev/vg_tank/lv_data /dev/vg_tank/lv_backup
```
```
/dev/vg_tank/lv_data: UUID="a1b2c3d4-..." TYPE="ext4"
/dev/vg_tank/lv_backup: UUID="e5f6a7b8-..." TYPE="ext4"
```

Add to `/etc/fstab`:
```
UUID=a1b2c3d4-...  /mnt/data    ext4  defaults  0 2
UUID=e5f6a7b8-...  /mnt/backup  ext4  defaults  0 2
```

Test without rebooting:
```bash
mount -a
```

If no errors, the mounts are correct.

---

## Complete flow at a glance

```
pvcreate /dev/sdb /dev/sdc           # Tag the disks
vgcreate vg_tank /dev/sdb /dev/sdc   # Pool them
lvcreate -L 1T   -n lv_data   vg_tank  # Carve out lv_data
lvcreate -l 100%FREE -n lv_backup vg_tank  # Use the rest
mkfs.ext4 /dev/vg_tank/lv_data         # Format
mkfs.ext4 /dev/vg_tank/lv_backup       # Format
mount /dev/vg_tank/lv_data    /mnt/data    # Mount
mount /dev/vg_tank/lv_backup  /mnt/backup  # Mount
```

---

## Useful verification commands

```bash
pvs          # Summary of all Physical Volumes
vgs          # Summary of all Volume Groups
lvs          # Summary of all Logical Volumes

pvdisplay    # Detailed PV info
vgdisplay    # Detailed VG info
lvdisplay    # Detailed LV info

lsblk        # See the full block device tree including LVM
```

`lsblk` output will show you the structure clearly:
```
NAME                  MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sdb                     8:16   0   500G  0 disk
└─vg_tank-lv_backup   253:1    0   492G  0 lvm  /mnt/backup
sdc                     8:32   0     1T  0 disk
└─vg_tank-lv_data     253:0    0     1T  0 lvm  /mnt/data
```

{: .important }
Never run `pvcreate`, `vgremove`, or `lvremove` on a disk or volume that's in use or that you're not sure about. These operations destroy data.
