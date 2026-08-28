# **DevOps Mock Interviews & Scenario-Based Case Studies**

Welcome to the **DevOps Mock Interviews & Scenario-Based Case Studies** module. This section provides complete end-to-end interview simulations for **Senior DevOps Engineer**, **Lead SRE**, and **Principal Platform Engineer** roles, covering system design, live incident debugging walkthroughs, and executive architectural discussions.

---

## 🎯 **Mock Interview Round 1: DevOps & Platform Engineering System Design**

### **Question 1: System Design – Design a Globally Scalable, Ephemeral CI/CD Platform for 1,000+ Software Engineers.**

**Interviewer Prompt:**
*"Our engineering organization has 1,000 developers running 20,000 pipeline builds per day across 300 microservices. Builds take too long, developers complain about pipeline queue wait times, and we frequently hit API rate limits and security vulnerabilities. Design an enterprise-grade CI/CD platform from the ground up."*

**Candidate Architectural Response:**

#### **1. Architecture & Compute Infrastructure:**
- **Control Plane:** GitHub Actions Enterprise / GitLab CI.
- **Compute Runners:** Deploy **Actions Runner Controller (ARC)** on an **Amazon EKS** cluster.
  - Runners run as ephemeral Kubernetes Pods created on-demand via GitHub webhook events (`workflow_job`).
  - Runner pods are destroyed immediately upon job completion to ensure clean security isolation.
  - **Autoscaling:** Use **Karpenter** to provision underlying EC2 Spot instances (e.g., `c6i.4xlarge` and Graviton `c7g.4xlarge`) in under 45 seconds to scale runners from 0 to 1,000+ instances during morning peak hours.

#### **2. Performance & Caching Strategy:**
- **Dependency Caching:** Mount high-speed AWS EFS / Lustre NVMe caching volumes for Maven, Gradle, NPM, and Go module caches.
- **Docker Layer Caching:** Deploy self-hosted **Harbor** / **AWS ECR Pull Through Cache** inside the VPC to cache Docker base images, eliminating external Docker Hub rate limits.
- **BuildKit Remote Cache:** Use inline BuildKit remote caching to reuse intermediate compilation layers across independent runner instances.

#### **3. Security & Compliance Guardrails:**
- **Zero Static Credentials (OIDC):** Configure OpenID Connect (OIDC) between GitHub Actions and AWS IAM / GCP Workload Identity. No long-lived AWS Access Keys are stored in secrets.
- **Supply Chain Security:**
  - **SBOM Generation:** Syft generates CycloneDX SBOMs in pipeline.
  - **Image Signing:** Sigstore Cosign signs container images with keyless OIDC tokens.
  - **Vulnerability Scanning:** Trivy blocks builds containing High/Critical CVEs with available fixes.
- **Runtime Sandboxing:** Enforce `gVisor` runtime on runner pods to prevent untrusted build scripts from escaping containers.

#### **4. Monorepo & Build Optimization:**
- Implement **Turborepo** or **Bazel** with change-graph detection so PRs only build and test the specific microservices modified in the commit.

---

### **Question 2: System Design – Design a Zero-Downtime, Multi-Region Kubernetes Platform for Financial Payments.**

**Interviewer Prompt:**
*"Design a payment processing platform across two cloud regions (`us-east-1` and `eu-west-1`) that can withstand an entire AWS region going offline with zero customer-perceived downtime, strict ACID transaction consistency, and sub-200ms latency."*

**Candidate Architectural Response:**

#### **1. Global Traffic & Ingress:**
- **Anycast Layer:** AWS Global Accelerator provides static Anycast IPs and routes traffic over AWS private fiber backbones directly to the nearest regional Application Load Balancers.
- **DNS Failover:** Route 53 latency routing with automated health checks probing regional `/healthz` endpoints every 10 seconds.
- **Ingress Controller:** Kubernetes **Gateway API** with Envoy Proxy handling Layer 7 routing and mTLS termination.

#### **2. Distributed Compute & GitOps:**
- **Kubernetes Clusters:** Dedicated EKS clusters in each region.
- **Continuous Deployment:** **ArgoCD** deployed in a centralized management cluster synchronizing identical declarative manifests to both regional clusters.
- **Progressive Delivery:** **Argo Rollouts** manages automated Canary deployments with Prometheus metric analysis.

