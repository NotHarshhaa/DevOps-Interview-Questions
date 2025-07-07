# **📌 Containers (Docker & Kubernetes) - 60 Questions**

- **Beginner (1-20)**
- **Intermediate (21-40)**
- **Advanced (41-60)**  

---

## **🚀 Beginner-Level Docker & Kubernetes Questions (1-20)**  

### **Docker Basics**  

### **1. What is Docker, and why is it used?**  

**Answer:**  
Docker is a **containerization platform** that allows developers to package applications along with their dependencies into a single unit called a **container**.  

- **Why use Docker?**  
  ✅ Ensures **consistent environments** across different machines.  
  ✅ **Lightweight & faster** than virtual machines.  
  ✅ **Easy scaling** of applications in microservices architectures.  

---

### **2. What is the difference between Docker and a Virtual Machine (VM)?**  

**Answer:**  

| Feature | Docker | Virtual Machine |
|---------|--------|----------------|
| **Isolation** | Uses **containers** to isolate apps | Uses **hypervisor** to run separate OS instances |
| **Performance** | **Faster, lightweight** | **Slower, resource-intensive** |
| **Startup Time** | **Milliseconds** | **Minutes** |
| **Use Case** | Ideal for **microservices** | Best for **full OS emulation** |

---

### **3. What is a Docker image?**  

**Answer:**  
A **Docker image** is a **read-only template** containing everything needed to run an application, including:  

- Source code  
- Libraries & dependencies  
- Configuration files  

A container is created from a **Docker image** using the `docker run` command.  

---

### **4. What is a Docker container?**  

**Answer:**  
A **Docker container** is a **running instance of a Docker image**. It is:  
✅ **Lightweight** (shares OS kernel)  
✅ **Isolated** (has its own filesystem, network, and process space)  
✅ **Portable** (can run on any system with Docker installed)  

---

### **5. How do you create and run a Docker container?**  

**Answer:**  
To run a container from an image:  

```sh
docker run -d --name myapp nginx
```

- `-d`: Run in **detached mode** (background).  
- `--name myapp`: Name the container `myapp`.  
- `nginx`: Use the **nginx image**.  

---

### **6. What is the purpose of the Dockerfile?**  

**Answer:**  
A **Dockerfile** is a script that contains **instructions to build a Docker image**.  
Example `Dockerfile`:  

```Dockerfile
FROM node:16
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "app.js"]
```

- `FROM`: Base image.  
- `WORKDIR`: Set working directory.  
- `COPY`: Copy files.  
- `RUN`: Execute commands (install dependencies).  
- `CMD`: Define the default command to run.  

---

### **7. What are Docker volumes?**  

**Answer:**  
Docker **volumes** store persistent data outside a container's filesystem.  

- **Types:**  
  - **Anonymous Volumes**: `docker run -v /data nginx`  
  - **Named Volumes**: `docker volume create mydata`  
  - **Bind Mounts**: `docker run -v /host/path:/container/path nginx`  

---

### **8. How do you list running Docker containers?**  

**Answer:**  
Use the command:  

```sh
docker ps
```

To list **all containers** (including stopped ones):  

```sh
docker ps -a
```

---

### **9. What is Docker Compose?**  

**Answer:**  
Docker Compose is a tool for **defining and running multi-container applications**.  

- Example `docker-compose.yml`:  

  ```yaml
  version: "3"
  services:
    web:
      image: nginx
      ports:
        - "80:80"
    db:
      image: mysql
      environment:
        MYSQL_ROOT_PASSWORD: root
  ```

- Start with: `docker-compose up -d`  
- Stop with: `docker-compose down`  

---

### **10. What is the difference between CMD and ENTRYPOINT in Docker?**  

**Answer:**  

