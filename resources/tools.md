# Security Tools Reference

Organized by category. Includes open source and commercial tools commonly used in professional security operations, penetration testing, and forensic analysis.

---

## Reconnaissance and OSINT

| Tool | Type | Purpose | Source |
|------|------|---------|--------|
| Shodan | Commercial/Free tier | Internet-connected device search engine | shodan.io |
| Censys | Commercial/Free tier | Internet-wide scan data and certificate data | censys.io |
| theHarvester | Open source | Email, domain, and subdomain enumeration | github.com/laramies |
| Amass | Open source | Subdomain enumeration, asset discovery | github.com/owasp-amass |
| Subfinder | Open source | Fast subdomain discovery | github.com/projectdiscovery |
| SpiderFoot | Open source | OSINT automation platform | spiderfoot.net |
| Maltego | Commercial | Graphical link analysis and OSINT | maltego.com |
| recon-ng | Open source | Web reconnaissance framework | github.com/lanmaster53 |

---

## Network Scanning and Analysis

| Tool | Type | Purpose | Source |
|------|------|---------|--------|
| Nmap | Open source | Port scanner, service detection, NSE scripting | nmap.org |
| Masscan | Open source | High-speed TCP port scanner | github.com/robertdavidgraham |
| Wireshark | Open source | GUI packet analyzer | wireshark.org |
| tshark | Open source | CLI packet analyzer (Wireshark CLI) | wireshark.org |
| tcpdump | Open source | CLI packet capture | tcpdump.org |
| Zeek | Open source | Network analysis framework, threat detection | zeek.org |
| Suricata | Open source | IDS/IPS/NSM | suricata.io |
| Snort | Open source | IDS/IPS | snort.org |
| Netflow tools (ntopng, nfdump) | Open source | Flow analysis | ntop.org |

---

## Web Application Security

| Tool | Type | Purpose | Source |
|------|------|---------|--------|
| Burp Suite (Community/Pro) | Commercial/Free | Web application proxy and scanner | portswigger.net |
| OWASP ZAP | Open source | Web application scanner and proxy | zaproxy.org |
| sqlmap | Open source | Automated SQL injection detection/exploitation | sqlmap.org |
| Nikto | Open source | Web server scanner | cirt.net/Nikto2 |
| gobuster | Open source | Directory and DNS brute forcer | github.com/OJ |
| ffuf | Open source | Fast web fuzzer | github.com/ffuf |
| nuclei | Open source | Template-based vulnerability scanner | projectdiscovery.io |
| wfuzz | Open source | Web application fuzzer | github.com/xmendez |
| whatweb | Open source | Web application fingerprinting | morningstarsecurity.com |

---

## Exploitation Frameworks

| Tool | Type | Purpose | Source |
|------|------|---------|--------|
| Metasploit Framework | Open source/Commercial | Exploitation, payloads, post-exploitation | metasploit.com |
| Cobalt Strike | Commercial | Adversary simulation, C2 | cobaltstrike.com |
| Sliver | Open source | C2 framework, implants | github.com/BishopFox |
| Havoc | Open source | Modern C2 framework | github.com/HavocFramework |
| Empire | Open source | Post-exploitation framework (PowerShell/Python) | github.com/BC-SECURITY |
| impacket | Open source | Python classes for Windows protocols | github.com/fortra |

---

## Active Directory and Windows

| Tool | Type | Purpose | Source |
|------|------|---------|--------|
| BloodHound | Open source | AD attack path visualization | github.com/BloodHoundAD |
| SharpHound | Open source | BloodHound data collector | github.com/BloodHoundAD |
| PowerView | Open source | AD enumeration (PowerShell) | github.com/PowerShellMafia |
| CrackMapExec | Open source | SMB, LDAP, RDP, WMI pentesting | github.com/byt3bl33d3r |
| Responder | Open source | LLMNR/NBT-NS/mDNS poisoning | github.com/lgandx |
| Mimikatz | Open source | Credential extraction, Kerberos attacks | github.com/gentilkiwi |
| Rubeus | Open source | Kerberos interaction and attacks | github.com/GhostPack |
| Seatbelt | Open source | Windows security misconfiguration enumeration | github.com/GhostPack |
| WinPEAS / LinPEAS | Open source | Privilege escalation enumeration | github.com/carlospolop |

---

## Password Cracking

| Tool | Type | Purpose | Source |
|------|------|---------|--------|
| hashcat | Open source | GPU-accelerated password cracking | hashcat.net |
| John the Ripper | Open source | Password cracking, hash identification | openwall.com/john |
| CrackStation | Online | Hash lookup (pre-computed) | crackstation.net |
| Hydra | Open source | Network service brute force | github.com/vanhauser-thc |
| Medusa | Open source | Network service brute force | foofus.net |
| SecLists | Open source | Wordlist collection | github.com/danielmiessler |
| RockYou (wordlist) | Data | Common password wordlist | Kali: /usr/share/wordlists/ |

