# **CI/CD & GitOps - DevOps Interview Questions**

Welcome to the **CI/CD & GitOps** interview questions module. This section covers continuous integration, continuous delivery, modern GitOps patterns (ArgoCD, Flux), pipeline security (OIDC, SLSA, SBOM), progressive delivery (Argo Rollouts, Canary), and enterprise GitHub Actions/GitLab CI architectures.

---

## 🟢 **Beginner Level (Questions 1–20)**

### **1. What is CI/CD and what is the key difference between Continuous Delivery and Continuous Deployment?**
**Answer:**
**Continuous Integration (CI):** The automated process where developers frequently commit code to a shared repository, triggering automated builds, linters, and unit/integration test suites to detect bugs early.

**Continuous Delivery (CD) vs Continuous Deployment (CD):**
- **Continuous Delivery:** Code is automatically tested, packaged, and staged in a production-ready state. Triggering actual production deployment requires a manual one-click approval.
- **Continuous Deployment:** Every commit that passes the automated pipeline is deployed directly into production with zero human intervention.

---

### **2. What are the key stages of a robust production CI/CD pipeline?**
**Answer:**
```
[ Commit ] ➔ [ Lint & SAST ] ➔ [ Build & Unit Test ] ➔ [ Containerize & SCA/SBOM ] ➔ [ Deploy Staging ] ➔ [ DAST & E2E ] ➔ [ Production Rollout ]
```
1. **Source / Trigger:** Webhook on Git push or PR creation.
2. **Static Analysis & Linting:** Code formatting, SAST (SonarQube, Semgrep), Secret scanning (Gitleaks).
3. **Build & Unit Tests:** Compiling code, running fast unit test suites.
4. **Artifact Packaging & Security:** Multi-stage Docker builds, vulnerability scanning (Trivy), SBOM generation, cryptographic signing (Cosign).
5. **Staging Deployment:** Ephemeral preview environments or staging clusters.
6. **Dynamic Testing:** End-to-end (E2E) automated browser/API tests, load tests, DAST (OWASP ZAP).
7. **Production Rollout:** Canary, Blue-Green, or GitOps sync with metric analysis.

---

### **3. What is an Artifact in CI/CD and why should it be immutable?**
**Answer:**
An artifact is the compiled, packaged, deployable unit created during the build stage (e.g., Docker image, JAR, NPM package, Helm chart).

**Why Immutability Matters:**
- **"Build Once, Deploy Anywhere":** The exact same binary/container image tested in Dev and Staging must be promoted to Production.
- **Avoids Subtle Environmental Bugs:** Rebuilding code in each environment introduces risks of differing dependency versions or compiler changes.
- **Traceability:** Unique tags (Git commit SHA or SemVer) ensure deterministic rollbacks.

---

### **4. What are Reusable Workflows and Composite Actions in GitHub Actions?**
**Answer:**
- **Reusable Workflows (`workflow_call`):** Entire workflow files called from other workflows across repos. Can contain multiple jobs, define inputs/secrets, and enforce centralized organizational security policies.
- **Composite Actions (`action.yml`):** Reusable packaging of multiple shell steps or action steps into a single reusable action step within a single job.

---

### **5. What is GitOps and how does it transform traditional CI/CD?**
**Answer:**
GitOps uses Git repositories as the **single source of truth** for declarative infrastructure and application deployments.

**Differences:**
- **Traditional CI/CD (Push Model):** CI server (Jenkins, GitHub Actions) holds production cluster credentials and runs `kubectl apply` directly into the cluster. High security risk if CI is breached.
- **GitOps (Pull Model):** An agent inside the Kubernetes cluster (ArgoCD / Flux) continuously pulls desired state from Git and reconciles live cluster state. No external cluster credentials stored in CI.

---

### **6. What is Semantic Versioning (SemVer) and how is it automated in CI/CD?**
**Answer:**
Semantic Versioning uses the format `MAJOR.MINOR.PATCH` (e.g., `2.4.1`):
- **MAJOR:** Breaking API changes.
- **MINOR:** Backward-compatible new features.
- **PATCH:** Backward-compatible bug fixes.

