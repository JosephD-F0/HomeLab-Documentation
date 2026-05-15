Project 4 — Break the Network, Fix the Network
Date: May 2026
Status: ✅ Complete

Environment
ComponentDetailHost OSWindows 10Guest OSWindows 10 Pro (VirtualBox VM)Tools UsedControl Panel, PowerShell, Task ManagerCommands Usedping, ipconfig /release, ipconfig /renew

Objective
Simulate a common network misconfiguration by manually assigning an incorrect static IP address to the VM's network adapter, observe the resulting connectivity failure, and restore full network connectivity using standard helpdesk troubleshooting procedures.

Background Knowledge
Every device on a network needs a valid IP address to communicate. In most home and office environments IP addresses are assigned automatically by a DHCP server — the router hands out addresses to every device that connects. When a static IP is manually configured incorrectly the device can no longer communicate with the network or the internet because it is essentially telling the router "I'm at an address that doesn't exist."
This is one of the most common causes of sudden connectivity loss in real helpdesk environments.

Steps Performed
Step 1 — Verify Baseline Connectivity

Opened PowerShell
Ran ping 8.8.8.8 (Google's public DNS server) to confirm network was operational
Received successful replies confirming full connectivity


Step 2 — Assign a False Static IP Address

Navigated to Control Panel → Network and Sharing Center → Change Adapter Settings
Right-clicked the active network adapter → Properties
Selected Internet Protocol Version 4 (TCP/IPv4) → Properties
Manually configured the following static IP:

IP Address: 192.168.99.99
Subnet Mask: 255.255.255.0


Clicked OK


Step 3 — First Connectivity Test (Inconclusive Result)
Observation: Returned to PowerShell and ran ping 8.8.8.8 — connectivity was still showing as active.
Root Cause of This Result:
Windows caches the previous network configuration briefly after a change. The network adapter had not yet fully released the old valid IP and adopted the new fake static one. The ping was still riding on the previous connection state.
Lesson Learned:
After making network adapter changes always return to the adapter properties and click OK or Apply to fully validate/commit the change before testing. Changes are not always applied instantaneously and testing immediately after editing without confirming can produce misleading results.

Step 4 — Validate the Configuration Change

Returned to Control Panel → Network Adapter Properties → TCP/IPv4
Confirmed the static IP settings were saved and properly committed
Clicked OK to validate and fully apply the configuration


Step 5 — Second Connectivity Test (Expected Failure)

Returned to PowerShell
Ran ping 8.8.8.8
Request timed out — no connectivity

Outcome: Network connectivity confirmed broken. The fake static IP address 192.168.99.99 placed the adapter on a non-existent network segment with no valid gateway, preventing all outbound communication.

Step 6 — Restore Network Connectivity

Returned to Control Panel → Network Adapter Properties → TCP/IPv4
Switched from static IP back to Obtain an IP address automatically (DHCP)
Switched DNS to Obtain DNS server address automatically
Clicked OK to apply


Step 7 — Force IP Renewal via PowerShell

Opened PowerShell
Ran the following commands in order:

ipconfig /release
ipconfig /renew

ipconfig /release — drops the current IP address assignment from the adapter
ipconfig /renew — requests a fresh valid IP address from the DHCP server (router)
Ran ping 8.8.8.8 to confirm connectivity
Successful replies received — network fully restored


Additional Issue Encountered — Windows Explorer UI Crash
Symptom
During the lab session the Windows taskbar search bar (Windows Search) disappeared from the bottom of the screen unexpectedly.
Root Cause
Windows Explorer — the process responsible for rendering the taskbar, desktop, and search bar — crashed or became unresponsive. This is a known Windows behavior that can occur during periods of elevated system activity or resource pressure inside a VM.
Resolution

Opened Task Manager via Ctrl + Shift + Esc
Located Windows Explorer under the Processes tab
Right-clicked Windows Explorer → Restart
Taskbar and search bar reappeared immediately without requiring a full system reboot

Real World Relevance
Windows Explorer crashes are a common helpdesk complaint reported as "my taskbar disappeared" or "my desktop icons are gone." The fix is always the same — restart the Windows Explorer process via Task Manager. This requires zero rebooting and resolves the issue in seconds.
