# **DevOps Mock Interviews & Scenario-Based Case Studies (50 Scenarios)**

Welcome to the **DevOps Mock Interviews & Scenario-Based Case Studies** master collection containing **50 comprehensive, real-world mock interview simulations** for Senior DevOps, SRE, and Principal Platform Engineer roles.

---

## 🎯 **Round 1: System Design & Platform Architecture (Scenarios 1–20)**

### **Scenario 1: Design a Globally Scalable, Ephemeral CI/CD Platform for 1,000+ Software Engineers.**
**Prompt:** *"Our organization has 1,000 developers running 20,000 builds per day across 300 microservices. Builds take too long and runner queues back up. Design an enterprise CI/CD platform."*
**Detailed Architectural Solution:**
- **Control Plane:** GitHub Actions Enterprise / GitLab CI.
- **Compute Runners:** Actions Runner Controller (ARC) on Amazon EKS with ephemeral runner pods destroyed after each job. Karpenter provisions EC2 Spot instances (`c6i.4xlarge`) in under 45s to scale runners from 0 to 1,000+.
- **Caching Layer:** EFS/NVMe cache for dependencies (`node_modules`, Maven) and local Harbor Pull-Through Cache for Docker base images.
- **Security:** OIDC to AWS STS (zero static keys), Syft SBOM generation, and Cosign keyless image signing.

---

### **Scenario 2: Design a Zero-Downtime, Multi-Region Kubernetes Platform for Financial Payments.**
**Prompt:** *"Design a payment processing platform across `us-east-1` and `eu-west-1` with zero customer-perceived downtime, strict ACID transaction consistency, and sub-200ms latency."*
**Detailed Architectural Solution:**
- **Global Ingress:** AWS Global Accelerator (Anycast IPs) + Route 53 latency routing with automated health check failover.
- **Compute:** Multi-cluster Amazon EKS managed via ArgoCD GitOps and Argo Rollouts for automated canary releases.
- **Data Tier:** CockroachDB Dedicated or Amazon Aurora Global Database with regional customer partitioning.
- **Caching & Streaming:** ElastiCache Redis Global Datastore and Amazon MSK (Kafka) with MirrorMaker 2.

---

### **Scenario 3: Architect an Internal Developer Platform (IDP) with Self-Service Cloud Infrastructure.**
**Prompt:** *"Developers spend weeks waiting for DBAs and DevOps to provision databases and IAM roles. Design a self-service Platform as a Product."*
**Detailed Architectural Solution:**
- **Frontend Portal:** Spotify Backstage software catalog and templates.
- **Control Plane:** Crossplane on Kubernetes with Composite Resource Definitions (XRDs) for `DatabaseInstance` and `ServerlessApp`.
- **Policy Enforcement:** Kyverno validating admission webhooks and Checkov scanning.
- **GitOps Engine:** ArgoCD synchronizing application deployments into target clusters.

---

### **Scenario 4: Design a Multi-Tenant Kubernetes Cluster for 50 Independent Product Teams.**
**Prompt:** *"Design a shared Kubernetes cluster ensuring strict security isolation, fair resource allocation, and zero lateral network access between teams."*
**Detailed Architectural Solution:**
- **Control Plane Isolation:** `vcluster` providing isolated virtual API servers per team.
- **Network Isolation:** Cilium CNI with Default-Deny Layer 7 NetworkPolicies and WireGuard encryption.
- **Resource Governance:** Hierarchical ResourceQuotas, LimitRanges, and PriorityClasses.
- **Security:** Pod Security Standards (`Restricted`) and Kyverno image signature verification.

---

### **Scenario 5: Design a Zero-Trust Cloud Architecture for a FinTech Core Banking System.**
**Prompt:** *"Architect infrastructure compliant with PCI-DSS Level 1 and SOC2 Type II where the internal network is untrusted."*
**Detailed Architectural Solution:**
- **Identity & Attestation:** SPIFFE/SPIRE issuing short-lived X.509 SVID certificates to pods based on node/kernel attestation.
- **Data Protection:** HashiCorp Vault Dynamic Secrets for database credentials and KMS envelope encryption at rest.
- **Access Plane:** Teleport Zero-Trust Access Gateway replacing bastion hosts (no port 22 open).
- **Runtime Security:** Falco eBPF system call monitoring and AWS Security Hub.

