# ☸️ Kubernetes Interview Questions & Answers
### Complete Guide for DevOps Engineers

---

## 📋 Table of Contents

1. [Core Concepts & Architecture](#1-core-concepts--architecture)
2. [Workloads – Pods, Deployments, StatefulSets](#2-workloads--pods-deployments-statefulsets)
3. [Services & Networking](#3-services--networking)
4. [Storage – PV, PVC, StorageClass](#4-storage--pv-pvc-storageclass)
5. [Configuration – ConfigMaps & Secrets](#5-configuration--configmaps--secrets)
6. [Scheduling & Resource Management](#6-scheduling--resource-management)
7. [RBAC & Security](#7-rbac--security)
8. [Helm & Package Management](#8-helm--package-management)
9. [Monitoring, Logging & Observability](#9-monitoring-logging--observability)
10. [CI/CD & GitOps](#10-cicd--gitops)
11. [Cluster Maintenance & Upgrades](#11-cluster-maintenance--upgrades)
12. [🔥 Scenario-Based Questions](#12--scenario-based-questions)
13. [⚡ Quick-Fire Questions](#13--quick-fire-questions)

---

## 1. Core Concepts & Architecture

### Q1. What is Kubernetes and why is it used?
**Answer:**  
Kubernetes (K8s) is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

**Key reasons to use it:**
- **Auto-scaling** – scale pods up/down based on load
- **Self-healing** – restarts failed containers, replaces unresponsive nodes
- **Service discovery & load balancing** – automatic DNS + traffic distribution
- **Rolling updates & rollbacks** – zero-downtime deployments
- **Declarative configuration** – desired-state management via YAML/JSON

---

### Q2. Explain the Kubernetes architecture.
**Answer:**

**Control Plane (Master Node):**
| Component | Role |
|---|---|
| `kube-apiserver` | Front door of the cluster; handles all REST API calls |
| `etcd` | Distributed key-value store; stores all cluster state |
| `kube-scheduler` | Assigns pods to nodes based on resource availability and constraints |
| `kube-controller-manager` | Runs controllers (Node, Replication, Endpoint, etc.) |
| `cloud-controller-manager` | Integrates with cloud provider APIs |

**Worker Node:**
| Component | Role |
|---|---|
| `kubelet` | Agent on each node; ensures containers run as specified |
| `kube-proxy` | Manages iptables/IPVS rules for service networking |
| `Container Runtime` | Runs containers (containerd, CRI-O, Docker) |

---

### Q3. What is etcd and what happens if it goes down?
**Answer:**  
`etcd` is a distributed, consistent key-value store that holds the entire cluster state (pods, services, configs, secrets, etc.).

**If etcd goes down:**
- No new API requests can be processed (reads/writes fail)
- Existing running workloads **continue to run** (kubelet maintains them)
- No new scheduling, scaling, or deployments possible
- Recovery requires restoring from an etcd backup

**Best practice:** Run etcd in an odd-numbered cluster (3, 5, or 7 nodes) for high availability.

---

### Q4. What is the difference between a Node and a Pod?
**Answer:**

| Aspect | Node | Pod |
|---|---|---|
| Definition | A physical or virtual machine in the cluster | Smallest deployable unit in Kubernetes |
| Contains | Kubelet, kube-proxy, container runtime | One or more containers |
| Lifecycle | Long-lived infrastructure | Ephemeral; can be restarted |
| IP Address | Node IP (stable) | Pod IP (changes on restart) |

---

### Q5. What is a Namespace and when should you use multiple namespaces?
**Answer:**  
A Namespace is a virtual cluster within a Kubernetes cluster. It provides resource isolation and scope.

**Use multiple namespaces when:**
- Separating **environments** (dev, staging, prod) in the same cluster
- Multi-team isolation — each team gets their own namespace
- Applying different **RBAC policies** per namespace
- Setting separate **resource quotas** per team/project

**Default namespaces:**
- `default` – resources without a namespace
- `kube-system` – Kubernetes system components
- `kube-public` – publicly readable resources
- `kube-node-lease` – node heartbeat leases

---

## 2. Workloads – Pods, Deployments, StatefulSets

### Q6. What is the difference between a Deployment and a StatefulSet?
**Answer:**

| Feature | Deployment | StatefulSet |
|---|---|---|
| Pod identity | Pods are interchangeable | Each pod has a stable, unique identity |
| Pod name | Random suffix (e.g., `app-xyz12`) | Ordered suffix (e.g., `app-0`, `app-1`) |
| Storage | Shared or ephemeral | Each pod gets its own PVC |
| Scaling order | Random | Sequential (ordered) |
| Use case | Stateless apps (web servers, APIs) | Stateful apps (DBs, Kafka, Zookeeper) |
| DNS | Single service DNS | Stable DNS per pod (`app-0.svc`) |

---

### Q7. What is a DaemonSet? Give a use case.
**Answer:**  
A DaemonSet ensures that **one pod runs on every node** (or a subset of nodes) in the cluster. When a new node is added, the DaemonSet automatically schedules a pod on it.

**Common use cases:**
- Log collectors (Fluentd, Filebeat)
- Node monitoring agents (Prometheus Node Exporter)
- Network plugins (Calico, Weave)
- Security agents (Falco, Datadog)

---

### Q8. What is a ReplicaSet vs. a Deployment?
**Answer:**  
- A **ReplicaSet** ensures a specified number of pod replicas are running at all times.
- A **Deployment** manages ReplicaSets and adds capabilities like rolling updates, rollbacks, and revision history.

> In practice, you almost always use a **Deployment** — it creates and manages ReplicaSets automatically.

---

### Q9. Explain Pod lifecycle states.
**Answer:**

| Phase | Description |
|---|---|
| `Pending` | Pod accepted but not yet scheduled or containers not started |
| `Running` | Pod bound to a node; at least one container is running |
| `Succeeded` | All containers terminated successfully (exit 0) |
| `Failed` | All containers terminated; at least one failed (non-zero exit) |
| `Unknown` | Pod state cannot be determined (usually a node communication issue) |

---

### Q10. What are Init Containers?
**Answer:**  
Init containers run **before** the main application containers start. They must complete successfully before the next init container (or the app container) begins.

**Use cases:**
- Wait for a database to be ready before the app starts
- Download configuration files or secrets
- Set up file permissions on shared volumes
- Run database migrations

```yaml
initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'until nc -z db-service 5432; do sleep 2; done']
```

---

### Q11. What is a CronJob in Kubernetes?
**Answer:**  
A CronJob creates Jobs on a time-based schedule (like Unix cron). It is used for periodic tasks.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "0 2 * * *"  # Every day at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:latest
          restartPolicy: OnFailure
```

---

## 3. Services & Networking

### Q12. What are the different types of Kubernetes Services?
**Answer:**

| Type | Description | Access Scope |
|---|---|---|
| `ClusterIP` | Default; exposes service on an internal IP | Within cluster only |
| `NodePort` | Exposes service on each node's IP at a static port (30000–32767) | External via `<NodeIP>:<NodePort>` |
| `LoadBalancer` | Provisions an external cloud load balancer | External via LB IP |
| `ExternalName` | Maps service to an external DNS name | DNS-based external |

---

### Q13. What is an Ingress and how does it differ from a Service?
**Answer:**

| Aspect | Service | Ingress |
|---|---|---|
| OSI Layer | Layer 4 (TCP/UDP) | Layer 7 (HTTP/HTTPS) |
| Routing | Port-based | Path-based / Host-based |
| TLS termination | No (requires additional config) | Yes (native SSL/TLS) |
| Requires | Nothing extra | An Ingress Controller (Nginx, Traefik, etc.) |

**Ingress example:**
```yaml
rules:
  - host: api.example.com
    http:
      paths:
        - path: /v1
          backend:
            service:
              name: api-v1
              port:
                number: 80
```

---

### Q14. What is a NetworkPolicy?
**Answer:**  
A NetworkPolicy is a Kubernetes resource that controls **which pods can communicate with each other** and with external endpoints, acting as a firewall at the pod level.

By default, all pods can communicate with all other pods. NetworkPolicies restrict this.

```yaml
# Allow only pods with label "app: frontend" to reach "app: backend" on port 8080
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - port: 8080
```

---

### Q15. What is DNS resolution in Kubernetes?
**Answer:**  
Kubernetes uses CoreDNS for internal DNS. Every service gets a DNS name in the format:

```
<service-name>.<namespace>.svc.cluster.local
```

- Same namespace: `my-service`
- Cross-namespace: `my-service.other-namespace`
- Full FQDN: `my-service.other-namespace.svc.cluster.local`

Pods also get DNS records: `<pod-ip-dashed>.<namespace>.pod.cluster.local`

---

## 4. Storage – PV, PVC, StorageClass

### Q16. Explain PersistentVolume (PV), PersistentVolumeClaim (PVC), and StorageClass.
**Answer:**

| Component | Role |
|---|---|
| **PersistentVolume (PV)** | A piece of storage provisioned by an admin or dynamically. Cluster-level resource. |
| **PersistentVolumeClaim (PVC)** | A request for storage by a user/pod. Namespace-level. |
| **StorageClass** | Defines the "class" of storage (e.g., SSD, HDD) and enables dynamic provisioning. |

**Flow:**  
`Pod` → requests → `PVC` → binds to → `PV` → backed by → `Physical Storage`

---

### Q17. What are the access modes for PVs?
**Answer:**

| Mode | Short | Description |
|---|---|---|
| `ReadWriteOnce` | RWO | Mounted as read-write by a single node |
| `ReadOnlyMany` | ROX | Mounted as read-only by many nodes |
| `ReadWriteMany` | RWX | Mounted as read-write by many nodes |
| `ReadWriteOncePod` | RWOP | Read-write by a single pod (K8s 1.22+) |

---

### Q18. What is the Reclaim Policy for PVs?
**Answer:**

| Policy | Behavior |
|---|---|
| `Retain` | PV is kept after PVC is deleted; manual cleanup required |
| `Delete` | PV and underlying storage are deleted with the PVC |
| `Recycle` (deprecated) | Basic scrub (`rm -rf`) and made available again |

---

## 5. Configuration – ConfigMaps & Secrets

### Q19. What is the difference between ConfigMap and Secret?
**Answer:**

| Aspect | ConfigMap | Secret |
|---|---|---|
| Purpose | Non-sensitive configuration data | Sensitive data (passwords, tokens, keys) |
| Storage | Plain text in etcd | Base64-encoded in etcd (optionally encrypted) |
| Size limit | 1 MB | 1 MB |
| Usage | Environment variables, volumes, command args | Same, but with restricted access via RBAC |

> **Note:** Secrets are base64-encoded, not encrypted by default. Use **etcd encryption at rest** + **external secret managers** (Vault, AWS Secrets Manager) for true security.

---

### Q20. How do you inject configuration into a Pod?
**Answer:**

**3 ways to use ConfigMaps/Secrets:**

```yaml
# 1. Environment variable
env:
  - name: DB_HOST
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: database_host

# 2. Entire ConfigMap as environment variables
envFrom:
  - configMapRef:
      name: app-config

# 3. Mounted as a volume (file)
volumes:
  - name: config-vol
    configMap:
      name: app-config
volumeMounts:
  - name: config-vol
    mountPath: /etc/config
```

---

## 6. Scheduling & Resource Management

### Q21. How does the Kubernetes scheduler work?
**Answer:**  
The scheduler selects a node for a newly created pod in two phases:

1. **Filtering (Predicates):** Eliminates nodes that don't meet hard requirements (resource requests, node selectors, taints/tolerations, affinity rules).
2. **Scoring (Priorities):** Ranks the remaining nodes using scoring functions (least-requested, spread, image locality). The node with the highest score wins.

---

### Q22. What are Resource Requests and Limits?
**Answer:**

| Field | Description |
|---|---|
| `requests` | Minimum guaranteed resources for scheduling decisions |
| `limits` | Maximum resources the container can use |

```yaml
resources:
  requests:
    cpu: "250m"      # 0.25 CPU cores
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

- **CPU** is compressible — throttled when over limit
- **Memory** is incompressible — container is OOMKilled when over limit

---

### Q23. What are Taints and Tolerations?
**Answer:**  
- **Taint** is applied to a **node** to repel pods.
- **Toleration** is applied to a **pod** to allow it to be scheduled on tainted nodes.

```bash
# Taint a node
kubectl taint nodes node1 key=value:NoSchedule

# Toleration in pod spec
tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```

**Taint effects:**
- `NoSchedule` – New pods won't be scheduled
- `PreferNoSchedule` – Scheduler avoids the node if possible
- `NoExecute` – Existing pods are evicted too

---

### Q24. What is the difference between Node Affinity and Pod Affinity?
**Answer:**

| Type | Controls |
|---|---|
| **Node Affinity** | Which nodes a pod can be scheduled on (based on node labels) |
| **Pod Affinity** | Schedule pods close to (or away from) other pods |
| **Pod Anti-Affinity** | Spread pods across nodes (avoid co-location) |

**`requiredDuringScheduling`** = Hard rule (pod won't schedule if not met)  
**`preferredDuringScheduling`** = Soft rule (scheduler tries but doesn't enforce)

---

### Q25. What is a HorizontalPodAutoscaler (HPA)?
**Answer:**  
HPA automatically scales the number of pod replicas based on observed metrics (CPU, memory, or custom metrics).

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

> Requires **Metrics Server** to be installed.

---

### Q26. What is a VerticalPodAutoscaler (VPA)?
**Answer:**  
VPA automatically adjusts the CPU and memory **requests/limits** of containers based on actual usage, rather than changing the number of replicas.

- Useful when you can't easily scale horizontally (e.g., single-instance databases)
- Works in three modes: `Off` (recommendation only), `Initial` (set at pod creation), `Auto` (evict and recreate pods)

---

## 7. RBAC & Security

### Q27. Explain RBAC in Kubernetes.
**Answer:**  
Role-Based Access Control (RBAC) restricts access to Kubernetes API resources.

**Key components:**

| Component | Scope | Description |
|---|---|---|
| `Role` | Namespace | Defines permissions within a namespace |
| `ClusterRole` | Cluster-wide | Defines permissions across the whole cluster |
| `RoleBinding` | Namespace | Binds a Role to a user/group/service account |
| `ClusterRoleBinding` | Cluster-wide | Binds a ClusterRole to a user/group/service account |

```yaml
# Allow "dev-user" to get/list pods in "dev" namespace
kind: Role
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
---
kind: RoleBinding
subjects:
  - kind: User
    name: dev-user
roleRef:
  kind: Role
  name: pod-reader
```

---

### Q28. What is a ServiceAccount?
**Answer:**  
A ServiceAccount provides an identity for processes running inside pods to interact with the Kubernetes API.

- Every namespace has a `default` service account
- Pods use it to authenticate to the API server
- Best practice: Create dedicated service accounts with minimal permissions per application

---

### Q29. What is PodSecurityAdmission (PSA)?
**Answer:**  
PSA replaced the deprecated PodSecurityPolicy (PSP). It enforces security standards at the namespace level using built-in profiles:

| Profile | Description |
|---|---|
| `privileged` | Unrestricted (no restrictions) |
| `baseline` | Minimally restrictive; prevents known privilege escalations |
| `restricted` | Heavily restricted; follows pod hardening best practices |

Applied via namespace labels:
```bash
kubectl label namespace prod pod-security.kubernetes.io/enforce=restricted
```

---

## 8. Helm & Package Management

### Q30. What is Helm and what problem does it solve?
**Answer:**  
Helm is the package manager for Kubernetes. It allows you to define, install, and upgrade complex Kubernetes applications as **charts** (collections of templated YAML files).

**Problems it solves:**
- Eliminates repetitive YAML duplication
- Enables parameterized deployments across environments (dev/prod)
- Provides versioned releases with rollback support
- Simplifies complex multi-resource deployments (e.g., a full stack app)

---

### Q31. What is the difference between Helm 2 and Helm 3?
**Answer:**

| Feature | Helm 2 | Helm 3 |
|---|---|---|
| Tiller | Required (server-side component) | Removed (client-only) |
| Security | Tiller had full cluster access | Uses kubeconfig/RBAC directly |
| Release storage | ConfigMaps in `kube-system` | Secrets in the release namespace |
| CRD handling | Basic | Improved lifecycle management |

---

### Q32. Key Helm commands to know:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
helm install my-release bitnami/nginx -f values.yaml
helm upgrade my-release bitnami/nginx --set image.tag=1.21
helm rollback my-release 1
helm uninstall my-release
helm list -A
helm template my-release bitnami/nginx  # Render templates locally
```

---

## 9. Monitoring, Logging & Observability

### Q33. What is the standard monitoring stack for Kubernetes?
**Answer:**

**Prometheus + Grafana Stack:**
- **Prometheus** – Metrics collection and alerting (scrapes /metrics endpoints)
- **Grafana** – Visualization dashboards
- **Alertmanager** – Handles alert routing (email, Slack, PagerDuty)
- **kube-state-metrics** – Exposes cluster-level metrics
- **Node Exporter** – Exposes node-level OS metrics

**Logging Stack (EFK/ELK):**
- **Elasticsearch** – Log storage and search
- **Fluentd/Fluent Bit** – Log collection agent (runs as DaemonSet)
- **Kibana** – Log visualization

---

### Q34. What are the key metrics to monitor in Kubernetes?
**Answer:**

| Category | Key Metrics |
|---|---|
| **Node** | CPU/Memory utilization, Disk I/O, Network I/O |
| **Pod** | CPU/Memory requests vs limits, Restart count, OOMKills |
| **Cluster** | Total pod count, Pending pods, Failed deployments |
| **API Server** | Request rate, Error rate, Latency |
| **etcd** | Leader elections, DB size, Request latency |
| **Application** | HTTP error rate, p99 latency, throughput |

---

## 10. CI/CD & GitOps

### Q35. How does GitOps work with Kubernetes?
**Answer:**  
GitOps is a practice where **Git is the single source of truth** for declarative infrastructure and applications.

**Workflow:**
1. Developer pushes code → CI pipeline builds & pushes Docker image
2. CI updates image tag in Git manifests
3. GitOps controller (ArgoCD / Flux) detects diff between Git state and cluster state
4. Controller automatically syncs the cluster to match Git

**Tools:** ArgoCD, Flux CD

---

### Q36. ArgoCD vs. Flux – key differences?
**Answer:**

| Feature | ArgoCD | Flux |
|---|---|---|
| UI | Rich web UI | CLI-focused (GitOps Toolkit) |
| Multi-cluster | Yes (native) | Yes |
| Helm support | Yes | Yes |
| Kustomize | Yes | Yes |
| Architecture | Centralized | Decentralized (agent per cluster) |
| App-of-apps | Yes | Yes (Kustomization objects) |

---

## 11. Cluster Maintenance & Upgrades

### Q37. How do you safely drain a node for maintenance?
**Answer:**

```bash
# 1. Cordon the node (mark as unschedulable)
kubectl cordon node-1

# 2. Drain the node (evict all pods gracefully)
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data

# 3. Perform maintenance...

# 4. Uncordon after maintenance
kubectl uncordon node-1
```

> `--ignore-daemonsets` is needed because DaemonSet pods can't be rescheduled elsewhere.

---

### Q38. How do you perform a Kubernetes cluster upgrade?
**Answer:**

**For kubeadm-managed clusters:**

```bash
# 1. Upgrade kubeadm on control plane
apt-get update && apt-get install -y kubeadm=1.29.0-00

# 2. Verify upgrade plan
kubeadm upgrade plan

# 3. Apply upgrade
kubeadm upgrade apply v1.29.0

# 4. Upgrade kubelet & kubectl on control plane
apt-get install -y kubelet=1.29.0-00 kubectl=1.29.0-00
systemctl restart kubelet

# 5. Drain and upgrade each worker node
kubectl drain worker-1 --ignore-daemonsets
# SSH into worker-1
apt-get install -y kubeadm=1.29.0-00 kubelet=1.29.0-00
kubeadm upgrade node
systemctl restart kubelet
# Back on control plane
kubectl uncordon worker-1
```

> **Best practice:** Upgrade one minor version at a time (e.g., 1.27 → 1.28 → 1.29).

---

### Q39. How do you backup and restore etcd?
**Answer:**

```bash
# Backup
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify backup
etcdctl snapshot status /backup/etcd-snapshot.db

# Restore
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore
```

---

## 12. 🔥 Scenario-Based Questions

---

### Scenario 1: Pod Stuck in `Pending` State
**Question:** A pod has been in `Pending` state for 10 minutes. How do you troubleshoot it?

**Answer:**
```bash
# Step 1: Describe the pod to see events
kubectl describe pod <pod-name>

# Look for events like:
# - "0/3 nodes available: insufficient cpu"  → Resource shortage
# - "no nodes match node selector"            → Incorrect node selector
# - "had untolerated taint"                   → Missing toleration
# - "PersistentVolumeClaim not found"         → PVC issue
```

**Resolution steps:**
1. **Insufficient resources** → Scale cluster or reduce resource requests
2. **Node selector mismatch** → Fix labels or node selector in pod spec
3. **Taint/Toleration** → Add correct toleration to pod spec
4. **PVC not bound** → Check PVC status (`kubectl get pvc`) and StorageClass
5. **ImagePullBackOff** → Verify image name, tag, and registry credentials

---

### Scenario 2: Pod in `CrashLoopBackOff`
**Question:** Your application pod keeps crashing. How do you debug it?

**Answer:**
```bash
# Check pod status
kubectl get pods

# View current logs
kubectl logs <pod-name>

# View previous container logs (before crash)
kubectl logs <pod-name> --previous

# Describe for events
kubectl describe pod <pod-name>

# Exec into a healthy pod to test
kubectl exec -it <pod-name> -- /bin/sh
```

**Common causes:**
- Application startup error (wrong config, missing env vars)
- OOMKill (increase memory limits)
- Liveness probe failing too aggressively (increase `initialDelaySeconds`)
- Missing ConfigMap or Secret
- Wrong command/entrypoint in Dockerfile

---

### Scenario 3: Service Not Reachable
**Question:** An application can't connect to a backend service. How do you debug?

**Answer:**
```bash
# Step 1: Verify service exists and has endpoints
kubectl get svc backend-service
kubectl get endpoints backend-service  # Should show pod IPs

# Step 2: Check if pods are running and match selector
kubectl get pods -l app=backend
kubectl describe svc backend-service   # Check selector labels

# Step 3: Test DNS resolution from another pod
kubectl run debug --image=busybox --rm -it -- nslookup backend-service

# Step 4: Test connectivity
kubectl run debug --image=busybox --rm -it -- wget -O- http://backend-service:8080

# Step 5: Check NetworkPolicy
kubectl get networkpolicies
```

**Common causes:**
- Label mismatch between Service selector and Pod labels
- No running pods matching selector → empty endpoints
- NetworkPolicy blocking traffic
- Wrong port or protocol in Service spec
- Application listening on wrong port inside container

---

### Scenario 4: Deployment Rollout Stuck
**Question:** You ran `kubectl apply` on a new deployment but the rollout seems stuck. What do you do?

**Answer:**
```bash
# Check rollout status
kubectl rollout status deployment/my-app

# Check replica sets
kubectl get rs

# Describe deployment for events
kubectl describe deployment my-app

# Check pods in new RS for errors
kubectl get pods | grep my-app
kubectl describe pod <new-pod-name>
```

**Common causes:**
- New pods failing health checks → failing readiness probe blocks rollout
- Image pull failure (ImagePullBackOff)
- Insufficient cluster resources for new pods
- Quota exceeded in namespace

**Quick rollback:**
```bash
kubectl rollout undo deployment/my-app
kubectl rollout undo deployment/my-app --to-revision=2
```

---

### Scenario 5: Node Not Ready
**Question:** A worker node shows `NotReady` status. How do you investigate?

**Answer:**
```bash
# Step 1: Check node status
kubectl get nodes
kubectl describe node <node-name>

# Look for: MemoryPressure, DiskPressure, PIDPressure conditions

# Step 2: SSH into the node
ssh <node-ip>

# Step 3: Check kubelet status
systemctl status kubelet
journalctl -u kubelet -f

# Step 4: Check container runtime
systemctl status containerd

# Step 5: Check disk space
df -h

# Step 6: Check memory
free -m
```

**Common causes:**
- Kubelet crashed → restart kubelet (`systemctl restart kubelet`)
- Disk pressure → clean up old images (`crictl rmi --prune`)
- Memory pressure → identify memory-hungry pods
- Network issue → node can't reach API server
- Certificate expired → renew kubelet cert

---

### Scenario 6: High CPU/Memory Usage — Identify and Fix
**Question:** The cluster is under resource pressure. How do you identify the culprit?

**Answer:**
```bash
# Top nodes
kubectl top nodes

# Top pods across all namespaces
kubectl top pods -A --sort-by=cpu
kubectl top pods -A --sort-by=memory

# Find pods without resource limits
kubectl get pods -A -o json | jq '.items[] | select(.spec.containers[].resources.limits == null) | .metadata.name'

# Check for OOMKilled pods
kubectl get pods -A | grep OOMKilled
kubectl describe pod <oom-pod> | grep -A5 "Last State"
```

**Resolution:**
- Set proper `requests` and `limits` on all pods
- Enable VPA to auto-tune resource requests
- Use LimitRanges to enforce default limits per namespace
- Use ResourceQuotas to cap namespace usage

---

### Scenario 7: Zero-Downtime Deployment
**Question:** How do you ensure zero downtime during a deployment?

**Answer:**

**1. Use Rolling Update strategy (default):**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # Extra pods during rollout
    maxUnavailable: 0  # No pod goes down before new one is up
```

**2. Configure Readiness Probe:**
```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

**3. Set appropriate `terminationGracePeriodSeconds`:**
```yaml
terminationGracePeriodSeconds: 30
```

**4. Use PodDisruptionBudgets (PDB):**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-app
```

---

### Scenario 8: Multi-Tenant Cluster — Resource Isolation
**Question:** You have 3 teams (frontend, backend, data) sharing a cluster. How do you isolate them?

**Answer:**

```bash
# 1. Create namespaces per team
kubectl create namespace frontend
kubectl create namespace backend
kubectl create namespace data

# 2. Apply ResourceQuotas
kubectl apply -f - <<EOF
apiVersion: v1
kind: ResourceQuota
metadata:
  name: backend-quota
  namespace: backend
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "20"
EOF

# 3. Apply LimitRange (default limits per pod)
# 4. Apply RBAC (each team only sees their namespace)
# 5. Apply NetworkPolicies (namespace isolation)
```

---

### Scenario 9: Secret Management with External Vault
**Question:** How would you securely manage secrets in Kubernetes using HashiCorp Vault?

**Answer:**

**Option 1: Vault Agent Sidecar Injection**
- Vault Agent runs as a sidecar in the pod
- Authenticates to Vault using Kubernetes service account (k8s auth method)
- Writes secrets to a shared in-memory volume

**Option 2: External Secrets Operator (ESO)**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: app-secret
  data:
    - secretKey: db-password
      remoteRef:
        key: secret/app/prod
        property: db_password
```

**Option 3: Vault CSI Provider** — mounts secrets as files via CSI driver

---

### Scenario 10: Kubernetes Upgrade — Production Strategy
**Question:** How would you upgrade a production Kubernetes cluster with minimal risk?

**Answer:**

1. **Backup etcd** before starting
2. **Review release notes** — check deprecations and API changes
3. **Upgrade non-production first** — test in staging
4. **One minor version at a time** — never skip versions
5. **Upgrade control plane first**, then worker nodes
6. **Rolling node upgrades** — drain one node at a time, verify workloads, then proceed
7. **Validate after each node** — run smoke tests / health checks
8. **Monitor** — watch for errors, OOMKills, scheduling issues
9. **Have a rollback plan** — know how to restore etcd snapshot

---

### Scenario 11: Pod Can't Pull Image
**Question:** Pods are failing with `ImagePullBackOff`. How do you fix it?

**Answer:**
```bash
# Describe pod to see exact error
kubectl describe pod <pod-name>

# Common causes and fixes:

# 1. Wrong image name/tag
# Fix: Correct the image field in deployment spec

# 2. Private registry — missing imagePullSecret
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass

# Add to pod spec:
# imagePullSecrets:
#   - name: regcred

# 3. Rate limiting (Docker Hub)
# Fix: Authenticate or use a mirror

# 4. Image doesn't exist
# Fix: Check and push the image to registry
```

---

### Scenario 12: Liveness vs Readiness Probes
**Question:** What is the difference between Liveness and Readiness probes, and when would misconfiguring them cause issues?

**Answer:**

| Probe | Action on Failure | Purpose |
|---|---|---|
| **Liveness** | Restarts the container | Detects deadlock / hung application |
| **Readiness** | Removes pod from Service endpoints | Signals "not ready for traffic" |
| **Startup** | Restarts after threshold | For slow-starting containers |

**Common misconfiguration issues:**

- **Liveness probe too aggressive** (short `initialDelaySeconds`) → container restarts before app finishes starting → `CrashLoopBackOff`
- **Readiness probe missing** → traffic sent to pods that aren't ready → errors during rolling updates
- **Same endpoint for both** on an app that can't handle health checks during high load → liveness kills a busy but healthy pod

**Rule of thumb:** Always set readiness probe. Only add liveness probe when you know the app can deadlock. Always set `initialDelaySeconds` > app startup time.

---

## 13. ⚡ Quick-Fire Questions

| Question | Answer |
|---|---|
| What command shows all resources in a namespace? | `kubectl get all -n <namespace>` |
| How do you force delete a stuck pod? | `kubectl delete pod <name> --force --grace-period=0` |
| How do you view pod logs with timestamps? | `kubectl logs <pod> --timestamps` |
| How do you stream logs? | `kubectl logs -f <pod>` |
| How do you check events cluster-wide? | `kubectl get events -A --sort-by=.metadata.creationTimestamp` |
| What is `kubectl port-forward` used for? | Local access to a pod/service without exposing it |
| How do you patch a resource inline? | `kubectl patch deploy <name> -p '{"spec":{"replicas":3}}'` |
| What is the purpose of `kube-proxy`? | Manages network rules (iptables/IPVS) for Service routing |
| How do you check which node a pod is on? | `kubectl get pod <name> -o wide` |
| What is a PodDisruptionBudget? | Ensures minimum pods stay available during voluntary disruptions |
| What is `kubectl diff`? | Shows diff between current cluster state and local manifest |
| What are Finalizers? | Metadata markers that prevent object deletion until cleanup is done |
| What is an Operator? | A controller that encodes domain knowledge to manage complex apps |
| What tool checks K8s best practices? | `kube-score`, `kube-linter`, `Polaris` |
| How do you get a shell in a running pod? | `kubectl exec -it <pod> -- /bin/bash` |
| What is `kubectl apply` vs `kubectl create`? | `apply` = declarative (create+update); `create` = imperative (fails if exists) |
| How do you see resource usage per namespace? | `kubectl top pods -A` + ResourceQuota `kubectl describe quota -n <ns>` |
| What is a Mutating Admission Webhook? | Intercepts API requests and can modify resources before they're stored |

---

## 💡 Pro Tips for Interviews

1. **Practice kubectl commands** — Know `describe`, `logs`, `exec`, `top`, `events` by heart
2. **Understand the control loop** — Kubernetes is declarative; controllers reconcile desired vs actual state
3. **Know when NOT to use Kubernetes** — Small apps, monoliths, simple use cases don't need K8s
4. **YAML fluency matters** — Be comfortable writing deployment, service, ingress, HPA manifests from scratch
5. **Security first** — Always mention RBAC, NetworkPolicies, and secret management in architecture questions
6. **Troubleshooting framework** — `describe` → `logs` → `events` → `exec` → network test

---

*Good luck with your interviews! ☸️🚀*
