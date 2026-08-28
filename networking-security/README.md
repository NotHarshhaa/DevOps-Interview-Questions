# **Networking & DevSecOps - DevOps Interview Questions**

Welcome to the **Networking & DevSecOps** interview questions module. This section covers Zero Trust security, mTLS, SPIFFE/SPIRE, Kubernetes NetworkPolicies, Cilium eBPF network security, HashiCorp Vault secrets management, OPA Gatekeeper/Kyverno policy enforcement, supply chain security (SLSA, SBOM, Cosign), and runtime threat detection.

---

## 🟢 **Beginner Level (Questions 1–20)**

### **1. What is the OSI Model and what are the primary layers DevOps engineers interact with?**
**Answer:**
The 7-Layer OSI (Open Systems Interconnection) Model:
- **Layer 7 (Application):** HTTP, HTTPS, DNS, gRPC (API Gateways, ALBs, Ingress).
- **Layer 4 (Transport):** TCP, UDP (Network Load Balancers, Port forwarding).
- **Layer 3 (Network):** IP, ICMP, BGP routing (VPCs, Subnets, Routers).
- **Layer 2 (Data Link):** MAC addressing, Ethernet, ARP.
- **Layer 1 (Physical):** Cables, fiber, network interfaces.
*(DevOps primarily architects and debugs at Layers 3, 4, and 7).*

---

### **2. What is the difference between TCP and UDP?**
**Answer:**
- **TCP (Transmission Control Protocol):** Connection-oriented (3-way handshake `SYN` $\rightarrow$ `SYN-ACK` $\rightarrow$ `ACK`), reliable, guarantees in-order packet delivery with retransmission and flow control. Used for HTTP/HTTPS, databases, SSH.
- **UDP (User Datagram Protocol):** Connectionless, lightweight, "fire-and-forget", no handshake or retransmission guarantee, minimal latency. Used for DNS queries, video streaming, VoIP, gaming, and HTTP/3 (QUIC).

---

### **3. What is DNS and how does a DNS resolution query work?**
**Answer:**
DNS (Domain Name System) translates human-readable hostnames (e.g., `api.example.com`) into IP addresses.
**Resolution Flow:**
1. Browser cache $\rightarrow$ OS Resolver cache $\rightarrow$ Local DNS Server (e.g., router/ISP).
2. **Root Name Server (`.`):** Directs to Top-Level Domain (TLD) server (`.com`).
3. **TLD Name Server:** Directs to the Authoritative Name Server for `example.com`.
4. **Authoritative Name Server:** Returns the final `A`/`AAAA` record (`192.0.2.1`).

---

