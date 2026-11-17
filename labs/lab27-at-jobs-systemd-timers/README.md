# Lab 27 — at Jobs and systemd Timers

## Objective

Practice scheduling one-time jobs with `at` and creating custom systemd timers for automated tasks.

This lab covers scheduling, inspecting, canceling, and verifying `at` jobs as well as creating a custom `.service` and `.timer` unit.

## Environment

- Rocky Linux
- VMware Workstation
- systemd
- at / atd
- Screenshots: [Lab27 - at Jobs and systemd Timers](Lab27%20-%20at%20Jobs%20and%20systemd%20Timers/)

## Concepts Covered

| Command | Purpose |
|----------|----------|
| which at | Confirm `at` is installed |
| systemctl status atd | Check the `at` scheduler service |
| at TIME | Schedule a one-time job |
| atq | List pending `at` jobs |
| atrm ID | Remove a pending `at` job |
| systemctl list-timers | List systemd timers |
| systemctl cat UNIT | Show a unit file |
| systemctl daemon-reload | Reread unit files |
| systemctl enable --now | Enable at boot and start immediately |
| systemctl is-active / is-enabled | Check runtime state and boot persistence |

---

## Part 1 — at Jobs

### Verify at Is Installed

```bash
which at
```

Result:

```text
/usr/bin/at
```

![Verify at, Check atd, and Schedule a Job](Lab27%20-%20at%20Jobs%20and%20systemd%20Timers/01-verify-at-and-schedule-job.jpg)

### Verify the atd Service

```bash
systemctl status atd
```

The output is in the same screenshot as the check above.

Confirmed that `atd` was active and enabled.

```text
Loaded: enabled; preset: enabled
Active: active (running)
```

Observation:

The status output also shows an informational line about an unset `OPTS` variable. It is harmless and does not stop `atd` from running.

### Schedule a One-Time Job

The VM clock was checked first:

```bash
date
```

Then the job was scheduled:

```bash
at 7:35 PM
```

At the `at>` prompt:

```bash
echo "Lab 27 at job worked" > /tmp/at-lab-test.txt
```

Press:

```text
Ctrl+D
```

to submit the job.

The output is in the same screenshot as the check above.

Output:

```text
warning: commands will be executed using /bin/sh
job 1 at Mon Nov 16 19:35:00 2026
```

Observation:

The scheduled job was assigned job ID `1`. `at` runs the commands with `/bin/sh`, and `Ctrl+D` shows as `<EOT>` when the job is submitted.

### Inspect Pending at Jobs

```bash
atq
```

`atq` displays jobs that are waiting to execute.

After the scheduled job ran, the queue was empty.

### Verify the Job Executed

```bash
cat /tmp/at-lab-test.txt
```

Result:

```text
Lab 27 at job worked
```

### Schedule and Cancel an at Job

A second job was scheduled:

```bash
at 7:50 PM
```

At the `at>` prompt:

```bash
echo "This job should never run" > /tmp/cancel-test.txt
```

After pressing `Ctrl+D`, the job was confirmed with:

```bash
atq
```

Output:

```text
2       Mon Nov 16 19:50:00 2026 a jross
```

The job was then removed using its job ID:

```bash
atrm 2
```

Verification:

```bash
atq
```

![Schedule and Cancel an at Job](Lab27%20-%20at%20Jobs%20and%20systemd%20Timers/02-schedule-and-cancel-at-job.jpg)

The empty queue confirmed that the job had been removed.

---

## Part 2 — systemd Timers

### Inspect Existing Timers

```bash
systemctl list-timers
```

![Existing Timers and fstrim.timer](Lab27%20-%20at%20Jobs%20and%20systemd%20Timers/03-list-timers-and-fstrim-timer.jpg)

Observation:

This displayed 7 existing systemd timers and the services they activate, including `dnf-makecache.timer`, `logrotate.timer`, `fstrim.timer`, and `systemd-tmpfiles-clean.timer`.

A timer represents **WHEN** a task runs, while its associated service represents **WHAT** the system executes.

