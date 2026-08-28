# **CI/CD & GitOps - DevOps Interview Questions**

Welcome to the **CI/CD & GitOps** interview questions master guide. This module provides in-depth, exhaustive technical explanations, production-grade YAML configurations, security architectures, and scenario-based interview discussions covering Continuous Integration, Continuous Delivery, GitOps (ArgoCD/Flux), progressive delivery, and software supply chain security.

---

## 🟢 **Beginner Level (Questions 1–20)**

### **1. What is CI/CD, what is the fundamental difference between Continuous Delivery and Continuous Deployment, and why is this distinction critical for enterprise risk management?**

**Detailed Answer:**
**CI/CD** represents the automated backbone of modern software engineering, comprising Continuous Integration (CI) and Continuous Delivery/Deployment (CD).

```
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                                CONTINUOUS INTEGRATION (CI)                                  │
│  Commit ➔ Linting & SAST ➔ Automated Build ➔ Unit & Integration Tests ➔ Artifact Storage    │
└──────────────────────────────────────────────┬──────────────────────────────────────────────┘
                                               ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                                CONTINUOUS DELIVERY (CD)                                     │
│  Automated Deploy to Staging ➔ End-to-End Testing ➔ Ready for Prod ➔ [ MANUAL APPROVAL ]   │
└──────────────────────────────────────────────┬──────────────────────────────────────────────┘
                                               ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                                CONTINUOUS DEPLOYMENT (CD)                                   │
│  Fully Automated Production Deployment (Zero Human Intervention, Continuous Canary/Rollout) │
└─────────────────────────────────────────────────────────────────────────────────────────────┘
```

#### **1. Continuous Integration (CI):**
- Developers commit small, incremental changes to a shared mainline repository (`main`) frequently (multiple times per day).
- Every commit triggers an automated pipeline executing static analysis, security linters, unit tests, and packaging.
- **Goal:** Fast feedback loop ($< 10$ minutes) to detect integration bugs and broken dependencies immediately.

#### **2. Continuous Delivery (CD):**
- Automatically deploys tested code to staging and pre-production environments.
- Ensures the software artifact is **always in a deployable, releasable state**.
- The actual trigger to push the release to live production requires an explicit **manual business or operational approval** (e.g., a Release Manager clicking "Approve" in GitHub Actions or Jira).
- **Enterprise Use Case:** Heavily regulated industries (banking, healthcare, aerospace) requiring formal change-advisory board (CAB) reviews and compliance auditing.

#### **3. Continuous Deployment (CD):**
- Completely eliminates manual human gates.
- Every commit that passes the automated CI/CD pipeline is deployed automatically into production environments.
- Relies on automated canary analysis, synthetic monitoring, and instant automated rollback triggers.
- **Enterprise Use Case:** SaaS platforms, consumer applications, and high-velocity digital product teams.

---

### **2. What are the key stages of an enterprise-grade production CI/CD pipeline? Walk through each phase and its security tooling.**

**Detailed Answer:**

```
 ┌──────────────────────────────────────────────────────────────────────────────────────────┐
 │                               ENTERPRISE CI/CD PIPELINE                                  │
 ├─────────────┬─────────────┬─────────────┬─────────────┬─────────────┬────────────────────┤
 │ 1. Source   │ 2. Lint &   │ 3. Build &  │ 4. Package  │ 5. Staging  │ 6. Production      │
 │    Trigger  │    Security │    Test     │    & Sign   │    Deploy   │    Rollout         │
 ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┼────────────────────┤
 │ Webhook on  │ • Gitleaks  │ • Unit test │ • Dockerfile│ • Ephemeral │ • ArgoCD GitOps    │
 │ Git PR /    │ • SonarQube │ • Compila-  │ • Syft SBOM │   Preview   │ • Argo Rollouts    │
 │ push        │ • Semgrep   │   tion      │ • Cosign    │ • DAST test │ • Prometheus SLO   │
 │             │ • Trivy SCA │ • Coverage  │   signing   │ • E2E tests │   analysis         │
 └─────────────┴─────────────┴─────────────┴─────────────┴─────────────┴────────────────────┘
```

#### **Detailed Stage Breakdown:**
1. **Source / Trigger:** Webhook triggers pipeline upon pull request creation or merge to `main`.
2. **Pre-Build Static Analysis & Linting:**
   - *Secret Scanning:* **Gitleaks** blocks committed AWS tokens or SSH keys.
   - *SAST (Static Analysis):* **SonarQube** / **Semgrep** scans raw source code for SQLi, XSS, and memory leaks.
   - *SCA (Software Composition Analysis):* **Trivy** / **Snyk** checks third-party dependencies against CVE databases.
3. **Build & Automated Testing:**
   - Compiles code in parallel across matrix builds.
   - Runs unit tests and integration tests with code coverage reporting (e.g., Codecov).
