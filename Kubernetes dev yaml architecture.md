# Kubernetes Architecture: End-to-End Flow of Injecting `dev.yaml`
### Python Sign-In Portal Deployment

---

## 📌 Overview

This document traces **every stage** that occurs when you apply a `dev.yaml` manifest to a Kubernetes cluster — from your terminal command all the way to a running Pod serving your Python sign-in portal.

---

## 🗂️ Sample `dev.yaml` — Python Sign-In Portal

```yaml
# dev.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: signin-portal
  namespace: dev
  labels:
    app: signin-portal
    env: dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: signin-portal
  template:
    metadata:
      labels:
        app: signin-portal
    spec:
      containers:
        - name: signin-app
          image: myregistry/signin-portal:v1.0
          ports:
            - containerPort: 8000
          env:
            - name: DB_HOST
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: host
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 15
            periodSeconds: 20
---
apiVersion: v1
kind: Service
metadata:
  name: signin-portal-svc
  namespace: dev
spec:
  selector:
    app: signin-portal
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: signin-portal-ingress
  namespace: dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: signin.dev.myapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: signin-portal-svc
                port:
                  number: 80
```

---

## 🏗️ Full Kubernetes Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          KUBERNETES CLUSTER ARCHITECTURE                            │
│                                                                                     │
│  ┌──────────────────────────────────────────────────────────────────────────────┐   │
│  │                         CONTROL PLANE (Master Node)                          │   │
│  │                                                                              │   │
│  │  ┌─────────────────┐  ┌──────────────────┐  ┌──────────────────────────┐    │   │
│  │  │   API Server    │  │  Scheduler       │  │  Controller Manager      │    │   │
│  │  │  (kube-apiserver│  │ (kube-scheduler) │  │ (kube-controller-manager)│    │   │
│  │  │                 │  │                  │  │                          │    │   │
│  │  │ • REST endpoint │  │ • Watches for    │  │ • Deployment Controller  │    │   │
│  │  │ • Auth/AuthZ    │  │   unscheduled    │  │ • ReplicaSet Controller  │    │   │
│  │  │ • Validation    │  │   Pods           │  │ • Node Controller        │    │   │
│  │  │ • Admission     │  │ • Selects best   │  │ • Service Account Ctrl   │    │   │
│  │  │   Webhooks      │  │   Node           │  │ • Endpoints Controller   │    │   │
│  │  │ • etcd gateway  │  │ • Resource fit   │  │                          │    │   │
│  │  └────────┬────────┘  └────────┬─────────┘  └──────────────────────────┘    │   │
│  │           │                    │                         │                   │   │
│  │           ▼                    │                         │                   │   │
│  │  ┌─────────────────┐           │                         │                   │   │
│  │  │      etcd       │◄──────────┴─────────────────────────┘                   │   │
│  │  │  (Key-Value DB) │                                                          │   │
│  │  │                 │                                                          │   │
│  │  │ • Cluster state │                                                          │   │
│  │  │ • All manifests │                                                          │   │
│  │  │ • Secrets       │                                                          │   │
│  │  │ • ConfigMaps    │                                                          │   │
│  │  └─────────────────┘                                                          │   │
│  └──────────────────────────────────────────────────────────────────────────────┘   │
│                                        │                                             │
│                          Kubelet watch loop (each node)                             │
│                                        │                                             │
│  ┌─────────────────────────────────────▼────────────────────────────────────────┐   │
│  │                            WORKER NODE 1                                     │   │
│  │                                                                              │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────────────────┐  │   │
│  │  │   Kubelet    │  │  Kube-proxy  │  │        Container Runtime           │  │   │
│  │  │              │  │              │  │       (containerd / CRI-O)         │  │   │
│  │  │ • Manages    │  │ • IPTables/  │  │                                    │  │   │
│  │  │   pod        │  │   IPVS rules │  │ • Pulls image from registry        │  │   │
│  │  │   lifecycle  │  │ • Service    │  │ • Creates containers               │  │   │
│  │  │ • Health     │  │   routing    │  │ • Manages namespaces               │  │   │
│  │  │   probes     │  │ • Load       │  │ • cgroups resource limiting        │  │   │
│  │  │ • Reports to │  │   balancing  │  │                                    │  │   │
│  │  │   API server │  │   within     │  └──────────────────┬─────────────────┘  │   │
│  │  └──────────────┘  │   cluster    │                     │                    │   │
│  │                    └──────────────┘                     ▼                    │   │
│  │                                        ┌────────────────────────────────┐    │   │
│  │                                        │         POD: signin-portal-1   │    │   │
│  │                                        │  ┌──────────────────────────┐  │    │   │
│  │                                        │  │  Container: signin-app   │  │    │   │
│  │                                        │  │  Image: signin-portal:v1 │  │    │   │
│  │                                        │  │  Port: 8000 (gunicorn)   │  │    │   │
│  │                                        │  │  Python Flask/Django App  │  │    │   │
│  │                                        │  └──────────────────────────┘  │    │   │
│  │                                        │  IP: 10.244.1.5                │    │   │
│  │                                        └────────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                     │
│  ┌──────────────────────────────────────────────────────────────────────────────┐   │
│  │                            WORKER NODE 2                                     │   │
│  │                                                                              │   │
│  │                                        ┌────────────────────────────────┐    │   │
│  │                                        │         POD: signin-portal-2   │    │   │
│  │                                        │  ┌──────────────────────────┐  │    │   │
│  │                                        │  │  Container: signin-app   │  │    │   │
│  │                                        │  │  Image: signin-portal:v1 │  │    │   │
│  │                                        │  │  Port: 8000              │  │    │   │
│  │                                        │  └──────────────────────────┘  │    │   │
│  │                                        │  IP: 10.244.2.7                │    │   │
│  │                                        └────────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                     │
│  ┌──────────────────────────────────────────────────────────────────────────────┐   │
│  │                         NETWORKING LAYER                                     │   │
│  │                                                                              │   │
│  │  ┌─────────────────────────┐        ┌─────────────────────────────────────┐  │   │
│  │  │    Ingress Controller   │        │       Service (ClusterIP)           │  │   │
│  │  │   (NGINX)               │        │      signin-portal-svc              │  │   │
│  │  │                         │───────►│                                     │  │   │
│  │  │  signin.dev.myapp.com   │        │  Port: 80 → TargetPort: 8000        │  │   │
│  │  │  path: /                │        │  Selector: app=signin-portal        │  │   │
│  │  └─────────────────────────┘        └──────────────┬──────────────────────┘  │   │
│  │                                                    │                          │   │
│  │                                     ┌──────────────▼──────────────┐           │   │
│  │                                     │   Endpoints Object          │           │   │
│  │                                     │   10.244.1.5:8000           │           │   │
│  │                                     │   10.244.2.7:8000           │           │   │
│  │                                     └─────────────────────────────┘           │   │
│  └──────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Stage-by-Stage Flow: `kubectl apply -f dev.yaml`

```
╔══════════════════════════════════════════════════════════════════════════════════════╗
║              COMPLETE FLOW: kubectl apply -f dev.yaml                               ║
╚══════════════════════════════════════════════════════════════════════════════════════╝

 Developer Machine
 ┌──────────────┐
 │  dev.yaml    │
 │  (manifest)  │
 └──────┬───────┘
        │  kubectl apply -f dev.yaml
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 1: kubectl CLI Processing                     ║
╠══════════════════════════════════════════════════════╣
║  • Reads and parses dev.yaml                         ║
║  • Resolves kubeconfig (~/.kube/config)              ║
║  • Identifies cluster endpoint                       ║
║  • Sets up TLS/mTLS connection                       ║
║  • Sends HTTP PATCH/POST to API Server               ║
║                                                      ║
║  Request:                                            ║
║  POST /apis/apps/v1/namespaces/dev/deployments       ║
║  Authorization: Bearer <token>                       ║
╚══════════════════════════════════════════════════════╝
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 2: API Server — Authentication               ║
╠══════════════════════════════════════════════════════╣
║  • Validates Bearer Token / Client Certificate       ║
║  • Checks ServiceAccount / OIDC provider            ║
║  • If invalid → 401 Unauthorized                    ║
║  • If valid → proceed to Authorization              ║
╚══════════════════════════════════════════════════════╝
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 3: API Server — Authorization (RBAC)         ║
╠══════════════════════════════════════════════════════╣
║  • Checks RBAC policies                             ║
║  • Can user CREATE Deployments in namespace "dev"?  ║
║  • Can user CREATE Services?                        ║
║  • Can user CREATE Ingress?                         ║
║  • If denied → 403 Forbidden                        ║
║  • If allowed → proceed to Admission                ║
╚══════════════════════════════════════════════════════╝
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 4: Admission Controllers                     ║
╠══════════════════════════════════════════════════════╣
║                                                      ║
║  [MUTATING ADMISSION WEBHOOKS]                       ║
║  • Inject sidecar containers (e.g., Istio)          ║
║  • Add default resource limits                      ║
║  • Inject environment variables                     ║
║  • Label injection                                  ║
║  • Mutates the object → returns modified manifest   ║
║                                                      ║
║  [VALIDATING ADMISSION WEBHOOKS]                    ║
║  • PodSecurityPolicy / OPA Gatekeeper checks        ║
║  • Image policy enforcement (allowed registries)    ║
║  • Namespace quota checks                           ║
║  • LimitRanger (resource limits must be set)        ║
║  • If rejected → 422 Unprocessable Entity           ║
╚══════════════════════════════════════════════════════╝
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 5: Schema Validation                         ║
╠══════════════════════════════════════════════════════╣
║  • Object validated against OpenAPI schema          ║
║  • Required fields present?                         ║
║  • Field types correct?                             ║
║  • apiVersion/kind recognized?                      ║
╚══════════════════════════════════════════════════════╝
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 6: Persist to etcd                           ║
╠══════════════════════════════════════════════════════╣
║  • Serializes object to protobuf                    ║
║  • Writes to etcd key-value store                   ║
║  Key:                                               ║
║  /registry/apps/deployments/dev/signin-portal       ║
║                                                     ║
║  • etcd confirms write (Raft consensus)             ║
║  • API Server responds 201 Created to kubectl       ║
╚══════════════════════════════════════════════════════╝
        │
        ├──────────────────────────┐
        ▼                          ▼
╔══════════════════════╗  ╔═══════════════════════════╗
║  STAGE 7A:           ║  ║  STAGE 7B:                ║
║  Deployment          ║  ║  Service Controller        ║
║  Controller          ║  ║                           ║
╠══════════════════════╣  ╠═══════════════════════════╣
║  • Watches etcd for  ║  ║  • Detects new Service    ║
║    Deployment events ║  ║    object (ClusterIP)     ║
║  • Sees new          ║  ║  • Allocates ClusterIP    ║
║    Deployment with   ║  ║    from IP pool           ║
║    replicas: 2       ║  ║  • Creates Endpoints obj  ║
║  • Creates           ║  ║    (initially empty)      ║
║    ReplicaSet object ║  ║                           ║
╚══════════════════════╝  ╚═══════════════════════════╝
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 8: ReplicaSet Controller                     ║
╠══════════════════════════════════════════════════════╣
║  • Reads ReplicaSet spec (replicas: 2)              ║
║  • Counts existing matching Pods = 0                ║
║  • Creates 2 Pod objects in etcd                    ║
║  • Pods are in "Pending" state                      ║
║  • Pods have NO node assignment yet                 ║
╚══════════════════════════════════════════════════════╝
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 9: Scheduler — Pod Assignment                ║
╠══════════════════════════════════════════════════════╣
║                                                      ║
║  [FILTERING PHASE]                                   ║
║  • NodeSelector match                               ║
║  • Taints and tolerations                           ║
║  • Resource availability (CPU/Memory requests)      ║
║    → CPU: 250m per pod                              ║
║    → Memory: 128Mi per pod                          ║
║  • Port availability                                ║
║  • Volume availability                              ║
║  • Affinity/Anti-affinity rules                     ║
║                                                      ║
║  [SCORING PHASE]                                    ║
║  • Least-requested resource (spread pods)           ║
║  • Node affinity preference                         ║
║  • Image locality (node already has image)          ║
║  • Topology spread                                  ║
║                                                      ║
║  RESULT:                                            ║
║  Pod 1 → Worker Node 1                              ║
║  Pod 2 → Worker Node 2  (spread for HA)             ║
║                                                      ║
║  • Updates Pod spec in etcd with nodeName           ║
╚══════════════════════════════════════════════════════╝
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 10: Kubelet (on each Worker Node)            ║
╠══════════════════════════════════════════════════════╣
║  • Watches API Server for Pods assigned to its node ║
║  • Sees Pod "signin-portal-1" assigned to Node 1    ║
║                                                      ║
║  [SECRET/CONFIGMAP RESOLUTION]                      ║
║  • Fetches Secret "db-secret" from API Server       ║
║  • Mounts into container as env var                 ║
║                                                      ║
║  [NETWORK SETUP — CNI Plugin]                       ║
║  • Calls CNI (e.g., Calico/Flannel)                 ║
║  • Creates veth pair                                ║
║  • Assigns Pod IP: 10.244.1.5                       ║
║  • Sets up routes on node                           ║
║                                                      ║
║  [VOLUME MOUNTS]                                    ║
║  • Mounts ServiceAccount token                      ║
║  • Mounts any PVC/ConfigMap volumes                 ║
╚══════════════════════════════════════════════════════╝
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 11: Container Runtime (containerd)           ║
╠══════════════════════════════════════════════════════╣
║                                                      ║
║  [IMAGE PULL]                                        ║
║  • Calls registry: myregistry/signin-portal:v1.0    ║
║  • Checks local image cache first                   ║
║  • Authenticates with imagePullSecret if set        ║
║  • Downloads image layers (union filesystem)        ║
║  • Stores in /var/lib/containerd                    ║
║                                                      ║
║  [CONTAINER CREATION]                               ║
║  • Creates Linux namespaces:                        ║
║    - PID namespace (isolated process tree)          ║
║    - Network namespace (isolated network stack)     ║
║    - Mount namespace (isolated filesystem)          ║
║    - UTS namespace (isolated hostname)              ║
║  • Sets up cgroups:                                 ║
║    - CPU limit: 500m                                ║
║    - Memory limit: 256Mi                            ║
║  • Injects environment variables (DB_HOST from      ║
║    Secret)                                          ║
║                                                      ║
║  [CONTAINER START]                                  ║
║  • Runs entrypoint:                                 ║
║    gunicorn --bind 0.0.0.0:8000 app:app             ║
║  • Python Flask/Django app starts                   ║
║  • Listening on port 8000                           ║
╚══════════════════════════════════════════════════════╝
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 12: Health Probes                            ║
╠══════════════════════════════════════════════════════╣
║                                                      ║
║  [STARTUP DELAY]                                    ║
║  • initialDelaySeconds: 5 (readiness)               ║
║  • initialDelaySeconds: 15 (liveness)               ║
║  • Pod stays in "Running" but NOT "Ready"           ║
║                                                      ║
║  [READINESS PROBE]                                  ║
║  • Kubelet → HTTP GET /health:8000                  ║
║  • Python app returns 200 OK                        ║
║  • Pod marked as "Ready"                            ║
║  • Endpoints object UPDATED:                        ║
║    10.244.1.5:8000 added ✓                          ║
║    10.244.2.7:8000 added ✓                          ║
║                                                      ║
║  [LIVENESS PROBE] (ongoing)                         ║
║  • Every 20 seconds: GET /health:8000               ║
║  • If 3 failures → container restart                ║
╚══════════════════════════════════════════════════════╝
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 13: Service & kube-proxy Routing             ║
╠══════════════════════════════════════════════════════╣
║  • kube-proxy watches Endpoints object              ║
║  • Programs iptables/IPVS rules on every node:      ║
║                                                      ║
║  DNAT rule:                                         ║
║  ClusterIP:80 → 10.244.1.5:8000 (50%)              ║
║               → 10.244.2.7:8000 (50%)              ║
║                                                      ║
║  • Round-robin load balancing between 2 pods        ║
╚══════════════════════════════════════════════════════╝
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 14: Ingress Controller                       ║
╠══════════════════════════════════════════════════════╣
║  • NGINX Ingress Controller watches Ingress objects  ║
║  • Detects new Ingress for signin.dev.myapp.com     ║
║  • Dynamically updates nginx.conf:                  ║
║                                                      ║
║  server {                                           ║
║    server_name signin.dev.myapp.com;                ║
║    location / {                                     ║
║      proxy_pass http://signin-portal-svc:80;        ║
║    }                                                ║
║  }                                                  ║
║                                                      ║
║  • NGINX reloads config (zero downtime)             ║
╚══════════════════════════════════════════════════════╝
        │
        ▼
╔══════════════════════════════════════════════════════╗
║  STAGE 15: LIVE — Request reaches Sign-In Portal    ║
╠══════════════════════════════════════════════════════╣
║                                                      ║
║  User Browser                                        ║
║       │                                              ║
║       │  GET https://signin.dev.myapp.com/login      ║
║       ▼                                              ║
║  DNS Resolver → LoadBalancer IP                     ║
║       │                                              ║
║       ▼                                              ║
║  Ingress Controller (NGINX)                         ║
║       │                                              ║
║       ▼                                              ║
║  Service: signin-portal-svc (ClusterIP:80)          ║
║       │                                              ║
║       ├──────────────────┐                           ║
║       ▼                  ▼                           ║
║  Pod 1 (Node 1)    Pod 2 (Node 2)                   ║
║  :8000             :8000                             ║
║  Python App        Python App                        ║
║       │                                              ║
║       ▼                                              ║
║  Returns: 200 OK — Sign-In HTML Page                ║
╚══════════════════════════════════════════════════════╝
```

---

## 📊 Object Hierarchy Created by dev.yaml

```
kubectl apply -f dev.yaml
│
├─► Deployment: signin-portal
│       │
│       └─► ReplicaSet: signin-portal-7d9f8b (auto-generated hash)
│                   │
│                   ├─► Pod: signin-portal-7d9f8b-xk2p1  (Node 1)
│                   │         └─► Container: signin-app
│                   │               └─► Image: signin-portal:v1.0
│                   │
│                   └─► Pod: signin-portal-7d9f8b-m3nt9  (Node 2)
│                             └─► Container: signin-app
│                                   └─► Image: signin-portal:v1.0
│
├─► Service: signin-portal-svc
│       └─► Endpoints: [10.244.1.5:8000, 10.244.2.7:8000]
│
└─► Ingress: signin-portal-ingress
        └─► Rule: signin.dev.myapp.com → signin-portal-svc:80
```

---

## 🔍 Detailed Component Reference

### Control Plane Components

| Component | Role | What It Does for Our Deployment |
|---|---|---|
| **kube-apiserver** | API Gateway | Receives `kubectl apply`, validates, stores to etcd |
| **etcd** | State Store | Persists Deployment, ReplicaSet, Pod, Service, Ingress objects |
| **kube-scheduler** | Pod Placer | Assigns Pod 1 → Node 1, Pod 2 → Node 2 |
| **kube-controller-manager** | Reconciler | Creates ReplicaSet from Deployment; creates Pods from ReplicaSet |

### Worker Node Components

| Component | Role | What It Does for Our Deployment |
|---|---|---|
| **kubelet** | Node Agent | Watches for pods assigned to its node; manages their lifecycle |
| **containerd** | Container Runtime | Pulls `signin-portal:v1.0` image; creates/starts containers |
| **kube-proxy** | Network Proxy | Sets iptables rules for Service ClusterIP routing |
| **CNI Plugin** | Network Plugin | Assigns pod IPs; sets up pod-to-pod routing |

### Kubernetes Objects Created

| Object | Kind | Purpose |
|---|---|---|
| `signin-portal` | Deployment | Declarative spec for 2 replicas, rolling update strategy |
| `signin-portal-7d9f8b` | ReplicaSet | Maintains exactly 2 running pods at all times |
| `signin-portal-7d9f8b-xxxxx` (×2) | Pod | Actual running instances of the Python app |
| `signin-portal-svc` | Service | Stable DNS + ClusterIP for pods (load balancing) |
| `signin-portal-svc` | Endpoints | Live list of ready pod IPs backing the Service |
| `signin-portal-ingress` | Ingress | HTTP routing rule from domain to Service |

---

## 🔄 Pod Lifecycle State Machine

```
               kubectl apply
                    │
                    ▼
             ┌──────────────┐
             │   PENDING    │ ← Waiting for scheduler
             │              │   or image pull
             └──────┬───────┘
                    │  Scheduler assigns node
                    │  Image pulled
                    │  Containers created
                    ▼
             ┌──────────────┐
             │   RUNNING    │ ← All containers started
             │              │   (but may not be "Ready")
             └──────┬───────┘
                    │  readinessProbe passes
                    ▼
             ┌──────────────┐
             │    READY     │ ← Added to Service Endpoints
             │  (condition) │   Traffic now flows to pod
             └──────┬───────┘
         ┌──────────┴──────────┐
         ▼                     ▼
  ┌─────────────┐       ┌─────────────────┐
  │  SUCCEEDED  │       │     FAILED      │
  │ (Completed) │       │                 │
  └─────────────┘       │ CrashLoopBackOff│
                        │ OOMKilled       │
                        │ ImagePullBackOff│
                        └────────┬────────┘
                                 │ restartPolicy: Always
                                 └──────► Back to PENDING
```

---

## 🌐 Network Traffic Flow (End-to-End)

```
  Internet
     │
     │  HTTPS: signin.dev.myapp.com
     ▼
  ┌─────────────────────────────────┐
  │  Cloud Load Balancer            │
  │  (AWS ALB / GCP LB / MetalLB)  │
  │  External IP: 203.0.113.10      │
  └──────────────┬──────────────────┘
                 │  Port 443/80
                 ▼
  ┌─────────────────────────────────┐
  │  Ingress Controller Pod         │
  │  (NGINX)                        │
  │                                 │
  │  nginx.conf rule:               │
  │  server_name signin.dev.myapp   │
  │  proxy_pass ClusterIP:80        │
  └──────────────┬──────────────────┘
                 │  Internal cluster routing
                 ▼
  ┌─────────────────────────────────┐
  │  Service: signin-portal-svc     │
  │  ClusterIP: 10.96.42.100        │
  │  Port: 80                       │
  │                                 │
  │  kube-proxy iptables DNAT:      │
  │  50% → 10.244.1.5:8000         │
  │  50% → 10.244.2.7:8000         │
  └──────────┬──────────────────────┘
             │
      ┌──────┴──────┐
      ▼             ▼
  ┌───────────┐  ┌───────────┐
  │  Pod 1    │  │  Pod 2    │
  │Node 1     │  │ Node 2    │
  │:8000      │  │ :8000     │
  │Python App │  │Python App │
  └───────────┘  └───────────┘
```

---

## ⚡ Self-Healing: What Happens When a Pod Crashes

```
  Pod crash detected by Kubelet
            │
            ▼
  Kubelet reports to API Server
  Pod status → Failed
            │
            ▼
  ReplicaSet Controller reconciles:
  desired=2, actual=1 → CREATE new Pod
            │
            ▼
  Scheduler assigns Pod to available Node
            │
            ▼
  Kubelet on node pulls image (if needed)
            │
            ▼
  New Pod starts, passes readiness probe
            │
            ▼
  Endpoints updated → traffic routes to new Pod
  Downtime: ~10–30 seconds (based on probe config)
```

---

## 📈 Rolling Update: What Happens on `kubectl set image`

```
  kubectl set image deployment/signin-portal signin-app=signin-portal:v2.0
            │
            ▼
  Deployment Controller creates new ReplicaSet (v2)
            │
            ├── maxSurge: 1 → temporarily creates extra Pod
            ├── maxUnavailable: 0 → never less than 2 Ready pods
            │
            ▼
  New ReplicaSet scales UP (v2 pods)
  Old ReplicaSet scales DOWN (v1 pods)
            │
            ▼
  Pattern (default strategy):
  [v1:2, v2:0] → [v1:2, v2:1] → [v1:1, v2:1] → [v1:1, v2:2] → [v1:0, v2:2]
            │
            ▼
  Zero downtime rolling update ✓
  Old ReplicaSet kept (for rollback):
  kubectl rollout undo deployment/signin-portal
```

---

## 🔐 Secrets Flow for DB_HOST

```
  dev.yaml references:
  secretKeyRef:
    name: db-secret
    key: host
            │
            ▼
  ┌──────────────────────────────────────────┐
  │  Secret: db-secret (stored in etcd)      │
  │  Encrypted at rest (etcd encryption)     │
  │  data:                                   │
  │    host: cG9zdGdyZXMuZGV2LmludGVybmFs  │  (base64)
  └──────────────────────────────────────────┘
            │  Kubelet fetches via API Server
            ▼
  Injected into container as:
  ENV DB_HOST=postgres.dev.internal
```

---

## 📋 Quick Status Check Commands

```bash
# Overall deployment status
kubectl get all -n dev

# Watch pods come up in real time
kubectl get pods -n dev -w

# Describe pod for events & errors
kubectl describe pod <pod-name> -n dev

# View application logs
kubectl logs -f deployment/signin-portal -n dev

# Check endpoints (confirms pods are ready)
kubectl get endpoints signin-portal-svc -n dev

# Inspect ingress routing
kubectl describe ingress signin-portal-ingress -n dev

# Verify scheduler decisions
kubectl get events -n dev --sort-by='.lastTimestamp'

# Check HPA (if configured)
kubectl get hpa -n dev
```

---

## 🗓️ Complete Timeline Summary

| Time | Event |
|---|---|
| T+0s | `kubectl apply -f dev.yaml` sent to API Server |
| T+0.1s | Auth/AuthZ validated, Admission webhooks run |
| T+0.2s | Objects persisted to etcd |
| T+0.3s | Deployment Controller creates ReplicaSet |
| T+0.4s | ReplicaSet Controller creates 2 Pod objects (Pending) |
| T+1s | Scheduler assigns Pods to nodes |
| T+2s | Kubelet on each node detects assigned Pods |
| T+5–30s | Image pulled from registry (if not cached) |
| T+30s | Containers created and started |
| T+35s | Readiness probes begin (initialDelaySeconds: 5) |
| T+35s | `/health` returns 200 → Pod marked Ready |
| T+36s | Endpoints object updated with Pod IPs |
| T+36s | kube-proxy updates iptables rules |
| T+37s | Ingress controller updates NGINX config |
| T+37s | **🟢 Sign-In Portal is LIVE and serving traffic** |

---

*Generated for: Python Sign-In Portal on Kubernetes | Namespace: dev | Replicas: 2*
