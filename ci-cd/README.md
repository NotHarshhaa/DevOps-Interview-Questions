# **CI/CD & GitOps - DevOps Interview Questions (200 Questions)**

Welcome to the **CI/CD & GitOps** master collection containing **200 comprehensive interview questions and detailed answers** covering Continuous Integration, Continuous Delivery, GitHub Actions, GitLab CI, Jenkins, ArgoCD, Flux, Tekton, Progressive Delivery, and Supply Chain Security.

---

## 🟢 **Part 1: CI/CD Fundamentals & Workflows (Questions 1–50)**

### **1. What is CI/CD and what core problems does it solve?**
**Answer:** CI/CD stands for Continuous Integration and Continuous Delivery/Deployment. It automates the building, testing, packaging, and deployment of software to eliminate manual human errors, reduce release cycle times from months to minutes, catch integration bugs early, and ensure code in `main` is always production-ready.

### **2. What is Continuous Integration (CI)?**
**Answer:** The practice where developers frequently commit code to a shared repository (multiple times daily). Each commit triggers automated builds, linters, and unit/integration tests to provide immediate feedback on code health.

### **3. What is Continuous Delivery (CD)?**
**Answer:** An extension of CI where code is automatically built, tested, and staged in a production-ready state. Deploying to live production requires an explicit manual business approval (e.g., clicking a button).

### **4. What is Continuous Deployment (CD)?**
**Answer:** The fully automated release practice where every commit that passes all automated pipeline tests is deployed directly into production with zero human intervention.

### **5. What are the key stages of a production CI/CD pipeline?**
**Answer:** 1. Source Trigger (webhook), 2. Static Analysis & Linting (SAST, secret scanning), 3. Build & Compilation, 4. Unit & Integration Testing, 5. Artifact Packaging & Signing (Docker, SBOM, Cosign), 6. Staging Deployment & DAST/E2E Testing, 7. Production Progressive Rollout (Canary/GitOps).

### **6. What is an Artifact in CI/CD?**
**Answer:** The compiled, packaged, immutable deployable unit produced by a build pipeline (Docker image, JAR, NPM package, Helm chart) that is promoted across environments.

### **7. What is the "Build Once, Deploy Anywhere" principle?**
**Answer:** The architectural rule that code is compiled and packaged into an immutable container/binary once during CI, and the exact same binary is deployed to Dev, Staging, and Production by injecting environment-specific configs at runtime.

### **8. What is a Build Matrix in CI/CD?**
**Answer:** A configuration that spawns multiple parallel pipeline jobs across combinations of operating systems (Ubuntu, macOS, Windows) and runtime versions (Node 18, 20, 22).

### **9. What is Pipeline Caching vs Artifact Storage?**
**Answer:** Caching stores temporary dependencies (`node_modules`, Maven caches) to speed up subsequent builds; Artifact Storage persists output binaries and compliance test reports long-term.

### **10. What is Semantic Versioning (SemVer)?**
**Answer:** Formatting release versions as `MAJOR.MINOR.PATCH` (e.g., `2.4.1`) where MAJOR indicates breaking changes, MINOR indicates backward-compatible features, and PATCH indicates bug fixes.

### **11. What are Conventional Commits?**
**Answer:** A structured commit message specification (`feat:`, `fix:`, `chore:`, `feat!:`) that enables automated tools (Semantic Release) to calculate version bumps and generate changelogs.

### **12. What is a Linter and why must it run early?**
**Answer:** A static analysis tool checking code formatting, syntax errors, and style rules without running code. Running linters first provides sub-minute feedback and saves expensive build compute minutes.

### **13. What is Secret Masking in CI/CD?**
**Answer:** Automatically detecting registered secret strings in pipeline logs and replacing them with `***` to prevent accidental credential leakage in build logs.

### **14. What is a Webhook in CI/CD?**
**Answer:** An HTTP POST callback sent from a Git repository (GitHub/GitLab) to a CI server upon events (`push`, `pull_request_opened`) to trigger immediate pipeline execution.

### **15. What are Ephemeral / Preview Environments?**
**Answer:** Short-lived, isolated environments spun up automatically when a Pull Request is opened and destroyed upon merge or closure, allowing live feature testing.

### **16. What is a Blue-Green Deployment?**
**Answer:** Running two identical production environments (Blue and Green) where live traffic is routed to Blue while Green is tested, switching traffic instantly via load balancer upon validation.

### **17. What is a Canary Deployment?**
**Answer:** Rolling out a release to a small fraction of real users (e.g., 5%), monitoring error rates and latency, and incrementally shifting traffic to 100% if healthy.

### **18. What is Dark Launching?**
**Answer:** Deploying backend code to production completely hidden behind feature flags to validate performance and database queries under live load without exposing UI features.

### **19. What is a Feature Flag (Toggle)?**
**Answer:** A conditional code branch that decouples code deployment from feature release, enabling features to be enabled/disabled instantly via an API or management UI.

### **20. What is Trunk-Based Development?**
**Answer:** A branching strategy where developers merge small, frequent commits into a single shared branch (`main`), enabling continuous integration and fast delivery.