---

### **Scenario 6: Design a High-Throughput Real-Time Log Analytics Pipeline Ingesting 50TB Daily.**
**Prompt:** *"Architect a logging pipeline processing 500,000 log events per second with sub-5 second search availability and 1-year compliance retention."*
**Detailed Architectural Solution:**
- **Edge Log Shipper:** Vector or Fluent Bit running as DaemonSets on Kubernetes nodes with memory buffering.
- **Ingestion Buffer:** Apache Kafka cluster (Amazon MSK) absorbing peak traffic spikes and preventing backpressure.
- **Processing Layer:** OpenTelemetry Collector / Vector processing fleet executing PII redaction and JSON schema normalization.
- **Storage & Indexing:** Grafana Loki indexing metadata labels in object storage (Amazon S3 / GCS) with automated daily chunk compaction.

---

### **Scenario 7: Architect a Disaster Recovery Failover System with RPO < 10s and RTO < 2 Minutes.**
**Prompt:** *"Design an automated DR failover solution for an e-commerce platform across AWS primary and secondary regions."*
**Detailed Architectural Solution:**
- **Storage Layer:** Amazon Aurora Global Database with sub-second replication latency across regions.
- **Compute Layer:** Warm Standby EKS cluster running minimal baseline pod replicas in the secondary region.
- **Health Probing:** Route 53 Application Recovery Controller (ARC) continuously evaluating synthetic health check probes.
- **Automated Failover:** Step Functions state machine triggers `aws rds failover-global-cluster`, scales EKS replicas via KEDA, and flips Route 53 DNS routing in under 90 seconds.

---

### **Scenario 8: Design a Cost-Optimized Kubernetes Platform Saving 60% on Cloud Spend.**
**Prompt:** *"Our Kubernetes cloud bill is $200,000/month due to idle compute over-provisioning. Design a comprehensive FinOps optimization architecture."*
**Detailed Architectural Solution:**
- **Autoscaling:** Replace Cluster Autoscaler with **Karpenter** leveraging EC2 Spot instances for stateless workloads and Graviton ARM nodes.
- **Workload Right-Sizing:** Deploy Goldilocks and VPA in recommendation mode; tune container CPU/memory requests to historical p95 usage.
- **Scale-to-Zero:** Deploy KEDA to scale non-production environments to 0 replicas outside business hours.
- **Shift-Left FinOps:** Infracost integrated into CI/CD PR builds to display monthly cost impacts before merging.

---

### **Scenario 9: Design a Secure Software Supply Chain Conforming to SLSA Level 3.**
**Prompt:** *"Prevent malicious code injections, dependency tampering, and unauthorized container builds across 500 repositories."*
**Detailed Architectural Solution:**
- **Build Isolation:** Hermetic, ephemeral GitHub Actions runners executing in isolated containers with zero external internet access.
- **SBOM Generation:** Syft generates CycloneDX Software Bill of Materials during build steps.
- **Keyless Signing:** Sigstore Cosign cryptographically signs container images and SBOMs with short-lived OIDC certificates from Fulcio.
- **Admission Verification:** Kyverno admission controller in Kubernetes verifies Cosign signatures against Rekor transparency logs before admitting pods.

---

### **Scenario 10: Architect a High-Availability Multi-Region Redis Caching Layer.**
**Prompt:** *"Design a global caching layer for a social media application providing sub-5ms read latency across US and Europe."*
**Detailed Architectural Solution:**
- **Service:** Amazon ElastiCache (Redis) Global Datastore with active cluster in `us-east-1` and read replicas in `eu-west-1`.
- **Cache Invalidation:** EventBridge fan-out publishing cache eviction events to regional Redis clusters upon database updates.
- **Resilience:** Singleflight mutex locking in client SDKs preventing Cache Breakdown and Stampede storms.

---

### **Scenario 11: Design a GitOps-Driven Multi-Cluster Kubernetes Deployment Platform.**
**Prompt:** *"Manage 100 Kubernetes clusters across dev, staging, and production globally from a single declarative control plane."*
**Detailed Architectural Solution:**
- **Management Plane:** Dedicated central management EKS cluster running **ArgoCD** with SSO and RBAC.
- **Generators:** `ApplicationSet` using Git Directory and Cluster generators to automatically instantiate applications across all registered clusters.
- **Sync Ordering:** ArgoCD Sync Waves and Phases coordinating database migrations (PreSync) before application updates.
- **Drift Governance:** Self-healing auto-sync overwriting unauthorized out-of-band manual changes in cluster state.

