#  Lab Project 05 — Service Management & Log Analysis

**Date:** 2026-05-22
**Environment:** Ubuntu 26.04 LTS | VirtualBox | Host: Windows 10 Pro
**Objective:** Install and manage the SSH service, simulate a real service
failure by disabling auto-start, restore full functionality, and analyze
system logs to diagnose service issues.

---

## Scenario

A user reports that SSH remote access to their machine has stopped working.
As the helpdesk tech you need to check the service status, stop and start it,
ensure it auto-starts on reboot, dig through logs to find what went wrong,
and verify other critical services are running correctly.

---

## Steps Performed

### 1. Installed SSH server
```bash
sudo apt install openssh-server
ssh -V
```
- Installed the OpenSSH server package
- Verified installation and version with `ssh -V`
- SSH is the standard protocol for remotely accessing and managing
  Linux machines in professional environments

### 2. Checked initial service status
```bash
sudo systemctl status ssh
sudo systemctl is-enabled ssh
```
- Status showed active (running) but enabled showed disabled in yellow
- This revealed an important real world scenario — a service can be
  running right now but not set to survive a reboot
- Confirmed the difference between a service being started vs enabled

### 3. Stopped and started the service
```bash
sudo systemctl stop ssh
sudo systemctl status ssh
sudo systemctl start ssh
sudo systemctl status ssh
```
- Stopped SSH and confirmed status changed to inactive (dead)
- Restarted SSH and confirmed status returned to active (running)
- Simulates the most common helpdesk fix for a frozen or misbehaving service

### 4. Tested restart and reload
```bash
sudo systemctl restart ssh
sudo systemctl reload ssh
```
- Restart completely stops then starts the service — used when a service
  is fully broken or unresponsive
- Reload applies new configuration without stopping the service — zero
  downtime, used after config file changes
- Understanding the difference is critical in production environments
  where stopping a service interrupts active users

### 5. Simulated service failure (the break)
```bash
sudo systemctl disable ssh
sudo systemctl is-enabled ssh
sudo systemctl stop ssh
sudo systemctl status ssh
```
- Disabled SSH from startup sequence and stopped it completely
- Confirmed disabled status and inactive (dead) state
- Simulates a common real world issue where a service loses its
  auto-start setting after a system update or accidental config change

### 6. Restored full service functionality (the fix)
```bash
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
sudo systemctl is-enabled ssh
```
- Re-enabled SSH in the startup sequence
- Started the service immediately without needing a reboot
- Confirmed active (running) and enabled — fully restored
- Service is now running immediately AND will survive future reboots

### 7. Analyzed service logs
```bash
sudo journalctl -u ssh
sudo journalctl -u ssh --since "1 hour ago"
sudo journalctl -xe
sudo tail -f /var/log/syslog
```
- `journalctl -u ssh` showed full log history for SSH including all
  start, stop, and error events from this session
- `--since "1 hour ago"` filtered logs to the relevant timeframe —
  useful when you know roughly when a problem started
- `journalctl -xe` showed system wide logs with extra detail — the
  go-to command when something just broke and you don't know what
- `tail -f /var/log/syslog` opened a live log feed showing new entries
  in real time — used when actively monitoring a system during an incident
- Pressed Ctrl + C to exit live feed

### 8. Checked status of multiple services
```bash
sudo systemctl status ufw
sudo systemctl status cron
systemctl list-units --type=service --state=running
```
- Checked UFW firewall service status
- Checked cron task scheduler service status
- Listed all currently running services on the system
- Gives a full picture of system health — useful for auditing and
  identifying unexpected or missing services

---

## Issues Encountered

**Problem:** After running `sudo systemctl start ssh` the service showed
active (running) but `is-enabled` still showed disabled in yellow.
**Cause:** Start and enable are completely independent commands. Start turns
a service on right now. Enable makes it auto-start on every reboot. A service
can be running but still set to not survive a reboot.
**Fix:** Ran `sudo systemctl enable ssh` separately to set the auto-start.
Then verified both active (running) AND enabled were confirmed.
**Why this matters:** This is a very common real world mistake. A tech
manually starts a service and thinks the problem is solved. After the next
reboot the service is dead again and the user calls back with the same issue.
The correct fix is always both enable AND start.

---

## What I Learned

- `systemctl status` should always be the first command run before
  touching any service — establishes current state before making changes
- Start and enable are two different things — a service can be running
  but not set to survive a reboot, or enabled but not currently running
- The correct fully fixed state is always both active (running) AND enabled
- Restart completely interrupts a service while reload applies config
  changes with zero downtime — knowing which to use matters in production
- `journalctl` is the primary tool for reading service logs and finding
  the root cause of failures
- Rebooting fixes the symptom — logs tell you the actual cause
- `tail -f /var/log/syslog` is used for live monitoring during an
  active incident
- Checking multiple services with `list-units` gives a full picture
  of system health beyond just the service you're troubleshooting

---

## CompTIA A+ Relevance

- Domain 1.9 — Explain features and tools of Linux operating systems
- Domain 4.9 — Best practices for securing a workstation including
  service management
- Domain 5.3 — Troubleshoot common issues with services and processes
- Domain 5.6 — Troubleshoot common OS problems using logs and diagnostic tools
