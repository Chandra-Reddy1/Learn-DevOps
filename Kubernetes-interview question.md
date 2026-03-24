# Kubernetes Interview Questions & Answers (Top 20)

---

## 1. What is Kubernetes?

Kubernetes is a container orchestration platform used to deploy, manage, and scale containerized applications.

---

## 2. What is a pod?

The smallest deployable unit in Kubernetes that runs one or more containers.

---

## 3. What is a node?

A machine (VM/physical) that runs pods.

---

## 4. What is a cluster?

A group of nodes managed by Kubernetes.

---

## 5. What is a Deployment?

Manages application deployment and ensures desired number of pods are running.

---

## 6. What is a ReplicaSet?

Ensures a specified number of pod replicas are running.

---

## 7. What is a Service?

Exposes pods for communication.

Types:
- ClusterIP
- NodePort
- LoadBalancer

---

## 8. What is Ingress?

Manages external HTTP/HTTPS access to services.

---

## 9. What is ConfigMap?

Stores non-sensitive configuration data.

---

## 10. What is Secret?

Stores sensitive data (passwords, keys).

---

## 11. What is namespace?

Logical separation within a cluster.

---

## 12. What is kubectl?

CLI tool to interact with Kubernetes.

---

## 13. What is rolling update?

Updates application gradually without downtime.

---

## 14. What is liveness probe?

Checks if container is alive.

---

## 15. What is readiness probe?

Checks if container is ready to serve traffic.

---

## 16. What is Horizontal Pod Autoscaler (HPA)?

Scales pods based on CPU/memory usage.

---

## 17. What is volume in Kubernetes?

Storage attached to pods.

---

## 18. What is Persistent Volume (PV)?

Cluster-wide storage resource.

---

## 19. What is Persistent Volume Claim (PVC)?

Request for storage by pods.

---

## 20. What is kubelet?

Agent running on each node to manage pods.

---

# Bonus (High-Impact DevOps Questions)

---

## 21. What is difference between Deployment and StatefulSet?

- Deployment → stateless apps  
- StatefulSet → stateful apps (DB, etc.)

---

## 22. What is DaemonSet?

Runs one pod per node (e.g., logging agents)

---

## 23. How does networking work in Kubernetes?

- Each pod gets unique IP  
- Services provide stable access  

---

## 24. How to expose application externally?

- Service (LoadBalancer)
- Ingress + ALB

---

## 25. What is etcd?

Key-value store that holds cluster state.

---

## 26. What is scheduler?

Assigns pods to nodes.

---

## 27. What is controller manager?

Maintains desired state.

---

## 28. What is RBAC?

Role-Based Access Control for permissions.

---

## 29. What is Helm?

Package manager for Kubernetes.

---

## 30. How to troubleshoot Kubernetes?

- kubectl logs  
- kubectl describe  
- kubectl get events  

---

# Final Interview Tips

- Always explain:
  - Pod → Service → Ingress flow
- Talk about:
  - scaling (HPA)
  - deployment (rolling update)
  - real usage

  # Kubernetes Core Interview Explanations

---

# 1. Pod → Service → Ingress Flow

## Step-by-Step Explanation

### Pod
- Smallest unit in Kubernetes
- Runs application container(s)
- Pods are dynamic (IP can change)

---

### Service
- Provides stable access to pods
- Uses label selectors to route traffic
- Types:
  - ClusterIP (internal)
  - NodePort
  - LoadBalancer

---

### Ingress
- Handles external HTTP/HTTPS traffic
- Provides routing rules (path/host-based)
- Works with Ingress Controller (e.g., ALB)

---

## Flow Summary

User → Ingress → Service → Pod → Response

---

## One-Line Interview Answer

"User traffic enters through Ingress, which routes requests to a Service, and the Service forwards traffic to the appropriate Pods."

---

# 2. Deployment vs StatefulSet

## Deployment

### Use Case
- Stateless applications (web apps, APIs)

### Features
- Rolling updates
- Replica management
- Pods are interchangeable

---

## StatefulSet

### Use Case
- Stateful applications (databases, Kafka)

### Features
- Stable pod names (e.g., db-0, db-1)
- Persistent storage
- Ordered deployment and scaling

---

## Key Difference

- Deployment → stateless, no identity  
- StatefulSet → stateful, stable identity  

---

## One-Line Interview Answer

"Deployment is used for stateless applications, while StatefulSet is used for stateful applications that require stable identity and persistent storage."

---

# 3. HPA (Horizontal Pod Autoscaler)

## Step-by-Step Explanation

### What it does
- Automatically scales number of pods

---

### How it works
- Monitors metrics (CPU/memory)
- If usage increases → scale up pods
- If usage decreases → scale down pods

---

### Example

- CPU > threshold → increase replicas  
- CPU < threshold → decrease replicas  

---

## Flow Summary

Traffic ↑ → CPU ↑ → HPA scales pods  
Traffic ↓ → CPU ↓ → HPA reduces pods  

---

## Important Note

- HPA scales pods, NOT nodes  
- Cluster Autoscaler is used for node scaling  

---

## One-Line Interview Answer

"HPA automatically scales pods based on resource usage like CPU or memory to handle varying traffic."

---

# Common Mistakes

- Skipping Service in traffic flow  
- Confusing Ingress with Service  
- Saying HPA scales nodes (wrong)  
- Not explaining StatefulSet identity  

---

# Final Tip

Always explain in order:
1. Where app runs → Pod  
2. How it is exposed → Service  
3. How external traffic comes → Ingress  
4. How it scales → HPA  