### **21. What is GitFlow?**
**Answer:** A branching model with long-lived branches (`develop`, `feature`, `release`, `hotfix`, `master`) that often leads to merge conflicts and slow delivery cycles.

### **22. What is Static Application Security Testing (SAST)?**
**Answer:** Whitebox security testing that scans uncompiled source code for vulnerabilities (SQL injection, buffer overflows, insecure cryptography) before compilation.

### **23. What is Dynamic Application Security Testing (DAST)?**
**Answer:** Blackbox security testing that attacks a running application from the outside to discover runtime vulnerabilities, authentication bypasses, and misconfigurations.

### **24. What is Software Composition Analysis (SCA)?**
**Answer:** Scanning open-source third-party dependencies against national vulnerability databases (NVD) for known CVEs.

### **25. What is Mutation Testing in CI?**
**Answer:** Introducing small intentional bugs (mutations) into source code to verify if unit tests fail; if tests pass, test assertions are weak.

### **26. What is Concurrency Control in CI/CD?**
**Answer:** Canceling obsolete in-progress pipeline runs on pull request branches when a newer commit is pushed (`cancel-in-progress: true`), saving compute resources.

### **27. What is a Merge Queue in GitHub?**
**Answer:** An automated system that tests pull requests in an integrated sequential train against the anticipated merge result of prior queued PRs to ensure `main` never breaks.

### **28. What is Self-Hosted Runner vs Cloud-Hosted Runner?**
**Answer:** Cloud-hosted runners are fully managed VMs by GitHub/GitLab; Self-hosted runners run in private VPCs with custom hardware, GPUs, and private network access.

### **29. What is Actions Runner Controller (ARC)?**
**Answer:** A Kubernetes operator that deploys and autoscales ephemeral GitHub Actions runner pods on Kubernetes based on webhook events.

### **30. What is OIDC (OpenID Connect) in CI/CD?**
**Answer:** Federated authentication allowing CI runners to request short-lived, temporary cloud credentials (AWS STS / GCP IAM) using JWT tokens without static API keys.

### **31. What is an SBOM (Software Bill of Materials)?**
**Answer:** A formal, machine-readable nested inventory of all software packages, libraries, and transitive dependencies bundled inside a software container or binary.

### **32. What is Sigstore Cosign?**
**Answer:** An open-source tool for cryptographically signing and verifying container images in OCI registries using keyless OIDC tokens.

### **33. What is the SLSA Framework?**
**Answer:** Supply-chain Levels for Software Artifacts—a security framework defining standards for build platform isolation, non-falsifiable provenance, and source integrity.

### **34. What is Hermetic Build?**
**Answer:** A build executed in a sandboxed container with zero outbound internet access, ensuring all dependencies are pre-fetched and cryptographically hashed for 100% reproducibility.

### **35. What is Docker BuildKit layer caching?**
**Answer:** Caching intermediate container build layers in remote OCI registries, allowing ephemeral CI runners to reuse cached layers across independent builds.

### **36. What is SonarQube Quality Gate?**
**Answer:** A policy enforcing code quality thresholds (e.g., coverage $\ge 80\%$, 0 critical vulnerabilities) that blocks PR merges if not satisfied.

### **37. What is Code Coverage?**
**Answer:** The percentage of application source code executed when automated test suites run (measured via tools like JaCoCo, Istanbul, pytest-cov).

### **38. What is Pipeline as Code?**
**Answer:** Defining CI/CD workflows, build steps, and environment targets in declarative version-controlled files (`.github/workflows/*.yml`, `.gitlab-ci.yml`, `Jenkinsfile`).

### **39. What is a Monorepo CI Strategy?**
**Answer:** Using path filtering and change-dependency graph tools (Turborepo, Nx, Bazel) so commits only rebuild and test the specific microservices modified.

### **40. What is a Pull-Through Cache?**
**Answer:** A local container registry (Harbor, AWS ECR Pull Through Cache) that caches public images locally inside the VPC, eliminating external rate limits.

### **41. What is Automated Canary Analysis (ACA)?**
**Answer:** Using automated statistical telemetry queries (Prometheus, Datadog) to compare canary error rates against baseline pods and trigger automatic rollbacks on anomalies.

### **42. What is Flagger?**
**Answer:** A CNCF progressive delivery Kubernetes operator that automates canary routing and metric analysis using Istio, Linkerd, or Nginx Ingress.

### **43. What is Argo Rollouts?**
**Answer:** A Kubernetes controller providing advanced deployment capabilities (Canary, Blue-Green, experimentation) with automated metric verification.

### **44. What is GitOps?**
**Answer:** An operational model where Git repositories serve as the single source of truth for declarative infrastructure and application deployments.

### **45. What is ArgoCD?**
**Answer:** A declarative, GitOps continuous delivery tool for Kubernetes that continuously reconciles desired state in Git with live cluster state.

### **46. What is FluxCD?**
**Answer:** A modular, headless Kubernetes GitOps toolkit that automatically synchronizes cluster state from Git repositories and Helm charts.

