# Lab 12 — Log Management

## Objective

Demonstrate Linux log management by exploring systemd journal logs, filtering logs by service and boot session, monitoring logs in real time, and analyzing traditional log files within `/var/log`.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation

## Concepts Covered

| Command | Purpose |
|----------|----------|
| journalctl | View system journal |
| journalctl -n | Display recent log entries |
| journalctl -u | Display logs for a specific service |
| journalctl -b | Display logs from current boot |
| journalctl -k | Display kernel logs |
| journalctl -f | Follow logs in real time |
| tail | Display end of log file |
| grep | Search log contents |
| /var/log/messages | General system log |
| /var/log/secure | Authentication and security log |
| /var/log/dnf.log | Package management log |

---

## Understanding Linux Logs

Linux logs provide a history of system activity.

Logs are often used to:

- Troubleshoot problems
- Investigate authentication events
- Verify service activity
- Analyze boot events
- Track software installation activity

Think:

```text
Processes
↓
Create Events
↓
Events Become Logs
↓
Logs Become Evidence
```

---

## View Recent Journal Entries

Command:

```bash
journalctl -n 20
```

Purpose:

```text
Show the most recent 20 journal entries.
```

Observation:

Recent system activity appeared including:

- systemd events
- package activity
- desktop environment messages

Examples seen:

```text
PackageKit: Skipping refresh of media: Cannot update read-only repo
systemd: Created slice user.slice
gnome-shell: Invalid sequence for VSYNC frame info
```

`journalctl` color-codes entries by priority. The yellow lines in this output are warnings.

![journalctl -n 20 output](Lab12%20-%20Log%20Management/01-journalctl-n20.jpg)

---

## View Service Logs

Command:

```bash
journalctl -u sshd -n 20
```

Purpose:

```text
Display recent SSH service events.
```

Observation:

Displayed service startup activity including:

```text
Starting sshd.service - OpenSSH server daemon...
Server listening on 0.0.0.0 port 22.
Server listening on :: port 22.
Started sshd.service - OpenSSH server daemon.
```

Only 4 entries were returned even though 20 were requested. `-n 20` sets a maximum, and only 4 entries existed for `sshd`. All were logged at boot time.

![journalctl -u sshd output](Lab12%20-%20Log%20Management/02-journalctl-sshd-and-boot.jpg)

---

## View Current Boot Logs

Command:

```bash
journalctl -b
```

Purpose:

```text
Display logs generated during the current boot.
```

Observation:

Displayed:

- Linux kernel startup
- Hardware detection
- VMware platform detection
- Memory initialization

Example:

```text
Linux version ...
Hypervisor detected: VMware
```

The output opens in a pager (`less`). The `lines 1-40` indicator at the bottom shows the pager is active.

Exit:

```text
q
```

![journalctl -b in the pager](Lab12%20-%20Log%20Management/03-journalctl-boot-pager.jpg)

---

## View Kernel Logs

Command:

```bash
journalctl -k -n 20
```

Purpose:

```text
Display the last 20 kernel messages.
```

Observation:

Displayed:

- VMware device detection
- Network interface initialization
- Hardware-related messages

Example:

```text
vmxnet3 0000:03:00.0 ens160: NIC Link is Up 10000 Mbps
```

Other entries included `rfkill` input handler messages, ISO 9660 extensions (the mounted install DVD), and a block of kernel warning lines (highlighted yellow) from the earlier boot.

The top of this screenshot also shows the `^C` that ended `journalctl -f` in the next section.

![journalctl -k output](Lab12%20-%20Log%20Management/05-journalctl-kernel.jpg)

---

## Follow Logs Live

Command:

```bash
journalctl -f
```

Purpose:

```text
Display new log messages in real time.
```

Observation:

Showed the latest entries, then waited for new ones, leaving the cursor on a blank line.

Examples included:

```text
PackageKit
NetworkManager
systemd services
```

Specific entries seen:

- `packagekit.service: Deactivated successfully`
- `systemd-tmpfiles-clean.service` running a cleanup of temporary directories
- `NetworkManager` reporting a DHCP lease change on `ens160`
- `NetworkManager-dispatcher.service` starting and stopping

Exit:

```text
Ctrl + C
```

![journalctl -f output](Lab12%20-%20Log%20Management/04-journalctl-follow.jpg)

