# Project: Windows Server 2022 Domain Controller + Windows 11 Client — Lab Build & Setup

## Overview

This document covers the full build of a two-VM Active Directory lab environment from scratch, including extended troubleshooting across multiple hypervisors, repeated VM boot failures, BSOD errors during domain controller promotion, and account lockout during domain join. Every failure is documented here deliberately — real IT work involves dead ends, and the troubleshooting process itself is the skill.

**End state:** A fully functional `homelab.local` domain running on VMware Workstation Pro, with a Windows Server 2022 Domain Controller (`DC01`) and a Windows 11 Pro client (`WIN11-01`) successfully domain-joined and snapshotted as a clean baseline for a 14-project helpdesk/sysadmin curriculum.

---

## Environment

| Component | Details |
|---|---|
| Host Machine | Windows 11 Pro laptop |
| Primary Hypervisor | VMware Workstation Pro (free for personal use) |
| Domain Controller VM | `DC01` — Windows Server 2022 Standard Evaluation (Desktop Experience) |
| Client VM | `WIN11-01` — Windows 11 Pro |
| Virtual Network | VMware LAN Segment — `labnet` (isolated, no internet) |
| Domain Name | `homelab.local` |
| DC01 IP | `192.168.50.10` (static) |
| WIN11-01 IP | `192.168.50.20` (static) |
| DNS Server | DC01 (`192.168.50.10`) |

---

## Phase 1 — Hypervisor Selection

### Initial plan: VirtualBox 7.2.8

VirtualBox was already installed and had previously and successfully hosted a Windows 10 Pro VM and an Ubuntu 24.04 LTS VM on this machine. The plan was to build both new VMs there as well.

### What happened

Both the Windows 11 ISO and the Windows Server 2022 ISO failed to boot in every newly created VirtualBox VM, despite the existing VMs continuing to work normally. The failures were consistent and identical across both ISOs, which ruled out file corruption as the cause.

**Troubleshooting steps attempted in VirtualBox:**

- Corrected OS type (defaulted to Windows NT 4 due to Easy Install auto-detection failure)
- Manually attached ISOs via Storage settings with Live CD/DVD checked
- Removed Mark of the Web file block (Windows flags internet-downloaded files — unchecking this in file Properties is required)
- Switched optical drive from SATA controller to IDE controller
- Toggled EFI on and off — EFI caused a pure silent black screen; legacy BIOS produced a "No bootable device found" loop
- Installed VirtualBox Extension Pack 7.2.10
- Verified hardware virtualization (Intel VT-x) enabled at BIOS level
- Confirmed graphics controller set to VBoxSVGA
- Re-downloaded Windows 11 ISO using the Media Creation Tool (which validates integrity automatically) — identical failure
- Confirmed both ISOs failed identically despite being sourced from completely separate Microsoft portals

**Root cause:** Never definitively isolated within VirtualBox. The identical failure across two independent ISOs while existing VMs remained functional pointed to a configuration-level difference between the new and old VMs rather than a file or host-level problem. A direct config diff via `VBoxManage showvminfo` was identified as the correct next diagnostic step, but the decision was made to switch hypervisors instead given the time already invested.

**Decision:** Switch to VMware Workstation Pro, which is now free for personal use. Both ISOs booted successfully on the first attempt in VMware.

**Portfolio note:** Extended, unresolved troubleshooting with systematic elimination of variables is documented separately in `troubleshooting-vm-boot-failure.md`. The VirtualBox experience itself is a valid portfolio artifact demonstrating methodical diagnosis under uncertainty.

---

## Phase 2 — Windows 11 Pro VM Build (WIN11-01)

### VM configuration in VMware

| Setting | Value |
|---|---|
| VM Name | `WIN11-Client` (renamed to WIN11-01 post-install) |
| Memory | 4096 MB |
| Processors | 2 |
| Hard Disk | 60 GB |
| Network Adapter | LAN Segment — `labnet` |
| ISO | Windows 11 Pro (multi-edition, en-US) |
| Install method | Manual (Easy Install bypassed) |

### Installation steps

1. Created VM in VMware using Typical configuration
2. Attached Windows 11 ISO via Settings → CD/DVD
3. Booted VM — pressed any key at the "Press any key to boot from CD" prompt
4. Selected language/keyboard → Install Now
5. Selected **Windows 11 Pro** (required for domain join — Home edition cannot join a domain)
6. At Microsoft account screen: clicked **I don't have internet** → **Continue with limited setup** to create a local account instead
7. Created local account: `localadmin`
8. Disabled all optional privacy toggles

### Post-install configuration

- Renamed PC to `WIN11-01` via Settings → System → About → Rename
- Restarted after rename
- Took snapshot: `WIN11-01 - clean install - pre domain join`

---

## Phase 3 — Windows Server 2022 VM Build (DC01)

