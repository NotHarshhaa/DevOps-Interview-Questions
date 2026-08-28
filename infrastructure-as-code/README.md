# **Infrastructure as Code (IaC) - DevOps Interview Questions**

Welcome to the **Infrastructure as Code (IaC)** interview questions module. This section covers Terraform, OpenTofu, Terragrunt, Crossplane, Pulumi, Ansible, state file management, drift detection, IaC security scanning, and automated testing.

---

## 🟢 **Beginner Level (Questions 1–20)**

### **1. What is Infrastructure as Code (IaC) and what are its primary benefits?**
**Answer:**
IaC is the practice of provisioning, configuring, and managing cloud infrastructure using machine-readable definition files rather than manual web console clicks or imperative CLI commands.

**Primary Benefits:**
- **Consistency & Idempotency:** Eliminates configuration drift; identical environments can be stood up deterministically.
- **Speed & Agility:** Automates hours of manual infrastructure setup into minutes.
- **Version Control & Collaboration:** Changes are tracked in Git, peer-reviewed via Pull Requests, and audited.
- **Disaster Recovery:** Entire regions can be recreated from code repositories.

---

### **2. What is the difference between Declarative vs Imperative IaC?**
**Answer:**
- **Declarative (e.g., Terraform, OpenTofu, CloudFormation, Kubernetes YAML):** You specify the **desired target state**, and the IaC engine determines what actions (create, update, delete) are needed to reach that state.
- **Imperative (e.g., AWS CLI, Bash scripts, Python Boto3, Chef):** You specify the **exact sequence of commands** needed to reach the desired state.

---

### **3. What is Terraform State and why is it necessary?**
**Answer:**
The Terraform state file (`terraform.tfstate`) is a JSON file mapping your declarative configuration to real-world cloud resource IDs and their metadata.

**Why it is necessary:**
- **Mapping:** Maps Terraform resource names (e.g., `aws_instance.web`) to cloud provider IDs (e.g., `i-0a1b2c3d4e5f`).
- **Metadata Tracking:** Tracks resource dependencies and ordering.
- **Performance:** Caches resource attributes locally to avoid querying hundreds of slow cloud APIs on every plan.

---

### **4. What are the core Terraform CLI workflow commands?**
**Answer:**
1. `terraform init`: Initializes the working directory, downloads required provider plugins and modules, and configures the remote backend.
2. `terraform validate`: Validates syntax and internal consistency of configuration files without accessing cloud APIs.
3. `terraform plan`: Generates an execution plan by comparing declared code against remote state and live infrastructure.
4. `terraform apply`: Executes the changes required to reach the desired configuration state.
5. `terraform destroy`: Destroys all remote objects managed by the current state file.

---

### **5. What is the difference between Terraform `count` and `for_each`?**
**Answer:**
- **`count`:** Creates resources indexed by an integer list (`[0, 1, 2]`).
  - *Problem:* If you remove an item from the middle of the list, Terraform will reindex and force-recreate all subsequent resources.
- **`for_each`:** Creates resources keyed by unique map or set strings (`["web", "api", "db"]`).
  - *Advantage:* Removing or adding an item affects *only* that specific key, preserving all other resources.

---

