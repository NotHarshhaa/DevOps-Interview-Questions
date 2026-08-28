# **Automation & Scripting - DevOps Interview Questions**

Welcome to the **Automation & Scripting** interview questions module. This section covers production-grade Bash (`set -euo pipefail`, signal handling, `jq`, `yq`), Python for DevOps (`boto3`, Kubernetes client library, Prometheus scrapers), CLI tool development, and automated operational runbooks.

---

## 🟢 **Beginner Level (Questions 1–20)**

### **1. Why is `set -euo pipefail` considered the gold standard in production Bash scripts?**
**Answer:**
By default, Bash scripts continue executing even after errors occur, masking failures.
- **`-e` (`errexit`):** Immediately exits if any command returns a non-zero exit status.
- **`-u` (`nounset`):** Exits if an undefined variable is referenced (prevents catastrophic bugs like `rm -rf /${UNSET_VAR}`).
- **`-o pipefail`:** Ensures that a pipeline (e.g., `cmd1 | cmd2 | cmd3`) fails if *any* command in the pipeline fails, rather than only returning the exit status of the final command.

---

### **2. How do you handle cleanup and signal trapping in Bash with `trap`?**
**Answer:**
The `trap` command registers cleanup functions executed when the script exits or receives interruption signals (`SIGINT`, `SIGTERM`, `EXIT`):
```bash
#!/usr/bin/env bash
set -euo pipefail

TEMP_DIR=$(mktemp -d)

cleanup() {
    echo "[INFO] Cleaning up temporary directory: ${TEMP_DIR}..."
    rm -rf "${TEMP_DIR}"
}
# Execute cleanup function on exit or termination
trap cleanup EXIT INT TERM

echo "Processing data inside ${TEMP_DIR}..."
```

---

### **3. What is the difference between single quotes `' '` and double quotes `" "` in Bash?**
**Answer:**
- **Single Quotes (`' '`):** Preserves the literal value of every character inside the quotes. Variable expansion (`$VAR`) and command substitution (`$(cmd)`) are **disabled**.
- **Double Quotes (`" "`):** Enables variable expansion (`$VAR`), command substitution (`$(cmd)`), and arithmetic evaluation (`$((1+1))`), while preserving whitespace and preventing word splitting.

---

### **4. How do you parse and filter JSON in Bash using `jq`?**
**Answer:**
Extracting fields from API responses or Kubernetes output:
```bash
# Extract names of running pods in JSON format
kubectl get pods -n production -o json | jq -r '.items[] | select(.status.phase=="Running") | .metadata.name'

# Extract specific nested value
echo '{"database": {"host": "10.0.0.1", "port": 5432}}' | jq -r '.database.host'
```

---

### **5. How do you modify YAML files safely in CLI using `yq`?**
**Answer:**
Using `yq` (YAML processor):
```bash
# Update replica count in deployment YAML in-place
yq eval '.spec.replicas = 5' -i deployment.yaml

# Extract container image from YAML
IMAGE=$(yq eval '.spec.template.spec.containers[0].image' deployment.yaml)
```

---

### **6. What is the difference between `[[ ... ]]` and `[ ... ]` in Bash?**
**Answer:**
- **`[ ... ]` (POSIX standard test):** Older, simple test command. Prone to word splitting errors and requires escaping special characters.
- **`[[ ... ]]` (Bash extended test):** Modern standard. Supports regex pattern matching (`=~`), logical operators (`&&`, `||`) without escaping, and does not perform word splitting on empty variables.

---

### **7. What are the common exit codes in Linux and Bash?**
**Answer:**
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
**Answer:**
```bash
# Download 10 URLs in parallel with maximum 4 concurrent processes
cat urls.txt | xargs -n 1 -P 4 curl -O
```
- `-n 1`: Pass one argument per command invocation.
- `-P 4`: Run up to 4 parallel processes simultaneously.

---

### **9. What is Python `virtualenv` / `venv` and why is it mandatory for DevOps automation scripts?**
**Answer:**
`venv` creates isolated Python runtime environments with independent package dependencies:
- Prevents dependency conflicts between different automation tools.
- Avoids polluting the system-wide OS Python packages (`/usr/lib/python`), preventing OS package manager breakage.

---

