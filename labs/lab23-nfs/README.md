# Lab 23 — NFS (Network File System)

## Objective

Demonstrate installation, configuration, export, mounting, testing, and persistence of an NFS share using Rocky Linux.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation
- Screenshots: [Lab23 - NFS](Lab23%20-%20NFS/)

## Concepts Covered

| Command | Purpose |
|----------|----------|
| dnf install nfs-utils | Install NFS utilities |
| systemctl status nfs-server | Check NFS service |
| systemctl start nfs-server | Start NFS service |
| systemctl enable nfs-server | Enable NFS at boot |
| showmount -e | View exported shares |
| exportfs -rav | Apply NFS exports |
| mount | Mount NFS shares |
| umount | Unmount NFS shares |
| mount -a | Mount entries from fstab |

---

## Understanding NFS

NFS stands for:

```text
Network File System
```

NFS allows one Linux system to share directories with another Linux system over the network.

Think:

```text
Shared Network Drive
```

---

## Install NFS Utilities

Verify installation status:

```bash
rpm -q nfs-utils
```

Initial result:

```text
package nfs-utils is not installed
```

Install:

```bash
sudo dnf install nfs-utils -y
```

![Install nfs-utils](Lab23%20-%20NFS/01-install-nfs-utils.jpg)

Observation:

DNF resolved 7 packages: `nfs-utils` plus its dependencies (`gssproxy`, `libev`, `libnfsidmap`, `libverto-libev`, `rpcbind`, and `sssd-nfs-idmap`).

Verify:

```bash
rpm -q nfs-utils
```

Output:

```text
nfs-utils-2.8.3-5.el10.x86_64
```

![Verify nfs-utils Installed](Lab23%20-%20NFS/02-verify-nfs-utils-installed.jpg)

---

## Verify NFS Service

Check service:

```bash
systemctl status nfs-server
```

![NFS Server Status Before Starting](Lab23%20-%20NFS/03-nfs-server-status-before.jpg)

Initial observation:

```text
Active: inactive (dead)
Loaded: disabled
```

The service was installed but not running, and not set to start at boot.

---

## Start NFS Service

Command:

```bash
sudo systemctl start nfs-server
```

Verify:

```bash
systemctl status nfs-server
```

![Start and Enable NFS Server](Lab23%20-%20NFS/04-start-and-enable-nfs-server.jpg)

Observation:

```text
Active: active (exited)
```

`active (exited)` is normal for `nfs-server`. The unit runs its start-up commands (`exportfs -r`, `rpc.nfsd`) and then finishes, while the NFS server itself keeps running inside the kernel.

---

## Enable NFS Service

Command:

```bash
sudo systemctl enable nfs-server
```

Verify:

```bash
systemctl status nfs-server
```

The output is in the same screenshot as starting the service above.

Observation:

```text
Loaded: enabled
```

NFS will now start automatically at boot.

---

## Verify Existing Exports

Command:

```bash
showmount -e localhost
```

![Existing Exports and Share Directory](Lab23%20-%20NFS/05-create-export-directory.jpg)

Observation:

```text
Export list for localhost:
```

No exports had been configured.

---

## Create NFS Share Directory

Create directory:

```bash
sudo mkdir /nfs-share
```

Configure permissions:

```bash
sudo chmod 777 /nfs-share
```

Verify:

```bash
ls -ld /nfs-share
```

The output is in the same screenshot as the existing exports check above.

Output:

```text
drwxrwxrwx
```

> Security note: `777` lets every user read, write, and execute. That is fine for a practice lab, but production shares use tighter ownership and permissions.

---

## Configure Export

Edit:

```bash
sudo nano /etc/exports
```

Add:

```text
/nfs-share *(rw,sync,no_root_squash)
```

The `/etc/exports` file is opened in the screenshot in the next section.

Meaning of the options:

| Option | Meaning |
|----------|----------|
| `*` | Any client may connect |
| `rw` | Read and write access |
| `sync` | Write changes to disk before replying |
| `no_root_squash` | Root on the client keeps root privileges on the share |

> Security note: `*` and `no_root_squash` are convenient for a lab. In production, limit exports to specific hosts or subnets and use `root_squash`.

---

## Apply Export Configuration

Command:

```bash
sudo exportfs -rav
```

![Configure and Verify Export](Lab23%20-%20NFS/06-configure-and-verify-export.jpg)

Output:

```text
exporting *:/nfs-share
```

---

## Verify Export

Command:

```bash
showmount -e localhost
```

The output is in the same screenshot as applying the export above.

Output:

```text
Export list for localhost:
/nfs-share *
```

Observation:

The NFS export was active.

---

## Create NFS Client Mount Point

Command:

```bash
sudo mkdir /mnt/nfs-client
```

Verify:

```bash
ls -ld /mnt/nfs-client
```

![Create Mount Point and Mount NFS Share](Lab23%20-%20NFS/07-mount-nfs-share.jpg)

Observation:

The empty mount point directory was created and is owned by root.

---

## Mount NFS Share

Command:

```bash
sudo mount -t nfs localhost:/nfs-share /mnt/nfs-client
```

Verify:

```bash
mount | grep nfs
```

