# Kubernetes - Comprehensive Q&A Guide

> A complete reference covering Kubernetes concepts, architecture, security, and troubleshooting.

---

## Table of Contents

1. [What is Kubernetes and why is it used?](#1-what-is-kubernetes-and-why-is-it-used)
2. [Core components of Kubernetes architecture](#2-what-are-the-core-components-of-kubernetes-architecture)
3. [What is a Pod?](#3-what-is-a-pod-in-kubernetes)
4. [What is a Deployment and how does it work?](#4-what-is-a-deployment-and-how-does-it-work)
5. [What is a Service?](#5-what-is-a-service-in-kubernetes)
6. [Types of Services](#6-what-are-different-types-of-services-clusterip-nodeport-loadbalancer)
7. [What is a Namespace?](#7-what-is-a-namespace-and-why-is-it-used)
8. [Deployment vs StatefulSet](#8-what-is-the-difference-between-deployment-and-statefulset)
9. [What is a DaemonSet?](#9-what-is-a-daemonset-and-when-would-you-use-it)
10. [What is Ingress?](#10-what-is-ingress-and-how-does-it-work)
11. [ConfigMaps and Secrets](#11-what-are-configmaps-and-secrets)
12. [Liveness and Readiness Probes](#12-what-are-liveness-and-readiness-probes)
13. [Horizontal Pod Autoscaler (HPA)](#13-what-is-horizontal-pod-autoscaler-hpa)
14. [RBAC in Kubernetes](#14-what-is-rbac-in-kubernetes)
15. [Role vs ClusterRole](#15-what-is-the-difference-between-role-and-clusterrole)
16. [Kubernetes Networking Internals](#16-how-does-kubernetes-networking-work-internally)
17. [What is CNI?](#17-what-is-cni-and-how-does-it-function)
18. [Service Discovery](#18-how-does-kubernetes-handle-service-discovery)
19. [etcd backup and restore](#19-what-is-etcd-and-how-do-you-manage-its-backup-and-restore)
20. [Taints and Tolerations](#20-what-are-taints-and-tolerations)
21. [Affinity and Anti-Affinity Rules](#21-what-are-affinity-and-anti-affinity-rules)
22. [Securing a Kubernetes Cluster](#22-how-do-you-secure-a-kubernetes-cluster)
23. [What is Helm?](#23-what-is-helm-and-how-does-it-simplify-deployments)
24. [Troubleshooting CrashLoopBackOff](#24-a-pod-is-stuck-in-crashloopbackoff--how-do-you-troubleshoot-it)
25. [Application not accessible via Service](#25-your-application-is-not-accessible-via-service--what-steps-will-you-take)
26. [Zero-Downtime Deployment](#26-how-would-you-perform-zero-downtime-deployment-in-kubernetes)
27. [Node Not Ready](#27-a-node-is-not-ready--how-do-you-investigate)
28. [High CPU/Memory Usage](#28-how-do-you-handle-high-cpumemory-usage-in-a-cluster)
29. [Auto-scaling Applications](#29-how-do-you-scale-applications-automatically-in-kubernetes)
30. [Monitoring and Logging](#30-how-do-you-monitor-and-log-kubernetes-workloads-effectively)

---

## 1. What is Kubernetes and why is it used?

**Kubernetes** (also known as K8s) is an open-source container orchestration platform originally developed by Google and now maintained by the Cloud Native Computing Foundation (CNCF). It automates the deployment, scaling, and management of containerized applications.

### Why is it used?

| Problem | Kubernetes Solution |
|---|---|
| Manual container management at scale | Automated scheduling and lifecycle management |
| Application downtime during updates | Rolling updates and rollbacks |
| Unpredictable traffic spikes | Horizontal auto-scaling (HPA/VPA) |
| Container crash recovery | Self-healing via restartPolicy and probes |
| Multi-environment consistency | Declarative YAML manifests |
| Microservice networking | Built-in DNS and Service abstraction |

### Key Benefits

- **Self-healing**: Automatically restarts failed containers, replaces and reschedules them on healthy nodes.
- **Horizontal scaling**: Scale applications up or down with a single command or automatically based on CPU/memory metrics.
- **Service discovery & load balancing**: No need to modify your application; Kubernetes manages traffic routing.
- **Automated rollouts & rollbacks**: Progressively roll out changes and roll back if something goes wrong.
- **Storage orchestration**: Automatically mount storage systems (local, cloud, NFS, etc.).
- **Secret and configuration management**: Deploy and update secrets without rebuilding container images.

---

## 2. What are the core components of Kubernetes architecture?

Kubernetes follows a **master-worker (control plane - data plane)** architecture.

```
┌─────────────────────────────────────────────────────────┐
│                    CONTROL PLANE                        │
│  ┌──────────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │  API Server  │  │  etcd    │  │ Controller Mgr   │  │
│  └──────────────┘  └──────────┘  └──────────────────┘  │
│  ┌──────────────┐  ┌──────────────────────────────────┐ │
│  │  Scheduler   │  │  Cloud Controller Manager (opt.) │ │
│  └──────────────┘  └──────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
              │              │
   ┌──────────┘              └───────────┐
   ▼                                     ▼
┌──────────────────┐          ┌──────────────────┐
│    WORKER NODE   │          │    WORKER NODE   │
│  ┌────────────┐  │          │  ┌────────────┐  │
│  │  kubelet   │  │          │  │  kubelet   │  │
│  ├────────────┤  │          │  ├────────────┤  │
│  │  kube-proxy│  │          │  │  kube-proxy│  │
│  ├────────────┤  │          │  ├────────────┤  │
│  │ Container  │  │          │  │ Container  │  │
│  │  Runtime   │  │          │  │  Runtime   │  │
│  └────────────┘  │          │  └────────────┘  │
└──────────────────┘          └──────────────────┘
```

### Control Plane Components

- **kube-apiserver**: The front-end of the control plane. All internal and external communication goes through it. Exposes the Kubernetes REST API.
- **etcd**: A distributed key-value store that holds the entire cluster state and configuration. The single source of truth.
- **kube-scheduler**: Watches for newly created Pods with no assigned node and selects a node for them to run on based on resource requirements, constraints, and policies.
- **kube-controller-manager**: Runs controller loops — Node Controller, Replication Controller, Endpoints Controller, etc.
- **cloud-controller-manager** *(optional)*: Integrates with cloud provider APIs for load balancers, storage, etc.

### Worker Node Components

- **kubelet**: An agent that runs on each node. It ensures containers in a Pod are running and healthy.
- **kube-proxy**: Maintains network rules on nodes to allow network communication to your Pods.
- **Container Runtime**: The software that runs containers (e.g., containerd, CRI-O, Docker).

---

## 3. What is a Pod in Kubernetes?

A **Pod** is the smallest and most basic deployable unit in Kubernetes. It represents a single instance of a running process in your cluster and can contain one or more containers that share:

- The same **network namespace** (same IP address and port space)
- The same **storage volumes**
- The same **lifecycle**

### Example Pod manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
    - name: my-container
      image: nginx:1.21
      ports:
        - containerPort: 80
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"
```

### Key Points

- Pods are **ephemeral** — they are not self-healing. If a Pod dies, it's not automatically replaced unless managed by a higher-level controller (Deployment, StatefulSet, etc.).
- Containers within a Pod communicate via `localhost`.
- Multi-container Pods are useful for **sidecar** patterns (e.g., a logging agent alongside an app container).
- Pods share the same lifecycle — if the Pod restarts, all containers restart.

---

## 4. What is a Deployment and how does it work?

A **Deployment** is a higher-level Kubernetes object that manages a ReplicaSet, which in turn manages Pods. It provides declarative updates for Pods and ReplicaSets.

### How it works

```
Deployment
    └── ReplicaSet (new)
            ├── Pod
            ├── Pod
            └── Pod
    └── ReplicaSet (old — scaled down during rolling update)
```

1. You define the desired state in a Deployment spec.
2. The Deployment Controller compares desired state to actual state.
3. If they differ, it creates/updates/deletes ReplicaSets to reconcile.

### Example Deployment manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:v2
          ports:
            - containerPort: 8080
```

### Key Features

- **Rolling updates**: Updates Pods incrementally without downtime.
- **Rollback**: `kubectl rollout undo deployment/my-deployment`
- **Pause/Resume**: Pause a rollout mid-way to inspect, then resume.
- **History**: `kubectl rollout history deployment/my-deployment`

---

## 5. What is a Service in Kubernetes?

A **Service** is a stable abstraction that defines a logical set of Pods and a policy to access them. Since Pods are ephemeral and their IPs change, Services provide a stable endpoint.

### How it works

- Services use **label selectors** to identify the Pods they route traffic to.
- kube-proxy on each node watches for Service and Endpoint updates and sets up iptables/IPVS rules.
- Services get a stable **ClusterIP** and a DNS name (`<service-name>.<namespace>.svc.cluster.local`).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80        # Service port
      targetPort: 8080  # Container port
```

---

## 6. What are different types of Services (ClusterIP, NodePort, LoadBalancer)?

### ClusterIP *(default)*

- Exposes the Service on an internal IP within the cluster.
- Only accessible from within the cluster.
- Use case: Internal microservice-to-microservice communication.

```yaml
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 8080
```

### NodePort

- Exposes the Service on each Node's IP at a static port (range: `30000–32767`).
- Accessible from outside the cluster via `<NodeIP>:<NodePort>`.
- Use case: Development/testing, or when you don't have a cloud load balancer.

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

### LoadBalancer

- Provisions an external load balancer from the cloud provider (AWS ELB, GCP LB, etc.).
- Accessible from the internet via the external IP/DNS.
- Use case: Production workloads in cloud environments.

```yaml
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
```

### ExternalName *(bonus)*

- Maps a Service to a DNS name (no proxying, just CNAME).
- Use case: Access external services (e.g., databases outside the cluster) using an in-cluster DNS name.

---

## 7. What is a Namespace and why is it used?

A **Namespace** is a virtual cluster within a Kubernetes cluster. It provides a mechanism for isolating groups of resources within a single cluster.

### Default Namespaces

| Namespace | Purpose |
|---|---|
| `default` | Default namespace for user workloads |
| `kube-system` | Kubernetes system components (kube-dns, kube-proxy, etc.) |
| `kube-public` | Publicly readable, used for cluster info |
| `kube-node-lease` | Node heartbeat leases |

### Why use Namespaces?

- **Multi-tenancy**: Separate teams, projects, or environments (dev/staging/prod) in the same cluster.
- **Resource quotas**: Limit CPU, memory, and object counts per namespace.
- **Access control**: Apply RBAC policies per namespace.
- **Scoped DNS**: Services are accessible within a namespace by short name; cross-namespace requires FQDN.

```bash
# Create a namespace
kubectl create namespace dev-team

# Set resource quota on a namespace
kubectl apply -f - <<EOF
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev-team
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
EOF
```

---

## 8. What is the difference between Deployment and StatefulSet?

| Feature | Deployment | StatefulSet |
|---|---|---|
| **Pod identity** | Random names (`app-xyz123`) | Stable, ordered names (`app-0`, `app-1`) |
| **Pod ordering** | Simultaneous creation/deletion | Sequential (ordered) creation and deletion |
| **Storage** | Shared or ephemeral | Each Pod gets its own PersistentVolumeClaim |
| **Network identity** | Shared Service IP | Stable network identity via headless service |
| **Use case** | Stateless apps (web servers, APIs) | Stateful apps (databases, message queues) |
| **Rolling updates** | All Pods use same strategy | Ordered rolling updates from last to first |

### When to use StatefulSet

- **Databases**: MySQL, PostgreSQL, MongoDB, Cassandra
- **Message brokers**: Kafka, RabbitMQ
- **Distributed systems**: Elasticsearch, ZooKeeper
- Any workload that requires **stable network IDs** or **persistent, dedicated storage**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
  volumeClaimTemplates:           # Each Pod gets its own PVC
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

---

## 9. What is a DaemonSet and when would you use it?

A **DaemonSet** ensures that a copy of a Pod runs on **all (or some) nodes** in the cluster. When a new node is added to the cluster, a Pod is automatically added to it. When a node is removed, the Pod is garbage collected.

### Use Cases

- **Log collection**: Fluentd, Filebeat — collect logs from every node.
- **Monitoring agents**: Prometheus Node Exporter, Datadog Agent.
- **Networking**: CNI plugins (Calico, Flannel), kube-proxy itself.
- **Storage plugins**: Ceph, GlusterFS daemons.
- **Security**: Falco, intrusion detection agents.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      tolerations:                  # Allow running on control-plane nodes
        - key: node-role.kubernetes.io/control-plane
          operator: Exists
          effect: NoSchedule
      containers:
        - name: fluentd
          image: fluent/fluentd-kubernetes-daemonset:v1
          volumeMounts:
            - name: varlog
              mountPath: /var/log
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
```

---

## 10. What is Ingress and how does it work?

**Ingress** is an API object that manages external HTTP/HTTPS access to Services within the cluster. It provides:

- **Path-based routing**: Route `/api` to one service, `/web` to another.
- **Host-based routing**: Route `api.example.com` to one service, `app.example.com` to another.
- **TLS termination**: Handle SSL/TLS at the Ingress level.

### How it works

```
Internet
   │
   ▼
Ingress Controller (e.g., nginx, Traefik, AWS ALB)
   │
   ├── /api  → backend-service:8080
   ├── /web  → frontend-service:80
   └── TLS termination
```

An **Ingress Controller** (like nginx-ingress, Traefik, or cloud-native controllers) must be deployed — it reads Ingress resources and configures itself accordingly.

### Example Ingress manifest

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
    - hosts:
        - myapp.example.com
      secretName: myapp-tls
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 8080
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

---

## 11. What are ConfigMaps and Secrets?

Both ConfigMaps and Secrets decouple configuration from container images, but serve different purposes.

### ConfigMap

Stores **non-sensitive** configuration data as key-value pairs.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
  DB_HOST: postgres.default.svc.cluster.local
```

### Secret

Stores **sensitive** data (passwords, tokens, keys). Values are base64-encoded (not encrypted by default — use encryption at rest + RBAC).

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQxMjM=    # base64 of "password123"
  API_KEY: c2VjcmV0a2V5MTIz          # base64 of "secretkey123"
```

### Consuming them in Pods

```yaml
spec:
  containers:
    - name: app
      image: myapp:latest
      envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secret
      # OR mount as files:
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: app-config
```

### Key Differences

| Feature | ConfigMap | Secret |
|---|---|---|
| Data sensitivity | Non-sensitive | Sensitive |
| Encoding | Plain text | Base64 |
| Encryption at rest | Not by default | Can be enabled |
| Use for | App config, env vars | Passwords, tokens, certs |

---

## 12. What are liveness and readiness probes?

Probes are periodic health checks that the kubelet performs on containers.

### Liveness Probe

Determines if the **container is alive**. If it fails, the kubelet kills the container and restarts it (based on `restartPolicy`).

> Use case: Detect deadlocks or states where the app is running but cannot make progress.

### Readiness Probe

Determines if the **container is ready to receive traffic**. If it fails, the Pod is removed from Service endpoints (no traffic routed to it) — but the container is NOT restarted.

> Use case: Wait for app startup, or temporarily remove from load balancing during heavy processing.

### Startup Probe *(bonus)*

Used for slow-starting applications. Disables liveness and readiness probes until the startup probe succeeds, preventing premature restarts.

### Example

```yaml
spec:
  containers:
    - name: app
      image: myapp:latest
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 5
        failureThreshold: 3
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 3
        failureThreshold: 2
      startupProbe:
        httpGet:
          path: /healthz
          port: 8080
        failureThreshold: 30
        periodSeconds: 10
```

### Probe Types

| Type | Description |
|---|---|
| `httpGet` | HTTP GET request — success if 200-399 |
| `tcpSocket` | TCP connection check |
| `exec` | Execute command inside container — success if exit code 0 |
| `grpc` | gRPC health check protocol |

---

## 13. What is Horizontal Pod Autoscaler (HPA)?

The **Horizontal Pod Autoscaler** automatically scales the number of Pods in a Deployment, ReplicaSet, or StatefulSet based on observed CPU/memory utilization or custom metrics.

### How it works

1. HPA queries the Metrics Server (or custom metrics API) at regular intervals.
2. Calculates desired replicas: `desiredReplicas = ceil(currentReplicas × (currentMetric / desiredMetric))`
3. Updates the replica count of the target resource.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: AverageValue
          averageValue: 500Mi
```

### Prerequisites

- **Metrics Server** must be installed in the cluster for CPU/memory metrics.
- Pods must have **resource requests** defined for CPU/memory-based scaling.

```bash
# Check HPA status
kubectl get hpa
kubectl describe hpa my-hpa
```

---

## 14. What is RBAC in Kubernetes?

**Role-Based Access Control (RBAC)** is a method of regulating access to Kubernetes resources based on the roles of individual users or service accounts. It is enabled by default in modern Kubernetes clusters.

### Core RBAC Objects

| Object | Description |
|---|---|
| `Role` | Defines permissions within a **specific namespace** |
| `ClusterRole` | Defines permissions **cluster-wide** |
| `RoleBinding` | Binds a Role to a user/group/service account **within a namespace** |
| `ClusterRoleBinding` | Binds a ClusterRole to a user/group/service account **cluster-wide** |

### Example

```yaml
# Role: allow reading pods in "dev" namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]

---
# Bind the role to a user
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: dev
subjects:
  - kind: User
    name: jane
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 15. What is the difference between Role and ClusterRole?

| Feature | Role | ClusterRole |
|---|---|---|
| **Scope** | Single namespace | Cluster-wide (all namespaces) |
| **Can manage** | Namespaced resources only | Namespaced + cluster-scoped resources |
| **Cluster-scoped resources** | Cannot access (nodes, PVs, etc.) | Can access |
| **Bound via** | RoleBinding | ClusterRoleBinding (or RoleBinding for namespace-scoped use) |

### Key Insight: ClusterRole can be bound in two ways

1. **ClusterRoleBinding**: Grants the permissions cluster-wide.
2. **RoleBinding (referencing a ClusterRole)**: Grants the ClusterRole's permissions **only within a specific namespace**. This is useful for reusing a common role definition across multiple namespaces.

```yaml
# This grants cluster-admin ClusterRole only within the "dev" namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-admin
  namespace: dev
subjects:
  - kind: User
    name: jane
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole          # References a ClusterRole
  name: admin
  apiGroup: rbac.authorization.k8s.io
```

---

## 16. How does Kubernetes networking work internally?

Kubernetes networking is governed by a flat network model with these core requirements:

- Every **Pod gets a unique IP** address.
- All Pods can communicate with each other **without NAT**.
- Nodes can communicate with Pods **without NAT**.
- The IP that a Pod sees for itself is the same that other Pods see.

### Traffic Flow Scenarios

**Pod-to-Pod (same node):** Traffic flows through a virtual ethernet (veth) pair and a Linux bridge on the node.

**Pod-to-Pod (different nodes):** Handled by the CNI plugin using overlay networks (VXLAN, BGP, etc.).

**Pod-to-Service:** kube-proxy maintains iptables/IPVS rules that translate the Service ClusterIP to a Pod IP (load balancing).

**External-to-Service (LoadBalancer):** Traffic flows from the cloud load balancer → NodePort → kube-proxy → Pod.

### DNS Resolution

CoreDNS runs as a Deployment in `kube-system` and provides cluster-internal DNS:

```
<service-name>.<namespace>.svc.cluster.local → ClusterIP
<pod-ip-with-dashes>.<namespace>.pod.cluster.local → Pod IP
```

---

## 17. What is CNI and how does it function?

**CNI (Container Network Interface)** is a specification and set of libraries for configuring network interfaces in Linux containers. Kubernetes uses CNI plugins to set up Pod networking.

### How CNI works

1. When a Pod is created, kubelet calls the CNI plugin.
2. The CNI plugin creates a network interface (veth pair), assigns an IP, sets up routes.
3. When a Pod is deleted, the plugin tears down the interface and releases the IP.

### Popular CNI Plugins

| Plugin | Mechanism | Key Feature |
|---|---|---|
| **Calico** | BGP / overlay | Network policies, high performance |
| **Flannel** | VXLAN overlay | Simple, widely used |
| **Cilium** | eBPF | Advanced observability, L7 policies |
| **Weave Net** | Overlay mesh | Easy setup, encryption |
| **AWS VPC CNI** | AWS ENI | Native VPC IPs for Pods |

### Network Policies (enforced by CNI)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: database
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api-server
      ports:
        - protocol: TCP
          port: 5432
```

---

## 18. How does Kubernetes handle service discovery?

Kubernetes provides two mechanisms for service discovery:

### 1. Environment Variables

When a Pod starts, kubelet injects environment variables for each active Service in the same namespace:

```bash
MY_SERVICE_SERVICE_HOST=10.96.0.100
MY_SERVICE_SERVICE_PORT=80
```

**Limitation**: Only Services that existed before the Pod was created are injected.

### 2. DNS (Recommended)

**CoreDNS** runs in the cluster and provides DNS-based service discovery. Every Service gets a DNS record automatically:

```
# Service DNS format
<service-name>.<namespace>.svc.cluster.local

# Examples
my-service.default.svc.cluster.local         → ClusterIP
my-service.dev.svc.cluster.local              → Cross-namespace access
_http._tcp.my-service.default.svc.cluster.local → SRV record (port discovery)
```

Within the same namespace, you can use just the service name:
```bash
curl http://my-service/api
```

Cross-namespace:
```bash
curl http://my-service.other-namespace.svc.cluster.local/api
```

---

## 19. What is etcd and how do you manage its backup and restore?

**etcd** is a distributed, consistent key-value store that acts as the **backing store for all Kubernetes cluster data** — all objects (Pods, Services, Deployments, Secrets, etc.) are persisted here.

### Characteristics

- Uses the **Raft consensus algorithm** for consistency.
- Typically runs as a 3 or 5 node cluster for high availability (odd number to achieve quorum).
- Quorum = `(N/2) + 1` nodes must be healthy.

### Backup

```bash
# Using etcdctl (set ETCDCTL_API=3)
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify the backup
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-snapshot.db --write-out=table
```

### Restore

```bash
# Stop the API server first, then:
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restored \
  --initial-cluster=master=https://127.0.0.1:2380 \
  --initial-advertise-peer-urls=https://127.0.0.1:2380 \
  --name=master

# Update etcd manifest to point to the new data directory
# Then restart etcd and the API server
```

### Best Practices

- **Automate backups** with a CronJob (every 6 hours or before any major change).
- Store backups **off-cluster** (S3, GCS, Azure Blob).
- Test restores **regularly** in a non-production environment.
- Encrypt backups as they contain all Secrets.

---

## 20. What are taints and tolerations?

**Taints** and **Tolerations** work together to ensure Pods are not scheduled onto inappropriate nodes.

### Taints (applied to nodes)

A taint repels Pods from being scheduled on a node unless the Pod tolerates it.

```bash
# Add a taint to a node
kubectl taint nodes node1 key=value:NoSchedule
kubectl taint nodes node1 dedicated=gpu:NoSchedule

# Remove a taint
kubectl taint nodes node1 key=value:NoSchedule-
```

### Taint Effects

| Effect | Behavior |
|---|---|
| `NoSchedule` | Pods that don't tolerate will NOT be scheduled (existing Pods unaffected) |
| `PreferNoSchedule` | Soft version — scheduler tries to avoid, but will schedule if no other option |
| `NoExecute` | Pods that don't tolerate will NOT be scheduled AND existing Pods will be evicted |

### Tolerations (applied to Pods)

```yaml
spec:
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"
    # Tolerate any taint with effect NoExecute for 3600s (useful for node failures)
    - key: "node.kubernetes.io/not-ready"
      operator: "Exists"
      effect: "NoExecute"
      tolerationSeconds: 3600
```

### Common Use Cases

- **Dedicated nodes**: Taint GPU nodes for ML workloads only.
- **Control plane isolation**: Default taint on control-plane nodes prevents regular workloads.
- **Node maintenance**: Taint a node with `NoExecute` to drain it gracefully.

---

## 21. What are affinity and anti-affinity rules?

Affinity rules provide more expressive Pod scheduling constraints compared to `nodeSelector`. They allow **required** (`requiredDuringSchedulingIgnoredDuringExecution`) or **preferred** (`preferredDuringSchedulingIgnoredDuringExecution`) rules.

### Node Affinity

Schedule Pods on specific nodes based on node labels.

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/arch
                operator: In
                values:
                  - amd64
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values:
                  - us-east-1a
```

### Pod Affinity

Co-locate Pods with other Pods (e.g., place cache near the app).

```yaml
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - redis-cache
          topologyKey: kubernetes.io/hostname
```

### Pod Anti-Affinity

Spread Pods across nodes/zones for high availability.

```yaml
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - my-app
          topologyKey: kubernetes.io/hostname   # One Pod per node
```

> **Tip**: Use `topologyKey: topology.kubernetes.io/zone` to spread across availability zones.

---

## 22. How do you secure a Kubernetes cluster?

Security in Kubernetes is multi-layered. The **4Cs of Cloud Native Security** are: Cloud, Cluster, Container, Code.

### Authentication & Authorization

- Use **OIDC/LDAP** for user authentication instead of static credentials.
- Enable and enforce **RBAC** — follow the principle of least privilege.
- Use **Service Accounts** for workload identity; disable default service account auto-mounting where not needed.

### Control Plane

- **Restrict API server access**: Use firewall rules, private endpoints.
- **Enable audit logging**: Track who did what in the cluster.
- **Enable encryption at rest** for etcd (especially for Secrets).
- **Rotate certificates** and use short-lived tokens.

### Workload Security

- Use **Pod Security Standards** (Restricted, Baseline, Privileged policies).
- Never run containers as **root**; use `runAsNonRoot: true`.
- Set **read-only root filesystem**: `readOnlyRootFilesystem: true`.
- Drop unnecessary **Linux capabilities**: `capabilities: { drop: ["ALL"] }`.
- Use **Network Policies** to restrict Pod-to-Pod communication.

```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
    - name: app
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop: ["ALL"]
```

### Image Security

- Use **minimal base images** (distroless, Alpine).
- Scan images with tools like **Trivy, Snyk, or Grype**.
- Use image **digest pinning** (`image: nginx@sha256:...`) instead of mutable tags.
- Use a **private registry** with access controls.

### Secrets Management

- Avoid storing secrets in plain YAML/Git — use **Sealed Secrets**, **External Secrets Operator**, or **HashiCorp Vault**.
- Enable **encryption at rest** for the `secrets` resource in etcd.

### Runtime Security

- Use **Falco** for runtime threat detection.
- Use **OPA/Gatekeeper** or **Kyverno** for policy enforcement (admission controllers).

---

## 23. What is Helm and how does it simplify deployments?

**Helm** is the package manager for Kubernetes. It allows you to define, install, and upgrade complex Kubernetes applications using templates called **Charts**.

### Key Concepts

| Concept | Description |
|---|---|
| **Chart** | A package of pre-configured Kubernetes resources |
| **Release** | A running instance of a Chart in the cluster |
| **Repository** | A place to store and share Charts (like npm or apt) |
| **Values** | Configuration parameters to customize a Chart |

### How it simplifies deployments

1. **Templating**: Parameterize YAML files — no more copy-pasting manifests with manual edits.
2. **Package management**: Bundle all resources (Deployment, Service, Ingress, ConfigMap) into one chart.
3. **Release management**: Track revision history; `helm rollback` to previous state.
4. **Dependency management**: Charts can declare dependencies on other charts.

### Basic Commands

```bash
# Add a repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Install a chart
helm install my-nginx bitnami/nginx

# Install with custom values
helm install my-app ./my-chart -f values-prod.yaml

# Upgrade a release
helm upgrade my-app ./my-chart --set image.tag=v2.0

# Rollback to previous revision
helm rollback my-app 1

# List releases
helm list

# Uninstall a release
helm uninstall my-app
```

### Chart structure

```
my-chart/
├── Chart.yaml          # Chart metadata (name, version, description)
├── values.yaml         # Default configuration values
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl    # Template helpers
└── charts/             # Dependent sub-charts
```

---

## 24. A Pod is stuck in CrashLoopBackOff — how do you troubleshoot it?

`CrashLoopBackOff` means the container keeps crashing and Kubernetes is backing off restarting it (with increasing delays).

### Step-by-step Troubleshooting

```bash
# 1. Check Pod status and recent events
kubectl get pod <pod-name> -o wide
kubectl describe pod <pod-name>
# Look for: "Last State", "Exit Code", "Events" section

# 2. Check current logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>   # for multi-container pods

# 3. Check PREVIOUS container logs (most useful for crash diagnosis)
kubectl logs <pod-name> --previous
kubectl logs <pod-name> -c <container-name> --previous

# 4. Check events in the namespace
kubectl get events --sort-by=.metadata.creationTimestamp

# 5. Exec into a running init container or debug
kubectl debug -it <pod-name> --image=busybox --target=<container-name>
```

### Common Causes and Fixes

| Exit Code | Cause | Fix |
|---|---|---|
| `1` | Application error | Check app logs, fix code/config |
| `137` | OOM Killed | Increase memory limits |
| `139` | Segfault | Check application for bugs |
| `143` | SIGTERM not handled | Fix graceful shutdown handling |
| `127` | Command not found | Fix `command`/`entrypoint` in spec |

### Other Causes

- **Wrong image** or tag: Verify with `kubectl describe pod` → `Image`
- **Missing ConfigMap/Secret**: Check for "secret not found" errors in events
- **Failing liveness probe**: Probe failing too early → increase `initialDelaySeconds`
- **Resource limits too low**: Pod OOMKilled → increase memory limit
- **Wrong entrypoint/command**: Verify `command` and `args` in spec

---

## 25. Your application is not accessible via Service — what steps will you take?

### Systematic Debugging Approach

```bash
# Step 1: Verify the Service exists and has the right config
kubectl get svc my-service -o yaml
# Check: selector, port, targetPort, type

# Step 2: Check if the Service has Endpoints
kubectl get endpoints my-service
# If "No endpoints" → label selector mismatch

# Step 3: Verify Pod labels match Service selector
kubectl get pods --show-labels
# Service selector must match at least one Pod label

# Step 4: Check if Pods are Running and Ready
kubectl get pods -l app=my-app
kubectl describe pod <pod-name>
# Check readiness probe — unhealthy Pods are removed from endpoints

# Step 5: Test connectivity from inside the cluster
kubectl run debug --image=busybox -it --rm -- /bin/sh
# Inside the debug pod:
wget -qO- http://my-service.default.svc.cluster.local
nslookup my-service

# Step 6: Check if kube-proxy is running on nodes
kubectl get pods -n kube-system | grep kube-proxy

# Step 7: For NodePort/LoadBalancer — check firewall/security groups
# Ensure the port is open in cloud provider security groups

# Step 8: Check network policies
kubectl get networkpolicies
# A NetworkPolicy might be blocking traffic
```

### Checklist

- [ ] Service selector matches Pod labels
- [ ] Pod is Running and Ready
- [ ] `targetPort` matches the container's `containerPort`
- [ ] Readiness probe is passing
- [ ] No NetworkPolicy blocking the traffic
- [ ] For NodePort: firewall allows the port
- [ ] CoreDNS is running: `kubectl get pods -n kube-system | grep coredns`

---

## 26. How would you perform zero-downtime deployment in Kubernetes?

### 1. Rolling Update Strategy (Default)

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # Max Pods above desired count during update
      maxUnavailable: 0    # ZERO unavailability: always keep desired count running
```

This ensures new Pods are started and pass readiness checks before old ones are terminated.

### 2. Readiness Probes (Critical)

Without a readiness probe, Kubernetes sends traffic to new Pods before they're ready.

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 3
```

### 3. Graceful Shutdown

Ensure your app handles `SIGTERM` gracefully — finish in-flight requests before exiting.

```yaml
spec:
  terminationGracePeriodSeconds: 60    # Give app 60s to finish requests
  containers:
    - lifecycle:
        preStop:
          exec:
            command: ["/bin/sh", "-c", "sleep 5"]  # Let load balancer drain
```

### 4. Blue-Green Deployment

Run two identical environments (blue = current, green = new). Switch traffic by updating the Service selector.

```bash
# Update service selector to point to green deployment
kubectl patch service my-service -p '{"spec":{"selector":{"version":"green"}}}'
```

### 5. Canary Deployment

Route a small percentage of traffic to the new version, gradually increasing it.

```bash
# Run 1 canary Pod alongside 9 stable Pods (10% canary traffic)
# Both share the same Service label selector
kubectl scale deployment my-app-stable --replicas=9
kubectl scale deployment my-app-canary --replicas=1
```

> For advanced traffic splitting (header-based, weight-based), use **Argo Rollouts**, **Flagger**, or a service mesh like **Istio**.

---

## 27. A node is not ready — how do you investigate?

### Step-by-step Investigation

```bash
# Step 1: Identify the node
kubectl get nodes
# Look for: NotReady, SchedulingDisabled

# Step 2: Describe the node for details
kubectl describe node <node-name>
# Look for: Conditions section (MemoryPressure, DiskPressure, PIDPressure, Ready=False)
# Look for: Events, allocatable resources, running Pods

# Step 3: Check node conditions
kubectl get node <node-name> -o jsonpath='{.status.conditions}'

# Step 4: SSH into the node (if accessible)
# Check kubelet status
systemctl status kubelet
journalctl -u kubelet -f --since "10 minutes ago"

# Check container runtime
systemctl status containerd
# or: systemctl status docker

# Check system resources
df -h              # disk space
free -h            # memory
top                # CPU

# Check system logs
journalctl -xeu kubelet
dmesg | tail -50

# Step 5: Check network connectivity
# Can the node reach the API server?
curl -k https://<apiserver-ip>:6443

# Step 6: Check certificates (common issue after cert rotation)
ls /etc/kubernetes/pki/
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -dates
```

### Common Causes

| Cause | Symptom | Fix |
|---|---|---|
| kubelet crashed | kubelet inactive | `systemctl restart kubelet` |
| Disk pressure | `DiskPressure=True` | Clean up images/logs: `docker system prune` |
| Memory pressure | `MemoryPressure=True` | Investigate and evict pods |
| Certificate expired | kubelet auth failure | Rotate certificates |
| Network partition | Node unreachable | Check network/firewall |
| Container runtime down | Runtime socket missing | Restart containerd/docker |

---

## 28. How do you handle high CPU/memory usage in a cluster?

### Immediate Actions

```bash
# 1. Identify resource-heavy nodes
kubectl top nodes

# 2. Identify resource-heavy pods
kubectl top pods --all-namespaces --sort-by=cpu
kubectl top pods --all-namespaces --sort-by=memory

# 3. Describe problematic pods
kubectl describe pod <pod-name>

# 4. Check if HPA is configured and working
kubectl get hpa --all-namespaces
kubectl describe hpa <hpa-name>
```

### Node-Level Response

```bash
# Cordon the node (prevent new pods from being scheduled)
kubectl cordon <node-name>

# Drain the node (evict all pods gracefully)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# After fixing, uncordon
kubectl uncordon <node-name>
```

### Long-Term Solutions

- **Set resource requests and limits** on all containers — prevents "noisy neighbor" issues.
- **Configure HPA** to scale out automatically under load.
- **Configure VPA (Vertical Pod Autoscaler)** to right-size resource requests.
- **Configure Cluster Autoscaler** to add/remove nodes based on cluster resource pressure.
- **Use LimitRange** to enforce default limits in namespaces.
- **Use ResourceQuota** to cap total namespace resource consumption.
- **Profile the application** to fix actual memory leaks or CPU-heavy operations.

---

## 29. How do you scale applications automatically in Kubernetes?

### 1. Horizontal Pod Autoscaler (HPA) — Scale replicas

```bash
# Quick HPA creation
kubectl autoscale deployment my-app --cpu-percent=70 --min=2 --max=10

# With custom metrics (KEDA or custom metrics API)
```

### 2. Vertical Pod Autoscaler (VPA) — Right-size resource requests

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"   # or "Off" for recommendations only
```

### 3. Cluster Autoscaler — Scale nodes

Automatically adjusts the number of nodes in a node pool when:
- Pods fail to schedule due to insufficient resources → **scale up**.
- Nodes are underutilized for an extended period → **scale down**.

```yaml
# Typically configured via cloud provider (EKS, GKE, AKS)
# AWS EKS example annotation on node group:
cluster-autoscaler.kubernetes.io/safe-to-evict: "true"
```

### 4. KEDA — Event-driven autoscaling

**KEDA (Kubernetes Event-driven Autoscaling)** extends HPA to scale based on external event sources: Kafka lag, SQS queue depth, HTTP request rate, cron schedule, etc.

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: kafka-consumer-scaler
spec:
  scaleTargetRef:
    name: kafka-consumer
  minReplicaCount: 0
  maxReplicaCount: 20
  triggers:
    - type: kafka
      metadata:
        bootstrapServers: kafka:9092
        topic: my-topic
        lagThreshold: "100"
```

---

## 30. How do you monitor and log Kubernetes workloads effectively?

### Monitoring Stack

#### Prometheus + Grafana (Industry Standard)

- **Prometheus**: Scrapes metrics from Pods, nodes, and Kubernetes components.
- **Alertmanager**: Routes alerts to Slack, PagerDuty, email, etc.
- **Grafana**: Visualizes metrics with dashboards.

```bash
# Install using kube-prometheus-stack Helm chart
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
```

Key metrics to monitor:
- Pod CPU/Memory utilization
- Node pressure (DiskPressure, MemoryPressure)
- Pod restart count
- Deployment rollout status
- API server latency and error rate

#### Expose custom metrics via annotations

```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
```

### Logging Stack

#### EFK Stack (Elasticsearch + Fluentd + Kibana)

- **Fluentd** (DaemonSet) — collects logs from every node
- **Elasticsearch** — stores and indexes logs
- **Kibana** — search and visualize logs

#### ELK Stack (Elasticsearch + Logstash + Kibana)

Similar to EFK but uses Logstash for more complex log transformation pipelines.

#### Loki + Grafana (Lightweight Alternative)

- **Loki**: Log aggregation system (only indexes metadata, not log content → much cheaper).
- **Promtail**: Agent that ships logs to Loki.
- **Grafana**: Unified dashboard for both metrics and logs.

```bash
helm install loki-stack grafana/loki-stack \
  --set grafana.enabled=true \
  --set promtail.enabled=true
```

### Distributed Tracing

Use **Jaeger** or **Zipkin** with **OpenTelemetry** instrumentation for tracing requests across microservices.

### Monitoring Best Practices

- **USE Method**: Utilization, Saturation, Errors — for resources (CPU, memory, disk, network).
- **RED Method**: Rate, Errors, Duration — for services/APIs.
- Set **meaningful alerts** (avoid alert fatigue — don't alert on symptoms, alert on symptoms impacting users).
- Use **recording rules** in Prometheus to pre-compute expensive queries.
- Retain **short-term metrics** in Prometheus (15-30 days) and long-term in Thanos or Cortex.
- Set **log retention policies** — store critical logs in cold storage (S3 + Athena).

---

*This guide covers the most critical Kubernetes concepts for both day-to-day operations and interview preparation. Always refer to the [official Kubernetes documentation](https://kubernetes.io/docs/) for the most up-to-date information.*