### **10. What is `boto3` in Python and how do you handle AWS authentication?**
**Answer:**
`boto3` is the official AWS SDK for Python.
**Authentication Hierarchy (Boto3 automatically resolves in order):**
1. Explicit credentials passed to `boto3.client(..., aws_access_key_id=...)`.
2. Environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`).
3. Shared credentials file (`~/.aws/credentials`).
4. AWS IAM Roles for Service Accounts (IRSA) / ECS Task Roles / EC2 Instance Metadata (`IMDSv2`).

---

### **11. What is the difference between `boto3.client` and `boto3.resource`?**
**Answer:**
- **`boto3.client`:** Low-level, 1-to-1 mapping to AWS REST APIs. Returns raw Python dictionaries. Faster, supports 100% of AWS services and API parameters.
- **`boto3.resource`:** High-level, object-oriented abstraction (e.g., `s3.Bucket('my-bucket').objects.all()`). Easier syntax, but does not support all new AWS services.

---

### **12. How do you parse CLI arguments in Python using `argparse`?**
**Answer:**
```python
import argparse

parser = argparse.ArgumentParser(description="Clean up old AWS snapshots")
parser.add_argument("--retention-days", type=int, default=30, help="Days to retain snapshots")
parser.add_argument("--dry-run", action="store_true", help="Preview actions without deleting")

args = parser.parse_args()
print(f"Running cleanup for older than {args.retention_days} days. Dry-run: {args.dry_run}")
```

---

### **13. What is the difference between `os.system()`, `subprocess.run()`, and `subprocess.Popen()` in Python?**
**Answer:**
- **`os.system()` (Legacy):** Executes command in a subshell. Unsafe (vulnerable to shell injection), cannot capture stdout/stderr easily.
- **`subprocess.run()` (Standard):** Recommended for synchronous execution. Waits for command to complete and returns `CompletedProcess` with captured stdout, stderr, and return code.
- **`subprocess.Popen()` (Advanced):** Asynchronous, non-blocking process execution. Allows streaming stdout/stderr in real time and interacting with stdin.

---

### **14. How do you safely read and parse environment variables in Python?**
**Answer:**
```python
import os

# Safe lookup with default fallback
LOG_LEVEL = os.getenv("LOG_LEVEL", "INFO")

# Mandatory lookup (raises KeyError if missing)
DATABASE_URL = os.environ["DATABASE_URL"]
```

---

### **15. How do you parse and write YAML in Python using `PyYAML`?**
**Answer:**
```python
import yaml

with open("config.yaml", "r") as f:
    config = yaml.safe_load(f)

config["replicas"] = 5

with open("config.yaml", "w") as f:
    yaml.dump(config, f, default_flow_style=False)
```
*(Always use `yaml.safe_load()` instead of `yaml.load()` to prevent arbitrary code execution vulnerabilities).*

---

### **16. How do you implement retry logic with exponential backoff in Python?**
**Answer:**
Using the `tenacity` library or standard `time.sleep`:
```python
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
import requests

@retry(
    stop=stop_after_attempt(5),
    wait=wait_exponential(multiplier=1, min=2, max=10),
    retry=retry_if_exception_type(requests.RequestException)
)
def fetch_data(url: str):
    response = requests.get(url, timeout=5)
    response.raise_for_status()
    return response.json()
```

---

### **17. How do you check if a port is open on a remote server using Bash?**
**Answer:**
```bash
# Using native Bash pseudo-device /dev/tcp (No external tools required)
if timeout 2 bash -c "</dev/tcp/10.0.0.1/5432" &>/dev/null; then
    echo "PostgreSQL port 5432 is OPEN"
else
    echo "PostgreSQL port 5432 is CLOSED"
fi

# Using Netcat
nc -zvw 2 10.0.0.1 5432
```

---

### **18. How do you find and delete files older than $N$ days in Linux?**
**Answer:**
```bash
# Find and delete log files older than 14 days
find /var/log/app/ -type f -name "*.log" -mtime +14 -delete
```

---

### **19. What is the difference between `awk`, `sed`, and `grep`?**
**Answer:**
- **`grep`:** Fast pattern searching and line filtering based on regular expressions.
- **`sed` (Stream Editor):** Text transformation, substitution, and inline file replacement (`sed -i 's/old/new/g' file.txt`).
- **`awk`:** Powerful text processing and data extraction programming language operating on column-based and structured tabular text.

---

### **20. What is a Shebang (`#!/usr/bin/env bash`)?**
**Answer:**
The shebang on line 1 tells the Linux kernel which interpreter to invoke to execute the script.
- `#!/usr/bin/env bash` is preferred over `#!/bin/bash` because it dynamically resolves the `bash` executable location from the user's `$PATH`, ensuring portability across Linux, macOS, and BSD systems.

