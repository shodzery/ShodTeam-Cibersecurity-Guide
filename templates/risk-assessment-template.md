# Risk Assessment Template

**Organization**: [Organization Name]
**Assessment Scope**: [System / Business Process / Organization-wide]
**Assessment Date**: [YYYY-MM-DD]
**Assessment Lead**: [Name, Title]
**Review Date**: [YYYY-MM-DD] (next scheduled reassessment)
**Classification**: [Confidential / Internal]

---

## Purpose and Scope

### Purpose

*Describe the objective of this risk assessment. Example: This assessment evaluates the information security risks associated with the Customer Relationship Management (CRM) system and its supporting infrastructure.*

### Scope

**In Scope**:
- [System / component / process / data type]
- [Network segment or boundary]
- [User population]

**Out of Scope**:
- [Explicitly excluded items with rationale]

### Methodology

This assessment uses [Qualitative / Semi-quantitative / Quantitative] risk analysis based on [NIST SP 800-30 / ISO 27005 / other framework].

**Risk Rating Formula**:
```
Risk = Likelihood × Impact

Likelihood Scale: 1 (Very Low) to 5 (Very High)
Impact Scale:     1 (Very Low) to 5 (Very High)
Risk Score:       1–25

Risk Level:
  1–4:   Low
  5–9:   Medium
  10–15: High
  16–25: Critical
```

---

## Asset Inventory

| Asset ID | Asset Name | Type | Owner | Classification | Criticality |
|---------|-----------|------|-------|---------------|-------------|
| A-001 | [Customer database] | Data | [Data Owner] | Confidential | Critical |
| A-002 | [CRM application server] | System | [IT Owner] | Internal | High |
| A-003 | [Customer PII] | Data | [CISO] | Restricted | Critical |
| | | | | | |

---

## Threat Identification

| Threat ID | Threat Category | Threat Description | Threat Source |
|-----------|----------------|-------------------|--------------|
| T-001 | External / Adversarial | Ransomware targeting organizational data | Financially motivated threat actors |
| T-002 | External / Adversarial | SQL injection against customer-facing web application | Opportunistic and targeted attackers |
| T-003 | Internal / Accidental | Misconfiguration of cloud storage permissions | IT administration error |
| T-004 | Internal / Intentional | Insider data theft by departing employee | Malicious insider |
| T-005 | Environmental | Extended power outage affecting data center | Environmental hazard |
| T-006 | External / Adversarial | Credential stuffing against customer accounts | Cybercriminal groups |
| | | | |

---

## Vulnerability Identification

| Vulnerability ID | Description | Asset(s) Affected | Source |
|----------------|-------------|------------------|--------|
| V-001 | Outdated SSL/TLS configuration (TLS 1.0/1.1 enabled) | A-002 | Automated scan |
| V-002 | No MFA enforced for privileged accounts | A-002 | Configuration review |
| V-003 | Default credentials on network device | [Device] | Penetration test |
| V-004 | Excessive IAM permissions on service account | A-001 | Access review |
| V-005 | Missing input validation on API endpoint | A-002 | Code review |
| | | | |

---

## Risk Register

*Each risk represents a specific threat exploiting a specific vulnerability affecting a specific asset.*

### Risk Template

| Field | Value |
|-------|-------|
| Risk ID | R-[NNN] |
| Risk Statement | [Threat] exploits [Vulnerability] resulting in [Impact] to [Asset] |
| Threat | [Threat ID reference] |
| Vulnerability | [Vulnerability ID reference] |
| Affected Asset | [Asset ID reference] |
| Current Controls | [Existing controls partially mitigating this risk] |
| Likelihood (1–5) | [Score] — [Rationale] |
| Impact (1–5) | [Score] — [Rationale] |
| Inherent Risk Score | [L × I = Score] |
| Inherent Risk Level | [Low / Medium / High / Critical] |
| Residual Risk Score | [After existing controls] |
| Residual Risk Level | [Low / Medium / High / Critical] |
| Risk Treatment | [Accept / Mitigate / Transfer / Avoid] |
| Treatment Plan | [Specific actions if not accepting] |
| Risk Owner | [Name / Role] |
| Target Date | [YYYY-MM-DD] |

