---
title: LVM Guide
nav_order: 1
---

# LVM — Logical Volume Manager

[![Linux](https://img.shields.io/badge/Linux-any%20distro-f59e0b?style=flat-square&logo=linux&logoColor=white)]()
[![Level](https://img.shields.io/badge/Level-Beginner%20friendly-22c55e?style=flat-square)]()
[![Topic](https://img.shields.io/badge/Topic-Storage%20Management-6366f1?style=flat-square)]()

---

## The problem LVM solves

You install Linux. You give `/home` 200 GB and `/var` 50 GB. Six months later `/home` is full and `/var` has 40 GB free. With traditional partitions, you're stuck. You cannot just move space from one partition to another — the boundaries are fixed.

**LVM removes that limitation.** It sits between your physical disks and your filesystems, turning rigid fixed-size partitions into flexible pools you can resize, combine, snapshot, and move at will — usually without even unmounting.

---

## Three ways to understand LVM

Pick whichever one clicks for you:

---

### 🧱 The Lego analogy

Imagine your hard drives as **boxes of Lego bricks**. Each box has a fixed number of bricks — that's your disk space.

**Without LVM:** You build a house using *only one box at a time*. Once that box is empty, you can't add more bricks to the house — you'd have to knock it down and start again with a bigger box.

**With LVM:** You tip all your boxes into one giant pile (the **Volume Group**). Then you build whatever you want from that pile. You need your house bigger? Just grab more bricks from the pile. You bought a new box of bricks? Tip it into the pile and keep going.

| LVM term | Lego equivalent |
|---|---|
| Physical disk | A box of Lego |
| Physical Volume (PV) | That box, opened and ready to contribute |
| Volume Group (VG) | The big communal pile of all your bricks |
| Logical Volume (LV) | The thing you built from the pile |

---

### 💧 The water tank analogy

Think of your disks as **individual water tanks** — a 500 GB tank, a 1 TB tank, a 2 TB tank — all sitting separately.

**Without LVM:** Each application gets piped to exactly one tank. When that tank runs dry, it's done, even if the other tanks are half full.

**With LVM:** You connect all the tanks together into one **reservoir** (Volume Group). You then run separate pipes (Logical Volumes) out of the reservoir — one pipe for `/home`, one for `/var`, one for `/data`. If one pipe needs more water, you open a valve. If the reservoir runs low, you add another tank.

---

### 🗂️ The whiteboard analogy

Picture a big whiteboard (your Volume Group). You divide it with tape into sections: one section labelled `/home`, one `/var`, one `/data`.

**Without LVM:** The tape is glued permanently. Moving it means wiping everything and starting fresh.

**With LVM:** The tape is just marker lines. Move them any time. The data stays where it is; you just shift the boundary.

---

## The three building blocks

```
Physical Disk          →    Physical Volume (PV)    →    Volume Group (VG)    →    Logical Volume (LV)
/dev/sda  /dev/sdb              tagged for LVM              combined pool             what you use
```

| Layer | What it is | Created with |
|---|---|---|
| **Physical Volume (PV)** | A disk (or partition) that LVM knows about | `pvcreate` |
| **Volume Group (VG)** | One big pool made from one or more PVs | `vgcreate` |
| **Logical Volume (LV)** | A slice of the VG — this is what you format and mount | `lvcreate` |

---

## Why use LVM?

| Capability | Without LVM | With LVM |
|---|---|---|
| Resize a partition | Dangerous, often requires reboot | `lvextend` + `resize2fs`, online |
| Span two disks as one | Not possible | Yes — add a PV to a VG |
| Take an instant backup snapshot | No | `lvcreate -s` |
| Move data to a new disk | Manual, risky | `pvmove` — LVM does it live |
| Use drives of different sizes | Wastes space | Perfectly fine |

---

## Quick overview diagram

<div style="overflow-x:auto;margin:2rem 0;">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 660 180" style="width:100%;max-width:660px;display:block;margin:0 auto;">
  <defs>
    <marker id="arr0" viewBox="0 0 10 8" refX="10" refY="4" markerWidth="7" markerHeight="7" orient="auto">
      <path d="M0,0 L10,4 L0,8 Z" fill="#585b70"/>
    </marker>
  </defs>
  <rect width="660" height="180" rx="10" fill="#181825"/>
  <!-- Disk boxes -->
  <rect x="10"  y="20" width="110" height="44" rx="6" fill="#1e1e2e" stroke="#45475a"/>
  <text x="65"  y="38" text-anchor="middle" font-family="ui-monospace,monospace" font-size="10" fill="#cdd6f4">/dev/sda</text>
  <text x="65"  y="54" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9"  fill="#45475a">500 GB</text>
  <rect x="130" y="20" width="110" height="44" rx="6" fill="#1e1e2e" stroke="#45475a"/>
  <text x="185" y="38" text-anchor="middle" font-family="ui-monospace,monospace" font-size="10" fill="#cdd6f4">/dev/sdb</text>
  <text x="185" y="54" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9"  fill="#45475a">1 TB</text>
  <rect x="250" y="20" width="110" height="44" rx="6" fill="#1e1e2e" stroke="#45475a"/>
  <text x="305" y="38" text-anchor="middle" font-family="ui-monospace,monospace" font-size="10" fill="#cdd6f4">/dev/sdc</text>
  <text x="305" y="54" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9"  fill="#45475a">2 TB</text>
  <!-- Label -->
  <text x="8" y="15" font-family="ui-monospace,monospace" font-size="8" fill="#585b70" letter-spacing="1">DISKS</text>
  <!-- Arrow right from disks to VG -->
  <line x1="360" y1="42" x2="390" y2="42" stroke="#585b70" stroke-width="1.5" marker-end="url(#arr0)"/>
  <text x="363" y="38" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">pvcreate</text>
  <text x="360" y="50" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">vgcreate</text>
  <!-- VG box -->
  <rect x="390" y="14" width="130" height="56" rx="6" fill="#1a3320" stroke="#a6e3a1" stroke-width="1.5"/>
  <text x="455" y="38" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#a6e3a1" font-weight="bold">vg_data</text>
  <text x="455" y="55" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9"  fill="#45475a">3.5 TB pool</text>
  <text x="392" y="12" font-family="ui-monospace,monospace" font-size="8" fill="#585b70" letter-spacing="1">VOLUME GROUP</text>
  <!-- Arrow right from VG to LVs -->
  <line x1="520" y1="42" x2="545" y2="42" stroke="#585b70" stroke-width="1.5" marker-end="url(#arr0)"/>
  <text x="523" y="38" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">lvcreate</text>
  <!-- LV boxes stacked -->
  <text x="547" y="12" font-family="ui-monospace,monospace" font-size="8" fill="#585b70" letter-spacing="1">LOGICAL VOLUMES</text>
  <rect x="547" y="14" width="104" height="22" rx="4" fill="#2d1b4a" stroke="#cba6f7" stroke-width="1"/>
  <text x="599" y="29" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#cba6f7">lv_home  200G</text>
  <rect x="547" y="40" width="104" height="22" rx="4" fill="#3a2010" stroke="#fab387" stroke-width="1"/>
  <text x="599" y="55" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#fab387">lv_var   800G</text>
  <!-- Mount points -->
  <text x="8" y="100" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">How it all maps together:</text>
  <text x="8" y="118" font-family="ui-monospace,monospace" font-size="10" fill="#cdd6f4">  /dev/sda  +  /dev/sdb  +  /dev/sdc   →   vg_data   →   /home  /var  /data</text>
  <text x="8" y="138" font-family="ui-monospace,monospace" font-size="10" fill="#585b70">  500 GB        1 TB         2 TB           3.5 TB       resizable at any time</text>
  <text x="8" y="162" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1">  ✓  Multiple disks become one pool</text>
  <text x="200" y="162" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1">  ✓  Resize any volume live</text>
  <text x="390" y="162" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1">  ✓  Snapshots in seconds</text>
</svg>
</div>

---

## Where to go next

| Page | What you'll learn |
|---|---|
| [The Three Layers](01-concepts/) | PVs, VGs, and LVs explained with diagrams |
| [Your First LVM Setup](02-setup/) | Step-by-step: create PV → VG → LV → mount |
| [Resizing Volumes](03-resize/) | Grow live, shrink safely, add new disks |
| [Snapshots](04-snapshots/) | Instant backups with copy-on-write |
| [LVM with SAN / LUNs on Proxmox](05-san-luns-proxmox/) | iSCSI, Fibre Channel, multipath, Proxmox storage |
| [VMware vs Proxmox Storage](06-vmware-vs-proxmox/) | Concept mapping: VMFS, VMDK, RDM, vSAN → Proxmox |
| [Migrating VMs from VMware](07-migration/) | OVA export, qemu-img, virt-v2v, SAN reuse |
| [Cheat Sheet](cheatsheet/) | All commands in one place |