4. **Packaging, SBOM & Cryptographic Signing:**
   - Multi-stage Docker build produces a hardened distroless container image.
   - **Syft** generates a CycloneDX Software Bill of Materials (SBOM).
   - **Sigstore Cosign** cryptographically signs the container image using keyless OIDC tokens.
5. **Staging & Dynamic Testing:**
   - Deploys ephemeral environment. Runs DAST (Dynamic Application Security Testing) via **OWASP ZAP** and automated Playwright E2E browser tests.
6. **Production Rollout & Telemetry Validation:**
   - GitOps operator (ArgoCD) syncs desired state; Argo Rollouts executes automated Canary traffic shifting with Prometheus metric verification.

---

### **3. What is an Artifact in CI/CD, why must it be strictly immutable, and what is the "Build Once, Deploy Anywhere" principle?**

**Detailed Answer:**
An **Artifact** is the compiled, packaged, deployable binary or package generated during the CI build stage (e.g., Docker container image, JAR file, NPM package, Helm chart).

#### **The "Build Once, Deploy Anywhere" Principle:**
A single immutable artifact must be compiled once in the CI pipeline and promoted sequentially through Development $\rightarrow$ Staging $\rightarrow$ Production without ever recompiling the source code between environments.

```
                  BUILD ONCE, DEPLOY ANYWHERE
┌─────────────────────────────────────────────────────────────┐
│  CI Pipeline: Compiles code & builds container image        │
│  Output: ghcr.io/my-org/payment-service:v1.4.2-abc1234      │
└──────────────────────────────┬──────────────────────────────┘
                               │
            ┌──────────────────┼──────────────────┐
            ▼                  ▼                  ▼
┌──────────────────────┐ ┌──────────────────────┐ ┌──────────────────────┐
│   DEV ENVIRONMENT    │ │  STAGING ENVIRONMENT │ │   PROD ENVIRONMENT   │
│ Injects Dev Config   │ │ Injects Staging Config│ │ Injects Prod Config  │
│ (Same Image Tag)     │ │ (Same Image Tag)     │ │ (Same Image Tag)     │
└──────────────────────┘ └──────────────────────┘ └──────────────────────┘
```

#### **Why Immutability is Mandatory:**
- **Eliminates Environmental Discrepancies:** If code is recompiled in staging and recompiled again in production, differences in compiler versions, upstream dependency patches, or build machine states introduce subtle, untracked bugs.
- **Cryptographic Traceability:** The container image deployed to production matches the exact cryptographic hash (digest) that passed automated security and load testing in staging.
- **Deterministic Rollbacks:** Rolling back to a previous release guarantees deploying the exact historical binary that was previously proven stable.

---

### **4. Compare Reusable Workflows vs Composite Actions in GitHub Actions with practical use cases.**

**Detailed Answer:**

#### **1. Reusable Workflows (`workflow_call`):**
- Modular workflow files located in `.github/workflows/` that can be invoked from other repositories across an entire organization.
- Can contain **multiple independent jobs**, configure matrix strategies, run across multiple runner environments, and define required inputs and secrets.
- **Primary Use Case:** Enforcing standardized organization-wide compliance pipelines (e.g., a mandatory security scan and deployment workflow that all 50 microservice repos must call).

```yaml
# Caller Workflow (.github/workflows/main.yml)
name: Main Pipeline
on: [push]
jobs:
  call-security-pipeline:
    uses: my-org/shared-workflows/.github/workflows/standard-ci.yml@v2
    with:
      environment: production
    secrets: inherit
```

#### **2. Composite Actions (`action.yml`):**
- Packages multiple shell commands and action steps into a **single step** within a single job.
- Cannot define multiple jobs or runner matrices; acts as a custom action step.
- **Primary Use Case:** Eliminating boilerplate repetitive steps within a single job (e.g., a composite action that installs Node.js, configures caching, and runs `npm ci`).

---

### **5. What is GitOps and how does the Pull-Based GitOps model fundamentally improve security compared to Traditional Push-Based CI/CD?**

**Detailed Answer:**

```
                     TRADITIONAL PUSH-BASED CI/CD (High Attack Surface)
┌──────────────────────┐                            ┌───────────────────────────────────┐
│   CI Server          │ ──(Holds Prod Kubeconfig)─►│    Production Kubernetes Cluster  │
│ (Jenkins / GitHub)   │                            │  (Must expose API port to world)  │
└──────────────────────┘                            └───────────────────────────────────┘
 * If CI is breached, attacker gains full admin access to the production cluster!

──────────────────────────────────────────────────────────────────────────────────────────

                         PULL-BASED GITOPS (Zero External Access)
┌──────────────────────┐                            ┌───────────────────────────────────┐
│    Git Repository    │ ◄──(Pulls Desired State)── │    Production Kubernetes Cluster  │
│ (Source of Truth)    │                            │    [ ArgoCD / Flux Inside ]       │
└──────────────────────┘                            │  (Cluster API is 100% Private)    │
                                                    └───────────────────────────────────┘
 * No production cluster credentials ever leave the cluster or sit in CI systems!
```

