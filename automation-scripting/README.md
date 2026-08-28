# **Automation & Scripting - DevOps Interview Questions**

Welcome to the **Automation & Scripting** interview questions master guide. This module provides in-depth, exhaustive technical explanations, complete production-ready Bash scripts (`set -euo pipefail`, signal traps, `jq`, `yq`, `flock`), Python for DevOps (`boto3`, official Kubernetes client library, Prometheus scrapers), CLI tool development, and automated operational runbooks.

---

## 🟢 **Beginner Level (Questions 1–20)**

### **1. Why is `set -euo pipefail` considered the gold standard in production Bash scripts? Explain each flag with concrete failure scenarios.**

**Detailed Answer:**
By default, Bash ignores errors, uses uninitialized variables as empty strings, and only evaluates the exit code of the final command in a pipeline.

#### **1. `-e` (`errexit`):**
- *Behavior:* Immediately exits the script if any command returns a non-zero exit status.
- *Without `-e`:* If `cd /nonexistent/dir` fails, a subsequent command like `rm -rf *` will execute in the current working directory, deleting critical project files.

#### **2. `-u` (`nounset`):**
- *Behavior:* Exits immediately if an unset/undefined variable is referenced.
- *Without `-u`:* If `TARGET_DIR` is unset, running `rm -rf /${TARGET_DIR}` expands to `rm -rf /`, destroying the root filesystem.

#### **3. `-o pipefail`:**
- *Behavior:* Ensures that a pipeline (e.g., `cmd1 | cmd2 | cmd3`) fails if **any** command in the chain fails.
- *Without `-o pipefail`:* In `failing_build_command | tee build.log`, `failing_build_command` exits with `1`, but `tee` exits with `0`, masking the build failure from CI/CD.

---

### **2. How do you handle cleanup and signal trapping in Bash with `trap`? Write a complete script.**

**Detailed Answer:**
The `trap` command registers cleanup functions executed when the script exits or receives interruption signals (`SIGINT`, `SIGTERM`, `EXIT`):

```bash
#!/usr/bin/env bash
set -euo pipefail

TEMP_DIR=$(mktemp -d /tmp/deploy_XXXXXX)

cleanup() {
    local exit_code=$?
    echo "[INFO] Cleaning up temporary directory: ${TEMP_DIR} (Exit code: ${exit_code})..."
    rm -rf "${TEMP_DIR}"
}
# Trap normal exit and interrupt signals
trap cleanup EXIT INT TERM

echo "[INFO] Processing payload inside ${TEMP_DIR}..."
# Script workflow logic here
```

---

### **3. Compare single quotes `' '` vs double quotes `" "` in Bash.**

**Detailed Answer:**
- **Single Quotes (`' '`):** Preserves the literal value of every character inside. Variable expansion (`$VAR`), command substitution (`$(cmd)`), and arithmetic evaluation (`$((1+1))`) are completely disabled.
- **Double Quotes (`" "`):** Enables variable expansion (`$VAR`), command substitution (`$(cmd)`), and arithmetic evaluation while preserving whitespace and preventing word splitting.

---

### **4. How do you parse and filter JSON in Bash using `jq`? Provide real-world examples.**

**Detailed Answer:**
```bash
# 1. Extract pod names in 'Running' state from Kubernetes output
kubectl get pods -n production -o json | jq -r \
  '.items[] | select(.status.phase=="Running") | .metadata.name'

# 2. Extract nested database host from JSON string
echo '{"database": {"primary": {"host": "10.0.0.1", "port": 5432}}}' | \
  jq -r '.database.primary.host'

# 3. Construct dynamic JSON payload
jq -n --arg env "prod" --arg ver "1.2.0" '{environment: $env, version: $ver, deployed: true}'
```

---

### **5. How do you modify YAML files safely in CLI using `yq`?**

**Detailed Answer:**
```bash
# 1. Update replica count in-place
yq eval '.spec.replicas = 5' -i deployment.yaml

# 2. Extract container image name
IMAGE=$(yq eval '.spec.template.spec.containers[0].image' deployment.yaml)

# 3. Merge two YAML files
yq eval-all 'select(fileIndex == 0) * select(fileIndex == 1)' base.yaml overlay.yaml
```

---

### **6. Compare `[[ ... ]]` vs `[ ... ]` in Bash.**

