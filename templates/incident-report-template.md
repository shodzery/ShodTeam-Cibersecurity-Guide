# Incident Report Template

**Document Classification**: [Confidential / Internal / Restricted]
**Report Version**: [1.0]
**Status**: [Draft / Under Review / Final]

---

## Incident Identification

| Field | Value |
|-------|-------|
| Incident ID | IR-[YYYY]-[NNNN] |
| Date/Time Detected | [YYYY-MM-DD HH:MM UTC] |
| Date/Time Reported | [YYYY-MM-DD HH:MM UTC] |
| Date/Time Contained | [YYYY-MM-DD HH:MM UTC] |
| Date/Time Resolved | [YYYY-MM-DD HH:MM UTC] |
| Report Date | [YYYY-MM-DD] |
| Severity | [Critical / High / Medium / Low] |
| Status | [Open / Contained / Resolved / Closed] |
| Incident Commander | [Name, Title] |
| Report Author | [Name, Title] |

---

## Executive Summary

*Provide a 2–4 paragraph non-technical summary suitable for executive and board-level audiences. Cover: what happened, what was affected, what actions were taken, and current status. Do not use technical jargon without explanation.*

[Summary text here]

---

## Incident Classification

**Incident Type**: [Select all that apply]
- [ ] Malware / Ransomware
- [ ] Data Breach / Exfiltration
- [ ] Unauthorized Access
- [ ] Denial of Service
- [ ] Insider Threat
- [ ] Phishing / Social Engineering
- [ ] Business Email Compromise
- [ ] Supply Chain Compromise
- [ ] Other: [specify]

**MITRE ATT&CK Techniques**: [List primary TTPs observed, e.g., T1566.001, T1059.001]

**Regulatory Notification Required**: [Yes / No / Under Assessment]
- Applicable regulations: [GDPR / HIPAA / PCI DSS / State breach notification / Other]
- Notification deadline: [Date or N/A]
- Notification status: [Not required / Pending / Submitted on YYYY-MM-DD]

---

## Impact Assessment

### Affected Systems

| System | Role | Classification | Data Exposed | Availability Impact |
|--------|------|---------------|-------------|---------------------|
| [hostname/IP] | [Web server, DB, workstation] | [Critical/High/Medium] | [Yes/No/Unknown] | [None/Partial/Full] |
| | | | | |

### Affected Data

| Data Type | Estimated Records | Classification | Exfiltrated? |
|-----------|------------------|---------------|--------------|
| [e.g., Customer PII] | [Approximate count] | [Confidential] | [Yes/No/Unknown] |
| | | | |

### Business Impact

| Impact Category | Description | Estimated Cost |
|----------------|-------------|---------------|
| Operational downtime | [Duration and affected services] | [$X] |
| Data recovery | [Labor and tool costs] | [$X] |
| Regulatory exposure | [Potential fines] | [$X] |
| Reputational impact | [Customer notifications, PR costs] | [$X] |
| **Total Estimated Impact** | | **[$X]** |

---

## Incident Timeline

*Provide a chronological reconstruction of events from initial compromise through resolution. Include all times in UTC.*

| Date/Time (UTC) | Event | Source | Analyst |
|----------------|-------|--------|---------|
| [YYYY-MM-DD HH:MM] | Estimated attacker initial access via [vector] | [Log source / hypothesis] | [Name] |
| [YYYY-MM-DD HH:MM] | [Next event] | [Source] | [Name] |
| [YYYY-MM-DD HH:MM] | Incident detected by [method] | [Alert / user report / tool] | [Name] |
| [YYYY-MM-DD HH:MM] | Incident escalated to IR Lead | [Name] | [Name] |
| [YYYY-MM-DD HH:MM] | Containment action: [describe] | [IR team] | [Name] |
| [YYYY-MM-DD HH:MM] | [Continue chronologically] | | |
| [YYYY-MM-DD HH:MM] | Incident declared resolved | [IR Lead] | [Name] |

**Total Dwell Time** (initial access to detection): [Duration]
**Mean Time to Contain** (detection to containment): [Duration]
**Mean Time to Resolve** (detection to resolution): [Duration]

---

## Technical Analysis

### Initial Access Vector

*Describe how the attacker gained initial access. Reference specific log evidence.*

**Method**: [Phishing / Exploitation of vulnerability / Credential compromise / Other]
**Details**: [Specific vulnerability, CVE if applicable, delivery mechanism]
**Evidence**: [Log entries, email headers, network captures]

### Attack Progression

*Describe what the attacker did after gaining initial access.*

**Reconnaissance / Discovery**:
[What the attacker enumerated or discovered]

