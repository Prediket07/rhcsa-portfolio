# Lab 18 — README screenshot snippets

Paste each block into the README under the heading named above it, after that section's
command/output and before its "Observation" text (same pattern as Labs 07 and 08).
Image paths assume the README sits next to the `Lab18 - Boot Process and Recovery` folder,
the same way the Lab 07 README sits next to its folder. If the README lives somewhere else,
only the folder prefix in each path needs to change.

---

## Environment (add one line right under the Environment list)

```markdown
- Screenshots: [Lab18 - Boot Process and Recovery](Lab18%20-%20Boot%20Process%20and%20Recovery/)
```

---

## Default Boot Target

```markdown
![Default Boot Target](Lab18%20-%20Boot%20Process%20and%20Recovery/01-default-boot-target.jpg)
```

## Failed Services

```markdown
![Failed Services](Lab18%20-%20Boot%20Process%20and%20Recovery/02-failed-services.jpg)
```

## Investigating the Failed Unit

```markdown
![Investigate Failed Unit](Lab18%20-%20Boot%20Process%20and%20Recovery/04-investigate-failed-unit.jpg)
```

## Boot Logs

```markdown
![Boot Logs](Lab18%20-%20Boot%20Process%20and%20Recovery/03-boot-logs.jpg)
```

## Systemd Targets

```markdown
![Active Systemd Targets](Lab18%20-%20Boot%20Process%20and%20Recovery/05-list-active-targets.jpg)
```

## Multi-User Target

Three screenshots, in this order: the isolate command prompting for authentication, the
text-mode login prompt, and the console session confirming the default target is unchanged.

```markdown
![Isolate Multi-User Target - Authentication](Lab18%20-%20Boot%20Process%20and%20Recovery/06-isolate-multi-user-auth.jpg)

![Multi-User Login Prompt](Lab18%20-%20Boot%20Process%20and%20Recovery/07-multi-user-login-prompt.jpg)

![Multi-User Console Session](Lab18%20-%20Boot%20Process%20and%20Recovery/08-multi-user-console-session.jpg)
```

## Rescue Target

This one screenshot shows both `systemctl cat rescue.target` and `systemctl cat emergency.target`.

```markdown
![Rescue and Emergency Target Units](Lab18%20-%20Boot%20Process%20and%20Recovery/09-rescue-emergency-target-units.jpg)
```

## Emergency Target (optional pointer, since the screenshot above covers it)

```markdown
See the screenshot in the Rescue Target section; it also shows `emergency.target`.
```

## Permanent Boot Target Configuration

```markdown
![Default Target Symlink](Lab18%20-%20Boot%20Process%20and%20Recovery/10-default-target-symlink.jpg)
```

---

## Screenshots section (replace the list at the bottom)

```markdown
## Screenshots

All screenshots for this lab: [Lab18 - Boot Process and Recovery](Lab18%20-%20Boot%20Process%20and%20Recovery/)

| # | File | Shows |
|---|------|-------|
| 01 | `01-default-boot-target.jpg` | `systemctl get-default` |
| 02 | `02-failed-services.jpg` | `systemctl --failed` |
| 03 | `03-boot-logs.jpg` | `journalctl -b \| head -20` |
| 04 | `04-investigate-failed-unit.jpg` | `systemctl status` on the failed vmblock-fuse mount |
| 05 | `05-list-active-targets.jpg` | `systemctl list-units --type=target` |
| 06 | `06-isolate-multi-user-auth.jpg` | `systemctl isolate multi-user.target` asking for authentication |
| 07 | `07-multi-user-login-prompt.jpg` | Text-mode login prompt after switching targets |
| 08 | `08-multi-user-console-session.jpg` | Console login, `whoami`, `systemctl get-default` |
| 09 | `09-rescue-emergency-target-units.jpg` | `systemctl cat rescue.target` and `emergency.target` |
| 10 | `10-default-target-symlink.jpg` | `ls -l /etc/systemd/system/default.target` |
```
