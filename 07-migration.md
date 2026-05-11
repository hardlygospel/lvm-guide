---
title: 7. Migrating VMs from VMware
nav_order: 9
---

# Migrating VMs from VMware to Proxmox

This chapter walks through the full process: exporting VMs from VMware/ESXi, converting them, importing them into Proxmox, and making them boot correctly.

{: .note }
Chapter 6 covers the conceptual differences. This chapter is the hands-on procedure.

---

## Overview: three migration paths

<div style="overflow-x:auto;margin:2rem 0;">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 760 320" style="width:100%;max-width:760px;display:block;margin:0 auto;">
  <defs>
    <marker id="arr7" viewBox="0 0 10 8" refX="10" refY="4" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M0,0 L10,4 L0,8 Z" fill="#585b70"/>
    </marker>
  </defs>
  <rect width="760" height="320" rx="10" fill="#181825"/>
  <text x="380" y="28" text-anchor="middle" font-family="ui-monospace,monospace" font-size="13" fill="#cdd6f4" font-weight="bold">VMware → Proxmox Migration Paths</text>

  <!-- Path A -->
  <rect x="20"  y="50" width="140" height="40" rx="5" fill="#1a2a4a" stroke="#89b4fa"/>
  <text x="90"  y="65" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#89b4fa">VMware ESXi</text>
  <text x="90"  y="79" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">running VM</text>
  <line x1="160" y1="70" x2="200" y2="70" stroke="#585b70" stroke-width="1.5" marker-end="url(#arr7)"/>
  <text x="180" y="65" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">OVA/OVF</text>
  <text x="180" y="78" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">export</text>
  <rect x="200" y="50" width="140" height="40" rx="5" fill="#2d1b4a" stroke="#cba6f7"/>
  <text x="270" y="65" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#cba6f7">qemu-img convert</text>
  <text x="270" y="79" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">VMDK → qcow2/raw</text>
  <line x1="340" y1="70" x2="380" y2="70" stroke="#585b70" stroke-width="1.5" marker-end="url(#arr7)"/>
  <text x="360" y="65" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">qm import</text>
  <rect x="380" y="50" width="140" height="40" rx="5" fill="#1a3320" stroke="#a6e3a1"/>
  <text x="450" y="65" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1">Proxmox VM</text>
  <text x="450" y="79" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">converted disk</text>
  <rect x="560" y="52" width="150" height="36" rx="4" fill="#3a2010" stroke="#fab387"/>
  <text x="635" y="66" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#fab387" font-weight="bold">Path A — Manual</text>
  <text x="635" y="80" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#585b70">Full control, any ESXi version</text>

  <!-- Path B -->
  <rect x="20"  y="120" width="140" height="40" rx="5" fill="#1a2a4a" stroke="#89b4fa"/>
  <text x="90"  y="135" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#89b4fa">VMware ESXi</text>
  <text x="90"  y="149" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">running VM</text>
  <line x1="160" y1="140" x2="200" y2="140" stroke="#585b70" stroke-width="1.5" marker-end="url(#arr7)"/>
  <text x="180" y="135" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">virt-v2v</text>
  <text x="180" y="148" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">direct pull</text>
  <rect x="200" y="120" width="140" height="40" rx="5" fill="#1a3320" stroke="#a6e3a1"/>
  <text x="270" y="135" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1">Proxmox VM</text>
  <text x="270" y="149" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">auto-imported</text>
  <rect x="560" y="122" width="150" height="36" rx="4" fill="#3a2010" stroke="#fab387"/>
  <text x="635" y="136" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#fab387" font-weight="bold">Path B — virt-v2v</text>
  <text x="635" y="150" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#585b70">Automated, handles drivers</text>

  <!-- Path C (SAN reuse) -->
  <rect x="20"  y="190" width="140" height="40" rx="5" fill="#1a2a4a" stroke="#89b4fa"/>
  <text x="90"  y="205" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#89b4fa">SAN LUN</text>
  <text x="90"  y="219" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">VMFS on LUN</text>
  <line x1="160" y1="210" x2="200" y2="210" stroke="#585b70" stroke-width="1.5" marker-end="url(#arr7)"/>
  <text x="180" y="203" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">reformat or</text>
  <text x="180" y="216" text-anchor="middle" font-family="ui-monospace,monospace" font-size="7" fill="#45475a">reuse LUN</text>
  <rect x="200" y="190" width="140" height="40" rx="5" fill="#1a3320" stroke="#a6e3a1"/>
  <text x="270" y="205" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1">LVM on SAN</text>
  <text x="270" y="219" text-anchor="middle" font-family="ui-monospace,monospace" font-size="9" fill="#585b70">Proxmox storage</text>
  <rect x="560" y="192" width="150" height="36" rx="4" fill="#3a2010" stroke="#fab387"/>
  <text x="635" y="206" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#fab387" font-weight="bold">Path C — SAN reuse</text>
  <text x="635" y="220" text-anchor="middle" font-family="ui-monospace,monospace" font-size="8" fill="#585b70">No data copy, fastest cutover</text>

  <!-- Labels -->
  <text x="20"  y="270" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1">Use Path A when: you have OVA exports or ESXi access is limited</text>
  <text x="20"  y="288" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1">Use Path B when: you want automation and driver fixes applied automatically</text>
  <text x="20"  y="306" font-family="ui-monospace,monospace" font-size="9" fill="#a6e3a1">Use Path C when: VMs are on SAN and you're re-presenting LUNs to Proxmox</text>
