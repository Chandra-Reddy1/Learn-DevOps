# Kubernetes Control Plane – Detailed Notes

---

## 1. kube-apiserver

### Use

* Entry point to Kubernetes cluster
* All components communicate through it (kubectl, UI, CI/CD)
* Processes REST API requests

### How it works

* Receives request → Auth → Authorization → Admission → etcd
* Returns response after updating/reading state

### Example

```bash
kubectl apply -f pod.yaml
```

Flow:

* kubectl → kube-apiserver
* API validates request
* Stores pod definition in etcd

### Architecture Flow

```
User / CI Tool
      |
      v
kube-apiserver
  |   |   |
Auth  |  Admission
      v
    etcd
```

### Key Point

If kube-apiserver is down:

* No kubectl access
* No deployments
* Cluster cannot be managed

---

## 2. etcd

### Use

* Distributed key-value store
* Stores entire cluster state

### What it stores

* Pods
* Secrets
* ConfigMaps
* Nodes
* Deployments

### Example

```yaml
apiVersion: v1
kind: Pod
```

→ Stored in etcd

### Architecture Flow

```
kube-apiserver
      |
      v
     etcd
  (Cluster State DB)
```

### Key Point

If etcd is lost:

* All cluster data is lost
* Recovery only possible with backups

---

## 3. kube-scheduler

### Use

* Decides which node a pod should run on

### Factors considered

* CPU / Memory
* Affinity / Anti-affinity
* Taints & Tolerations
* Resource requests

### Example

* Node1 → High CPU usage
* Node2 → Low CPU usage
  → Scheduler selects Node2

### Important

* Chooses node only
* Does NOT run containers

### Architecture Flow

```
Pod (Pending)
     |
     v
kube-scheduler
     |
     v
Assigned Node
```

---

## 4. kube-controller-manager

### Use

* Maintains desired state

### Controllers

* Node Controller
* ReplicaSet Controller
* Deployment Controller

### Concept

Desired State vs Actual State

### Example

```yaml
replicas: 3
```

Actual:

* 2 pods running

Action:
→ Creates 1 more pod

### Architecture Flow

```
Desired State
     |
     v
Controller Manager
     |
     v
Compare with Actual
     |
     v
Fix (Create/Delete)
```

### Key Point

Continuous loop:
Check → Compare → Fix

---

## 5. cloud-controller-manager

### Use

* Connects Kubernetes to cloud providers

### Supports

* AWS
* Azure
* GCP

### Responsibilities

* Load balancers
* Volumes
* Node lifecycle

### Example

```yaml
type: LoadBalancer
```

→ Creates cloud load balancer

### Architecture Flow

```
Kubernetes
     |
     v
Cloud Controller
     |
     v
Cloud API
     |
     v
Resources Created
```

---

## Full Control Plane Flow

```
User (kubectl / CI/CD)
        |
        v
kube-apiserver
   |        |
   v        v
 etcd   Controllers
             |
             v
      kube-scheduler
             |
             v
           Node
             |
             v
         kubelet → Pod Running
```

---

## Summary

* kube-apiserver → entry point
* etcd → storage
* scheduler → placement
* controller-manager → state management
* cloud-controller → cloud integration
