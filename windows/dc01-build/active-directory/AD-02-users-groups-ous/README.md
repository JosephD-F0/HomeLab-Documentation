# AD-02: Users, Groups & Organizational Units
 
## Scenario
 
First week on the job. Manager has requested the Active Directory structure be built out for Homelab Corp before any users begin onboarding. Task involves creating a department-based OU hierarchy, provisioning user accounts for all new hires via both GUI and PowerShell, creating security groups for each department, and verifying everything is correctly organized and functional before users begin logging in.
 
---
 
## Environment
 
| Component | Details |
|---|---|
| Server | `DC01` — Windows Server 2022 Standard Evaluation (Desktop Experience) |
| Domain | `homelab.local` |
| DC IP | `192.168.50.10` (static) |
| Client | `WIN11-01` — `192.168.50.20` — domain joined |
| Hypervisor | VMware Workstation Pro |
 
---
 
## Company Structure Built
 
| Department | Users |
|---|---|
| IT | jsmith (John Smith) |
| HR | mwilliams (Mary Williams), tjohnson (Tom Johnson) |
| Finance | rbrown (Rachel Brown), lgarcia (Luis Garcia) |
| Sales | dlee (David Lee), kmartinez (Karen Martinez) |
 
---
 
## Part 1 — OU Structure
 
Built nested OU structure under the base OUs created in AD-01. Each department received its own child OU under both HOMELAB_USERS and HOMELAB_GROUPS to keep user accounts and security groups organized separately.
 
**Created via Active Directory Users and Computers:**
- Right-click `HOMELAB_USERS` → New → Organizational Unit → repeated for each department
- Right-click `HOMELAB_GROUPS` → New → Organizational Unit → repeated for each department
**Final OU structure:**
```
homelab.local
├── HOMELAB_USERS
│   ├── IT
│   ├── HR
│   ├── Finance
│   └── Sales
├── HOMELAB_GROUPS
│   ├── IT
│   ├── HR
│   ├── Finance
│   └── Sales
├── HOMELAB_COMPUTERS
│   └── WIN11-01
└── HOMELAB_ADMINS
```
 
**Why separate OUs for users and groups:** Keeping user accounts and security groups in separate OUs allows Group Policy to be applied independently to each. A GPO linked to HOMELAB_USERS applies only to user objects. A GPO linked to HOMELAB_GROUPS applies only to group objects. Mixing them in the same OU creates ambiguity and makes targeted policy application harder as the environment grows.
 
---
 
## Part 2 — User Account Creation (GUI)
 
First two users created manually via Active Directory Users and Computers to practice the GUI workflow used for individual new hire provisioning.
 
**jsmith — IT Department:**
- HOMELAB_USERS → IT → right-click → New → User
- First name: `John` Last name: `Smith`
- User logon name: `jsmith`
- Password: `Homelab@123!`
- Password never expires: checked (lab environment)
- User must change password at next logon: unchecked
**mwilliams — HR Department:**
- HOMELAB_USERS → HR → right-click → New → User
- First name: `Mary` Last name: `Williams`
- User logon name: `mwilliams`
- Password: `Homelab@123!`
- Same settings as above
**Why use the GUI for individual users:** The GUI is appropriate for one-off account creation — a single new hire, an emergency account, or a situation where you need to carefully review each field. For bulk provisioning, PowerShell is faster and less error-prone.
 
---
 
## Part 3 — User Account Creation (PowerShell)
 
Remaining five users created via PowerShell to practice bulk provisioning. All commands entered as single lines due to clipboard/multiline limitations in the VM environment.
 
```powershell
# tjohnson - HR
New-ADUser -Name "Tom Johnson" -GivenName "Tom" -Surname "Johnson" -SamAccountName "tjohnson" -UserPrincipalName "tjohnson@homelab.local" -Path "OU=HR,OU=HOMELAB_USERS,DC=homelab,DC=local" -AccountPassword (ConvertTo-SecureString "Homelab@123!" -AsPlainText -Force) -Enabled $true -PasswordNeverExpires $true
 
# rbrown - Finance
New-ADUser -Name "Rachel Brown" -GivenName "Rachel" -Surname "Brown" -SamAccountName "rbrown" -UserPrincipalName "rbrown@homelab.local" -Path "OU=Finance,OU=HOMELAB_USERS,DC=homelab,DC=local" -AccountPassword (ConvertTo-SecureString "Homelab@123!" -AsPlainText -Force) -Enabled $true -PasswordNeverExpires $true
 
# lgarcia - Finance
New-ADUser -Name "Luis Garcia" -GivenName "Luis" -Surname "Garcia" -SamAccountName "lgarcia" -UserPrincipalName "lgarcia@homelab.local" -Path "OU=Finance,OU=HOMELAB_USERS,DC=homelab,DC=local" -AccountPassword (ConvertTo-SecureString "Homelab@123!" -AsPlainText -Force) -Enabled $true -PasswordNeverExpires $true
 
# dlee - Sales
New-ADUser -Name "David Lee" -GivenName "David" -Surname "Lee" -SamAccountName "dlee" -UserPrincipalName "dlee@homelab.local" -Path "OU=Sales,OU=HOMELAB_USERS,DC=homelab,DC=local" -AccountPassword (ConvertTo-SecureString "Homelab@123!" -AsPlainText -Force) -Enabled $true -PasswordNeverExpires $true
 
# kmartinez - Sales
New-ADUser -Name "Karen Martinez" -GivenName "Karen" -Surname "Martinez" -SamAccountName "kmartinez" -UserPrincipalName "kmartinez@homelab.local" -Path "OU=Sales,OU=HOMELAB_USERS,DC=homelab,DC=local" -AccountPassword (ConvertTo-SecureString "Homelab@123!" -AsPlainText -Force) -Enabled $true -PasswordNeverExpires $true
```
 
