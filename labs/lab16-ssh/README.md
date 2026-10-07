# Lab 16 — SSH (Secure Shell)

## Objective

Demonstrate secure remote administration using SSH by verifying the SSH service, identifying listening ports, connecting over the network, generating an SSH key pair, configuring passwordless (key-based) authentication, and inspecting the files and permissions SSH uses.

## Environment

- OS: Rocky Linux 10.2
- Hostname: rhcsa-lab.local
- Primary User: jross
- Hypervisor: VMware Workstation
- Note: this lab connected to the VM's own IP address (192.168.243.128) from the same machine. A second system was not used.

## Concepts Covered

| Command | Purpose |
|----------|----------|
| systemctl status sshd | Verify SSH service status |
| ss -tulpn | Verify listening ports |
| ip -4 addr show | Identify system IP address |
| ssh | Connect to a system over SSH |
| ssh-keygen | Generate SSH key pair |
| ls -l ~/.ssh | View key files and permissions |
| cat | View public key |
| ssh-copy-id | Install public key on server |
| ls -la ~/.ssh | Confirm authorized_keys was created |

---

## Understanding SSH

SSH (Secure Shell) allows secure remote administration of Linux systems.

Think:

```text
Local Computer
      ↓
Encrypted Connection
      ↓
Remote Linux Server
```

SSH replaces insecure protocols such as Telnet.

Default SSH Port:

```text
22
```

---

## Verify SSH Service

Command:

```bash
sudo systemctl status sshd
```

Observation:

```text
Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
Active: active (running)
Main PID: 1405 (sshd)
```

The SSH daemon was running and configured to start automatically at boot. The log lines at the bottom of the output show it listening on port 22.

![sshd service status and listening port](Lab16%20-%20SSH/01-sshd-status-and-listening-port.jpg)

---

## Verify SSH Listening Port

Command (shown at the bottom of the same screenshot):

```bash
sudo ss -tulpn | grep sshd
```

Observation:

```text
tcp  LISTEN  0  128  0.0.0.0:22  0.0.0.0:*  users:(("sshd",pid=1405,fd=7))
tcp  LISTEN  0  128     [::]:22     [::]:*  users:(("sshd",pid=1405,fd=8))
```

The SSH service was listening on:

```text
Port 22
```

for both IPv4 (`0.0.0.0`) and IPv6 (`[::]`), owned by the sshd process (PID 1405).

---

## Identify System Address

Command:

```bash
ip -4 addr show
```

Observation:

```text
ens160: ... state UP
inet 192.168.243.128/24 brd 192.168.243.255 scope global dynamic ens160
```

System IP address:

```text
192.168.243.128
```

This address was used for SSH connection testing.

![ip addr show output](Lab16%20-%20SSH/02-ip-addr-show.jpg)

---

## Establish SSH Connection

Command:

```bash
ssh jross@192.168.243.128
```

First connection produced:

```text
The authenticity of host '192.168.243.128 (192.168.243.128)' can't be established.
ED25519 key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Response:

```text
yes
```

Observation:

```text
Warning: Permanently added '192.168.243.128' (ED25519) to the list of known hosts.
```

The host key was added to:

```text
~/.ssh/known_hosts
```

SSH then asked for the account password, and login was successful.

![first SSH connection and host key prompt](Lab16%20-%20SSH/03-first-ssh-connection.jpg)

---

## Verify SSH Session

Commands:

```bash
whoami
hostname
pwd
```

Results:

```text
jross
rhcsa-lab.local
/home/jross
```

Observation:

Verified successful authentication and a working SSH session. The same screenshot shows `ssh-keygen` being started, which is the next step.

![session verification and ssh-keygen start](Lab16%20-%20SSH/04-session-verify-and-keygen-start.jpg)

---

## Generate an SSH Key Pair

Command:

```bash
ssh-keygen
```

Responses:

```text
Enter file in which to save the key: (pressed Enter for the default, /home/jross/.ssh/id_ed25519)
Enter passphrase: (left empty)
```

Observation:

```text
Your identification has been saved in /home/jross/.ssh/id_ed25519
Your public key has been saved in /home/jross/.ssh/id_ed25519.pub
```

An ED25519 key pair was created. The private key stays secret. The public key is the one shared with servers.

![ssh-keygen completed](Lab16%20-%20SSH/05-ssh-keygen-complete.jpg)

---

## Inspect SSH Key Files and Permissions

Commands:

```bash
ls ~/.ssh
ls -l ~/.ssh
```

Observation:

```text
-rw-------. 1 jross jross 411 Oct  6 19:49 id_ed25519
-rw-r--r--. 1 jross jross 103 Oct  6 19:49 id_ed25519.pub
-rw-------. 1 jross jross 843 Oct  6 19:45 known_hosts
-rw-r--r--. 1 jross jross  97 Oct  6 19:44 known_hosts.old
```

The private key (`id_ed25519`) is readable only by its owner (`600`). The public key is world-readable (`644`), which is safe because it is meant to be shared. SSH refuses to use a private key that other users can read.

![ssh directory and file permissions](Lab16%20-%20SSH/06-ssh-directory-permissions.jpg)

---

## Install the Public Key (Passwordless Setup)

Commands:

```bash
cat ~/.ssh/id_ed25519.pub
ssh-copy-id jross@192.168.243.128
```

Observation:

```text
jross@192.168.243.128's password:
Number of key(s) added: 1
```

`cat` displayed the public key (`ssh-ed25519 AAAA... jross@rhcsa-lab.local`). `ssh-copy-id` asked for the account password one last time and installed the key on the server.

![public key and ssh-copy-id](Lab16%20-%20SSH/07-ssh-copy-id.jpg)

---

## Test Passwordless Login

Commands:

```bash
exit
ssh jross@192.168.243.128
```

Observation:

```text
logout
Connection to 192.168.243.128 closed.
Last login: Tue Oct  6 19:45:00 2026 from 192.168.243.128
```

After exiting the first session, the new `ssh` command logged in immediately with no password prompt. This confirms key-based authentication works.

![passwordless SSH login](Lab16%20-%20SSH/08-passwordless-login.jpg)

---

## Confirm authorized_keys

Command:

```bash
ls -la ~/.ssh
```

Observation:

```text
drwx------.  2 jross jross  111 Oct  6 19:56 .
-rw-------.  1 jross jross  103 Oct  6 19:56 authorized_keys
-rw-------.  1 jross jross  411 Oct  6 19:49 id_ed25519
-rw-r--r--.  1 jross jross  103 Oct  6 19:49 id_ed25519.pub
```

`ssh-copy-id` created `authorized_keys`, the list of public keys allowed to log in. Its size (103 bytes) matches `id_ed25519.pub`. The `.ssh` directory is `700` and `authorized_keys` is `600`, the permissions SSH expects.

![authorized_keys created](Lab16%20-%20SSH/09-authorized-keys.jpg)

---

## Verification Checklist

- [x] sshd service is active (running) and enabled
- [x] sshd is listening on port 22 (IPv4 and IPv6)
- [x] System IP address identified (192.168.243.128)
- [x] First SSH connection accepted the host key and logged in
- [x] ED25519 key pair generated
- [x] Private key permissions are 600, public key 644
- [x] Public key installed with ssh-copy-id
- [x] Passwordless login confirmed
- [x] authorized_keys exists with 600 permissions

---

## RHCSA Notes

- **sshd** = SSH Daemon, the background service that accepts SSH connections. `systemctl enable --now sshd` starts it now and at every boot.
- `ssh-keygen` creates the key pair. `ssh-copy-id` adds the public key to the server's `~/.ssh/authorized_keys`.
- Key permissions matter: `~/.ssh` should be `700`. Private keys and `authorized_keys` should be `600`.
- `known_hosts` stores the fingerprints of servers you have connected to, so SSH can warn you if one changes.
- `ss -tulpn` shows which service owns which port (t = TCP, u = UDP, l = listening, p = process, n = numeric).
- Not covered in this lab (RHCSA topics still to practice): editing `/etc/ssh/sshd_config` (for example `PermitRootLogin no`), restarting sshd after a change, `scp` file copy, and firewall/SELinux handling for a non-default SSH port.