---

## Digital Forensics and Incident Response

| Tool | Type | Purpose | Source |
|------|------|---------|--------|
| Autopsy | Open source | Digital forensics GUI platform | autopsy.com |
| The Sleuth Kit | Open source | CLI forensic analysis tools | sleuthkit.org |
| Volatility 3 | Open source | Memory forensics framework | volatilityfoundation.org |
| KAPE | Free | Artifact collection and processing | kroll.com/kape |
| Eric Zimmerman's Tools | Free | Windows artifact analysis suite | ericzimmerman.github.io |
| Plaso / log2timeline | Open source | Super-timeline creation | github.com/log2timeline |
| FTK Imager | Free | Disk imaging, memory capture | exterro.com |
| Velociraptor | Open source | Endpoint visibility and DFIR | velocidex.com |
| GRR Rapid Response | Open source | Remote forensic investigation | github.com/google/grr |
| Timesketch | Open source | Collaborative forensic timeline | timesketch.org |

---

## Malware Analysis

| Tool | Type | Purpose | Source |
|------|------|---------|--------|
| Ghidra | Open source | Disassembler and decompiler (NSA) | ghidra-sre.org |
| IDA Pro | Commercial | Industry-standard disassembler | hex-rays.com |
| Binary Ninja | Commercial | Disassembler and analysis platform | binary.ninja |
| x64dbg | Open source | Windows debugger | x64dbg.com |
| OllyDbg | Free | 32-bit Windows debugger | ollydbg.de |
| Cuckoo Sandbox | Open source | Automated malware analysis | cuckoosandbox.org |
| CAPE Sandbox | Open source | Malware analysis with unpacking | github.com/kevoreilly |
| Any.run | Commercial/Free tier | Interactive malware sandbox | any.run |
| VirusTotal | Commercial/Free tier | Multi-AV scan and sandbox | virustotal.com |
| Detect-It-Easy (DIE) | Open source | File type and packer detection | github.com/horsicq |
| PE-sieve | Open source | Scan process memory for PE injection | github.com/hasherezade |
| YARA | Open source | Pattern matching for malware | github.com/VirusTotal/yara |

---

## SIEM and Log Management

| Tool | Type | Purpose | Source |
|------|------|---------|--------|
| Splunk | Commercial/Free | SIEM, log analysis platform | splunk.com |
| Elastic (ELK Stack) | Open source/Commercial | Log aggregation, search, visualization | elastic.co |
| Graylog | Open source/Commercial | Log management | graylog.org |
| Wazuh | Open source | SIEM, HIDS, XDR | wazuh.com |
| TheHive | Open source | Incident response platform | thehive-project.org |
| Cortex | Open source | Observable analysis engine (TheHive) | thehive-project.org |
| MISP | Open source | Threat intelligence sharing | misp-project.org |
| OpenCTI | Open source | Threat intelligence platform | filigran.io |
| Shuffle | Open source | SOAR workflow automation | shuffler.io |

---

## Cloud Security

| Tool | Type | Purpose | Source |
|------|------|---------|--------|
| ScoutSuite | Open source | Cloud security auditing | github.com/nccgroup |
| Prowler | Open source | AWS/GCP/Azure security auditing | github.com/prowler-cloud |
| CloudMapper | Open source | AWS environment visualization | github.com/duo-labs |
| Pacu | Open source | AWS exploitation framework | github.com/RhinoSecurityLabs |
| Stratus Red Team | Open source | Cloud attack technique emulation | github.com/DataDog |
| Trivy | Open source | Container/image vulnerability scanner | aquasecurity.github.io |
| Falco | Open source | Cloud-native runtime threat detection | falco.org |
| checkov | Open source | IaC security scanning | checkov.io |
| tfsec | Open source | Terraform security analysis | tfsec.dev |

---

## Vulnerability Management

| Tool | Type | Purpose | Source |
|------|------|---------|--------|
| Nessus | Commercial/Free (Essentials) | Vulnerability scanner | tenable.com |
| OpenVAS | Open source | Vulnerability scanner | openvas.org |
| Qualys | Commercial | Cloud-based vulnerability management | qualys.com |
| Rapid7 InsightVM | Commercial | Vulnerability management | rapid7.com |
| nuclei | Open source | Template-based scanning | projectdiscovery.io |
| Semgrep | Open source/Commercial | SAST, code analysis | semgrep.dev |
| OWASP Dependency Check | Open source | SCA, known vulnerable libraries | github.com/jeremylong |

---

## Linux Distributions for Security

| Distribution | Purpose | URL |
|-------------|---------|-----|
| Kali Linux | Penetration testing | kali.org |
| Parrot OS | Penetration testing | parrotsec.org |
| REMnux | Malware analysis | remnux.org |
| FLARE VM | Windows malware analysis | github.com/mandiant |
| SIFT Workstation | DFIR | github.com/teamdfir |
| Tails | Anonymity | tails.boum.org |
