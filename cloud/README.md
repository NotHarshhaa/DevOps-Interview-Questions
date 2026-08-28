# **Cloud Computing & Architecture - DevOps Interview Questions**

Welcome to the **Cloud Computing & Architecture** interview questions module. This section covers multi-cloud design patterns across AWS, Microsoft Azure, and Google Cloud Platform (GCP), IAM workload federation, cloud networking, FinOps cost optimization, serverless, and multi-region high availability architectures.

---

## 🟢 **Beginner Level (Questions 1–20)**

### **1. What is Cloud Computing and what are the three primary service models (IaaS, PaaS, SaaS)?**
**Answer:**
Cloud computing is the on-demand delivery of IT resources over the internet with pay-as-you-go pricing.

- **Infrastructure as a Service (IaaS):** Provides raw compute, networking, and storage. Full OS control (e.g., AWS EC2, Azure VMs, GCP Compute Engine).
- **Platform as a Service (PaaS):** Provides managed runtimes where developers deploy code without managing VMs or OS patches (e.g., AWS Elastic Beanstalk, Azure App Services, Google App Engine).
- **Software as a Service (SaaS):** Complete end-user software delivered over the web (e.g., GitHub, Slack, Microsoft 365).

---

### **2. What is the Cloud Shared Responsibility Model?**
**Answer:**
Security and compliance are shared between the Cloud Service Provider (CSP) and the Customer:
- **Security OF the Cloud (Provider Responsibility):** Physical data center security, host hardware, virtualization hypervisor layer, network infrastructure, and facilities.
- **Security IN the Cloud (Customer Responsibility):** Customer data, IAM permissions, OS configuration and patches (in IaaS), firewall and security group rules, network configuration, and application code.

---

### **3. What is a Cloud Region vs an Availability Zone (AZ) vs a Local Zone / Edge Location?**
**Answer:**
- **Region:** A geographical area (e.g., `us-east-1` in N. Virginia) containing multiple isolated data centers.
- **Availability Zone (AZ):** One or more discrete data centers within a region, each with redundant power, networking, and cooling, interconnected via ultra-low-latency private fiber.
- **Edge Location / CloudFront PoP:** Content delivery network (CDN) points of presence located worldwide to cache static content and terminate TLS close to end users.

---

### **4. What is a Virtual Private Cloud (VPC) and what are its core components?**
**Answer:**
A VPC is a logically isolated virtual network dedicated to your cloud account.
- **Subnets:** Subdivisions of a VPC IP range (Public Subnets have direct route to Internet Gateway; Private Subnets route to NAT Gateway).
- **Internet Gateway (IGW):** Enables bidirectional communication between public subnets and the internet.
- **NAT Gateway:** Allows private subnet resources to initiate outbound internet traffic (e.g., software updates) while blocking unsolicited inbound connections.
- **Route Tables:** Rules determining where network packets are directed based on destination CIDRs.

---

### **5. What is the difference between Security Groups and Network ACLs (NACLs)?**
**Answer:**
| Feature | Security Group (SG) | Network ACL (NACL) |
| :--- | :--- | :--- |
| **Level** | Virtual firewall at the **Instance / ENI** level | Firewall at the **Subnet** level |
| **State** | **Stateful** (Return traffic is automatically allowed) | **Stateless** (Inbound & outbound rules evaluated separately) |
| **Rule Type** | **Allow rules only** (Everything else denied) | **Allow and Deny rules** (Evaluated by rule number) |
| **Evaluation** | All rules evaluated simultaneously | Evaluated in numerical order (lowest first) |

---

