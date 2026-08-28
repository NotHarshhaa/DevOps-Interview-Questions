# **Automation & Scripting - DevOps Interview Questions (200 Questions)**

Welcome to the **Automation & Scripting** master collection containing **200 comprehensive interview questions and detailed answers** covering Production Bash (`set -euo pipefail`, `trap`, `jq`, `yq`, `flock`), Python for DevOps (`boto3`, Kubernetes official SDK, Prometheus scrapers), Go for DevOps, CLI tool development, and automated operational runbooks.

---

## 🟢 **Part 1: Production Bash Scripting Fundamentals (Questions 1–60)**

### **1. Why is `set -euo pipefail` considered the gold standard in production Bash scripts?**
**Answer:**
By default, Bash ignores errors, treats undefined variables as empty strings, and only evaluates the exit status of the last command in a pipeline.
- **`-e` (`errexit`):** Exits immediately if any command returns a non-zero exit status.
- **`-u` (`nounset`):** Exits immediately if an undefined variable is referenced.
- **`-o pipefail`:** Prevents masked pipeline errors by ensuring a pipeline (e.g., `cmd1 | cmd2`) fails if **any** command in the chain returns non-zero.

### **2. How do you handle cleanup and signal trapping in Bash with `trap`?**
**Answer:**
The `trap` command registers cleanup functions executed when the script exits or receives interruption signals (`SIGINT`, `SIGTERM`, `EXIT`):
```bash
#!/usr/bin/env bash
set -euo pipefail

TEMP_DIR=$(mktemp -d /tmp/deploy_XXXXXX)

cleanup() {
    local exit_code=$?
    echo "[INFO] Cleaning up ${TEMP_DIR} (Exit code: ${exit_code})..."
    rm -rf "${TEMP_DIR}"
}
trap cleanup EXIT INT TERM
```

### **3. Compare single quotes `' '` vs double quotes `" "` in Bash.**
**Answer:**
- **Single Quotes (`' '`):** Preserves the literal value of every character inside. Variable expansion (`$VAR`) and command substitution (`$(cmd)`) are disabled.
- **Double Quotes (`" "`):** Enables variable expansion (`$VAR`), command substitution (`$(cmd)`), and arithmetic evaluation while preserving whitespace and preventing word splitting.

### **4. How do you parse and filter JSON in Bash using `jq`?**
**Answer:**
```bash
# 1. Extract pod names in 'Running' state from Kubernetes output
kubectl get pods -n production -o json | jq -r \
  '.items[] | select(.status.phase=="Running") | .metadata.name'

# 2. Extract nested database host
echo '{"db": {"host": "10.0.0.1"}}' | jq -r '.db.host'

# 3. Construct dynamic JSON payload
jq -n --arg env "prod" --arg ver "1.2.0" '{environment: $env, version: $ver}'
```

### **5. How do you modify YAML files safely in CLI using `yq`?**
**Answer:**
```bash
# 1. Update replica count in-place
yq eval '.spec.replicas = 5' -i deployment.yaml

# 2. Extract container image name
IMAGE=$(yq eval '.spec.template.spec.containers[0].image' deployment.yaml)
```

### **6. Compare `[[ ... ]]` vs `[ ... ]` in Bash.**
**Answer:**
- **`[ ... ]` (POSIX Test):** Older standard; prone to word splitting on empty variables and requires escaping logical operators (`-a`, `-o`).
- **`[[ ... ]]` (Extended Test):** Supports regex pattern matching (`=~`), logical operators (`&&`, `||`) without escaping, and does not word-split unset variables.

### **7. Explain common Linux / Bash exit codes.**
**Answer:**
- `0`: Success.
- `1`: General error.
- `2`: Misuse of shell builtins (syntax error).
- `126`: Command invoked cannot execute (permission denied).
- `127`: Command not found.
- `130`: Script terminated by `Ctrl+C` (`SIGINT` = $128 + 2$).
- `137`: Process killed forcibly (`SIGKILL` = $128 + 9$, typical for OOMKilled).
- `143`: Process terminated gracefully (`SIGTERM` = $128 + 15$).

### **8. How do you run commands in parallel in Bash using `xargs`?**
**Answer:**
```bash
cat urls.txt | xargs -n 1 -P 4 curl -O
```
- `-n 1`: Pass one argument per command invocation.
- `-P 4`: Run up to 4 parallel processes concurrently.

### **9. Why is Python `venv` mandatory for DevOps automation scripts?**
**Answer:** Creates isolated environments with independent package dependencies, preventing version conflicts between tools and avoiding polluting system Python packages (`/usr/lib/python`), which would break OS package managers (`apt`/`dnf`).