| Feature | CMD | ENTRYPOINT |
|---------|-----|-----------|
| **Purpose** | Default command | Fixed executable command |
| **Overridable?** | Yes | No (unless `--entrypoint` is used) |
| **Example** | `CMD ["python", "app.py"]` | `ENTRYPOINT ["nginx", "-g", "daemon off;"]` |

---

## **Kubernetes Basics**  

### **11. What is Kubernetes?**  

**Answer:**  
Kubernetes (K8s) is an **orchestration platform** for managing containerized applications.  

- **Features:**  
  ✅ **Automated scaling**  
  ✅ **Self-healing** (restarts failed containers)  
  ✅ **Load balancing**  
  ✅ **Rolling updates**  

---

### **12. What is a Kubernetes Pod?**  

**Answer:**  
A **Pod** is the smallest unit in Kubernetes. It **groups one or more containers** that share the same network and storage.  

---

### **13. What is a Kubernetes Deployment?**  

**Answer:**  
A **Deployment** manages Pod creation and updates.  
Example YAML:  

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: app
          image: nginx
```

- `replicas: 3` → Runs **3 instances**.  
- `matchLabels` → Ensures the correct Pods are managed.  

---

### **14. What is a Kubernetes Service?**  

**Answer:**  
A **Service** exposes a set of Pods over a network.  

- **Types:**  
  - **ClusterIP** (default)  
  - **NodePort** (exposes on a fixed port)  
  - **LoadBalancer** (uses cloud provider's load balancer)  

Example YAML:  

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30007
```

---

### **15. What is the purpose of Kubernetes ConfigMaps and Secrets?**  

**Answer:**  

- **ConfigMaps** store non-sensitive configuration data.  
- **Secrets** store **sensitive** data (passwords, API keys).  

Example Secret:  

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
data:
  password: bXlwYXNzd29yZA==
```

---

### **16. What is a Kubernetes Namespace?**  

**Answer:**  
Namespaces **logically separate resources** within a cluster.  

```sh
kubectl create namespace dev
kubectl get namespaces
```

---

### **17. What is a StatefulSet in Kubernetes?**  

**Answer:**  
A **StatefulSet** is used for **stateful applications** like databases. Unlike Deployments, it maintains:  
✅ **Stable pod identity**  
✅ **Persistent storage**  

---

### **18. How do you scale a Deployment in Kubernetes?**  

**Answer:**  
Manually scale using:  

```sh
kubectl scale deployment my-app --replicas=5
```

---

### **19. What is a DaemonSet?**  

**Answer:**  
A **DaemonSet** ensures that **one Pod runs on every node** (e.g., logging agents, monitoring).  

---

### **20. How do you update a Kubernetes Deployment?**  

**Answer:**  
Update the image and apply changes:  

```sh
kubectl set image deployment/my-app my-container=nginx:latest
```

---

## **🚀 Intermediate-Level Docker & Kubernetes Questions (21-40)**  

### **Docker Intermediate Questions**  

### **21. What is the difference between Docker ADD and COPY?**  

**Answer:**  

| Feature | ADD | COPY |
|---------|----|------|
| **Function** | Copies files & extracts compressed files | Copies files only |
| **Supports URLs?** | Yes | No |
| **Best Practice** | Use for archives (`.tar.gz`) | Use for simple file copies |

Example:  

```Dockerfile
COPY config.json /app/config.json
ADD myapp.tar.gz /app/
```

---

### **22. How do you optimize Docker images?**  

**Answer:**  

- Use **smaller base images** (e.g., `alpine` instead of `ubuntu`).  
- **Multi-stage builds** to reduce image size:  

  ```Dockerfile
  FROM node:16 AS build
  WORKDIR /app
  COPY . .
  RUN npm install && npm run build

  FROM nginx:alpine
  COPY --from=build /app/dist /usr/share/nginx/html
  ```

- Use `.dockerignore` to avoid unnecessary files.  

---

### **23. What is the difference between Docker ENTRYPOINT and CMD?**  

**Answer:**  

- `ENTRYPOINT` is **not overridden by command-line arguments**, while `CMD` can be.  
- Best practice: Use `ENTRYPOINT` for fixed commands.  

Example:  

```Dockerfile
ENTRYPOINT ["nginx", "-g", "daemon off;"]
CMD ["-p", "80"]
```

---

### **24. How do you debug a running Docker container?**  

**Answer:**  

- **Get container logs:** `docker logs my-container`  
- **Attach to a running container:** `docker exec -it my-container /bin/sh`  
- **Inspect container details:** `docker inspect my-container`  

---

### **25. What is a Docker Multi-Stage Build?**  

**Answer:**  
A **multi-stage build** reduces image size by using multiple `FROM` statements.  

```Dockerfile
FROM golang:1.17 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

