# **Containers & Kubernetes - DevOps Interview Questions**

Welcome to the **Containers & Kubernetes** interview questions master guide. This module provides in-depth, exhaustive technical explanations, architecture diagrams, container runtime internals (Docker, containerd, CRI-O), modern Kubernetes (1.28–1.32+), Gateway API, Karpenter, Cilium & eBPF, Pod Security Standards, KEDA, and real-world production incident troubleshooting.

---

## 🟢 **Beginner Level (Questions 1–20)**

### **1. What is the fundamental architectural difference between a Virtual Machine (VM) and a Container? Compare them across isolation, performance, resource overhead, and startup latency.**

**Detailed Answer:**

```
               VIRTUAL MACHINES (Hardware-Level Virtualization)
 ┌──────────────────────────────────────────────────────────────────────────┐
 │  ┌──────────────────────┐  ┌──────────────────────┐  ┌─────────────────┐ │
 │  │ App A + Libraries    │  │ App B + Libraries    │  │ App C + Libs    │ │
 │  ├──────────────────────┤  ├──────────────────────┤  ├─────────────────┤ │
 │  │ Guest OS (Full Linux)│  │ Guest OS (Windows)   │  │ Guest OS (RHEL) │ │
 │  └──────────┬───────────┘  └──────────┬───────────┘  └────────┬────────┘ │
 │             └─────────────────────────┼───────────────────────┘          │
 │                                       ▼                                  │
 │                      HYPERVISOR (Type 1 or Type 2)                       │
 │ ──────────────────────────────────────────────────────────────────────── │
 │                      HOST PHYSICAL HARDWARE (CPU/RAM)                    │
 └──────────────────────────────────────────────────────────────────────────┘

──────────────────────────────────────────────────────────────────────────────

                  CONTAINERS (Operating System-Level Virtualization)
 ┌──────────────────────────────────────────────────────────────────────────┐
 │  ┌──────────────────────┐  ┌──────────────────────┐  ┌─────────────────┐ │
 │  │ App A + Dependencies │  │ App B + Dependencies │  │ App C + Deps    │ │
 │  ├──────────────────────┤  ├──────────────────────┤  ├─────────────────┤ │
 │  │ Isolated User Space  │  │ Isolated User Space  │  │ Isolated User   │ │
 │  └──────────┬───────────┘  └──────────┬───────────┘  └────────┬────────┘ │
 │             └─────────────────────────┼───────────────────────┘          │
 │                                       ▼                                  │
 │                     CONTAINER RUNTIME (containerd / CRI-O)               │
 │ ──────────────────────────────────────────────────────────────────────── │
 │               SHARED HOST LINUX KERNEL (Namespaces & cgroups)            │
 │ ──────────────────────────────────────────────────────────────────────── │
 │                      HOST PHYSICAL HARDWARE (CPU/RAM)                    │
 └──────────────────────────────────────────────────────────────────────────┘
```

#### **Comprehensive Comparison Matrix:**

| Dimension | Virtual Machine (VM) | Container |
| :--- | :--- | :--- |
| **Virtualization Level** | **Hardware Virtualization** via Hypervisor (KVM, VMware ESXi, Hyper-V). | **OS-Level Virtualization** sharing the host Linux kernel. |
| **Operating System** | Each VM runs its own independent **Guest OS** (kernel, device drivers, init system). | Containers package only user-space binaries and libraries; share the **Host OS Kernel**. |
| **Resource Overhead** | Heavy (requires reserving multiple GBs of RAM, vCPUs, and 20GB+ virtual disk per VM). | Ultra-lightweight (MBs of RAM; consumes only what the process actively uses). |
| **Startup Latency** | **Minutes** (Must boot full OS kernel, start systemd services, mount hardware). | **Milliseconds to Seconds** (Spawns as a standard Linux process). |
| **Isolation Boundary** | **Strong Hardware Isolation** enforced by CPU virtualization instructions (VT-x/AMD-V). | **Process / Namespace Isolation** enforced by Linux kernel primitives. |
| **Density & Portability** | Low density (tens of VMs per physical host). | High density (hundreds of containers per physical host); OCI-compliant portability. |

---

### **2. How do Linux Namespaces and Control Groups (cgroups v2) create a container under the hood? Explain all 7 core namespaces.**