#### **Core Security & Architectural Advantages of Pull-Based GitOps:**
1. **Zero External Cluster Access:** The Kubernetes API server remains in a private subnet with zero inbound internet ports open.
2. **Credential Isolation:** CI systems (GitHub Actions) only hold permissions to push container images to an OCI registry and commit YAML updates to a Git repository. No `kubeconfig` or cluster admin tokens exist in CI.
3. **Continuous Drift Detection & Self-Healing:** If an attacker or engineer manually modifies a production deployment via `kubectl`, the GitOps operator immediately detects the drift and overwrites the live state back to the declared Git state.

---

### **6. What is Semantic Versioning (SemVer) and how is it automated via Conventional Commits in CI/CD?**

**Detailed Answer:**
**Semantic Versioning (SemVer)** formats release versions as `MAJOR.MINOR.PATCH` (e.g., `2.4.1`):
- **MAJOR:** Breaking API changes (e.g., removing a REST endpoint).
- **MINOR:** Backward-compatible new features.
- **PATCH:** Backward-compatible bug fixes.

#### **Automated Release Workflow (Conventional Commits):**
Tools like **Semantic Release** or **Release Please** parse commit messages in CI to automatically calculate version bumps and publish changelogs:
- `fix: resolve database connection timeout` $\rightarrow$ Triggers **PATCH** bump (`1.2.0` $\rightarrow$ `1.2.1`).
- `feat: add Google SSO login endpoint` $\rightarrow$ Triggers **MINOR** bump (`1.2.0` $\rightarrow$ `1.3.0`).
- `feat!: change authentication to OAuth2 (BREAKING CHANGE)` $\rightarrow$ Triggers **MAJOR** bump (`1.2.0` $\rightarrow$ `2.0.0`).

---

### **7. What is Matrix Strategy in CI/CD pipelines and how does it optimize multi-platform testing?**

**Detailed Answer:**
A **Matrix Strategy** generates a multi-dimensional array of parallel jobs from a single job configuration.

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false  # Continue other jobs even if one fails
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        node-version: [18.x, 20.x, 22.x]
        include:
          - os: ubuntu-latest
            node-version: 22.x
            experimental: true
    steps:
      - uses: actions/checkout@v4
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```
**Advantage:** Automatically runs $3 \times 3 = 9$ parallel test suites concurrently across platforms, reducing test execution time from 45 minutes to 5 minutes.

---

### **8. What is Caching in CI/CD vs Artifact Storage? What should and should not be cached?**

**Detailed Answer:**
- **Caching (`actions/cache`):** Temporary storage for intermediate dependencies (e.g., `~/.npm`, `~/.m2/repository`, Python wheels, Docker build layers) designed solely to accelerate pipeline build speeds.
  - *Lifecycle:* Ephemeral; if a cache expires or is evicted, the build still succeeds by downloading packages freshly.
- **Artifact Storage (`actions/upload-artifact`):** Long-term persistent storage for compiled binaries, test reports, and compliance logs passed between jobs or retained for auditing.

#### **Rules for What NOT to Cache:**
- **Never cache secrets or dynamic environment files** (`.env`, private keys).
- **Never cache output binaries that should be freshly compiled.**
- **Never cache lockfiles** (`package-lock.json`); lockfiles must be checked into Git to drive cache keys (`${{ runner.os }}-build-${{ hashFiles('**/package-lock.json') }}`).

---

### **9. Explain Blue-Green Deployment in Kubernetes with manifest examples.**

**Detailed Answer:**
In Kubernetes, Blue-Green deployment runs two distinct Deployments (`payment-blue` and `payment-green`) behind a single `Service`.

```yaml
# Step 1: Kubernetes Service routing 100% traffic to Blue
apiVersion: v1
kind: Service
metadata:
  name: payment-service
spec:
  selector:
    app: payment
    version: blue  # Target version
  ports:
    - port: 80
      targetPort: 8080