### **6. What is the difference between Object Storage (S3), Block Storage (EBS), and File Storage (EFS)?**
**Answer:**
- **Object Storage (Amazon S3 / GCS / Azure Blob):** Flat storage structure accessed via REST APIs (HTTP GET/PUT). Unlimited scale, metadata-rich, ideal for static assets, backups, and data lakes.
- **Block Storage (Amazon EBS / Azure Managed Disk / GCP Persistent Disk):** High-performance raw block volumes attached to a single VM (or multi-attach with IOPS provisioning). Ideal for databases and OS root drives.
- **File Storage (Amazon EFS / Azure Files / GCP Filestore):** Managed POSIX-compliant network file system (NFS) shareable across hundreds of instances simultaneously (`ReadWriteMany`).

---

### **7. What is Horizontal Scaling (Scale-Out) vs Vertical Scaling (Scale-Up)?**
**Answer:**
- **Vertical Scaling (Scale-Up):** Adding more CPU, RAM, or disk to an existing single instance (e.g., upgrading `t3.medium` to `r6i.4xlarge`). Has hardware limits and requires downtime.
- **Horizontal Scaling (Scale-Out):** Adding more instances/nodes behind a load balancer. Provides elasticity, fault tolerance, and theoretically limitless scale.

---

### **8. What is Cloud Auto Scaling and how does it work?**
**Answer:**
Auto Scaling dynamically adjusts compute capacity based on traffic demands:
- **Metrics-based (Dynamic):** Scales out when average CPU utilization $> 70\%$ or request count spikes.
- **Target Tracking:** Automatically scales capacity up/down to maintain a specific target metric (e.g., ALB Request Count per Target = 1000).
- **Scheduled:** Pre-provisions instances before anticipated traffic spikes (e.g., Black Friday).

---

### **9. What is an Elastic Load Balancer (ELB) and what are its main types in AWS?**
**Answer:**
- **ALB (Application Load Balancer - Layer 7):** Routes HTTP/HTTPS traffic based on request attributes (URL path, host headers, HTTP methods).
- **NLB (Network Load Balancer - Layer 4):** Ultra-high performance, millions of requests per second, handles raw TCP/UDP, preserves source IP, provides static IP addresses per AZ.
- **GLB (Gateway Load Balancer - Layer 3):** Routes traffic through third-party virtual security appliances (firewalls, IDS/IPS).

---

### **10. What is Serverless Computing and what are its trade-offs?**
**Answer:**
Serverless executes code on-demand in response to events without provisioning or managing servers (e.g., AWS Lambda, Azure Functions, Cloud Run).
- **Pros:** Zero server administration, pay-per-millisecond execution, scales to zero when idle.
- **Cons:** Cold start latencies, execution time limits (e.g., 15 minutes in Lambda), debugging complexity, vendor lock-in.

---

### **11. What is IAM (Identity and Access Management) and the Principle of Least Privilege?**
**Answer:**
IAM controls **Who** (Authentication) can access **What** resources (Authorization).
- **Least Privilege:** A security principle stipulating that users, services, and workloads must be granted only the minimum necessary permissions required to perform their specific job functions, and nothing more.

---

### **12. What is the difference between IAM Users, IAM Groups, and IAM Roles?**
**Answer:**
- **IAM User:** An individual human or system identity with long-term credentials (password, static access keys). *Modern best practice: Avoid human IAM users; use SSO.*
- **IAM Group:** A collection of IAM users used to attach common permissions.
- **IAM Role:** An identity with temporary security credentials assumed by trusted entities (EC2 instances, Lambda functions, EKS Pods, or cross-account federated users).

---

### **13. What is CloudWatch vs CloudTrail in AWS?**
**Answer:**
- **CloudWatch (Performance & Health):** Monitors operational performance metrics (CPU, disk I/O), application logs, and sets up metric alarms.
- **CloudTrail (Auditing & Governance):** Records **every API call** made in the AWS account (who called what API, when, from which IP, and what parameters were passed).

---

### **14. What is a Managed Database Service (e.g., AWS RDS / Cloud SQL)?**
**Answer:**
Managed relational database engines (PostgreSQL, MySQL, SQL Server) where the cloud provider handles OS installation, security patching, automated daily backups, multi-AZ high availability failover, and point-in-time recovery (PITR).

