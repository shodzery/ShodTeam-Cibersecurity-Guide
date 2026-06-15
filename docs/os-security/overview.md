# Operating System Security

## Overview

Operating system hardening reduces the attack surface of servers and workstations by removing unnecessary components, applying least privilege configurations, enabling security logging, and deploying monitoring. A hardened OS baseline is the foundation upon which application and network security controls operate.

---

## Linux Security Hardening

### Filesystem Security

```bash
# Check for world-writable files (potential backdoor locations)
find / -xdev -type f -perm -0002 2>/dev/null

# Check for SUID/SGID binaries (privilege escalation paths)
find / -xdev \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null

# Verify critical file permissions
ls -la /etc/passwd /etc/shadow /etc/sudoers /etc/crontab
# Expected:
# /etc/passwd: -rw-r--r-- root root (644)
# /etc/shadow: -rw-r----- root shadow (640)
# /etc/sudoers: -r--r----- root root (440)

# Mount options for security
# Add to /etc/fstab for appropriate partitions:
# noexec: prevents execution of binaries from partition
# nosuid: prevents SUID bit on partition
# nodev: prevents device files on partition
# /tmp: defaults,noexec,nosuid,nodev
# /var/tmp: defaults,noexec,nosuid,nodev
# /home: defaults,nosuid,nodev
```

### User and Account Management

```bash
# Remove unnecessary accounts
# Identify accounts with login shells that should not have them
awk -F: '($7 != "/sbin/nologin" && $7 != "/bin/false") {print $1, $7}' /etc/passwd

# Lock unused accounts
passwd -l username
usermod -s /sbin/nologin username

# Check for accounts with empty passwords (should return empty)
awk -F: '($2 == "" || $2 == "!") {print $1}' /etc/shadow

# Check sudo configuration
visudo
# Best practice: Use specific commands, not ALL
# user hostname = (root) NOPASSWD: /usr/bin/systemctl restart nginx
# Not: user ALL=(ALL) NOPASSWD: ALL

# Password aging
# /etc/login.defs
PASS_MAX_DAYS 90
PASS_MIN_DAYS 1
PASS_WARN_AGE 14
PASS_MIN_LEN 14  # Minimum length

# Apply to existing accounts
chage -M 90 -m 1 -W 14 username
```

### SSH Hardening

```bash
# /etc/ssh/sshd_config - security-focused configuration

# Disable password authentication (use SSH keys only)
PasswordAuthentication no
ChallengeResponseAuthentication no

# Disable root login
PermitRootLogin no

# Allow only specific users
AllowUsers adminuser deployuser

# Use SSH protocol 2 only (default in modern OpenSSH)
# Protocol 2

# Limit authentication attempts
MaxAuthTries 3

# Set login timeout
LoginGraceTime 30

# Disable X11 forwarding (unless required)
X11Forwarding no

# Disable TCP forwarding (unless required)
AllowTcpForwarding no

# Enable verbose logging
LogLevel VERBOSE

# Restrict SSH to specific network interfaces if applicable
# ListenAddress 10.0.0.1

# Set idle timeout (disconnect after 5 minutes of inactivity)
ClientAliveInterval 300
ClientAliveCountMax 0

# Reload after changes
systemctl reload sshd
```

### Kernel Security Parameters (sysctl)

```bash
# /etc/sysctl.d/99-security.conf

# Network security
net.ipv4.ip_forward = 0               # Disable IP forwarding (unless router)
net.ipv4.conf.all.send_redirects = 0  # Disable ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.rp_filter = 1       # Enable reverse path filtering
net.ipv4.icmp_echo_ignore_broadcasts = 1  # Ignore broadcast pings
net.ipv4.tcp_syncookies = 1           # Enable SYN cookies (SYN flood protection)
net.ipv4.conf.all.log_martians = 1   # Log spoofed/source-routed packets

# IPv6 (disable if not used)
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

# Kernel security
kernel.randomize_va_space = 2         # Full ASLR
kernel.dmesg_restrict = 1             # Restrict dmesg to root
kernel.kptr_restrict = 2              # Hide kernel pointers
kernel.yama.ptrace_scope = 1          # Restrict ptrace (anti-debugger)
kernel.sysrq = 0                      # Disable SysRq key

# Apply immediately
sysctl -p /etc/sysctl.d/99-security.conf
```

