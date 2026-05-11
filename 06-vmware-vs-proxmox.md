---
title: 6. VMware vs Proxmox Storage
nav_order: 8
---

# VMware vs Proxmox: Storage Concepts

If you've managed VMware/ESXi and you're moving to Proxmox, everything works — but it has a different name and a different shape. This chapter maps every VMware storage concept to its Proxmox equivalent.

{: .note }
This chapter is about **understanding the conceptual differences**. Chapter 7 covers the actual migration steps.

---

## The core difference in philosophy

**VMware** bundles storage management into vSphere: VMFS, datastores, and RDM are proprietary formats tightly coupled to ESXi.

**Proxmox** uses the Linux storage stack directly: LVM, ZFS, Ceph, NFS, iSCSI — all standard tools you can use from the command line without Proxmox at all.

---

## Side-by-side comparison

<div style="overflow-x:auto;margin:2rem 0;">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 760 540" style="width:100%;max-width:760px;display:block;margin:0 auto;">
  <defs>
    <marker id="arr6" viewBox="0 0 10 8" refX="10" refY="4" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M0,0 L10,4 L0,8 Z" fill="#585b70"/>
    </marker>
  </defs>
  <rect width="760" height="540" rx="10" fill="#181825"/>

  <!-- Header -->
  <rect x="10"  y="10" width="365" height="36" rx="6" fill="#1a2a4a" stroke="#89b4fa" stroke-width="1.5"/>
  <text x="192" y="34" text-anchor="middle" font-family="ui-monospace,monospace" font-size="14" fill="#89b4fa" font-weight="bold">VMware / ESXi</text>
  <rect x="385" y="10" width="365" height="36" rx="6" fill="#1a3320" stroke="#a6e3a1" stroke-width="1.5"/>
  <text x="567" y="34" text-anchor="middle" font-family="ui-monospace,monospace" font-size="14" fill="#a6e3a1" font-weight="bold">Proxmox VE</text>

  <!-- Row 1: Storage pool -->
  <rect x="10"  y="58" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="192" y="78" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#cdd6f4" font-weight="bold">Datastore</text>
  <text x="192" y="96" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Named storage bucket — VMs stored here</text>
  <text x="192" y="108" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">e.g. "datastore1"  on VMFS or NFS</text>
  <rect x="385" y="58" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="567" y="78" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#cdd6f4" font-weight="bold">Storage</text>
  <text x="567" y="96" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Configured in /etc/pve/storage.cfg</text>
  <text x="567" y="108" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">e.g. "local-lvm"  "ceph-pool"  "nfs-backup"</text>
  <text x="375" y="92" text-anchor="middle" font-family="ui-monospace,monospace" font-size="14" fill="#585b70">≈</text>

  <!-- Row 2: Block filesystem -->
  <rect x="10"  y="126" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="192" y="146" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#89b4fa" font-weight="bold">VMFS</text>
  <text x="192" y="163" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">VMware's cluster-aware block filesystem</text>
  <text x="192" y="175" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Proprietary, ESXi-only, on raw block devices</text>
  <rect x="385" y="126" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="567" y="146" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#a6e3a1" font-weight="bold">LVM-thin / ZFS / ext4 / xfs</text>
  <text x="567" y="163" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Standard Linux filesystems</text>
  <text x="567" y="175" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Open, portable, works on any Linux</text>
  <text x="375" y="160" text-anchor="middle" font-family="ui-monospace,monospace" font-size="14" fill="#585b70">→</text>

  <!-- Row 3: VM disk format -->
  <rect x="10"  y="194" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="192" y="214" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#89b4fa" font-weight="bold">VMDK</text>
  <text x="192" y="231" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Virtual Machine Disk format</text>
  <text x="192" y="243" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Can be thick (pre-alloc) or thin (lazy)</text>
  <rect x="385" y="194" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="567" y="214" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#a6e3a1" font-weight="bold">qcow2 / raw</text>
  <text x="567" y="231" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">qcow2 = thin, snapshots, portable</text>
  <text x="567" y="243" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">raw = thick/fast, no overhead</text>
  <text x="375" y="228" text-anchor="middle" font-family="ui-monospace,monospace" font-size="14" fill="#585b70">→</text>

  <!-- Row 4: RDM -->
  <rect x="10"  y="262" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="192" y="282" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#89b4fa" font-weight="bold">RDM (Raw Device Mapping)</text>
  <text x="192" y="299" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Pass a SAN LUN directly to a VM</text>
  <text x="192" y="311" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">VM sees raw block device, not a VMDK</text>
  <rect x="385" y="262" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="567" y="282" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#a6e3a1" font-weight="bold">Disk Passthrough</text>
  <text x="567" y="299" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">qm set &lt;vmid&gt; --scsi1 /dev/disk/by-id/...</text>
  <text x="567" y="311" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">VM gets raw block device access</text>
  <text x="375" y="296" text-anchor="middle" font-family="ui-monospace,monospace" font-size="14" fill="#585b70">≈</text>

  <!-- Row 5: vSAN -->
  <rect x="10"  y="330" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="192" y="350" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#89b4fa" font-weight="bold">vSAN</text>
  <text x="192" y="367" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Distributed storage from local disks</text>
  <text x="192" y="379" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Built into vSphere, proprietary</text>
  <rect x="385" y="330" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="567" y="350" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#a6e3a1" font-weight="bold">Ceph</text>
  <text x="567" y="367" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Distributed storage, fully integrated</text>
  <text x="567" y="379" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Open source, built into Proxmox GUI</text>
  <text x="375" y="364" text-anchor="middle" font-family="ui-monospace,monospace" font-size="14" fill="#585b70">≈</text>

  <!-- Row 6: Storage vMotion -->
  <rect x="10"  y="398" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="192" y="418" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#89b4fa" font-weight="bold">Storage vMotion</text>
  <text x="192" y="435" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Move VM disk to different datastore live</text>
  <text x="192" y="447" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Requires vMotion license</text>
  <rect x="385" y="398" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="567" y="418" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#a6e3a1" font-weight="bold">qm move_disk</text>
  <text x="567" y="435" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">qm move_disk &lt;vmid&gt; scsi0 &lt;storage&gt;</text>
  <text x="567" y="447" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Built-in, works online with QEMU snapshots</text>
  <text x="375" y="432" text-anchor="middle" font-family="ui-monospace,monospace" font-size="14" fill="#585b70">≈</text>

  <!-- Row 7: Tools -->
  <rect x="10"  y="466" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="192" y="486" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#89b4fa" font-weight="bold">VMware Tools</text>
  <text x="192" y="503" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Guest agent for heartbeat, freeze, snapshots</text>
  <text x="192" y="515" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Closed source, vmware-specific package</text>
  <rect x="385" y="466" width="365" height="56" rx="4" fill="#1e1e2e" stroke="#45475a"/>
  <text x="567" y="486" text-anchor="middle" font-family="ui-monospace,monospace" font-size="11" fill="#a6e3a1" font-weight="bold">QEMU Guest Agent</text>
  <text x="567" y="503" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">apt install qemu-guest-agent</text>
  <text x="567" y="515" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Open source, package in every distro</text>
  <text x="375" y="500" text-anchor="middle" font-family="ui-monospace,monospace" font-size="14" fill="#585b70">→</text>