---

### **15. What is Multi-AZ vs Read Replicas in Amazon RDS?**
**Answer:**
- **Multi-AZ (High Availability):** Synchronous standby replica in a different AZ. Automatically promotes to primary during hardware/AZ failure with zero data loss. Standby cannot serve read traffic.
- **Read Replica (Read Scalability):** Asynchronous replica that serves read-only queries to offload heavy read traffic from the primary database.

---

### **16. What is a CDN (Content Delivery Network) like CloudFront or Cloudflare?**
**Answer:**
A globally distributed network of proxy caching servers that caches web content (images, videos, HTML, JS) geographically close to users to reduce latency, accelerate TLS handshakes, and absorb DDoS attacks.

---

### **17. What is AWS Route 53 and what routing policies does it support?**
**Answer:**
A highly available, scalable Cloud DNS and domain registration service.
- **Routing Policies:** Simple, Weighted, Latency-based, Failover (Health checks), Geolocation, Geoproximity, and Multi-value answer.

---

### **18. What is Cloud Cost Optimization (FinOps) and what are Savings Plans / Reserved Instances?**
**Answer:**
- **On-Demand:** Maximum flexibility, highest hourly cost.
- **Reserved Instances (RI) / Savings Plans:** 1- or 3-year commitment to a consistent amount of compute usage (measured in $/hour) in exchange for up to **72% discount** compared to On-Demand rates.

---

### **19. What is Spot Instance and when should you use it?**
**Answer:**
Spare cloud compute capacity offered at up to **90% discount** compared to On-Demand.
- **Catch:** Cloud provider can reclaim the instance with a **2-minute notification** if demand rises.
- **Use Cases:** Fault-tolerant, stateless, batch processing workloads, containerized workers managed by Karpenter, big data processing (Spark/Hadoop).

---

### **20. What is Infrastructure Drift and how do you prevent it in the cloud?**
**Answer:**
Drift occurs when resources are manually modified via the cloud console or CLI outside of IaC templates.
- **Prevention:** Enforce strict IAM policies denying manual resource modification; manage 100% of infrastructure via CI/CD pipelines and scheduled drift detection.

---

## 🟡 **Intermediate Level (Questions 21–40)**

### **21. What is AWS EKS Pod Identity / IRSA (IAM Roles for Service Accounts) and how does it work?**
**Answer:**
IRSA allows Kubernetes Pods to assume specific AWS IAM roles securely without embedding long-lived AWS Access Keys in container secrets or granting broad IAM permissions to the worker node EC2 instance.

