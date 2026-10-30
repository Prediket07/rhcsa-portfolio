# Lab 24 — Firewalld Zones

## Objective

Demonstrate how to inspect, compare, modify, and verify Firewalld zones, including default zones, active zones, interface assignments, and service differences between zones.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation
- Screenshots: [Lab24 - Firewalld Zones](Lab24%20-%20Firewalld%20Zones/)

## Concepts Covered

| Command | Purpose |
|----------|----------|
| firewall-cmd --get-default-zone | View default zone |
| firewall-cmd --get-active-zones | View active zones |
| firewall-cmd --list-all | View current zone configuration |
| firewall-cmd --get-zones | View all available zones |
| firewall-cmd --zone=ZONE --list-all | Inspect a specific zone |
| firewall-cmd --zone=ZONE --list-interfaces | View interfaces assigned to a zone |
| firewall-cmd --set-default-zone=ZONE | Change default zone |
| firewall-cmd --get-zone-of-interface=INTERFACE | Determine interface zone assignment |

---

## Understanding Firewalld Zones

A Firewalld zone is a predefined collection of firewall rules.

Think:

```text
Zone = Firewall Personality
```

Examples:

```text
public
home
work
trusted
```

Each zone has its own set of allowed services.

---

## View Default Zone

Command:

```bash
sudo firewall-cmd --get-default-zone
```

![Default Zone, Active Zone, and Public Configuration](Lab24%20-%20Firewalld%20Zones/01-default-active-zone-and-public-config.jpg)

Output:

```text
public
```

Observation:

The system was using the public zone by default.

---

## View Active Zones

Command:

```bash
sudo firewall-cmd --get-active-zones
```

The output is in the same screenshot as the default zone above. That screenshot shows the shortened form `--get-active-zone`, which firewall-cmd also accepted.

Output:

```text
public (default)
  interfaces: ens160
```

Observation:

Network interface:

```text
ens160
```

was assigned to the public zone.

---

## View Public Zone Configuration

Command:

```bash
sudo firewall-cmd --list-all
```

The output is in the same screenshot as the default zone above.

Output included:

```text
public (default, active)
  interfaces: ens160
  services: cockpit dhcpv6-client ssh
```

Observation:

Only a limited set of services were permitted.

---

## View All Available Zones

Command:

```bash
sudo firewall-cmd --get-zones
```

![All Zones, Home Zone, and Trusted Zone](Lab24%20-%20Firewalld%20Zones/02-all-zones-home-and-trusted.jpg)

Output:

```text
block dmz drop external home internal nm-shared public trusted work
```

Observation:

Firewalld provides multiple predefined security profiles.

---

## Inspect Home Zone

Command:

```bash
sudo firewall-cmd --zone=home --list-all
```

The output is in the same screenshot as the list of zones above.

Output included:

```text
home
  interfaces:
  services: cockpit dhcpv6-client mdns samba-client ssh
```

Observation:

The home zone allows more services than public: `mdns` and `samba-client` are added. No interface is assigned to it yet.

---

## Inspect Trusted Zone

Command:

```bash
sudo firewall-cmd --zone=trusted --list-all
```

The output is in the same screenshot as the list of zones above.

Output included:

```text
trusted
  target: ACCEPT
  services:
```

Observation:

The trusted zone lists no services because its target is `ACCEPT`, which allows all traffic. It should only be used for networks and hosts that are fully trusted.

---

## Compare Zones

| Zone | Target | Services |
|----------|----------|----------|
| public | default | cockpit, dhcpv6-client, ssh |
| home | default | cockpit, dhcpv6-client, mdns, samba-client, ssh |
| trusted | ACCEPT | none listed (all traffic accepted) |

---

## Change the Default Zone

Command:

```bash
sudo firewall-cmd --set-default-zone=home
```

Verify:

```bash
sudo firewall-cmd --get-default-zone
sudo firewall-cmd --get-active-zones
```

![Set Default Zone to Home](Lab24%20-%20Firewalld%20Zones/03-set-default-zone-home.jpg)

