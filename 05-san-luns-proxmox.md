---
title: 5. SAN, LUNs & Proxmox
nav_order: 7
---

# SAN, LUNs and Proxmox

This chapter is about connecting enterprise shared storage to Proxmox and using LVM on top of it. If you already know LVM from previous chapters, this is where it gets applied in the real world.

---

## What is a SAN?

A **SAN (Storage Area Network)** is a dedicated high-speed network whose only job is carrying storage traffic between servers and storage arrays. It is separate from your regular LAN.

Think of it this way:
- Your LAN carries emails, web traffic, user data
- Your SAN carries only one thing: reads and writes to disks

The storage array itself is a box full of fast disks, managed centrally. Instead of each server having its own local disks, they all reach out to the array over the SAN fabric and get blocks of storage — these blocks are called **LUNs**.

---

## What is a LUN?

A **LUN (Logical Unit Number)** is a slice of storage from the array, presented to a server as if it were a local disk.

From Linux's perspective, a LUN looks identical to `/dev/sda` — it's just a block device. You can run `fdisk`, `pvcreate`, `mkfs.ext4`, or anything else on it. The fact that it lives on a SAN a rack away is invisible to the operating system.

```
Storage Array                      Linux Server
─────────────────────────────      ─────────────────────────
  Internal disks: 48 × 4TB         "I have a disk /dev/sdb"
  
  LUN 0: 2 TB slice ──────────────►  /dev/sdb  (to the OS it's just a disk)
  LUN 1: 500 GB slice ────────────►  /dev/sdc
  LUN 2: 4 TB slice ──────────────►  /dev/sdd
```

The assignment of LUNs to servers is done through:
- **LUN masking** — the array decides which server can see which LUN
- **Zoning** (FC only) — the fabric switch decides which HBA ports can talk to which storage ports

---

## SAN protocols: iSCSI vs Fibre Channel

Two protocols dominate enterprise SANs:

| | iSCSI | Fibre Channel (FC) |
|---|---|---|
| **Physical network** | Standard Ethernet (10GbE, 25GbE) | Dedicated FC switches (4/8/16/32 Gbps) |
| **Cost** | Lower — uses existing network gear | Higher — dedicated HBAs and switches |
| **Setup** | Software initiator built into Linux | Requires HBA cards |
| **Performance** | Very good on 10GbE+ | Excellent, low latency |
| **Common in** | SMB, smaller enterprise, homelab | Large enterprise, finance, healthcare |
| **Linux device** | `/dev/sdX` via `open-iscsi` | `/dev/sdX` via HBA driver |

Both result in the same thing on Linux — a block device you can use with LVM.

---

## The full architecture

