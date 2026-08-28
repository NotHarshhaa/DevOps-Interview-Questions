# **Infrastructure as Code (IaC) - DevOps Interview Questions**

Welcome to the **Infrastructure as Code (IaC)** interview questions master guide. This module provides in-depth, exhaustive technical explanations, complete HCL/YAML configurations, state management internals, drift detection pipelines, and scenario-based interview discussions covering Terraform, OpenTofu, Terragrunt, Crossplane, Pulumi, and Ansible.

---

## 🟢 **Beginner Level (Questions 1–20)**

### **1. What is Infrastructure as Code (IaC), what are its core principles, and how does it eliminate operational anti-patterns like "Snowflake Servers"?**

**Detailed Answer:**
**Infrastructure as Code (IaC)** is the engineering practice of provisioning, configuring, managing, and versioning compute, storage, networking, and cloud services using machine-readable definition files (HCL, YAML, JSON, Python, TypeScript) rather than manual GUI clicks or ad-hoc CLI commands.

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                              INFRASTRUCTURE AS CODE LIFECYCLE                           │
│                                                                                         │
│  [ Git Commit ] ──► [ Automated CI Linting ] ──► [ Plan Diff Validation ] ──► [ Apply ] │
│   (HCL Code)          (Checkov, tfsec)             (Infracost Cost & OPA)       (To Cloud)│
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

#### **How IaC Eliminates Operational Anti-Patterns:**
- **"Snowflake Servers" & Configuration Drift:** In manual administration, individual servers accumulate undocumented hotfixes, differing package versions, and unique manual settings over time, making them impossible to replicate. IaC ensures every server and cloud resource is provisioned from the exact same version-controlled template.
- **Environment Inconsistencies:** Spin up identical Dev, Staging, and Production environments in minutes with 100% parity.
- **Disaster Recovery:** If an entire cloud region goes offline, thousands of cloud resources can be recreated programmatically in a replacement region by running a single pipeline.

---

### **2. Compare Declarative vs Imperative IaC with concrete code examples.**

**Detailed Answer:**

#### **1. Declarative IaC (e.g., Terraform, OpenTofu, CloudFormation, Kubernetes YAML):**
- **Philosophy:** Declares **WHAT** the eventual infrastructure state should be.
- The IaC engine computes dependency graphs and handles all CRUD operations to reach the declared state.

```hcl
# Declarative Terraform / OpenTofu
resource "aws_s3_bucket" "logs" {
  bucket = "company-audit-logs-prod"
}

resource "aws_s3_bucket_versioning" "versioning" {
  bucket = aws_s3_bucket.logs.id
  versioning_configuration {
    status = "Enabled"
  }
}
```

#### **2. Imperative IaC (e.g., AWS CLI, Bash scripts, Python Boto3, Chef):**
- **Philosophy:** Specifies **HOW** to reach the state via a sequential series of explicit commands.

```bash
# Imperative AWS CLI
aws s3api create-bucket --bucket company-audit-logs-prod --region us-east-1
aws s3api put-bucket-versioning --bucket company-audit-logs-prod --versioning-configuration Status=Enabled
```
*Difference on Re-run:* Declarative checks state and reports "No changes needed". Imperative fails with an error `BucketAlreadyOwnedByYou` unless complex manual error handling is coded.

---

### **3. What is Terraform State and why is it necessary?**

**Detailed Answer:**
The Terraform state file (`terraform.tfstate`) is a JSON document that maps your declarative HCL configuration to real-world cloud resource IDs and attributes.

#### **Core Functions of State:**
1. **Resource Mapping:** Maps human-readable HCL identifiers (`aws_instance.web`) to cloud provider resource IDs (`i-0a1b2c3d4e5f`).
2. **Metadata & Dependency Tracking:** Tracks resource dependencies and ordering to safely destroy or update resources in DAG order.
3. **Performance Caching:** Caches resource attributes locally to avoid querying hundreds of slow cloud APIs during every `terraform plan`.

---

### **4. Explain the core Terraform CLI workflow commands.**

