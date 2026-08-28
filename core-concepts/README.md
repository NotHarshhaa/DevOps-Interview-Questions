# **Core Concepts - DevOps, SRE & Platform Engineering (200 Questions)**

Welcome to the **Core Concepts** master collection containing **200 comprehensive interview questions and detailed answers** covering DevOps Fundamentals, Site Reliability Engineering (SRE), Platform Engineering, DORA Metrics, Cloud FinOps, SDLC Integration, and Architectural Frameworks.

---

## 🟢 **Part 1: DevOps Fundamentals & Culture (Questions 1–50)**

### **1. What is DevOps and what core problem does it solve?**
**Answer:** DevOps is a cultural, organizational, and technical philosophy combining Software Development (Dev) and IT Operations (Ops). It breaks down organizational silos to enable faster, more reliable software delivery through automation, continuous integration, continuous delivery, and shared responsibility. It eliminates the "Wall of Confusion" where developers wrote code without operational awareness and operators deployed code without understanding development context.

### **2. What are the Three Ways of DevOps?**
**Answer:**
1. **Flow (Left-to-Right):** Accelerate the flow of work from development to operations to customers by minimizing work-in-progress (WIP) and automating delivery pipelines.
2. **Feedback (Right-to-Left):** Establish rapid, continuous feedback loops from production to development via telemetry and retrospectives.
3. **Continuous Learning & Experimentation:** Foster a high-trust culture that encourages risk-taking, experimentation, and institutional learning from failure.

### **3. What is the difference between DevOps and Agile?**
**Answer:** Agile focuses on iterative software development, rapid customer feedback, and adaptive planning during the coding and product management phases. DevOps extends Agile principles across the entire lifecycle, encompassing build, test, infrastructure provisioning, deployment, monitoring, and live operations.

### **4. What are DORA Metrics? List all four core metrics.**
**Answer:**
1. **Deployment Frequency (DF):** How often an organization deploys code to production.
2. **Lead Time for Changes (LTFC):** Time from code commit to running in production.
3. **Change Failure Rate (CFR):** Percentage of deployments requiring immediate hotfix/rollback.
4. **Time to Restore Service (TTRS / MTTR):** Time to recover from a production outage.

### **5. What benchmarks define an "Elite" DORA performer?**
**Answer:** Elite teams achieve: Deployment Frequency on-demand (multiple times per day), Lead Time for Changes $< 1$ hour, Change Failure Rate between 0%–15%, and Time to Restore Service $< 1$ hour.

### **6. What is Continuous Integration (CI)?**
**Answer:** CI is the engineering practice where developers integrate code into a shared repository frequently. Every merge triggers automated builds, linters, and unit/integration test suites to detect bugs early.

### **7. What is Continuous Delivery (CD) vs Continuous Deployment (CD)?**
**Answer:** Continuous Delivery automates software release staging so code is always production-ready, requiring a single manual approval to deploy. Continuous Deployment automates the entire journey—every passing commit deploys directly into live production with zero human intervention.

### **8. What is Infrastructure as Code (IaC)?**
**Answer:** Managing and provisioning compute, storage, and networking infrastructure via machine-readable definition files (e.g., Terraform HCL, Kubernetes YAML) rather than manual console interaction.

### **9. What is Configuration Drift?**
**Answer:** The divergence of live infrastructure settings from the declared IaC templates in Git, usually caused by manual emergency changes made directly in cloud consoles or servers.

### **10. What is Immutable Infrastructure?**
**Answer:** An operational paradigm where servers or containers are never patched or modified in-place. Updates require building a new image/container from scratch, testing it, and terminating the old instance.

### **11. What is the "You Build It, You Run It" model?**
**Answer:** A foundational DevOps organizational model coined by Werner Vogels (CTO of Amazon) where the engineering team that writes a service is also responsible for deploying, operating, and supporting it in production.

### **12. What is Shift-Left Security (DevSecOps)?**
**Answer:** Integrating security testing (SAST, SCA, secret scanning, container scanning) early in the software development lifecycle rather than conducting security reviews as an afterthought before release.

### **13. What is a Monolith vs Microservices Architecture?**
**Answer:** Monoliths bundle all business logic, database access, and UI into a single codebase and deployment unit. Microservices decompose functionality into autonomous, loosely coupled services communicating over network APIs (REST, gRPC, Kafka).

### **14. What are the Twelve Factors of the 12-Factor App?**
**Answer:** 1. Codebase, 2. Dependencies, 3. Config, 4. Backing Services, 5. Build/Release/Run, 6. Processes, 7. Port Binding, 8. Concurrency, 9. Disposability, 10. Dev/Prod Parity, 11. Logs, 12. Admin Processes.

### **15. What is Trunk-Based Development?**
**Answer:** A branching strategy where all developers commit small, frequent changes directly to a single shared branch (`main`), avoiding long-lived feature branches and merge conflicts.

### **16. What is GitFlow and why is it often considered an anti-pattern for CI/CD?**
**Answer:** GitFlow uses long-lived branches (`develop`, `feature`, `release`, `hotfix`, `master`). It delays integration testing, causes painful merge conflicts ("merge hell"), and slows deployment velocity.