<div style="overflow-x:auto;margin:2rem 0;">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 760 380" style="width:100%;max-width:760px;display:block;margin:0 auto;">
  <defs>
    <marker id="a5a" viewBox="0 0 10 8" refX="10" refY="4" markerWidth="6" markerHeight="6" orient="auto"><path d="M0,0 L10,4 L0,8 Z" fill="#585b70"/></marker>
    <marker id="a5g" viewBox="0 0 10 8" refX="10" refY="4" markerWidth="6" markerHeight="6" orient="auto"><path d="M0,0 L10,4 L0,8 Z" fill="#a6e3a1"/></marker>
    <marker id="a5o" viewBox="0 0 10 8" refX="10" refY="4" markerWidth="6" markerHeight="6" orient="auto"><path d="M0,0 L10,4 L0,8 Z" fill="#fab387"/></marker>
  </defs>
  <rect width="760" height="380" rx="12" fill="#181825"/>
  <text x="380" y="24" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#6c7086" letter-spacing="2">SAN + PROXMOX CLUSTER ARCHITECTURE</text>

  <!-- Storage Array -->
  <rect x="10" y="36" width="150" height="220" rx="8" fill="#1e2b4a" stroke="#89b4fa" stroke-width="1.5"/>
  <text x="85" y="56" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#89b4fa" letter-spacing="1">STORAGE ARRAY</text>
  <rect x="22" y="64" width="126" height="24" rx="4" fill="#2d1b4a" stroke="#cba6f7" stroke-width="1"/>
  <text x="85" y="80" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#cba6f7">LUN 0 — 2 TB</text>
  <rect x="22" y="93" width="126" height="24" rx="4" fill="#2d1b4a" stroke="#cba6f7" stroke-width="1"/>
  <text x="85" y="109" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#cba6f7">LUN 1 — 1 TB</text>
  <rect x="22" y="122" width="126" height="24" rx="4" fill="#2d1b4a" stroke="#cba6f7" stroke-width="1"/>
  <text x="85" y="138" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#cba6f7">LUN 2 — 500 GB</text>
  <rect x="22" y="151" width="126" height="24" rx="4" fill="#2d1b4a" stroke="#cba6f7" stroke-width="1"/>
  <text x="85" y="167" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#cba6f7">LUN 3 — 4 TB</text>
  <text x="85" y="210" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">LUN masking</text>
  <text x="85" y="224" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">controls access</text>
  <text x="85" y="244" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">2× redundant</text>
  <text x="85" y="256" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">storage controllers</text>

  <!-- Fabric A switch -->
  <rect x="200" y="60" width="80" height="50" rx="6" fill="#1a3320" stroke="#a6e3a1" stroke-width="1.5"/>
  <text x="240" y="81" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1" font-weight="bold">FC Switch</text>
  <text x="240" y="98" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">Fabric A</text>

  <!-- Fabric B switch -->
  <rect x="200" y="160" width="80" height="50" rx="6" fill="#3a2010" stroke="#fab387" stroke-width="1.5"/>
  <text x="240" y="181" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#fab387" font-weight="bold">FC Switch</text>
  <text x="240" y="198" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">Fabric B</text>

  <!-- Array → Switches -->
  <line x1="160" y1="100" x2="200" y2="90" stroke="#a6e3a1" stroke-width="1.5" marker-end="url(#a5g)"/>
  <line x1="160" y1="170" x2="200" y2="188" stroke="#fab387" stroke-width="1.5" marker-end="url(#a5o)"/>

  <!-- Labels on fabric lines -->
  <text x="168" y="94" font-family="ui-monospace,monospace" font-size="8" fill="#a6e3a1">Path A</text>
  <text x="168" y="180" font-family="ui-monospace,monospace" font-size="8" fill="#fab387">Path B</text>

  <!-- Proxmox Node 1 -->
  <rect x="330" y="36" width="180" height="220" rx="8" fill="#1e1e2e" stroke="#45475a" stroke-width="1.5"/>
  <text x="420" y="56" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#cdd6f4" letter-spacing="1">PROXMOX NODE 1</text>
  <rect x="345" y="64" width="70" height="32" rx="4" fill="#1e2b4a" stroke="#89b4fa" stroke-width="1"/>
  <text x="380" y="77" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#89b4fa">HBA 0</text>
  <text x="380" y="90" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">Fabric A port</text>
  <rect x="425" y="64" width="70" height="32" rx="4" fill="#3a2010" stroke="#fab387" stroke-width="1"/>
  <text x="460" y="77" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#fab387">HBA 1</text>
  <text x="460" y="90" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">Fabric B port</text>
  <!-- MPIO box -->
  <rect x="345" y="110" width="150" height="36" rx="4" fill="#1a3320" stroke="#a6e3a1" stroke-width="1"/>
  <text x="420" y="126" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1" font-weight="bold">dm-multipath</text>
  <text x="420" y="140" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">combines both paths</text>
  <!-- Device path -->
  <rect x="345" y="158" width="150" height="30" rx="4" fill="#313244" stroke="#585b70" stroke-width="1"/>
  <text x="420" y="178" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#cdd6f4">/dev/mapper/mpath0</text>
  <!-- LVM stack -->
  <rect x="345" y="198" width="150" height="22" rx="4" fill="#2d1b4a" stroke="#cba6f7" stroke-width="1"/>
  <text x="420" y="213" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#cba6f7">LVM PV → VG → LV</text>
  <rect x="345" y="228" width="150" height="22" rx="4" fill="#10303a" stroke="#89dceb" stroke-width="1"/>
  <text x="420" y="243" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#89dceb">Proxmox VM disk</text>

  <!-- Proxmox Node 2 -->
  <rect x="530" y="36" width="180" height="220" rx="8" fill="#1e1e2e" stroke="#45475a" stroke-width="1.5"/>
  <text x="620" y="56" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#cdd6f4" letter-spacing="1">PROXMOX NODE 2</text>
  <rect x="545" y="64" width="70" height="32" rx="4" fill="#1e2b4a" stroke="#89b4fa" stroke-width="1"/>
  <text x="580" y="77" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#89b4fa">HBA 0</text>
  <text x="580" y="90" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">Fabric A port</text>
  <rect x="625" y="64" width="70" height="32" rx="4" fill="#3a2010" stroke="#fab387" stroke-width="1"/>
  <text x="660" y="77" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#fab387">HBA 1</text>
  <text x="660" y="90" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">Fabric B port</text>
  <rect x="545" y="110" width="150" height="36" rx="4" fill="#1a3320" stroke="#a6e3a1" stroke-width="1"/>
  <text x="620" y="126" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1" font-weight="bold">dm-multipath</text>
  <text x="620" y="140" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">combines both paths</text>
  <rect x="545" y="158" width="150" height="30" rx="4" fill="#313244" stroke="#585b70" stroke-width="1"/>
  <text x="620" y="178" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#cdd6f4">/dev/mapper/mpath0</text>
  <rect x="545" y="198" width="150" height="22" rx="4" fill="#2d1b4a" stroke="#cba6f7" stroke-width="1"/>
  <text x="620" y="213" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#cba6f7">LVM PV → VG → LV</text>
  <rect x="545" y="228" width="150" height="22" rx="4" fill="#10303a" stroke="#89dceb" stroke-width="1"/>
  <text x="620" y="243" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#89dceb">Proxmox VM disk</text>

  <!-- Switch → Nodes (Path A - green) -->
  <line x1="280" y1="82" x2="330" y2="80" stroke="#a6e3a1" stroke-width="1.5" marker-end="url(#a5g)"/>
  <line x1="280" y1="88" x2="530" y2="78" stroke="#a6e3a1" stroke-width="1" stroke-dasharray="4,3"/>

  <!-- Switch → Nodes (Path B - orange) -->
  <line x1="280" y1="185" x2="330" y2="135" stroke="#fab387" stroke-width="1.5" marker-end="url(#a5o)"/>
  <line x1="280" y1="192" x2="530" y2="132" stroke="#fab387" stroke-width="1" stroke-dasharray="4,3"/>

  <!-- Bottom note -->
  <rect x="10" y="270" width="740" height="50" rx="6" fill="#1e1e2e" stroke="#313244" stroke-width="1"/>
  <text x="380" y="290" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1">⚠  Shared SAN LUNs: each node sees the same LUN — co-ordinate access carefully.</text>
  <text x="380" y="310" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#45475a">For shared VM storage across nodes use Proxmox + iSCSI/LVM with one active node, or Ceph for true concurrent shared storage.</text>
