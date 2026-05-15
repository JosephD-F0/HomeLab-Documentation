Project 2 — File Permissions and NTFS Security
Date: May 2026
Status: ✅ Complete

Environment
ComponentDetailHost OSWindows 10Guest OSWindows 10 Pro (VirtualBox VM)Tools UsedFile Explorer, Security Tab, NTFS PermissionsAccounts UsedLabUser, ITAdmin, JohnDoe

Objective
Create a restricted folder simulating a real enterprise shared directory, configure NTFS permissions to control which users can access and modify its contents, test those permissions from a standard user account, and practice resolving Access Denied errors — one of the most common helpdesk tickets in any IT environment.

Background Knowledge
NTFS — New Technology File System — is the file system Windows uses on all modern drives. It has a built-in permissions system that controls exactly what each user or group can do with any file or folder on the system. Every file and folder has an Access Control List (ACL) attached to it that defines who can read, write, modify, or take full control of it.
In a real enterprise environment file permission errors account for a significant percentage of daily helpdesk tickets. A user calls saying "I can't open this folder" or "it says Access Denied" and the resolution almost always involves reviewing and correcting NTFS permissions.
The five core NTFS permission levels are:
PermissionWhat It AllowsFull ControlRead, write, modify, delete, change permissionsModifyRead, write, modify and delete filesRead and ExecuteOpen and run files but not change themReadOpen and view files onlyWriteCreate new files and write data but not read existing

Steps Performed
Step 1 — Create a Simulated Restricted Folder

Logged in as ITAdmin
Navigated to the Desktop
Created a new folder named HR_Files
Created a sample text file inside named employee-records.txt
Added placeholder content to the file to simulate real data

Why: In real environments sensitive departments like HR, Finance and Legal have folders that only specific users or groups are permitted to access. This simulates that exact scenario.

Step 2 — Configure NTFS Permissions on the Folder

Right-clicked HR_Files → Properties
Clicked the Security tab
Reviewed the existing permissions — by default the folder inherited permissions allowing all users broad access
Clicked Edit to modify permissions
Clicked Add → typed JohnDoe → clicked Check Names to validate → clicked OK
With JohnDoe selected configured the following permission:

✅ Read — Allowed
❌ Write — Not allowed
❌ Modify — Not allowed
❌ Full Control — Not allowed


Clicked Apply → OK

Why: This mirrors how IT configures folder access for employees who need to view files but must not be able to alter or delete them — common for shared reference documents, HR records, or finance reports.

Step 3 — Test Read Permission as JohnDoe

Signed out of ITAdmin
Logged in as JohnDoe
Navigated to the HR_Files folder on the Desktop
Successfully opened employee-records.txt and read its contents
Confirmed Read permission working as configured


Step 4 — Test Write Restriction as JohnDoe

While still logged in as JohnDoe attempted to edit and save changes to employee-records.txt
Windows displayed an Access Denied error — file could not be saved
Attempted to delete the file — received Access Denied again
Attempted to create a new file inside HR_Files — blocked

Outcome: All write and modify actions correctly blocked for JohnDoe. Read-only access confirmed working exactly as configured.

Step 5 — Test Access Denied on Another User's Profile

While logged in as JohnDoe opened File Explorer
Navigated to C:\Users
Attempted to open C:\Users\ITAdmin
Received immediate Access Denied error
Confirmed standard users cannot access other user profile directories by default

Why this matters: This is a question that appears directly on the CompTIA A+ exam and is a scenario helpdesk techs encounter constantly. Users sometimes attempt to browse other users' folders and receive Access Denied — understanding that this is NTFS permissions working correctly rather than a system error is important.

Step 6 — Remove Own Account from Permissions (Break It)

Returned to HR_Files → Properties → Security tab
Removed LabUser from the permissions list entirely
Attempted to access HR_Files while logged in as LabUser
Received Access Denied — locked out of own folder

Why: This simulates an accidental permission misconfiguration — a real scenario where an admin removes the wrong account from a folder's ACL and loses access.

Step 7 — Restore Access (Fix It)

Logged in as ITAdmin which retained Full Control
Navigated to HR_Files → Properties → Security tab
Clicked Edit → Add → typed LabUser → Check Names → OK
Granted Full Control to LabUser
Clicked Apply → OK
Logged back in as LabUser — access to HR_Files fully restored

Lesson: Always have at least one administrator account with Full Control on every folder. Losing access to your own resources due to permission misconfiguration is a real helpdesk scenario with a predictable fix.

Ticket Summary
StepActionResultCreate HR_Files folderDesktop → New Folder✅
CreatedConfigure NTFS permissionsSecurity tab → Add JohnDoe → Read only✅
AppliedTest Read as JohnDoeOpened employee-records.txt✅
AccessibleTest Write as JohnDoeAttempted to edit and save❌
Access Denied — correctTest Delete as JohnDoeAttempted to delete file❌
Access Denied — correctTest C:\Users\ITAdminNavigated as JohnDoe❌
Access Denied — correctRemove LabUser from ACLSecurity tab❌ 
LabUser locked outRestore LabUser permissionsITAdmin restored Full Control✅ 
Access restored

Key Takeaways

NTFS permissions are stored directly on the file system and control access at the folder and file level independently of who is logged in
Every file and folder has an Access Control List defining exactly what each user or group is permitted to do
Standard users receive Access Denied when attempting to write to Read-only folders — this is the system working correctly not a malfunction
C:\Users folders are protected by NTFS permissions by default preventing users from accessing each other's profile directories
Accidentally removing an account from a folder's ACL is a real misconfiguration scenario — always retain at least one admin account with Full Control on any folder you configure
Permission changes take effect immediately without requiring a reboot or logoff