**Automation:** Tools like **Semantic Release** or **Release Please** parse Conventional Commits (`feat:`, `fix:`, `feat!:`) to automatically calculate version bumps, generate changelogs, and create Git release tags in CI.

---

### **7. What is Matrix Strategy in CI/CD pipelines?**
**Answer:**
A matrix build allows running a single job across multiple combinations of operating systems, language runtime versions, or configurations concurrently.

**Example (GitHub Actions):**
```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest]
    node-version: [18.x, 20.x, 22.x]
```
This single definition automatically generates and executes $2 \times 3 = 6$ parallel jobs.

---

### **8. What is Caching in CI/CD and what should be cached vs preserved as artifacts?**
**Answer:**
- **Caching (`actions/cache`):** Stores intermediate dependencies (e.g., `~/.npm`, `~/.m2`, Docker layer caches) to speed up subsequent pipeline runs. Ephemeral; missing caches only slow down builds.
- **Artifacts (`actions/upload-artifact`):** Persists output binaries, test reports, and binaries passed between pipeline jobs or retained for auditing.

---

### **9. What is Blue-Green Deployment and how do you achieve it in Kubernetes?**
**Answer:**
Blue-Green maintains two identical environments: Blue (active production) and Green (idle new release).
In Kubernetes, you deploy the new version as a separate Deployment (Green). Once health checks pass, update the `Service` selector to point to the Green deployment:
```yaml
spec:
  selector:
    app: my-service
    version: v2  # Switch from v1 to v2 instantly
```

---

### **10. What is Canary Deployment and how does it mitigate release risk?**
**Answer:**
Canary deployment routes a small percentage of real user traffic (e.g., 5%) to the new version while 95% remains on stable. Automated analysis observes HTTP 5xx error rates and p99 latency. If metrics remain healthy, traffic is incrementally shifted to 100%.

---

### **11. What is the difference between Self-Hosted Runners and Cloud-Hosted Runners?**
**Answer:**
- **Cloud-Hosted (GitHub/GitLab managed):** Ephemeral, zero maintenance, auto-scaled, but costlier for high-volume pipelines and lack access to internal private VPC resources.
- **Self-Hosted:** Deployed inside your private cloud/VPC. Has direct access to private subnets, GPUs, custom hardware, and persistent local caches. Requires management (scaling, patching, security isolation).

---

### **12. What is OIDC (OpenID Connect) in CI/CD and why should you eliminate static cloud credentials?**
**Answer:**
Static cloud credentials (e.g., `AWS_ACCESS_KEY_ID` stored as long-lived secrets in GitHub) are vulnerable to leakage, require manual rotation, and present high security blast radius.

**OIDC Authentication:**
1. CI Runner requests a short-lived JSON Web Token (JWT) from GitHub/GitLab.
2. Runner exchanges this token with AWS STS / GCP Workload Identity / Azure AD.
3. Cloud provider validates the token claims (repo name, branch) and issues temporary, scoped IAM credentials (valid for 1 hour). No static credentials exist.

---

### **13. What is a Linter and why must it run in the earliest pipeline stage?**
**Answer:**
A linter statically analyzes code against style conventions, potential syntax errors, unhandled exceptions, and dead code without executing it (e.g., ESLint, Flake8, GolangCI-Lint, ShellCheck). Running linters first provides sub-minute feedback and saves expensive build/test compute minutes.

---

### **14. What are Ephemeral / Preview Environments?**
**Answer:**
Ephemeral environments are dynamic, short-lived environments spun up automatically when a Pull Request is opened and destroyed when the PR is merged or closed.

**Value:** Enables Product Managers, QA, and security engineers to test live feature branches in isolation before merging into `main`.

---