The output is in the same screenshot as creating the mount point above.

Output included:

```text
localhost:/nfs-share on /mnt/nfs-client type nfs4
```

Observation:

The NFS share was mounted successfully, using NFS version 4.2 over TCP.

---

## Test Server To Client Visibility

Create file:

```bash
sudo touch /nfs-share/server-test.txt
```

Verify from the mounted path:

```bash
ls -l /mnt/nfs-client
```

![Test File Sharing in Both Directions](Lab23%20-%20NFS/08-test-file-sharing.jpg)

Observation:

```text
server-test.txt
```

The file appeared through the mounted share. The listing also reported `Permission denied` while reading details of the file, because it is owned by root with mode `-rw-r-----`. The file itself was visible.

---

## Test Client To Server Visibility

Create file through the mount:

```bash
sudo touch /mnt/nfs-client/client-test.txt
```

Verify on the export:

```bash
ls -l /nfs-share
```

The output is in the same screenshot as the server-to-client test above.

Output:

```text
client-test.txt
server-test.txt
```

Observation:

Files created through the mounted share appeared on the export.

---

## Configure Persistent Mount

Edit:

```bash
sudo nano /etc/fstab
```

Add:

```text
localhost:/nfs-share /mnt/nfs-client nfs defaults 0 0
```

Verify:

```bash
tail -n 5 /etc/fstab
```

![Persistent Mount in fstab](Lab23%20-%20NFS/09-persistent-mount-fstab.jpg)

Observation:

The NFS entry was added below the existing `/`, `/boot`, and swap entries.

---

## Test Persistence

Unmount:

```bash
sudo umount /mnt/nfs-client
```

Verify mount removal:

```bash
mount | grep nfs
```

Apply fstab configuration:

```bash
sudo systemctl daemon-reload
sudo mount -a
```

Verify:

```bash
mount | grep nfs
```

![Test Persistence](Lab23%20-%20NFS/10-test-persistence.jpg)

Output:

```text
localhost:/nfs-share on /mnt/nfs-client type nfs4
```

Observation:

The NFS mount was successfully restored from the fstab configuration.

---

## Key Lessons Learned

- NFS allows Linux systems to share directories over a network.
- The `nfs-utils` package provides NFS server and client functionality.
- NFS shares are defined in `/etc/exports`.
- `exportfs -rav` applies export configuration changes.
- `showmount -e` displays exported shares.
- NFS exports can be mounted locally or remotely.
- Mounted shares behave like local directories.
- `/etc/fstab` provides persistent mounts across reboots.
- `nfs-server` showing `active (exited)` is normal.

---

## Verification Checklist

- [x] Installed nfs-utils
- [x] Verified installation
- [x] Started nfs-server
- [x] Enabled nfs-server
- [x] Created export directory
- [x] Configured export
- [x] Applied export
- [x] Verified export
- [x] Created mount point
- [x] Mounted NFS share
- [x] Verified mount
- [x] Created test file on export
- [x] Verified file through mount
- [x] Created test file through mount
- [x] Verified file on export
- [x] Configured persistent mount
- [x] Tested persistence

---

## Screenshots

All screenshots for this lab: [Lab23 - NFS](Lab23%20-%20NFS/)

| # | File | Shows |
|---|------|-------|
| 01 | `01-install-nfs-utils.jpg` | `rpm -q nfs-utils` (not installed) and `dnf install nfs-utils` |
| 02 | `02-verify-nfs-utils-installed.jpg` | End of install and `rpm -q nfs-utils` |
| 03 | `03-nfs-server-status-before.jpg` | `systemctl status nfs-server` (inactive, disabled) |
| 04 | `04-start-and-enable-nfs-server.jpg` | `start`, `enable`, and status (active, enabled) |
| 05 | `05-create-export-directory.jpg` | `showmount -e` (empty), `mkdir`, `chmod 777`, `ls -ld` |
| 06 | `06-configure-and-verify-export.jpg` | `nano /etc/exports`, `exportfs -rav`, `showmount -e` |
| 07 | `07-mount-nfs-share.jpg` | `mkdir /mnt/nfs-client`, `mount -t nfs`, `mount \| grep nfs` |
| 08 | `08-test-file-sharing.jpg` | Test files in both directions, `ls -l` on mount and export |
| 09 | `09-persistent-mount-fstab.jpg` | `nano /etc/fstab` and `tail -n 5 /etc/fstab` |
| 10 | `10-test-persistence.jpg` | `umount`, `mount -a`, and `mount \| grep nfs` |

---

## RHCSA Notes

Install NFS:

```bash
sudo dnf install nfs-utils -y
```

Start NFS:

```bash
sudo systemctl start nfs-server
```

Enable NFS:

```bash
sudo systemctl enable nfs-server
```

View exports:

```bash
showmount -e localhost
```

Apply exports:

```bash
sudo exportfs -rav
```

Mount export:

```bash
sudo mount -t nfs localhost:/nfs-share /mnt/nfs-client
```

View NFS mounts:

```bash
mount | grep nfs
```

Persist mounts:

```text
/etc/fstab
```

Most Important Concept:

```text
NFS
=
Shared Linux Storage Over The Network
```