**Privilege Escalation**:
[How the attacker gained elevated privileges, if applicable]

**Lateral Movement**:
[How the attacker moved to additional systems, if applicable]

**Persistence**:
[What persistence mechanisms were installed]

**Actions on Objectives**:
[What the attacker ultimately achieved: data theft, encryption, etc.]

### Indicators of Compromise

*List all identified IOCs. These should be added to threat intelligence systems and blocking rules.*

**File Artifacts**:
| Filename | SHA-256 | Location |
|----------|---------|---------|
| [filename.exe] | [hash] | [path] |

**Network Indicators**:
| Type | Value | Context |
|------|-------|---------|
| IP | [x.x.x.x] | [C2 server observed in connections] |
| Domain | [evil.com] | [Phishing domain] |
| URL | [https://...] | [Payload delivery URL] |

**Host Artifacts**:
| Type | Value | Location |
|------|-------|---------|
| Registry Key | [HKLM\...] | [Persistence] |
| Scheduled Task | [TaskName] | [Persistence] |
| File Path | [C:\Users\...] | [Dropped file] |

---

## Response Actions

### Containment

| Action | Date/Time | Performed By | Outcome |
|--------|-----------|-------------|---------|
| Isolated system [hostname] from network | [YYYY-MM-DD HH:MM] | [Name] | [Success/Partial] |
| Disabled user account [username] | [YYYY-MM-DD HH:MM] | [Name] | [Success] |
| Revoked API keys / tokens | [YYYY-MM-DD HH:MM] | [Name] | [Success] |
| [Additional actions] | | | |

### Eradication

| Action | Date/Time | Performed By | Outcome |
|--------|-----------|-------------|---------|
| Removed malware from [system] | [YYYY-MM-DD HH:MM] | [Name] | [Success] |
| Reset credentials for [account] | [YYYY-MM-DD HH:MM] | [Name] | [Success] |
| Removed persistence mechanism [describe] | [YYYY-MM-DD HH:MM] | [Name] | [Success] |
| Patched vulnerability [CVE] | [YYYY-MM-DD HH:MM] | [Name] | [Success] |

### Recovery

| Action | Date/Time | Performed By | Outcome |
|--------|-----------|-------------|---------|
| Restored [system] from backup dated [date] | [YYYY-MM-DD HH:MM] | [Name] | [Success] |
| Validated system integrity | [YYYY-MM-DD HH:MM] | [Name] | [Success] |
| Returned [system] to production | [YYYY-MM-DD HH:MM] | [Name] | [Success] |

---

## Root Cause Analysis

**Primary Root Cause**: [The fundamental condition that allowed this incident to occur]

**Contributing Factors**:
1. [Factor 1: e.g., MFA not enforced on VPN]
2. [Factor 2: e.g., Patching SLA exceeded for critical vulnerability]
3. [Factor 3]

**Why Was It Not Detected Earlier?**:
[Describe detection gaps: missing log sources, no alerting rule for this technique, alert fatigue, etc.]

---

## Lessons Learned and Recommendations

### Immediate Actions (within 30 days)

| Action | Owner | Priority | Target Date |
|--------|-------|----------|------------|
| [Specific control improvement] | [Team/Person] | [Critical/High] | [Date] |
| [Detection gap remediation] | [Team/Person] | [High] | [Date] |
| [Additional monitoring] | [Team/Person] | [High] | [Date] |

### Short-Term Actions (30–90 days)

| Action | Owner | Priority | Target Date |
|--------|-------|----------|------------|
| [Process improvement] | [Team/Person] | [Medium] | [Date] |
| [Training] | [Team/Person] | [Medium] | [Date] |

### Long-Term Actions (90+ days)

| Action | Owner | Priority | Target Date |
|--------|-------|----------|------------|
| [Architecture change] | [Team/Person] | [Medium] | [Date] |
| [Program improvement] | [Team/Person] | [Low] | [Date] |

---

## Appendices

### A: Evidence Inventory

| Evidence ID | Description | Hash | Storage Location | Chain of Custody |
|-------------|-------------|------|-----------------|-----------------|
| EV-001 | [Memory image from server] | SHA256: [hash] | [Location] | [Custodian] |
| | | | | |

### B: Communications Log

| Date/Time | From | To | Summary |
|-----------|------|----|---------|
| | | | |

### C: External Notifications

| Organization | Contact | Notification Type | Date Sent | Reference |
|-------------|---------|-----------------|-----------|-----------|
| [Regulator] | [Contact] | [Breach notification] | [Date] | [Case #] |

### D: References

- [Relevant threat intelligence reports]
- [CVE references]
- [Log file locations]
