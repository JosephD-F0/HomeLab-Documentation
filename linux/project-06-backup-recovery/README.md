# Lab Project 06 — Backup & Recovery

**Date:** 2026-05-22
**Environment:** Ubuntu 26.04 LTS | VirtualBox | Host: Windows 10 Pro
**Objective:** Create a compressed backup of important files, simulate
accidental deletion, restore from backup, automate backups with cron,
and take a full system snapshot.

---

## Scenario

A user accidentally deleted their entire documents folder and needs it
recovered immediately. As the helpdesk tech you need to restore the files
from backup, verify the contents are intact, and then set up automated
backups so this situation can be recovered from quickly in the future.

---

## Steps Performed

### 1. Created test documents to back up
```bash
mkdir -p ~/documents/important
echo "This is file 1" > ~/documents/important/file1.txt
echo "This is file 2" > ~/documents/important/file2.txt
echo "This is file 3" > ~/documents/important/file3.txt
ls -la ~/documents/important/
```
- Created a documents folder simulating a real user's important files
- Verified all three files existed before proceeding with backup
- Establishing a known good state before backup is best practice

### 2. Created compressed backup archive
```bash
mkdir -p ~/backups
tar -czvf ~/backups/important_backup.tar.gz ~/documents/important/
```
- Created a dedicated backups folder to store archives
- Used tar to create a compressed gzip archive of the documents folder
- Flag breakdown:
  - `-c` = create new archive
  - `-z` = compress using gzip
  - `-v` = verbose, shows each file being added
  - `-f` = specifies the archive filename

### 3. Verified backup contents
```bash
ls -lh ~/backups/
tar -tzvf ~/backups/important_backup.tar.gz
```
- Confirmed archive file existed with correct size
- Listed archive contents without extracting to verify all files
  were captured correctly
- Always verify a backup before relying on it — a corrupt or
  incomplete backup is useless during a real recovery

### 4. Simulated accidental deletion
```bash
rm -rf ~/documents/important/
ls ~/documents/
ls ~/documents/important/
```
- Permanently deleted the entire important folder and all contents
- Confirmed deletion — important folder no longer existed
- `rm -rf` is irreversible in Linux — no recycle bin, files are
  gone instantly
- Second ls confirmed no such file or directory error

### 5. Restored files from backup
```bash
tar -xzvf ~/backups/important_backup.tar.gz -C ~/
```
- Extracted backup archive back to home directory
- Flag breakdown:
  - `-x` = extract files from archive
  - `-z` = decompress gzip
  - `-v` = verbose, shows each file being restored
  - `-f` = specifies the archive file
  - `-C ~/` = specifies extraction destination

### 6. Located and moved restored files
```bash
find ~/ -name "file1.txt" 2>/dev/null
cp -r /home/helpdesk1/home/helpdesk1/documents/important/ ~/documents/
ls -la ~/documents/important/
cat ~/documents/important/file1.txt
```
- Used find to locate where tar actually extracted the files
- Files were found at nested path due to tar path issue (see below)
- Copied files from nested location back to correct directory
- Verified all three files restored correctly
- Confirmed file contents were fully intact with cat command

### 7. Automated backup with cron
```bash
crontab -e
```
Added the following line:
```
0 2 * * * tar -czvf ~/backups/daily_backup.tar.gz ~/documents/important/
```
```bash
crontab -l
```
- Opened cron schedule editor using nano
- Added daily automated backup job scheduled for 2am every day
- Cron syntax breakdown:
  - `0` = minute 0
  - `2` = hour 2am
  - `* * *` = every day, every month, every day of week
- Verified job was saved correctly with `crontab -l`

### 8. Took full system snapshot
- Took VirtualBox snapshot named "Linux Projects Complete - All 6 Done"
- Captures entire VM state at this exact moment
- Allows instant rollback if anything goes wrong in future sessions

---

## Issues Encountered

**Problem:** After extracting the backup with `tar -xzvf` the files were
not found at `~/documents/important/` — ls returned no such file or directory.
**Cause:** tar stored the full absolute path inside the archive when it was
created. When extracting it recreated that full path from the extraction
destination — resulting in a nested path:
`/home/helpdesk1/home/helpdesk1/documents/important/`
instead of the expected:
`/home/helpdesk1/documents/important/`
**Diagnosis:** Used `find ~/ -name "file1.txt" 2>/dev/null` to locate
where the files actually extracted to — revealed the nested path immediately.
**Fix:** Used `cp -r` to copy the entire folder from the nested location
back to the correct directory:
```bash
cp -r /home/helpdesk1/home/helpdesk1/documents/important/ ~/documents/
```
**What this taught:** This is a well known tar behavior that trips up
real sysadmins regularly. The production fix is to use the
`--strip-components` flag during extraction to remove leading path
components and avoid nesting. Always use `find` to locate files when
an extraction doesn't go where expected.

---

## What I Learned

- `tar` is the standard Linux tool for creating and extracting
  compressed backup archives
- Always verify backup contents with `tar -tzvf` before relying
  on them — never assume a backup is good until you've checked it
- `rm -rf` is permanent and irreversible — there is no undo or
  recycle bin in the Linux terminal
- tar stores the full path when archiving which can cause nested
  extraction paths — always verify where files actually restored to
- `find` is essential for locating files when they don't appear
  where expected after an extraction
- cron automates repetitive tasks like backups so they happen
  reliably without depending on someone remembering to do it
- The difference between file level backup (tar) and system level
  backup (VM snapshot) — both serve different recovery purposes
- The 3-2-1 backup rule — 3 copies of data, on 2 different types
  of media, with 1 copy offsite — is the industry standard backup
  strategy
- A backup you have never tested is not a real backup — recovery
  testing is just as important as the backup itself

---

## CompTIA A+ Relevance

- Domain 1.9 — Explain features and tools of Linux operating systems
- Domain 4.1 — Summarize best practices associated with documentation
- Domain 4.2 — Explain basic change management best practices including
  backup and recovery procedures
- Domain 4.9 — Best practices for securing a workstation including
  backup and recovery
