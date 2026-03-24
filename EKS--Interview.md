# AWS EKS Interview Questions & Answers (Top 20)

---

## 1. What is Amazon EKS?

Amazon EKS (Elastic Kubernetes Service) is a managed Kubernetes service where AWS manages the control plane.

---

## 2. What is Kubernetes control plane?

Manages cluster state:
- API Server
- Scheduler
- Controller Manager
- etcd

(In EKS, AWS manages this)

---

## 3. What are worker nodes in EKS?

EC2 instances or Fargate that run containers (pods).

---

## 4. What is a pod?

Smallest deployable unit in Kubernetes that runs containers.

---

## 5. What is a node group?

Group of worker nodes:
- Managed Node Group (AWS manages)
- Self-managed nodes

---

## 6. What is Fargate in EKS?

Serverless compute for running pods without managing EC2.

---

## 7. What is kubeconfig?

Configuration file used to connect kubectl to EKS cluster.

---

## 8. What is IAM role in EKS?

Controls permissions:
- Cluster access (IAM users/roles)
- Node permissions
- Service accounts (IRSA)

---

## 9. What is IRSA (IAM Roles for Service Accounts)?

Allows pods to access AWS services securely using IAM roles.

---

## 10. What is VPC role in EKS?

- Cluster runs inside VPC
- Pods and services use VPC networking

---

## 11. What is CNI plugin in EKS?

Amazon VPC CNI assigns IPs to pods from VPC subnet.

---

## 12. What is service in Kubernetes?

Exposes pods internally or externally.

Types:
- ClusterIP
- NodePort
- LoadBalancer

---

## 13. How to expose application in EKS?

- Use Service (LoadBalancer)
- Use Ingress + ALB

---

## 14. What is Ingress?

Manages external HTTP/HTTPS access to services.

---

## 15. What is autoscaling in EKS?

- Horizontal Pod Autoscaler (HPA)
- Cluster Autoscaler (node scaling)

---

## 16. How does EKS handle scaling?

- Pods scale using HPA
- Nodes scale using Cluster Autoscaler

---

## 17. What is ECR?

Elastic Container Registry to store Docker images used in EKS.

---

## 18. How to deploy application in EKS?

- Build Docker image
- Push to ECR
- Use Kubernetes manifests (Deployment, Service)

---

## 19. What is ConfigMap and Secret?

- ConfigMap → non-sensitive config
- Secret → sensitive data (passwords, keys)

---

## 20. What is logging and monitoring in EKS?

- CloudWatch
- Prometheus + Grafana

---

# Bonus (High-Impact DevOps Questions)

---

## 21. Difference between EKS and ECS?

- EKS → Kubernetes-based
- ECS → AWS native container service

---

## 22. How to secure EKS cluster?

- IAM roles (least privilege)
- Security groups
- Network policies
- RBAC

---

## 23. What is rolling deployment?

Gradual update of pods without downtime.

---

## 24. What is blue-green deployment?

Deploy new version alongside old and switch traffic.

---

## 25. How does networking work in EKS?

- Each pod gets IP from VPC
- Uses VPC CNI plugin

---

## 26. What is namespace?

Logical separation of resources in cluster.

---

## 27. What is Helm?

Package manager for Kubernetes to deploy apps easily.

---

## 28. What is persistent storage in EKS?

- EBS (block storage)
- EFS (shared file storage)

---

## 29. What is readiness vs liveness probe?

- Readiness → is pod ready to serve traffic?
- Liveness → is pod healthy?

---

## 30. How to troubleshoot EKS issues?

- kubectl logs
- kubectl describe
- Check events
- Check CloudWatch logs

---

# EKS Core Interview Flows (Deployment, Traffic, Scaling)

---

# 1. Deployment Flow (Docker → ECR → EKS)

## Step-by-Step

1. Build Docker Image
- Package application into a Docker image using Dockerfile

2. Push Image to ECR
- Tag image
- Push to AWS Elastic Container Registry (ECR)

3. Create Kubernetes Deployment
- Define Deployment YAML
- Specify:
  - Image (from ECR)
  - Replica count
  - Resource limits

4. Apply Deployment
- Use kubectl apply -f deployment.yaml
- Pods are created in EKS cluster

---

## Flow Summary

Developer → Docker Build → ECR → Kubernetes Deployment → Pods running in EKS

---

## One-Line Interview Answer

"We build Docker images, push them to ECR, and deploy them to EKS using Kubernetes manifests, which creates pods running the application."

---

# 2. Traffic Flow (ALB → Ingress → Service → Pod)

## Step-by-Step

1. User Request
- Client sends request (HTTP/HTTPS)

2. Load Balancer (ALB)
- ALB receives request
- Created via Ingress Controller

3. Ingress
- Defines routing rules (path-based / host-based)
- Routes traffic to appropriate service

4. Service
- Kubernetes Service exposes pods
- Typically ClusterIP

5. Pod
- Actual application container processes request

---

## Flow Summary

User → ALB → Ingress → Service → Pod → Response back

---

## One-Line Interview Answer

"User requests hit the ALB, which routes through Ingress to a Kubernetes Service, and finally reaches the application pods."

---

# 3. Scaling (HPA + Cluster Autoscaler)

## Step-by-Step

### Horizontal Pod Autoscaler (HPA)
- Monitors metrics (CPU/memory)
- Increases/decreases number of pods

### Cluster Autoscaler
- Adds/removes EC2 nodes
- Triggered when:
  - Pods cannot be scheduled (no resources)

---

## Combined Flow

- Traffic increases → CPU increases  
- HPA scales pods  
- If nodes insufficient → Cluster Autoscaler adds nodes  

---

## Flow Summary

Traffic ↑ → HPA scales pods → Cluster Autoscaler scales nodes

---

## One-Line Interview Answer

"We use HPA to scale pods based on load and Cluster Autoscaler to dynamically adjust the number of nodes when required."

---

# Common Mistakes

- Not mentioning ECR in deployment flow  
- Skipping Ingress in traffic flow  
- Confusing Service with Ingress  
- Ignoring node scaling  

---

# Final Tip

Always explain in this order:
1. Where app is stored (ECR)
2. Where app runs (Pods in EKS)
3. How traffic reaches it (ALB → Ingress → Service)
4. How it scales (HPA + Cluster Autoscaler)

