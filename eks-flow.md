# EKS Application Deployment — Full Architecture & Step-by-Step Guide

> Goal: Deploy one application on Amazon EKS, expose it to end users, and stream logs to CloudWatch — with a clear explanation of what happens at every stage.

---

## ⚠️ Important — Read Before You Start (AWS Free Trial)

The **EKS control plane is NOT covered by AWS Free Tier**. It costs **~$0.10/hour (~$73/month)** no matter how small your workload is. Free Tier only covers things like a limited amount of EC2 (t2.micro/t3.micro), S3, etc.

So on a free trial account, expect to pay for:
- EKS control plane: ~$0.10/hr
- EC2 worker nodes (if not free-tier eligible instance type): billed hourly
- NAT Gateway (if used): ~$0.045/hr + data processing — **this is the most commonly forgotten cost**
- Application Load Balancer: ~$0.0225/hr + LCU charges
- CloudWatch Logs: ingestion + storage (small for a test app, but not $0)

**Recommendation:** Follow this guide, test your app, take screenshots/notes, then run the **Cleanup** section at the end immediately. Don't leave an idle cluster running.

---

## 1. High-Level Architecture

```
                                   ┌─────────────────────────────────────────┐
                                   │                  AWS Cloud                │
                                   │                                           │
  End User                        │   ┌───────────────────────────────────┐   │
  (Browser) ───── HTTPS/HTTP ─────┼──▶│   Application Load Balancer (ALB)  │   │
                                   │   │   (created by AWS Load Balancer    │   │
                                   │   │    Controller via Ingress/Service) │   │
                                   │   └────────────────┬────────────────────┘   │
                                   │                     │                       │
                                   │            (routes to NodePort/Target)      │
                                   │                     ▼                       │
                                   │   ┌───────────────────────────────────┐   │
                                   │   │            VPC (10.0.0.0/16)       │   │
                                   │   │  ┌─────────────┐ ┌─────────────┐  │   │
                                   │   │  │ Public Subnet│ │ Public Subnet│  │   │
                                   │   │  │   (AZ-a)     │ │   (AZ-b)     │  │   │
                                   │   │  └─────┬────────┘ └──────┬──────┘  │   │
                                   │   │        │  NAT GW          │        │   │
                                   │   │  ┌─────▼────────┐ ┌──────▼──────┐ │   │
                                   │   │  │Private Subnet │ │Private Subnet│ │   │
                                   │   │  │   (AZ-a)      │ │   (AZ-b)     │ │   │
                                   │   │  │ ┌───────────┐ │ │ ┌──────────┐│ │   │
                                   │   │  │ │EC2 Worker │ │ │ │EC2 Worker││ │   │
                                   │   │  │ │Node (EKS  │ │ │ │Node (EKS ││ │   │
                                   │   │  │ │Managed    │ │ │ │Managed   ││ │   │
                                   │   │  │ │Node Group)│ │ │ │Node Group││ │   │
                                   │   │  │ │           │ │ │ │          ││ │   │
                                   │   │  │ │ [Pod: app]│ │ │ │[Pod: app]││ │   │
                                   │   │  │ └───────────┘ │ │ └──────────┘│ │   │
                                   │   │  └───────────────┘ └─────────────┘ │   │
                                   │   └───────────────────┬─────────────────┘   │
                                   │                        │                     │
                                   │            ┌───────────▼────────────┐        │
                                   │            │  EKS Control Plane      │        │
                                   │            │  (Managed by AWS,       │        │
                                   │            │   API server, etcd,     │        │
                                   │            │   scheduler, controller)│        │
                                   │            └───────────┬────────────┘        │
                                   │                        │                     │
                                   │            ┌───────────▼────────────┐        │
                                   │            │   CloudWatch Logs        │        │
                                   │            │  - /aws/eks/<cluster>/   │        │
                                   │            │    cluster (control      │        │
                                   │            │    plane logs)           │        │
                                   │            │  - Container Insights    │        │
                                   │            │    (pod/node logs,       │        │
                                   │            │    metrics)              │        │
                                   │            └──────────────────────────┘        │
                                   └───────────────────────────────────────────┘
```