### **10. What is `boto3` and what is its credential resolution hierarchy?**
**Answer:** The official AWS SDK for Python.
**Authentication Hierarchy:**
1. Explicit credentials passed to `boto3.client(..., aws_access_key_id=...)`.
2. Environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`).
3. Shared credentials file (`~/.aws/credentials`).
4. AWS IAM Roles for Service Accounts (IRSA) / ECS Task Roles / EC2 Instance Metadata (`IMDSv2`).

### **11. Compare `boto3.client` vs `boto3.resource`.**
**Answer:**
- **`boto3.client`:** Low-level, 1-to-1 mapping to AWS REST APIs returning raw dictionaries; faster and supports 100% of AWS services.
- **`boto3.resource`:** High-level object-oriented abstraction (`s3.Bucket('b').objects.all()`); cleaner syntax, but does not support all new services.

### **12. How do you parse CLI arguments in Python using `argparse`?**
**Answer:**
```python
import argparse
parser = argparse.ArgumentParser(description="Audit AWS snapshots")
parser.add_argument("--retention-days", type=int, default=30)
parser.add_argument("--dry-run", action="store_true")
args = parser.parse_args()
print(f"Retention: {args.retention_days}, Dry run: {args.dry_run}")
```

### **13. Compare `os.system()`, `subprocess.run()`, and `subprocess.Popen()` in Python.**
**Answer:**
- **`os.system()`:** Legacy; executes in subshell, vulnerable to shell injection.
- **`subprocess.run()`:** Synchronous execution; captures stdout/stderr and return codes.
- **`subprocess.Popen()`:** Asynchronous, non-blocking process execution for streaming real-time output.

### **14. How do you safely read and parse environment variables in Python?**
**Answer:**
```python
import os
LOG_LEVEL = os.getenv("LOG_LEVEL", "INFO") # Fallback default
DATABASE_URL = os.environ["DATABASE_URL"]    # Mandatory (raises KeyError)
```

### **15. How do you parse and write YAML in Python safely using `PyYAML`?**
**Answer:**
```python
import yaml
with open("config.yaml", "r") as f:
    config = yaml.safe_load(f) # Always safe_load to prevent code execution!
config["replicas"] = 5
with open("config.yaml", "w") as f:
    yaml.dump(config, f, default_flow_style=False)
```

### **16. How do you implement retry logic with exponential backoff in Python?**
**Answer:**
```python
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
import requests

@retry(stop=stop_after_attempt(5), wait=wait_exponential(multiplier=1, min=2, max=10))
def fetch_api_data(url: str):
    res = requests.get(url, timeout=5)
    res.raise_for_status()
    return res.json()
```

### **17. How do you check if a port is open on a remote server using Bash without external tools?**
**Answer:**
```bash
if timeout 2 bash -c "</dev/tcp/10.0.0.1/5432" &>/dev/null; then
    echo "Port 5432 OPEN"
else
    echo "Port 5432 CLOSED"
fi
```

### **18. How do you find and delete files older than $N$ days in Linux?**
**Answer:**
```bash
find /var/log/app/ -type f -name "*.log" -mtime +14 -delete
```

### **19. Compare `awk` vs `sed` vs `grep`.**
**Answer:**
- **`grep`:** Pattern matching and line filtering using regex.
- **`sed`:** Stream Editor for text transformation and substitution (`sed -i 's/old/new/g' file`).
- **`awk`:** Text-processing language operating on structured column data (`awk '{print $1, $9}' access.log`).

### **20. What is a Shebang (`#!/usr/bin/env bash`)?**
**Answer:** Instructs the Linux kernel which interpreter to invoke. `#!/usr/bin/env bash` is preferred because it dynamically resolves `bash` from the user's `$PATH`.

### **21. How do you implement robust file locking in Bash with `flock`?**
**Answer:**
```bash
#!/usr/bin/env bash
set -euo pipefail

LOCK_FILE="/var/lock/backup_job.lock"
exec 200>"${LOCK_FILE}"

if ! flock -n 200; then
    echo "[WARN] Another instance is running. Exiting."
    exit 1
fi
echo "[INFO] Running backup..."
```

### **22. How do you read a large file line-by-line in Bash without memory exhaustion?**
**Answer:**
```bash
while IFS= read -r line || [[ -n "$line" ]]; do
    echo "Processing line: ${line}"
done < "large_access_log.txt"
```

### **23. How do you pass multiline strings and heredocs safely in Bash without variable expansion?**
**Answer:**
Quoting the delimiter (`'EOF'`) disables variable expansion:
```bash
cat <<'EOF' > /tmp/template.sh
echo "Generated at $(date) for user $USER"
EOF
```

### **24. How do you execute remote commands concurrently across 50 servers using Python `asyncio` and `asyncssh`?**
**Answer:**
```python
import asyncio
import asyncssh

async def run_cmd(host: str, cmd: str):
    async with asyncssh.connect(host, username="ubuntu", known_hosts=None) as conn:
        res = await conn.run(cmd)
        print(f"[{host}] {res.stdout.strip()}")

async def main(hosts):
    await asyncio.gather(*(run_cmd(h, "uptime") for h in hosts))

asyncio.run(main([f"10.0.0.{i}" for i in range(1, 50)]))
```

### **25. How do you interact with HashiCorp Vault in Python using `hvac`?**
**Answer:**
```python
import hvac, os
client = hvac.Client(url="https://vault.company.com:8200", token=os.environ["VAULT_TOKEN"])
res = client.secrets.kv.v2.read_secret_version(mount_point="secret", path="prod/db")
password = res["data"]["data"]["password"]
```

### **26. What are Python Generators (`yield`) and why are they crucial for gigabyte log files?**
**Answer:** Generators evaluate items on-demand (lazy evaluation) rather than allocating lists in memory:
```python
def stream_logs(filepath):
    with open(filepath, "r") as f:
        for line in f:
            if "ERROR 500" in line:
                yield line.strip()
```

### **27. How do you write a Bash function that accepts named parameter flags?**
**Answer:**
```bash
deploy() {
    local env="" ver=""
    while [[ $# -gt 0 ]]; do
        case "$1" in
            -e|--env) env="$2"; shift 2 ;;
            -v|--ver) ver="$2"; shift 2 ;;
            *) echo "Unknown: $1"; return 1 ;;
        esac
    done
    echo "Deploying ${ver} to ${env}"
}
deploy --env production --ver v2.0.1
```

