# Lab 06 — Networking Basics

## Objective

Demonstrate fundamental Linux networking concepts including hostname identification, network interfaces, IP addressing, routing, connectivity testing, and DNS configuration.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation

## Concepts Covered

| Command | Purpose |
|----------|----------|
| hostname | Display system hostname |
| ip addr | Display network interfaces and IP addresses |
| ip route | Display routing table |
| ping | Test connectivity |
| cat /etc/resolv.conf | Display DNS configuration |

---

## Understanding Networking

A Linux system requires several components to communicate on a network:

```text
Hostname = Computer name

IP Address = Network address

Gateway = Path to other networks

DNS = Name resolution service
```

---

## Display Hostname

```bash
hostname
```

Output:

```text
rhcsa-lab.local
```

Observation:

The hostname identifies the system on the network.

---

## Display Network Configuration

```bash
ip addr
```

Important findings:

### Loopback Interface

```text
lo
127.0.0.1
```

Purpose:

```text
The system communicating with itself.
```

### Network Interface

```text
ens160
```

Purpose:

```text
Primary network adapter.
```

### IPv4 Address

```text
192.168.243.128/24
```

Purpose:

```text
Primary network address assigned to the VM.
```

---

## View Routing Information

```bash
ip route
```

Output:

```text
default via 192.168.243.2 dev ens160
```

### Default Gateway

```text
192.168.243.2
```

Purpose:

The default gateway acts as the path to external networks.

Without a default gateway, systems can usually communicate only with devices on the local network.

---

## Test Network Connectivity

```bash
ping -c 4 google.com
```

Output:

```text
4 packets transmitted
4 received
0% packet loss
```

Observation:

The host successfully communicated with an external destination.

Successful results confirmed:

- Network connectivity
- Gateway functionality
- DNS functionality

---

## Understanding Ping Results

Example:

```text
ttl=128
time=6.88 ms
```

Meaning:

```text
ttl = Time To Live

time = Response time
```

Lower response times generally indicate faster communication.

---

## View DNS Configuration

```bash
cat /etc/resolv.conf
```

Output:

```text
nameserver 192.168.243.2
```

Purpose:

The DNS server translates hostnames into IP addresses.

Example:

```text
google.com
```

becomes:

```text
216.239.x.x
```

Without DNS, systems would require direct IP addresses instead of hostnames.

---

## Key Lessons Learned

- Every Linux system has a hostname.
- The loopback address is always used for local system communication.
- Network interfaces provide connectivity.
- IP addresses identify systems on a network.
- Default gateways provide access beyond the local network.
- DNS servers translate names into IP addresses.
- Ping verifies connectivity and name resolution.

---

## Verification Checklist

- [x] Verified hostname
- [x] Identified loopback interface
- [x] Identified primary network interface
- [x] Identified IPv4 address
- [x] Identified default gateway
- [x] Tested network connectivity
- [x] Verified DNS configuration
- [x] Confirmed successful name resolution

---

## Screenshots

### Hostname and Interface Information

![Hostname and IP address](Lab06%20-%20Networking%20Basics/01-hostname-and-ip-addr.jpg)

Screenshot: 01-hostname-and-ip-addr

Demonstrated:

- Hostname
- Loopback interface
- Network interface
- IPv4 address

---

### Routing and Connectivity

![ip route and ping](Lab06%20-%20Networking%20Basics/02-ip-route-and-ping.jpg)

Screenshot: 02-ip-route-and-ping

Demonstrated:

- Default gateway
- Successful connectivity test
- 0% packet loss

---

### DNS Configuration

![DNS Configuration](Lab06%20-%20Networking%20Basics/03%20-resolv-conf.jpg)

Screenshot: `03-resolv-conf.jpg`

Demonstrated:

- DNS server configuration
- Name resolution settings

---

## RHCSA Notes

Useful networking commands:

```bash
hostname

ip addr

ip route

ping -c 4 google.com

cat /etc/resolv.conf
```

Networking concepts:

```text
Hostname = Computer name

IP Address = Device address

Gateway = Path to other networks

DNS = Phone book of the internet
```