---

### **Scenario 12: Design a Zero-Trust Developer Access Gateway for Cloud Infrastructure.**
**Prompt:** *"Replace shared SSH keys, VPNs, and static database passwords for 200 engineers with a secure, auditable access model."*
**Detailed Architectural Solution:**
- **Identity Plane:** Enterprise Okta SSO with mandatory FIDO2 hardware MFA keys.
- **Access Gateway:** Teleport Access Plane providing short-lived (8-hour) cryptographic certificates for SSH, Kubernetes API, and PostgreSQL databases.
- **Session Auditing:** 100% of interactive terminal sessions recorded in JSON/video format stored in encrypted S3 buckets for SOC2 compliance.
- **Just-In-Time Access:** Dual-authorization workflows where engineers request temporary elevated privileges approved in Slack.

---

### **Scenario 13: Architect a Serverless Event-Driven Processing Pipeline Handling 100M Daily Events.**
**Prompt:** *"Design an IoT telemetry processing pipeline handling 100M events per day with variable traffic bursts and zero server management."*
**Detailed Architectural Solution:**
- **Ingestion:** Amazon API Gateway / AWS IoT Core writing to Amazon Kinesis Data Streams partitioned by `device_id`.
- **Processing:** AWS Lambda functions consuming Kinesis batches with parallelization factor, executing validation and transformations.
- **Storage:** DynamoDB Single-Table design storing real-time state with TTL; Amazon S3 data lake via Kinesis Firehose in Parquet format.
- **Error Handling:** Dead Letter Queues (DLQs) capturing malformed payloads with automated Step Functions redrive workflows.

---

### **Scenario 14: Design a Global Anycast Network Architecture for DDoS Resilience.**
**Prompt:** *"Protect an enterprise web platform from 1 Tbps Layer 3/4 and Layer 7 volumetric DDoS attacks."*
**Detailed Architectural Solution:**
- **Global Edge:** Cloudflare / AWS Global Accelerator Anycast IP routing absorbing traffic across 300+ edge data centers worldwide.
- **WAF Layer:** AWS WAF / Cloudflare WAF with automated Rate-Based rules and Bot Control managed rule sets.
- **Origin Cloaking:** Backend ALBs placed in private subnets accessible *only* from CDN edge IPs via custom HTTP header validation.
- **Kernel Tuning:** SYN Cookies enabled (`net.ipv4.tcp_syncookies=1`) and TCP backlog tuning on origin servers.

---

### **Scenario 15: Design a Microservices Service Mesh with End-to-End Encryption (mTLS).**
**Prompt:** *"Implement zero-trust mutual TLS, traffic splitting, and distributed tracing across 80 Kubernetes microservices without changing application code."*
**Detailed Architectural Solution:**
- **Control Plane:** Istio Service Mesh (`istiod`) or Istio Ambient Mesh.
- **Data Plane:** Envoy sidecars / node-level `ztunnel` daemons enforcing strict mTLS (`PeerAuthentication: STRICT`).
- **Traffic Management:** `VirtualService` and `DestinationRule` managing 90/10 canary traffic splitting.
- **Observability:** OpenTelemetry distributed tracing spans injected automatically by Envoy and streamed to Grafana Tempo.

---

### **Scenario 16: Architect a Multi-Cloud Disaster Recovery Strategy Across AWS and Google Cloud.**
**Prompt:** *"Ensure business continuity even if an entire primary cloud provider (AWS) experiences a catastrophic nationwide outage."*
**Detailed Architectural Solution:**
- **Compute:** Kubernetes (EKS on AWS, GKE on GCP) deploying container images built from identical Git commits.
- **Data Tier:** CockroachDB Dedicated spanning nodes across AWS and GCP with multi-cloud Raft consensus.
- **Routing:** Cloudflare DNS Anycast routing with automated health checks probing endpoints across both clouds.
- **CI/CD:** GitHub Actions building multi-cloud images and deploying via centralized ArgoCD.

---