---

## Explore Traditional Log Files

Command:

```bash
ls /var/log
```

Observation:

Identified common Linux log files including:

```text
messages
secure
cron
dnf.log
boot.log
```

Many files also had dated copies, such as `messages-20260927` and `secure-20260927`. These are older, rotated versions kept by log rotation, so the main file stays a manageable size.

![ls /var/log output](Lab12%20-%20Log%20Management/06-var-log-directory.jpg)

---

## Analyze System Log

Attempt:

```bash
tail /var/log/messages
```

Result:

```text
tail: cannot open '/var/log/messages' for reading: Permission denied
```

Resolution:

```bash
sudo tail /var/log/messages
```

Observation:

Displayed:

- system services
- runtime events
- user session activity
- service startup messages

Examples seen:

```text
Starting fprintd.service - Fingerprint Authentication Daemon...
New session 4 of user root.
Created slice user-0.slice
```

Purpose:

```text
General system activity log.
```

---

## Analyze Security Log

Command:

```bash
sudo tail /var/log/secure
```

Observation:

Displayed:

- authentication events
- login activity
- sudo commands
- session creation

Example:

```text
jross : TTY=pts/0 ; PWD=/home/jross/rhcsa-portfolio ; USER=root ; COMMAND=/bin/tail /var/log/messages
```

Meaning:

```text
jross used sudo to run:

tail /var/log/messages
```

The log also recorded the sudo session being opened for `root` by `jross (uid=1000)` and then closed.

Important lesson:

```text
Logs can record administrator activity.
```

![Permission denied, sudo tail messages, and sudo tail secure](Lab12%20-%20Log%20Management/08-messages-and-secure.jpg)

---

## Analyze DNF Log

Command:

```bash
tail /var/log/dnf.log
```

Observation:

Displayed package management activity including:

```text
Metadata refresh
Repository access
Cache creation
Plugin activity
```

Specific entries: `INFO Metadata cache created.`, `Completion plugin: Generating completion cache...`, and `Plugins were unloaded.`

This command worked without `sudo`, unlike `/var/log/messages`. Not every log file is restricted.

Purpose:

```text
Track software installation and update activity.
```

![tail /var/log/dnf.log output](Lab12%20-%20Log%20Management/07-dnf-log.jpg)

---

## Log Categories

### Journal

```bash
journalctl
```

Think:

```text
Master system journal
```

---

### General System Events

```text
/var/log/messages
```

Contains:

- service activity
- system events
- runtime operations

---

### Authentication Events

```text
/var/log/secure
```

Contains:

- sudo usage
- user authentication
- login events
- SSH activity

---

### Software Activity

```text
/var/log/dnf.log
```

Contains:

- installations
- updates
- repository actions

---

## Key Lessons Learned

- Logs provide evidence of system activity.
- systemd journal centralizes logging information.
- Logs can be filtered by service.
- Logs can be filtered by boot session.
- Kernel logs help troubleshoot hardware and drivers.
- Log files often require elevated privileges, though some (like `dnf.log`) are readable without `sudo`.
- `/var/log/messages` stores general activity.
- `/var/log/secure` tracks authentication events.
- `/var/log/dnf.log` tracks package management activity.
- `journalctl -n` sets a maximum, so it can return fewer lines than requested.
- Long `journalctl` output opens in a pager; press `q` to exit.
- Rotated log copies with dates in their names are older versions of the same log.
- Logs often answer questions about what happened, when it happened, and who performed an action.

---

## Verification Checklist

- [x] Viewed recent journal entries
- [x] Viewed service logs
- [x] Viewed current boot logs
- [x] Viewed kernel logs
- [x] Followed logs live
- [x] Explored `/var/log`
- [x] Analyzed system log
- [x] Analyzed security log
- [x] Analyzed package management log

---

## RHCSA Notes

Useful Log Commands:

```bash
journalctl -n 20

journalctl -u sshd

journalctl -b

journalctl -k

journalctl -f

sudo tail /var/log/messages

sudo tail /var/log/secure

tail /var/log/dnf.log
```

Troubleshooting Flow:

```text
Problem
↓
Gather Evidence
↓
Read Logs
↓
Determine Cause
↓
Fix Issue
↓
Verify Solution
```