**Detailed Answer:**
A container is not an actual physical entity in the Linux kernel. A container is simply a standard Linux process isolated by **Namespaces** (which restrict what a process can *see*) and bounded by **Control Groups (cgroups)** (which restrict how much resource a process can *use*).

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                             ANATOMY OF A LINUX CONTAINER                         │
│                                                                                  │
│   ┌──────────────────────────────────────────────────────────────────────────┐   │
│   │                         NAMESPACES (Isolation Boundary)                  │   │
│   │                                                                          │   │
│   │  • PID: Isolated process tree (container thinks it is PID 1)             │   │
│   │  • NET: Virtual network interfaces, private routing tables & port space  │   │
│   │  • MNT: Isolated root filesystem mount points (via pivot_root)           │   │
│   │  • IPC: Isolated System V IPC and POSIX message queues                   │   │
│   │  • UTS: Independent hostname and domain name                             │   │
│   │  • USER: UID/GID mapping (root in container = unprivileged on host)      │   │
│   │  • CGROUP: Isolated view of the cgroup hierarchy                         │   │
│   └─────────────────────────────────────┬────────────────────────────────────┘   │
│                                         │                                        │
│   ┌─────────────────────────────────────┴────────────────────────────────────┐   │
│   │                     CONTROL GROUPS - cgroups v2 (Resource Limits)        │   │
│   │                                                                          │   │
│   │  • CPU: CFS bandwidth quota (e.g., max 2.0 cores per 100ms period)       │   │
│   │  • MEMORY: Hard memory ceiling (OOM Killer triggers if breached)         │   │
│   │  • IO: Throttles read/write IOPS and bytes/sec on storage block devices   │   │
│   │  • PIDS: Limits max process/thread count (prevents fork bombs)           │   │
│   └──────────────────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────────────┘
```

#### **The 7 Core Linux Namespaces:**
1. **`pid` (Process ID):** Isolates the process ID space. The main container process becomes PID 1 inside the container, while appearing as a standard high-numbered PID (e.g., PID 49281) on the host OS.
2. **`net` (Network):** Provides dedicated virtual network interfaces (`veth` pairs), loopback devices, IP routing tables, and firewall filter rules.
3. **`mnt` (Mount):** Isolates filesystem mount points. Combined with `pivot_root`, it changes the container's root directory (`/`) to the container image rootfs.
4. **`ipc` (Inter-Process Communication):** Prevents processes in different containers from communicating via shared memory segments or POSIX message queues.
5. **`uts` (UNIX Timesharing System):** Allows setting custom hostnames for containers independently of the host machine.
6. **`user` (User ID):** Maps user and group IDs. Allows a process to run as `root` (UID 0) inside the container namespace while mapping to an unprivileged UID (e.g., UID 10001) on the host OS, protecting against host root compromise.
7. **`cgroup` (Control Group View):** Masks the host's full cgroup hierarchy, ensuring processes only see their own assigned cgroup subtrees.

---

### **3. What is a Docker Multi-Stage Build, why is it critical for enterprise security, and how does it shrink container image size from 1GB to 15MB?**

**Detailed Answer:**
In standard single-stage Dockerfiles, build tools, package managers, compilers (GCC, Go, Rust, Maven), header files, and intermediate build artifacts remain bundled in the final container image, creating bloated images ($> 1\text{GB}$) riddled with CVE vulnerabilities.

#### **Multi-Stage Build Architecture:**
Multi-stage builds allow using multiple `FROM` statements in a single Dockerfile. Heavy compilers and source code live exclusively in the build stage, and only the final compiled static binary is copied into an ultra-minimal production runtime base.

```dockerfile
# ==========================================
# STAGE 1: Builder Stage (Heavy tooling)
# ==========================================
FROM golang:1.24-alpine AS builder

WORKDIR /build

# 1. Cache dependencies first
COPY go.mod go.sum ./
RUN go mod download

# 2. Copy source code and compile static binary
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -ldflags="-w -s" -o payment-service .

# ==========================================
# STAGE 2: Minimal Production Runtime Stage
# ==========================================
FROM gcr.io/distroless/static-debian12:nonroot

WORKDIR /
# Copy ONLY the compiled binary from the builder stage
COPY --from=builder /build/payment-service /payment-service

# Run as non-root user (UID 65532)
USER nonroot:nonroot

EXPOSE 8080
ENTRYPOINT ["/payment-service"]
```

#### **Security and Performance Benefits:**
- **Image Size Reduction:** Shrinks image from ~1.2GB (full Go SDK + build tools) to **~15MB**.
- **Massive CVE Reduction:** Eliminates package managers (`apt`, `apk`), shells (`/bin/sh`, `/bin/bash`), and core utilities (`curl`, `wget`), leaving zero tools for an attacker to download malware or spawn reverse shells upon code exploitation.
- **Faster CI/CD Deployments:** Pushing and pulling 15MB images takes under 2 seconds across Kubernetes worker nodes.

---

### **4. What are Distroless and Scratch base images? Compare their security postures.**

**Detailed Answer:**
- **`scratch` (0 Bytes Base Image):**
  - An explicitly empty Docker base image containing literally zero files or directories.
  - Used for statically compiled binaries (Go, Rust, C++) that require zero dynamic shared libraries (`glibc`).
  - *Trade-off:* Does not contain SSL root certificates or timezone data (`tzdata`) unless explicitly copied into the image.
- **`distroless` (Maintained by Google):**
  - Contains strictly the bare minimum runtime dependencies for specific language runtimes (e.g., `distroless/java`, `distroless/nodejs`, `distroless/static`).
  - Contains CA root certificates, `/etc/passwd`, `/tmp`, and basic system libraries, but **completely omits package managers, shells, and standard OS utilities**.
- **Security Posture:** Prevents remote code execution (RCE) exploits from pivoting to interactive shell sessions or running shell scripts.

---

### **5. What is the Kubernetes Control Plane and what are its core components? Explain their internal communication flow.**

**Detailed Answer:**
The **Kubernetes Control Plane** maintains the desired state of the cluster, orchestrates automatic scheduling, handles API requests, and detects/responds to failure events.

```
                                  KUBERNETES CONTROL PLANE
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │                                                                                        │
 │  ┌──────────────────────────────────────────────────────────────────────────────────┐  │
 │  │                         KUBE-APISERVER (The Gateway)                             │  │
 │  │  • Exposes REST API • Authenticates & Authorizes (RBAC) • Admission Webhooks    │  │
 │  └───────┬──────────────────────────┬───────────────────────────────┬───────────────┘  │
 │          │                          │                               │                  │
 │          ▼                          ▼                               ▼                  │
 │  ┌───────────────┐        ┌───────────────────┐          ┌───────────────────────┐     │
 │  │     ETCD      │        │  KUBE-SCHEDULER   │          │KUBE-CONTROLLER-MANAGER│     │
 │  │ Distributed   │        │ Evaluates nodes,  │          │ Runs control loops:   │     │
 │  │ Key-Value DB  │        │ filters & scores  │          │ Node, Deployment,     │     │
 │  │ (Cluster State│        │ to assign pending │          │ ServiceAccount,       │     │
 │  │  Store)       │        │ pods              │          │ StatefulSet ctrls     │     │
 │  └───────────────┘        └───────────────────┘          └───────────────────────┘     │
 └────────────────────────────────────────────────────────────────────────────────────────┘