#### **3. High-Consistency Data Architecture:**
- **Relational Data (Transactions):** **CockroachDB Dedicated** or **Amazon Aurora Global Database** with Write-Forwarding.
  - User accounts partitioned regionally by country code to ensure 95% of transactions are local reads/writes (sub-20ms).
  - Cross-region distributed consensus managed via Raft.
- **Caching Layer:** Amazon ElastiCache (Redis) Global Datastore with active-passive replication.
- **Asynchronous Processing:** Apache Kafka (Amazon MSK) with MirrorMaker 2 replicating event topics across regions.

#### **4. Disaster Recovery & Failover Mechanism:**
- If Region A fails, Route 53 health check triggers within 30 seconds $\rightarrow$ shifts 100% of global traffic to Region B.
- Region B's **Karpenter** scales up EKS worker nodes to absorb 2x load in under 60 seconds.

---

## 🚨 **Mock Interview Round 2: Live Production Incident Troubleshooting**

### **Question 3: Live Incident – "Production is Throwing HTTP 504 Gateway Timeouts During Peak Traffic."**

**Interviewer Prompt:**
*"It is 2:00 PM on Black Friday. Our e-commerce checkout service is failing with HTTP 504 Gateway Timeouts. Customers cannot place orders. Walk me through your real-time incident diagnosis and resolution process step-by-step."*

**Candidate Response:**

#### **Phase 1: Immediate Triage & Incident Command (Minutes 0–5)**
1. **Declare Incident:** Open PagerDuty incident bridge, assign Incident Commander (IC), Ops Lead, and Communications Lead.
2. **Post Public Status:** Update status page: *"Investigating checkout delays."*
3. **Check High-Level Dashboards (Grafana):**
   - Identify where 504 originates: Cloudflare $\rightarrow$ ALB $\rightarrow$ Ingress $\rightarrow$ Application Pod $\rightarrow$ Database.
   - ALB 504 indicates that the backend application pod did not respond within the configured idle timeout window (e.g., 60 seconds).

#### **Phase 2: Deep Diagnostics (Minutes 5–15)**
1. **Check Kubernetes Pod Health:**
   ```bash
   kubectl get pods -n prod -l app=checkout-service -o wide
   kubectl top pods -n prod -l app=checkout-service
   ```
2. **Inspect Distributed Traces (Tempo / Jaeger):**
   - Query traces returning HTTP 504.
   - *Finding:* Trace shows `checkout-service` span is blocked for 60 seconds waiting on a PostgreSQL database query: `SELECT * FROM orders WHERE user_id = ? FOR UPDATE`.
3. **Inspect Database Telemetry (PostgreSQL / RDS Performance Insights):**
   - CPU is 100%, Active Connections = 500 (Max Limit reached).
   - `pg_stat_activity` shows dozens of long-running exclusive row locks blocking incoming read/write transactions.

#### **Phase 3: Mitigation & Stabilization (Minutes 15–25)**
1. **Kill Blocking Queries:**
   ```sql
   SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state = 'active' AND query_start < NOW() - INTERVAL '2 minutes';
   ```
2. **Shed Load at Ingress:** Temporarily rate-limit non-critical cart updates and background analytics traffic at Cloudflare WAF.
3. **Scale RDS Proxy / PgBouncer:** Enable AWS RDS Proxy to multiplex incoming connections and prevent connection pool exhaustion.
4. **Verify Telemetry:** Confirm ALB 504 error rate drops to 0% and p99 latency returns to $< 150\text{ms}$.

#### **Phase 4: Blameless Post-Mortem & Permanent Engineering Actions**
- **Root Cause:** A newly introduced query lacked an index on `orders(user_id, status)` and held an exclusive table lock during high concurrency.
- **Action Items:**
  1. Add composite index `CREATE INDEX CONCURRENTLY idx_orders_user_status`.
  2. Implement strict query timeout in application database pool (`statement_timeout = 3000ms`).
  3. Enforce automated database migration review and query analysis in CI pipelines.

---

### **Question 4: Live Incident – "Kubernetes Worker Nodes Randomly Flapping Between `Ready` and `NotReady`."**

