# Lab 07 — Storage Management

## Objective

Demonstrate the complete lifecycle of Linux storage administration by adding a new disk, creating a partition, formatting it with XFS, mounting it, configuring persistent mounting with `/etc/fstab`, and verifying functionality after a reboot.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation

## Concepts Covered

| Command | Purpose |
|----------|----------|
| lsblk | View disks and partitions |
| fdisk | Create partitions |
| mkfs.xfs | Create XFS filesystem |
| blkid | View filesystem UUIDs |
| mkdir | Create mount point |
| mount | Mount filesystem |
| umount | Unmount filesystem |
| df -h | Verify mounted storage |
| /etc/fstab | Configure persistent mounts |
| mount -a | Test fstab entries |

---

## Storage Workflow

Linux storage follows this general process:

```text
Disk
↓
Partition
↓
Filesystem
↓
Mount Point
↓
Mount
↓
Persistent Mount
↓
Verification
```

---

## Add New Disk in VMware

Before Linux can see a new disk, it has to exist at the hypervisor level first.

Open VM settings:

```text
VM > Settings > Hardware > Add... > Hard Disk
```

A new 10 GB virtual NVMe disk was added to the VM (shown here as "Hard Disk 3 (NVMe)").

![Add Disk in VMware](Lab07%20-%20storage%20management/02-add-disk-vmware.jpg)

Observation:

The disk exists in VMware but is not yet visible to the guest OS until the VM is powered on and `lsblk` is run.

---

## Detect New Disk

![Detect New Disk](Lab07%20-%20storage%20management/01-detect-new-disk.jpg)

A new 10 GB virtual disk was added to the VM.

Command:

```bash
lsblk
```

Result:

```text
nvme0n3
```

Observation:

The new disk contained no partitions or filesystems.

---

## Create Partition

![Create Partition with fdisk](Lab07%20-%20storage%20management/03-create-partition-fdisk.jpg)

Launch fdisk:

```bash
sudo fdisk /dev/nvme0n3
```

Actions performed:

```text
n
p
1
Enter
Enter
w
```

Verify:

```bash
lsblk
```

Result:

```text
nvme0n3
└─nvme0n3p1
```

Observation:

A new partition was created using the entire disk.

---

## Create Filesystem

![Create XFS Filesystem](Lab07%20-%20storage%20management/04-create-xfs-filesystem.jpg)

Create an XFS filesystem:

```bash
sudo mkfs.xfs /dev/nvme0n3p1
```

Observation:

The partition was formatted and prepared for use.

---

## Verify Filesystem, Create Mount Point, and Mount

Verify the filesystem was created correctly:

```bash
sudo blkid /dev/nvme0n3p1
```

Result:

```text
TYPE="xfs"
```

Create a directory to mount the new filesystem:

```bash
sudo mkdir /lab07data
```

Verify:

```bash
ls -ld /lab07data
```

Mount the filesystem:

```bash
sudo mount /dev/nvme0n3p1 /lab07data
```

Verify:

```bash
df -h
```

Result:

```text
/dev/nvme0n3p1
```

mounted on:

```text
/lab07data
```

![Verify, Mkdir, and Mount](Lab07%20-%20storage%20management/05-verify-mkdir-mount.jpg)

Observation:

The filesystem was confirmed as XFS, a mount point was created successfully, and the storage became available to the operating system.

---

## Configure Persistent Mount

Retrieve UUID:

```bash
sudo blkid /dev/nvme0n3p1
```

Example:

```text
UUID=f92de691-fb47-4016-95a8-58b123332003
```

Edit:

```bash
sudo nano /etc/fstab
```

Entry added:

```text
UUID=f92de691-fb47-4016-95a8-58b123332003 /lab07data xfs defaults 0 0
```

Observation:

The filesystem is configured to mount automatically at boot.

---

## Test fstab Configuration

Unmount:

```bash
sudo umount /lab07data
```

Verify:

```bash
df -h
```

Mount disappeared.

![Fstab Entry and Unmount](Lab07%20-%20storage%20management/06-fstab-entry-and-unmount.jpg)

Test the fstab entry:

```bash
sudo mount -a
```

Result:

```text
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

![Mount -a Failed - Troubleshooting](Lab07%20-%20storage%20management/07-mount-a-failed-troubleshoot.jpg)

Reload systemd to pick up the fstab change:

```bash
sudo systemctl daemon-reload
```

Test again:

```bash
sudo mount -a
```

Verify:

```bash
df -h
```

Result:

```text
/dev/nvme0n3p1
```

mounted on:

```text
/lab07data
```

![Mount -a Success](Lab07%20-%20storage%20management/08-mount-a-success.jpg)

Observation:

The fstab entry initially failed because systemd caches the fstab file in memory and doesn't automatically notice edits. Running `daemon-reload` forces systemd to re-read the file, after which `mount -a` succeeded.

---

## Reboot Verification

Reboot system:

```bash
sudo reboot
```

![Reboot Command Issued](Lab07%20-%20storage%20management/09-reboot-command-issued.jpg)

After reboot:

```bash
df -h
```

and:

```bash
lsblk
```

Result:

```text
nvme0n3p1
```

automatically mounted on:

```text
/lab07data
```

![Post-Reboot Verification](Lab07%20-%20storage%20management/10-post-reboot-verification.jpg)

Observation:

Persistent mounting worked successfully.

---

## Key Lessons Learned

- Disks must be partitioned before use.
- Filesystems must be created before data can be stored.
- A mount point provides a location where storage becomes accessible.
- UUIDs should be used in `/etc/fstab` for reliable mounting.
- `mount -a` is an excellent way to test fstab entries before rebooting.
- systemd caches `/etc/fstab` in memory, so edits require `systemctl daemon-reload` before `mount -a` will pick them up.
- A successful reboot confirms proper storage configuration.

---

## Verification Checklist

- [x] Added new disk in VMware
- [x] Detected new disk
- [x] Created partition
- [x] Created XFS filesystem
- [x] Verified filesystem
- [x] Created mount point
- [x] Mounted storage
- [x] Added fstab entry
- [x] Tested with mount -a (troubleshot initial failure)
- [x] Rebooted system
- [x] Verified persistent mount after reboot

---

## RHCSA Notes

Common storage commands:

```bash
lsblk

fdisk

mkfs.xfs

blkid

mount

umount

df -h

mount -a
```

Most important workflow:

```text
Disk
↓
Partition
↓
Filesystem
↓
Mount Point
↓
Mount
↓
fstab
↓
Verify
```