### **15. What is SonarQube / Quality Gate in CI/CD?**
**Answer:**
SonarQube is a static code analysis platform that calculates code coverage, code smells, technical debt, and security vulnerabilities. A **Quality Gate** is a conditional pass/fail rule (e.g., *Coverage $\ge$ 80%*, *0 New Critical Vulnerabilities*); if failed, the CI pipeline blocks the PR from merging.

---

### **16. What is Secret Masking in CI/CD pipelines?**
**Answer:**
Secret masking is a security feature where CI engines detect registered secret strings in stdout/stderr logs and replace them with `***` to prevent accidental credential leakage in build outputs.

---

### **17. What is Webhook in CI/CD?**
**Answer:**
A webhook is an HTTP POST callback triggered by Git repository events (e.g., `push`, `pull_request_opened`, `tag_created`) sent to the CI/CD server to trigger immediate pipeline execution asynchronously.

---

### **18. What is Jenkins Pipeline as Code (Declarative vs Scripted Jenkinsfile)?**
**Answer:**
- **Declarative Pipeline:** Strict, structured syntax with predefined stages (`pipeline { agent any; stages { ... } }`). Easier to read, maintain, and validate.
- **Scripted Pipeline:** Groovy-based procedural syntax (`node { ... }`). Offers maximum programmatic flexibility but is harder to maintain and test.

---

### **19. What is a Monorepo CI Strategy and how do you avoid rebuilding everything on every commit?**
**Answer:**
In a monorepo, changing a single microservice should not trigger builds for 50 other services.

**Solutions:**
- **Path Filtering:** Trigger pipelines only when files under specific directories change (e.g., `paths: ['services/payment/**']`).
- **Build Systems with Change Graphs:** Tools like **Bazel**, **Turborepo**, or **Nx** compute dependency DAGs and execute builds/tests only for affected modules.

---

### **20. What is Dark Launching with Feature Flags in CI/CD?**
**Answer:**
Dark launching is deploying code to production completely silently (behind a feature flag set to disabled) before enabling it for actual users. This decouples deployment from feature enablement and validates production compatibility under live data traffic.

---

## 🟡 **Intermediate Level (Questions 21–40)**

### **21. How do you configure GitHub Actions with AWS using OIDC (Zero Static Keys)?**
**Answer:**
**GitHub Actions Workflow Configuration:**
```yaml
name: Deploy to AWS EKS
on:
  push:
    branches: [main]

permissions:
  id-token: write   # Required for requesting OIDC JWT token
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Configure AWS Credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsEKSRole
          aws-region: us-east-1

      - name: Verify AWS Identity
        run: aws sts get-caller-identity
```

---

### **22. What is ArgoCD ApplicationSet and how does it automate multi-cluster deployments?**
**Answer:**
An `ApplicationSet` controller automates the generation of multiple ArgoCD `Application` resources from a single template using **Generators**:
- **Cluster Generator:** Automatically deploys apps across all Kubernetes clusters registered with ArgoCD.
- **Git Directory Generator:** Dynamically creates applications for every folder matching `apps/*` in a repository.
- **Matrix / List Generator:** Combines generators to support complex multi-tenant, multi-region matrices.

---

### **23. What are ArgoCD Sync Waves and Sync Phases?**
**Answer:**
Sync Waves control the exact execution order of Kubernetes resources during an ArgoCD synchronization:
- Defined via annotation: `argocd.argoproj.io/sync-wave: "1"` (negative numbers run first).
- **Phases:** `PreSync` (e.g., DB migration Job) $\rightarrow$ `Sync` (Deployments, Services) $\rightarrow$ `PostSync` (Slack notifications, cache warming) $\rightarrow$ `SyncFail`.

---

### **24. How does Progressive Delivery work with Argo Rollouts and Prometheus metric analysis?**
**Answer:**
Argo Rollouts replaces the native Kubernetes `Deployment` with a custom `Rollout` CRD that supports canary traffic shifting and automated metric analysis.

