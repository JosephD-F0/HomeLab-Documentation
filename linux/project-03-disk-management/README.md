 Lab Project 03 — Disk & File System Management

**Date:** 2026-05-22
**Environment:** Ubuntu 26.04 LTS | VirtualBox | Host: Windows 10 Pro
**Objective:** Simulate a real helpdesk storage issue — identify what is consuming
disk space, clean it up, and verify the system's partition and storage configuration.

---

## Scenario

A user reports their home directory is full and the system is running low on
storage. As the helpdesk tech you need to investigate what is consuming space,
locate the large files, clean them up, and verify the disk and partition layout.

---

## Steps Performed

### 1. Checked overall disk usage
```bash
df -h
```
- Displays total size, used space, available space, usage percentage, and mount points
- First command to run when a user reports a full drive

### 2. Identified large directories
```bash
sudo du -sh /*
```
- Shows size of all top level directories
- Used to pinpoint which folders are consuming the most storage

### 3. Created a test directory and simulated junk files
```bash
mkdir -p ~/homelab_junk
cd ~/homelab_junk
dd if=/dev/zero of=bigfile.bin bs=1M count=50
dd if=/dev/zero of=junk2.bin bs=1M count=120
dd if=/dev/zero of=junk3.bin bs=1M count=200
```
- Created a test folder to simulate a storage problem
- Used `dd` to generate large files totaling 370 MB of fake junk data
- Simulates what accumulates on a real user's machine over time

### 4. Verified the junk files existed
```bash
ls -lh
du -sh ~/homelab_junk
```
- Confirmed files were created and visible
- Verified space consumption before cleanup

### 5. Located large files
```bash
find ~/homelab_junk -type f -size +100000k
```
- Searched for files larger than 100 MB
- Used kilobyte format after discovering M suffix behaved unexpectedly
  in this environment

### 6. Removed junk files
```bash
rm junk2.bin junk3.bin
```
- Deleted unnecessary files simulating a real cleanup
- Verified removal with `ls -lh` and `du -sh ~/homelab_junk`

### 7. Reviewed partition layout
```bash
sudo fdisk -l
```
- Displayed full partition structure of the virtual disk
- Shows how storage is divided and how the system is using it

### 8. Verified block devices
```bash
lsblk
```
- Confirmed which virtual disk VirtualBox attached
- Shows how Linux recognizes and organizes storage devices

---

## Issues Encountered

**Problem 1:** `fallocate` did not reliably create test files as expected.
**Fix:** Switched to `dd if=/dev/zero` which successfully generated large
test files of exact sizes.

**Problem 2:** `find` command using `-size +100M` did not return expected results.
**Cause:** The M suffix behaved differently than expected in this environment.
**Fix:** Used kilobyte format `-size +100000k` instead which correctly located
files larger than 100 MB.

---

## What I Learned

- `df -h` gives a fast overview of disk usage — always the first command
  to run for storage complaints
- `du -sh /*` narrows down which directories are consuming the most space
- `dd` is a reliable way to generate test files of exact sizes for lab simulations
- `find` with `-size` is a powerful way to hunt down large files on a system
- `fdisk -l` and `lsblk` reveal how the disk is partitioned and how Linux
  sees attached storage devices
- Real world troubleshooting requires adapting commands when they behave
  unexpectedly — flexibility is a core helpdesk skill

---

## CompTIA A+ Relevance

- Domain 1.9 — Explain features and tools of Linux
- Domain 4.5 — Summarize environmental impacts and local controls
- Domain 5.3 — Troubleshoot common issues with storage devices
