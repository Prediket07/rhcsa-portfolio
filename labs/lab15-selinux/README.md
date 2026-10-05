# Lab 15 — SELinux

## Objective

Demonstrate basic SELinux administration by viewing SELinux status, examining security contexts, switching between enforcing and permissive modes, reviewing SELinux booleans, and validating file contexts.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation

## Concepts Covered

| Command | Purpose |
|----------|----------|
| getenforce | Display SELinux operating mode |
| sestatus | View SELinux configuration and status |
| ls -Z | Display SELinux security contexts |
| ls -Zd | Display SELinux context for a directory |
| setenforce | Change SELinux mode temporarily |
| getsebool | View SELinux booleans |
| restorecon | Restore default SELinux context |

---

## Understanding SELinux

SELinux adds an additional layer of security beyond standard Linux permissions.

Traditional permissions:

```text
Owner
Group
Others
```

SELinux adds:

```text
Security Contexts
```

Think:

```text
Linux Permissions
=
Door Lock
```

```text
SELinux
=
Security Guard Behind The Door
```

Access is determined by:

```text
Permissions
+
SELinux Context
=
Access Allowed
```

---

## What the Command Names Mean

```text
getenforce = GET the ENFORCE mode
setenforce = SET the ENFORCE mode
sestatus   = SElinux STATUS
getsebool  = GET SELinux BOOLean
restorecon = RESTORE CONtext
ls -Z      = list files with the SELinux context (Z = context)
```

---

## View SELinux Mode

Command:

```bash
getenforce
```

Output:

```text
Enforcing
```

Observation:

SELinux was active and enforcing security policy.

![getenforce output](Lab15%20-%20SELinux/01-getenforce.jpg)

---

## View SELinux Status

Command:

```bash
sestatus
```

Observation:

Key settings included:

```text
SELinux status:        enabled
Loaded policy name:    targeted
Current mode:          enforcing
Mode from config file: enforcing
```

Other lines in the output:

```text
SELinuxfs mount:              /sys/fs/selinux
SELinux root directory:       /etc/selinux
Policy MLS status:            enabled
Policy deny_unknown status:   allowed
Memory protection checking:   actual (secure)
Max kernel policy version:    33
```

Explanation:

```text
enabled
=
SELinux running

enforcing
=
actively enforcing policy

targeted
=
default Rocky Linux policy
```

`Current mode` is the mode right now. `Mode from config file` is the mode SELinux will start in at boot. Both read `enforcing` here. `setenforce` changes only the current mode.

![sestatus output](Lab15%20-%20SELinux/02-sestatus.jpg)

---

## View SELinux Contexts

Command:

```bash
ls -Z
```

Observation:

Files and directories displayed contexts such as:

```text
unconfined_u:object_r:user_home_t:s0
```

Examples:

```text
03
lab07data
labs
README.md
screenshots
```

All were labeled:

```text
user_home_t
```

indicating normal user-home content.

A context has four parts:

```text
unconfined_u : object_r : user_home_t : s0
user           role       type           level
```

The type (`user_home_t`) is the part the targeted policy uses most to decide what is allowed.

![ls -Z output](Lab15%20-%20SELinux/03-ls-z-contexts.jpg)

---

## View Directory Context

Command:

```bash
ls -Zd .
```

Output:

```text
unconfined_u:object_r:user_home_t:s0 .
```

Observation:

The current working directory had the expected home-directory context.

`ls -Zd` with no path (run first) gave the same result, because `ls -d` shows the current directory (`.`) when no path is given.

![ls -Zd output](Lab15%20-%20SELinux/04-ls-zd-directory.jpg)

---

## Switch SELinux to Permissive Mode

Command:

```bash
sudo setenforce 0
```

Verify:

```bash
getenforce
```

Output:

```text
Permissive
```

Observation:

SELinux continued monitoring activity but stopped enforcing policy.

![setenforce 0 and getenforce showing Permissive](Lab15%20-%20SELinux/05-setenforce-permissive.jpg)

---

## Return to Enforcing Mode

Command:

```bash
sudo setenforce 1
```

Verify:

```bash
getenforce
```

Output:

```text
Enforcing
```

Observation:

SELinux resumed active policy enforcement.

Important note:

```text
setenforce changes are temporary
and do not survive reboot.
```

![setenforce 1 and getenforce showing Enforcing](Lab15%20-%20SELinux/06-setenforce-enforcing.jpg)

---

## View SELinux Booleans

Command:

```bash
getsebool -a | head
```

Sample Output:

```text
abrt_anon_write --> off
abrt_handle_event --> on
abrt_upload_watch_anon_write --> on
auditadm_exec_content --> on
authlogin_nsswitch_use_ldap --> off
authlogin_radius --> off
authlogin_yubikey --> off
cdrecord_read_content --> off
cluster_can_network_connect --> off
cluster_manage_all_files --> off
```

Observation:

SELinux booleans enable or disable specific security behaviors.

`getsebool -a` lists every boolean, and `head` limits the output to the first 10 lines.

Think:

```text
SELinux Feature Switches
```

![getsebool -a output](Lab15%20-%20SELinux/07-getsebool.jpg)

---

## Create Test File

Command:

```bash
touch selinux-test.txt
```

Verify Context:

```bash
ls -Z selinux-test.txt
```

Output:

```text
unconfined_u:object_r:user_home_t:s0 selinux-test.txt
```

Observation:

The file inherited the appropriate context for a user home directory.

![selinux-test.txt context](Lab15%20-%20SELinux/08-test-file-context.jpg)

---

## Validate Context with restorecon

Command:

```bash
restorecon -v selinux-test.txt
```

Observation:

No changes were required because the file already had the correct label.

Purpose:

```text
restorecon restores default SELinux contexts.
```

This is a common troubleshooting tool when labels become incorrect.

---

## Understanding SELinux Modes

### Enforcing

```text
Policy enforced
Access denied when policy is violated
```

### Permissive

```text
Policy violations logged
Access still allowed
```

### Disabled

```text
SELinux inactive
```

---

## Key Lessons Learned

- SELinux provides an additional security layer beyond Linux permissions.
- Security contexts control how files and directories are used.
- A context has four parts: user, role, type, and level.
- `getenforce` quickly displays the active SELinux mode.
- `sestatus` provides detailed SELinux information, including the mode from the config file.
- `setenforce` switches between enforcing and permissive modes, but only until reboot.
- SELinux booleans act as feature switches.
- Files inherit context labels from their location when created.
- `restorecon` restores a file to its default context.

---

## Verification Checklist

- [x] Viewed SELinux mode
- [x] Viewed SELinux status
- [x] Viewed file contexts
- [x] Viewed directory context
- [x] Switched to permissive mode
- [x] Returned to enforcing mode
- [x] Viewed SELinux booleans
- [x] Created test file and verified its context
- [x] Validated context with restorecon

---

## RHCSA Notes

View mode:

```bash
getenforce
```

View status:

```bash
sestatus
```

View contexts:

```bash
ls -Z
ls -Zd .
```

Change mode temporarily:

```bash
sudo setenforce 0
sudo setenforce 1
```

View booleans:

```bash
getsebool -a
```

Restore default context:

```bash
restorecon -v filename
```

Most important concept:

```text
Permissions
+
SELinux Context
=
Access Allowed

setenforce
=
Temporary change
```
