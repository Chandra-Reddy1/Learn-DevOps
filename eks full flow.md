| Priority | Topic                   | What to learn                                               |
| -------- | ----------------------- | ----------------------------------------------------------- |
| 🔴 1     | **EKS fundamentals**    | Cluster, control plane, data plane, node groups, Pods       |
| 🔴 2     | **EKS networking**      | VPC, public/private subnets, ENI, VPC CNI, SG, NAT, routing |
| 🔴 3     | **Managed Node Groups** | EC2 nodes, AMI, instance types, scaling, upgrades, draining |
| 🔴 4     | **IAM + EKS security**  | IAM roles, policies, EKS access, Pod Identity/IRSA, RBAC    |
| 🔴 5     | **ECR**                 | Build image → push to ECR → pull image from EKS             |
| 🔴 6     | **Load balancing**      | ALB, NLB, Ingress, AWS Load Balancer Controller             |
| 🟠 7     | **Storage**             | EBS, EFS, PV, PVC, StorageClass, CSI drivers                |
| 🟠 8     | **Scaling**             | HPA, Cluster Autoscaler, Karpenter                          |
| 🟠 9     | **Monitoring**          | CloudWatch, Container Insights, Prometheus/Grafana          |
| 🟠 10    | **Troubleshooting**     | Pods, nodes, networking, deployments, logs, events          |


# AWS EKS Application Deployment — Services and Components

This guide explains the AWS services and Kubernetes components to learn when deploying a Python Flask application on Amazon EKS.

The example application is:

```text
Python Flask Application
        |
        v
Docker Image
        |
        v
Amazon ECR
        |
        v
Amazon EKS
        |
        v
Kubernetes Pods
        |
        v
PostgreSQL / Amazon RDS
```

---

# 1. Amazon EKS

## What is EKS?

Amazon Elastic Kubernetes Service (EKS) is AWS's managed Kubernetes service.

Kubernetes manages containerized applications. EKS provides a managed Kubernetes control plane and integrates Kubernetes with AWS infrastructure.

```text
Kubernetes + AWS
       |
       v
      EKS
```

## EKS Cluster

An EKS cluster is the overall Kubernetes environment.

```text
EKS Cluster
|
+-- Control Plane
|
+-- Data Plane
    |
    +-- Node Group
        |
        +-- EC2 Node
        |   |
        |   +-- Pod
        |   +-- Pod
        |
        +-- EC2 Node
            |
            +-- Pod
            +-- Pod
```

## Control Plane

The control plane is the brain of Kubernetes.

Important components:

- API Server
- etcd
- Scheduler
- Controller Manager

### API Server

The API Server is the main entry point for Kubernetes operations.

For example:

```bash
kubectl get pods
```

Conceptually:

```text
kubectl
   |
   v
EKS API Server
   |
   v
Kubernetes
```

When you run:

```bash
kubectl apply -f deployment.yaml
```

the request goes to the Kubernetes API Server.

### etcd

etcd is Kubernetes' data store.

It stores cluster state such as:

```text
Deployment:
    welcome-app

Desired replicas:
    2

Service:
    welcome-service
```

Conceptually:

```text
API Server
    |
    v
  etcd
    |
    v
Cluster State
```

AWS manages the EKS control-plane infrastructure.

### Scheduler

The scheduler decides which node should run a Pod.

Example:

```text
Node 1 -> 90% CPU
Node 2 -> 30% CPU
Node 3 -> 40% CPU

New Pod
   |
   v
Scheduler
   |
   v
Node 2
```

The scheduler selects a suitable node. It does not itself run the container.

### Controller Manager

Kubernetes controllers continuously work toward the desired state.

Example:

```yaml
replicas: 2
```

If one Pod crashes:

```text
Desired = 2
Actual  = 1
```

Kubernetes creates another Pod.

```text
Deployment
    |
    v
Controller
    |
    v
New Pod
```

The key Kubernetes concept is:

```text
Desired State = Actual State
```

---

## Managed Node Groups

Managed Node Groups provide EC2 worker nodes for EKS.

```text
EKS
 |
Managed Node Group
 |
+---------+---------+
|                   |
EC2 Node 1       EC2 Node 2
|                   |
Pod                 Pod
```

AWS manages several aspects of the node-group lifecycle, while you configure items such as:

- Instance type
- Minimum nodes
- Maximum nodes
- Desired nodes
- Subnets
- Capacity type

