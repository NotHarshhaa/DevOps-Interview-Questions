# **Networking & DevSecOps - DevOps Interview Questions**

Welcome to the **Networking & DevSecOps** interview questions master guide. This module provides in-depth, exhaustive technical explanations, Zero Trust architectures, mTLS/SPIFFE implementations, Kubernetes NetworkPolicies, HashiCorp Vault secrets management, OPA/Kyverno policy enforcement, supply chain security (SLSA/SBOM), and runtime threat detection.

---

## 🟢 **Beginner Level (Questions 1–20)**

### **1. Explain the 7-Layer OSI Model and the specific layers DevOps engineers interact with daily.**

**Detailed Answer:**

```
                                  THE 7-LAYER OSI MODEL
 ┌──────────────────────┬──────────────────────────────────────────┬────────────────────────┐
 │ Layer                │ Core Protocols                           │ DevOps Focus Areas     │
 ├──────────────────────┼──────────────────────────────────────────┼────────────────────────┤
 │ 7. Application       │ HTTP, HTTPS, DNS, gRPC, SSH, TLS         │ Ingress, API Gateway   │
 │ 6. Presentation      │ TLS/SSL, JSON, Protobuf, Compression     │ SSL Termination, Serial│
 │ 5. Session           │ Sockets, RPC, NetBIOS                    │ Session persistence    │
 ├──────────────────────┼──────────────────────────────────────────┼────────────────────────┤
 │ 4. Transport         │ TCP, UDP, QUIC                           │ NLB, Port forwarding   │
 ├──────────────────────┼──────────────────────────────────────────┼────────────────────────┤
 │ 3. Network           │ IP, ICMP, BGP, IPsec                     │ VPC, Subnets, Routers  │
 ├──────────────────────┼──────────────────────────────────────────┼────────────────────────┤
 │ 2. Data Link         │ Ethernet, MAC, ARP, VLAN                 │ Switch, CNI bridge     │
 │ 1. Physical          │ Fiber, Cables, Radio, Physical Network   │ Physical data center   │
 └──────────────────────┴──────────────────────────────────────────┴────────────────────────┘
```
*(DevOps and Cloud Engineers primarily architect and debug systems at Layers 3, 4, and 7).*

---

### **2. Compare TCP vs UDP across connection state, reliability, header overhead, and production use cases.**

**Detailed Answer:**
- **TCP (Transmission Control Protocol):**
  - **Connection-Oriented:** Establishes connection via **3-Way Handshake** (`SYN` $\rightarrow$ `SYN-ACK` $\rightarrow$ `ACK`).
  - **Reliable:** Guarantees in-order delivery via sequence numbers, acknowledgment packets, and automatic retransmission of lost packets.
  - **Overhead:** 20-byte header; higher latency due to flow control (sliding window) and congestion management.
  - **Use Cases:** Web applications (HTTP/HTTPS), databases (PostgreSQL/MySQL), SSH, file transfers.
- **UDP (User Datagram Protocol):**
  - **Connectionless:** No handshake; "fire-and-forget" datagram delivery.
  - **Unreliable:** No retransmission, no ordering guarantee, no flow control.
  - **Overhead:** Minimal 8-byte header; ultra-low latency.
  - **Use Cases:** DNS lookups, video streaming, VoIP, gaming, and **HTTP/3 (QUIC)**.

---

### **3. Walk through the complete step-by-step DNS resolution lifecycle when accessing `api.stripe.com`.**

**Detailed Answer:**
1. **Local Caches:** Browser cache $\rightarrow$ OS Resolver cache $\rightarrow$ Local router DNS cache.
2. **Recursive Resolver (e.g., 8.8.8.8 / ISP DNS):** If cache misses, the recursive resolver queries:
3. **Root Name Server (`.`):** Directs resolver to the `.com` Top-Level Domain (TLD) name server.
4. **TLD Name Server (`.com`):** Directs resolver to the Authoritative Name Server for `stripe.com`.
5. **Authoritative Name Server:** Queries zone file and returns the `A` (IPv4) or `AAAA` (IPv6) record (`192.0.2.1`) along with TTL.
6. **Client Connection:** Resolver caches the IP and returns it to the client browser to initiate the TLS handshake.

---

### **4. Compare Public IP vs Private IP (RFC 1918) vs NAT (Network Address Translation).**