```

#### **Core Component Responsibilities:**
1. **`kube-apiserver`:** The single point of contact for the entire cluster. All components (etcd, scheduler, controllers, and worker node kubelets) communicate **strictly** with the API server. No component ever talks directly to etcd except the API server.
2. **`etcd`:** A strongly consistent, distributed key-value store implementing the Raft consensus algorithm. Holds 100% of cluster configuration, secrets, and live state data.
3. **`kube-scheduler`:** Watches for newly created Pods with no assigned node (`spec.nodeName == ""`). Filters nodes (checking resource requests, taints, affinities) and scores surviving nodes to select the optimal placement.
4. **`kube-controller-manager`:** Runs continuous controller reconciliation loops:
   - *DeploymentController:* Ensures desired replica counts match live pods.
   - *NodeController:* Detects when nodes go offline and initiates pod eviction.
   - *EndpointSliceController:* Manages IP endpoints for Kubernetes Services.
5. **`cloud-controller-manager`:** Interfaces with underlying cloud provider APIs to provision cloud load balancers, manage node routing, and attach storage volumes (EBS/PD).

---

### **6. What are the core components running on a Kubernetes Worker Node?**

**Detailed Answer:**
Worker nodes execute containerized workloads and report telemetry back to the control plane.
1. **`kubelet`:** An in-node agent that watches the API server for PodSpecs assigned to its node. Coordinates with the container runtime via the Container Runtime Interface (CRI) to pull images, configure storage mounts, start containers, and execute health probes.
2. **`kube-proxy`:** Manages network routing rules on the node (via iptables or IPVS) to handle Service ClusterIP load balancing (often bypassed/replaced by Cilium eBPF).
3. **Container Runtime (CRI):** The low-level runtime daemon responsible for pulling container images and managing container lifecycles (e.g., **`containerd`**, **`CRI-O`**).

---

### **7. What is a Pod and why does Kubernetes use Pods instead of running individual Containers directly?**

**Detailed Answer:**
A **Pod** is the smallest deployable atomic unit in Kubernetes. It encapsulates one or more tightly coupled containers that:
- **Share the Network Namespace:** All containers in a Pod share the exact same IP address, port space, and can communicate with each other over `localhost`.
- **Share IPC & Storage Volumes:** Containers can share shared-memory segments and mount the same in-memory or disk volumes.
- **Are Co-Scheduled:** Guaranteed to run on the exact same physical/virtual worker node.

#### **Why Pods Exist (The Sidecar Pattern):**
Applications frequently require auxiliary helper processes (e.g., an Envoy proxy sidecar for mTLS, a Fluent Bit agent tailing logs, or a Vault Agent fetching dynamic tokens). Treating the Pod as the atomic scheduling unit allows scaling and managing the main container and its auxiliary helper sidecars as a single cohesive unit.

---

### **8. What are the three Kubernetes Probe types? Explain their exact behaviors, failure thresholds, and recovery actions.**

**Detailed Answer:**

```
 Pod Lifecycle ➔ [ Startup Probe ] ──(Passed)──► [ Liveness Probe ] (Restarts container on fail)
                         │                              │
                    (Fails) ➔ Kills & restarts          └──► [ Readiness Probe ] (Removes from Service endpoints on fail)
```

#### **1. Startup Probe:**
- *Purpose:* Determines whether slow-starting applications (e.g., legacy Java apps taking 90 seconds to initialize) have finished booting.
- *Behavior:* While the startup probe is executing, all Liveness and Readiness probes are completely disabled.
- *On Failure:* If `failureThreshold * periodSeconds` timeout is exceeded, the container is killed and restarted.

#### **2. Liveness Probe:**
- *Purpose:* Determines whether the running container process has deadlocked or entered an unrecoverable crash state.
- *Behavior:* Periodically pings `/livez` or runs a command.
- *On Failure:* `kubelet` immediately terminates the container and restarts it according to its `restartPolicy` (`Always`, `OnFailure`).

#### **3. Readiness Probe:**
- *Purpose:* Determines whether the container is actively ready to receive incoming user network traffic.
- *Behavior:* Periodically pings `/readyz` (verifying database connections and cache warm-up).
- *On Failure:* **The container is NOT restarted.** Instead, the pod's IP is immediately removed from the Service's `EndpointSlice`, stopping all incoming load-balanced traffic until the probe passes again.

---

### **9. Compare `CMD` vs `ENTRYPOINT` in Dockerfiles with concrete operational examples.**

**Detailed Answer:**
- **`ENTRYPOINT`:** Defines the fixed binary executable that should always execute when the container starts.
- **`CMD`:** Defines default parameters/arguments passed into the `ENTRYPOINT`.

#### **Best Practice Pattern:**
```dockerfile
ENTRYPOINT ["/usr/local/bin/vault-agent"]
CMD ["-config=/etc/vault/config.hcl", "-log-level=info"]
```

#### **Runtime Behavior:**
1. Running `docker run my-vault`:
   Executes: `/usr/local/bin/vault-agent -config=/etc/vault/config.hcl -log-level=info`
2. Running `docker run my-vault -log-level=debug`:
   Executes: `/usr/local/bin/vault-agent -log-level=debug` *(Overrides `CMD` while keeping the fixed `ENTRYPOINT` executable intact)*.

---

### **10. What are Kubernetes Namespaces and what are their security and isolation limitations?**

**Detailed Answer:**
Namespaces divide a single physical Kubernetes cluster into virtual sub-clusters.
- **What Namespaces Isolate:** Scopes resource names (Pods, Services, Deployments), RBAC permissions (`RoleBinding`), and ResourceQuotas.
- **Security Limitations (What Namespaces DO NOT Isolate):**
  - **Zero Network Isolation by Default:** Any pod in `namespace-a` can communicate directly with any pod in `namespace-b` over the cluster network unless a `NetworkPolicy` explicitly denies traffic.
  - **Shared Linux Kernel:** All pods across all namespaces share the same underlying worker node Linux kernel.
  - **Non-Namespaced Cluster Resources:** `StorageClass`, `PersistentVolume`, `Node`, and `ClusterRole` exist at the global cluster level.

---

### **11. Compare Deployment vs StatefulSet vs DaemonSet.**

**Detailed Answer:**
- **Deployment:** For **Stateless Applications** (API servers, web frontends). Pods have random hash names (`app-6d8f-abc12`), are completely interchangeable, and can scale up/down in any order.
- **StatefulSet:** For **Stateful Applications** (PostgreSQL, Kafka, Elasticsearch). Pods have unique, sticky ordinal identities (`kafka-0`, `kafka-1`, `kafka-2`), ordered graceful rollout/termination, and attach to dedicated PersistentVolumeClaims (PVCs) that re-attach to the same pod ordinal upon reschedule.
- **DaemonSet:** Guarantees that **exactly one copy of a Pod runs on every single worker node** across the cluster. Used for node-level agents (Fluent Bit log shippers, Prometheus node-exporter, Cilium CNI plugins).

---

### **12. What is a Headless Service in Kubernetes and when is it required?**

**Detailed Answer:**
A **Headless Service** is defined with `spec.clusterIP: None`.
- Instead of provisioning a virtual ClusterIP that load-balances traffic across pods, Kubernetes CoreDNS returns direct `A`/`AAAA` DNS records containing the individual IP addresses of all underlying ready pods.
- **Primary Use Case:** Essential for StatefulSets (MongoDB, Cassandra, Kafka) where client applications or cluster peer nodes require direct peer-to-peer network discovery to communicate with specific primary or replica instances.

---

### **13. Compare `ClusterIP`, `NodePort`, and `LoadBalancer` Service types.**

**Detailed Answer:**
```
                     KUBERNETES SERVICE TYPES HIERARCHY