### **28. How do you convert Bash arrays into JSON using `jq`?**
**Answer:**
```bash
REGIONS=("us-east-1" "us-west-2" "eu-west-1")
printf '%s\n' "${REGIONS[@]}" | jq -R . | jq -s .
# Output: ["us-east-1", "us-west-2", "eu-west-1"]
```

### **29. How do you calculate file SHA256 checksums in Python?**
**Answer:**
```python
import hashlib
def get_sha256(filepath):
    sha = hashlib.sha256()
    with open(filepath, "rb") as f:
        while chunk := f.read(8192):
            sha.update(chunk)
    return sha.hexdigest()
```

### **30. How do you debug Bash scripts in real time?**
**Answer:** Run `bash -x script.sh` or wrap specific blocks in `set -x` (enable trace) and `set +x` (disable trace).

### **31. What is Python `dataclass` and why is it used for infrastructure models?**
**Answer:** Automatically generates `__init__`, `__repr__`, and `__eq__` methods:
```python
from dataclasses import dataclass
@dataclass(frozen=True)
class Instance:
    instance_id: str
    ip: str
    cores: int
```

### **32. What is `pydantic` for configuration validation?**
**Answer:** Enforces strict data validation and type coercion from environment variables:
```python
from pydantic_settings import BaseSettings
class Settings(BaseSettings):
    db_host: str
    db_port: int = 5432
```

### **33. How do you implement a Token Bucket Rate Limiter in Python?**
**Answer:**
```python
import time
class RateLimiter:
    def __init__(self, rate: int):
        self.interval = 1.0 / rate
        self.last = 0.0
    def wait(self):
        diff = time.time() - self.last
        if diff < self.interval:
            time.sleep(self.interval - diff)
        self.last = time.time()
```

### **34. How do you sanitize user inputs in Bash scripts to prevent Shell Injection?**
**Answer:** Whitelist inputs using regex and avoid `eval`:
```bash
if [[ ! "${INPUT}" =~ ^[a-zA-Z0-9_-]+$ ]]; then
    echo "[ERROR] Invalid input." >&2; exit 1
fi
```

### **35. How do you build a CLI tool in Python using `Click`?**
**Answer:**
```python
import click
@click.command()
@click.option("--env", required=True, type=click.Choice(["dev", "prod"]))
def deploy(env):
    click.echo(f"Deploying to {env}...")
if __name__ == "__main__":
    deploy()
```

### **36. What is Python `asyncio` and the Event Loop?**
**Answer:** A single-threaded concurrency framework that manages asynchronous tasks. When an I/O operation starts (`await`), the coroutine yields control back to the event loop to execute other tasks.

### **37. How do you profile Python memory consumption with `tracemalloc`?**
**Answer:**
```python
import tracemalloc
tracemalloc.start()
# workload
cur, peak = tracemalloc.get_traced_memory()
print(f"Peak: {peak / 10**6:.2f} MB")
tracemalloc.stop()
```

### **38. What is `shutil` in Python?**
**Answer:** High-level file operations module (`shutil.copytree()`, `shutil.rmtree()`, `shutil.disk_usage()`).

### **39. What is `pathlib` in Python?**
**Answer:** Object-oriented filesystem path manipulation library replacing legacy `os.path`.

### **40. What is `logging` module best practice in Python?**
**Answer:** Configure handlers, formatters, and log levels centrally; avoid `print()` statements in production code.

### **41. What is Bash Parameter Expansion Default Value (`${VAR:-default}`)?**
**Answer:** Uses `default` if `VAR` is unset or empty, without modifying `VAR`.

### **42. What is Bash Parameter Expansion Assign Default (`${VAR:=default}`)?**
**Answer:** Uses `default` if `VAR` is unset or empty, and assigns `default` to `VAR`.

### **43. What is Bash Parameter Expansion Error Check (`${VAR:?error_msg}`)?**
**Answer:** Exits script with `error_msg` if `VAR` is unset or empty.

### **44. What is Bash String Length (`${#VAR}`)?**
**Answer:** Returns the character length of the string stored in `VAR`.

### **45. What is Bash String Substring (`${VAR:offset:length}`)?**
**Answer:** Extracts a substring from `offset` of designated `length`.

### **46. What is Bash String Replacement (`${VAR//pattern/replacement}`)?**
**Answer:** Replaces all occurrences of `pattern` with `replacement` within `VAR`.

### **47. What is Bash Subshell (`( cd /tmp && ls )`)?**
**Answer:** Commands inside parentheses execute in an isolated child subshell; working directory changes do not affect the parent shell.

### **48. What is Command Grouping (`{ cd /tmp; ls; }`) in Bash?**
**Answer:** Commands inside braces execute within the current shell context without spawning a subshell.

### **49. What is Process Substitution (`diff <(cmd1) <(cmd2)`)?**
**Answer:** Connects the output of commands to temporary file descriptors (`/dev/fd/63`), passing command outputs as files to programs that expect file arguments.

### **50. What is Standard I/O Redirection (`2>&1`)?**
**Answer:** Redirects Standard Error (file descriptor 2) to Standard Output (file descriptor 1).

### **51. What is `/dev/null` in Linux?**
**Answer:** A pseudo-device file that discards all data written to it (black hole) and returns EOF when read.

### **52. What is Python `requests.Session`?**
**Answer:** Reuses underlying TCP connections (HTTP Keep-Alive) across multiple requests, drastically improving REST API throughput.

