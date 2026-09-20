# Lab 02 — File Permissions Management

## Objective

Demonstrate Linux file permission management using both symbolic and numeric modes, modify ownership and group ownership, and verify changes from the command line.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab
- Primary User: jross
- Hypervisor: VMware Workstation

## Concepts Covered

| Command | Purpose |
|----------|----------|
| chmod | Change file permissions |
| chown | Change file ownership |
| chgrp | Change file group ownership |
| ls -l | Display detailed permissions and ownership |

---

## Walkthrough

### Create Practice Files

```bash
mkdir permission-lab
cd permission-lab

touch file1
touch file2
touch script.sh
```

Verify:

```bash
ls -l
```

Initial permissions:

```text
-rw-r--r-- file1
-rw-r--r-- file2
-rw-r--r-- script.sh
```

---

## Symbolic Permissions

### Add Execute Permission for Owner

```bash
chmod u+x script.sh
```

Before:

```text
-rw-r--r--
```

After:

```text
-rwxr--r--
```

Lesson:

- `u` = user (owner)
- `+` = add
- `x` = execute

---

### Remove Execute Permission for Owner

```bash
chmod u-x script.sh
```

Before:

```text
-rwxr--r--
```

After:

```text
-rw-r--r--
```

---

### Add Write Permission for Group

```bash
chmod g+w file1
```

Before:

```text
-rw-r--r--
```

After:

```text
-rw-rw-r--
```

---

### Remove Write Permission for Group

```bash
chmod g-w file1
```

Before:

```text
-rw-rw-r--
```

After:

```text
-rw-r--r--
```

---

### Remove Read Permission for Others

```bash
chmod o-r file2
```

Before:

```text
-rw-r--r--
```

After:

```text
-rw-r-----
```

---

### Add Read Permission for All

```bash
chmod a+r file2
```

Before:

```text
-rw-r-----
```

After:

```text
-rw-r--r--
```

---

## Numeric Permissions

### chmod 755

```bash
chmod 755 script.sh
```

Result:

```text
-rwxr-xr-x
```

Breakdown:

```text
7 = rwx
5 = r-x
5 = r-x
```

Common use:

- Executable files and scripts

---

### chmod 600

```bash
chmod 600 file2
```

Result:

```text
-rw-------
```

Breakdown:

```text
6 = rw-
0 = ---
0 = ---
```

Common use:

- Private files

---

### chmod 644

```bash
chmod 644 file2
```

Result:

```text
-rw-r--r--
```

Breakdown:

```text
6 = rw-
4 = r--
4 = r--
```

Common use:

- Standard text and configuration files

---

## Change Group Ownership

```bash
sudo chgrp labteam file1
```

Verify:

```bash
ls -l
```

Before:

```text
jross jross
```

After:

```text
jross labteam
```

Lesson:

`chgrp` changes a file's group ownership.

---

## Change File Ownership

```bash
sudo chown labuser file2
```

Verify:

```bash
ls -l
```

Before:

```text
jross jross
```

After:

```text
labuser jross
```

Lesson:

`chown` changes file ownership.

---

## Key Lessons Learned

- Symbolic mode modifies specific permissions without affecting others.
- Numeric mode sets the entire permission set at once.
- `chmod` changes permissions.
- `chgrp` changes group ownership.
- `chown` changes file ownership.
- Always verify changes with `ls -l`.

---

## Verification Checklist

- [x] Created test files
- [x] Modified permissions using symbolic mode
- [x] Modified permissions using numeric mode
- [x] Changed file group ownership
- [x] Changed file ownership
- [x] Verified all changes with `ls -l`

---

## RHCSA Notes

- Symbolic mode is useful when changing a specific permission.
- Numeric mode is useful when assigning an exact permission set.
- Common permission values:
  - 755 = executable script
  - 644 = standard file
  - 600 = private file
- Always verify ownership and permissions after making changes.
