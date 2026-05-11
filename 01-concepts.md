---
title: 1. The Three Layers
nav_order: 2
---

# The Three Layers of LVM

LVM has exactly three concepts. Once these click, everything else follows naturally.

---

## The full picture

<div style="overflow-x:auto;margin:2rem 0;">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 660 440" style="width:100%;max-width:660px;display:block;margin:0 auto;">
  <defs>
    <marker id="arr1" viewBox="0 0 10 8" refX="10" refY="4" markerWidth="7" markerHeight="7" orient="auto">
      <path d="M0,0 L10,4 L0,8 Z" fill="#585b70"/>
    </marker>
  </defs>
  <rect width="660" height="440" rx="12" fill="#181825"/>
  <text x="330" y="24" text-anchor="middle" font-family="ui-monospace,monospace" font-size="12" fill="#6c7086" letter-spacing="2">LVM ARCHITECTURE — FULL STACK</text>

  <!-- LAYER 1: Physical Disks -->
  <text x="12" y="46" font-family="ui-monospace,monospace" font-size="8" fill="#585b70" letter-spacing="2">PHYSICAL DISKS — what your machine actually has</text>
  <rect x="10"  y="52" width="190" height="50" rx="7" fill="#1e1e2e" stroke="#45475a" stroke-width="1"/>
  <text x="105" y="74"  text-anchor="middle" font-family="ui-monospace,monospace" font-size="12" fill="#cdd6f4">/dev/sda</text>
  <text x="105" y="92"  text-anchor="middle" font-family="ui-monospace,monospace" font-size="9"  fill="#45475a">500 GB — SSD</text>
  <rect x="210" y="52" width="190" height="50" rx="7" fill="#1e1e2e" stroke="#45475a" stroke-width="1"/>
  <text x="305" y="74"  text-anchor="middle" font-family="ui-monospace,monospace" font-size="12" fill="#cdd6f4">/dev/sdb</text>
  <text x="305" y="92"  text-anchor="middle" font-family="ui-monospace,monospace" font-size="9"  fill="#45475a">1 TB — HDD</text>
  <rect x="410" y="52" width="240" height="50" rx="7" fill="#1e1e2e" stroke="#45475a" stroke-width="1"/>
  <text x="530" y="74"  text-anchor="middle" font-family="ui-monospace,monospace" font-size="12" fill="#cdd6f4">/dev/sdc</text>
  <text x="530" y="92"  text-anchor="middle" font-family="ui-monospace,monospace" font-size="9"  fill="#45475a">2 TB — HDD</text>

  <!-- Arrows: Disks → PVs -->
  <line x1="105" y1="102" x2="105" y2="127" stroke="#585b70" stroke-width="1.5" marker-end="url(#arr1)"/>
  <line x1="305" y1="102" x2="305" y2="127" stroke="#585b70" stroke-width="1.5" marker-end="url(#arr1)"/>
  <line x1="530" y1="102" x2="530" y2="127" stroke="#585b70" stroke-width="1.5" marker-end="url(#arr1)"/>
  <text x="112" y="120" font-family="ui-monospace,monospace" font-size="8" fill="#89b4fa">pvcreate</text>
  <text x="312" y="120" font-family="ui-monospace,monospace" font-size="8" fill="#89b4fa">pvcreate</text>
  <text x="537" y="120" font-family="ui-monospace,monospace" font-size="8" fill="#89b4fa">pvcreate</text>

  <!-- LAYER 2: Physical Volumes -->
  <text x="12" y="144" font-family="ui-monospace,monospace" font-size="8" fill="#89b4fa" letter-spacing="2">PHYSICAL VOLUMES (PV) — tagged so LVM owns them</text>
  <rect x="10"  y="150" width="190" height="52" rx="7" fill="#1e2b4a" stroke="#89b4fa" stroke-width="1.5"/>
  <text x="105" y="173" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#89b4fa" font-weight="bold">PV: /dev/sda</text>
  <text x="105" y="191" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9"  fill="#45475a">500 GB available</text>
  <rect x="210" y="150" width="190" height="52" rx="7" fill="#1e2b4a" stroke="#89b4fa" stroke-width="1.5"/>
  <text x="305" y="173" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#89b4fa" font-weight="bold">PV: /dev/sdb</text>
  <text x="305" y="191" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9"  fill="#45475a">1 TB available</text>
  <rect x="410" y="150" width="240" height="52" rx="7" fill="#1e2b4a" stroke="#89b4fa" stroke-width="1.5"/>
  <text x="530" y="173" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#89b4fa" font-weight="bold">PV: /dev/sdc</text>
  <text x="530" y="191" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9"  fill="#45475a">2 TB available</text>

  <!-- Arrows: PVs → VG (converge) -->
  <line x1="105" y1="202" x2="105" y2="228" stroke="#585b70" stroke-width="1.5"/>
  <line x1="305" y1="202" x2="305" y2="228" stroke="#585b70" stroke-width="1.5"/>
  <line x1="530" y1="202" x2="530" y2="228" stroke="#585b70" stroke-width="1.5"/>
  <line x1="105" y1="228" x2="530" y2="228" stroke="#585b70" stroke-width="1.5"/>
  <line x1="330" y1="228" x2="330" y2="248" stroke="#585b70" stroke-width="1.5" marker-end="url(#arr1)"/>
  <text x="338" y="242" font-family="ui-monospace,monospace" font-size="8" fill="#a6e3a1">vgcreate</text>

  <!-- LAYER 3: Volume Group -->
  <text x="12" y="264" font-family="ui-monospace,monospace" font-size="8" fill="#a6e3a1" letter-spacing="2">VOLUME GROUP (VG) — one unified storage pool</text>
  <rect x="10" y="270" width="640" height="58" rx="7" fill="#1a3320" stroke="#a6e3a1" stroke-width="2"/>
  <text x="330" y="296" text-anchor="middle" font-family="ui-monospace,monospace" font-size="14" fill="#a6e3a1" font-weight="bold">vg_storage</text>
  <text x="330" y="318" text-anchor="middle" font-family="ui-monospace,monospace" font-size="10" fill="#45475a">3.5 TB total  ·  all three drives unified  ·  allocate as needed</text>

  <!-- Arrows: VG → LVs (fan out) -->
  <line x1="330" y1="328" x2="330" y2="350" stroke="#585b70" stroke-width="1.5"/>
  <line x1="120" y1="350" x2="540" y2="350" stroke="#585b70" stroke-width="1.5"/>
  <line x1="120" y1="350" x2="120" y2="368" stroke="#585b70" stroke-width="1.5" marker-end="url(#arr1)"/>
  <line x1="330" y1="350" x2="330" y2="368" stroke="#585b70" stroke-width="1.5" marker-end="url(#arr1)"/>
  <line x1="540" y1="350" x2="540" y2="368" stroke="#585b70" stroke-width="1.5" marker-end="url(#arr1)"/>
  <text x="40"  y="346" font-family="ui-monospace,monospace" font-size="8" fill="#585b70">lvcreate</text>

  <!-- LAYER 4: Logical Volumes -->
  <text x="12" y="366" font-family="ui-monospace,monospace" font-size="8" fill="#cba6f7" letter-spacing="2">LOGICAL VOLUMES (LV) — what you actually format and mount</text>
  <rect x="10"  y="372" width="220" height="56" rx="7" fill="#2d1b4a" stroke="#cba6f7" stroke-width="1.5"/>
  <text x="120" y="396" text-anchor="middle" font-family="ui-monospace,monospace" font-size="12" fill="#cba6f7" font-weight="bold">lv_home</text>
  <text x="120" y="416" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9"  fill="#45475a">200 GB → ext4 → /home</text>
  <rect x="240" y="372" width="180" height="56" rx="7" fill="#3a2010" stroke="#fab387" stroke-width="1.5"/>
  <text x="330" y="396" text-anchor="middle" font-family="ui-monospace,monospace" font-size="12" fill="#fab387" font-weight="bold">lv_var</text>
  <text x="330" y="416" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9"  fill="#45475a">800 GB → ext4 → /var</text>
  <rect x="430" y="372" width="220" height="56" rx="7" fill="#10303a" stroke="#89dceb" stroke-width="1.5"/>
  <text x="540" y="396" text-anchor="middle" font-family="ui-monospace,monospace" font-size="12" fill="#89dceb" font-weight="bold">lv_data</text>
  <text x="540" y="416" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9"  fill="#45475a">2.5 TB → xfs → /data</text>
