# Lab 03 — Special Permissions

## Objective

Demonstrate Linux special permissions including SUID, SGID, and Sticky Bit, understand their purpose, and verify their effects using the command line.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab
- Primary User: jross
- Hypervisor: VMware Workstation

## Concepts Covered

| Permission | Numeric Value | Purpose |
|------------|---------------|---------|
| SUID | 4000 | Run a file with the permissions of the file owner |
| SGID | 2000 | New files inherit the directory's group ownership |
| Sticky Bit | 1000 | Users can only delete their own files in a shared directory |

---

## Commands Used

```bash
chmod 4755
chmod 2755
chmod 1777
ls -l
ls -ld
```

---

## Lab Setup

Created a test file and directory:

```bash
mkdir special-permissions-lab
cd special-permissions-lab

touch testfile
mkdir sharedir
```

Verify:

```bash
ls -l
```

Initial output:

```text
drwxr-xr-x sharedir
-rw-r--r-- testfile
```

---

## SUID (Set User ID)

### Apply SUID

```bash
chmod 4755 testfile
```

Verify:

```bash
ls -l testfile
```

Result:

```text
-rwsr-xr-x
```

### Observation

The owner execute bit changed from:

```text
x
```

to:

```text
s
```

Visual pattern:

```text
rwx -> rws
```

### Purpose

SUID allows a program to run with the permissions of the file owner rather than the permissions of the user executing it.

Example:

```bash
/usr/bin/passwd
```

uses SUID so normal users can change their passwords.

---

## SGID (Set Group ID)

### Apply SGID

```bash
chmod 2755 sharedir
```

Verify:

```bash
ls -ld sharedir
```

Result:

```text
drwxr-sr-x
```

### Observation

The group execute bit changed from:

```text
x
```

to:

```text
s
```

Visual pattern:

```text
r-x -> r-s
```

### Purpose

Files and directories created inside an SGID directory inherit the group's ownership.

Common use:

```text
Shared team directories
Project collaboration folders
```

---

## Sticky Bit

### Apply Sticky Bit

```bash
chmod 1777 sharedir
```

Verify:

```bash
ls -ld sharedir
```

Result:

```text
drwxrwxrwt
```

### Observation

The others execute bit changed from:

```text
x
```

to:

```text
t
```

Visual pattern:

```text
rwx -> rwt
```

### Purpose

Users can create files in the directory but cannot delete files owned by other users.

Example:

```bash
/tmp
```

---

## Key Lessons Learned

Special permissions are identified by replacing execute permissions with:

```text
s = SUID
s = SGID
t = Sticky Bit
```

Quick memory aid:

```text
4 = Owner execute becomes s (SUID)

2 = Group execute becomes s (SGID)

1 = Others execute becomes t (Sticky Bit)
```

---

## Verification Checklist

- [x] Created test file
- [x] Created test directory
- [x] Applied SUID
- [x] Applied SGID
- [x] Applied Sticky Bit
- [x] Verified all changes with ls

---

## RHCSA Notes

Common special permission values:

```text
4755 = SUID
2755 = SGID
1777 = Sticky Bit
```

Recognition patterns:

```text
-rwsr-xr-x  -> SUID

drwxr-sr-x -> SGID

drwxrwxrwt -> Sticky Bit
```

Always verify special permissions using:

```bash
ls -l
ls -ld
```
## Screenshots

### Initial Setup

Lab03%20-%20special%20permissions/01.jpg

### SUID Example

Lab03%20-%20special%20permissions/02.jpg

### SGID Example

Lab03%20-%20special%20permissions/03.jpg

### Sticky Bit Example

Lab03%20-%20special%20permissions/04.jpg

### Final Verification

Lab03%20-%20special%20permissions/05.jpg
### Final Verification

Lab03%20-%20special%20permissions/05.jpg