┌─────────────────────────────────────────────────────────────────────────────┐
│                          LoadBalancer Service                               │
│  Provisions Cloud Provider Load Balancer (AWS NLB/ALB, GCP LB)              │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                          NodePort Service                             │  │
│  │  Exposes static high port (30000-32767) on EVERY worker node IP       │  │
│  │                                                                       │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐  │  │
│  │  │                       ClusterIP Service                         │  │  │
│  │  │  Default. Internal IP accessible ONLY from inside cluster       │  │  │
│  │  └─────────────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### **14. Explain Kubernetes Resource Requests vs Limits and the Completely Fair Scheduler (CFS) throttling behavior.**

**Detailed Answer:**
- **`requests` (Guaranteed Minimum):** The amount of CPU and Memory guaranteed to the container. The `kube-scheduler` uses requests to find nodes with adequate capacity.
- **`limits` (Hard Ceiling):** The maximum allowable resource consumption.
  - **Memory Limits Exceeded:** The Linux kernel OOM (Out Of Memory) killer immediately terminates the container process with **Exit Code 137 (OOMKilled)**.
  - **CPU Limits Exceeded:** The Linux CFS (Completely Fair Scheduler) **throttles** the container by freezing its CPU execution time in 100ms quota periods. The container is not killed, but experiences severe latency spikes.

---

### **15. What is a Kubernetes ConfigMap vs Secret, and why is etcd encryption at rest mandatory?**

**Detailed Answer:**
- **`ConfigMap`:** Stores non-sensitive configuration parameters in plain text key-value pairs.
- **`Secret`:** Stores sensitive tokens, passwords, and TLS certificates encoded in base64.
- **Crucial Security Truth:** Base64 is **encoding, not encryption**. Anyone with read access to etcd or Kubernetes API can run `echo "cGFzc3dvcmQ=" | base64 -d`.
- **Mandatory Hardening:** Production clusters must enable **etcd Encryption at Rest** using KMS (AWS KMS, Vault) so that secrets stored in etcd disks are encrypted with AES-CBC or AES-GCM envelope keys.

---

### **16. What is the Kubernetes Garbage Collector and how does Cascading Deletion work?**

**Detailed Answer:**
The Garbage Collector cleans up cluster resources that no longer have owner references.
- **Foreground Cascading Deletion:** Owner object enters "deletion in progress", dependent child pods are terminated first, and finally the owner is deleted.
- **Background Cascading Deletion (Default):** The owner object is deleted immediately from etcd, and the garbage collector asynchronously deletes child pods in the background.
- **Non-Cascading Deletion:** Deletes the owner object while leaving dependent pods orphaned and running.

---

### **17. Compare Docker Volumes vs Bind Mounts.**

**Detailed Answer:**
- **Docker Volumes (`docker volume create`):** Managed completely by Docker within host storage (`/var/lib/docker/volumes/`). Safe, isolated, portable, and independent of host directory structures.
- **Bind Mounts (`-v /host/path:/container/path`):** Binds an arbitrary directory on the host filesystem directly into the container. Highly dependent on host file permissions; ideal for local development hot-reloading.

---

### **18. How do you roll back a Kubernetes Deployment using `kubectl rollout`?**

**Detailed Answer:**
```bash
# 1. View rollout history and revisions
kubectl rollout history deployment/payment-service -n production

# 2. Inspect a specific historical revision details
kubectl rollout history deployment/payment-service --revision=2 -n production

# 3. Roll back to the immediately previous revision
kubectl rollout undo deployment/payment-service -n production

# 4. Roll back to a specific target revision
kubectl rollout undo deployment/payment-service --to-revision=1 -n production
```

---

### **19. Compare Kubernetes Jobs vs CronJobs.**

**Detailed Answer:**
- **Job:** Creates one or more pods and ensures that a specified number of them terminate successfully with Exit Code 0 (e.g., database schema migrations, batch image processing).
- **CronJob:** Executes a Job on a recurring cron schedule (`0 2 * * *`). Supports concurrency policies:
  - `Allow`: Concurrent executions permitted.
  - `Forbid`: Skips next run if previous run is still active.
  - `Replace`: Cancels currently running job and starts new one.

---

### **20. What is an Init Container and what are its common production use cases?**

**Detailed Answer:**
An Init Container is a specialized container that runs and must complete to Exit Code 0 **before** the application containers in the Pod start.
- **Common Production Use Cases:**
  1. **Dependency Waiting:** Blocking pod startup until a PostgreSQL database port is reachable (`nc -z postgres 5432`).
  2. **Security Token Retrieval:** Fetching dynamic secrets from Vault and writing them to a shared in-memory `emptyDir` volume.
  3. **Kernel Parameter Tuning:** Running `sysctl -w net.core.somaxconn=1024` (requires elevated capability `NET_ADMIN` in init container only, keeping the app container unprivileged).

