# Learning Paths

Structured paths by role and goal. Each path includes free and paid resources, with realistic time estimates. All paths assume starting from general IT familiarity.

---

## Path 1: SOC Analyst (0 to Job-Ready)

**Target outcome**: Junior SOC Analyst (Tier 1) capable of alert triage, basic investigation, and escalation.

**Estimated time**: 6–12 months with consistent study (15–20 hours/week)

### Phase 1: Foundations (Month 1–2)

| Resource | Type | Cost | Hours |
|----------|------|------|-------|
| Professor Messer's CompTIA Security+ Course | Video (free) | Free | 20 |
| CompTIA Security+ Study Guide | Book | ~$40 | 30 |
| TryHackMe: Pre-Security Path | Interactive labs | Free/Subscription | 30 |
| Networking Fundamentals (CompTIA Network+ material) | Self-study | Free | 20 |

**Milestone**: CompTIA Security+ exam

### Phase 2: SOC-Specific Skills (Month 3–5)

| Resource | Type | Cost | Hours |
|----------|------|------|-------|
| TryHackMe: SOC Level 1 Path | Interactive labs | Subscription (~$14/mo) | 40 |
| Blue Team Labs Online: Free Labs | Interactive labs | Free | 20 |
| Splunk Core User Course (free on Splunk) | Online course | Free | 15 |
| Boss of the SOC (BOTS) v1–v3 | CTF/labs | Free | 20 |

**Milestone**: Comfortable with basic log analysis, Splunk queries, alert triage

### Phase 3: Depth and Credentialing (Month 6–10)

| Resource | Type | Cost | Hours |
|----------|------|------|-------|
| CompTIA CySA+ Study | Self-study | ~$40 book | 40 |
| MITRE ATT&CK Navigator practice | Self-directed | Free | 10 |
| HackTheBox Starting Point (Blue Team perspective) | Labs | Free | 20 |
| Eric Zimmerman's Tools (Windows forensics) | Hands-on | Free | 15 |

**Milestone**: CompTIA CySA+ exam

### Portfolio Building

- Set up a home lab: Splunk/Elastic, Windows VM, Kali VM
- Complete SOC-related CTFs and document findings
- Practice writing investigation reports
- Build and maintain a GitHub repository of custom detection rules

---

## Path 2: Penetration Tester (0 to OSCP)

**Target outcome**: Junior penetration tester capable of conducting basic web application and network assessments, with OSCP certification.

**Estimated time**: 12–18 months. OSCP preparation alone typically requires 3–6 months.

### Phase 1: Technical Foundations (Month 1–3)

| Resource | Type | Cost | Hours |
|----------|------|------|-------|
| TryHackMe: Pre-Security + Jr Pentester Path | Labs | Free/Subscription | 60 |
| Linux Fundamentals (OverTheWire: Bandit) | CTF | Free | 15 |
| Networking: subnetting, TCP/IP, common protocols | Self-study | Free | 20 |
| Python for security: Automate the Boring Stuff + networking | Book + practice | Free | 25 |

### Phase 2: Web and Network Basics (Month 4–6)

| Resource | Type | Cost | Hours |
|----------|------|------|-------|
| PortSwigger Web Academy | Labs | Free | 60 |
| HackTheBox Starting Point | Labs | Free | 30 |
| TCM Security Practical Ethical Hacking | Course | ~$30 | 25 |
| Metasploit Unleashed | Online guide | Free | 15 |

**Milestone**: eJPT or PNPT certification (practical, job-relevant, cost-effective)

### Phase 3: OSCP Preparation (Month 7–14)

| Resource | Type | Cost | Hours |
|----------|------|------|-------|
| Offensive Security PEN-200 (includes lab access) | Official course | $1,499 (90-day lab) | 200+ |
| HackTheBox: Easy/Medium machines | Labs | Free/VIP | 60 |
| Proving Grounds Practice (Offensive Security) | Labs | $19/mo | 40 |
| TJ Null's OSCP-like machines list | Lab target list | Free | 60 |

**Milestone**: OSCP certification

### Key Skills to Develop

- Manual exploitation (not just automated tools)
- Buffer overflow (32-bit, for OSCP)
- Active Directory attack paths
- Report writing — practice with every machine
- Note-taking methodology (Obsidian, Cherrytree, Notion)

---

## Path 3: Cloud Security Engineer (AWS Focus)

**Target outcome**: AWS security engineer capable of designing and implementing secure cloud architectures.

**Estimated time**: 8–14 months

### Phase 1: AWS Fundamentals (Month 1–3)