### **53. What is Python `concurrent.futures.ThreadPoolExecutor`?**
**Answer:** High-level interface for executing I/O-bound tasks concurrently across a pool of threads.

### **54. What is Python `multiprocessing` vs `threading`?**
**Answer:** `threading` is constrained by the Global Interpreter Lock (GIL) and is optimal for I/O tasks; `multiprocessing` spawns independent OS processes and is optimal for CPU-bound tasks.

### **55. What is Python `functools.lru_cache`?**
**Answer:** Decorator that caches the return values of expensive functions in memory based on arguments.

### **56. What is Python `contextlib.contextmanager`?**
**Answer:** Decorator allowing the creation of custom `with` statement context managers using generators.

### **57. What is `cron` syntax (`* * * * *`)?**
**Answer:** 5 fields: Minute (0-59), Hour (0-23), Day of Month (1-31), Month (1-12), Day of Week (0-6, 0=Sun).

### **58. What is `systemd-run` for Ephemeral Timers?**
**Answer:** Spawns ad-hoc systemd transient timer units from CLI without creating persistent service files.

### **59. What is `logrotate` Configuration?**
**Answer:** Linux daemon that rotates, compresses, and purges system logs based on file size or time intervals.

### **60. What is `rsync` with Archive Mode (`-avz`)?**
**Answer:** Synchronizes files remotely preserving permissions, timestamps, symbolic links (`-a`), with verbose output (`-v`) and compression (`-z`).

---

## 🟡 **Part 2: Python for Cloud & Kubernetes (Questions 61–130)**

### **61. Write a Python script using `boto3` to audit and delete unattached EBS volumes older than 30 days.**
**Answer:**
```python
import boto3
from datetime import datetime, timezone, timedelta

def cleanup_unattached_ebs(days=30, dry_run=True):
    ec2 = boto3.client("ec2", region_name="us-east-1")
    cutoff = datetime.now(timezone.utc) - timedelta(days=days)
    res = ec2.describe_volumes(Filters=[{"Name": "status", "Values": ["available"]}])
    for vol in res.get("Volumes", []):
        if vol["CreateTime"] < cutoff:
            print(f"[ACTION] Volume {vol['VolumeId']} ({vol['Size']}GB) eligible.")
            if not dry_run:
                ec2.delete_volume(VolumeId=vol["VolumeId"])

if __name__ == "__main__":
    cleanup_unattached_ebs(30, dry_run=False)
```

### **62. Write a Python script using the official `kubernetes` client to trigger a rolling restart of a deployment.**
**Answer:**
```python
from kubernetes import client, config
from datetime import datetime, timezone

def restart_deployment(ns: str, name: str):
    try: config.load_incluster_config()
    except config.ConfigException: config.load_kube_config()
    apps = client.AppsV1Api()
    now = datetime.now(timezone.utc).isoformat()
    body = {"spec": {"template": {"metadata": {"annotations": {"kubectl.kubernetes.io/restartedAt": now}}}}}
    apps.patch_namespaced_deployment(name=name, namespace=ns, body=body)

if __name__ == "__main__":
    restart_deployment("production", "payment-service")
```

### **63. Write a Python script using `boto3` to scan all S3 buckets and enable KMS encryption.**
**Answer:**
```python
import boto3
from botocore.exceptions import ClientError

def enforce_s3_encryption():
    s3 = boto3.client("s3")
    for bucket in s3.list_buckets().get("Buckets", []):
        name = bucket["Name"]
        try:
            s3.get_bucket_encryption(Bucket=name)
        except ClientError as e:
            if e.response["Error"]["Code"] == "ServerSideEncryptionConfigurationNotFoundError":
                print(f"Enabling AES256 on {name}")
                s3.put_bucket_encryption(
                    Bucket=name,
                    ServerSideEncryptionConfiguration={
                        "Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]
                    }
                )

if __name__ == "__main__":
    enforce_s3_encryption()
```

### **64. How do you scrape and parse Prometheus metrics endpoints using Python?**
**Answer:**
```python
import requests
from prometheus_client.parser import text_string_to_metric_families

def parse_metrics(url: str, metric_name: str):
    res = requests.get(url, timeout=5)
    for family in text_string_to_metric_families(res.text):
        if family.name == metric_name:
            for sample in family.samples:
                print(f"Name: {sample.name}, Labels: {sample.labels}, Value: {sample.value}")

if __name__ == "__main__":
    parse_metrics("http://localhost:9090/metrics", "http_requests_total")
```

### **65. How do you drain and replace worker nodes in an AWS ASG using Python?**
**Answer:**
```python
import boto3, time
from kubernetes import client, config

def rolling_asg_update(asg_name: str):
    config.load_kube_config()
    core = client.CoreV1Api()
    asg = boto3.client("autoscaling")
    ec2 = boto3.client("ec2")
    instances = asg.describe_auto_scaling_groups(AutoScalingGroupNames=[asg_name])["AutoScalingGroups"][0]["Instances"]
    for inst in instances:
        iid = inst["InstanceId"]
        node_name = ec2.describe_instances(InstanceIds=[iid])["Reservations"][0]["Instances"][0]["PrivateDnsName"]
        core.patch_node(node_name, {"spec": {"unschedulable": True}})
        asg.terminate_instance_in_auto_scaling_group(InstanceId=iid, ShouldDecrementDesiredCapacity=False)
        time.sleep(60)
```

---

## 🔴 **Part 3: Go for DevOps, Advanced Automation & Incident Runbooks (Questions 66–200)**

