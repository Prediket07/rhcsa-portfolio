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
sudo blkid /dev
