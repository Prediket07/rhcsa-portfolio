# Lab 26 — TuneD Profiles

## Objective

Practice managing system performance profiles with TuneD. This lab covers checking the TuneD service, identifying the active and recommended profiles, changing profiles, verifying the configuration, and reading the TuneD log.

## Environment

- Rocky Linux
- VMware Workstation
- systemd
- TuneD
- Screenshots: [Lab26 - TuneD Profiles](Lab26%20-%20TuneD%20Profiles/)

## Concepts Covered

| Command | Purpose |
|----------|----------|
| systemctl status tuned | Check the TuneD service |
| systemctl is-active tuned | Check that the service is running |
| tuned-adm active | Show the active profile |
| tuned-adm recommend | Show the profile TuneD recommends |
| tuned-adm list | List available profiles |
| tuned-adm profile PROFILE | Change the active profile |
| tuned-adm verify | Compare system settings to the active profile |
| cat /etc/tuned/active_profile | Show the saved profile |
| systemctl restart tuned | Restart the TuneD daemon |

---

## Understanding TuneD

TuneD is a service that tunes system settings for a type of workload. Each profile is a ready-made set of settings.

Think:

```text
Profile = Performance Preset
```

Examples:

```text
balanced
virtual-guest
throughput-performance
powersave
```

---

## 1. Check the TuneD Service

Command:

```bash
systemctl status tuned
```

![TuneD Service Status and Profile List](Lab26%20-%20TuneD%20Profiles/01-service-status-and-profile-list.jpg)

Observation:

```text
Loaded: enabled
Active: active (running)
```

TuneD was installed, enabled at boot, and running.

---

## 2. Check the Active and Recommended Profiles

Commands:

```bash
tuned-adm active
tuned-adm recommend
```

The output is in the same screenshot as the service status above.

Output:

```text
Current active profile: balanced
virtual-guest
```

Observation:

The system was using `balanced`, but TuneD recommended `virtual-guest` because this machine is a virtual guest. The recommended profile is not always the active profile.

---

## 3. List Available Profiles

Command:

```bash
tuned-adm list
```

The output is in the same screenshot as the service status above.

Observation:

TuneD listed 15 profiles, including `balanced`, `virtual-guest`, `virtual-host`, `throughput-performance`, `latency-performance`, `network-latency`, `network-throughput`, `powersave`, and `desktop`. The list ends with the current active profile, `balanced`.

---

## 4. Change the Profile

Command:

```bash
sudo tuned-adm profile virtual-guest
```

Verify:

```bash
tuned-adm active
```

![Set virtual-guest and Verify](Lab26%20-%20TuneD%20Profiles/02-set-virtual-guest-and-verify.jpg)

Output:

```text
Current active profile: virtual-guest
```

Observation:

The active profile changed from `balanced` to the recommended `virtual-guest`.

---

## 5. Verify the Configuration

Command:

```bash
sudo tuned-adm verify
```

The output is in the same screenshot as the profile change above.

Output:

```text
Verification failed, current system settings differ from the preset profile.
You can mostly fix this by restarting the TuneD daemon, e.g.:
  systemctl restart tuned
```

Observation:

`tuned-adm verify` compares the running system with what the profile asks for. It reported a difference, so the next steps look at why.

---

## 6. Restart TuneD and Re-apply the Profile

Commands:

```bash
sudo systemctl restart tuned
systemctl is-active tuned
tuned-adm active
cat /etc/tuned/active_profile
sudo tuned-adm profile virtual-guest
cat /etc/tuned/active_profile
tuned-adm active
sudo tuned-adm verify
```

![Restart TuneD and Re-apply the Profile](Lab26%20-%20TuneD%20Profiles/03-restart-tuned-and-reapply-profile.jpg)

Output included:

```text
active
Current active profile: balanced
balanced
virtual-guest
Current active profile: virtual-guest
Verification failed, current system settings differ from the preset profile.
```

Observation:

- After the restart, TuneD was `active` and the saved profile in `/etc/tuned/active_profile` read `balanced`.
- After re-applying the profile, both `/etc/tuned/active_profile` and `tuned-adm active` showed `virtual-guest`. The saved file is what keeps the choice across reboots.
- Verification still failed after the restart, so the restart did not clear the difference.

---

## 7. Read the TuneD Log

Command:

```bash
sudo tail -n 30 /var/log/tuned/tuned.log
```

![TuneD Log Verify Details](Lab26%20-%20TuneD%20Profiles/04-tuned-log-verify-details.jpg)