</svg>
</div>

---

## Datastores → Proxmox Storage

In VMware, a **datastore** is where VM disks live. Proxmox uses the same concept but calls it **storage**, and it's configured in `/etc/pve/storage.cfg`.

### VMware datastore types and their Proxmox equivalents

| VMware | Proxmox | Notes |
|---|---|---|
| VMFS on local disk | `dir` or `lvm-thin` | Local block storage |
| VMFS on SAN/iSCSI | `lvm` or `lvmthin` over iSCSI | Block storage from SAN |
| NFS datastore | `nfs` | Identical concept, same protocol |
| vSAN datastore | `cephfs` or `rbd` (Ceph) | Distributed, cluster-wide |
| vVol | No direct equivalent | Use LVM-thin or Ceph instead |

### Checking Proxmox storage config

```bash
cat /etc/pve/storage.cfg
pvesm status          # Live status of all storages
pvesm list local-lvm  # List contents of a storage
```

---

## VMFS → LVM-thin (the most common switch)

VMFS is a cluster filesystem that lets multiple ESXi hosts share the same raw block device. LVM-thin on Proxmox is the equivalent for a single node. For cluster-shared block storage, use Ceph.

### What VMFS gave you

- Multiple VMs stored on one block device
- VM files named `<name>.vmdk`, `<name>-flat.vmdk`
- Thin-provisioned disks that grow as data is written
- Snapshot capability

### What LVM-thin gives you

- Multiple VMs stored in one LVM thin pool
- VM disks are LVM thin LVs (e.g., `vm-100-disk-0`)
- Thin-provisioned — only used blocks consume space
- Snapshot capability via LVM

```bash
# Check thin pool usage on Proxmox
lvs -o lv_name,pool_lv,data_percent,metadata_percent vg_name
```

---

## VMDK → qcow2 and raw

VMware stores VM disks as `.vmdk` files. Proxmox uses two formats:

| Format | VMware equivalent | When to use |
|---|---|---|
| `raw` | Thick-provisioned VMDK | Maximum performance, no overhead |
| `qcow2` | Thin-provisioned VMDK | Flexible, supports snapshots, portable |

When a VM disk lives in an LVM-thin storage, even a `raw` format disk gets thin-provisioning benefits from LVM — you get raw performance AND thin allocation.

---

## RDM → Disk Passthrough

In VMware, **Raw Device Mapping (RDM)** gives a VM direct access to a SAN LUN, bypassing VMFS. This is used for clustered applications (Oracle RAC, Windows Server Failover Clustering) that need to see raw block devices.

In Proxmox, this is **disk passthrough** — you pass the block device directly to the VM config:

