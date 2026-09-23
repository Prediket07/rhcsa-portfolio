# Lab 05 — systemd Services

## Objective

Demonstrate management of Linux services using systemd, including starting, stopping, restarting, enabling, disabling, and viewing service logs.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab
- Primary User: jross
- Hypervisor: VMware Workstation

## Concepts Covered

| Command | Purpose |
|----------|----------|
| systemctl status | View service status |
| systemctl start | Start a service |
| systemctl stop | Stop a service |
| systemctl restart | Restart a service |
| systemctl enable | Enable at boot |
| systemctl disable | Disable at boot |
| systemctl is-enabled | Check startup state |
| journalctl | View logs |
| journalctl -u | View logs for a specific service |

---

## Understanding Active vs Enabled

A key lesson from this lab:

```text
Active = Running right now

Enabled = Starts automatically at boot
```

These are different states.

---

## Check Service Status

![sshd status](Lab05%20-%20systemd%20Services/01-sshd-status.jpg)

```bash
systemctl status sshd
```

Verified:

```text
Loaded: loaded
Active: active (running)
Enabled: enabled
```

Observation:

The SSH daemon was installed, running, and configured to start automatically.

---

## Stop a Service

![sshd stop](Lab05%20-%20systemd%20Services/02-sshd-stop.jpg)

```bash
sudo systemctl stop sshd
```

Verify:

```bash
systemctl status sshd
```

Result:

```text
Active: inactive
```

Observation:

The service stopped immediately.

---

## Start a Service

![sshd start](Lab05%20-%20systemd%20Services/03-sshd-start.jpg)

```bash
sudo systemctl start sshd
```

Verify:

```bash
systemctl status sshd
```

Result:

```text
Active: active (running)
```

Observation:

The service was returned to an active state.

---

## Restart a Service



```bash
sudo systemctl restart sshd
```

Observation:

Restart performs a stop followed by a start.

Common use:

```text
After configuration changes
```

---

## Check Startup State

![sshd is-enabled](Lab05%20-%20systemd%20Services/04-sshd-is-enabled.jpg)

```bash
systemctl is-enabled sshd
```

Result:

```text
enabled
```

Meaning:

```text
The service will start automatically at boot.
```

---

## Disable a Service

![sshd disabled](Lab05%20-%20systemd%20Services/05-sshd-disabled.jpg)

```bash
sudo systemctl disable sshd
```

Verify:

```bash
systemctl is-enabled sshd
```

Result:

```text
disabled
```

Observation:

The service remained running but would not start automatically after reboot.

---

## Enable a Service

```bash
sudo systemctl enable sshd
```

Verify:

```bash
systemctl is-enabled sshd
```

Result:

```text
enabled
```

Observation:

The service is configured to start automatically at boot.

---

## View System Logs

```bash
journalctl
```

Observation:

Displays all system logs.

---

## View Service Logs

![sshd journalctl](Lab05%20-%20systemd%20Services/06-sshd-journalctl.jpg)

```bash
journalctl -u sshd
```

Observation:

Displays only SSH-related service logs.

Examples:

```text
Starting sshd.service
Started sshd.service
Stopping sshd.service
Stopped sshd.service
```

---

## View Firewall Logs

```bash
journalctl -u firewalld
```

Observation:

Displays only firewall-related service logs.

---

## Key Lessons Learned

- systemd manages Linux services.
- start, stop, and restart affect current operation.
- enable and disable affect future boot behavior.
- Active and Enabled are different states.
- journalctl is used to investigate service activity.
- journalctl -u filters logs for a specific service.

---

## Verification Checklist

- [x] Viewed service status
- [x] Stopped a service
- [x] Started a service
- [x] Restarted a service
- [x] Checked service startup state
- [x] Disabled a service
- [x] Enabled a service
- [x] Viewed system logs
- [x] Viewed service-specific logs

---

## RHCSA Notes

Common commands:

```bash
systemctl status sshd
systemctl start sshd
systemctl stop sshd
systemctl restart sshd
systemctl enable sshd
systemctl disable sshd
systemctl is-enabled sshd
journalctl -u sshd
```

Most important concept:

```text
Running Now
≠
Starts At Boot
```