</svg>
</div>

---

## Multipath I/O — why it matters and how to set it up

In production, a server connects to the SAN via **two separate physical paths** — different HBAs, different cables, different switches. This is called multipath and gives you:

- **Redundancy** — if one path fails, the other takes over automatically
- **Load balancing** — traffic can be spread across both paths

Without multipath configured, Linux sees two separate block devices for the same LUN (`/dev/sdb` and `/dev/sdc` might be the same LUN via different paths). Writing to both would corrupt data. **dm-multipath** combines them into one logical device.

### Setting up multipath on Proxmox / Debian

```bash
# Install
apt install multipath-tools

# Enable on boot
systemctl enable multipathd
systemctl start multipathd

# Check what LUNs are visible
multipath -ll

# Typical output:
# mpatha (360000000000000001) dm-0 VENDOR,PRODUCT
# size=2.0T features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
# |-+- policy='service-time 0' prio=50 status=active
# | `- 2:0:0:1  sdb 8:16  active ready running
# `-+- policy='service-time 0' prio=10 status=enabled
#   `- 3:0:0:1  sdc 8:32  active ready running
```

The LUN now appears as `/dev/mapper/mpatha` (or `/dev/mapper/mpath0` — the name depends on your config). **Always use the multipath device path, not the raw `/dev/sdb` or `/dev/sdc` paths.**

### /etc/multipath.conf — basic setup

```bash
defaults {
    user_friendly_names yes    # names like mpatha instead of WWID
    find_multipaths yes
    polling_interval 5
}