**Detailed Answer:**
- **`[ ... ]` (POSIX Test):** Older standard. Prone to word splitting on empty variables and requires escaping logical operators (`-a`, `-o`).
- **`[[ ... ]]` (Bash Extended Test):** Modern standard. Supports regex pattern matching (`=~`), logical operators (`&&`, `||`) without escaping, and does not perform word splitting on unset variables.

---

### **7. Explain common Linux / Bash exit codes.**

**Detailed Answer:**
- `0`: Success / Normal completion.
- `1`: General error / catchall.
- `2`: Misuse of shell builtins (syntax error).
- `126`: Command invoked cannot execute (permission denied).
- `127`: Command not found (wrong path or missing binary).
- `130`: Script terminated by `Ctrl+C` (`SIGINT` = $128 + 2$).
- `137`: Process killed forcibly (`SIGKILL` = $128 + 9$, typical for OOMKilled).
- `143`: Process terminated gracefully (`SIGTERM` = $128 + 15$).

---

### **8. How do you run commands in parallel in Bash using `xargs`?**

**Detailed Answer:**
```bash
# Download 20 files in parallel with maximum 4 concurrent worker processes
cat urls.txt | xargs -n 1 -P 4 curl -O
```
- `-n 1`: Pass one argument per command invocation.
- `-P 4`: Run up to 4 parallel processes concurrently.

---

### **9. Why is Python `venv` mandatory for DevOps automation scripts?**

**Detailed Answer:**
Creates isolated Python environments with independent package dependencies, preventing version conflicts between different automation tools and avoiding polluting system-wide OS Python libraries (`/usr/lib/python`), which would break OS package managers (`apt`/`dnf`).

---

### **10. What is `boto3` and what is its credential resolution hierarchy?**

**Detailed Answer:**
`boto3` is the official AWS SDK for Python.
**Authentication Hierarchy (Evaluated sequentially):**
1. Explicit credentials passed to `boto3.client(..., aws_access_key_id=...)`.
2. Environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`).
3. Shared credentials file (`~/.aws/credentials`).
4. AWS IAM Roles for Service Accounts (IRSA) / ECS Task Roles / EC2 Instance Metadata (`IMDSv2`).

---

### **11. Compare `boto3.client` vs `boto3.resource`.**

**Detailed Answer:**
- **`boto3.client`:** Low-level, 1-to-1 mapping to AWS REST APIs. Returns raw Python dictionaries. Faster, supports 100% of AWS services and API parameters.
- **`boto3.resource`:** High-level, object-oriented abstraction (`s3.Bucket('my-bucket').objects.all()`). Easier syntax, but does not support all new AWS services.

---

### **12. How do you parse CLI arguments in Python using `argparse`?**

**Detailed Answer:**
```python
import argparse

parser = argparse.ArgumentParser(description="Audit and clean up AWS EBS snapshots")
parser.add_argument("--retention-days", type=int, default=30, help="Days to retain snapshots")
parser.add_argument("--dry-run", action="store_true", help="Preview actions without deleting")

args = parser.parse_args()
print(f"Cleanup retention: {args.retention_days} days. Dry run: {args.dry_run}")
```

---

### **13. Compare `os.system()`, `subprocess.run()`, and `subprocess.Popen()` in Python.**

**Detailed Answer:**
- **`os.system()` (Legacy/Unsafe):** Executes command in a subshell. Vulnerable to shell injection, cannot capture stdout/stderr easily.
- **`subprocess.run()` (Standard):** Synchronous execution. Waits for command to complete and returns `CompletedProcess` with captured stdout, stderr, and return code.
- **`subprocess.Popen()` (Advanced):** Asynchronous, non-blocking process execution. Allows streaming stdout/stderr in real time and interacting with stdin.

---

### **14. How do you safely read and parse environment variables in Python?**

**Detailed Answer:**
```python
import os

# Safe lookup with fallback default
LOG_LEVEL = os.getenv("LOG_LEVEL", "INFO")

# Mandatory lookup (raises KeyError if missing)
DATABASE_URL = os.environ["DATABASE_URL"]
```

---

### **15. How do you parse and write YAML in Python safely using `PyYAML`?**

**Detailed Answer:**
```python
import yaml

with open("config.yaml", "r") as f:
    config = yaml.safe_load(f)  # Always safe_load to prevent arbitrary code execution!

config["replicas"] = 5

