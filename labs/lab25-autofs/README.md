# Lab 25 — Configure autofs with NFS

## Objective

Configure `autofs` on Rocky Linux to automatically mount an NFS export on demand.

This lab demonstrates how to:

- Install and configure `autofs`
- Create and verify an NFS export
- Configure an indirect autofs map
- Configure an autofs master map
- Enable and manage the `autofs` service
- Automatically mount an NFS resource when its path is accessed
- Verify the resulting mount

---

## Environment

- Rocky Linux
- VMware Workstation
- NFS
- autofs
- systemd
- Screenshots: [Lab25 - Autofs](Lab25%20-%20Autofs/)

---

## 1. Verify and Install autofs

First, check whether the `autofs` package is installed:

```bash
rpm -q autofs
```

The package was not initially installed.

Install it:

```bash
sudo dnf install -y autofs
```

![Install autofs - Start](Lab25%20-%20Autofs/01-install-autofs-start.jpg)

Verify the installation:

```bash
rpm -q autofs
```

![Install autofs - Complete](Lab25%20-%20Autofs/02-install-autofs-complete.jpg)

Observation:

DNF installed `autofs` and its dependency `libsss_autofs`. The final `rpm -q autofs` returned `autofs-5.1.9-13.el10.x86_64`.

---

## 2. Inspect autofs Configuration Files

List the default autofs configuration files:

```bash
ls -l /etc/auto*
```

Inspect the master configuration:

```bash
cat /etc/auto.master
```

![autofs Configuration Files and Master Map](Lab25%20-%20Autofs/03-autofs-config-files-and-master-map.jpg)

Observation:

The default files included `auto.master`, `auto.misc`, `auto.net`, `auto.smb`, and `autofs.conf`. The `/etc/auto.master.d/` directory was empty. The master file contains the line `+dir:/etc/auto.master.d`, which tells autofs to also read any `*.autofs` file placed in that directory.

The autofs master map defines the parent mount point and identifies which map file should be used for resources underneath that location.

---

## 3. Verify NFS Support

Verify that the NFS utilities are installed:

```bash
rpm -q nfs-utils
```

Check the currently advertised NFS exports:

```bash
showmount -e localhost
```

![Verify nfs-utils and Existing Exports](Lab25%20-%20Autofs/04-verify-nfs-utils-and-exports.jpg)

Observation:

`nfs-utils` was installed. The only export listed was `/nfs-share` from the previous NFS lab, so a dedicated NFS export was created for this lab.

---

## 4. Create the NFS Export

Create a directory that will be exported through NFS:

```bash
sudo mkdir -p /srv/autofs-share
```

Create a test file inside the directory:

```bash
echo "RHCSA Lab 25 autofs test" | sudo tee /srv/autofs-share/test.txt
```

Verify:

```bash
sudo ls -l /srv/autofs-share
```

![Create Export Directory and Test File](Lab25%20-%20Autofs/05-create-export-directory-and-test-file.jpg)

Observation:

Listing the directory as `jross` was denied. With `sudo`, the listing showed `test.txt`, owned by root with mode `-rw-r-----`.

Configure the export in:

```text
/etc/exports
```

Add:

```text
/srv/autofs-share localhost(rw,sync)
```

![/etc/exports Entry](Lab25%20-%20Autofs/06-etc-exports-entry.jpg)

Observation:

The `/etc/exports` file now contains only this one line, so the `/nfs-share` export from the previous lab is no longer active.

Apply the NFS export configuration:

```bash
sudo exportfs -rav
```

Verify the active export:

```bash
sudo exportfs -v
```

Verify that the NFS server advertises the export:

```bash
showmount -e localhost
```

![Apply and Verify the Export](Lab25%20-%20Autofs/07-apply-and-verify-export.jpg)

The export was successfully advertised as:

```text
/srv/autofs-share localhost
```

---

## 5. Create the autofs Parent Directory

Create the parent directory that will be managed by `autofs`:

```bash
sudo mkdir -p /mnt/autofs
```

Verify the directory:

```bash
sudo ls -ld /mnt/autofs
```

![Create the autofs Parent Directory](Lab25%20-%20Autofs/08-create-autofs-parent-directory.jpg)

The final automounted path will be:

```text
/mnt/autofs/share
```

The `share` key is handled by `autofs`. The NFS filesystem does not need to be manually mounted with the `mount` command.

---

## 6. Configure the Indirect Map

Create the indirect map:

```text
/etc/auto.lab25
```

Add:

```text
share -rw localhost:/srv/autofs-share
```

This entry contains three important components:

```text
share    -rw    localhost:/srv/autofs-share
```

- `share` is the key used underneath `/mnt/autofs`
- `-rw` specifies read/write mount options
- `localhost:/srv/autofs-share` is the NFS resource

Set the correct ownership:

```bash
sudo chown root:root /etc/auto.lab25
```

Set appropriate permissions:

```bash
sudo chmod 644 /etc/auto.lab25
```

Verify:

```bash
sudo cat /etc/auto.lab25
ls -l /etc/auto.lab25
```

The loaded entry is confirmed in section 9 with `automount -m`.

---

## 7. Configure the Master Map

Create:

```text
/etc/auto.master.d/lab25.autofs
```

Add:

```text
/mnt/autofs /etc/auto.lab25
```

![Edit the Master Map File](Lab25%20-%20Autofs/09-master-map-file-edit.jpg)

Set appropriate permissions:

```bash
sudo chmod 644 /etc/auto.master.d/lab25.autofs
```

Verify:

```bash
sudo cat /etc/auto.master.d/lab25.autofs
ls -l /etc/auto.master.d/lab25.autofs
```

The master map tells autofs to use:

```text
/etc/auto.lab25
```

for resources accessed underneath:

```text
/mnt/autofs
```

Combined with the indirect map, the configuration becomes:

```text
/mnt/autofs
      |
      +-- share
            |
            +-- localhost:/srv/autofs-share
```

The resulting path is:

```text
/mnt/autofs/share
```

---

## 8. Enable and Start autofs

Enable the service at boot and start it immediately:

```bash
sudo systemctl enable --now autofs
```

Verify the runtime state:

```bash
systemctl is-active autofs
```

Output:

```text
active
```

Verify boot persistence:

```bash
systemctl is-enabled autofs
```

Output:

```text
enabled
```

![Enable autofs - Active and Enabled](Lab25%20-%20Autofs/10-enable-autofs-active-enabled.jpg)

This follows the same systemd distinction used with other services:

- `active` means the service is running now
- `enabled` means the service is configured to start automatically at boot

---

## 9. Inspect the Loaded autofs Maps

Inspect the maps recognized by autofs:

```bash
sudo automount -m
```

![automount -m Shows the Loaded Map](Lab25%20-%20Autofs/11-automount-map-loaded.jpg)

The output shows the mount point:

```text
Mount point: /mnt/autofs
```

using:

```text
map: /etc/auto.lab25
```

with the entry:

```text
share | -rw localhost:/srv/autofs-share
```

This confirms that the master map and indirect map have been successfully loaded.

---

## 10. Trigger the Automatic Mount

Access the configured path:

```bash
sudo ls -l /mnt/autofs/share
```

![Verify the Automount](Lab25%20-%20Autofs/12-verify-automount.jpg)

Accessing the share key triggers autofs to mount:

```text
localhost:/srv/autofs-share
```

at:

```text
/mnt/autofs/share
```

The test file becomes accessible through the automounted path:

```text
test.txt
```

Listing the directory as `jross` was denied, because the directory and file are owned by root. With `sudo`, `test.txt` was listed.

---

## 11. Verify the Automatic Mount

Verify the actual mounted filesystem:

```bash
findmnt /mnt/autofs/share
```

The target was:

```text
/mnt/autofs/share
```

The output is in the same screenshot as the previous section.

Observation:

`findmnt` reported the source as the local volume `/dev/mapper/rl_rhcsa--lab-root[/srv/autofs-share]` with type `xfs`. Because the NFS server in this lab is the same machine (`localhost`), autofs mounted the local directory directly instead of going over the network. With a remote NFS server, `findmnt` would show `server:/path` with type `nfs4`.

Read the test file through the automounted path:

```bash
sudo cat /mnt/autofs/share/test.txt
```

Output:

```text
RHCSA Lab 25 autofs test
```

This confirms that the share defined in the autofs maps was mounted automatically when the path was accessed.

---

## Verification Commands

Commands used to verify the completed configuration:

```bash
rpm -q autofs
rpm -q nfs-utils
showmount -e localhost
sudo exportfs -v
systemctl is-active autofs
systemctl is-enabled autofs
sudo automount -m
sudo ls -l /mnt/autofs/share
findmnt /mnt/autofs/share
sudo cat /mnt/autofs/share/test.txt
```

---

## autofs Workflow

The completed configuration follows this path:

```text
NFS Server
/srv/autofs-share
        |
        v
NFS Export
localhost:/srv/autofs-share
        |
        v
Master Map
/etc/auto.master.d/lab25.autofs
        |
        v
/mnt/autofs
        |
        v
Indirect Map
/etc/auto.lab25
        |
        v
share
        |
        v
/mnt/autofs/share
```

When `/mnt/autofs/share` is accessed, autofs automatically mounts the resource.

---

## Key Takeaways

- autofs automatically mounts filesystems when configured paths are accessed.
- NFS provides the remote filesystem while autofs controls when the filesystem is mounted.
- The autofs master map associates a parent directory with a map file.
- An indirect map defines keys underneath the parent mount point.
- The indirect map entry defines the remote filesystem and mount options.
- `systemctl is-active` verifies the current service state.
- `systemctl is-enabled` verifies boot persistence.
- `showmount -e` helps verify NFS exports.
- `exportfs -v` displays active NFS export configuration.
- `automount -m` is useful for inspecting the maps recognized by autofs.
- `journalctl -u autofs` helps diagnose configuration and map problems.
- `findmnt` verifies the real mounted filesystem and its source.
- When the NFS server is the local machine, autofs mounts the directory locally, so `findmnt` shows the local filesystem rather than `nfs4`.
- Correct ownership and permissions on configuration files are good practice.

---

## Result

Successfully configured autofs to automatically mount a share at:

```text
/mnt/autofs/share
```

The resource:

```text
localhost:/srv/autofs-share
```

was mounted automatically when the configured path was accessed.

---

## Verification Checklist

- [x] Installed autofs
- [x] Inspected autofs configuration files
- [x] Verified nfs-utils
- [x] Created NFS export
- [x] Applied and verified export
- [x] Created autofs parent directory
- [x] Configured indirect map
- [x] Configured master map
- [x] Enabled and started autofs
- [x] Verified map loaded with `automount -m`
- [x] Verified automatic mount with `findmnt`
- [x] Read the test file through the automounted path

---

## Screenshots

All screenshots for this lab: [Lab25 - Autofs](Lab25%20-%20Autofs/)

| # | File | Shows |
|---|------|-------|
| 01 | `01-install-autofs-start.jpg` | `rpm -q autofs` (not installed) and `dnf install -y autofs` starting |
| 02 | `02-install-autofs-complete.jpg` | Install complete and `rpm -q autofs` |
| 03 | `03-autofs-config-files-and-master-map.jpg` | `ls -l /etc/auto*` and `cat /etc/auto.master` |
| 04 | `04-verify-nfs-utils-and-exports.jpg` | `rpm -q nfs-utils` and `showmount -e localhost` |
| 05 | `05-create-export-directory-and-test-file.jpg` | `mkdir /srv/autofs-share`, test file, directory listing |
| 06 | `06-etc-exports-entry.jpg` | `/etc/exports` with the autofs export |
| 07 | `07-apply-and-verify-export.jpg` | `exportfs -rav`, `exportfs -v`, `showmount -e` |
| 08 | `08-create-autofs-parent-directory.jpg` | `mkdir -p /mnt/autofs` and `ls -ld` |
| 09 | `09-master-map-file-edit.jpg` | Editing `/etc/auto.master.d/lab25.autofs` |
| 10 | `10-enable-autofs-active-enabled.jpg` | `systemctl enable --now autofs`, `is-active`, `is-enabled` |
| 11 | `11-automount-map-loaded.jpg` | `automount -m` with the `share` entry loaded |
| 12 | `12-verify-automount.jpg` | `ls`, `findmnt`, and `sudo cat` through `/mnt/autofs/share` |

---

## RHCSA Notes

Install autofs:

```bash
sudo dnf install -y autofs
```

Master map entry (in `/etc/auto.master.d/NAME.autofs`):

```text
/mnt/autofs /etc/auto.lab25
```

Indirect map entry (in `/etc/auto.lab25`):

```text
share -rw localhost:/srv/autofs-share
```

Enable and start:

```bash
sudo systemctl enable --now autofs
```

Restart after changing maps:

```bash
sudo systemctl restart autofs
```

Inspect loaded maps:

```bash
sudo automount -m
```

Check logs:

```bash
sudo journalctl -u autofs -n 30 --no-pager
```

Verify the mount:

```bash
findmnt /mnt/autofs/share
```

Most Important Concept:

```text
Master Map   = Which directory autofs manages, and which map to use
Indirect Map = The keys underneath that directory and what each one mounts
autofs       = Mounts on demand, when the path is accessed
```