blacklist {
    devnode "^sda"             # Don't multipath your boot disk
}
```

After editing:
```bash
systemctl restart multipathd
multipath -ll                  # Verify
```

---

## iSCSI with Proxmox — step by step

iSCSI uses your existing Ethernet network. The storage array is the **target** (server side), your Proxmox host is the **initiator** (client side).

### Step 1 — Install and configure the initiator

```bash
apt install open-iscsi

# Set the initiator name (unique per node — change node1 for each host)
echo "InitiatorName=iqn.2024-01.com.proxmox:node1" > /etc/iscsi/initiatorname.iscsi

systemctl enable iscsid
systemctl start iscsid
```

### Step 2 — Discover targets on the SAN

```bash
# Replace 192.168.10.100 with your SAN's IP
iscsiadm -m discovery -t sendtargets -p 192.168.10.100
```

Output:
```
192.168.10.100:3260,1 iqn.2020-01.com.storage:target1
192.168.10.100:3260,1 iqn.2020-01.com.storage:target2
```

### Step 3 — Log in to a target

```bash
iscsiadm -m node -T iqn.2020-01.com.storage:target1 -p 192.168.10.100 --login

# Make it persist across reboots
iscsiadm -m node -T iqn.2020-01.com.storage:target1 -o update -n node.startup -v automatic
```

### Step 4 — Verify the LUN appeared

```bash
lsblk
# You'll see a new /dev/sdX device — this is your LUN
```

---

## Proxmox storage types for SAN

Proxmox has several storage backends that work with SAN LUNs. These are configured in `/etc/pve/storage.cfg` or the Proxmox web GUI under **Datacenter → Storage → Add**.

### Option A: iSCSI + LVM (recommended for block storage)

Use iSCSI to present the LUN, then LVM on top to manage VM disks:

```
# /etc/pve/storage.cfg

iscsi: san-iscsi
    portal 192.168.10.100
    target iqn.2020-01.com.storage:target1
    content none               # iSCSI itself doesn't store VMs directly

lvm: san-lvm
    base san-iscsi:0.0.0.1000000000000   # references the iSCSI LUN
    vgname vg_san
    content rootdir,images
    shared 0                   # set to 1 if multiple nodes share this VG (needs CLVM)
```

Or add via GUI: **Datacenter → Storage → Add → LVM**

### Option B: iSCSI direct (LUNs as raw VM disks)

Each LUN becomes a potential VM disk. No LVM, less flexible:

```
iscsi: san-raw
    portal 192.168.10.100
    target iqn.2020-01.com.storage:target1
    content images
```

### Option C: LVM-thin over SAN LUN

Best for VM environments — enables thin provisioning and fast snapshots:

```
lvmthin: san-thin
    thinpool data
    vgname vg_san
    content rootdir,images