**Example Rollout with Analysis:**
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
          - templateName: success-rate
        args:
          - name: service-name
            value: payment-service
      steps:
        - setWeight: 10
        - pause: { duration: 5m }
        - setWeight: 50
        - pause: { duration: 10m }
```
If the `success-rate` Prometheus query returns `< 99.5%` during the pause, the rollout automatically aborts and rolls back to 0%.

---

### **25. What is an SBOM (Software Bill of Materials) and how do you generate and scan one in CI/CD?**
**Answer:**
An SBOM is a formal, machine-readable inventory of all software components, third-party libraries, dependencies, and license metadata bundled within a container or binary.

**Generation in CI (using Syft):**
```bash
syft packages my-app:v1.0.0 -o cyclonedx-json=sbom.json
```
**Scanning SBOM for Vulnerabilities (using Grype):**
```bash
grype sbom:sbom.json --fail-on high
```

---

### **26. What is Cosign / Sigstore and how do you sign and verify container images in CI/CD?**
**Answer:**
Cosign provides cryptographic signing and verification for container images in OCI registries.

**Signing in CI (Keyless with OIDC):**
```bash
cosign sign --yes $IMAGE_URI
```
**Kubernetes Admission Enforcement:**
Kyverno or OPA Gatekeeper checks image signatures before allowing pods to start in the cluster:
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: check-image-signature
spec:
  rules:
    - name: verify-signature
      match:
        resources:
          kinds: ["Pod"]
      verifyImages:
        - imageReferences: ["ghcr.io/my-org/*"]
          keyless:
            issuer: "https://token.actions.githubusercontent.com"
            subject: "https://github.com/my-org/my-repo/.github/workflows/deploy.yml@refs/heads/main"
```

---

### **27. What is Actions Runner Controller (ARC) for Kubernetes?**
**Answer:**
ARC is a Kubernetes operator that deploys and autoscales GitHub Actions self-hosted runners as pods inside your Kubernetes cluster.
- **Autoscaling:** Dynamically scales runner pods from 0 to hundreds based on GitHub webhook events (`workflow_job`).
- **Security:** Each runner pod is ephemeral—created for a single job and immediately destroyed upon completion to prevent state pollution.

---

### **28. What is Concurrency Control and Cancel-in-Progress in CI/CD?**
**Answer:**
When multiple commits are pushed in rapid succession to a pull request, running pipelines on outdated commits wastes compute resources.

**GitHub Actions Solution:**
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```
This automatically cancels any running pipeline job for older commits on that branch as soon as a newer commit is pushed.

---

### **29. What is a GitHub Merge Queue and what problem does it solve?**
**Answer:**
In high-velocity teams, multiple PRs pass CI independently against outdated base branches. When merged simultaneously, the combination may break `main`.

**Merge Queue:** Automatically tests PRs in an integrated train against the anticipated merge result of prior queued PRs. If a PR fails the combined test, it is dropped from the train without breaking `main`.

---

### **30. How do you handle database rollbacks in automated CI/CD pipelines?**
**Answer:**
- Standard code rollbacks (`kubectl rollout undo`) cannot revert destructive database schema changes (e.g., dropped columns).
- **Rule:** Never execute backward-incompatible DB changes in a single step.
- Implement **Backward-Compatible Multi-Phase Migrations** (Expand $\rightarrow$ Contract) so older application versions remain 100% operational if a code rollback is triggered.

---

### **31. What is Docker Layer Caching in CI pipelines and how do BuildKit inline caches work?**
**Answer:**
Docker builds execute in layers. If a layer hasn't changed, Docker reuses the cached filesystem layer.

**Optimization in CI:**
```dockerfile
# Step 1: Copy only dependency manifests first
COPY package.json package-lock.json ./
RUN npm ci

