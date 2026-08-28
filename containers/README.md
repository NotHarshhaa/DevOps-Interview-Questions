# **Containers & Kubernetes - DevOps Interview Questions**

Welcome to the **Containers & Kubernetes** interview questions module. This section covers Docker, containerd, CRI-O, modern Kubernetes (1.28–1.32+), Gateway API, Karpenter, eBPF & Cilium networking, Pod Security Standards, KEDA, and real-world production troubleshooting.

---

## 🟢 **Beginner Level (Questions 1–20)**

### **1. What is the fundamental difference between a Virtual Machine (VM) and a Container?**
**Answer:**
| Feature | Virtual Machine (VM) | Container |
| :--- | :--- | :--- |
| **Architecture** | Hypervisor runs full guest OS on virtualized hardware | Shares host Linux kernel; isolated via namespaces & cgroups |
| **Startup Time** | Minutes | Milliseconds to seconds |
| **Resource Usage** | Heavy (GBs of RAM, fixed disk allocation) | Ultra-lightweight (MBs of RAM, shared binaries) |
| **Isolation** | Hardware-level isolation | OS-level isolation (Process / Namespace) |
| **Portability** | Hypervisor dependent | High (OCI standard runtimes) |

---

### **2. How do Linux Namespaces and Control Groups (cgroups) create a container?**
**Answer:**
Containers are not physical constructs; they are standard Linux processes isolated by two kernel primitives:
1. **Namespaces (Isolation - What a container can *see*):**
   - `pid`: Process tree isolation (container process sees itself as PID 1).
   - `net`: Network interfaces, IP routing tables, port bindings.
   - `mnt`: Mount points and root filesystem isolation.
   - `ipc`: Inter-process communication isolation.
   - `uts`: Hostname isolation.
   - `user`: UID/GID mapping (root in container maps to unprivileged user on host).
2. **Control Groups - cgroups v2 (Resource Limits - How much a container can *use*):**
   - Limits and measures CPU time, memory, disk I/O, and process count (`pids.max`).

---

### **3. What is a Docker Multi-Stage Build and why is it essential for production?**
**Answer:**
Multi-stage builds allow using multiple `FROM` instructions in a single Dockerfile. Heavy build tools, compilers (e.g., Go/Rust compilers, JDK), and source files are kept in the builder stage, while only the resulting compiled binary is copied into a minimal, secure runtime image.

**Example:**
```dockerfile
# Stage 1: Build stage
FROM golang:1.24-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o server .

# Stage 2: Production runtime stage
FROM gcr.io/distroless/static-debian12:nonroot
WORKDIR /
COPY --from=builder /app/server /server
USER nonroot:nonroot
ENTRYPOINT ["/server"]
```
**Benefits:** Reduces image size from 800MB+ to < 20MB, eliminates shell/package managers, and minimizes attack surface.

---

### **4. What are Distroless and Scratch container base images?**
**Answer:**
- **`scratch`:** An explicitly empty Docker image (`0 bytes`). Used for statically compiled binaries (Go, Rust) requiring zero OS libraries.
- **`distroless` (by Google):** Contains only the minimal runtime dependencies (like SSL certificates, `glibc`, tzdata) for specific languages (Java, Node.js, Python) **without** a package manager (`apt`/`apk`), shells (`/bin/sh`, `/bin/bash`), or standard Linux utilities.
- **Why use them:** Prevents attackers who exploit code vulnerabilities from spawning interactive shells or downloading malware via `curl`/`wget`.

---

### **5. What is the Kubernetes Control Plane and what are its core components?**
**Answer:**
The Control Plane makes global decisions about the cluster (scheduling, detecting and responding to cluster events):
1. **`kube-apiserver`:** The front end for the control plane; exposes the Kubernetes REST API. All internal and external communication goes through it.
2. **`etcd`:** Consistent, highly-available distributed key-value store holding all cluster data and state.
3. **`kube-scheduler`:** Watches for newly created Pods with no assigned node and selects the optimal worker node based on resource requests, taints, and affinities.
4. **`kube-controller-manager`:** Runs controller loops (Node Controller, Deployment Controller, Job Controller, ServiceAccount Controller).
5. **`cloud-controller-manager`:** Integrates with underlying cloud provider APIs (load balancers, routing, storage volumes).

---

### **6. What are the core components running on a Kubernetes Worker Node?**
**Answer:**
1. **`kubelet`:** An agent that ensures containers described in PodSpecs are running and healthy on the node.
2. **`kube-proxy`:** Maintains network routing rules on nodes (via iptables or IPVS) to handle Service ClusterIP and NodePort traffic (often replaced by Cilium eBPF).
3. **Container Runtime (CRI):** The software responsible for running containers (e.g., `containerd`, `CRI-O`).

---

### **7. What is the difference between a Pod and a Container?**
**Answer:**
A **Pod** is the smallest deployable unit in Kubernetes. It encapsulates one or more tightly coupled containers that:
- Share the same Network namespace (same IP address, port space, and `localhost` communication).
- Share the same IPC namespace and shared storage Volumes.
- Are co-scheduled and run on the exact same physical/virtual worker node.

---