---

### Risk Register Entries

**R-001: Ransomware via phishing**

| Field | Value |
|-------|-------|
| Risk ID | R-001 |
| Risk Statement | Financially motivated threat actors deliver ransomware via phishing email, encrypting operational data and demanding payment |
| Threat | T-001 |
| Vulnerability | No advanced email filtering; user susceptibility to phishing |
| Affected Assets | A-001, A-002 |
| Current Controls | Basic email spam filtering; quarterly security awareness training; daily backups |
| Likelihood | 4 — High. Ransomware campaigns are widespread; sector is actively targeted |
| Impact | 5 — Critical. Encryption would halt operations; customer data potentially exfiltrated |
| Inherent Risk Score | 20 — Critical |
| Residual Risk Score | 12 — High (reduced by existing backup and training controls) |
| Risk Treatment | Mitigate |
| Treatment Plan | Deploy advanced email filtering with sandbox analysis; enforce MFA on all accounts; implement immutable backups; conduct phishing simulation quarterly; deploy EDR |
| Risk Owner | CISO |
| Target Date | [YYYY-MM-DD] |

---

**R-002: SQL injection on customer portal**

| Field | Value |
|-------|-------|
| Risk ID | R-002 |
| Risk Statement | External attacker exploits SQL injection vulnerability in customer portal to extract customer PII |
| Threat | T-002 |
| Vulnerability | V-005 — Missing input validation on API endpoint |
| Affected Assets | A-001, A-003 |
| Current Controls | None specific to this vulnerability |
| Likelihood | 3 — Medium. Vulnerability confirmed; exploitation requires technical knowledge |
| Impact | 5 — Critical. Customer PII breach triggers regulatory notification, reputational damage |
| Inherent Risk Score | 15 — High |
| Residual Risk Score | 15 — High (no mitigating controls) |
| Risk Treatment | Mitigate (urgent) |
| Treatment Plan | Fix input validation defect in next sprint (parameterized queries); deploy WAF with SQL injection rules; conduct security code review of all API endpoints |
| Risk Owner | CTO / AppSec Lead |
| Target Date | [Immediate — within 14 days] |

---

*(Add additional risk entries following the same template)*

---

## Risk Summary

### Risk Distribution

| Risk Level | Count | Percentage |
|-----------|-------|------------|
| Critical | [N] | [%] |
| High | [N] | [%] |
| Medium | [N] | [%] |
| Low | [N] | [%] |
| **Total** | **[N]** | **100%** |

### Risk Heatmap

```
Impact
  5 |  Med  | High  | Crit  | Crit  | Crit  |
  4 |  Low  |  Med  | High  | Crit  | Crit  |
  3 |  Low  |  Med  |  Med  | High  | Crit  |
  2 |  Low  |  Low  |  Med  |  Med  | High  |
  1 |  Low  |  Low  |  Low  |  Low  |  Med  |
    |   1   |   2   |   3   |   4   |   5   
                              Likelihood
```

*Mark each risk on the heatmap using its Risk ID.*

---

## Risk Treatment Plan

### Critical and High Risks (Require management attention)

| Risk ID | Risk Description | Treatment | Owner | Target Date | Status |
|---------|----------------|-----------|-------|-------------|--------|
| R-001 | Ransomware via phishing | Mitigate | CISO | [Date] | In progress |
| R-002 | SQL injection on portal | Mitigate | CTO | [Date] | Not started |
| | | | | | |

### Medium Risks

| Risk ID | Risk Description | Treatment | Owner | Target Date | Status |
|---------|----------------|-----------|-------|-------------|--------|
| | | | | | |

---

## Risk Acceptance

The following risks have been formally accepted by management. Acceptance is documented with the approving authority and rationale.

| Risk ID | Risk Level | Rationale for Acceptance | Accepted By | Date | Review Date |
|---------|-----------|------------------------|------------|------|------------|
| | | | | | |

---

## Approvals

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Risk Assessment Lead | | | |
| Business / System Owner | | | |
| CISO | | | |
| [Executive Sponsor] | | | |
