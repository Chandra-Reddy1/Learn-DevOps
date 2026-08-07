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
