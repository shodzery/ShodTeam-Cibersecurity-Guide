# Security Frameworks and Standards

## Overview

Security frameworks provide structured guidance for building, assessing, and improving an organization's security posture. They represent accumulated industry experience and are used for compliance, certification, risk management, and operational guidance.

---

## NIST Cybersecurity Framework (CSF)

The NIST Cybersecurity Framework was developed in 2014 (updated 2.0 in 2024) through collaboration between government, industry, and academia. It provides a policy framework for private sector organizations to assess and improve their ability to prevent, detect, and respond to cyber attacks.

### CSF Core Functions (Version 2.0)

```mermaid
graph LR
    GV[GOVERN] --> ID[IDENTIFY]
    ID --> PR[PROTECT]
    PR --> DE[DETECT]
    DE --> RS[RESPOND]
    RS --> RC[RECOVER]
    RC --> GV
```

| Function | Purpose | Categories (examples) |
|----------|---------|----------------------|
| GOVERN | Establish and monitor cybersecurity risk management strategy | Organizational context, risk management strategy, supply chain risk |
| IDENTIFY | Understand cybersecurity risk to systems, people, assets | Asset management, risk assessment, improvement |
| PROTECT | Safeguards to manage cybersecurity risk | Identity management, awareness training, data security, platform security |
| DETECT | Identify cybersecurity attacks and compromises | Continuous monitoring, adverse event analysis |
| RESPOND | Take action on detected attacks | Incident management, analysis, containment, communication |
| RECOVER | Restore assets and operations after an attack | Incident recovery plan execution, communication, improvements |

### CSF Implementation Tiers

| Tier | Name | Characteristics |
|------|------|----------------|
| Tier 1 | Partial | Ad hoc, reactive processes; no organizational-wide risk management |
| Tier 2 | Risk Informed | Risk practices approved by management but not organization-wide |
| Tier 3 | Repeatable | Formally approved, consistently applied risk management practices |
| Tier 4 | Adaptive | Organization adapts practices based on lessons learned and predictive indicators |

---

## NIST SP 800-53

NIST Special Publication 800-53 (Revision 5) provides a catalog of security and privacy controls for federal information systems and organizations. It is the most comprehensive control catalog in wide use.

### Control Families

| ID | Family | Key Controls |
|----|--------|-------------|
| AC | Access Control | AC-2 (Account Management), AC-3 (Access Enforcement), AC-17 (Remote Access) |
| AT | Awareness and Training | AT-2 (Literacy Training), AT-3 (Role-Based Training) |
| AU | Audit and Accountability | AU-2 (Event Logging), AU-6 (Audit Review), AU-9 (Protection of Audit Info) |
| CA | Assessment, Authorization, Monitoring | CA-7 (Continuous Monitoring), CA-8 (Penetration Testing) |
| CM | Configuration Management | CM-2 (Baseline Configuration), CM-6 (Configuration Settings) |
| CP | Contingency Planning | CP-4 (Contingency Plan Testing), CP-9 (System Backup) |
| IA | Identification and Authentication | IA-2 (Identification and Authentication - Org Users), IA-5 (Authenticator Management) |
| IR | Incident Response | IR-4 (Incident Handling), IR-5 (Incident Monitoring), IR-6 (Incident Reporting) |
| MA | Maintenance | MA-3 (Maintenance Tools), MA-5 (Maintenance Personnel) |
| MP | Media Protection | MP-2 (Media Access), MP-6 (Media Sanitization) |
| PE | Physical and Environmental Protection | PE-2 (Physical Access Authorizations), PE-6 (Monitoring Physical Access) |
| PL | Planning | PL-2 (System Security Plan) |
| PM | Program Management | PM-9 (Risk Management Strategy), PM-30 (Supply Chain Risk Management) |
| PS | Personnel Security | PS-3 (Personnel Screening), PS-6 (Access Agreements) |
| PT | PII Processing and Transparency | PT-1 through PT-8 |
| RA | Risk Assessment | RA-3 (Risk Assessment), RA-5 (Vulnerability Monitoring and Scanning) |
| SA | System and Services Acquisition | SA-8 (Security and Privacy Engineering Principles), SA-11 (Developer Testing) |
| SC | System and Communications Protection | SC-7 (Boundary Protection), SC-8 (Transmission Confidentiality/Integrity) |
| SI | System and Information Integrity | SI-2 (Flaw Remediation), SI-3 (Malicious Code Protection), SI-4 (System Monitoring) |
| SR | Supply Chain Risk Management | SR-2 (Supply Chain Risk Management Plan), SR-11 (Component Authenticity) |

---

## ISO/IEC 27001

ISO 27001 is an international standard for information security management systems (ISMS). Certification demonstrates to customers, partners, and regulators that an organization follows a systematic approach to managing sensitive information.

### ISMS Requirements

ISO 27001 follows the Plan-Do-Check-Act (PDCA) model:

| Phase | Activities |
|-------|-----------|
| Plan | Define ISMS scope, security policy, risk assessment methodology, risk treatment plan |
| Do | Implement controls, operate the ISMS, provide security awareness training |
| Check | Monitor, measure, audit ISMS performance; conduct internal audits and management reviews |
| Act | Corrective and preventive actions; continual improvement |

### Annex A Control Domains (ISO 27001:2022)

| Domain | Theme |
|--------|-------|
| Organizational controls (5) | Policies, roles, segregation of duties, threat intelligence, information security in projects |
| People controls (6) | Screening, terms and conditions, security awareness, disciplinary process |
| Physical controls (7) | Physical security perimeters, clear desk/screen, equipment security |
| Technological controls (8) | User endpoint devices, access control, authentication, cryptography, logging |

### Certification Process