### **17. What is a Blue-Green Deployment?**
**Answer:** Maintaining two identical production environments: Blue (active production) and Green (idle staging). Traffic is switched instantly via the load balancer from Blue to Green upon successful verification.

### **18. What is a Canary Deployment?**
**Answer:** Releasing a new software version to a small percentage of live users (e.g., 5%) while monitoring telemetry. If error rates remain healthy, traffic is incrementally shifted to 100%.

### **19. What is a Feature Flag (Toggle)?**
**Answer:** A conditional code statement that decouples code deployment from feature release, allowing features to be toggled on/off in production via an API or UI without redeploying code.

### **20. What is Dark Launching?**
**Answer:** Deploying new backend code to production to process live data or shadow requests in the background without exposing any visible UI changes to users.

### **21. What is Observability vs Monitoring?**
**Answer:** Monitoring tracks predefined metrics against known thresholds ("Is the system up?"). Observability allows engineers to infer the internal state of a system based on its telemetry outputs ("Why is the system broken?").

### **22. What are the Three Pillars of Observability?**
**Answer:** Metrics (aggregable numbers), Logs (timestamped discrete event records), and Distributed Traces (request lifecycle spans across services).

### **23. What is Chaos Engineering?**
**Answer:** The discipline of proactively injecting controlled failures (killing pods, adding network latency, severing database links) into distributed systems to verify resilience before real outages occur.

### **24. What is FinOps?**
**Answer:** Cloud Financial Operations—a cross-functional framework bringing financial accountability to variable cloud spending, optimizing cloud unit economics while maintaining engineering velocity.

### **25. What is SRE (Site Reliability Engineering)?**
**Answer:** A discipline pioneered by Google applying software engineering principles to operations and infrastructure problems to maximize system reliability and availability.

### **26. What is an Internal Developer Platform (IDP)?**
**Answer:** A self-service portal (e.g., Backstage) providing "Golden Paths" that automate infrastructure provisioning, CI/CD, and compliance to reduce developer cognitive load.

### **27. What is a Golden Path (Paved Road)?**
**Answer:** A supported, standardized, and automated route for building and deploying software that incorporates all organizational security, infrastructure, and compliance best practices by default.

### **28. What is Cognitive Load in software engineering?**
**Answer:** The total mental effort required for a developer to perform their job. High cognitive load (managing Docker, K8s, Terraform, IAM, monitoring) slows feature velocity; Platform Engineering aims to minimize it.

### **29. What is Conway’s Law?**
**Answer:** Organizations design systems that mirror their own communication structures. Siloed teams build monolithic architectures; decoupled cross-functional teams build microservice architectures.

### **30. What is Idempotency?**
**Answer:** A property of an operation where executing it once or multiple times produces the exact same end-state result without side effects.

### **31. What is Configuration Management?**
**Answer:** Automating the installation of packages, OS kernel parameters, and configuration files across servers in a consistent, repeatable state (e.g., Ansible, Puppet).

### **32. What is a Deployment Pipeline?**
**Answer:** An automated implementation of the build, test, package, and deploy steps that software undergoes from developer commit to production release.

### **33. What is a Rollback Strategy?**
**Answer:** A predefined, automated procedure to revert a system to its previous known stable version when a deployment introduces defects.

### **34. What is Artifact Repository Management?**
**Answer:** Storing and versioning compiled binaries, container images, Helm charts, and language packages in centralized registries (e.g., Nexus, Artifactory, GitHub Packages).

### **35. What is Continuous Feedback?**
**Answer:** Continuously gathering APM metrics, user analytics, error logs, and customer feedback from production and integrating them into product backlogs.

### **36. What is the Single Source of Truth principle?**
**Answer:** Storing all application configurations, infrastructure code, and operational policies in a single centralized, version-controlled repository (typically Git).

### **37. What is Technical Debt in DevOps?**
**Answer:** The implied cost of choosing quick, suboptimal workarounds (manual server patches, hardcoded configs) over well-architected solutions, compounding future operational maintenance overhead.

### **38. What is Zero-Downtime Deployment?**
**Answer:** Deploying new application versions without interrupting active user requests or causing connection drops (achieved via Rolling Updates, Blue-Green, or Canary deployments).

### **39. What is Backward Compatibility in API design?**
**Answer:** Ensuring that updates to APIs or database schemas do not break older client applications or existing data consumers.

### **40. What is Semantic Versioning (SemVer)?**
**Answer:** A standard versioning convention: `MAJOR.MINOR.PATCH` (e.g., `2.1.0`), where MAJOR indicates breaking changes, MINOR indicates backward-compatible features, and PATCH indicates bug fixes.

### **41. What is Self-Healing Infrastructure?**
**Answer:** Systems designed to automatically detect and remediate failures without human intervention (e.g., Kubernetes restarting crashed containers or cloud Auto Scaling Groups replacing unhealthy instances).

### **42. What is a Monorepo vs Polyrepo?**
**Answer:** Monorepos hold multiple projects/microservices in a single repository; Polyrepos store each microservice in its own independent repository.

### **43. What is Shift-Right Testing?**
**Answer:** Testing software in live production environments under real user traffic using feature flags, canary rollouts, synthetic monitoring, and chaos experiments.