---

## 🟡 **Intermediate Level (Questions 21–40)**

### **21. What is the Kubernetes Gateway API and why is it superior to the legacy Ingress resource?**

**Detailed Answer:**

```
                               KUBERNETES GATEWAY API ROLES
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │ 1. Infrastructure Provider: Configures GatewayClass (Envoy, Cilium, AWS VPC Lattice)  │
 ├────────────────────────────────────────────────────────────────────────────────────────┤
 │ 2. Cluster Administrator: Provisions Gateway (Declares ports 80/443, TLS certs, IPs)   │
 ├────────────────────────────────────────────────────────────────────────────────────────┤
 │ 3. Application Developers: Attaches HTTPRoute, GRPCRoute, TCPRoute                     │
 │    (Defines canary traffic splitting, path rewrites, headers without cluster admin!)   │
 └────────────────────────────────────────────────────────────────────────────────────────┘
```

#### **Why Gateway API Replaces Ingress:**
- **Role-Oriented Separation:** Ingress forced developers and cluster admins to edit the same resource. Gateway API cleanly splits responsibilities.
- **Native Advanced Traffic Splitting:** Built-in canary traffic weighting and header-based routing without messy vendor-specific annotations.
- **Cross-Namespace Routing:** Allows a shared central Gateway in the `infra` namespace to route traffic to `HTTPRoute` resources defined in independent application namespaces.

---

### **22. What is Karpenter and how does its declarative node provisioning outperform the legacy Cluster Autoscaler?**

**Detailed Answer:**
Karpenter is an open-source, high-performance node autoscaler designed by AWS and CNCF.

#### **Detailed Comparison:**
- **Cluster Autoscaler:** Bound to rigid cloud Auto Scaling Groups (ASGs). Sizing is slow (2–5 minutes), cannot easily mix heterogeneous instance types, and scaling out requires waiting for ASG node provisioning.
- **Karpenter:** Directly invokes cloud EC2 Fleet APIs, bypassing ASGs entirely.
  - Provisions right-sized nodes in **under 45 seconds**.
  - Dynamically evaluates pending pods and selects optimal combinations of Spot/On-Demand, Graviton/ARM64 and x86 architectures.
  - Continuously executes **Node Consolidation**: identifies underutilized nodes, computes an optimal single replacement node, drains pods, and terminates waste to minimize cloud bills.

---

### **23. What is Cilium and why is eBPF replacing iptables and kube-proxy in modern Kubernetes clusters?**

**Detailed Answer:**
Traditional `kube-proxy` writes sequential Linux `iptables` rules for every Service and Endpoint.
- **The iptables Bottleneck:** `iptables` is an $O(N)$ sequential packet filter. In clusters with 5,000+ services, traversing linear iptables rules causes severe packet latency and high CPU overhead.

#### **The Cilium eBPF Solution:**
- Replaces `kube-proxy` with **eBPF (Extended Berkeley Packet Filter)** bytecode attached directly to Linux socket layers (`sockops`) and network device interfaces (XDP / TC).
- Achieves **$O(1)$ hash table lookups** for service routing, processing packets at near-wire speeds.
- Enables transparent Layer 7 observability (Hubble) and kernel-level encryption (WireGuard) with zero proxy sidecars.

---

### **24. Explain Kubernetes Pod Security Standards (PSS) and Pod Security Admission (PSA).**

**Detailed Answer:**
PSA replaced legacy PodSecurityPolicies (PSP) in Kubernetes 1.25+.

#### **The Three Standard Profiles:**
1. **Privileged:** Unrestricted; pods can run as root, use host networking, and mount raw host paths.
2. **Baseline:** Minimally restrictive; prevents known privilege escalations (blocks host network, host ports, raw devices).
3. **Restricted:** Hardened enterprise best-practice; mandates `runAsNonRoot: true`, drops all default capabilities except `NET_BIND_SERVICE`, enforces read-only root filesystems, and restricts volume types.

```yaml
# Enforce Restricted profile on production namespace
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/warn: restricted
```

---

### **25. How do you troubleshoot a Pod stuck in `CrashLoopBackOff`? Provide the exact command sequence.**

**Detailed Answer:**
```bash
# Step 1: Check Pod Status, Node Assignment, and Restart Count
kubectl get pods -n production -o wide

# Step 2: Describe Pod to inspect Exit Code, Reason, and Lifecycle Events
kubectl describe pod payment-service-84f9b -n production

# Step 3: Check Current Container stdout/stderr Logs
kubectl logs payment-service-84f9b -n production --all-containers

# Step 4: Check PREVIOUS Container Crash Logs (Crucial for fast crashes!)
kubectl logs payment-service-84f9b -n production --previous

# Step 5: Check Node-Level Cluster Events
kubectl get events -n production --sort-by='.metadata.creationTimestamp'
```

---

### **26. What are the common Container Exit Codes and their underlying causes?**

**Detailed Answer:**
- **Exit Code 0:** Clean execution / Success (normal for completed Jobs/Init containers).
- **Exit Code 1:** Application runtime exception / uncaught error in code.
- **Exit Code 126:** Command cannot be executed (binary permissions issue).
- **Exit Code 127:** Command not found (wrong path or missing binary in image).
- **Exit Code 137 ($128 + 9$ = SIGKILL):** Process was killed immediately by the OS. Almost always indicates **OOMKilled** (exceeded memory limit).
- **Exit Code 143 ($128 + 15$ = SIGTERM):** Graceful shutdown signal sent by Kubernetes during pod scale-down or node drain.

---

### **27. What is `kubectl debug` and how do Ephemeral Containers work?**