### **8. What are the three Kubernetes Probe types and how do they differ?**
**Answer:**
1. **Startup Probe:** Determines if the application inside the container has initialized. All other probes are disabled until the startup probe succeeds. Ideal for slow-starting legacy apps.
2. **Liveness Probe:** Determines if the container is still running properly. If it fails, `kubelet` terminates and **restarts** the container according to its `restartPolicy`.
3. **Readiness Probe:** Determines if the container is ready to accept incoming user traffic. If it fails, the pod's IP is removed from the associated Service Endpoints/EndpointsSlice so no traffic is routed to it.

---

### **9. What is the difference between `CMD` and `ENTRYPOINT` in a Dockerfile?**
**Answer:**
- **`ENTRYPOINT`:** Defines the fixed executable that should always run when the container starts.
- **`CMD`:** Defines default arguments passed to the `ENTRYPOINT`. These can be easily overridden by passing CLI arguments to `docker run`.

**Best Practice Pattern:**
```dockerfile
ENTRYPOINT ["/app/service"]
CMD ["--config", "/etc/service.yaml"]
```
*(Running `docker run my-image --verbose` overrides `CMD` while keeping `/app/service` as the entrypoint).*

---

### **10. What are Kubernetes Namespaces and what are their limitations?**
**Answer:**
Namespaces provide virtual isolation within a single physical Kubernetes cluster for multi-team or multi-environment sharing.
- **What they isolate:** Scopes naming of resources (Deployments, Services, ConfigMaps), RBAC permissions, and ResourceQuotas.
- **Limitations:** Namespaces do **not** provide network isolation by default (any pod can talk to any pod in another namespace unless `NetworkPolicy` is enforced) and do not isolate nodes or the Linux kernel.

---

### **11. What is the difference between Deployment, StatefulSet, and DaemonSet?**
**Answer:**
- **Deployment:** Manages stateless pod replicas. Pods are interchangeable, have random hashes in their names (`app-6d4c5-8xk2q`), and can be rescheduled on any node.
- **StatefulSet:** Manages stateful workloads. Pods have unique, persistent, ordinal identities (`db-0`, `db-1`), ordered graceful scaling/termination, and dedicated PersistentVolumeClaims.
- **DaemonSet:** Ensures that **exactly one copy** of a Pod runs on all (or selected) worker nodes across the cluster (used for log forwarders like Fluentbit, monitoring agents like Prometheus node-exporter, or CNI plugins like Cilium).

---

### **12. What is a Headless Service in Kubernetes and when is it used?**
**Answer:**
A Headless Service is a Service defined with `spec.clusterIP: None`.
Instead of load-balancing requests across pods via a single virtual ClusterIP, CoreDNS returns direct DNS `A`/`AAAA` records containing the individual IP addresses of all underlying Pods.

**Use Case:** Essential for StatefulSets (e.g., MongoDB, Kafka, Elasticsearch) where client applications need direct peer-to-peer discovery to communicate with specific primary or replica nodes.

---

### **13. What is the difference between `ClusterIP`, `NodePort`, and `LoadBalancer` Service types?**
**Answer:**
- **`ClusterIP` (Default):** Exposes the Service on an internal IP reachable only from *inside* the Kubernetes cluster.
- **`NodePort`:** Exposes the Service on a static high-range port (`30000-32767`) on every worker node's external IP address.
- **`LoadBalancer`:** Requests a dedicated cloud load balancer (e.g., AWS NLB/ALB, GCP Cloud Load Balancer) that forwards external traffic directly to the service pods.

---

### **14. What are Kubernetes Resource Requests vs Limits?**
**Answer:**
- **Requests:** The guaranteed minimum amount of CPU and Memory that Kubernetes reserves for the container. The `kube-scheduler` uses requests to decide which worker node has sufficient capacity to host the pod.
- **Limits:** The hard upper ceiling of resources the container is allowed to consume on the node.
  - **CPU exceeded:** The kernel throttles container CPU time (CFS bandwidth throttling); the container does *not* terminate.
  - **Memory exceeded:** The kernel OOM (Out Of Memory) killer immediately terminates the container process with **Exit Code 137 (OOMKilled)**.

---

### **15. What is a Kubernetes ConfigMap vs Secret?**
**Answer:**
- **`ConfigMap`:** Stores non-sensitive configuration data in plain text key-value pairs or configuration files.
- **`Secret`:** Stores sensitive data (passwords, tokens, TLS certs) encoded in base64.
  - *Security Note:* Base64 is encoding, not encryption! Production clusters must enable **Encryption at Rest in etcd** using AWS KMS / HashiCorp Vault transit keys.

---

### **16. What is the Kubernetes Garbage Collector and how does Cascading Deletion work?**
**Answer:**
The Garbage Collector cleans up cluster resources that no longer have owner references.
- **Foreground Cascading Deletion:** The owner object enters "deletion in progress" state, its dependents (children) are deleted first, and finally the owner is deleted.
- **Background Cascading Deletion (Default):** Kubernetes deletes the owner object immediately, and the garbage collector asynchronously removes the orphaned child pods in the background.
- **Non-Cascading:** Deletes the owner while leaving dependent pods orphaned.

---

### **17. What is a Docker Volume vs Bind Mount?**
**Answer:**
- **Docker Volume (`docker volume create`):** Managed completely by Docker in a dedicated storage area (`/var/lib/docker/volumes/`). Highly portable, isolated from host OS directory structure, and safe for production.
- **Bind Mount (`-v /host/path:/container/path`):** Maps any arbitrary file or directory on the host directly into the container. Highly dependent on host filesystem permissions; useful for local dev hot-reloads.