Output:

```text
success
home
home (default)
  interfaces: ens160
```

Observation:

The default zone changed from `public` to `home`. The interface `ens160` moved to the home zone automatically because it had no zone assigned directly, so it follows the default zone.

---

## Verify the Home Zone Is Active

Command:

```bash
sudo firewall-cmd --list-all
```

![Home Zone Active Configuration](Lab24%20-%20Firewalld%20Zones/04-home-zone-active-config.jpg)

Output included:

```text
home (default, active)
  interfaces: ens160
  services: cockpit dhcpv6-client mdns samba-client ssh
```

Observation:

The active configuration now showed the home zone's service list on `ens160`.

---

## Restore the Public Zone

Command:

```bash
sudo firewall-cmd --set-default-zone=public
```

Verify:

```bash
sudo firewall-cmd --get-default-zone
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all
```

![Restore Public as Default Zone](Lab24%20-%20Firewalld%20Zones/05-restore-public-default-zone.jpg)

Output included:

```text
success
public
public (default)
  interfaces: ens160
services: cockpit dhcpv6-client ssh
```

Observation:

The lab environment was returned to its original configuration.

---

## Determine Interface Zone

Command:

```bash
sudo firewall-cmd --get-zone-of-interface=ens160
```

![Zone of Interface ens160](Lab24%20-%20Firewalld%20Zones/06-zone-of-interface.jpg)

Output:

```text
public
```

Observation:

The interface `ens160` was assigned to the public zone.

---

## Key Lessons Learned

- Firewalld zones are predefined sets of firewall rules.
- Each zone allows a different set of services.
- `--get-default-zone` shows the zone used for interfaces with no zone assigned.
- `--get-active-zones` shows which zones are in use and their interfaces.
- `--zone=ZONE --list-all` inspects any zone, even one that is not active.
- `--set-default-zone` moves interfaces that follow the default zone to the new zone.
- The `trusted` zone accepts all traffic.
- `--get-zone-of-interface` confirms which zone an interface belongs to.

---

## Verification Checklist

- [x] Viewed default zone
- [x] Viewed active zones
- [x] Viewed public zone configuration
- [x] Listed all available zones
- [x] Inspected home zone
- [x] Inspected trusted zone
- [x] Changed default zone to home
- [x] Verified interface moved to home zone
- [x] Restored public as default zone
- [x] Verified interface zone assignment

---

## Screenshots

All screenshots for this lab: [Lab24 - Firewalld Zones](Lab24%20-%20Firewalld%20Zones/)

| # | File | Shows |
|---|------|-------|
| 01 | `01-default-active-zone-and-public-config.jpg` | `--get-active-zone`, `--get-default-zone`, and `--list-all` (public) |
| 02 | `02-all-zones-home-and-trusted.jpg` | `--get-zones`, `--zone=home --list-all`, `--zone=trusted --list-all` |
| 03 | `03-set-default-zone-home.jpg` | `--set-default-zone=home`, default zone and active zones |
| 04 | `04-home-zone-active-config.jpg` | `--list-all` with home as the active default zone |
| 05 | `05-restore-public-default-zone.jpg` | `--set-default-zone=public` and verification |
| 06 | `06-zone-of-interface.jpg` | `--get-zone-of-interface=ens160` |

---

## RHCSA Notes

View default zone:

```bash
sudo firewall-cmd --get-default-zone
```

View active zones:

```bash
sudo firewall-cmd --get-active-zones
```

View current zone configuration:

```bash
sudo firewall-cmd --list-all
```

View all zones:

```bash
sudo firewall-cmd --get-zones
```

Inspect a specific zone:

```bash
sudo firewall-cmd --zone=home --list-all
```

Change default zone:

```bash
sudo firewall-cmd --set-default-zone=home
```

Find the zone of an interface:

```bash
sudo firewall-cmd --get-zone-of-interface=ens160
```

Most Important Concept:

```text
Zone = Set of Firewall Rules
Interface -> Zone -> Allowed Services
```