FROM alpine
COPY --from=builder /app/myapp /myapp
ENTRYPOINT ["/myapp"]
```

The final image **only contains the built binary**.

---

### **26. How does Docker handle networking?**  

**Answer:**  

- **Bridge network (default):** Containers communicate via virtual network.  
- **Host network:** Container shares the host’s networking stack.  
- **Overlay network:** Used in **Docker Swarm** for multi-host networking.  

Example:  

```sh
docker network create mynetwork
docker run --network=mynetwork nginx
```

---

### **27. What is the difference between Docker Swarm and Kubernetes?**  

**Answer:**  

| Feature | Docker Swarm | Kubernetes |
|---------|-------------|------------|
| **Orchestration** | Lightweight, built into Docker | Advanced, feature-rich |
| **Scaling** | Manual | Auto-scaling |
| **Service Discovery** | Built-in | Needs external setup (DNS, Ingress) |

---

### **28. How do you remove unused Docker images and containers?**  

**Answer:**  

```sh
docker system prune -a
```

This removes **stopped containers, unused networks, and dangling images**.

---

### **29. What is Docker BuildKit?**  

**Answer:**  
Docker **BuildKit** improves build speed and caching.  
Enable it with:  

```sh
DOCKER_BUILDKIT=1 docker build .
```

Benefits:  
✅ **Faster builds**  
✅ **Parallel execution**  
✅ **Improved caching**  

---

### **30. How do you limit container resource usage?**  

**Answer:**  
Use `--memory` and `--cpus`:  

```sh
docker run --memory=512m --cpus=1 nginx
```

This limits memory to **512MB** and CPU usage to **1 core**.

---

## **Kubernetes Intermediate Questions**  

### **31. How does Kubernetes handle high availability?**  

**Answer:**  

- Uses **multiple master nodes** to avoid single points of failure.  
- Deployments use **replica sets** to keep applications running.  
- **Load balancing & failover mechanisms** ensure availability.  

---

### **32. What is the role of kubelet in Kubernetes?**  

**Answer:**  
Kubelet runs on each node and:  
✅ **Communicates with the master node**  
✅ **Ensures containers are running**  
✅ **Monitors container health**  

---

### **33. How do you check logs of a running Pod in Kubernetes?**  

**Answer:**  

```sh
kubectl logs my-pod
kubectl logs -f my-pod  # Stream logs in real-time
```

---

### **34. What are Kubernetes Labels and Selectors?**  

**Answer:**  
Labels **identify** resources, while selectors **filter resources**.  
Example:  

```yaml
metadata:
  labels:
    app: my-app