### **66. Why is Go (Golang) the dominant language for Cloud-Native and DevOps tooling?**
**Answer:** Compiles to a single static binary with zero external runtime dependencies, ultra-fast boot time, low memory footprint, first-class concurrency (Goroutines and Channels), and strong type safety (Docker, Kubernetes, Terraform, Prometheus are all written in Go).

### **67. What are Goroutines vs OS Threads in Go?**
**Answer:** Goroutines are lightweight userspace threads managed by the Go runtime scheduler (m:n scheduler). A Goroutine starts with only **2KB of stack memory** (compared to 1–8MB for OS threads) and switches context in nanoseconds.

### **68. What are Go Channels and how do they coordinate concurrent tasks?**
**Answer:** Typed conduits for sending and receiving data between Goroutines safely without explicit mutex locks ("Do not communicate by sharing memory; instead, share memory by communicating").

### **69. Write a Go program using the official `client-go` library to list all pods in a namespace.**
**Answer:**
```go
package main

import (
    "context"
    "fmt"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/tools/clientcmd"
)

func main() {
    config, _ := clientcmd.BuildConfigFromFlags("", clientcmd.RecommendedHomeFile)
    clientset, _ := kubernetes.NewForConfig(config)
    pods, _ := clientset.CoreV1().Pods("production").List(context.TODO(), metav1.ListOptions{})
    for _, pod := range pods.Items {
        fmt.Printf("Pod: %s, Status: %s\n", pod.Name, pod.Status.Phase)
    }
}
```

### **70. Write a Go HTTP health check probe with timeout.**
**Answer:**
```go
package main

import (
    "net/http"
    "time"
)

func checkHealth(url string) bool {
    client := http.Client{Timeout: 2 * time.Second}
    resp, err := client.Get(url)
    if err != nil || resp.StatusCode != http.StatusOK {
        return false
    }
    return true
}
```

### **71. What is `sync.WaitGroup` in Go?**
**Answer:** Coordinates waiting for a collection of concurrent Goroutines to finish execution using `Add()`, `Done()`, and `Wait()`.

### **72. What is `context.Context` in Go and why is it mandatory for cloud operations?**
**Answer:** Carries deadlines, cancellation signals, and request-scoped values across API boundaries, ensuring downstream network calls terminate when parent requests time out.

### **73. What is Go `defer` statement?**
**Answer:** Schedules a function call to execute immediately before the surrounding function returns, heavily used for resource cleanup (`defer file.Close()`, `defer resp.Body.Close()`).

### **74. What is Go `select` statement?**
**Answer:** Allows a Goroutine to wait on multiple channel communications simultaneously, executing the first channel that is ready.

### **75. What is Go Mutex (`sync.Mutex` vs `sync.RWMutex`)?**
**Answer:**
- **`sync.Mutex`:** Exclusive mutual exclusion lock.
- **`sync.RWMutex`:** Allows multiple concurrent readers or a single exclusive writer.

### **76. What is Go Custom Kubernetes Controller architecture?**
**Answer:** Uses `InformerFactory`, `Lister`, and `Workqueue` to process reconcile events from the Kubernetes API Server.

### **77. What is Go Cobra Framework?**
**Answer:** The standard library for building modern CLI tools with subcommands, flags, and auto-generated help docs (powers `kubectl` and `hugo`).

### **78. What is Go Viper Framework?**
**Answer:** A complete configuration solution for Go applications supporting JSON, YAML, environment variables, and remote key-value stores.

### **79. What is Go Cross-Compilation?**
**Answer:** Compiling binaries for different OS and architectures from a single machine using environment variables: `GOOS=linux GOARCH=arm64 go build -o app_arm64`.

### **80. What is Go `cgo` and why is `CGO_ENABLED=0` mandatory for container scratch images?**
**Answer:** `CGO_ENABLED=0` disables linking to host C libraries (`glibc`), producing a 100% statically linked binary that runs inside an empty Docker `FROM scratch` container.

### **81. What is Bash Array Slicing (`${ARR[@]:1:2}`)?**
**Answer:** Extracts a subset of array elements from offset index for designated length.

### **82. What is Bash Associative Array (`declare -A MAP`)?**
**Answer:** Key-value hash maps in Bash 4+:
```bash
declare -A SERVERS
SERVERS["web"]="10.0.0.1"
SERVERS["db"]="10.0.0.2"
```

### **83. What is Bash Indirect Variable Reference (`${!VAR}`)?**
**Answer:** Evaluates the value of the variable whose name is stored in `VAR`.

### **84. What is Bash Read with Timeout (`read -t 5`)?**
**Answer:** Waits up to 5 seconds for user input before returning non-zero.

### **85. What is Bash Math Evaluation (`$(( 5 * 2 ))`)?**
**Answer:** Native integer arithmetic evaluation inside double parentheses.

### **86. What is Python `boto3` Paginator?**
**Answer:** Automatically iterates across multi-page API responses (`paginator.paginate()`) to fetch thousands of cloud objects without manual `NextToken` handling.

### **87. What is Python `boto3` Waiter?**
**Answer:** Polls AWS service state until a designated condition is reached (`waiter.wait(InstanceIds=['...'])`), avoiding custom while-sleep loops.

### **88. What is Python `fabric` for SSH automation?**
**Answer:** High-level Python library executing shell commands remotely over SSH and uploading/downloading files.

### **89. What is Python `pexpect`?**
**Answer:** Automates interactive console applications (prompting for passwords or confirmations) by spawning child processes and matching output patterns.

