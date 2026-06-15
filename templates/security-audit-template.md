# Security Audit Template

**Audit Name**: [e.g., Annual Information Security Audit / ISO 27001 Internal Audit]
**Organization**: [Organization Name]
**Audit Scope**: [Systems / Processes / Business Units in scope]
**Audit Period**: [Start Date] to [End Date]
**Lead Auditor**: [Name, Certification]
**Audit Team**: [Names and roles]
**Report Date**: [YYYY-MM-DD]
**Classification**: [Confidential]
**Next Scheduled Audit**: [YYYY-MM-DD]

---

## Audit Scope and Objectives

### Objectives

*State the specific objectives of this audit. Examples:*
- Assess compliance with [framework: ISO 27001 / NIST CSF / PCI DSS / internal policy]
- Evaluate the effectiveness of information security controls protecting [specific systems/data]
- Identify control gaps and deficiencies
- Assess progress against prior audit recommendations

### Scope Definition

**In Scope:**
- [System / application / business unit 1]
- [Network segment 2]
- [Process or procedure 3]

**Out of Scope:**
- [Excluded item with rationale]

### Audit Criteria

This audit was conducted against:
- [Framework/Standard: e.g., ISO/IEC 27001:2022 Annex A]
- [Internal Policy: e.g., Information Security Policy v3.2 dated YYYY-MM-DD]
- [Regulatory requirement: e.g., PCI DSS v4.0]

---

## Methodology

### Approach

This audit used a combination of:
- **Document review**: Policies, procedures, standards, and records
- **Technical testing**: Configuration review, automated scanning, manual verification
- **Interviews**: Discussions with system owners, process owners, and security staff
- **Observation**: Direct observation of processes and controls in operation
- **Sampling**: Representative sample of configurations, access reviews, and records examined

### Evidence Collected