```

To filter pods by label:  

```sh
kubectl get pods -l app=my-app
```

---

### **35. What is a Kubernetes Ingress?**  

**Answer:**  
An **Ingress** manages external access to services.  
Example:  

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - host: myapp.com
      http:
        paths:
          - path: /
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

Use **Ingress controllers (NGINX, Traefik)** to manage Ingress resources.

---

### **36. What is the difference between Horizontal Pod Autoscaler (HPA) and Vertical Pod Autoscaler (VPA)?**  

**Answer:**  

| Feature | HPA | VPA |
|---------|----|----|
| **Scaling Type** | Adds/removes pods | Adjusts CPU/memory of existing pods |
| **Use Case** | High traffic apps | Resource optimization |

Example of **HPA**:  

```sh
kubectl autoscale deployment my-app --cpu-percent=50 --min=2 --max=10
```

---

### **37. What is a Kubernetes Persistent Volume (PV) and Persistent Volume Claim (PVC)?**  

**Answer:**  
A **Persistent Volume (PV)** is a storage resource, and a **Persistent Volume Claim (PVC)** requests storage.  
Example:  

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

### **38. How do you upgrade a running application in Kubernetes?**  

**Answer:**  
Modify the image and apply the deployment:  

```sh
kubectl set image deployment/my-app my-container=nginx:1.20
kubectl rollout status deployment my-app
```

---

### **39. What is a Kubernetes Job and CronJob?**  

**Answer:**  

- **Job**: Runs **once** and exits.  
- **CronJob**: Runs **on a schedule** (like a Linux cron).  

Example:  

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "0 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: hello
              image: busybox
              command: ["echo", "Hello from Kubernetes"]
          restartPolicy: OnFailure
```

---

### **40. How do you debug Kubernetes pods that are stuck in "CrashLoopBackOff"?**  

**Answer:**  

1. **Check pod logs:**  

   ```sh
   kubectl logs my-pod
   ```

2. **Describe the pod for errors:**  

   ```sh
   kubectl describe pod my-pod
   ```

3. **Exec into the container:**  

   ```sh
   kubectl exec -it my-pod -- /bin/sh
   ```

---

## **🚀 Advanced-Level Docker & Kubernetes Questions (41-60)**  

### **Docker Advanced Questions**  

### **41. How do Docker namespaces and cgroups work together to provide container isolation?**

Namespaces isolate the container’s view of the system (process IDs, network interfaces, mount points, etc.), making each container appear as a separate system. Cgroups (control groups) limit and prioritize resource usage (CPU, memory, I/O) for containers. Together, namespaces provide *security and separation*, while cgroups enforce *resource constraints*, enabling lightweight, secure containers without full virtualization.

---

### **42. Explain the difference between Docker Volumes, Bind Mounts, and tmpfs mounts. When would you use each?**

- **Docker Volumes:** Managed by Docker, stored in Docker’s storage area, best for persistent data that needs to survive container restarts and be shared between containers.
- **Bind Mounts:** Mount a host directory/file into a container, useful for development when you want live code changes reflected immediately.
- **tmpfs mounts:** Store data in the host’s memory only, ephemeral and fast, used for sensitive data or temporary files that should not persist.

---

### **43. What are the advantages of Docker BuildKit, and how does it improve the Docker build process?**

BuildKit improves build speed with parallel builds, better caching, and efficient layer reuse. It supports advanced features like build secrets, SSH forwarding, and inline cache export/import. BuildKit also produces smaller images by allowing multi-stage builds with better control and reduces build context size.

---

### **44. How do you secure a Docker container in production?**

Key practices include:

- Use minimal base images to reduce attack surface.
- Run containers with least privileges (non-root user).
- Use Docker Content Trust to verify image signatures.
- Limit container resource usage with cgroups.
- Use seccomp, AppArmor, or SELinux profiles to restrict syscalls.
- Regularly scan images for vulnerabilities.
- Isolate containers using user namespaces and network policies.

---

### **45. What are Docker namespaces and cgroups? How do they contribute to containerization?**  

**Answer:**  

- **Namespaces** isolate resources (PID, network, mount points, etc.) for each container.  
- **Cgroups (Control Groups)** limit CPU, memory, and disk usage.  
- Together, they **ensure process isolation and resource allocation**.  

Example:  

```sh
cat /proc/self/cgroup
```

---

### **46. What is the difference between Docker Volumes, Bind Mounts, and tmpfs?**  

