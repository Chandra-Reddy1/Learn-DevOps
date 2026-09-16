# Amazon EKS — 70 Interview Questions

**🔥 = Top 10 / highest priority**

---

## 🔥 TOP 10 EKS QUESTIONS

### 1. 🔥 Explain EKS architecture end-to-end.

**Answer should cover:**
- AWS-managed control plane
- API Server + etcd
- Worker nodes / managed node groups
- VPC and subnets
- VPC CNI
- Kubernetes Services
- ALB/NLB
- Pods
- IAM integration

**Interview flow:**

```text
User → Route 53 → ALB → Target Group → Kubernetes Service → Pod → Container
```

---

### 2. 🔥 Explain traffic flow from Route 53 to a pod in EKS.

**Answer:**

```text
User
 ↓
Route 53
 ↓
ALB
 ↓
ALB Listener
 ↓
Target Group
 ↓
Kubernetes Service
 ↓
Pod
 ↓
Container
```

Mention whether you're using **AWS Load Balancer Controller**, and whether the ALB targets nodes or pod IPs depending on the target type.

---

### 3. 🔥 A pod is stuck in Pending. How do you troubleshoot it?

Check:

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl get nodes
kubectl describe node <node>
```

Look for:
- Insufficient CPU/memory
- NodeSelector
- Affinity/anti-affinity
- Taints/tolerations
- No available nodes
- Pod IP/CNI problems
- Resource requests too large

---

### 4. 🔥 A pod is in CrashLoopBackOff. What do you do?

```bash
kubectl get pod
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
```

Check:
- Application error
- Configuration
- Secrets
- Environment variables
- Liveness probe
- OOMKilled
- Exit code
- Dependency/database connectivity

---

### 5. 🔥 Explain IRSA in EKS.

**IRSA = IAM Roles for Service Accounts.**

It allows a Kubernetes pod to assume an AWS IAM role without giving AWS credentials directly to the pod.

**Flow:**

```text
Pod
 ↓
Kubernetes Service Account
 ↓
OIDC Provider
 ↓
IAM Role
 ↓
AWS Service
```

Example: pod needs access to S3 → service account is associated with an IAM role → pod obtains temporary credentials.

---

### 6. 🔥 Pod cannot access the internet. How would you troubleshoot it?

Check:
- Pod IP
- AWS VPC CNI
- Private subnet route table
- NAT Gateway
- Internet Gateway
- Security Groups
- Network ACL
- DNS/CoreDNS

Typical architecture:

```text
Pod
 ↓
Node
 ↓
Private Subnet
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

For private nodes, **NAT Gateway is commonly required for outbound internet access**.

---

### 7. 🔥 EKS pods are failing because there are no IP addresses available. What could cause it?

Main areas:
- VPC CIDR exhaustion
- Subnet IP exhaustion
- AWS VPC CNI limits
- ENI/IP allocation limits
- Too many pods per node

Check:

```bash
kubectl get nodes -o wide
kubectl describe node <node>
```

Then inspect VPC/subnet available IPs and CNI configuration.

---

### 8. 🔥 Explain HPA, Cluster Autoscaler and Karpenter.

**HPA:** increases/decreases **pod replicas**.

**Cluster Autoscaler:** adds/removes **nodes** based on unschedulable pods.

**Karpenter:** dynamically provisions suitable EC2 capacity based on pending pod requirements.

Typical flow:

```text
High workload
 ↓
HPA increases pods
 ↓
Pods cannot fit on existing nodes
 ↓
Cluster Autoscaler/Karpenter
 ↓
New node capacity
 ↓
Pods scheduled
```

---

### 9. 🔥 How would you upgrade an EKS cluster with minimal downtime?

Mention:
1. Check Kubernetes version compatibility.
2. Check add-ons.
3. Upgrade control plane.
4. Upgrade managed node groups.
5. Upgrade CoreDNS/kube-proxy/VPC CNI as appropriate.
6. Test workloads.
7. Use rolling node replacement.
8. Ensure multiple replicas.
9. Configure readiness probes and PDBs.
10. Monitor during/after upgrade.

**Important:** Don't say simply, "Change the version in Terraform and run apply." That's incomplete.

---

### 10. 🔥 Application is running but not accessible through ALB. How do you troubleshoot?

Follow the traffic path:

```text
DNS
 ↓
ALB
 ↓
Listener
 ↓
Target Group
 ↓
Service
 ↓
Endpoints
 ↓
Pod
```

Check:

```bash
kubectl get ingress
kubectl describe ingress <name>
kubectl get svc
kubectl get endpoints
kubectl get pods -o wide
```

Then check:
- ALB target health
- Security groups
- Listener rules
- Target group
- Service selector
- Service port/targetPort
- Pod readiness
- AWS Load Balancer Controller logs

---

# CATEGORY 1 — EKS ARCHITECTURE

### 11. What is Amazon EKS?

Managed Kubernetes service from AWS where AWS manages the Kubernetes control plane.

### 12. What components does AWS manage in EKS?

Primarily the Kubernetes control-plane infrastructure, including API server and etcd.

### 13. What do you manage?

Typically:
- Worker nodes
- Node groups
- Pods
- Deployments
- Services
- Networking configuration
- IAM
- Applications
- Add-ons

### 14. What is an EKS managed node group?

AWS-managed group of EC2 worker nodes running Kubernetes workloads.

### 15. Managed node group vs self-managed node group?

Managed node groups reduce operational work because AWS handles much of the node lifecycle.

Self-managed nodes provide more control but require more management.

### 16. What is Fargate in EKS?

Serverless compute for running Kubernetes pods without managing EC2 worker nodes.

### 17. EKS vs ECS?

EKS runs Kubernetes.

ECS is AWS's own container orchestration platform.

---

# CATEGORY 2 — EKS NETWORKING

### 18. What is AWS VPC CNI?

It allows Kubernetes pods to receive IP addresses from the AWS VPC networking infrastructure.

### 19. Why does EKS need subnets?

The EKS cluster and its networking resources operate within your VPC environment.

### 20. Public vs private subnet in EKS?

Common architecture:

**Public:**
- ALB
- NAT Gateway

**Private:**
- Worker nodes
- Pods
- Databases

### 21. Why put worker nodes in private subnets?

To avoid directly exposing worker nodes to the internet and reduce the attack surface.

### 22. What is a NAT Gateway?

Allows resources in private subnets to initiate outbound internet connections.

### 23. Internet Gateway vs NAT Gateway?

**IGW:** enables internet connectivity for resources with appropriate public routing/public IP.

**NAT Gateway:** provides outbound internet connectivity from private subnets.

### 24. What happens if NAT Gateway is unavailable?

Private workloads may lose outbound internet connectivity, depending on their routing and whether they have another egress path.

---

# CATEGORY 3 — SERVICES / ALB / INGRESS

### 25. What is a Kubernetes Service?

A stable networking abstraction used to expose a group of pods.

### 26. Explain ClusterIP, NodePort and LoadBalancer.

**ClusterIP:** internal access.

**NodePort:** exposes service on a port on nodes.

**LoadBalancer:** provisions/integrates with an external load balancer depending on the environment/controller.

### 27. What is Ingress?

A Kubernetes API resource defining HTTP/HTTPS routing rules.

### 28. What is AWS Load Balancer Controller?

It provisions and manages AWS load balancers for Kubernetes resources such as Ingress and Service.

### 29. ALB vs NLB?

**ALB:** Layer 7, HTTP/HTTPS, routing based on application-level information.

**NLB:** Layer 4, TCP/UDP/TLS, designed for high-performance network traffic.

### 30. What is a target group?

A group of backend targets to which the load balancer sends traffic.

### 31. What causes ALB targets to become unhealthy?

Possible causes:
- Wrong port
- Wrong target
- Security group issue
- Application not listening
- Readiness/application health issue
- Incorrect health-check path

---

# CATEGORY 4 — IAM / SECURITY

### 32. How does EKS integrate with IAM?

IAM controls AWS API permissions while Kubernetes RBAC controls Kubernetes API permissions.

### 33. IAM vs Kubernetes RBAC?

**IAM:** AWS resources.

**RBAC:** Kubernetes resources.

Example:

```text
IAM → S3 permissions
RBAC → permission to get/list Kubernetes pods
```

### 34. What is an EKS OIDC provider?

It establishes trust between the EKS cluster's Kubernetes identities and AWS IAM roles, commonly used with IRSA.

### 35. Why shouldn't you put AWS access keys inside Kubernetes Secrets?

Because static credentials are harder to rotate and increase security risk.

Prefer IAM roles with temporary credentials.

### 36. What is Kubernetes Secret?

Object used to store sensitive configuration such as credentials, tokens or certificates.

### 37. Is Kubernetes Secret encrypted by default?