### **47. What is Tekton?**
**Answer:** A cloud-native Kubernetes framework for building flexible, serverless CI/CD execution pipelines using Kubernetes Custom Resource Definitions (Tasks, Pipelines).

### **48. What is Spinnaker?**
**Answer:** An open-source multi-cloud continuous delivery platform developed by Netflix for managing complex, multi-stage deployment pipelines across AWS, GCP, and Kubernetes.

### **49. What is Gitleaks?**
**Answer:** A fast, open-source secret scanning tool used in pre-commit hooks and CI pipelines to detect committed API keys, tokens, and private keys.

### **50. What is Trivy?**
**Answer:** A comprehensive vulnerability scanner for container images, filesystems, Git repos, and Kubernetes configurations.

---

## 🟡 **Part 2: GitHub Actions, GitLab CI & Jenkins Deep Dive (Questions 51–100)**

### **51. What is a GitHub Actions Workflow?**
**Answer:** An automated process defined in YAML under `.github/workflows/` composed of one or more jobs triggered by events (`push`, `pull_request`, `schedule`).

### **52. What is a GitHub Actions Job vs Step?**
**Answer:** A Job is a collection of sequential steps executed on the same runner environment. Steps are individual tasks (running shell scripts or actions). Multiple jobs run in parallel by default.

### **53. What is a Reusable Workflow in GitHub Actions?**
**Answer:** A workflow triggered by `workflow_call` that can be called from other repositories to enforce standardized organization-wide compliance and deployment pipelines.

### **54. What is a Composite Action in GitHub Actions?**
**Answer:** A custom action (`action.yml`) that packages multiple shell commands and action steps into a single reusable step within a job.

### **55. How do you share files between jobs in GitHub Actions?**
**Answer:** Since jobs run on independent virtual machines, files must be uploaded as artifacts using `actions/upload-artifact` in Job A and downloaded using `actions/download-artifact` in Job B.

### **56. How do you configure OIDC with AWS in GitHub Actions?**
**Answer:** Set `permissions: { id-token: write, contents: read }`, then use `aws-actions/configure-aws-credentials@v4` with `role-to-assume` to exchange the GitHub JWT for temporary AWS IAM credentials.

### **57. What is `needs` in GitHub Actions?**
**Answer:** An attribute that defines sequential job dependencies (e.g., `needs: [build, test]` ensures the deploy job runs only after both build and test succeed).

### **58. What is `fail-fast` in GitHub Actions Matrix builds?**
**Answer:** A boolean setting under `strategy:`. If `true` (default), GitHub cancels all running matrix jobs if any single job fails. Setting `fail-fast: false` allows all matrix variations to run to completion.

### **59. What are GitHub Actions Environments and Protection Rules?**
**Answer:** Deployment targets (e.g., `production`) configured with required reviewers, wait timers, and branch protection rules that must be approved before deployment jobs execute.

### **60. How do you securely pass secrets to Reusable Workflows?**
**Answer:** Use `secrets: inherit` in the calling workflow to pass all caller secrets, or explicitly pass specific secrets via `secrets: { DB_PASSWORD: ${{ secrets.DB_PASS }} }`.

### **61. What is the `.gitlab-ci.yml` architecture?**
**Answer:** A declarative configuration file defining stages, jobs, scripts, and artifact handling natively integrated into GitLab repositories and container registries.

### **62. What are GitLab CI Stages?**
**Answer:** Sequential execution blocks (e.g., `stages: [build, test, deploy]`). All jobs within the same stage run concurrently; the next stage starts only after all jobs in the current stage succeed.

### **63. What is GitLab CI `rules` keyword?**
**Answer:** A powerful conditional syntax determining whether a job is included in the pipeline based on branch names, commit messages, file changes, or pipeline variables.

### **64. What is a GitLab Runner?**
**Answer:** An open-source application that executes pipeline jobs defined in `.gitlab-ci.yml`, supporting Docker, Kubernetes, SSH, and Shell executors.

### **65. What is GitLab Auto DevOps?**
**Answer:** A pre-configured CI/CD pipeline template that automatically detects programming languages, builds container images, executes security tests, and deploys to Kubernetes without manual pipeline coding.

### **66. What is a Declarative Jenkinsfile?**
**Answer:** A structured, syntax-checked pipeline format (`pipeline { agent any; stages { ... } }`) with built-in directives for environments, parameters, and post-build actions.

### **67. What is a Scripted Jenkinsfile?**
**Answer:** A Groovy-based procedural pipeline format (`node { ... }`) offering unlimited programmatic flexibility at the cost of higher maintenance complexity.

### **68. What is Jenkins Configuration as Code (JCasC)?**
**Answer:** Defining the entire Jenkins controller configuration (plugins, security realms, credentials, node settings) in declarative YAML files stored in Git.

### **69. What is the Jenkins Kubernetes Plugin?**
**Answer:** A plugin that allows the Jenkins controller to dynamically spawn ephemeral agent pods in a Kubernetes cluster to execute build jobs, terminating pods immediately on job completion.

