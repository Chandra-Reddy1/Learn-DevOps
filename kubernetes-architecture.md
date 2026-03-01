# ☸ Kubernetes Architecture — Complete Guide
> **Control Plane · Worker Nodes · Real-Time Scenarios · Production Patterns**

---

## 📌 Table of Contents

1. [Cluster Overview](#cluster-overview)
2. [Control Plane (Master Node)](#control-plane-master-node)
   - [kube-apiserver](#1-kube-apiserver)
   - [etcd](#2-etcd)
   - [kube-scheduler](#3-kube-scheduler)
   - [controller-manager](#4-controller-manager)
   - [cloud-controller-manager](#5-cloud-controller-manager)
3. [Worker Node](#worker-node)
   - [kubelet](#1-kubelet)
   - [kube-proxy](#2-kube-proxy)
   - [Container Runtime](#3-container-runtime)
   - [Pod](#4-pod)
   - [CNI Plugin](#5-cni-plugin)
   - [CSI / Volumes](#6-csi--volumes)
4. [End-to-End Request Flow](#end-to-end-request-flow)
5. [Real-World Scenarios Deep Dive](#real-world-scenarios-deep-dive)
6. [Component Interaction Cheatsheet](#component-interaction-cheatsheet)

---

## Cluster Overview

A Kubernetes cluster is made up of two types of machines:

| Node Type | Role | Who Manages It |
|---|---|---|
| **Control Plane (Master)** | Brains of the cluster — makes all decisions | Kubernetes itself / Cloud Provider |
| **Worker Node** | Muscles of the cluster — runs your actual workloads (pods) | Control Plane |

```
┌─────────────────────────────────────────────────────────────────────┐
│                        KUBERNETES CLUSTER                           │
│                                                                     │
│  ┌─────────────────────────┐      ┌──────────┐  ┌──────────┐      │
│  │   CONTROL PLANE         │──────│ Worker-1 │  │ Worker-2 │  ... │
│  │  (Master Node)          │      └──────────┘  └──────────┘      │
│  └─────────────────────────┘                                        │
└─────────────────────────────────────────────────────────────────────┘
```

> 💡 **Key Mental Model:** The Control Plane is the **manager** (plans & decides). Worker Nodes are the **employees** (execute the plan). You talk to the manager (API server), not the employees directly.

---

## Control Plane (Master Node)

The Control Plane is the decision-making center of Kubernetes. It stores cluster state, schedules workloads, and watches for problems to fix them automatically.

```
┌──────────────────────────────────────────────────────────┐
│                   CONTROL PLANE                          │
│                                                          │
│   ┌─────────────┐   ┌──────┐   ┌────────────────────┐  │
│   │ API Server  │◄──│ etcd │   │ Controller Manager  │  │
│   │  (Gateway)  │──►│ (DB) │   │  (Autopilot Loops) │  │
│   └──────┬──────┘   └──────┘   └────────────────────┘  │
│          │                                               │
│   ┌──────┴──────────────────────────────┐               │
│   │          kube-scheduler             │               │
│   │         (Pod Placement)             │               │
│   └─────────────────────────────────────┘               │
│                                                          │
│   ┌──────────────────────────────────────┐              │
│   │      cloud-controller-manager        │              │
│   │   (AWS / GCP / Azure Integration)    │              │
│   └──────────────────────────────────────┘              │
└──────────────────────────────────────────────────────────┘
```

---

### 1. kube-apiserver

**Role:** The single front door for all operations in the cluster.

Every single command — whether from a developer, a controller, or a kubelet — goes through the API server. Nothing in Kubernetes communicates directly with each other; they all talk through the API server.

**What it does:**
- Accepts REST API calls (from `kubectl`, SDKs, CI/CD systems)
- Authenticates the caller (certificates, bearer tokens, OIDC)
- Authorizes the caller (RBAC — does this user have permission?)
- Validates the request (is the YAML schema valid?)
- Reads/writes state to `etcd`
- Sends back responses and watches (live streams of events)

---

#### 🟠 Real-Time Scenario 1 — Developer deploys a new version

```
Your team ships v2.4 of the checkout service:

$ kubectl apply -f checkout-deployment.yaml

1. kubectl sends HTTP PATCH to https://api-server:6443/apis/apps/v1/deployments/checkout
2. API Server checks your identity (JWT token from your kubeconfig)
3. API Server checks RBAC: Can this user PATCH deployments in namespace "production"? ✅
4. API Server validates the YAML schema — correct replicas, valid image name, etc.
5. API Server writes new desired state to etcd
6. API Server returns 200 OK to kubectl → you see "deployment.apps/checkout configured"
```

#### 🟠 Real-Time Scenario 2 — CI/CD pipeline via GitHub Actions

```
GitHub Actions pipeline auto-deploys on merge to main:

1. GitHub Actions runner has a kubeconfig with a ServiceAccount token
2. Runner runs: kubectl set image deployment/api api=myrepo/api:sha-abc123
3. API Server verifies the ServiceAccount token (OIDC with GitHub OIDC provider)
4. RBAC checks: ServiceAccount "github-deployer" has role "deployer" → allowed
5. Deployment image is updated in etcd
6. Rolling update begins automatically
```

---

### 2. etcd

**Role:** The cluster's single source of truth — a distributed key-value database.

`etcd` stores **everything** about your cluster: every pod, every node, every config, every secret, every service definition. If you want to know the current state of the cluster — etcd has it. Uses the **Raft consensus algorithm** to stay consistent across typically 3 or 5 replicas.

**What it stores:**
- Node registrations and health
- Pod specs and their assigned nodes
- ConfigMaps and Secrets
- Service definitions and Endpoints
- RBAC policies
- Custom Resource Definitions (CRDs)

> ⚠️ **Critical:** Back up etcd regularly. If etcd is lost and there's no backup, your cluster configuration is gone. Running workloads will continue temporarily but the cluster cannot make new decisions.

---

#### 🟡 Real-Time Scenario 1 — Cluster survives a master node restart

```
Your cloud provider briefly restarts the control plane for patching:

1. etcd (3 replicas) maintains quorum — 2 of 3 nodes are still up
2. API server restarts and reconnects to etcd — state is fully intact
3. All running pods on worker nodes are UNAFFECTED (they don't need master to keep running)
4. New deployments/scaling are paused only during the restart window (<2 min)
5. API server comes back, reads etcd, resumes all watches and reconciliation loops
```

#### 🟡 Real-Time Scenario 2 — Disaster recovery from etcd backup

```
A misconfigured script deletes the entire "production" namespace:

$ kubectl delete namespace production   # OOPS! 500 pods gone

Recovery steps:
1. Fetch latest etcd snapshot from S3 (automated nightly backup)
2. Restore: etcdctl snapshot restore backup.db --data-dir /var/lib/etcd-restored
3. Restart API server pointing to restored etcd
4. All 500 pod specs, configmaps, secrets restored — Kubernetes recreates everything
5. Downtime: ~8 minutes from snapshot restoration to pods Running
```

---

### 3. kube-scheduler

**Role:** Decides which worker node gets each new pod.

When a pod is created but has no assigned node, it enters a "Pending" state. The scheduler watches for these unbound pods, scores available nodes, and assigns (binds) the pod to the best node. It writes this binding back via the API server.

**Two-phase process:**

| Phase | Name | What Happens |
|---|---|---|
| Phase 1 | **Filtering** | Remove nodes that don't meet hard requirements (not enough CPU/RAM, wrong nodeSelector, has a Taint the pod doesn't tolerate) |
| Phase 2 | **Scoring** | Score remaining nodes 0–100 on multiple criteria, pick the winner |

**Scoring factors include:**
- Available CPU and memory
- Pod affinity/anti-affinity rules
- Data locality (is the PersistentVolume in this zone?)
- Pod spread (don't pile all replicas on one node)
- Custom scheduler plugins (you can write your own!)

---

#### 🔵 Real-Time Scenario 1 — Auto-scaling during traffic spike

```
HPA (Horizontal Pod Autoscaler) detects CPU > 80% — scales from 3 → 8 pods:

Scheduler sees 5 new unbound pods. Available nodes:
  Worker-01: 1.2 vCPU free, 2GB RAM free  → Filtered IN (meets 0.5 CPU request)
  Worker-02: 3.0 vCPU free, 8GB RAM free  → Filtered IN
  Worker-03: 0.1 vCPU free               → Filtered OUT (insufficient CPU)

Scoring:
  Worker-01 scores 42 (less free resources)
  Worker-02 scores 91 (more free resources)

Result: 4 pods → Worker-02, 1 pod → Worker-01 (to spread load)
All 5 pods scheduled in ~50ms total.
```

#### 🔵 Real-Time Scenario 2 — Topology spread for high availability

```
Your payment-service has this rule:
  topologySpreadConstraints:
    maxSkew: 1
    topologyKey: topology.kubernetes.io/zone

This forces pods to spread across AWS availability zones:

Before: 3 pods → us-east-1a (DANGER: one AZ outage = full outage)
After scheduler enforces spread:
  2 pods → us-east-1a
  2 pods → us-east-1b  
  2 pods → us-east-1c
  
Result: Any single AZ can die — 4 pods survive. Payment service stays up. ✅
```

---

### 4. controller-manager

**Role:** Runs all the built-in control loops that keep your cluster healthy.

This is Kubernetes' **self-healing engine**. It runs multiple controllers in a single binary, each watching a specific resource type. Every controller follows the same pattern: **observe current state → compare to desired state → take action to reconcile them**.

**Key controllers inside it:**

| Controller | Watches | Action Taken |
|---|---|---|
| **ReplicaSet Controller** | ReplicaSets | Creates/deletes pods to match replica count |
| **Deployment Controller** | Deployments | Manages rolling updates and rollbacks |
| **Node Controller** | Nodes | Marks nodes NotReady, evicts pods from dead nodes |
| **Job Controller** | Jobs | Runs pods to completion, retries on failure |
| **CronJob Controller** | CronJobs | Creates Jobs on a schedule |
| **Namespace Controller** | Namespaces | Cleans up resources when namespace is deleted |
| **ServiceAccount Controller** | ServiceAccounts | Creates default SAs in new namespaces |

---

#### 🟢 Real-Time Scenario 1 — Pod crash self-healing

```
3AM: Your order-service pod crashes due to OOMKilled (out of memory):

T+0s   Pod status changes to "Failed" in etcd
T+1s   ReplicaSet controller detects: desired=3 replicas, actual=2
T+1s   Controller creates a new Pod spec, writes to etcd via API server
T+2s   Scheduler binds new pod to Worker-02
T+3s   kubelet on Worker-02 pulls image (already cached), starts container
T+8s   Pod enters "Running" state, readiness probe passes
T+10s  Pod added back to Service endpoints, traffic flows to it

Total downtime for this pod: 10 seconds. No pager alert needed.
```

#### 🟢 Real-Time Scenario 2 — Rolling deployment update

```
You deploy v2.5 of the user-service (5 replicas, maxSurge=1, maxUnavailable=1):

T+0s   Deployment controller detects new image: v2.4 → v2.5
T+0s   Creates 1 new v2.5 pod (surge) → 6 pods total
T+15s  New v2.5 pod passes readiness probe
T+15s  Kills 1 old v2.4 pod → back to 5 pods
T+30s  Creates another v2.5 pod, kills another v2.4 pod
...continues rolling...
T+90s  All 5 pods running v2.5. Zero downtime. Users never noticed.

If v2.5 crashes during rollout → Deployment controller stops the rollout.
$ kubectl rollout undo deployment/user-service  # Back to v2.4 in 90 seconds
```

#### 🟢 Real-Time Scenario 3 — Node failure eviction

```
Worker-02 loses network connectivity at 2AM:

T+0s   Node heartbeat stops reaching API server
T+40s  Node controller marks Worker-02 as "NotReady"  
T+5m   Node controller starts evicting all pods from Worker-02
T+5m   ReplicaSet controller detects missing pods, creates replacements
T+6m   Scheduler places replacement pods on Worker-01 and Worker-03
T+7m   All evicted services are running again elsewhere

Engineers wake up to a Slack alert, not a P0 incident. ✅
```

---

### 5. cloud-controller-manager

**Role:** Bridges Kubernetes with your cloud provider's APIs.

Separates cloud-specific logic from core Kubernetes. Handles anything that requires making API calls to AWS, GCP, or Azure.

**What it manages:**
- **Load Balancers** — creates AWS ALB/NLB when you create `type: LoadBalancer` Services
- **Cloud Storage** — dynamically provisions EBS volumes, GCE Persistent Disks, Azure Disks
- **Node registration** — adds cloud metadata (instance type, zone, region) to Node objects
- **Route management** — updates cloud VPC routes when new nodes join

---

#### ☁️ Real-Time Scenario 1 — Auto-provisioned load balancer

```
New microservice needs public traffic:

$ kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: store-frontend
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
    - port: 443
      targetPort: 8080
EOF

Cloud Controller Manager:
1. Detects new Service of type LoadBalancer
2. Calls AWS API: create Application Load Balancer in VPC
3. Configures target group pointing to all 3 worker nodes on port 30080
4. Requests ACM certificate for TLS termination
5. Writes external IP back: kubectl get svc store-frontend
   → EXTERNAL-IP: a1b2c3d4.us-east-1.elb.amazonaws.com

Done. No AWS console needed. Entire process: ~45 seconds.
```

---

## Worker Node

Each worker node runs the actual application containers. A cluster can have anywhere from 1 to thousands of worker nodes. Each node has three core components.

```
┌──────────────────────────────────────────────────────────────┐
│                        WORKER NODE                           │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   kubelet    │  │  kube-proxy  │  │ Container Runtime│  │
│  │ (Node Agent) │  │  (Networking)│  │   (containerd)   │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                        PODS                          │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │   │
│  │  │   Pod A     │  │   Pod B     │  │   Pod C     │  │   │
│  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │  │   │
│  │  │ │Container│ │  │ │Container│ │  │ │Container│ │  │   │
│  │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

---

### 1. kubelet

**Role:** The primary agent on every worker node. Ensures containers described by pod specs are running and healthy.

The kubelet is how the control plane actually **does things on the node**. It watches the API server for pods assigned to its node, then tells the container runtime to start them. It also continuously runs health checks and reports status back up.

**What it does:**
- Registers the node with the API server on startup
- Watches for pods assigned to this node (via informer/watch)
- Calls the Container Runtime Interface (CRI) to start/stop containers
- Runs liveness, readiness, and startup probes
- Reports pod status back to the API server (Running, CrashLoopBackOff, OOMKilled, etc.)
- Manages volume mounts and secrets/configmap injection
- Enforces resource limits via cgroups

---

#### 🤖 Real-Time Scenario 1 — Liveness probe auto-restart

```
Your Node.js API has a memory leak. After ~2 hours it stops responding:

Liveness probe config:
  livenessProbe:
    httpGet:
      path: /healthz
      port: 3000
    failureThreshold: 3
    periodSeconds: 10

Timeline:
T+0m    Memory leak starts
T+120m  App stops responding to /healthz
T+120m  kubelet: Probe 1 FAILED (HTTP 502)
T+121m  kubelet: Probe 2 FAILED
T+122m  kubelet: Probe 3 FAILED → threshold reached
T+122m  kubelet kills container (SIGTERM → SIGKILL after 30s)
T+123m  kubelet starts fresh container (restart policy: Always)
T+124m  Container healthy again, serving traffic

Engineers never paged. Users saw at most ~2 min degradation every 2 hours
(until the memory leak is actually fixed in code).
```

#### 🤖 Real-Time Scenario 2 — Readiness probe traffic gating

```
New pod starts but the app takes 30 seconds to warm up (load cache, DB connections):

Readiness probe:
  readinessProbe:
    httpGet:
      path: /ready
      port: 8080
    initialDelaySeconds: 10
    periodSeconds: 5

Timeline:
T+0s   Container starts
T+10s  kubelet starts checking /ready → returns 503 (still loading)
T+15s  Still 503 — kubelet does NOT add pod to Service endpoints
T+30s  App finishes loading → /ready returns 200
T+35s  kubelet reports pod as Ready → kube-proxy adds it to Service
T+35s  Traffic starts flowing to new pod

Zero bad requests sent to a pod that wasn't ready. ✅
```

---

### 2. kube-proxy

**Role:** Implements the Kubernetes Service abstraction through network rules.

Every time you create a Kubernetes Service, kube-proxy translates that into actual network rules (iptables or IPVS) on every node so that traffic can reach the right pods — regardless of which node the caller is on.

**Three Service modes it handles:**

| Service Type | What kube-proxy does |
|---|---|
| `ClusterIP` | Creates iptables rules to DNAT ClusterIP → one of the pod IPs |
| `NodePort` | Opens a port on every node, forwards to pods |
| `LoadBalancer` | NodePort + tells cloud controller to create LB in front |

---

#### 🌐 Real-Time Scenario 1 — Service discovery between microservices

```
frontend-pod (10.244.1.5) calls http://auth-service:8080/validate

Step by step:
1. DNS lookup: auth-service → resolves to ClusterIP 10.96.45.120
2. Packet leaves frontend-pod heading to 10.96.45.120:8080
3. iptables rule on Worker-01 (installed by kube-proxy) intercepts:
   "If destination is 10.96.45.120, randomly pick one of these real pod IPs"
4. Rule selects 10.244.3.8 (auth-service pod on Worker-03)
5. Packet forwarded to Worker-03, delivered to pod
6. Response comes back through same path

auth-service currently has 4 pods — kube-proxy load balances randomly across all 4.
If one pod fails and is removed from Endpoints, kube-proxy updates rules within ~5 seconds.
```

#### 🌐 Real-Time Scenario 2 — NodePort for external staging traffic

```
QA team needs external access to staging app without a load balancer:

kind: Service
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 31080

kube-proxy opens port 31080 on ALL worker nodes:
  http://worker-01-ip:31080  → routes to app pods
  http://worker-02-ip:31080  → routes to app pods
  http://worker-03-ip:31080  → routes to app pods

QA uses: http://worker-01-ip:31080
Even if the app pod is running on Worker-03, kube-proxy on Worker-01
proxies the traffic to the right pod transparently. ✅
```

---

### 3. Container Runtime

**Role:** The actual engine that runs containers on the node.

Kubernetes uses the **Container Runtime Interface (CRI)** — a standard API — so it can work with any compliant runtime. The most common are `containerd` (default in EKS, GKE, AKS) and `CRI-O` (common in OpenShift).

**What it handles:**
- Pulling container images from registries (Docker Hub, ECR, GCR, ACR)
- Storing image layers using overlayFS (shared base layers = less disk usage)
- Setting up Linux namespaces (PID, network, mount, UTS) per container
- Enforcing CPU/memory limits via cgroups
- Applying seccomp and AppArmor security profiles
- Managing container lifecycle: create → start → stop → remove

---

#### 📦 Real-Time Scenario 1 — Efficient image layering

```
Your app image: myapp:v3.1 (total: 800MB)
  Layer 1: ubuntu:22.04           (80MB)   ← shared by 10 other images on this node
  Layer 2: python:3.11            (120MB)  ← shared by 5 other images
  Layer 3: pip install -r req.txt (500MB)  ← shared between v3.0 and v3.1
  Layer 4: COPY app/ /app/        (100MB)  ← unique to v3.1

When deploying v3.1 (v3.0 was already on this node):
  containerd only downloads Layer 4 → 100MB instead of 800MB
  Pull time: 8 seconds instead of 65 seconds
  Disk saved: 700MB per node across the cluster
```

#### 📦 Real-Time Scenario 2 — Resource limit enforcement

```
A bug in a data processing job causes a memory leak:

Pod spec:
  resources:
    requests: { memory: "512Mi", cpu: "500m" }
    limits:   { memory: "1Gi",  cpu: "1" }

Timeline:
T+0m    Job starts, using 300MB
T+5m    Memory leak kicks in, usage climbs to 900MB
T+8m    Usage hits 1024MB (limit)
T+8m    Linux kernel OOMKiller (triggered by cgroup limit) kills the container
T+8m    containerd reports exit code 137 (OOMKilled) to kubelet
T+8m    kubelet restarts container (RestartPolicy: OnFailure)
T+9m    Kubernetes event: "OOMKilled" visible in kubectl describe pod

Other pods on the same node are completely unaffected.
The faulty pod cannot exceed its cgroup memory ceiling. ✅
```

---

### 4. Pod

**Role:** The smallest deployable unit in Kubernetes — a group of one or more containers.

Containers inside a pod share the same network namespace (same IP, same port space) and can communicate via `localhost`. They can also share storage volumes.

**Key characteristics:**
- Every pod gets its own IP address
- Pods are ephemeral — they're replaced, not restarted in-place
- Containers in the same pod can share volumes
- Pods are scheduled as a single unit — always land on the same node

---

#### 🫙 Real-Time Scenario 1 — Sidecar pattern (Istio service mesh)

```
Your payment-service pod has 2 containers:

Pod: payment-service-7d9f8b-xk2p
  Container 1: payment-app (your code)     → port 8080
  Container 2: envoy-proxy (Istio sidecar) → port 15001

Traffic flow into the pod:
  External → Envoy (port 15001)
              ↓ (mutual TLS verification)
              ↓ (request logging)
              ↓ (circuit breaker check)
           → App (localhost:8080)

Your app code has ZERO knowledge of mTLS, tracing, or traffic policies.
Envoy handles all of that as a sidecar in the same pod.
Shared network namespace = zero-cost localhost communication.
```

#### 🫙 Real-Time Scenario 2 — Init container for database migrations

```
Before your API pod starts, you need to run DB migrations safely:

Pod spec:
  initContainers:
    - name: run-migrations
      image: myapp:v3.1
      command: ["python", "manage.py", "migrate"]
  containers:
    - name: api
      image: myapp:v3.1

Kubernetes guarantees:
  1. run-migrations starts first, runs DB migrations
  2. If migration fails → Pod stays Pending, main app NEVER starts
  3. If migration succeeds → main api container starts
  4. With 5 replicas deploying, only ONE migration pod runs at a time

Zero risk of running a new app version against an un-migrated database. ✅
```

---

### 5. CNI Plugin

**Role:** Provides network connectivity for pods — assigns IPs and routes traffic between pods across nodes.

Kubernetes defines the Container Network Interface (CNI) standard but doesn't include a network implementation. You choose a plugin: **Flannel** (simple, VXLAN overlay), **Calico** (BGP routing, NetworkPolicies), or **Cilium** (eBPF-based, highest performance).

---

#### 🔌 Real-Time Scenario 1 — Calico NetworkPolicy (Zero Trust networking)

```
Before NetworkPolicy: Any pod can talk to any pod (open cluster)
                      → A compromised pod can reach your database!

With Calico NetworkPolicy:
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: allow-only-api
    namespace: production
  spec:
    podSelector:
      matchLabels:
        app: postgres
    ingress:
      - from:
          - podSelector:
              matchLabels:
                app: api-service

Result: Only api-service pods can reach postgres pods.
Frontend, ML workers, or compromised pods are BLOCKED at the kernel level.
This is microsegmentation — like a firewall between every service.
```

#### 🔌 Real-Time Scenario 2 — Cilium eBPF performance

```
High-frequency trading app: 100,000 requests/second between microservices

Old setup (kube-proxy + iptables):
  Every packet traverses a chain of 5,000+ iptables rules (at 500 services × 10 rules each)
  Latency: 2.5ms per service call
  CPU overhead: 8 cores just for network processing

With Cilium (eBPF):
  Packet is intercepted at kernel level before it even hits iptables
  Direct socket handoff — packet never leaves kernel space
  Latency: 0.4ms per service call
  CPU overhead: 1.5 cores for network processing

Result: 6x latency improvement, 5x CPU savings.
At 100K RPS, this matters enormously.
```

---

### 6. CSI / Volumes

**Role:** Provides persistent storage that survives pod restarts and rescheduling.

Containers are ephemeral — when a pod dies, its filesystem dies with it. For databases or any stateful app, you need storage that persists independently of the pod lifecycle. The Container Storage Interface (CSI) allows cloud storage drivers to plug into Kubernetes.

**Key concepts:**

| Concept | What It Is |
|---|---|
| `PersistentVolume (PV)` | Actual storage resource (e.g., 100GB EBS volume) |
| `PersistentVolumeClaim (PVC)` | Pod's request for storage ("I need 100GB SSD") |
| `StorageClass` | Template for dynamic provisioning ("create EBS gp3 volumes on demand") |
| `CSI Driver` | Plugin that talks to cloud APIs to create/attach/detach volumes |

---

#### 💽 Real-Time Scenario 1 — PostgreSQL surviving pod rescheduling

```
postgres-0 pod is running on Worker-02 with a 500GB EBS volume:

Worker-02 goes offline for maintenance:
T+0m   Worker-02 marked NotReady
T+5m   Kubernetes evicts postgres-0
T+5m   EBS CSI driver detaches volume from Worker-02's EC2 instance
T+6m   Scheduler places postgres-0 on Worker-03
T+6m   EBS CSI driver attaches the SAME 500GB volume to Worker-03
T+7m   Volume mounted at /var/lib/postgresql/data on Worker-03
T+8m   postgres-0 starts, reads its data from the same volume

500GB of data survived the node failure with ZERO data loss.
The database sees no corruption — it's the same disk, same files.
```

#### 💽 Real-Time Scenario 2 — Dynamic volume provisioning for StatefulSet

```
You deploy a Kafka StatefulSet with 3 brokers:

spec:
  volumeClaimTemplates:
    - metadata:
        name: kafka-data
      spec:
        storageClassName: gp3-encrypted
        accessModes: [ReadWriteOnce]
        resources:
          requests:
            storage: 1Ti  # 1 Terabyte

Kubernetes automatically:
1. Creates 3 PVCs: kafka-data-kafka-0, kafka-data-kafka-1, kafka-data-kafka-2
2. EBS CSI driver provisions 3 × 1TB gp3 EBS volumes in AWS
3. Each Kafka broker gets its own dedicated 1TB volume
4. Volumes are encrypted at rest (StorageClass has encryptionKey set)

Zero manual AWS console work. Three encrypted terabyte volumes in 2 minutes. ✅
```

---

## End-to-End Request Flow

Here's what happens from `kubectl apply` to a pod serving traffic:

```
kubectl apply -f deployment.yaml
        │
        ▼
┌─────────────────┐
│  1. API Server  │  Authenticate → Authorize → Validate → Write to etcd
└────────┬────────┘
         │ etcd write complete
         ▼
┌─────────────────┐
│    2. etcd      │  Desired state persisted: "3 replicas of nginx:1.25"
└────────┬────────┘
         │ controller watches etcd via API Server
         ▼
┌─────────────────────────┐
│  3. Controller Manager  │  ReplicaSet controller: 0 pods exist, need 3 → creates 3 Pod objects
└────────┬────────────────┘
         │ 3 pods are Pending (no node assigned)
         ▼
┌─────────────────────┐
│   4. Scheduler      │  Filter nodes → Score nodes → Bind pods to nodes
└────────┬────────────┘
         │ Pod objects updated with nodeName field
         ▼
┌──────────────────────────┐
│   5. kubelet (Worker)    │  Detects pod assigned to its node → calls containerd
└────────┬─────────────────┘
         │
         ▼
┌───────────────────────────────┐
│   6. Container Runtime        │  Pull image → Create namespace → Start container
└────────┬──────────────────────┘
         │ Container is running
         ▼
┌─────────────────────────────┐
│   7. kubelet reports status  │  Pod status → Running, readiness probe passes
└────────┬────────────────────┘
         │
         ▼
┌────────────────────────┐
│   8. kube-proxy        │  Updates iptables: add pod IP to Service endpoint rules
└────────┬───────────────┘
         │
         ▼
    🚀 Pod is live and serving traffic
```

---

## Real-World Scenarios Deep Dive

### Scenario A — Full E-Commerce Black Friday Scale-Up

```
8:00 AM  Traffic starts spiking. CPU across checkout-service pods hits 75%.

8:01 AM  HPA (Horizontal Pod Autoscaler) reads CPU metrics from metrics-server.
         Desired replicas = ceil(5 × 0.75/0.50) = 8 replicas. Scaling 5 → 8.

8:01 AM  Deployment controller creates 3 new pod specs in etcd.

8:01 AM  Scheduler: 
           New pod 1 → Worker-02 (most free CPU)
           New pod 2 → Worker-04 (second most free)
           New pod 3 → Worker-01 (spread constraint)

8:02 AM  kubelet on each worker: pulls checkout:v4.2 image (already cached!)
         Containers start in 4 seconds.

8:02 AM  Readiness probes pass. kube-proxy adds new pod IPs to Service.

8:02 AM  8 pods handling traffic. CPU back down to ~47%.

1:00 PM  Traffic normalizes. HPA scales back down to 5. Cluster cost optimized.
```

---

### Scenario B — Zero-Downtime Rolling Deployment

```
Team deploys v5.0 of the user-service during business hours.
Config: 10 replicas, maxSurge=2, maxUnavailable=0

This means: Always have 10 healthy pods. Can go up to 12 temporarily.

T+0s    Deployment controller creates 2 new v5.0 pods (surge to 12 total)
T+30s   2 new v5.0 pods pass readiness probes
T+30s   2 old v4.9 pods terminated (back to 10, but 2 are v5.0)
T+60s   2 more new v5.0 pods created, 2 more old terminated
...
T+3m    All 10 pods running v5.0. Zero downtime. Zero dropped requests.

If v5.0 introduces a bug causing crashes:
T+0s    v5.0 pods CrashLoopBackOff
T+10s   Deployment controller pauses rollout (progress deadline exceeded)
T+10s   Alerting fires: "Deployment checkout-v5.0 not progressing"
T+15s   kubectl rollout undo deployment/user-service
T+3m    All pods back on v4.9. Incident resolved.
```

---

### Scenario C — Security Breach Containment

```
A pod in the "frontend" namespace has a vulnerability and gets compromised.

Without any policies: The attacker can reach your database from the compromised pod.

With Kubernetes security layers:

1. NetworkPolicy (Calico):
   frontend pods CANNOT reach the database namespace.
   Attacker's lateral movement is stopped at layer 3.

2. RBAC:
   The compromised pod's ServiceAccount has NO permissions to list secrets.
   kubectl get secrets → "Forbidden". Cannot steal credentials.

3. Pod Security Admission:
   All pods run as non-root (runAsNonUser: true).
   readOnlyRootFilesystem: true — attacker cannot install tools.
   No privilege escalation allowed.

4. Seccomp profile:
   The compromised container cannot make dangerous syscalls
   (ptrace, mount, chmod /etc/shadow, etc.)

Result: Attacker is trapped inside one pod, cannot access data,
cannot escalate, cannot spread. Blast radius: 1 pod.
```

---

### Scenario D — Multi-Tenant Namespace Isolation

```
SaaS platform: 50 customers sharing one Kubernetes cluster.

Each customer gets:
  - Dedicated namespace: customer-acme, customer-globex, customer-initech
  - ResourceQuota: max 10 CPU, 20GB RAM, 50 pods
  - LimitRange: each pod max 2 CPU, 4GB RAM
  - NetworkPolicy: no cross-namespace pod communication
  - RBAC: customer users can only see their namespace

Customer ACME deploys a bad app that tries to consume all cluster resources:
  kubectl run stress --image=stress-ng -- --cpu 100 --vm 50

LimitRange kicks in: pod request capped at 2 CPU / 4GB
ResourceQuota kicks in: after 5 such pods, quota exceeded → new pods rejected

ACME cannot affect GLOBEX's workloads in any way. ✅
Cluster admin sees: kubectl describe quota -n customer-acme
                    Used CPU: 10/10 ← at limit, ACME can't do more
```

---

## Component Interaction Cheatsheet

```
WHO TALKS TO WHOM:

kubectl           → kube-apiserver    (all user operations)
kube-apiserver    → etcd              (read/write cluster state)
kube-scheduler    → kube-apiserver    (watch unbound pods, write bindings)
controller-manager→ kube-apiserver    (watch resources, create/update/delete objects)
kubelet           → kube-apiserver    (register node, watch pod specs, report status)
kubelet           → container runtime  (CRI: create/start/stop/delete containers)
kubelet           → CNI plugin         (set up pod networking)
kubelet           → CSI driver         (mount/unmount volumes)
kube-proxy        → kube-apiserver    (watch Services and Endpoints)
kube-proxy        → iptables/ipvs     (write network rules on the node)
cloud-controller  → cloud provider API (provision LBs, volumes, routes)

NOTHING talks directly to etcd except kube-apiserver. ✅
```

---

```
QUICK COMPONENT SUMMARY:

┌──────────────────────┬────────────────┬──────────────────────────────┐
│ Component            │ Lives On       │ One-line job                 │
├──────────────────────┼────────────────┼──────────────────────────────┤
│ kube-apiserver       │ Control Plane  │ REST gateway + auth          │
│ etcd                 │ Control Plane  │ The cluster's database       │
│ kube-scheduler       │ Control Plane  │ Decide where pods run        │
│ controller-manager   │ Control Plane  │ Keep desired == actual state │
│ cloud-controller     │ Control Plane  │ Talk to AWS/GCP/Azure        │
│ kubelet              │ Worker Node    │ Run & monitor pods on node   │
│ kube-proxy           │ Worker Node    │ Route Service traffic        │
│ container runtime    │ Worker Node    │ Actually run the containers  │
│ CNI plugin           │ Worker Node    │ Pod networking + IPs         │
│ CSI driver           │ Worker Node    │ Persistent storage           │
└──────────────────────┴────────────────┴──────────────────────────────┘
```

---

*Last updated: 2026 · Kubernetes v1.30+ · For questions, refer to [kubernetes.io/docs](https://kubernetes.io/docs)*
