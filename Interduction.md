What is Kubernetes?
Kubernetes is a container orchestration platform that automates deployment, scaling, and management of containerized applications.

## More Clearly

- You put your application into containers (Docker).
- Kubernetes decides where to run them.
- It keeps them running if they crash.
- It scales them up or down based on load.
- It handles networking between services.
- It rolls out updates safely without downtime.

## Simple Real-World Analogy

Think of Kubernetes as:

- **Manager** → decides who does what  
- **Supervisor** → restarts workers if they fail  
- **Traffic Controller** → sends users to healthy apps  

# Kubernetes Components and Their Uses

Kubernetes components are broadly divided into **two main groups**:

1. Control Plane Components  
2. Node (Worker) Components  

There are also **add-on components** used in real-world clusters.

---

## 1. Control Plane Components (Cluster Management)

These components manage the cluster and decide **what should happen**.

### 1. kube-apiserver
**Use:**
- Entry point to the Kubernetes cluster
- All requests (`kubectl`, CI/CD tools, UI) go through it
- Validates and processes API requests

**Important:**  
If the API server is down, the cluster cannot be managed.

---

### 2. etcd
**Use:**
- Distributed key-value database
- Stores the entire cluster state (pods, configs, secrets)

**Important:**  
If etcd data is lost, the cluster state is lost.

---

### 3. kube-scheduler
**Use:**
- Decides which node a pod should run on
- Considers CPU, memory, constraints, affinity, and taints

**Note:**  
Scheduler only decides placement; it does not run containers.

---

### 4. kube-controller-manager
**Use:**
- Runs controllers to maintain desired state
- Examples:
  - Node controller
  - ReplicaSet controller
  - Deployment controller

**Purpose:**  
Ensures the actual state matches the desired state.

---

### 5. cloud-controller-manager (Cloud Environments)
**Use:**
- Integrates Kubernetes with cloud providers (AWS, Azure, GCP)
- Manages:
  - Load balancers
  - Volumes
  - Node lifecycle

---

## 2. Node Components (Runs Applications)

These components run on **every worker node**.

### 6. kubelet
**Use:**
- Node-level agent
- Communicates with the API server
- Ensures containers are running as defined in pod specs

**Important:**  
kubelet is responsible for pod execution on the node.

---

### 7. Container Runtime
**Use:**
- Runs containers on the node
- Examples:
  - containerd
  - CRI-O

**Note:**  
Kubernetes does not run containers directly.

---

### 8. kube-proxy
**Use:**
- Manages networking rules
- Implements Kubernetes Services
- Handles load balancing between pods

---

## 3. Add-on Components (Used in Real Clusters)

These are not core but are critical in production environments.

### 9. CoreDNS
**Use:**
- Provides DNS resolution inside the cluster
- Enables pod-to-pod communication using service names

---

### 10. CNI Plugin (Calico, Flannel, Cilium)
**Use:**
- Handles pod networking
- Assigns IP addresses to pods
- Enforces network policies

---

### 11. CSI Plugin
**Use:**
- Integrates storage systems
- Enables dynamic volume provisioning

---

## Quick Summary

### Control Plane Components
- kube-apiserver
- etcd
- kube-scheduler
- kube-controller-manager
- cloud-controller-manager

### Node Components
- kubelet
- container runtime
- kube-proxy

### Add-ons
- CoreDNS
- CNI plugins
- CSI plugins

---

## Interview One-Liner
Kubernetes consists of control plane components that manage the cluster, node components that run workloads, and add-ons for networking, DNS, and storage.

# Kubernetes Objects – Uses

This document lists the **primary uses** of core Kubernetes objects.

---

## Container

**Uses:**
- Runs the application process
- Packages application code with dependencies
- Ensures consistency across environments (dev, test, prod)
- Provides isolation between applications
- Enables portability across systems

---

## Pod

**Uses:**
- Acts as the smallest deployable unit in Kubernetes
- Groups one or more tightly coupled containers
- Shares networking (IP address and ports) between containers
- Shares storage volumes between containers
- Manages container lifecycle together

---

## ReplicaSet

**Uses:**
- Ensures a specified number of pod replicas are always running
- Provides high availability for applications
- Automatically recreates pods if they crash or are deleted
- Maintains application reliability
- Supports horizontal scaling

---

## Deployment

**Uses:**
- Manages ReplicaSets and Pods declaratively
- Enables rolling updates with zero or minimal downtime
- Supports rollbacks to previous application versions
- Scales applications up or down easily
- Maintains application lifecycle in production environments

---

## Summary

```text
Container  → Runs the application
Pod        → Groups containers
ReplicaSet → Maintains pod count
Deployment → Controls lifecycle and updates


