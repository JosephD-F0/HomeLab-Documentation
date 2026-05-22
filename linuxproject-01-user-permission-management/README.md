# Lab Project 01 — Linux User & Permission Management

**Date:** 2026-05-21
**Environment:** Ubuntu 26.04 LTS | VirtualBox | Host: Windows 10 Pro
**Objective:** Create and manage Linux user accounts, assign group permissions,
restrict folder access, and simulate employee offboarding by locking an account.

---

## Scenario

A new helpdesk technician (helpdesk1) joined the team and needs:
- A user account created
- Administrator (sudo) privileges assigned
- A restricted folder in their home directory
- Account locked when they leave the company

---

## Steps Performed

### 1. Created new user account
```bash
sudo adduser helpdesk1
```
- Set password during prompt
- Confirmed creation with `id helpdesk1`

### 2. Granted sudo (admin) privileges
```bash
sudo usermod -aG sudo helpdesk1
```
- Verified with `getent group sudo`
- helpdesk1 appeared in sudo group confirming admin access

### 3. Created and restricted a folder
```bash
sudo mkdir /home/helpdesk1/restricted
sudo chmod 700 /home/helpdesk1/restricted
```
- Verified with `ls -la /home/helpdesk1/`
- Permission 700 means only the owner can read/write/execute
- Group and others have zero access

### 4. Locked account (simulating offboarding)
```bash
sudo passwd -l helpdesk1
```
- Verified with `sudo passwd -S helpdesk1`
- Output showed L flag confirming account is locked

### 5. Unlocked account
```bash
sudo passwd -u helpdesk1
```
- Verified with `sudo passwd -S helpdesk1`
- Output changed confirming account is active again

---

## Issues Encountered

**Problem:** `cat /etc/passwd | grep helpdesk1` returned no such file or directory
on first attempt.
**Cause:** Likely a syntax error in the path during initial attempt. Additionally
the first `adduser` command did not complete successfully, requiring a second attempt.
**Fix:** Re-ran `sudo adduser helpdesk1` successfully. Used `id helpdesk1` as an
alternative verification command which confirmed user and groups correctly.

---

## What I Learned

- Linux uses octal notation for permissions (700 = owner full access,
  group/others no access)
- `usermod -aG` adds a user to a group without removing existing group memberships
- Locking an account with `passwd -l` prevents login without deleting the user,
  which is important for auditing and compliance
- `id` is a reliable command to verify a user exists and see all their group
  memberships at once
- The L flag in `passwd -S` output confirms an account is locked
