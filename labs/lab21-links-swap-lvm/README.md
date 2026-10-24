# Lab 21 — Links, Swap & Extending LVM

## Objective

Demonstrate the use of hard links, symbolic links, swap space, and Logical Volume Manager (LVM) administration by inspecting storage components and extending a logical volume.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation
- Screenshots: [Lab21 - Links, Swap And Extending LVM](Lab21%20-%20Links%2C%20Swap%20And%20Extending%20LVM/)

## Concepts Covered

| Command | Purpose |
|----------|----------|
| ln | Create hard links |
| ln -s | Create symbolic links |
| free -h | View memory and swap usage |
| swapon --show | Display active swap |
| lsblk | View block devices |
| pvs | View physical volumes |
| vgs | View volume groups |
| lvs | View logical volumes |
| lvdisplay | View logical volume details |
| lvextend | Extend logical volumes |

---

## Understanding Linux Links

Linux supports two primary link types:

### Hard Link

A hard link creates an additional filename that references the same inode.

Think:

```text
One File
Two Names
```

### Symbolic Link

A symbolic link points to another file.

Think:

```text
Windows Shortcut
```

---

## Create Practice File

Command:

```bash
echo "RHCSA Practice" > testfile.txt
```

Verify:

```bash
cat testfile.txt
```

Output:

```text
RHCSA Practice
```

---

## Create Symbolic Link

Command:

```bash
ln -s testfile.txt softlink.txt
```

Verify:

```bash
ls -li
```

Observation:

```text
softlink.txt -> testfile.txt
```

---

## Create Hard Link

Command:

```bash
ln testfile.txt hardlink.txt
```

Verify:

```bash
ls -li
```

![Creating the test file, soft link, and hard link](Lab21%20-%20Links%2C%20Swap%20And%20Extending%20LVM/01-create-soft-and-hard-links.jpg)

Observation:

```text
hardlink.txt
testfile.txt
```

shared the same inode number (`17839128`). `softlink.txt` has its own inode (`17839129`), and the hard-linked files show a link count of 2.

Explanation:

```text
Same inode
=
Same underlying file
```

---

## Delete Original File

Command:

```bash
rm testfile.txt
```

Verify:

```bash
ls -li
```

![Soft link breaks, hard link survives after rm](Lab21%20-%20Links%2C%20Swap%20And%20Extending%20LVM/02-delete-original-link-behavior.jpg)

Observation:

`softlink.txt` is now highlighted as a broken link, and the link count on `hardlink.txt` dropped from 2 to 1.

---

## Test Symbolic Link

Command:

```bash
cat softlink.txt
```

Result:

```text
No such file or directory
```

Observation:

The symbolic link failed because its target file no longer existed.

(Shown in the same screenshot as the previous section.)

---

## Test Hard Link

Command:

```bash
cat hardlink.txt
```

Output:

```text
RHCSA Practice
```

Observation:

The hard link remained functional because it referenced the same inode.

(Shown in the same screenshot as the previous section.)

---

## Hard Link vs Symbolic Link

### Soft Link

```text
Shortcut

Delete original:
Link breaks
```

### Hard Link

```text
Additional filename

Delete original:
Still works
```

---

## View Memory and Swap

Command:

```bash
free -h
```

Output:

```text
RAM: 3.5 GiB
Swap: 2.0 GiB
```

![free -h showing memory and swap](Lab21%20-%20Links%2C%20Swap%20And%20Extending%20LVM/03-free-h-memory-and-swap.jpg)

Observation:

The system had:

```text
3.5 GiB RAM
2.0 GiB Swap
```

---

## View Active Swap

Command:

```bash
swapon --show
```

Observation:

```text
/dev/dm-1
```

Swap space was active.

![swapon --show and lsblk](Lab21%20-%20Links%2C%20Swap%20And%20Extending%20LVM/04-swapon-and-lsblk.jpg)

---

## Understanding Swap

Think:

```text
RAM
=
Work Desk

Swap
=
Emergency Storage Cabinet
```

When RAM becomes full, Linux can temporarily move less-active data into swap.

---

## View Block Devices

Command:

```bash
lsblk
```

Observation:

Displayed:

```text
nvme0n1
nvme0n2
```

and associated logical volumes.

Purpose:

```text
View disks, partitions, and storage layout.
```

(Shown in the same screenshot as the swap section.)

---

## Understanding LVM

Storage hierarchy:

```text
Physical Volume (PV)
↓
Volume Group (VG)
↓
Logical Volume (LV)
↓
Filesystem
```

Think:

```text
Land
↓
Warehouse
↓
Room
↓
Stored Items
```

---

## View Physical Volumes

Command:

```bash
sudo pvs
```

Observation:

Displayed:

```text
/dev/nvme0n1p3
/dev/nvme0n2
```

Purpose:

```text
Physical storage devices contributing to LVM.
```

---

## View Volume Groups

Command:

```bash
sudo vgs
```

Observation:

Displayed:

```text
rhcsa_vg
rl_rhcsa-lab
```

Important finding:

```text
rhcsa_vg
```

contained approximately:

```text
3 GiB free
```

available for expansion.

![vgs and lvs output](Lab21%20-%20Links%2C%20Swap%20And%20Extending%20LVM/05-vgs-and-lvs.jpg)

Troubleshooting note:

Running `vgs` as a regular user failed with:

```text
WARNING: Running as a non-root user. Functionality may be unavailable.
/run/lock/lvm/P_global:aux: open failed: Permission denied
```

LVM commands need root privileges, so running `sudo vgs` fixed it.

---

## View Logical Volumes

Command:

```bash
sudo lvs
```

Observation:

Displayed:

```text
rhcsa_lv
root
swap
```

Purpose:

```text
View storage allocations inside volume groups.
```

(Shown in the same screenshot as the previous section.)

---

## Display Logical Volume Details

Command:

```bash
sudo lvdisplay
```

![df -h and lvdisplay](Lab21%20-%20Links%2C%20Swap%20And%20Extending%20LVM/06-df-and-lvdisplay.jpg)

Observation:

`lvdisplay` listed `/dev/rhcsa_vg/rhcsa_lv` at 2.00 GiB with `# open 0`. The `df -h` output above it shows no mount for this logical volume.

Filtered to the size line:

```bash
sudo lvdisplay /dev/rhcsa_vg/rhcsa_lv | grep "LV Size"
```

Output:

```text
LV Size 2.00 GiB
```

Observation:

Logical volume size before expansion:

```text
2.00 GiB
```

---

## Extend Logical Volume

Command:

```bash
sudo lvextend -L +1G /dev/rhcsa_vg/rhcsa_lv
```

Output:

```text
Logical volume successfully resized
```

![lvextend growing the logical volume from 2 GiB to 3 GiB](Lab21%20-%20Links%2C%20Swap%20And%20Extending%20LVM/07-lvextend-logical-volume.jpg)

Observation:

1 GiB was allocated from available free space within the volume group.

---

## Verify Extension

Command:

```bash
sudo lvdisplay /dev/rhcsa_vg/rhcsa_lv | grep "LV Size"
```

Output:

```text
LV Size 3.00 GiB
```

Observation:

Logical volume increased from:

```text
2.00 GiB
```

to:

```text
3.00 GiB
```

successfully.

(Shown in the same screenshot as the previous section.)

---

## Key Lessons Learned

- Hard links and symbolic links serve different purposes.
- Symbolic links depend on the original file.
- Hard links reference the same inode.
- Swap provides supplemental memory when RAM becomes constrained.
- Physical Volumes (PV) provide storage to LVM.
- Volume Groups (VG) pool physical storage resources.
- Logical Volumes (LV) consume storage from Volume Groups.
- LVM allows storage expansion without reinstalling Linux.
- Logical volumes can be extended dynamically.
- LVM commands need `sudo`; as a regular user they fail with permission errors.

---

## Verification Checklist

- [x] Created symbolic link
- [x] Created hard link
- [x] Compared inode numbers
- [x] Deleted original file
- [x] Verified symbolic link failure
- [x] Verified hard link persistence
- [x] Viewed memory information
- [x] Viewed active swap
- [x] Reviewed block devices
- [x] Viewed physical volumes
- [x] Viewed volume groups
- [x] Viewed logical volumes
- [x] Extended a logical volume
- [x] Verified volume growth

---

## Screenshots

All screenshots for this lab: [Lab21 - Links, Swap And Extending LVM](Lab21%20-%20Links%2C%20Swap%20And%20Extending%20LVM/)

| # | File | Shows |
|---|------|-------|
| 01 | `01-create-soft-and-hard-links.jpg` | `echo`, `cat`, `ln -s`, `ln`, and `ls -li` |
| 02 | `02-delete-original-link-behavior.jpg` | `rm testfile.txt`, `ls -li`, and `cat` on both links |
| 03 | `03-free-h-memory-and-swap.jpg` | `free -h` |
| 04 | `04-swapon-and-lsblk.jpg` | `swapon --show` and `lsblk` |
| 05 | `05-vgs-and-lvs.jpg` | `vgs` (permission error), `sudo vgs`, and `sudo lvs` |
| 06 | `06-df-and-lvdisplay.jpg` | `df -h` and `sudo lvdisplay` |
| 07 | `07-lvextend-logical-volume.jpg` | `lvdisplay` size check, `lvextend -L +1G`, and size check after |

---

## RHCSA Notes

Create symbolic link:

```bash
ln -s source target
```

Create hard link:

```bash
ln source target
```

View memory:

```bash
free -h
```

View swap:

```bash
swapon --show
```

View physical volumes:

```bash
sudo pvs
```

View volume groups:

```bash
sudo vgs
```

View logical volumes:

```bash
sudo lvs
```

Extend logical volume:

```bash
sudo lvextend -L +1G /dev/rhcsa_vg/rhcsa_lv
```

Most Important Concepts:

```text
Soft Link
=
Shortcut

Hard Link
=
Second Name For Same File
```

```text
PV
↓
VG
↓
LV
↓
Filesystem
```

```text
LVM
=
Flexible Storage Management
```