**Detailed Answer:**
Production distroless and scratch container images contain no debugging tools (`sh`, `curl`, `netstat`).
- **`kubectl debug` Solution:** Dynamically injects an **Ephemeral Container** containing full debugging utilities (`nicolaka/netshoot`) into the running Pod's shared Linux namespaces:
```bash
kubectl debug -it pod/payment-service-84f9b -n production \
  --image=nicolaka/netshoot --target=payment-container
```
Allows running `tcpdump`, `curl`, and inspecting `/proc` of the live container without restarting the pod.

---

### **28. Compare Istio Sidecar Architecture vs Istio Ambient Mesh.**

**Detailed Answer:**
- **Sidecar Architecture (Classic):** Injects an Envoy proxy container into every application pod. High CPU/RAM overhead; requires application restarts on proxy upgrades.
- **Ambient Mesh (Sidecarless):** Decouples mesh processing into two shared node-level layers:
  1. **`ztunnel` (Zero Trust Tunnel):** A lightweight per-node daemon handling mTLS and Layer 4 identity.
  2. **Waypoint Proxy:** Dedicated per-namespace Envoy instances handling Layer 7 routing only when explicitly required.
  - *Result:* Over 70% reduction in compute overhead and zero pod restarts on upgrades.

---

### **29. What is KEDA and how does it implement event-driven autoscaling for message queues?**

**Detailed Answer:**
KEDA (Kubernetes Event-driven Autoscaling) extends Kubernetes HPA to scale workloads based on external event sources (AWS SQS, Kafka, RabbitMQ).

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: sqs-scaler
spec:
  scaleTargetRef:
    name: order-processor
  minReplicaCount: 0   # Supports scale-to-zero!
  maxReplicaCount: 50
  triggers:
    - type: aws-sqs-queue
      metadata:
        queueURL: https://sqs.us-east-1.amazonaws.com/123456789012/order-queue
        queueLength: "10"  # 1 pod per 10 pending messages
```

---

### **30. Explain Node Affinity, Pod Anti-Affinity, Taints, and Tolerations.**

**Detailed Answer:**
- **Node Affinity:** Directs pods to nodes matching labels (`topology.kubernetes.io/zone: us-east-1a`).
- **Pod Anti-Affinity:** Prevents pods of the same application from co-locating on the same node/zone, guaranteeing high availability.
- **Taints (Node):** Repels pods unless they explicitly tolerate the taint (`gpu=true:NoSchedule`).
- **Tolerations (Pod):** Allows pods to schedule on matching tainted nodes.

---

### **31. Walk through the Kubernetes Pod Graceful Termination Lifecycle step-by-step.**

**Detailed Answer:**
1. **Endpoint Removal:** Pod is marked `Terminating` and immediately removed from Service EndpointSlices; kube-proxy/Cilium stops routing new traffic.
2. **`preStop` Hook:** Kubernetes executes `preStop` script inside the container (e.g., `sleep 10` allowing in-flight requests to complete).
3. **`SIGTERM` Signal:** `kubelet` sends `SIGTERM` to container PID 1, initiating graceful application connection draining.
4. **Grace Period Countdown:** Kubernetes waits up to `terminationGracePeriodSeconds` (default: 30s).
5. **`SIGKILL` Signal:** If processes remain alive after timeout, `kubelet` sends `SIGKILL` (Exit Code 137), forcibly terminating the container.

---

### **32. What is a Pod Disruption Budget (PDB) and why is it critical during node upgrades?**

**Detailed Answer:**
A PDB limits the number of pods of a replicated service that can be simultaneously offline during voluntary disruptions (`kubectl drain`, node patching):
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-pdb
spec:
  minAvailable: 80%
  selector:
    matchLabels:
      app: api
```
If draining a node causes available replicas to drop below 80%, the drain operation safely blocks until replacement pods become ready on another node.

---

### **33. Compare StorageClass, PersistentVolume (PV), and PersistentVolumeClaim (PVC).**

**Detailed Answer:**
- **`StorageClass`:** Defines dynamic volume provisioners (AWS EBS CSI, GCP PD) and volume parameters (IOPS, encryption).
- **`PersistentVolume (PV)`:** A piece of provisioned physical/cloud storage in the cluster.
- **`PersistentVolumeClaim (PVC)`:** A user's request for storage. Binds to a matching PV.

---

### **34. Compare CNI Plugins: AWS VPC CNI vs Cilium vs Calico.**

**Detailed Answer:**
- **AWS VPC CNI:** Assigns native AWS VPC secondary IP addresses directly to Pods. High throughput, but can exhaust VPC IP subnets.
- **Cilium:** High-performance eBPF CNI supporting overlay and direct routing, Hubble L7 observability, and transparent WireGuard encryption.
- **Calico:** High-performance CNI widely used for enterprise BGP routing and scalable Layer 3/4 NetworkPolicies.

---

### **35. What is API Priority and Fairness (APF) in `kube-apiserver`?**

**Detailed Answer:**
APF protects the Kubernetes API server from request flooding. It classifies incoming API requests into PriorityLevels and FlowSchemas, guaranteeing that control plane controllers and leader election requests are never starved by high-volume third-party query storms.

---

### **36. What is ValidatingAdmissionPolicy with Common Expression Language (CEL) in Kubernetes 1.30+?**

**Detailed Answer:**
CEL allows writing in-process admission validation rules directly in Kubernetes manifests without external webhook controllers (like OPA/Kyverno):
```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingAdmissionPolicy
metadata:
  name: require-non-root
spec:
  validations:
    - expression: "object.spec.securityContext.runAsNonRoot == true"
      message: "Pods must run as non-root!"
```

---

### **37. What is Rootless Container Execution and why is it superior for security?**

**Detailed Answer:**
Standard Docker runs its daemon as the host `root` user. In **Rootless execution** (Rootless Docker / Podman), the daemon and containers run inside an unprivileged user namespace. Root inside the container maps to a normal unprivileged UID (e.g., UID 10001) on the host; a container breakout leaves the attacker trapped without host privileges.

---

### **38. Compare containerd vs Docker vs CRI-O.**