### **Scenario 17: Design an Enterprise Observability Platform Correlating Metrics, Logs, and Traces.**
**Prompt:** *"Eliminate 5 disconnected monitoring tools and build a unified, high-scale observability stack for 200 microservices."*
**Detailed Architectural Solution:**
- **Collection Layer:** OpenTelemetry Collector daemonsets on all nodes ingesting OTLP metrics, logs, and traces.
- **Metrics Store:** Grafana Mimir / Prometheus with long-term S3 storage and Alertmanager SLO burn-rate alerts.
- **Logs Store:** Grafana Loki indexing stream labels and storing compressed chunks in S3.
- **Traces Store:** Grafana Tempo with Exemplar linking, enabling 1-click navigation from PromQL metric spikes to distributed traces.

---

### **Scenario 18: Architect a Microservices Database Migration with Zero Customer Downtime.**
**Prompt:** *"Migrate a 2TB PostgreSQL database with 5,000 active transactions/sec from on-premise to AWS Aurora with zero data loss."*
**Detailed Architectural Solution:**
- **Schema Preparation:** AWS Schema Conversion Tool (SCT) validating schema parity.
- **Initial Load & Replication:** AWS DMS (Database Migration Service) performing initial full load and ongoing Change Data Capture (CDC).
- **Application Architecture:** Expand and Contract pattern with dual-writing and asynchronous data backfilling.
- **Cutover:** Validate replication lag is $< 100\text{ms}$; point application connection strings to Aurora endpoint during a 30-second maintenance window.

---

### **Scenario 19: Design an Automated Database Sharding Architecture for 100M Users.**
**Prompt:** *"A single relational database is maxing out write IOPS. Design a horizontally sharded database architecture."*
**Detailed Architectural Solution:**
- **Sharding Strategy:** Horizontal range or hash sharding based on `hash(user_id) % num_shards`.
- **Routing Layer:** Vitess or Citus Data proxying SQL queries, routing requests directly to the target shard instance.
- **Cross-Shard Operations:** Minimize cross-shard joins by co-locating related entity tables on the same physical shard.
- **Re-sharding:** Consistent hashing algorithms allowing dynamic addition of new shards with minimal data rebalancing.

---

### **Scenario 20: Design an Automated Secret Rotation Pipeline for 500 Databases.**
**Prompt:** *"Eliminate static passwords across 500 relational databases and automate zero-downtime secret rotation every 30 days."*
**Detailed Architectural Solution:**
- **Secrets Engine:** HashiCorp Vault Database Engine configured with dynamic PostgreSQL/MySQL user generators.
- **Workload Injection:** External Secrets Operator (ESO) syncing dynamic credentials into in-memory Kubernetes Secret volumes.
- **Rotation Lifecycle:** Vault generates new database credentials, updates the secret, waits for the lease TTL, and drops expired users automatically.

---

## 🚨 **Round 2: Live Production Incident Troubleshooting (Scenarios 21–40)**

### **Scenario 21: Live Incident – "Production is Throwing HTTP 504 Gateway Timeouts During Black Friday Peak Traffic."**
**Triage Walkthrough:**
1. Check ALB metrics: 504 indicates backend pods did not respond within idle timeout.
2. Query distributed traces in Tempo: Locate slow spans blocked on PostgreSQL `SELECT ... FOR UPDATE`.
3. Check database telemetry: Active connection pool exhausted (500/500 connections) due to exclusive row locks.
4. Immediate Mitigation: Terminate blocking queries (`pg_terminate_backend`), shed non-critical traffic at Cloudflare WAF, and enable AWS RDS Proxy.
5. Permanent Fix: Add missing composite index, enforce `statement_timeout = 3000ms`, and review query lock semantics.

---

### **Scenario 22: Live Incident – "Kubernetes Worker Nodes Randomly Flapping Between Ready and NotReady."**
**Triage Walkthrough:**
1. Run `kubectl describe node <node>`: Check for `PIDPressure` or `Kubelet stopped posting status`.
2. Inspect node via SSM: `top`, `journalctl -u kubelet`, `dmesg -T`.
3. Root Cause: Buggy microservice spawned 30,000 zombie threads, exhausting `/proc/sys/kernel/pid_max`. Kubelet could not fork health check processes.
4. Mitigation: Kill rogue process, increase `pid_max`, and configure `podPidsLimit: 4096` in `kubelet-config.yaml`.

---