Example:

```text
min     = 2
desired = 2
max     = 4
```

---

## Pods

A Pod is the smallest deployable Kubernetes unit.

For the Flask application:

```text
Pod
 |
 +-- Flask container
```

If you run two replicas:

```text
welcome-app
|
+-- Pod 1
|
+-- Pod 2
```

The Pod runs your Docker image:

```text
public.ecr.aws/o5l5z4t1/my-app:<version>
```

---

## Deployments

A Deployment defines how an application should run.

Example:

```yaml
replicas: 2
```

Conceptually:

```text
Deployment
    |
    v
ReplicaSet
    |
 +--+--+
 |     |
Pod   Pod
```

If one Pod dies, Kubernetes recreates it to maintain the desired replica count.

---

## Kubernetes Services

Pods are temporary and their IP addresses can change.

A Kubernetes Service provides a stable network endpoint for Pods.

```text
Service
 |
 +-- Pod 1
 |
 +-- Pod 2
```

Example:

```text
welcome-service:80
       |
       v
Pod:5000
```

The Service can expose port 80 while forwarding to the Flask container on port 5000.

---

## Ingress

Ingress defines HTTP/HTTPS routing into Kubernetes.

Example:

```text
myapp.example.com
       |
       v
Ingress
       |
       v
welcome-service
       |
       v
Pods
```

In AWS, the AWS Load Balancer Controller can use an Ingress resource to create/configure an ALB.

---

## EKS Add-ons

Important EKS add-ons/components to understand:

### CoreDNS

Provides DNS-based service discovery inside Kubernetes.

```text
Pod
 |
CoreDNS
 |
Service DNS
```

### kube-proxy

Provides Kubernetes Service networking behavior on nodes.

### VPC CNI

Provides AWS VPC networking for Pods.

This is especially important in EKS because Pod networking is integrated with the AWS VPC.

### EBS CSI Driver

Allows Kubernetes workloads to use Amazon EBS storage.

### EKS Pod Identity Agent

Supports EKS Pod Identity for giving AWS permissions to Kubernetes workloads.

---

## EKS Upgrades

An EKS upgrade involves more than changing the Kubernetes version.

Typical process:

```text
EKS Control Plane Upgrade
          |
          v
Check Add-ons
          |
          v
Upgrade Node Groups
          |
          v
Drain / Replace Nodes
          |
          v
Verify Workloads
```

Consider:

- Kubernetes version compatibility
- Add-on versions
- Node AMIs
- Workload compatibility
- Deprecated Kubernetes APIs
- Pod disruption
- Node draining

---

# 2. Amazon VPC

VPC is the networking foundation for the EKS environment.

Typical architecture:

```text
VPC
|
+-- Public Subnets
|    |
|    +-- ALB
|
+-- Private Subnets
     |
     +-- EKS Nodes
     |
     +-- Pods
     |
     +-- RDS
```

---

## VPC

A VPC is an isolated virtual network in AWS.

Example:

```text
10.0.0.0/16
```

Inside it, you create subnets.

```text
VPC 10.0.0.0/16
|
+-- Public subnet
|   10.0.1.0/24
|
+-- Public subnet
|   10.0.2.0/24
|
+-- Private subnet
|   10.0.11.0/24
|
+-- Private subnet
    10.0.12.0/24
```

---

## CIDR

CIDR defines an IP address range.

Example:

```text
10.0.0.0/16
```

A subnet could be:

```text
10.0.1.0/24
```

You need to understand CIDR because EKS depends heavily on available IP addresses.

---

## Public Subnet

A public subnet has a route toward an Internet Gateway.

Typical path:

```text
Public Subnet
      |
      v
Internet Gateway
      |
      v
Internet
```

An internet-facing ALB can be placed in public subnets.

---

## Private Subnet

A private subnet does not have a direct route to the Internet Gateway.

EKS worker nodes are commonly placed in private subnets.

For outbound Internet access:

```text
Private Subnet
      |
      v
NAT Gateway
      |
      v
Internet Gateway
      |
      v
Internet
```

This allows private resources to initiate outbound connections without making them directly reachable from the Internet.

---

## Route Tables

Route tables determine where network traffic goes.

Public subnet:

```text
0.0.0.0/0
     |
     v
Internet Gateway
```

Private subnet:

```text
0.0.0.0/0
     |
     v
NAT Gateway
```

