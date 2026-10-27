# Lab 22 — Time Synchronization (Chrony)

## Objective

Demonstrate how to verify, monitor, and inspect time synchronization on Rocky Linux using Chrony.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation
- Screenshots: [Lab22 - Time Synchronization](Lab22%20-%20Time%20Synchronization/)

## Concepts Covered

| Command | Purpose |
|----------|----------|
| systemctl status chronyd | Verify Chrony service status |
| chronyc tracking | View synchronization status |
| chronyc sources | Display time sources |
| grep /etc/chrony.conf | Review Chrony configuration |

---

## Understanding Time Synchronization

Linux systems must maintain accurate time.

Accurate system clocks are important for:

- Log analysis
- Authentication
- Security auditing
- Certificates
- Troubleshooting

Chrony is the default time synchronization service used by Rocky Linux.

Think:

```text
Chrony
=
Internet Clock
```

---

## Verify Chrony Service

Command:

```bash
systemctl status chronyd
```

![Chronyd Status and Tracking](Lab22%20-%20Time%20Synchronization/01-chronyd-status-and-tracking.jpg)

Observation:

```text
Active: active (running)
Loaded: enabled
```

The Chrony service was running and configured to start automatically at boot.

---

## Understanding Chronyd

Chronyd is responsible for:

- Maintaining accurate system time
- Synchronizing with remote time servers
- Correcting clock drift

Think:

```text
Chronyd
=
Linux Clock Manager
```

---

## Verify Synchronization Status

Command:

```bash
chronyc tracking
```

The output is in the same screenshot as `systemctl status chronyd` above.

Observation:

Displayed:

- Reference ID
- Stratum
- System Time
- Last Offset
- Leap Status

Important Output:

```text
Leap status : Normal
```

Observation:

The system was successfully synchronized and operating normally.

---

## Understanding Tracking Information

### Reference ID

Example:

```text
68.183.147.240
```

Represents the currently selected synchronization source.

### Stratum

Example:

```text
Stratum 4
```

Represents the distance from the primary reference clock.

Lower numbers are generally closer to the reference source.

### Leap Status

Example:

```text
Normal
```

Indicates that synchronization is healthy.

---

## View Time Sources

Command:

```bash
chronyc sources
```

![Chrony Time Sources](Lab22%20-%20Time%20Synchronization/02-tracking-and-sources.jpg)

Observation:

Displayed multiple time sources.

Examples included:

```text
sushi.ruselabs.com
ntp3.radio-sunshine.org
69.9.131.113
68.183.147.240
```

---

## Understanding Source Indicators

Chrony evaluates multiple sources and selects the most appropriate one.

Important symbol:

```text
^*
```

Meaning:

```text
Currently selected synchronization source
```

This source is actively providing time updates.

---

## View Chrony Configuration

Command:

```bash
grep -v '^#' /etc/chrony.conf | grep -v '^$'
```

![Chrony Configuration](Lab22%20-%20Time%20Synchronization/03-chrony-configuration.jpg)

Observation:

Displayed active configuration without comments.

Key entry:

```text
pool 2.rocky.pool.ntp.org iburst
```

---

## Understanding Configuration

### NTP Pool

```text
2.rocky.pool.ntp.org
```

Provides multiple available synchronization servers.

### iburst

```text
iburst
```

Allows Chrony to synchronize quickly during startup.

Think:

```text
Fast Initial Time Sync
```

---

## Key Lessons Learned

- Chrony maintains accurate Linux system time.
- chronyd runs as a system service.
- Multiple time sources improve reliability.
- Chrony automatically selects preferred sources.
- The active source is identified with `^*`.
- Synchronization health can be verified using `chronyc tracking`.
- Configuration can be reviewed through `/etc/chrony.conf`.

---

## Verification Checklist

- [x] Verified Chrony service status
- [x] Confirmed service was running
- [x] Verified service was enabled
- [x] Reviewed synchronization tracking information
- [x] Verified healthy Leap Status
- [x] Listed available time sources
- [x] Identified active source
- [x] Reviewed Chrony configuration

---

## Screenshots

All screenshots for this lab: [Lab22 - Time Synchronization](Lab22%20-%20Time%20Synchronization/)

| # | File | Shows |
|---|------|-------|
| 01 | `01-chronyd-status-and-tracking.jpg` | `systemctl status chronyd` and `chronyc tracking` |
| 02 | `02-tracking-and-sources.jpg` | `chronyc tracking` and `chronyc sources` |
| 03 | `03-chrony-configuration.jpg` | `grep -v '^#' /etc/chrony.conf \| grep -v '^$'` |

---

## RHCSA Notes

Verify service:

```bash
systemctl status chronyd
```

Check synchronization:

```bash
chronyc tracking
```

View sources:

```bash
chronyc sources
```

View configuration:

```bash
grep -v '^#' /etc/chrony.conf | grep -v '^$'
```

Most Important Indicators:

```text
active (running)
```

Service operational.

```text
enabled
```

Starts automatically at boot.

```text
Leap status: Normal
```

Time synchronization healthy.

```text
^*
```

Active synchronization source.

Most Important Concept:

```text
Chrony
=
Linux Time Synchronization Service
```
