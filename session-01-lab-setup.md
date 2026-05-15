## Home Lab Portfolio Entry — Session 1

---

### Overview

**Date:** April 2026
**Session Duration:** Multi-hour build and configuration session
**Objective:** Build a functional home lab environment using VirtualBox and Windows 10 Pro for CompTIA A+ exam preparation and entry-level helpdesk skill development
**Status:** ✅ Complete

---

### Environment

| Component | Detail |
| --- | --- |
| Host OS | Windows 10 |
| Hypervisor | Oracle VirtualBox |
| Guest OS | Windows 10 Pro (unactivated) |
| VM RAM | 4096 MB |
| VM Storage | 50 GB Dynamically Allocated VDI |
| VM Location | D:\VMs\Win10-Lab\ |
| Host C Drive (start) | ~3 GB available |
| Host C Drive (end) | ~17 GB available |

---

### Part 1 — Lab Environment Setup

### Objective

Install and configure a virtualized Windows 10 Pro environment on a personal machine using Oracle VirtualBox as the hypervisor.

### Steps Performed

- Downloaded and installed Oracle VirtualBox and the VirtualBox Extension Pack from virtualbox.org
- Downloaded the official Windows 10 ISO from Microsoft's software download page
- Created a new VM in VirtualBox named Win10-Lab with 4 GB RAM and a 50 GB dynamically allocated virtual disk
- Attached the Windows 10 ISO to the VM's virtual optical drive
- Proceeded through Windows 10 installation selecting Pro edition with no product key for lab purposes
- Completed Windows setup using a local account with no Microsoft account association
- Created local account: LabUser
- Took initial snapshot upon successful installation: "Clean Install — Day 1"

---

### Part 2 — Troubleshooting Encountered During Setup

Four distinct real-world issues were encountered and independently resolved during the setup process. Each issue is documented below in ticket format.

---

### Issue 1 — VERR_DISK_FULL During Windows Installation

**Error:** `The I/O cache encountered an error while updating data in medium "ahci-0-0" (rc=VERR_DISK_FULL)`

**Error ID:** BLKCACHE_IOERR

**Root Cause:**
VirtualBox uses the host machine's C drive for temporary cache and I/O operations during VM installation even when the VM's virtual disk is stored on a separate drive. The host C drive had approximately 3 GB of available space which was insufficient for the dynamic disk to expand during the Windows installation process.

**Troubleshooting Steps:**

- Ran Disk Cleanup on C drive including system file cleanup
- Identified that the VirtualBox VMs folder was stored in the Documents folder on C drive consuming approximately 8 GB
- Moved the VirtualBox VMs folder from C drive to D drive which had over 300 GB of available space
- Updated VirtualBox Default Machine Folder path to D drive via File → Preferences → General

**Resolution:**
Freed approximately 14 GB on C drive by relocating the VM folder and running disk cleanup. Host C drive increased from 3 GB to 17 GB available. Installation resumed successfully.

**Lesson Learned:**
VirtualBox requires available space on the host system drive for cache operations regardless of where the VM files are stored. Always verify host C drive free space before beginning a VM installation. Minimum recommended free space is 20–25 GB.

---

### Issue 2 — UUID Conflict After Manual VM Migration

**Error:** `Trying to open a VM config 'D:/VMs/Win10-Lab/Win10-Lab.vbox' which has the same UUID as an existing virtual machine.`

**Result Code:** E_FAIL (0x80004005)

**Root Cause:**
When the VirtualBox VMs folder was manually copied from C drive to D drive, VirtualBox retained the original registration of the VM pointing to the C drive path. When attempting to add the D drive copy, VirtualBox detected two registrations sharing an identical UUID causing a conflict and refusing to open either instance.

**Troubleshooting Steps:**

- Attempted Machine → Add to point VirtualBox directly to the .vbox file on D drive — failed due to UUID conflict
- Identified that the original broken C drive VM entry was still registered in VirtualBox
- Right-clicked all existing VM entries in VirtualBox sidebar → Remove → Remove Only (not delete files) to fully clear all registrations
- Re-added the VM fresh from D drive via Machine → Add → navigated to Win10-Lab.vbox on D drive

**Resolution:**
Clearing all stale VM registrations from VirtualBox and re-adding the D drive copy fresh eliminated the UUID conflict. VM opened successfully.

**Alternative Resolution (if above fails):**
UUID can be manually reassigned using VBoxManage via command:
`"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" internalcommands sethduuid "D:\VMs\Win10-Lab\Win10-Lab.vdi"`

**Lesson Learned:**
Never move VM files manually while they are registered in VirtualBox. VirtualBox tracks VMs by UUID and retains stale registrations when files are moved outside of the application. Always use VirtualBox's built-in tools to move or copy VMs, or fully remove the registration before re-adding from the new location.

---

### Issue 3 — Mouse Captured Inside VM

**Symptom:** Mouse cursor disappeared after clicking inside the VirtualBox window and could not be moved back to the host desktop.

**Root Cause:**
VirtualBox captures the mouse input when a user clicks inside the VM window. This is standard expected behavior in VirtualBox when VirtualBox Guest Additions are not yet installed.

**Resolution:**
Pressing the **Right Ctrl key** (VirtualBox default host key) immediately releases mouse capture and returns control to the host machine.

**Lesson Learned:**
Right Ctrl is the VirtualBox host key. It releases mouse and keyboard capture at any time. This key combination is fundamental to VirtualBox operation and should be memorized.

---

### Issue 4 — Forgotten Standard User Password

**Symptom:** Password and security question answers for standard user account JohnDoe were unknown.

**Root Cause:**
User error — password not recorded after account creation.

**Resolution:**

- Logged into ITAdmin administrator account
- Opened Computer Management via compmgmt.msc
- Navigated to Local Users and Groups → Users
- Right-clicked JohnDoe → Set Password → Proceed
- Set new password without requiring knowledge of the previous password or security question answers

**Lesson Learned:**
Administrator accounts can reset any local user password without knowledge of the existing password or security answers. Security questions are irrelevant when administrator access is available. This is the standard helpdesk procedure for forgotten password tickets in environments without Active Directory.
