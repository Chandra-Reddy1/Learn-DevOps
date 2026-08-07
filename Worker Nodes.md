# Amazon EKS - Managed Node Groups

## What is a Managed Node Group?

A **Managed Node Group** is a group of EC2 instances (Worker Nodes) managed by AWS inside an EKS cluster.

Pods run on these worker nodes.

```text
EKS Cluster
│
├── Control Plane (Managed by AWS)
│
└── Managed Node Group
      ├── EC2 Node 1
      ├── EC2 Node 2
      └── EC2 Node 3
              │
              ▼
            Pods
```

---

# Why Do We Need Managed Node Groups?

In Kubernetes, applications run on Worker Nodes.

Without Managed Node Groups, you must:

- Create EC2 instances manually
- Configure Kubernetes components
- Join nodes to the cluster
- Upgrade nodes
- Replace failed nodes
- Manage scaling

This creates additional operational work.

Managed Node Groups automate most of these tasks.

---

# What Does AWS Manage?

## 1. Provisioning

### Without Managed Node Groups

You must:

- Launch EC2 instances
- Install kubelet
- Configure networking
- Join the node to the cluster

```text
You → Create EC2 → Install Software → Join Cluster
```

### With Managed Node Groups

AWS automatically:

- Creates EC2 instances
- Configures them
- Installs required Kubernetes components
- Joins them to the EKS cluster

```text
You → Select Node Type & Count

AWS →
  Create EC2
  Configure Node
  Join EKS Cluster
```

### Example

You specify:

```text
Instance Type: t3.medium
Desired Nodes : 3
```

AWS creates and configures all three nodes.

---

## 2. Updates

Kubernetes versions are frequently updated.

Worker nodes should match supported Kubernetes versions.

### Without Managed Node Groups

You must:

- Upgrade nodes manually
- Drain nodes
- Replace nodes
- Ensure application availability

### With Managed Node Groups

AWS assists with:

- Node upgrades
- Rolling updates
- Safe node replacement
- Maintaining cluster health

```text
Old Version
     ↓
AWS Upgrade
     ↓
New Version
```

---

## 3. Scaling

As traffic grows, more worker nodes may be required.

### Before Scaling

```text
Node 1
Node 2
```

### After Scaling

```text
Node 1
Node 2
Node 3
Node 4
```

AWS automatically increases or decreases the number of EC2 instances based on scaling settings.

### Scaling Parameters

```text
Minimum Size : 2
Desired Size : 3
Maximum Size : 10
```

Example:

If application demand increases,

```text
Current Nodes : 3
Traffic Increases
        ↓
AWS Creates Additional Nodes
        ↓
Current Nodes : 5
```

---

## 4. Health Monitoring & Auto Replacement

Worker nodes can fail due to:

- Hardware issues
- OS problems
- EC2 failures
- Network issues

### Without Managed Node Groups

You must:

- Detect failure
- Launch replacement node
- Join replacement node to cluster

### With Managed Node Groups

AWS automatically:

```text
Node Failure
      ↓
Detect Failure
      ↓
Terminate Bad Node
      ↓
Create New Node
      ↓
Join Cluster
```

This reduces downtime and manual effort.

---

# What Do We Still Manage?

Even with Managed Node Groups, AWS does not manage your applications.

You still manage:

✅ Pods

✅ Deployments

✅ Services

✅ Ingress

✅ Application Configuration

✅ Storage Configuration

✅ CI/CD Deployment

---
# EKS Workloads Explained with Real-Time Example

Let's take a simple example:

**You want to deploy a Banking Application in EKS.**

```text
Banking Application
        │
        ▼
     Deployment
        │
        ▼
       Pods
        │
        ▼
     Service
        │
        ▼
     Ingress
        │
        ▼
      Users
```

---

# 1. Application

The actual software you want to run.

Examples:

- Banking App
- E-commerce App
- NGINX Website
- Java Application
- Python Flask App
- NodeJS Application