</svg>
</div>

---

## Layer 1 — Physical Volume (PV)

A Physical Volume is a **disk or partition that LVM has claimed**. Nothing more. You're just stamping it with an LVM header that says "I own this — don't use it for regular partitions."

```
pvcreate /dev/sda
```

LVM writes a small metadata header to the beginning of the disk. The rest of the space becomes a pile of **Physical Extents (PEs)** — fixed-size chunks (usually 4 MB each) that LVM uses as its basic unit of allocation. Think of PEs as the individual Lego bricks.

**What a PV looks like:**
```
/dev/sda  ──────────────────────────────────────────────────
          │ LVM header │ PE  │ PE  │ PE  │ PE  │ PE  │ ... │
          ──────────────────────────────────────────────────
                          ↑ 4 MB chunks, ~125,000 of them on a 500 GB disk
```

**Key PV commands:**
```bash
pvcreate /dev/sda          # Create a PV from /dev/sda
pvs                        # List all PVs (quick summary)
pvdisplay                  # Detailed PV info
pvremove /dev/sda          # Remove LVM from a disk (destroys data)
```

{: .note }
You can create a PV from a whole disk (`/dev/sda`) or from a single partition (`/dev/sda1`). Using whole disks is simpler and more flexible.

---

## Layer 2 — Volume Group (VG)