---

## 🟡 **Intermediate Level (Questions 21–40)**

### **21. Write a production Python script using `boto3` to audit and delete unattached EBS volumes older than 30 days.**
**Answer:**
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
            print(f"[ACTION] Volume {vol_id} ({size_gb}GB, created {create_time}) is eligible for deletion.")
            if not dry_run:
                ec2.delete_volume(VolumeId=vol_id)
                print(f"[DELETED] Successfully deleted {vol_id}")

if __name__ == "__main__":
    cleanup_unattached_ebs(days_threshold=30, dry_run=False)
```

---

### **22. Write a Python script using the official `kubernetes` client to restart all Pods in a specific deployment.**
**Answer:**
```python
from kubernetes import client, config
from datetime import datetime, timezone

def restart_deployment(namespace: str, deployment_name: str):
    # Load in-cluster config if running in a pod, else local kubeconfig
    try:
        config.load_incluster_config()
    except config.ConfigException:
        config.load_kube_config()

    apps_v1 = client.AppsV1Api()
    
    # Update deployment annotation to trigger a rolling restart
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

### **23. How do you implement robust error handling with lock files in Bash to prevent duplicate script runs?**
**Answer:**
Using `flock` (Linux file locking mechanism):
```bash
#!/usr/bin/env bash
set -euo pipefail

LOCK_FILE="/var/lock/my_backup_job.lock"

# Open file descriptor 200 for the lock file
exec 200>"${LOCK_FILE}"

# Attempt non-blocking lock; exit immediately if already locked
if ! flock -n 200; then
    echo "[WARN] Another instance of the backup job is already running. Exiting."
    exit 1
fi

echo "[INFO] Lock acquired. Executing critical backup workflow..."
sleep 10
echo "[INFO] Job completed successfully."
```

---

### **24. How do you parse and validate JSON payloads against a JSON Schema in Python?**
**Answer:**
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
**Answer:**
```python
import requests
from prometheus_client.parser import text_string_to_metric_families

def get_metric_value(endpoint_url: str, target_metric: str):
    response = requests.get(endpoint_url, timeout=5)
    response.raise_for_status()
    
    for family in text_string_to_metric_families(response.text):
        if family.name == target_metric:
            for sample in family.samples:
                print(f"Metric: {sample.name} | Labels: {sample.labels} | Value: {sample.value}")

if __name__ == "__main__":
    get_metric_value("http://localhost:9090/metrics", "http_requests_total")
```

---

### **26. How do you implement a robust logging setup in Python for DevOps CLI tools?**
**Answer:**
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

logger = logging.getLogger("devops-tool")
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)

logger.info("Automation task initialized successfully.")
```

---

### **27. How do you pass multiline strings and heredocs safely in Bash without variable expansion?**
**Answer:**
Quoting the heredoc delimiter (`'EOF'`) disables parameter expansion and command substitution:
```bash
cat <<'EOF' > /tmp/template.sh
#!/bin/bash
# $USER and $(date) will NOT be evaluated here; written as literal strings
echo "Welcome $USER! Generated at $(date)"
EOF
```

---

### **28. How do you read a large file line-by-line in Bash without loading everything into memory?**
**Answer:**
```bash
#!/usr/bin/env bash
while IFS= read -r line || [[ -n "$line" ]]; do
    # Process each line efficiently
    echo "Processing: ${line}"
done < "large_access_log.txt"
```
- `IFS=`: Prevents stripping leading/trailing whitespace.
- `-r`: Prevents backslash escapes from being interpreted.
- `|| [[ -n "$line" ]]`: Ensures the last line is processed even if it lacks a trailing newline.

---

### **29. How do you execute remote commands securely across 100 Linux servers using Python `asyncio` and `asyncssh`?**
**Answer:**
```python
import asyncio
import asyncssh

async def run_remote_command(host: str, command: str):
    try:
        async with asyncssh.connect(host, username="ubuntu", known_hosts=None) as conn:
            result = await conn.run(command, check=True)
            print(f"[{host}] SUCCESS:\n{result.stdout.strip()}")
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
**Answer:**
```python
import hvac
import os

client = hvac.Client(
    url="https://vault.company.com:8200",
    token=os.environ["VAULT_TOKEN"]
)

# Read secret from KV v2 engine
read_response = client.secrets.kv.v2.read_secret_version(
    mount_point="secret",
    path="production/database"
)

password = read_response["data"]["data"]["password"]
print(f"Retrieved database password securely from Vault.")
```

