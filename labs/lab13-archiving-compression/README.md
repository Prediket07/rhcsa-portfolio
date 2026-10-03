# Lab 13 — Archiving & Compression

## Objective

Demonstrate Linux archiving and compression by creating archives, compressing files, inspecting archive contents, extracting archives, and restoring files.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation

## Concepts Covered

| Command | Purpose |
|----------|----------|
| tar | Archive files and directories |
| gzip | Compress a file |
| gunzip | Decompress a file |
| ls | Verify files |
| rm | Remove files for recovery testing |

---

## Understanding Archiving vs Compression

Archiving:

```text
Many Files
↓
Single Package
```

Compression:

```text
Reduce File Size
```

Think:

```text
tar
=
Put files into a box

gzip
=
Vacuum-seal the box
```

---

## Create Test Files

Created a working directory and moved into it:

```bash
mkdir lab13files
cd lab13files
```

Created:

```bash
touch file1.txt
touch file2.txt
touch file3.txt
```

Verify:

```bash
ls
```

Result:

```text
file1.txt
file2.txt
file3.txt
```

Observation:

`touch` creates empty files, so all three files are 0 bytes. This matters later when comparing file sizes.

![Create working directory and test files](Lab13%20-%20Archiving%20and%20Compression/01-setup-files.jpg)

---

## Create Compressed Archive

Command:

```bash
tar -czvf backup.tar.gz file1.txt file2.txt file3.txt
```

Breakdown:

```text
c = create archive
z = compress using gzip
v = verbose output
f = filename
```

Observation:

Archive created successfully:

```text
backup.tar.gz
```

Because of `v` (verbose), `tar` printed each filename as it added it to the archive.

![tar -czvf creating backup.tar.gz](Lab13%20-%20Archiving%20and%20Compression/02-tar-create.jpg)

---

## Verify Archive Creation

Command:

```bash
ls -lh
```

Result:

```text
-rw-r--r--. 1 jross jross 145 Oct  2 18:29 backup.tar.gz
-rw-r--r--. 1 jross jross   0 Oct  2 18:18 file1.txt
-rw-r--r--. 1 jross jross   0 Oct  2 18:18 file2.txt
-rw-r--r--. 1 jross jross   0 Oct  2 18:18 file3.txt
```

Observation:

Archive was successfully created and appeared alongside the original files.

The archive is 145 bytes even though the three files inside are 0 bytes each. `tar` stores a header for every file (name, permissions, owner, timestamp), so an archive of empty files still takes up space. With real file contents, compression would make the archive smaller than the originals.

The `.` at the end of the permissions (`-rw-r--r--.`) means the file has an SELinux security context.

![ls -lh showing backup.tar.gz and tar -tvf output](Lab13%20-%20Archiving%20and%20Compression/03-tar-list.jpg)

---

## View Archive Contents

Command:

```bash
tar -tvf backup.tar.gz
```

Purpose:

```text
Display archive contents without extracting.
```

Observation:

Archive contained:

```text
file1.txt
file2.txt
file3.txt
```

along with metadata including:

- permissions (`-rw-r--r--`)
- ownership (`jross/jross`)
- size (`0`)
- timestamps (`2026-10-02 18:18`)

The `t` flag means "list the contents" (**t**able of contents).

![tar -tvf listing archive contents](Lab13%20-%20Archiving%20and%20Compression/03-tar-list.jpg)

---

## Test Recovery Process

Remove original files:

```bash
rm file1.txt file2.txt file3.txt
```

Verify:

```bash
ls
```

Result:

```text
backup.tar.gz
```

was the only remaining file.

![Original files deleted, only the archive remains](Lab13%20-%20Archiving%20and%20Compression/04-delete-originals.jpg)

---

## Extract Archive

Command:

```bash
tar -xzvf backup.tar.gz
```

Breakdown:

```text
x = extract
z = gzip archive
v = verbose
f = filename
```

Observation:

Files were restored:

```text
file1.txt
file2.txt
file3.txt
```

![tar -xzvf restoring the files](Lab13%20-%20Archiving%20and%20Compression/05-tar-extract.jpg)

---

## Verify Recovery

Commands:

```bash
ls
ls -lh
```

Result:

```text
backup.tar.gz
file1.txt
file2.txt
file3.txt
```

Observation:

All original files were successfully recovered.

The restored files show their original timestamp (`Oct 2 18:18`), not the time of extraction. `tar` preserves timestamps when archiving and extracting.

![Files recovered after extraction](Lab13%20-%20Archiving%20and%20Compression/05-tar-extract.jpg)

---

## Compress Single File

Command:

```bash
gzip file1.txt
```

Result:

```text
file1.txt.gz
```

Observation:

The original file was replaced by a compressed version. `gzip` printed no output, and `ls` showed `file1.txt.gz` with no `file1.txt`.

![gzip replacing file1.txt with file1.txt.gz](Lab13%20-%20Archiving%20and%20Compression/06-gzip.jpg)

---

## Understanding gunzip

Command used:

```bash
gunzip file1.txt
```

Purpose:

```text
Restore compressed file back to original form.
```

Observation:

`gunzip` was given the name `file1.txt` without the `.gz` ending. It still found `file1.txt.gz`, decompressed it, and `file1.txt` came back (the `.gz` file was removed).

`gunzip` works only on existing:

```text
.gz
```

files. The standard form is `gunzip file1.txt.gz`.

![gunzip restoring file1.txt](Lab13%20-%20Archiving%20and%20Compression/07-gunzip.jpg)

---

## Key Lessons Learned

- Archiving and compression are different operations.
- `tar` combines files and directories into a single archive.
- `gzip` compresses data.
- `tar -czvf` creates a compressed archive.
- `tar -xzvf` extracts a compressed archive.
- Archives can be inspected without extraction.
- Files can be deleted and recovered from an archive.
- Single-file compression uses `gzip` and `gunzip`.
- An archive of empty files still has a size because `tar` stores a header for each file.
- `tar` preserves file timestamps when restoring files.

---

## Verification Checklist

- [x] Created test files
- [x] Created archive
- [x] Compressed archive
- [x] Verified archive
- [x] Viewed archive contents
- [x] Deleted original files
- [x] Extracted archive
- [x] Verified file recovery
- [x] Compressed individual file
- [x] Demonstrated decompression workflow

---

## RHCSA Notes

Create archive:

```bash
tar -czvf backup.tar.gz files
```

View archive contents:

```bash
tar -tvf backup.tar.gz
```

Extract archive:

```bash
tar -xzvf backup.tar.gz
```

Compress file:

```bash
gzip filename
```

Decompress file:

```bash
gunzip filename.gz
```

Most important concept:

```text
Archive
≠
Compression

tar
=
Package files

gzip
=
Compress files
```