### Linux Auditing (auditd)

```bash
# Install and enable auditd
apt-get install auditd audispd-plugins
systemctl enable --now auditd

# /etc/audit/rules.d/security.rules

# Monitor administrative access
-w /etc/sudoers -p wa -k sudoers_change
-w /etc/sudoers.d/ -p wa -k sudoers_change
-w /etc/passwd -p wa -k password_change
-w /etc/shadow -p wa -k shadow_change
-w /etc/group -p wa -k group_change

# Monitor authentication
-w /var/log/auth.log -p wa -k auth_log
-w /var/run/faillock -p wa -k faillock

# Monitor privileged command execution
-a always,exit -F arch=b64 -S execve -F euid=0 -k root_exec

# Monitor SSH key management
-w /root/.ssh -p wa -k root_ssh
-w /home -p wa -k user_ssh

# Monitor cron modifications
-w /etc/cron.d/ -p wa -k cron_change
-w /etc/cron.daily/ -p wa -k cron_change
-w /etc/crontab -p wa -k cron_change
-w /var/spool/cron/crontabs -p wa -k cron_change

# Monitor kernel module loading
-w /sbin/insmod -p x -k module_load
-w /sbin/rmmod -p x -k module_remove
-w /sbin/modprobe -p x -k module_load
-a always,exit -F arch=b64 -S init_module -k module_load

# Make audit rules immutable (requires reboot to change)
-e 2

# Reload
augenrules --load
```

### AppArmor and SELinux

**AppArmor** (Ubuntu, Debian, SUSE) and **SELinux** (RHEL, CentOS, Fedora) implement Mandatory Access Control (MAC) that restricts what processes can do regardless of the user running them.

```bash
# AppArmor status
aa-status

# Check profiles
ls /etc/apparmor.d/

# Put a profile in enforce mode
aa-enforce /etc/apparmor.d/usr.sbin.nginx

# SELinux status
sestatus
getenforce  # Returns: Enforcing, Permissive, or Disabled

# Set SELinux to enforcing
setenforce 1
# Persist: edit /etc/selinux/config: SELINUX=enforcing

# View SELinux denials
ausearch -m avc -ts today
audit2why < /var/log/audit/audit.log
```

---

## Windows Security Hardening

### Local Security Policy

```powershell
# Set password policy via Group Policy or local policy
# Account Policies > Password Policy
# Minimum password length: 14
# Password complexity: Enabled
# Maximum password age: 90 days
# Minimum password age: 1 day
# Password history: 24

# Account lockout
# Account lockout threshold: 5 invalid logon attempts
# Account lockout duration: 30 minutes
# Reset account lockout counter: 30 minutes

# Configure via secedit or Group Policy
secedit /export /cfg current.cfg
# Edit current.cfg, then apply:
secedit /configure /db secedit.sdb /cfg hardened.cfg /areas SECURITYPOLICY
```

### Windows Defender and Security Features

```powershell
# Enable Windows Defender real-time protection
Set-MpPreference -DisableRealtimeMonitoring $false

# Enable cloud protection
Set-MpPreference -MAPSReporting Advanced
Set-MpPreference -SubmitSamplesConsent SendAllSamples

# Enable attack surface reduction (ASR) rules
# Block Office applications from spawning child processes
Set-MpPreference -AttackSurfaceReductionRules_Ids D4F940AB-401B-4EFC-AADC-AD5F3C50688A `
    -AttackSurfaceReductionRules_Actions Enabled