1. **Gap assessment**: Compare current security posture against ISO 27001 requirements
2. **Implementation**: Develop and implement the ISMS, policies, and controls
3. **Internal audit**: Conduct internal audit to identify non-conformities
4. **Stage 1 audit**: Certification body reviews documentation
5. **Stage 2 audit**: Certification body conducts on-site assessment
6. **Certification**: Certificate issued for 3 years
7. **Surveillance audits**: Annual audits in years 1 and 2
8. **Recertification audit**: Full audit at year 3

---

## MITRE ATT&CK

MITRE ATT&CK (Adversarial Tactics, Techniques, and Common Knowledge) is a globally accessible knowledge base of adversary tactics and techniques based on real-world observations.

### Matrix Structure

ATT&CK organizes techniques under 14 tactics representing the adversary's goal during each phase:

| Tactic | ID | Purpose |
|--------|-----|---------|
| Reconnaissance | TA0043 | Gather information to support targeting |
| Resource Development | TA0042 | Establish resources for operations |
| Initial Access | TA0001 | Gain initial foothold in environment |
| Execution | TA0002 | Execute adversary-controlled code |
| Persistence | TA0003 | Maintain foothold across restarts |
| Privilege Escalation | TA0004 | Gain higher-level permissions |
| Defense Evasion | TA0005 | Avoid detection |
| Credential Access | TA0006 | Steal credentials |
| Discovery | TA0007 | Understand the environment |
| Lateral Movement | TA0008 | Move through the environment |
| Collection | TA0009 | Gather data of interest |
| Command and Control | TA0011 | Communicate with compromised systems |
| Exfiltration | TA0010 | Steal data |
| Impact | TA0040 | Manipulate, interrupt, or destroy systems |

### ATT&CK Use Cases

**Threat intelligence**: Map reported TTPs from threat intelligence reports to ATT&CK IDs for structured comparison and tracking.

**Detection coverage**: Map SIEM rules and EDR detections to ATT&CK techniques to visualize coverage gaps using ATT&CK Navigator.

**Red team planning**: Select techniques relevant to emulating a specific threat actor's documented behavior.

**Gap analysis**: Compare your coverage against the techniques used by threat actors targeting your sector.

**ATT&CK Navigator**: Web-based tool for creating and sharing annotated ATT&CK matrices
```
https://mitre-attack.github.io/attack-navigator/
```

---

## CIS Controls (Version 8)

The CIS Critical Security Controls (formerly SANS Top 20) are a prioritized set of actions for cyber defense, developed by a community of security practitioners.

### Control Groups

**Basic CIS Controls (Essential):**

| Control | Name |
|---------|------|
| CIS 1 | Inventory and Control of Enterprise Assets |
| CIS 2 | Inventory and Control of Software Assets |
| CIS 3 | Data Protection |
| CIS 4 | Secure Configuration of Enterprise Assets and Software |
| CIS 5 | Account Management |
| CIS 6 | Access Control Management |

**Foundational CIS Controls:**

| Control | Name |
|---------|------|
| CIS 7 | Continuous Vulnerability Management |
| CIS 8 | Audit Log Management |
| CIS 9 | Email and Web Browser Protections |
| CIS 10 | Malware Defenses |
| CIS 11 | Data Recovery |
| CIS 12 | Network Infrastructure Management |
| CIS 13 | Network Monitoring and Defense |
| CIS 14 | Security Awareness and Skills Training |
| CIS 15 | Service Provider Management |
| CIS 16 | Application Software Security |

**Organizational CIS Controls:**

| Control | Name |
|---------|------|
| CIS 17 | Incident Response Management |
| CIS 18 | Penetration Testing |

### Implementation Groups

CIS Controls are organized into Implementation Groups (IGs) based on organizational risk profile:

| IG | Profile | Description |
|----|---------|-------------|
| IG1 | Basic | Small, limited IT resources, lower risk; implement all IG1 Safeguards |
| IG2 | Medium | More complex environments; implement IG1 + IG2 Safeguards |
| IG3 | Large | Security teams, sensitive data, high-profile targets; implement all Safeguards |

---

## SOC 2

SOC 2 is an auditing standard developed by the AICPA for service organizations. It evaluates controls relevant to the Trust Services Criteria: Security, Availability, Processing Integrity, Confidentiality, and Privacy.

### Type I vs. Type II

| Aspect | Type I | Type II |
|--------|--------|---------|
| What is assessed | Design of controls at a point in time | Design and operating effectiveness over a period |
| Observation period | Single date | Minimum 6 months (typically 12) |
| Value to customers | Limited; no evidence controls operate effectively | Demonstrates sustained control effectiveness |
| Time to complete | Faster (2-4 months) | Longer (6-12+ months) |

### Common Criteria (Security TSC)

CC1: Control Environment
CC2: Communication and Information
CC3: Risk Assessment
CC4: Monitoring Activities
CC5: Control Activities
CC6: Logical and Physical Access Controls
CC7: System Operations
CC8: Change Management
CC9: Risk Mitigation

---

## Framework Comparison

| Dimension | NIST CSF | ISO 27001 | CIS Controls | SOC 2 |
|-----------|---------|-----------|-------------|-------|
| Type | Framework | Standard (certifiable) | Control set | Audit standard |
| Scope | Risk management | ISMS management | Specific technical controls | Service organization trust |
| Certification | No | Yes (third-party) | No | Yes (Type I/II report) |
| Prescriptiveness | Low (principles) | Medium | High (specific safeguards) | Medium |
| Primary audience | All sectors | All sectors | All sectors | B2B service providers |
| Customer value | Limited direct | High (certification) | Limited direct | High (SOC 2 report) |
| Regulatory alignment | HIPAA, FISMA | GDPR, many | Various | SOX, HIPAA, GDPR |
