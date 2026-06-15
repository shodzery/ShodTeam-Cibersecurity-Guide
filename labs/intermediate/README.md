# Intermediate Labs

These labs assume completion of the beginner labs or equivalent knowledge. They require a working knowledge of networking, web application behavior, and basic command-line proficiency. Each lab is designed to develop skills directly applicable to real-world security operations and testing scenarios.

---

## Lab Index

| Lab | Topic | Time Estimate |
|-----|-------|---------------|
| [Lab 01: Active Directory Enumeration](#lab-01-active-directory-enumeration) | Red Team / IAM | 90 minutes |
| [Lab 02: SIEM Detection Rule Development](#lab-02-siem-detection-rule-development) | Detection Engineering | 120 minutes |
| [Lab 03: Web Application Exploitation with Burp Suite](#lab-03-web-application-exploitation-with-burp-suite) | Web Security | 120 minutes |
| [Lab 04: Memory Forensics with Volatility](#lab-04-memory-forensics-with-volatility) | Digital Forensics | 90 minutes |
| [Lab 05: Phishing Simulation Analysis](#lab-05-phishing-simulation-analysis) | Blue Team / Social Engineering | 60 minutes |

---

## Lab 01: Active Directory Enumeration

### Objective

Enumerate an Active Directory environment from a domain-joined machine or with valid domain credentials, identifying attack paths using BloodHound and standard AD query tools.

### Environment

- Kali Linux or Windows attack machine
- Domain-joined Windows lab environment (TryHackMe "Active Directory Basics", HackTheBox "Forest" or "Active")
- Tools: BloodHound, SharpHound, PowerView, impacket

### Instructions

**Step 1: Basic AD enumeration with built-in tools**

```powershell
# From a domain-joined Windows machine

# Domain information
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
net user /domain
net group /domain

# Domain controllers
nltest /dclist:domain.local
nslookup -type=SRV _ldap._tcp.dc._msdcs.domain.local

# Domain admins
net group "Domain Admins" /domain

# Users with Service Principal Names (Kerberoasting targets)
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName |
    Select Name, ServicePrincipalName

# Users without pre-authentication (AS-REP Roasting targets)
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} -Properties DoesNotRequirePreAuth |
    Select Name
```

**Step 2: PowerView enumeration**

```powershell
# Import PowerView (from PowerSploit or standalone)
Import-Module .\PowerView.ps1

# Domain information
Get-Domain
Get-DomainController

# User enumeration
Get-DomainUser | Select samaccountname, pwdlastset, lastlogon
Get-DomainUser -SPN  # Kerberoastable accounts

# Computer enumeration
Get-DomainComputer | Select name, operatingsystem, lastlogontimestamp

# Group membership
Get-DomainGroupMember "Domain Admins"

# Local admin rights: find computers where current user has local admin
Find-LocalAdminAccess

# Find sessions on domain computers (where users are logged in)
Find-DomainUserLocation -UserGroupIdentity "Domain Admins"

# ACL enumeration - find misconfigured permissions
Find-InterestingDomainAcl -ResolveGUIDs | 
    Where-Object {$_.ActiveDirectoryRights -match "GenericAll|GenericWrite|WriteOwner|WriteDacl"}
```

**Step 3: BloodHound collection and analysis**

```powershell
# Collect data using SharpHound
.\SharpHound.exe -c All --OutputDirectory C:\temp\bloodhound

# Or using BloodHound Python for remote collection
bloodhound-python -u username -p 'password' -d domain.local -ns 10.10.10.1 -c All
```

```
BloodHound Analysis:
1. Import the collected ZIP file into BloodHound
2. Run pre-built queries:
   - "Find all Domain Admins"
   - "Shortest Paths to Domain Admins"
   - "Find Principals with DCSync Rights"
   - "Find AS-REP Roastable Users"
   - "Find Kerberoastable Users"
3. Identify shortest attack paths from your current user to Domain Admin
4. Document each step in the attack path with the required technique
```

**Step 4: Impacket enumeration (from Linux, with credentials)**

```bash
# Enumerate shares
impacket-smbclient domain.local/user:password@10.10.10.1

# List users via LDAP
impacket-GetADUsers -all domain.local/user:password

# Find Kerberoastable accounts
impacket-GetUserSPNs domain.local/user:password -request

# AS-REP Roasting
impacket-GetNPUsers domain.local/ -usersfile users.txt -format hashcat -outputfile asrep_hashes.txt
```

### Analysis Questions

1. How many users with SPNs did you identify? What services do they represent?
2. What attack paths exist from your starting position to Domain Admin?
3. Were any ACL misconfigurations found that would allow privilege escalation?
4. What controls would prevent the enumeration techniques used here?

---

## Lab 02: SIEM Detection Rule Development

### Objective

Develop, test, and tune detection rules for common attack techniques using Splunk (or Elastic/Sigma) targeting MITRE ATT&CK techniques.

### Environment

- Splunk Free (8.0+) or Elastic Stack
- Windows Event Log data (can import sample logs from Boss of the SOC datasets)
- MITRE ATT&CK reference

### Instructions

**Step 1: Ingest test data**

```bash
# Download BOTS (Boss of the SOC) dataset
# https://github.com/splunk/botsv3

# Or generate test Windows events
# Import sample evtx files from GitHub threat detection repositories
```

**Step 2: Detect PowerShell encoded commands (T1059.001)**

```splunk
# Splunk Search Processing Language (SPL)

# Base search
index=windows_events EventCode=4688 
| where match(CommandLine, "(?i)(-enc|-EncodedCommand|-e\s)")

# Add context and alert fields
index=windows_events EventCode=4688
    NewProcessName="*\\powershell.exe" 
    CommandLine IN ("*-enc *", "*-EncodedCommand*", "*-e *")
| table _time, host, user, NewProcessName, CommandLine, ParentProcessName
| sort -_time

# Tune to reduce false positives
index=windows_events EventCode=4688
    NewProcessName="*\\powershell.exe"
    CommandLine IN ("*-enc *", "*-EncodedCommand*")
    NOT CommandLine IN ("*SCCM*", "*ConfigMgr*")
| stats count by host, user, ParentProcessName
| where count > 0
```

**Step 3: Detect credential dumping (T1003.001 - LSASS Memory)**

```splunk
# LSASS memory access - look for non-system processes reading LSASS
index=windows_events EventCode=10
    TargetImage="*\\lsass.exe"
    NOT SourceImage IN ("*\\MsMpEng.exe", "*\\csrss.exe", "*\\wininit.exe", "*\\services.exe")
| table _time, host, SourceImage, GrantedAccess, CallTrace
| sort -_time

# Access rights 0x1010 or 0x1410 commonly seen in mimikatz
index=windows_events EventCode=10 TargetImage="*\\lsass.exe"
    (GrantedAccess="0x1010" OR GrantedAccess="0x1410" OR GrantedAccess="0x1438")
| table _time, host, SourceImage, GrantedAccess
```

**Step 4: Detect scheduled task creation (T1053.005)**

```splunk
# Windows Event ID 4698 - Scheduled task created
index=windows_events EventCode=4698
| eval task_details=mvzip(TaskName, TaskContent, "|")
| table _time, host, SubjectUserName, TaskName
| sort -_time

# Look for tasks with suspicious paths or encoded commands
index=windows_events EventCode=4698
    (TaskContent="*cmd.exe*" OR TaskContent="*powershell*" OR TaskContent="*wscript*")
    (TaskContent="*\\AppData\\*" OR TaskContent="*\\Temp\\*" OR TaskContent="*\\Public\\*")
| table _time, host, SubjectUserName, TaskName, TaskContent
```

**Step 5: Statistical anomaly detection**

```splunk
# Detect unusual outbound connection volume (potential exfiltration)
index=firewall_logs action=allowed
| timechart span=1h sum(bytes_out) by dest_ip
| anomalydetection bytes_out

# Detect rare parent-child process relationships
index=windows_events EventCode=4688
| stats count by ParentProcessName, NewProcessName
| where count < 5
| sort count asc
```

**Step 6: Create alert**

```
1. In Splunk: Save your search as Alert
2. Schedule: Every 15 minutes (for operational rules)
3. Trigger condition: Number of results > 0
4. Alert action: Log event / email / webhook
5. Add suppression if needed to prevent alert storms
```

---

## Lab 03: Web Application Exploitation with Burp Suite

### Objective

Use Burp Suite Professional (or Community) to identify and exploit web application vulnerabilities through manual proxy interception, active scanning, and targeted exploit techniques.

### Environment

- Burp Suite (Community or Professional)
- OWASP WebGoat, PortSwigger Web Security Academy labs, or DVWA
- Firefox configured to use Burp proxy

### Instructions

**Step 1: Configure Burp and intercept traffic**

```
1. Start Burp Suite
2. Go to Proxy > Intercept: ensure intercept is OFF for browsing
3. Configure Firefox proxy: 127.0.0.1:8080
4. Install Burp CA certificate: browse to http://burpsuite and download
5. Install cert in Firefox: Preferences > Privacy & Security > Certificates > Import
```

**Step 2: Explore target application with Burp**

```
1. Browse the target application normally with intercept off
2. Review the Site Map in Target > Site map
3. Identify authentication endpoints, API calls, and parameter usage
4. Right-click interesting requests > Send to Repeater for manual testing
```

**Step 3: SQL injection with Burp Repeater**

```
1. Find a login form or search parameter
2. Intercept the request and send to Repeater (Ctrl+R)
3. In Repeater, modify parameters:

Original request:
POST /login HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded

username=admin&password=password

Modified (authentication bypass):
username=admin'--&password=anything

Modified (UNION injection - after confirming injection point):
username=1' UNION SELECT null,null--&password=x
username=1' UNION SELECT username,password FROM users--&password=x
```

**Step 4: Parameter tampering with IDOR**

```
1. Find a URL or parameter referencing a resource by ID:
   GET /api/orders/1234

2. In Repeater, test other IDs:
   GET /api/orders/1233
   GET /api/orders/1235
   GET /api/orders/1000

3. If you can access other users' orders: IDOR confirmed

4. Automate with Burp Intruder:
   - Send to Intruder
   - Mark §1234§ as payload position
   - Add payload list: numbers 1000-1300
   - Start attack
   - Sort by response length (different length = likely different content)
```

**Step 5: XSS identification**

```
In Repeater, test reflection points:
GET /search?q=<script>alert(1)</script>

Look for:
- Direct reflection in HTML: confirmed XSS
- Reflection in attribute: test ' onmouseover="alert(1)" x="
- Reflection in JavaScript: test ';alert(1)//

Check response:
- Is the script tag returned unencoded? = XSS
- Is it HTML encoded? = Likely mitigated
- Is it reflected elsewhere in the page? = Check all reflection points
```

**Step 6: Burp Scanner (Professional)**

```
1. Right-click target in site map
2. Scan > Active Scan
3. Configure scan type: Lightweight, Medium, or Thorough
4. Review findings in Dashboard > Issues
5. For each finding:
   - Read the severity and confidence
   - Use the provided evidence to confirm manually
   - Document remediation recommendation
```

---

## Lab 04: Memory Forensics with Volatility

### Objective

Analyze a memory image from a compromised system to identify malware, running processes, network connections, and injected code.

### Environment

- Kali Linux or Ubuntu with Python 3
- Volatility 3
- Memory image (download from MemLabs CTF or Volatility Foundation samples)

### Setup

```bash
# Install Volatility 3
pip3 install volatility3

# Or clone and install
git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3 && pip3 install -e .

# Download a sample memory image
# MemLabs: https://github.com/stuxnet999/MemLabs
# Volatility samples: https://github.com/volatilityfoundation/volatility/wiki/Memory-Samples
```

### Instructions

**Step 1: Initial image analysis**

```bash
MEMFILE="memory.raw"  # Adjust to your file

# Identify OS and profile information
python3 vol.py -f $MEMFILE windows.info

# Verify image integrity
python3 vol.py -f $MEMFILE windows.hashdump 2>&1 | head -5
```

**Step 2: Process analysis**

```bash
# List all processes
python3 vol.py -f $MEMFILE windows.pslist

# Tree view (parent-child relationships)
python3 vol.py -f $MEMFILE windows.pstree

# Hidden processes (compares pool scanning vs process list)
python3 vol.py -f $MEMFILE windows.psscan

# Compare pslist vs psscan to find hidden processes
python3 vol.py -f $MEMFILE windows.pslist | awk '{print $3}' | sort > /tmp/pslist.txt
python3 vol.py -f $MEMFILE windows.psscan | awk '{print $3}' | sort > /tmp/psscan.txt
diff /tmp/pslist.txt /tmp/psscan.txt
# Processes in psscan but not pslist = hidden
```

**Step 3: Network analysis**

```bash
# Active and recently closed network connections
python3 vol.py -f $MEMFILE windows.netstat
python3 vol.py -f $MEMFILE windows.netscan

# Look for:
# - Connections to unusual external IPs
# - Connections from unexpected processes (svchost, powershell)
# - Listening ports not expected on this system type
```

**Step 4: Process injection detection**

```bash
# Find suspicious memory regions (executable memory not backed by DLL)
python3 vol.py -f $MEMFILE windows.malfind

# Review output:
# - PID and process name
# - Memory region (virtual address)
# - PE header? (MZ signature = injected PE)
# - Hex dump showing shellcode patterns

# Dump suspicious memory regions for further analysis
python3 vol.py -f $MEMFILE windows.malfind --dump

# Analyze dumped files
file pid.*.dmp
strings pid.*.dmp | grep -i "http\|cmd\|powershell\|key\|pass"
```

**Step 5: Registry and persistence analysis**

```bash
# List registry hives in memory
python3 vol.py -f $MEMFILE windows.registry.hivelist

# Check run keys for persistence
python3 vol.py -f $MEMFILE windows.registry.printkey \
    --key "SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

python3 vol.py -f $MEMFILE windows.registry.printkey \
    --key "SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce"
```

**Step 6: Command history**

```bash
# Console history (commands typed in cmd.exe windows)
python3 vol.py -f $MEMFILE windows.cmdline

# PowerShell history
python3 vol.py -f $MEMFILE windows.cmdline | grep -i powershell

# Clipboard contents
python3 vol.py -f $MEMFILE windows.clipboard
```

### Analysis Questions

1. Did you find any hidden processes? What technique was used to hide them?
2. Are there network connections to suspicious external IPs?
3. What did malfind reveal? What type of code injection occurred?
4. Are there persistence mechanisms installed?
5. Based on your analysis, what did the attacker do on this system?

---

## Lab 05: Phishing Simulation Analysis

### Objective

Analyze a phishing email and associated infrastructure to extract IOCs, assess the attack technique, and develop defensive recommendations.

### Sample Phishing Email (for analysis)

```
From: IT-Security@microsoft-helpdesk.support
To: john.doe@targetcorp.com
Subject: [URGENT] Your Microsoft 365 account will be suspended in 24 hours
Date: Mon, 15 Jan 2024 09:22:13 +0000

Dear John Doe,

Our security systems have detected unusual activity on your Microsoft 365 account. 
To prevent suspension, verify your identity immediately:

https://microsoft-helpdesk.support/secure/verify?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9

This link expires in 24 hours. Failure to verify will result in account suspension.

Microsoft Security Team
```

### Instructions

**Step 1: Header analysis**

```bash
# Key email headers to analyze (use email headers analyzer):
# Received: from - trace routing
# Return-Path: - where bounces go (often differs from From:)
# DKIM-Signature: - verify if signed
# Authentication-Results: - SPF/DKIM/DMARC results
# X-Originating-IP: - original sending IP

# Use online tools:
# MXToolbox Email Header Analyzer
# Mail-tester.com
# Google Admin Toolbox: toolbox.googleapps.com/apps/messageheader/
```

**Step 2: Domain analysis**

```bash
# Investigate the sending/linked domain
whois microsoft-helpdesk.support
dig microsoft-helpdesk.support
dig mx microsoft-helpdesk.support
dig txt microsoft-helpdesk.support  # SPF, DMARC records?

# Check domain age (recently registered = high suspicion)
# Check VirusTotal, URLScan
curl "https://urlscan.io/api/v1/search/?q=domain:microsoft-helpdesk.support" \
    -H "API-Key: YOUR_KEY"

# Screenshot without visiting directly
# Use: urlscan.io, any.run URL analysis, VirusTotal
```

**Step 3: URL and link analysis**

```bash
# Decode URL parameters
echo "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9" | base64 -d
# This is a JWT header: {"alg":"HS256","typ":"JWT"}

# Expand shortened URLs safely (without visiting)
curl -I "https://suspicious-url.com/x" 2>/dev/null | grep Location:

# Submit URL to sandbox without visiting
# urlscan.io, any.run (URL mode), Hybrid Analysis
```

**Step 4: Identify social engineering techniques**

```
Analyze the email for:
1. Urgency: "account suspended in 24 hours" - creates panic
2. Authority: Impersonates Microsoft, a trusted brand
3. Fear: Threat of account loss
4. Action demand: Click a link immediately
5. Generic greeting OR targeted: Is it personalized?
6. Technical mismatch: "From: microsoft-helpdesk.support" - not microsoft.com
7. Grammar and formatting: Any inconsistencies?
```

**Step 5: Develop defensive recommendations**

```
Based on analysis, document:

1. Technical controls:
   - Email gateway rules to catch similar emails (sender domain pattern, subject keywords)
   - DMARC enforcement would have blocked/quarantined this (if SPF/DKIM fail)
   - URL filtering to block the phishing domain

2. IOCs to add to SIEM/firewall:
   - Domain: microsoft-helpdesk.support
   - IP: [resolved from dig]
   - URL pattern: /secure/verify?token=

3. Detection rules:
   - Alert on external emails with "Microsoft" in display name but non-microsoft.com sender domain
   - Alert on clicks to newly registered domains from corporate endpoints

4. User awareness:
   - What makes this email detectable?
   - What should the user do upon receiving it?
```