### **70. What is Jenkins Shared Libraries?**
**Answer:** A centralized Git repository of reusable Groovy code and pipeline steps that can be imported and executed across multiple independent Jenkinsfiles.

### **71. How do you prevent credentials leakage in Jenkins logs?**
**Answer:** Use `withCredentials([string(credentialsId: '...', variable: 'API_TOKEN')]) { ... }` which automatically masks the secret value in console logs.

### **72. What is Jenkins Multibranch Pipeline?**
**Answer:** A Jenkins job type that automatically scans a Git repository, creates pipeline jobs for every detected branch with a `Jenkinsfile`, and deletes jobs for merged branches.

### **73. What is Blue Ocean in Jenkins?**
**Answer:** A modern, visual user interface for Jenkins designed to visualize complex multi-stage pipeline executions and failure points.

### **74. What is GitLab CI `cache` vs `artifacts`?**
**Answer:** `cache` is used to speed up subsequent runs by caching project dependencies across pipelines; `artifacts` are files passed between sequential stages within the same pipeline.

### **75. What is GitHub Actions `hashFiles()` function?**
**Answer:** Computes an MD5/SHA256 hash of matching files (e.g., `hashFiles('**/package-lock.json')`) used as a dynamic cache key to invalidate dependencies when lockfiles change.

### **76. What is GitHub Actions `workflow_dispatch`?**
**Answer:** An event trigger allowing users to trigger workflows manually from the GitHub UI or API with customizable input parameters.

### **77. What is GitHub Actions `repository_dispatch`?**
**Answer:** An HTTP webhook trigger allowing external systems (e.g., third-party webhooks, custom microservices) to trigger a GitHub Actions workflow with a JSON payload.

### **78. How do you debug GitHub Actions workflows in real time?**
**Answer:** Enable runner diagnostic logging by setting repository secrets `ACTIONS_RUNNER_DEBUG=true` and `ACTIONS_STEP_DEBUG=true`, or use tools like `tmate` for interactive SSH debugging.

### **79. What is a Jenkins Blue-Green deployment plugin?**
**Answer:** Plugins or custom pipeline scripts that orchestrate swapping load balancer target groups or updating DNS records between Blue and Green environments.

### **80. What is GitLab CI DAG (Directed Acyclic Graph) Pipeline?**
**Answer:** Using the `needs` keyword in GitLab CI to allow jobs to start immediately once their specific prerequisites complete, regardless of stage ordering.

### **81. What is GitHub Actions Concurrency Group?**
**Answer:** A named grouping (`concurrency: ${{ github.workflow }}-${{ github.ref }}`) that limits concurrent execution of workflows, queuing or canceling redundant runs.

### **82. How do you implement automated semantic releases in GitLab CI?**
**Answer:** Run the `semantic-release` NPM package in the release stage, which analyzes commit messages, creates Git tags, generates release notes, and publishes packages.

### **83. What is the Jenkins Pipeline `post` section?**
**Answer:** Directives (`always`, `success`, `failure`, `unstable`, `cleanup`) executed at the completion of a pipeline or stage to send Slack alerts or clean workspace disks.

### **84. What is a Jenkins Agent vs Controller?**
**Answer:** The Controller manages the web UI, parses pipeline scripts, and schedules builds; Agents are worker instances that execute the actual build steps.

### **85. How do you secure Jenkins Controller from malicious agents?**
**Answer:** Enable Agent-to-Controller Access Control, disable CLI access over remoting, and run build agents in isolated ephemeral containers with minimal host permissions.

### **86. What is GitHub Actions `step-security/harden-runner`?**
**Answer:** A security action that monitors outbound network traffic from GitHub runners, blocks DNS exfiltration, and detects file tampering during CI execution.

### **87. What is GitHub Actions Composite Action `using: "composite"`?**
**Answer:** The declaration in `action.yml` indicating the action is built using composite steps rather than a Docker container or JavaScript runtime.

### **88. What is GitLab CI Include keyword?**
**Answer:** Directives (`include:local`, `include:file`, `include:remote`, `include:template`) allowing pipelines to modularize and import external YAML files.

### **89. What is GitHub Actions `runner.temp` vs `runner.workspace`?**
**Answer:** `runner.temp` is an isolated temporary directory wiped after job completion; `runner.workspace` is the directory where the repository is cloned.

### **90. How do you manage Docker-in-Docker (dind) securely in GitLab CI?**
**Answer:** Use rootless Docker or TLS-enabled Docker daemons (`DOCKER_TLS_CERTDIR="/certs"`), or switch to daemonless build tools like **Kaniko**.

### **91. What is Kaniko and why is it preferred for building containers in Kubernetes?**
**Answer:** A Google open-source tool that builds container images from a Dockerfile inside a Kubernetes pod **without requiring a Docker daemon or privileged host root access**.

### **92. What is Buildah?**
**Answer:** A command-line tool for building OCI container images without requiring a background container daemon or root privileges.

### **93. What is GitHub Actions Token (`GITHUB_TOKEN`)?**
**Answer:** An automatically generated, short-lived secret token provided to each workflow run with scoped permissions defined via the `permissions:` block.