**Answer:**  

| Type | Persistent? | Use Case |
|------|------------|----------|
| **Volumes** | Yes | Best for data persistence |
| **Bind Mounts** | Yes | Direct host file access |
| **tmpfs** | No | In-memory storage for performance |

Example (Volume):  

```sh
docker run -v myvolume:/data nginx
```

---

### **47. What are Docker BuildKit advantages?**  

**Answer:**  

- **Parallel execution** speeds up builds.  
- **Efficient caching** reduces rebuild time.  
- **Security improvements** via secret mounts.  

Enable BuildKit:  

```sh
DOCKER_BUILDKIT=1 docker build .
```

---

### **48. How do you secure a Docker container?**  

**Answer:**  

- **Use minimal base images** (e.g., `alpine`).  
- **Run as non-root user**.  
- **Limit container capabilities** (`--cap-drop=ALL`).  
- **Use read-only filesystems** (`--read-only`).  

Example:  

```sh
docker run --user 1001 --read-only nginx
```

---

### **49. How do multi-stage builds improve security in Docker?**  

**Answer:**  

- Keeps **sensitive files out of the final image**.  
- Reduces **attack surface** by discarding unnecessary dependencies.  

Example:  

```Dockerfile
FROM golang AS build
COPY . .  
RUN go build -o myapp

FROM alpine
COPY --from=build /myapp /myapp
ENTRYPOINT ["/myapp"]
```

---

### **50. What are immutable infrastructure principles, and how do they apply to Docker?**  

**Answer:**  

- Containers should be **replaced, not modified**.  
- Use **image versioning** instead of patching live containers.  
- Example: Deploy **new image versions** instead of updating running containers.  

---

### **51. How does Docker Content Trust (DCT) improve security?**  

**Answer:**  

- **Ensures image integrity** with digital signatures.  
- Enable DCT:  

  ```sh
  export DOCKER_CONTENT_TRUST=1
  ```

---

### **52. How do you troubleshoot a Docker daemon issue?**  

**Answer:**  

- **Check logs:** `journalctl -u docker.service`  
- **Restart service:** `systemctl restart docker`  
- **Debug mode:** `dockerd --debug`

---

### **53. What is the difference between Docker stack and Docker compose?**  

**Answer:**  

- **Docker Compose** is for single-host deployments.  
- **Docker Stack** is for multi-node Swarm clusters.  

---

### **54. How do you handle container networking in a multi-host Docker Swarm?**  

**Answer:**  

- **Overlay networks** span multiple hosts.  
- Example:  

  ```sh
  docker network create -d overlay mynetwork
  ```

---

## **Kubernetes Advanced Questions**

### **55. What is a PodDisruptionBudget (PDB) and how does it help maintain high availability?**

A PDB defines the minimum number or percentage of pods that must remain available during voluntary disruptions (e.g., node drain, rolling updates). It prevents Kubernetes from evicting too many pods at once, ensuring application availability during maintenance or upgrades.

---

### **56. How do Kubernetes Admission Controllers enhance cluster security?**

Admission Controllers intercept requests to the Kubernetes API server after authentication and authorization but before persistence. They can enforce policies, mutate requests, or reject unsafe operations. Examples include PodSecurityPolicy, ResourceQuota, and NetworkPolicy controllers, which enforce security, resource limits, and network segmentation.

---

### **57. Explain how Kubernetes Horizontal Pod Autoscaler (HPA) works internally.**

HPA monitors metrics like CPU or custom metrics via the Metrics API. It compares current usage against target thresholds and adjusts the number of pod replicas accordingly by updating the Deployment or ReplicaSet spec. It uses a control loop running every 15 seconds by default to maintain desired load levels.

---

### **58. How does Kubernetes handle stateful applications?**  

**Answer:**  

- Uses **StatefulSets** instead of Deployments.  
- Provides **stable network identities and persistent storage**.  