```
#### **Execution Sequence:**
1. Deploy `payment-green` running the new code version.
2. Run internal smoke tests against Green via a private test Service.
3. Update the production Service selector from `version: blue` to `version: green`.
4. **Instant Switch:** Kube-proxy / Cilium updates endpoint routing within milliseconds.
5. If errors occur, revert selector back to `version: blue` immediately.

---

### **10. Explain Canary Deployment with progressive traffic shifting and automated metric verification.**

**Detailed Answer:**
Canary deployment routes a small percentage of production traffic to the new software version to validate stability against real customer workloads.
- **Progression:** 5% traffic $\rightarrow$ evaluate 10 minutes $\rightarrow$ 25% traffic $\rightarrow$ evaluate 10 minutes $\rightarrow$ 100% traffic.
- **Automated Verification:** Metrics evaluated against baseline:
  - Error rate must not exceed baseline by $> 0.5\%$.
  - p99 latency must not increase by $> 10\%$.

---

### **11. Compare Self-Hosted Runners vs Cloud-Hosted Runners in CI/CD.**

**Detailed Answer:**
| Feature | Cloud-Hosted Runners (GitHub/GitLab) | Self-Hosted Runners (ARC on K8s) |
| :--- | :--- | :--- |
| **Management Overhead** | Zero (fully managed by provider) | Requires managing scaling, OS patches, and security isolation |
| **Network Access** | Cannot access private VPC resources without complex VPNs | Native private VPC access to internal databases, registries, and staging clusters |
| **Hardware Customization** | Fixed CPU/RAM tiers | Unlimited (GPUs, ARM64 Graviton, NVMe SSDs, high memory) |
| **Cost at High Volume** | Expensive per-minute billing | Substantially cheaper using ephemeral Spot instances via Karpenter |
| **Security Isolation** | Clean VM per job | Ephemeral Kubernetes pods destroyed after each job |

---

### **12. What is OIDC (OpenID Connect) authentication in CI/CD and why must static cloud credentials be eliminated?**

**Detailed Answer:**
Static cloud credentials (e.g., `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` stored in GitHub repository secrets) represent a severe security risk:
- Vulnerable to exfiltration if repository permissions are misconfigured.
- Require manual rotation every 90 days.
- Hard to enforce fine-grained, short-lived least-privilege scoping.

```
                      OIDC CLOUD AUTHENTICATION WORKFLOW
┌──────────────────────┐                                 ┌──────────────────────┐
│ GitHub Actions Runner│                                 │ AWS STS / Identity   │
└──────────┬───────────┘                                 └──────────┬───────────┘
           │ 1. Requests signed JWT OIDC Token                      │
           │    (Claims: repo, branch, commit SHA)                  │
           ▼                                                        │
┌──────────────────────┐                                            │
│  GitHub OIDC Provider│ ──(Issues signed JWT)─────────────────────►│
└──────────────────────┘                                            │
                                                                    │ 2. Validates JWT Signature
                                                                    │    & checks IAM Trust Policy
                                                                    ▼
                                                         ┌──────────────────────┐
                                                         │ Temporary IAM Creds  │
                                                         │ (Valid for 1 hour)   │
                                                         └──────────┬───────────┘
                                                                    │
           ◄──(Returns temporary scoped AWS credentials)────────────┘
```

---

### **13. What is a Linter and why must it run in the earliest pipeline stage?**

**Detailed Answer:**
A **linter** performs static code analysis to enforce syntax rules, coding conventions, type safety, and detect anti-patterns without compiling or executing the code.
- **Why First:** Linters execute in under 30 seconds and consume minimal compute resources. Failing early prevents wasting expensive multi-minute build and integration test runner minutes on malformed code.

---

### **14. What are Ephemeral / Preview Environments and how are they managed in CI/CD?**

**Detailed Answer:**
Ephemeral environments are dynamic, isolated environments spun up automatically when a Pull Request is opened and destroyed when the PR is closed.
- **How they work:** CI builds the PR container, creates a dedicated Kubernetes namespace (`preview-pr-42`), deploys the service via Helm, configures dynamic DNS (`https://pr-42.preview.company.com`), and posts the URL on the PR.
- **Benefit:** Enables product managers, QA engineers, and security reviewers to test live functionality before merging to `main`.

---

### **15. What is a SonarQube Quality Gate and how does it block risky code merges?**

**Detailed Answer:**
A **Quality Gate** is a policy enforcing threshold conditions that a project must pass before it can be merged:
- *Standard Conditions:* Code coverage on new code $\ge 80\%$, 0 New Critical/Blocker Vulnerabilities, 0 Security Hotspots, Duplicated Lines on new code $< 3\%$.
- If any condition fails, SonarQube sends a failure status check to GitHub, blocking the PR merge button.

---

### **16. What is Secret Masking in CI/CD logs and what are its limitations?**

**Detailed Answer:**
Secret masking automatically intercepts pipeline stdout/stderr output and replaces known secret strings with `***`.
- **Limitations:**
  - Masking only detects exact string matches.
  - If a secret is base64-encoded, URL-encoded, or split across lines, the CI engine will not recognize it and may print it in plain text.
  - *Best Practice:* Never print environment variables or debug dumps in production pipelines.

---

### **17. What is a Webhook in CI/CD and how is payload security verified?**

**Detailed Answer:**
A **webhook** is an automated HTTP POST payload sent from a source (GitHub) to a target (CI server / ArgoCD) upon events like `git push`.
- **Payload Verification:** The webhook provider hashes the request body with a shared secret key using HMAC-SHA256 and includes it in the `X-Hub-Signature-256` header. The receiver calculates the same hash; if hashes match, the payload is authentic and untampered.

---

### **18. Compare Declarative Jenkinsfile vs Scripted Jenkinsfile.**