with open("config.yaml", "w") as f:
    yaml.dump(config, f, default_flow_style=False)
```

---

### **16. How do you implement retry logic with exponential backoff in Python?**

**Detailed Answer:**
```python
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
import requests

@retry(
    stop=stop_after_attempt(5),
    wait=wait_exponential(multiplier=1, min=2, max=10),
    retry=retry_if_exception_type(requests.RequestException)
)
def fetch_api_data(url: str):
    response = requests.get(url, timeout=5)
    response.raise_for_status()
    return response.json()
```

---

### **17. How do you check if a port is open on a remote server using Bash without external tools?**

**Detailed Answer:**
```bash
# Using native Bash pseudo-device /dev/tcp
if timeout 2 bash -c "</dev/tcp/10.0.0.1/5432" &>/dev/null; then
    echo "PostgreSQL port 5432 is OPEN"
else
    echo "PostgreSQL port 5432 is CLOSED"
fi
```

---

### **18. How do you find and delete files older than $N$ days in Linux?**

**Detailed Answer:**
```bash
# Find and delete log files older than 14 days
find /var/log/app/ -type f -name "*.log" -mtime +14 -delete
```

---

### **19. Compare `awk` vs `sed` vs `grep`.**

**Detailed Answer:**
- **`grep`:** Pattern matching and line filtering using regex.
- **`sed` (Stream Editor):** Text transformation, substitution, and inline file replacement (`sed -i 's/old/new/g' file.txt`).
- **`awk`:** Complete text-processing language operating on structured column data (`awk '{print $1, $9}' access.log`).

---

### **20. What is a Shebang (`#!/usr/bin/env bash`)?**

**Detailed Answer:**
Instructs the Linux kernel which interpreter to invoke. `#!/usr/bin/env bash` is preferred over `#!/bin/bash` because it dynamically resolves the `bash` executable from the user's `$PATH`, ensuring portability across Linux, macOS, and BSD.

---

## 🟡 **Intermediate Level (Questions 21–40)**

### **21. Write a Python script using `boto3` to audit and delete unattached EBS volumes older than 30 days.**

**Detailed Answer:**
```python
import boto3
from datetime import datetime, timezone, timedelta

def cleanup_unattached_ebs(days_threshold=30, dry_run=True):
    ec2 = boto3.client("ec2", region_name="us-east-1")
    cutoff_date = datetime.now(timezone.utc) - timedelta(days=days_threshold)
    
    # Filter only unattached (available) volumes
    response = ec2.describe_volumes(Filters=[{"Name": "status", "Values": ["available"]}])
    
    for volume in response.get("Volumes", []):
        vol_id = volume["VolumeId"]
        create_time = volume["CreateTime"]
        size_gb = volume["Size"]
        
        if create_time < cutoff_date:
            print(f"[ACTION] Volume {vol_id} ({size_gb}GB, created {create_time}) eligible for deletion.")
            if not dry_run:
                ec2.delete_volume(VolumeId=vol_id)
                print(f"[DELETED] Successfully deleted {vol_id}")

if __name__ == "__main__":
    cleanup_unattached_ebs(days_threshold=30, dry_run=False)
```

---

### **22. Write a Python script using the official `kubernetes` client to trigger a rolling restart of a deployment.**

**Detailed Answer:**
```python
from kubernetes import client, config
from datetime import datetime, timezone

def restart_deployment(namespace: str, deployment_name: str):
    try:
        config.load_incluster_config()
    except config.ConfigException:
        config.load_kube_config()

    apps_v1 = client.AppsV1Api()
    now = datetime.now(timezone.utc).isoformat()
    body = {
        "spec": {
            "template": {
                "metadata": {
                    "annotations": {
                        "kubectl.kubernetes.io/restartedAt": now
                    }
                }
            }
        }
    }
    apps_v1.patch_namespaced_deployment(name=deployment_name, namespace=namespace, body=body)
    print(f"Triggered rolling restart for {deployment_name} in {namespace}")

if __name__ == "__main__":
    restart_deployment("production", "payment-service")
```

---

### **23. How do you implement robust file locking in Bash with `flock` to prevent duplicate script execution?**

