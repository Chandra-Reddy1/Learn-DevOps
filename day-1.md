# Amazon EKS (Elastic Kubernetes Service) - Complete Beginner Guide

## What is EKS?

Amazon EKS (Elastic Kubernetes Service) is a **managed Kubernetes service provided by AWS**.

Instead of installing and managing Kubernetes yourself, AWS manages the Kubernetes Control Plane for you.

### Simple Definition

> EKS = Kubernetes + AWS Management

---

# Why Do We Need EKS?

In a Self-Managed Kubernetes Cluster, you need to:

- Install Kubernetes
- Configure Control Plane
- Manage etcd
- Apply Security Patches
- Perform Upgrades
- Configure High Availability
- Monitor Cluster Health

This requires significant effort.

With EKS:

- AWS manages the Control Plane
- AWS provides High Availability
- AWS simplifies upgrades
- AWS handles Control Plane monitoring

You focus on deploying applications.

---

# Kubernetes vs EKS

| Kubernetes (Self Managed) | EKS |
|------------|------------|
| You manage Control Plane | AWS manages Control Plane |
| Manual upgrades | AWS assisted upgrades |
| More operational effort | Less operational effort |
| Any cloud/on-prem | AWS Cloud |
| Need HA setup | HA built-in |

### Easy Interview Answer

> EKS is a managed Kubernetes service where AWS manages the control plane, reducing operational overhead and simplifying cluster management.

---

# EKS Architecture

```text
                 Internet
                     |
              Load Balancer
                     |
                 Service
                     |
                Deployment
                     |
                   Pods
                     |
              Worker Nodes
                     |
          Amazon EKS Cluster
                     |
                   VPC
```

---

# Components of EKS

## 1. Control Plane

Managed by AWS.

Responsible for:

- API Server
- Scheduler
- Controller Manager
- etcd

### Control Plane Duties

- Cluster Management
- Scheduling Pods
- Maintaining Cluster State
- Authentication

You do not manage it.

AWS manages it.

---

## 2. Worker Nodes

Worker nodes run applications.

Can be:

### Managed Node Groups

AWS manages:

- Provisioning
- Updates
- Scaling

Most commonly used.

### Self-Managed Nodes

You manage:

- EC2 Instances
- Security
- Updates

More administration effort.

### AWS Fargate

No servers to manage.

AWS runs Pods directly.

---

## 3. Pods

Smallest deployable unit in Kubernetes.

Example:

```text
Pod
 ├── Nginx Container
 └── Sidecar Container
```

A pod may contain one or more containers.

---

## 4. Deployment

Used to manage Pods.

Benefits:

- Scaling
- Auto-Healing
- Rolling Updates
- Rollbacks

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
```

---

## 5. Service

Pods have dynamic IP addresses.

Service provides a stable endpoint.

Types:

- ClusterIP
- NodePort
- LoadBalancer

Most used in EKS:

```text
LoadBalancer
```

Because AWS automatically creates an ELB/NLB.

---

# Networking in EKS

EKS uses AWS VPC.

Components:

```text
VPC
├── Public Subnet
├── Private Subnet
├── Route Tables
├── Security Groups
└── NAT Gateway
```

### Pod Networking

AWS VPC CNI Plugin assigns VPC IP addresses directly to pods.

Benefit:

- Better networking
- Native AWS integration

---

# Storage in EKS

Containers are temporary.

Need persistent storage.

### EBS

Used for block storage.

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
EBS Volume
```

### EFS

Shared storage.

Multiple pods can access the same storage.

Useful for:

- Shared files
- Web applications
- CMS tools

---

# Security in EKS

## IAM

AWS Identity and Access Management.

Controls AWS access.

Example:

```text
User
 ↓
IAM Role
 ↓
EKS Access
```

---

## RBAC

Role-Based Access Control.

Controls Kubernetes permissions.

Examples:

- View Pods
- Create Deployments
- Delete Services

---

## IRSA

IAM Roles for Service Accounts.

Allows a pod to access AWS services securely.

Example:

```text
Pod
 ↓
IAM Role
 ↓
S3 Bucket
```

No need to store AWS Access Keys.

Very important interview topic.

---

# Scaling in EKS

## Horizontal Pod Autoscaler (HPA)

Automatically increases pod count.

Example:

```text
CPU High
   ↓
More Pods Created
```

---

## Cluster Autoscaler

Automatically adds worker nodes.

Example:

```text
No Space For Pods
       ↓
Add New Node
```

---

# Monitoring in EKS

Common tools:

## CloudWatch

AWS monitoring service.

Used for:

- Logs
- Metrics
- Alerts

---

## Prometheus

Collects Kubernetes metrics.

Example:

- CPU
- Memory
- Network

---

## Grafana

Dashboard and visualization tool.

Shows cluster health visually.

---

# Common EKS Commands

## View Nodes

```bash
kubectl get nodes
```

---

## View Pods

```bash
kubectl get pods
```

---

## View Services

```bash
kubectl get svc
```

---

## Describe Pod

```bash
kubectl describe pod <pod-name>
```

---

## View Logs

```bash
kubectl logs <pod-name>
```

---

## Access Pod

```bash
kubectl exec -it <pod-name> -- bash
```

---

# EKS Workflow

```text
Developer
    ↓
kubectl
    ↓
EKS API Server
    ↓
Scheduler
    ↓
Worker Node
    ↓
Pod Creation
```

---

# EKS Real World Architecture

```text
Users
  ↓
AWS Load Balancer
  ↓
Ingress Controller
  ↓
Service
  ↓
Application Pods
  ↓
Worker Nodes
  ↓
Amazon EKS
```

---

# Advantages of EKS

✅ AWS manages Control Plane

✅ High Availability by default

✅ Easy AWS Integration

✅ Scalable

✅ Secure

✅ Less maintenance

✅ Production Ready

✅ Supports CI/CD Pipelines

---

# Disadvantages of EKS

❌ More expensive than self-managed clusters

❌ AWS-specific solution

❌ Worker nodes still need management (unless using Fargate)

---

# Important Interview Questions

## What is EKS?

Amazon EKS is a managed Kubernetes service provided by AWS.

---

## Why EKS instead of Kubernetes?

Because AWS manages the Kubernetes Control Plane, reducing management overhead.

---

## What does AWS manage in EKS?

- API Server
- Scheduler
- Controller Manager
- etcd

---

## What are Worker Nodes?

EC2 instances that run application Pods.

---

## What is a Pod?

Smallest deployable unit in Kubernetes.

---

## What is a Deployment?

Kubernetes object that manages Pods and supports scaling and updates.

---

## What is IRSA?

IAM Roles for Service Accounts that allow Pods to access AWS services securely.

---

## What is HPA?

Horizontal Pod Autoscaler automatically scales Pods based on resource usage.

---

# Quick Revision (2-Minute Summary)

```text
EKS = Managed Kubernetes Service by AWS

AWS Manages:
- Control Plane
- High Availability
- etcd
- Scheduler
- API Server

You Manage:
- Worker Nodes
- Pods
- Applications

Main Benefits:
- Less Maintenance
- Easy Scaling
- AWS Integration
- Better Security
- High Availability
```

## Golden Interview Answer

"EKS is AWS's managed Kubernetes service. AWS manages the Kubernetes control plane, high availability, and cluster management, while we focus on deploying and managing applications. This reduces operational overhead and provides seamless integration with AWS services like IAM, VPC, ALB, EBS, and CloudWatch."