**Detailed Answer:**
- **Declarative Pipeline (`pipeline { agent any; stages { ... } }`):**
  - Strict, structured, syntax-checked format.
  - Built-in error handling, post-actions (`always`, `success`, `failure`), and easier readability.
- **Scripted Pipeline (`node { ... }`):**
  - Groovy-based imperative programming.
  - Unlimited programmatic flexibility (loops, dynamic methods), but difficult to maintain and test.

---

### **19. How do you optimize Monorepo CI pipelines to avoid rebuilding 100 microservices on every commit?**

**Detailed Answer:**
1. **Path Filtering:** Configure CI to trigger jobs only when files in specific directories change (e.g., `paths: ['services/payment/**']`).
2. **Build Systems with Change Dependency Graphs:** Tools like **Turborepo**, **Nx**, or **Bazel** calculate dependency DAGs. If `libs/common` changes, only dependent services are rebuilt; unchanged microservices reuse cached remote build artifacts.

---

### **20. What is Dark Launching with Feature Flags in CI/CD?**

**Detailed Answer:**
Dark launching is deploying code to production completely silently (behind a feature flag disabled for public users). It allows validating database migrations, cache warming, and backend performance under live production load without exposing UI elements to users.

---

## 🟡 **Intermediate Level (Questions 21–40)**

### **21. Provide a complete, production-grade GitHub Actions Workflow deploying to AWS EKS using OIDC without static keys.**

**Detailed Answer:**

```yaml
name: Deploy to Production EKS
on:
  push:
    branches: [main]

permissions:
  id-token: write  # Mandatory for OIDC JWT token generation
  contents: read

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Configure AWS Credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsEKSRole
          aws-region: us-east-1
          audience: sts.amazonaws.com

      - name: Log in to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and Push Docker Image
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          REPOSITORY: payment-service
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $REGISTRY/$REPOSITORY:$IMAGE_TAG .
          docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG

      - name: Update Kubeconfig
        run: |
          aws eks update-kubeconfig --region us-east-1 --name production-cluster

      - name: Deploy to Kubernetes
        env:
          IMAGE: ${{ steps.login-ecr.outputs.registry }}/payment-service:${{ github.sha }}
        run: |
          kubectl set image deployment/payment-service payment=$IMAGE -n production
          kubectl rollout status deployment/payment-service -n production --timeout=180s
```

---

### **22. What is an ArgoCD ApplicationSet and what generators does it support for multi-cluster scaling?**

**Detailed Answer:**
The **`ApplicationSet`** controller automates the dynamic generation and lifecycle management of multiple ArgoCD `Application` resources from a single declarative template.

#### **Core Generators:**
1. **List Generator:** Targets a static list of cluster names and environments.
2. **Cluster Generator:** Automatically deploys applications to all Kubernetes clusters registered with ArgoCD matching label selectors (e.g., `environment: production`).
3. **Git Directory Generator:** Dynamically creates applications for every subdirectory discovered in a Git repository (`apps/*`).
4. **Matrix Generator:** Combines generators (e.g., combine 5 clusters $\times$ 10 Git directories = dynamically generates 50 ArgoCD applications).

---

### **23. What are ArgoCD Sync Waves and Sync Phases? Provide an example managing ordered migrations.**

**Detailed Answer:**
Sync Waves control the exact sequential order in which Kubernetes manifests are applied during an ArgoCD sync.
- Defined via annotation: `argocd.argoproj.io/sync-wave: "1"` (lower and negative numbers run first).

```yaml
# Step 1: Database Migration Job (Runs in Wave 0)
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  annotations:
    argocd.argoproj.io/sync-wave: "0"
spec:
  template:
    spec:
      containers:
        - name: migrate
          image: my-app:v2
          command: ["./migrate-db.sh"]
      restartPolicy: Never
---
# Step 2: Deployment Rollout (Runs in Wave 1 - ONLY after Job succeeds!)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  replicas: 5
```

---

### **24. How does Progressive Delivery work with Argo Rollouts and Prometheus Metric Analysis?**

**Detailed Answer:**
Argo Rollouts replaces the native Kubernetes `Deployment` with a custom `Rollout` CRD that coordinates automated canary traffic shifts and metric evaluation.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: payment-service
spec:
  replicas: 10
  strategy:
    canary:
      analysis:
        templates:
          - templateName: success-rate-check
      steps:
        - setWeight: 10
        - pause: { duration: 5m }
        - setWeight: 50
        - pause: { duration: 10m }
```
If Prometheus reports that the success rate metric falls below $99.5\%$ during the 5-minute pause, Argo Rollouts **aborts the deployment and rolls back to 0% traffic automatically**.

---

### **25. What is an SBOM (Software Bill of Materials) and how do you generate and scan one in CI/CD pipelines?**

**Detailed Answer:**
An **SBOM** is a formal, machine-readable nested inventory of all software components, third-party libraries, dependencies, and license metadata bundled within a container image.

#### **CI/CD Pipeline Generation & Scanning:**
```bash
# 1. Generate CycloneDX SBOM using Syft
syft packages ghcr.io/my-org/payment:v1.0.0 -o cyclonedx-json=sbom.json