---

## Internet Gateway

An Internet Gateway connects a VPC to the Internet.

Example:

```text
ALB
 |
Public Subnet
 |
Route Table
 |
Internet Gateway
 |
Internet
```

A resource still requires appropriate routes and security rules to communicate with the Internet.

---

## NAT Gateway

A NAT Gateway provides outbound Internet connectivity for resources in private subnets.

Example:

```text
EKS Node
   |
Private Subnet
   |
Route Table
   |
NAT Gateway
   |
Internet Gateway
   |
Internet
```

Important:

NAT Gateway provides outbound connectivity. It does not make the private node directly reachable from the Internet.

---

## Security Groups

Security Groups are virtual firewalls associated with AWS resources or network interfaces.

Example for RDS:

```text
RDS Security Group
|
+-- Allow TCP 5432
    from the application/network source
```

Then:

```text
Application
    |
    v
RDS Security Group
    |
    v
PostgreSQL :5432
```

Security Groups are stateful.

---

## Network ACLs

Network ACLs operate at the subnet level.

```text
Internet
   |
   v
NACL
   |
   v
Subnet
   |
   v
Resource
```

NACLs are stateless and use explicit inbound and outbound rules.

For initial EKS learning, understand NACLs, but focus more heavily on Security Groups.

---

## ENI

ENI means Elastic Network Interface.

It is a virtual network interface in AWS.

Conceptually:

```text
EC2
 |
ENI
 |
Private IP
 |
VPC
```

EKS networking uses ENIs heavily.

With the AWS VPC CNI, Pod networking is integrated with the AWS VPC.

---

## DNS

You will encounter multiple DNS layers.

Internet-facing DNS:

```text
Route 53
   |
   v
ALB DNS
```

Kubernetes internal DNS:

```text
Pod
 |
CoreDNS
 |
Kubernetes Service
```

---

# 3. EC2 / Managed Node Groups

EC2 provides compute capacity for EKS worker nodes.

```text
EKS
 |
Managed Node Group
 |
EC2
 |
Pod
```

---

## Instance Types

An EC2 instance type determines resources such as:

- CPU
- Memory
- Network capability
- Storage characteristics

Example:

```text
t3.medium
```

The appropriate instance type depends on workload requirements.

---

## AMI

AMI means Amazon Machine Image.

An EKS node AMI contains the operating system and software required for the node to function as a Kubernetes worker.

Conceptually:

```text
AMI
|
+-- Operating System
+-- kubelet
+-- Container runtime
+-- EKS node components
```

---

## Node Groups

A node group is a collection of worker nodes with similar configuration.

```text
application-node-group
|
+-- EC2
+-- EC2
+-- EC2
```

You can use different node groups for different workload types.

---

## Desired / Minimum / Maximum

Example:

```text
min     = 2
desired = 2
max     = 5
```

Initial state:

```text
2 nodes
```

The node group can scale within the configured range when an appropriate node autoscaling mechanism is configured.

---

## Node Scaling

There are two important concepts:

### Horizontal Pod Autoscaler

Scales application Pods.

```text
2 Pods -> 5 Pods
```

### Node Autoscaling

Provides additional compute capacity.

```text
2 Nodes -> 4 Nodes
```

Remember:

```text
HPA
|
+-- Pod scaling

Cluster/node autoscaling
|
+-- Compute capacity scaling
```

---

## Node Health

Check node status with:

```bash
kubectl get nodes
```

Possible states include:

```text
Ready
NotReady
```

For details:

```bash
kubectl describe node <node-name>
```

Investigate:

- EC2 health
- kubelet
- Networking
- Disk
- Memory
- CPU
- IAM
- VPC CNI

---

## Node Draining

Before maintenance or removal, workloads can be drained from a node.

Conceptually:

```text
Node
|
+-- Pod 1
+-- Pod 2
+-- Pod 3
```

After drain:

```text
Node
|
+-- No new workloads
```

Pods can be recreated or scheduled elsewhere according to their controllers and scheduling rules.

---

## Node Upgrades

Typical sequence:

```text
Upgrade EKS Control Plane
          |
          v
Check Add-ons
          |
          v
Upgrade Node Group
          |
          v
Drain / Replace Nodes
          |
          v
Verify Applications
```

---

# 4. Amazon ECR

ECR stores container images.

Your deployment pipeline is:

```text
Python Application
       |
       v
Docker Build
       |
       v
Docker Image
       |
       v
Amazon ECR
       |
       v
Amazon EKS
```

Your current repository is:

```text
public.ecr.aws/o5l5z4t1/my-app
```

---

## Repository

A repository stores container images.

```text
ECR
 |
my-app
```

---

## Image

The image packages the application and its runtime dependencies.

For your Flask application, the image can contain:

```text
Python 3.12
Flask
welcome.py
requirements
Application files
```

---

## Tags

Tags identify image versions.

Examples:

```text
my-app:latest
my-app:v1
my-app:v2
```

A Git commit SHA is another useful version identifier:

```text
my-app:a81c23f
```

For production deployments, immutable version identifiers such as commit SHAs are generally safer than relying only on `latest`.

---

## docker push

Your CI pipeline builds and pushes the image:

```bash
docker build -t <repository>:<tag> .
docker push <repository>:<tag>
```

---

## docker pull

The EKS node/container runtime obtains the required image from the registry.

```text
EKS Node
   |
Container Runtime
   |
ECR
   |
Docker Image
```

---

## ECR Authentication

The entity performing the push needs permission to publish to the repository.

For private ECR, EKS nodes/workloads also need appropriate permissions to pull images.

---

## Image Versioning

A good deployment pattern is:

```text
Git Commit
    |
    v
Docker Image
    |
    v
ECR
    |
    +-- my-app:8f31a72
```

Then Kubernetes can deploy that exact version.

---

# 5. IAM

IAM controls access to AWS resources.

Think:

```text
WHO
 |
WHAT ACTION
 |
ON WHICH RESOURCE
```

---

## IAM User

An IAM user represents a long-lived AWS identity.

For modern AWS environments, avoid unnecessary long-lived access keys and prefer temporary credentials/roles where possible.

---

## IAM Role

An IAM role is an identity that can be assumed by an appropriate principal.

Example:

```text
EKS Pod
   |
   v
IAM Role
   |
   v
AWS Service
```

---

## IAM Policy

A policy defines permissions.

Example concept:

```json
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject"
  ],
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

This allows the identity to read objects matching the resource.

---

## Trust Policy

A trust policy defines who or what is allowed to assume a role.

Conceptually:

```text
EKS Workload
     |
     v
IAM Role
```

The role's trust relationship must permit the relevant identity mechanism.

---

## EKS Access

A developer/operator needs appropriate permissions to access the EKS cluster.

Conceptually:

```text
Developer
   |
   v
AWS IAM
   |
   v
EKS Access
   |
   v
kubectl
```

AWS IAM permissions and Kubernetes authorization are related but distinct concepts.

---

## Node IAM Role

EC2 worker nodes require AWS permissions for tasks such as interacting with AWS APIs and supporting EKS networking and image retrieval.

Conceptually:

```text
EC2 Node
   |
   v
IAM Role
   |
   v
AWS APIs
```

Do not automatically give the node role every permission your application needs.

---

## EKS Pod Identity

EKS Pod Identity allows an EKS workload to obtain AWS permissions through an IAM role.

Example:

```text
Pod
 |
EKS Pod Identity
 |
IAM Role
 |
IAM Policy
 |
S3
```

If your Flask application needs to read an S3 object, this avoids embedding AWS access keys in the container.

---

## IRSA

IRSA means IAM Roles for Service Accounts.

It is an established EKS mechanism for assigning IAM roles to Kubernetes workloads.

You should understand IRSA because many existing EKS environments use it.

For new environments, also understand EKS Pod Identity.

---

# 6. ALB / Load Balancing

Load balancing exposes the application and distributes traffic.

For the Flask application:

```text
Browser
   |
   v
ALB
   |
   v
Kubernetes
   |
   v
Flask Pod
```

---

## Application Load Balancer

ALB operates at the application layer and supports HTTP/HTTPS routing.

It can route based on:

- Host
- Path
- Listener rules

Example:

```text
api.example.com
       |
       v
      ALB
       |
       v
API Application
```

For your Flask web application, ALB is the natural load balancer to learn first.

---

## Network Load Balancer

NLB is designed for Layer 4 network load balancing and supports TCP/UDP/TLS use cases.

Simple distinction:

```text
ALB
|
+-- HTTP/HTTPS application routing

