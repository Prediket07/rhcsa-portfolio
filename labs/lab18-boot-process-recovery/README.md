# Lab 18 — Boot Process & Recovery

## Objective

Demonstrate an understanding of the Linux boot process, systemd targets, boot logs, failed services, recovery targets, and the difference between temporary and permanent boot target changes.

## Environment

- OS: Rocky Linux 10.2
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation
- Screenshots: [Lab18 - Boot Process and Recovery](Lab18%20-%20Boot%20Process%20and%20Recovery/)

## Concepts Covered

| Command | Purpose |
|----------|----------|
| systemctl get-default | Display default boot target |
| systemctl --failed | Display failed services and units |
| systemctl status | Investigate service failures |
| journalctl -b | Display current boot logs |
| systemctl list-units --type=target | Display active boot targets |
| systemctl isolate | Temporarily switch targets |
| systemctl cat | View target configuration |
| ls -l /etc/systemd/system/default.target | View permanent boot target |

---

## Understanding the Linux Boot Process

When a Linux system starts, several components work together before the login screen appears.

```text
UEFI / BIOS
↓
GRUB Boot Loader
↓
Linux Kernel
↓
systemd
↓
Target
↓
Login Screen
```

Think of starting a car:

```text
Engine Starts
↓
Systems Activate
↓
Ready to Drive
```

---

## Default Boot Target

Command:

```bash
systemctl get-default
```

Output:

```text
graphical.target
```

![Default Boot Target](Lab18%20-%20Boot%20Process%20and%20Recovery/01-default-boot-target.jpg)

Observation:

The system is configured to boot into the graphical desktop environment.

---

## Failed Services

Command:

```bash
systemctl --failed
```

Output:

```text
run-vmblock\x2dfuse.mount   loaded failed failed   VMware vmblock Fuse Mount
```

![Failed Services](Lab18%20-%20Boot%20Process%20and%20Recovery/02-failed-services.jpg)

Observation:

One failed mount unit was identified. Checking for failed units is a quick way to find services requiring attention after boot.

---

## Investigating the Failed Unit

Command:

```bash
systemctl status run-vmblock\\x2dfuse.mount
```

Key output:

```text
Active: failed (Result: exit-code) since Fri 2026-10-16 19:27:00 CDT
Where: /run/vmblock-fuse
What: vmware-vmblock-fuse
mount: /run/vmblock-fuse: wrong fs type, bad option, bad superblock on vmware-vmblock-fuse,
missing codepage or helper program, or other error.
```

![Investigate Failed Unit](Lab18%20-%20Boot%20Process%20and%20Recovery/04-investigate-failed-unit.jpg)

Observation:

The VMware vmblock Fuse Mount process exited with an error during startup. This unit belongs to the VMware guest integration, not to anything configured in this lab, and the rest of the system booted normally. Administrators use `systemctl status` to investigate startup failures.

---

## Boot Logs

Command:

```bash
journalctl -b | head -20
```

![Boot Logs](Lab18%20-%20Boot%20Process%20and%20Recovery/03-boot-logs.jpg)

Observation:

The output showed the earliest messages from the current boot, including:

- Kernel version and startup command line
- BIOS memory map
- Hardware and virtualization detection (VMware Virtual Platform)
- Security features such as NX protection

These logs record what occurred during system startup.

---

## Systemd Targets

Command:

```bash
systemctl list-units --type=target
```

![Active Systemd Targets](Lab18%20-%20Boot%20Process%20and%20Recovery/05-list-active-targets.jpg)

Observation:

24 targets were loaded and active, including:

```text
graphical.target   multi-user.target   network.target   basic.target
```

Targets are groups of units that define startup destinations within systemd.

---

## Exploring Common Targets

| Target | Purpose | Typical Use |
|----------|----------|----------|
| graphical.target | Full desktop with GUI-based login | Desktop systems |
| multi-user.target | Command line, networking enabled, services enabled | Server systems |
| rescue.target | Minimal services, troubleshooting environment | Repairing a system |
| emergency.target | Minimal environment, root shell access | System recovery |

---

## Multi-User Target

Switch to the multi-user target temporarily:

```bash
systemctl isolate multi-user.target
```

Switching targets required authentication.

![Isolate Multi-User Target - Authentication](Lab18%20-%20Boot%20Process%20and%20Recovery/06-isolate-multi-user-auth.jpg)

Result:

The desktop environment stopped and the system switched to a text-based login prompt.

![Multi-User Login Prompt](Lab18%20-%20Boot%20Process%20and%20Recovery/07-multi-user-login-prompt.jpg)

After logging in at the console:

```bash
whoami
systemctl get-default
```

Output:

```text
jross
graphical.target
```

![Multi-User Console Session](Lab18%20-%20Boot%20Process%20and%20Recovery/08-multi-user-console-session.jpg)

Observation:

The system was running in server-style operation, but the default boot target was still `graphical.target`. Isolating a target only changes the current session.

---

## Graphical Target

Return to the desktop:

```bash
systemctl isolate graphical.target
```

Observation:

The graphical desktop environment loaded again without a reboot.

---

## Rescue Target

Command:

```bash
systemctl cat rescue.target
```

Key output:

```text
Description=Rescue Mode
Requires=sysinit.target rescue.service
After=sysinit.target rescue.service
AllowIsolate=yes
```

![Rescue and Emergency Target Units](Lab18%20-%20Boot%20Process%20and%20Recovery/09-rescue-emergency-target-units.jpg)

Observation:

Rescue mode starts a minimal environment with basic system initialization, used for troubleshooting. This lab viewed the target definition only; it did not boot into rescue mode.

---

## Emergency Target

Command:

```bash
systemctl cat emergency.target
```

Key output:

```text
Description=Emergency Mode
Requires=emergency.service
After=emergency.service
AllowIsolate=yes
```

The screenshot in the Rescue Target section shows both unit files.

Observation:

Emergency mode requires only `emergency.service`, so it is more minimal than rescue mode. It provides a root shell for system recovery when even rescue mode will not start.

---

## Permanent Boot Target Configuration

Command:

```bash
ls -l /etc/systemd/system/default.target
```

Output:

```text
/etc/systemd/system/default.target -> /usr/lib/systemd/system/graphical.target
```

![Default Target Symlink](Lab18%20-%20Boot%20Process%20and%20Recovery/10-default-target-symlink.jpg)

Observation:

The default boot target is a symbolic link. This is how the system is configured to boot at startup.

---

## Temporary vs Permanent Changes

Temporary change:

```bash
systemctl isolate multi-user.target
```

Affects the current session only.

Permanent change:

```bash
systemctl set-default multi-user.target
```

Changes the target the system boots into. (Documented for reference; the default was not changed in this lab.)

---

## Key Lessons Learned

- The boot process is managed by systemd.
- Targets define the system's operational state.
- `systemctl --failed` is the first stop for startup problems.
- `systemctl status` and `journalctl` show why a unit failed.
- Multi-user target provides server-style operation without the desktop.
- Rescue and emergency targets are recovery environments.
- Temporary (`isolate`) and permanent (`set-default`) target changes are different concepts.

---

## Verification Checklist

- [x] Verified default boot target
- [x] Listed failed services
- [x] Investigated failed unit
- [x] Reviewed boot logs
- [x] Listed active targets
- [x] Switched to multi-user target
- [x] Verified default target was unchanged
- [x] Returned to graphical target
- [x] Viewed rescue target configuration
- [x] Viewed emergency target configuration
- [x] Verified permanent default target

---

## Screenshots

All screenshots for this lab: [Lab18 - Boot Process and Recovery](Lab18%20-%20Boot%20Process%20and%20Recovery/)

| # | File | Shows |
|---|------|-------|
| 01 | `01-default-boot-target.jpg` | `systemctl get-default` |
| 02 | `02-failed-services.jpg` | `systemctl --failed` |
| 03 | `03-boot-logs.jpg` | `journalctl -b` |
| 04 | `04-investigate-failed-unit.jpg` | `systemctl status` on the failed vmblock-fuse mount |
| 05 | `05-list-active-targets.jpg` | `systemctl list-units --type=target` |
| 06 | `06-isolate-multi-user-auth.jpg` | `systemctl isolate multi-user.target` asking for authentication |
| 07 | `07-multi-user-login-prompt.jpg` | Text-mode login prompt after switching targets |
| 08 | `08-multi-user-console-session.jpg` | Console login, `whoami`, `systemctl get-default` |
| 09 | `09-rescue-emergency-target-units.jpg` | `systemctl cat rescue.target` and `emergency.target` |
| 10 | `10-default-target-symlink.jpg` | `ls -l /etc/systemd/system/default.target` |

---

## RHCSA Notes

Useful boot and target commands:

```bash
systemctl get-default

systemctl --failed

systemctl status <unit>

journalctl -b

systemctl list-units --type=target

systemctl isolate multi-user.target

systemctl isolate graphical.target

systemctl cat rescue.target

systemctl set-default multi-user.target
```

Most important concept:

```text
isolate      = temporary (current session)
set-default  = permanent (survives reboot)
```