**Verified all seven users created:**
```powershell
Get-ADUser -Filter * -SearchBase "OU=HOMELAB_USERS,DC=homelab,DC=local" | Select-Object Name, SamAccountName, DistinguishedName
```
 
All seven users returned with correct OU paths confirmed in DistinguishedName field.
 
**What each parameter does:**
 
| Parameter | Purpose |
|---|---|
| `-Name` | Full display name shown in ADUC |
| `-SamAccountName` | The logon name used to authenticate — this is what users type at login |
| `-UserPrincipalName` | Email-style logon name — `user@domain.local` format |
| `-Path` | Exact OU location in distinguished name format — determines where in AD the object is created |
| `-AccountPassword` | Sets password using SecureString — required for account to be enabled |
| `-Enabled $true` | Activates the account immediately — without this the account is created but can't log in |
| `-PasswordNeverExpires $true` | Lab setting — in production accounts should have expiring passwords per policy |
 
---
 
## Part 4 — Security Group Creation
 
Created one security group per department plus an All-Staff parent group. Security groups are used to manage resource permissions at scale — assign permissions once to the group, then manage access by adding or removing members rather than editing individual permissions.
 
```powershell
# IT group
New-ADGroup -Name "IT-Staff" -GroupScope Global -GroupCategory Security -Path "OU=IT,OU=HOMELAB_GROUPS,DC=homelab,DC=local" -Description "IT Department Staff"
 
# HR group
New-ADGroup -Name "HR-Staff" -GroupScope Global -GroupCategory Security -Path "OU=HR,OU=HOMELAB_GROUPS,DC=homelab,DC=local" -Description "HR Department Staff"
 
# Finance group
New-ADGroup -Name "Finance-Staff" -GroupScope Global -GroupCategory Security -Path "OU=Finance,OU=HOMELAB_GROUPS,DC=homelab,DC=local" -Description "Finance Department Staff"
 
# Sales group
New-ADGroup -Name "Sales-Staff" -GroupScope Global -GroupCategory Security -Path "OU=Sales,OU=HOMELAB_GROUPS,DC=homelab,DC=local" -Description "Sales Department Staff"
 
# All-Staff parent group
New-ADGroup -Name "All-Staff" -GroupScope Global -GroupCategory Security -Path "OU=HOMELAB_GROUPS,DC=homelab,DC=local" -Description "All company staff"
```
 
**Group scope explanation:**
 
| Scope | Meaning |
|---|---|
| `Global` | Can contain users from the same domain — correct choice for department groups in a single-domain lab |
| `DomainLocal` | Used to assign permissions to resources — typically used on the resource side |
| `Universal` | Spans multiple domains in a forest — used in multi-domain enterprise environments |
 
**Group category explanation:**
 
| Category | Meaning |
|---|---|
| `Security` | Used to assign permissions to resources like folders, printers, and GPOs |
| `Distribution` | Used for email distribution lists only — cannot be used for permissions |
 
---
 
## Part 5 — Group Membership Assignment
 
```powershell
# Add users to department groups
Add-ADGroupMember -Identity "IT-Staff" -Members "jsmith"
Add-ADGroupMember -Identity "HR-Staff" -Members "mwilliams","tjohnson"
Add-ADGroupMember -Identity "Finance-Staff" -Members "rbrown","lgarcia"
Add-ADGroupMember -Identity "Sales-Staff" -Members "dlee","kmartinez"
 
# Add department groups to All-Staff parent group
Add-ADGroupMember -Identity "All-Staff" -Members "IT-Staff","HR-Staff","Finance-Staff","Sales-Staff"
```
 
**Verified group memberships:**
```powershell
Get-ADGroupMember -Identity "HR-Staff" | Select-Object Name, SamAccountName
Get-ADGroupMember -Identity "All-Staff" | Select-Object Name, SamAccountName
```
 
All members confirmed in correct groups. All-Staff returned all four department groups as members — nested group membership working correctly.
 
**Why nest department groups inside All-Staff:** This is called group nesting and it's standard enterprise AD design. Instead of adding every individual user to All-Staff manually, you add the department groups. When a new user joins HR and gets added to HR-Staff, they automatically inherit All-Staff membership too. When someone leaves and gets removed from their department group, they lose All-Staff access automatically. One change cascades correctly through the structure.
 