NLB
|
+-- Network-level TCP/UDP/TLS traffic
```

---

## Target Groups

A target group contains targets that receive traffic.

Conceptually:

```text
ALB
 |
Target Group
 |
+-- Target 1
+-- Target 2
```

With EKS, the AWS Load Balancer Controller can configure AWS load-balancing resources based on Kubernetes configuration.

---

## Listener

A listener accepts incoming traffic on a configured port/protocol.

Example:

```text
ALB
 |
HTTPS :443
 |
Listener
 |
Target Group
```

Another listener may use:

```text
HTTP :80
```

and redirect traffic to HTTPS.

---

## Listener Rules

Rules determine how requests are routed.

Example:

```text
Host = myapp.example.com
        |
        v
Welcome Application
```

Another example:

```text
Path = /api/*
        |
        v
API Service
```

---

## Security Groups

The ALB can use a Security Group to control inbound traffic.

Example:

```text
Internet
   |
   v
ALB Security Group
   |
   v
ALB :443
```

Traffic from the ALB toward workloads should also be restricted appropriately.

---

## Health Checks

The ALB checks whether application targets are healthy.

Your Flask application exposes:

```text
GET /health
```

which returns a healthy response.

Example:

```text
ALB
 |
GET /health
 |
Pod
 |
200 OK
 |
Healthy
```

If the health check fails repeatedly, the target can be marked unhealthy.

---

## AWS Load Balancer Controller

The AWS Load Balancer Controller connects Kubernetes resources to AWS load-balancing services.

Conceptually:

```text
Kubernetes Ingress
       |
       v
AWS Load Balancer Controller
       |
       v
AWS ALB
```

This is a critical EKS concept.

---

## Kubernetes Ingress

Ingress defines HTTP/HTTPS routing rules.

Example:

```text
Internet
   |
   v
ALB
   |
   v
Ingress
   |
   v
welcome-service
   |
   v
Pods
```

---

# 7. Amazon Route 53

Route 53 is AWS's DNS service.

Example:

```text
app.example.com
       |
       v
Route 53
       |
       v
ALB
```

---

## Hosted Zone

A hosted zone contains DNS records for a domain.

Example:

```text
example.com
|
+-- app.example.com
+-- api.example.com
+-- www.example.com
```

---

## A Record

An A record maps a DNS name to an IPv4 address.

For AWS resources such as ALBs, an Alias record is typically more appropriate than manually managing changing IP addresses.

---

## CNAME

CNAME maps one DNS name to another DNS name.

Example:

```text
www.example.com
       |
       v
app.example.com
```

---

## Alias Record

Route 53 Alias records can point a DNS name to supported AWS resources such as an ALB.

Example:

```text
Route 53
    |
    v
ALB
```

You don't need to manually manage the ALB's changing IP addresses.

---

## DNS Resolution

When a user enters:

```text
https://app.example.com
```

the flow is approximately:

```text
Browser
   |
   v
DNS Resolver
   |
   v
Route 53
   |
   v
ALB DNS
```

The HTTP request then proceeds to the ALB.

---

# 8. Amazon RDS

RDS is AWS's managed relational database service.

For this application, use PostgreSQL as the example.

```text
Flask Pod
   |
   | TCP 5432
   v
RDS PostgreSQL
```

---

## PostgreSQL / MySQL

RDS supports several relational database engines.

For learning:

```text
Application
   |
   v
PostgreSQL
```

is a good choice.

Example:

```text
Database:
welcomeapp

Tables:
users
orders
```

---

## DB Subnet Group

RDS can be associated with a DB subnet group containing subnets across Availability Zones.

For a normal application architecture, keep the database in private/database subnets.

```text
VPC
 |
+-- Database Subnet AZ-A
|
+-- Database Subnet AZ-B
       |
       v
      RDS
```

---

## RDS Security Group

The RDS Security Group controls which network sources can connect to the database.

Example:

```text
Allow TCP 5432
from the application/network source
```

Then:

```text
Flask Application
       |
       | 5432
       v
RDS Security Group
       |
       v
PostgreSQL
```

---

## Private RDS

For an application database, RDS is normally kept private rather than directly accessible from the Internet.

Preferred architecture:

```text
Internet
   X
   |
RDS
```

instead:

```text
EKS
 |
Private Network
 |
RDS
```

---

## Backups

RDS provides automated backup and snapshot capabilities depending on configuration.

Backups allow recovery from data loss or operational mistakes.

---

## Multi-AZ

Multi-AZ improves database availability.

Conceptually:

```text
Availability Zone A
RDS Primary
      |
      | synchronous replication
      v
Availability Zone B
RDS Standby
```

Important:

Multi-AZ is primarily an availability/failover capability, not simply a read-scaling mechanism.

---

## Parameter Groups

Parameter groups control database engine configuration.

Examples include:

- Connection settings
- Logging settings
- Engine parameters

You don't need to memorize every database parameter.

Understand what parameter groups are used for.

---

# 9. AWS Secrets Manager

Secrets Manager securely stores sensitive values.

Examples:

- Database passwords
- API keys
- Third-party credentials

Do not put secrets directly in application source code or container images.

Bad example:

```yaml
env:
  DB_PASSWORD: "password123"
```

Preferred concept:

```text
AWS Secrets Manager
       |
       v
EKS Workload
       |
       v
Flask Application
```

---

## Secret

Example:

```text
welcomeapp/database
```

The secret may contain:

```text
username
password
host
database
```

---

## Secret Rotation

Secrets Manager supports secret rotation for supported services and configurations.

Rotation reduces the need to manually maintain long-lived credentials.

---

## IAM Integration

The Pod needs permission to read the secret.

Conceptually:

```text
Pod
 |
EKS Pod Identity
 |
IAM Role
 |
Secrets Manager
 |
Secret
```

This connects the IAM, EKS, and Secrets Manager concepts.

---

# 10. Amazon CloudWatch

CloudWatch provides AWS monitoring and observability capabilities.

Important areas:

```text
CloudWatch
|
+-- Logs
+-- Metrics
+-- Alarms
```

---

## CloudWatch Logs

Application and infrastructure logs can be centralized in CloudWatch when the appropriate EKS logging/observability configuration is enabled.

Conceptually:

```text
Flask Application
       |
       v
Container Logs
       |
       v
CloudWatch Logs
```

---

## Metrics

Metrics measure resource and service behavior.

Examples:

```text
CPU
Memory
Network
Request Count
Latency
Errors
```

You can monitor resources/services such as:

```text
EKS
EC2
ALB
RDS
```

depending on the configured observability integrations.

---

## Alarms

An alarm evaluates a metric against a configured condition.

Example:

```text
CPU > 80%
for 5 minutes
       |
       v
CloudWatch Alarm
```

The alarm can integrate with notification and incident-response workflows.

---

## EKS / Container Logs

For immediate Kubernetes troubleshooting:

```bash
kubectl logs <pod-name>
```

For centralized monitoring, configure the EKS observability/logging pipeline to send relevant logs to CloudWatch.

These are complementary:

```text
kubectl logs
|
+-- Immediate troubleshooting

CloudWatch
|
+-- Centralized monitoring / historical investigation
```

---

## Node Metrics

Useful node metrics include:

```text
CPU
Memory
Disk
Network
```

Example:

```text
Node CPU = 95%
       |
       v
Investigate workload
       |
       +-- Scale Pods
       |
       +-- Add nodes
```

---

## Application Logs

For the Flask application:

```text
Flask
 |
stdout / stderr
 |
Container logs
 |
CloudWatch
```

Useful application logs include:

```text
Timestamp
Request
Status
Latency
Error
```

Never log passwords, access keys, tokens, or other sensitive values.

---

# Complete Application Architecture

Now connect everything.

```text
                         INTERNET
                             |
                             v
                       Route 53 DNS
                             |
                             v
                          AWS ALB
                             |
                             v
                AWS Load Balancer Controller
                             |
                             v
                     Kubernetes Ingress
                             |
                             v
                    Kubernetes Service
                             |
                   +---------+---------+
                   |                   |
                   v                   v
                 Pod 1               Pod 2
                   |                   |
                   +---------+---------+
                             |
                         EKS Nodes
                             |
                    Managed Node Group
                             |
                            EC2
                             |
                       Private Subnet
                             |
                            VPC
                             |
                     +-------+-------+
                     |               |
                     v               v
                   NAT             RDS
                     |           PostgreSQL
                     v
                  Internet
```

Supporting services:

```text
                    +----------------+
                    |      IAM       |
                    +----------------+
                           |
                 +---------+---------+
                 |                   |
                 v                   v
           EKS Pod Identity       ECR
                 |                   |
                 v                   v
          AWS Services          Docker Images


                    +----------------+
                    |    Secrets     |
                    |    Manager     |
                    +----------------+
                           |
                           v
                         EKS Pod


                    +----------------+
                    |   CloudWatch   |
                    +----------------+
                           |
              +------------+------------+
              |            |            |
             EKS          ALB          RDS
           /EC2        Metrics/Logs   Metrics
```

---

# Complete Request Flow

Suppose the user opens:

```text
https://app.example.com
```

## Step 1 — Route 53

```text
app.example.com
       |
       v
Route 53
       |
       v
ALB
```

DNS resolves the application hostname to the ALB endpoint.

## Step 2 — ALB

The ALB receives HTTPS traffic.

```text
Internet
   |
   v
ALB :443
```

## Step 3 — Ingress

The AWS Load Balancer Controller manages the ALB based on Kubernetes configuration.

```text
ALB
 |
 v
Ingress
```

## Step 4 — Service

Ingress routes traffic to:

```text
welcome-service
```

The Service selects the appropriate Pods.

```text
Service
 |
 +-- Pod 1
 |
 +-- Pod 2
```

## Step 5 — Pod

The Pod runs the Flask container.

```text
Pod
 |
Container
 |
welcome.py
 |
Port 5000
```

## Step 6 — Database

If the application needs database data:

```text
Flask Pod
   |
   | PostgreSQL :5432
   v
RDS PostgreSQL
```

The RDS Security Group controls whether the connection is allowed.

## Step 7 — Credentials

The application gets database credentials through the appropriate secret/IAM integration.

```text
Pod
 |
EKS Pod Identity
 |
IAM Role
 |
Secrets Manager
 |
Database Credentials
```

## Step 8 — Logs and Monitoring

```text
Flask
 |
Container Logs
 |
CloudWatch
```

Infrastructure metrics can include:

```text
ALB
EKS
EC2
RDS
 |
v
CloudWatch
```

---

# Deployment Flow

Before the user can access the application, the image has to be built and stored.

```text
Developer
    |
    v
GitHub
    |
    v
GitHub Actions
    |
    v
Docker Build
    |
    v
Docker Image
    |
    v
Amazon ECR
    |
    v
EKS Node
    |
    v
Container
    |
    v
Pod
```

Then the application becomes accessible:

```text
User
 |
 v
Route 53
 |
 v
ALB
 |
 v
Ingress
 |
 v
Service
 |
 v
Pod
 |
 v
RDS
```

---

# Interview-Level Explanation

A useful answer to:

> Explain how you deployed your application on EKS.

is:

> We have a Flask application packaged as a Docker image. GitHub Actions builds the image and pushes it to Amazon ECR. The EKS cluster runs inside a VPC, with worker nodes in private subnets. The application is deployed using a Kubernetes Deployment, which maintains multiple Pods. A Kubernetes Service provides stable access to those Pods. We use an Ingress with the AWS Load Balancer Controller, which provisions an Application Load Balancer. Route 53 resolves the application domain to the ALB. The application connects privately to PostgreSQL running on RDS. IAM and EKS Pod Identity provide AWS permissions to workloads, while database credentials are stored in Secrets Manager. CloudWatch provides centralized logs, metrics, and alarms.

---

# Recommended Learning Order

Do not learn these services randomly.

Follow this order:

```text
1. EKS
      |
      v
2. Kubernetes
   Pod / Deployment / Service / Ingress
      |
      v
3. VPC
      |
      v
4. EC2 / Managed Node Groups
      |
      v
5. ECR
      |
      v
6. IAM
      |
      v
7. ALB + AWS Load Balancer Controller
      |
      v
8. Route 53
      |
      v
9. RDS
      |
      v
10. Secrets Manager
      |
      v
11. CloudWatch
```

The real objective is not memorizing eleven AWS services.

You should be able to trace the entire lifecycle:

```text
GitHub
  ↓
GitHub Actions
  ↓
Docker
  ↓
ECR
  ↓
EKS
  ↓
VPC
  ↓
EC2 / Node
  ↓
Pod
  ↓
Service
  ↓
Ingress
  ↓
ALB
  ↓
Route 53
  ↓
User
```

And, when the application needs data:

```text
Pod
  ↓
RDS PostgreSQL
```

With:

```text
IAM
Secrets Manager
CloudWatch
```

supporting the application.