**Detailed Answer:**
```bash
#!/usr/bin/env bash
set -euo pipefail

LOCK_FILE="/var/lock/backup_job.lock"
exec 200>"${LOCK_FILE}"

if ! flock -n 200; then
    echo "[WARN] Another instance of the backup job is already running. Exiting."
    exit 1
fi

echo "[INFO] Lock acquired. Running backup workflow..."
sleep 10
echo "[INFO] Job completed successfully."
```

---

### **24. How do you validate JSON payloads against a JSON Schema in Python?**

**Detailed Answer:**
```python
from jsonschema import validate, ValidationError

schema = {
    "type": "object",
    "properties": {
        "service": {"type": "string"},
        "replicas": {"type": "integer", "minimum": 1, "maximum": 100},
        "environment": {"type": "string", "enum": ["dev", "staging", "prod"]}
    },
    "required": ["service", "replicas", "environment"]
}

payload = {"service": "auth-api", "replicas": 3, "environment": "prod"}

try:
    validate(instance=payload, schema=schema)
    print("Payload is valid!")
except ValidationError as e:
    print(f"Validation failed: {e.message}")
```

---

### **25. How do you scrape and parse Prometheus metrics endpoints using Python?**

**Detailed Answer:**
```python
import requests
from prometheus_client.parser import text_string_to_metric_families

def get_metric_samples(endpoint_url: str, target_metric: str):
    response = requests.get(endpoint_url, timeout=5)
    response.raise_for_status()
    
    for family in text_string_to_metric_families(response.text):
        if family.name == target_metric:
            for sample in family.samples:
                print(f"Sample: {sample.name} | Labels: {sample.labels} | Value: {sample.value}")

if __name__ == "__main__":
    get_metric_samples("http://localhost:9090/metrics", "http_requests_total")
```

---

### **26. How do you implement structured JSON logging in Python for DevOps CLI tools?**

**Detailed Answer:**
```python
import logging
import json
from datetime import datetime, timezone

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_obj = {
            "timestamp": datetime.now(timezone.utc).isoformat(),
            "level": record.levelname,
            "message": record.getMessage(),
            "logger": record.name
        }
        return json.dumps(log_obj)

logger = logging.getLogger("devops-cli")
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)

logger.info("Automation task initialized successfully.")
```

---

### **27. How do you pass multiline strings and heredocs safely in Bash without variable expansion?**

**Detailed Answer:**
Quoting the delimiter (`'EOF'`) disables variable expansion:
```bash
cat <<'EOF' > /tmp/template.sh
#!/bin/bash
# $USER and $(date) will NOT expand here; written as literal text
echo "Welcome $USER! Generated at $(date)"
EOF
```

---

### **28. How do you read a large file line-by-line in Bash without memory exhaustion?**

**Detailed Answer:**
```bash
#!/usr/bin/env bash
while IFS= read -r line || [[ -n "$line" ]]; do
    echo "Processing line: ${line}"
done < "large_access_log.txt"
```

---

### **29. How do you execute remote commands concurrently across 50 servers using Python `asyncio` and `asyncssh`?**

**Detailed Answer:**
```python
import asyncio
import asyncssh

async def run_remote_command(host: str, command: str):
    try:
        async with asyncssh.connect(host, username="ubuntu", known_hosts=None) as conn:
            result = await conn.run(command, check=True)
            print(f"[{host}] SUCCESS: {result.stdout.strip()}")
    except Exception as e:
        print(f"[{host}] FAILED: {e}")

async def main(hosts: list[str]):
    tasks = [run_remote_command(host, "uptime") for host in hosts]
    await asyncio.gather(*tasks)

if __name__ == "__main__":
    servers = [f"10.0.0.{i}" for i in range(1, 50)]
    asyncio.run(main(servers))
```

---

### **30. How do you interact with HashiCorp Vault API in Python using `hvac`?**

**Detailed Answer:**
```python
import hvac
import os

client = hvac.Client(
    url="https://vault.company.com:8200",
    token=os.environ["VAULT_TOKEN"]
)

read_response = client.secrets.kv.v2.read_secret_version(
    mount_point="secret",
    path="production/database"
)

password = read_response["data"]["data"]["password"]
print("Successfully retrieved database secret from Vault.")
```

---

### **31. Compare string interpolation in Python (f-strings vs `.format()` vs `%`).**

**Detailed Answer:**
- **f-strings (`f"{var}"` - Python 3.6+):** Modern standard. Evaluated at runtime, fastest execution, cleanest syntax.
- **`.format()` (`"{}".format(var)`):** Older method. More verbose.
- **`%` operator (`"%s" % var`):** Legacy C-style formatting. Prone to type mismatch errors.

