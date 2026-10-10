# Lab 17 — Shell Scripting

## Objective

Demonstrate basic Bash shell scripting by creating executable scripts, using variables, passing command-line arguments, applying conditional logic, looping through values, and using command substitution.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation

## Concepts Covered

| Concept | Purpose |
|----------|----------|
| `#!/bin/bash` | Shebang: tells the system which interpreter runs the script |
| `chmod +x` | Make a script executable (chmod = change mode) |
| `./script.sh` | Run a script from the current directory |
| Variables | Store values for reuse |
| `$1`, `$2` | Positional arguments passed to a script |
| `if / then / else / fi` | Conditional logic |
| `for ... do ... done` | Loop through a list of values |
| `$(command)` | Command substitution: store command output in a variable |

---

## Understanding Shell Scripting

A shell script is a text file containing commands that the shell runs in order.

Think:

```text
Commands typed by hand
↓
Saved in a file
↓
Run again anytime
```

Every script in this lab starts with:

```bash
#!/bin/bash
```

This is the shebang. It tells Linux to run the file with Bash.

A script must be executable before it can be run directly:

```bash
chmod +x script.sh
./script.sh
```

`./` means "run it from the current directory."

---

## Create First Script

Script: `firstscript.sh`

```bash
#!/bin/bash
echo "Hello RHCSA"
date
whoami
hostname
```

Create it:

```bash
nano firstscript.sh
```

![firstscript.sh in nano](Lab17%20-%20Shell%20Scripting/01-firstscript-nano.jpg)

Make executable and run:

```bash
chmod +x firstscript.sh
./firstscript.sh
```

![firstscript.sh output](Lab17%20-%20Shell%20Scripting/02-firstscript-run.jpg)

Observation:

The script printed the greeting, the current date, the logged-in user, and the hostname, one command after another.

---

## Working With Variables

Script: `variables.sh`

Variables store a value under a name so it can be reused.

```bash
NAME="value"
echo "$NAME"
```

Rules:

```text
No spaces around =
Use $ to read the value
```

![variables.sh output](Lab17%20-%20Shell%20Scripting/03-variables-run.jpg)

Observation:

The script stored values in variables and printed them back using `$VARIABLE`.

---

## Using Script Arguments

Script: `hello.sh`

```bash
#!/bin/bash

echo "Hello $1"
echo "Welcome to RHCSA"
```

Run:

```bash
./hello.sh Jairus
```

`$1` holds the first argument typed after the script name.

![hello.sh with an argument](Lab17%20-%20Shell%20Scripting/04-hello-args.jpg)

Troubleshooting note:

One attempt had a typo (`,/hello.sh` instead of `./hello.sh`) and Bash returned a command-not-found error. Retyping the command correctly fixed it.

---

## Conditional Logic

Script: `checkname.sh`

Conditional logic lets a script make decisions.

```bash
#!/bin/bash

if [ "$1" = "Bob" ]
then
    echo "Welcome Bob"
else
    echo "You are not Bob"
fi
```

Run with no argument and with `Bob`:

```bash
./checkname.sh
./checkname.sh Bob
```

![checkname.sh runs](Lab17%20-%20Shell%20Scripting/05-checkname-run.jpg)

Observation:

With no argument, `$1` was empty, so the test failed and the script printed `You are not Bob`. With `Bob`, the test passed and it printed `Welcome Bob`.

Syntax reminders:

```text
Spaces required inside [ ]
if ... then ... else ... fi
fi closes the if block
```

---

## For Loops

Script: `names.sh`

A loop repeats a block of commands for each item in a list.

```bash
#!/bin/bash

for NAME in Bob Jason Mike
do
    echo "Hello $NAME"
done
```

![names.sh in nano](Lab17%20-%20Shell%20Scripting/06-names-for-loop-nano.jpg)

Run:

```bash
chmod +x names.sh
./names.sh
```

![names.sh output](Lab17%20-%20Shell%20Scripting/07-names-for-loop-run.jpg)

Observation:

The loop ran three times, once for each name, printing a greeting each time.

---

## Loop Example: users.sh

Script: `users.sh`

This script used the same loop pattern with a list of usernames.

![users.sh output](Lab17%20-%20Shell%20Scripting/08-users-loop-run.jpg)

Observation:

The output printed `Creating user: ...` for each name in the list. This script only echoed messages. No accounts were actually created.

To create real accounts, the loop body would need `sudo useradd "$USER_NAME"` instead of `echo`.

---

## Multiple Arguments

Script: `userinfo.sh`

```bash
#!/bin/bash

echo "Username: $1"
echo "Department: $2"
```

![userinfo.sh in nano](Lab17%20-%20Shell%20Scripting/09-userinfo-nano.jpg)

Run:

```bash
./userinfo.sh <username> <department>
```

![userinfo.sh output](Lab17%20-%20Shell%20Scripting/10-userinfo-run.jpg)

Observation:

`$1` and `$2` captured the first and second arguments in order.

```text
$1 = first argument
$2 = second argument
```

---

## Command Substitution

Script: `systeminfo.sh`

Command substitution runs a command and stores its output in a variable.

```bash
#!/bin/bash

CURRENT_USER=$(whoami)
CURRENT_HOST=$(hostname)
CURRENT_DATE=$(date)

echo "User: $CURRENT_USER"
echo "Host: $CURRENT_HOST"
echo "Date: $CURRENT_DATE"
```

![systeminfo.sh in nano](Lab17%20-%20Shell%20Scripting/11-systeminfo-nano.jpg)

First run:

![systeminfo.sh error](Lab17%20-%20Shell%20Scripting/12-systeminfo-run-error.jpg)

Troubleshooting:

```text
unexpected EOF while looking for matching '`'
```

Cause: a stray backtick (`` ` ``) on its own line at the end of the file. Bash treated it as the start of an old-style command substitution that was never closed.

Fix:

```text
Open the file in nano
Delete the stray backtick line
Save and re-run
```

Lesson:

```text
Quotes and backticks must always be paired.
An "unexpected EOF" error usually means something was opened and never closed.
```

---

## Key Lessons Learned

- A shell script is a saved list of commands.
- The shebang `#!/bin/bash` selects the interpreter.
- Scripts need `chmod +x` before running with `./script.sh`.
- Variables are assigned with no spaces around `=` and read with `$`.
- `$1` and `$2` are positional arguments.
- `if / then / else / fi` lets scripts make decisions.
- `for ... do ... done` repeats work for each item in a list.
- `$(command)` stores command output in a variable.
- Typos and stray characters produce syntax errors. Read the error and the line number.

---

## Verification Checklist

- [x] Created first script
- [x] Made scripts executable
- [x] Used variables
- [x] Passed an argument with `$1`
- [x] Used `if / else` logic
- [x] Looped through a list with `for`
- [x] Used multiple arguments (`$1`, `$2`)
- [x] Used command substitution
- [x] Troubleshot a syntax error

---

## RHCSA Notes

Make a script executable and run it:

```bash
chmod +x script.sh
./script.sh
```

Arguments:

```bash
echo "$1 $2"
```

Conditional:

```bash
if [ "$1" = "Bob" ]
then
    echo "Welcome Bob"
else
    echo "Not Bob"
fi
```

Loop:

```bash
for ITEM in one two three
do
    echo "$ITEM"
done
```

Command substitution:

```bash
TODAY=$(date)
```

Most important concept:

```text
Script
=
Commands + Logic + Input
```