### VM configuration in VMware

| Setting | Value |
|---|---|
| VM Name | `DC01` |
| Memory | 4096 MB |
| Processors | 2 |
| Hard Disk | 60 GB |
| Network Adapter | LAN Segment — `labnet` |
| ISO | Windows Server 2022 Evaluation (en-US) |
| Install method | Manual (Easy Install bypassed via .vmx edit) |

### Issue encountered — Easy Install interference

VMware's Easy Install feature was selected during VM creation. This caused a recurring error during OS setup:

```
Windows cannot find Microsoft Software License Terms.
Make sure the installation sources are valid and restart the installation.
```

This error appeared after the Windows logo during the setup loading screen, before the installer fully launched. It recurred on every boot attempt regardless of retrying or remounting the ISO.

**Root cause:** Easy Install injects an `autoinst.flp` floppy automation file into the boot process. This file interferes with how the Server 2022 installer reads its own license terms from the ISO, causing the setup process to fail before it can present the install wizard.

**Fix — .vmx file edit to remove Easy Install:**

1. Powered off DC01
2. Right-clicked DC01 in VMware → Open VM directory
3. Opened `DC01.vmx` in Notepad
4. Removed the following three lines entirely:

```
floppy0.fileType = "file"
floppy0.fileName = "autoinst.flp"
floppy0.clientDevice = "FALSE"
```

5. Changed the guest OS line from:

```
guestOS = "windows2019srvnext-64"
```

to:

```
guestOS = "windows2022srvnext-64"
```

6. Saved the file and rebooted the VM
7. Pressed any key immediately at the "Press any key to boot from CD" prompt
8. Installer loaded cleanly into the manual Windows Server 2022 setup wizard

**Lesson:** VMware Easy Install is convenient for basic setups but unreliable for Windows Server ISOs. Manual installation via .vmx edit is the more reliable and controlled path for lab VMs. The `autoinst.flp` floppy file is the specific artifact to look for and remove.

### OS edition selected

**Windows Server 2022 Standard Evaluation (Desktop Experience)**

- Standard vs Datacenter: Datacenter is designed for large enterprise environments running dozens of VMs. Standard is the correct fit for small-to-mid IT environments and lab work.
- Desktop Experience vs Server Core: Desktop Experience provides a full GUI. Server Core is command-line only. Desktop Experience was selected for initial lab builds — Server Core can be practiced separately once the GUI workflows are understood.

### Post-install configuration

**Renamed server:**
- Server Manager → Local Server → clicked computer name → Change → `DC01` → restarted

**Set static IP (required for a domain controller):**

A domain controller must have a fixed IP address. If it changes, domain-joined clients cannot locate the DC and the domain breaks. DNS, Kerberos authentication, and AD replication all depend on the DC being reachable at a consistent address.

- Network adapter → IPv4 Properties:
  - IP address: `192.168.50.10`
  - Subnet mask: `255.255.255.0`
  - Default gateway: (blank — no internet routing needed on isolated lab network)
  - Preferred DNS: `192.168.50.10` (DC points to itself — becomes its own DNS server once AD DS is installed)

**Verified with:**
```
ipconfig /all
```
Confirmed static IP with no DHCP lease.

**Took snapshot:** `DC01 - clean install - pre AD DS`

---

## Phase 4 — Active Directory Domain Services Installation

### Adding the AD DS role

1. Server Manager → Manage → **Add Roles and Features**
2. Role-based or feature-based installation → Next
3. Selected server: `DC01` → Next
4. Checked **Active Directory Domain Services** → Add Features when prompted
5. Clicked Next through remaining screens → **Install**
6. Waited for "Installation succeeded" confirmation

### Promoting DC01 to Domain Controller

Clicked the yellow flag notification in Server Manager → **Promote this server to a domain controller**

**Configuration selected:**

| Setting | Value |
|---|---|
| Deployment operation | Add a new forest |
| Root domain name | `homelab.local` |
| Forest functional level | Windows Server 2016 |
| Domain functional level | Windows Server 2016 |
| DNS server | Checked (installs DNS role on DC01) |
| DSRM password | Set (emergency recovery password — stored securely) |

Clicked through DNS Options (delegation warning is expected and normal in a new lab forest), NetBIOS name auto-filled as `HOMELAB`, left all default paths, ran prerequisites check — passed with expected warnings.

Clicked **Install** — server restarted automatically on completion.

### Verification after restart

Login screen changed from `Administrator` to `HOMELAB\Administrator` — confirms domain is active.

```
# Verified AD DS installation
Server Manager → Tools → Active Directory Users and Computers
# Expanded homelab.local — confirmed Computers, Domain Controllers, Users containers present
# DC01 visible under Domain Controllers

# Verified DNS resolution
nslookup homelab.local
# Returned 192.168.50.10 — DNS resolving correctly
```