---

### **18. What is `kubectl rollout` and how do you undo a bad deployment?**
**Answer:**
Kubernetes keeps a revision history of Deployments.
- View revision history:
  ```bash
  kubectl rollout history deployment/payment-service
  ```
- Roll back to the immediately preceding revision:
  ```bash
  kubectl rollout undo deployment/payment-service
  ```
- Roll back to a specific target revision:
  ```bash
  kubectl rollout undo deployment/payment-service --to-revision=3
  ```

---

### **19. What is a Job vs CronJob in Kubernetes?**
**Answer:**
- **Job:** Creates one or more pods and ensures that a specified number of them successfully terminate with Exit Code 0 (batch processing, data migrations).
- **CronJob:** Runs a Job on a recurring schedule using standard cron format (`spec.schedule: "0 2 * * *"`). Supports concurrency policies (`Allow`, `Forbid`, `Replace`).

---

### **20. What is an Init Container and how does it differ from app containers?**
**Answer:**
An Init Container is a specialized container that runs and must complete to exit code 0 **before** the application containers in the Pod start.
- Runs sequentially (if multiple init containers exist, each runs after the previous completes).
- **Common Use Cases:** Waiting for a database port to become reachable (`nc -z db 5432`), downloading configuration files, or setting kernel `sysctl` parameters.

---

## 🟡 **Intermediate Level (Questions 21–40)**

### **21. What is the Kubernetes Gateway API and why is it superior to Ingress?**
**Answer:**
The **Gateway API** is an open-source standard created by the CNCF SIG-NETWORK to replace the legacy Ingress resource.

**Architectural Advantages:**
- **Role-Oriented Design:**
  - *Infrastructure Provider:* Configures `GatewayClass` (e.g., Envoy, Cilium, AWS VPC Lattice).
  - *Cluster Operator:* Provisions `Gateway` (declares ports, TLS certificates, IP addresses).
  - *Application Developer:* Attaches `HTTPRoute`, `GRPCRoute`, `TCPRoute` to route their specific services without requiring cluster admin rights.
- **Native Advanced Routing:** Built-in support for header-based routing, URL rewriting, traffic splitting (canary weights), and cross-namespace routing without messy vendor-specific annotations.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: api-route
spec:
  parentRefs:
    - name: prod-gateway
  rules:
    - matches:
        - path: { type: PathPrefix, value: /v2 }
      backendRefs:
        - name: api-service-v2
          port: 8080
          weight: 20
        - name: api-service-v1
          port: 8080
          weight: 80
```

---

### **22. What is Karpenter and how does it optimize Kubernetes node provisioning?**
**Answer:**
Karpenter is an open-source, high-performance node autoscaler built for Kubernetes (by AWS/CNCF).

**Comparison with Cluster Autoscaler:**
- **Cluster Autoscaler:** Relies on cloud provider Auto Scaling Groups (ASGs). Scaling is slow (2–5 minutes), instance sizing is rigid, and heterogeneous node mixing is difficult.
- **Karpenter:** Directly talks to cloud EC2/Compute APIs.
  - Observes unschedulable pods and launches right-sized compute nodes in **under 45 seconds**.
  - Automatically mixes Spot and On-Demand instances, Graviton/ARM64 and x86 architectures.
  - Continuously executes **Node Consolidation**: identifies underutilized nodes, spins up a single smaller replacement instance, drains pods, and terminates waste.

---

### **23. What is Cilium and why is eBPF replacing iptables / kube-proxy in modern clusters?**
**Answer:**
Traditional Kubernetes networking (`kube-proxy`) uses Linux `iptables` or `IPVS` to manage Service routing:
- `iptables` is $O(N)$ sequential packet filtering; in clusters with 5,000+ services, packet evaluation causes severe latency and high CPU usage.

**Cilium with eBPF:**
- Uses **eBPF (Extended Berkeley Packet Filter)** bytecode attached directly to socket layers (`sockops`) and network interfaces (XDP / TC).
- Achieves $O(1)$ hash table lookups for Service routing, reducing packet traversal overhead.
- Provides transparent Layer 7 observability (Hubble) and kernel-level network security policies without proxy sidecars.

---

### **24. What are Kubernetes Pod Security Standards (PSS) and Pod Security Admission (PSA)?**
**Answer:**
Pod Security Admission (PSA) replaced the deprecated PodSecurityPolicies (PSP) in Kubernetes 1.25+.

**Three Built-in Levels:**
1. **Privileged:** Unrestricted; pods can run as root, mount host paths, and use host namespaces.
2. **Baseline:** Minimally restrictive; prevents known privilege escalations (blocks host network, host ports, raw devices).
3. **Restricted:** Hardened best-practice; requires rootless execution (`runAsNonRoot: true`), drops all default capabilities except `NET_BIND_SERVICE`, forbids privilege escalation, and restricts volume types.

**Enforcement via Namespace Labels:**
```yaml
metadata:
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/warn: restricted
```

---

### **25. How do you troubleshoot a Pod stuck in `CrashLoopBackOff`? Walk through the exact command sequence.**
**Answer:**
1. **Check Pod Status and Restart Count:**
   ```bash
   kubectl get pods -n <namespace> -o wide
   ```
2. **Inspect Container Exit Code and Reason:**
   ```bash
   kubectl describe pod <pod-name> -n <namespace>
   # Look for Last State: Terminated -> Exit Code & Reason (e.g., Error, OOMKilled, Completed)
   ```
3. **Check Current Logs:**
   ```bash
   kubectl logs <pod-name> -n <namespace> --all-containers
   ```
4. **Check Previous Container Crash Logs (Crucial):**
   ```bash
   kubectl logs <pod-name> -n <namespace> --previous
   ```
5. **Check Node Events:**
   ```bash
   kubectl get events -n <namespace> --sort-by='.metadata.creationTimestamp'
   ```

---

### **26. What are the common Container Exit Codes and what do they indicate?**
**Answer:**
- **Exit Code 0:** Success / Clean execution (normal for completed Jobs/Init containers).
- **Exit Code 1:** Application error / uncaught exception in code.
- **Exit Code 126:** Command invoked cannot execute (permission issue on binary).
- **Exit Code 127:** Command not found (wrong path or missing binary in container image).
- **Exit Code 137 (128 + 9 / SIGKILL):** Container was killed forcibly by OS. Almost always indicates **OOMKilled** (exceeded memory limit) or killed by custom agent.
- **Exit Code 143 (128 + 15 / SIGTERM):** Graceful termination signal sent by Kubernetes during pod scale-down or node drain.

---

### **27. What is `kubectl debug` and how do Ephemeral Containers work?**
**Answer:**
When troubleshooting production pods built with `distroless` or `scratch` base images, there are no shells or diagnostic tools (`curl`, `netstat`, `sh`) inside the container.

**`kubectl debug` Solution:**
Attaches an **Ephemeral Container** containing full diagnostic utilities (e.g., `nicolaka/netshoot`) directly into the running Pod's shared Linux namespaces:
```bash
kubectl debug -it pod/payment-service-8f7d9 -n prod \
  --image=nicolaka/netshoot --target=payment-container
