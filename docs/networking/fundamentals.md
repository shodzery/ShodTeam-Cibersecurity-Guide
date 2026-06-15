# Network Security Fundamentals

## TCP/IP Stack Security

The TCP/IP model consists of four layers, each with distinct security concerns and attack surfaces.

```mermaid
graph TD
    A[Application Layer] --> B[Transport Layer]
    B --> C[Internet Layer]
    C --> D[Network Access Layer]
    
    A --> A1[HTTP/S, DNS, SMTP, SSH, FTP]
    B --> B1[TCP, UDP - ports and sessions]
    C --> C1[IP, ICMP, ARP - routing and addressing]
    D --> D1[Ethernet, Wi-Fi - physical transmission]
```

### IP Layer Attacks

**IP Spoofing**
An attacker crafts packets with a forged source IP address. Used in DDoS amplification attacks, bypass of IP-based access controls, and session hijacking in poorly implemented protocols.

*Mitigations:*
- Ingress filtering (BCP 38): routers should not forward packets where the source address does not belong to the originating network
- Reverse path forwarding (RPF) checks on routers
- Do not rely on source IP for authentication

**ICMP Abuse**
ICMP provides network diagnostics but can be misused for reconnaissance (ping sweeps), covert channels (ICMP tunneling), and historically for denial-of-service (ping flood, smurf attack).

*Mitigations:*
- Rate-limit ICMP at perimeter
- Block ICMP types not required for network operation (types 0, 3, 8, 11 are typically needed)

### TCP Layer Attacks

**TCP SYN Flood**
An attacker sends a large volume of SYN packets, leaving many half-open connections and exhausting server resources.

*Mitigations:*
- SYN cookies: allows the server to handle SYN requests without allocating state until the handshake completes
- Rate limiting at the firewall or upstream
- DDoS scrubbing services for large-scale attacks

**TCP Session Hijacking**
An attacker in a privileged network position injects packets into an existing TCP session by predicting or observing sequence numbers.

*Mitigations:*
- Encrypt all sessions (TLS) to prevent injection of meaningful data even if sequence numbers are predictable
- Strong source port randomization

---

## DNS Security

The Domain Name System (DNS) translates human-readable domain names to IP addresses. It is a critical dependency for almost every networked service and a frequent target.

### DNS Cache Poisoning

An attacker causes a DNS resolver to cache a malicious mapping, redirecting users to an attacker-controlled IP for a legitimate domain.

The Kaminsky attack (2008) demonstrated that DNS cache poisoning was far more practical than previously believed by exploiting the predictability of DNS transaction IDs and source ports.

*Mitigations:*
- **DNSSEC**: Adds cryptographic signatures to DNS records. Resolvers can verify that responses have not been tampered with. Requires zone signing and validation at each DNS tier.
- Source port randomization for DNS queries
- Response Rate Limiting (RRL) on authoritative servers

### DNS over HTTPS (DoH) and DNS over TLS (DoT)

Traditional DNS queries are sent in plaintext, visible to ISPs, network operators, and anyone with network access.

- **DNS over TLS (DoT)**: Encrypts DNS using TLS on port 853. Preserves the DNS protocol.
- **DNS over HTTPS (DoH)**: Wraps DNS in HTTPS on port 443. Blends with normal web traffic, making inspection and filtering more difficult.

**Security considerations:**
- DoH can bypass enterprise DNS controls and content filtering
- Both options move trust to the configured DNS provider rather than the network operator
- Enterprise environments should configure managed DoH/DoT resolvers rather than blocking DNS-over-HTTPS entirely

### DNS Reconnaissance

DNS provides valuable reconnaissance information:
- Zone transfers (`dig AXFR @ns1.example.com example.com`) if misconfigured can reveal all DNS records
- Subdomain enumeration reveals attack surface

*Mitigations:*
- Restrict zone transfers to authorized secondary nameservers only
- Monitor for unusual DNS query patterns

### DNS Record Types

| Record | Purpose | Security Relevance |
|--------|---------|-------------------|
| A | IPv4 address | Primary target for cache poisoning |
| AAAA | IPv6 address | Same as A |
| MX | Mail server | SPF, DKIM, DMARC configuration |
| TXT | Free text | SPF, DKIM, DMARC policies live here |
| CNAME | Canonical name alias | Subdomain takeover risk if target resource is deleted |
| NS | Nameserver | Primary target for domain hijacking |
| SOA | Start of authority | Zone transfer and serial information |
| PTR | Reverse DNS lookup | Can reveal internal hostnames |
| TLSA | TLS certificate association | DANE protocol |