### **90. What is Python `paramiko`?**
**Answer:** Low-level native Python implementation of the SSHv2 protocol for custom SFTP and SSH clients.

### **91. What is Bash Here-String (`<<< "$STRING"`)?**
**Answer:** Passes a string variable directly to a command's standard input without `echo | cmd`.

### **92. What is Bash Null Command (`:`)?**
**Answer:** A no-op builtin that returns exit code 0, used in infinite loops (`while :; do ... done`) or setting default variables.

### **93. What is Bash Output to Multiple Files (`tee`)?**
**Answer:** Reads standard input and writes it simultaneously to standard output and one or more files.

### **94. What is Bash Split String by Delimiter (`IFS=',' read -ra ARR <<< "$STR"`)?**
**Answer:** Splits a delimited string into an array using Internal Field Separator.

### **95. What is Python `uuid.uuid4()`?**
**Answer:** Generates cryptographically secure random UUIDs for distributed correlation IDs and request tracking.

### **96. What is Python `tempfile.NamedTemporaryFile`?**
**Answer:** Creates secure temporary files with automated deletion on context manager exit.

### **97. What is Python `secrets` module?**
**Answer:** Generates cryptographically secure random numbers and tokens for passwords and authentication keys.

### **98. What is Python `hashlib.pbkdf2_hmac`?**
**Answer:** Secure key derivation function applying HMAC with SHA256 over thousands of iterations for password hashing.

### **99. What is Python `urllib3.PoolManager`?**
**Answer:** Thread-safe connection pool manager used internally by `requests` for client connection reuse.

### **100. What is Python `jinja2.Template`?**
**Answer:** Renders configuration files dynamically with variable substitutions, conditionals, and loops.

### **101. What is Bash `mktemp`?**
**Answer:** Creates a secure temporary file or directory with unique randomized name in `/tmp`.

### **102. What is Bash `basename` and `dirname`?**
**Answer:**
- `basename /var/log/app.log` $\rightarrow$ `app.log`
- `dirname /var/log/app.log` $\rightarrow$ `/var/log`

### **103. What is Bash `realpath`?**
**Answer:** Resolves relative paths, symbolic links, and parent references (`..`) to canonical absolute paths.

### **104. What is Bash `getopts`?**
**Answer:** Builtin utility for parsing single-character command-line options and arguments in scripts.

### **105. What is Bash Subshell Exit Code (`$?`)?**
**Answer:** Holds the exit status of the most recently executed foreground command.

### **106. What is Python `dotenv` (`python-dotenv`)?**
**Answer:** Reads key-value pairs from a `.env` file and adds them to environment variables (`os.environ`).

### **107. What is Python `attrs` vs `pydantic`?**
**Answer:** `attrs` focuses on class creation and boilerplate reduction; `pydantic` focuses on runtime data parsing and strict schema validation.

### **108. What is Python `marshmallow`?**
**Answer:** Object serialization and deserialization library converting complex objects to and from native Python datatypes.

### **109. What is Python `celery`?**
**Answer:** Distributed task queue managing background jobs across Redis or RabbitMQ message brokers.

### **110. What is Python `fastapi` in DevOps Tooling?**
**Answer:** High-performance web framework for building microservices, webhook receivers, and internal DevOps APIs with automated OpenAPI docs.

### **111. What is Bash `shuf`?**
**Answer:** Generates random permutations or picks random lines from input files (used for random canary node selection).

### **112. What is Bash `comm`?**
**Answer:** Compares two sorted files line by line, outputting lines unique to file 1, unique to file 2, or common to both.

### **113. What is Bash `cut`?**
**Answer:** Extracts specific sections or columns from each line of a file using field delimiters (`cut -d: -f1 /etc/passwd`).

### **114. What is Bash `tr`?**
**Answer:** Translates, deletes, or squeezes characters from standard input (`tr '[:lower:]' '[:upper:]'`).

### **115. What is Bash `sort -u` vs `uniq`?**
**Answer:** `uniq` only removes adjacent duplicate lines; `sort -u` sorts the entire file and removes all duplicates globally.

### **116. What is Python `typer`?**
**Answer:** CLI library built on top of Click that uses Python type hints to generate command-line interfaces automatically.

### **117. What is Python `rich`?**
**Answer:** Terminal formatting library rendering rich colored text, tables, progress bars, and markdown directly in CLI tools.

### **118. What is Python `watchdog`?**
**Answer:** Filesystem event monitoring library that triggers callbacks when files are created, modified, or deleted on disk.

### **119. What is Python `schedule`?**
**Answer:** In-process Python job scheduling library for running recurring jobs with human-readable syntax.

### **120. What is Python `pytest` in DevOps Automation?**
**Answer:** Testing framework for writing unit and integration tests against infrastructure scripts, API responses, and CLI tools.

### **121. What is Bash `head -n` and `tail -n`?**
**Answer:** Outputs the first $N$ or last $N$ lines of a file (`tail -f` streams live appended lines).

### **122. What is Bash `wc -l`?**
**Answer:** Counts the total number of lines in standard input or files.

### **123. What is Bash `grep -E` (Extended Regex)?**
**Answer:** Enables extended regular expressions (`+`, `?`, `|`, `()`) without backslash escaping.

### **124. What is Bash `grep -v`?**
**Answer:** Inverts match to select non-matching lines.

### **125. What is Bash `grep -o`?**
**Answer:** Prints *only* the exact matched parts of a matching line.