| Resource | Type | Cost | Hours |
|----------|------|------|-------|
| AWS Cloud Practitioner (optional) | Certification | $100 exam | 20 |
| AWS Solutions Architect Associate (A Cloud Guru or Stephane Maarek) | Course | ~$30 | 40 |
| AWS Free Tier account practice | Hands-on | Free (within limits) | 40 |
| AWS documentation: IAM, VPC, S3, EC2, CloudTrail | Reading | Free | 20 |

**Milestone**: AWS Solutions Architect Associate

### Phase 2: Security Focus (Month 4–8)

| Resource | Type | Cost | Hours |
|----------|------|------|-------|
| Adrian Cantrill's AWS Security Specialty course | Course | ~$40 | 40 |
| CloudGoat (Rhino Security Labs) | Vulnerable AWS labs | Free | 20 |
| Flaws.cloud (Scott Piper) | AWS security CTF | Free | 10 |
| Flaws2.cloud | AWS security CTF | Free | 10 |
| Prowler and ScoutSuite documentation | Tool practice | Free | 10 |
| CCSK study | Certification prep | $395 exam | 20 |

**Milestone**: AWS Security Specialty or CCSK

### Phase 3: Advanced (Month 9–14)

| Resource | Type | Cost | Hours |
|----------|------|------|-------|
| Kubernetes security (CKS preparation) | Self-study + labs | Free + ~$395 exam | 40 |
| Terraform / IaC security: checkov, tfsec | Hands-on | Free | 15 |
| CCSP preparation | Study | $599 exam | 40 |
| Cloud architecture review practice | Hands-on | Free | 20 |

---

## Path 4: Incident Responder / Digital Forensics

**Target outcome**: Incident responder capable of leading IR investigations and performing digital forensics on Windows endpoints.

**Estimated time**: 10–16 months

### Phase 1: Foundation (Month 1–3)

| Resource | Type | Cost | Hours |
|----------|------|------|-------|
| CompTIA Security+ | Certification | ~$400 exam | 40 |
| Windows internals: Windows Sysinternals documentation | Reading | Free | 20 |
| Introduction to Windows Forensics (SANS Posters) | Reading | Free | 5 |
| TryHackMe: SOC Level 1 | Labs | Subscription | 40 |

### Phase 2: Forensics Core Skills (Month 4–7)

| Resource | Type | Cost | Hours |
|----------|------|------|-------|
| MemLabs CTF challenges (memory forensics) | CTF | Free | 20 |
| Eric Zimmerman's Tools (EZ Tools) | Hands-on | Free | 20 |
| Volatility 3 practice with public memory samples | Hands-on | Free | 15 |
| BlueTeamLabs Online (DFIR scenarios) | Labs | Free/Paid | 25 |
| DFIR.training resources | Self-study | Free | 15 |

**Milestone**: Confident with Windows artifact analysis, memory forensics

### Phase 3: Professional Level (Month 8–14)

| Resource | Type | Cost | Hours |
|----------|------|------|-------|
| SANS FOR508 (if budget available) or equivalent materials | Course/Self-study | $6,000+ SANS or books/labs | 80 |
| GIAC GCFA exam preparation | Certification | ~$1,000+ exam | 40 |
| Practice IR tabletop exercises | Hands-on | Free | 20 |
| Malware analysis introduction (REMnux + Cuckoo) | Hands-on | Free | 20 |

---

## Free Resources Reference

### Practice Platforms

| Platform | Cost | Best For |
|----------|------|---------|
| TryHackMe | Free / $14/mo | Beginners, SOC, pentesting |
| HackTheBox | Free / $14/mo | Intermediate-Advanced pentesting |
| PortSwigger Web Academy | Free | Web application security |
| Blue Team Labs Online | Free / Paid | Defensive, forensics |
| DFIR.training | Free | DFIR resources and tools |
| MemLabs | Free | Memory forensics CTF |
| Flaws.cloud / Flaws2.cloud | Free | AWS security |
| CloudGoat | Free | AWS attack/defense |

### Free Courses and Reading

| Resource | Provider | Topic |
|----------|----------|-------|
| NIST SP 800 series | NIST | All security domains |
| OWASP Testing Guide | OWASP | Web application security |
| ATT&CK Knowledge Base | MITRE | Threat intelligence, TTPs |
| PortSwigger Web Security Academy | PortSwigger | Web security |
| Cybrary free courses | Cybrary | Various security topics |
| Professor Messer (CompTIA) | Self | Security+, Network+ |
| OpenSecurityTraining2 | OST2 | Low-level, reverse engineering |
| Malware Unicorn workshops | Andrea Fortuna | Malware analysis |