Output included:

```text
tuned.plugins.plugin_cpu: Unable to set boost on cpu 'cpu1'. Neither per-policy boost nor intel_pstate no_turbo is available.
tuned.daemon.daemon: static tuning from profile 'virtual-guest' applied
tuned.plugins.base: verify: failed: device cpu1: 'boost' = 'None', expected '1'
tuned.plugins.base: verify: failed: device cpu0: 'boost' = 'None', expected '1'
tuned.plugins.base: verify: passed: 'vm.swappiness' = '30'
```

Observation:

- The profile itself was applied: the log shows `static tuning from profile 'virtual-guest' applied`, and the `vm.swappiness` check passed with the value `30`.
- The only failures were the CPU `boost` setting. The virtual CPUs in this VM do not expose a boost control, so TuneD cannot set it and the check reports `None`.
- This is a limit of the virtual machine, not a mistake in the configuration. On hardware that exposes CPU boost, the same check would be expected to pass.
- `tuned-adm verify` is useful, but the log shows exactly which setting failed and why.

---

## 8. Return to the Original Profile

Commands:

```bash
sudo tuned-adm profile balanced
tuned-adm active
tuned-adm recommend
```

![Restore the balanced Profile](Lab26%20-%20TuneD%20Profiles/05-restore-balanced-profile.jpg)

Output:

```text
Current active profile: balanced
virtual-guest
```

Observation:

The lab environment was returned to its original `balanced` profile. `tuned-adm recommend` still suggested `virtual-guest`, which confirms that the recommendation does not change when the active profile does.

---

## Key Takeaways

- TuneD manages system performance using predefined profiles.
- `tuned-adm active` shows the currently active profile.
- `tuned-adm recommend` shows the profile TuneD recommends for the system.
- `tuned-adm list` displays the available profiles.
- `tuned-adm profile PROFILE` changes the active profile.
- `/etc/tuned/active_profile` stores the selected profile.
- `tuned-adm verify` checks the system settings against the active profile.
- A failed verify does not always mean a bad configuration. The log in `/var/log/tuned/tuned.log` shows which setting failed.
- Some settings, such as CPU boost, are not available inside a virtual machine.
- The recommended profile is not necessarily the active profile.
- Changes should always be verified after modifying the system configuration.

---

## Verification Checklist

- [x] Checked the TuneD service
- [x] Viewed the active profile
- [x] Viewed the recommended profile
- [x] Listed available profiles
- [x] Changed the profile to `virtual-guest`
- [x] Verified the active profile
- [x] Checked the saved profile in `/etc/tuned/active_profile`
- [x] Ran `tuned-adm verify`
- [x] Restarted TuneD
- [x] Read the TuneD log to understand the verify result
- [x] Restored the `balanced` profile

---

## Screenshots

All screenshots for this lab: [Lab26 - TuneD Profiles](Lab26%20-%20TuneD%20Profiles/)

| # | File | Shows |
|---|------|-------|
| 01 | `01-service-status-and-profile-list.jpg` | `systemctl status tuned`, `tuned-adm active`, `recommend`, and `list` |
| 02 | `02-set-virtual-guest-and-verify.jpg` | `tuned-adm profile virtual-guest`, `active`, and `verify` |
| 03 | `03-restart-tuned-and-reapply-profile.jpg` | Restart, `is-active`, `active_profile` file, re-apply, and `verify` |
| 04 | `04-tuned-log-verify-details.jpg` | `tail -n 30 /var/log/tuned/tuned.log` with the verify results |
| 05 | `05-restore-balanced-profile.jpg` | `tuned-adm profile balanced`, `active`, and `recommend` |

---

## RHCSA Notes

Check the service:

```bash
systemctl status tuned
```

Enable and start TuneD:

```bash
sudo systemctl enable --now tuned
```

Show the active profile:

```bash
tuned-adm active
```

Show the recommended profile:

```bash
tuned-adm recommend
```

List profiles:

```bash
tuned-adm list
```

Change profile:

```bash
sudo tuned-adm profile virtual-guest
```

Verify settings:

```bash
sudo tuned-adm verify
```

Check the TuneD log:

```bash
sudo tail -n 30 /var/log/tuned/tuned.log
```

Most Important Concept:

```text
Profile = Preset of System Tuning
tuned-adm active    = what is in use
tuned-adm recommend = what TuneD suggests
tuned-adm profile   = change it
```