### **44. What is Telemetry in DevOps?**
**Answer:** The automated collection and transmission of operational measurements, performance data, and diagnostics from distributed systems to monitoring tools.

### **45. What is a Blameless Culture?**
**Answer:** A high-trust workplace culture that treats errors as systemic opportunities to improve tooling and processes rather than punishing individual human mistakes.

### **46. What is a GameDay in SRE?**
**Answer:** A scheduled exercise where engineering teams simulate production outages and disaster recovery failovers to validate monitoring, alerting, and operational runbooks.

### **47. What is a Dead Letter Queue (DLQ)?**
**Answer:** A dedicated message queue that isolates unprocessable or malformed messages after repeated retry failures to prevent consumer queue blocking.

### **48. What is the Bulkhead Pattern?**
**Answer:** Isolating critical system resources (thread pools, memory, database connections) into distinct compartments so failure of one dependency does not crash the entire application.

### **49. What is Circuit Breaking?**
**Answer:** A design pattern that prevents an application from repeatedly executing an operation that is failing, allowing downstream dependencies time to recover.

### **50. What is Rate Limiting vs Throttling?**
**Answer:** Rate limiting restricts how many requests a specific user/IP can make in a given time window; throttling regulates server-side processing speed to match downstream resource capacity.

---

## 🟡 **Part 2: Site Reliability Engineering (SRE) & Reliability (Questions 51–100)**

### **51. What is an SLI (Service Level Indicator)?**
**Answer:** A quantifiable metric that measures service performance at a specific moment in time (e.g., Proportion of HTTP requests returning status 200 in $< 200\text{ms}$).

### **52. What is an SLO (Service Level Objective)?**
**Answer:** An internal reliability target agreed upon between Dev and SRE teams (e.g., 99.9% of requests must meet the SLI over a rolling 30-day window).

### **53. What is an SLA (Service Level Agreement)?**
**Answer:** A legally binding contract with customers specifying penalties (financial credits, refunds) if the service fails to meet agreed uptime levels.

### **54. What is an Error Budget?**
**Answer:** The exact mathematical inverse of an SLO: $\text{Error Budget} = 100\% - \text{SLO}$. It defines the acceptable headroom of downtime or failures a service can accumulate without breaching user trust.

### **55. What happens when an Error Budget is exhausted (0%)?**
**Answer:** Non-critical feature deployments are frozen, and 100% of engineering bandwidth shifts to bug fixing, technical debt reduction, and reliability engineering.

### **56. What is Toil in SRE?**
**Answer:** Operational work that is manual, repetitive, automatable, tactical, devoid of enduring engineering value, and scales linearly with service growth.

### **57. What is the Google SRE 50% Rule for Toil?**
**Answer:** A maximum of 50% of an SRE's time should be spent on operational toil and on-call duties; the remaining 50%+ must be dedicated to software engineering projects that permanently eliminate toil.

### **58. What are the Four Golden Signals of Monitoring?**
**Answer:** Latency (time to process requests), Traffic (demand/throughput), Errors (rate of failed requests), and Saturation (how close resources are to full capacity).

### **59. What is the RED Method?**
**Answer:** A monitoring framework for request-driven microservices: **R**ate (req/sec), **E**rrors (failed req/sec), and **D**uration (latency distribution).

### **60. What is the USE Method?**
**Answer:** A monitoring framework for infrastructure resources: **U**tilization (time resource was busy), **S**aturation (queue depth), and **E**rrors (error counts).

### **61. What is MTTD (Mean Time to Detect)?**
**Answer:** The average time elapsed between the inception of a system incident and when the engineering team or monitoring system detects it.

### **62. What is MTTR (Mean Time to Resolve / Restore)?**
**Answer:** The average time taken from detecting a production failure to completely restoring normal system operations for users.

### **63. What is MTBF (Mean Time Between Failures)?**
**Answer:** The average time elapsed between operational system failures, measuring overall infrastructure stability.

### **64. What is MTTF (Mean Time to Failure)?**
**Answer:** The expected operating time of a non-repairable system or hardware component before it completely fails.

### **65. What is a Blameless Post-Mortem?**
**Answer:** A structured, collaborative retrospective conducted after an incident that assumes engineers acted with good intentions, focusing on systemic, architectural, and procedural root causes.

### **66. What are the core sections of a Post-Mortem document?**
**Answer:** 1. Executive Summary & Impact, 2. UTC Incident Timeline, 3. Root Cause Analysis (5 Whys), 4. What Went Well / Poorly, 5. SMART Action Items with owners.

### **67. What is the 5 Whys technique in root cause analysis?**
**Answer:** An iterative interrogative technique where you ask "Why?" five consecutive times to drill down past surface symptoms to the fundamental organizational or architectural root cause.

### **68. What is Alert Fatigue and how do you prevent it?**
**Answer:** Overwhelming on-call engineers with noisy, non-actionable alerts, leading to missed critical incidents. Prevented by alerting strictly on user-facing SLO symptoms and dynamic burn rates.

### **69. What is a Multi-Window Multi-Burn-Rate Alert?**
**Answer:** Google SRE alerting standard that calculates the consumption speed of an error budget across short (1 hour) and long (6 hours) windows to page engineers only when error budget is rapidly burning.