### **94. What is GitHub Actions Least-Privilege Permissions?**
**Answer:** Explicitly defining `permissions: { contents: read, id-token: write }` at the top of workflows to override overly permissive default repository settings.

### **95. How do you run scheduled cron jobs in GitHub Actions?**
**Answer:** Using the `schedule` trigger: `on: schedule: - cron: '0 2 * * *'` (runs daily at 2:00 AM UTC).

### **96. What is GitLab CI Environment Variables Hierarchy?**
**Answer:** Variables are resolved with precedence: Project Variables $>$ Group Variables $>$ Instance Variables $>$ Pipeline Variables $>$ YAML Variables.

### **97. What is Jenkins Pipeline Syntax Generator?**
**Answer:** A built-in web tool (`/pipeline-syntax`) that generates accurate Groovy code snippets for specific plugins and steps.

### **98. What is SonarQube Scanner in CI?**
**Answer:** A CLI tool that executes static code analysis, uploads AST reports to the SonarQube server, and awaits Quality Gate webhook evaluation.

### **99. What is GitLab Review Apps?**
**Answer:** Ephemeral dynamic environments spun up automatically per branch in GitLab CI, integrated directly with merge request review pages.

### **100. What is GitHub Actions `continue-on-error`?**
**Answer:** A step-level attribute that prevents a job from failing if a non-critical step (e.g., an experimental linter) returns a non-zero exit code.

---

## 🔴 **Part 3: GitOps, Progressive Delivery & Supply Chain Security (Questions 101–200)**

### **101. What is GitOps?**
**Answer:** An operational model where Git repositories serve as the single source of truth for declarative infrastructure and application deployments, using automated pull-based reconciliation operators.

### **102. What are the Four Principles of GitOps (OpenGitOps)?**
**Answer:** 1. Declarative Desired State, 2. Versioned and Immutable Storage in Git, 3. Pulled Automatically by In-Cluster Agents, 4. Continuously Reconciled with Automated Drift Correction.

### **103. What is ArgoCD?**
**Answer:** A declarative GitOps continuous delivery operator for Kubernetes that continuously monitors Git repositories and synchronizes live cluster state with desired state.

### **104. What is the ArgoCD Application CRD?**
**Answer:** A Kubernetes Custom Resource defining the source repository, path, target cluster, target namespace, and sync policies for an application.

### **105. What is an ArgoCD ApplicationSet?**
**Answer:** A controller that automates the generation and multi-cluster deployment of multiple ArgoCD `Application` resources using List, Cluster, Git Directory, or Matrix generators.

### **106. What are ArgoCD Sync Waves?**
**Answer:** Annotations (`argocd.argoproj.io/sync-wave: "1"`) that control the exact numerical execution order of Kubernetes resources during synchronization (lower/negative numbers apply first).

### **107. What are ArgoCD Sync Phases?**
**Answer:** PreSync $\rightarrow$ Sync $\rightarrow$ PostSync $\rightarrow$ SyncFail. Allows running prerequisite jobs (database migrations) before updating application deployments.

### **108. What is ArgoCD Auto-Sync and Self-Healing?**
**Answer:** Auto-Sync automatically applies new Git commits to the cluster; Self-Healing detects manual out-of-band cluster edits and overwrites them back to the Git source of truth.

### **109. What is ArgoCD Prune?**
**Answer:** An automated synchronization setting that deletes Kubernetes resources from the cluster when their corresponding manifest files are removed from Git.

### **110. What is FluxCD?**
**Answer:** A set of continuous and progressive delivery controllers for Kubernetes (Source Controller, Kustomize Controller, Helm Controller) implementing GitOps.

### **111. What is FluxCD Kustomization CRD?**
**Answer:** A resource defining a pipeline for applying Kustomize overlays from a Git repository to a target Kubernetes cluster with health checking and automated rollback.

### **112. What is FluxCD HelmRelease?**
**Answer:** A declarative resource that manages the lifecycle of Helm chart releases, automatically pulling charts from OCI or HTTP repositories and reconciling values.

### **113. What is Progressive Delivery?**
**Answer:** Advanced continuous delivery combining canary deployments, traffic routing, feature flags, and automated metric analysis to minimize blast radius during releases.

### **114. What is Argo Rollouts?**
**Answer:** A Kubernetes operator replacing standard Deployments with a `Rollout` CRD that coordinates canary traffic splitting and automated metric analysis.

### **115. What is an AnalysisTemplate in Argo Rollouts?**
**Answer:** A declarative template defining Prometheus, Datadog, or CloudWatch queries used to evaluate canary health during progressive rollouts.

### **116. What is Flagger?**
**Answer:** A progressive delivery operator that automates canary releases, A/B testing, and blue-green deployments on Kubernetes using Service Meshes (Istio, Linkerd) and Ingress controllers.

### **117. What is Software Supply Chain Security?**
**Answer:** Protecting against unauthorized modifications, malicious dependency injections, and compromised build systems across the entire software development lifecycle.

