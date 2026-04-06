Here is a **clean, interview-ready Markdown file** with **direct, no-BS answers + practical insights + troubleshooting steps**.

---

# 📘 Kubernetes Interview Q&A (Complete Guide)

## 1. What is Kubernetes and why is it used?

Kubernetes is an open-source container orchestration platform used to automate deployment, scaling, and management of containerized applications.

**Why used:**

* Automates deployment & scaling
* Ensures high availability
* Self-healing (restarts failed containers)
* Efficient resource utilization

---

## 2. Core Components of Kubernetes Architecture

**Control Plane:**

* API Server → Entry point
* etcd → Key-value store
* Scheduler → Assigns pods to nodes
* Controller Manager → Maintains desired state

**Worker Node:**

* Kubelet → Node agent
* Kube-proxy → Networking
* Container runtime (Docker/containerd)

---

## 3. What is a Pod?

Smallest deployable unit in Kubernetes.

* Contains one or more containers
* Shares network & storage
* Has a single IP

---

## 4. What is a Deployment?

Manages stateless applications.

**Features:**

* Rolling updates
* Rollback support
* Replica management

---

## 5. What is a Service?

Exposes a set of Pods via a stable IP.

* Decouples Pod lifecycle from access

---

## 6. Types of Services

| Type         | Description            |
| ------------ | ---------------------- |
| ClusterIP    | Internal access only   |
| NodePort     | Exposes via node IP    |
| LoadBalancer | External load balancer |

---

## 7. What is a Namespace?

Logical isolation within a cluster.

**Why:**

* Resource separation
* Multi-team environments

---

## 8. Deployment vs StatefulSet

| Feature  | Deployment     | StatefulSet   |
| -------- | -------------- | ------------- |
| Use case | Stateless apps | Stateful apps |
| Identity | Random         | Stable        |
| Storage  | Not persistent | Persistent    |

---

## 9. What is a DaemonSet?

Runs one Pod per node.

**Use cases:**

* Logging agents
* Monitoring agents

---

## 10. What is Ingress?

Manages external HTTP/HTTPS access.

* Works with Ingress Controller
* Supports routing, SSL

---

## 11. ConfigMaps and Secrets

| Type      | Purpose                          |
| --------- | -------------------------------- |
| ConfigMap | Non-sensitive config             |
| Secret    | Sensitive data (passwords, keys) |

---

## 12. Liveness vs Readiness Probes

| Probe     | Purpose                   |
| --------- | ------------------------- |
| Liveness  | Restart container if dead |
| Readiness | Controls traffic routing  |

---

## 13. Horizontal Pod Autoscaler (HPA)

Automatically scales Pods based on CPU/memory.

---

## 14. What is RBAC?

Role-Based Access Control.

* Controls permissions for users/services

---

## 15. Role vs ClusterRole

| Role            | ClusterRole  |
| --------------- | ------------ |
| Namespace-level | Cluster-wide |

---

## 16. Kubernetes Networking (Internal)

* Each Pod gets unique IP
* Flat network (no NAT)
* Pod-to-Pod communication across nodes

---

## 17. What is CNI?

Container Network Interface.

**Function:**

* Assigns IPs to Pods
* Handles networking (Calico, Flannel)

---

## 18. Service Discovery

* DNS-based
* Service name resolves to ClusterIP

Example:

```
my-service.default.svc.cluster.local
```

---

## 19. What is etcd?

* Distributed key-value store
* Stores cluster state

### Backup:

```
ETCDCTL_API=3 etcdctl snapshot save backup.db
```

### Restore:

```
etcdctl snapshot restore backup.db
```

---

## 20. Taints and Tolerations

* Taints → Restrict Pods
* Tolerations → Allow Pods

---

## 21. Affinity & Anti-Affinity

* Affinity → Place Pods together
* Anti-affinity → Spread Pods apart

---

## 22. How to Secure Kubernetes Cluster?

* Use RBAC
* Enable TLS
* Network policies
* Secrets encryption
* Limit access to API server

---

## 23. What is Helm?

Package manager for Kubernetes.

**Benefits:**

* Reusable templates
* Version control
* Simplified deployments

---

# 🚨 SCENARIO-BASED QUESTIONS

---

## 24. Pod stuck in CrashLoopBackOff — Troubleshoot

**Steps:**

1. Check logs:

```
kubectl logs <pod>
```

2. Describe pod:

```
kubectl describe pod <pod>
```

3. Check:

* Wrong command
* Missing env variables
* Resource limits

---

## 25. Service not accessible

**Check:**

* Pod running?
* Service selector correct?
* Port mismatch?
* Network policy blocking?

---

## 26. Zero-Downtime Deployment

Use:

* Rolling updates
* Readiness probes

Command:

```
kubectl rollout restart deployment <name>
```

---

## 27. Node Not Ready — Investigation

**Steps:**

* Check kubelet:

```
systemctl status kubelet
```

* Disk/memory issues
* Network problems

---

## 28. High CPU/Memory Usage

**Actions:**

* Check metrics:

```
kubectl top pods
```

* Increase limits
* Enable autoscaling

---

## 29. Auto Scaling in Kubernetes

* HPA → Pod scaling
* VPA → Resource adjustment
* Cluster Autoscaler → Node scaling

---

## 30. Monitoring & Logging

**Tools:**

* Prometheus → Metrics
* Grafana → Visualization
* ELK stack → Logs

---

# 🎯 Final Tip (Important for Interviews)

If you just memorize, you’ll fail.

Focus on:

* Real-time scenarios
* kubectl commands
* Debugging approach

---

If you want, next step I can:

* Turn this into **real-time project explanation (interview storytelling)**
* Or give you **mock interview Q&A (hard level)**
