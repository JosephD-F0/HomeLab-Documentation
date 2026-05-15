### Project 1: User Account Management

### Objective

Practice creating, configuring, managing, and troubleshooting local user accounts on a Windows 10 Pro machine using both GUI and administrative tools. Mirror real-world helpdesk account management scenarios.

### Tools Used

- Control Panel → User Accounts
- Computer Management (compmgmt.msc)
- Local Users and Groups
- Group Policy Editor (gpedit.msc)
- Windows Sign-in Screen

---

### Task 1 — Account Creation

**Actions Performed:**

- Created Standard user account: JohnDoe via Control Panel → User Accounts → Manage another account
- Created Administrator account: ITAdmin via the same method, then elevated to Administrator via Change account type
- Created both accounts alternatively using compmgmt.msc → Local Users and Groups → New User for professional practice

**Outcome:** Both accounts successfully created and verified. Standard vs Administrator account types confirmed and understood.

---

### Task 2 — Account Disable and Enable

**Actions Performed:**

- Opened compmgmt.msc → Local Users and Groups → Users
- Right-clicked JohnDoe → Properties → checked Account is disabled → Apply
- Navigated to Windows sign-in screen to observe result

**Observation:**
Disabling an account does not simply block login — it completely removes the account tile from the Windows sign-in screen. The account becomes entirely invisible to anyone at the login screen. This is expected Windows behavior.

**Real World Relevance:**
When a user reports their account has disappeared from the login screen entirely this indicates a disabled account rather than a deleted one. Resolution is re-enabling via compmgmt.msc.

**Re-enabled account:** unchecked Account is disabled → Apply → account tile reappeared on sign-in screen.

---

### Task 3 — UAC Testing (User Account Control)

**Actions Performed:**

- Logged into JohnDoe standard user account
- Attempted to download and install Google Chrome
- UAC prompt appeared requesting administrator credentials
- Cancelled the prompt to confirm Standard user cannot self-authorize installations

**Unexpected Finding — Per-User Installer Behavior:**
Despite cancelling the UAC prompt, Google Chrome successfully installed. Investigation revealed that Google Chrome uses a per-user installer that deploys to `C:\Users\JohnDoe\AppData\Local\Google\Chrome` rather than `C:\Program Files`. Per-user installations do not require administrator privileges because they only affect the individual user's profile rather than the system.

**Real World Implication:**
UAC alone is insufficient to fully prevent software installation in an enterprise environment. Organizations requiring complete application control must implement Group Policy software restriction policies or AppLocker to block per-user level installations in addition to system-wide UAC enforcement.

**CompTIA A+ Note:**
Know the difference between per-user installations (AppData) and system-wide installations (Program Files) and their relationship to UAC.

---

### Task 4 — NTFS Permissions and Access Denied

**Actions Performed:**

- While logged in as JohnDoe navigated to C:\Users in File Explorer
- Attempted to open C:\Users\ITAdmin folder
- Received Access Denied error as expected

**Outcome:** Confirmed that standard users cannot access other user profile folders by default due to NTFS permission restrictions. This is the root cause of the majority of Access Denied helpdesk tickets.

---

### Task 5 — Password Reset via compmgmt.msc

**Actions Performed:**

- Logged into ITAdmin
- Opened compmgmt.msc → Local Users and Groups → Users
- Right-clicked JohnDoe → Set Password → read and acknowledged the warning prompt
- Reset password to new value without knowledge of previous password

**Warning Note:**
Windows displays a warning during forced password resets advising that encrypted files, personal certificates, and stored passwords may become inaccessible. In enterprise environments this is handled more safely through Active Directory which maintains password history and encryption key continuity.

---

### Task 6 — Password Policy via Group Policy Editor

**Actions Performed:**

- Opened gpedit.msc (Group Policy Editor — available on Pro edition only)
- Navigated to Computer Configuration → Windows Settings → Security Settings → Account Policies → Password Policy
- Configured the following:
    - Minimum password length: 8 characters
    - Password must meet complexity requirements: Enabled

**Outcome:** Password policy enforced across all local accounts. This setting mirrors how IT administrators enforce password standards across corporate machines.

**CompTIA A+ Note:**
Group Policy Editor is only available on Windows Pro and Enterprise editions. This is a direct exam objective and a primary reason Windows Pro is standard in business environments.

---

### Task 7 — Account Lockout Policy

**Actions Performed:**

- In gpedit.msc navigated to Account Policies → Account Lockout Policy
- Set Account lockout threshold to 3 invalid logon attempts
- Windows automatically configured lockout duration and reset counter
- Signed out and deliberately entered incorrect password for JohnDoe 3 consecutive times
- Account locked as expected after third failed attempt
- Returned to ITAdmin → compmgmt.msc → JohnDoe Properties → unchecked Account is locked out → Apply
- Confirmed JohnDoe account accessible again

**Real World Relevance:**
Account lockout due to too many failed password attempts is one of the single most common helpdesk tickets in enterprise environments. The resolution is always the same — locate the user in the user management tool and uncheck the locked status. In enterprise environments this is performed in Active Directory Users and Computers rather than local Computer Management.

---

### Project 1 Summary
TaskTool UsedCompletedCreate Standard and Admin accountsControl Panel, compmgmt.msc✅
Disable and re-enable accountcompmgmt.msc✅
Test UAC as Standard userWindows installer, UAC prompt✅
Investigate per-user Chrome installFile Explorer, AppData✅
Test NTFS Access DeniedFile Explorer, C:\Users✅
Reset forgotten passwordcompmgmt.msc✅
Configure Password Policygpedit.msc✅
Configure and test Account Lockoutgpedit.msc, compmgmt.msc✅
