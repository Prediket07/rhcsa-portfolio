# Lab 01 — User & Group Management

## Objective
Create and manage local user accounts and groups on Rocky Linux, apply group-based access control, and validate identity/permissions from the command line.

## Environment

- OS: Rocky Linux
- Hostname: rhcsa-lab
- Primary User: jross

## Commands Used

- useradd
- passwd
- groupadd
- usermod -aG
- id
- groups
- su -

## Walkthrough

### Create User

```bash
sudo useradd labuser
sudo passwd labuser