**Detailed Answer:**
- **Docker:** High-level developer platform (CLI, build engine, desktop UI).
- **containerd:** Core lightweight container runtime daemon implementing OCI and CRI standards.
- **CRI-O:** Minimalist, purpose-built container runtime designed exclusively for Kubernetes CRI.

---

### **39. Compare Mutating Webhooks vs Validating Webhooks in Kubernetes.**

**Detailed Answer:**
- **Mutating Webhook:** Intercepts API requests *first* and can modify/inject data (e.g., injecting Istio sidecars).
- **Validating Webhook:** Intercepts API requests *after* mutation and accepts or rejects the resource based on compliance rules.

---

### **40. What is OIDC Authentication with the Kubernetes API Server?**

**Detailed Answer:**
Connects `kube-apiserver` to enterprise Identity Providers (Okta, Keycloak, Azure AD). Users authenticate via corporate SSO to obtain short-lived JWT tokens, and Kubernetes validates claims to enforce RBAC without managing static client certificates.

---

## 🔴 **Advanced & Scenario-Based Level (Questions 41–60)**

### **41. Scenario: CoreDNS in your Kubernetes cluster is maxing out CPU and pods experience 5-second connection timeouts. Walk through the root cause analysis and resolution.**

**Detailed Answer:**
**Root Cause: The Linux glibc `ndots:5` Resolution Storm.**
By default, `/etc/resolv.conf` in pods specifies `options ndots:5`. When an app queries `api.stripe.com` (2 dots), glibc appends all local search domains sequentially:
1. `api.stripe.com.default.svc.cluster.local` $\rightarrow$ NXDOMAIN
2. `api.stripe.com.svc.cluster.local` $\rightarrow$ NXDOMAIN
3. `api.stripe.com.cluster.local` $\rightarrow$ NXDOMAIN
4. `api.stripe.com` $\rightarrow$ Resolved!
This creates 4 DNS queries per outbound connection, overwhelming CoreDNS.

#### **Permanent Fixes:**
1. **Deploy NodeLocal DNSCache:** Runs a DaemonSet caching DNS queries locally on each node loopback (`169.254.20.10`).
2. **Use FQDNs:** Append trailing dots in code configs (`api.stripe.com.`), bypassing search domain expansion.
3. **Configure Pod `dnsConfig`:** Set `ndots: "2"`.

---

### **42. Scenario: A Java application pod is repeatedly terminated with Exit Code 137 (OOMKilled) even though JVM heap is set to 4GB and Pod memory limit is 6GB. What is consuming the memory?**

**Detailed Answer:**
**Root Cause: Off-Heap / Native Memory Consumption.**
Container memory limits constrain the entire Linux cgroup, not just the JVM Heap.
- **Off-Heap Consumers:** Metaspace class metadata, thread stacks ($N \text{ threads} \times 1\text{MB}$ `-Xss`), direct byte buffers (Netty/gRPC buffers), and GC tracking tables.
- **Fix:** Enable Native Memory Tracking (`-XX:NativeMemoryTracking=summary`), limit Metaspace (`-XX:MaxMetaspaceSize=512m`), and tune JVM container support (`-XX:MaxRAMPercentage=75.0`).

---

### **43. Scenario: A worker node enters `NotReady` status. What internal kubelet and Linux diagnostics do you perform?**

**Detailed Answer:**
1. **Check Node Conditions:** `kubectl describe node <node-name>` (check `MemoryPressure`, `DiskPressure`, `PIDPressure`).
2. **Inspect Kubelet Service:** `systemctl status kubelet` and `journalctl -u kubelet -n 100 --no-pager`.
3. **Check Container Runtime:** `systemctl status containerd` and `crictl ps`.
4. **Check Kernel Logs for OOM or I/O Stalls:** `dmesg -T | grep -E -i 'oom|hung_task'`.
5. **Check Disk Space:** `df -h /var/lib/containerd`.

---

### **44. How does Kubernetes CFS Bandwidth Throttling cause latency spikes on latency-critical microservices?**

**Detailed Answer:**
When `resources.limits.cpu: "1000m"` is set, the Linux kernel CFS quota enforces this in 100ms periods. If a multi-threaded app bursts and consumes its 100ms quota in the first 20ms, the kernel **hard-freezes the process for the remaining 80ms**, causing severe p99 latency spikes.
- **Best Practice:** Remove CPU limits on latency-critical workloads while keeping accurate CPU `requests`.

---

### **45. Scenario: Architect a Multi-Tenant Kubernetes cluster ensuring strong isolation between multiple enterprise customers.**

**Detailed Answer:**
1. **Control Plane Isolation:** Deploy **`vcluster`** (virtual Kubernetes control planes per tenant).
2. **Network Isolation:** Enforce Default-Deny Cilium NetworkPolicies.
3. **Compute Sandboxing:** Enforce `gVisor` (`runsc`) or Kata Containers runtime for untrusted workloads.
4. **Admission Governance:** Kyverno enforcing Restricted Pod Security Standards.
5. **Resource Quotas:** Hard limits on CPU, memory, and storage per tenant namespace.

---

### **46. How do you safely upgrade a production Kubernetes cluster across minor versions with zero customer downtime?**

**Detailed Answer:**
1. **Check API Deprecations:** Run `kubent` / Pluto.
2. **Upgrade Control Plane:** Upgrade API server, etcd, scheduler, controller manager.
3. **Upgrade Add-ons:** Update Cilium CNI, CoreDNS, Gateway API CRDs.
4. **Worker Node Rolling Upgrade:**
   - `kubectl cordon <node>` $\rightarrow$ `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data` (respects PDBs) $\rightarrow$ upgrade `kubelet`/`containerd` $\rightarrow$ `kubectl uncordon <node>`.

---

### **47. How do you recover an etcd cluster after losing a majority of master nodes?**

