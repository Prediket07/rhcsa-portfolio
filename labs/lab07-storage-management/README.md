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

## Detect New Disk

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

Create an XFS filesystem:

```bash
sudo mkfs.xfs /dev/nvme0n3p1
```

Observation:

The partition was formatted and prepared for use.

---

## Verify Filesystem

```bash
sudo blkid /dev/nvme0n3p1
```

Result:

```text
TYPE="xfs"
```

Observation:

The filesystem was successfully created.

---

## Create Mount Point

Create a directory to mount the new filesystem:

```bash
sudo mkdir /lab07data
```

Verify:

```bash
ls -ld /lab07data
```

Observation:

The mount point was created successfully.

---

## Mount Filesystem

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

Observation:

The storage became available to the operating system.

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

Reload configuration:

```bash
sudo systemctl daemon-reload
```

Test:

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

Observation:

The fstab entry works correctly.

---

## Reboot Verification

Reboot system:

```bash
sudo reboot
```

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

Observation:

Persistent mounting worked successfully.

---

## Key Lessons Learned

- Disks must be partitioned before use.
- Filesystems must be created before data can be stored.
- A mount point provides a location where storage becomes accessible.
- UUIDs should be used in `/etc/fstab` for reliable mounting.
- `mount -a` is an excellent way to test fstab entries before rebooting.
- A successful reboot confirms proper storage configuration.

---

## Verification Checklist

- [x] Detected new disk
- [x] Created partition
- [x] Created XFS filesystem
- [x] Created mount point
- [x] Mounted storage
- [x] Verified filesystem
- [x] Added fstab entry
- [x] Tested with mount -a
- [x] Rebooted system
- [x] Verified persistent mount after reboot

---

## Screenshots

### Detect New Disk

`01-detect-new-disk.jpg`

### Create Partition with fdisk

`02-create-partition-fdisk.jpg`

### Partition Created

`03-partition-created.jpg`

### Create XFS Filesystem

`04-create-xfs-filesystem.jpg`

### Verify Filesystem with blkid

`05-verify-filesystem-blkid.jpg`

### Create Mount Point

`06-create-mount-point.jpg`

### Mount Filesystem

`07-mount-filesystem.jpg`

### Add fstab Entry

`08-add-fstab-entry.jpg`

### Unmount Filesystem

`09-unmount-filesystem.jpg`

### Test with mount -a

`10-mount-a-test.jpg`

### Reboot System

`11-reboot-system.jpg`

### Post-Reboot Verification

`12-post-reboot-verification.jpg`

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