### **118. What is the SLSA Framework (Levels 1–3)?**
**Answer:** Level 1: Scripted build generating provenance; Level 2: Hosted build service with source integrity; Level 3: Isolated, ephemeral, hermetic build platform with cryptographically signed, non-falsifiable provenance.

### **119. What is Keyless Signing with Sigstore Cosign?**
**Answer:** Signing container images using short-lived OIDC tokens exchanged for X.509 certificates from Fulcio, recording signatures in the Rekor transparency log.

### **120. What is Rekor in Sigstore?**
**Answer:** An immutable, tamper-evident, append-only transparency log that records signed artifact metadata and proof of provenance.

### **121. What is Fulcio in Sigstore?**
**Answer:** A free, public Certificate Authority that issues short-lived (10-minute) X.509 certificates bound to OpenID Connect identities (e.g., GitHub Actions workflows).

### **122. What is Syft?**
**Answer:** An open-source CLI tool and library for generating Software Bill of Materials (SBOMs) from container images, filesystems, and archives in CycloneDX and SPDX formats.

### **123. What is Grype?**
**Answer:** An open-source vulnerability scanner specifically designed to scan container images and SBOM files for known security vulnerabilities.

### **124. What is In-Toto?**
**Answer:** A framework for cryptographic verification of software supply chain integrity, ensuring every step from commit to build to packaging was performed by authorized actors.

### **125. What is Kyverno Image Verification?**
**Answer:** An admission policy rule that verifies container image cryptographic signatures against Sigstore Cosign before allowing pods to schedule in Kubernetes.

### **126. What is Bitnami Sealed Secrets?**
**Answer:** A GitOps secret management tool where secrets are encrypted client-side with a public key and committed to Git, decrypted inside the cluster by a controller possessing the private key.

### **127. What is External Secrets Operator (ESO)?**
**Answer:** A Kubernetes operator that synchronizes secrets from enterprise vaults (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault) into native Kubernetes `Secret` resources.

### **128. What is HashiCorp Vault Agent Sidecar Injector?**
**Answer:** A Kubernetes mutating webhook that injects a Vault agent container into application pods to dynamically fetch and render secrets into an in-memory volume.

### **129. What is Atlantis for Terraform?**
**Answer:** An open-source application that executes `terraform plan` and `terraform apply` directly inside GitHub/GitLab Pull Request comments with automated state locking.

### **130. What is Spacelift?**
**Answer:** A specialized CI/CD management platform for Infrastructure as Code (Terraform, OpenTofu, Pulumi, CloudFormation, Kubernetes) with policy enforcement via OPA.

### **131. What is GitHub Actions Runner Controller (ARC) Autoscaling?**
**Answer:** Autoscaling runner pods dynamically based on GitHub API metrics (`workflow_job` queue depth) from 0 to hundreds of instances.

### **132. What is Ephemeral Runner Security?**
**Answer:** Destroying self-hosted runner pods/VMs immediately after completing a single job to prevent lateral movement, credential theft, and state pollution.

### **133. What is Snyk?**
**Answer:** A commercial developer security platform scanning source code (SAST), dependencies (SCA), container images, and IaC templates for security vulnerabilities.

### **134. What is SonarQube?**
**Answer:** A continuous code quality platform evaluating code coverage, duplication, complexity, code smells, and security vulnerabilities.

### **135. What is OWASP ZAP in CI/CD?**
**Answer:** An open-source web application security scanner used in CI pipelines to execute automated Dynamic Application Security Testing (DAST).

### **136. What is Checkov?**
**Answer:** A static analysis tool for IaC that scans Terraform, Kubernetes manifests, Helm charts, and Dockerfiles for security misconfigurations.

### **137. What is tfsec?**
**Answer:** A fast static analysis security scanner for Terraform code, now integrated into Trivy.

### **138. What is Infracost?**
**Answer:** A FinOps tool that parses Terraform code in pull requests to calculate monthly cloud cost impact before merging code.

### **139. What is Kustomize in GitOps?**
**Answer:** A template-free configuration manager that customizes Kubernetes YAML manifests using declarative overlays (dev, staging, prod) over a base configuration.

### **140. What is Helm in GitOps?**
**Answer:** A package manager for Kubernetes that bundles related manifests into reusable Charts, parameterized using `values.yaml` files.

### **141. What is Helmfile?**
**Answer:** A declarative spec for deploying multiple Helm charts across multiple Kubernetes clusters in dependency order.

### **142. What is Tekton Pipelines?**
**Answer:** A Kubernetes-native CRD defining a Directed Acyclic Graph (DAG) of Tasks executed sequentially or in parallel inside ephemeral pods.

### **143. What is Spinnaker Automated Canary Analysis (Kayenta)?**
**Answer:** An automated statistical analysis engine comparing canary metrics against baseline metrics over time to make automated promotion decisions.

### **144. What is Chaos Mesh in CI/CD?**
**Answer:** Injecting automated network delays, pod failures, and disk stress into staging environments during CI pipeline execution to validate resilience.

### **145. What is Semantic Release?**
**Answer:** An automated tool that analyzes commit messages to determine the next SemVer version, generates changelogs, creates Git tags, and publishes releases.