Example:

```text
Internet Banking Portal
```

Container Image:

```text
banking-app:v1
```

This is what customers use.

---

# 2. Pod

A Pod is the smallest unit in Kubernetes.

It contains the application container.

Example:

```text
Pod-1
 └── banking-app:v1

Pod-2
 └── banking-app:v1
```

Think of a Pod as:

```text
Container + Network + Storage
```

### Real Example

1000 users access the banking application.

One pod may become overloaded.

Create more pods:

```text
Pod-1
Pod-2
Pod-3
```

Now traffic is shared.

---

# 3. Deployment

Deployment manages Pods.

Without Deployment:

```text
Manually Create Pods
```

If a Pod crashes:

```text
Pod Deleted
   ↓
Application Down
```

With Deployment:

```text
Deployment
      ↓
Maintains 3 Pods
```

If one pod crashes:

```text
Pod-2 Failed
      ↓
Deployment Detects
      ↓
Creates New Pod
```

Automatically.

---

## Banking Example

```text
Deployment: banking-app

Desired Pods = 3
```

Running:

```text
Pod-1
Pod-2
Pod-3
```

Pod-2 crashes.

Deployment automatically creates:

```text
Pod-1
Pod-3
Pod-4
```

Still 3 pods.

---

# 4. Service

Problem:

Pods get random IP addresses.

Example:

```text
Pod-1 → 10.0.1.10
Pod-2 → 10.0.1.11
Pod-3 → 10.0.1.12
```

If Pod-2 dies:

```text
New Pod → 10.0.1.50
```

IP changes.

Applications cannot depend on changing IPs.

---

## Solution: Service

Service provides one stable endpoint.

```text
Service
    │
    ├── Pod-1
    ├── Pod-2
    └── Pod-3
```

Users talk to:

```text
banking-service
```

not individual Pods.

---

## Load Balancing

Service distributes traffic.

```text
Request-1 → Pod-1
Request-2 → Pod-2
Request-3 → Pod-3
```

Traffic is balanced automatically.

---

# 5. Ingress

Service works inside the cluster.

But users are outside.

Need internet access.

---

## Without Ingress

```text
User
  ↓
Load Balancer
  ↓
Service
```

Each application may need a separate load balancer.

Expensive.

---

## With Ingress

```text
Internet
     ↓
Ingress
 ┌───────────┬───────────┐
 ↓           ↓
Bank App    HR App
 ↓           ↓
Service     Service
```

Single entry point.

---

## Example

User opens:

```text
bank.com
```

Ingress routes traffic:

```text
bank.com
  ↓
bank-service
```

Another URL:

```text
admin.bank.com
```

Ingress routes to:

```text
admin-service
```

---

# 6. Storage

Pods are temporary.

If pod dies:

```text
Pod Deleted
      ↓
Data Lost
```

Not acceptable for databases.

---

## Example

Customer uploads:

```text
loan_document.pdf
```

Stored inside pod.

Pod crashes.

```text
Document Lost
```

Bad.

---

## Solution

Use Storage.

### EBS

```text
Pod
 ↓
PVC
 ↓
EBS Volume
```

File remains even if pod restarts.

---

## Example

```text
Customer Uploaded File
       ↓
Stored In EBS
       ↓
Pod Restarted
       ↓
File Still Exists
```

---

# 7. Application Security

Protects the application.

---

## Layer 1: IAM

Controls who can access AWS resources.

Example:

```text
Application
    ↓
S3 Bucket
```

Only authorized application can access S3.

---

## Layer 2: RBAC

Controls Kubernetes permissions.

Developer Team:

```text
View Pods
```

Admin Team:

```text
Create Pods
Delete Pods
Update Pods
```

Different permissions.

---

## Layer 3: Secrets

Store passwords securely.

Bad:

```text
DB_PASSWORD=admin123
```