**Flow in one sentence:** User hits a public ALB DNS name → ALB forwards traffic into the VPC → traffic reaches a Kubernetes Service → Service load-balances across Pods running on EC2 worker nodes inside private subnets → app responds → all cluster/app/node logs stream to CloudWatch.

---

## 2. Stage-by-Stage Explanation (What Actually Happens)

| Stage | What Happens | AWS/K8s Component |
|---|---|---|
| 1 | You define a Virtual Private Cloud with public + private subnets across 2+ Availability Zones | VPC, Subnets, IGW, NAT Gateway, Route Tables |
| 2 | You create the EKS "control plane" — AWS runs and manages the Kubernetes API server, etcd, scheduler for you in an AWS-owned account | EKS Cluster |
| 3 | You attach worker capacity — EC2 instances that actually run your containers | EKS Managed Node Group |
| 4 | You authenticate your local machine to talk to the cluster's API server | `kubectl` + `aws eks update-kubeconfig` |
| 5 | You package your app and push it to a registry | Docker + Amazon ECR |
| 6 | You tell Kubernetes to run N copies of your container, self-heal if they crash | Deployment (Pods + ReplicaSet) |
| 7 | You give the Pods a stable internal network identity | Service (ClusterIP) |
| 8 | You expose the Service to the internet via a real AWS Load Balancer | Ingress + AWS Load Balancer Controller → ALB |
| 9 | DNS routes users to the ALB's public address | ALB DNS name (or Route 53 if you want a custom domain) |
| 10 | Node, pod, application, and control-plane logs stream out | CloudWatch Container Insights + Control Plane Logging |

---

## 3. Prerequisites

Install these on your local machine (or use AWS CloudShell, which has most pre-installed):

```bash
# AWS CLI v2
aws --version

# kubectl (matching your EKS version, e.g. 1.30)
kubectl version --client

# eksctl (the fastest way to create an EKS cluster + nodegroup)
eksctl version

# Docker (to build your app image)
docker --version

# Helm (used to install the AWS Load Balancer Controller)
helm version
```

Configure AWS credentials:
```bash
aws configure
# AWS Access Key ID, Secret Access Key, region (e.g. ap-south-1), output format: json
```

---

## 4. Step 1 — Create the EKS Cluster + VPC + Node Group

`eksctl` creates the VPC, subnets, NAT gateway, IAM roles, the EKS control plane, and the node group in one shot — this is the fastest reliable path.

Create a config file `cluster.yaml`:

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: demo-eks-cluster
  region: ap-south-1     # change to your region
  version: "1.30"

iam:
  withOIDC: true          # REQUIRED — enables IAM Roles for Service Accounts (IRSA)

managedNodeGroups:
  - name: demo-ng
    instanceType: t3.medium     # free tier does NOT cover EKS nodes fully, pick smallest that works
    desiredCapacity: 2
    minSize: 1
    maxSize: 3
    volumeSize: 20
    privateNetworking: true     # nodes sit in private subnets (best practice)
    ssh:
      allow: false

cloudWatch:
  clusterLogging:
    enableTypes: ["api", "audit", "authenticator", "controllerManager", "scheduler"]
```

Create the cluster (takes ~15–20 minutes):
```bash
eksctl create cluster -f cluster.yaml
```

**What just happened:**
- A new VPC with 2 public + 2 private subnets (across 2 AZs) was created.
- The EKS control plane was provisioned inside an AWS-managed account (you never see or pay for its servers directly — you pay the flat hourly control-plane fee).
- An EC2 Auto Scaling Group of worker nodes was created inside the **private** subnets.
- `withOIDC: true` set up an **OIDC identity provider** for the cluster — this is what lets Kubernetes Service Accounts assume IAM Roles later (needed for the Load Balancer Controller and CloudWatch agent).
- `cloudWatch.clusterLogging` turned on **control plane log export** to CloudWatch (API server, audit, authenticator, controller manager, scheduler logs).

Verify:
```bash
kubectl get nodes
kubectl get svc
```

---

## 5. Step 2 — Push Your App Image to Amazon ECR

```bash
# Create a repository
aws ecr create-repository --repository-name demo-app --region ap-south-1

# Authenticate Docker to ECR
aws ecr get-login-password --region ap-south-1 | \
  docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com

# Build and tag your image
docker build -t demo-app .
docker tag demo-app:latest <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/demo-app:latest