**Detailed Answer:**
1. `terraform init`: Initializes the directory, downloads required provider plugins and modules, and configures the backend.
2. `terraform validate`: Validates syntax and internal consistency of configuration files without making cloud API calls.
3. `terraform plan`: Computes the execution plan by comparing declared code against remote state and live infrastructure.
4. `terraform apply`: Applies the changes required to reach the desired state.
5. `terraform destroy`: Destroys all remote objects managed by the current state file.

---

### **5. Compare Terraform `count` vs `for_each`.**

**Detailed Answer:**
- **`count`:** Creates resources indexed by an integer list (`[0, 1, 2]`).
  - *Major Flaw:* Removing an item from the middle of the list causes Terraform to reindex and force-recreate all subsequent resources.
- **`for_each`:** Creates resources keyed by unique map or set strings (`["web", "api", "db"]`).
  - *Advantage:* Removing or adding an item affects *only* that specific key, preserving all other resources without unwanted destruction.

---

### **6. What is a Terraform Module and how does it enforce DRY principles?**

**Detailed Answer:**
A Terraform module is a container for multiple resources used together.
- **Root Module:** The working directory containing your primary `.tf` files.
- **Child Module:** Reusable components called from other configurations (`module "vpc" { source = "./modules/vpc" }`).
- **Benefits:** Enforces DRY (Don't Repeat Yourself) patterns, standardized enterprise security baselines, and shared infrastructure patterns across teams.

---

### **7. Compare Terraform Variables, Locals, and Outputs.**

**Detailed Answer:**
- **Input Variables (`variable`):** Parameters passed into modules to customize behavior (like function arguments).
- **Local Values (`locals`):** Internal reusable constants or computed expressions scoped strictly to the current module.
- **Output Values (`output`):** Return values exposed by a module to the root or CLI stdout (like function return values).

---

### **8. What is a Remote Backend and why must teams use State Locking?**

**Detailed Answer:**
Storing state locally (`terraform.tfstate`) causes state loss and concurrency conflicts when multiple engineers run applies simultaneously.
- **Remote Backend with State Locking (e.g., AWS S3 + DynamoDB / Terraform Cloud):**
  - Encrypts state files at rest in S3 with KMS.
  - DynamoDB distributed locking prevents race conditions, ensuring only one engineer or pipeline can execute changes at a time.

---

### **9. Compare Terraform vs Ansible.**

**Detailed Answer:**
- **Terraform / OpenTofu (Infrastructure Provisioning):** Declarative state management for provisioning cloud resources (VPCs, EKS, RDS, IAM).
- **Ansible (Configuration Management):** Agentless procedural/declarative execution for configuring operating systems, installing packages, and managing software configs over SSH.

---

### **10. What is OpenTofu and why did the open-source community fork Terraform?**

**Detailed Answer:**
In August 2023, HashiCorp switched Terraform's license from open-source MPL 2.0 to the Business Source License (BSL 1.1). In response, the Linux Foundation and community created **OpenTofu** as a 100% open-source, community-governed, backward-compatible fork of Terraform.

---

### **11. What is Configuration Drift in IaC and how is it resolved?**

**Detailed Answer:**
Drift is the discrepancy between declared code in Git and actual live cloud resources caused by manual console edits. Resolved by running scheduled automated drift detection in CI/CD and re-applying Terraform to overwrite unmanaged changes.

---

### **12. What is `terraform refresh`?**

**Detailed Answer:**
Reconciles the state file with real-world resources by querying cloud APIs and updating the state file without modifying live infrastructure. *(Note: `terraform plan` runs an automatic refresh by default).*

---

### **13. What are Terraform Data Sources?**

**Detailed Answer:**
Data sources allow Terraform to query and read read-only attributes from external cloud resources defined outside the current configuration or in separate state files (e.g., querying the latest official Ubuntu AMI ID).

---

### **14. Compare Terraform Workspaces vs Directory-Based Environments.**

**Detailed Answer:**
- **Workspaces:** Manages multiple state files from a single directory (`terraform workspace select staging`). Good for ephemeral feature branches, but risky for production because configurations cannot vary across environments.
- **Directory-Based (or Terragrunt):** Dedicated directories per environment (`environments/prod/`, `environments/staging/`). Preferred for production isolation and independent versioning.

---

### **15. Compare `terraform taint` vs `terraform apply -replace`.**

**Detailed Answer:**
- **`terraform taint` (Legacy):** Marks a resource as degraded in the state file so it will be recreated on the next apply.
- **`-replace` (Modern Best Practice):** Replaces a resource in a single step without modifying state beforehand: `terraform apply -replace="aws_instance.web"`.

---

### **16. What is a Terraform Provider?**

**Detailed Answer:**
A plugin binary (written in Go) distributed via the registry that implements CRUD operations against a specific cloud or service API (AWS, Azure, GCP, Kubernetes, Datadog, GitHub).

---

### **17. What is Pulumi and how does it differ from Terraform?**

**Detailed Answer:**
Pulumi allows defining cloud infrastructure using real programming languages (**TypeScript, Python, Go, C#**) instead of domain-specific languages (HCL), providing full language power (loops, classes, standard unit test frameworks).

---

### **18. What is Crossplane?**

**Detailed Answer:**
A CNCF open-source project that turns Kubernetes into a universal cloud control plane. Cloud resources (S3, RDS) are defined as Kubernetes CRDs and continuously reconciled by Kubernetes controllers.

---

### **19. Compare Checkov vs tfsec vs Trivy for IaC security.**

**Detailed Answer:**
Static security analysis tools scanning Terraform, CloudFormation, and Helm manifests in CI pipelines to block security misconfigurations (unencrypted storage, public S3 buckets, open security groups) before deployment.

---

### **20. What is `terraform fmt`?**

**Detailed Answer:**
A built-in command that automatically formats all `.tf` files in the directory according to canonical Terraform style conventions (indentation, alignment).

---

## 🟡 **Intermediate Level (Questions 21–40)**

### **21. What is the `moved` block in Terraform and how does it prevent resource destruction during refactoring?**

**Detailed Answer:**
Prior to Terraform 1.1, renaming a resource or moving it into a child module caused Terraform to destroy the old resource and create a new one, causing catastrophic production database downtime.
- **`moved` Block Solution:**
```hcl
moved {
  from = aws_instance.web
  to   = module.compute.aws_instance.web
}
```
Terraform updates the internal state address without touching or destroying the live cloud resource.

---

### **22. What is the `import {}` block in Terraform 1.5+?**

**Detailed Answer:**
Allows importing existing cloud resources natively in code:
```hcl
import {
  to = aws_s3_bucket.logs
  id = "company-audit-logs-prod"
}

resource "aws_s3_bucket" "logs" {
  bucket = "company-audit-logs-prod"
}
```
Running `terraform plan` previews the import and verifies that the code matches the live cloud resource before applying.

---

### **23. Explain Terraform Custom Preconditions and Postconditions.**

**Detailed Answer:**
Enforces custom validation rules within resource lifecycles:
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type

  lifecycle {
    precondition {
      condition     = data.aws_ami.ubuntu.architecture == "x86_64"
      error_message = "Selected AMI must be x86_64 architecture."
    }
    postcondition {
      condition     = self.associate_public_ip_address == false
      error_message = "Web instances must not have a public IP address."
    }
  }
}
```

---

### **24. What is Terragrunt and what problems does it solve?**

**Detailed Answer:**
A thin wrapper for Terraform/OpenTofu providing:
- **DRY Remote State:** Defines backend configurations once in root `terragrunt.hcl` and inherits it across all subdirectories.
- **Dependency Management:** Automatically executes modules in DAG dependency order (`terragrunt run-all apply`).
- **Mock Outputs:** Supplies mock outputs during `plan` for un-deployed upstream modules.

---

### **25. How do Terraform Dynamic Blocks work?**

**Detailed Answer:**
Constructs repeatable nested configuration blocks dynamically based on a collection:
```hcl
resource "aws_security_group" "web_sg" {
  name = "web-sg"
  dynamic "ingress" {
    for_each = var.ingress_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["10.0.0.0/16"]
    }
  }
}
```

---

### **26. Compare `.terraform.lock.hcl` vs `.tfstate`.**

**Detailed Answer:**
- **`.terraform.lock.hcl` (Dependency Lock File):** Records exact provider plugin versions and cryptographic hashes. Committed to Git.
- **`.tfstate` (State File):** Records live cloud resource IDs and attributes. Never committed to Git; stored in an encrypted remote backend.

---

### **27. How do you protect sensitive values in Terraform outputs and state?**

**Detailed Answer:**
Marking outputs as `sensitive = true` prevents values from displaying in CLI stdout or CI logs.
*Crucial Caveat:* `sensitive = true` does **not** encrypt values inside `.tfstate`. State files must be stored in KMS-encrypted S3 backends with restricted IAM access.

---

### **28. What is Terraform Drift Detection automation?**

**Detailed Answer:**
Scheduled CI pipeline executing `terraform plan -detailed-exitcode`:
- Exit Code 0 = No drift.
- Exit Code 1 = Error.
- Exit Code 2 = Drift detected $\rightarrow$ triggers PagerDuty alert or auto-reconciliation.

---

### **29. What is Terratest and how is it used for IaC testing?**

**Detailed Answer:**
A Go library developed by Gruntwork that provisions real infrastructure, runs integration tests against live endpoints, and automatically tears everything down (`defer terraform.Destroy(...)`).

---

### **30. How do Crossplane Compositions and XRDs work?**

**Detailed Answer:**
1. **XRD (Composite Resource Definition):** Platform team defines an abstract API (`kind: PostgreSQLInstance`).
2. **Composition:** Binds the abstract XRD to concrete cloud resources (RDS Instance, Subnet Group, Security Group, KMS Key).
3. **Claim:** Developers request a database via a simple 5-line Kubernetes manifest without knowing AWS details.

---

### **31. What is the `ignore_changes` lifecycle rule in Terraform?**

**Detailed Answer:**
Tells Terraform to ignore updates to specific resource attributes caused by external systems or autoscaling (e.g., `lifecycle { ignore_changes = [desired_capacity] }` on an Auto Scaling Group).

---

### **32. What is Terraform Provider Alias?**

**Detailed Answer:**
Allows configuring multiple instances of the same provider (e.g., deploying resources across multiple AWS regions or accounts in a single Terraform run) using `alias = "west"`.

---

### **33. What is Ansible Idempotency and how is it enforced?**

**Detailed Answer:**
A task is idempotent if running it multiple times produces the exact same result. For `command`/`shell` modules, enforce idempotency using `creates` or `removes` arguments.

---

### **34. What is Ansible Vault?**

**Detailed Answer:**
Encrypts sensitive variables, files, or playbooks with AES-256 (`ansible-vault encrypt secrets.yml`).

---

### **35. Compare Ansible Roles vs Collections.**

**Detailed Answer:**
- **Role:** Structured directory packaging reusable tasks, handlers, variables, and templates.
- **Collection:** Modern distribution format packaging multiple roles, custom Python modules, action plugins, and playbooks.

---

### **36. Compare `terraform state rm` vs `terraform state mv`.**

**Detailed Answer:**
- **`state rm`:** Removes a resource from state; live cloud resource is **not deleted**, Terraform simply stops managing it.
- **`state mv`:** Renames or moves a resource within state without destroying or recreating it.

---

### **37. What is an IaC Blast Radius and how do you minimize it?**

**Detailed Answer:**
Blast radius is the maximum potential damage from a failed apply. Minimized by breaking monolithic state files into layered, decoupled state files (Networking $\rightarrow$ Data $\rightarrow$ Compute $\rightarrow$ App).

---

### **38. What is Terraform Provider Mirroring?**

**Detailed Answer:**
Configuring CLI config (`.terraformrc`) to read pre-downloaded provider binaries from a local directory or private artifact repository (Artifactory) in air-gapped environments without internet access.

---

### **39. What is Atlantis for Pull Request-driven Terraform?**

**Detailed Answer:**
A self-hosted application that runs `terraform plan` on PR creation, posts the diff, locks state, and applies changes only when authorized reviewers comment `atlantis apply`.

---

### **40. What is Spacelift / Terraform Cloud?**

**Detailed Answer:**
Collaborative IaC management platforms providing remote runners, state locking, RBAC, cost estimation, OPA policy enforcement, and automated drift detection.

---

## 🔴 **Advanced & Scenario-Based Level (Questions 41–50)**

### **41. Scenario: An engineer accidentally ran `terraform destroy` against staging and cancelled it midway. How do you recover?**

**Detailed Answer:**
1. Clear stuck lock: `terraform force-unlock <LOCK-ID>`.
2. Sync state: `terraform refresh`.
3. Generate recovery plan: `terraform plan -out=recovery.plan`.
4. Apply plan to recreate destroyed resources and restore consistency: `terraform apply recovery.plan`.

---

### **42. Scenario: Refactor a monolithic 500-resource Terraform state into 5 decoupled state files with zero downtime. Walk through the migration strategy.**

**Detailed Answer:**
1. Backup production state file.
2. Create target directory structure (`01-network/`, `02-database/`, `03-compute/`).
3. Migrate resources using `terraform state mv -state=monolith.tfstate -state-out=02-database/terraform.tfstate <source> <dest>`.
4. Initialize remote S3 backends in each new directory.
5. Verify with `terraform plan` in each directory to confirm **0 to add, 0 to change, 0 to destroy**.

---

### **43. Scenario: A live production AWS S3 bucket containing 50TB of data was created manually. Bring it under Terraform management with zero data loss risk.**

**Detailed Answer:**
1. Declare resource in HCL: `resource "aws_s3_bucket" "prod_data" { bucket = "company-prod-data" }`.
2. Import resource via `import {}` block (Terraform 1.5+).
3. Run `terraform plan` and reconcile attributes until plan shows **0 to add, 0 to change, 0 to destroy**.
4. Apply to bind state.

---

### **44. How do you implement Policy as Code with OPA / Conftest to block unencrypted cloud resources in CI/CD?**

**Detailed Answer:**
1. Export plan to JSON: `terraform show -json tfplan.binary > tfplan.json`.
2. Write Rego policy checking `resource.change.after.server_side_encryption_configuration`.
3. Test in CI: `conftest test tfplan.json -p policy/` (blocks pipeline if non-compliant).

---

### **45. Compare Crossplane vs Terraform at enterprise scale.**

**Detailed Answer:**
- **Terraform:** Pipeline-driven batch execution; drift detected only on pipeline runs.
- **Crossplane:** Continuous control plane loop inside Kubernetes; continuously auto-heals drift and provides native Kubernetes CRD abstractions for developers.

---

### **46. How do you structure a production multi-environment Terragrunt repository?**

**Detailed Answer:**
```
root/
├── terragrunt.hcl               # Root config: S3 backend + provider generation
├── environments/
│   ├── dev/
│   │   ├── vpc/terragrunt.hcl
│   │   └── eks/terragrunt.hcl   # dependencies = ["../vpc"]
│   ├── staging/
│   └── prod/
└── modules/                     # Pure Terraform source modules
```

---

### **47. How do you resolve circular dependencies between Terraform modules?**

**Detailed Answer:**
1. Decompose circular resources into standalone items (e.g., create Security Groups first, define `aws_security_group_rule` in a separate step).
2. Pass resource IDs instead of entire module objects.

---

### **48. How do you implement zero-trust secrets injection in Terraform without writing credentials to disk?**

**Detailed Answer:**
Configure Terraform with the **HashiCorp Vault Dynamic Secrets Provider** (`data "vault_aws_access_credentials"`). Terraform requests short-lived (1-hour) cloud credentials generated on-demand by Vault, eliminating static keys entirely.

---

### **49. What is the Terraform Plugin Framework (v6)?**

**Detailed Answer:**
Modern Go SDK for authoring Terraform providers. Replaced legacy `plugin-sdk/v2` with native support for nested object types, custom types, structured attribute validation, and gRPC protocol buffers.

---

### **50. Scenario: An engineer accidentally committed a production `.tfstate` file containing database passwords to GitHub. Walk through the complete response procedure.**

**Detailed Answer:**
1. **Rotate All Secrets Immediately:** Rotate database master passwords, private TLS keys, and IAM credentials in the cloud provider.
2. **Purge Git History:** Use `git-filter-repo` to permanently strip `.tfstate` from all commits, branches, and tags. Force-push clean history.
3. **Move to Remote S3 Backend:** Configure remote encrypted S3 backend with DynamoDB locking.
4. **Enforce Prevention:** Add `*.tfstate` to `.gitignore` and configure GitHub Secret Scanning.