### **126. What is Python `boto3` Client Error Handling (`botocore.exceptions.ClientError`)?**
**Answer:** Catching AWS API exceptions and inspecting `e.response['Error']['Code']` to handle specific errors (`NoSuchBucket`, `ThrottlingException`).

### **127. What is Python `kubernetes.watch.Watch`?**
**Answer:** Streams real-time event updates for Kubernetes resources from the API Server without polling.

### **128. What is Python `docker` SDK?**
**Answer:** Official Python library for controlling Docker daemons, building images, managing containers, and reading logs via API.

### **129. What is Python `gitpython`?**
**Answer:** Python library for interacting with Git repositories programmatically (cloning, checking out branches, creating commits).

### **130. What is Python `pyOpenSSL`?**
**Answer:** Python wrapper around OpenSSL for generating keys, creating CSRs, and inspecting X.509 certificate fields.

### **131. What is Bash `seq`?**
**Answer:** Generates a sequence of numbers (`seq 1 10`).

### **132. What is Bash `column -t`?**
**Answer:** Formats unaligned tabular text into clean, evenly spaced visual columns.

### **133. What is Bash `fold`?**
**Answer:** Wraps input lines to fit a specified column width.

### **134. What is Bash `paste`?**
**Answer:** Merges corresponding lines of multiple files side-by-side separated by tabs.

### **135. What is Bash `split`?**
**Answer:** Splits a large file into smaller chunks based on line count or byte size (`split -b 100M large.tar.gz`).

### **136. What is Python `pytest-mock`?**
**Answer:** Plugin providing fixture-based mocking of cloud SDKs and external APIs during unit test execution.

### **137. What is Python `responses`?**
**Answer:** Utility library for mocking Python `requests` calls during unit testing.

### **138. What is Python `moto`?**
**Answer:** Comprehensive library that mocks AWS services locally, allowing `boto3` scripts to be tested without AWS accounts.

### **139. What is Python `freezegun`?**
**Answer:** Freezes and mocks time in Python unit tests to test TTLs and expiration logic.

### **140. What is Python `hypothesis`?**
**Answer:** Property-based testing framework that generates diverse test cases to discover edge-case bugs in automation logic.

### **141. What is Bash `tac`?**
**Answer:** Concatenates and prints files in reverse order (last line first).

### **142. What is Bash `nl`?**
**Answer:** Numbers lines of files during output.

### **143. What is Bash `fold -s`?**
**Answer:** Wraps lines at space boundaries to prevent breaking words across lines.

### **144. What is Bash `rev`?**
**Answer:** Reverses lines character-by-character.

### **145. What is Bash `expand` and `unexpand`?**
**Answer:** Converts tabs to spaces and spaces to tabs in text files.

### **146. What is Python `structlog`?**
**Answer:** Structured logging library binding contextual key-value pairs across application call chains to emit structured JSON logs.

### **147. What is Python `loguru`?**
**Answer:** Zero-config Python logging library with automatic stack formatting, file rotation, and colorized output.

### **148. What is Python `coloredlogs`?**
**Answer:** Colorizes Python standard logging output for CLI terminal tools.

### **149. What is Python `psutil`?**
**Answer:** Cross-platform library for retrieving hardware utilization (CPU, memory, disks, network) and running process information.

### **150. What is Python `cryptography` library?**
**Answer:** Standard cryptographic primitives library providing AES-GCM encryption, RSA/ECC key generation, and X.509 parsing.

### **151. What is Bash `readlink -f`?**
**Answer:** Canonicalizes paths by following every symlink recursively.

### **152. What is Bash `env -i`?**
**Answer:** Executes a command in a clean, empty environment with zero inherited environment variables.

### **153. What is Bash `exec`?**
**Answer:** Replaces the current shell process with the specified command without creating a new process ID.

### **154. What is Bash `time` builtin?**
**Answer:** Measures real, user, and system CPU time consumed by command execution.

### **155. What is Bash `ulimit`?**
**Answer:** Controls process resource limits (max open files, max stack size, max user processes).

### **156. What is Python `scapy`?**
**Answer:** Powerful interactive packet manipulation library for generating, sniffing, and forging network packets.

### **157. What is Python `netaddr`?**
**Answer:** Library for parsing, manipulating, and calculating IP addresses, subnets, CIDRs, and MAC addresses.

### **158. What is Python `socket` module?**
**Answer:** Low-level network interface accessing BSD socket APIs directly.

### **159. What is Python `ssl` module?**
**Answer:** TLS/SSL encryption wrapper for socket objects.

### **160. What is Python `ipaddress` module?**
**Answer:** Standard library for inspecting and calculating IPv4 and IPv6 network properties and subnet overlap.

### **161. What is Bash `alias` vs Function?**
**Answer:** Aliases are simple string replacements; Functions accept positional arguments, local variables, and logic branches.

### **162. What is Bash `history` command?**
**Answer:** Displays the chronological list of previously executed shell commands with line numbers.

### **163. What is Bash `bind`?**
**Answer:** Binds Readline keyboard shortcuts to shell editing functions.

### **164. What is Bash `type` command?**
**Answer:** Identifies whether a command is a shell builtin, alias, function, or external binary on disk.

### **165. What is Bash `which` vs `command -v`?**
**Answer:** `command -v` is POSIX-compliant and faster because it is a shell builtin; `which` is an external binary that may behave inconsistently across OSs.

### **166. What is Python `multiprocessing.Pool`?**
**Answer:** Distributes CPU-intensive tasks across a fixed pool of worker processes.

