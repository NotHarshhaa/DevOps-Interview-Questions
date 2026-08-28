# 📜 Contributing to DevOps Interview Questions (2,200+ Questions)

Thank you for your interest in contributing to the **DevOps Interview Questions** repository! 🎉 This project is a community-driven repository containing **2,200+ battle-tested, enterprise-grade interview questions and detailed answers** covering DevOps, SRE, Cloud, Kubernetes, CI/CD, IaC, Security, and System Administration.

---

## 🚀 Contribution Workflow

```
 1. Fork Repo ➔ 2. Create Branch ➔ 3. Add/Edit Content ➔ 4. Validate Formatting ➔ 5. Open PR
```

### 1. Fork & Clone the Repository
1. Click the **Fork** button on GitHub (top right).
2. Clone your fork locally:
   ```bash
   git clone https://github.com/<your-username>/DevOps-Interview-Questions.git
   cd DevOps-Interview-Questions
   ```
3. Set up the upstream remote:
   ```bash
   git remote add upstream https://github.com/NotHarshhaa/DevOps-Interview-Questions.git
   ```

### 2. Create a Feature Branch
Always create a descriptive branch for your changes:
```bash
git checkout -b feat/add-karpenter-question
```

---

## 📝 Contribution Standards & Question Format

To maintain consistency and high technical depth across all 2,200+ questions, please follow these guidelines:

### Question & Answer Structure
All interview questions must adhere to the standard numbered markdown format:

```markdown
### **<N>. Question Title or Scenario?**
**Answer:**
Detailed technical explanation covering:
- **Core Concept:** Direct, high-level summary.
- **Implementation / Code:** Production-grade YAML, Bash, Python, or Go snippet.
- **Failure Modes & Trade-offs:** Edge cases, gotchas, or mitigations.
```

### Example:
```markdown
### **42. What is Cache Penetration vs Cache Breakdown vs Cache Avalanche?**
**Answer:**
- **Cache Penetration:** Queries for non-existent keys bypass cache and hit database directly (fixed via Bloom Filters).
- **Cache Breakdown:** A single hot key expires, triggering concurrent DB queries (fixed via mutex locking / Singleflight).
- **Cache Avalanche:** Many cached keys expire simultaneously, overwhelming the database (fixed by adding randomized TTL jitter).
```

### Quality Guidelines
1. **Technical Depth:** Answers should go beyond simple definitions. Include architectural rationale, trade-offs, and failure recovery.
2. **Production-Grade Code:** Provide valid, syntactically correct YAML manifests, Terraform HCL, Bash scripts (`set -euo pipefail`), or Python/Go code.
3. **No Outdated Practices:** Use modern standards (e.g., OpenTofu, Kubernetes Gateway API, eBPF, OpenTelemetry, SLSA Level 3, Sigstore Cosign).
4. **Clean Markdown:** Use GitHub Flavored Markdown (tables, callout alerts `> [!NOTE]`, code fences with language syntax).

---

## 🔍 Validation & Testing

Before submitting your Pull Request, verify that your markdown is properly formatted and question headers follow the regex pattern:

```bash
# Verify question syntax matching
git diff --name-only | grep "\.md$" | xargs -I {} grep -E "^###\s+\*\*?[0-9]+\." {}
```

---

## 💬 Commit Guidelines (Conventional Commits)

We enforce the [Conventional Commits](https://www.conventionalcommits.org/) specification:

- `feat(k8s): add 5 new Gateway API interview questions`
- `fix(ci-cd): correct OIDC token configuration example`
- `docs(core): update FinOps unit cost economics definition`
- `refactor(bash): optimize script error handling example`

---

## 📬 Submitting a Pull Request

1. Push your branch to your GitHub fork:
   ```bash
   git push origin feat/add-karpenter-question
   ```
2. Open a **Pull Request** against `main` on the upstream repository.
3. Complete the PR template with:
   - Summary of changes.
   - Module(s) modified.
   - Confirmation of markdown formatting and syntax checks.

---

## 📄 Code of Conduct
By participating in this project, you agree to abide by our [Code of Conduct](CODE_OF_CONDUCT.md).

Thank you for helping make this the best DevOps interview preparation resource! 🚀