### **6. What is a Terraform Module and why should you use it?**
**Answer:**
A Terraform module is a container for multiple resources that are used together.
- **Root Module:** The default working directory containing your `.tf` files.
- **Child Module:** Reusable components called from other configurations (`module "vpc" { source = "./modules/vpc" }`).
- **Benefits:** Enforces DRY (Don't Repeat Yourself) patterns, standardized security baselines, and reusability across teams.

---

### **7. What are Terraform Variables, Locals, and Outputs?**
**Answer:**
- **Input Variables (`variable`):** Parameters passed into modules to customize behavior (like function arguments).
- **Local Values (`locals`):** Internal reusable constants or computed expressions scoped to the current module.
- **Output Values (`output`):** Return values exposed by a module to the root or CLI output (like function return values).

---

### **8. What is a Remote Backend and why must teams use State Locking?**
**Answer:**
Storing state locally (`terraform.tfstate`) causes state loss, credential leakage, and concurrency conflicts when multiple engineers apply changes simultaneously.

**Remote Backend with State Locking (e.g., AWS S3 + DynamoDB / Terraform Cloud):**
- **Central Storage:** State is encrypted at rest in S3.
- **DynamoDB State Locking:** Acquires a distributed lock during `terraform plan` and `terraform apply` to prevent race conditions and state file corruption.

---

### **9. What is the difference between Terraform and Ansible?**
**Answer:**
| Dimension | Terraform / OpenTofu | Ansible |
| :--- | :--- | :--- |
| **Primary Domain** | **Infrastructure Provisioning** (VPCs, EKS, RDS, IAM) | **Configuration Management** (installing packages, OS tuning, app configs) |
| **Paradigm** | Declarative state management | Procedural / Declarative task execution |
| **State Tracking** | Maintains strict `.tfstate` | Stateless (queries live node directly) |
| **Agent** | Agentless (API-driven) | Agentless (SSH / WinRM-driven) |

---

### **10. What is OpenTofu and why did the open-source community fork Terraform?**
**Answer:**
In August 2023, HashiCorp switched Terraform's license from open-source MPL 2.0 to the Business Source License (BSL 1.1). In response, the Linux Foundation created **OpenTofu** as an open-source, community-driven, backward-compatible fork of Terraform.

---

### **11. What is Configuration Drift in IaC?**
**Answer:**
Drift is the discrepancy between the declared IaC configuration in Git and the actual live state of resources in the cloud provider (typically caused by manual changes in the web console).

---

### **12. What is `terraform refresh`?**
**Answer:**
`terraform refresh` reconciles the local state file with the real-world resources in the cloud provider. It reads the current settings from cloud APIs and updates the state file without modifying live infrastructure. *(Note: `terraform plan` and `apply` run an automatic refresh by default).*

---

### **13. What are Terraform Data Sources?**
**Answer:**
Data sources allow Terraform to fetch and read read-only information about external resources defined outside of Terraform or in separate state files.
```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}
```

---

### **14. What are Terraform Workspaces and when should you use (or avoid) them?**
**Answer:**
Workspaces manage multiple state files from a single configuration directory (`terraform workspace select staging`).
- **Good Use Case:** Creating short-lived feature branch environments.
- **Anti-Pattern for Production/Staging:** Workspaces share the exact same configuration code, making it difficult to test new module versions or apply environment-specific IAM guardrails. Directory-based separation (or Terragrunt) is preferred for long-lived environments.

---

### **15. What is `terraform taint` vs `terraform apply -replace`?**
**Answer:**
- **`terraform taint` (Legacy):** Manually marks a resource as degraded so it will be destroyed and recreated on the next apply.
- **`-replace` (Terraform 0.15+ Best Practice):** Replaces a resource in a single step without altering state ahead of time:
  ```bash
  terraform apply -replace="aws_instance.web"
  ```

---

### **16. What is a Terraform Provider?**
**Answer:**
A provider is a binary plugin (written in Go) distributed via the Terraform Registry that implements CRUD operations against a specific cloud or service API (e.g., AWS, Azure, GCP, Kubernetes, GitHub, Datadog).

---

### **17. What is Pulumi and how does it differ from Terraform?**
**Answer:**
Pulumi is an IaC platform that allows engineers to define cloud infrastructure using real programming languages (**TypeScript, Python, Go, C#, Java**) instead of domain-specific languages (HCL).
- **Advantages:** Full language power (loops, classes, unit testing with standard test frameworks, IDE autocompletion).

---

### **18. What is Crossplane?**
**Answer:**
Crossplane is a CNCF open-source project that transforms Kubernetes into a universal cloud control plane. Infrastructure resources (S3 buckets, RDS databases) are defined as Kubernetes Custom Resource Definitions (CRDs) and continuously reconciled by Kubernetes controllers.

---

### **19. What is Checkov / tfsec / Trivy for IaC?**
**Answer:**
Static security analysis tools for IaC files that scan Terraform, CloudFormation, and Helm charts for security vulnerabilities, compliance violations, and misconfigurations (e.g., unencrypted S3 buckets, overly permissive security groups `0.0.0.0/0`) before deployment.

---

### **20. What is `terraform fmt`?**
**Answer:**
A built-in CLI command that automatically formats all `.tf` files in the directory according to canonical Terraform style conventions (indentation, alignment).

---

## 🟡 **Intermediate Level (Questions 21–40)**

### **21. What is the `moved` block in Terraform and how does it prevent resource destruction during refactoring?**
**Answer:**
Prior to Terraform 1.1, renaming a resource or moving it into a child module caused Terraform to interpret the change as "destroy old resource and create new resource", causing catastrophic downtime in production databases.

**`moved` Block Solution:**
```hcl
moved {
  from = aws_instance.web
  to   = module.compute.aws_instance.web
}
```
Terraform automatically updates the internal state address without touching or recreating the live cloud resource.

---

### **22. What is the `import {}` block in Terraform 1.5+?**
**Answer:**
Previously, importing existing cloud resources into Terraform required running imperative CLI commands (`terraform import`). 
In Terraform 1.5+, imports are declared natively in code:
```hcl
import {
  to = aws_s3_bucket.logs
  id = "my-company-audit-logs"
}

resource "aws_s3_bucket" "logs" {
  bucket = "my-company-audit-logs"
}
```
Running `terraform plan` previews the import and verifies that the code matches the live cloud resource before applying.

---

### **23. What are Terraform Custom Preconditions and Postconditions?**
**Answer:**
Introduced to enforce custom validation rules within resource and data source lifecycles:
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type

  lifecycle {
    precondition {
      condition     = data.aws_ami.ubuntu.architecture == "x86_64"
      error_message = "The selected AMI must be x86_64 architecture."
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
**Answer:**
Terragrunt is a thin wrapper for Terraform/OpenTofu that provides tooling for keeping configurations DRY, managing remote state automatically, and orchestrating multi-module dependencies.

**Key Capabilities:**
- **DRY Remote State:** Defines backend configuration once in a root `terragrunt.hcl` and inherits it across hundreds of subdirectories.
- **Dependency Management:** Automatically executes modules in DAG dependency order (`terragrunt run-all apply`).
- **Mock Outputs:** Supplies mock outputs during `terragrunt plan` for modules whose upstream dependencies haven't been deployed yet.

---

### **25. How do Terraform Dynamic Blocks work?**
**Answer:**
Dynamic blocks construct repeatable nested configuration blocks within resources dynamically based on a collection:
```hcl
variable "ingress_ports" {
  type    = list(number)
  default = [80, 443, 8080]
}

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

### **26. What is the difference between `terraform.lock.hcl` and `.tfstate`?**
**Answer:**
- **`.terraform.lock.hcl` (Dependency Lock File):** Records exact provider plugin versions and their cryptographic checksums (hashes). Must be committed to Git to ensure all team members and CI runners use the exact same provider binaries.
- **`.tfstate` (State File):** Records the live cloud resource IDs and attributes. Must **never** be committed to Git; stored in a secure remote backend.

---

### **27. What are sensitive values and how do you protect secrets in Terraform outputs and state?**
**Answer:**
Marking variables and outputs as `sensitive = true` prevents values (passwords, tokens) from being displayed in CLI stdout or CI/CD logs:
```hcl
output "db_password" {
  value     = aws_db_instance.db.password
  sensitive = true
}
```
*Crucial Caveat:* `sensitive = true` does **not** encrypt values inside the `terraform.tfstate` file. To protect secrets at rest, the S3 backend must have KMS encryption enabled and IAM access restricted.

---

### **28. What is Terraform Drift Detection automation and how is it implemented?**
**Answer:**
Scheduled CI/CD job (e.g., GitHub Actions Cron or Spacelift):
```bash
terraform plan -detailed-exitcode -no-color
```
- **Exit Code 0:** Succeeded with zero diff (no drift).
- **Exit Code 1:** Error encountered.
- **Exit Code 2:** Succeeded and differences/drift were detected $\rightarrow$ Triggers alert on PagerDuty/Slack or auto-applies reconciliation.

---

### **29. What is Terratest and how is it used for IaC automated testing?**
**Answer:**
Terratest is a Go library developed by Gruntwork that provisions real infrastructure, runs integration tests against live endpoints, and tears everything down (`defer terraform.Destroy(t, terraformOptions)`).

```go
func TestVpcModule(t *testing.T) {
    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../modules/vpc",
    })
    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)

    vpcId := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcId)
}
```

---

### **30. How does Crossplane Compositions and Composite Resource Definitions (XRDs) work?**
**Answer:**
1. **XRD (Composite Resource Definition):** Platform team defines an abstract, standardized API (e.g., `kind: PostgreSQLInstance`).
2. **Composition:** Links the abstract XRD to concrete cloud provider resources (e.g., an AWS RDS DB Instance, Subnet Group, Security Group, and KMS Key).
3. **Claim (XRC):** Developers request a database via a simple 5-line Kubernetes manifest without needing to know AWS-specific details.

---

### **31. What is the `ignore_changes` lifecycle rule in Terraform?**
**Answer:**
Tells Terraform to ignore updates to specific resource attributes caused by external systems or autoscaling:
```hcl
resource "aws_autoscaling_group" "app" {
  min_size         = 2
  max_size         = 10
  desired_capacity = 2

  lifecycle {
    ignore_changes = [desired_capacity] # Prevents Terraform from overwriting HPA/KEDA changes
  }
}
```

---

### **32. What is Terraform Provider Alias and when is it required?**
**Answer:**
Allows defining multiple configurations for the same provider (e.g., deploying resources across multiple AWS regions or multiple AWS accounts in a single Terraform run):
```hcl
provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

