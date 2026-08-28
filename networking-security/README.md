# **Networking & DevSecOps - DevOps Interview Questions (200 Questions)**

Welcome to the **Networking & DevSecOps** master collection containing **200 comprehensive interview questions and detailed answers** covering Zero Trust Architecture, mTLS, SPIFFE/SPIRE, Kubernetes NetworkPolicies, HashiCorp Vault, OPA/Kyverno Policy as Code, Supply Chain Security (SLSA/SBOM), and Runtime Threat Detection (Falco).

---

## 🟢 **Part 1: Networking Fundamentals, OSI & Transport Protocols (Questions 1–60)**

### **1. Explain the 7-Layer OSI Model and the specific layers DevOps engineers interact with daily.**
**Answer:**
1. **Application (Layer 7):** HTTP, HTTPS, DNS, gRPC, SSH, TLS (Ingress, API Gateways).
2. **Presentation (Layer 6):** TLS/SSL encryption, JSON/Protobuf serialization.
3. **Session (Layer 5):** Sockets, RPC sessions.
4. **Transport (Layer 4):** TCP, UDP, QUIC (NLBs, Port forwarding).
5. **Network (Layer 3):** IP, ICMP, BGP, IPsec (VPCs, Subnets, Routers).
6. **Data Link (Layer 2):** Ethernet, MAC, ARP, VLAN (Switching, CNI virtual bridges).
7. **Physical (Layer 1):** Fiber optics, physical data center cables.
*(DevOps and Cloud Engineers primarily architect and debug systems at Layers 3, 4, and 7).*

### **2. Compare TCP vs UDP across connection state, reliability, header overhead, and production use cases.**
**Answer:**
- **TCP (Transmission Control Protocol):**
  - **Connection-Oriented:** Establishes connection via **3-Way Handshake** (`SYN` $\rightarrow$ `SYN-ACK` $\rightarrow$ `ACK`).
  - **Reliable:** Guarantees in-order delivery via sequence numbers, acknowledgment packets, and automatic retransmission of lost packets.
  - **Overhead:** 20-byte header; higher latency due to sliding window flow control and congestion management.
  - **Use Cases:** Web applications (HTTP/HTTPS), databases (PostgreSQL/MySQL), SSH, file transfers.
- **UDP (User Datagram Protocol):**
  - **Connectionless:** No handshake; "fire-and-forget" datagram delivery.
  - **Unreliable:** No retransmission, no ordering guarantee, no flow control.
  - **Overhead:** Minimal 8-byte header; ultra-low latency.
  - **Use Cases:** DNS lookups, video streaming, VoIP, gaming, and **HTTP/3 (QUIC)**.

### **3. Walk through the complete step-by-step DNS resolution lifecycle when accessing `api.stripe.com`.**
**Answer:**
1. **Local Caches:** Browser cache $\rightarrow$ OS Resolver cache $\rightarrow$ Local router DNS cache.
2. **Recursive Resolver (e.g., 8.8.8.8 / ISP DNS):** If cache misses, the recursive resolver queries:
3. **Root Name Server (`.`):** Directs resolver to the `.com` Top-Level Domain (TLD) name server.
4. **TLD Name Server (`.com`):** Directs resolver to the Authoritative Name Server for `stripe.com`.
5. **Authoritative Name Server:** Queries zone file and returns the `A` (IPv4) or `AAAA` (IPv6) record (`192.0.2.1`) along with TTL.
6. **Client Connection:** Resolver caches the IP and returns it to the client browser to initiate the TLS handshake.