Secrets are encoded in base64 at the Kubernetes API level; base64 is **not encryption**. EKS can use encryption at rest with AWS KMS.

---

# CATEGORY 5 — NODES

### 38. What happens when an EKS node goes down?

Kubernetes detects the node failure and workloads managed by controllers such as Deployments can be recreated on healthy nodes, assuming sufficient capacity.

### 39. How do you check node health?

```bash
kubectl get nodes
kubectl describe node <node>
```

Check conditions such as:

```text
Ready
MemoryPressure
DiskPressure
PIDPressure
```

### 40. What is a taint?

A mechanism that prevents pods from being scheduled onto a node unless they have a matching toleration.

### 41. What is a toleration?

Allows a pod to be scheduled onto a node with a matching taint.

### 42. What is node affinity?

Rules that influence which nodes a pod can or should run on based on node labels.

### 43. How do you drain a node?

Typical command:

```bash
kubectl drain <node> --ignore-daemonsets
```

The exact flags depend on the workload and situation.

---

# CATEGORY 6 — PODS / RESOURCES

### 44. What is OOMKilled?

The container exceeded its available memory and was killed.

Common Linux exit code:

```text
137
```

### 45. How do you troubleshoot OOMKilled?

Check:

```bash
kubectl describe pod <pod>
kubectl top pod
kubectl logs <pod> --previous
```

Then investigate:
- Memory limits
- Memory requests
- Application memory usage
- Memory leaks
- Traffic/load changes

### 46. Requests vs limits?

**Request:** amount Kubernetes uses for scheduling.

**Limit:** maximum resource consumption allowed for the container.

### 47. What happens if CPU request is too high?

Pods may remain Pending because available nodes don't have enough allocatable CPU.

### 48. What happens if memory limit is too low?

The container can be OOMKilled when it exceeds the limit.

---

# CATEGORY 7 — AUTOSCALING

### 49. What does HPA do?

Automatically adjusts pod replica count based on metrics.

### 50. What does Metrics Server do?

Provides resource metrics such as CPU and memory that can be consumed by Kubernetes components such as HPA.

### 51. HPA isn't getting CPU metrics. What do you check?

```bash
kubectl get pods -n kube-system
kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes"
kubectl top pods
```

Check Metrics Server health and configuration.

### 52. HPA increases replicas but pods remain Pending. Why?

Because there isn't enough node capacity.

Investigate:
- Node resources
- Scheduling constraints
- Cluster Autoscaler/Karpenter
- IP availability

---

# CATEGORY 8 — CORE KUBERNETES INSIDE EKS

### 53. What is kube-apiserver?

The central API endpoint through which Kubernetes components and clients communicate.

### 54. What is etcd?

Distributed key-value store containing Kubernetes cluster state.

### 55. What does kube-scheduler do?

Selects an appropriate node for unscheduled pods.

### 56. What does kube-controller-manager do?

Runs controllers that continuously reconcile desired state with actual state.

### 57. Deployment vs StatefulSet?

**Deployment:** generally stateless applications.

**StatefulSet:** applications requiring stable identity and/or persistent storage characteristics.

### 58. What is a DaemonSet?

Ensures a pod runs on eligible nodes, commonly used for agents such as logging or monitoring.

---

# CATEGORY 9 — TROUBLESHOOTING

### 59. Pod is Running but application isn't responding. What do you check?

Check:

```bash
kubectl logs <pod>
kubectl describe pod <pod>
kubectl exec -it <pod> -- <command>
```

Then verify:
- Application process
- Listening port
- Service
- Endpoints
- Network policy
- Readiness probe

### 60. Readiness probe is failing. What does it mean?

The pod is running but isn't considered ready to receive traffic.

### 61. Liveness probe vs readiness probe?

**Liveness:** should the container be restarted?

**Readiness:** should the pod receive traffic?

### 62. Service has no endpoints. Why?

Usually:
- Selector doesn't match pod labels
- Pods aren't Ready
- Wrong namespace
- Incorrect configuration

Check:

```bash
kubectl get svc
kubectl get endpoints
kubectl get pods --show-labels
```

### 63. ImagePullBackOff. What do you check?

- Image name/tag
- Registry availability
- ECR permissions
- Node/pod network connectivity
- ImagePullSecrets
- IAM permissions

### 64. DNS isn't working inside pods. What do you check?