| Evidence Type | Description | Source |
|--------------|-------------|--------|
| Document | Information Security Policy | [Document repository] |
| Technical | Firewall rule export | [Firewall management console] |
| Technical | Vulnerability scan results | [Nessus/OpenVAS report dated YYYY-MM-DD] |
| Interview | Network Administrator interview notes | [Auditor's workpapers] |
| Record | Access review log Q3 2024 | [IAM system export] |

---

## Control Assessment

### Access Control

| Control | Requirement | Evidence Reviewed | Finding | Status |
|---------|-------------|------------------|---------|--------|
| AC-01 | Account provisioning follows formal request and approval process | HR/IT ticket system | 48 of 50 sampled accounts have approved request; 2 lack documentation | Partial |
| AC-02 | Privileged accounts reviewed quarterly | Access review records | Last review conducted [date]; 12-week cadence instead of 13-week — within tolerance | Pass |
| AC-03 | MFA enforced for all privileged access | IdP configuration | MFA enforced for 98% of admin accounts; 2 service accounts excluded without documented exception | Finding |
| AC-04 | Terminated employee accounts disabled within 1 business day | HR offboarding + AD | 4 of 20 sampled terminations exceeded 1-day SLA (average: 3.2 days) | Finding |

### Cryptography

| Control | Requirement | Evidence Reviewed | Finding | Status |
|---------|-------------|------------------|---------|--------|
| CR-01 | Data at rest encrypted with AES-256 | Storage configuration | Production database encrypted; 1 development environment unencrypted | Finding |
| CR-02 | TLS 1.2+ enforced for all data in transit | TLS configuration scan | TLS 1.0 and 1.1 still enabled on legacy application server (decommission planned Q2) | Finding |
| CR-03 | Certificate inventory maintained and monitored | Certificate management records | 3 of 47 certificates lack documented owner; certificate monitoring in place | Partial |

### Vulnerability Management

| Control | Requirement | Evidence Reviewed | Finding | Status |
|---------|-------------|------------------|---------|--------|
| VM-01 | Critical vulnerabilities patched within 72 hours | Vulnerability scanner reports + patch records | 87% of critical CVEs patched within SLA; 13% exceeded (average excess: 2.3 days) | Finding |
| VM-02 | Monthly vulnerability scans across all in-scope systems | Scan job history | All scans completing monthly; 1 segment missed 2 consecutive months (network change broke scan access) | Finding |
| VM-03 | Penetration test conducted annually | Pentest report | Most recent test dated [YYYY-MM-DD]; within 12-month window; remediation tracked | Pass |

### Incident Response

| Control | Requirement | Evidence Reviewed | Finding | Status |
|---------|-------------|------------------|---------|--------|
| IR-01 | Documented incident response plan | IR plan document | IR plan exists, dated [YYYY-MM-DD]; last tabletop exercise [date] | Pass |
| IR-02 | Incident response contact list maintained and tested | Contact list | Contact list updated quarterly; not tested via out-of-band channel in past 12 months | Partial |
| IR-03 | Incidents documented in tracking system | Incident records | 19 of 20 sampled incidents have complete records; 1 minor incident not logged | Finding |

### Business Continuity

| Control | Requirement | Evidence Reviewed | Finding | Status |
|---------|-------------|------------------|---------|--------|
| BC-01 | BCP/DRP documented and approved | BCP document | BCP exists but last reviewed 22 months ago; review overdue | Finding |
| BC-02 | Backup tested quarterly via restoration exercise | Backup test records | Last successful restoration test: [date]; within 3-month window | Pass |
| BC-03 | RTO and RPO defined and achievable | BCP documentation + infrastructure review | RTO: 4 hours (defined). Last test achieved 6.5 hours (exceeds target) | Finding |

---

## Findings Summary

### Finding Severity Definitions

| Severity | Definition |
|----------|------------|
| Critical | Immediate risk of significant harm; requires immediate corrective action |
| High | Significant control deficiency; requires prompt corrective action within 30 days |
| Medium | Moderate control gap; corrective action required within 90 days |
| Low | Minor control gap or improvement opportunity; corrective action within 6 months |
| Observation | No current control deficiency; informational for management awareness |

### Finding Register

| Finding ID | Title | Severity | Control Area | Status |
|-----------|-------|----------|-------------|--------|
| F-001 | MFA not enforced on 2 privileged service accounts | High | Access Control | Open |
| F-002 | Terminated employee account deactivation SLA exceeded | High | Access Control | Open |
| F-003 | Development environment database unencrypted at rest | Medium | Cryptography | Open |
| F-004 | TLS 1.0/1.1 enabled on legacy server | Medium | Cryptography | Open |
| F-005 | Critical patch SLA not met for 13% of vulnerabilities | High | Vulnerability Management | Open |
| F-006 | Vulnerability scan coverage gap on 1 network segment | Medium | Vulnerability Management | Open |
| F-007 | BCP last reviewed 22 months ago (review overdue) | Medium | Business Continuity | Open |
| F-008 | RTO target not achieved in last recovery test | High | Business Continuity | Open |

### Detailed Findings

---

**F-001: MFA not enforced on 2 privileged service accounts**

**Severity**: High

**Condition**: Two service accounts with Domain Administrator privileges in Active Directory do not have MFA enforced. These accounts are used for automated administrative tasks.

**Criteria**: Information Security Policy Section 4.3 requires MFA for all privileged accounts. Industry best practice (NIST SP 800-63B, CIS Control 6.5) aligns with this requirement.

**Cause**: Service accounts predate the MFA enforcement policy; no remediation plan was created when the policy was adopted.

**Effect**: If these service accounts' credentials are compromised, an attacker gains privileged access without requiring a second factor, increasing the risk of domain compromise.

**Evidence**: Active Directory configuration review; IdP MFA policy settings.

**Recommendation**: 
1. Document a formal exception with business justification and compensating controls, or
2. Convert these accounts to Group Managed Service Accounts (gMSA) with automatically rotating credentials, eliminating the need for stored credentials and the MFA exception

**Management Response**: [To be completed by management]

**Target Remediation Date**: [YYYY-MM-DD]

**Owner**: [Name / Role]

---

*(Continue with each finding in the same format)*

---

## Prior Audit Findings: Remediation Status

| Prior Finding ID | Description | Severity | Target Date | Status | Evidence |
|----------------|-------------|----------|------------|--------|---------|
| PF-2023-001 | Weak password policy on external portal | High | [date] | Closed | Policy updated; confirmed via configuration review |
| PF-2023-002 | Missing security awareness training records | Medium | [date] | Closed | LMS records reviewed; 97% completion |
| PF-2023-003 | No network segmentation between PCI and non-PCI systems | High | [date] | Open — overdue | Project delayed; partial segmentation in place |

---

## Overall Audit Opinion

**Opinion**: [Qualified / Unqualified / Adverse / Disclaimer of Opinion]

*[Example: This audit identified 8 findings across access control, cryptography, vulnerability management, and business continuity domains. While a majority of sampled controls operate effectively, 3 High severity findings require prompt management attention. The information security control environment is partially effective in managing risks within the defined scope.]*

---

## Recommendations Summary

| Priority | Finding | Recommendation | Owner | Target Date |
|----------|---------|---------------|-------|------------|
| High | F-001 | Remediate or formally except service accounts from MFA policy | IAM Team | [Date] |
| High | F-002 | Reduce account deactivation SLA to 4 hours; automate via HR/IT integration | IT Operations | [Date] |
| High | F-005 | Implement automated patch compliance reporting with escalation | Patch Management | [Date] |
| High | F-008 | Conduct failover test and update BCP with realistic RTO | IT / Business Continuity | [Date] |
| Medium | F-003 | Enable encryption on development database | DBA | [Date] |
| Medium | F-004 | Disable TLS 1.0/1.1 per decommission schedule; expedite if feasible | Infrastructure | [Date] |
| Medium | F-006 | Restore vulnerability scan coverage to isolated segment | Security Operations | [Date] |
| Medium | F-007 | Conduct BCP review and obtain management approval | Business Continuity | [Date] |

---

## Auditor Statements

This audit was conducted in accordance with [applicable standards] and with due professional care. Evidence was collected through document review, interviews, technical testing, and sampling. Findings represent the auditor's professional judgment based on evidence gathered during the audit period.

**Lead Auditor**: [Name] | [Certification] | [Date]

**Management Acknowledgment**: [Name / Title] | [Date]