### **4. Compare Public IP vs Private IP (RFC 1918) vs NAT (Network Address Translation).**
**Answer:**
- **Public IP:** Globally unique and directly routable on the public internet.
- **Private IP (RFC 1918):** Reserved for internal networks (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`); not routable on the public internet.
- **NAT:** Translates multiple private IP addresses to a single public IP address, allowing private instances to initiate outbound internet traffic (e.g., downloading OS patches) while preventing unsolicited incoming internet connections.

### **5. Explain CIDR Notation and subnet sizing calculations.**
**Answer:** CIDR represents an IP range using base IP and prefix length (`/XX` bits for network portion):
- `/24` $\rightarrow 2^{(32-24)} = 256$ IP addresses.
- `/16` $\rightarrow 2^{(32-16)} = 65,536$ IP addresses.
*(AWS VPC subnets reserve 5 IPs per subnet: Network, VPC Router, DNS, Future use, Broadcast).*

### **6. Explain the TLS 1.3 Handshake and compare it to TLS 1.2.**
**Answer:**
- **TLS 1.2 (2-RTT Handshake):** Required two full network round-trips to negotiate cipher suites and exchange keys before sending application data.
- **TLS 1.3 (1-RTT / 0-RTT Handshake):**
  1. **ClientHello:** Client sends supported ciphers and key share guess in the very first packet.
  2. **ServerHello:** Server selects cipher, returns certificate and server key share. Both compute shared secret immediately.
  3. **0-RTT Resumption:** Returning clients resume encrypted sessions in 0 round-trips.

### **7. Compare Symmetric vs Asymmetric Encryption.**
**Answer:**
- **Asymmetric Encryption (RSA, ECC):** Uses mathematically linked public/private key pairs. Slower; used for digital signatures and TLS key exchange.
- **Symmetric Encryption (AES-256, ChaCha20):** Uses a single shared key for encryption and decryption. Ultra-fast; used for encrypting bulk data in-transit and at-rest.

### **8. Compare Reverse Proxy vs Forward Proxy.**
**Answer:**
- **Forward Proxy:** Sits in front of client devices to inspect, filter, or anonymize outbound traffic to the internet (corporate enterprise proxies).
- **Reverse Proxy:** Sits in front of backend web servers to handle SSL termination, load balancing, compression, caching, and rate limiting (Nginx, Envoy).

### **9. What is a Web Application Firewall (WAF)?**
**Answer:** A Layer 7 security appliance that inspects HTTP/HTTPS traffic to block OWASP Top 10 web vulnerabilities (SQL Injection, XSS, CSRF, malicious scrapers, rate floods).

### **10. What is Zero Trust Architecture (ZTA)?**
**Answer:** A cybersecurity model based on **"Never trust, always verify"**. Eliminates the concept of a trusted internal network; every single request is authenticated, authorized, and encrypted (mTLS) regardless of whether it originates inside or outside the VPC.

### **11. What is Mutual TLS (mTLS)?**
**Answer:** Standard TLS authenticates only the server to the client. In **Mutual TLS (mTLS)**, both client and server present X.509 cryptographic certificates to each other, establishing bidirectional encryption and strict cryptographic identity verification.

### **12. What is DevSecOps and the "Shift-Left" philosophy?**
**Answer:** Integrating security controls, vulnerability testing, and compliance guardrails into the software delivery pipeline from initial code commit ("Shift-Left") rather than testing security as an afterthought before production.

### **13. Compare SAST vs DAST vs IAST vs SCA.**
**Answer:**
- **SAST (Static Analysis):** Scans source code for vulnerabilities before compilation (Semgrep, SonarQube).
- **DAST (Dynamic Analysis):** Attacks running staging applications from the outside (OWASP ZAP).
- **IAST (Interactive Analysis):** Instruments application runtime from the inside during test execution.
- **SCA (Software Composition Analysis):** Scans open-source third-party dependencies for known CVEs (Trivy, Snyk).

### **14. What is HashiCorp Vault and what core capabilities does it provide?**
**Answer:** An identity-based secrets management platform providing secure static and dynamic secret storage, automatic secret rotation, time-to-live (TTL) revocation, encryption as a service (Transit engine), and dynamic PKI X.509 certificate generation.

### **15. What is a CVE and CVSS score?**
**Answer:**
- **CVE (Common Vulnerabilities and Exposures):** Standardized identifier for a publicly known vulnerability (e.g., `CVE-2021-44228`).
- **CVSS (Common Vulnerability Scoring System):** Severity score from 0.0 to 10.0 (Low: 0.1–3.9, Medium: 4.0–6.9, High: 7.0–8.9, Critical: 9.0–10.0).

### **16. Explain Kubernetes RBAC (Role, ClusterRole, RoleBinding, ClusterRoleBinding).**
**Answer:**
- **`Role` / `ClusterRole`:** Defines permission rules (verbs: `get`, `list`, `create` on resources: `pods`, `secrets`). Scoped to a namespace (`Role`) or entire cluster (`ClusterRole`).
- **`RoleBinding` / `ClusterRoleBinding`:** Grants defined permissions to subjects (Users, Groups, ServiceAccounts).

### **17. Compare Bastion Hosts vs AWS SSM Session Manager.**
**Answer:**
- **Bastion Host:** Exposed instance in a public subnet requiring open inbound port 22 and SSH key management.
- **AWS SSM Session Manager (Modern Standard):** Agent-driven shell access with **zero public IPs, zero open inbound ports (port 22 closed)**, authenticated via AWS IAM SSO with complete session audit logging.

### **18. What is DDoS and how is it mitigated at the edge?**
**Answer:** A malicious flood overwhelming server availability using distributed botnets. Mitigated via Cloudflare / AWS Shield, Anycast routing, edge caching, rate limiting, and WAF rules.

### **19. What is a Container Escape?**
**Answer:** A vulnerability where an attacker running code inside a container exploits a kernel flaw or misconfigured privilege (`--privileged`, root execution, host volume mount) to break out of namespaces and gain full root control over the host OS.

### **20. What is Secret Masking and Secret Scanning in Git?**
**Answer:**
- **Secret Scanning (Gitleaks, Trufflehog):** Scans commit diffs for patterns matching AWS keys, SSH private keys, and API tokens, blocking pushes containing exposed secrets.
- **Secret Masking:** CI/CD feature that replaces detected sensitive variable values with `***` in build output logs.

### **21. Write a complete Kubernetes Default-Deny NetworkPolicy and explain how whitelisting works.**
**Answer:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}  # Selects all pods
  policyTypes:
    - Ingress
    - Egress
```
Once applied, all incoming and outgoing pod traffic is blocked unless explicitly whitelisted by subsequent NetworkPolicy rules.

### **22. What is Cilium NetworkPolicy and how does it provide Layer 7 security?**
**Answer:**
Standard Kubernetes NetworkPolicies only filter on IP and Port (Layer 3/4).
**Cilium L7 Policy (via eBPF):** Enforces rules on HTTP methods, URL paths, and gRPC methods:
```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l7-payment-policy
spec:
  endpointSelector:
    matchLabels: { app: payment }
  ingress:
    - fromEndpoints:
        - matchLabels: { app: frontend }
      toPorts:
        - ports: [{ port: "8080", protocol: TCP }]
          rules:
            http:
              - method: "POST"
                path: "/v1/charge"
```

### **23. What is SPIFFE/SPIRE and how does Workload Identity Attestation work?**
**Answer:** SPIFFE defines standardized cryptographic identities (`spiffe://prod.example.com/ns/finance/sa/payment-service`).
- **Node & Workload Attestation:** SPIRE agent queries the local kernel/kubelet to verify the calling process's UID, container ID, and namespace, issuing short-lived, auto-rotating X.509 SVID certificates into memory without long-lived keys on disk.

### **24. How do HashiCorp Vault Dynamic Secrets eliminate static database credentials?**
**Answer:**
1. App authenticates to Vault using its Kubernetes ServiceAccount token.
2. App requests dynamic DB credentials from Vault Database Engine.
3. Vault dynamically executes `CREATE USER 'v-token-xyz' WITH PASSWORD 'temp_pass' VALID UNTIL '...'` in PostgreSQL and returns credentials.
4. When TTL expires, Vault drops the database user automatically.

### **25. What is the Kubernetes External Secrets Operator (ESO)?**
**Answer:** Syncs secrets from external secrets managers (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault) into native Kubernetes `Secret` objects, allowing GitOps repositories to commit declarative `ExternalSecret` manifests safely without storing plaintext values in Git.

### **26. Compare OPA Gatekeeper vs Kyverno for Kubernetes Admission Control.**
**Answer:**
- **OPA Gatekeeper:** Uses **Rego** query language. Highly expressive, but steep learning curve.
- **Kyverno:** Kubernetes-native policy engine written **100% in YAML**. Easier to author and test; supports native payload mutation and image verification.

### **27. Compare CycloneDX vs SPDX SBOM formats.**
**Answer:**
- **CycloneDX (OWASP):** Lightweight, designed specifically for application security, vulnerability identification, and CI/CD automation.
- **SPDX (Linux Foundation / ISO standard):** Rich standard frequently used for open-source software licensing and intellectual property compliance.

### **28. What is Sigstore Cosign and Keyless Container Image Signing?**
**Answer:** Cosign cryptographically signs container images using OIDC tokens from GitHub Actions. Fulcio generates short-lived X.509 certificates, and Rekor records signatures in an immutable transparency log, eliminating static PGP private keys.

### **29. What is Falco and how does it detect Runtime Threats in Kubernetes?**
**Answer:** A CNCF runtime security engine that parses Linux kernel system calls via eBPF probes, triggering instant alerts on suspicious behavior (spawning `/bin/bash` in production, modifying `/etc/shadow`, unauthorized outbound connections).

### **30. Compare BGP Routing vs Anycast DNS.**
**Answer:**
- **BGP:** Core routing protocol of the internet exchanging reachability information between autonomous systems.
- **Anycast:** Multiple globally distributed servers share the exact same IP address. BGP routes client traffic to the topologically closest server, providing low latency and automatic DDoS absorption.

### **31. What is HTTP/3 (QUIC) and why is it superior to HTTP/2?**
**Answer:**
- **HTTP/2 (TCP):** Suffers from **Head-of-Line Blocking**—if one TCP packet is dropped, the entire connection stalls.
- **HTTP/3 (UDP with QUIC):** Multiplexed streams are independent. Packet loss on Stream A does not stall Stream B; provides 0-RTT connection resumption.

### **32. Explain AWS KMS Envelope Encryption.**
**Answer:**
1. KMS Customer Master Key (CMK) generates a plaintext Data Encryption Key (DEK) and an encrypted DEK.
2. Application encrypts bulk data locally in memory using the plaintext DEK.
3. The plaintext DEK is erased from memory; only the encrypted DEK is stored alongside ciphertext.

### **33. What is CIS Benchmark and how do you enforce it in Kubernetes?**
**Answer:** The Center for Internet Security (CIS) Benchmark provides prescriptive hardening rules for control plane, worker node, and etcd security, automated via tools like **`kube-bench`**.

### **34. What is a Zero-Day Vulnerability and how does Defense in Depth mitigate it?**
**Answer:** A flaw actively exploited before a patch exists. Mitigated by runtime behavioral anomaly detection (Falco), strict egress NetworkPolicies (blocking reverse shell C2 traffic), and non-root, read-only container filesystems.

### **35. What is DNSSEC?**
**Answer:** Adds cryptographic digital signatures to DNS records to protect against DNS spoofing, cache poisoning, and man-in-the-middle attacks.

### **36. Compare SQL Injection (SQLi) vs Cross-Site Scripting (XSS).**
**Answer:**
- **SQLi:** Attacker injects SQL fragments into input fields to manipulate database queries (prevented via prepared statements/parameterized queries).
- **XSS:** Attacker injects malicious JavaScript into web pages viewed by other users to steal session cookies (prevented via Content Security Policy and input sanitization).

### **37. What is a PKI and Certificate Authority (CA)?**
**Answer:** A Public Key Infrastructure manages digital certificates. A CA is a trusted entity (Let's Encrypt, DigiCert, Vault CA) that validates identities and issues cryptographically signed X.509 certificates.

### **38. Compare AWS GuardDuty vs AWS Security Hub.**
**Answer:**
- **GuardDuty:** Intelligent threat detection analyzing CloudTrail, VPC Flow Logs, and EKS audit logs via machine learning.
- **Security Hub:** Centralized dashboard aggregating compliance posture (CIS, PCI-DSS) and alerts from GuardDuty, Inspector, and Macie.

### **39. What is Trivy and what artifact types does it scan?**
**Answer:** Scans container images, Git repositories, filesystems, IaC templates (Terraform, Helm), and live Kubernetes clusters for CVE vulnerabilities and misconfigurations.

### **40. Compare SSL Termination vs End-to-End Encryption.**
**Answer:**
- **SSL Termination:** Load Balancer decrypts HTTPS and forwards unencrypted HTTP to private backend pods.
- **End-to-End Encryption:** Traffic remains encrypted all the way down to the individual application container process (mandatory for Zero Trust, HIPAA, PCI-DSS).

### **41. What is Address Resolution Protocol (ARP) and ARP Poisoning?**
**Answer:** Maps Layer 3 IP addresses to Layer 2 physical MAC addresses on a local Ethernet segment. ARP Poisoning occurs when an attacker broadcasts forged ARP messages to link their MAC address with a legitimate IP (gateway), executing Man-in-the-Middle attacks.

### **42. What is Maximum Transmission Unit (MTU) and Path MTU Discovery (PMTUD)?**
**Answer:** MTU is the largest packet size in bytes that can be transmitted over a network (standard Ethernet MTU: 1500 bytes; Jumbo Frame: 9000 bytes). PMTUD dynamically discovers the minimum MTU along the entire network path to prevent packet fragmentation.

### **43. What is TCP SYN Flood and SYN Cookies?**
**Answer:** An attacker floods a server with `SYN` packets without sending final `ACK` packets, exhausting the server's TCP half-open connection backlog table. SYN Cookies encode connection state into the `SYN-ACK` sequence number, eliminating memory allocation until the valid client `ACK` arrives.

### **44. What is Dynamic Host Configuration Protocol (DHCP)?**
**Answer:** A network protocol that dynamically assigns IP addresses, subnet masks, default gateways, and DNS server IPs to client devices on a local network.

### **45. What is a Virtual LAN (VLAN)?**
**Answer:** A logical broadcast domain created at Layer 2 across physical switches, isolating network traffic between departments or tenants on shared physical cabling.

### **46. What is Border Gateway Protocol (BGP)?**
**Answer:** The core routing protocol of the internet that exchanges routing and reachability information between Autonomous Systems (AS) using path-vector algorithms.

### **47. What is Internet Control Message Protocol (ICMP)?**
**Answer:** A network-layer protocol used by network devices to send error messages and operational information (e.g., `ping` uses ICMP Echo Request/Reply; `traceroute` uses ICMP Time Exceeded).

### **48. What is a Subnet Mask and Default Gateway?**
**Answer:**
- **Subnet Mask:** Defines which portion of an IP address belongs to the network and which portion belongs to the host (e.g., `255.255.255.0` for `/24`).
- **Default Gateway:** The router IP address that a host sends packets to when destination IPs lie outside the local subnet.

### **49. What is DNS CNAME vs A vs AAAA vs ALIAS Record?**
**Answer:**
- **`A`:** Maps a hostname to an IPv4 address.
- **`AAAA`:** Maps a hostname to an IPv6 address.
- **`CNAME`:** Maps a hostname to another canonical hostname (cannot be placed at root apex domain).
- **`ALIAS` (Route 53):** AWS-specific record mapping root apex domains (`example.com`) directly to AWS resources (CloudFront, ALB).

### **50. What is DNS TTL (Time to Live)?**
**Answer:** The duration in seconds that DNS resolvers and client caches are allowed to cache a DNS record before querying authoritative name servers again.

### **51. What is OpenID Connect (OIDC) vs OAuth 2.0?**
**Answer:**
- **OAuth 2.0:** Framework for authorization (issuing Access Tokens for API access).
- **OIDC:** Identity layer built on top of OAuth 2.0 (issuing ID Tokens for user authentication).

### **52. What is SAML 2.0 (Security Assertion Markup Language)?**
**Answer:** An XML-based open standard for exchanging authentication and authorization data between an Identity Provider (IdP) and a Service Provider (SP), heavily used in enterprise SSO.

### **53. What is JSON Web Token (JWT) Structure?**
**Answer:** Base64URL encoded string with three parts separated by dots: **Header** (algorithm), **Payload** (claims: `sub`, `exp`, `iss`), and **Signature** (HMAC/RSA hash).

### **54. What is HashiCorp Vault Transit Secrets Engine?**
**Answer:** Provides "Cryptography as a Service"—encrypts and decrypts application data in-transit via API calls without storing the encryption keys on application servers.

### **55. What is HashiCorp Vault PKI Secrets Engine?**
**Answer:** Dynamically generates X.509 TLS certificates on-demand with short lifespans (e.g., 24 hours), automating internal certificate authority operations.

### **56. What is Kyverno Mutation Rule?**
**Answer:** Automatically modifies or injects default fields into Kubernetes manifests upon creation (e.g., auto-injecting resource requests, labels, or security contexts).

### **57. What is Open Policy Agent (OPA) Rego Language?**
**Answer:** A declarative query language designed for expressing complex structural and semantic compliance policies over JSON/YAML data.

### **58. What is Trivy Secret Scanning?**
**Answer:** Scans container image layers and Git filesystems for exposed private keys, AWS tokens, and Slack API tokens during CI.

### **59. What is Gitleaks Pre-Commit Hook?**
**Answer:** Blocks developers from executing `git commit` if high-entropy secrets or private tokens are detected in staged files locally.

### **60. What is Trufflehog?**
**Answer:** An open-source secret scanning tool that searches Git repositories, S3 buckets, and filesystems for exposed secrets and actively verifies if discovered keys are live against cloud APIs.

---

## 🟡 **Part 2: Kubernetes Security, Zero Trust & PKI (Questions 61–130)**

### **61. Scenario: Your Kubernetes cluster is hit with a container escape attack. How does Defense-in-Depth contain the blast radius?**
**Answer:**
1. **Rootless Execution (`runAsNonRoot: true`):** Attacker lands on host OS as an unprivileged user (UID 10001).
2. **Read-Only Root Filesystem (`readOnlyRootFilesystem: true`):** Attacker cannot download or execute malicious scripts in `/tmp`.
3. **Dropped Capabilities (`capabilities.drop: ["ALL"]`):** Stripping `CAP_SYS_ADMIN` disables kernel exploitation vectors.
4. **Sandboxed Runtime (gVisor):** Intercepts system calls in userspace sandbox.
5. **Egress NetworkPolicy:** Blocks outbound internet access, preventing reverse shell C2 connections.

### **62. Scenario: Design an automated End-to-End Supply Chain Security Pipeline conforming to SLSA Level 3.**
**Answer:**
```
[ Developer Commit ] ➔ [ GitHub PR (2-Person Review) ]
                             │
                             ▼
[ Ephemeral GitHub Actions Build Runner (Isolated) ]
  ├── 1. Generate CycloneDX SBOM (Syft)
  ├── 2. Build Container Image (Hermetic Dockerfile)
  ├── 3. Sign Image & SBOM with Cosign (Fulcio OIDC Keyless)
  └── 4. Push In-Toto Provenance to Rekor Transparency Log
                             │
                             ▼
[ OCI Registry (Amazon ECR / GHCR) ]
                             │
                             ▼
[ Kubernetes Admission Controller (Kyverno) ]
  └── Policy: Verify Cosign Signature & Provenance before admitting Pod
```

### **63. Scenario: Migrate a legacy application with hardcoded database passwords to HashiCorp Vault Dynamic Secrets with zero downtime.**
**Answer:**
1. Deploy External Secrets Operator (ESO) / Vault Agent Sidecar injecting credentials into an in-memory volume (`/vault/secrets/database.env`).
2. Configure PostgreSQL to accept both legacy static passwords and dynamic Vault-generated user roles concurrently.
3. Update application to read dynamic credentials from `/vault/secrets/database.env`.
4. Deploy pods rolling fashion.
5. Delete legacy static database user after all old pods terminate.

### **64. How do you mitigate TCP SYN Flood DDoS attacks at the Linux Kernel and Load Balancer layers?**
**Answer:**
1. **Enable SYN Cookies:** `sysctl -w net.ipv4.tcp_syncookies=1` (server encodes state into `SYN-ACK` sequence number instead of allocating half-open connection memory).
2. **Tune Kernel Backlog:** `sysctl -w net.ipv4.tcp_max_syn_backlog=4096` and `sysctl -w net.core.somaxconn=4096`.
3. **Cloudflare / CloudFront Anycast:** Absorbs flood traffic at the global edge network before reaching origin servers.

### **65. How do you implement Zero-Trust Microsegmentation in Kubernetes using Cilium and WireGuard Transparent Encryption?**
**Answer:**
- **Cilium eBPF:** Enforces Layer 3 to Layer 7 NetworkPolicies without modifying application pods.
- **WireGuard Encryption:** Set `encryption.type: wireguard` in Cilium Helm values to automatically encrypt all node-to-node and pod-to-pod network traffic in kernel-space with zero proxy sidecar overhead.

### **66. Scenario: An audit discovers that engineers have direct SSH access to production EC2 instances. Implement a Zero-Trust Access Gateway with Teleport or AWS SSM.**
**Answer:**
1. Delete security group rules allowing inbound port 22; disable public IPs.
2. Deploy AWS SSM Managed Instance Core or Teleport Auth/Proxy service.
3. Integrate with corporate Okta / Entra ID SSO with mandatory MFA.
4. Engineers authenticate via SSO to receive short-lived SSH certificates (valid for 8 hours); all interactive sessions are recorded for compliance auditing.

### **67. What is Secrets Sprawl and how do you design an enterprise remediation pipeline?**
**Answer:**
1. **Pre-Commit:** Developers install `pre-commit` running `gitleaks protect` locally.
2. **Push Protection:** GitHub blocks commits containing detected high-entropy keys.
3. **Scheduled Scanning:** Nightly org-wide Trufflehog scans across all repositories.
4. **Automated Key Revocation:** Trigger automated Lambda functions via webhook to deactivate exposed keys within 60 seconds of commit detection.

### **68. What is DNS Tunneling / DNS Exfiltration and how is it blocked?**
**Answer:**
- **Attack:** Malware encodes stolen sensitive data into DNS subdomains (`stolen_data.evil-domain.com`) and queries an attacker-controlled authoritative name server, bypassing standard firewall HTTP filters.
- **Defense:** Enforce AWS Route 53 Resolver DNS Firewall / Pi-hole; detect high volumes of high-entropy subdomain queries and non-standard DNS query types (`TXT`, `NULL`).

### **69. Scenario: Securely connect two Kubernetes clusters in different AWS VPCs without public internet exposure.**
**Answer:**
1. Connect the two VPCs via **AWS Transit Gateway** or **VPC Peering** with non-overlapping CIDR blocks.
2. Use **Cilium Cluster Mesh** or **Submariner** for cross-cluster pod IP routing and service discovery.
3. Enforce mutual TLS (mTLS) across clusters using a shared root SPIFFE/SPIRE trust bundle or Istio Multi-Cluster Mesh.

### **70. What is Kubernetes Pod Security Standards (Privileged vs Baseline vs Restricted)?**
**Answer:**
- **Privileged:** Unrestricted; permits privileged containers, host networking, hostPath volumes.
- **Baseline:** Minimally restrictive default preventing known privilege escalation vectors.
- **Restricted:** Hardened standard mandating non-root execution, dropping all capabilities, read-only root filesystems, and seccomp profiles.

---

## 🔴 **Part 3: Advanced DevSecOps, Cryptography & Threat Detection (Questions 71–200)**

### **71. What is In-Toto Attestation Framework?**
**Answer:** A framework for cryptographically certifying the chain of custody across the entire software supply chain, recording metadata at each step (source commit, build runner, test execution).

### **72. What is Rekor Transparency Log?**
**Answer:** A public, append-only, tamper-evident cryptographic Merkle tree ledger that records software supply chain metadata and Cosign signatures.

### **73. What is Fulcio Certificate Authority?**
**Answer:** A free, public Certificate Authority managed by Sigstore that issues short-lived (10-minute) X.509 certificates linked to OIDC identities (GitHub Actions).

### **74. What is HashiCorp Vault Shamir's Secret Sharing?**
**Answer:** Cryptographic algorithm that splits the master unseal key into $N$ key shares, requiring a minimum threshold of $M$ keys (e.g., 3 of 5) to reconstruct the master key and unseal Vault.

### **75. What is Vault Auto-Unseal?**
**Answer:** Automates unsealing HashiCorp Vault using cloud KMS services (AWS KMS, Azure Key Vault, Google Cloud KMS) without manual human key entry.

### **76. What is Kubernetes KMS v2 Provider?**
**Answer:** High-performance encryption at rest API in Kubernetes 1.28+ that encrypts Secret resources in etcd using envelope encryption with cached DEKs.

### **77. What is AWS IAM Session Policies?**
**Answer:** Advanced policies passed when programmatically assuming an IAM role to further restrict the permissions granted by the role's identity-based policy for that specific session.

### **78. What is Confused Deputy Problem in Cloud IAM?**
**Answer:** A security issue where an entity that doesn't have permission to perform an action coerces a privileged service into performing it (mitigated using `sts:ExternalId` in IAM trust policies).

### **79. What is Certificate Transparency (CT)?**
**Answer:** An open framework of public, append-only logs recording all issued SSL/TLS certificates, preventing CAs from issuing fraudulent certificates secretly.

### **80. What is OCSP Stapling (Online Certificate Status Protocol)?**
**Answer:** A mechanism where the web server queries the CA periodically for certificate revocation status and attaches ("staples") the timestamped OCSP response directly to the TLS handshake, improving client speed and privacy.

### **81. What is HTTP Strict Transport Security (HSTS)?**
**Answer:** A security response header (`Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`) informing client browsers to communicate exclusively over HTTPS, blocking SSL stripping attacks.

### **82. What is Content Security Policy (CSP)?**
**Answer:** An HTTP header restricting the resources (scripts, images, stylesheets) that a browser is allowed to load, mitigating Cross-Site Scripting (XSS) and data injection attacks.

### **83. What is Cross-Origin Resource Sharing (CORS)?**
**Answer:** A browser security mechanism that uses HTTP headers (`Access-Control-Allow-Origin`) to allow or restrict resources requested on a web page from a different domain.

### **84. What is SameSite Cookie Attribute (`Strict`, `Lax`, `None`)?**
**Answer:** Protects against Cross-Site Request Forgery (CSRF) by controlling whether cookies are sent with cross-site requests.

### **85. What is Linux Capabilities Dropping in Containers?**
**Answer:** Explicitly stripping POSIX capabilities (`--cap-drop=ALL`) from container processes to prevent kernel exploitation during escapes.

### **86. What is Linux Seccomp (Secure Computing Mode) BPF Filter?**
**Answer:** Using Berkeley Packet Filters to intercept and allow or deny system calls invoked by container processes (e.g., blocking `reboot`, `sys_chroot`).

### **87. What is AppArmor Profile for Docker?**
**Answer:** A Linux kernel security module enforcing filesystem path access restrictions, networking limits, and raw socket creation rules on container processes.

### **88. What is SELinux Mandatory Access Control (MAC)?**
**Answer:** Assigns security labels (`system_u:system_r:container_t`) to processes and files, enforcing strict kernel authorization rules regardless of Linux root permissions.

### **89. What is eBPF Falco Rule Syntax?**
**Answer:** Declarative YAML rules defining matching conditions over kernel syscalls:
```yaml
- rule: Unauthorized Shell in Container
  desc: Detect shell spawned inside production container
  condition: container and proc.name in (bash, sh, zsh)
  output: Shell spawned (user=%user.name pod=%k8s.pod.name)
  priority: WARNING
```

### **90. What is BGP Route Hijacking?**
**Answer:** An attacker falsely announcing ownership of IP address blocks to upstream BGP routers, intercepting or blackholing internet traffic (mitigated via RPKI).

### **91. What is Resource Public Key Infrastructure (RPKI)?**
**Answer:** A cryptographic framework securing internet BGP routing by validating that an Autonomous System is authorized to announce specific IP address prefixes.

### **92. What is Anycast BGP Routing in Cloudflare?**
**Answer:** Multiple globally distributed edge servers announcing the exact same IP address via BGP, allowing client traffic to route to the topologically nearest data center.

### **93. What is WireGuard VPN Protocol?**
**Answer:** An ultra-fast, modern cryptographic VPN protocol running in Linux kernel-space using state-of-the-art cryptography (Curve25519, ChaCha20-Poly1305, BLAKE2s).

### **94. What is IPsec ESP (Encapsulating Security Payload)?**
**Answer:** An IPsec protocol providing data confidentiality, origin authentication, and anti-replay protection for IP network packets.

### **95. What is IPsec IKEv2 (Internet Key Exchange)?**
**Answer:** The protocol used to establish IPsec security associations (SAs), negotiating encryption algorithms and authenticating endpoints.

### **96. What is Generic Routing Encapsulation (GRE)?**
**Answer:** A tunneling protocol that encapsulates a wide variety of network-layer protocols inside virtual point-to-point links over IP networks.

### **97. What is VXLAN (Virtual Extensible LAN)?**
**Answer:** An overlay network encapsulation protocol that packages Layer 2 Ethernet frames inside Layer 4 UDP packets (port 4789), scaling network segmentation from 4,096 VLANs to 16 million VXLAN segments.

### **98. What is Geneve Protocol in CNI Networking?**
**Answer:** Generic Network Virtualization Encapsulation—a flexible network virtualization protocol used by Cilium and OVN to transmit metadata across overlay networks.

### **99. What is Proxy Protocol (v1 and v2)?**
**Answer:** A protocol prepending a header to TCP connections that conveys client connection information (real client source IP and port) across Layer 4 proxies (NLB/HAProxy) to backend servers.

### **100. What is Server Name Indication (SNI)?**
**Answer:** An extension to the TLS protocol where the client indicates the hostname it is connecting to in the `ClientHello` packet, allowing reverse proxies to serve multiple SSL certificates on a single IP address.

### **101. What is TCP Keepalive?**
**Answer:** Periodic probe packets sent over idle TCP connections to verify that the remote peer is still alive and prevent intermediate firewalls from dropping idle NAT entries.

### **102. What is TCP Window Size and Flow Control?**
**Answer:** The amount of data in bytes that a receiver is willing to buffer without receiving an acknowledgment, preventing fast senders from overwhelming slow receivers.

### **103. What is TCP Congestion Control (BBR vs CUBIC)?**
**Answer:**
- **CUBIC:** Loss-based congestion algorithm; scales back throughput when packet loss occurs.
- **BBR (Bottleneck Bandwidth and RTT):** Model-based congestion algorithm developed by Google that optimizes throughput and minimizes queue latency regardless of packet loss.

### **104. What is Ephemeral Port Exhaustion?**
**Answer:** When high volumes of short-lived outbound connections consume all available OS client ports (`1024-65535`), causing connection failures (`EADDRNOTAVAIL`). Prevented by enabling `tcp_tw_reuse` and using connection pooling.

### **105. What is TIME_WAIT Socket State in TCP?**
**Answer:** The final TCP connection state where the closing socket waits for $2 \times \text{MSL}$ (Maximum Segment Lifetime, usually 60s) to ensure delayed packets in-flight do not interfere with new connections.

### **106. What is SO_REUSEPORT Socket Option?**
**Answer:** Allows multiple application processes/threads on the same machine to bind to the exact same TCP/UDP port, with the Linux kernel balancing incoming connections across processes in kernel-space.

### **107. What is TCP Splicing (Zero-Copy)?**
**Answer:** Transferring data directly between two network sockets in kernel space (`splice()` syscall) without copying data into userspace memory buffers.

### **108. What is Network Interface Card (NIC) Offloading (TSO, LRO, Checksum Offload)?**
**Answer:** Delegating network packet processing tasks (TCP segmentation, checksum computation) to physical NIC hardware to save host CPU cycles.

### **109. What is Single Root I/O Virtualization (SR-IOV)?**
**Answer:** A hardware specification allowing a physical PCIe network card to present itself as multiple virtual network cards (Virtual Functions) directly to VMs/containers, bypassing hypervisor latency.

### **110. What is Data Plane Development Kit (DPDK)?**
**Answer:** A set of libraries that enables fast packet processing by moving network drivers into userspace and using poll-mode drivers, bypassing the Linux kernel network stack.

### **111. What is Linux Socket Filtering with eBPF (sockops)?**
**Answer:** Accelerates local inter-pod communication by redirecting packets directly from sending sockets to receiving sockets within kernel memory, bypassing the entire TCP/IP network stack.

### **112. What is BPF Compiler Collection (BCC)?**
**Answer:** A toolkit for authoring eBPF programs in C and Python to execute system analysis and network traffic inspection.

### **113. What is Calico Felix Agent?**
**Answer:** The daemon running on each node in a Calico CNI cluster that translates declarative Kubernetes NetworkPolicies into Linux iptables rules or eBPF maps.

### **114. What is HashiCorp Vault Agent Auto-Auth?**
**Answer:** Automates authentication of application workloads to Vault using Kubernetes ServiceAccount tokens or cloud IAM instance profiles.

### **115. What is Vault Response Wrapping?**
**Answer:** Encrypts a secret into a single-use token with a short TTL (e.g., 5 minutes). If an attacker intercepts the token, attempting to unwrap it alerts the legitimate recipient that the secret was compromised.

### **116. What is HashiCorp Vault Lease ID and Revocation?**
**Answer:** Every dynamic secret has an associated Lease ID. Applications must renew the lease before expiration; revoking the lease drops the associated database credentials immediately.

### **117. What is HashiCorp Vault Performance Replication vs Disaster Recovery Replication?**
**Answer:**
- **Performance Replication:** Replicates secrets to secondary clusters for local low-latency reads.
- **Disaster Recovery (DR) Replication:** Replicates secrets and configuration to a secondary standby cluster that can be promoted to Primary during total outages.

### **118. What is HashiCorp Sentinel vs OPA Rego?**
**Answer:**
- **Sentinel:** Proprietary HashiCorp policy engine embedded in Terraform Cloud and Vault Enterprise.
- **OPA Rego:** CNCF open-source, general-purpose policy engine supported across Kubernetes, Terraform, and cloud gateways.

### **119. What is Kyverno Generate Rule?**
**Answer:** Automatically generates new Kubernetes resources (e.g., generating default NetworkPolicies, ResourceQuotas, or Secret syncs) whenever a new Namespace is created.

### **120. What is Kyverno Pod Security Standards Validation?**
**Answer:** Pre-configured Kyverno policy packs that enforce CIS Kubernetes Benchmarks and Pod Security Standards in pure YAML.

### **121. What is Trivy Operator for Kubernetes?**
**Answer:** An in-cluster operator that continuously scans running workloads and container images for vulnerabilities, publishing results as native Kubernetes Custom Resources (`VulnerabilityReport`).

### **122. What is Grype SBOM Vulnerability Matching?**
**Answer:** Fast vulnerability scanner developed by Anchore that scans CycloneDX/SPDX SBOM files against upstream CVE databases without pulling container image layers.

### **123. What is Semgrep for SAST?**
**Answer:** A lightweight, fast static analysis tool matching code patterns using syntax tree queries to find security bugs across multiple languages.

### **124. What is CodeQL in GitHub Advanced Security?**
**Answer:** A semantic code analysis engine that treats application source code as a queryable database, discovering data flow paths and SQL injection vulnerabilities.

### **125. What is Dependabot Security Advisory Alert?**
**Answer:** Automated notifications in GitHub alerting developers when a repository dependency has a published CVE, submitting automated PRs with version patches.

### **126. What is GitHub Secret Push Protection?**
**Answer:** Scans commit payloads during `git push` and blocks the push if high-entropy secrets (AWS keys, Slack tokens) are detected.

### **127. What is Trufflehog Canary Tokens?**
**Answer:** Deceptively placed fake credentials (honeytokens) across infrastructure that trigger high-priority alerts to SecOps if an attacker attempts to use them.

### **128. What is Teleport Access Plane?**
**Answer:** An identity-native infrastructure access gateway providing zero-trust SSH, Kubernetes, database, and web access with short-lived X.509/SSH certificates and session recording.

### **129. What is Cloudflare Zero Trust (Access / Tunnel)?**
**Answer:** Replaces legacy corporate VPNs by exposing internal web applications securely through outbound-only encrypted tunnels behind cloud identity providers (Okta, Google Workspace).

### **130. What is AWS Verified Access?**
**Answer:** Provides secure, VPN-less access to internal corporate web applications using AWS IAM and third-party identity and device health signals.

### **131. What is DNS over HTTPS (DoH) vs DNS over TLS (DoT)?**
**Answer:**
- **DoH (Port 443):** Encapsulates DNS queries inside standard HTTPS traffic, blending in with web traffic.
- **DoT (Port 853):** Encrypts DNS queries directly over dedicated TLS connections.

### **132. What is DNS Anycast Routing in Root Servers?**
**Answer:** The 13 root name server IP addresses (`A` through `M`) are distributed across thousands of physical server nodes globally using Anycast BGP routing.

### **133. What is BGP Multi-Exit Discriminator (MED)?**
**Answer:** An optional non-transitive BGP attribute used to suggest to external peers which path is preferred into your Autonomous System when multiple ingress links exist.

### **134. What is BGP AS-Path Prepending?**
**Answer:** Artificially lengthening the AS-Path of advertised BGP routes by repeating your own AS number, making that path less attractive to external routers to establish primary/backup links.

### **135. What is BGP Local Preference?**
**Answer:** A BGP attribute configured locally within an Autonomous System to dictate which outbound exit path is preferred for outbound traffic (higher value is preferred).

### **136. What is IPsec Dead Peer Detection (DPD)?**
**Answer:** Periodic keepalive messages exchanged between IPsec endpoints to verify that the remote VPN gateway is reachable and drop stale security associations during network failures.

### **137. What is IPsec Perfect Forward Secrecy (PFS)?**
**Answer:** Ensures that the compromise of a long-term private key does not compromise past encrypted session keys, generating unique ephemeral Diffie-Hellman keys for every session.

### **138. What is SSL Stripping Attack?**
**Answer:** An attacker intercepts the initial unencrypted HTTP connection and prevents the client from upgrading to HTTPS, communicating with the server over HTTPS while proxying plain HTTP to the client (mitigated by HSTS).

### **139. What is Certificate Revocation List (CRL)?**
**Answer:** A signed, timestamped list published by a Certificate Authority containing serial numbers of certificates that have been revoked before their expiration date.

### **140. What is Let's Encrypt ACME Protocol (Automated Certificate Management Environment)?**
**Answer:** An automated protocol allowing web servers and cert-manager to request, validate (via HTTP-01 or DNS-01 challenges), and install free X.509 TLS certificates automatically.

### **141. What is ACME HTTP-01 vs DNS-01 Challenge?**
**Answer:**
- **HTTP-01:** Let's Encrypt verifies domain ownership by fetching a token hosted on `http://<domain>/.well-known/acme-challenge/`.
- **DNS-01:** Let's Encrypt verifies ownership by checking for a `_acme-challenge.<domain>` TXT record (required for Wildcard certificates).

### **142. What is Elliptic Curve Cryptography (ECC) vs RSA?**
**Answer:** ECC (e.g., ECDSA with Curve25519) provides equivalent cryptographic security to RSA with significantly smaller key sizes (256-bit ECC $\approx$ 3072-bit RSA), resulting in faster TLS handshakes and reduced CPU utilization.

### **143. What is ChaCha20-Poly1305 Cipher Suite?**
**Answer:** A high-speed symmetric stream cipher with authenticated data, optimized for mobile devices and processors lacking dedicated AES hardware acceleration.

### **144. What is Quantum-Resistant Cryptography (Post-Quantum Cryptography)?**
**Answer:** Cryptographic algorithms (e.g., Kyber, Dilithium) designed to withstand attacks by future quantum computers executing Shor's algorithm.

### **145. What is Diffie-Hellman Ephemeral (DHE / ECDHE)?**
**Answer:** Key exchange protocol where client and server exchange public parameters to establish a shared symmetric secret over an untrusted network with Perfect Forward Secrecy.

### **146. What is Hardware Security Module (HSM)?**
**Answer:** A dedicated physical cryptographic processor that securely generates, stores, and manages digital keys inside tamper-resistant hardware boundaries (FIPS 140-2 Level 3).

### **147. What is AWS KMS Multi-Party Computation (MPC)?**
**Answer:** Cryptographic protocol allowing multiple parties to compute a function over their inputs while keeping those inputs private, used in digital asset custody.

### **148. What is Zero-Knowledge Proof (ZKP)?**
**Answer:** A cryptographic method by which one party (prover) can prove to another party (verifier) that a statement is true without revealing any information beyond the validity of the statement.

### **149. What is HashiCorp Vault Key-Value (KV) v2 Secrets Engine?**
**Answer:** Provides versioned key-value storage with support for historical secret revisions, soft deletion, and undeletion.

### **150. What is HashiCorp Vault Cubbyhole Secrets Engine?**
**Answer:** An internal storage area scoped strictly to a specific authentication token, completely inaccessible to any other token.

### **151. What is HashiCorp Vault Token TTL and Max TTL?**
**Answer:**
- **TTL:** The lifespan of a token; can be renewed before expiration.
- **Max TTL:** The hard upper limit after which a token cannot be renewed and is permanently revoked.

### **152. What is HashiCorp Vault Periodic Token?**
**Answer:** A long-running service token that can be renewed indefinitely within its specified renewal interval without reaching a Max TTL limit.

### **153. What is HashiCorp Vault Entity and Alias?**
**Answer:**
- **Entity:** Represents an individual human or machine identity inside Vault.
- **Alias:** Maps identity records from external authentication backends (GitHub, Okta, Kubernetes) to a single unified Vault Entity.

### **154. What is HashiCorp Vault Sentinel Integration?**
**Answer:** Enforcing fine-grained organizational policy rules (Role-Governing Policies - RGPs) evaluated before Vault allows secret read/write operations.

### **155. What is HashiCorp Vault Transform Secrets Engine?**
**Answer:** Provides Format-Preserving Encryption (FPE) and data tokenization, encrypting sensitive fields (credit card numbers) while preserving data formats.

### **156. What is HashiCorp Vault KMIP Secrets Engine?**
**Answer:** Acts as a centralized Key Management Interoperability Protocol (KMIP) server for managing encryption keys used by VMware vSphere and storage arrays.

### **157. What is Kyverno ClusterAdmissionReport?**
**Answer:** A Kubernetes CRD generated by Kyverno listing all live resources in the cluster that violate defined governance policies.

### **158. What is Open Policy Agent Gatekeeper Constraint?**
**Answer:** The declarative instantiation of a `ConstraintTemplate` that targets specific Kubernetes kinds and namespaces with enforcement actions (`deny`, `warn`, `dryrun`).

### **159. What is Falco gRPC Output?**
**Answer:** Streaming runtime threat alerts from Falco directly to centralized SIEM platforms or automated remediation operators via gRPC.

### **160. What is Falco Sidekick?**
**Answer:** An open-source companion daemon that routes Falco security alerts to over 50 output destinations (Slack, PagerDuty, AWS Lambda, Kafka, Datadog).

### **161. What is Kubernetes Security Context `readOnlyRootFilesystem`?**
**Answer:** Enforces that the container's root filesystem is mounted as read-only, preventing attackers from downloading malware or backdoors into `/tmp` or `/bin`.

### **162. What is Kubernetes Security Context `allowPrivilegeEscalation: false`?**
**Answer:** Prevents child processes from gaining more privileges than their parent process (disables `setuid` binary execution).

### **163. What is Linux Network Namespace Veth Pair?**
**Answer:** Virtual Ethernet cable connecting a container's private network namespace to the host bridge or CNI router.

### **164. What is Linux Bridge (`brctl`) in Container Networking?**
**Answer:** A software-based Layer 2 switch created on the host OS to forward Ethernet frames between container veth interfaces and physical NICs.

### **165. What is Linux Iptables Connection Tracking (`conntrack`)?**
**Answer:** A kernel subsystem that tracks the state of all active network connections (NEW, ESTABLISHED, RELATED) to implement stateful firewall filtering and NAT.

### **166. What is Conntrack Table Exhaustion?**
**Answer:** When high concurrency network traffic exceeds the maximum entries in `/proc/sys/net/netfilter/nf_conntrack_max`, causing the Linux kernel to drop new packets.

### **167. What is Linux IP Forwarding (`net.ipv4.ip_forward=1`)?**
**Answer:** A kernel parameter allowing the Linux host to forward network packets between different network interfaces, required for container routing and Kubernetes CNIs.

### **168. What is Linux Reverse Path Filtering (`rp_filter`)?**
**Answer:** A kernel security feature that validates whether incoming packets on an interface arrive from an IP that matches the host's return route, mitigating IP spoofing.

### **169. What is Linux TCP Fast Open (TFO)?**
**Answer:** Enables data exchange directly within the initial TCP `SYN` packet for returning clients with valid TFO cookies, eliminating 1 RTT during connection setup.

### **170. What is Linux TCP SACK (Selective Acknowledgment)?**
**Answer:** Allows a receiver to acknowledge out-of-order packets, enabling the sender to retransmit *only* the specific missing segments rather than the entire window.

### **171. What is Linux Epoll vs Select?**
**Answer:** `select()` scales $O(N)$ with monitored file descriptors; `epoll()` uses event-driven kernel callbacks scaling $O(1)$, enabling servers to handle hundreds of thousands of concurrent connections.

### **172. What is Linux Socket SO_LINGER Option?**
**Answer:** Dictates whether an application process blocks on `close()` to wait for unsent data to transmit, or immediately sends a `RST` packet to terminate the socket.

### **173. What is TLS 1.3 Key Schedule?**
**Answer:** The cryptographic key derivation process utilizing HKDF (HMAC-based Key Derivation Function) to derive handshake and application traffic keys.

### **174. What is Encrypted Server Name Indication (ESNI) / Encrypted Client Hello (ECH)?**
**Answer:** Encrypts the destination hostname in the TLS `ClientHello` packet, preventing eavesdroppers and ISPs from observing which web domain a user is visiting.

### **175. What is HTTP/2 HPACK Header Compression?**
**Answer:** Compresses HTTP headers using Huffman coding and shared client/server dynamic header tables, eliminating redundant header transmission across requests.

### **176. What is HTTP/3 QPACK Header Compression?**
**Answer:** A header compression format designed specifically for QUIC over UDP that eliminates Head-of-Line blocking between independent streams.

### **177. What is DNS Rebinding Attack?**
**Answer:** An attacker tricks a client browser into making requests to an external domain that dynamically changes its DNS `A` record to `127.0.0.1` (localhost) or internal private IPs, bypassing the Same-Origin Policy.

### **178. What is Cross-Site Request Forgery (CSRF)?**
**Answer:** Tricking an authenticated user's browser into submitting unauthorized requests to a trusted web application using existing session cookies.

### **179. What is Server-Side Request Forgery (SSRF)?**
**Answer:** An attacker forces a backend web server to execute network requests to internal systems (e.g., querying `http://169.254.169.254` to steal AWS EC2 metadata credentials).

### **180. What is Command Injection Attack?**
**Answer:** An attacker executes arbitrary shell commands on the host OS by injecting malicious characters (`;`, `&&`, `|`) into un-sanitized user inputs passed to system execution functions.

### **181. What is Path Traversal Attack (Directory Traversal)?**
**Answer:** An attacker uses `../` sequences in input fields to access unauthorized files outside the web root directory (e.g., `/etc/passwd`).

### **182. What is Insecure Deserialization?**
**Answer:** An attacker passes crafted serialized objects (Python pickle, Java serialization) that execute malicious code during object reconstruction.

### **183. What is XML External Entity (XXE) Injection?**
**Answer:** Exploiting vulnerable XML parsers that process external entity references to read local server files or execute internal port scans.

### **184. What is Man-in-the-Middle (MitM) Attack?**
**Answer:** An attacker intercepts and potentially alters communication between two systems without their knowledge (mitigated via mTLS and certificate pinning).

### **185. What is SSL Strip / Downgrade Attack?**
**Answer:** An attacker intercepts TLS negotiation and forces the connection to fall back to insecure, legacy cipher suites or unencrypted HTTP.

### **186. What is Memory Corruption Attack (Buffer Overflow)?**
**Answer:** Writing more data into a memory buffer than allocated, overwriting adjacent memory and executing arbitrary payload code (mitigated via ASLR and memory-safe languages like Rust/Go).

### **187. What is Address Space Layout Randomization (ASLR)?**
**Answer:** A Linux kernel security feature that randomizes the memory addresses of program execution areas (stack, heap, libraries), preventing buffer overflow exploits.

### **188. What is Non-Executable Stack (NX / DEP)?**
**Answer:** Hardware feature marking data memory pages as non-executable, preventing code injected into buffers from executing.

### **189. What is Linux Kernel Module (LKM) Rootkit?**
**Answer:** Malicious code loaded directly into kernel memory that hooks system calls and hides processes, files, and network connections from userland monitoring tools.

### **190. What is eBPF Rootkit?**
**Answer:** Stealthy malware loaded into eBPF kernel hooks that modifies packet payloads or hides processes directly in kernel memory without modifying disk files.

### **191. What is Supply Chain Dependency Confusion Attack?**
**Answer:** An attacker registers a malicious package on a public repository (npm/PyPI) with the exact same name as an internal private package, tricking CI systems into downloading the public version.

### **192. What is Supply Chain Typosquatting Attack?**
**Answer:** Registering package names with slight misspellings of popular packages (e.g., `cross-env` vs `crossenv`) containing embedded cryptominers or backdoors.

### **193. What is Signed Commits in Git (GPG / SSH / S/MIME)?**
**Answer:** Cryptographically signing Git commits using private PGP or SSH keys, verifying that commits originated from an authentic developer identity.

### **194. What is Git Branch Protection Rule?**
**Answer:** Enforcing mandatory code review approvals, passing CI status checks, signed commits, and linear history before allowing merges to protected branches (`main`).

### **195. What is Two-Person Rule in Production Access?**
**Answer:** Requiring two independent authorized individuals to approve any privileged change or access request before execution.

### **196. What is Just-In-Time (JIT) Privileged Access?**
**Answer:** Granting temporary, time-bounded (e.g., 2-hour) elevated production permissions that automatically expire upon task completion.

### **197. What is Honeytoken in DevOps Security?**
**Answer:** Fake API tokens or database accounts intentionally planted in source code or repositories that alert SecOps when accessed, detecting attacker presence.

### **198. What is Zero Trust Device Posture Checking?**
**Answer:** Verifying client device compliance (disk encryption enabled, OS updated, antivirus running) before granting access to corporate resources.

### **199. What is Dynamic Data Masking (DDM)?**
**Answer:** Real-time obfuscation of sensitive database columns (masking credit card numbers to `XXXX-XXXX-XXXX-1234`) based on user querying permissions.

### **200. What is Enterprise DevSecOps Maturity Framework?**
**Answer:**
1. **Level 1 (Reactive):** Ad-hoc vulnerability patching after release.
2. **Level 2 (Shift-Left CI):** Automated SAST, SCA, and container image scanning in pipelines.
3. **Level 3 (Policy as Code):** Kyverno/OPA admission controllers, Cosign image signing, and SLSA Level 3 supply chain security.
4. **Level 4 (Zero Trust & Runtime):** SPIFFE/SPIRE workload attestation, mTLS microsegmentation, and Falco eBPF threat detection.