### **146. What is Release Please?**
**Answer:** A Google tool that generates Release PRs containing updated changelogs and version bumps based on conventional commit history.

### **147. What is Dependabot / Renovate?**
**Answer:** Automated dependency update bots that scan repository manifests, check for new package releases, and automatically open PRs with changelog summaries.

### **148. What is Renovate Bot?**
**Answer:** A highly configurable, multi-platform dependency update tool supporting automated merging of non-breaking security patches.

### **149. What is a Pull Request Builder Job?**
**Answer:** An automated CI job that triggers when a PR is created or updated to compile code, run tests, and report status checks back to the PR.

### **150. What is a GitOps Out-of-Sync State?**
**Answer:** A condition where the live state of Kubernetes cluster resources differs from the declared configuration stored in Git.

### **151. What is GitOps Drift Correction?**
**Answer:** The automatic overwrite of unauthorized out-of-band manual changes in a cluster back to the version declared in Git.

### **152. What is an ArgoCD PreSync Hook?**
**Answer:** A Kubernetes resource (Job) executed before any other deployment resources are applied, commonly used for database schema migrations.

### **153. What is an ArgoCD PostSync Hook?**
**Answer:** A script or notification Job executed only after all deployment resources have successfully become healthy in the cluster.

### **154. What is an ArgoCD SyncFail Hook?**
**Answer:** A remediation Job executed when an ArgoCD synchronization operation fails.

### **155. What is Secret Masking Bypass Risk?**
**Answer:** If secrets are base64-encoded or split across multiple strings, CI engines will fail to match the secret string and print it in plain text.

### **156. What is Git Credential Helper in CI?**
**Answer:** A utility allowing CI/CD runners to authenticate with remote Git repositories using short-lived OAuth tokens instead of hardcoded passwords.

### **157. What is GitHub Actions Reusable Workflow Inheritance?**
**Answer:** Passing all secrets and context from a caller workflow to a reusable workflow using `secrets: inherit`.

### **158. What is Container Layer Squashing?**
**Answer:** Merging all intermediate build layers of a container into a single layer to reduce image size and discard temporary files.

### **159. What is Multi-Arch Container Building?**
**Answer:** Using Docker `buildx` to compile container images for multiple CPU architectures (`linux/amd64`, `linux/arm64`) from a single Dockerfile.

### **160. What is OCI (Open Container Initiative)?**
**Answer:** An open governance industry standard defining specifications for container image formats (Image Spec) and runtimes (Runtime Spec).

### **161. What is Cosign Attestation?**
**Answer:** Cryptographically signing metadata predicates (SBOMs, test results, vulnerability scan reports) and attaching them to the container image in the registry.

### **162. What is Kyverno ClusterPolicy vs Policy?**
**Answer:** `ClusterPolicy` applies to all resources across the entire cluster; `Policy` is scoped strictly to a single namespace.

### **163. What is OPA Gatekeeper ConstraintTemplate?**
**Answer:** A Custom Resource defining the declarative Rego logic for a policy, instantiated by `Constraint` CRDs.

### **164. What is a Continuous Integration Feedback Loop?**
**Answer:** The time elapsed from a developer pushing a commit to receiving automated test results; must be $< 10$ minutes to maintain high velocity.

### **165. What is Monorepo Incremental Building?**
**Answer:** Using cached compilation outputs so that changing one file only recompiles that file and its direct dependents.

### **166. What is Bazel?**
**Answer:** A fast, scalable, multi-language build system developed by Google that enforces hermetic, reproducible builds and aggressive caching.

### **167. What is Turborepo?**
**Answer:** A high-performance build system for JavaScript/TypeScript monorepos that caches build and test execution outputs remotely.

### **168. What is Nx?**
**Answer:** A smart, extensible build framework with advanced dependency graph visualization and distributed task execution for monorepos.

### **169. What is a Git Pre-Commit Hook?**
**Answer:** A client-side script executed automatically before `git commit` runs, used to lint code, format files, and check for committed secrets locally.

### **170. What is `pre-commit` framework?**
**Answer:** A multi-language package manager for managing and maintaining pre-commit hooks via a declarative `.pre-commit-config.yaml` file.

### **171. What is Trunk-Based Feature Flag Lifecycle?**
**Answer:** 1. Create flag $\rightarrow$ 2. Implement logic behind flag $\rightarrow$ 3. Merge to `main` $\rightarrow$ 4. Enable flag in production $\rightarrow$ 5. Clean up flag conditional code.

### **172. What is Dark Launching vs Shadowing?**
**Answer:** Dark launching deploys backend code with no UI changes; Shadowing duplicates live incoming HTTP traffic and replays it against the new version.

### **173. What is an Artifact Provenance Document?**
**Answer:** Cryptographically signed metadata recording exactly who built the artifact, from which commit SHA, on which CI runner, and using which build parameters.

### **174. What is CycloneDX?**
**Answer:** An OWASP-backed, lightweight Software Bill of Materials (SBOM) standard designed for application security and vulnerability analysis.