### **70. What is RTO (Recovery Time Objective)?**
**Answer:** The maximum acceptable duration of system downtime after a disaster before service is restored (measures downtime duration).

### **71. What is RPO (Recovery Point Objective)?**
**Answer:** The maximum acceptable amount of data loss measured in time backward from the point of disaster (measures data loss).

### **72. What is an On-Call Escalation Policy?**
**Answer:** A predefined workflow in paging systems (PagerDuty, Opsgenie) that routes alerts to a primary on-call engineer, escalating to secondary engineers or engineering managers if unacknowledged within 5–10 minutes.

### **73. What is an Incident Commander (IC)?**
**Answer:** The designated leader during a critical production outage responsible for coordinating investigation tasks, assigning roles, managing communications, and making high-level recovery decisions.

### **74. What is a Runbook (Playbook)?**
**Answer:** A documented, step-by-step technical guide for on-call engineers describing how to diagnose, triage, and resolve specific alert conditions or system failures.

### **75. What is Automated Remediation?**
**Answer:** Using software routines (EventBridge, Lambda, Kubernetes Operators) to detect specific operational failures and execute corrective actions (restarting pods, resizing disks) with zero human intervention.

### **76. What is Capacity Planning in SRE?**
**Answer:** Forecasting future resource demands (compute, memory, storage, database IOPS) based on organic user growth and marketing spikes to ensure systems scale proactively before saturation occurs.

### **77. What is Load Shedding?**
**Answer:** Intentionally dropping low-priority background requests when a server approaches 100% CPU/memory saturation to guarantee critical transactions succeed with low latency.

### **78. What is Graceful Degradation?**
**Answer:** Designing applications to remain functional with reduced features when non-essential downstream services fail (e.g., showing static recommendations if the recommendation engine is down).

### **79. What is the Thundering Herd Problem?**
**Answer:** A scenario where thousands of clients retry a failed network request simultaneously at the exact same millisecond, overwhelming recovering backend servers. Prevented by **Exponential Backoff with Jitter**.

### **80. What is Exponential Backoff with Jitter?**
**Answer:** A retry algorithm where wait intervals double with each attempt ($2^n$) combined with randomized variance (jitter) to desynchronize retry storms across clients.

### **81. What is High Availability (HA)?**
**Answer:** Designing systems with redundant components across multiple fault domains (Availability Zones, regions) to ensure continuous operation without a Single Point of Failure (SPOF).

### **82. What does "Three Nines" vs "Four Nines" vs "Five Nines" uptime mean?**
**Answer:** 99.9% (Three Nines) = max 43.2 min downtime/month; 99.99% (Four Nines) = max 4.32 min downtime/month; 99.999% (Five Nines) = max 25.9 sec downtime/month.

### **83. What is Active-Active vs Active-Passive High Availability?**
**Answer:** Active-Active runs multiple redundant systems processing live traffic simultaneously; Active-Passive runs a primary active system while a secondary standby system remains idle, ready to take over on failover.

### **84. What is Split-Brain in distributed systems?**
**Answer:** A state where a network partition isolates nodes in a cluster, causing both sides to believe the other has failed and independently elect leaders, leading to severe data corruption.

### **85. How is Split-Brain prevented?**
**Answer:** By enforcing a strict quorum consensus requirement ($\lfloor N/2 \rfloor + 1$) on leader election and running odd numbers of master nodes (3, 5, 7).

### **86. What is the CAP Theorem?**
**Answer:** In a distributed data store with a Network Partition (**P**), you must choose between Consistency (**C** - all nodes see the same data) or Availability (**A** - every request receives a non-error response).

### **87. What is the PACELC Theorem?**
**Answer:** An extension of CAP: If there is a **P**artition, choose **A**vailability or **C**onsistency; **E**lse (normal state), choose **L**atency or **C**onsistency.

### **88. What is Eventual Consistency?**
**Answer:** A consistency model where, given no new updates, all replicas across a distributed system will eventually converge and return the latest value.

### **89. What is Distributed Tracing?**
**Answer:** Tracking the end-to-end flow of a single request across multiple microservices to measure latency contributions and identify bottleneck spans.

### **90. What is Continuous Profiling?**
**Answer:** Low-overhead, continuous collection of CPU, memory, and thread contention call stacks directly from production workloads using kernel eBPF probes (e.g., Pyroscope).

### **91. What is Blackbox Probing?**
**Answer:** Pinging external endpoints (HTTP GET, TCP SYN, DNS lookup) from outside the infrastructure to verify user-facing service availability.

### **92. What is Whitebox Monitoring?**
**Answer:** Inspecting internal application runtime metrics (JVM heap, thread counts, database connection pool stats) emitted from within the code.

### **93. What is High Cardinality in metrics?**
**Answer:** A metric with a large number of unique label key-value pairs (e.g., tagging Prometheus metrics with `user_id` or `email`), which exhausts time-series database memory.

### **94. What is Synthetic Monitoring?**
**Answer:** Running automated scripts or headless browser bots that periodically execute user journeys (e.g., searching, adding to cart, checking out) to detect failures before real users encounter them.