# Step 2: Copy application source code (frequently changing)
COPY . .
RUN npm run build
```
Using BuildKit OCI registry caching (`--cache-to=type=registry --cache-from=type=registry`) allows CI runners to reuse remote cache layers across independent runner instances.

---

### **32. What is Flagger and how does it implement Progressive Delivery on Kubernetes?**
**Answer:**
Flagger is a CNCF progressive delivery operator that automates release promotions using Service Meshes (Istio, Linkerd) or Ingress Controllers (Nginx, Traefik). It creates Canary CRDs that gradually adjust traffic weights (0% $\rightarrow$ 10% $\rightarrow$ 20% $\rightarrow$ 50% $\rightarrow$ 100%) while querying Prometheus for latency and 5xx error metrics.

---

### **33. What is Trunk-Based CI vs Feature-Branch CI?**
**Answer:**
- **Feature-Branch CI:** Pipelines only validate isolated branches that live for days/weeks. Hides integration bugs until large merges occur.
- **Trunk-Based CI:** Developers integrate code into `main` multiple times a day. Pipelines run fast, rigorous test suites (< 10 minutes) ensuring `main` is always in a deployable, releasable state.

---

### **34. What is DAST (Dynamic Application Security Testing) in CI/CD?**
**Answer:**
DAST evaluates running web applications from the outside by attacking endpoints, injecting payloads, and testing authentication vulnerabilities (e.g., OWASP ZAP, Burp Suite). Unlike SAST (which inspects source code), DAST detects runtime misconfigurations and authentication bypasses.

---

### **35. What is the difference between ArgoCD and FluxCD?**
**Answer:**
- **ArgoCD:** Visual Web UI, Application/ApplicationSet CRDs, centralized multi-cluster management, excellent SSO/RBAC, widely favored by platform teams.
- **FluxCD:** Highly modular toolkit (Source Controller, Kustomize Controller, Helm Controller), headless (CLI/Git-first without mandatory UI), native integration with Git notifications and Helm releases.

---

### **36. What is Secret Management in GitOps (Sealed Secrets vs External Secrets Operator)?**
**Answer:**
Since Git is public or accessible across the organization, plaintext secrets cannot be committed.
- **Bitnami Sealed Secrets:** Secrets are encrypted client-side with a public key and safely stored in Git. Only the Sealed Secrets controller inside the cluster possesses the private key to decrypt them.
- **External Secrets Operator (ESO):** Git contains only declarative references (`ExternalSecret`). The ESO controller fetches actual secret values dynamically from AWS Secrets Manager, HashiCorp Vault, or Azure Key Vault and synchronizes native Kubernetes Secrets.

---

### **37. What is Pipeline as Code in GitLab CI (`.gitlab-ci.yml`) vs Jenkinsfile?**
**Answer:**
- **GitLab CI:** Declarative YAML format natively integrated with GitLab repos, container registry, auto-DevOps, and runners.
- **Jenkinsfile:** Groovy-based pipeline script running on external Jenkins controller/agent architecture. Requires extensive plugin management and server administration.

---

### **38. What is Mutation Testing in CI pipelines?**
**Answer:**
Mutation testing (e.g., Stryker, Mutmut) evaluates the **quality of unit tests** by introducing small intentional bugs (mutations) into source code. If unit tests still pass, the mutation survived, indicating weak test assertions.

---

### **39. What is a GitOps Pull Request automation bot (e.g., Atlantis for Terraform)?**
**Answer:**
Atlantis runs Terraform workflows directly inside Pull Request comments:
- Developer opens PR $\rightarrow$ Atlantis runs `terraform plan` and comments the diff on the PR.
- Team reviews and approves $\rightarrow$ Engineer types `atlantis apply` in the PR comment.
- Atlantis applies changes, posts output, and automatically merges the PR.

---

### **40. What is SLSA (Supply-chain Levels for Software Artifacts) Level 3 in CI/CD?**
**Answer:**
SLSA Level 3 requires:
1. **Source Integrity:** Verified version control history and two-person code reviews.
2. **Build Isolation:** Builds executed on dedicated, ephemeral, isolated build platforms (not developer laptops).
3. **Non-falsifiable Provenance:** Build metadata is cryptographically generated by the CI build service itself, documenting the source repository, commit SHA, and build steps.

---

## 🔴 **Advanced & Scenario-Based Level (Questions 41–50)**

### **41. Scenario: An engineer triggers an ArgoCD sync on production, but the sync hangs indefinitely in "Progressing" state due to a failed PreSync Hook. How do you troubleshoot and recover?**
**Answer:**
**Root Cause:** ArgoCD PreSync hooks (e.g., a DB migration `Job`) must complete successfully (`Complete` status) before ArgoCD creates or updates the core deployment resources. If the hook enters `Error` or `CrashLoopBackOff`, the entire sync halts.

**Resolution Steps:**
1. Check hook pod logs and status:
   ```bash
   kubectl get jobs -n production
   kubectl logs -n production job/db-migration-job
   ```
2. If the hook is broken, terminate the hung sync:
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

### **42. Scenario: Your GitHub Actions CI workflow suddenly starts hitting API rate limits and build jobs fail randomly with HTTP 429. How do you architect a solution?**
**Answer:**
1. **Switch to Authenticated Requests:** Ensure all API queries (e.g., Docker Hub, GitHub API, NPM registry) use authenticated tokens rather than anonymous IP lookups.
2. **Local Registry / Dependency Mirrors:** Deploy pull-through caches (e.g., Harbor, Sonatype Nexus, AWS ECR Pull Through Cache) in your VPC so runners don't pull common base images from external Docker Hub.
3. **Dependency Caching:** Leverage actions caching (`actions/cache`) and self-hosted persistent volume mounts for Gradle/Maven/NPM.
4. **Implement Exponential Backoff with Jitter** in custom CLI scripts.

---

### **43. Scenario: A developer committed an AWS IAM Access Key to a public GitHub repository. CI failed, but the key is now in the Git commit history. Walk through the complete remediation.**
**Answer:**
1. **Immediate Revocation:** Do not simply delete the commit. Immediately log into AWS IAM Console / CLI and **deactivate and delete the exposed Access Key ID**.
2. **Audit CloudTrail:** Review AWS CloudTrail logs for that specific access key over the past 24 hours to check for unauthorized resource creation, data exfiltration, or IAM privilege escalation.
3. **Rewrite Git History:**
   - Use `git-filter-repo` or BFG Repo-Cleaner to purge the sensitive string from all commits, branches, and tags:
     ```bash
     git-filter-repo --replace-text <(echo 'AKIAEXAMPLESECRET==>REDACTED')
     ```
   - Force push cleaned branches: `git push origin --force --all`.
4. **Implement Prevention:**
   - Configure GitHub Secret Scanning & Push Protection to block commits containing secrets before push.
   - Enforce pre-commit hooks (`gitleaks protect`).

---

### **44. How do you implement Zero-Downtime Database Migrations in a GitOps Pipeline without locking production tables?**
**Answer:**
1. **Decouple Migrations from App Deployments:** Run migrations via dedicated Kubernetes Jobs executed *before* application pod rollout.
2. **Non-Blocking Schema Operations:**
   - In PostgreSQL, use `CREATE INDEX CONCURRENTLY` to prevent table-level write locks.
   - In MySQL, use tools like `gh-ost` or `pt-online-schema-change`.
3. **Expand and Contract Sequence:**
   - *Stage 1 (GitOps Release A):* Add new nullable columns / non-breaking schema additions.
   - *Stage 2 (GitOps Release B):* Deploy application writing to both columns and reading from new.
   - *Stage 3 (GitOps Release C):* Drop legacy unused columns.

---

### **45. Scenario: How do you design an ephemeral preview environment pipeline in Kubernetes triggered on PR creation and destroyed on merge?**
**Answer:**
1. **Trigger:** PR opened/synchronized in GitHub Actions.
2. **Build & Tag:** Build container image tagged with `pr-${{ github.event.pull_request.number }}` and push to container registry.
3. **Dynamic Namespace Creation:** Create an isolated namespace `preview-pr-${PR_NUMBER}`.
4. **Deploy via Helm / Kustomize:** Deploy the application and mock backend services using Helm values scoped to the dynamic namespace.
5. **Dynamic DNS Ingress:** Route incoming traffic via wildcard DNS:
   `https://pr-${PR_NUMBER}.preview.example.com`