A Volume Group is **one or more PVs combined into a single pool**. Once you have a VG, you forget about the individual disks — to LVM and to you, it's just one big pile of space.

```
vgcreate vg_storage /dev/sda /dev/sdb /dev/sdc
```

This pools all three disks into `vg_storage`. LVM now has 3.5 TB to allocate as it sees fit. If you later plug in a fourth disk, you add it to the same VG — no reformatting, no data movement.

**What a VG looks like internally:**
```
vg_storage (3.5 TB)
┌──────────────────────────────────────────────────────────────────┐
│ PEs from /dev/sda  │ PEs from /dev/sdb  │   PEs from /dev/sdc   │
│ ░░░░░░░░░░░░░░░░░░ │ ░░░░░░░░░░░░░░░░░░ │ ░░░░░░░░░░░░░░░░░░░░░ │
└──────────────────────────────────────────────────────────────────┘
  500 GB               1 TB                 2 TB
```

**Key VG commands:**
```bash
vgcreate vg_storage /dev/sda /dev/sdb   # Create VG from PVs
vgextend vg_storage /dev/sdc            # Add another disk later
vgs                                      # Quick VG summary
vgdisplay vg_storage                    # Detailed info
vgremove vg_storage                     # Delete VG (destroys data)
```

{: .tip }
Name your VG something meaningful (`vg_data`, `vg_storage`, `ubuntu-vg`). You'll be typing it a lot.

---

## Layer 3 — Logical Volume (LV)

A Logical Volume is **a chunk of space carved from the VG** — this is what you actually format with a filesystem (`ext4`, `xfs`) and mount.

```
lvcreate -L 200G -n lv_home vg_storage
```

This creates a 200 GB LV named `lv_home` from `vg_storage`. LVM finds 200 GB worth of PEs in the VG (from whichever disk has them free) and assigns them to this LV. To the filesystem above, it looks exactly like a regular block device at `/dev/vg_storage/lv_home`.

**Logical Volumes can span multiple physical disks seamlessly:**
```
lv_home (200 GB) — could be spread across sda and sdb — you don't care, LVM handles it
```

**Key LV commands:**
```bash
lvcreate -L 200G -n lv_home vg_storage  # Create 200 GB LV
lvcreate -l 100%FREE -n lv_data vg_storage  # Use all remaining space
lvs                                      # Quick LV summary
lvdisplay                               # Detailed info
lvremove /dev/vg_storage/lv_home        # Delete LV (destroys data)
```

---

## Before and after LVM

Here's why this matters in practice:

```
WITHOUT LVM                             WITH LVM
──────────────────────────────────      ────────────────────────────────────

 /dev/sda (500 GB)                       /dev/sda  /dev/sdb  /dev/sdc
 ┌────────┬────────┬────────┬──────┐         │         │         │
 │/dev/   │/dev/   │/dev/   │/dev/ │         └────────┬┘         │
 │sda1    │sda2    │sda3    │sda4  │                  │          │
 │/boot   │/       │swap    │/home │         ┌────────▼──────────▼────────┐
 │500 MB  │100 GB  │4 GB    │395 GB│         │         vg_storage         │
 └────────┴────────┴────────┴──────┘         │           3.5 TB           │
                                             └──────────────────────────┬─┘
 6 months later:                                                         │
   /home is 90% full                         ┌──────┬──────────┬────────▼──┐
   /     has 60 GB free                      │lv_   │lv_var    │lv_home    │
                                             │root  │100 GB    │200 GB     │
 You can't resize /home — it's              │100 GB│(growable)│(growable) │
 physically fixed on the disk.              └──────┴──────────┴───────────┘

 Your only option: backup everything,
 repartition, restore. Hours of work.        6 months later — /home 90% full?
                                             lvextend -L +100G /dev/vg_storage/lv_home
                                             resize2fs /dev/vg_storage/lv_home
                                             Done. 30 seconds. No downtime.
```

---

## The device path for LVs

When you create `lv_home` in `vg_storage`, it appears at two equivalent paths:

```
/dev/vg_storage/lv_home   ← the symlink (easier to read)
/dev/mapper/vg_storage-lv_home   ← the actual device (both work)
```

Use whichever you prefer — they point to the same thing.

---

## Summary table

| Concept | Role | Command | Example result |
|---|---|---|---|
| **Physical Volume** | Marks a disk for LVM | `pvcreate /dev/sda` | `/dev/sda` is LVM-ready |
| **Volume Group** | Pools PVs into one | `vgcreate vg_data /dev/sda /dev/sdb` | `vg_data` = 1.5 TB |
| **Logical Volume** | Slices from the pool | `lvcreate -L 200G -n lv_home vg_data` | `/dev/vg_data/lv_home` exists |
| **Format** | Puts a filesystem on it | `mkfs.ext4 /dev/vg_data/lv_home` | Ready to mount |
| **Mount** | Makes it accessible | `mount /dev/vg_data/lv_home /home` | Files in `/home` |