### **95. What is Real User Monitoring (RUM)?**
**Answer:** Telemetry captured directly within end users' client browsers or mobile apps to measure real-world page load times, network latencies, and JavaScript errors.

### **96. What is a Service Dependency Graph?**
**Answer:** A dynamic visual topology mapping all upstream and downstream network relationships between microservices, databases, and third-party APIs.

### **97. What is Dark Traffic Mirroring (Shadowing)?**
**Answer:** Duplicating live production HTTP requests and sending an asynchronous copy to a new version without returning responses to users, testing performance under real payloads.

### **98. What is an SRE Error Budget Policy?**
**Answer:** A formal business contract defining agreed-upon operational restrictions (deployment freezes, focus on reliability) when an error budget reaches zero.

### **99. What is Chaos Mesh?**
**Answer:** A cloud-native chaos engineering platform for Kubernetes that injects faults (pod failure, network latency, CPU stress, I/O corruption) via Custom Resource Definitions (CRDs).

### **100. What is Observability as Code (OaC)?**
**Answer:** Defining Grafana dashboards, Prometheus alerting rules, and OpenTelemetry collector configs in declarative code files versioned in Git.

---

## 🔴 **Part 3: Platform Engineering, FinOps & Architecture (Questions 101–200)**

### **101. What is Platform Engineering?**
**Answer:** The discipline of designing and building toolchains and self-service workflows (Internal Developer Platforms) that enable software developers to build, test, deploy, and operate software independently.

### **102. What is "Platform as a Product"?**
**Answer:** Treating the internal developer platform as a software product where internal developers are the customers, employing product managers to gather feedback, measure adoption, and iterate on Developer Experience (DevEx).

### **103. What is Spotify Backstage?**
**Answer:** An open-source CNCF framework for building unified developer portals, featuring a centralized Software Catalog, documentation as code (TechDocs), and software templates (Golden Paths).

### **104. What is a Software Template in Platform Engineering?**
**Answer:** A pre-configured boilerplate repository (scaffolding) that provisions a fully compliant microservice with CI/CD pipelines, Dockerfile, Helm charts, monitoring dashboards, and security scans in 1 click.

### **105. What is Developer Experience (DevEx)?**
**Answer:** How friction-free, intuitive, and efficient it is for software engineers to write code, run tests, debug services, and deploy changes to production.

### **106. What is Developer Velocity Index (DVI)?**
**Answer:** A metric framework measuring how effectively an organization's tooling, culture, and practices empower developers to innovate and deliver business value rapidly.

### **107. What is Crossplane?**
**Answer:** An open-source CNCF project that turns Kubernetes into a universal control plane, allowing platform teams to manage cloud resources (S3, RDS) as Kubernetes Custom Resources (CRDs).

### **108. What is a Composite Resource Definition (XRD) in Crossplane?**
**Answer:** A custom Kubernetes API defined by a platform team that bundles multiple cloud resources into a single self-service abstraction for developers (e.g., `kind: DatabaseInstance`).

### **109. What is Kratix?**
**Answer:** A framework for building multi-cloud, multi-cluster platforms on Kubernetes that delivers reusable infrastructure and service "Promises" to development teams.

### **110. What is the Open Feature standard?**
**Answer:** A vendor-agnostic CNCF standard providing unified SDKs and APIs for feature flag management across languages.

### **111. What is the Score specification in Platform Engineering?**
**Answer:** A developer-centric, workload-oriented specification that describes how an application runs without needing to know target environment Kubernetes or Terraform details.

### **112. What is FinOps and what are its three core phases?**
**Answer:** Cloud Financial Operations. The three phases are: **Inform** (visibility and tagging), **Optimize** (rate reduction and right-sizing), and **Operate** (continuous governance).

### **113. What is Cloud Unit Economics?**
**Answer:** Measuring cloud spend relative to direct business metrics (e.g., *Cost per Active User*, *Cost per Transaction Processed*, *Cost per GB Streamed*).

### **114. What is a Cloud Savings Plan?**
**Answer:** A flexible pricing model offering up to 72% discount on compute usage (EC2, Fargate, Lambda) in exchange for a 1- or 3-year hourly spend commitment.

### **115. What are Reserved Instances (RIs)?**
**Answer:** A discount commitment model tied to specific cloud instance families, OS, and regions in exchange for 1- or 3-year term commitments.

### **116. What are Cloud Spot Instances?**
**Answer:** Spare cloud provider compute capacity offered at up to 90% discount, with the caveat that the provider can reclaim the instance with a 2-minute notification.

### **117. What is Infracost?**
**Answer:** A FinOps tool that parses Terraform code in CI pull requests to display the exact monthly cost delta of proposed infrastructure changes before deployment.

### **118. What is Cloud Custodian?**
**Answer:** An open-source rules engine for cloud security and cost management that automatically stops untagged instances, terminates idle databases, and enforces compliance via YAML policies.

### **119. What is Kubernetes Right-Sizing?**
**Answer:** Adjusting container CPU/memory `requests` and `limits` based on historical usage telemetry to eliminate over-provisioning and minimize cloud waste.

