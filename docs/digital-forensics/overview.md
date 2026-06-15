# Digital Forensics

## Principles and Legal Admissibility

Digital forensics is the application of scientific methods to collect, preserve, analyze, and present digital evidence. Evidence collected for legal proceedings must satisfy requirements of authenticity, integrity, and proper chain of custody.

**Locard's Exchange Principle** — every interaction leaves a trace. Applied to digital forensics: accessing a system always modifies it. The forensic examiner's goal is to minimize the modification of evidence while extracting maximum investigative value.

**Order of volatility**: Evidence from the most volatile sources must be collected first, as this data is lost when power is removed or the system reboots.

---

## Chain of Custody

The chain of custody is the chronological documentation establishing the seizure, custody, control, transfer, and disposition of evidence. Without it, evidence may be inadmissible in legal proceedings.

**Documentation requirements for each evidence item:**
- Unique evidence identifier or tag number
- Case number
- Description of item (make, model, serial number, physical description)
- Date and time of collection
- Location of collection (physical address, system name, IP)
- Collector's name and role
- Reason for collection
- Condition of item at collection
- Cryptographic hash (SHA-256) of acquired image taken immediately after collection
- All subsequent transfers: who received it, when, why, hash verification

**Chain of custody form fields:**

```
Evidence Item:    EV-001
Case Number:      IR-2024-0042
Description:      HP EliteBook 840 G9, S/N 5CD2345678, 16GB RAM, 512GB NVMe
Collected by:     Jane Smith, Senior Forensic Examiner
Date/Time:        2024-03-15 14:32:00 UTC
Location:         Chicago Office, Floor 12, Workstation C-122
SHA-256 (disk):   e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
Reason:           Suspected ransomware infection, malware analysis required

Transfer Log:
  2024-03-15 16:00  Jane Smith → Forensic Lab Locker #7 (sealed, signed)
  2024-03-16 09:15  Forensic Lab Locker → Forensic Workstation (Jane Smith)
  2024-03-20 14:30  Forensic Workstation → Evidence Storage Room (sealed)
```

---

## Evidence Acquisition

### Disk Imaging

A forensic disk image is a bit-for-bit copy of the original storage media, including unallocated space, deleted files, and slack space. The original must not be modified during acquisition.

**Write blockers**: Hardware or software devices that permit read-only access to storage media, preventing any writes that would modify evidence.

**Imaging tools:**
```bash
# dd - basic block-level copy
dd if=/dev/sdb of=/forensics/case001/disk.img bs=4096 conv=noerror,sync status=progress

# dcfldd - enhanced dd with hashing and progress
dcfldd if=/dev/sdb hash=sha256 hashlog=/forensics/case001/hash.txt \
    of=/forensics/case001/disk.img bs=4096

# FTK Imager (GUI, Windows) - also produces E01 format

# Guymager (Linux GUI)
guymager /dev/sdb

# Imaging with automatic verification
ewfacquire /dev/sdb -t /forensics/case001/disk -C "Case IR-2024-0042" -e "Jane Smith"
```

**Image formats:**

| Format | Features | Use Case |
|--------|----------|---------|
| RAW (.img, .dd) | Bit-for-bit copy, no metadata, large | Simple, compatible with all tools |
| E01 (Expert Witness) | Compressed, metadata, hash verification | Industry standard for legal proceedings |
| AFF4 | Modern, compressed, signed | Growing adoption, open standard |

**Hash verification:**
```bash
# Immediately after acquisition
sha256sum /forensics/case001/disk.img > /forensics/case001/disk.sha256
sha256sum -c /forensics/case001/disk.sha256  # Verify later

# Compare against original (using write-blocked original)
sha256sum /dev/sdb
# Must match the image hash
```

### Memory Acquisition

RAM contains currently running processes, decrypted data, network connections, cryptographic keys, and artifacts that do not persist to disk. Acquisition must occur before shutdown.

**Linux memory acquisition:**
```bash
# LiME (Linux Memory Extractor) - loadable kernel module
git clone https://github.com/504ensicsLabs/LiME
cd LiME/src && make
# Acquire to file
sudo insmod lime.ko "path=/forensics/case001/memory.lime format=lime"
# Acquire over network (avoids writing to suspect disk)
sudo insmod lime.ko "path=tcp:4444 format=lime"
# On collector machine:
nc suspect_ip 4444 > /forensics/case001/memory.lime
```

**Windows memory acquisition:**
```
- WinPmem: winpmem_mini_x64.exe /forensics/case001/memory.raw
- FTK Imager: File > Capture Memory
- Belkasoft RAM Capturer (works when other tools are blocked)
- Magnet RAM Capture
```

---

## Memory Forensics

The Volatility Framework is the primary open-source tool for memory forensics analysis.

### Volatility 3 Usage

```bash
# Identify OS and profile
python3 vol.py -f memory.lime isfinfo

# Running processes
python3 vol.py -f memory.lime windows.pslist
python3 vol.py -f memory.lime windows.pstree  # Tree view showing parent-child

# Look for hidden/injected processes
python3 vol.py -f memory.lime windows.psscan   # Scan for EPROCESS structures (finds hidden processes)
python3 vol.py -f memory.lime windows.cmdline  # Command lines for each process

# Network connections
python3 vol.py -f memory.lime windows.netstat
python3 vol.py -f memory.lime windows.netscan

# Detect process injection
python3 vol.py -f memory.lime windows.malfind   # Suspicious memory regions (executable + not image-backed)
python3 vol.py -f memory.lime windows.hollowfind  # Process hollowing

# DLL analysis
python3 vol.py -f memory.lime windows.dlllist --pid 1234
python3 vol.py -f memory.lime windows.modules   # Loaded kernel modules

# Extract files from memory
python3 vol.py -f memory.lime windows.dumpfiles --pid 1234
python3 vol.py -f memory.lime windows.filescan  # List files in memory

# Registry hives
python3 vol.py -f memory.lime windows.registry.hivelist
python3 vol.py -f memory.lime windows.registry.printkey --key "SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# Credentials (if hashdump is available)
python3 vol.py -f memory.lime windows.hashdump
```

