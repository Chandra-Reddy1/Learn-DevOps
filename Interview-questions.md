# Interview Question Bank
## AWS & DevOps — Complete Question Bank
### All Topics Covered | Questions + Answers

---

> 📌 **Total Topics: 15 | Total Questions: 150+**
> 🎯 Covers: Basic → Intermediate → Advanced
> 📅 Prepared: March 2026

---

## Table of Contents

1. [AWS EC2](#1-aws-ec2--elastic-compute-cloud)
2. [AWS S3](#2-aws-s3--simple-storage-service)
3. [AWS VPC](#3-aws-vpc--virtual-private-cloud)
4. [AWS RDS](#4-aws-rds--relational-database-service)
5. [Amazon EKS](#5-amazon-eks--elastic-kubernetes-service)
6. [ELB / ALB](#6-elb--alb--load-balancers)
7. [Terraform](#7-terraform--infrastructure-as-code)
8. [Ansible](#8-ansible--configuration-management)
9. [AWS CloudFormation](#9-aws-cloudformation)
10. [CI/CD — Jenkins & GitHub Actions](#10-cicd--jenkins--github-actions)
11. [High Availability & Disaster Recovery](#11-high-availability-ha--disaster-recovery-dr)
12. [AWS Security & IAM](#12-aws-security--iam)
13. [AWS Cost Optimization](#13-aws-cost-optimization)
14. [Cloud Migration](#14-cloud-migration)
15. [Linux Administration](#15-linux-administration)

---

## 1. AWS EC2 — Elastic Compute Cloud

---

**Q1. What is AWS EC2?**

> EC2 (Elastic Compute Cloud) is a web service that provides resizable virtual servers (instances) in the cloud. You can launch, stop, start, and terminate instances on demand — paying only for what you use.

---

**Q2. What are EC2 instance types and families?**

> EC2 instances are grouped into families based on use case:
> - **t3/t4g** — General purpose, burstable (dev/test)
> - **m5/m6g** — General purpose production workloads
> - **c5/c6g** — Compute optimized (CPU-heavy tasks)
> - **r5/r6g** — Memory optimized (large DBs, in-memory cache)
> - **p3/p4** — GPU instances (ML, AI, video rendering)
> - **i3/i4** — Storage optimized (high IOPS workloads)

---

**Q3. What is the difference between Stop, Terminate, and Hibernate?**

| Action | Effect | Data |
|--------|--------|------|
| **Stop** | Instance shuts down, can restart | EBS data kept |
| **Terminate** | Instance deleted permanently | EBS deleted (default) |
| **Hibernate** | RAM saved to EBS, fast resume | RAM + EBS kept |

---

**Q4. What are EC2 pricing models?**

> - **On-Demand** — Pay per hour/second, no commitment
> - **Reserved (1yr/3yr)** — Commit and save 40–60%
> - **Spot** — Use spare capacity, up to 90% cheaper, can be interrupted
> - **Savings Plans** — Flexible commitment, save 40%
> - **Dedicated Host** — Physical server for compliance/licensing

---

**Q5. What is an AMI?**

> AMI (Amazon Machine Image) is a pre-configured template containing OS, software, and settings used to launch EC2 instances. You can use AWS-provided AMIs, Marketplace AMIs, or create custom AMIs from your own instances.

---

**Q6. What is the difference between EBS and Instance Store?**

| | EBS | Instance Store |
|--|-----|---------------|
| Persistence | Persists after stop | Lost on stop/terminate |
| Speed | Fast | Faster (physically attached) |
| Backup | Snapshots to S3 | No backup option |
| Use case | Production, databases | Temp cache, buffer |

---

**Q7. What is EC2 Auto Scaling?**

> Auto Scaling automatically adds or removes EC2 instances based on demand using scaling policies. It ensures the right number of instances are always running — scaling out during high traffic and scaling in during low traffic to save cost.

---

**Q8. What is a Security Group and how does it work?**

> A Security Group is a stateful virtual firewall at the instance level. It controls inbound and outbound traffic using Allow rules only. Being stateful means if inbound traffic is allowed, the response is automatically allowed outbound.

---

**Q9. What is the difference between vertical and horizontal scaling?**

> - **Vertical Scaling** — Upgrade the instance size (t3.micro → m5.large). Has limits and requires restart.
> - **Horizontal Scaling** — Add more instances. Unlimited scale, no downtime. Preferred for production.

---

**Q10. What is EC2 User Data?**

> User Data is a script that runs automatically when an EC2 instance first launches. Used to install software, configure settings, or bootstrap the instance without manual intervention.

```bash
#!/bin/bash
yum update -y
yum install nginx -y
systemctl start nginx
```

---

## 2. AWS S3 — Simple Storage Service

---

**Q11. What is AWS S3?**

> S3 (Simple Storage Service) is AWS's object storage service. You store files (objects) inside containers (buckets). It offers 99.999999999% (11 nines) durability, unlimited storage, and global accessibility.

---

**Q12. What are S3 storage classes?**

| Class | Access | Cost | Use Case |
|-------|--------|------|---------|
| **Standard** | Frequent | High | Active data |
| **Standard-IA** | Infrequent | Medium | Monthly access |
| **One Zone-IA** | Infrequent | Lower | Non-critical data |
| **Glacier** | Rare | Low | Archives |
| **Glacier Deep Archive** | Very rare | Lowest | Compliance archives |
| **Intelligent-Tiering** | Unknown pattern | Auto | Auto cost optimization |

---

**Q13. What is S3 versioning?**

> Versioning keeps multiple versions of the same object. When enabled, every overwrite or delete creates a new version instead of permanently removing the object. This protects against accidental deletion and allows rollback.

---

**Q14. What is an S3 bucket policy vs ACL?**

> - **Bucket Policy** — JSON-based resource policy attached to a bucket. Controls access for IAM users, roles, accounts, and public access. More powerful and recommended.
> - **ACL (Access Control List)** — Legacy method, grants basic read/write permissions. AWS recommends disabling ACLs and using bucket policies instead.

---

**Q15. What is S3 Cross-Region Replication (CRR)?**

> CRR automatically replicates objects from a source bucket in one region to a destination bucket in another region. Used for disaster recovery, compliance, and reducing latency for global users. Requires versioning enabled on both buckets.

---

**Q16. How do you secure an S3 bucket?**

> - Block all public access (enabled by default)
> - Use bucket policies to restrict access
> - Enable S3 encryption (SSE-S3, SSE-KMS, or SSE-C)
> - Enable versioning to protect against deletion
> - Use VPC Endpoints to access S3 privately
> - Enable access logging and CloudTrail for audit

---

**Q17. What is S3 lifecycle policy?**

> A lifecycle policy automates transitioning objects between storage classes or deleting them after a defined period.
> Example: Move to Standard-IA after 30 days → Glacier after 90 days → Delete after 365 days.

---

**Q18. What is the maximum size of an S3 object?**

> Single object: **5 TB** maximum.
> Single PUT upload: **5 GB** maximum.
> For objects larger than 5 GB, you must use **Multipart Upload** which splits the file into parts and uploads them in parallel.

---

**Q19. What is S3 Transfer Acceleration?**

> S3 Transfer Acceleration speeds up uploads to S3 by routing traffic through AWS CloudFront edge locations. Data enters AWS's fast backbone network at the nearest edge location instead of traveling across the public internet to the S3 bucket region.

---

**Q20. What is the difference between S3 and EBS?**

| | S3 | EBS |
|--|----|----|
| Type | Object storage | Block storage |
| Access | Via HTTP/API | Mounted to EC2 |
| Use case | Files, backups, static websites | OS, databases, apps |
| Scalability | Unlimited | Up to 64 TB per volume |
| Availability | Multi-AZ by default | Single AZ |

---

## 3. AWS VPC — Virtual Private Cloud

---

**Q21. What is a VPC?**

> VPC (Virtual Private Cloud) is a logically isolated virtual network within AWS where you launch resources. You control the IP range, subnets, route tables, gateways, and security settings completely.

---

**Q22. What is the difference between Public and Private Subnet?**

> - **Public Subnet** — Has a route to the Internet Gateway. Resources can communicate with the internet directly.
> - **Private Subnet** — No route to IGW. Resources cannot be reached from internet. Use NAT Gateway for outbound-only internet access.

---

**Q23. What is an Internet Gateway?**

> An Internet Gateway (IGW) is a VPC component that allows communication between the VPC and the internet. It is horizontally scaled, redundant, and highly available. Only one IGW can be attached per VPC.

---

**Q24. What is a NAT Gateway?**

> NAT Gateway allows instances in a private subnet to initiate outbound internet connections (for updates/patches) while preventing the internet from initiating inbound connections. It must be placed in a public subnet with an Elastic IP.

---

**Q25. What is the difference between Security Group and NACL?**

| | Security Group | NACL |
|--|---------------|------|
| Level | Instance | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow + Deny |
| Evaluation | All rules | Lowest number first |

---

**Q26. What is VPC Peering?**

> VPC Peering is a networking connection between two VPCs that allows private IP communication. Traffic stays on AWS backbone. No transitive peering — if A peers B and B peers C, A cannot reach C via B.

---

**Q27. What is a VPC Endpoint?**

> A VPC Endpoint allows private connections to AWS services (S3, DynamoDB, etc.) without requiring IGW, NAT Gateway, or internet. Two types: Gateway (S3/DynamoDB — free) and Interface (other services — paid).

---

**Q28. What is Transit Gateway?**

> Transit Gateway is a hub-and-spoke network transit service that connects multiple VPCs and on-premises networks through a single gateway. It eliminates complex peering meshes and supports transitive routing — something VPC Peering cannot do.

---

**Q29. How many IPs does AWS reserve per subnet?**

> AWS reserves 5 IPs per subnet: network address (.0), VPC router (.1), DNS (.2), reserved for future (.3), and broadcast (.255). A /24 subnet gives 251 usable IPs, not 256.

---

**Q30. What is a Bastion Host?**

> A Bastion Host is an EC2 instance in a public subnet that serves as a secure entry point (jump server) for SSH/RDP access to private subnet instances. It exposes only port 22 to a restricted admin IP.

---

## 4. AWS RDS — Relational Database Service

---

**Q31. What is AWS RDS?**

> RDS (Relational Database Service) is a managed database service where AWS handles OS patching, DB installation, backups, failover, and scaling. You focus on schema design and queries, not infrastructure.

---

**Q32. What database engines does RDS support?**

> MySQL, PostgreSQL, MariaDB, Oracle, Microsoft SQL Server, and Amazon Aurora (MySQL-compatible and PostgreSQL-compatible).

---

**Q33. What is Multi-AZ in RDS?**

> Multi-AZ maintains a synchronous standby replica in a different AZ. If the primary fails, AWS automatically fails over to the standby (~60 seconds). Same endpoint — no application change needed. Zero data loss (RPO = 0).

---

**Q34. What is a Read Replica?**

> A Read Replica is a read-only asynchronous copy of the primary DB used to offload read-heavy traffic. It has its own endpoint, can be in a different region, and can be manually promoted to a standalone DB if needed.

---

**Q35. What is the difference between Multi-AZ and Read Replica?**

| | Multi-AZ | Read Replica |
|--|----------|-------------|
| Purpose | High Availability | Read Scaling |
| Replication | Synchronous | Asynchronous |
| Readable | No | Yes |
| Auto Failover | Yes | No |

---

**Q36. What is RDS automated backup and PITR?**

> RDS takes daily automated snapshots plus continuous transaction logs. PITR (Point-in-Time Recovery) allows you to restore your database to any specific second within the backup retention window (1–35 days).

---

**Q37. How do you encrypt an existing unencrypted RDS instance?**

> You cannot encrypt in place. Process: take snapshot → copy snapshot with encryption enabled → restore new instance from encrypted snapshot → update app to point to new instance → delete old instance.

---

**Q38. What is RDS Proxy?**

> RDS Proxy is a managed connection pooler between your application and RDS. It reduces DB connections (critical for Lambda), improves failover speed by 66%, and enforces IAM authentication. Ideal for serverless workloads.

---

**Q39. What is Amazon Aurora?**

> Aurora is AWS's cloud-native relational database. It is 5× faster than MySQL and 3× faster than PostgreSQL. Storage replicates 6 copies across 3 AZs automatically, supports up to 15 read replicas, and has near-zero failover time (~30 sec).

---

**Q40. How do you secure an RDS instance?**

> Place in private subnet, allow only app server security group on DB port, enable encryption at rest (KMS) and in transit (SSL), store credentials in Secrets Manager with auto-rotation, and enable enhanced monitoring + audit logs.

---

## 5. Amazon EKS — Elastic Kubernetes Service

---

**Q41. What is Amazon EKS?**

> EKS (Elastic Kubernetes Service) is AWS's fully managed Kubernetes service. AWS manages the control plane (API server, etcd, scheduler) while you manage the worker nodes. It eliminates the complexity of running your own Kubernetes control plane.

---

**Q42. What is the difference between EKS and ECS?**

| | EKS | ECS |
|--|-----|-----|
| Orchestration | Kubernetes | AWS-native |
| Portability | High (K8s standard) | AWS-only |
| Complexity | Higher | Lower |
| Community | Huge K8s ecosystem | AWS ecosystem |
| Use case | Multi-cloud, K8s teams | AWS-native container apps |

---

**Q43. What are EKS node types?**

> - **Managed Node Groups** — AWS provisions and manages EC2s for worker nodes
> - **Self-managed Nodes** — You provision and manage EC2 worker nodes manually
> - **AWS Fargate** — Serverless — no nodes to manage, pay per pod

---

**Q44. What is a Kubernetes Pod?**

> A Pod is the smallest deployable unit in Kubernetes. It contains one or more containers that share the same network namespace, storage, and lifecycle. All containers in a pod communicate via localhost.

---

**Q45. What is a Kubernetes Deployment?**

> A Deployment manages a desired number of Pod replicas, ensures they are running, performs rolling updates, and rolls back if something goes wrong. It is the standard way to run stateless applications in Kubernetes.

---

**Q46. What is a Kubernetes Service?**

> A Service provides a stable network endpoint (DNS name + IP) to access a group of Pods. Types: ClusterIP (internal), NodePort (exposed on node port), LoadBalancer (creates an AWS ALB/NLB), ExternalName (maps to external DNS).

---

**Q47. What is a ConfigMap and Secret in Kubernetes?**

> - **ConfigMap** — Stores non-sensitive configuration data (env vars, config files) as key-value pairs
> - **Secret** — Stores sensitive data (passwords, tokens, keys) in base64-encoded format. Should be encrypted with KMS in production.

---

**Q48. What is Helm in Kubernetes?**

> Helm is the package manager for Kubernetes. It uses Charts (pre-packaged Kubernetes manifests) to deploy complex applications with a single command. Like apt/yum but for Kubernetes applications.

---

**Q49. What is the EKS control plane?**

> The EKS control plane consists of the Kubernetes API server, etcd (distributed state store), scheduler, and controller manager. AWS fully manages this, ensuring high availability across multiple AZs. You only pay per cluster per hour.

---

**Q50. How does EKS integrate with AWS services?**

> EKS integrates with ALB (via AWS Load Balancer Controller), IAM (via IRSA — IAM Roles for Service Accounts), ECR (container registry), CloudWatch (logging/monitoring), VPC (networking), and EBS/EFS (persistent storage).

---

## 6. ELB / ALB — Load Balancers

---

**Q51. What is ELB and what are its types?**

> ELB (Elastic Load Balancer) distributes incoming traffic across multiple targets. Types:
> - **ALB** — Application Load Balancer (Layer 7, HTTP/HTTPS)
> - **NLB** — Network Load Balancer (Layer 4, TCP/UDP, ultra-low latency)
> - **GLB** — Gateway Load Balancer (Layer 3, security appliances)
> - **CLB** — Classic Load Balancer (legacy, avoid for new apps)

---

**Q52. What is ALB and when do you use it?**

> ALB (Application Load Balancer) operates at Layer 7 and routes traffic based on URL path, hostname, headers, and query strings. Use for web apps, REST APIs, microservices, and containerized applications on ECS/EKS.

---

**Q53. What is path-based routing in ALB?**

> ALB routes requests to different target groups based on the URL path. Example: `/api/*` routes to API servers, `/images/*` routes to media servers — all on the same ALB with one domain.

---

**Q54. What is host-based routing in ALB?**

> ALB routes requests based on the hostname in the HTTP header. Example: `api.myapp.com` routes to API servers, `admin.myapp.com` routes to admin servers — all handled by one ALB.

---

**Q55. What is a Target Group?**

> A Target Group is a logical group of targets (EC2 instances, IPs, Lambda functions, or containers) that ALB routes traffic to. Each target group has its own health check configuration, protocol, and port.

---

**Q56. What is SSL termination in ALB?**

> SSL termination means the ALB handles HTTPS encryption and decryption. Your backend EC2 instances receive plain HTTP — reducing CPU overhead on servers. The SSL certificate is attached to the ALB listener (via ACM or imported).

---

**Q57. What is the difference between ALB and NLB?**

| | ALB | NLB |
|--|-----|-----|
| Layer | 7 (Application) | 4 (Transport) |
| Protocol | HTTP/HTTPS | TCP/UDP/TLS |
| Routing | URL, headers, host | IP + Port only |
| Latency | Milliseconds | Ultra-low (microseconds) |
| Use case | Web apps, APIs | Gaming, IoT, real-time |

---

**Q58. What is ALB health check?**

> ALB periodically sends HTTP requests to registered targets to check if they are healthy. If a target fails the health check (returns non-2xx or times out), ALB stops routing traffic to it. When it recovers, it is automatically re-added.

---

**Q59. What is connection draining (deregistration delay)?**

> When a target is deregistered or unhealthy, ALB stops sending new requests but waits for in-flight requests to complete before fully removing the target. Default: 300 seconds. This enables zero-downtime deployments.

---

**Q60. How does ALB integrate with Auto Scaling?**

> Auto Scaling registers new EC2 instances with the ALB target group automatically when they launch, and deregisters them when they terminate. ALB only routes to healthy, registered instances — seamlessly handling scale events.

---

## 7. Terraform — Infrastructure as Code

---

**Q61. What is Terraform?**

> Terraform is an open-source Infrastructure as Code (IaC) tool by HashiCorp. It lets you define cloud infrastructure in declarative HCL (HashiCorp Configuration Language) files and provision it across any cloud provider using CLI commands.

---

**Q62. What are the core Terraform commands?**

| Command | Purpose |
|---------|---------|
| `terraform init` | Initialize, download providers |
| `terraform plan` | Preview changes before applying |
| `terraform apply` | Create/update infrastructure |
| `terraform destroy` | Delete all managed resources |
| `terraform validate` | Check config syntax |
| `terraform fmt` | Format code consistently |
| `terraform state` | Manage state file |

---

**Q63. What is Terraform state?**

> Terraform state is a JSON file (terraform.tfstate) that maps your configuration to real-world resources. It tracks what exists, what changed, and what needs updating. In teams, state is stored remotely in S3 with DynamoDB locking to prevent conflicts.

---

**Q64. What is a Terraform provider?**

> A provider is a plugin that translates Terraform configuration into API calls for a specific platform (AWS, Azure, GCP, Kubernetes). Each provider must be declared and initialized with `terraform init`.

```hcl
provider "aws" {
  region = "us-east-1"
}
```

---

**Q65. What is the difference between terraform plan and terraform apply?**

> - `terraform plan` — Shows what changes WILL happen without making any changes. Safe to run anytime.
> - `terraform apply` — Actually executes the changes. Creates, updates, or deletes real infrastructure.

---

**Q66. What are Terraform modules?**

> Modules are reusable packages of Terraform configurations. A module groups related resources (e.g., a VPC module with subnets, route tables, and IGW). They promote DRY (Don't Repeat Yourself) and consistency across environments.

---

**Q67. What is a Terraform backend?**

> A backend defines where Terraform stores its state file. Default: local file. Production: remote backend like S3 + DynamoDB for team collaboration, state locking, and versioning.

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock"
  }
}
```

---

**Q68. What is the difference between terraform.tfvars and variables.tf?**

> - `variables.tf` — Declares variable names, types, and default values
> - `terraform.tfvars` — Assigns actual values to those variables for a specific environment. Never commit sensitive values to git.

---

**Q69. What are Terraform workspaces?**

> Workspaces allow multiple state files within the same configuration — enabling separate environments (dev, staging, prod) from the same Terraform code. Each workspace has its own isolated state file.

---

**Q70. What is the difference between Terraform and CloudFormation?**

| | Terraform | CloudFormation |
|--|-----------|---------------|
| Provider | Multi-cloud | AWS only |
| Language | HCL | YAML/JSON |
| State | Managed by you | Managed by AWS |
| Community | Large | AWS-focused |
| Drift detection | Manual | Built-in |

---

## 8. Ansible — Configuration Management

---

**Q71. What is Ansible?**

> Ansible is an open-source configuration management, application deployment, and task automation tool. It uses YAML-based playbooks and is agentless — it connects to servers via SSH and executes tasks without installing any agent software.

---

**Q72. What is an Ansible Playbook?**

> A Playbook is a YAML file that defines a set of tasks to be executed on target hosts. It describes the desired state of the system — installing packages, copying files, starting services — in a human-readable format.

---

**Q73. What is Ansible Inventory?**

> Inventory is a file (INI or YAML) that lists the target hosts Ansible manages. It can group hosts (web servers, DB servers) and define variables per host or group. Can be static (file) or dynamic (pulled from AWS, GCP, etc.).

---

**Q74. What is an Ansible Role?**

> A Role is a structured, reusable way to organize playbooks. It separates tasks, variables, handlers, templates, and files into a standard directory structure. Roles promote reuse across projects and teams.

---

**Q75. What is the difference between Ansible and Terraform?**

| | Ansible | Terraform |
|--|---------|-----------|
| Purpose | Configuration management | Infrastructure provisioning |
| When used | After infrastructure exists | To create infrastructure |
| Language | YAML | HCL |
| State | Stateless | Stateful |
| Example | Install Nginx on EC2 | Create the EC2 instance |

---

**Q76. What are Ansible modules?**

> Modules are the units of work in Ansible — each module performs a specific task. Examples: `yum` (install packages), `copy` (copy files), `service` (manage services), `template` (render Jinja2 templates), `aws_ec2` (manage EC2).

---

**Q77. What is an Ansible Handler?**

> A Handler is a task that only runs when notified by another task. Used for actions that should only happen when something changes — like restarting Nginx only if the config file changed.

---

**Q78. What is Ansible Vault?**

> Ansible Vault encrypts sensitive data (passwords, API keys) in playbooks and variable files using AES-256 encryption. Encrypted files can be safely stored in version control.

```bash
ansible-vault encrypt secrets.yml
ansible-vault decrypt secrets.yml
ansible-playbook site.yml --ask-vault-pass
```

---

**Q79. What is idempotency in Ansible?**

> Idempotency means running the same playbook multiple times produces the same result — it only makes changes if the current state differs from the desired state. Running twice does not break anything or duplicate actions.

---

**Q80. What is an Ansible dynamic inventory?**

> Dynamic inventory automatically discovers and lists hosts from external sources like AWS EC2, Azure, or GCP instead of maintaining a static file. AWS dynamic inventory pulls all running EC2 instances and groups them by tags, region, or instance type.

---

## 9. AWS CloudFormation

---

**Q81. What is AWS CloudFormation?**

> CloudFormation is AWS's native Infrastructure as Code service. You define AWS resources in JSON or YAML templates, and CloudFormation provisions and manages them as stacks — handling dependencies and rollback automatically.

---

**Q82. What is a CloudFormation Stack?**

> A Stack is a collection of AWS resources managed as a single unit from a CloudFormation template. You create, update, and delete all resources in a stack together. If a stack creation fails, CloudFormation automatically rolls back all changes.

---

**Q83. What are CloudFormation template sections?**

| Section | Required | Purpose |
|---------|---------|---------|
| **AWSTemplateFormatVersion** | No | Template version |
| **Description** | No | Template description |
| **Parameters** | No | Input values at deploy time |
| **Mappings** | No | Static lookup tables |
| **Conditions** | No | Conditional resource creation |
| **Resources** | YES | AWS resources to create |
| **Outputs** | No | Values to export |

---

**Q84. What are CloudFormation Parameters?**

> Parameters allow you to pass custom values into your template at stack creation/update time — making templates reusable across environments (dev, staging, prod) without hardcoding values.

---

**Q85. What is CloudFormation StackSets?**

> StackSets allow you to deploy CloudFormation stacks across multiple AWS accounts and regions simultaneously using a single template. Useful for organizations that need to enforce consistent infrastructure (security baselines, logging) across all accounts.

---

**Q86. What is a CloudFormation Change Set?**

> A Change Set is a preview of what changes CloudFormation will make to your stack before you execute them. Like `terraform plan` — it shows additions, modifications, and deletions without making actual changes.

---

**Q87. What is CloudFormation drift detection?**

> Drift detection identifies when actual resource configurations differ from what CloudFormation expects based on the template. For example, if someone manually changed a Security Group rule, drift detection flags it.

---

**Q88. What is the difference between CloudFormation and Terraform?**

| | CloudFormation | Terraform |
|--|---------------|-----------|
| Scope | AWS only | Multi-cloud |
| State management | AWS managed | Self-managed (S3) |
| Language | YAML/JSON | HCL |
| Rollback | Automatic | Manual |
| Drift detection | Built-in | Manual (terraform plan) |

---

**Q89. What are CloudFormation Nested Stacks?**

> Nested Stacks allow you to reference other CloudFormation templates as resources within a parent template. This breaks large templates into smaller, reusable components — similar to modules in Terraform.

---

**Q90. What is the CloudFormation resource deletion policy?**

> DeletionPolicy controls what happens to a resource when the stack is deleted:
> - **Delete** (default) — Resource is deleted with the stack
> - **Retain** — Resource is kept even after stack deletion
> - **Snapshot** — Takes a snapshot before deletion (RDS, EBS)

---

## 10. CI/CD — Jenkins & GitHub Actions

---

**Q91. What is CI/CD?**

> - **CI (Continuous Integration)** — Automatically build and test code on every commit
> - **CD (Continuous Delivery)** — Automatically deploy tested code to staging/production
> Together they create a pipeline that takes code from developer's laptop to production automatically and reliably.

---

**Q92. What is Jenkins?**

> Jenkins is an open-source automation server for building CI/CD pipelines. It supports thousands of plugins, can run on any server, and orchestrates building, testing, and deploying applications through Freestyle jobs or Jenkinsfile pipelines.

---

**Q93. What is a Jenkinsfile?**

> A Jenkinsfile is a text file (checked into version control) that defines a Jenkins Pipeline as code. It uses Groovy DSL and defines stages like Build, Test, Deploy. Keeps pipeline config in the same repo as application code.

```groovy
pipeline {
  agent any
  stages {
    stage('Build')  { steps { sh 'mvn package' } }
    stage('Test')   { steps { sh 'mvn test' } }
    stage('Deploy') { steps { sh './deploy.sh' } }
  }
}
```

---

**Q94. What is the difference between Declarative and Scripted Pipeline in Jenkins?**

> - **Declarative** — Structured, opinionated syntax with `pipeline {}` block. Easier to read, write, and maintain. Recommended for most teams.
> - **Scripted** — Flexible Groovy-based syntax. More powerful but complex. Used for advanced customization.

---

**Q95. What is GitHub Actions?**

> GitHub Actions is GitHub's built-in CI/CD platform. Workflows are defined in YAML files stored in `.github/workflows/`. Triggered by events (push, PR, schedule), they run jobs on GitHub-hosted or self-hosted runners.

---

**Q96. What is a GitHub Actions Workflow?**

> A Workflow is a YAML file defining automated processes. It contains:
> - **on** — trigger events (push, pull_request)
> - **jobs** — units of work that run in parallel or sequence
> - **steps** — individual tasks within a job (actions or shell commands)

---

**Q97. What is the difference between Jenkins and GitHub Actions?**

| | Jenkins | GitHub Actions |
|--|---------|---------------|
| Hosting | Self-hosted | Cloud (GitHub) |
| Setup | Complex | Simple (YAML in repo) |
| Cost | Free (server cost) | Free for public; paid for private |
| Integration | Plugin-based | Native GitHub integration |
| Maintenance | You manage server | GitHub manages |

---

**Q98. What are GitHub Actions secrets?**

> Secrets are encrypted variables stored in GitHub repository or organization settings. They are available to workflows as environment variables but never exposed in logs. Used for AWS credentials, API keys, and passwords.

---

**Q99. What is a CI/CD pipeline for a Docker app on AWS?**

```
Developer pushes code to GitHub
        │
        ▼
GitHub Actions triggers workflow
        │
        ▼
Build Docker image
        │
        ▼
Run automated tests
        │
        ▼
Push image to ECR (AWS Container Registry)
        │
        ▼
Deploy to ECS / EKS
        │
        ▼
Health check → Notify team ✅
```

---

**Q100. What is a blue-green deployment?**

> Blue-green deployment runs two identical environments (blue = current, green = new). Traffic is switched from blue to green all at once after testing. If issues arise, traffic instantly switches back to blue — zero downtime and instant rollback.

---

## 11. High Availability (HA) & Disaster Recovery (DR)

---

**Q101. What is the difference between HA and DR?**

> - **HA** — Keeps the application running continuously by eliminating single points of failure (multiple AZs, Auto Scaling, Multi-AZ RDS). Protects against component/AZ failures.
> - **DR** — Recovers the application after a catastrophic event (region failure, data corruption). Protects against large-scale disasters.

---

**Q102. What is RTO and RPO?**

> - **RTO (Recovery Time Objective)** — Maximum acceptable time the app can be down after a disaster. "How fast must we recover?"
> - **RPO (Recovery Point Objective)** — Maximum acceptable data loss measured in time. "How much data can we afford to lose?"

---

**Q103. What are the 4 DR strategies?**

| Strategy | RTO | RPO | Cost |
|----------|-----|-----|------|
| Backup & Restore | Hours | Hours | Lowest |
| Pilot Light | 10–30 min | Minutes | Low |
| Warm Standby | Minutes | Seconds | Medium |
| Multi-Site Active-Active | ~Zero | ~Zero | Highest |

---

**Q104. What AWS services provide HA?**

> ALB (traffic distribution), EC2 Auto Scaling (replace failed instances), Multi-AZ RDS (DB failover), Route 53 (DNS health checks), S3 (11 nines durability), ElastiCache Multi-AZ, and deploying across multiple AZs.

---

**Q105. What AWS services support DR?**

> S3 Cross-Region Replication, RDS automated backups and snapshots, Aurora Global Database, AWS Backup, Route 53 failover routing, EC2 AMI cross-region copy, and DynamoDB Global Tables.

---

**Q106. What is an Availability Zone (AZ)?**

> An AZ is one or more discrete data centers within an AWS region, each with independent power, cooling, and networking. They are physically separated but connected with low-latency links. Deploying across AZs is the foundation of HA on AWS.

---

**Q107. What is Route 53 failover routing?**

> Route 53 failover routing monitors your primary endpoint with health checks. If the primary becomes unhealthy, Route 53 automatically routes DNS traffic to a secondary (failover) endpoint — enabling cross-region DR at the DNS level.

---

**Q108. What is Aurora Global Database?**

> Aurora Global Database spans multiple AWS regions with one primary read-write region and up to 5 secondary read-only regions. Replication lag is under 1 second. In a disaster, a secondary region can be promoted to primary in under 1 minute.

---

## 12. AWS Security & IAM

---

**Q109. What is AWS IAM?**

> IAM (Identity and Access Management) controls who can access what in AWS. It manages Users (people), Groups (collections of users), Roles (permissions for services/apps), and Policies (JSON documents defining permissions).

---

**Q110. What is the difference between IAM User, Group, and Role?**

> - **User** — Represents a person or application with long-term credentials (username/password or access keys)
> - **Group** — Collection of users sharing the same permissions. Manage permissions at group level.
> - **Role** — Temporary permissions assumed by AWS services, applications, or federated users. No long-term credentials.

---

**Q111. What is the principle of least privilege?**

> Grant only the minimum permissions needed to perform a task — nothing more. If a Lambda function only reads from S3, give it only `s3:GetObject` permission on that specific bucket, not full S3 access.

---

**Q112. What is an IAM Policy?**

> An IAM Policy is a JSON document that defines permissions. It specifies: Effect (Allow/Deny), Action (what API calls), Resource (what AWS resources), and optional Conditions.

```json
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

---

**Q113. What is AWS KMS?**

> KMS (Key Management Service) is a managed service for creating and controlling encryption keys. Used to encrypt S3 objects, EBS volumes, RDS databases, and Secrets Manager values. Integrates with CloudTrail for key usage auditing.

---

**Q114. What is AWS Secrets Manager?**

> Secrets Manager securely stores and automatically rotates credentials (DB passwords, API keys, tokens). Applications retrieve secrets at runtime via API — never hardcoding credentials in code or config files.

---

**Q115. What is the difference between KMS and Secrets Manager?**

> - **KMS** — Manages encryption keys used to encrypt/decrypt data
> - **Secrets Manager** — Stores and manages secrets (passwords, tokens) with auto-rotation. Uses KMS internally to encrypt the stored secrets.

---

**Q116. What is AWS CloudTrail?**

> CloudTrail records all API calls made in your AWS account — who did what, when, from where. It provides an audit trail for security analysis, compliance, and troubleshooting. Logs are stored in S3.

---

**Q117. What is AWS GuardDuty?**

> GuardDuty is a managed threat detection service that continuously monitors for malicious activity using ML and threat intelligence. It analyzes CloudTrail logs, VPC Flow Logs, and DNS logs to detect threats like compromised instances, unusual API calls, and crypto mining.

---

**Q118. What is MFA and why is it important in AWS?**

> MFA (Multi-Factor Authentication) requires a second form of verification (TOTP code from authenticator app) in addition to password. Essential for root account and IAM users with console access. Prevents account compromise even if passwords are stolen.

---

**Q119. What is the AWS Shared Responsibility Model?**

```
AWS Responsible For:               YOU Responsible For:
────────────────────               ────────────────────
Physical hardware                  Data encryption
Network infrastructure             IAM users and permissions
Hypervisor                         OS patching (EC2)
Managed service availability       Application security
                                   Network config (VPC, SG)
```

---

**Q120. What is AWS WAF?**

> WAF (Web Application Firewall) protects web applications from common exploits like SQL injection, cross-site scripting (XSS), and DDoS. It can be attached to ALB, CloudFront, or API Gateway and uses rules to block malicious requests.

---

## 13. AWS Cost Optimization

---

**Q121. What are the 5 pillars of AWS cost optimization?**

> Right Sizing (use correct instance size), Pricing Models (Reserved/Spot/Savings Plans), Auto Scaling (pay only for what you use), Storage Optimization (right storage class/tier), and Monitoring (track and alert on spend).

---

**Q122. What is the difference between Reserved Instances and Savings Plans?**

> - **Reserved Instances** — Commit to a specific instance type/region for 1 or 3 years. Save 40–60%.
> - **Savings Plans** — Commit to a $/hour spend. Flexible across instance types, sizes, regions, and services (EC2, Lambda, Fargate). Same savings but more flexibility.

---

**Q123. When should you use Spot Instances?**

> Use Spot Instances for fault-tolerant, flexible workloads that can handle interruption: batch processing, data analytics, CI/CD build agents, rendering, and dev/test environments. Save up to 90% vs On-Demand.

---

**Q124. What is AWS Compute Optimizer?**

> Compute Optimizer uses machine learning to analyze CloudWatch metrics and recommend right-sized instance types for EC2, EBS, Lambda, and ECS. It identifies over-provisioned resources to reduce cost without impacting performance.

---

**Q125. What is AWS Cost Explorer?**

> Cost Explorer is a tool to visualize, analyze, and forecast AWS spending. You can filter by service, region, account, and tag — identify cost trends, top spending services, and opportunities to switch to Reserved Instances.

---

**Q126. What is AWS Budgets?**

> AWS Budgets lets you set custom cost and usage alerts. You can create budgets for total spend, per-service spend, Reserved Instance utilization, or Savings Plans coverage — and receive email/SNS alerts when thresholds are breached.

---

**Q127. How do you reduce S3 costs?**

> Use lifecycle policies to move data to cheaper storage classes (Standard → Standard-IA → Glacier → Deep Archive), delete incomplete multipart uploads, clean old versions if versioning is enabled, and use S3 Intelligent-Tiering for unknown access patterns.

---

**Q128. What is NAT Gateway cost concern and how to fix it?**

> NAT Gateway charges per hour plus per GB of data processed. For high-volume AWS service traffic (S3, DynamoDB), use VPC Endpoints instead — data to these services goes through the endpoint (free) instead of NAT Gateway (charged).

---

## 14. Cloud Migration

---

**Q129. What are the 7 Rs of Cloud Migration?**

| Strategy | Description |
|----------|-------------|
| **Retire** | Decommission — no longer needed |
| **Retain** | Keep on-premises for now |
| **Rehost** | Lift and shift — move as-is to cloud (EC2) |
| **Replatform** | Lift, tinker, shift — minor optimizations (RDS instead of self-managed DB) |
| **Repurchase** | Move to SaaS (Salesforce, Office 365) |
| **Refactor/Re-architect** | Redesign for cloud-native (microservices, serverless) |
| **Relocate** | Move VMware to VMware Cloud on AWS |

---

**Q130. What is the difference between Rehost and Refactor?**

> - **Rehost (Lift & Shift)** — Move application to cloud with zero code changes. Fast, low risk, minimal cloud benefit. Example: Move VM to EC2.
> - **Refactor (Re-architect)** — Redesign app to leverage cloud-native features (Lambda, containers, managed services). Slower, higher effort, maximum cloud benefit.

---

**Q131. What is AWS Migration Hub?**

> AWS Migration Hub provides a central place to track migration progress across multiple AWS and partner migration tools. It gives visibility into the status of application migrations across the portfolio.

---

**Q132. What is AWS Database Migration Service (DMS)?**

> DMS migrates databases to AWS with minimal downtime. It supports homogeneous migrations (Oracle → Oracle on RDS) and heterogeneous migrations (Oracle → PostgreSQL). It continuously replicates changes during migration so the source stays live.

---

**Q133. What is the AWS Schema Conversion Tool (SCT)?**

> SCT automatically converts source database schema and most stored procedures to the target database format during heterogeneous migrations. Used with DMS for migrations like Oracle → Aurora PostgreSQL.

---

**Q134. What is AWS Application Migration Service (MGN)?**

> MGN (formerly CloudEndure) replicates on-premises servers to AWS in real-time. When ready to cut over, it launches target instances with minimal downtime. The primary tool for Rehost (lift-and-shift) migrations.

---

**Q135. What are the phases of a cloud migration?**

```
Phase 1 — ASSESS
  Discover all applications and infrastructure
  Analyze dependencies
  Build migration business case

Phase 2 — MOBILIZE
  Create migration plan
  Set up AWS landing zone
  Train teams
  Pilot migration of first app

Phase 3 — MIGRATE & MODERNIZE
  Execute migrations by wave
  Test each migrated app
  Optimize after migration
  Decommission old infrastructure
```

---

## 15. Linux Administration

---

**Q136. What is the difference between a process and a thread?**

> - **Process** — Independent program with its own memory space. Isolated from other processes.
> - **Thread** — Lightweight unit within a process sharing the same memory. Multiple threads run concurrently within one process.

---

**Q137. What are the most important Linux commands for a DevOps engineer?**

| Command | Purpose |
|---------|---------|
| `top / htop` | Real-time process monitoring |
| `df -h` | Disk space usage |
| `du -sh *` | Directory size |
| `ps aux` | List all running processes |
| `netstat -tulnp` | Open ports and connections |
| `ss -tulnp` | Modern netstat replacement |
| `grep / awk / sed` | Text processing |
| `tail -f` | Live log viewing |
| `chmod / chown` | File permissions |
| `systemctl` | Manage services |
| `journalctl` | View system logs |
| `curl / wget` | HTTP requests |

---

**Q138. What is the difference between chmod 755 and chmod 644?**

```
chmod 755:  rwxr-xr-x
  Owner: read + write + execute
  Group: read + execute
  Others: read + execute
  Use: Scripts, executables, directories

chmod 644:  rw-r--r--
  Owner: read + write
  Group: read only
  Others: read only
  Use: Config files, HTML files, logs
```

---

**Q139. What is systemctl and how do you use it?**

> systemctl is the command to manage systemd services in modern Linux distributions.

```bash
systemctl start nginx       # Start service
systemctl stop nginx        # Stop service
systemctl restart nginx     # Restart service
systemctl status nginx      # Check status
systemctl enable nginx      # Auto-start on boot
systemctl disable nginx     # Remove auto-start
systemctl list-units        # List all services
```

---

**Q140. What is the difference between soft link and hard link?**

> - **Hard Link** — Direct reference to inode. If original file is deleted, data still accessible via hard link.
> - **Soft Link (Symlink)** — Pointer to the file path. If original is deleted, symlink breaks.

```bash
ln file.txt hardlink.txt        # Hard link
ln -s file.txt softlink.txt     # Soft link
```

---

**Q141. How do you check and kill a process in Linux?**

```bash
# Find process
ps aux | grep nginx
pgrep nginx

# Kill process
kill PID              # Graceful (SIGTERM)
kill -9 PID          # Force kill (SIGKILL)
pkill nginx          # Kill by name

# Check what is using a port
lsof -i :80
ss -tulnp | grep :80
```

---

**Q142. What is crontab and how do you schedule a job?**

```bash
# Edit cron jobs
crontab -e

# Format: MIN HOUR DAY MONTH WEEKDAY COMMAND
# Examples:
0 2 * * *     /backup.sh          # Daily at 2:00 AM
*/5 * * * *   /check_health.sh    # Every 5 minutes
0 0 * * 0     /weekly_report.sh   # Every Sunday midnight
30 9 1 * *    /monthly_bill.sh    # 1st of month at 9:30 AM

# List cron jobs
crontab -l
```

---

**Q143. What is the difference between /etc/hosts and DNS?**

> - `/etc/hosts` — Local static file mapping hostnames to IPs. Checked before DNS. Fast but manual, not scalable.
> - **DNS** — Distributed network service resolving hostnames to IPs dynamically. Scalable, centrally managed, supports the entire internet.

---

**Q144. How do you troubleshoot a Linux server that is slow?**

```
Step 1: Check CPU
  top / htop → identify high CPU processes

Step 2: Check Memory
  free -h → check available RAM
  Look for high memory processes in top

Step 3: Check Disk
  df -h → check disk space
  iostat -x 1 → check disk I/O

Step 4: Check Network
  netstat -tulnp → check connections
  iftop → check bandwidth usage

Step 5: Check Logs
  tail -f /var/log/syslog
  journalctl -xe → recent errors
```

---

**Q145. What is SSH and how do you secure it?**

> SSH (Secure Shell) is a cryptographic protocol for secure remote access to Linux servers. To secure SSH:
> - Disable root login (`PermitRootLogin no`)
> - Use key-based auth only (`PasswordAuthentication no`)
> - Change default port from 22
> - Allow only specific IPs via Security Group
> - Use Fail2ban to block brute-force attempts
> - Enable MFA for SSH

---

## Quick Revision — One Line Answers

---

**Q146. What is the default region when none is specified in AWS CLI?**
> The region configured in `~/.aws/config` or the `AWS_DEFAULT_REGION` environment variable.

**Q147. What is an ARN?**
> ARN (Amazon Resource Name) is a unique identifier for every AWS resource. Format: `arn:aws:service:region:account-id:resource`

**Q148. What is the difference between horizontal and vertical scaling?**
> Horizontal = add more servers. Vertical = upgrade existing server. Horizontal is preferred for cloud-native apps.

**Q149. What is Infrastructure as Code (IaC)?**
> Managing and provisioning infrastructure through machine-readable code files instead of manual processes. Tools: Terraform, CloudFormation, Ansible.

**Q150. What is idempotency?**
> Running the same operation multiple times produces the same result. Critical in IaC and configuration management — applying a config twice should not break things.

---

## Last Minute Tips for Interview

```
✅ Always mention Security (SGs, IAM, encryption) when discussing any service
✅ Always mention Cost (right-size, Reserved, Spot) when asked about production
✅ Always mention HA (Multi-AZ, Auto Scaling) for production architecture questions
✅ Use real examples from your experience — "In my project, I used..."
✅ If unsure, explain what you DO know and how you would find the answer
✅ Draw architecture diagrams on paper if asked — shows system thinking
✅ For every AWS service — know: what it is, when to use, how to secure, how it costs
```

---

## Cheat Sheet — Full Forms

| Short | Full Form |
|-------|-----------|
| EC2 | Elastic Compute Cloud |
| S3 | Simple Storage Service |
| VPC | Virtual Private Cloud |
| RDS | Relational Database Service |
| EKS | Elastic Kubernetes Service |
| ELB | Elastic Load Balancer |
| ALB | Application Load Balancer |
| NLB | Network Load Balancer |
| IAM | Identity and Access Management |
| KMS | Key Management Service |
| ACM | AWS Certificate Manager |
| AMI | Amazon Machine Image |
| EBS | Elastic Block Store |
| IGW | Internet Gateway |
| NAT | Network Address Translation |
| NACL | Network Access Control List |
| SG | Security Group |
| ASG | Auto Scaling Group |
| DMS | Database Migration Service |
| MGN | Application Migration Service |
| WAF | Web Application Firewall |
| IaC | Infrastructure as Code |
| CI/CD | Continuous Integration / Continuous Delivery |
| HA | High Availability |
| DR | Disaster Recovery |
| RTO | Recovery Time Objective |
| RPO | Recovery Point Objective |
| PITR | Point-in-Time Recovery |

---

*Question Bank v1.0 | Prepared: March 2026 | All the best for your interview! 🚀*