resource "aws_s3_bucket" "dr_bucket" {
  provider = aws.west
  bucket   = "my-dr-backup-bucket"
}
```

---

### **33. What is Ansible Idempotency and how do you ensure custom tasks remain idempotent?**
**Answer:**
A task is idempotent if executing it multiple times produces the exact same result without unintended side effects.
- Standard modules (`apt`, `copy`, `systemd`, `template`) are natively idempotent.
- For `command` or `shell` modules, use `creates` or `removes` arguments to ensure commands run only when needed:
  ```yaml
  - name: Initialize application database
    ansible.builtin.command: /opt/app/init-db.sh
    args:
      creates: /var/lib/app/db_initialized.lock
  ```

---

### **34. What is Ansible Vault and how are sensitive variables encrypted?**
**Answer:**
Ansible Vault encrypts sensitive variables, files, or entire playbooks with AES-256:
- Encrypt file: `ansible-vault encrypt secrets.yml`
- Run playbook with vault: `ansible-playbook site.yml --vault-password-file ~/.vault_pass`

---

### **35. What is the difference between Ansible Roles and Collections?**
**Answer:**
- **Role:** A structured directory organization (`tasks/`, `handlers/`, `templates/`, `vars/`) for packaging reusable configuration tasks.
- **Collection:** The modern distribution format for Ansible content, packaging multiple roles, custom Python modules, action plugins, and playbooks together.

---

### **36. What is `terraform state rm` vs `terraform state mv`?**
**Answer:**
- **`terraform state rm <resource>`:** Removes a resource from the state file. The live cloud resource is **not deleted**; Terraform simply stops managing it.
- **`terraform state mv <source> <dest>`:** Renames or moves a resource within the state file without triggering deletion or recreation.

---

### **37. What is an IaC Blast Radius and how do you minimize it?**
**Answer:**
Blast radius is the maximum potential damage caused by a single failed execution or broken code change.
- **Monolithic State (High Risk):** Managing VPC, EKS, RDS, and DNS in a single 10,000-line state file.
- **Decoupled Architecture (Low Risk):** Splitting infrastructure into layered state files (Network $\rightarrow$ Cluster $\rightarrow$ Data $\rightarrow$ Application) with separate access controls.

---

### **38. What is Terraform Provider Mirroring / Air-gapped Execution?**
**Answer:**
In enterprise environments with no internet access, Terraform cannot reach `registry.terraform.io`.
- Configure `provider_installation` in CLI config (`.terraformrc`) to read pre-downloaded provider binaries from a local directory or private artifact repository (JFrog Artifactory / Nexus).

---

### **39. What is Atlantis for Pull Request-driven Terraform Workflows?**
**Answer:**
A self-hosted Go application that integrates with GitHub/GitLab webhooks. It runs `terraform plan` on PR creation, comments the diff on the PR, acquires a state lock, and runs `terraform apply` only when authorized reviewers comment `atlantis apply`.

---

### **40. What is Spacelift / Scalr / Terraform Cloud?**
**Answer:**
Collaborative IaC management platforms that provide remote runner execution, state management, state locking, role-based access control (RBAC), cost estimation, policy enforcement (OPA/Rego), and automated drift detection.

---

## 🔴 **Advanced & Scenario-Based Level (Questions 41–60)**

### **41. Scenario: A developer accidentally ran `terraform destroy` against a shared staging environment, but cancelled it midway (Ctrl+C). The state is now locked and partially destroyed. How do you recover?**
**Answer:**
1. **Clear Stuck Lock:**
   ```bash
   terraform force-unlock <LOCK-ID>
   ```
2. **Synchronize State:**
   ```bash
   terraform refresh
   ```
3. **Inspect Plan Diff:**
   ```bash
   terraform plan -out=recovery.plan
   ```
4. **Targeted Re-Apply:** Apply the plan to recreate destroyed resources and restore consistency:
   ```bash
   terraform apply recovery.plan
   ```

---

### **42. Scenario: You must refactor a monolithic 500-resource Terraform state file into 5 decoupled state files with zero downtime and zero cloud resource recreation. Walk through the exact migration strategy.**
**Answer:**
1. **Backup State:** Make a local copy of the production state file.
2. **Create Target Directory Structure:**
   ```
   ├── 01-networking/
   ├── 02-database/
   ├── 03-eks-cluster/
   └── 04-applications/
   ```
3. **State Migration via `terraform state mv`:**
   Use the `-state-out` flag to move specific resources directly between state files:
   ```bash
   terraform state mv \
     -state=monolith.tfstate \
     -state-out=02-database/terraform.tfstate \
     aws_db_instance.primary \
     aws_db_instance.primary
   ```
4. **Configure Remote Backends:** Initialize the target directories with their new S3 backend configurations.
5. **Verify with Plan:** Run `terraform plan` in each new directory. Verify that plan output shows **0 to add, 0 to change, 0 to destroy**.

---

### **43. Scenario: A live production AWS S3 bucket containing 50TB of data was created manually in the console. You must bring it under Terraform management without risk of data loss. How do you do it?**
**Answer:**
1. **Declare Resource in Code:**
   ```hcl
   resource "aws_s3_bucket" "prod_data" {
     bucket = "company-production-data-bucket"
   }
   ```
2. **Import Resource (Terraform 1.5+):**
   ```hcl
   import {
     to = aws_s3_bucket.prod_data
     id = "company-production-data-bucket"
   }
   ```
3. **Execute Plan:**
   ```bash
   terraform plan
   ```
4. **Reconcile Attributes:** If the plan shows pending changes, update the HCL code (e.g., versioning, server-side encryption, lifecycle rules) until `terraform plan` shows **0 to add, 0 to change, 0 to destroy**.
5. **Apply:** Run `terraform apply` to bind the state.

---

### **44. How do you implement Policy as Code guardrails with Open Policy Agent (OPA) / Conftest to block unencrypted cloud resources in CI/CD?**
**Answer:**
1. Export Terraform plan to JSON:
   ```bash
   terraform plan -out=tfplan.binary
   terraform show -json tfplan.binary > tfplan.json
   ```
2. Write Rego Policy (`policy/s3_encryption.rego`):
   ```rego
   package main

   deny[msg] {
       resource := input.resource_changes[_]
       resource.type == "aws_s3_bucket"
       not resource.change.after.server_side_encryption_configuration
       msg := sprintf("S3 Bucket '%v' must have server-side encryption enabled.", [resource.name])
   }
   ```
3. Test in CI:
   ```bash
   conftest test tfplan.json -p policy/
   ```
   If any non-compliant resource is found, Conftest exits with non-zero and blocks the pipeline.

---

### **45. What is the difference between Crossplane and Terraform when managing multi-cloud resources at enterprise scale?**
**Answer:**
- **Terraform:** Pipeline-driven, batch execution. Drift is only detected when someone runs a pipeline. Developers must learn HCL and state management.
- **Crossplane:** Continuous control plane loop running inside Kubernetes.
  - Actively checks for drift every few minutes and automatically reverts unauthorized manual changes.
  - Integrates natively with Kubernetes RBAC, GitOps (ArgoCD), and K8s CRDs.
  - Higher operational overhead (requires managing Kubernetes control planes).

---

### **46. How do you structure a production-grade multi-environment Terragrunt repository?**
**Answer:**
```
root/
├── terragrunt.hcl               # Root config: S3 backend + provider generation
├── environments/
│   ├── dev/
│   │   ├── env.hcl              # env = "dev", tags
│   │   ├── vpc/terragrunt.hcl
│   │   └── eks/terragrunt.hcl   # dependencies = ["../vpc"]
│   ├── staging/
│   └── prod/
└── modules/                     # Pure Terraform source modules
    ├── vpc/
    └── eks/