</svg>
</div>

---

## Pre-migration checklist

Before you touch anything:

- [ ] Document every VM: name, OS, CPU, RAM, disk size, NIC count, IP addresses
- [ ] Check VMware Tools version — if missing, install before migration
- [ ] Note any RDM disks — these need special handling (see Path C)
- [ ] Verify Proxmox storage has enough free space (plan for 1.2× the used disk space)
- [ ] Check Proxmox node can reach the VMware ESXi host or vCenter (for virt-v2v)
- [ ] Confirm OS is supported: most Linux distros and Windows Server 2012+ work fine
- [ ] For Windows: download the VirtIO driver ISO in advance

---

## Path A — Manual: OVA export + qemu-img convert

### Step 1 — Export the VM from VMware

**From vCenter GUI:**
File → Export → Export OVF Template → check "Include image files in OVF package (OVA)"

**From ESXi CLI (faster for large VMs):**
```bash
# SSH to ESXi host
ovftool vi://user:password@esxi-host/vm-name /tmp/myvm.ova
```

You'll get a `.ova` file — this is a tar archive containing `.ovf` (config XML) and `.vmdk` (disk image).

### Step 2 — Extract the VMDK

```bash
# Unpack the OVA
tar xf myvm.ova

# You'll see files like:
# myvm.ovf
# myvm-disk1.vmdk
# myvm-disk1-flat.vmdk   (the actual data, if thick)
ls -lh *.vmdk
```

### Step 3 — Convert VMDK to qcow2 or raw

```bash
# Convert to qcow2 (recommended — portable, supports snapshots)
qemu-img convert -f vmdk -O qcow2 myvm-disk1.vmdk myvm-disk1.qcow2

# Or convert to raw (faster for LVM-thin storage)
qemu-img convert -f vmdk -O raw myvm-disk1.vmdk myvm-disk1.raw

# Check the result
qemu-img info myvm-disk1.qcow2
```

{: .tip }
For large disks, add `-p` to see progress: `qemu-img convert -p -f vmdk -O qcow2 ...`

### Step 4 — Create a blank VM in Proxmox

```bash
# Create VM with no disk (we'll attach it manually)
qm create 200 --name myvm --memory 4096 --cores 4 --net0 virtio,bridge=vmbr0
```

Or use the GUI: Datacenter → node → Create VM — skip the disk step.

### Step 5 — Import the disk

```bash
# Import qcow2 into local-lvm storage
qm importdisk 200 myvm-disk1.qcow2 local-lvm

# Import raw disk
qm importdisk 200 myvm-disk1.raw local-lvm

# The command prints the new disk ID, e.g.: vm-200-disk-0
```

### Step 6 — Attach the imported disk to the VM

```bash
# Attach as SCSI disk (recommended for Linux)
qm set 200 --scsi0 local-lvm:vm-200-disk-0

# Set boot order
qm set 200 --boot order=scsi0

# Add a CD drive (needed for VirtIO drivers on Windows)
qm set 200 --ide2 none,media=cdrom
```

### Step 7 — Attach VirtIO drivers for Windows VMs

VMware uses VMXNET3 and PVSCSI drivers. Proxmox uses VirtIO. Windows won't boot without drivers.