# 2. Scan the SBOM for known CVEs using Grype
grype sbom:sbom.json --fail-on high
```

---

### **26. What is Sigstore Cosign and Keyless Container Image Signing?**

**Detailed Answer:**
Cosign cryptographically signs container images in OCI registries to ensure authenticity and integrity.
- **Keyless Signing:** Uses OIDC federation with GitHub Actions. Fulcio issues a short-lived X.509 certificate bound to the GitHub workflow identity, and Rekor records the signature in an immutable transparency log, eliminating static PGP private key management.

---

### **27. What is Actions Runner Controller (ARC) and how does it scale self-hosted runners on Kubernetes?**

**Detailed Answer:**
**ARC** is a Kubernetes operator that deploys and autoscales GitHub Actions self-hosted runner pods on Kubernetes:
- **Autoscaling:** Listens to GitHub webhook events (`workflow_job`) and scales runner pods dynamically from 0 to hundreds in seconds.
- **Ephemeral Pod Security:** Every runner pod executes exactly one job and is immediately terminated, ensuring no state or sensitive credentials persist between builds.

---

### **28. How do you implement Concurrency Control and Cancel-in-Progress in GitHub Actions?**

**Detailed Answer:**
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```
When a developer pushes three commits in rapid succession to a pull request, `cancel-in-progress: true` automatically terminates running pipeline jobs for older commits, freeing compute runners immediately.

---

### **29. What is a GitHub Merge Queue and how does it prevent broken main branches?**

**Detailed Answer:**
In high-velocity teams, multiple PRs pass CI independently against outdated base branches. When merged concurrently, their combined interactions can break `main`.
- **Merge Queue:** Automatically tests PRs in an integrated sequential train against the anticipated merge result of prior queued PRs. If a PR fails the combined integration test, it is dropped from the train without impacting `main`.

---

### **30. How do you handle database rollbacks in automated CI/CD pipelines?**

**Detailed Answer:**
Standard code rollbacks (`kubectl rollout undo`) cannot revert destructive database schema changes (e.g., dropped columns).
- **Rule:** Never execute backward-incompatible DB changes in a single release.
- Enforce **Expand and Contract (Parallel Run)** migrations so older application versions remain 100% operational if a code rollback is triggered.

---

### **31. How does Docker BuildKit inline caching optimize CI build speeds?**

**Detailed Answer:**
BuildKit allows caching intermediate layer artifacts directly in remote OCI container registries:
```bash
docker buildx build \
  --cache-to=type=registry,ref=my-registry/app:cache,mode=max \
  --cache-from=type=registry,ref=my-registry/app:cache \
  --push -t my-registry/app:v1.0.0 .
```
Enables independent ephemeral CI runners to reuse remote cache layers without mounting persistent local disk volumes.

---

### **32. What is Flagger and how does it implement Progressive Delivery on Kubernetes?**

**Detailed Answer:**
Flagger is a CNCF progressive delivery operator that integrates with Service Meshes (Istio, Linkerd) and Ingress Controllers (Nginx). It manages Canary CRDs, dynamically adjusting traffic weights while querying Prometheus for request latency and error rate metrics.

---

### **33. Compare Trunk-Based CI vs Feature-Branch CI.**

**Detailed Answer:**
- **Feature-Branch CI:** Pipelines validate isolated branches that live for days/weeks. Delays integration testing until large, painful merges occur.
- **Trunk-Based CI:** Developers merge code into `main` multiple times a day. Pipelines run fast, rigorous test suites ($< 10$ minutes) ensuring `main` is constantly in a deployable state.

---

### **34. What is DAST (Dynamic Application Security Testing) in CI/CD?**

**Detailed Answer:**
DAST evaluates running web applications from the outside by attacking endpoints, injecting malicious payloads (SQLi, XSS), and testing authentication vulnerabilities (e.g., OWASP ZAP). Unlike SAST (which inspects source code), DAST detects runtime misconfigurations and authentication bypasses.

---

### **35. Compare ArgoCD vs FluxCD for Kubernetes GitOps.**

**Detailed Answer:**
- **ArgoCD:** Visual Web UI, ApplicationSet controller, multi-cluster management, rich SSO/RBAC, widely favored by Platform Engineering teams.
- **FluxCD:** Highly modular toolkit (Source Controller, Kustomize Controller, Helm Controller), headless (Git/CLI-first), seamless integration with notification webhooks.

---

### **36. Compare Sealed Secrets vs External Secrets Operator (ESO) in GitOps.**