### Issue encountered — recurring BSOD during promotion (IRQL_NOT_LESS_OR_EQUAL)

During the first several promotion attempts, the VM hit a BSOD immediately after clicking Install in the promotion wizard. The error screen auto-restarted too quickly to read the stop code.

**Fix — disabled auto-restart to capture the error:**
- Right-click Start → System → Advanced system settings
- Startup and Recovery → Settings
- Unchecked **Automatically restart**
- Set memory dump to **Small memory dump**

With auto-restart disabled, the BSOD froze on screen long enough to capture the stop code: `IRQL_NOT_LESS_OR_EQUAL`

**What this error means:** A process attempted to access memory at an interrupt request level higher than it was permitted to use. In a VM context this typically indicates resource contention — the host machine being pushed hard during a CPU/RAM-intensive operation.

**What was happening:** AD DS promotion is one of the most resource-intensive operations a Windows Server VM performs — it writes the entire AD database, configures DNS, sets up SYSVOL and NETLOGON shares, and rewrites the boot configuration simultaneously. The host laptop had other applications open and competing for RAM and CPU at the same moment.

**Fix applied:**
- Closed all non-essential applications on the host machine before attempting promotion
- Reduced DC01 VM RAM from 4096 MB to 3072 MB to leave more headroom for the host OS
- Disabled VM auto-restart to reduce load spike at completion

Promotion completed successfully on the next attempt with these adjustments in place.

**Lesson:** VM resource allocation is a two-sided equation — giving a VM more RAM can sometimes cause problems if it leaves the host OS starved during peak operations. Leaving headroom on the host matters especially during intensive operations like OS promotion, large file operations, or heavy compilation tasks.

**Took snapshot:** `DC01 - AD DS complete - homelab.local - clean baseline`

---

## Phase 5 — Domain Join (WIN11-01)

### Network configuration on WIN11-01

- Powered off WIN11-01
- VMware Settings → Network Adapter → changed to **LAN Segment: labnet**
- Powered on WIN11-01
- Set static IP via IPv4 Properties:
  - IP address: `192.168.50.20`
  - Subnet mask: `255.255.255.0`
  - Default gateway: (blank)
  - Preferred DNS: `192.168.50.10`

### Verified connectivity between VMs

Both VMs running simultaneously:

```
# On WIN11-01
ping 192.168.50.10
# Received 4 replies — WIN11-01 can reach DC01

nslookup homelab.local
# Returned 192.168.50.10 — DNS resolution working from client
```

```
# On DC01
ping 192.168.50.20
# Received 4 replies — DC01 can reach WIN11-01
```

### Issue encountered — IP address skew

Initial ping from WIN11-01 to DC01 failed. Running `ipconfig /all` on WIN11-01 showed an APIPA address (`169.254.x.x`) instead of `192.168.50.20`.

**What APIPA means:** When a Windows machine is configured for DHCP but cannot locate a DHCP server, it self-assigns an Automatic Private IP Addressing address in the `169.254.0.0/16` range. This address is only meaningful on the local machine — it cannot communicate with other devices. Since `labnet` has no DHCP server, any VM not set to a manual static IP will fall back to APIPA.

**Fix:** Re-entered static IP settings manually in IPv4 Properties, confirmed with `ipconfig /all`, pinged successfully.

**Lesson:** APIPA addresses starting with `169.254` are a reliable indicator that a machine either has no network connection or cannot reach a DHCP server. Recognizing this pattern immediately narrows troubleshooting to network connectivity or DHCP configuration.

### Joining the domain

1. Right-click Start → System → Advanced system settings
2. Computer Name tab → **Change**
3. Member of → **Domain** → typed `homelab.local`
4. Clicked OK
5. Credentials prompt — entered `Administrator` with DC01 Administrator password
6. Received confirmation: **"Welcome to the homelab.local domain"**
7. Restarted WIN11-01

### Issue encountered — incorrect credential format at domain login

After restart, attempted to log in via Other User with `HOMELAB\Administrator`. Login failed repeatedly with incorrect password error.

**Cause 1 — Wrong slash direction:** Was entering `HOMELAB/Administrator` (forward slash) instead of `HOMELAB\Administrator` (backslash). Windows domain credential format requires backslash between domain and username. Forward slash is not recognized.

**Cause 2 — Account lockout:** Repeated failed login attempts triggered the default domain account lockout policy, locking the Administrator account.

**Fix — unlocked account from DC01:**

```
# In Active Directory Users and Computers on DC01
# homelab.local → Users → right-click Administrator → Properties → Account tab
# Checked "Unlock account" → Apply → OK

# Also ran from elevated Command Prompt on DC01:
net user Administrator /active:yes
net accounts /lockoutthreshold:0
```

Successfully logged into WIN11-01 as `HOMELAB\Administrator` after unlock.