```bash
# Download VirtIO ISO on the Proxmox node
wget -O /var/lib/vz/template/iso/virtio-win.iso \
  https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso

# Attach it to the VM
qm set 200 --ide0 local:iso/virtio-win.iso,media=cdrom
```

Boot the VM, install drivers from the ISO, then shut it down and switch to VirtIO NICs:
```bash
qm set 200 --net0 virtio,bridge=vmbr0
```

### Step 8 — Import OVF directly (alternative)

If you have the `.ovf` + `.vmdk` separately, `qm importovf` handles everything:

```bash
qm importovf 201 myvm.ovf local-lvm
```

This creates the VM, sets CPU/RAM from the OVF, and imports all disks in one step.

---

## Path B — Automated: virt-v2v

`virt-v2v` is a tool that connects to ESXi, copies the VM, converts drivers, and imports directly into Proxmox. It handles Windows driver injection automatically.

### Install virt-v2v on the Proxmox node

```bash
apt install virt-v2v nbdkit
```

### Migrate a Linux VM directly

```bash
virt-v2v \
  -ic vpx://administrator@vcenter.example.com/Datacenter/esxi-host \
  -it vddk \
  --vddk-libdir /opt/vmware-vix-disklib/ \
  -o local -os /var/lib/vz/images/100 \
  "MyLinuxVM"
```

For simpler setups without VDDK (slower but no VMware SDK needed):
```bash
virt-v2v \
  -ic esx://root@esxi-host/?no_verify=1 \
  -o local -os /var/lib/vz/images/100 \
  "MyLinuxVM"
```

### What virt-v2v does automatically

- Copies all VM disks over the network
- Converts VMDK to raw/qcow2
- Removes VMware Tools
- Installs VirtIO drivers (for Windows: injects drivers into the disk offline)
- Reconfigures GRUB for KVM (if needed)
- Outputs a ready-to-boot libvirt XML or local disk

### Import the output into Proxmox

After virt-v2v finishes, import the disk and create the VM:
```bash
# Find the output disk
ls /var/lib/vz/images/100/

# Create VM and import
qm create 100 --name mylinuxvm --memory 4096 --cores 4
qm importdisk 100 /var/lib/vz/images/100/MyLinuxVM-sda /var/lib/vz/images/100/
qm set 100 --scsi0 local:100/vm-100-disk-0.raw --boot order=scsi0
```

---

## Path C — SAN LUN reuse

If your VMs live on SAN LUNs, you may be able to re-present the same LUN to Proxmox — completely skipping the data copy.

{: .warning }
Never present a VMFS-formatted LUN to Proxmox without wiping it first. Proxmox cannot read VMFS. You must migrate the VM data off first (Path A or B), then reformat the LUN for Proxmox.

### Strategy for SAN reuse

```
1. Use Path A or B to copy VM data off the LUN to a temporary location
2. Shut down all VMware VMs using the LUN
3. Remove the LUN from VMware (unmask / unzone it from ESXi)
4. Present the LUN to Proxmox (mask / zone it to Proxmox nodes)
5. Wipe VMFS: wipefs -a /dev/mapper/mpathX
6. Create LVM: pvcreate → vgcreate → add as Proxmox storage
7. Import VMs onto the now-clean LUN
```

This approach gives you maximum storage reuse. The downtime window is only steps 2–7 — typically under an hour for a pre-planned cutover.

---

## Post-migration: making VMs boot correctly

### Linux VMs

Most Linux VMs just work. If the VM boots but can't find its root filesystem:

```bash
# Boot into rescue mode, then:
# 1. Update /etc/fstab if it referenced VMware device names
blkid                          # get new UUIDs
vi /etc/fstab                  # update any /dev/sdX entries to UUID=...

# 2. Rebuild initramfs with VirtIO modules
update-initramfs -u -k all     # Debian/Ubuntu
dracut --force                 # RHEL/Rocky

# 3. Check GRUB references the right device
grep -r 'root=' /boot/grub*
```

### Windows VMs

The most common issue: Windows BSODs on first boot because the SCSI driver changed.

**Fix — inject VirtIO drivers before booting:**

If using virt-v2v, this is automatic. For manual migrations:
1. Attach the VirtIO ISO to the VM in Proxmox
2. Boot the VM — it will BSOD
3. Boot from Windows ISO in repair mode
4. Open command prompt: `drvload D:\vioscsi\2k19\amd64\vioscsi.inf`
5. Reboot