### Inspect an Existing Timer

```bash
systemctl cat fstrim.timer
```

The output is in the same screenshot as the timer list above, and again in the next one.

Important timer configuration observed:

```ini
[Timer]
OnCalendar=weekly
AccuracySec=1h
Persistent=true
RandomizedDelaySec=100min
```

Observation:

- `OnCalendar=weekly` runs the timer once a week.
- `Persistent=true` makes systemd run a missed job after the system was off.
- `RandomizedDelaySec=100min` spreads the start time so many systems do not all run at once.

### Inspect the Associated Service

```bash
systemctl cat fstrim.service
```

![fstrim.timer and fstrim.service](Lab27%20-%20at%20Jobs%20and%20systemd%20Timers/04-fstrim-timer-and-service.jpg)

Important service configuration observed:

```ini
[Service]
Type=oneshot
ExecStart=/usr/sbin/fstrim --listed-in /etc/fstab:/proc/self/mountinfo --verbose --quiet-unsupported
```

This demonstrated the relationship:

```text
.timer
WHEN
  ↓
.service
WHAT
```

---

## Create a Custom systemd Service

Created:

```text
/etc/systemd/system/lab27-test.service
```

Contents:

```ini
[Unit]
Description=Lab 27 systemd timer test

[Service]
Type=oneshot
ExecStart=/usr/bin/touch /tmp/lab27-timer-worked.txt
```

The service uses `ExecStart=` to define the command systemd executes. `Type=oneshot` fits a command that runs once and exits.

## Create a Custom systemd Timer

Created:

```text
/etc/systemd/system/lab27-test.timer
```

Configuration:

```ini
[Unit]
Description=Lab 27 systemd timer

[Timer]
OnBootSec=2min
Unit=lab27-test.service

[Install]
WantedBy=timers.target
```

`OnBootSec=2min` schedules the service relative to system boot.

## Reload systemd

After creating or modifying unit files:

```bash
sudo systemctl daemon-reload
```

This causes systemd to reread its unit configuration.

Check the new timer:

```bash
systemctl status lab27-test.timer
```

![Timer Status Before Enabling](Lab27%20-%20at%20Jobs%20and%20systemd%20Timers/05-timer-status-before-enable.jpg)

Output included:

```text
Loaded: loaded (/etc/systemd/system/lab27-test.timer; disabled; preset: disabled)
Active: inactive (dead)
Triggers: lab27-test.service
```

Observation:

systemd found the new unit after the reload. The timer was loaded but disabled and inactive, and it points to `lab27-test.service`.

## Enable and Start the Timer

```bash
sudo systemctl enable --now lab27-test.timer
```

![Enable and Start the Timer](Lab27%20-%20at%20Jobs%20and%20systemd%20Timers/06-enable-timer-now.jpg)

Output:

```text
Created symlink '/etc/systemd/system/timers.target.wants/lab27-test.timer' → '/etc/systemd/system/lab27-test.timer'.
```

Verify runtime state:

```bash
systemctl is-active lab27-test.timer
```

Result:

```text
active
```

Verify boot persistence:

```bash
systemctl is-enabled lab27-test.timer
```

![Timer Active and Enabled](Lab27%20-%20at%20Jobs%20and%20systemd%20Timers/07-timer-active-and-enabled.jpg)

Result:

```text
enabled
```

## Verify Timer Execution

```bash
systemctl list-timers lab27-test.timer
ls -l /tmp/lab27-timer-worked.txt
```

![Verify Timer Execution](Lab27%20-%20at%20Jobs%20and%20systemd%20Timers/08-verify-timer-execution.jpg)

Output included:

```text
NEXT  LEFT  LAST                          PASSED        UNIT              ACTIVATES
-     -     Mon 2026-11-16 19:58:19 CST   3min 20s ago  lab27-test.timer  lab27-test.service

-rw-r--r--. 1 root root 0 Nov 16 19:58 /tmp/lab27-timer-worked.txt
```

Observation:

- The timer activated `lab27-test.service`, and the service created `/tmp/lab27-timer-worked.txt`.
- The file is owned by root because system services run as root.
- The timer ran at 19:58:19, the same moment it was enabled. `OnBootSec=2min` counts from system boot, and the VM had been running far longer than 2 minutes, so the time had already passed and the timer fired right away.
- `NEXT` and `LEFT` show `-` because `OnBootSec` fires once per boot and does not repeat.

---

## Key Takeaways

- `at` schedules a job to run once at a given time.
- `atd` must be running for `at` jobs to execute.
- `atq` lists pending jobs, and `atrm ID` removes one.
- A systemd timer defines **WHEN**, and its service defines **WHAT**.
- `systemctl cat UNIT` shows the full unit file.
- Run `sudo systemctl daemon-reload` after creating or changing unit files.
- `Type=oneshot` suits a command that runs once and exits.
- `OnCalendar=` schedules by calendar time, and `OnBootSec=` schedules relative to boot.
- A timer based on `OnBootSec` fires immediately if that time has already passed when it starts.
- `systemctl enable --now` enables a unit at boot and starts it immediately.
- `systemctl list-timers` shows when each timer last ran and when it runs next.

---

## Verification Checklist

- [x] Verified `at` is installed
- [x] Verified `atd` is active and enabled
- [x] Scheduled a one-time `at` job
- [x] Verified the job executed
- [x] Scheduled a second `at` job
- [x] Canceled the job with `atrm`
- [x] Inspected existing systemd timers
- [x] Inspected `fstrim.timer` and `fstrim.service`
- [x] Created a custom service
- [x] Created a custom timer
- [x] Reloaded systemd
- [x] Enabled and started the timer
- [x] Verified the timer was active and enabled
- [x] Verified the timer ran the service

---

## Screenshots

All screenshots for this lab: [Lab27 - at Jobs and systemd Timers](Lab27%20-%20at%20Jobs%20and%20systemd%20Timers/)

| # | File | Shows |
|---|------|-------|
| 01 | `01-verify-at-and-schedule-job.jpg` | `which at`, `systemctl status atd`, `date`, and `at 7:35 PM` |
| 02 | `02-schedule-and-cancel-at-job.jpg` | `at 7:50 PM`, `atq`, `atrm 2`, and an empty `atq` |
| 03 | `03-list-timers-and-fstrim-timer.jpg` | `systemctl list-timers` and `systemctl cat fstrim.timer` |
| 04 | `04-fstrim-timer-and-service.jpg` | `systemctl cat fstrim.timer` and `systemctl cat fstrim.service` |
| 05 | `05-timer-status-before-enable.jpg` | `daemon-reload` and `systemctl status lab27-test.timer` (disabled, inactive) |
| 06 | `06-enable-timer-now.jpg` | `systemctl enable --now lab27-test.timer` and the symlink created |
| 07 | `07-timer-active-and-enabled.jpg` | `is-active` (active) and `is-enabled` (enabled) |
| 08 | `08-verify-timer-execution.jpg` | `list-timers lab27-test.timer` and `ls -l /tmp/lab27-timer-worked.txt` |

---

## RHCSA Notes

Schedule a one-time job:

```bash
at 7:35 PM
```

List and remove jobs:

```bash
atq
atrm 2
```

List timers:

```bash
systemctl list-timers
```

Show a unit file:

```bash
systemctl cat fstrim.timer
```

Minimal service (`/etc/systemd/system/NAME.service`):

```ini
[Unit]
Description=Example service

[Service]
Type=oneshot
ExecStart=/usr/bin/touch /tmp/example.txt
```

Minimal timer (`/etc/systemd/system/NAME.timer`):

```ini
[Unit]
Description=Example timer

[Timer]
OnBootSec=2min
Unit=NAME.service

[Install]
WantedBy=timers.target
```

Reload, enable, and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now NAME.timer
```

Most Important Concept:

```text
.timer   = WHEN it runs
.service = WHAT it runs
at       = run once, later
timer    = run on a schedule, managed by systemd
```