**Mechanism:**
1. EKS cluster acts as an OpenID Connect (OIDC) identity provider.
2. Pod specifies `serviceAccountName: payment-sa`.
3. The Kubernetes ServiceAccount is annotated with the IAM Role ARN:
   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: payment-sa
     annotations:
       eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/PaymentAppS3Role
   ```
4. EKS Mutating Webhook injects AWS credentials token projected into the pod; AWS SDK automatically assumes the role via OIDC token exchange.

---

### **22. What is the difference between VPC Peering and AWS Transit Gateway?**
**Answer:**
- **VPC Peering:** Direct point-to-point network connection between two VPCs.
  - *Limitation:* **Non-transitive** (If VPC A peers with B, and B peers with C, A cannot reach C). Managing $N$ VPCs requires $\frac{N(N-1)}{2}$ peering connections.
- **AWS Transit Gateway (TGW):** Centralized hub-and-spoke router connecting hundreds of VPCs, on-prem VPNs, and Direct Connect connections through a single gateway. Simplifies routing and enables centralized firewall inspection.

---

### **23. What are VPC Endpoints (Gateway vs Interface Endpoints / AWS PrivateLink)?**
**Answer:**
Enables private connectivity between your VPC and AWS services (S3, DynamoDB, ECR) without routing traffic through the public internet or requiring a NAT Gateway.
- **Gateway Endpoints:** Free route table entry for Amazon S3 and DynamoDB only.
- **Interface Endpoints (AWS PrivateLink):** Uses Elastic Network Interfaces (ENIs) with private IP addresses in your subnet to securely access AWS or third-party SaaS services over the AWS private network backbone.

---

### **24. How do you optimize NAT Gateway costs in AWS?**
**Answer:**
AWS NAT Gateways charge both an hourly rate and a high per-GB data processing fee.
1. **Enable Gateway VPC Endpoints for S3 & DynamoDB:** Diverts heavy data traffic (e.g., container image pulls from S3-backed registries) away from NAT Gateway for free.
2. **Enable Interface Endpoints for High-Throughput Services (ECR, CloudWatch Logs):** Keeps traffic on private ENIs.
3. **Consolidate NAT Gateways in Non-Production:** Share a single NAT Gateway across Dev/QA environments instead of one per AZ.

---

### **25. What is AWS Aurora and how does its storage architecture differ from standard RDS?**
**Answer:**
Standard RDS replicates data at the database engine level across EBS volumes.

**Amazon Aurora Architecture:**
- Separates compute from storage.
- Uses a distributed, shared storage volume that spans 3 AZs with **6-way data replication** (2 copies per AZ).
- Quorum-based writes ($4/6$ ACKs required) and reads ($3/6$).
- Read replicas share the exact same underlying storage volume, reducing replication lag to single-digit milliseconds and supporting up to 15 auto-scaling read replicas.

---

### **26. What is the difference between AWS Global Accelerator and Amazon CloudFront?**
**Answer:**
- **CloudFront:** Content Delivery Network (Layer 7 HTTP/HTTPS) that caches static and dynamic content at edge locations worldwide.
- **Global Accelerator:** Networking service (Layer 4 TCP/UDP) that provides static Anycast IP addresses. Routes client traffic immediately into the closest AWS Edge PoP and traverses the private AWS global fiber backbone to backend ALBs/EC2 instances, bypassing congested public internet transit.

---

### **27. How does Workload Identity Federation work in GCP and Azure?**
**Answer:**
Eliminates static service account keys by allowing external workloads (GitHub Actions, on-prem servers, Kubernetes clusters) to authenticate directly to GCP/Azure using industry-standard OIDC tokens.

The cloud identity pool exchanges the signed external JWT token for a temporary, scoped OAuth2 / Azure AD access token.

---

### **28. What is AWS KMS (Key Management Service) Envelope Encryption?**
**Answer:**
Envelope encryption encrypts plaintext data with a **Data Encryption Key (DEK)**, and then encrypts the DEK with a **Root Key (KMS Key / KMS Master Key)**.
- Data is encrypted and decrypted locally in memory by the application using the DEK (high performance, avoids sending large payloads over the network to KMS).
- Only the encrypted DEK is stored alongside the encrypted data.

---

### **29. What is DynamoDB Single-Table Design and why is it used?**
**Answer:**
Single-Table design models all entities, relationships, and access patterns of an application within a **single DynamoDB table** using composite partition keys (`PK`) and sort keys (`SK`) along with Global Secondary Indexes (GSIs).
- Eliminates the need for multiple round-trip network queries.
- Fetches complex relational graphs (e.g., an Order and all its OrderItems) in a single high-performance `Query` API call.

---

### **30. What is Lambda SnapStart and how does it solve Java cold starts?**
**Answer:**
Cold starts for JVM-based Lambda functions can take 5–10 seconds due to JVM initialization and framework dependency injection (Spring Boot, Quarkus).
- **SnapStart:** AWS initializes the function during deployment, creates an encrypted Firecracker microVM snapshot of the initialized memory and disk state, and caches it.
- Subsequent cold start requests resume from the snapshot in **under 200ms**.

---

### **31. What is the difference between SQS Standard Queue and SQS FIFO Queue?**
**Answer:**
- **Standard SQS:** Unlimited throughput (nearly infinite TPS), **at-least-once delivery** (occasional duplicate messages), best-effort ordering (messages may arrive out of sequence).
- **FIFO SQS:** Strictly **first-in-first-out ordering**, exactly-once processing (deduplication IDs), limited throughput (300 msgs/sec without batching, 3000 with batching).

---

### **32. What is AWS EventBridge and how does it enable Event-Driven Architectures?**
**Answer:**
EventBridge is a serverless event bus that ingests events from AWS services, custom applications, and SaaS partners (Datadog, PagerDuty).
- Supports declarative JSON filtering rules to route specific events to targets (Lambda, SQS, Step Functions) without writing custom polling code.

---

### **33. What is S3 Intelligent-Tiering and how does it optimize storage costs?**
**Answer:**
Automatically monitors access patterns and moves objects between access tiers without performance impact or retrieval fees:
- **Frequent Access Tier $\rightarrow$ Infrequent Access Tier (after 30 days without access) $\rightarrow$ Archive Instant Access (after 90 days).**
- Automatically moves objects back to Frequent Access if accessed again.

---

### **34. What is AWS WAF (Web Application Firewall) and what layer does it protect?**
**Answer:**
AWS WAF operates at **Layer 7 (Application Layer)** on CloudFront, ALB, API Gateway, and AppSync.
- Protects against OWASP Top 10 vulnerabilities (SQL Injection, Cross-Site Scripting, HTTP flood rate limiting, bad bot scrapers).

---

### **35. What is AWS Shield Standard vs AWS Shield Advanced?**
**Answer:**
- **Shield Standard (Free):** Automatic protection against common Layer 3 and Layer 4 DDoS attacks (SYN floods, UDP reflection) across all AWS services.
- **Shield Advanced (Paid):** Dedicated 24/7 DDoS Response Team (SRT), real-time attack visibility, financial DDoS cost protection (reimbursing spike charges during an attack), and advanced Layer 7 DDoS mitigation on WAF.

---

### **36. What is a Landing Zone in Cloud Architecture (e.g., AWS Control Tower)?**
**Answer:**
A Landing Zone is a well-architected, multi-account cloud environment based on organizational best practices.
- Automates account provisioning via AWS Organizations.
- Implements centralized identity and access management (IAM Identity Center / SSO).
- Enforces guardrails and compliance via Service Control Policies (SCPs).
- Centralizes logging (CloudTrail, VPC Flow Logs) and security auditing into dedicated, isolated accounts.

---

### **37. What is an AWS Service Control Policy (SCP)?**
**Answer:**
An SCP is an organizational policy used to set the **maximum available permissions** across accounts in an AWS Organization.
- SCPs do *not* grant permissions; they define an impenetrable boundary.
- If an SCP denies an action (e.g., `Deny Delete CloudTrail`), even the root user of a member account cannot perform that action.

---

### **38. What is Cloud Custodian?**
**Answer:**
Cloud Custodian is an open-source rules engine for cloud security, governance, and cost management. It evaluates cloud accounts against YAML-defined policies (e.g., auto-tagging resources, stopping untagged EC2 instances after business hours, remediating unencrypted S3 buckets) in real time.

---

### **39. What is Azure Virtual WAN vs AWS Cloud WAN?**
**Answer:**
Managed global wide area networking services that automate building a transit network architecture connecting global enterprise branch offices, data centers, and multi-region cloud virtual networks across the cloud provider's dedicated global fiber infrastructure.

---

### **40. What is Infracost and how is it integrated into DevOps CI/CD pipelines?**
**Answer:**
Infracost is a FinOps tool that parses Terraform code before deployment, calculates the exact monthly cost delta of proposed infrastructure changes, and posts an interactive cost breakdown comment directly onto the Pull Request.

---

## 🔴 **Advanced & Scenario-Based Level (Questions 41–50)**

### **41. Scenario: You are designing a Multi-Region Active-Active web application across `us-east-1` and `eu-west-1`. How do you architect the data replication and handle conflict resolution?**
**Answer:**
**Architecture:**
1. **Global Traffic Management:** Route 53 Geolocation / Latency-based routing directs users to the nearest regional ALB/EKS cluster.
2. **Database Layer:**
   - **Option A (NoSQL):** Amazon DynamoDB Global Tables with multi-region active-active replication and Last-Write-Wins (LWW) conflict resolution.
   - **Option B (Relational):** Amazon Aurora Global Database. One region acts as Primary Writer; the secondary region maintains storage-level replication (< 1 second lag) and serves local read queries. Writes from the secondary region use Aurora Write Forwarding.
3. **Session & Cache Layer:** Amazon ElastiCache (Redis) Global Datastore replicating cache states across regions.
4. **Static Assets:** S3 Cross-Region Replication (CRR) behind CloudFront with multi-origin failover.

---

### **42. Scenario: Your company receives an unexpected $50,000 monthly cloud bill spike caused by CloudWatch Logs and S3 API calls. How do you investigate, remediate, and establish governance?**
**Answer:**
**Root Cause Investigation:**
1. **AWS Cost Explorer / Cost and Usage Report (CUR):** Filter by Usage Type.
2. **CloudWatch Logs:** Check for runaway microservices writing debug logs in loops. Identify log groups with massive ingestion rates.
3. **S3 Costs:** Check for S3 `ListObjectsV2` API loops executed by poorly written scripts or misconfigured Kubernetes backup jobs.

**Remediation & Governance:**
- Set retention policies (e.g., 14 days or 30 days) on all CloudWatch log groups; configure log lifecycles to transition to S3 Glacier.
- Switch microservices from `DEBUG` to `INFO`/`WARN` in production.
- Enable AWS Budgets with anomaly detection alerts on Slack/email when spend exceeds forecasted thresholds by 15%.

---

### **43. Scenario: Design an automated disaster recovery failover system for a mission-critical PostgreSQL database with an RPO < 1 minute and RTO < 5 minutes.**
**Answer:**
1. **Storage & Engine:** Deploy **Amazon Aurora Global Database** across Primary and Disaster Recovery (DR) regions (storage replication latency is typically < 1000ms, satisfying RPO < 1 min).
2. **Health Monitoring:** Route 53 health check probes a synthetic `/healthz` endpoint verifying database read/write queries in the primary region.
3. **Automated Failover Automation:**
   - Health check triggers an Amazon CloudWatch Alarm.
   - Alarm invokes an AWS Step Functions state machine or Lambda function.
4. **Execution Steps (Under 2 minutes, satisfying RTO < 5 min):**
   - Step 1: Promote the Aurora secondary DB cluster in the DR region to standalone Read/Write cluster (`aws rds failover-global-cluster`).
   - Step 2: Update Route 53 DNS records / Global Accelerator endpoints to point traffic to the DR region.
   - Step 3: Send incident notification to PagerDuty/Slack.

---

### **44. How do you implement Zero-Trust Cloud Network Segmentation without managing hundreds of individual firewall appliances?**
**Answer:**
1. **AWS Network Firewall / Azure Firewall:** Centrally deployed in an inspection VPC attached to AWS Transit Gateway.
2. **AWS Security Groups for Pods / VPC Lattice:** Enforces identity-based Layer 7 access controls directly between microservices.
3. **mTLS via Service Mesh (Istio / Linkerd):** Guarantees encrypted transit and cryptographic workload verification regardless of network topology.

---

### **45. What is the difference between AWS EKS Managed Node Groups, Self-Managed Node Groups, and Karpenter Nodes?**
**Answer:**
- **Self-Managed:** Manual Auto Scaling Groups, custom AMIs, manual rolling node updates and patching. Maximum administrative burden.
- **EKS Managed Node Groups:** AWS manages ASG provisioning, node bootstrapping, and automated rolling node version updates via the EKS console/API.
- **Karpenter:** Completely bypasses ASGs; launches nodes directly via EC2 Fleet API in seconds, dynamically packing diverse instance types, Spot/On-Demand mixes, and auto-consolidating nodes to cut cloud waste.

---

### **46. How do you securely connect an on-premises enterprise data center to multiple AWS VPCs with automatic failover?**
**Answer:**
1. **Primary Link:** **AWS Direct Connect (DX)** with a Direct Connect Gateway connected to an AWS Transit Gateway.
2. **Backup Link:** **IPsec Site-to-Site VPN** configured over the public internet connected to the same Transit Gateway.
3. **Routing Configuration:** Use Border Gateway Protocol (BGP). AWS Direct Connect routes are advertised with higher BGP priority (shorter AS-Path). If Direct Connect fails, BGP automatically fails over traffic to the IPsec VPN tunnel within seconds.

---

### **47. What is Cloud Spanner in GCP and how does it achieve global consistency without violating the CAP theorem?**
**Answer:**
Google Cloud Spanner is a globally distributed, synchronously replicated relational database providing ACID transactions and $99.999\%$ availability.
- **TrueTime API:** Uses synchronized atomic clocks and GPS receivers in every Google data center to bound clock uncertainty ($\epsilon \le 7\text{ms}$).
- This precise global clock allows Spanner to assign monotonically increasing commit timestamps without distributed locking overhead, providing strict external consistency (Serializability).

---

### **48. How do you design an enterprise-grade AWS Multi-Account Organization Structure?**
**Answer:**
```
[ Root Management Account (AWS Organizations) ]
  ├── 📂 Core OU
  │     ├── 🔒 Log Archive Account (Centralized S3 CloudTrail / VPC Flow Logs)
  │     └── 🛡️ Security Tooling Account (SecurityHub, GuardDuty, IAM Identity Center)
  ├── 📂 Infrastructure OU
  │     ├── 🌐 Shared Network Account (Transit Gateway, Direct Connect, Inspection VPC)
  │     └── 🛠️ Shared Services Account (Artifact Registry, CI/CD Runners, Vault)
  └── 📂 Workloads OU
        ├── 🧪 Dev / Staging Workload Accounts (Per-team isolated blast radius)
        └── 🚀 Production Workload Account (Strict SCPs, restricted IAM access)
```

---

### **49. What is Azure Confidential Computing and AWS Nitro Enclaves?**
**Answer:**
Hardware-level trusted execution environments (TEE) that protect data in-use:
- **AWS Nitro Enclaves:** Isolated, hardened compute environments with no persistent storage, no interactive access (no SSH), and no external networking. Connected only via a local virtual socket (`vsock`) to the parent EC2 instance.
- **Use Cases:** Processing sensitive cryptographic keys, PII data, credit cards, or training confidential machine learning models.

---

### **50. Scenario: An engineer accidentally modifies a production security group to open port 22 (`0.0.0.0/0`) to the entire internet. How do you detect and automatically auto-remediate this in under 30 seconds?**
**Answer:**
1. **Detection:** AWS Config rule `incoming-ssh-disabled` detects the non-compliant security group change via CloudTrail event.
2. **Trigger:** AWS Config publishes compliance state change to **Amazon EventBridge**.
3. **Remediation:** EventBridge rule triggers an **AWS Systems Manager (SSM) Automation Document** or Lambda function.
4. **Execution:** The script immediately calls `aws ec2 revoke-security-group-ingress` to remove the `0.0.0.0/0` SSH rule and posts an alert to the SecOps Slack channel with the engineer's IAM identity.
