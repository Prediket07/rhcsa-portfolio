# Lab 04 — Access Control Lists (ACLs)

## Objective

Demonstrate how ACLs provide more granular permissions than standard Linux owner, group, and other permissions by granting specific users access to files without changing ownership or group membership.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab
- Primary User: jross
- Hypervisor: VMware Workstation

## Concepts Covered

| Command | Purpose |
|----------|----------|
| getfacl | Display ACL entries on a file or directory |
| setfacl -m | Add or modify an ACL entry |
| setfacl -b | Remove all ACL entries |
| ls -l | Verify ACL presence using the + indicator |

---

## Lab Setup

Created a practice directory and file:

```bash
mkdir acl-lab
cd acl-lab

touch project.txt
```

Verify:

```bash
ls -l
```

Output:

```text
-rw-r--r-- project.txt
```

---

## Viewing ACLs

Display current ACL entries:

```bash
getfacl project.txt
```

Output:

```text
user::rw-
group::r--
other::r--
```

Observation:

ACLs initially reflect the standard Linux permissions shown by `ls -l`.

---

## Verify User Exists

```bash
id labuser
```

Output confirmed:

```text
uid=1006(labuser)
```

---

## Grant ACL Permissions

Grant full access to a specific user:

```bash
setfacl -m u:labuser:rwx project.txt
```

Verify:

```bash
getfacl project.txt
```

Output:

```text
user::rw-
user:labuser:rwx
group::r--
mask::rwx
other::r--
```

### Breakdown

```text
u = user

labuser = target user

rwx = permissions granted
```

Observation:

A special ACL entry was created:

```text
user:labuser:rwx
```

This grants permissions to labuser without changing ownership or group membership.

---

## ACL Mask

ACL output included:

```text
mask::rwx
```

Purpose:

The ACL mask defines the maximum permissions that ACL entries may use.

Think of it as a permissions ceiling for ACL assignments.

---

## ACL Indicator

Running:

```bash
ls -l project.txt
```

showed:

```text
-rw-r--r--+
```

Observation:

```text
+
```

indicates that extended ACL permissions exist on the file.

---

## Remove All ACL Entries

Remove ACL entries:

```bash
setfacl -b project.txt
```

Verify:

```bash
getfacl project.txt
```

Output returned to:

```text
user::rw-
group::r--
other::r--
```

Observation:

The special ACL entry for labuser was removed.

---

## Why ACLs Exist

Standard Linux permissions only support:

```text
Owner
Group
Others
```

ACLs allow administrators to create exceptions.

Example:

```text
Owner: jross
Group: labteam
```

Granting access to a single user:

```bash
setfacl -m u:labuser:rwx project.txt
```

does not require:

- Changing the owner
- Changing the group
- Granting permissions to everyone

This makes ACLs useful for temporary access, project collaboration, and file sharing.

---

## Key Lessons Learned

- ACLs provide more granular permissions than standard Linux permissions.
- ACLs can grant permissions to specific users without changing ownership.
- The `+` symbol in `ls -l` indicates ACLs are present.
- `getfacl` displays ACL entries.
- `setfacl -m` adds or modifies ACLs.
- `setfacl -b` removes all ACL entries.

---

## Verification Checklist

- [x] Created ACL practice environment
- [x] Viewed existing ACLs with getfacl
- [x] Verified user account
- [x] Added ACL permissions for a specific user
- [x] Verified ACL entry creation
- [x] Observed ACL mask
- [x] Identified ACL indicator (+)
- [x] Removed ACL entries
- [x] Verified ACL removal

---

## RHCSA Notes

ACL commands:

```bash
getfacl <file>

setfacl -m u:user:rwx <file>

setfacl -b <file>
```

Remember:

```text
Owner
Group
Others
```

ACLs provide:

```text
Specific User Permissions
```

without modifying ownership or group membership.