# Push
docker push <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/demo-app:latest
```

---

## 6. Step 3 — Deploy the Application to Kubernetes

`deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
        - name: demo-app
          image: <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/demo-app:latest
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "250m"
              memory: "256Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: demo-app-svc
  namespace: default
spec:
  selector:
    app: demo-app
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 8080
```

Apply:
```bash
kubectl apply -f deployment.yaml
kubectl get pods -w
```

**What happened:** The Deployment tells the control plane's **scheduler** to place 2 Pods on available worker nodes. The **kubelet** on each node pulls your image from ECR and starts the container. The Service creates a stable internal DNS name (`demo-app-svc.default.svc.cluster.local`) and a virtual IP that load-balances across the 2 Pods — this is internal-only so far, not reachable from the internet.

---

## 7. Step 4 — Expose the App to the Internet (ALB via Ingress)

### 7a. Install the AWS Load Balancer Controller

This controller watches for Kubernetes `Ingress` objects and automatically provisions a real AWS Application Load Balancer.

```bash
# 1. Create IAM policy for the controller
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.8.0/docs/install/iam_policy.json

aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json

# 2. Create IAM Role + Kubernetes Service Account (uses the OIDC provider from Step 1)
eksctl create iamserviceaccount \
  --cluster=demo-eks-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

# 3. Install via Helm
helm repo add eks https://aws.github.io/eks-charts
helm repo update
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=demo-eks-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

**What happened:** You created a real IAM Role, and using **IRSA (IAM Roles for Service Accounts)** — enabled by the OIDC provider from Step 1 — you bound that IAM Role to a Kubernetes Service Account. Only the Load Balancer Controller Pod can assume this role. This is the secure way to give a Pod AWS permissions (no long-lived credentials stored in the cluster).

### 7b. Create the Ingress

`ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-app-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: demo-app-svc
                port:
                  number: 80
```

```bash
kubectl apply -f ingress.yaml
kubectl get ingress demo-app-ingress
```

**What happened:** The Load Balancer Controller detected the new Ingress object and called the AWS API to provision an **internet-facing Application Load Balancer**, created **Target Groups** pointing directly at Pod IPs (because of `target-type: ip`), and configured a listener on port 80. After 2–3 minutes, `kubectl get ingress` will show an `ADDRESS` column with the ALB's public DNS name.

### 7c. Access the Application

```bash
kubectl get ingress demo-app-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

Open that hostname in a browser — that's the **end-to-end path**: `Browser → ALB → Target Group → Pod IP → Container → your app`.

(Optional) Point a custom domain at it via Route 53:
```bash
# Create an ALIAS record in your hosted zone pointing to the ALB DNS name
```

---

## 8. Step 5 — Connect CloudWatch (Logs)

There are two separate log sources people usually want. Set up both.

### 8a. Control Plane Logs (already enabled in Step 1)

Since `cluster.yaml` already had `cloudWatch.clusterLogging.enableTypes`, these are already flowing to:
```
Log group: /aws/eks/demo-eks-cluster/cluster
```
View them:
```bash
aws logs tail /aws/eks/demo-eks-cluster/cluster --follow
```

### 8b. Application/Pod Logs + Node Metrics via Container Insights

This installs the **CloudWatch Agent + Fluent Bit** as a DaemonSet, which tails container logs from every node and ships them to CloudWatch.

```bash
ClusterName=demo-eks-cluster
RegionName=ap-south-1
FluentBitHttpPort='2020'
FluentBitReadFromHead='Off'
FluentBitHttpServer='On'
FluentBitReadFromTail='On'

kubectl create namespace amazon-cloudwatch

curl -s https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluent-bit-quickstart.yaml | \
sed 's/{{cluster_name}}/'${ClusterName}'/;s/{{region_name}}/'${RegionName}'/;s/{{http_server_toggle}}/"'${FluentBitHttpServer}'"/;s/{{http_server_port}}/"'${FluentBitHttpPort}'"/;s/{{read_from_head}}/"'${FluentBitReadFromHead}'"/;s/{{read_from_tail}}/"'${FluentBitReadFromTail}'"/' | \
kubectl apply -f -
```

This needs the nodes' IAM role to have the `CloudWatchAgentServerPolicy`:
```bash
aws iam attach-role-policy \
  --role-name <eksctl-created-nodegroup-role-name> \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy
```

**What happened:** A **Fluent Bit** Pod now runs on every worker node (DaemonSet), tails every container's stdout/stderr log files from the node's filesystem (`/var/log/containers/`), and forwards them to CloudWatch under log group names like:
```
/aws/containerinsights/demo-eks-cluster/application
/aws/containerinsights/demo-eks-cluster/dataplane
/aws/containerinsights/demo-eks-cluster/host
```
The CloudWatch Agent also pushes CPU/memory/network metrics per pod/node, viewable in **CloudWatch → Container Insights → Resources**.

Check logs in the console: **CloudWatch → Log groups → `/aws/containerinsights/demo-eks-cluster/application`**, filter by your pod name `demo-app-...`.

Or via CLI:
```bash
aws logs tail /aws/containerinsights/demo-eks-cluster/application --follow --filter-pattern demo-app
```

---

## 9. End-to-End Request Path (Summary)

```
1. User types ALB DNS name / your domain in browser
2. DNS resolves to ALB's public IP (AWS-managed, multi-AZ)
3. ALB receives HTTP request on port 80
4. ALB checks target group health checks, forwards to a healthy Pod IP
5. Traffic enters the VPC's private subnet, hits the ENI of the Pod
6. Container running in the Pod processes request, sends response
7. Response travels back: Pod → ALB → User
8. Simultaneously: Pod's stdout log line "GET / 200" is written to node's log file
9. Fluent Bit DaemonSet on that node tails the file, batches, ships to CloudWatch
10. You view it in CloudWatch Logs Insights / Container Insights dashboard
```

---

## 10. Handy Verification Commands

```bash
kubectl get nodes -o wide
kubectl get pods -o wide
kubectl get svc
kubectl get ingress
kubectl describe ingress demo-app-ingress
kubectl logs -l app=demo-app --tail=50
aws eks describe-cluster --name demo-eks-cluster --query "cluster.status"
```

---

## 11. Cleanup (Do This to Avoid Charges)

Order matters — delete the Ingress/ALB first, then the cluster (which removes nodes, VPC, NAT gateway, etc.):

```bash
kubectl delete ingress demo-app-ingress
kubectl delete -f deployment.yaml
helm uninstall aws-load-balancer-controller -n kube-system
eksctl delete iamserviceaccount --cluster=demo-eks-cluster --namespace=kube-system --name=aws-load-balancer-controller
eksctl delete cluster -f cluster.yaml
```

Also check manually in the console for anything orphaned:
- EC2 → Load Balancers (make sure ALB is gone)
- VPC → NAT Gateways (these silently rack up cost — delete if any remain)
- CloudWatch → Log groups (delete if you don't need the log history; storage costs are small but not zero)
- ECR → delete the repository/images if not needed

```bash
aws ecr delete-repository --repository-name demo-app --force
aws logs delete-log-group --log-group-name /aws/eks/demo-eks-cluster/cluster
aws logs delete-log-group --log-group-name /aws/containerinsights/demo-eks-cluster/application
```

---

## 12. Component Cheat Sheet

| Term | Plain-English meaning |
|---|---|
| **EKS Control Plane** | AWS-managed Kubernetes brain (API server, etcd, scheduler). You don't manage servers for this. |
| **Node Group** | The EC2 instances that actually run your app containers. |
| **Pod** | Smallest deployable unit — one or more containers sharing network/storage. |
| **Deployment** | Declares "keep N replicas of this Pod running, self-heal if one dies." |
| **Service (ClusterIP)** | Stable internal virtual IP + DNS name load-balancing across matching Pods. |
| **Ingress** | A Kubernetes object describing desired external HTTP routing rules. |
| **AWS Load Balancer Controller** | Watches Ingress objects, creates/manages a real ALB to match. |
| **OIDC Provider / IRSA** | Lets specific Kubernetes Service Accounts securely assume specific IAM Roles — no static AWS keys in the cluster. |
| **Fluent Bit DaemonSet** | Runs on every node, tails container logs, ships them to CloudWatch. |
| **Container Insights** | CloudWatch's dashboard for Kubernetes-aware metrics + logs. |

---

*Generated for a single-app EKS deployment reference. Adjust region, instance types, and replica counts to your actual workload.*
