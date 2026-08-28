# 🔒 Security Policy

The **DevOps Interview Questions** project takes the security of our repository, code samples, automation scripts, and community seriously. This document outlines our security policies, how to report security vulnerabilities, and guidelines for secret management.

---

## 🛡️ Supported Versions

Because this repository is an educational and documentation resource, security updates and content fixes are applied continuously to the latest version on the `main` branch.

| Version / Branch | Supported | Notes |
| :--- | :---: | :--- |
| `main` | ✅ | Actively maintained and updated |
| Legacy releases / tags | ❌ | Please refer to `main` for latest corrections |

---

## 🚨 Reporting a Vulnerability

If you discover a security vulnerability, security flaw in code examples, or accidentally exposed secrets/tokens in this repository, please **do NOT report it publicly via GitHub Issues, Discussions, or Pull Requests**.

### How to Report Privately:
1. **GitHub Private Vulnerability Reporting (Preferred):**
   - Navigate to the **Security** tab of the repository on GitHub.
   - Click on **Advisories** and then click **"Report a vulnerability"**.
   - Provide detailed information, including the affected file, line number, potential risk, and suggested remediation.

2. **Direct Contact:**
   - If Private Vulnerability Reporting is unavailable, reach out directly to the project maintainers via GitHub Profile contact options.

### What to Include in Your Report:
- A clear description of the vulnerability or security concern.
- Steps to reproduce or the exact location in code/documentation.
- Potential impact or exploit scenario.
- Suggested fix or remediation patch (if available).

---

## ⏱️ Response Timelines & SLAs

We are committed to handling all reported vulnerabilities responsibly and promptly:

- **Initial Acknowledgment:** Within **48 hours** of report receipt.
- **Triage & Assessment:** Within **5 business days** to validate the report.
- **Fix & Disclosure:** Coordinated release and disclosure within **14 business days** (or sooner for high-severity issues).

---

## 🔑 Secret Scanning & Credential Safety

> [!CAUTION]
> **Never commit real secrets, API keys, passwords, private keys, or cloud access tokens into this repository.**

1. **Placeholder Convention:**
   All documentation, configuration files, and code examples must strictly use sanitized placeholder values:
   - AWS Account ID: `123456789012`
   - Domains: `example.com`, `company.internal`
   - IP Addresses: RFC 5737 documentation blocks (`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`) or private RFC 1918 blocks (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`).
   - Tokens / Passwords: `CHANGE_ME_IN_PRODUCTION`, `vault_token_example`

2. **Accidental Leak Response:**
   If credentials or secrets are inadvertently committed:
   - **Immediately rotate/revoke the credentials** on the respective cloud provider or platform.
   - Run history rewriting tools (`git-filter-repo` / BFG Repo-Cleaner) to purge the commit from all branches and tags.
   - Contact GitHub Support to invalidate cached commit views if the repository is public.

---

## 🧑‍💻 Code Sample Security Disclaimer

All code snippets, Terraform manifests, Kubernetes YAML files, and shell scripts provided in this repository are intended for **educational and interview preparation purposes**. Before deploying any configuration to production environments:
- Review against your organization's compliance and security standards.
- Run automated security scanners (e.g., `trivy`, `checkov`, `tfsec`, `bandit`, `semgrep`).
- Apply the **Principle of Least Privilege (PoLP)** for IAM roles and network policies.

---

Thank you for helping keep our open-source community safe and secure! 🛡️