Check CoreDNS:

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system <coredns-pod>
kubectl get svc -n kube-system
```

Also test DNS from inside a pod.

---

# CATEGORY 10 — REAL PRODUCTION SCENARIOS

### 65. 🔥 One pod dies during deployment. How do you prevent downtime?

Use:
- Multiple replicas
- RollingUpdate
- Readiness probes
- Proper resource requests
- PodDisruptionBudget where appropriate
- Load balancing

Don't rely on a single pod.

### 66. 🔥 Deployment is causing user sessions to disappear. What could be happening?

If session state is stored only inside individual pod memory/local filesystem, replacing the pod loses that state.

For scalable applications, externalize session state into a suitable shared/session store.

### 67. 🔥 Application deployment succeeded, but users receive 503 from ALB. What do you check?

Follow:

```text
ALB
 ↓
Target Group
 ↓
Service
 ↓
Endpoints
 ↓
Pod
```

Check:
- Target health
- Service selector
- targetPort
- readiness probe
- application listening port
- security groups

### 68. 🔥 EKS nodes have CPU available, but pod remains Pending. Why?

Don't assume CPU is the problem.

Check:

```bash
kubectl describe pod <pod>
```

Potential reasons:
- Taints
- Affinity
- NodeSelector
- Pod topology constraints
- Insufficient memory
- Volume constraints
- IP availability
- Other scheduling restrictions

### 69. 🔥 How would you troubleshoot high latency in an EKS application?

Work from outside → inside:

```text
Route53
 ↓
ALB
 ↓
Service
 ↓
Pod
 ↓
Application
 ↓
Database/external dependency
```

Check:
- ALB latency
- Target response time
- Pod CPU/memory
- Application logs
- Network connectivity
- Database latency
- Dynatrace/CloudWatch metrics

### 70. 🔥 You upgraded EKS and suddenly applications started failing. What would you investigate?

Check compatibility of:
- Kubernetes version
- AWS VPC CNI
- CoreDNS
- kube-proxy
- AWS Load Balancer Controller
- Ingress
- CRDs
- Node AMI
- Application dependencies

Then inspect:

```bash
kubectl get pods -A
kubectl get nodes
kubectl get events -A
kubectl logs <pod>
```

---

# 🎯 INTERVIEW PREPARATION PRIORITY

## Priority 1 — Absolutely Nail These

```text
1 → EKS architecture
2 → Route53 → ALB → Service → Pod
3 → Pending pod
4 → CrashLoopBackOff
5 → IRSA
6 → Pod internet connectivity
7 → Pod IP exhaustion
8 → HPA + Cluster Autoscaler/Karpenter
9 → EKS upgrade
10 → ALB/application not accessible
```

## Priority 2 — Networking & AWS Integration

```text
18 → VPC CNI
20 → Public/private subnets
22 → NAT Gateway
23 → IGW vs NAT
25 → Service
26 → Service types
27 → Ingress
28 → AWS Load Balancer Controller
29 → ALB vs NLB
31 → ALB target health
```

## Priority 3 — IAM, Nodes & Resources

```text
32 → EKS + IAM
33 → IAM vs RBAC
34 → OIDC
35 → AWS credentials
38 → Node failure
40 → Taints
41 → Tolerations
42 → Affinity
44 → OOMKilled
45 → OOM troubleshooting
46 → Requests vs limits
```

## Priority 4 — Autoscaling & Kubernetes

```text
49 → HPA
50 → Metrics Server
51 → HPA metrics troubleshooting
52 → HPA + node capacity
53 → kube-apiserver
54 → etcd
55 → scheduler
56 → controller manager
57 → Deployment vs StatefulSet
58 → DaemonSet
```

## Priority 5 — Production Scenarios

```text
59 → Running pod but app unavailable
60 → Readiness probe
61 → Liveness vs readiness
62 → Service with no endpoints
63 → ImagePullBackOff
64 → CoreDNS
65 → Pod failure during deployment
66 → Lost sessions
67 → ALB 503
68 → Pending pod despite CPU
69 → High latency
70 → Post-EKS-upgrade failure
```

---

# 🔥 Key Interview Rule

For scenario questions, don't randomly list commands.

Use this pattern:

```text
1. Identify the symptom
        ↓
2. Check Kubernetes object status
        ↓
3. Describe the object
        ↓
4. Check logs/events
        ↓
5. Trace the traffic/resource flow
        ↓
6. Identify the root cause
        ↓
7. Apply the fix
        ↓
8. Verify the result
        ↓
9. Add preventive measures
```

This makes your answer sound like **production troubleshooting experience**, rather than memorized Kubernetes theory.