**Interviewer Prompt:**
*"During a high-throughput load test, multiple worker nodes in our Kubernetes cluster start flapping between Ready and NotReady. Pods are being evicted and rescheduling storms are crashing the cluster. How do you troubleshoot this?"*

**Candidate Response:**

#### **Step 1: Inspect Node Status & Conditions**
```bash
kubectl describe node <flapping-node>
```
Look for Conditions: `MemoryPressure`, `DiskPressure`, `PIDPressure`, `Ready: False (Kubelet stopped posting node status)`.

#### **Step 2: Check Node System Logs & Resources**
SSH / SSM into the affected worker node:
```bash
# Check CPU, Memory, and Disk I/O Wait
top
iostat -xz 1 5

# Check Kubelet Service Logs
journalctl -u kubelet -n 100 --no-pager

# Check Kernel Logs for OOM or Paging Stalls
dmesg -T | grep -E -i 'oom|hung_task|out of memory'
```

#### **Step 3: Root Cause Analysis Scenarios**
- **Case A: PID Exhaustion (`pids.max`):** A buggy application spawned thousands of zombie threads, exhausting the Linux kernel PID limit. Kubelet cannot fork processes to perform health checks.
  - *Fix:* Increase `/proc/sys/kernel/pid_max` and enforce `podPidsLimit` in Kubelet configuration.
- **Case B: Disk I/O Starvation on `/var/lib/containerd`:** Heavy unbuffered log writes saturated node EBS volume IOPS (100% `%util`), causing `containerd` and `kubelet` heartbeats to time out.
  - *Fix:* Switch to provisioned IOPS (gp3/io2) and enforce container log rotation limits (`containerLogMaxSize: 50Mi`).
- **Case C: Missing System Resource Reservations:** Application pods consumed 100% of node RAM, starving `kubelet` and `systemd`.
  - *Fix:* Enforce `--system-reserved=cpu=500m,memory=1Gi` and `--kube-reserved=cpu=500m,memory=1Gi` in `kubelet-config.yaml`.

---

## 👥 **Mock Interview Round 3: SRE Leadership & Behavioral Scenarios**

### **Question 5: How do you resolve a high-stakes conflict between a Product Manager demanding a new feature release and an SRE Lead enforcing a freeze due to an exhausted Error Budget?**

**Answer:**
1. **Refer to the Pre-Agreed Error Budget Policy:**
   - Emphasize that the Error Budget policy is not an arbitrary SRE decision; it is a shared business agreement previously signed by Product, Engineering, and Leadership.
2. **Data-Driven Transparency:**
   - Present telemetry demonstrating how recent outages affected customer churn, SLA breach penalties, and team on-call burnout.
3. **Collaborative Compromise Options:**
   - **Option A (Feature Flag / Dark Launch):** Deploy the code completely disabled behind a feature flag so developers can proceed with testing without exposing real users to risk.
   - **Option B (Reliability-First Sprint):** Agree on a 2-week dedicated sprint to fix the technical debt causing error budget burn. Once reliability stabilizes and the error budget recovers, feature releases resume immediately.
4. **Executive Escalation:** If business requirements mandate an emergency override (e.g., critical compliance deadline), document the accepted risk formally with VP approval and assign dedicated SRE pairing for the deployment.

---

### **Question 6: How do you establish and nurture a Blameless Post-Mortem Culture across an engineering organization?**

**Answer:**
1. **Focus on Systems, Not Humans:**
   - Start every post-mortem with the **Etsy Blameless Post-Mortem Creed**: *"We assume that engineers act in good faith with the information and tools they had at the time. Human error is the starting point of an investigation, not the conclusion."*
2. **Standardize the Template:**
   - Executive Summary, Incident Timeline (normalized to UTC), Root Cause (5 Whys), What Went Well / What Went Poorly, and SMART Action Items.
3. **Enforce Accountability on Remediation:**
   - Track post-mortem action items in sprint backlogs with clear owners and completion deadlines; review open items in monthly engineering leadership meetings.
4. **Publish Post-Mortems Organically:**
   - Share internal post-mortems in an open `#engineering-retrospectives` Slack channel to foster organizational learning and cross-team empathy.