**Detailed Answer:**
- **Bitnami Sealed Secrets:** Secrets are encrypted client-side with a public key and safely stored in Git; decrypted inside the cluster by the Sealed Secrets controller.
- **External Secrets Operator (ESO):** Git contains only declarative references (`ExternalSecret`). The ESO controller fetches real secret values dynamically from AWS Secrets Manager or HashiCorp Vault.

---

### **37. What is Pipeline as Code in GitLab CI (`.gitlab-ci.yml`) vs Jenkinsfile?**

**Detailed Answer:**
- **GitLab CI:** Native declarative YAML format integrated directly with GitLab repositories, container registries, and auto-DevOps.
- **Jenkinsfile:** Groovy-based pipeline script running on external Jenkins controller/agent architecture requiring manual plugin management.

---

### **38. What is Mutation Testing in CI pipelines?**

**Detailed Answer:**
Mutation testing (e.g., Stryker, Mutmut) evaluates the **quality of unit tests** by introducing small intentional bugs (mutations) into source code. If unit tests still pass, the mutation survived, indicating weak test assertions.

---

### **39. What is Atlantis for Pull Request-driven Terraform Workflows?**

**Detailed Answer:**
Atlantis runs Terraform workflows directly inside Pull Request comments:
- Developer opens PR $\rightarrow$ Atlantis runs `terraform plan` and comments the diff on the PR.
- Team reviews and approves $\rightarrow$ Engineer types `atlantis apply` in the PR comment.
- Atlantis applies changes, posts output, and automatically merges the PR.

---

### **40. What is SLSA Level 3 in CI/CD pipelines?**

**Detailed Answer:**
SLSA Level 3 requires:
1. **Source Integrity:** Verified version control history and two-person code reviews.
2. **Build Isolation:** Builds executed on dedicated, ephemeral, isolated build platforms (not developer laptops).
3. **Non-falsifiable Provenance:** Build metadata is cryptographically generated by the CI build service itself, documenting the source repository, commit SHA, and build steps.

---

## 🔴 **Advanced & Scenario-Based Level (Questions 41–50)**

### **41. Scenario: An engineer triggers an ArgoCD sync on production, but the sync hangs indefinitely in "Progressing" state due to a failed PreSync Hook. Walk through step-by-step triage and recovery.**

**Detailed Answer:**
**Root Cause:** ArgoCD PreSync hooks (e.g., a DB migration `Job`) must complete successfully (`Complete` status) before ArgoCD creates or updates the core deployment resources. If the hook enters `Error` or `CrashLoopBackOff`, the entire sync halts.

#### **Resolution Steps:**
1. Check hook pod logs and status:
   ```bash
   kubectl get jobs -n production
   kubectl logs -n production job/db-migration-job
   ```
2. Terminate the hung sync operation via ArgoCD CLI:
   ```bash
   argocd app terminate-op payment-service
   ```
3. Fix the database migration script in Git or temporarily annotate the Job to bypass if safe:
   ```yaml
   annotations:
     argocd.argoproj.io/hook-delete-policy: HookFailed
   ```
4. Re-trigger the sync after pushing the corrected migration.

---

### **42. Scenario: Your GitHub Actions CI workflow suddenly starts hitting API rate limits and build jobs fail randomly with HTTP 429. How do you architect an enterprise-grade solution?**

**Detailed Answer:**
1. **Switch to Authenticated Requests:** Ensure all API queries (e.g., Docker Hub, GitHub API, NPM registry) use authenticated tokens rather than anonymous IP lookups.
2. **Deploy Local Pull-Through Caches:** Deploy **Harbor** or **AWS ECR Pull Through Cache** in your VPC so runners pull common base images locally rather than reaching Docker Hub.
3. **Dependency Caching:** Leverage actions caching (`actions/cache`) and self-hosted persistent volume mounts for Gradle/Maven/NPM.
4. **Implement Exponential Backoff with Jitter** in custom CLI scripts.

---

### **43. Scenario: A developer committed an AWS IAM Access Key to a public GitHub repository. CI failed, but the key is in Git history. Walk through the complete remediation procedure.**

**Detailed Answer:**
1. **Immediate Revocation:** Immediately log into AWS IAM Console / CLI and **deactivate and delete the exposed Access Key ID**.
2. **Audit CloudTrail:** Review AWS CloudTrail logs for that specific access key over the past 24 hours to check for unauthorized resource creation or data exfiltration.
3. **Rewrite Git History:**
   - Use `git-filter-repo` to purge the sensitive string from all commits, branches, and tags:
     ```bash
     git-filter-repo --replace-text <(echo 'AKIAEXAMPLESECRET==>REDACTED')
     ```
   - Force push cleaned branches: `git push origin --force --all`.
4. **Implement Prevention:** Configure GitHub Secret Scanning & Push Protection to block commits containing secrets before push.

---

### **44. How do you implement Zero-Downtime Database Migrations in a GitOps Pipeline without locking production tables?**