### **Scenario 23: Live Incident – "Cascading Database Crash Loop Caused by Thundering Herd Retries."**
**Triage Walkthrough:**
1. Database restarts, but 5,000 app pods immediately hammer it with reconnect queries, crashing it again in 10 seconds.
2. Mitigation: Scale down application deployments to 0 replicas, restart database cleanly, and configure connection poolers (PgBouncer).
3. Scale app back up incrementally with **Exponential Backoff and Jitter** in the database client library.

---

### **Scenario 24: Live Incident – "Prometheus Server Crashing with Out-Of-Memory (OOM) Every Morning."**
**Triage Walkthrough:**
1. Query TSDB Status API (`/api/v1/status/tsdb`): Identify metric with highest cardinality.
2. Root Cause: A newly deployed microservice tagged metrics with dynamic `user_id` labels, generating 5 million unique time series.
3. Fix: Apply `metric_relabel_configs` in Prometheus scrape config with `action: labeldrop` for `user_id`.

---

### **Scenario 25: Live Incident – "Production S3 Bucket Publicly Accessible After Terraform Apply."**
**Triage Walkthrough:**
1. Immediate Containment: Enable S3 Block Public Access at the AWS Account level (`aws s3control put-public-access-block`).
2. Audit CloudTrail: Query who created the change and verify if data was downloaded.
3. IaC Fix: Add `aws_s3_bucket_public_access_block` to Terraform and enforce Checkov policy blocking unblocked S3 buckets in CI.

---

### **Scenario 26: Live Incident – "CoreDNS Pods Crashing with OOMKilled Causing Cluster-Wide Outage."**
**Triage Walkthrough:**
1. Symptoms: Pods across all namespaces cannot resolve internal or external domain names; CoreDNS pods restarting with Exit Code 137.
2. Root Cause: Linux `ndots:5` configuration in 1,000 application pods generating 4 NXDOMAIN queries per lookup, overwhelming CoreDNS memory.
3. Mitigation: Increase CoreDNS memory limits, deploy **NodeLocal DNSCache** DaemonSet, and autoscale CoreDNS via `cluster-proportional-autoscaler`.

---

### **Scenario 27: Live Incident – "Kubernetes Node Disk 100% Full Due to Container Logs."**
**Triage Walkthrough:**
1. Symptoms: Kubelet evicts all pods on worker node; node condition shows `DiskPressure`.
2. Recovery: Delete rotated `.gz` log files under `/var/log/pods/` and prune unused images via `crictl rmi --prune`.
3. Permanent Fix: Enforce container runtime log rotation limits in `kubelet-config.yaml` (`containerLogMaxSize: 50Mi`, `containerLogMaxFiles: 3`).

---

### **Scenario 28: Live Incident – "Corrupted Git Master Branch Caused by Force Push."**
**Triage Walkthrough:**
1. Symptoms: An engineer ran `git push --force` on `main`, deleting 30 shared commits.
2. Recovery: Use `git reflog` on an engineer's machine who had the previous state, locate the commit SHA before the force push, and force-push the recovered commit back.
3. Prevention: Enable GitHub Branch Protection rules with "Block force pushes" on `main`.

---

### **Scenario 29: Live Incident – "Terraform State Lock Stuck During Production Deployment."**
**Triage Walkthrough:**
1. Symptoms: CI pipeline fails with `Error: Error acquiring the state lock: ConditionalCheckFailedException`.
2. Diagnosis: A previous pipeline runner crashed or was killed midway while holding the lock.
3. Recovery: Inspect lock metadata (Lock ID, Owner, Created timestamp). If the previous process is confirmed dead, run `terraform force-unlock <LOCK-ID>`.

---

### **Scenario 30: Live Incident – "SSL Certificate Expired on Production Ingress Controller."**
**Triage Walkthrough:**
1. Symptoms: Users receive browser SSL warnings (`ERR_CERT_DATE_INVALID`).
2. Diagnosis: `cert-manager` failed automated renewal due to a broken Let's Encrypt HTTP-01 challenge ingress route.
3. Mitigation: Fix ingress route, trigger manual certificate re-issuance (`cmctl renew <cert-name>`), and configure Prometheus Blackbox alert for cert expiration $< 14$ days.

---

