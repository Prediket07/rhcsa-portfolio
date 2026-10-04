# Lab 14 — User Password Policies & Privileged Access

## Objective

Demonstrate Linux password aging policies, password expiration management, forced password changes, default password policy configuration, and privileged access management using sudo.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation

## Concepts Covered

| Command | Purpose |
|----------|----------|
| chage -l | View password aging information |
| chage -M | Set password maximum age |
| chage -d 0 | Force password change at next login |
| useradd | Create a user account |
| passwd | Set user password |
| sudo -l | View sudo permissions |
| visudo | Safely edit sudoers configuration |
| cat /etc/login.defs | View the full default policy file |
| grep PASS /etc/login.defs | View password policy defaults |

---

## Understanding Password Aging

Linux can enforce password policies to improve account security.

Examples:

- Password expiration
- Password aging
- Password change requirements
- Account expiration

Think:

```text
User Account
↓
Password Rules
↓
Security Policy
```

---

## What the Command Names Mean

```text
chage   = CHange AGE (change password aging)
passwd  = password
useradd = add a user
sudo    = superuser do
visudo  = vi + sudoers (edit the sudoers file in vi)
```

---

## View chage Options

Command:

```bash
chage -l
```

Result:

```text
Usage: chage [options] LOGIN
```

Observation:

`chage -l` was run without a username, so `chage` printed its usage and option list instead. The `LOGIN` at the end of the usage line means a username is required.

Options shown:

| Option | Meaning |
|--------|---------|
| -d, --lastday | Set date of last password change |
| -E, --expiredate | Set account expiration date |
| -I, --inactive | Set password inactive after expiration |
| -l, --list | Show account aging information |
| -m, --mindays | Set minimum days before password change |
| -M, --maxdays | Set maximum days before password change |
| -W, --warndays | Set expiration warning days |

![chage option list](Lab14%20-%20User%20Password%20Policies%20and%20Privileged%20Access/01-chage-options.jpg)

---

## Review Current User Password Policy

Command:

```bash
chage -l jross
```

Result:

```text
Last password change                  : never
Password expires                      : never
Password inactive                     : never
Account expires                       : never
Minimum number of days between password change  : 0
Maximum number of days between password change  : 99999
Number of days of warning before password expires : 7
```

Observation:

The primary account did not have password expiration configured.

![chage -l jross, sudo -l, and new user creation](Lab14%20-%20User%20Password%20Policies%20and%20Privileged%20Access/02-user-policy-sudo-new-user.jpg)

---

## Review Sudo Privileges

Command:

```bash
sudo -l
```

Result:

```text
User jross may run the following commands on rhcsa-lab:
    (ALL) ALL
```

Observation:

User `jross` may execute all commands using sudo.

The output also lists "Matching Defaults entries", such as `env_reset` and `secure_path`, which are sudo's default settings for this user.

(Shown in the same screenshot as the previous section.)

---

## Create Test User

Command:

```bash
sudo useradd lab14user
```

Set password:

```bash
sudo passwd lab14user
```

Result:

```text
passwd: password updated successfully
```

Observation:

A dedicated test user was created for password policy experiments. `passwd` asked for the new password twice and did not display what was typed.

(Shown in the same screenshot as the previous sections.)

---

## Review Test User Password Policy

Command:

```bash
sudo chage -l lab14user
```

Result:

```text
Last password change                  : Oct 03, 2026
Password expires                      : never
Maximum number of days between password change  : 99999
```

Observation:

The new user inherited system defaults. The values (minimum 0, maximum 99999, warning 7) match the `PASS_MIN_DAYS`, `PASS_MAX_DAYS`, and `PASS_WARN_AGE` settings in `/etc/login.defs`, shown later in this lab.

The last password change reads Oct 03, 2026 even though the VM clock showed the evening of Oct 2. Password dates are stored as whole days since January 1, 1970, which is the likely reason for the one-day difference.

![chage -l lab14user output](Lab14%20-%20User%20Password%20Policies%20and%20Privileged%20Access/02-user-policy-sudo-new-user.jpg)

---

## Set Password Expiration

Command:

```bash
sudo chage -M 30 lab14user
```

Verify:

```bash
sudo chage -l lab14user
```

Result:

```text
Password expires                      : Nov 02, 2026
Maximum number of days between password change  : 30
```

Observation:

The password was configured to expire after 30 days. The expiration date (Nov 02, 2026) is the last password change date (Oct 03, 2026) plus 30 days.

![chage -M 30 and the updated expiration date](Lab14%20-%20User%20Password%20Policies%20and%20Privileged%20Access/03-chage-max-days.jpg)

---

## Force Password Change

Command:

```bash
sudo chage -d 0 lab14user
```

Verify:

```bash
sudo chage -l lab14user
```

Result:

```text
Last password change                  : password must be changed
Password expires                      : password must be changed
Password inactive                     : password must be changed
Account expires                       : never
Maximum number of days between password change  : 30
```

Observation:

Linux now requires the user to change the password at the next login.

Setting the last change date to day `0` makes the password count as long expired. The maximum of 30 days from the previous step stayed in place.

![chage -d 0 forcing a password change](Lab14%20-%20User%20Password%20Policies%20and%20Privileged%20Access/04-chage-force-change.jpg)

---

## View the Full login.defs File

Command:

```bash
cat /etc/login.defs
```

Observation:

The file is long and mostly made of comments (lines starting with `#`) that document each setting. The header comment states that the parameters control the shadow-utils tools, that these tools do not use the PAM mechanism, and that utilities that do use PAM (such as the `passwd` command) should be configured elsewhere. It points to `/etc/pam.d/system-auth` for more information.

Many entries read `Currently ... is not supported`, meaning those settings are not used on this system.

![Start of cat /etc/login.defs](Lab14%20-%20User%20Password%20Policies%20and%20Privileged%20Access/05-login-defs-top.jpg)

![Middle of cat /etc/login.defs](Lab14%20-%20User%20Password%20Policies%20and%20Privileged%20Access/06-login-defs-middle.jpg)

---

## Review Default Password Policies

Command:

```bash
grep PASS /etc/login.defs
```

Relevant settings:

```text
PASS_MAX_DAYS 99999
PASS_MIN_DAYS 0
PASS_MIN_LEN 8
PASS_WARN_AGE 7
```

Other entries in the output:

```text
PASS_CHANGE_TRIES 5
PASS_ALWAYS_WARN yes
#PASS_MAX_LEN 8
```

Observation:

These values determine default password settings applied to newly created users.

The first four lines of output start with `#`. They are comments that describe each setting. `#PASS_MAX_LEN 8` is also commented out, so it is not active.

![grep PASS /etc/login.defs output](Lab14%20-%20User%20Password%20Policies%20and%20Privileged%20Access/07-login-defs-grep-pass.jpg)

---

## Understanding login.defs

Think:

```text
/etc/login.defs
=
Company-wide password policy
```

Examples:

```text
PASS_MAX_DAYS
```

Maximum password age.

```text
PASS_MIN_DAYS
```

Minimum days before another password change.

```text
PASS_WARN_AGE
```

Days before expiration warnings begin.

```text
PASS_MIN_LEN
```

Minimum password length guideline.

Important note:

```text
The file header says passwd uses PAM, so password rules
for passwd are configured elsewhere (see /etc/pam.d/system-auth).
```

---

## Review Sudo Configuration

Command:

```bash
sudo visudo
```

Observation:

Opened the sudoers configuration safely.

![sudo visudo typed at the prompt](Lab14%20-%20User%20Password%20Policies%20and%20Privileged%20Access/08-visudo-command.jpg)

The editor that opened is `vi` (note the `1,1 Top` position indicator at the bottom right). The status line shows the file as `/etc/sudoers.tmp`, a temporary copy. `visudo` edits the copy and checks it before replacing the real `/etc/sudoers` file.

The header comment in the file reads: `This file must be edited with the 'visudo' command.`

The file also contains commented examples of Host_Alias, User_Alias, and Cmnd_Alias entries (NETWORKING, SOFTWARE, SERVICES, LOCATE, STORAGE).

![visudo editor showing /etc/sudoers.tmp](Lab14%20-%20User%20Password%20Policies%20and%20Privileged%20Access/09-visudo-editor.jpg)

Important note:

```text
visudo validates syntax before saving.
```

Best practice:

```text
Use visudo
Not direct editing of /etc/sudoers
```

---

## Key Lessons Learned

- Password aging can be viewed using `chage`.
- `chage -l` needs a username; without one it prints its usage.
- Password expiration can be enforced on a per-user basis.
- The expiration date is the last change date plus the maximum days.
- Password changes can be forced at next login.
- System-wide defaults are stored in `/etc/login.defs`.
- Users inherit password policy defaults unless overridden.
- `login.defs` does not control `passwd`, which uses PAM.
- Sudo access can be reviewed using `sudo -l`.
- Sudo configuration should be managed using `visudo`.
- `visudo` edits a temporary copy and validates it before saving.

---

## Verification Checklist

- [x] Viewed chage options
- [x] Reviewed current user password rules
- [x] Reviewed sudo privileges
- [x] Created test user
- [x] Configured password expiration
- [x] Forced password change requirement
- [x] Viewed the full login.defs file
- [x] Reviewed password defaults
- [x] Opened sudoers configuration safely
- [x] Verified policy changes

---

## RHCSA Notes

View password policy:

```bash
chage -l username
```

Set password expiration:

```bash
sudo chage -M 30 username
```

Force password change:

```bash
sudo chage -d 0 username
```

View defaults:

```bash
grep PASS /etc/login.defs
```

View sudo rights:

```bash
sudo -l
```

Edit sudoers safely:

```bash
sudo visudo
```

Most important concept:

```text
login.defs
=
Default Policy

chage
=
User-Specific Policy
```
