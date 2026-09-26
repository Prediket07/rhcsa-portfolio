# Lab 08 — Storage Decommissioning

## Objective

Demonstrate the safe removal of Linux storage resources by unmounting a filesystem, removing persistent mount configuration, deleting a partition, removing the mount point, and verifying storage cleanup.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation

## Concepts Covered

| Command | Purpose |
|----------|----------|
| umount | Unmount a filesystem |
| df -h | Verify mounted filesystems |
| lsblk | View disks and partitions |
| blkid | Verify filesystem information |
| nano /etc/fstab | Remove persistent mount configuration |
| mount -a | Validate fstab configuration |
| fdisk | Remove partition |
| rmdir | Remove empty mount-point directory |

---

## Understanding Storage Decommissioning

Provisioning storage is only half of storage administration.

Administrators must also know how to safely remove storage resources.

Typical workflow:

```text
Unmount Filesystem
↓
Remove fstab Entry
↓
Verify
↓
Delete Partition
↓
Remove Mount Point
↓
Remove Disk
↓
Verify Cleanup
```

---

## Initial State

Storage from Lab 07 existed as:

```text
Disk:        nvme0n3
Partition:   nvme0n3p1
Filesystem:  XFS
Mount Point: /lab07data
```

The filesystem was mounted and configured in:

```text
/etc/fstab
```

for automatic mounting.

![Initial State](Lab08%20-%20storage%20decommissioning/01-initial-state.jpg)

---

## Unmount Filesystem

Command:

```bash
sudo umount /lab07data
```

Verify:

```bash
df -h
```

Observation:

```text
/dev/nvme0n3p1
```

no longer appeared in mounted filesystems.

![Unmount Filesystem](Lab08%20-%20storage%20decommissioning/02-unmount-filesystem.jpg)

Key lesson:

```text
Unmounting does not delete storage.
```

The disk, partition, and filesystem still existed.

---

## Remove Persistent Mount Configuration

Edit:

```bash
sudo nano /etc/fstab
```

Removed:

```text
UUID=<storage-uuid> /lab07data xfs defaults 0 0
```

Reload configuration:

```bash
sudo systemctl daemon-reload
```

Verify:

```bash
cat /etc/fstab
```

![Remove Fstab Entry](Lab08%20-%20storage%20decommissioning/03-remove-fstab-entry.jpg)

Observation:

The `/lab07data` entry was removed.

---

## Validate Configuration

Run:

```bash
sudo mount -a
```

Verify:

```bash
df -h
```

![Validate Configuration with mount -a](Lab08%20-%20storage%20decommissioning/04-validate-mount-a.jpg)

Observation:

The storage was not remounted.

This confirmed that no persistent mount configuration remained.

---

## Verify Filesystem Still Exists

Command:

```bash
sudo blkid /dev/nvme0n3p1
```

![Verify Filesystem Still Exists](Lab08%20-%20storage%20decommissioning/05-verify-filesystem-exists.jpg)

Observation:

Linux still detected:

```text
TYPE="xfs"
```

and the filesystem UUID.

Key lesson:

```text
Removing the mount does not remove the filesystem.
```

---

## Delete Partition

Launch fdisk:

```bash
sudo fdisk /dev/nvme0n3
```

Commands:

```text
p
d
p
w
```

Purpose:

```text
View partition
Delete partition
Verify deletion
Write changes
```

Verify:

```bash
lsblk
```

Result:

```text
nvme0n3
```

remained, but:

```text
nvme0n3p1
```

was removed.

---

## Remove Mount Point

Remove:

```bash
sudo rmdir /lab07data
```

Verify:

```bash
ls -ld /lab07data
```

Result:

```text
No such file or directory
```

![Delete Partition and Remove Mount Point](Lab08%20-%20storage%20decommissioning/06-delete-partition-remove-mountpoint.jpg)

Observation:

The mount-point directory was removed successfully.

---

## Remove Virtual Disk

Power off system:

```bash
sudo poweroff
```

In VMware Workstation:

```text
VM Settings
↓
Select 10 GB Lab Disk
↓
Remove
```

Boot system.

Verify:

```bash
lsblk
```

Result:

```text
nvme0n3
```

no longer existed.

![Remove Virtual Disk](Lab08%20-%20storage%20decommissioning/07-remove-virtual-disk.jpg)

Observation:

The virtual disk was completely removed.

---

## Key Lessons Learned

- Unmounting a filesystem does not delete it.
- Removing an fstab entry prevents automatic mounting.
- Filesystems can exist without being mounted.
- Partitions must be removed separately.
- Mount points are normal directories and should be removed when no longer needed.
- Storage decommissioning follows a structured process.
- Verification should be performed after every step.

---

## Verification Checklist

- [x] Unmounted filesystem
- [x] Removed fstab entry
- [x] Reloaded systemd configuration
- [x] Validated with mount -a
- [x] Verified filesystem still existed
- [x] Deleted partition
- [x] Removed mount point
- [x] Removed virtual disk
- [x] Verified cleanup

---

## RHCSA Notes

Storage provisioning:

```text
Disk
↓
Partition
↓
Filesystem
↓
Mount
↓
fstab
```

Storage decommissioning:

```text
Unmount
↓
Remove fstab
↓
Verify
↓
Delete partition
↓
Remove mount point
↓
Remove disk
↓
Verify
```
