# **Infrastructure as Code (IaC) - DevOps Interview Questions (200 Questions)**

Welcome to the **Infrastructure as Code (IaC)** master collection containing **200 comprehensive interview questions and detailed answers** covering Terraform, OpenTofu, Terragrunt, Crossplane, Pulumi, Ansible, State Management, Drift Detection, Policy as Code, and Automated Testing.

---

## 🟢 **Part 1: IaC Fundamentals & Terraform / OpenTofu Core (Questions 1–60)**

### **1. What is Infrastructure as Code (IaC) and what core problems does it solve?**
**Answer:** Infrastructure as Code (IaC) is the practice of provisioning, configuring, managing, and versioning compute, storage, and networking resources using machine-readable definition files rather than manual console clicks. It eliminates human configuration errors, eradicates "Snowflake Servers," enforces version control, and allows tearing down and recreating entire production environments in minutes.

### **2. Compare Declarative vs Imperative IaC with concrete examples.**
**Answer:**
- **Declarative (e.g., Terraform, OpenTofu, Kubernetes YAML):** Declares **WHAT** the eventual infrastructure state should be. The engine computes dependencies and executes CRUD operations to match that state (`resource "aws_s3_bucket" "b" { ... }`).
- **Imperative (e.g., AWS CLI, Bash scripts, Python Boto3):** Specifies **HOW** to reach the state via sequential commands (`aws s3api create-bucket ...`). Re-running imperative scripts fails without explicit custom idempotency logic.

### **3. What is Terraform State and why is it necessary?**
**Answer:** The Terraform state file (`terraform.tfstate`) is a JSON document that maps your declarative HCL configuration to real-world cloud resource IDs and attributes. It tracks resource dependencies, caches metadata for plan performance, and coordinates concurrent execution via state locking.

### **4. Explain the core Terraform CLI workflow commands.**
**Answer:**
1. `terraform init`: Initializes working directory, downloads provider plugins/modules, and configures remote backends.
2. `terraform validate`: Validates syntax and internal consistency of HCL files without cloud API calls.
3. `terraform plan`: Computes the execution plan by comparing code against state and live infrastructure.
4. `terraform apply`: Executes the changes required to reach desired state.
5. `terraform destroy`: Destroys all remote objects managed by the state file.

### **5. Compare Terraform `count` vs `for_each`.**
**Answer:**
- **`count`:** Creates resources indexed by an integer list (`[0, 1, 2]`). Removing an item from the middle causes Terraform to reindex and force-recreate all subsequent resources.
- **`for_each`:** Creates resources keyed by unique map or set strings (`["web", "api", "db"]`). Adding or removing an item affects *only* that specific key without destroying others.

### **6. What is a Terraform Module and how does it enforce DRY principles?**
**Answer:** A container for multiple resources used together. Modules package reusable, standardized infrastructure components (VPCs, EKS clusters) with input variables and outputs, avoiding code duplication across environments.

### **7. Compare Terraform Variables, Locals, and Outputs.**
**Answer:**
- **Variables (`variable`):** Input parameters passed into modules to customize behavior.
- **Locals (`locals`):** Internal reusable constants or computed expressions scoped strictly to the current module.
- **Outputs (`output`):** Return values exposed by a module to the root module or CLI stdout.

### **8. What is a Remote Backend and why is State Locking mandatory?**
**Answer:** A remote storage location (e.g., AWS S3 + DynamoDB / Terraform Cloud) for state files. State locking prevents race conditions and corrupted state files by ensuring only one engineer or pipeline can execute `apply` at any time.

### **9. What is OpenTofu and why was it created?**
**Answer:** In August 2023, HashiCorp switched Terraform's license from open-source MPL 2.0 to the Business Source License (BSL 1.1). OpenTofu was created by the Linux Foundation and community as a 100% open-source, backward-compatible fork of Terraform.

### **10. What is Configuration Drift in IaC and how is it resolved?**
**Answer:** Discrepancies between declared IaC code and actual live cloud resources caused by manual console edits. Resolved by running automated scheduled drift detection (`terraform plan -detailed-exitcode`) and applying code to reconcile drift.

### **11. What is the `moved` block in Terraform?**
**Answer:** Allows refactoring resources (renaming or moving into child modules) without destroying and recreating live cloud resources:
```hcl
moved {
  from = aws_instance.web
  to   = module.compute.aws_instance.web
}
```

### **12. What is the `import {}` block in Terraform 1.5+?**
**Answer:** A declarative block that imports existing cloud resources into Terraform state natively in code, generating configuration templates automatically via `terraform plan -generate-config-out=generated.tf`.

### **13. What are Terraform Custom Preconditions and Postconditions?**
**Answer:** Declarative validation rules defined inside `lifecycle` blocks that validate assertions before or after resource provisioning:
```hcl
lifecycle {
  precondition {
    condition     = data.aws_ami.ubuntu.architecture == "x86_64"
    error_message = "Selected AMI must be x86_64."
  }
}
```

### **14. What is Terragrunt and what problems does it solve?**
**Answer:** A thin wrapper for Terraform/OpenTofu providing DRY remote state backend definitions, automated module dependency execution (`terragrunt run-all apply`), and mock outputs for plan evaluations.

### **15. What are Terraform Dynamic Blocks?**
**Answer:** Constructs repeatable nested configuration blocks (e.g., multiple `ingress` rules inside an `aws_security_group`) dynamically from a list or map variable.