**Detailed Answer:**
- **Public IP:** Globally unique and directly routable on the public internet.
- **Private IP (RFC 1918):** Reserved for internal networks (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`); not routable on the public internet.
- **NAT:** Translates multiple private IP addresses to a single public IP address, allowing private instances to initiate outbound internet traffic (e.g., downloading OS patches) while preventing unsolicited incoming internet connections.

---

### **5. Explain CIDR Notation and subnet sizing calculations.**

**Detailed Answer:**
CIDR represents an IP range using base IP and prefix length (`/XX` bits for network portion):
- `/24` $\rightarrow 2^{(32-24)} = 256$ IP addresses.
- `/16` $\rightarrow 2^{(32-16)} = 65,536$ IP addresses.
*(Note: AWS VPC subnets reserve 5 IPs per subnet: Network, VPC Router, DNS, Future use, Broadcast).*

---

### **6. Explain the TLS 1.3 Handshake and compare it to TLS 1.2.**

**Detailed Answer:**
- **TLS 1.2 (2-RTT Handshake):** Required two full network round-trips to negotiate cipher suites and exchange keys before sending application data.
- **TLS 1.3 (1-RTT / 0-RTT Handshake):**
  1. **ClientHello:** Client sends supported ciphers and key share guess in the very first packet.
  2. **ServerHello:** Server selects cipher, returns certificate and server key share. Both compute shared secret immediately.
  3. **0-RTT Resumption:** Returning clients resume encrypted sessions in 0 round-trips.

---

### **7. Compare Symmetric vs Asymmetric Encryption.**

**Detailed Answer:**
- **Asymmetric Encryption (RSA, ECC):** Uses mathematically linked public/private key pairs. Slower; used for digital signatures and TLS key exchange.
- **Symmetric Encryption (AES-256, ChaCha20):** Uses a single shared key for encryption and decryption. Ultra-fast; used for encrypting bulk data in-transit and at-rest.

---

### **8. Compare Reverse Proxy vs Forward Proxy.**

**Detailed Answer:**
- **Forward Proxy:** Sits in front of client devices to inspect, filter, or anonymize outbound traffic to the internet (corporate enterprise proxies).
- **Reverse Proxy:** Sits in front of backend web servers to handle SSL termination, load balancing, compression, caching, and rate limiting (Nginx, Envoy).

---

### **9. What is a Web Application Firewall (WAF)?**

**Detailed Answer:**
A Layer 7 security appliance that inspects HTTP/HTTPS traffic to block OWASP Top 10 web vulnerabilities (SQL Injection, XSS, CSRF, malicious scrapers, rate floods).

---

### **10. What is Zero Trust Architecture (ZTA)?**

**Detailed Answer:**
A cybersecurity model based on **"Never trust, always verify"**. Eliminates the concept of a trusted internal network; every single request is authenticated, authorized, and encrypted (mTLS) regardless of whether it originates inside or outside the VPC.

---

### **11. What is Mutual TLS (mTLS)?**

**Detailed Answer:**
Standard TLS authenticates only the server to the client. In **Mutual TLS (mTLS)**, both client and server present X.509 cryptographic certificates to each other, establishing bidirectional encryption and strict cryptographic identity verification.

---

### **12. What is DevSecOps and the "Shift-Left" philosophy?**

**Detailed Answer:**
Integrating security controls, vulnerability testing, and compliance guardrails into the software delivery pipeline from initial code commit ("Shift-Left") rather than testing security as an afterthought before production.

---

### **13. Compare SAST vs DAST vs IAST vs SCA.**

**Detailed Answer:**
- **SAST (Static Analysis):** Scans source code for vulnerabilities before compilation (Semgrep, SonarQube).
- **DAST (Dynamic Analysis):** Attacks running staging applications from the outside (OWASP ZAP).
- **IAST (Interactive Analysis):** Instruments application runtime from the inside during test execution.
- **SCA (Software Composition Analysis):** Scans open-source third-party dependencies for known CVEs (Trivy, Snyk).

---

### **14. What is HashiCorp Vault and what core capabilities does it provide?**

**Detailed Answer:**
An identity-based secrets management platform providing:
- Secure static and dynamic secret storage.
- Automatic secret rotation and time-to-live (TTL) revocation.
- Encryption as a Service (Transit engine).
- Automated dynamic PKI X.509 certificate generation.

---

### **15. What is a CVE and CVSS score?**

**Detailed Answer:**
- **CVE (Common Vulnerabilities and Exposures):** Standardized identifier for a publicly known vulnerability (e.g., `CVE-2021-44228`).
- **CVSS (Common Vulnerability Scoring System):** Severity score from 0.0 to 10.0 (Low: 0.1–3.9, Medium: 4.0–6.9, High: 7.0–8.9, Critical: 9.0–10.0).

---

### **16. Explain Kubernetes RBAC (Role, ClusterRole, RoleBinding, ClusterRoleBinding).**

**Detailed Answer:**
- **`Role` / `ClusterRole`:** Defines permission rules (verbs: `get`, `list`, `create` on resources: `pods`, `secrets`). Scoped to a namespace (`Role`) or entire cluster (`ClusterRole`).
- **`RoleBinding` / `ClusterRoleBinding`:** Grants defined permissions to subjects (Users, Groups, ServiceAccounts).

---

### **17. Compare Bastion Hosts vs AWS SSM Session Manager.**

**Detailed Answer:**
- **Bastion Host:** Exposed instance in a public subnet requiring open inbound port 22 and SSH key management.
- **AWS SSM Session Manager (Modern Standard):** Agent-driven shell access with **zero public IPs, zero open inbound ports (port 22 closed)**, authenticated via AWS IAM SSO with complete session audit logging.

---

### **18. What is DDoS and how is it mitigated at the edge?**

**Detailed Answer:**
A malicious flood overwhelming server availability using distributed botnets. Mitigated via Cloudflare / AWS Shield, Anycast routing, edge caching, rate limiting, and WAF rules.

---

### **19. What is a Container Escape?**

**Detailed Answer:**
A vulnerability where an attacker running code inside a container exploits a kernel flaw or misconfigured privilege (`--privileged`, root execution, host volume mount) to break out of namespaces and gain full root control over the host OS.

---

### **20. What is Secret Masking and Secret Scanning in Git?**

**Detailed Answer:**
- **Secret Scanning (Gitleaks, Trufflehog):** Scans commit diffs for patterns matching AWS keys, SSH private keys, and API tokens, blocking pushes containing exposed secrets.
- **Secret Masking:** CI/CD feature that replaces detected sensitive variable values with `***` in build output logs.

---

## 🟡 **Intermediate Level (Questions 21–40)**

### **21. Write a complete Kubernetes Default-Deny NetworkPolicy and explain how whitelisting works.**

**Detailed Answer:**
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

---

### **22. What is Cilium NetworkPolicy and how does it provide Layer 7 security?**

**Detailed Answer:**
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

---

### **23. What is SPIFFE/SPIRE and how does Workload Identity Attestation work?**

**Detailed Answer:**
SPIFFE defines standardized cryptographic identities (`spiffe://prod.example.com/ns/finance/sa/payment-service`).
- **Node & Workload Attestation:** SPIRE agent queries the local kernel/kubelet to verify the calling process's UID, container ID, and namespace, issuing short-lived, auto-rotating X.509 SVID certificates into memory without long-lived keys on disk.

---

### **24. How do HashiCorp Vault Dynamic Secrets eliminate static database credentials?**

**Detailed Answer:**
1. App authenticates to Vault using its Kubernetes ServiceAccount token.
2. App requests dynamic DB credentials from Vault Database Engine.
3. Vault dynamically executes `CREATE USER 'v-token-xyz' WITH PASSWORD 'temp_pass' VALID UNTIL '...'` in PostgreSQL and returns credentials.
4. When TTL expires, Vault drops the database user automatically.

---

### **25. What is the Kubernetes External Secrets Operator (ESO)?**

**Detailed Answer:**
Syncs secrets from external secrets managers (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault) into native Kubernetes `Secret` objects, allowing GitOps repositories to commit declarative `ExternalSecret` manifests safely without storing plaintext values in Git.

---

### **26. Compare OPA Gatekeeper vs Kyverno for Kubernetes Admission Control.**

**Detailed Answer:**
- **OPA Gatekeeper:** Uses **Rego** query language. Highly expressive, but steep learning curve.
- **Kyverno:** Kubernetes-native policy engine written **100% in YAML**. Easier to author and test; supports native payload mutation and image verification.

---

### **27. Compare CycloneDX vs SPDX SBOM formats.**

**Detailed Answer:**
- **CycloneDX (OWASP):** Lightweight, designed specifically for application security, vulnerability identification, and CI/CD automation.
- **SPDX (Linux Foundation / ISO standard):** Rich standard frequently used for open-source software licensing and intellectual property compliance.

---

### **28. What is Sigstore Cosign and Keyless Container Image Signing?**

**Detailed Answer:**
Cosign cryptographically signs container images using OIDC tokens from GitHub Actions. Fulcio generates short-lived X.509 certificates, and Rekor records signatures in an immutable transparency log, eliminating static PGP private keys.

---

### **29. What is Falco and how does it detect Runtime Threats in Kubernetes?**

**Detailed Answer:**
A CNCF runtime security engine that parses Linux kernel system calls via eBPF probes, triggering instant alerts on suspicious behavior (spawning `/bin/bash` in production, modifying `/etc/shadow`, unauthorized outbound connections).

---

### **30. Compare BGP Routing vs Anycast DNS.**

**Detailed Answer:**
- **BGP:** Core routing protocol of the internet exchanging reachability information between autonomous systems.
- **Anycast:** Multiple globally distributed servers share the exact same IP address. BGP routes client traffic to the topologically closest server, providing low latency and automatic DDoS absorption.

---

### **31. What is HTTP/3 (QUIC) and why is it superior to HTTP/2?**

**Detailed Answer:**
- **HTTP/2 (TCP):** Suffers from **Head-of-Line Blocking**—if one TCP packet is dropped, the entire connection stalls.
- **HTTP/3 (UDP with QUIC):** Multiplexed streams are independent. Packet loss on Stream A does not stall Stream B; provides 0-RTT connection resumption.

---

### **32. Explain AWS KMS Envelope Encryption.**

**Detailed Answer:**
1. KMS Customer Master Key (CMK) generates a plaintext Data Encryption Key (DEK) and an encrypted DEK.
2. Application encrypts bulk data locally in memory using the plaintext DEK.
3. The plaintext DEK is erased from memory; only the encrypted DEK is stored alongside ciphertext.

---

### **33. What is CIS Benchmark and how do you enforce it in Kubernetes?**

**Detailed Answer:**
The Center for Internet Security (CIS) Benchmark provides prescriptive hardening rules for control plane, worker node, and etcd security, automated via tools like **`kube-bench`**.

---

### **34. What is a Zero-Day Vulnerability and how does Defense in Depth mitigate it?**

**Detailed Answer:**
A flaw actively exploited before a patch exists. Mitigated by runtime behavioral anomaly detection (Falco), strict egress NetworkPolicies (blocking reverse shell C2 traffic), and non-root, read-only container filesystems.

---

### **35. What is DNSSEC?**

**Detailed Answer:**
Adds cryptographic digital signatures to DNS records to protect against DNS spoofing, cache poisoning, and man-in-the-middle attacks.

---

### **36. Compare SQL Injection (SQLi) vs Cross-Site Scripting (XSS).**

**Detailed Answer:**
- **SQLi:** Attacker injects SQL fragments into input fields to manipulate database queries (prevented via prepared statements/parameterized queries).
- **XSS:** Attacker injects malicious JavaScript into web pages viewed by other users to steal session cookies (prevented via Content Security Policy and input sanitization).

---

### **37. What is a PKI and Certificate Authority (CA)?**

**Detailed Answer:**
A Public Key Infrastructure manages digital certificates. A CA is a trusted entity (Let's Encrypt, DigiCert, Vault CA) that validates identities and issues cryptographically signed X.509 certificates.

---

### **38. Compare AWS GuardDuty vs AWS Security Hub.**

**Detailed Answer:**
- **GuardDuty:** Intelligent threat detection analyzing CloudTrail, VPC Flow Logs, and EKS audit logs via machine learning.
- **Security Hub:** Centralized dashboard aggregating compliance posture (CIS, PCI-DSS) and alerts from GuardDuty, Inspector, and Macie.

---

### **39. What is Trivy and what artifact types does it scan?**

**Detailed Answer:**
Scans container images, Git repositories, filesystems, IaC templates (Terraform, Helm), and live Kubernetes clusters for CVE vulnerabilities and misconfigurations.

---

### **40. Compare SSL Termination vs End-to-End Encryption.**

**Detailed Answer:**
- **SSL Termination:** Load Balancer decrypts HTTPS and forwards unencrypted HTTP to private backend pods.
- **End-to-End Encryption:** Traffic remains encrypted all the way down to the individual application container process (mandatory for Zero Trust, HIPAA, PCI-DSS).

---

## 🔴 **Advanced & Scenario-Based Level (Questions 41–50)**

### **41. Scenario: Your Kubernetes cluster is hit with a container escape attack. How does Defense-in-Depth contain the blast radius?**

**Detailed Answer:**
1. **Rootless Execution (`runAsNonRoot: true`):** Attacker lands on host OS as an unprivileged user (UID 10001).
2. **Read-Only Root Filesystem (`readOnlyRootFilesystem: true`):** Attacker cannot download or execute malicious scripts in `/tmp`.
3. **Dropped Capabilities (`capabilities.drop: ["ALL"]`):** Stripping `CAP_SYS_ADMIN` disables kernel exploitation vectors.
4. **Sandboxed Runtime (gVisor):** Intercepts system calls in userspace sandbox.
5. **Egress NetworkPolicy:** Blocks outbound internet access, preventing reverse shell C2 connections.

---

### **42. Scenario: Design an automated End-to-End Supply Chain Security Pipeline conforming to SLSA Level 3.**

**Detailed Answer:**
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

---

### **43. Scenario: Migrate a legacy application with hardcoded database passwords to HashiCorp Vault Dynamic Secrets with zero downtime.**

**Detailed Answer:**
1. Deploy External Secrets Operator (ESO) / Vault Agent Sidecar injecting credentials into an in-memory volume (`/vault/secrets/database.env`).
2. Configure PostgreSQL to accept both legacy static passwords and dynamic Vault-generated user roles concurrently.
3. Update application to read dynamic credentials from `/vault/secrets/database.env`.
4. Deploy pods rolling fashion.
5. Delete legacy static database user after all old pods terminate.

---

### **44. How do you mitigate TCP SYN Flood DDoS attacks at the Linux Kernel and Load Balancer layers?**

**Detailed Answer:**
1. **Enable SYN Cookies:** `sysctl -w net.ipv4.tcp_syncookies=1` (server encodes state into `SYN-ACK` sequence number instead of allocating half-open connection memory).
2. **Tune Kernel Backlog:** `sysctl -w net.ipv4.tcp_max_syn_backlog=4096` and `sysctl -w net.core.somaxconn=4096`.
3. **Cloudflare / CloudFront Anycast:** Absorbs flood traffic at the global edge network before reaching origin servers.

---

### **45. Compare OpenID Connect (OIDC) vs OAuth 2.0 vs SAML 2.0.**

**Detailed Answer:**
- **OAuth 2.0 (Authorization):** Issues Access Tokens allowing third-party apps to access API resources on behalf of a user.
- **OIDC (Authentication):** Built on OAuth 2.0; adds an **ID Token (JWT)** to verify user identity ("Sign in with Google").
- **SAML 2.0 (Enterprise Federation):** XML-based standard heavily used in legacy enterprise SSO (Okta, Active Directory).

---

### **46. How do you implement Zero-Trust Microsegmentation in Kubernetes using Cilium and WireGuard Transparent Encryption?**

**Detailed Answer:**
- **Cilium eBPF:** Enforces Layer 3 to Layer 7 NetworkPolicies without modifying application pods.
- **WireGuard Encryption:** Set `encryption.type: wireguard` in Cilium Helm values to automatically encrypt all node-to-node and pod-to-pod network traffic in kernel-space with zero proxy sidecar overhead.

---

### **47. Scenario: An audit discovers that engineers have direct SSH access to production EC2 instances. Implement a Zero-Trust Access Gateway with Teleport or AWS SSM.**

**Detailed Answer:**
1. Delete security group rules allowing inbound port 22; disable public IPs.
2. Deploy AWS SSM Managed Instance Core or Teleport Auth/Proxy service.
3. Integrate with corporate Okta / Entra ID SSO with mandatory MFA.
4. Engineers authenticate via SSO to receive short-lived SSH certificates (valid for 8 hours); all interactive sessions are recorded for compliance auditing.

---

### **48. What is Secrets Sprawl and how do you design an enterprise remediation pipeline?**

**Detailed Answer:**
1. **Pre-Commit:** Developers install `pre-commit` running `gitleaks protect` locally.
2. **Push Protection:** GitHub blocks commits containing detected high-entropy keys.
3. **Scheduled Scanning:** Nightly org-wide Trufflehog scans across all repositories.
4. **Automated Key Revocation:** Trigger automated Lambda functions via webhook to deactivate exposed keys within 60 seconds of commit detection.

---

### **49. What is DNS Tunneling / DNS Exfiltration and how is it blocked?**

**Detailed Answer:**
- **Attack:** Malware encodes stolen sensitive data into DNS subdomains (`stolen_data.evil-domain.com`) and queries an attacker-controlled authoritative name server, bypassing standard firewall HTTP filters.
- **Defense:** Enforce AWS Route 53 Resolver DNS Firewall / Pi-hole; detect high volumes of high-entropy subdomain queries and non-standard DNS query types (`TXT`, `NULL`).

---

### **50. Scenario: Securely connect two Kubernetes clusters in different AWS VPCs without public internet exposure.**

**Detailed Answer:**
1. Connect the two VPCs via **AWS Transit Gateway** or **VPC Peering** with non-overlapping CIDR blocks.
2. Use **Cilium Cluster Mesh** or **Submariner** for cross-cluster pod IP routing and service discovery.
3. Enforce mutual TLS (mTLS) across clusters using a shared root SPIFFE/SPIRE trust bundle or Istio Multi-Cluster Mesh.