inside application code.

Good:

```text
Kubernetes Secret
```

Application reads secret securely.

---

## Layer 4: Network Policies

Restrict communication.

Example:

```text
Frontend
    ↓
Backend
```

Allowed.

But:

```text
Unknown Pod
      ↓
Backend
```

Blocked.

---

# Complete Flow

```text
User
 │
 ▼
Ingress
 │
 ▼
Service
 │
 ▼
Deployment
 │
 ▼
Pods
 │
 ▼
Storage (EBS/EFS)
```

---

# Super Easy Interview Explanation

## Application

The software we deploy (Java, Python, NodeJS, NGINX).

## Pod

Smallest Kubernetes unit where the application runs.

## Deployment

Maintains desired number of Pods and performs updates.

## Service

Provides stable access and load balancing to Pods.

## Ingress

Exposes applications to external users through URLs.

## Storage

Persists data even if Pods are restarted.

## Application Security

Protects applications using IAM, RBAC, Secrets, and Network Policies.

---

# Real EKS Production Architecture

```text
Internet User
      │
      ▼
AWS ALB (Ingress)
      │
      ▼
Kubernetes Service
      │
      ▼
Deployment
      │
      ▼
Pods
      │
      ▼
EBS / EFS Storage

AWS Security:
- IAM
- RBAC
- Secrets
- Security Groups
```

### One-Line Memory Trick

```text
Application → Deployment → Pods → Service → Ingress → Users

Storage → Saves Data
Security → Protects Data
```
# Responsibility Split

## EKS + Managed Node Groups

### AWS Manages

```text
Control Plane

- API Server
- Scheduler
- Controller Manager
- etcd

Worker Nodes

- Provisioning
- Updates
- Scaling
- Health Monitoring
- Node Replacement
```

### You Manage

```text
Applications
Deployments
Pods
Services
Ingress
Storage
Application Security
```

---

# Real-World Example

Suppose you deploy an NGINX application.

```text
EKS Cluster
│
├── Managed Node Group
│     ├── EC2-1
│     ├── EC2-2
│     └── EC2-3
│
└── NGINX Pods
```

### Failure Scenario

```text
EC2-2 Crashes
```

AWS automatically:

```text
Detects Failure
      ↓
Removes Bad Node
      ↓
Creates New EC2
      ↓
Joins New Node To EKS
      ↓
Pods Get Scheduled
```

Application remains available.

---

# Managed Node Groups vs Self-Managed Nodes

| Feature | Self-Managed Nodes | Managed Node Groups |
|----------|----------|----------|
| Node Creation | Manual | AWS Managed |
| Cluster Join | Manual | Automatic |
| Updates | Manual | AWS Assisted |
| Health Checks | Manual | AWS Managed |
| Auto Replacement | Manual | Automatic |
| Scaling | Manual | Automatic |
| Operational Effort | High | Low |

---

# Interview Questions

## What is a Managed Node Group?

A Managed Node Group is a collection of EC2 Worker Nodes managed by AWS within an EKS cluster.

---

## What does AWS manage in Managed Node Groups?

AWS manages:

- Node Provisioning
- Node Updates
- Node Scaling
- Health Monitoring
- Node Replacement

---

## What do we manage?

We manage:

- Pods
- Deployments
- Services
- Applications
- Ingress
- Storage

---

## Does AWS manage Pods?

No.

AWS manages the worker node infrastructure, while customers manage the Kubernetes workloads and applications.

---

# Easy Memory Trick

```text
EKS
 ↓
AWS Manages Control Plane

Managed Node Groups
 ↓
AWS Manages Worker Nodes

You
 ↓
Manage Applications & Pods
```

# Golden Interview Answer

> EKS manages the Kubernetes Control Plane, and Managed Node Groups help AWS manage Worker Nodes. AWS handles node provisioning, scaling, updates, and health replacement, while we focus on deploying and managing applications.
