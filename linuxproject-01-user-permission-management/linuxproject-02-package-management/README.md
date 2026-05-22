# Lab Project 02 — Package Management & Software Installation

**Date:** 2026-05-21
**Environment:** Ubuntu 26.04 LTS | VirtualBox | Host: Windows 10 Pro
**Objective:** Install, verify, remove, and reinstall software using the apt
package manager. Simulate real helpdesk software troubleshooting workflow.

---

## Scenario

A user reports their software is broken and won't open. As the helpdesk tech
you need to remove the broken installation and perform a clean reinstall.
You also need to verify the system has no broken packages before proceeding.

---

## Steps Performed

### 1. Updated package list
```bash
sudo apt update && sudo apt upgrade
```
- Ensures system has latest package information before installing anything
- Best practice before any software installation

### 2. Checked for broken packages
```bash
sudo apt --fix-broken install
```
- Returned 0 upgrading — no broken packages found
- Clean system confirmed before proceeding

### 3. Installed htop
```bash
sudo apt install htop
```
- Accepted prompt with Y to confirm installation
- htop is a real time process and system monitor

### 4. Verified installation
```bash
dpkg -l | grep htop
```
- Output showed ii prefix confirming successful installation
- Launched htop to visually confirm it worked
- Pressed Q to exit htop

### 5. Removed htop
```bash
sudo apt remove htop
```
- Accepted prompt with Y to confirm removal
- Simulates removing broken or unwanted software

### 6. Verified removal
```bash
dpkg -l | grep htop
```
- Returned no output confirming htop was fully removed

### 7. Reinstalled htop
```bash
sudo apt install htop
```
- Clean reinstall simulating helpdesk fix for broken software
- Accepted prompt with Y

### 8. Final verification
```bash
dpkg -l | grep htop
```
- Output showed ii prefix again confirming successful reinstall

---

## Issues Encountered

**Problem:** Initial attempt to remove htop using `remove` command returned
command not found. Followed by `rm` returning no file or directory found.
**Cause:** `remove` is not a standalone Linux command. `rm` is for deleting
files, not uninstalling packages. htop also had not installed successfully
on the first attempt.
**Fix:** Used correct package manager syntax `sudo apt remove htop` after
first confirming htop was properly installed with `sudo apt install htop`
and verifying with `dpkg -l | grep htop`.

---

## What I Learned

- `apt` is the correct tool for installing and removing software on Ubuntu
- Always run `sudo apt update` before installing to get the latest package info
- `dpkg -l | grep <packagename>` is the correct way to verify if software
  is installed
- The `ii` prefix in dpkg output means the package is installed and working
- `rm` removes files, `apt remove` uninstalls packages — these are different
- `--fix-broken` is useful for diagnosing and repairing broken dependencies