```
This allows inspecting the live container's filesystem, network sockets, and processes without restarting the pod.

---

### **28. What is the difference between Istio Sidecar Architecture and Istio Ambient Mesh?**
**Answer:**
- **Sidecar Architecture (Classic):** Injects an Envoy proxy container into every single Pod.
  - *Drawbacks:* High memory/CPU overhead (every microservice runs an Envoy instance), slow pod startup, application restarts required when updating the mesh proxy.
- **Ambient Mesh (Sidecarless):** Decouples mesh processing into two shared node-level layers:
  1. **ztunnel (Zero Trust Tunnel):** A lightweight per-node daemon handling mTLS and Layer 4 identity.
  2. **Waypoint Proxy:** Optional per-namespace or per-service Envoy instance handling Layer 7 policies (authorization, canary routing) only when needed.
  - *Benefits:* Over 70% reduction in compute overhead, zero pod restarts on upgrades.

---

### **29. What is KEDA (Kubernetes Event-driven Autoscaling) and how does it scale based on message queues?**
**Answer:**
KEDA runs as a custom operator and metrics adapter that scales Kubernetes Deployments or Jobs based on external metrics (AWS SQS, Apache Kafka, RabbitMQ, Redis, PostgreSQL).

**Example ScaledObject for SQS:**
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: sqs-worker-scaler
spec:
  scaleTargetRef:
    name: order-consumer
  minReplicaCount: 0   # Supports scale-to-zero!
  maxReplicaCount: 50
  triggers:
    - type: aws-sqs-queue
      metadata:
        queueURL: https://sqs.us-east-1.amazonaws.com/12345/order-queue
        queueLength: "10"  # 1 pod per 10 pending messages
```

---

### **30. What is Kubernetes Affinity, Anti-Affinity, Taints, and Tolerations?**
**Answer:**
- **Node Affinity:** Directs pods to be scheduled on specific nodes matching labels (`kubernetes.io/arch: arm64`).
- **Pod Anti-Affinity:** Prevents pods of the same service from running on the same node or Availability Zone to ensure high availability across failures.
- **Taints (Node Property):** Repels pods from a node unless they tolerate the taint (e.g., `gpu=true:NoSchedule`).
- **Tolerations (Pod Property):** Allows (but does not force) a pod to schedule on a node with matching taints.

---

### **31. What happens under the hood during a Kubernetes Pod Graceful Termination Lifecycle?**
**Answer:**
1. **Endpoint Removal:** Pod is marked `Terminating` and simultaneously removed from Service Endpoints/EndpointsSlice; kube-proxy/Cilium stops routing new traffic to it.
2. **`preStop` Hook:** Kubernetes executes the `preStop` script (if defined) inside the container (e.g., sleeping 10s to allow in-flight load balancer requests to finish).
3. **`SIGTERM` Signal:** `kubelet` sends `SIGTERM` to the container PID 1, signaling the app to stop accepting new connections and flush buffers.
4. **Termination Grace Period:** Kubernetes waits up to `terminationGracePeriodSeconds` (default: 30s).
5. **`SIGKILL` Signal:** If processes are still running after the timeout, `kubelet` sends `SIGKILL` (Exit Code 137), forcibly terminating the container immediately.

---