---

### **31. What is the difference between string interpolation in Python (f-strings vs `%` vs `.format()`)?**
**Answer:**
- **f-strings (`f"{var}"` - Python 3.6+):** Modern standard. Evaluated at runtime, fastest execution, cleanest readability.
- **`.format()` (`"{}".format(var)`):** Older method. More verbose, slightly slower.
- **`%` operator (`"%s" % var`):** Legacy C-style formatting. Prone to type mismatch errors.

---

### **32. How do you implement rate limiting in a Python API client?**
**Answer:**
Using token bucket or sleep throttling:
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
**Answer:**
Generators produce items one at a time on-demand (lazy evaluation) rather than allocating the entire list in memory:
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
**Answer:**
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
**Answer:**
```bash
MY_ARRAY=("us-east-1" "us-west-2" "eu-west-1")

JSON_OUTPUT=$(printf '%s\n' "${MY_ARRAY[@]}" | jq -R . | jq -s .)
echo "${JSON_OUTPUT}"
# Output: ["us-east-1", "us-west-2", "eu-west-1"]
```

---

### **36. What is the difference between shallow copy and deep copy in Python?**
**Answer:**
- **Shallow Copy (`copy.copy()`):** Copies the top-level object structure, but nested objects/dictionaries are still referenced by memory address. Modifying a nested item alters the original.
- **Deep Copy (`copy.deepcopy()`):** Recursively clones the entire object and all nested children, creating a 100% independent duplicate in memory.

---

### **37. How do you calculate file MD5/SHA256 checksums in Python?**
**Answer:**
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

### **38. How do you debug a Bash script in real time?**
**Answer:**
1. Run script with debug flags: `bash -x script.sh` (prints each command before execution).
2. Enable debugging for a specific code block inside the script:
   ```bash
   set -x  # Start tracing
   problematic_command
   set +x  # Stop tracing
   ```

---

### **39. What is the Python `dataclass` and why is it preferred for infrastructure models?**
**Answer:**
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

### **40. What is `pydantic` and how is it used for configuration parsing in DevOps tools?**
**Answer:**
Pydantic enforces strict data validation, type conversion, and environment variable loading based on Python type hints:
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

## 🔴 **Advanced & Scenario-Based Level (Questions 41–60)**

### **41. Scenario: Write a complete Python CLI tool using `boto3` that scans an AWS account, finds all unencrypted S3 buckets, enables KMS default encryption, and outputs an audit report.**
**Answer:**
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
            # Check existing encryption
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

### **42. Scenario: Write a robust Bash script that connects to a Kubernetes cluster, detects any Pods in `CrashLoopBackOff`, captures their previous container crash logs, uploads them to an S3 incident bucket, and sends a Slack alert via webhook.**
**Answer:**
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

# Get crashing pods
CRASHING_PODS=$(kubectl get pods -n "${NAMESPACE}" -o json | jq -r \
  '.items[] | select(.status.containerStatuses[]?.state.waiting.reason=="CrashLoopBackOff") | .metadata.name' | sort -u)

if [[ -z "${CRASHING_PODS}" ]]; then
    echo "[INFO] No crashing pods found in namespace ${NAMESPACE}."
    exit 0
fi

for POD in ${CRASHING_PODS}; do
    LOG_FILE="${INCIDENT_DIR}/${POD}_previous.log"
    echo "[INFO] Capturing logs for ${POD}..."
    kubectl logs "${POD}" -n "${NAMESPACE}" --previous > "${LOG_FILE}" 2>&1 || true
    
    # Upload to S3
    aws s3 cp "${LOG_FILE}" "${S3_BUCKET}/${NAMESPACE}/${POD}.log"
    
    # Send Slack notification
    PAYLOAD=$(jq -n \
      --arg pod "$POD" \
      --arg ns "$NAMESPACE" \
      --arg s3 "${S3_BUCKET}/${NAMESPACE}/${POD}.log" \
      '{text: "🚨 *Alert:* Pod `\($pod)` in `\($ns)` is CrashLoopBackOff. Logs: `\($s3)`"}')
      
    curl -s -X POST -H 'Content-type: application/json' --data "${PAYLOAD}" "${SLACK_WEBHOOK_URL}"
