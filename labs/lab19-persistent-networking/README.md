# Lab 19 — Persistent Networking

## Objective

Demonstrate how to view, modify, and apply persistent network configuration using NetworkManager and `nmcli`.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation
- Screenshots: [Lab19 - Persistant Networking](Lab19%20-%20Persistant%20Networking/)

## Concepts Covered

| Command | Purpose |
|----------|----------|
| nmcli device status | View network devices |
| nmcli connection show | View saved network profiles |
| ip a | View IP configuration |
| ip route | View routing table |
| nmcli connection modify | Change network settings |
| nmcli connection down | Disable network profile |
| nmcli connection up | Enable network profile |
| ping | Test connectivity |

---

## Understanding Persistent Networking

Persistent networking can be configured using:

### DHCP

The router assigns an IP address automatically.

Example:

```text
IP Address
Gateway
DNS
```

Provided by the DHCP server.

### Static Addressing

The administrator assigns IP settings manually.

The system continues to use the same configuration after reboots.

Think of it like this:

```text
DHCP       = Hotel Room Assignment
Static IP  = Owning a House
```

---

## Identify Network Devices

Command:

```bash
nmcli device status
```

![Network Devices and Saved Connections](Lab19%20-%20Persistant%20Networking/01-network-devices-and-connections.jpg)

Observation:

NetworkManager displayed network interfaces including:

```text
ens160
lo
```

The primary active network adapter was:

```text
ens160
```

---

## View Saved Network Connections

Command:

```bash
nmcli connection show
```

The output is in the same screenshot as `nmcli device status` above.

Observation:

NetworkManager displayed saved connection profiles including:

```text
ens160
lo
```

To see every setting stored in one profile:

```bash
nmcli connection show ens160
```

![ens160 Connection Profile Details](Lab19%20-%20Persistant%20Networking/02-connection-profile-details.jpg)

Observation:

The `ens160` profile is an Ethernet connection bound to the `ens160` interface with `autoconnect` enabled, so NetworkManager brings it up automatically.

---

## View Current IP Address

Command:

```bash
ip a | grep inet
```

Output:

```text
inet 192.168.243.128/24 ... dynamic ... ens160
```

![Current IP Address](Lab19%20-%20Persistant%20Networking/03-current-ip-address.jpg)

Observation:

The address was dynamically assigned.

---

## Identify Address Assignment Method

Command:

```bash
nmcli connection show ens160 | grep ipv4.method
```

Output:

```text
ipv4.method:    auto
```

![Address Method and Routing](Lab19%20-%20Persistant%20Networking/04-address-method-and-routing.jpg)

Observation:

The system was configured to receive network settings automatically from DHCP.

---

## View Routing Information

Command:

```bash
ip route
```

Output:

```text
default via 192.168.243.2 dev ens160 proto dhcp src 192.168.243.128 metric 100
192.168.243.0/24 dev ens160 proto kernel scope link src 192.168.243.128 metric 100
```

The output is in the same screenshot as the address method above.

Observation:

The default gateway provides access to networks outside the local subnet. Note `proto dhcp`: the gateway came from DHCP.

---

## View DNS Configuration

Command:

```bash
nmcli connection show ens160 | grep dns
```

![DNS Configuration](Lab19%20-%20Persistant%20Networking/05-dns-configuration.jpg)

Observation:

No manually assigned DNS servers were configured (`ipv4.dns` was empty). DNS information was supplied automatically through DHCP.

---

## View IPv4 Settings

Command:

```bash
nmcli connection show ens160 | grep ipv4
```

![IPv4 Settings](Lab19%20-%20Persistant%20Networking/06-ipv4-settings.jpg)

Observation:

The profile used `ipv4.method: auto`, with `ipv4.addresses`, `ipv4.gateway`, and `ipv4.dns` all empty. This is the network configuration before making changes.

---

## Configure Static Network Settings

### Configure Static IP Address

Command:

```bash
nmcli connection modify ens160 ipv4.addresses 192.168.243.150/24
```

![Configure Static IP Address](Lab19%20-%20Persistant%20Networking/07-configure-static-ip.jpg)

Purpose:

Set a static address.

### Configure Gateway

Command:

```bash
nmcli connection modify ens160 ipv4.gateway 192.168.243.2
```

Purpose:

Set the default route.

### Configure DNS

Command:

```bash
nmcli connection modify ens160 ipv4.dns 8.8.8.8
```

Purpose:

Set a DNS server.

### Change Address Method

Command:

```bash
nmcli connection modify ens160 ipv4.method manual
```

Purpose:

Switch from DHCP to static configuration.

---

## Verify Saved Configuration

Command:

```bash
nmcli connection show ens160 | grep ipv4
```