# Block process creations from PSExec and WMI commands
Set-MpPreference -AttackSurfaceReductionRules_Ids D1E49AAC-8F56-4280-B9BA-993A6D77406C `
    -AttackSurfaceReductionRules_Actions Enabled

# Enable Credential Guard (requires UEFI, Secure Boot)
# Protects LSASS with Virtualization-Based Security
bcdedit /set hypervisorlaunchtype auto
# Enable via Group Policy: Device Guard > Virtualization Based Security
```

### Windows Audit Policy

```powershell
# Configure advanced audit policy for security monitoring
# Essential event categories:

auditpol /set /category:"Account Logon" /success:enable /failure:enable
auditpol /set /category:"Account Management" /success:enable /failure:enable
auditpol /set /category:"Detailed Tracking" /success:enable  # Process creation (4688)
auditpol /set /category:"Logon/Logoff" /success:enable /failure:enable
auditpol /set /category:"Object Access" /success:enable /failure:enable
auditpol /set /category:"Policy Change" /success:enable /failure:enable
auditpol /set /category:"Privilege Use" /success:enable /failure:enable
auditpol /set /category:"System" /success:enable /failure:enable

# Enable command line logging for process creation (critical for detection)
# Group Policy: Administrative Templates > System > Audit Process Creation
# Enable: "Include command line in process creation events"

# Verify settings
auditpol /get /category:*
```

### Windows Firewall

```powershell
# Enable Windows Firewall on all profiles
Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled True

# Set default inbound action to block
Set-NetFirewallProfile -Profile Domain,Private,Public -DefaultInboundAction Block

# Allow specific inbound traffic
New-NetFirewallRule -DisplayName "Allow RDP from Management" `
    -Direction Inbound -Protocol TCP -LocalPort 3389 `
    -RemoteAddress "10.0.1.0/24" -Action Allow -Profile Domain

# Block SMB from internet-facing interfaces (should always be blocked at perimeter)
New-NetFirewallRule -DisplayName "Block SMB Outbound" `
    -Direction Outbound -Protocol TCP -RemotePort 445 `
    -RemoteAddress Internet -Action Block

# Export current rules
Get-NetFirewallRule | Export-Csv firewall_rules_baseline.csv
```

### Windows Event Log Configuration

```powershell
# Increase event log sizes (defaults are too small for security monitoring)
# Security log: increase to 1GB minimum
wevtutil sl Security /ms:1073741824

# Application log
wevtutil sl Application /ms:524288000

# System log
wevtutil sl System /ms:524288000

# PowerShell logging (critical for detecting malicious PowerShell)
# Enable Script Block Logging
$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
New-Item -Path $path -Force
Set-ItemProperty -Path $path -Name EnableScriptBlockLogging -Value 1

# Enable Module Logging
$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging"
New-Item -Path $path -Force
Set-ItemProperty -Path $path -Name EnableModuleLogging -Value 1

# Enable Transcription
$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription"
New-Item -Path $path -Force
Set-ItemProperty -Path $path -Name EnableTranscripting -Value 1
Set-ItemProperty -Path $path -Name OutputDirectory -Value "C:\PSTranscripts"
```

---

## Hardening Benchmarks and Baselines

Rather than building baselines from scratch, use established benchmarks from authoritative sources:

| Benchmark | Publisher | Coverage |
|-----------|-----------|---------|
| CIS Benchmarks | Center for Internet Security | Windows, Linux, macOS, cloud, network devices, containers |
| DISA STIGs | Defense Information Systems Agency | US federal systems; very prescriptive |
| NIST SP 800-70 | NIST | National Checklist Program |
| Microsoft Security Baselines | Microsoft | Windows, Office, Edge, Defender |

**CIS-CAT (CIS Configuration Assessment Tool)**: Automated tool to assess systems against CIS Benchmarks and generate compliance reports. Community edition available at no cost.

```bash
# Example: Run CIS-CAT against local system
./cis-cat-full/Assessor.sh -b benchmarks/CIS_Ubuntu_Linux_22.04_LTS_Benchmark.xml
```