done
```

---

### **43. How do you implement a production-grade ThreadPool / ProcessPool in Python for concurrent API operations?**
**Answer:**
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

# Concurrent network I/O with 10 worker threads
with ThreadPoolExecutor(max_workers=10) as executor:
    futures = {executor.submit(check_endpoint_health, url): url for url in endpoints}
    for future in as_completed(futures):
        url, healthy = future.result()
        print(f"Endpoint: {url} | Healthy: {healthy}")
```

---

### **44. How do you safely execute remote commands on Linux nodes via AWS SSM Run Command using Python `boto3`?**
**Answer:**
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
    
    # Poll for execution result
    time.sleep(3)
    for instance_id in instance_ids:
        output = ssm.get_command_invocation(CommandId=command_id, InstanceId=instance_id)
        print(f"[{instance_id}] Status: {output['Status']}")
        print(output["StandardOutputContent"])

if __name__ == "__main__":
    run_ssm_command(["i-0123456789abcdef0"], "df -h && uptime")
```

---

### **45. How do you sanitize user inputs in Bash scripts to prevent Shell Injection vulnerabilities?**
**Answer:**
- **Never** pass untrusted variables directly to `eval` or execute inside subshells without sanitization.
- Validate inputs against strict whitelists using regex:
  ```bash
  USER_INPUT="$1"
  if [[ ! "${USER_INPUT}" =~ ^[a-zA-Z0-9_-]+$ ]]; then
      echo "[ERROR] Invalid input character detected." >&2
      exit 1
  fi
  ```

---

### **46. How do you build an interactive CLI tool in Python using `Click`?**
**Answer:**
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

### **47. What is Python `asyncio` and how does the Event Loop handle thousands of concurrent network requests?**
**Answer:**
`asyncio` uses single-threaded cooperative multitasking built on an **Event Loop**.
- When an I/O operation occurs (`await client.get(...)`), control is yielded back to the event loop.
- The event loop executes other ready coroutines while waiting for the OS socket to signal data availability.
- Handles thousands of concurrent network connections using minimal RAM without the overhead of heavy OS threads.

---

### **48. How do you parse and filter Linux `/var/log/syslog` using Python regex?**
**Answer:**
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
                print(f"OOM Triggered at {data['timestamp']} on {data['host']}: {data['message']}")
```

---

### **49. How do you profile Python code performance and memory consumption in production scripts?**
**Answer:**
- **CPU Profiling:** Use `cProfile`:
  ```bash
  python -m cProfile -s cumtime my_script.py
  ```
- **Memory Profiling:** Use `memory_profiler` or `tracemalloc`:
  ```python
  import tracemalloc
  tracemalloc.start()
  # Execute workload
  current, peak = tracemalloc.get_traced_memory()
  print(f"Peak memory: {peak / 10**6:.2f} MB")
  tracemalloc.stop()
  ```

---

### **50. Scenario: Design an automated script that drains and deletes older Kubernetes nodes across an AWS ASG during a rolling AMI update.**
**Answer:**
```python
import boto3
from kubernetes import client, config
import time

def rolling_node_update(asg_name: str):
    config.load_kube_config()
    core_v1 = client.CoreV1Api()
    asg = boto3.client("autoscaling", region_name="us-east-1")
    ec2 = boto3.client("ec2", region_name="us-east-1")

    # Get instances in ASG
    res = asg.describe_auto_scaling_groups(AutoScalingGroupNames=[asg_name])
    instances = res["AutoScalingGroups"][0]["Instances"]

    for inst in instances:
        instance_id = inst["InstanceId"]
        # Get private DNS name (K8s node name)
        ec2_res = ec2.describe_instances(InstanceIds=[instance_id])
        node_name = ec2_res["Reservations"][0]["Instances"][0]["PrivateDnsName"]

        print(f"[CORDON] Cordoning node {node_name}...")
        core_v1.patch_node(node_name, {"spec": {"unschedulable": True}})

        print(f"[DRAIN] Draining pods on node {node_name}...")
        # Evict pods respecting PDBs (using Eviction API)

        print(f"[TERMINATE] Terminating instance {instance_id}...")
        asg.terminate_instance_in_auto_scaling_group(
            InstanceId=instance_id,
            ShouldDecrementDesiredCapacity=False
        )
        time.sleep(60) # Wait for replacement node to join cluster
```