---

### **32. How do you implement a Token Bucket Rate Limiter in Python?**

**Detailed Answer:**
```python
import time

class RateLimiter:
    def __init__(self, max_per_second: int):
        self.interval = 1.0 / max_per_second
        self.last_called = 0.0

    def wait(self):
        elapsed = time.time() - self.last_called
        if elapsed < self.interval:
            time.sleep(self.interval - elapsed)
        self.last_called = time.time()
```

---

### **33. What are Python Generators (`yield`) and why are they crucial for processing gigabytes of log files?**

**Detailed Answer:**
Generators produce items on-demand (lazy evaluation) rather than allocating lists in memory:
```python
def stream_large_log(file_path: str):
    with open(file_path, "r") as f:
        for line in f:
            if "ERROR 500" in line:
                yield line.strip()

# Memory footprint remains under 10MB even on a 50GB log file
for error_line in stream_large_log("/var/log/huge_app.log"):
    process_error(error_line)
```

---

### **34. How do you write a Bash function that accepts named parameters (flags)?**

**Detailed Answer:**
```bash
deploy_app() {
    local env=""
    local version=""
    while [[ $# -gt 0 ]]; do
        case "$1" in
            -e|--environment) env="$2"; shift 2 ;;
            -v|--version)     version="$2"; shift 2 ;;
            *) echo "Unknown parameter: $1"; return 1 ;;
        esac
    done
    echo "Deploying version ${version} to ${env}..."
}

deploy_app --environment production --version v2.4.1
```

---

### **35. How do you convert Bash arrays into JSON using `jq`?**

**Detailed Answer:**
```bash
MY_ARRAY=("us-east-1" "us-west-2" "eu-west-1")
JSON_OUTPUT=$(printf '%s\n' "${MY_ARRAY[@]}" | jq -R . | jq -s .)
echo "${JSON_OUTPUT}"
# Output: ["us-east-1", "us-west-2", "eu-west-1"]
```

---

### **36. Compare shallow copy vs deep copy in Python.**

**Detailed Answer:**
- **Shallow Copy (`copy.copy()`):** Copies top-level object structure, but nested objects are still referenced by memory address. Modifying a nested child alters the original.
- **Deep Copy (`copy.deepcopy()`):** Recursively clones the entire object and all nested children, creating a 100% independent duplicate.

---

### **37. How do you calculate file SHA256 checksums in Python?**

**Detailed Answer:**
```python
import hashlib

def get_file_sha256(filepath: str) -> str:
    sha256 = hashlib.sha256()
    with open(filepath, "rb") as f:
        while chunk := f.read(8192):
            sha256.update(chunk)
    return sha256.hexdigest()
```

---

### **38. How do you debug Bash scripts in real time?**

**Detailed Answer:**
1. Run with debug flags: `bash -x script.sh` (prints each command before execution).
2. Enable tracing for specific blocks inside script:
   ```bash
   set -x  # Enable tracing
   problematic_command
   set +x  # Disable tracing
   ```

---

### **39. What is Python `dataclass` and why is it preferred for infrastructure models?**

**Detailed Answer:**
`dataclass` (Python 3.7+) automatically generates `__init__`, `__repr__`, and `__eq__` methods for structured data classes:
```python
from dataclasses import dataclass

@dataclass(frozen=True)
class ServerConfig:
    hostname: str
    ip_address: str
    cpu_cores: int
    memory_gb: int
```

---

### **40. What is `pydantic` and how is it used for configuration validation?**

**Detailed Answer:**
Enforces strict data validation, type conversion, and environment variable loading based on Python type hints:
```python
from pydantic_settings import BaseSettings

class AppSettings(BaseSettings):
    db_host: str
    db_port: int = 5432
    debug_mode: bool = False

    class Config:
        env_file = ".env"

settings = AppSettings()
```

---

## 🔴 **Advanced & Scenario-Based Level (Questions 41–50)**

### **41. Scenario: Write a complete Python CLI tool using `boto3` that scans an AWS account, finds all unencrypted S3 buckets, enables KMS default encryption, and outputs an audit report.**