**Detailed Answer:**
1. Stop etcd on the surviving node.
2. Create emergency snapshot: `etcdctl snapshot save snapshot.db`.
3. Restore snapshot into a new single-node cluster using `--force-new-cluster`:
   ```bash
   etcdctl snapshot restore snapshot.db --name master-1 --initial-cluster master-1=https://10.0.0.1:2380
   ```
4. Start etcd and join replacement nodes sequentially.

---

### **48. How do you implement End-to-End mTLS and Cryptographic Workload Identity with SPIFFE/SPIRE?**

**Detailed Answer:**
- **SPIRE DaemonSet:** Attests workload identity by querying `kubelet` for pod UID, namespace, and ServiceAccount.
- **SVID Issuance:** Projects short-lived, auto-rotating X.509 certificates into memory via Unix domain sockets.
- **Result:** Workloads establish zero-trust mTLS without managing static private keys.

---

### **49. What are Kubernetes Generic Ephemeral Volumes?**

**Detailed Answer:**
Generic Ephemeral Volumes create dynamic, pod-scoped PersistentVolumeClaims that share the exact lifecycle of the Pod (created when pod schedules, deleted when pod terminates), allowing workloads to leverage high-speed cloud SSD storage with full StorageClass features without managing independent PVC lifecycles.

---

### **50. Walk through the exact control loop sequence when you run `kubectl delete pod <pod-name>`.**

**Detailed Answer:**
1. **API Server:** Sets deletion timestamp on Pod in etcd.
2. **Kubelet:** Executes graceful shutdown (`preStop` $\rightarrow$ `SIGTERM` $\rightarrow$ grace period $\rightarrow$ `SIGKILL`) and notifies API server.
3. **ReplicaSet Controller:** Detects actual pods ($4$) $<$ desired replicas ($5$), and creates a new Pod object in `Pending` state.
4. **Kube-Scheduler:** Filters and scores nodes, assigning the new pod to a node.
5. **Kubelet on New Node:** Pulls image, creates namespaces/cgroups, mounts storage, and starts containers.
6. **EndpointSlice Controller:** Adds new pod IP to Service once readiness probe passes.

---

### **51. Compare iptables mode vs IPVS mode in kube-proxy.**

**Detailed Answer:**
- **iptables:** $O(N)$ sequential rule evaluation; high CPU and latency in large clusters ($> 5,000$ services).
- **IPVS:** $O(1)$ hash table lookups in kernel transport layer; supports advanced load balancing (least connection, locality-based) and scales to tens of thousands of services.

---

### **52. What is the Kubernetes Native Sidecar Container Lifecycle feature (KEP-753)?**

**Detailed Answer:**
Declared inside `initContainers` with `restartPolicy: Always`:
- Starts *before* main application containers and blocks app startup until its startup probe passes.
- Terminates *after* main application containers finish in batch Jobs, eliminating hanging job issues.

---

### **53. Compare cgroups v1 vs cgroups v2 in Kubernetes.**

**Detailed Answer:**
- **cgroups v1:** Fragmented hierarchies; poor memory writeback tracking and inaccurate OOM kills.
- **cgroups v2:** Unified hierarchy; accurate memory tracking, PSI (Pressure Stall Information), and native rootless container support.

---

### **54. What is Container Image Layer Squashing and what are its trade-offs?**

**Detailed Answer:**
- **Squashing:** Merges all build layers into a single filesystem layer.
- **Trade-offs:** Shrinks image size by discarding intermediate deleted files, but destroys Docker layer caching across images on the same worker node.

---

### **55. How do you prevent Node Eviction storms during memory pressure?**

**Detailed Answer:**
1. Configure `--eviction-hard=memory.available<500Mi` in Kubelet.
2. Reserve system resources via `system-reserved` and `kube-reserved`.
3. Use **Guaranteed QoS** (`requests == limits`) on mission-critical pods.

---

### **56. Explain Kubernetes Quality of Service (QoS) Classes.**

**Detailed Answer:**
- **Guaranteed:** `requests == limits` for all CPU and memory containers. Lowest eviction priority.
- **Burstable:** Requests defined, but limits are higher or unset. Evicted when Guaranteed pods need reserved memory.
- **BestEffort:** No requests or limits. Evicted first during node resource pressure.

---

### **57. Explain HPA v2 with Custom and External Metrics.**

**Detailed Answer:**
HPA v2 scales pods using:
- **Resource Metrics:** CPU / Memory percentages.
- **Custom Metrics:** In-cluster metrics (e.g., Ingress HTTP request rate).
- **External Metrics:** Out-of-cluster metrics (e.g., AWS SQS queue depth, Datadog monitors).

---

### **58. What is a Kubernetes Finalizer and how do you fix a Namespace stuck in "Terminating"?**

**Detailed Answer:**
A Finalizer is a pre-delete hook key in metadata preventing deletion until cleanup finishes.
- **Fix:** Remove finalizers array via API replace:
```bash
kubectl get ns <stuck-ns> -o json | jq '.spec.finalizers = []' | kubectl replace --raw "/api/v1/namespaces/<stuck-ns>/finalize" -f -
```

---

### **59. Compare eBPF Tetragon vs Falco for Runtime Security.**

**Detailed Answer:**
- **Falco:** Parses kernel system calls in userspace and triggers alerts on suspicious behavior.
- **Tetragon (by Cilium):** Executes security enforcement directly **in-kernel via eBPF**, capable of instantly killing malicious processes before dangerous syscalls execute.

---

### **60. Scenario: Migrate 200 stateful and stateless microservices to AWS EKS with zero downtime.**

**Detailed Answer:**
1. **Network Link:** Direct Connect / Site-to-Site VPN to AWS VPC.
2. **Data Sync:** Live streaming replication for databases (PostgreSQL/MySQL) and AWS DataSync for object storage.
3. **Deploy Workloads:** ArgoCD synchronizes deployments onto EKS.
4. **DNS Traffic Shift:** Route 53 Weighted routing (5% $\rightarrow$ 25% $\rightarrow$ 50% $\rightarrow$ 100%).
5. **Database Cutover:** Promote EKS database replica to Primary during a 30-second maintenance window.