### **120. What is Scale-to-Zero in serverless and Kubernetes?**
**Answer:** Automatically scaling application pod replicas down to 0 when there is no incoming traffic or queue backlog (via KEDA or Knative), saving 100% of compute cost during idle periods.

### **121. What is Karpenter Node Consolidation?**
**Answer:** An automated optimization feature where Karpenter identifies underutilized worker nodes, computes an optimal single replacement node, drains pods, and terminates empty nodes.

### **122. What is a Multi-Tenant Kubernetes Cluster?**
**Answer:** A single physical cluster shared across multiple development teams or external customers, isolated via Namespaces, NetworkPolicies, RBAC, and ResourceQuotas.

### **123. What is `vcluster` (Virtual Cluster)?**
**Answer:** An open-source tool that runs fully functional, isolated virtual Kubernetes control planes inside namespaces of an underlying host cluster, providing hard multi-tenancy.

### **124. What is Zero Trust Network Architecture (ZTNA)?**
**Answer:** A security model requiring strict identity verification and encrypted communication (mTLS) for every user, device, and service regardless of whether it resides inside the private VPC.

### **125. What is SPIFFE/SPIRE?**
**Answer:** Standardized cryptographic workload identity framework that issues short-lived, auto-rotating X.509 certificates to microservices based on kernel and kubelet attestation.

### **126. What is Policy as Code (PaC)?**
**Answer:** Defining, testing, and enforcing compliance and operational rules using machine-readable code files (Open Policy Agent Rego, Kyverno) across CI/CD and admission controllers.

### **127. What is Open Policy Agent (OPA)?**
**Answer:** An open-source, general-purpose policy engine that evaluates structured JSON/YAML inputs against declarative logic written in the **Rego** language.

### **128. What is Kyverno?**
**Answer:** A Kubernetes-native policy engine written 100% in YAML that validates, mutates, and generates Kubernetes resources and verifies container image signatures.

### **129. What is HashiCorp Vault?**
**Answer:** A secure secrets management system that provides centralized dynamic secrets generation, PKI certificate authority management, and data encryption as a service.

### **130. What is External Secrets Operator (ESO)?**
**Answer:** A Kubernetes operator that syncs secrets from external enterprise vaults (AWS Secrets Manager, HashiCorp Vault) into native Kubernetes `Secret` objects.

### **131. What is Keyless Signing with Sigstore Cosign?**
**Answer:** Cryptographically signing container images using short-lived OIDC certificates from Fulcio and transparency logs from Rekor, eliminating static PGP private keys.

### **132. What is an SBOM (Software Bill of Materials)?**
**Answer:** A machine-readable nested inventory of all software packages, libraries, and transitive dependencies bundled inside a software container or binary (e.g., CycloneDX).

### **133. What is the SLSA Framework (Supply-chain Levels for Software Artifacts)?**
**Answer:** A security framework establishing standards for software build integrity, hermetic build sandboxing, and non-falsifiable build provenance.

### **134. What is eBPF (Extended Berkeley Packet Filter)?**
**Answer:** A Linux kernel technology allowing sandboxed programs to execute safely inside the kernel without changing kernel source code, used for high-speed networking (Cilium), observability (Hubble), and runtime security (Falco).

### **135. What is Falco?**
**Answer:** A CNCF runtime security tool that parses Linux kernel system calls via eBPF probes to detect and alert on unauthorized container activities (spawning shells, reading `/etc/shadow`).

### **136. What is Cilium?**
**Answer:** An eBPF-based Kubernetes CNI, Service Mesh, and network security plugin providing high-performance Layer 3 to Layer 7 routing and transparent encryption.

### **137. What is the Gateway API in Kubernetes?**
**Answer:** The modern successor to Kubernetes Ingress providing role-oriented resource separation (`GatewayClass`, `Gateway`, `HTTPRoute`) and native traffic splitting.

### **138. What is KEDA (Kubernetes Event-driven Autoscaling)?**
**Answer:** An autoscaler that scales Kubernetes workloads from 0 to hundreds based on external event metrics (AWS SQS queue length, Kafka consumer group lag).

### **139. What is Istio Ambient Mesh?**
**Answer:** A sidecarless service mesh architecture that splits mesh processing into per-node `ztunnel` daemons (Layer 4 mTLS) and optional per-namespace waypoint proxies (Layer 7 routing).

### **140. What is Distributed Consensus (Raft / Paxos)?**
**Answer:** Algorithms that enable distributed clusters (etcd, Consul, CockroachDB) to agree on data values and state machine logs across independent nodes despite network partitions.

### **141. What is Write-Ahead Logging (WAL)?**
**Answer:** A database reliability technique where changes are recorded to append-only disk storage before being applied to memory, guaranteeing durability during crashes.

### **142. What is Database Connection Pooling?**
**Answer:** Reusing a fixed pool of established database connections (via PgBouncer / RDS Proxy) to prevent thousands of client requests from exhausting database memory and thread limits.

### **143. What is Sharding in databases?**
**Answer:** Horizontally partitioning database rows across multiple independent physical database instances based on a shard key (e.g., `user_id` % 10).