### **Scenario 31: Live Incident – "AWS NAT Gateway Reaching Bandwidth Limit."**
**Triage Walkthrough:**
1. Symptoms: EC2 instances in private subnets experience packet drops and extreme download latencies.
2. Diagnosis: CloudWatch metric `PacketsDropCount` is elevated on NAT Gateway.
3. Mitigation: Deploy **Amazon S3 Gateway Endpoints** and **ECR Interface VPC Endpoints** to keep heavy storage and container image traffic on private internal networks for free.

---

### **Scenario 32: Live Incident – "PostgreSQL Database Max Connections Reached."**
**Triage Walkthrough:**
1. Symptoms: Application logs show `FATAL: remaining connection slots are reserved for non-superuser connections`.
2. Immediate Action: Terminate idle connections (`pg_terminate_backend`) and temporarily increase `max_connections`.
3. Permanent Fix: Deploy **AWS RDS Proxy** or **PgBouncer** connection poolers between application pods and the database.

---

### **Scenario 33: Live Incident – "JVM Memory Leak Causing Silent Degradation and CPU Spikes."**
**Triage Walkthrough:**
1. Symptoms: Application latency gradually climbs over 24 hours; Garbage Collection (GC) CPU usage spikes to 100%.
2. Diagnosis: Capture Java heap dump (`jcmd <PID> GC.heap_dump /tmp/dump.hprof`) and thread dump (`jstack`).
3. Analysis: Analyze heap dump in Eclipse Memory Analyzer (MAT) to identify leaking static HashMaps or unclosed database connections.

---

### **Scenario 34: Live Incident – "Kafka Consumer Group Lag Spikes to Millions of Messages."**
**Triage Walkthrough:**
1. Symptoms: Event processing delay exceeds hours; Prometheus metric `kafka_consumergroup_lag` is spiking.
2. Diagnosis: A poisoned message payload is causing consumer pods to throw unhandled exceptions and crash in retry loops.
3. Mitigation: Route poisoned message to a Dead Letter Queue (DLQ), commit the offset, and scale up consumer pod replicas via KEDA.

---

### **Scenario 35: Live Incident – "Elasticsearch Cluster Status RED Due to Unassigned Shards."**
**Triage Walkthrough:**
1. Symptoms: Search queries failing; Elasticsearch cluster health reports `RED`.
2. Diagnosis: Query `_cluster/allocation/explain` to identify why shards cannot allocate (disk watermark breached).
3. Mitigation: Increase disk volume capacity on Elasticsearch nodes and run `_cluster/reroute` to force shard allocation.

---

### **Scenario 36: Live Incident – "Kubernetes Pod Stuck in Terminating State for Hours."**
**Triage Walkthrough:**
1. Symptoms: `kubectl delete pod` hangs indefinitely.
2. Diagnosis: Check for blocking Finalizers: `kubectl get pod <name> -o jsonpath='{.metadata.finalizers}'`.
3. Mitigation: Remove finalizers (`kubectl patch pod <name> -p '{"metadata":{"finalizers":null}}'`) or force delete with `--grace-period=0 --force`.

---

### **Scenario 37: Live Incident – "Linux Kernel OOM Killer Terminating Critical MySQL Database."**
**Triage Walkthrough:**
1. Symptoms: Database service crashes abruptly with no error in database logs; `dmesg -T` shows `Out of memory: Kill process (mysqld)`.
2. Root Cause: Application processes on the same host consumed RAM, or MySQL `innodb_buffer_pool_size` was set to 90% of total host RAM.
3. Fix: Set `innodb_buffer_pool_size` to 70% of RAM and set `oom_score_adj = -1000` to prevent the kernel from killing the database.

---

### **Scenario 38: Live Incident – "BGP Route Flapping Causing Intermittent Direct Connect Outages."**
**Triage Walkthrough:**
1. Symptoms: On-premises to AWS connectivity drops every 5 minutes.
2. Diagnosis: Check BGP state in AWS Direct Connect router logs (BGP session flapping due to keepalive timeouts).
3. Fix: Tune BGP keepalive and hold timers on Customer Gateway router and enable BFD (Bidirectional Forwarding Detection) for sub-second failover.

---

### **Scenario 39: Live Incident – "AWS IAM Role Compromised via Leaked Credentials."**
**Triage Walkthrough:**
1. Immediate Revocation: Attach an inline IAM policy denying all actions (`Effect: Deny, Action: *`) with a condition matching the compromised session name.
2. Invalidate Sessions: Use `aws iam put-role-policy` to revoke all active temporary STS sessions issued prior to the current timestamp.
3. Forensic Audit: Query CloudTrail logs in Athena to determine all API calls made by the attacker.