**Lesson:** The backslash vs forward slash distinction is a common source of domain login failures for end users and new technicians alike. Domain format is always `DOMAIN\username` — this comes up constantly in real helpdesk work. Account lockout from repeated failed attempts is one of the most frequent helpdesk tickets in any Active Directory environment.

### Verified domain join on DC01

```
# Active Directory Users and Computers → homelab.local → Computers
# WIN11-01 visible in the Computers container — domain join confirmed
```

**Took snapshots of both VMs:**
- `DC01 - AD DS complete - homelab.local - clean baseline`
- `WIN11-01 - domain joined - clean baseline`

---

## Final Lab State

```
labnet (VMware LAN Segment - isolated, no internet)
│
├── DC01 (192.168.50.10)
│   ├── Windows Server 2022 Standard Evaluation (Desktop Experience)
│   ├── Active Directory Domain Services
│   ├── DNS Server
│   └── Domain: homelab.local
│
└── WIN11-01 (192.168.50.20)
    ├── Windows 11 Pro
    ├── Domain member: homelab.local
    └── DNS: 192.168.50.10
```

---

## Issues Encountered Summary

| Issue | Cause | Fix |
|---|---|---|
| VirtualBox — no bootable device loop | Suspected configuration diff vs working VMs; never isolated | Switched to VMware Workstation Pro |
| VMware Easy Install — license terms error | `autoinst.flp` floppy interfering with installer | Removed floppy lines from .vmx file |
| BSOD during DC promotion (IRQL_NOT_LESS_OR_EQUAL) | Host resource contention during intensive AD DS write operations | Closed host apps, reduced VM RAM to 3072 MB, disabled auto-restart |
| Win11 ping failure — APIPA address | Static IP not saved correctly, VM fell back to APIPA | Re-entered static IP settings manually |
| Domain login failure — incorrect password | Wrong slash direction (/ instead of \) + account lockout from repeated failures | Corrected to HOMELAB\Administrator format, unlocked account via ADUC on DC01 |

---

## Lessons Learned

- VMware Workstation Pro (now free for personal use) is significantly more reliable than VirtualBox for Windows Server and Windows 11 ISO installations on modern hardware
- VMware Easy Install should be bypassed for Server ISOs — the `autoinst.flp` floppy file is the specific artifact to remove from the .vmx file
- A domain controller must have a static IP before AD DS installation — DHCP and domain controllers are incompatible in practice
- The DC's preferred DNS must point to itself (`192.168.50.10`) — this is what allows it to resolve the domain name it's authoritative for
- APIPA addresses (`169.254.x.x`) immediately indicate no DHCP server reachable — recognize this pattern and go straight to checking static IP config
- Domain credential format is always `DOMAIN\username` with a backslash — one of the most common end-user login issues in any AD environment
- VM resource allocation affects the host as much as the guest — leaving headroom on the host matters during intensive operations
- Disabling auto-restart on BSOD is standard practice for capturing stop codes during troubleshooting
- Snapshot before every major operation — DC promotion and domain join are both points of no easy return without one

---

## CompTIA A+ Domain Relevance

- **Domain 1.0 — Mobile Devices / Hardware:** Virtual hardware configuration (RAM, CPU, storage controllers, network adapters) directly mirrors physical hardware concepts tested in the A+ Core 1 exam
- **Domain 2.0 — Networking:** Static IP assignment, DNS configuration, subnet masks, default gateways, ping/nslookup diagnostics, APIPA recognition — all Core 1 networking objectives
- **Domain 3.0 — Hardware:** Virtualization concepts, hypervisor types (Type 1 vs Type 2), VM resource allocation — Core 1 virtualization and cloud computing objectives
- **Domain 4.0 — Virtualization and Cloud Computing:** Direct hands-on experience with Type 2 hypervisor (VMware Workstation Pro), VM creation, snapshot management, virtual networking
- **Domain 5.0 — Operating Systems:** Windows Server roles and features, Active Directory, domain join process, local vs domain accounts — Core 2 Windows objectives
- **Domain 5.7 — Troubleshooting:** BSOD diagnosis and stop code capture, systematic elimination of variables, documentation of dead ends alongside fixes — Core 2 troubleshooting methodology

---

## Next Steps

This lab is the foundation for a 14-project curriculum covering:

- **Track 1 — Active Directory (AD-01 through AD-04):** Users, groups, OUs, Group Policy, account troubleshooting
- **Track 2 — Helpdesk Scenarios (HD-01 through HD-04):** Onboarding/offboarding, printers, remote support, software deployment
- **Track 3 — Network Troubleshooting (NET-01 through NET-03):** IP/DNS diagnostics, firewall rules, file shares and SMB
- **Track 4 — Sysadmin Skills (SYS-01 through SYS-03):** PowerShell automation, Event Viewer and log analysis, backup and recovery