```

{: .tip }
**LVM-thin is the closest Proxmox equivalent to VMware's VMFS thin-provisioned datastores.** Use it when you want snapshots and clone efficiency.

---

## LUN to VM disk — the full stack

<div style="overflow-x:auto;margin:2rem 0;">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 700 110" style="width:100%;max-width:700px;display:block;margin:0 auto;">
  <defs>
    <marker id="a5r" viewBox="0 0 10 8" refX="10" refY="4" markerWidth="6" markerHeight="6" orient="auto"><path d="M0,0 L10,4 L0,8 Z" fill="#585b70"/></marker>
  </defs>
  <rect width="700" height="110" rx="10" fill="#181825"/>
  <!-- Boxes left to right -->
  <rect x="10"  y="30" width="90"  height="50" rx="6" fill="#1e2b4a" stroke="#89b4fa" stroke-width="1.5"/>
  <text x="55"  y="52" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#89b4fa" font-weight="bold">SAN LUN</text>
  <text x="55"  y="68" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">2 TB block</text>

  <line x1="100" y1="55" x2="120" y2="55" stroke="#585b70" stroke-width="1.5" marker-end="url(#a5r)"/>
  <text x="110" y="48" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">MPIO</text>

  <rect x="120" y="30" width="100" height="50" rx="6" fill="#1e2b4a" stroke="#a6e3a1" stroke-width="1.5"/>
  <text x="170" y="52" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1" font-weight="bold">/dev/mapper</text>
  <text x="170" y="66" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">/mpatha</text>

  <line x1="220" y1="55" x2="240" y2="55" stroke="#585b70" stroke-width="1.5" marker-end="url(#a5r)"/>
  <text x="230" y="48" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">pvcreate</text>

  <rect x="240" y="30" width="80" height="50" rx="6" fill="#2d1b4a" stroke="#cba6f7" stroke-width="1.5"/>
  <text x="280" y="52" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#cba6f7" font-weight="bold">PV</text>
  <text x="280" y="66" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">lvm header</text>

  <line x1="320" y1="55" x2="340" y2="55" stroke="#585b70" stroke-width="1.5" marker-end="url(#a5r)"/>
  <text x="330" y="48" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">vgcreate</text>

  <rect x="340" y="30" width="80" height="50" rx="6" fill="#1a3320" stroke="#a6e3a1" stroke-width="1.5"/>
  <text x="380" y="52" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1" font-weight="bold">vg_san</text>
  <text x="380" y="66" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">2 TB pool</text>

  <line x1="420" y1="55" x2="440" y2="55" stroke="#585b70" stroke-width="1.5" marker-end="url(#a5r)"/>
  <text x="430" y="48" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">Proxmox</text>

  <rect x="440" y="30" width="110" height="50" rx="6" fill="#10303a" stroke="#89dceb" stroke-width="1.5"/>
  <text x="495" y="50" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#89dceb" font-weight="bold">vm-100-disk-0</text>
  <text x="495" y="64" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">logical volume</text>

  <line x1="550" y1="55" x2="570" y2="55" stroke="#585b70" stroke-width="1.5" marker-end="url(#a5r)"/>

  <rect x="570" y="30" width="120" height="50" rx="6" fill="#3a2010" stroke="#fab387" stroke-width="1.5"/>
  <text x="630" y="50" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#fab387" font-weight="bold">Proxmox VM</text>
  <text x="630" y="64" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#45475a">sees it as virtio disk</text>

  <text x="350" y="100" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#585b70">SAN LUN → multipath device → LVM Physical Volume → Volume Group → Logical Volume → VM disk</text>
</svg>
</div>

---

## Shared SAN storage in a Proxmox cluster

{: .warning }
**Do not use standard LVM on a SAN LUN that multiple Proxmox nodes access simultaneously.** Standard LVM does not have cluster awareness — two nodes writing to the same VG will corrupt it.

Your options for shared SAN in a Proxmox cluster:

| Approach | How it works | Use case |
|---|---|---|
| **One node owns the VG** | Only one node has the iSCSI/FC connection active | HA with node pinning |
| **iSCSI + LVM shared=1** | LVM with file-based locking via Proxmox cluster | Active/passive HA |
| **Ceph** | Software-defined storage, built into Proxmox | True concurrent shared storage |
| **NFS on top of SAN** | SAN presents NFS share, all nodes mount it | Simple, works everywhere |

For most enterprise migrations from VMware (where all nodes shared VMFS datastores), **Ceph** is the closest equivalent to vSAN, and **NFS over SAN** is the simplest drop-in replacement for a shared VMFS datastore.

---

## Adding SAN storage in the Proxmox GUI

**Datacenter → Storage → Add → iSCSI**

| Field | Value |
|---|---|
| ID | `san-iscsi` (your label) |
| Portal | `192.168.10.100` (SAN IP) |
| Target | Select from dropdown (auto-discovered) |
| Content | `Disk image, Container` |

Then add LVM on top:

**Datacenter → Storage → Add → LVM**

| Field | Value |
|---|---|
| ID | `san-lvm` |
| Base Storage | `san-iscsi` |
| Base Volume | Select the LUN |
| Volume group | `vg_san` |
| Content | `Disk image, Container` |
