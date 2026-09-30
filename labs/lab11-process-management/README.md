# Lab 11 — Process Management

## Objective

Demonstrate Linux process management by identifying running processes, monitoring system activity, creating background processes, locating process IDs (PIDs), terminating processes, and verifying process removal.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation

## Concepts Covered

| Command | Purpose |
|----------|----------|
| ps | Display active processes |
| ps aux | View all running processes |
| ps -ef | View all running processes (alternate format) |
| pstree | Display process hierarchy |
| top | Real-time process monitoring |
| pgrep | Find PID by process name |
| pidof | Display process IDs |
| kill | Terminate process by PID |
| killall | Terminate process by name |
| sudo ss -tulpn | List listening network ports and their processes |
| watch | Repeat a command at a set interval |

---

## Understanding Processes

A process is:

```text
A running program
```

Examples:

- sshd
- firewalld
- chronyd
- bash

Every process receives a unique:

```text
PID (Process ID)
```

used to identify and control it.

---

## View Active Processes

Command:

```bash
ps
```

Observation:

Displayed processes associated with the current terminal session: `bash` (PID 5592) and the `ps` command itself (PID 7066).

![ps and ps aux output](01-ps-and-ps-aux.jpg)

---

## View All Processes

Command:

```bash
ps aux | head
```

Observation:

Displayed running processes on the system regardless of ownership. `head` limited the output to the first 10 lines. PID 1 is `systemd`, the parent of all other processes.

![ps aux output](01-ps-and-ps-aux.jpg)

---

## Display Process Hierarchy

Command:

```bash
pstree
```

Observation:

Displayed parent-child relationships between running processes, with `systemd` at the root.

This helped visualize how services and applications are related. Services such as `chronyd`, `firewalld`, `sshd`, and `crond` all appear as children of `systemd`.

![pstree output](02-pstree.jpg)

---

## Monitor Processes in Real Time

Command:

```bash
top
```

Observation:

Displayed:

- CPU utilization
- Memory utilization
- Running processes
- Process IDs
- System load

The header showed 324 total tasks (1 running, 323 sleeping, 0 zombie) and load averages of 0.00, 0.04, 0.08, meaning the system was idle.

Important fields:

```text
PID
CPU
Memory
Command
```

The `NI` column also showed nice values (such as `-20` on some kernel workers), which connects to priority management.

Exit:

```text
q
```

![top output](03-top.jpg)

---

## Create Background Process

Command:

```bash
sleep 300 &
```

Result:

```text
[1] 7565
```

Observation:

Linux created a background process and assigned a PID.

The ampersand (`&`) allowed the command to run without occupying the terminal.

![Background process, ps, pgrep, and kill attempt](04-background-process-and-natural-exit.jpg)

---

## Locate Process with ps

Command:

```bash
ps aux | grep sleep
```

Observation:

Displayed:

```text
sleep 300
```

with PID:

```text
7565
```

The second line in the output is the `grep` command itself, which also contains the word "sleep".

(See the screenshot in the previous section.)

---

## Locate Process with pgrep

Command:

```bash
pgrep sleep
```

Output:

```text
7565
```

Observation:

Displayed the PID directly.

(See the screenshot in the Create Background Process section.)

---

## Process Completed Naturally

Attempted:

```bash
kill 7565
```

Result:

```text
No such process
```

Shortly afterward:

```text
Done sleep 300
```

Observation:

The process completed before the termination request reached it. `sleep 300` waits five minutes, and enough time had passed between starting it and running `kill`.

Important lesson:

```text
Processes may terminate normally without administrative intervention.
```

![Process completed naturally](04-background-process-and-natural-exit.jpg)

---

## Kill Process by PID

Verify the old process is gone:

```bash
kill 7565
pgrep sleep
```

Result:

```text
No such process
(no output from pgrep)
```

Create new process:

```bash
sleep 1000 &
```

Locate PID:

```bash
pgrep sleep
```

Result:

```text
7600
```

Terminate:

```bash
kill 7600
```

Result:

```text
[1]+ Terminated    sleep 1000
```

Verify:

```bash
pgrep sleep
```

Result:

```text
(no output)
```

Observation:

Process terminated successfully.

Troubleshooting note:

```bash
kill <PID>
```

typed literally produced:

```text
bash: syntax error near unexpected token `newline'
```

`<PID>` is a placeholder, not part of the command. The shell reads `<` as input redirection, so the real PID number must be typed instead (`kill 7600`).

![Kill by PID](05-kill-by-pid.jpg)

---

## Kill Process by Name

Create another process:

```bash
sleep 1000 &
```

Result:

```text
[1] 7617
```

Terminate:

```bash
killall sleep
```

Result:

```text
[1]+ Terminated    sleep 1000
```

Verify:

```bash
pgrep sleep
```

Result:

```text
(no output)
```

Observation:

All processes named:

```text
sleep
```

were terminated.

![killall sleep](06-killall.jpg)

---

## Retrieve Process IDs

Chrony process:

```bash
pidof chronyd
```

Output:

```text
1221
```

Observation:

Displayed the active process ID for the chronyd service. The same PID (1221) appears in the `ps -ef` output below.

---

## Investigate Firewalld Process

Command:

```bash
pidof firewalld
```

Result:

```text
(no output)
```

Additional investigation:

```bash
ps aux | grep firewalld
```

Result:

```text
python3 ... firewalld
```

Observation:

The service was running but was not identified by `pidof` as expected.

Why: `pidof` searches by the process name, and firewalld is a Python script. Its process name is `python3`, with `/usr/sbin/firewalld` passed as an argument. The full command line was:

```text
/usr/bin/python3 -sP /usr/sbin/firewalld --nofork --nopid
```

Important lesson:

```text
Multiple tools may be required when troubleshooting processes.
```

---

## Alternate Process View

Commands:

```bash
ps -ef | grep chronyd
```

and

```bash
ps -ef | grep firewalld
```

Observation:

Provided alternative process information including:

- PID
- Parent PID
- Start Time
- Process Command

Results:

| Service | PID | PPID | Command |
|----------|-----|------|---------|
| chronyd | 1221 | 1 | /usr/sbin/chronyd -n -F 2 |
| firewalld | 1294 | 1 | /usr/bin/python3 -sP /usr/sbin/firewalld --nofork --nopid |

Both services have a PPID of 1, meaning `systemd` started them.

![ps -ef for chronyd and firewalld](07-ps-ef-chronyd-firewalld.jpg)

---

## Additional Exploration

### List Listening Ports with ss

Command:

```bash
sudo ss -tulpn
```

Flags:

```text
-t  TCP sockets
-u  UDP sockets
-l  Listening sockets only
-p  Show the process using each socket
-n  Show numeric ports instead of service names
```

Observation:

Listed every port the system was listening on, along with the process that owns it:

| Protocol | Local Address:Port | Process | PID |
|----------|--------------------|---------|-----|
| UDP | 127.0.0.1:323 and [::1]:323 | chronyd | 1221 |
| UDP | 0.0.0.0:5353 and [::]:5353 | avahi-daemon | 1220 |
| TCP | 0.0.0.0:22 and [::]:22 | sshd | 1411 |
| TCP | 127.0.0.1:631 and [::1]:631 | cupsd | 1410 |
| TCP | *:9090 | systemd | 1 |

Notes:

- `sudo` was needed for the `Process` column. An earlier run as the regular user (`jross`) showed the same ports but left that column empty, because `ss -p` can only show processes the current user owns.
- The chronyd PID (1221) matches the `ps -ef` output earlier in this lab.
- Port 9090 is owned by `systemd` (PID 1) rather than a named service. This is consistent with socket activation, where systemd listens on the port and starts the service only when a connection arrives. 9090 is the default port for the Cockpit web console.
- The services listed here (`chronyd`, `sshd`, `cupsd`, `avahi-daemon`) also appear in the `pstree` output, connecting running processes to the network ports they open.

![sudo ss -tulpn output](08-ss-tulpn.jpg)

---

### Monitor Disk Space Live with watch

Command:

```bash
watch -n 2 df -h
```

Breakdown:

```text
watch    Repeat a command
-n 2     Every 2 seconds
df -h    Disk free, human-readable sizes
```

Observation:

The header showed `Every 2.0s: df -h` with a timestamp that refreshed on each cycle. Key filesystems:

| Filesystem | Size | Used | Avail | Use% | Mounted on |
|------------|------|------|-------|------|------------|
| /dev/mapper/rl_rhcsa--lab-root | 16G | 5.3G | 11G | 33% | / |
| /dev/nvme0n1p2 | 2.0G | 449M | 1.5G | 23% | /boot |
| /dev/sr0 | 9.6G | 9.6G | 0 | 100% | /run/media/jross/Rocky-10-2-x86_64-dvd |

Notes:

- `watch` is a live view like `top`, but it works with any command.
- The DVD at `/dev/sr0` shows 100% because installation media is read-only and full by design.
- Exit with `Ctrl+C`.

![watch df -h output](09-watch-df.jpg)

---

## Key Lessons Learned

- Every running program is a process.
- Processes receive unique PIDs.
- Multiple commands exist for locating processes.
- `top` provides live process monitoring.
- `kill` terminates processes by PID.
- `killall` terminates processes by name.
- Processes may terminate naturally without intervention.
- Placeholders like `<PID>` must be replaced with real values.
- `pidof` matches the process name, so scripts (like firewalld under python3) can be missed.
- Process troubleshooting often requires multiple tools.
- `ss -tulpn` shows listening ports, but `-p` needs `sudo` to reveal the processes behind them.
- `watch` repeats any command at a set interval for live monitoring.

---

## Verification Checklist

- [x] Viewed current processes
- [x] Viewed all system processes
- [x] Displayed process hierarchy
- [x] Used top for monitoring
- [x] Created background process
- [x] Located process with ps
- [x] Located process with pgrep
- [x] Terminated process by PID
- [x] Terminated process by name
- [x] Retrieved process IDs with pidof
- [x] Investigated service processes
- [x] Used ps -ef format
- [x] Listed listening ports with ss
- [x] Monitored disk space live with watch

---

## RHCSA Notes

Useful Process Management Commands:

```bash
ps

ps aux

ps -ef

pstree

top

pgrep

pidof

kill

killall

sudo ss -tulpn

watch
```

Process workflow:

```text
Create Process
↓
Find Process
↓
Get PID
↓
Monitor Process
↓
Terminate Process
↓
Verify Removal
```