---
 
## Part 6 — Verification in Active Directory Users and Computers
 
Verified complete structure visually in ADUC:
 
| OU Path | Contents Verified |
|---|---|
| HOMELAB_USERS → IT | jsmith ✅ |
| HOMELAB_USERS → HR | mwilliams, tjohnson ✅ |
| HOMELAB_USERS → Finance | rbrown, lgarcia ✅ |
| HOMELAB_USERS → Sales | dlee, kmartinez ✅ |
| HOMELAB_GROUPS → IT | IT-Staff ✅ |
| HOMELAB_GROUPS → HR | HR-Staff ✅ |
| HOMELAB_GROUPS → Finance | Finance-Staff ✅ |
| HOMELAB_GROUPS → Sales | Sales-Staff ✅ |
 
Double-clicked each group → Members tab → confirmed correct users listed in each group.
 
---
 
## Part 7 — Domain User Login Test
 
Tested domain user login from WIN11-01 to confirm provisioned accounts are functional.
 
- Signed out of WIN11-01
- Login screen → Other user
- Username: `HOMELAB\mwilliams`
- Password: `Homelab@123!`
- Login successful — new user profile created on WIN11-01
Signed back out and returned to Administrator session.
 
**Why this matters:** Creating an account in AD and successfully logging in with it are two separate things. A disabled account, wrong password setting, or missing group membership can all cause a login failure even when the account appears correctly configured in ADUC. Always test a login after provisioning.
 
---
 
## Issues Encountered
 
### PowerShell multiline command failures
 
**Symptom:** Commands using backtick (`` ` ``) line continuation threw errors when entered line by line in PowerShell.
 
**Cause:** PowerShell's backtick line continuation is extremely sensitive — a single space after the backtick breaks the command silently. Manually typing multiline commands across separate lines compounds this risk.
 
**Fix:** All commands entered as single continuous lines with no line breaks. This is more reliable in a VM environment where copy/paste behavior can be inconsistent.
 
**Paste method discovered:** Right-clicking inside the PowerShell window pastes clipboard content. Standard Ctrl+V does not work in PowerShell on Windows Server. If right-click paste is unavailable, enable QuickEdit Mode via PowerShell title bar → Properties → Options → QuickEdit Mode.
 
### Recurring BSODs during session
 
**Symptom:** Multiple BSODs occurred during AD-02 session interrupting work.
 
**Stop code:** `IRQL_NOT_LESS_OR_EQUAL`
 
**Status:** Under investigation. Floppy device was removed and VMware Tools was repaired in a previous session. BSODs are less frequent but still occurring intermittently. Event log analysis scheduled for next session to identify remaining root cause.
 
**Workaround applied:** Took snapshot before each major operation so work could be resumed from a clean state after each crash without losing progress.
 
---
 
## Snapshots Taken
 
| Snapshot Name | Timing |
|---|---|
| `DC01 - AD-02 complete - users groups OUs` | After full verification of all users, groups, and login test |
 
---
 
## Lessons Learned
 
- Creating users via PowerShell `New-ADUser` is significantly faster than the GUI for bulk provisioning — the `-Path` parameter places the user directly in the correct OU at creation time, eliminating the need to move objects afterward
- The `-Enabled $true` flag must be explicitly set when creating users via PowerShell — accounts created without it are disabled by default and cannot log in even with correct credentials
- Security groups should always be `Global` scope and `Security` category for department-level access management in a single-domain environment
- Group nesting (department groups inside a parent All-Staff group) is the correct enterprise design pattern — permissions cascade automatically when users are added to or removed from department groups
- Always test a domain user login from a client machine after provisioning — a successful account creation in ADUC does not guarantee a successful login
- Right-click pastes into PowerShell on Windows Server — Ctrl+V does not work
- Snapshots before each major operation are essential when working in an unstable VM — they allow work to resume from a clean point after unexpected crashes without losing completed steps
- DistinguishedName output from `Get-ADUser` confirms exact OU placement — always verify this after bulk creation rather than assuming the `-Path` parameter was accepted correctly
---
 
## CompTIA A+ / Network+ Domain Relevance
 
- **Domain 2.0 — Networking:** Understanding how domain user accounts authenticate over the network — Kerberos ticket requests to DC01 during WIN11-01 login
- **Domain 4.0 — Virtualization:** Snapshot strategy for maintaining lab stability during multi-step projects
- **Domain 5.0 — Operating Systems:** Active Directory Users and Computers navigation, user account properties, group types and scopes, OU design principles
- **Domain 5.3 — Windows OS Features:** Domain vs local accounts, user provisioning, group membership and its effect on resource access
- **Domain 5.7 — Troubleshooting:** PowerShell error diagnosis, BSOD pattern recognition, workaround implementation to maintain productivity during unresolved VM instability
---
 
## Next Steps
 
- **Immediate:** Investigate and resolve recurring DC01 BSODs via event log analysis before proceeding further
- **AD-03:** Group Policy Objects — enforce password policies, map drives, restrict access, and push settings to department OUs using the user and group structure built in this project