### **32. What is a Pod Disruption Budget (PDB) and why is it critical during cluster upgrades?**
**Answer:**
A PDB limits the number of pods of a replicated application that can be simultaneously down during voluntary disruptions (e.g., `kubectl drain` during node OS patching or Karpenter node consolidation).

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
If draining a node would cause the number of available pods to drop below 80%, the drain operation safely blocks until new replacement pods become ready on another node.

---

### **33. What is the difference between StorageClass, PersistentVolume (PV), and PersistentVolumeClaim (PVC)?**
**Answer:**
- **`StorageClass`:** Defines the provisioner (AWS EBS CSI, GCP PD, Ceph) and dynamic volume parameters (IOPS, volume type, retain policy).
- **`PersistentVolume (PV)`:** A piece of actual provisioned storage in the cluster created dynamically by the StorageClass or manually by an admin.
- **`PersistentVolumeClaim (PVC)`:** A request for storage by a user/pod specifying storage size and access mode (`ReadWriteOnce`, `ReadWriteMany`). The PVC binds to an available matching PV.

---

### **34. What is Container Network Interface (CNI) and what are the main CNIs?**
**Answer:**
CNI is a CNCF library defining how third-party network plugins attach network interfaces and configure IP addresses for containers.
- **AWS VPC CNI:** Assigns native VPC secondary IP addresses directly to Pods (high network throughput, native VPC security groups, but can exhaust VPC IP address space).
- **Cilium:** eBPF-based high performance CNI supporting overlay and direct routing, Hubble observability, and transparent encryption (WireGuard/IPsec).
- **Calico:** High-performance CNI widely used for advanced BGP routing and scalable NetworkPolicies.

---

### **35. What is the Kubernetes API Priority and Fairness (APF)?**
**Answer:**
APF protects the `kube-apiserver` from being overloaded by request storms. It categorizes incoming API requests into priority levels and flow schemas (e.g., leader election requests from control plane controllers get top priority, while high-volume batch queries or CI runners get queued and fair-shared).

---

### **36. What is ValidatingAdmissionPolicy with Common Expression Language (CEL) in Kubernetes?**
**Answer:**
Introduced as GA in Kubernetes 1.30+, CEL allows writing declarative in-process admission validation rules directly in Kubernetes manifests without needing external webhook controllers (like OPA or Kyverno).

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingAdmissionPolicy
metadata:
  name: "require-run-as-non-root"
spec:
  failurePolicy: Fail
  matchConstraints:
    resourceRules:
      - apiGroups: [""]
        apiVersions: ["v1"]
        operations: ["CREATE", "UPDATE"]
        resources: ["pods"]
  validations:
    - expression: "object.spec.securityContext.runAsNonRoot == true"
      message: "Pods must define securityContext.runAsNonRoot: true"