### **144. What is Read-Write Splitting?**
**Answer:** Routing write operations (`INSERT`, `UPDATE`) to the primary database while directing read queries (`SELECT`) to asynchronous read replicas.

### **145. What is Database Replication Lag?**
**Answer:** The delay in time between a transaction being committed on a primary database and that transaction being applied on asynchronous read replicas.

### **146. What is Dual-Write Pattern in microservices?**
**Answer:** The anti-pattern of an application attempting to write to both a database and a message queue in a single transaction, leading to data inconsistency when one write fails.

### **147. What is the Transactional Outbox Pattern?**
**Answer:** Saving events to an "Outbox" database table within the same ACID transaction as business data, and using a separate change-data-capture (Debezium) process to publish events to Kafka.

### **148. What is the Saga Pattern in distributed transactions?**
**Answer:** Managing distributed transactions across microservices via a sequence of local transactions, executing compensating transactions (reversals) if a step fails.

### **149. What is Event Sourcing?**
**Answer:** An architectural pattern where state changes are stored as an append-only sequence of immutable events rather than overwriting current state values in place.

### **150. What is CQRS (Command Query Responsibility Segregation)?**
**Answer:** Separating read and write operations into distinct data models and databases optimized specifically for high-throughput writes (Commands) or fast reads (Queries).

### **151. What is an Anycast IP Address?**
**Answer:** Routing client traffic to the topologically closest server among multiple globally distributed servers sharing the exact same IP address via BGP.

### **152. What is a Content Delivery Network (CDN)?**
**Answer:** A globally distributed network of proxy caching servers that caches web assets close to users to minimize latency and absorb traffic surges.

### **153. What is Head-of-Line (HoL) Blocking?**
**Answer:** A performance bottleneck where a single dropped or delayed packet in a FIFO queue blocks all subsequent packets from processing (resolved by HTTP/3 QUIC over UDP).

### **154. What is AWS Direct Connect / Azure ExpressRoute?**
**Answer:** Dedicated private physical fiber connections between an enterprise on-premise data center and a cloud provider, bypassing public internet routing.

### **155. What is AWS Transit Gateway?**
**Answer:** A centralized hub-and-spoke router connecting hundreds of VPCs, VPNs, and Direct Connect links through a single managed gateway.

### **156. What is AWS PrivateLink (VPC Endpoint)?**
**Answer:** Private network interfaces connecting VPC subnets to cloud services or SaaS vendors over cloud backbone networks without public internet transit.

### **157. What is Multi-Region Active-Active Architecture?**
**Answer:** Deploying fully functional infrastructure across multiple geographical cloud regions simultaneously, routing live users to the closest region with real-time data replication.

### **158. What is Disaster Recovery Failover Routing?**
**Answer:** DNS routing policies (Route 53) that automatically redirect global user traffic to a backup disaster recovery region when primary health check probes fail.

### **159. What is Pilot Light Disaster Recovery?**
**Answer:** Replicating critical databases continuously to a secondary region while keeping minimal core infrastructure running, provisioning full compute only during a disaster.

### **160. What is Warm Standby Disaster Recovery?**
**Answer:** Running a scaled-down, fully functional copy of production infrastructure in a secondary region that can be scaled up instantly upon failover.

### **161. What is Cold Start in Serverless?**
**Answer:** The latency penalty incurred when an incoming request requires the cloud provider to download the container image, provision a microVM, and initialize the language runtime.

### **162. What is AWS Lambda SnapStart?**
**Answer:** Caching an encrypted Firecracker microVM snapshot of an initialized JVM runtime during deployment, resuming subsequent cold starts in $< 200\text{ms}$.

### **163. What is an API Gateway?**
**Answer:** An application-layer reverse proxy that manages incoming API traffic, handling authentication, rate limiting, request validation, and analytics.

### **164. What is gRPC vs REST?**
**Answer:** REST uses HTTP/1.1 with JSON text payloads; gRPC uses HTTP/2 with binary Protocol Buffers (Protobuf), providing multiplexing, bi-directional streaming, and high throughput.

### **165. What is GraphQL vs REST?**
**Answer:** REST exposes fixed data endpoints; GraphQL allows clients to request the exact fields they need in a single request, eliminating over-fetching and under-fetching.

### **166. What is Webhook Payload Verification (HMAC-SHA256)?**
**Answer:** Validating that an incoming webhook originated from an authentic sender by comparing the hash of the request body with a shared secret key.

### **167. What is JWT (JSON Web Token) Structure?**
**Answer:** Encoded string with three parts separated by dots: **Header** (algorithm), **Payload** (claims: `sub`, `exp`), and **Signature** (cryptographic hash).

### **168. What is OAuth 2.0 vs OpenID Connect (OIDC)?**
**Answer:** OAuth 2.0 is an authorization framework issuing Access Tokens for API permissions; OIDC is an identity layer on top of OAuth 2.0 issuing ID Tokens for user authentication.

### **169. What is Role-Based Access Control (RBAC)?**
**Answer:** Assigning system permissions to roles rather than individual users, granting users permissions based on role memberships.

### **170. What is Attribute-Based Access Control (ABAC)?**
**Answer:** Evaluating context attributes (user department, resource tags, time of day, IP address) to determine real-time access authorization.