6. **PR Comment:** Post the preview URL back to the GitHub PR thread.
7. **Cleanup Trigger (`on: pull_request, types: [closed]`):** Delete namespace `kubectl delete ns preview-pr-${PR_NUMBER}`, freeing all cloud resources.

---

### **46. What is the difference between In-Tree vs Out-of-Tree CI/CD pipeline plugins and what are the security trade-offs?**
**Answer:**
- **In-Tree Plugins:** Built directly into the core CI engine. High performance and tight integration, but updates require upgrading the entire CI server.
- **Out-of-Tree / Marketplace Actions:** Third-party community plugins fetched dynamically at runtime (e.g., `uses: actions/setup-node@v4` or third-party marketplace actions).
  - *Security Risk:* Vulnerable to supply chain attacks if the action repository is compromised or unpinned.
  - *Hardening:* Always pin third-party actions to full commit SHAs (`uses: actions/setup-node@60edb5dd545a775178f525247833781be2afd1ce`) rather than mutable tags (`@v4`).

---

### **47. How do you architect a High-Availability, Fault-Tolerant Jenkins architecture on Kubernetes?**
**Answer:**
- **Stateless Agent Execution:** Jenkins Master runs on Kubernetes with persistent storage for configuration (`/var/jenkins_home` backed by EBS/EFS CSI driver).
- **Ephemeral Kubernetes Cloud Plugin:** Jenkins controller spawns dynamic agent pods per build job that terminate immediately on job completion.
- **Job Configuration as Code (JCasC):** Jenkins configuration defined 100% in YAML and stored in Git.
- **Disaster Recovery:** If Jenkins master pod dies, Kubernetes restarts it in seconds, re-mounting the persistent volume and reloading JCasC state automatically.

