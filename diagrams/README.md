# Diagrams

This directory contains Mermaid diagrams covering key cybersecurity architectures and processes. All diagrams are in Mermaid format and can be rendered in any Mermaid-compatible viewer or directly in GitHub Markdown.

---

## Network Security Architecture

```mermaid
graph TD
    Internet[Internet] --> |HTTPS 443| WAF[Web Application Firewall]
    Internet --> |DDoS Scrubbing| CDN[CDN / DDoS Protection]
    CDN --> WAF
    WAF --> |Filtered Traffic| FW1[Perimeter Firewall]
    
    subgraph DMZ [DMZ - Demilitarized Zone]
        FW1 --> RevProxy[Reverse Proxy / Load Balancer]
        RevProxy --> WebTier[Web Servers]
    end
    
    subgraph AppZone [Application Zone]
        WebTier --> |Internal API| AppFW[Internal Firewall]
        AppFW --> AppServers[Application Servers]
        AppServers --> CacheLayer[Cache Layer - Redis]
    end
    
    subgraph DataZone [Data Zone - Restricted]
        AppServers --> |Encrypted connection| DBServers[(Database Cluster)]
        AppServers --> |IAM-controlled| SecretsVault[Secrets Manager]
    end
    
    subgraph Mgmt [Management Network - Isolated]
        Bastion[Bastion Host / PAM] --> |Jump connection| AppServers
        Bastion --> DBServers
        SIEM[SIEM / Log Collector] --> |Receive logs| WebTier
        SIEM --> AppServers
        SIEM --> DBServers
    end
    
    Admin[Security Admin] --> |MFA + VPN| Bastion
```

---

## Incident Response Workflow

```mermaid
flowchart TD
    A[Alert Generated] --> B{Tier 1 Triage}
    B --> |False Positive| C[Document and Tune]
    B --> |Needs Investigation| D[Initial Investigation]
    
    D --> E{Confirmed Incident?}
    E --> |No| C
    E --> |Yes| F[Classify Severity]
    
    F --> G{Critical / High?}
    G --> |Yes| H[Escalate to IR Lead]
    G --> |No| I[Tier 2 Investigation]
    
    H --> J[Assemble Response Team]
    I --> K[Containment Actions]
    J --> K
    
    K --> L[Preserve Evidence]
    L --> M[Eradication]
    M --> N[Recovery]
    N --> O[Validation]
    O --> P[Post-Incident Review]
    P --> Q[Update Detections and Playbooks]
    Q --> R[Close Incident]
    
    C --> R
```

---

## Kill Chain Mapping

```mermaid
graph LR
    subgraph KC [Cyber Kill Chain]
        direction LR
        R[Reconnaissance] --> W[Weaponization]
        W --> D[Delivery]
        D --> E[Exploitation]
        E --> I[Installation]
        I --> CC[Command and Control]
        CC --> AO[Actions on Objectives]
    end
    
    subgraph Controls [Defensive Controls by Phase]
        R --> |OSINT reduction, attack surface management| RC[Recon Defense]
        W --> |Threat intelligence| WC[Weaponization Intel]
        D --> |Email gateway, web proxy, AV| DC[Delivery Defense]
        E --> |Patching, input validation, WAF| EC[Exploit Defense]
        I --> |EDR, application whitelisting| IC[Installation Defense]
        CC --> |DNS sinkholing, network monitoring, firewall| CCC[C2 Defense]
        AO --> |Data encryption, DLP, IR| AOC[Impact Defense]
    end
```

---

## PKI Certificate Lifecycle

```mermaid
stateDiagram-v2
    [*] --> KeyGeneration : Request certificate
    KeyGeneration --> CSRCreation : Generate key pair
    CSRCreation --> CASubmission : Create Certificate Signing Request
    CASubmission --> Validation : Submit to CA
    Validation --> Issued : Pass DV/OV/EV validation
    Validation --> Rejected : Fail validation
    Rejected --> CSRCreation : Re-apply
    Issued --> Active : Install on server
    Active --> Expired : Past notAfter date
    Active --> Revoked : Key compromise or policy violation
    Expired --> [*] : Renew (back to KeyGeneration)
    Revoked --> CRL : Added to CRL and OCSP
    CRL --> [*] : Superseded by new certificate
```

---

## Zero Trust Network Model

```mermaid
graph TD
    subgraph Anywhere [User - Any Location]
        Device[Managed Device]
        ID[User Identity]
    end
    
    subgraph Evaluation [Policy Evaluation]
        PDP[Policy Decision Point]
        TI[Threat Intelligence]
        DC[Device Compliance Check]
        IC[Identity Verification - MFA]
        RC[Risk Context - Location, Behavior]
    end
    
    subgraph Enforcement [Access Enforcement]
        PEP[Policy Enforcement Point - ZTNA]
    end
    
    subgraph Resources [Protected Resources]
        App1[Internal Application]
        App2[SaaS Application]
        API[API Gateway]
    end
    
    Device --> DC
    ID --> IC
    DC --> PDP
    IC --> PDP
    TI --> PDP
    RC --> PDP
    
    PDP --> |Allow / Deny / Step-up| PEP
    PEP --> |Per-session access| App1
    PEP --> |Per-session access| App2
    PEP --> |Per-session access| API
```

---

## SOC Alert Triage Flow

```mermaid
flowchart TD
    A[Alert from SIEM / EDR] --> B[Tier 1 Analyst]
    B --> C{Initial Assessment}
    
    C --> |Known False Positive| D[Close and Document]
    C --> |Obvious True Positive| E[Immediate Escalation]
    C --> |Requires Investigation| F[Investigate]
    
    F --> G{Check baseline and context}
    G --> |Normal behavior| H[Tune rule / Close]
    G --> |Abnormal but low risk| I[Monitor]
    G --> |Abnormal, significant| J[Escalate to Tier 2]
    
    J --> K[Deep Investigation]
    K --> L{Confirmed Incident?}
    
    L --> |No| M[Document findings and close]
    L --> |Yes| N[Incident Declaration]
    
    N --> O{Severity}
    O --> |Critical / High| P[Invoke IR Plan - Full Response]
    O --> |Medium| Q[IR Plan - Standard Response]
    O --> |Low| R[Handle at Tier 2]
    
    E --> P
    
    P --> S[Notify IR Lead and Management]
    Q --> T[IR Lead Notified]
    R --> U[Remediate and Document]
```

---

## MITRE ATT&CK Technique Lifecycle in Detection

```mermaid
graph LR
    A[New ATT&CK Technique Published] --> B[Threat Intelligence Review]
    B --> C{Applicable to our environment?}
    C --> |No| D[Document and Archive]
    C --> |Yes| E[Develop Detection Hypothesis]
    E --> F[Identify Required Log Sources]
    F --> G{Log sources available?}
    G --> |No| H[Onboard missing log source]
    H --> F
    G --> |Yes| I[Write Detection Rule]
    I --> J[Test Against Known-Good Baseline]
    J --> K{Acceptable False Positive Rate?}
    K --> |No| L[Tune Rule]
    L --> J
    K --> |Yes| M[Peer Review]
    M --> N[Deploy to Production SIEM]
    N --> O[Monitor Performance]
    O --> P{Rule effective?}
    P --> |Needs tuning| L
    P --> |Yes| Q[Document Coverage in ATT&CK Navigator]
```