### **171. What is CloudTrail vs CloudWatch?**
**Answer:** CloudTrail records every API call made in an AWS account for auditing; CloudWatch monitors operational performance metrics and aggregates logs.

### **172. What is Amazon GuardDuty?**
**Answer:** Intelligent threat detection analyzing CloudTrail, VPC Flow Logs, and EKS audit logs via machine learning to detect compromised instances.

### **173. What is AWS Security Hub?**
**Answer:** A centralized dashboard aggregating compliance posture (CIS, PCI-DSS) and security findings across AWS accounts.

### **174. What is Amazon Inspector?**
**Answer:** Automated vulnerability management scanning EC2 instances, ECR container images, and Lambda functions for software vulnerabilities.

### **175. What is AWS KMS (Key Management Service)?**
**Answer:** Managed service for creating and controlling cryptographic encryption keys used across cloud storage and applications.

### **176. What is Envelope Encryption?**
**Answer:** Encrypting high-volume plaintext data with a Data Encryption Key (DEK), and encrypting the DEK with a root KMS Key.

### **177. What is AWS Secrets Manager vs Parameter Store?**
**Answer:** Secrets Manager supports automated secret rotation and cross-account access; Parameter Store is a simpler, hierarchical key-value configuration store.

### **178. What is Infrastructure Drift Detection?**
**Answer:** Automatically comparing running cloud resources against IaC state files to detect and alert on unauthorized manual modifications.

### **179. What is Terratest?**
**Answer:** A Go library for writing automated integration tests that provision real cloud infrastructure via Terraform, validate endpoints, and tear down resources.

### **180. What is Checkov?**
**Answer:** A static code analysis tool for Infrastructure as Code that scans Terraform, CloudFormation, and Kubernetes manifests for security misconfigurations.

### **181. What is OpenTofu?**
**Answer:** An open-source, community-governed fork of Terraform (MPL 2.0) maintained by the Linux Foundation.

### **182. What is Prometheus PromQL?**
**Answer:** A powerful read-only query language for filtering, aggregating, and computing mathematical functions over Prometheus time-series metrics.

### **183. What is Grafana Loki LogQL?**
**Answer:** A Prometheus-inspired query language for filtering log streams and generating real-time metrics from raw log text.

### **184. What is Grafana Tempo?**
**Answer:** A distributed tracing backend that stores raw spans directly in cheap cloud object storage without needing Elasticsearch.

### **185. What is OpenTelemetry (OTel) Collector?**
**Answer:** A proxy pipeline component that receives, processes (batches, redacts, samples), and exports telemetry to multiple backend storage engines.

### **186. What is eBPF Hubble?**
**Answer:** An observability platform running on eBPF to visualize service-to-service communication dependency graphs and network latency.

### **187. What is Chaos Mesh?**
**Answer:** A Kubernetes-native chaos engineering operator that injects faults via CRDs to test system resiliency.

### **188. What is DAST (Dynamic Application Security Testing)?**
**Answer:** Attacking running web applications from the outside to discover runtime misconfigurations, XSS, and SQL injection flaws.

### **189. What is SAST (Static Application Security Testing)?**
**Answer:** Analyzing uncompiled source code to detect security vulnerabilities, syntax errors, and coding flaws.

### **190. What is SCA (Software Composition Analysis)?**
**Answer:** Scanning open-source third-party dependencies against national vulnerability databases for known CVEs.

### **191. What is a Container Escape?**
**Answer:** An exploit where an attacker running code inside a container escapes isolation boundaries to gain control over the host operating system.

### **192. What is Rootless Docker / Podman?**
**Answer:** Running container daemons and processes inside unprivileged user namespaces, preventing host root compromise during breakouts.

### **193. What is Pod Security Admission (PSA)?**
**Answer:** Kubernetes built-in admission controller enforcing Privileged, Baseline, or Restricted security standards per namespace.

### **194. What is a Kubernetes Pod Disruption Budget (PDB)?**
**Answer:** A policy limiting how many pods of a replicated service can be down simultaneously during voluntary disruptions (node drains).

### **195. What is Kubernetes Horizontal Pod Autoscaler (HPA)?**
**Answer:** Automatically adjusting the number of pod replicas based on observed CPU, memory, or custom external metrics.

### **196. What is Kubernetes Vertical Pod Autoscaler (VPA)?**
**Answer:** Automatically right-sizing container CPU and memory resource requests/limits based on historical usage analysis.

### **197. What is NodeLocal DNSCache?**
**Answer:** A DaemonSet caching DNS queries locally on worker nodes to eliminate CoreDNS network bottlenecks and glibc `ndots:5` latency storms.

### **198. What is GitOps Reconciliation Loop?**
**Answer:** A continuous background process where a GitOps operator compares desired state in Git against live cluster state and automatically auto-heals drift.

### **199. What is Progressive Delivery?**
**Answer:** An evolution of Continuous Delivery that combines canary deployments, automated metric analysis, and feature flags to roll out features with minimal blast radius.

### **200. What is an SRE Blameless Post-Mortem Creed?**
**Answer:** The guiding philosophy that human error is the starting point of an incident investigation, not the conclusion, focusing on improving systemic safeguards.
