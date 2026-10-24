# Lab 20 — Software Management

## Objective

Demonstrate software management on Rocky Linux using DNF and RPM by searching for packages, viewing package information, installing software, verifying installations, managing services, testing functionality, and removing packages.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation
- Screenshots: [Lab20 - Software Management](Lab20%20-%20Software%20Management/)

## Concepts Covered

| Command | Purpose |
|----------|----------|
| dnf repolist | View available repositories |
| dnf search | Search for packages |
| dnf info | View package details |
| dnf install | Install software |
| dnf remove | Remove software |
| dnf list installed | View installed packages |
| rpm -q | Verify package installation |
| systemctl start | Start services |
| systemctl enable | Enable services at boot |
| systemctl status | Verify service status |
| ss -tulpn | View listening ports |
| curl | Test web services |

---

## Understanding Software Management

Linux software is managed through repositories and package managers.

Think:

```text
Repository
=
App Store

DNF
=
App Store Employee

RPM
=
Inventory Clerk
```

DNF installs and removes software while RPM verifies package information.

---

## View Available Repositories

Command:

```bash
sudo dnf repolist
```

Repositories:

```text
appstream
baseos
extras
```

![Repository list and installed packages](Lab20%20-%20Software%20Management/01-repolist-and-installed-packages.jpg)

Observation:

Repositories provide software packages to the operating system.

---

## View Installed Packages

Command:

```bash
sudo dnf list installed | head
```

Observation:

Displayed currently installed software packages and versions.

Examples:

```text
NetworkManager
ModemManager
```

(Shown in the same screenshot as the previous section.)

---

## Search For Software

Command:

```bash
dnf search nginx
```

![dnf search and dnf info for nginx](Lab20%20-%20Software%20Management/02-search-and-info-nginx.jpg)

Observation:

Located available nginx packages including:

```text
nginx
nginx-core
nginx-all-modules
```

Purpose:

```text
Find software before installation.
```

---

## View Package Information

Command:

```bash
dnf info nginx
```

Observation:

Displayed:

```text
Version
Repository
Architecture
Summary
Description
```

Purpose:

```text
Review package details before installation.
```

(Shown in the same screenshot as the previous section.)

---

## Install Nginx

Command:

```bash
sudo dnf install nginx -y
```

![Installing nginx and its dependencies](Lab20%20-%20Software%20Management/03-install-nginx.jpg)

Observation:

DNF installed:

```text
nginx
nginx-core
nginx-filesystem
rocky-logos-httpd
```

Explanation:

```text
Dependencies
=
Additional packages required by the software.
```

---

## Verify Package Installation

### Verify With RPM

Command:

```bash
rpm -q nginx
```

Output:

```text
nginx-1.26.3-6.el10_2.7.x86_64
```

![rpm -q, dnf list installed, and initial service status](Lab20%20-%20Software%20Management/04-verify-install-and-initial-status.jpg)

Observation:

Confirmed successful installation.

### Verify With DNF

Command:

```bash
dnf list installed nginx
```

Observation:

Nginx appeared in the installed package inventory.

(Shown in the same screenshot as the previous section.)

---

## Check Service Status

Command:

```bash
systemctl status nginx
```

Initial Observation:

```text
inactive (dead)
disabled
```

Important lesson:

```text
Installed
≠
Running
```

(Shown in the same screenshot as the Verify With RPM section.)

---

## Start Service

Command:

```bash
sudo systemctl start nginx
```

Verify:

```bash
systemctl status nginx
```

Result:

```text
active (running)
```

![nginx started, then enabled](Lab20%20-%20Software%20Management/05-start-and-enable-nginx.jpg)

Observation:

Nginx service successfully started.

---

## Enable Service At Boot

Command:

```bash
sudo systemctl enable nginx
```

Verify:

```bash
systemctl is-enabled nginx
```

Output:

```text
enabled
```

Observation:

Nginx will start automatically after reboot.

The enable command is shown in the previous screenshot. The `is-enabled` check is shown in the next section's screenshot.

---

## Verify Network Listening Port

Command:

```bash
sudo ss -tulpn | grep nginx
```

![is-enabled check and nginx listening on port 80](Lab20%20-%20Software%20Management/06-verify-enabled-and-listening-port.jpg)

Observation:

Displayed:

```text
0.0.0.0:80
[::]:80
```

Explanation:

```text
Port 80
=
Default HTTP web server port
```

Observation:

Nginx was listening for connections.

---

## Test Web Service

Command:

```bash
curl localhost
```

Observation:

Returned HTML content containing:

```text
HTTP Server Test Page powered by Rocky Linux
```

Purpose:

```text
Verify that the web server is responding correctly.
```

---

## Remove Software

Command:

```bash
sudo dnf remove nginx
```

![Removing nginx and unused dependencies](Lab20%20-%20Software%20Management/07-remove-nginx.jpg)

Observation:

Removed:

```text
nginx
nginx-core
nginx-filesystem
rocky-logos-httpd
```

Explanation:

```text
Unused dependencies were removed automatically.
```

---

## Key Lessons Learned

- DNF is used for package management.
- RPM verifies package installation and details.
- Repositories provide software packages.
- Software can be searched and inspected before installation.
- Package installation may include dependencies.
- Installed software is not automatically running.
- Services must be started separately.
- Services can be enabled to start at boot.
- Listening ports verify network availability.
- `curl` can test web services.
- Software can be completely removed when no longer needed.

---

## Verification Checklist

- [x] Viewed repositories
- [x] Viewed installed packages
- [x] Searched for software
- [x] Examined package information
- [x] Installed software
- [x] Verified package installation
- [x] Started a service
- [x] Enabled a service at boot
- [x] Verified listening ports
- [x] Tested web service functionality
- [x] Removed software
- [x] Removed dependencies

---

## Screenshots

All screenshots for this lab: [Lab20 - Software Management](Lab20%20-%20Software%20Management/)

| # | File | Shows |
|---|------|-------|
| 01 | `01-repolist-and-installed-packages.jpg` | `dnf repolist` and `dnf list installed \| head` |
| 02 | `02-search-and-info-nginx.jpg` | `dnf search nginx` and `dnf info nginx` |
| 03 | `03-install-nginx.jpg` | `sudo dnf install nginx -y` |
| 04 | `04-verify-install-and-initial-status.jpg` | `rpm -q`, `dnf list installed nginx`, and `systemctl status nginx` (inactive, disabled) |
| 05 | `05-start-and-enable-nginx.jpg` | `systemctl start`, `systemctl enable`, and status after each |
| 06 | `06-verify-enabled-and-listening-port.jpg` | `systemctl is-enabled nginx` and `sudo ss -tulpn \| grep nginx` |
| 07 | `07-remove-nginx.jpg` | `sudo dnf remove nginx` |

---

## RHCSA Notes

View repositories:

```bash
dnf repolist
```

Search packages:

```bash
dnf search nginx
```

View package information:

```bash
dnf info nginx
```

Install:

```bash
sudo dnf install nginx -y
```

Verify package:

```bash
rpm -q nginx
```

Start service:

```bash
sudo systemctl start nginx
```

Enable service:

```bash
sudo systemctl enable nginx
```

Verify listening ports:

```bash
ss -tulpn
```

Test service:

```bash
curl localhost
```

Remove package:

```bash
sudo dnf remove nginx
```

Most Important Concept:

```text
Install
≠
Running
≠
Enabled
```

These are three separate administrative tasks.