---

### **Scenario 40: Live Incident – "Kubernetes Ingress Controller Returning HTTP 502 Bad Gateway."**
**Triage Walkthrough:**
1. Diagnosis: Nginx Ingress logs show `502 Bad Gateway: connect() failed (111: Connection refused) while connecting to upstream`.
2. Root Cause: Application container crashed or readiness probe failed, but EndpointSlice had not yet updated.
3. Fix: Add `preStop` sleep hook (`sleep 5`) to application PodSpec and ensure readiness probe accurately reflects process health.

---

## 👥 **Round 3: SRE Leadership & Behavioral Scenarios (Scenarios 41–50)**

### **Scenario 41: Resolving Conflict Between Product Velocity and SRE Error Budget Freezes.**
**Answer:**
- Emphasize that the Error Budget policy is an agreed-upon business contract signed by leadership.
- Present concrete telemetry on customer churn and SLA breach financial penalties.
- Propose pragmatic compromises: deploy code disabled behind feature flags (Dark Launching) while dedicating a 2-week sprint to technical debt and reliability fixes.

---

### **Scenario 42: Establishing a Blameless Post-Mortem Culture.**
**Answer:**
- Reiterate that human error is the starting point of an investigation, not the conclusion.
- Standardize post-mortem templates: Executive Summary, UTC Timeline, 5 Whys Root Cause, and SMART Action Items.
- Review and track remediation items in sprint backlogs to ensure systemic fixes are delivered.

---

### **Scenario 43: Leading a Cross-Functional Chaos Engineering GameDay.**
**Answer:**
- Define hypothesis (e.g., "When a 50% packet loss is injected on payment pods, circuit breakers trip and checkout success rate stays $> 99.9\%$").
- Deploy Chaos Mesh on staging, execute network delay experiments, observe telemetry dashboards, and document weaknesses found.

---

### **Scenario 44: Managing On-Call Burnout and Alert Fatigue in SRE Teams.**
**Answer:**
- Conduct alert audits: delete noisy, non-actionable threshold alerts.
- Shift to SLO Multi-Window Multi-Burn-Rate alerting.
- Enforce the 50% toil cap rule and rotate on-call schedules with secondary backups.

---

### **Scenario 45: Driving FinOps Cloud Cost Reduction Across 20 Engineering Teams.**
**Answer:**
- Implement mandatory tagging policies (`Owner`, `Environment`, `CostCenter`) enforced via AWS SCPs.
- Embed Infracost cost estimates directly into GitHub PR comments.
- Migrate compute workloads to Graviton ARM processors and configure Karpenter Spot instance consolidation.

---

### **Scenario 46: Designing an On-Call Handoff and Rotation Process.**
**Answer:**
- Conduct weekly scheduled 30-minute on-call handoff meetings between outgoing and incoming on-call engineers.
- Review all alerts that fired during the shift, identify noisy alerts for tuning, and transfer open incident follow-up action items.

---

### **Scenario 47: Managing Stakeholder Communications During a Major Severity 1 Outage.**
**Answer:**
- Appoint a dedicated Communications Lead on the incident bridge, freeing technical responders to focus on diagnosis.
- Publish public status page updates every 15–30 minutes with clear, empathetic summaries of current mitigation progress without technical jargon.

---

### **Scenario 48: Driving Organization-Wide Migration to an Internal Developer Platform (IDP).**
**Answer:**
- Treat the platform as a product: interview product engineering teams to discover daily pain points.
- Build "Golden Paths" that make compliant development the path of least resistance (e.g., 1-click compliant microservice scaffolding).

---

### **Scenario 49: Mentoring Junior DevOps Engineers into High-Performing SREs.**
**Answer:**
- Implement structured shadow on-call rotations where junior engineers observe incident responses before holding the primary pager.
- Pair program on complex IaC refactoring and automation scripting projects.

---

### **Scenario 50: Justifying Infrastructure Investment and Technical Debt Refactoring to Executive Leadership.**
**Answer:**
- Translate technical debt into direct business risk metrics: lost revenue per hour of downtime, customer churn rates, developer wait time costs, and security compliance exposure.