**Detailed Answer:**
```python
import boto3
from botocore.exceptions import ClientError
import json

def remediate_s3_encryption():
    s3 = boto3.client("s3")
    buckets = s3.list_buckets().get("Buckets", [])
    report = []

    for bucket in buckets:
        name = bucket["Name"]
        try:
            s3.get_bucket_encryption(Bucket=name)
            report.append({"bucket": name, "status": "ALREADY_ENCRYPTED"})
        except ClientError as e:
            if e.response["Error"]["Code"] == "ServerSideEncryptionConfigurationNotFoundError":
                print(f"[REMEDIATING] Enabling AES256 encryption on bucket: {name}")
                s3.put_bucket_encryption(
                    Bucket=name,
                    ServerSideEncryptionConfiguration={
                        "Rules": [{
                            "ApplyServerSideEncryptionByDefault": {
                                "SSEAlgorithm": "AES256"
                            }
                        }]
                    }
                )
                report.append({"bucket": name, "status": "REMEDIATED"})
            else:
                report.append({"bucket": name, "status": f"ERROR: {str(e)}"})

    print(json.dumps(report, indent=2))

if __name__ == "__main__":
    remediate_s3_encryption()
```

---

### **42. Scenario: Write a robust Bash script that detects crashing Kubernetes Pods, captures previous logs, uploads to S3, and posts a Slack webhook alert.**

**Detailed Answer:**
```bash
#!/usr/bin/env bash
set -euo pipefail

NAMESPACE="${1:-production}"
S3_BUCKET="s3://company-incident-logs-bucket"
SLACK_WEBHOOK_URL="https://hooks.slack.com/services/T00/B00/X00"
INCIDENT_DIR=$(mktemp -d)

cleanup() {
    rm -rf "${INCIDENT_DIR}"
}
trap cleanup EXIT

CRASHING_PODS=$(kubectl get pods -n "${NAMESPACE}" -o json | jq -r \
  '.items[] | select(.status.containerStatuses[]?.state.waiting.reason=="CrashLoopBackOff") | .metadata.name' | sort -u)

if [[ -z "${CRASHING_PODS}" ]]; then
    echo "[INFO] No crashing pods found in ${NAMESPACE}."
    exit 0
fi

for POD in ${CRASHING_PODS}; do
    LOG_FILE="${INCIDENT_DIR}/${POD}_previous.log"
    echo "[INFO] Capturing crash logs for ${POD}..."
    kubectl logs "${POD}" -n "${NAMESPACE}" --previous > "${LOG_FILE}" 2>&1 || true
    
    aws s3 cp "${LOG_FILE}" "${S3_BUCKET}/${NAMESPACE}/${POD}.log"
    
    PAYLOAD=$(jq -n \
      --arg pod "$POD" \
      --arg ns "$NAMESPACE" \
      --arg s3 "${S3_BUCKET}/${NAMESPACE}/${POD}.log" \
      '{text: "🚨 *Alert:* Pod `\($pod)` in `\($ns)` is CrashLoopBackOff. Logs: `\($s3)`"}')
      
    curl -s -X POST -H 'Content-type: application/json' --data "${PAYLOAD}" "${SLACK_WEBHOOK_URL}"
done
```

---

### **43. How do you implement a ThreadPoolExecutor in Python for concurrent API operations?**

**Detailed Answer:**
```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import requests

def check_endpoint_health(url: str) -> tuple[str, bool]:
    try:
        res = requests.get(url, timeout=3)
        return (url, res.status_code == 200)
    except requests.RequestException:
        return (url, False)

endpoints = [f"https://service-{i}.internal.company.com/healthz" for i in range(50)]

with ThreadPoolExecutor(max_workers=10) as executor:
    futures = {executor.submit(check_endpoint_health, url): url for url in endpoints}
    for future in as_completed(futures):
        url, healthy = future.result()
        print(f"Endpoint: {url} | Healthy: {healthy}")
```

---

### **44. How do you execute remote commands via AWS SSM Run Command using Python `boto3`?**

**Detailed Answer:**
```python
import boto3
import time

def run_ssm_command(instance_ids: list[str], shell_command: str):
    ssm = boto3.client("ssm", region_name="us-east-1")
    response = ssm.send_command(
        InstanceIds=instance_ids,
        DocumentName="AWS-RunShellScript",
        Parameters={"commands": [shell_command]}
    )
    command_id = response["Command"]["CommandId"]
    
    time.sleep(3)
    for instance_id in instance_ids:
        output = ssm.get_command_invocation(CommandId=command_id, InstanceId=instance_id)
        print(f"[{instance_id}] Status: {output['Status']}")
        print(output["StandardOutputContent"])

if __name__ == "__main__":
    run_ssm_command(["i-0123456789abcdef0"], "df -h && uptime")
```

