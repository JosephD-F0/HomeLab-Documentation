Project 5 — Blue Screen of Death (BSOD) Simulation and Recovery
Date: May 2026
Status: ✅ Complete

Environment
ComponentDetailHost OSWindows 10Guest OSWindows 10 Pro (VirtualBox VM)Tools UsedDevice Manager, NotMyFault64.exe, VirtualBox SnapshotsSysinternals ToolNotMyFault v4.0 (Microsoft Sysinternals)Crash Type SimulatedHigh IRQL Fault

Objective
Safely simulate a Windows Blue Screen of Death (BSOD) inside a controlled virtual machine environment using Microsoft's Sysinternals NotMyFault tool. Observe the BSOD behavior, read the resulting STOP error code, and perform a full system recovery using a previously saved VirtualBox snapshot.

Background Knowledge
A Blue Screen of Death is Windows' critical failure response. When the operating system encounters an error it cannot safely recover from — typically involving a driver, memory, or kernel-level conflict — it halts all processes, displays a STOP error code, and either reboots or waits for user action depending on system settings.
In a real helpdesk environment BSODs are commonly caused by:

Outdated or corrupt device drivers
Failing hardware such as RAM or hard drives
Incompatible software operating at the kernel level
Overheating causing hardware instability

Understanding how to read a STOP code and trace it back to a root cause is a core CompTIA A+ skill and a real-world helpdesk competency.

Steps Performed
Step 1 — Pre-Lab Snapshot

Before performing any potentially destructive actions a new snapshot was taken in VirtualBox
Snapshot name: Pre-BSOD Lab
This ensured a clean recovery point existed regardless of outcome
VM state confirmed stable before proceeding


Step 2 — Disable Display Adapter via Device Manager

Opened Device Manager via right-clicking Start → Device Manager
Expanded Display Adapters
Right-clicked the active display adapter → Disable device
Screen resolution changed immediately upon disabling confirming the driver was deactivated
This step simulated what happens when a display driver is removed or becomes corrupt — a common real world BSOD trigger


Step 3 — Configure Memory Dump Settings via Registry

Opened Command Prompt as Administrator
Executed the following registry command to ensure memory dump data would be retained after the crash:

REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\CrashControl" /v AlwaysKeepMemoryDump /t REG_DWORD /d 1 /f

This setting instructs Windows to always preserve the memory dump file after a BSOD rather than deleting it on reboot
In a real environment memory dumps are analyzed by IT staff to identify the exact driver or process that caused the crash


Step 4 — Download and Prepare NotMyFault

Downloaded NotMyFault from Microsoft Sysinternals
Extracted the zip file contents to the Desktop
Identified two executable versions inside the extracted folder:

NotMyFault.exe — 32-bit version
NotMyFault64.exe — 64-bit version


Selected NotMyFault64.exe appropriate for the 64-bit Windows 10 Pro environment
Right-clicked → Run as administrator to launch with full system privileges


Step 5 — Trigger the BSOD

Inside the NotMyFault interface located the crash type list
Selected High IRQL Fault — the most representative real world BSOD cause simulating a driver attempting to access memory at an improper interrupt request level
Clicked the Crash button
VM immediately displayed a full Blue Screen of Death as expected


Step 6 — Observe the BSOD Screen

BSOD displayed on screen with the following components visible:

A sad face emoticon — standard Windows 10 BSOD design
A STOP error code displayed on screen
A percentage counter indicating Windows was collecting error information
A reference to a memory dump being written



What the STOP Code Means:
The High IRQL Fault crash type generates an IRQL_NOT_LESS_OR_EQUAL stop code. This is one of the most common BSODs encountered in real environments and almost always points to a driver accessing memory it is not permitted to access. In real world troubleshooting this stop code directs an IT technician to investigate recently installed or updated drivers as the primary suspect.

Step 7 — Recovery via VirtualBox Snapshot

Allowed the BSOD screen to complete its memory dump process
Closed the VM window in VirtualBox
Opened Snapshots in VirtualBox
Selected the Pre-BSOD Lab snapshot
Clicked Restore
VM booted back to a fully clean and stable state with no trace of the crash
Confirmed all functionality restored — desktop, network, user accounts all intact


Where Memory Dumps Are Stored
In a real environment where snapshot recovery is not available the memory dump file written during the BSOD can be found at:
C:\Windows\Minidump
Full memory dumps are stored at:
C:\Windows\MEMORY.DMP
These files can be analyzed using WinDbg (Windows Debugger) to identify exactly which driver or process triggered the crash. This is standard procedure for recurring BSOD diagnosis in enterprise helpdesk environments.

Ticket Summary
StepActionResultPre-lab snapshotSnapshot taken in VirtualBox✅
Save point confirmedDisable display adapterDevice Manager✅
Resolution changed — driver removedConfigure memory dumpRegistry edit via CMD✅
Dump retention enabledLaunch NotMyFault64Run as administrator✅
Interface openedSelect crash typeHigh IRQL Fault selected✅
Ready to crashTrigger BSODClicked Crash button✅
BSOD displayed as expectedObserve STOP codeRead BSOD screen✅
IRQL_NOT_LESS_OR_EQUALRecoveryRestored Pre-BSOD snapshot✅ 
VM fully restored

Key Takeaways

A BSOD is not a random event — every crash has a specific STOP code that points directly to a category of failure making them diagnosable with the right tools and knowledge
The IRQL_NOT_LESS_OR_EQUAL stop code almost always indicates a driver attempting to access protected memory and directs troubleshooting toward recently installed or updated drivers
Memory dumps written during a BSOD are stored in C:\Windows\Minidump and contain detailed forensic information about what caused the crash
VirtualBox snapshots are the lab equivalent of a system restore point — they enable completely safe destructive testing with instant recovery
Disabling a display adapter in Device Manager immediately changes screen resolution confirming how tightly drivers are coupled to hardware functionality
In real enterprise environments recurring BSODs are analyzed using WinDbg to parse the minidump file and identify the offending driver by name
The NotMyFault tool is a legitimate Microsoft Sysinternals utility used by IT professionals to intentionally trigger crashes for testing and training purposes