```

---

### **37. What is Rootless Docker / Podman and why is it superior for security?**
**Answer:**
Standard Docker runs its daemon as the host `root` user. If an attacker achieves container escape, they immediately obtain full root privileges over the host OS.

**Rootless Container Engines:**
- Docker/Podman executes the daemon and containers inside an unprivileged user namespace.
- Root inside the container maps to a normal, unprivileged UID (e.g., UID 10001) on the host. A container breakout leaves the attacker trapped with no host privileges.

---

### **38. What is Containerd vs Docker vs CRI-O?**
**Answer:**
- **Docker:** High-level platform designed for human developers (includes CLI, image building, desktop UI, swarm).
- **Containerd:** High-performance, lightweight core container runtime daemon originally spun out of Docker. Implements the OCI and Kubernetes CRI specifications.
- **CRI-O:** Minimalist, purpose-built container runtime designed exclusively to serve as the Kubernetes CRI runtime.

---

### **39. What is a Mutating Webhook vs Validating Webhook in Kubernetes?**
**Answer:**
- **Mutating Webhook:** Intercepts API requests *first* and can modify/inject data into the object before it is stored in etcd (e.g., injecting Istio sidecar proxies or setting default security contexts).
- **Validating Webhook:** Intercepts API requests *after* mutation and evaluates compliance. Returns an accept/reject decision to block invalid or insecure resources.

---

### **40. What is OIDC Authentication with Kubernetes API Server?**
**Answer:**
Instead of managing static client certificates or ServiceAccount tokens for humans, `kube-apiserver` integrates with external enterprise Identity Providers (Okta, Keycloak, Azure AD) via OIDC. Users authenticate with their corporate SSO, and short-lived JWT tokens containing user groups are validated by the API server to enforce RBAC.

---

## 🔴 **Advanced & Scenario-Based Level (Questions 41–60)**

### **41. Scenario: CoreDNS in your Kubernetes cluster is maxing out CPU, and pods across the cluster experience 5-second connection timeouts when connecting to internal services. What is the root cause and how do you fix it?**
**Answer:**
**Root Cause: The Linux glibc `ndots:5` Resolution Bottleneck.**
In default `/etc/resolv.conf` generated by Kubernetes:
```
search <namespace>.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```
If an application queries an external domain with fewer than 5 dots (e.g., `api.stripe.com` has 2 dots), glibc appends every search domain first:
1. `api.stripe.com.<namespace>.svc.cluster.local` $\rightarrow$ NXDOMAIN
2. `api.stripe.com.svc.cluster.local` $\rightarrow$ NXDOMAIN
3. `api.stripe.com.cluster.local` $\rightarrow$ NXDOMAIN
4. `api.stripe.com` $\rightarrow$ Resolved!
This creates 4 DNS queries per connection, overwhelming CoreDNS.

**Fixes:**
1. **Deploy NodeLocal DNSCache:** Runs a DaemonSet caching DNS lookups locally on each worker node loopback (`169.254.20.10`), eliminating CoreDNS network hops.
2. **FQDN in Code:** Append a trailing dot in application configs (e.g., `api.stripe.com.` tells glibc it is absolute, bypassing search domains).
3. **Pod-level DNS Config:**
   ```yaml
   spec:
     dnsConfig:
       options:
         - name: ndots
           value: "2"
   ```

---

### **42. Scenario: A high-throughput Java application pod is repeatedly terminated with Exit Code 137 (OOMKilled) even though JVM heap usage is set to 4GB and Pod memory limit is set to 6GB. What is consuming the memory?**
**Answer:**
**Root Cause: Off-Heap / Native Memory Consumption.**
Container memory limits constrain the **entire Linux cgroup**, not just the JVM Heap (`-Xmx4g`).

**Off-Heap Memory Consumers:**
1. **Metaspace:** Class metadata loading (`-XX:MaxMetaspaceSize`).
2. **Thread Stacks:** Each thread reserves memory (e.g., 500 threads $\times$ 1MB `-Xss` = 500MB).
3. **Direct Byte Buffers & JNI:** Netty / gRPC network buffers allocated via `ByteBuffer.allocateDirect()`.
4. **Garbage Collection Overhead:** Memory used by G1/ZGC internal tracking tables.

**Solution:**
- Enable Native Memory Tracking: `-XX:NativeMemoryTracking=summary`.
- Limit Metaspace and direct memory: `-XX:MaxDirectMemorySize=1g`.
- Right-size pod limit to 8GB or tune JVM parameters using `-XX:+UseContainerSupport` and `-XX:MaxRAMPercentage=75.0`.

---

### **43. Scenario: A worker node enters `NotReady` status in production. What internal kubelet and Linux checks do you perform to diagnose?**
**Answer:**
1. **Check Node Conditions:**
   ```bash
   kubectl describe node <node-name>
   # Look for Conditions: MemoryPressure, DiskPressure, PIDPressure, NetworkUnavailable
   ```
2. **SSH into Node and Check Kubelet Service:**
   ```bash
   systemctl status kubelet
   journalctl -u kubelet -n 100 --no-pager
   ```
3. **Check Container Runtime Daemon:**
   ```bash
   systemctl status containerd
   crictl ps
   ```
4. **Check Linux Kernel Logs for OOM or Disk I/O Stalls:**
   ```bash
   dmesg -T | grep -E -i 'oom|hung_task|ext4|xfs'
   df -h /var/lib/containerd
   ```
5. **Check Network Connectivity to API Server:**
   ```bash
   curl -k https://<KUBE_API_SERVER_ENDPOINT>:6443/healthz
   ```

---

### **44. How does Kubernetes CFS (Completely Fair Scheduler) Bandwidth Throttling impact latency-sensitive microservices?**
**Answer:**
When you set a `resources.limits.cpu: "1000m"`, the Linux kernel enforces this via CFS quota in 100ms periods (100ms quota per 100ms period).
- If a multi-threaded application bursts and consumes its 100ms quota in the first 20ms of the period, the kernel **hard-throttles and freezes the process for the remaining 80ms**.
- This creates severe p99 latency spikes even when average CPU utilization across 1 minute appears as only 30%.

**Modern SRE Best Practice:**
- Remove CPU limits on latency-critical workloads while keeping accurate CPU `requests` (or rely on Kubernetes CPU Manager with static policy and integer CPU cores for core pinning).

---

### **45. Scenario: How do you architect a Multi-Tenant Kubernetes cluster ensuring strong isolation between multiple enterprise customers?**
**Answer:**
1. **Virtual Clusters (`vcluster`):** Runs separate, lightweight virtual Kubernetes API servers and control planes inside namespaces of the underlying host cluster.
2. **Network Isolation:** Default-deny all ingress/egress using Cilium/Calico NetworkPolicies.
3. **Compute Sandboxing:** Enforce `runtimeClassName: gVisor` or Kata Containers for untrusted workloads.
4. **Admission Governance:** Kyverno / Gatekeeper enforcing Restricted Pod Security Standards.
5. **Resource Quotas & LimitRanges:** Hard CPU/Memory limits and storage constraints per tenant namespace.

---

### **46. How do you safely upgrade a production Kubernetes cluster across minor versions (e.g., from 1.30 to 1.31) with zero customer downtime?**
**Answer:**
1. **Pre-Upgrade Deprecation Check:** Use tools like `kubent` (Kube-No-Trouble) or Pluto to detect deprecated API versions.
2. **Upgrade Control Plane:** Upgrade `kube-apiserver`, `etcd`, `kube-controller-manager`, and `kube-scheduler` (managed automatically in EKS/GKE).
3. **Upgrade Core Add-ons:** Update CNI (Cilium/AWS VPC CNI), CoreDNS, kube-proxy, and Gateway API CRDs.
4. **Worker Node Rolling Upgrade:**
   - Cordon node: `kubectl cordon <node>` (prevents new pods).
   - Drain node: `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data` (respects PodDisruptionBudgets).
   - Upgrade `kubelet` and `containerd` on the node.
   - Uncordon node: `kubectl uncordon <node>`.
   - Repeat node-by-node or replace ASG/NodePool instances rolling fashion.

---

### **47. What is etcd Quorum, and how do you recover an etcd cluster after losing a majority of master nodes?**
**Answer:**
etcd uses the Raft consensus algorithm. Quorum requires $\lfloor N/2 \rfloor + 1$ healthy members (in a 3-node cluster, 2 nodes must be alive).

**Disaster Recovery (Majority Lost):**
1. Stop etcd service on the remaining surviving node.
2. Create an emergency snapshot:
   ```bash
   etcdctl snapshot save snapshot.db
   ```
3. Restore the snapshot into a new single-node cluster using `--force-new-cluster`:
   ```bash
   etcdctl snapshot restore snapshot.db \
     --name master-1 \
     --initial-cluster master-1=https://10.0.0.1:2380 \
     --initial-advertise-peer-urls https://10.0.0.1:2380
   ```
4. Start etcd with the new cluster configuration and join replacement nodes one by one.

---

### **48. How do you implement End-to-End mTLS and Cryptographic Workload Identity in Kubernetes with SPIFFE/SPIRE?**
**Answer:**
SPIFFE (Secure Production Identity Framework for Everyone) defines a standard SPIFFE ID format (`spiffe://prod.example.com/ns/finance/sa/payment-service`).
- **SPIRE (SPIFFE Runtime Engine):** Runs as a DaemonSet on every node.
- It attest the pod's identity by querying the node's `kubelet` for the pod's UID, namespace, and ServiceAccount.
- Issues short-lived, auto-rotating X.509 SVID certificates directly into the pod via an in-memory Unix domain socket.
- Applications establish zero-trust mTLS without managing certificate files or static private keys.