```

---

### **47. How do you handle circular dependencies between Terraform modules?**
**Answer:**
Circular dependencies occur when Module A needs an output from Module B, while Module B needs an output from Module A (e.g., Security Group A referencing Security Group B).

**Solutions:**
1. **Split the Resource:** Decompose the circular resource into standalone items. For security groups, create the security group containers in modules first, and define the `aws_security_group_rule` resources in a separate step.
2. **Pass Resource IDs instead of Modules:** Refactor modules to accept external IDs rather than managing cross-resource linkages internally.

---

### **48. How do you securely handle zero-trust secrets injection in Terraform without writing credentials to disk?**
**Answer:**
1. **HashiCorp Vault Dynamic Secrets Provider:** Configure Terraform to request short-lived, dynamic cloud credentials generated on-demand by Vault:
   ```hcl
   provider "vault" { address = "https://vault.company.com" }
   data "vault_aws_access_credentials" "creds" {
     backend = "aws"
     role    = "terraform-deployment-role"
   }
   provider "aws" {
     access_key = data.vault_aws_access_credentials.creds.access_key
     secret_key = data.vault_aws_access_credentials.creds.secret_key
   }
   ```
2. Dynamic credentials expire automatically in 1 hour; no static keys ever exist.

---

### **49. What is Terraform Provider Protocol v6 and the Terraform Plugin Framework?**
**Answer:**
- **Plugin Framework (modern Go SDK):** Replaced legacy `terraform-plugin-sdk/v2`.
- Native support for nested object types, custom types, structured attribute validation, dynamic types, and simplified error handling.
- Uses Protocol Buffers / gRPC to communicate efficiently between the Terraform CLI core and provider binary processes.

---

### **50. Scenario: An engineer accidentally committed a production Terraform state file containing database passwords and private keys to a private GitHub repository. Walk through the complete response procedure.**
**Answer:**
1. **Rotate All Credentials Immediately:** Treat all secrets contained within that state file as compromised. Rotate database master passwords, private TLS keys, and IAM credentials in the cloud provider.
2. **Purge State from Git History:**
   - Use `git-filter-repo` to permanently strip the `.tfstate` file from all commits, branches, and tags.
   - Force-push clean history: `git push origin --force --all`.
3. **Configure Remote Backend:** Move the state file to a secure, encrypted S3 bucket with DynamoDB locking.
4. **Enforce Repository Prevention:**
   - Add `*.tfstate` and `*.tfstate.*` to `.gitignore`.
   - Configure GitHub Secret Scanning and pre-commit hooks (`gitleaks`).