### **4. What is the difference between Public IP, Private IP, and NAT (Network Address Translation)?**
**Answer:**
- **Public IP:** Globally routable on the public internet.
- **Private IP (RFC 1918):** Reserved for internal networks (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`); not routable on the public internet.
- **NAT:** Translates private IP addresses to a single public IP address, allowing private instances to access external internet resources without exposing their private addresses.

---

### **5. What is a Subnet and CIDR Notation (Classless Inter-Domain Routing)?**
**Answer:**
CIDR represents an IP range using base IP and prefix length (`/XX` bits for network portion):
- `/24` $\rightarrow 2^{(32-24)} = 256$ IP addresses.
- `/16` $\rightarrow 2^{(32-16)} = 65,536$ IP addresses.
*(Note: In AWS VPC subnets, 5 IPs are reserved per subnet: Network, Router, DNS, Future use, Broadcast).*

---

### **6. What is SSL/TLS and how does a TLS 1.3 Handshake work?**
**Answer:**
TLS encrypts and authenticates communication between client and server.
**TLS 1.3 Handshake (1-RTT):**
1. **ClientHello:** Client sends supported cipher suites and key share.
2. **ServerHello:** Server selects cipher suite, sends digital certificate and key share.
3. Both sides calculate shared symmetric session keys (Diffie-Hellman) and begin sending encrypted HTTP traffic immediately.

---

### **7. What is the difference between Symmetric and Asymmetric Encryption?**
**Answer:**
- **Asymmetric Encryption (RSA, ECC):** Uses a public key to encrypt and a private key to decrypt. Slower; used for digital signatures and initial TLS key exchange.
- **Symmetric Encryption (AES-256, ChaCha20):** Uses the exact same secret key to encrypt and decrypt. Ultra-fast; used for encrypting bulk data in-transit and at-rest.

---

### **8. What is a Reverse Proxy vs Forward Proxy?**
**Answer:**
- **Forward Proxy:** Sits in front of client devices to inspect, filter, or anonymize outbound requests to the internet (e.g., corporate enterprise web proxies).
- **Reverse Proxy:** Sits in front of backend web servers to handle SSL termination, load balancing, compression, caching, and rate limiting (e.g., Nginx, HAProxy, Envoy).

---

### **9. What is a Web Application Firewall (WAF)?**
**Answer:**
A Layer 7 firewall that monitors and blocks malicious HTTP/HTTPS traffic targeting web applications. Protects against OWASP Top 10 vulnerabilities (SQL Injection, XSS, CSRF, malicious bots, and rate floods).

---

### **10. What is Zero Trust Architecture (ZTA)?**
**Answer:**
A modern cybersecurity paradigm based on the principle **"Never trust, always verify"**.
- Eliminates the traditional concept of a "trusted internal network".
- Authenticates and authorizes every single user, device, and service request explicitly using identity, context, and encryption (mTLS).

---

### **11. What is mTLS (Mutual TLS)?**
**Answer:**
Standard TLS only requires the server to present a certificate to the client.
**Mutual TLS (mTLS):** Both client and server authenticate each other by presenting cryptographic X.509 certificates, establishing bidirectional encryption and strict identity verification.

---

### **12. What is DevSecOps and the "Shift-Left" philosophy?**
**Answer:**
DevSecOps integrates security practices, automated vulnerability testing, and compliance guardrails directly into the software delivery pipeline from the initial code commit stage ("Shift-Left") rather than testing security as an afterthought before production.

---

### **13. What is SAST vs DAST vs IAST vs SCA?**
**Answer:**
- **SAST (Static Analysis):** Scans uncompiled source code for vulnerabilities (SonarQube, Semgrep).
- **DAST (Dynamic Analysis):** Attacks running applications from the outside (OWASP ZAP).
- **IAST (Interactive Analysis):** Instruments application runtime from the inside to detect vulnerabilities during test execution.
- **SCA (Software Composition Analysis):** Scans open-source third-party dependencies for known CVEs (Snyk, Trivy).

---

### **14. What is HashiCorp Vault and what core capabilities does it provide?**
**Answer:**
Vault is an identity-based secrets management platform providing:
- Secure static and dynamic secret storage.
- Fine-grained access control with token/IAM authentication.
- Automatic secret rotation and time-to-live (TTL) revocation.
- Encryption as a Service (Transit Secrets Engine).
- Automated PKI X.509 certificate generation.

---

### **15. What is a Vulnerability (CVE) and CVSS score?**
**Answer:**
- **CVE (Common Vulnerabilities and Exposures):** A standardized identifier for a publicly known cybersecurity vulnerability (e.g., `CVE-2021-44228` for Log4Shell).
- **CVSS (Common Vulnerability Scoring System):** A score from 0.0 to 10.0 reflecting vulnerability severity (Low: 0.1–3.9, Medium: 4.0–6.9, High: 7.0–8.9, Critical: 9.0–10.0).

---

### **16. What is Kubernetes RBAC (Role-Based Access Control)?**
**Answer:**
Enforces authorization rules in the Kubernetes API server using:
- **`Role` / `ClusterRole`:** Defines permissions (verbs: `get`, `list`, `watch`, `create` on resources: `pods`, `services`).
- **`RoleBinding` / `ClusterRoleBinding`:** Grants defined permissions to subjects (Users, Groups, ServiceAccounts).

---

### **17. What is a Bastion Host / Jump Box vs AWS SSM Session Manager?**
**Answer:**
- **Bastion Host:** An exposed EC2 instance in a public subnet used by engineers to SSH into private subnet resources. High maintenance (requires managing SSH keys, opening port 22).
- **AWS SSM Session Manager (Modern Standard):** Secure, agent-driven browser/CLI shell access. No public IPs, no open inbound ports (port 22 closed), authenticated via AWS IAM SSO, with complete audit logging.

---

### **18. What is DDoS (Distributed Denial of Service) and how is it mitigated?**
**Answer:**
A malicious attempt to disrupt server availability by overwhelming it with flood traffic from thousands of distributed botnet IPs.
- **Mitigation:** Cloudflare / AWS Shield, Anycast DNS, edge caching, rate limiting, SYN proxying, and WAF rules.

---

### **19. What is a Container Escape / Container Breakout?**
**Answer:**
A security vulnerability where an attacker running code inside a container exploits a Linux kernel vulnerability or misconfigured privilege (`--privileged`, root execution, host volume mount) to break out of namespaces and gain full root control over the host operating system.

---

### **20. What is Secret Masking and Secret Scanning in Git?**
**Answer:**
- **Secret Scanning (Gitleaks, Trufflehog):** Scans commit diffs for patterns matching AWS keys, SSH private keys, and API tokens, blocking pushes containing exposed secrets.
- **Secret Masking:** CI/CD feature that replaces detected sensitive variable values with `***` in build output logs.

---

## 🟡 **Intermediate Level (Questions 21–40)**

### **21. What is a Kubernetes NetworkPolicy and how do you write a Default-Deny rule?**
**Answer:**
By default, Kubernetes pods are non-isolated and accept traffic from any source. A `NetworkPolicy` configures Layer 3/4 firewall rules between pods.

**Default-Deny Ingress and Egress Policy:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}  # Selects all pods in the namespace
  policyTypes:
    - Ingress
    - Egress
```
Once applied, all incoming and outgoing pod traffic is blocked unless explicitly whitelisted by subsequent NetworkPolicy rules.

---

### **22. What is Cilium NetworkPolicy and how does it provide Layer 7 security?**
**Answer:**
Standard Kubernetes NetworkPolicies only filter on IP and Port (Layer 3/4).

**Cilium L7 NetworkPolicy (via eBPF):**
Enforces rules at the application layer (HTTP methods, URL paths, gRPC methods, Kafka topics, DNS domain names):
```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l7-payment-policy
spec:
  endpointSelector:
    matchLabels:
      app: payment
  ingress:
    - fromEndpoints:
        - matchLabels:
            app: frontend
      toPorts:
        - ports:
            - port: "8080"
              protocol: TCP
          rules:
            http:
              - method: "POST"
                path: "/v1/charge"
```

---

### **23. What is SPIFFE/SPIRE and how does it implement Workload Identity Attestation?**
**Answer:**
SPIFFE (Secure Production Identity Framework for Everyone) issues standardized cryptographic identities (`spiffe://example.com/ns/prod/sa/payment-service`) to workloads.
- **Node Attestation:** SPIRE server verifies that the node agent is authentic.
- **Workload Attestation:** SPIRE agent queries the local kernel/kubelet to verify the calling process's UID, container ID, and namespace.
- **Issuance:** Delivers short-lived, auto-rotating X.509 SVID certificates into the pod without storing long-lived keys on disk.

---

### **24. What is HashiCorp Vault Dynamic Secrets and how do they eliminate static database credentials?**
**Answer:**
Instead of applications sharing a static database password:
1. Application authenticates to Vault using its Kubernetes ServiceAccount token.
2. App requests dynamic database credentials from Vault Database Engine.
3. Vault connects to PostgreSQL/MySQL, dynamically executes `CREATE USER 'v-token-xyz' WITH PASSWORD 'temp_pass' VALID UNTIL '2026-08-28 14:00:00'`, and returns credentials to the app.
4. When the TTL expires, Vault automatically drops the database user.

---

### **25. What is the Kubernetes External Secrets Operator (ESO)?**
**Answer:**
ESO is a Kubernetes operator that syncs secrets from external enterprise secrets managers (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault, GCP Secret Manager) into native Kubernetes `Secret` objects.
- Allows GitOps repositories to commit declarative `ExternalSecret` manifests safely without storing plaintext values in Git.

---

### **26. What is OPA (Open Policy Agent) Gatekeeper vs Kyverno for Kubernetes Admission Control?**
**Answer:**
Both intercept Kubernetes API requests via admission webhooks to enforce organizational security policies:
- **OPA Gatekeeper:** Uses **Rego** (a declarative query language). Highly expressive, but has a steep learning curve.
- **Kyverno:** Kubernetes-native policy engine written **100% in YAML**. Much easier to author, test, and supports native payload mutation and image verification.

---

### **27. What is an SBOM (Software Bill of Materials) and what formats (CycloneDX vs SPDX) are used?**
**Answer:**
An SBOM is a formal, machine-readable nested inventory of all software libraries, modules, transitive dependencies, and licenses bundled in a software artifact.
- **CycloneDX (OWASP):** Lightweight, designed specifically for application security, vulnerability identification, and CI/CD automation.
- **SPDX (Linux Foundation / ISO standard):** Rich standard frequently used for open-source software licensing and intellectual property compliance.

---

### **28. What is Sigstore Cosign and Keyless Container Image Signing?**
**Answer:**
Cosign cryptographically signs container images to guarantee authenticity and prevent tampering.
**Keyless Signing Flow:**
1. CI pipeline requests an OIDC identity token from GitHub Actions / GitLab.
2. Cosign sends the token to **Fulcio** (a lightweight PKI certificate authority), which generates a short-lived X.509 certificate bound to the GitHub repo/workflow identity.
3. Signature metadata is stored in **Rekor** (an immutable, append-only transparency log).
4. Eliminates the burden of managing and rotating private PGP signing keys.

---

### **29. What is Falco and how does it detect Runtime Threats in Kubernetes?**
**Answer:**
Falco is a CNCF runtime security engine that parses Linux kernel system calls in real time via eBPF probes.
- Compares syscalls against security rules and triggers instant alerts on suspicious behavior:
  - Spawning a shell (`/bin/bash`) inside a running production container.
  - Modifying sensitive system files (`/etc/shadow`, `/etc/pam.d`).
  - Outbound network connections to unauthorized IPs from an isolated database pod.

---

### **30. What is BGP (Border Gateway Protocol) and Anycast DNS?**
**Answer:**
- **BGP:** The core routing protocol of the internet that exchanges routing and reachability information between autonomous systems (AS).
- **Anycast Routing:** Multiple globally distributed servers share the exact same IP address. BGP routes incoming client traffic to the topologically closest server, providing high availability, low latency, and automatic DDoS absorption.

---

### **31. What is HTTP/3 (QUIC) and why is it superior to HTTP/2?**
**Answer:**
- **HTTP/2 (Over TCP):** Suffers from **Head-of-Line (HoL) Blocking**—if a single TCP packet is dropped, the entire connection freezes until retransmission completes.
- **HTTP/3 (Over UDP with QUIC):** Implements multiplexed streams independently. Packet loss on Stream A does not stall Stream B. Provides 0-RTT connection resumption and seamless IP migration when mobile clients switch networks.

---

### **32. What is AWS KMS Envelope Encryption?**
**Answer:**
1. KMS Customer Master Key (CMK) generates a plaintext Data Encryption Key (DEK) and an encrypted DEK.
2. The application encrypts high-volume data locally using the plaintext DEK.
3. The plaintext DEK is erased from memory; only the encrypted DEK is stored alongside the ciphertext.
4. Decryption requires sending only the encrypted DEK back to KMS to unwrap.

---

### **33. What is CIS Benchmark and how do you enforce it in Kubernetes?**
**Answer:**
The **Center for Internet Security (CIS) Kubernetes Benchmark** provides prescriptive security configuration guidelines for control plane, worker node, and etcd hardening.
- Automated evaluation using open-source tools like **kube-bench**.

---

### **34. What is a Zero-Day Vulnerability and how do you protect against it?**
**Answer:**
A zero-day is a security vulnerability that is actively exploited before the software vendor is aware of it or has released a security patch.
- **Defense in Depth:** Defense cannot rely on patch signatures alone. Must enforce:
  - Runtime behavior anomaly detection (Falco, Tetragon).
  - Strict egress NetworkPolicies (blocking reverse shell C2 connections).
  - Non-root, distroless read-only container filesystems.

---

### **35. What is DNSSEC (Domain Name System Security Extensions)?**
**Answer:**
DNSSEC adds cryptographic digital signatures (using public key cryptography) to existing DNS records to protect against DNS spoofing, cache poisoning, and man-in-the-middle attacks, ensuring responses originate from the genuine authoritative name server.

---

### **36. What is Cross-Site Scripting (XSS) vs SQL Injection (SQLi)?**
**Answer:**
- **SQLi:** Attacker injects malicious SQL fragments into input fields to manipulate database queries (prevented via parameterized queries / prepared statements).
- **XSS:** Attacker injects malicious JavaScript into web pages viewed by other users to steal session cookies or credentials (prevented via Content Security Policy - CSP and input sanitization).

---

### **37. What is a Public Key Infrastructure (PKI) and Certificate Authority (CA)?**
**Answer:**
A PKI manages digital certificates and public-key encryption. A **Certificate Authority (CA)** is a trusted entity (e.g., Let's Encrypt, DigiCert, internal Vault CA) that validates identities and issues cryptographically signed X.509 certificates.

---

### **38. What is AWS GuardDuty vs AWS Security Hub?**
**Answer:**
- **Amazon GuardDuty:** Intelligent threat detection service that continuously analyzes CloudTrail logs, VPC Flow Logs, DNS logs, and EKS audit logs using machine learning to detect compromised instances, unauthorized crypto-mining, or credential exfiltration.
- **AWS Security Hub:** Centralized dashboard aggregating security findings, compliance posture (CIS, PCI-DSS), and alerts from GuardDuty, Inspector, Macie, and IAM Access Analyzer.

---

### **39. What is Trivy and what artifact types does it scan?**
**Answer:**
Trivy is a comprehensive, open-source vulnerability scanner that scans:
- Container Images (OS packages and language dependencies).
- Git repositories (source code vulnerabilities and secrets).
- Filesystems and rootfs.
- IaC configurations (Terraform, CloudFormation, Kubernetes YAML, Dockerfile).
- Kubernetes clusters (live workload compliance).

---

### **40. What is TLS Offloading / SSL Termination vs End-to-End Encryption?**
**Answer:**
- **SSL Termination:** Load Balancer / Ingress decrypts incoming HTTPS traffic and forwards unencrypted HTTP to backend pods inside the private VPC (reduces pod CPU overhead).
- **End-to-End Encryption:** Traffic remains encrypted all the way from the client browser through the load balancer down to the individual application container process (mandatory for zero-trust, HIPAA, and PCI-DSS compliance).

---

## 🔴 **Advanced & Scenario-Based Level (Questions 41–60)**

### **41. Scenario: Your Kubernetes cluster is hit with a container escape attack exploiting a novel Linux kernel privilege escalation bug. How does a Defense-in-Depth architecture contain the blast radius?**
**Answer:**
1. **Rootless Execution (`runAsNonRoot: true`):** The attacker escapes the container but lands on the host OS as an unprivileged user (UID 10001), preventing root control.
2. **Read-Only Root Filesystem (`readOnlyRootFilesystem: true`):** Attacker cannot download or execute malicious scripts/binaries in `/tmp` or `/bin`.
3. **Dropped Capabilities (`capabilities.drop: ["ALL"]`):** Stripping `CAP_SYS_ADMIN` and `CAP_NET_RAW` disables kernel exploitation vectors.
4. **Sandboxed Runtime (gVisor / Kata Containers):** Intercepts system calls in a userspace sandbox, preventing the attacker from interacting directly with the host Linux kernel.
5. **Egress NetworkPolicy:** Blocks outbound internet access, preventing data exfiltration or reverse shell connections back to the attacker's Command & Control (C2) server.

---

### **42. Scenario: How do you design and implement an automated End-to-End Supply Chain Security Pipeline conforming to SLSA Level 3 standards?**
**Answer:**
```
[ Developer Commit ] ➔ [ GitHub PR (2-Person Review) ]
                             │
                             ▼
[ Ephemeral GitHub Actions Build Runner (Isolated) ]
  ├── 1. Generate CycloneDX SBOM (Syft)
  ├── 2. Build Container Image (Hermetic Dockerfile)
  ├── 3. Sign Image & SBOM with Cosign (Fulcio OIDC Keyless)
  └── 4. Push In-Toto Provenance Attestation to Rekor Transparency Log
                             │
                             ▼
[ OCI Registry (Amazon ECR / GHCR) ]
                             │
                             ▼
[ Kubernetes Cluster Admission Controller (Kyverno) ]
  └── Policy: Verify Cosign Signature & Provenance before admitting Pod
```

---

### **43. Scenario: A legacy microservice application is running in production with hardcoded database passwords in its configuration files. How do you migrate it to HashiCorp Vault Dynamic Secrets with Zero Downtime?**
**Answer:**
1. **Deploy External Secrets Operator (ESO) / Vault Agent Sidecar:** Injects secrets into an in-memory volume mount (`/vault/secrets/database.env`).
2. **Dual-Authentication Transition:**
   - Configure PostgreSQL to accept both the legacy static password and dynamic Vault-generated user roles.
3. **Application Update:**
   - Update application to read database credentials dynamically from `/vault/secrets/database.env` (or live-reload credentials on file modification).
4. **Deploy Pods Rolling Fashion:** Deploy new pods connected to Vault; verify connection stability.
5. **Decommission Static User:** Delete the legacy hardcoded database user from PostgreSQL after all old pods terminate.

---

### **44. How do you mitigate TCP SYN Flood DDoS attacks at the Linux Kernel and Load Balancer layers?**
**Answer:**
- **Mechanism:** Attacker sends thousands of `SYN` packets from spoofed IPs without sending the final `ACK`, exhausting the server's SYN backlog queue (`tcp_max_syn_backlog`).
- **Mitigation:**
  1. **Enable SYN Cookies (`sysctl -w net.ipv4.tcp_syncookies=1`):** Server does not allocate memory for half-open connections; encodes connection state into the `SYN-ACK` sequence number.
  2. **Tune Kernel Backlog:**
     ```bash
     sysctl -w net.ipv4.tcp_max_syn_backlog=4096
     sysctl -w net.core.somaxconn=4096
     ```
  3. **Cloudflare / AWS CloudFront Anycast:** Absorbs flood traffic at the global edge network before it reaches the origin server.

---

### **45. What is the difference between OpenID Connect (OIDC), OAuth 2.0, and SAML 2.0?**
**Answer:**
- **OAuth 2.0 (Authorization):** Issues Access Tokens allowing third-party apps to access API resources on behalf of a user (e.g., "Allow App X to read your Google Drive").
- **OIDC (Authentication):** Built on top of OAuth 2.0; adds an **ID Token (JWT)** to verify the user's identity (e.g., "Sign in with Google").
- **SAML 2.0 (Enterprise Federation):** XML-based standard heavily used in legacy enterprise SSO (Okta, Ping, Active Directory) for web browser single sign-on.

---

### **46. How do you implement Zero-Trust Microsegmentation in Kubernetes using Cilium and WireGuard Transparent Encryption?**
**Answer:**
- **Cilium eBPF:** Enforces Layer 3 to Layer 7 NetworkPolicies without modifying application pods.
- **WireGuard Transparent In-Transit Encryption:**
  - Configure `encryption.type: wireguard` in Cilium Helm values.
  - Automatically encrypts all node-to-node and pod-to-pod network traffic using kernel-space WireGuard encryption with zero application configuration or sidecar overhead.

---

### **47. Scenario: An audit discovers that engineers have direct SSH access to production EC2 instances. How do you implement an enterprise Zero-Trust Access Gateway with Teleport or AWS SSM?**
**Answer:**
1. **Block Port 22:** Delete all security group rules allowing inbound port 22; disable public IP addresses on instances.
2. **Deploy AWS SSM / Teleport:**
   - Deploy Teleport Auth/Proxy service or attach AWS SSM Managed Instance Core IAM roles.
3. **Identity Integration:** Integrate with corporate Okta / Azure AD SSO with mandatory MFA.
4. **Ephemeral Certificates & Audit:**
   - Engineers authenticate via SSO to receive short-lived SSH certificates (valid for 8 hours).
   - All interactive SSH/CLI sessions are fully recorded and video-playable for compliance auditing.

---

### **48. What is Secrets Sprawl and how do you design an enterprise remediation pipeline?**
**Answer:**
1. **Pre-Commit Enforcement:** Developers install `pre-commit` running `gitleaks protect` locally.
2. **GitHub Push Protection:** Blocks commits containing detected high-entropy keys from being pushed to remote repositories.
3. **Scheduled Scanning:** Run nightly org-wide Trufflehog scans across all GitHub repositories, wikis, and commit histories.
4. **Automated Key Revocation:** Trigger automated Lambda functions via webhook to deactivate exposed AWS/Stripe API keys within 60 seconds of commit detection.

---

### **49. What is DNS Tunneling / DNS Exfiltration and how do you detect and block it?**
**Answer:**
- **Attack Mechanism:** Malware encodes stolen sensitive data into DNS subdomains (e.g., `stolen_credit_card_data.evil-domain.com`) and queries an attacker-controlled authoritative name server, bypassing standard firewall HTTP filters.
- **Detection & Blocking:**
  - Enforce AWS Route 53 Resolver DNS Firewall / Pi-hole / NextDNS.
  - Detect high volumes of high-entropy, long subdomain queries and non-standard DNS query types (`TXT`, `NULL`).

---

### **50. Scenario: An engineer needs to securely connect two Kubernetes clusters running in different AWS VPCs without exposing endpoints to the public internet. How do you architect this?**
**Answer:**
1. **Networking Layer:** Connect the two VPCs via **AWS Transit Gateway** or **AWS VPC Peering** with non-overlapping CIDR blocks.
2. **Cluster Networking:** Use **Cilium Cluster Mesh** or **Submariner** to establish cross-cluster pod IP routing and service discovery.
3. **Security:** Enforce mutual TLS (mTLS) across clusters using a shared root SPIFFE/SPIRE trust bundle or Istio Multi-Cluster Mesh.