**Email Authentication Records:**

- **SPF** (Sender Policy Framework): TXT record listing IP addresses authorized to send mail for a domain
- **DKIM** (DomainKeys Identified Mail): Asymmetric signature on outgoing messages, verified using public key in DNS
- **DMARC** (Domain-based Message Authentication, Reporting, and Conformance): Policy defining what to do when SPF and DKIM checks fail (quarantine, reject, none)

---

## HTTP and HTTPS

### HTTP Request/Response Structure

HTTP is the application protocol used for web communication. Understanding its structure is prerequisite to web security.

```
GET /api/v1/users HTTP/1.1
Host: example.com
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...
Accept: application/json
User-Agent: Mozilla/5.0...
```

```
HTTP/1.1 200 OK
Content-Type: application/json
X-Frame-Options: DENY
Content-Security-Policy: default-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### HTTPS and TLS

HTTPS is HTTP transported over TLS. TLS provides confidentiality, integrity, and server authentication for HTTP traffic.

**TLS Handshake (TLS 1.3):**

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    C->>S: ClientHello (supported cipher suites, key share)
    S->>C: ServerHello (selected cipher suite, key share)
    S->>C: Certificate (server certificate chain)
    S->>C: CertificateVerify (proof of private key ownership)
    S->>C: Finished
    C->>S: Finished
    C->>S: Application Data (encrypted)
    S->>C: Application Data (encrypted)
```

**TLS Configuration Best Practices:**
- Minimum TLS version: 1.2. Prefer TLS 1.3 only where possible.
- Disable: SSLv2, SSLv3, TLS 1.0, TLS 1.1
- Cipher suite order: prefer forward-secret ciphers (ECDHE, DHE) over RSA key exchange
- HSTS (HTTP Strict Transport Security): instructs browsers to only connect over HTTPS
- OCSP stapling: server includes a pre-fetched OCSP response to avoid client-side OCSP lookup latency and privacy leakage

**Deprecated and Vulnerable:**
| Protocol/Feature | Status | Reason |
|-----------------|--------|--------|
| SSLv2 | Prohibited (RFC 6176) | Multiple fundamental flaws |
| SSLv3 | Prohibited (RFC 7568) | POODLE attack |
| TLS 1.0 | Deprecated (RFC 8996) | BEAST, POODLE, other attacks |
| TLS 1.1 | Deprecated (RFC 8996) | No modern cipher support |
| RC4 | Prohibited (RFC 7465) | Practical bias attacks |
| 3DES | Avoid | SWEET32 attack |
| Export-grade ciphers | Prohibited | Intentionally weakened, FREAK/Logjam attacks |

---

## Firewalls

A firewall inspects and filters network traffic based on configured rules. Firewalls are a primary network security control.

### Firewall Types

**Stateless (Packet-Filtering) Firewall**
Examines individual packets in isolation based on IP addresses, ports, and protocol. Does not track connection state.

- Fast and lightweight
- Cannot understand application-layer context
- Cannot distinguish between a new connection and a response to an established connection without explicit rules for both directions

**Stateful Firewall**
Maintains a state table tracking active connections. Automatically permits response traffic for established connections. More flexible and accurate than stateless filtering.

**Application-Layer Firewall / Web Application Firewall (WAF)**
Inspects traffic at the application layer. Can understand HTTP, DNS, SMTP, and other protocols. A WAF specifically protects web applications by inspecting HTTP request and response content.

**Next-Generation Firewall (NGFW)**
Combines stateful inspection with application identification (independent of port), user identity awareness, SSL/TLS inspection, intrusion prevention, and threat intelligence integration.

### Firewall Rule Design

Rules should follow these principles:
1. **Least privilege**: Allow only traffic explicitly required. Default-deny is the correct stance.
2. **Specificity**: Rules should be as specific as possible (specific source, destination, port, protocol).
3. **Order matters**: In most firewall implementations, rules are evaluated top-to-bottom and the first match wins.
4. **Implicit deny**: The final rule should deny all traffic not explicitly permitted.

Example rule set structure:

```
Priority  Action  Source          Destination     Port     Protocol
1         ALLOW   10.1.0.0/24    10.2.0.10       443      TCP       # Web tier to DB
2         ALLOW   0.0.0.0/0      10.0.0.5        443      TCP       # Public HTTPS
3         ALLOW   10.1.0.0/24    0.0.0.0/0       53       UDP       # DNS resolution
4         DENY    0.0.0.0/0      0.0.0.0/0       ANY      ANY       # Default deny
```

---

## Intrusion Detection and Prevention Systems

### IDS vs IPS

| Feature | IDS | IPS |
|---------|-----|-----|
| Placement | Out-of-band (passive) | Inline (active) |
| Response | Alert only | Block and alert |
| Performance impact | Minimal | Higher (introduces latency) |
| False positive risk | Lower consequence | Can block legitimate traffic |

### Detection Methods

**Signature-based detection**
Matches traffic against known attack patterns (signatures). Highly accurate for known attacks, zero detection capability for novel techniques. Requires continuous signature updates.

**Anomaly-based detection**
Establishes a baseline of normal behavior and alerts on deviations. Can detect novel attacks but generates more false positives. Requires a learning period and careful tuning.

**Protocol analysis (specification-based)**
Compares traffic against RFC specifications for each protocol. Effective for detecting malformed packets and protocol abuse.

### Common IDS/IPS Evasion Techniques

- **Fragmentation**: Split malicious payload across multiple IP fragments; some inspection engines do not reassemble
- **TTL manipulation**: Send packets with TTLs that expire before the IDS but reach the target
- **Encoding**: Encode payloads in ways the IDS does not decode but the target does (URL encoding, Unicode normalization)
- **Protocol ambiguity**: Exploit differences in how the IDS and the target parse protocol edge cases
- **Slow attacks**: Spread attack traffic over time to avoid rate-based thresholds

---

## VPN Technologies

VPNs create encrypted tunnels over untrusted networks to provide confidentiality, integrity, and authenticated access.

### IPsec

IPsec operates at the network layer (Layer 3) and encrypts IP packets. Two modes:

- **Transport mode**: Encrypts only the payload of each IP packet. The original IP header is preserved. Used for host-to-host communication.
- **Tunnel mode**: Encrypts the entire original IP packet and encapsulates it in a new packet. Used for site-to-site VPNs and remote access gateways.

IPsec components:
- **Authentication Header (AH)**: Provides integrity and authentication, no encryption
- **Encapsulating Security Payload (ESP)**: Provides encryption, integrity, and authentication
- **IKE/IKEv2**: Key exchange protocol for establishing security associations

### WireGuard

WireGuard is a modern VPN protocol using state-of-the-art cryptography with a minimal codebase (approximately 4,000 lines of code vs. hundreds of thousands for OpenVPN/IPsec).

**Cryptographic primitives:**
- Key exchange: Curve25519
- Encryption: ChaCha20-Poly1305
- Hashing: BLAKE2s, SipHash

**Security properties:**
- Cryptographic agility is deliberately removed (no cipher negotiation), eliminating downgrade attacks
- Forward secrecy with ephemeral keys
- Silence by default: does not respond to unauthenticated packets

---

## Network Segmentation

Network segmentation divides a network into isolated zones, limiting lateral movement by attackers who have obtained access to one segment.

```mermaid
graph LR
    Internet --> DMZ[DMZ Zone]
    DMZ --> FW1[Firewall]
    FW1 --> Web[Web Tier]
    Web --> FW2[Firewall]
    FW2 --> App[Application Tier]
    App --> FW3[Firewall]
    FW3 --> DB[Database Tier]
    FW3 --> Mgmt[Management Network]
```

### VLAN-Based Segmentation

VLANs create logical network segments on shared physical infrastructure. Traffic between VLANs must traverse a Layer 3 device (router or Layer 3 switch), where access control can be applied.

**VLAN security considerations:**
- VLAN hopping: Attackers can potentially send traffic to a different VLAN via double-tagging or switch spoofing. Mitigation: disable Dynamic Trunking Protocol (DTP), use dedicated native VLANs not used for user traffic.
- Trunk port security: Restrict which VLANs are allowed on trunk ports.

### Microsegmentation

Microsegmentation applies access control at the individual workload level, often implemented through host-based firewalls, software-defined networking (SDN), or network security groups in cloud environments.

This limits lateral movement to a specific subset of reachable targets even after a host is compromised, rather than providing access to an entire network segment.