### **167. What is Python `asyncio.Semaphore`?**
**Answer:** Limits the maximum number of concurrent coroutines accessing a shared resource simultaneously.

### **168. What is Python `asyncio.Queue`?**
**Answer:** Thread-safe FIFO queue for coordinating producer and consumer coroutines in asynchronous pipelines.

### **169. What is Python `aiohttp`?**
**Answer:** Asynchronous HTTP client and server framework for high-throughput API integrations.

### **170. What is Python `httpx`?**
**Answer:** Modern HTTP client supporting both synchronous and asynchronous APIs with HTTP/2 support.

### **171. What is Bash `logger` command?**
**Answer:** Sends custom log messages directly to the Linux system logger (`/var/log/syslog` / systemd journal).

### **172. What is Bash `watch` command?**
**Answer:** Executes a command repeatedly at regular intervals (default: 2s), displaying live output in full screen.

### **173. What is Bash `timeout` command?**
**Answer:** Runs a command and terminates it with `SIGTERM`/`SIGKILL` if it exceeds a specified duration.

### **174. What is Bash `yes` command?**
**Answer:** Outputs an endless stream of `y` (or designated string) to automate interactive CLI prompts.

### **175. What is Bash `nohup` command?**
**Answer:** Runs a command immune to hangups (`SIGHUP`), allowing processes to continue running in background after terminal disconnection.

### **176. What is Python `boto3` Resource Cost Tracking?**
**Answer:** Tagging all dynamically created AWS resources (`Environment`, `CreatedBy`, `Owner`) via Python scripts to enable FinOps attribution.

### **177. What is Python AWS Lambda Handler Structure?**
**Answer:** `def lambda_handler(event, context):` receives trigger payload (`event`) and runtime metadata (`context`).

### **178. What is Python AWS Step Functions Local Execution?**
**Answer:** Testing serverless state machines locally using Docker containers and Python SDKs.

### **179. What is Python `sqlalchemy` in DevOps?**
**Answer:** Object Relational Mapper (ORM) and SQL toolkit for interacting with relational databases in automation pipelines.

### **180. What is Python `alembic`?**
**Answer:** Lightweight database migration tool for PostgreSQL/MySQL managed in Python code.

### **181. What is Bash `chown` vs `chmod`?**
**Answer:** `chown` changes user and group file ownership; `chmod` changes read, write, and execute permissions.

### **182. What is Bash `umask`?**
**Answer:** Sets default permission masks for newly created files and directories (e.g., `022` results in `755` directories and `644` files).

### **183. What is Bash Sticky Bit (`chmod +t`)?**
**Answer:** Applied to directories (like `/tmp`) so that only the file's owner or root can delete or rename files within the directory.

### **184. What is Bash SUID Bit (`chmod u+s`)?**
**Answer:** Executes binary with the permissions of the file owner rather than the running user (e.g., `/usr/bin/passwd`).

### **185. What is Bash SGID Bit (`chmod g+s`)?**
**Answer:** Ensures newly created files inside a directory inherit the group ownership of the directory rather than the primary group of the user.

### **186. What is Python `click.group`?**
**Answer:** Creates nested multi-level command hierarchies for complex CLI utilities (`cli deploy backend`).

### **187. What is Python `click.prompt`?**
**Answer:** Interactively prompts the user for inputs with optional masking for passwords (`hide_input=True`).

### **188. What is Python `click.progressbar`?**
**Answer:** Renders visual progress bars for long-running iterative tasks in terminal tools.

### **189. What is Python `pyinstrument`?**
**Answer:** Statistical Python code profiler that outputs hierarchical call trees to identify execution bottlenecks.

### **190. What is Python `line_profiler`?**
**Answer:** Line-by-line CPU profiling tool analyzing execution time spent on each individual statement inside functions.

### **191. What is Bash `crontab -e` vs `/etc/cron.d/`?**
**Answer:** `crontab -e` edits user-specific cron tables; `/etc/cron.d/` contains system-wide modular cron job files with explicit username fields.

### **192. What is Bash `at` command?**
**Answer:** Schedules a one-time command execution at a specific future timestamp (`at 2:00 AM tomorrow`).

### **193. What is Bash `systemctl list-timers`?**
**Answer:** Lists all active systemd timer units, their next trigger timestamp, and last execution time.

### **194. What is Bash `journalctl -u`?**
**Answer:** Filters and displays systemd logs for a specific service unit (`journalctl -u nginx.service -f`).

### **195. What is Bash `dmesg -T`?**
**Answer:** Prints human-readable timestamped Linux kernel ring buffer messages.

### **196. What is Python `pytest-cov`?**
**Answer:** Pytest plugin measuring code coverage percentage across test suites.

### **197. What is Python `black`?**
**Answer:** Uncompromising Python code formatter ensuring consistent codebase style across teams.

### **198. What is Python `ruff`?**
**Answer:** Extremely fast Python linter and code formatter written in Rust, replacing flake8, isort, and black.

### **199. What is Python `mypy`?**
**Answer:** Static type checker for Python analyzing type hints to prevent runtime `TypeError` bugs.

### **200. What is an Enterprise Automation Runbook Standard?**
**Answer:**
1. **Idempotency:** Re-running produces identical state without errors.
2. **Dry-Run Mode (`--dry-run`):** Previews actions before executing modifications.
3. **Structured Logging:** Emits machine-readable JSON logs.
4. **Signal Handling:** Cleans up temporary resources on `SIGINT`/`SIGTERM`.
5. **Auditing & Telemetry:** Records who ran the script, parameters used, and execution duration.