```bash
# Find the stable device ID (always use by-id, not /dev/sdX)
ls -l /dev/disk/by-id/ | grep wwn

# Add the raw disk to VM 200
qm set 200 --scsi1 /dev/disk/by-id/wwn-0x5000c50015ea71aa,cache=none

# Verify
qm config 200
```

{: .warning }
Always use `/dev/disk/by-id/` paths, not `/dev/sdb`. Device names like `sdb` change between reboots; WWN-based IDs are stable.

---

## vSAN → Ceph

Both are software-defined distributed storage that pools local disks across cluster nodes. Ceph is the open-source equivalent of vSAN.

| Feature | vSAN | Ceph on Proxmox |
|---|---|---|
| Minimum nodes | 3 | 3 |
| Data protection | FTT (Failure to Tolerate) policy | Replication factor (usually 3) |
| Access | vSAN datastore | `rbd` (block) or `cephfs` (filesystem) |
| Config location | vCenter GUI | Proxmox GUI → Ceph section |
| CLI | `esxcli vsan` | `ceph status`, `ceph osd tree` |

```bash
# Check Ceph cluster health from any Proxmox node
ceph status
ceph osd tree
ceph df
```

---

## Storage vMotion → qm move_disk

Storage vMotion moves a running VM's disk from one datastore to another without downtime. Proxmox has `qm move_disk`:

```bash
# Move VM 100's scsi0 disk to ceph-pool storage
qm move_disk 100 scsi0 ceph-pool

# Move and delete the old copy once confirmed
qm move_disk 100 scsi0 ceph-pool --delete 1
```

The VM stays running. For Windows VMs, QEMU freezes the filesystem briefly (via guest agent) to ensure consistency.

---

## Thick vs Thin provisioning

VMware distinguishes between **Thick (Eager Zeroed)**, **Thick (Lazy Zeroed)**, and **Thin**. Proxmox maps to this naturally:

| VMware type | Proxmox equivalent | How |
|---|---|---|
| Thin provisioned | LVM-thin LV or qcow2 | Default on LVM-thin storage |
| Thick lazy zeroed | `raw` format on dir storage | `fallocate` pre-allocates without zeroing |
| Thick eager zeroed | `raw` + `dd if=/dev/zero` | Manual zeroing at creation |

For most workloads, LVM-thin (`raw` format on thin pool) gives you the best of both: thin allocation + raw performance.

---

## Snapshots

| Feature | VMware | Proxmox |
|---|---|---|
| Snapshot mechanism | VMFS delta disk (.vmdk) | LVM snapshot or qcow2 overlay |
| VM memory snapshot | Yes (suspend state) | Yes (with `-vmstate`) |
| Snapshot tree | Yes, GUI-managed | Yes, GUI-managed |
| Max recommended depth | 3 (performance degrades) | 3 (same guidance) |
| Snapshot deletion | Consolidate in vCenter | `qm delsnapshot` |

```bash
# Create a snapshot of VM 100
qm snapshot 100 pre-upgrade --description "Before kernel update"

# List snapshots
qm listsnapshot 100

# Rollback
qm rollback 100 pre-upgrade

# Delete snapshot
qm delsnapshot 100 pre-upgrade
```

---

## VMware Tools → QEMU Guest Agent

VMware Tools provides heartbeat, guest IP reporting, quiesced snapshots, and graceful shutdown. Proxmox uses the **QEMU guest agent** for the same functions.

**Linux guests:**
```bash
apt install qemu-guest-agent    # Debian/Ubuntu
dnf install qemu-guest-agent    # RHEL/Rocky
systemctl enable --now qemu-guest-agent
```

**Windows guests:** Install the VirtIO drivers ISO from Proxmox (includes the guest agent MSI).

Enable in Proxmox:
```bash
qm set 100 --agent 1
```

Or in the GUI: VM → Options → QEMU Guest Agent → enabled.

---

## Network storage: same protocols, different names

| What | VMware | Proxmox |
|---|---|---|
| NFS shared storage | NFS datastore | `nfs` storage type |
| iSCSI SAN | iSCSI adapter + VMFS | `iscsi` + `lvm` storage type |
| FC SAN | FC adapter + VMFS | `lvm` over FC device |
| SMB/CIFS | Not natively supported for VM disks | `cifs` storage type |

---

## Quick mental map

```
VMware concept          →    Proxmox equivalent
─────────────────────────────────────────────────
Datastore               →    Storage (storage.cfg)
VMFS                    →    LVM-thin or ZFS
VMDK (thin)             →    qcow2 or raw on LVM-thin
VMDK (thick eager)      →    raw + pre-zeroed
RDM                     →    disk passthrough (by-id)
vSAN                    →    Ceph
Storage vMotion         →    qm move_disk
VMware Tools            →    QEMU guest agent
vCenter inventory        →    Proxmox web GUI / pvesh
ESXi host               →    Proxmox node
```

---

Next: [Chapter 7 — Migrating VMs from VMware to Proxmox](../07-migration/)
