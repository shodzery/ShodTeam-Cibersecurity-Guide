# Changelog

This project is authored and maintained by [Shodzery](https://github.com/shodzery) and [ShodTeam](https://github.com/ShodTeam).

All significant changes to this repository are documented here. Entries are organized by version and date, with the most recent changes at the top.

Format based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [Unreleased]

### Planned
- Mobile security documentation (Android, iOS, OWASP MASVS)
- Kubernetes and container security section
- Advanced malware analysis lab exercises
- MITRE ATT&CK technique mapping across all documentation
- CI/CD pipeline security documentation
- Hardware security module (HSM) documentation

---

## [1.0.0] - 2024-01-15

### Added

**Core Documentation**
- Repository structure and README with complete navigation
- Foundational security concepts: CIA triad, AAA, defense in depth, attack surface
- Networking security: TCP/IP, DNS, HTTP/HTTPS, firewalls, IDS/IPS, VPN, segmentation
- Cryptography: hashing, AES, RSA, ECC, PKI, TLS handshake, common vulnerabilities
- Web security: OWASP Top 10 (2021 edition), XSS, SQL injection, CSRF, SSRF, XXE, path traversal, security headers
- Cloud security: AWS, Azure, GCP, shared responsibility model, IAM, common misconfigurations
- Malware analysis: taxonomy, static analysis, dynamic analysis, ransomware, rootkits, botnets
- Blue Team operations: SIEM, EDR, threat hunting, logging strategy, detection engineering
- Red Team operations: reconnaissance, enumeration, exploitation, post-exploitation, persistence, OPSEC
- Digital forensics: chain of custody, evidence acquisition, memory forensics, disk analysis, timeline analysis
- Incident response: NIST SP 800-61 lifecycle, containment, eradication, recovery, lessons learned
- Governance, Risk, and Compliance: risk management, security policies, compliance frameworks
- Security frameworks reference: NIST CSF, NIST 800-53, ISO 27001, MITRE ATT&CK, CIS Controls, OWASP, SOC 2
- Identity and Access Management: IAM, MFA, RBAC, PAM, SSO, federation
- Zero Trust architecture: principles, network model, implementation guidance
- SOC operations: alert triage, escalation, playbooks, metrics
- Purple Team: adversary emulation, ATT&CK alignment, feedback loop process
- Career roadmap by role

**Labs**
- Beginner: Network scanning with Nmap, web application vulnerability identification, basic log analysis, password policy testing
- Intermediate: Exploitation with Metasploit, SIEM rule creation, phishing simulation analysis, forensic artifact examination
- Advanced: Custom payload development, adversary emulation campaign, memory forensics, detection rule development

**Resources**
- Books list organized by domain and experience level
- Certification guide organized by role and vendor
- Tool reference organized by category
- Blog and research resource list
- Role-based learning paths

**Templates**
- Incident report template
- Risk assessment template
- Threat model template
- Security audit template

**Diagrams**
- Network security architecture diagram
- Incident response workflow
- Kill chain mapping
- PKI lifecycle diagram
- Zero Trust network model
- SOC alert triage workflow

**GitHub Meta**
- Issue templates for bug reports and feature requests
- Pull request template
- Contributing guidelines
- Code of conduct
- License (Apache 2.0)

---

## Version Numbering

This repository uses semantic versioning adapted for documentation:

- **Major version**: Significant structural reorganization or addition of entirely new domains
- **Minor version**: Addition of new sections, labs, or substantial documentation within existing domains
- **Patch version**: Corrections, clarifications, and minor updates to existing content