---

### **48. What is Chaos Testing in CI/CD pipelines and how do you implement Automated Resilience Gating?**
**Answer:**
Instead of testing only happy paths in staging, pipelines execute automated chaos experiments before production release:
1. Deploy new application version to staging.
2. Run automated synthetic load test.
3. Inject faults via **Chaos Mesh** or **LitmusChaos** (e.g., terminate 30% of backend pods, inject 200ms network packet latency, sever Redis connection).
4. Automated Pipeline Gate checks if application error rate remained $< 1\%$ and circuit breakers handled degradation gracefully.

---

### **49. What is Hermetic Build in enterprise CI/CD and why is it crucial for reproducible builds?**
**Answer:**
A **hermetic build** is executed in a completely isolated container/sandbox with **zero outbound internet access**.
- All dependencies, compilers, and toolchains are pre-fetched, cryptographically hashed, and provided locally into the build sandbox.
- **Why it matters:** Guarantees that compiling code from commit `abc1234` in 2026 will produce the exact bit-for-bit identical binary as in 2030, preventing external package registry outages or compromised upstream packages from altering builds.

---

### **50. How do you implement a secure Multi-Tenant CI/CD platform where untrusted customer code runs safely?**
**Answer:**
1. **Sandboxed Container Runtimes:** Use **gVisor** (`runsc`) or **Kata Containers** (microVMs based on QEMU/Firecracker) instead of standard runc to prevent kernel privilege escalation and container breakouts.
2. **Ephemeral Isolated Runners:** Each build executes in an isolated short-lived VM/pod created on-demand and wiped after execution.
3. **Strict Network Policies:** Block access to the Kubernetes API server, cloud metadata service (`169.254.169.254`), and internal private VPC networks.
4. **No Host Docker Socket Mounting:** Prohibit mounting `/var/run/docker.sock`. Use rootless container build tools like **Kaniko**, **Buildah**, or **Podman**.