**Detailed Answer:**
1. **Decouple Migrations from App Deployments:** Run migrations via dedicated Kubernetes Jobs executed *before* application pod rollout.
2. **Non-Blocking Schema Operations:**
   - In PostgreSQL, use `CREATE INDEX CONCURRENTLY` to prevent table-level write locks.
   - In MySQL, use tools like `gh-ost` or `pt-online-schema-change`.
3. **Expand and Contract Sequence:**
   - *Stage 1 (Release A):* Add new nullable columns.
   - *Stage 2 (Release B):* Deploy application writing to both columns and reading from new.
   - *Stage 3 (Release C):* Drop legacy unused columns.

---

### **45. Scenario: How do you design an ephemeral preview environment pipeline in Kubernetes triggered on PR creation and destroyed on merge?**

**Detailed Answer:**
1. **Trigger:** PR opened/synchronized in GitHub Actions.
2. **Build & Tag:** Build container image tagged with `pr-${{ github.event.pull_request.number }}` and push to container registry.
3. **Dynamic Namespace Creation:** Create an isolated namespace `preview-pr-${PR_NUMBER}`.
4. **Deploy via Helm / Kustomize:** Deploy the application and mock backend services using Helm values scoped to the dynamic namespace.
5. **Dynamic DNS Ingress:** Route incoming traffic via wildcard DNS: `https://pr-${PR_NUMBER}.preview.example.com`.
6. **PR Comment:** Post the preview URL back to the GitHub PR thread.
7. **Cleanup Trigger (`on: pull_request, types: [closed]`):** Delete namespace `kubectl delete ns preview-pr-${PR_NUMBER}`, freeing all cloud resources.

---

### **46. What is the difference between In-Tree vs Out-of-Tree CI/CD pipeline plugins and what are the security trade-offs?**

**Detailed Answer:**
- **In-Tree Plugins:** Built directly into the core CI engine. High performance, but updates require upgrading the entire CI server.
- **Out-of-Tree / Marketplace Actions:** Third-party community plugins fetched dynamically at runtime (e.g., `uses: actions/setup-node@v4`).
  - *Security Risk:* Vulnerable to supply chain attacks if the action repository is compromised.
  - *Hardening:* Always pin third-party actions to full commit SHAs (`uses: actions/setup-node@60edb5dd545a775178f525247833781be2afd1ce`) rather than mutable tags (`@v4`).

---

### **47. How do you architect a High-Availability, Fault-Tolerant Jenkins architecture on Kubernetes?**

**Detailed Answer:**
- **Stateless Agent Execution:** Jenkins Master runs on Kubernetes with persistent storage for configuration (`/var/jenkins_home` backed by EBS/EFS CSI driver).
- **Ephemeral Kubernetes Cloud Plugin:** Jenkins controller spawns dynamic agent pods per build job that terminate immediately on job completion.
- **Job Configuration as Code (JCasC):** Jenkins configuration defined 100% in YAML and stored in Git.
- **Disaster Recovery:** If Jenkins master pod dies, Kubernetes restarts it in seconds, re-mounting the persistent volume and reloading JCasC state automatically.

---

### **48. What is Chaos Testing in CI/CD pipelines and how do you implement Automated Resilience Gating?**

**Detailed Answer:**
1. Deploy new application version to staging.
2. Run automated synthetic load test.
3. Inject faults via **Chaos Mesh** or **LitmusChaos** (e.g., terminate 30% of backend pods, inject 200ms network packet latency, sever Redis connection).
4. Automated Pipeline Gate checks if application error rate remained $< 1\%$ and circuit breakers handled degradation gracefully.

---

### **49. What is Hermetic Build in enterprise CI/CD and why is it crucial for reproducible builds?**

**Detailed Answer:**
A **hermetic build** is executed in a completely isolated container/sandbox with **zero outbound internet access**.
- All dependencies, compilers, and toolchains are pre-fetched, cryptographically hashed, and provided locally into the build sandbox.
- **Why it matters:** Guarantees that compiling code from commit `abc1234` in 2026 will produce the exact bit-for-bit identical binary as in 2030, preventing external package registry outages or compromised upstream packages from altering builds.

---

### **50. How do you implement a secure Multi-Tenant CI/CD platform where untrusted customer code runs safely?**

**Detailed Answer:**
1. **Sandboxed Container Runtimes:** Use **gVisor** (`runsc`) or **Kata Containers** (microVMs based on QEMU/Firecracker) instead of standard runc to prevent kernel privilege escalation and container breakouts.
2. **Ephemeral Isolated Runners:** Each build executes in an isolated short-lived VM/pod created on-demand and wiped after execution.
3. **Strict Network Policies:** Block access to the Kubernetes API server, cloud metadata service (`169.254.169.254`), and internal private VPC networks.
4. **No Host Docker Socket Mounting:** Prohibit mounting `/var/run/docker.sock`. Use rootless container build tools like **Kaniko**, **Buildah**, or **Podman**.