### **175. What is SPDX?**
**Answer:** An open standard (ISO/IEC 5962:2021) for communicating Software Bill of Materials data, including components, licenses, and copyrights.

### **176. What is a Vulnerability Exploitability eXchange (VEX)?**
**Answer:** A machine-readable companion to an SBOM that declares whether a specific CVE in a dependency is actually exploitable in the context of the application.

### **177. What is Supply Chain Security SLSA Provenance?**
**Answer:** An in-toto attestation generated during the build step certifying the source repository, commit, and build environment.

### **178. What is Docker Content Trust (DCT)?**
**Answer:** A legacy Docker feature using Notary and digital signatures to verify the integrity and publisher of specific image tags.

### **179. What is a Pipeline Deadlock?**
**Answer:** A state where two or more pipeline jobs wait indefinitely on mutually dependent resources or locks (e.g., job A holds lock 1 waiting for job B, while job B holds lock 2 waiting for job A).

### **180. What is CI/CD Pipeline Blast Radius?**
**Answer:** The maximum potential damage caused if a CI/CD system is compromised (mitigated by using pull-based GitOps and ephemeral runners).

### **181. What is Automated Rollback in Argo Rollouts?**
**Answer:** Automatically aborting a canary deployment and reverting traffic to stable pods when an AnalysisTemplate metric breach occurs.

### **182. What is a Canary Step Weight?**
**Answer:** The percentage of total traffic allocated to the canary version during a specific phase of a progressive rollout (e.g., `setWeight: 20`).

### **183. What is a Flagger Metric Template?**
**Answer:** A Custom Resource defining Prometheus PromQL queries (e.g., error rate $< 1\%$, latency $< 500\text{ms}$) checked during canary analysis.

### **184. What is Blue-Green Cutover Latency?**
**Answer:** The time taken for load balancers or DNS to update routing rules from Blue to Green; with Kubernetes Services, this occurs in sub-seconds.

### **185. What is a Pipeline Concurrency Lock?**
**Answer:** Restricting deployment pipelines to execute one at a time per target environment to prevent concurrent overlapping state modifications.

### **186. What is a GitOps Sync Window?**
**Answer:** Scheduled maintenance windows in ArgoCD that allow or deny automated synchronizations during specific hours (e.g., blocking prod syncs on weekends).

### **187. What is ArgoCD SSO (Single Sign-On)?**
**Answer:** Integrating ArgoCD with enterprise identity providers (Okta, Azure AD, GitHub) via OIDC or SAML to enforce role-based access control.

### **188. What is FluxCD Source Controller?**
**Answer:** A dedicated Flux controller that watches Git repositories, Helm charts, and S3 buckets for changes and produces artifact archives for other controllers.

### **189. What is FluxCD Kustomize Controller?**
**Answer:** A Flux controller that takes artifacts from the Source Controller, generates Kubernetes manifests via Kustomize, and applies them to the cluster.

### **190. What is FluxCD Notification Controller?**
**Answer:** A Flux controller that handles inbound webhooks from Git providers and dispatches outbound event notifications to Slack, MS Teams, and Discord.

### **191. What is Tekton PipelineRun?**
**Answer:** A Custom Resource that instantiates and executes a `Pipeline` on a Kubernetes cluster, binding concrete parameters and workspaces.

### **192. What is Tekton TaskRun?**
**Answer:** A Custom Resource that instantiates and executes a single `Task` inside a Kubernetes pod composed of sequential container steps.

### **193. What is a Matrix Job in GitLab CI?**
**Answer:** Using the `parallel: matrix:` keyword in `.gitlab-ci.yml` to run multiple job variations across defined variables concurrently.

### **194. What is a Pipeline Secret Store CSI Driver?**
**Answer:** Mounting secrets stored in enterprise vaults directly into Kubernetes pods as in-memory files without persisting them as Kubernetes Secret objects.

### **195. What is a Zero-Downtime Database Migration sequence in CI/CD?**
**Answer:** 1. Add nullable columns (Expand), 2. Deploy app writing to both old and new columns, 3. Backfill data, 4. Deploy app reading from new column, 5. Drop old columns (Contract).

### **196. What is In-Tree vs Out-of-Tree CI Plugins?**
**Answer:** In-tree plugins are compiled directly into the core CI engine; Out-of-tree plugins are external community actions downloaded dynamically at runtime.

### **197. What is Action Pinning in GitHub Actions?**
**Answer:** Pinning third-party actions to full commit SHAs (`uses: actions/setup-node@60edb5...`) rather than mutable tags (`@v4`) to protect against supply chain attacks.

### **198. What is Hermetic Container Building with Kaniko?**
**Answer:** Executing container image builds inside an isolated pod with no host Docker socket mount, pushing directly to an OCI registry.

### **199. What is a Pipeline Flakiness Rate?**
**Answer:** The percentage of pipeline runs that fail due to unstable tests or network timeouts rather than actual code defects, eroding developer trust.

### **200. What is Progressive Delivery Automated Metric Gating?**
**Answer:** Evaluating real-time Prometheus golden signal telemetry during canary deployments to autonomously promote or abort releases without human intervention.