### **16. Compare `.terraform.lock.hcl` vs `terraform.tfstate`.**
**Answer:**
- **`.terraform.lock.hcl`:** Dependency lock file recording exact provider plugin versions and cryptographic hashes; committed to Git.
- **`terraform.tfstate`:** Records live cloud resource IDs and attributes; stored in an encrypted remote backend, never committed to Git.

### **17. How do you protect sensitive values in Terraform?**
**Answer:** Mark variables/outputs with `sensitive = true` to prevent display in CLI stdout or CI logs. *Caveat: State files still contain plain text sensitive values and must be stored in KMS-encrypted S3 backends with restricted IAM.*

### **18. What is Terratest?**
**Answer:** A Go library developed by Gruntwork that provisions real cloud infrastructure via Terraform, executes integration tests against live endpoints, and tears down resources.

### **19. What is Crossplane?**
**Answer:** A CNCF open-source project that turns Kubernetes into a universal cloud control plane, allowing platform teams to manage cloud resources (S3, RDS) as Kubernetes Custom Resources (CRDs).

### **20. What is Pulumi?**
**Answer:** An open-source IaC engine that allows defining cloud infrastructure using general-purpose programming languages (**TypeScript, Python, Go, C#**) with full IDE auto-complete and unit test support.

### **21. What is Ansible and how does its agentless architecture work?**
**Answer:** An open-source automation and configuration management tool that connects to target nodes over standard SSH (Linux) or WinRM (Windows) without installing daemons or agents on managed nodes.

### **22. What is Ansible Idempotency?**
**Answer:** The property of an Ansible playbook where executing it once or multiple times produces the exact same end state without unintended side effects.

### **23. Compare Terraform vs Ansible.**
**Answer:** Terraform is optimized for **infrastructure provisioning** (orchestrating VPCs, EKS, RDS) using declarative state; Ansible is optimized for **configuration management** (installing packages, configuring OS files) on running servers.

### **24. What is `terraform refresh`?**
**Answer:** Queries cloud APIs to update the local state file with real-world resource attributes without modifying live infrastructure.

### **25. What is a Terraform Data Source?**
**Answer:** A read-only query that fetches attributes from external cloud resources defined outside the current configuration (e.g., querying the latest official Ubuntu AMI ID).

### **26. What is `terraform taint` vs `terraform apply -replace`?**
**Answer:** `terraform taint` is legacy; `terraform apply -replace="aws_instance.web"` forces recreation of a specific resource in a single step without modifying state beforehand.

### **27. What are Terraform Workspaces?**
**Answer:** Separate state files managed from a single working directory (`terraform workspace select staging`). Best for ephemeral feature branches; directory-based layouts are preferred for production isolation.

### **28. What is `terraform fmt`?**
**Answer:** Automatically rewrites all `.tf` files in the directory to conform to canonical Terraform style and indentation standards.

### **29. What is `terraform graph`?**
**Answer:** Generates a visual Directed Acyclic Graph (DAG) of all resource dependencies in DOT format for graphviz visualization.

### **30. What is Checkov?**
**Answer:** A static code analysis tool scanning Terraform, CloudFormation, and Kubernetes manifests in CI pipelines for security misconfigurations and compliance violations.

### **31. What is tfsec / Trivy for IaC?**
**Answer:** Fast static security scanners that parse HCL code and flag security risks (unencrypted storage, public S3 buckets, open security groups) before deployment.

### **32. What is Infracost?**
**Answer:** A CLI tool that parses Terraform code in CI/CD pull requests, calculates the exact monthly cost delta of proposed infrastructure changes, and posts an interactive breakdown comment onto the PR.

### **33. What is Atlantis for Terraform?**
**Answer:** A self-hosted application that executes `terraform plan` on PR creation, posts the diff, locks state, and applies changes only when authorized reviewers comment `atlantis apply`.

### **34. What is HashiCorp Cloud Platform (HCP) Terraform / Terraform Cloud?**
**Answer:** A managed collaborative platform providing remote execution runners, centralized state locking, private module registries, cost estimation, and Sentinel policy enforcement.

### **35. What is Spacelift?**
**Answer:** A collaborative infrastructure orchestration platform for Terraform, OpenTofu, Pulumi, and CloudFormation with built-in OPA policy enforcement and drift detection.

### **36. What is the `ignore_changes` lifecycle rule in Terraform?**
**Answer:** Instructs Terraform to ignore updates to specific resource attributes caused by external systems or autoscaling (e.g., `lifecycle { ignore_changes = [tags, desired_capacity] }`).

### **37. What is `create_before_destroy` lifecycle rule?**
**Answer:** Inverts Terraform's default replacement behavior by creating the replacement resource *first* before destroying the old resource, minimizing service downtime.

### **38. What is `prevent_destroy` lifecycle rule?**
**Answer:** Causes Terraform to reject any execution plan that would destroy the annotated critical resource (e.g., production database).

### **39. What is a Terraform Provider Alias?**
**Answer:** Allows configuring multiple instances of the same provider (e.g., deploying resources across multiple AWS regions or accounts in a single configuration) using `alias = "west"`.

### **40. What is Terraform Variable Precedence?**
**Answer:** Evaluated with increasing priority: Environment variables (`TF_VAR_name`) $\rightarrow$ `terraform.tfvars` $\rightarrow$ `terraform.tfvars.json` $\rightarrow$ `*.auto.tfvars` $\rightarrow$ CLI flags (`-var`).

### **41. What is `terraform state rm` vs `terraform state mv`?**
**Answer:**
- **`state rm`:** Removes a resource from state; live cloud resource is **not deleted**, Terraform simply stops managing it.
- **`state mv`:** Renames or moves a resource within state without destroying or recreating it.

### **42. What is Terraform Provider Mirroring?**
**Answer:** Configuring CLI config (`.terraformrc`) to read pre-downloaded provider binaries from local disk or private registries in air-gapped environments without internet access.

### **43. What is Terraform Blast Radius and how do you minimize it?**
**Answer:** The maximum potential damage from a failed apply. Minimized by breaking monolithic state files into layered, decoupled state files (Networking $\rightarrow$ Data $\rightarrow$ Compute $\rightarrow$ App).

### **44. What is Ansible Playbook vs Role vs Collection?**
**Answer:**
- **Playbook:** YAML file mapping tasks and configuration states to target hosts.
- **Role:** Reusable directory packaging tasks, handlers, variables, and templates.
- **Collection:** Modern packaging format containing multiple roles, custom Python modules, and plugins.

### **45. What is Ansible Inventory (Static vs Dynamic)?**
**Answer:**
- **Static:** INI/YAML file listing static hostnames and IP addresses.
- **Dynamic:** Plugin querying cloud APIs (AWS EC2, Azure) to dynamically discover target instances based on tags.

### **46. What is Ansible Vault?**
**Answer:** Encrypts sensitive variables, files, or playbooks with AES-256 (`ansible-vault encrypt secrets.yml`).

### **47. What is OpenTofu State Encryption?**
**Answer:** A native OpenTofu feature allowing state files and plan files to be encrypted end-to-end at the client level using AES-GCM or AWS KMS before being transmitted to remote backends.

### **48. What is Terraform Plugin Framework (v6)?**
**Answer:** Modern Go SDK for authoring Terraform providers with native support for structured types, nested object attributes, and gRPC protocol buffers.

### **49. What is Crossplane Composite Resource (XR) vs Claim (XRC)?**
**Answer:**
- **XRD:** Platform team defines the abstract schema (`kind: DatabaseInstance`).
- **Composition:** Binds XRD to concrete cloud resources (RDS, Security Group, KMS Key).
- **Claim:** Developers request a database via a simple 5-line Kubernetes manifest without knowing AWS details.

### **50. What is HashiCorp Sentinel?**
**Answer:** A proprietary Policy as Code framework embedded in Terraform Cloud that enforces compliance rules (e.g., blocking non-whitelisted EC2 instance types) before apply.

### **51. What is Open Policy Agent (OPA) / Conftest for Terraform?**
**Answer:** Evaluates exported Terraform execution plans in JSON format against declarative **Rego** security policies in CI pipelines.

### **52. What is KubeVela?**
**Answer:** A modern application delivery platform on Kubernetes built on the Open Application Model (OAM) providing application-level abstractions over underlying infrastructure.

### **53. What is Terraform `terraform_remote_state` Data Source?**
**Answer:** A data source that reads root output values from a separate remote state file, enabling loose coupling between infrastructure layers.

### **54. What is Terraform `templatefile()` function?**
**Answer:** Renders an external template file with dynamic variable interpolation (e.g., rendering `user_data.sh` scripts).

### **55. What is Terraform `null_resource` vs `terraform_data`?**
**Answer:**
- **`null_resource` (Legacy):** Requires downloading a separate provider plugin to execute local scripts (`local-exec`).
- **`terraform_data` (Built-in since 1.4):** Native replacement for `null_resource` requiring zero external provider downloads.

### **56. What is `local-exec` vs `remote-exec` Provisioner?**
**Answer:**
- **`local-exec`:** Runs a local command on the machine executing Terraform.
- **`remote-exec`:** Connects to the provisioned remote instance over SSH/WinRM to execute shell commands. *Best practice: Avoid provisioners; use cloud-init or Ansible.*

### **57. What is Terraform Provider Dependency Lock?**
**Answer:** Records exact cryptographic provider checksums in `.terraform.lock.hcl` to ensure all team members and CI runners download identical provider binaries.

### **58. What is Terraform Self Reference (`self`)?**
**Answer:** An object referring to the resource's own attributes within `provisioner` or `connection` blocks.

### **59. What is Terraform Module Registry?**
**Answer:** A public or private repository for discovering and distributing versioned, reusable Terraform modules.

### **60. What is `terraform force-unlock`?**
**Answer:** Releases a stuck remote state lock in DynamoDB/Consul when a previous run crashed unexpectedly: `terraform force-unlock <LOCK-ID>`.

---

## 🟡 **Part 2: Advanced Terraform, OpenTofu & Terragrunt (Questions 61–130)**

### **61. Explain the step-by-step lifecycle of `terraform plan`.**
**Answer:**
1. Loads configuration files and builds the Dependency Graph (DAG).
2. Reads the current state file from the remote backend.
3. Refreshes state: Queries cloud APIs to fetch current live resource attributes.
4. Compares live state against declared code to determine CRUD deltas (Add, Change, Destroy).
5. Outputs the execution plan diff.

### **62. What is Terraform Graph Cycle and how is it resolved?**
**Answer:** A circular dependency error where Resource A depends on Resource B, and Resource B depends on Resource A. Resolved by decomposing circular references (e.g., creating security groups first, defining `aws_security_group_rule` as independent resources).

### **63. How do you implement Zero-Downtime AMI updates in Auto Scaling Groups using Terraform?**
**Answer:**
1. Use `aws_launch_template` with `create_before_destroy = true`.
2. Configure `instance_refresh` block in `aws_autoscaling_group` to orchestrate rolling instance replacements automatically upon template updates.

### **64. What is the difference between `lookup()`, `try()`, and `can()` functions in HCL?**
**Answer:**
- **`lookup(map, key, default)`:** Retrieves value from a map with a fallback default.
- **`try(expr1, expr2, ...)`:** Evaluates expressions sequentially, returning the first one that does not produce an error.
- **`can(expr)`:** Evaluates an expression and returns `true` if it succeeds without errors, or `false` otherwise.

### **65. What is OpenTofu Early Evaluation of Variables?**
**Answer:** An OpenTofu feature allowing variables and locals to be evaluated during backend configuration blocks, eliminating hardcoded backend S3 bucket names.

### **66. What is Terragrunt `read_terragrunt_config()`?**
**Answer:** Imports helper HCL files into a Terragrunt configuration, allowing centralized sharing of environment variables, AWS account IDs, and common tags.

### **67. What is Terragrunt `dependency` Block?**
**Answer:** Declares a dependency on another module, automatically resolving remote outputs and establishing execution order (`terragrunt run-all apply`).

### **68. What are Terragrunt `mock_outputs`?**
**Answer:** Supplies mock return values during `terragrunt plan` when an upstream dependency module has not yet been applied, preventing plan failures.

### **69. How do you implement automated Terraform Drift Detection in CI/CD?**
**Answer:**
A scheduled cron pipeline executing:
```bash
terraform plan -detailed-exitcode -no-color
```
- Exit 0: Succeeded, no differences.
- Exit 1: Malformed configuration or API error.
- Exit 2: Succeeded with non-empty diff (Drift Detected $\rightarrow$ alerts Slack).

### **70. How do you test Terraform modules with Terratest in Go?**
**Answer:**
```go
func TestVpcModule(t *testing.T) {
    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../modules/vpc",
        Vars: map[string]interface{}{"cidr_block": "10.0.0.0/16"},
    })
    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)
    vpcId := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcId)
}
```

### **71. What is Policy as Code with Rego and Conftest?**
**Answer:**
Export plan to JSON (`terraform show -json tfplan.binary > tfplan.json`), then evaluate against Rego rules in CI:
```rego
package main
deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "aws_s3_bucket"
    not resource.change.after.server_side_encryption_configuration
    msg := "S3 buckets must have server-side encryption enabled!"
}
```

### **72. What is OpenTofu Provider Plugin Signing and Verification?**
**Answer:** OpenTofu verifies cryptographic PGP signatures and SHA256 checksums of all downloaded provider binaries against registry public keys before loading them into memory.

### **73. What is Terraform Module Semantic Versioning?**
**Answer:** Pinning module sources to specific SemVer versions (`source = "terraform-aws-modules/vpc/aws?version=5.1.0"`) to prevent untested breaking changes from being applied automatically.

### **74. What is the difference between `merge()` and `concat()` in HCL?**
**Answer:**
- `merge(map1, map2)`: Combines two or more maps into a single map (keys overwritten by rightmost map).
- `concat(list1, list2)`: Combines two or more lists into a single sequential list.

### **75. What is Terraform Custom Provider development?**
**Answer:** Authoring custom Go plugins using the HashiCorp Terraform Plugin Framework that implement `Create`, `Read`, `Update`, and `Delete` methods against internal proprietary APIs.

---

## 🔴 **Part 3: Crossplane, Pulumi, Ansible & Enterprise Scenarios (Questions 76–200)**

### **76. Scenario: An engineer accidentally ran `terraform destroy` against staging and cancelled it midway. How do you recover?**
**Answer:**
1. Release any stuck state locks: `terraform force-unlock <LOCK-ID>`.
2. Sync state: `terraform refresh`.
3. Generate targeted recovery plan: `terraform plan -out=recovery.plan`.
4. Apply plan to recreate destroyed resources and restore consistency: `terraform apply recovery.plan`.

### **77. Scenario: Refactor a monolithic 500-resource Terraform state into 5 decoupled state files with zero downtime.**
**Answer:**
1. Backup production state file.
2. Create target directory structure (`01-network/`, `02-database/`, `03-compute/`).
3. Migrate resources using `terraform state mv -state=monolith.tfstate -state-out=02-database/terraform.tfstate <source> <dest>`.
4. Initialize remote S3 backends in each new directory.
5. Verify with `terraform plan` in each directory to confirm **0 to add, 0 to change, 0 to destroy**.

### **78. Scenario: A live production AWS S3 bucket containing 50TB of data was created manually. Bring it under Terraform management with zero data loss risk.**
**Answer:**
1. Declare resource in HCL: `resource "aws_s3_bucket" "prod_data" { bucket = "company-prod-data" }`.
2. Import resource via `import {}` block (Terraform 1.5+).
3. Run `terraform plan` and reconcile attributes until plan shows **0 to add, 0 to change, 0 to destroy**.
4. Apply to bind state.

### **79. Compare Crossplane vs Terraform at enterprise scale.**
**Answer:**
- **Terraform:** Pipeline-driven batch execution; drift detected only on pipeline runs.
- **Crossplane:** Continuous control plane loop inside Kubernetes; continuously auto-heals drift and provides native Kubernetes CRD abstractions for developers.

### **80. How do you structure a production multi-environment Terragrunt repository?**
**Answer:**
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

### **81. How do you implement zero-trust secrets injection in Terraform without writing credentials to disk?**
**Answer:**
Configure Terraform with the **HashiCorp Vault Dynamic Secrets Provider** (`data "vault_aws_access_credentials"`). Terraform requests short-lived (1-hour) cloud credentials generated on-demand by Vault, eliminating static keys entirely.

### **82. Scenario: An engineer accidentally committed a production `.tfstate` file containing database passwords to GitHub. Walk through the complete response procedure.**
**Answer:**
1. **Rotate All Secrets Immediately:** Rotate database master passwords, private TLS keys, and IAM credentials in the cloud provider.
2. **Purge Git History:** Use `git-filter-repo` to permanently strip `.tfstate` from all commits, branches, and tags. Force-push clean history.
3. **Move to Remote S3 Backend:** Configure remote encrypted S3 backend with DynamoDB locking.
4. **Enforce Prevention:** Add `*.tfstate` to `.gitignore` and configure GitHub Secret Scanning.

### **83. What is Ansible Dynamic Inventory for AWS EC2?**
**Answer:** An Ansible plugin (`aws_ec2`) that queries AWS APIs to automatically group managed instances based on tags (e.g., `tag:Role_webservers`), eliminating static IP files.

### **84. What is Pulumi Stack?**
**Answer:** An isolated, independently configurable instance of a Pulumi program (e.g., `dev`, `staging`, `prod`) with its own state and secrets encryption provider.

### **85. What is Pulumi Crosswalk?**
**Answer:** A collection of high-level libraries (in TypeScript/Python) that implement well-architected cloud infrastructure best practices (e.g., creating a production VPC in 5 lines of code).

### **86. What is Crossplane Provider Family?**
**Answer:** Modular, decoupled Crossplane provider packages (e.g., `provider-aws-s3`, `provider-aws-rds`) that install only the specific CRD controllers needed, avoiding cluster CRD limits.

### **87. What is Crossplane Composition Function?**
**Answer:** Advanced gRPC functions (written in Go, Python, or KCL) that dynamically render complex compositions and conditionals that static YAML cannot express.

### **88. What is Terraform Provider Factory in Unit Tests?**
**Answer:** Mocking cloud provider responses during unit tests to validate HCL logic and module transformations without making real cloud API calls.

### **89. What is Terraform `replace_triggered_by`?**
**Answer:** A lifecycle attribute that forces a resource to be replaced whenever another specified resource changes (e.g., replacing an EC2 instance whenever a security group changes).

### **90. What is OpenTofu Client-Side State Encryption Key Rotation?**
**Answer:** Rotating state encryption keys by declaring the new key in the OpenTofu configuration and running `tofu apply`, which re-encrypts the state file transparently.

### **91. What is Ansible Asynchronous Actions (`async` and `poll`)?**
**Answer:** Running long-running tasks (e.g., database backups) asynchronously in the background while polling for status periodically.

### **92. What is Ansible Handler?**
**Answer:** A special task executed only when notified by another task that resulted in a `changed` status (e.g., restarting Nginx only when `nginx.conf` changes).

### **93. What is Ansible Serial Execution (`serial`)?**
**Answer:** Rolling execution of playbooks across batches of servers (e.g., `serial: "20%"`) to perform zero-downtime rolling updates.

### **94. What is Ansible Tower / AWX?**
**Answer:** A web-based UI, REST API, and task engine for centrally managing Ansible playbooks, inventories, credentials, and RBAC across enterprise teams.

### **95. What is Terraform `nonsensitive()` function?**
**Answer:** Exposes a sensitive value as non-sensitive when strictly required for display or passing to external non-sensitive modules.

### **96. What is Terraform Output `depends_on`?**
**Answer:** Explicitly ordering output evaluation after specified resources complete provisioning.

### **97. What is Terraform Module Nesting Best Practice?**
**Answer:** Keep module nesting to a maximum of 2 levels (Root $\rightarrow$ Child Module). Excessive nesting creates brittle dependency graphs and unmaintainable code.

### **98. What is Terraform CLI Configuration (`.terraformrc`)?**
**Answer:** A configuration file on the developer workstation or CI runner managing plugin cache directories, credentials helpers, and provider installation mirrors.

### **99. What is Terraform Plugin Cache Directory (`TF_PLUGIN_CACHE_DIR`)?**
**Answer:** Caching downloaded provider binaries locally on disk across multiple directories, saving gigabytes of bandwidth and minutes of CI build time.

### **100. What is Policy as Code Shift-Left Testing?**
**Answer:** Executing OPA/Checkov linters directly in pre-commit hooks and PR builds to catch compliance violations before code is merged to `main`.

### **101. What is Terraform JSON Syntax (`*.tf.json`)?**
**Answer:** Allowing Terraform configurations to be written in standard JSON rather than HCL, useful for machine-generated infrastructure templates.

### **102. What is Pulumi Automation API?**
**Answer:** An SDK that embeds Pulumi infrastructure provisioning directly inside application code (e.g., a SaaS app provisioning tenant databases programmatically in Go).

### **103. What is Crossplane Environment Config?**
**Answer:** Injecting shared global configuration variables (AWS Account IDs, environment names) dynamically into Crossplane Compositions.

### **104. What is Ansible Fact Gathering (`setup` module)?**
**Answer:** Automatically discovering hardware, OS, network, and disk facts from target hosts upon playbook execution and storing them in `ansible_facts`.

### **105. What is Terraform `terraform_version` Constraint?**
**Answer:** Locking configurations to compatible engine versions (`required_version = ">= 1.5.0, < 2.0.0"`) to prevent syntax incompatibility issues.

### **106. What is Terraform Provider Source Address?**
**Answer:** The fully qualified global namespace of a provider: `registry.terraform.io/hashicorp/aws`.

### **107. What is OpenTofu Registry?**
**Answer:** An open-source, community-operated public registry (`get.opentofu.org`) hosting verified providers and modules for OpenTofu.

### **108. What is Terragrunt Run-All?**
**Answer:** Recursively executes Terraform commands across all subdirectories in DAG dependency order (`terragrunt run-all plan`).

### **109. What is Terraform Remote State Locking Mechanism in DynamoDB?**
**Answer:** Uses an MD5 hash of the state file and a unique lock ID written to a DynamoDB table with strong consistency to prevent concurrent applies.

### **110. What is Infracost FinOps Shift-Left?**
**Answer:** Posting automated cloud cost delta comments on GitHub Pull Requests to make developers financially aware before merging infrastructure changes.

### **111. What is Trivy IaC Scanning?**
**Answer:** Scanning Terraform, CloudFormation, and Dockerfile code for over 1,000 built-in security misconfiguration checks.

### **112. What is Checkov Custom Rego Policies?**
**Answer:** Writing custom organizational compliance rules in Rego that Checkov evaluates against Terraform AST structures in CI.

### **113. What is Ansible Galaxy?**
**Answer:** The public community repository for discovering, downloading, and sharing Ansible roles and collections.

### **114. What is Pulumi Secrets Provider?**
**Answer:** Encrypting sensitive configuration values at rest in state using cloud KMS (AWS KMS, Azure Key Vault, HashiCorp Vault).

### **115. What is Crossplane Package Manager?**
**Answer:** Distributing and installing Providers and Configurations as OCI artifacts packaged and versioned in standard container registries.

### **116. What is Terraform State Inspect (`terraform state show`)?**
**Answer:** Prints the exact attributes and current live state of a single specific resource managed by Terraform.

### **117. What is Terraform State List (`terraform state list`)?**
**Answer:** Lists all resource addresses currently tracked in the state file.

### **118. What is Terraform Plan Output File (`-out=tfplan`)?**
**Answer:** Saves the computed plan binary to disk, guaranteeing that `terraform apply tfplan` executes the exact evaluated diff without re-querying cloud APIs.

### **119. What is Terraform Variable Validation (`validation` block)?**
**Answer:** Enforcing custom regex and logic rules on input variables before execution:
```hcl
variable "environment" {
  type = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

### **120. What is Terraform Sensitive Variable in Output?**
**Answer:** Output values derived from sensitive variables are automatically marked sensitive and suppressed from CLI output.

### **121. What is Terraform Dynamic Provider Credentials?**
**Answer:** Exchanging OIDC tokens from CI systems (GitHub Actions) for temporary cloud provider credentials dynamically during `terraform apply`.

### **122. What is Terraform State Pull (`terraform state pull`)?**
**Answer:** Downloads and outputs the raw JSON state file from the remote backend to stdout.

### **123. What is Terraform State Push (`terraform state push`)?**
**Answer:** Manually uploads a modified state file to the remote backend (used for state disaster recovery).

### **124. What is Terraform Enterprise Private Module Registry?**
**Answer:** An internal organization module registry with built-in versioning, documentation generation, and access control.

### **125. What is Terragrunt Scaffold?**
**Answer:** A CLI command that automatically generates boilerplate Terragrunt configuration files from a source Terraform module.

### **126. What is OpenTofu State Migration?**
**Answer:** OpenTofu natively reads and writes existing Terraform state files with 100% backward compatibility.

### **127. What is Ansible Jinja2 Templating?**
**Answer:** Dynamically generating configuration files (`template` module) with variable substitution and loops using Jinja2 syntax.

### **128. What is Ansible Block and Rescue?**
**Answer:** Exception handling in playbooks: tasks in `block` are executed; if any fail, tasks in `rescue` execute to remediate.

### **129. What is Ansible Check Mode (`--check`)?**
**Answer:** Dry-run execution that reports what changes would be made without actually modifying managed target servers.

### **130. What is Pulumi Policy as Code (CrossGuard)?**
**Answer:** Enforcing organizational compliance rules across Pulumi stacks written in TypeScript or Python before resources are provisioned.

### **131. What is Crossplane Provider AWS vs Upjet AWS?**
**Answer:** Upjet generates Crossplane providers automatically from Terraform provider schemas, ensuring 100% cloud resource coverage.

### **132. What is Terraform Cloud Agent?**
**Answer:** Self-hosted worker processes deployed in private VPCs to execute Terraform Cloud runs against internal private resources.

### **133. What is Terraform Variable Definition File (`terraform.tfvars`)?**
**Answer:** A file automatically loaded by Terraform containing key-value assignments for declared input variables.

### **134. What is Terraform Workspace Isolation Anti-Pattern?**
**Answer:** Using workspaces for staging and production when environments require different AWS accounts, IAM roles, or backend configurations.

### **135. What is Terraform State File Locking Timeout (`-lock-timeout`)?**
**Answer:** Instructs Terraform to wait up to $N$ duration (e.g., `-lock-timeout=5m`) to acquire a state lock before failing.

### **136. What is Terraform Output Value Interpolation?**
**Answer:** Referencing child module outputs in parent modules using `module.<name>.<output_name>`.

### **137. What is Terraform Implicit Dependency?**
**Answer:** Dependencies established automatically when one resource references an attribute of another resource (`vpc_id = aws_vpc.main.id`).

### **138. What is Terraform Explicit Dependency (`depends_on`)?**
**Answer:** Manually declaring dependency order when no direct attribute reference exists in HCL code.

### **139. What is Terraform Provider Alias in Child Modules?**
**Answer:** Passing custom provider configurations into child modules via the `providers` map block.

### **140. What is OpenTofu Early Backend Evaluation?**
**Answer:** Evaluating variables and functions inside backend configuration blocks natively in OpenTofu.

### **141. What is Ansible Delegate_To?**
**Answer:** Executing a specific task on a different host (e.g., taking an instance out of a load balancer on the control node) during a host playbook.

### **142. What is Ansible Local_Action?**
**Answer:** Shorthand syntax for delegating a task to execute locally on the Ansible control machine.

### **143. What is Pulumi Resource Transformation?**
**Answer:** Intercepting and programmatically modifying resource arguments across all resources in a Pulumi stack (e.g., auto-tagging all resources).

### **144. What is Crossplane Connection Secret?**
**Answer:** Writing database passwords and endpoint URLs generated by cloud providers directly into Kubernetes Secret objects for application consumption.

### **145. What is Terraform S3 Backend Key Naming Convention?**
**Answer:** Storing state files in structured S3 paths: `env://prod/network/terraform.tfstate`.

### **146. What is Terraform Backend Re-initialization (`terraform init -reconfigure`)?**
**Answer:** Reconfiguring remote backend settings from scratch, disregarding any existing backend configuration cached in `.terraform/`.

### **147. What is Terraform Backend Migration (`terraform init -migrate-state`)?**
**Answer:** Migrating state data automatically from one backend type to another (e.g., local state to remote S3).

### **148. What is Terraform Custom Function in Provider Framework?**
**Answer:** User-defined HCL functions exposed by modern provider plugins to execute specialized string/crypto transformations.

### **149. What is Terragrunt Hooks?**
**Answer:** Executing custom scripts or CLI commands before or after Terraform execution (`before_hook`, `after_hook`).

### **150. What is OpenTofu Registry Mirroring?**
**Answer:** Redirecting provider and module downloads to private enterprise mirrors in air-gapped environments.

### **151. What is Ansible Meta Dependencies (`meta/main.yml`)?**
**Answer:** Declaring prerequisite roles that must be executed automatically before the parent role runs.

### **152. What is Ansible Custom Filter Plugin?**
**Answer:** Writing custom Python functions to extend Jinja2 template transformations inside playbooks.

### **153. What is Pulumi ComponentResource?**
**Answer:** A higher-level logical resource that aggregates multiple primitive cloud resources into a single reusable component.

### **154. What is Crossplane Provider Config?**
**Answer:** A Kubernetes CRD defining authentication credentials and assume-role configurations for a specific cloud provider in Crossplane.

### **155. What is Terraform Flatten Function (`flatten()`)?**
**Answer:** Flattens nested lists of lists into a single flat list, commonly used with nested dynamic block collections.

### **156. What is Terraform Element Function (`element()`)?**
**Answer:** Retrieves an element from a list at a given index with wrap-around modulo behavior.

### **157. What is Terraform SetProduct Function (`setproduct()`)?**
**Answer:** Computes the Cartesian product of multiple sets or lists to generate combinations of environments and subnets.

### **158. What is Terraform Zipmap Function (`zipmap()`)?**
**Answer:** Constructs a map from a list of keys and a corresponding list of values.

### **159. What is Terraform One Function (`one()`)?**
**Answer:** Converts a zero- or one-element list into a scalar value or `null`, failing if the list contains multiple elements.

### **160. What is OpenTofu State Encryption Key Derivation?**
**Answer:** Using PBKDF2 with SHA256 to derive encryption keys from passphrases for local and remote state encryption.

### **161. What is Terragrunt Auto-Init?**
**Answer:** Terragrunt automatically executes `terraform init` when backend settings or provider configurations change.

### **162. What is Ansible Lookups (`lookup()`)?**
**Answer:** Querying external data sources (environment variables, HashiCorp Vault, AWS SSM) dynamically during playbook execution.

### **163. What is Ansible Lineinfile Module?**
**Answer:** Ensuring a specific line of text exists or is modified in a file using regex matching.

### **164. What is Pulumi Stack Output Reference?**
**Answer:** Reading exported output values from another Pulumi stack (`new StackReference("org/proj/prod")`).

### **165. What is Crossplane Management Policies?**
**Answer:** Granularly controlling which CRUD operations Crossplane is allowed to perform on a cloud resource (`ObserveOnly`, `Create`, `Update`, `Delete`).

### **166. What is Terraform Module Git Source Syntax?**
**Answer:** Referencing modules in Git with tags: `source = "git::https://github.com/org/repo.git?ref=v1.2.0"`.

### **167. What is Terraform Sub-directory Git Source?**
**Answer:** Referencing modules in subdirectories of a Git repo: `source = "git::https://github.com/org/repo.git//modules/vpc?ref=v1.2.0"`.

### **168. What is Terraform Provider Invalidation?**
**Answer:** Forcing re-download of provider plugins by deleting the `.terraform/providers` directory.

### **169. What is Terraform Cloud Workspace VCS Integration?**
**Answer:** Automatically triggering `terraform plan` and `terraform apply` on Git commits to specific branches in GitHub/GitLab.

### **170. What is OpenTofu Test Framework (`tofu test`)?**
**Answer:** Native testing framework evaluating HCL assertion blocks against ephemeral infrastructure without third-party test engines.

### **171. What is Terragrunt Include Block?**
**Answer:** Inheriting configuration from a parent `terragrunt.hcl` file into child environment directories.

### **172. What is Ansible Become (`become: true`)?**
**Answer:** Privilege escalation system executing tasks with `sudo` or root permissions on target nodes.

### **173. What is Ansible Tags (`--tags`)?**
**Answer:** Executing a subset of tasks in a playbook marked with specific tag labels.

### **174. What is Pulumi Cloud Provider Dynamic Providers?**
**Answer:** Authoring custom providers in TypeScript/Python implementing CRUD methods for bespoke REST APIs.

### **175. What is Crossplane Composition Patching?**
**Answer:** Mapping fields from an abstract Claim manifest (e.g., storage size) into underlying concrete cloud resources (e.g., EBS volume spec).

### **176. What is Terraform State Refresh Only (`terraform apply -refresh-only`)?**
**Answer:** Updates the state file to reflect live cloud attributes without modifying any live cloud resources.

### **177. What is Terraform Target Flag (`-target`) and why is it an anti-pattern?**
**Answer:** Restricting plan/apply to a single resource. *Risk: Bypasses dependency graph tracking, causing orphaned resources and drift.*

### **178. What is Terraform Parallelism Flag (`-parallelism=N`)?**
**Answer:** Controls how many concurrent resource operations Terraform executes in parallel (default: 10).

### **179. What is OpenTofu Provider Signature Verification Failure?**
**Answer:** Occurs when a downloaded provider plugin's checksum does not match `.terraform.lock.hcl`, indicating corrupted files or tampering.

### **180. What is Terragrunt Generate Block?**
**Answer:** Automatically generates boilerplate `.tf` files (provider configs, backend configs) in working directories dynamically before execution.

### **181. What is Ansible Stat Module?**
**Answer:** Retrieves filesystem status and metadata (existence, permissions, checksum) of a file on remote hosts.

### **182. What is Ansible Template vs Copy Module?**
**Answer:** `copy` copies static files byte-for-byte; `template` renders dynamic variables and loops via Jinja2 before copying.

### **183. What is Pulumi Dynamic Secrets?**
**Answer:** Integrating Pulumi with HashiCorp Vault to issue short-lived credentials dynamically during cloud provisioning.

### **184. What is Crossplane Reconcile Rate Limit?**
**Answer:** Configuring controller reconcile intervals to prevent hitting cloud provider API rate limits.

### **185. What is Terraform State Lock ID?**
**Answer:** A unique UUID generated when an operation begins, recorded in the lock store to identify the locking process.

### **186. What is Terraform Plan Compact Warnings (`-compact-warnings`)?**
**Answer:** Condenses warning outputs in CLI terminal output for cleaner logs.

### **187. What is Terraform Debug Logging (`TF_LOG=DEBUG`)?**
**Answer:** Enables verbose internal debugging logs printing raw cloud API HTTP requests and response payloads.

### **188. What is OpenTofu Migration Guide from Terraform 1.6+?**
**Answer:** OpenTofu provides drop-in CLI replacement for Terraform configs and state files.

### **189. What is Terragrunt Dry-Run?**
**Answer:** Validating configuration inheritance without executing underlying Terraform binaries.

### **190. What is Ansible Vault Rekey (`ansible-vault rekey`)?**
**Answer:** Changing the encryption password of an existing Ansible Vault encrypted file.

### **191. What is Ansible Facts Cache?**
**Answer:** Persisting gathered host facts across playbook runs using Redis or JSON files to speed up execution.

### **192. What is Pulumi Native Providers?**
**Answer:** Auto-generated providers derived directly from cloud provider OpenAPI specs (e.g., `pulumi-aws-native`), providing zero-day support for new cloud APIs.

### **193. What is Crossplane Provider-Kubernetes?**
**Answer:** Managing arbitrary Kubernetes resources in target clusters using Crossplane CRDs.

### **194. What is Terraform Workspace Delete (`terraform workspace delete`)?**
**Answer:** Deletes an empty workspace; fails if resources are still tracked in the workspace state.

### **195. What is Terraform Output JSON Format (`terraform output -json`)?**
**Answer:** Emits all module output values in machine-readable JSON format for scripting.

### **196. What is OpenTofu Registry Authentication?**
**Answer:** Configuring access tokens in CLI config to authenticate to private OpenTofu module registries.

### **197. What is Terragrunt Local Inputs?**
**Answer:** Passing environment-specific variables directly into Terraform modules from `terragrunt.hcl`.

### **198. What is Ansible Galaxy Requirements (`requirements.yml`)?**
**Answer:** Declaring third-party roles and collections to be downloaded automatically before playbook execution.

### **199. What is Pulumi Policy as Code Enforcement Level (`mandatory` vs `advisory`)?**
**Answer:** `advisory` prints warnings; `mandatory` blocks stack updates if compliance rules are violated.

### **200. What is Crossplane Health Status (`Ready: True`)?**
**Answer:** Kubernetes condition indicating that the underlying live cloud resource has been successfully provisioned, initialized, and is fully functional.