Output included:

```text
ipv4.method:      manual
ipv4.dns:         8.8.8.8
ipv4.addresses:   192.168.243.150/24
ipv4.gateway:     192.168.243.2
```

![Verify Saved Configuration](Lab19%20-%20Persistant%20Networking/08-verify-saved-configuration.jpg)

Observation:

NetworkManager successfully saved the new static configuration. The settings are stored in the profile but are not active until the connection is reloaded.

---

## Apply Configuration Changes

Disable the connection:

```bash
sudo nmcli connection down ens160
```

Enable the connection:

```bash
sudo nmcli connection up ens160
```

![Apply Configuration Changes](Lab19%20-%20Persistant%20Networking/09-apply-configuration-changes.jpg)

Observation:

The connection profile reloaded using the new settings.

---

## Verify New IP Address

Command:

```bash
ip a | grep inet
```

Output:

```text
inet 192.168.243.150/24 ... ens160
```

![Verify IP Address, Routing, and Connectivity](Lab19%20-%20Persistant%20Networking/10-verify-ip-routing-connectivity.jpg)

Observation:

The system successfully switched from DHCP to the configured static address. The word `dynamic` no longer appears in the output.

---

## Verify Routing

Command:

```bash
ip route
```

Output:

```text
default via 192.168.243.2 dev ens160 proto static metric 100
```

The output is in the same screenshot as the new IP address above.

Observation:

Gateway configuration remained functional after applying the new profile. The route now shows `proto static` instead of `proto dhcp`.

---

## Verify Connectivity

Command:

```bash
ping -c 4 google.com
```

Output:

```text
4 packets transmitted, 4 received, 0% packet loss
```

The output is in the same screenshot as the new IP address above.

Observation:

Successful DNS resolution and network communication were verified.

---

## Key Lessons Learned

- NetworkManager stores persistent network profiles.
- `nmcli` is the primary command-line tool for network management.
- DHCP automatically provides addressing information.
- Static addressing requires manual configuration.
- Gateway configuration allows external communication.
- DNS enables hostname resolution.
- Network profiles can be modified and reloaded without rebooting.
- Connectivity should always be verified after changes.

---

## Verification Checklist

- [x] Viewed network devices
- [x] Viewed saved connection profiles
- [x] Reviewed current IP address
- [x] Reviewed routing information
- [x] Verified DHCP configuration
- [x] Configured static IP address
- [x] Configured gateway
- [x] Configured DNS
- [x] Switched to manual addressing
- [x] Applied configuration changes
- [x] Verified static IP assignment
- [x] Verified gateway routing
- [x] Verified internet connectivity

---

## Screenshots

All screenshots for this lab: [Lab19 - Persistant Networking](Lab19%20-%20Persistant%20Networking/)

| # | File | Shows |
|---|------|-------|
| 01 | `01-network-devices-and-connections.jpg` | `nmcli device status` and `nmcli connection show` |
| 02 | `02-connection-profile-details.jpg` | `nmcli connection show ens160` |
| 03 | `03-current-ip-address.jpg` | `ip a \| grep inet` (DHCP address) |
| 04 | `04-address-method-and-routing.jpg` | `ipv4.method` check and `ip route` |
| 05 | `05-dns-configuration.jpg` | `nmcli connection show ens160 \| grep dns` |
| 06 | `06-ipv4-settings.jpg` | `nmcli connection show ens160 \| grep ipv4` (before) |
| 07 | `07-configure-static-ip.jpg` | `nmcli connection modify ens160 ipv4.addresses` |
| 08 | `08-verify-saved-configuration.jpg` | `nmcli connection show ens160 \| grep ipv4` (after) |
| 09 | `09-apply-configuration-changes.jpg` | `nmcli connection down` / `up` |
| 10 | `10-verify-ip-routing-connectivity.jpg` | `ip a`, `ip route`, and `ping` after the change |

---

## RHCSA Notes

View devices:

```bash
nmcli device status
```

View connections:

```bash
nmcli connection show
```

View IP address:

```bash
ip a
```

View routing table:

```bash
ip route
```

Configure static IP:

```bash
nmcli connection modify ens160 ipv4.addresses 192.168.243.150/24
```

Configure gateway:

```bash
nmcli connection modify ens160 ipv4.gateway 192.168.243.2
```

Configure DNS:

```bash
nmcli connection modify ens160 ipv4.dns 8.8.8.8
```

Set manual method:

```bash
nmcli connection modify ens160 ipv4.method manual
```

Apply configuration:

```bash
nmcli connection down ens160 && nmcli connection up ens160
```

Important Concept:

```text
DHCP = Router Decides
Static IP = Administrator Decides
Persistent Networking = Configuration Survives Reboots
```