Or — easier — before migrating, **change the SCSI controller to IDE** in VMware, migrate, boot in Proxmox, install VirtIO drivers, then switch back to VirtIO SCSI.

### Network driver changes

| VMware NIC | Proxmox equivalent | Notes |
|---|---|---|
| VMXNET3 | virtio | Better performance on Proxmox |
| E1000 | e1000 | Works but slower; change to virtio after migration |
| PVSCSI | virtio-scsi | Best performance on Proxmox |

After migration, update NIC in Proxmox:
```bash
# Change to virtio NIC
qm set 100 --net0 virtio,bridge=vmbr0,macaddr=XX:XX:XX:XX:XX:XX
```

Keep the same MAC address if the guest OS has IP/license tied to it.

---

## Removing VMware Tools from Linux guests

```bash
# Check if installed
vmware-toolsd --version 2>/dev/null || echo "not running"

# Debian/Ubuntu
apt remove open-vm-tools open-vm-tools-desktop

# RHEL/Rocky
dnf remove open-vm-tools

# If installed from tar (legacy):
/usr/bin/vmware-uninstall-tools.pl
```

Then install QEMU guest agent:
```bash
apt install qemu-guest-agent
systemctl enable --now qemu-guest-agent
```

Enable in Proxmox:
```bash
qm set 100 --agent 1
```

---

## Migrating SAN-attached VMs (RDM → disk passthrough)

If a VMware VM uses RDM to access a LUN directly:

1. **Identify the LUN WWN** from VMware storage adapter view
2. **Re-zone/mask the LUN** from ESXi to Proxmox node (coordinate with SAN admin)
3. **Verify Proxmox sees it:**
   ```bash
   multipath -ll   # find the mpath device
   ls /dev/disk/by-id/ | grep wwn
   ```
4. **Attach to VM:**
   ```bash
   qm set 200 --scsi1 /dev/disk/by-id/wwn-0x5000c50015ea71aa,cache=none
   ```

No data conversion needed — the guest OS reads the same raw blocks.

---

## Migration tracking table

Useful format for planning a large migration:

| VM Name | OS | Size | Path | Exported | Converted | Imported | Tested | DNS updated |
|---|---|---|---|---|---|---|---|---|
| web01 | Ubuntu 22.04 | 80G | A | ✓ | ✓ | ✓ | ✓ | ✓ |
| db01 | RHEL 9 | 500G | B | ✓ | auto | ✓ | - | - |
| sql01 | Win Server 2019 | 200G | A | - | - | - | - | - |

---

## Quick command reference

```bash
# Export OVA from ESXi
ovftool vi://user:pass@host/vmname /tmp/export.ova

# Unpack OVA
tar xf export.ova

# Convert VMDK to qcow2
qemu-img convert -p -f vmdk -O qcow2 disk.vmdk disk.qcow2

# Convert VMDK to raw
qemu-img convert -p -f vmdk -O raw disk.vmdk disk.raw

# Create blank VM
qm create 200 --name myvm --memory 4096 --cores 4 --net0 virtio,bridge=vmbr0

# Import disk to storage
qm importdisk 200 disk.qcow2 local-lvm

# Attach imported disk
qm set 200 --scsi0 local-lvm:vm-200-disk-0 --boot order=scsi0

# Import entire OVF
qm importovf 201 myvm.ovf local-lvm

# List all VMs
qm list

# Start VM
qm start 200

# Check boot logs
qm terminal 200
```

---

## Summary

| Task | Command / tool |
|---|---|
| Export from VMware | `ovftool` or vCenter GUI |
| Convert disk format | `qemu-img convert -f vmdk -O qcow2` |
| Create Proxmox VM | `qm create` or GUI |
| Import disk | `qm importdisk` |
| Import full OVF | `qm importovf` |
| Automated migration | `virt-v2v` |
| Fix Linux boot | Update `/etc/fstab` UUIDs, rebuild initramfs |
| Fix Windows boot | VirtIO drivers from ISO |
| Remove VMware Tools | `apt remove open-vm-tools` |
| Install guest agent | `apt install qemu-guest-agent` |
| SAN LUN passthrough | `qm set --scsi1 /dev/disk/by-id/wwn-...` |