---

### **45. How do you sanitize user inputs in Bash scripts to prevent Shell Injection?**

**Detailed Answer:**
Never pass untrusted inputs to `eval` or execute inside subshells without strict whitelist validation:
```bash
USER_INPUT="$1"
if [[ ! "${USER_INPUT}" =~ ^[a-zA-Z0-9_-]+$ ]]; then
    echo "[ERROR] Invalid input character detected." >&2
    exit 1
fi
```

---

### **46. How do you build an interactive CLI tool in Python using `Click`?**

**Detailed Answer:**
```python
import click

@click.group()
def cli():
    """DevOps Infrastructure Management CLI."""
    pass

@cli.command()
@click.option("--environment", "-e", required=True, type=click.Choice(["dev", "staging", "prod"]))
@click.option("--force", is_flag=True, help="Force deployment without prompt")
def deploy(environment, force):
    if not force and not click.confirm(f"Deploy to {environment}?"):
        click.echo("Aborted!")
        return
    click.echo(f"Deploying to {environment}...")

if __name__ == "__main__":
    cli()
```

---

### **47. Explain Python `asyncio` and the single-threaded cooperative multitasking Event Loop.**

**Detailed Answer:**
`asyncio` uses an **Event Loop** running on a single thread. When an I/O operation starts (`await client.get(...)`), the coroutine yields control back to the event loop, which executes other ready coroutines while waiting for network socket responses, handling thousands of concurrent connections with minimal RAM.

---

### **48. How do you parse and filter Linux `/var/log/syslog` using Python regex?**

**Detailed Answer:**
```python
import re

SYSLOG_PATTERN = re.compile(
    r"^(?P<timestamp>\w{3}\s+\d+\s+\d+:\d+:\d+)\s+(?P<host>\S+)\s+(?P<process>[\w\-\/\.]+)(?:\[(?P<pid>\d+)\])?:\s+(?P<message>.*)$"
)

with open("/var/log/syslog", "r") as f:
    for line in f:
        match = SYSLOG_PATTERN.match(line)
        if match:
            data = match.groupdict()
            if "oom-killer" in data["message"].lower():
                print(f"OOM at {data['timestamp']} on {data['host']}: {data['message']}")
```

---

### **49. How do you profile Python script performance and memory consumption?**

**Detailed Answer:**
- **CPU Profiling:** `python -m cProfile -s cumtime my_script.py`
- **Memory Profiling:** Using `tracemalloc`:
```python
import tracemalloc
tracemalloc.start()
# Execute workload
current, peak = tracemalloc.get_traced_memory()
print(f"Peak memory: {peak / 10**6:.2f} MB")
tracemalloc.stop()
```

---

### **50. Scenario: Write a script to drain and replace Kubernetes worker nodes in an AWS ASG during a rolling AMI update.**

**Detailed Answer:**
```python
import boto3
from kubernetes import client, config
import time

def rolling_node_update(asg_name: str):
    config.load_kube_config()
    core_v1 = client.CoreV1Api()
    asg = boto3.client("autoscaling", region_name="us-east-1")
    ec2 = boto3.client("ec2", region_name="us-east-1")

    res = asg.describe_auto_scaling_groups(AutoScalingGroupNames=[asg_name])
    instances = res["AutoScalingGroups"][0]["Instances"]

    for inst in instances:
        instance_id = inst["InstanceId"]
        ec2_res = ec2.describe_instances(InstanceIds=[instance_id])
        node_name = ec2_res["Reservations"][0]["Instances"][0]["PrivateDnsName"]

        print(f"[CORDON] Cordoning node {node_name}...")
        core_v1.patch_node(node_name, {"spec": {"unschedulable": True}})

        print(f"[TERMINATE] Terminating instance {instance_id} in ASG...")
        asg.terminate_instance_in_auto_scaling_group(
            InstanceId=instance_id,
            ShouldDecrementDesiredCapacity=False
        )
        time.sleep(60) # Wait for replacement node to join cluster
```