### Key Memory Artifacts

| Artifact | Significance | Volatility Plugin |
|----------|-------------|-------------------|
| Running processes | Identifies active programs including malware | pslist, pstree |
| Network connections | Shows active and recently closed connections | netstat, netscan |
| Command line arguments | Reveals how processes were launched | cmdline |
| Process environment variables | May reveal C2 URLs, paths | envars |
| Loaded DLLs | Injected DLLs indicate code injection | dlllist, modules |
| Suspicious memory regions | Executable memory not backed by a file on disk | malfind |
| Decrypted strings | Malware may decrypt config in memory | strings + grep |
| Browser history/credentials | Cached in browser process memory | Various; dump and strings |

---

## Disk Forensics

### File System Analysis

**Mount image read-only for analysis:**
```bash
# Create loop device
losetup --read-only --show --find disk.img

# Mount read-only
mount -o ro,norecovery /dev/loop0 /mnt/evidence
# For NTFS
mount -o ro,noexec,nosuid,noatime -t ntfs-3g /dev/loop0 /mnt/evidence
```

**Filesystem tools:**
```bash
# Autopsy (GUI - The Sleuth Kit frontend)
# The Sleuth Kit (CLI)

# List files in a filesystem image
fls -r -l disk.img

# Recover deleted files
tsk_recover -e disk.img /forensics/recovered/

# Search for specific file types by magic bytes (carving)
foremost -t jpg,pdf,docx -i disk.img -o /forensics/carved/

# Scalpel - configurable file carving
scalpel -c /etc/scalpel.conf disk.img -o /forensics/scalpel/
```

### Windows Forensic Artifacts

**Registry hives** (located in `C:\Windows\System32\config\` and user profiles):

| Hive | Location | Key Artifacts |
|------|----------|--------------|
| SYSTEM | system32\config\SYSTEM | ComputerName, timezone, mounted devices, services, USB history |
| SOFTWARE | system32\config\SOFTWARE | Installed programs, run keys, user profile list, recent documents |
| SAM | system32\config\SAM | Local user accounts and password hashes |
| NTUSER.DAT | C:\Users\<username>\ | Per-user run keys, RecentDocs, TypedURLs, MRU lists |
| USRCLASS.DAT | AppData\Local\Microsoft\Windows\ | ShellBags (folder access history) |

**Windows event logs:**
```powershell
# Export all security events
wevtutil epl Security C:\forensics\Security.evtx

# Query with Get-WinEvent
Get-WinEvent -Path Security.evtx -FilterXPath "*[System[EventID=4624]]" |
    Select-Object TimeCreated, @{n='User';e={$_.Properties[5].Value}}, 
                  @{n='LogonType';e={$_.Properties[8].Value}},
                  @{n='Source';e={$_.Properties[18].Value}}
```

**Prefetch files** (Windows execution evidence):
```
C:\Windows\Prefetch\*.pf
Each .pf file records:
- Application name
- Execution count
- Last 8 execution timestamps
- Files and directories accessed during execution

Tool: PECmd.exe (Eric Zimmerman's Tools)
PECmd.exe -d C:\Windows\Prefetch --csv C:\forensics\prefetch.csv
```

**Browser artifacts:**
```
Chrome:
  History:  %LOCALAPPDATA%\Google\Chrome\User Data\Default\History (SQLite)
  Downloads: same database
  Cookies:  %LOCALAPPDATA%\Google\Chrome\User Data\Default\Network\Cookies
  Cache:    %LOCALAPPDATA%\Google\Chrome\User Data\Default\Cache\

Firefox:
  History:  %APPDATA%\Mozilla\Firefox\Profiles\*.default-release\places.sqlite
```

---

## Timeline Analysis

Timeline analysis reconstructs the sequence of events by correlating timestamps from multiple sources.

### MACB Timestamps (NTFS)

| Letter | Meaning | Modified by |
|--------|---------|------------|
| M | Modified ($STANDARD_INFORMATION and $FILE_NAME) | File content written |
| A | Accessed ($STANDARD_INFORMATION) | File read (may be disabled) |
| C | Changed (MFT entry changed) | Metadata changes |
| B | Born (created time) | File creation |

**Note**: Timestamps are frequently manipulated by attackers (timestomping). Cross-reference $STANDARD_INFORMATION timestamps with $FILE_NAME timestamps — they are stored independently and manipulation usually only targets $STANDARD_INFORMATION.

### Building a Timeline

```bash
# Plaso / log2timeline
log2timeline.py /forensics/case001/timeline.plaso disk.img

# Add memory artifacts
log2timeline.py --parsers mactime /forensics/case001/timeline.plaso \
    /forensics/case001/timeline.body

# Export to CSV for analysis
psort.py -o dynamic -w /forensics/case001/timeline.csv /forensics/case001/timeline.plaso

# Filter for specific time window
psort.py -o dynamic -w /forensics/case001/filtered.csv /forensics/case001/timeline.plaso \
    "date > '2024-03-14 00:00:00' AND date < '2024-03-16 00:00:00'"
```

**Timeline analysis approach:**
1. Establish known-good bookmarks (login times, documented changes, business events)
2. Identify the attack window based on initial findings
3. Work backward from impact (encrypted files, exfiltration) to identify initial access
4. Work forward from initial access to map full attacker activity
5. Document gaps: missing logs, suspicious time periods with no activity