Example:  

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"
  replicas: 3
```

---

### **59. What are PodDisruptionBudgets (PDBs)?**  

**Answer:**  

- Ensures **minimum availability** during voluntary disruptions.  
- Example:  

  ```yaml
  apiVersion: policy/v1
  kind: PodDisruptionBudget
  metadata:
    name: my-pdb
  spec:
    minAvailable: 2
    selector:
      matchLabels:
        app: my-app
  ```

---

### **60. How do you secure Kubernetes Secrets?**  

**Answer:**  

- Use **encryption at rest**.  
- Store secrets in **external vaults** (e.g., HashiCorp Vault).  
- Example:  

  ```sh
  kubectl create secret generic db-secret --from-literal=password=mysecurepassword
  ```

---

### **61. What are Kubernetes Admission Controllers?**  

**Answer:**  

- They **intercept API requests** before they reach the cluster.  
- Example: `PodSecurityPolicies`, `ValidatingWebhookConfiguration`.

---

### **62. How does Kubernetes handle node failures?**  

**Answer:**  

- **Kubelet marks node as NotReady**.  
- **Pods are rescheduled** onto healthy nodes.  
- **Node auto-repair** triggers in cloud-managed clusters.  

---

### **63. What is a Kubernetes Mutating Webhook?**  

**Answer:**  

- **Modifies requests dynamically** before they reach the cluster.  
- Example: Injecting sidecars into Pods.  

---

### **64. How do you debug networking issues in Kubernetes?**  

**Answer:**  

- Check **Pod-to-Pod connectivity**:  

  ```sh
  kubectl exec -it pod1 -- ping pod2
  ```

- Inspect **network policies**:  

  ```sh
  kubectl get networkpolicy
  ```

- Validate **DNS resolution**:  

  ```sh
  kubectl exec -it pod -- nslookup my-service
  ```

---

### **65. How does Kubernetes Horizontal Pod Autoscaler (HPA) work internally?**  

**Answer:**  

- Uses **metrics API** (CPU/memory usage).  
- Adjusts **replica count dynamically**.  
- Example:  

  ```sh
  kubectl autoscale deployment my-app --cpu-percent=50 --min=2 --max=10
  ```

---

### **66. How do you implement multi-tenancy in Kubernetes?**  

**Answer:**  

- Use **Namespaces** to isolate workloads.  
- Apply **RBAC (Role-Based Access Control)**.  
- Example:  

  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    namespace: team-a
    name: team-a-role
  rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "list", "watch"]
  ```

---

### **67. What is Kubernetes Cluster Federation?**  

**Answer:**  

- Manages **multiple clusters** as a **single entity**.  
- Benefits: **Cross-region high availability, policy consistency**.  
- Example tool: `kubefed`  

---

## **📢 Contribute & Stay Updated**  

💡 **Want to contribute?**  
We **welcome contributions!** If you have insights, new tools, or improvements, feel free to submit a **pull request**.  

📌 **How to Contribute?**

- Read the **[CONTRIBUTING.md](https://github.com/NotHarshhaa/DevOps-Interview-Questions/blob/master/CONTRIBUTING.md)** guide.  
- Fix errors, add missing topics, or suggest improvements.  
- Submit a **pull request** with your updates.  

📢 **Stay Updated:**  
⭐ **Star the repository** to get notified about new updates and additions.  
💬 **Join discussions** in **[GitHub Issues](https://github.com/NotHarshhaa/DevOps-Interview-Questions/issues)** to suggest improvements.  

---

## **🌍 Community & Support**  

🔗 **GitHub:** [@NotHarshhaa](https://github.com/NotHarshhaa)  
📝 **Blog:** [ProDevOpsGuy](https://blog.prodevopsguy.xyz)  
💬 **Telegram Community:** [Join Here](https://t.me/prodevopsguy)  

![Follow Me](https://imgur.com/2j7GSPs.png)