---

### **49. What are Kubernetes Ephemeral Volumes and Generic Ephemeral Volumes?**
**Answer:**
- **Standard Ephemeral Volumes:** `emptyDir` (lives in node disk or RAM), `configMap`, `secret`, `downwardAPI`.
- **Generic Ephemeral Volumes:** Dynamic PVCs created per pod that share the lifecycle of the Pod (created when pod is scheduled, automatically deleted when pod is deleted). Allows pods to utilize high-speed cloud SSDs (EBS/PD) with full StorageClass features without manually managing independent PVC lifecycles.

---

### **50. Scenario: A Kubernetes Deployment has 5 replicas. You run `kubectl delete pod <pod-name>` on one replica. Walk through the exact internal control loop sequence that happens.**
**Answer:**
1. **API Server:** Receives delete request, records deletion timestamp on the Pod object in `etcd`.
2. **Kubelet:** Observes deletion timestamp, executes graceful shutdown sequence (`preStop` hook $\rightarrow$ `SIGTERM` $\rightarrow$ wait grace period $\rightarrow$ `SIGKILL`), terminates containers, and notifies API server.
3. **ReplicaSet Controller:** Watches the cluster state, detects that actual running pods ($4$) is less than desired replicas ($5$).
4. **Pod Creation:** ReplicaSet controller creates a new Pod definition in `Pending` state in `etcd`.
5. **Kube-Scheduler:** Detects pending pod with no assigned node, filters and scores nodes, and writes the selected node assignment back to the API server (`Binding` object).
6. **Kubelet on New Node:** Watches API server, detects assigned pod, pulls container image, sets up cgroups/namespaces via CRI, configures network via CNI, mounts volumes, and starts the container.
7. **Readiness Probe:** Once readiness probe passes, EndpointSlice controller adds new pod IP to the Service.

---

### **51. What is the difference between iptables mode vs IPVS mode in kube-proxy?**
**Answer:**
- **iptables mode:** Kube-proxy writes sequential packet inspection rules for each Service and Endpoint. As services grow into thousands, traversing linear iptables rules increases latency and CPU overhead on every packet.
- **IPVS (IP Virtual Server) mode:** Operates in Linux kernel transport layer using hash tables ($O(1)$ complexity). Supports advanced load balancing algorithms (least connection, weighted round-robin, locality-based) and scales effortlessly to tens of thousands of services.

---

### **52. What is Kubernetes Sidecar Container Lifecycle feature (KEP-753) in Kubernetes 1.28+?**
**Answer:**
Prior to K8s 1.28, sidecar containers (like Istio proxy, Vault agent, log forwarders) were treated like regular containers with unpredictable startup and shutdown order. This caused apps to fail if they made network calls before the sidecar proxy started, or jobs to hang indefinitely because sidecar containers never exited.

**Native Sidecar Support:**
Declared as an `initContainers` with `restartPolicy: Always`:
```yaml
initContainers:
  - name: vault-agent
    image: vault:1.15
    restartPolicy: Always  # Signals Kubernetes this is a long-running native sidecar!
```
- Starts *before* main application containers and blocks app startup until its startup probe passes.
- Automatically terminates *after* main application containers finish in batch Jobs.

---

### **53. What is the difference between cgroups v1 and cgroups v2 in Kubernetes?**
**Answer:**
- **cgroups v1:** Fragmented controllers (cpu, memory, blkio were independent hierarchies). Memory throttling did not account for page cache writebacks, leading to inaccurate OOM triggering and poor I/O isolation.
- **cgroups v2 (Unified Hierarchy):** Single unified process hierarchy. Enables accurate memory tracking (including page cache and swap), rootless containers, PSI (Pressure Stall Information), and improved CPU bandwidth management.

---

### **54. What is Container Image Layer Squashing and what are its trade-offs?**
**Answer:**
- **Squashing:** Merging all intermediate build layers of a Docker image into a single final filesystem layer.
- **Trade-offs:**
  - *Pros:* Shrinks image size by discarding files created and deleted in intermediate layers; hides sensitive files if improperly created in earlier steps.
  - *Cons:* Destroys Docker layer sharing across multiple images on the same host (if 10 microservices share a 100MB base layer, squashing forces downloading 10 separate full layers).

---

### **55. How do you prevent Node Eviction storms in memory-constrained Kubernetes clusters?**
**Answer:**
1. **Set Accurate Kubelet Eviction Thresholds:**
   Configure `--eviction-hard=memory.available<500Mi,nodefs.available<10%` in `KubeletConfiguration`.
2. **Reserve System Resources (`system-reserved` & `kube-reserved`):**
   Allocate dedicated CPU and RAM for host OS daemons (`sshd`, `systemd`) and Kubernetes daemons (`kubelet`, `containerd`) so application pods cannot exhaust node stability memory.
3. **Use Guaranteed QoS Classes:** Pods with `requests == limits` for both CPU and memory get `Guaranteed` QoS and are the last to be evicted.

---

### **56. What is the Kubernetes Quality of Service (QoS) Class and how is it assigned?**
**Answer:**
Kubernetes assigns QoS classes automatically based on resource definitions to determine eviction priority under node resource pressure:
1. **Guaranteed:** Every container in the pod has `requests == limits` for both CPU and memory. Lowest eviction priority.
2. **Burstable:** At least one container has a memory or CPU request defined, but limits are higher or unset. Evicted when Guaranteed pods need reserved memory.
3. **BestEffort:** No requests or limits specified. Highest eviction priority; terminated first during memory pressure.

---

### **57. What is Horizontal Pod Autoscaler (HPA) v2 with Custom and External Metrics?**
**Answer:**
HPA v2 allows scaling workloads using:
- **Resource Metrics:** Native CPU/Memory utilization percentages.
- **Custom Metrics (Custom Metrics API):** Metrics associated with Kubernetes objects (e.g., HTTP request rate per second from Ingress).
- **External Metrics (External Metrics API):** Metrics from outside the cluster (e.g., Datadog monitor value, GCP Pub/Sub unacknowledged message count, CloudWatch queue depth).

---

### **58. What is a Kubernetes Finalizer and how do you resolve a Namespace stuck in "Terminating"?**
**Answer:**
A **Finalizer** is a pre-delete hook metadata key (`metadata.finalizers`) that tells Kubernetes controllers to perform cleanup tasks (e.g., deleting cloud load balancers or storage volumes) before the resource is purged from etcd.

**Resolving Stuck Namespace:**
1. Identify blocking resources holding finalizers:
   ```bash
   kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found -n <stuck-namespace>
   ```
2. If resources are already deleted and the controller is gone, edit the namespace JSON and remove the finalizers array:
   ```bash
   kubectl get ns <stuck-namespace> -o json | jq '.spec.finalizers = []' | kubectl replace --raw "/api/v1/namespaces/<stuck-namespace>/finalize" -f -
   ```

---

### **59. What is eBPF Tetragon vs Falco for Container Runtime Security?**
**Answer:**
- **Falco:** Parses Linux kernel system calls in userspace or via kernel module/eBPF probe. Compares syscall events against rule sets (e.g., detecting unauthorized shell spawns, sensitive file reads in `/etc/shadow`) and triggers alerts.
- **Tetragon (by Cilium):** Executes security enforcement directly **in-kernel** via eBPF. Can not only detect but **instantly block/kill** malicious processes (e.g., preventing kernel privilege escalation or unauthorized namespace transitions) before the syscall even returns to userspace.

---

### **60. Scenario: How do you migrate 200 stateful and stateless services from an on-prem Kubernetes cluster to AWS EKS with zero user-perceived downtime?**
**Answer:**
1. **Networking Layer:** Establish AWS Direct Connect or Site-to-Site VPN between on-prem and AWS VPC.
2. **Stateful Data Synchronization:**
   - Database: Configure live asynchronous replication (e.g., PostgreSQL streaming replication from on-prem primary to AWS EKS replica).
   - Object Storage: Sync data to S3 via AWS DataSync.
3. **Deploy Workloads via GitOps:** ArgoCD synchronizes identical application deployments and config onto the EKS cluster.
4. **Traffic Shifting via Global DNS (Route53 / Cloudflare):**
   - Use Weighted DNS routing: Route 5% traffic to EKS $\rightarrow$ monitor error rates and latency $\rightarrow$ increase to 25%, 50%, 100%.
5. **Database Cutover:** Promote EKS database replica to Primary during a low-traffic 30-second maintenance window.
6. **Decommission:** Drain and shut down on-prem workloads after validating stability.
