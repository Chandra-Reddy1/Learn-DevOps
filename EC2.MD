# AWS EC2 – Elastic Compute Cloud
## Complete Guide from Scratch

---

## Table of Contents

1. [What is AWS EC2?](#what-is-aws-ec2)
2. [Core Concepts & Terminology](#core-concepts--terminology)
3. [EC2 Instance Types](#ec2-instance-types)
4. [AMI – Amazon Machine Image](#ami--amazon-machine-image)
5. [EC2 Pricing Models](#ec2-pricing-models)
6. [Storage Options](#storage-options)
7. [Security Groups & Key Pairs](#security-groups--key-pairs)
8. [Networking – VPC, Subnets & Elastic IPs](#networking--vpc-subnets--elastic-ips)
9. [Launching an EC2 Instance (Step-by-Step)](#launching-an-ec2-instance-step-by-step)
10. [Connecting to EC2](#connecting-to-ec2)
11. [User Data & Bootstrap Scripts](#user-data--bootstrap-scripts)
12. [EC2 Auto Scaling](#ec2-auto-scaling)
13. [Load Balancing with EC2](#load-balancing-with-ec2)
14. [EC2 Monitoring – CloudWatch](#ec2-monitoring--cloudwatch)
15. [IAM Roles with EC2](#iam-roles-with-ec2)
16. [EC2 Placement Groups](#ec2-placement-groups)
17. [Hibernate, Stop & Terminate](#hibernate-stop--terminate)
18. [AWS CLI – EC2 Examples](#aws-cli--ec2-examples)
19. [Sample Outputs](#sample-outputs)
20. [Most Asked Interview Questions](#most-asked-interview-questions)

---

## What is AWS EC2?

**Amazon Elastic Compute Cloud (EC2)** is a web service that provides **resizable compute capacity** in the cloud. It is designed to make web-scale cloud computing easier for developers.

> Think of EC2 as a **virtual server** (or many servers) running in Amazon's data centers that you can rent by the hour/second.

### Why EC2?

| Feature | Description |
|---|---|
| **Elastic** | Scale up or down in minutes |
| **Flexible** | Choose OS, CPU, RAM, storage |
| **Reliable** | 99.99% SLA availability |
| **Secure** | VPC, Security Groups, KMS encryption |
| **Integrated** | Works with S3, RDS, IAM, CloudWatch, etc. |

---

## Core Concepts & Terminology

| Term | Meaning |
|---|---|
| **Instance** | A virtual machine running on EC2 |
| **AMI** | Amazon Machine Image – a template to launch instances |
| **Instance Type** | Defines CPU, memory, network, storage specs |
| **Key Pair** | SSH key used to connect to Linux/Windows instances |
| **Security Group** | Virtual firewall controlling inbound/outbound traffic |
| **EBS** | Elastic Block Store – persistent storage for instances |
| **Elastic IP** | Static public IP address |
| **Region** | Geographic area with multiple data centers |
| **Availability Zone (AZ)** | Isolated location within a region |
| **VPC** | Virtual Private Cloud – your own network in AWS |
| **Subnet** | A range of IP addresses within a VPC |
| **User Data** | Bootstrap script run at instance launch |
| **Metadata** | Instance info accessible at `169.254.169.254` |

---

## EC2 Instance Types

Instance types are organized into **families** based on workload:

```
Family Format:  [Family][Generation].[Size]
Example:        t3.micro, m5.large, c6i.xlarge
```

### Instance Families

| Family | Use Case | Examples |
|---|---|---|
| **T** (Burstable) | Dev, test, low-traffic web apps | t2.micro, t3.small |
| **M** (General Purpose) | Balanced CPU/Memory | m5.large, m6i.xlarge |
| **C** (Compute Optimized) | High CPU – ML, gaming, batch | c5.xlarge, c6i.2xlarge |
| **R** (Memory Optimized) | In-memory DBs, big data | r5.large, r6g.xlarge |
| **I** (Storage Optimized) | High IOPS – NoSQL DBs | i3.large, i4i.xlarge |
| **G** (GPU) | ML training, video rendering | g4dn.xlarge, p3.2xlarge |
| **P** (GPU – AI/ML) | Deep Learning | p4d.24xlarge |
| **X** (Memory Intensive) | SAP HANA, in-memory | x1e.xlarge |
| **Inf** (Inferentia) | ML inference | inf1.xlarge |

### Instance Size Reference (t3 family example)

| Size | vCPU | RAM (GiB) |
|---|---|---|
| t3.nano | 2 | 0.5 |
| t3.micro | 2 | 1 |
| t3.small | 2 | 2 |
| t3.medium | 2 | 4 |
| t3.large | 2 | 8 |
| t3.xlarge | 4 | 16 |
| t3.2xlarge | 8 | 32 |

---

## AMI – Amazon Machine Image

An **AMI** is a pre-configured template that contains:
- Operating System (Linux, Windows, macOS)
- Application server
- Pre-installed software

### AMI Types

| Type | Description |
|---|---|
| **AWS-provided** | Amazon Linux 2, Ubuntu, Windows Server |
| **AWS Marketplace** | Third-party (e.g., Bitnami LAMP, Palo Alto) |
| **Community AMIs** | Public AMIs from community |
| **Custom AMIs** | Your own created from a running instance |

### Creating a Custom AMI

```bash
# Create AMI from running instance
aws ec2 create-image \
  --instance-id i-0abcd1234efgh5678 \
  --name "MyCustomAMI-v1" \
  --description "Custom AMI with NGINX pre-installed" \
  --no-reboot

# Output:
# {
#   "ImageId": "ami-0123456789abcdef0"
# }
```

---

## EC2 Pricing Models

### 1. On-Demand
- Pay per second (Linux) or per hour (Windows)
- No commitment
- Best for: unpredictable workloads, development

### 2. Reserved Instances (RI)
- 1 or 3 year commitment
- Up to **72% discount** vs On-Demand
- Types: Standard RI, Convertible RI, Scheduled RI

### 3. Spot Instances
- Bid for unused EC2 capacity
- Up to **90% cheaper** than On-Demand
- Can be interrupted with 2-minute warning
- Best for: batch jobs, data analysis, fault-tolerant apps

### 4. Savings Plans
- Flexible pricing model
- Commit to consistent usage ($/hour) for 1 or 3 years
- Up to **66% savings**

### 5. Dedicated Hosts
- Physical server dedicated to you
- Compliance, licensing requirements (BYOL)

### 6. Dedicated Instances
- Instances run on hardware dedicated to you
- No sharing with other AWS accounts

### Pricing Comparison Table

| Model | Discount | Use Case |
|---|---|---|
| On-Demand | 0% (baseline) | Dev/test, short-term |
| Reserved (1yr) | ~40% | Steady-state production |
| Reserved (3yr) | ~60-72% | Long-term production |
| Spot | ~70-90% | Batch, fault-tolerant |
| Savings Plans | ~66% | Flexible workloads |
| Dedicated Host | Premium | Compliance, BYOL |

---

## Storage Options

### EBS – Elastic Block Store

Persistent block storage attached to EC2 instances.

| Volume Type | Use Case | Max IOPS | Max Throughput |
|---|---|---|---|
| **gp3** (General SSD) | Boot, dev/test | 16,000 | 1,000 MB/s |
| **gp2** (General SSD) | General workloads | 16,000 | 250 MB/s |
| **io1/io2** (Provisioned IOPS) | Databases | 64,000 | 1,000 MB/s |
| **st1** (Throughput HDD) | Big data, log processing | 500 | 500 MB/s |
| **sc1** (Cold HDD) | Infrequent access | 250 | 250 MB/s |

### Instance Store (Ephemeral Storage)

- **Temporary** – data lost when instance stops/terminates
- Very high I/O performance
- Physically attached to host machine

### EFS – Elastic File System

- Managed NFS file system
- Can be shared across multiple EC2 instances
- Automatically scales

### S3 (Object Storage)

- Not a direct mount (use AWS CLI or SDK)
- Infinite scalability
- Best for backups, static assets

---

## Security Groups & Key Pairs

### Security Groups

Security groups act as a **stateful virtual firewall**.

```
Rules:
- Inbound: Controls incoming traffic
- Outbound: Controls outgoing traffic (default: allow all)
```

#### Sample Security Group Rules

| Type | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| SSH | TCP | 22 | My IP | Admin access |
| HTTP | TCP | 80 | 0.0.0.0/0 | Web traffic |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Secure web |
| Custom TCP | TCP | 8080 | 10.0.0.0/16 | App port |
| MySQL | TCP | 3306 | SG-id | DB access |

#### Key Security Group Facts

- **Stateful**: If inbound is allowed, response is automatically allowed
- Multiple security groups can be assigned to one instance
- Security groups are **allow-only** (no deny rules)
- Changes take effect immediately

### Key Pairs

```bash
# Create a key pair
aws ec2 create-key-pair \
  --key-name MyKeyPair \
  --query 'KeyMaterial' \
  --output text > MyKeyPair.pem

# Set correct permissions
chmod 400 MyKeyPair.pem
```

---

## Networking – VPC, Subnets & Elastic IPs

### VPC Architecture

```
Region (us-east-1)
└── VPC (10.0.0.0/16)
    ├── Public Subnet (10.0.1.0/24)  ← has Internet Gateway route
    │   └── EC2 Instance (Public IP)
    ├── Private Subnet (10.0.2.0/24) ← no direct Internet access
    │   └── EC2 Instance (no Public IP)
    └── Internet Gateway
```

### Elastic IP (EIP)

- Static public IPv4 address
- Stays associated even after instance stop/start
- Free when associated to a running instance

```bash
# Allocate Elastic IP
aws ec2 allocate-address --domain vpc

# Associate to an instance
aws ec2 associate-address \
  --instance-id i-0abcd1234efgh5678 \
  --allocation-id eipalloc-12345678
```

---

## Launching an EC2 Instance (Step-by-Step)

### Using AWS Management Console

```
Step 1: Choose AMI
  → Amazon Linux 2023 AMI (free tier eligible)

Step 2: Choose Instance Type
  → t2.micro (free tier eligible)

Step 3: Configure Instance
  → Network: Default VPC
  → Subnet: Auto-assign Public IP: Enable

Step 4: Add Storage
  → 8 GiB gp3 (Root volume)

Step 5: Add Tags
  → Key: Name | Value: MyFirstEC2

Step 6: Configure Security Group
  → Add rule: SSH | TCP | 22 | My IP

Step 7: Review & Launch
  → Select or create Key Pair
  → Launch
```

### Using AWS CLI

```bash
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --count 1 \
  --instance-type t2.micro \
  --key-name MyKeyPair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0bb1c79de3EXAMPLE \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyFirstEC2}]'
```

---

## Connecting to EC2

### SSH (Linux/macOS)

```bash
# Connect to Amazon Linux / Ubuntu
ssh -i "MyKeyPair.pem" ec2-user@ec2-54-123-45-67.compute-1.amazonaws.com

# For Ubuntu AMI use 'ubuntu' instead of 'ec2-user'
ssh -i "MyKeyPair.pem" ubuntu@54.123.45.67
```

### EC2 Instance Connect (Browser-based)

```bash
# No key pair needed — uses temporary SSH certificates
# From AWS Console: EC2 → Instances → Connect → EC2 Instance Connect
```

### Session Manager (No SSH, No Open Ports)

```bash
# Install Session Manager plugin, then:
aws ssm start-session --target i-0abcd1234efgh5678
```

### Windows RDP

```
1. Get Windows Password:
   EC2 Console → Connect → Get Password → Upload Key Pair

2. Open RDP Client:
   Host: Public DNS or IP
   Username: Administrator
   Password: Decrypted password from above
```

---

## User Data & Bootstrap Scripts

User Data runs **once** at instance launch as root.

### Example: Install and start Apache on Amazon Linux

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from EC2 - $(hostname -f)</h1>" > /var/www/html/index.html
```

### Example: Install Docker

```bash
#!/bin/bash
yum update -y
yum install -y docker
systemctl start docker
systemctl enable docker
usermod -aG docker ec2-user
```

### Passing User Data via CLI

```bash
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name MyKeyPair \
  --user-data file://bootstrap.sh
```

### Viewing User Data from Inside Instance

```bash
curl http://169.254.169.254/latest/user-data
```

---

## EC2 Auto Scaling

Auto Scaling automatically adjusts the number of EC2 instances.

### Components

```
Launch Template
    ↓
Auto Scaling Group (ASG)
    ↓
Scaling Policies
    ↓
CloudWatch Alarms
```

### Create a Launch Template

```bash
aws ec2 create-launch-template \
  --launch-template-name MyLaunchTemplate \
  --version-description "v1" \
  --launch-template-data '{
    "ImageId": "ami-0c55b159cbfafe1f0",
    "InstanceType": "t2.micro",
    "KeyName": "MyKeyPair",
    "SecurityGroupIds": ["sg-0123456789abcdef0"]
  }'
```

### Create Auto Scaling Group

```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name MyASG \
  --launch-template LaunchTemplateName=MyLaunchTemplate,Version='$Latest' \
  --min-size 1 \
  --max-size 5 \
  --desired-capacity 2 \
  --vpc-zone-identifier "subnet-abc123,subnet-def456"
```

### Scaling Policy Types

| Policy | Description |
|---|---|
| **Target Tracking** | Maintain a target metric (e.g., CPU 50%) |
| **Step Scaling** | Scale based on CloudWatch alarm thresholds |
| **Simple Scaling** | Single scaling action per alarm |
| **Scheduled Scaling** | Scale at specific times |

---

## Load Balancing with EC2

### Types of Load Balancers

| Type | Layer | Use Case |
|---|---|---|
| **ALB** (Application LB) | Layer 7 | HTTP/HTTPS, path-based routing |
| **NLB** (Network LB) | Layer 4 | TCP/UDP, extreme performance |
| **GWLB** (Gateway LB) | Layer 3 | Security appliances |
| **CLB** (Classic LB) | Layer 4/7 | Legacy, not recommended |

### ALB Path-Based Routing Example

```
ALB Listener: Port 80
  ├── /api/*     → Target Group: API Servers
  ├── /images/*  → Target Group: Image Servers
  └── /*         → Target Group: Web Servers
```

---

## EC2 Monitoring – CloudWatch

### Default Metrics (every 5 minutes, free)

- CPUUtilization
- NetworkIn / NetworkOut
- DiskReadOps / DiskWriteOps
- StatusCheckFailed

### Detailed Monitoring (every 1 minute, paid)

```bash
# Enable detailed monitoring
aws ec2 monitor-instances --instance-ids i-0abcd1234efgh5678
```

### Creating a CloudWatch Alarm

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "HighCPU" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-0abcd1234efgh5678 \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:MyTopic
```

### Viewing Instance Metadata from Inside EC2

```bash
# IMDSv1
curl http://169.254.169.254/latest/meta-data/

# IMDSv2 (more secure, uses token)
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```

---

## IAM Roles with EC2

Attach an IAM Role to an EC2 instance to grant AWS service access **without hardcoding credentials**.

### Attach IAM Role to Instance

```bash
# Create an instance profile
aws iam create-instance-profile --instance-profile-name MyEC2Profile

# Add role to profile
aws iam add-role-to-instance-profile \
  --instance-profile-name MyEC2Profile \
  --role-name MyEC2Role

# Associate profile to running instance
aws ec2 associate-iam-instance-profile \
  --instance-id i-0abcd1234efgh5678 \
  --iam-instance-profile Name=MyEC2Profile
```

### Test from Inside EC2 (with S3 read role)

```bash
aws s3 ls s3://my-bucket/
# Works without any aws configure keys!
```

---

## EC2 Placement Groups

Control **where** instances are physically placed in AWS.

| Strategy | Description | Use Case |
|---|---|---|
| **Cluster** | Same rack, same AZ | Ultra-low latency (HPC, big data) |
| **Spread** | Different racks, up to 7/AZ | Critical apps, max fault isolation |
| **Partition** | Different partitions, multi-AZ | Hadoop, Kafka, Cassandra |

```bash
# Create a cluster placement group
aws ec2 create-placement-group \
  --group-name MyClusterPG \
  --strategy cluster
```

---

## Hibernate, Stop & Terminate

| Action | RAM | EBS Root | EBS Data | Public IP | Billing |
|---|---|---|---|---|---|
| **Stop** | Lost | Persists | Persists | Lost | No (EBS charged) |
| **Hibernate** | Saved to EBS | Persists | Persists | Lost | No (EBS charged) |
| **Terminate** | Lost | Deleted* | Persists* | Lost | No |
| **Reboot** | Preserved | Persists | Persists | Same | Yes |

*Depends on "Delete on Termination" setting

### Enable Hibernation (must be set at launch)

```bash
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type m5.large \
  --hibernation-options Configured=true \
  --block-device-mappings '[{
    "DeviceName": "/dev/xvda",
    "Ebs": {
      "Encrypted": true,
      "VolumeSize": 30
    }
  }]'
```

---

## AWS CLI – EC2 Examples

### Common EC2 CLI Commands

```bash
# List all running instances
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[*].Instances[*].[InstanceId,InstanceType,PublicIpAddress,Tags[?Key=='Name'].Value|[0]]" \
  --output table

# Start an instance
aws ec2 start-instances --instance-ids i-0abcd1234efgh5678

# Stop an instance
aws ec2 stop-instances --instance-ids i-0abcd1234efgh5678

# Terminate an instance
aws ec2 terminate-instances --instance-ids i-0abcd1234efgh5678

# Reboot an instance
aws ec2 reboot-instances --instance-ids i-0abcd1234efgh5678

# Describe instance status
aws ec2 describe-instance-status --instance-ids i-0abcd1234efgh5678

# List available AMIs (Amazon Linux)
aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" \
  --query "sort_by(Images, &CreationDate)[-1].ImageId"

# Describe security groups
aws ec2 describe-security-groups --group-ids sg-0123456789abcdef0

# Allocate Elastic IP
aws ec2 allocate-address --domain vpc

# List key pairs
aws ec2 describe-key-pairs

# Describe volumes
aws ec2 describe-volumes --filters "Name=attachment.instance-id,Values=i-0abcd1234efgh5678"

# Create snapshot of EBS volume
aws ec2 create-snapshot \
  --volume-id vol-049df61146c4d7901 \
  --description "Backup snapshot"
```

---

## Sample Outputs

### `aws ec2 describe-instances` (abbreviated)

```json
{
  "Reservations": [
    {
      "Instances": [
        {
          "InstanceId": "i-0abcd1234efgh5678",
          "InstanceType": "t2.micro",
          "State": {
            "Code": 16,
            "Name": "running"
          },
          "PublicIpAddress": "54.123.45.67",
          "PrivateIpAddress": "10.0.1.25",
          "PublicDnsName": "ec2-54-123-45-67.compute-1.amazonaws.com",
          "ImageId": "ami-0c55b159cbfafe1f0",
          "LaunchTime": "2024-03-10T08:30:00.000Z",
          "Placement": {
            "AvailabilityZone": "us-east-1a"
          },
          "Tags": [
            {
              "Key": "Name",
              "Value": "MyFirstEC2"
            }
          ],
          "SecurityGroups": [
            {
              "GroupId": "sg-0123456789abcdef0",
              "GroupName": "MySecurityGroup"
            }
          ]
        }
      ]
    }
  ]
}
```

### `aws ec2 describe-instances` (table output)

```
----------------------------------------------------------------------
|                        DescribeInstances                           |
+---------------------+----------+---------------+------------------+
| i-0abcd1234efgh5678 | t2.micro | 54.123.45.67  | MyFirstEC2       |
| i-0xyz9876mnop5432  | t3.small | 54.200.10.11  | WebServer-Prod   |
| i-0lmn1234qrst6789  | m5.large | 54.100.20.30  | DatabaseServer   |
+---------------------+----------+---------------+------------------+
```

### SSH Connection Output

```
$ ssh -i "MyKeyPair.pem" ec2-user@54.123.45.67

   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|       AL2023 End of Support: 2028-03-15
  ~~       \#/ ___
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'

[ec2-user@ip-10-0-1-25 ~]$
```

### Instance Metadata Output

```bash
$ curl http://169.254.169.254/latest/meta-data/
ami-id
ami-launch-index
ami-manifest-path
block-device-mapping/
events/
hostname
iam/
identity-credentials/
instance-action
instance-id
instance-life-cycle
instance-type
local-hostname
local-ipv4
mac
network/
placement/
profile
public-hostname
public-ipv4
public-keys/
reservation-id
security-groups
services/

$ curl http://169.254.169.254/latest/meta-data/instance-id
i-0abcd1234efgh5678

$ curl http://169.254.169.254/latest/meta-data/instance-type
t2.micro

$ curl http://169.254.169.254/latest/meta-data/public-ipv4
54.123.45.67
```

### CloudWatch Describe Alarms Output

```json
{
  "MetricAlarms": [
    {
      "AlarmName": "HighCPU",
      "AlarmDescription": null,
      "StateValue": "OK",
      "StateReason": "Threshold Crossed: 1 datapoint [12.5 (10/03/24 08:00:00)] was not greater than the threshold (80.0).",
      "MetricName": "CPUUtilization",
      "Namespace": "AWS/EC2",
      "Statistic": "Average",
      "Period": 300,
      "Threshold": 80.0,
      "ComparisonOperator": "GreaterThanThreshold"
    }
  ]
}
```

### Auto Scaling Group Description

```
$ aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names MyASG

AutoScalingGroupName: MyASG
MinSize: 1
MaxSize: 5
DesiredCapacity: 2
Instances:
  - InstanceId: i-0aaa111bbb222ccc3
    AvailabilityZone: us-east-1a
    LifecycleState: InService
    HealthStatus: Healthy
  - InstanceId: i-0ddd333eee444fff5
    AvailabilityZone: us-east-1b
    LifecycleState: InService
    HealthStatus: Healthy
```

---

## Most Asked Interview Questions

### Beginner Level

**Q1. What is EC2 and what are its key features?**
> EC2 is Amazon's virtual server service that provides resizable compute capacity in the cloud. Key features include: multiple instance types, various pricing models (On-Demand, Reserved, Spot), integration with other AWS services, elastic scaling, and support for multiple operating systems.

---

**Q2. What is an AMI? How is it different from a snapshot?**
> An **AMI** is a complete template for launching an EC2 instance (includes OS, software, configuration). A **Snapshot** is a point-in-time backup of an EBS volume. An AMI is created *from* snapshots, but a snapshot alone can't launch an instance — you'd need to create a volume from it first.

---

**Q3. What is the difference between stopping and terminating an EC2 instance?**
> - **Stop**: Instance shuts down; EBS root volume is retained; you can restart it. Billing stops (but EBS storage continues).
> - **Terminate**: Instance is permanently deleted; root EBS volume is deleted by default (unless "Delete on Termination" is disabled).

---

**Q4. What are Security Groups? Are they stateful or stateless?**
> Security Groups are virtual firewalls controlling inbound/outbound traffic. They are **stateful** — if you allow inbound traffic, the response is automatically allowed outbound (and vice versa). Compare with NACLs which are stateless.

---

**Q5. What is the difference between a Security Group and a Network ACL (NACL)?**

| Feature | Security Group | NACL |
|---|---|---|
| Level | Instance level | Subnet level |
| State | Stateful | Stateless |
| Rules | Allow only | Allow & Deny |
| Order | All rules evaluated | Rules in order (numbered) |
| Default | All outbound allowed | All traffic allowed |

---

**Q6. What EC2 instance types are free tier eligible?**
> **t2.micro** (or **t3.micro** in some regions) — 750 hours/month free for 12 months.

---

### Intermediate Level

**Q7. What is the difference between On-Demand, Reserved, and Spot instances?**
> - **On-Demand**: Pay per use, no commitment, full price.
> - **Reserved**: 1 or 3-year commitment, up to 72% cheaper, good for steady workloads.
> - **Spot**: Bid on unused capacity, up to 90% cheaper, but can be interrupted with 2-minute notice.

---

**Q8. Can you change the instance type of a running instance?**
> No. You must **stop** the instance first, then change the instance type (requires compatible instance family), then restart. You cannot change instance type while it's running.

---

**Q9. What is User Data in EC2? When does it run?**
> User Data is a script (bash or PowerShell) that runs **once automatically during the first boot** of an EC2 instance as the root/administrator user. It's used for bootstrapping — installing software, configuring settings, etc. It does NOT run on subsequent starts unless configured to do so.

---

**Q10. What is an Elastic IP? Why would you use one?**
> An Elastic IP is a **static public IPv4 address** that doesn't change when you stop/start an instance. By default, EC2 instances get a new public IP every time they're stopped and started. Use EIP when you need a consistent, fixed IP address for DNS records, whitelisting, or accessing the instance after restarts.

---

**Q11. What is EC2 Instance Metadata? How do you access it?**
> Instance metadata is information about your running EC2 instance accessible from within the instance at the link-local address:
> ```bash
> curl http://169.254.169.254/latest/meta-data/
> ```
> You can get instance ID, IP, IAM role credentials, public keys, etc.

---

**Q12. What are EC2 Placement Groups? Name the types.**
> Placement Groups control the physical placement of instances:
> - **Cluster**: All instances in same rack (low latency, high throughput)
> - **Spread**: Each instance on separate hardware (high availability)
> - **Partition**: Groups of instances in different partitions (distributed apps like Hadoop)

---

**Q13. What is the difference between EBS and Instance Store?**

| Feature | EBS | Instance Store |
|---|---|---|
| Persistence | Persistent | Ephemeral (lost on stop/terminate) |
| Performance | Good (depends on type) | Very high (NVMe SSD) |
| Snapshots | Yes | No |
| Flexibility | Can detach/attach | Cannot |

---

**Q14. How does EC2 Auto Scaling work?**
> Auto Scaling uses a **Launch Template** (or Launch Configuration) to define instance settings, an **Auto Scaling Group** to manage the fleet (min, max, desired), and **Scaling Policies** triggered by CloudWatch Alarms to add or remove instances automatically.

---

**Q15. What is the difference between Horizontal and Vertical scaling in EC2?**
> - **Vertical Scaling (Scale Up)**: Change instance type to a bigger one (e.g., t2.micro → m5.xlarge). Requires downtime.
> - **Horizontal Scaling (Scale Out)**: Add more instances. No downtime, more resilient. Achieved via Auto Scaling Groups.

---

### Advanced Level

**Q16. Explain EC2 Hibernate. How does it differ from Stop?**
> When you **hibernate** an EC2 instance:
> - RAM contents are saved to the EBS root volume
> - On restart, the OS resumes from where it left off (no boot process)
> - Faster startup, preserves in-memory application state
>
> Requirements: EBS root volume must be encrypted, sufficient EBS space, supported instance types only.

---

**Q17. What are the steps to troubleshoot an EC2 instance that is not reachable?**

```
1. Check Instance State (running? stopped?)
2. Check System Status Check & Instance Status Check
3. Verify Security Group — is port 22 (SSH) or 3389 (RDP) open?
4. Check NACL rules on the subnet
5. Verify route tables — internet gateway attached?
6. Check if correct key pair is being used
7. Check Elastic IP / Public IP assignment
8. Review system logs: Actions → Monitor & Troubleshoot → Get System Log
9. Verify IAM roles and permissions if using SSM
10. Check if instance has a public IP (needs it for direct access)
```

---

**Q18. What is the difference between IMDSv1 and IMDSv2?**
> - **IMDSv1**: Simple HTTP GET request to `169.254.169.254`. Vulnerable to SSRF attacks.
> - **IMDSv2**: Requires a session token obtained via a PUT request first. More secure. AWS recommends using IMDSv2 always.

```bash
# IMDSv2 Example
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```

---

**Q19. How do you ensure an EC2 instance is highly available?**

```
✅ Deploy across multiple Availability Zones (AZs)
✅ Use Auto Scaling Groups (min 2 instances)
✅ Place behind an Application Load Balancer
✅ Use Multi-AZ RDS for database
✅ Store state externally (S3, ElastiCache, RDS) — stateless EC2
✅ Enable detailed CloudWatch monitoring + alarms
✅ Use health checks on load balancer
```

---

**Q20. What happens when a Spot Instance is interrupted?**
> AWS sends a **2-minute interruption notice** via:
> - Instance metadata: `http://169.254.169.254/latest/meta-data/spot/interruption-notice`
> - CloudWatch Events
>
> You should handle this by: saving state, draining connections, using Spot interruption handlers in your application, or using **Spot Fleet** with fallback to On-Demand.

---

**Q21. What is an EC2 Launch Template vs Launch Configuration?**

| Feature | Launch Template | Launch Configuration |
|---|---|---|
| Versioning | Yes (multiple versions) | No |
| Mixed instances | Yes | No |
| T2/T3 Unlimited | Yes | No |
| Recommended | ✅ Yes | ❌ Legacy |

---

**Q22. How would you copy an EC2 instance to another region?**

```
Step 1: Create an AMI from the instance
  aws ec2 create-image --instance-id i-xxxx --name "MyAMI"

Step 2: Copy the AMI to target region
  aws ec2 copy-image \
    --source-image-id ami-0123456789abcdef0 \
    --source-region us-east-1 \
    --region ap-south-1 \
    --name "MyAMI-Copy"

Step 3: Launch a new instance in the target region using the copied AMI
```

---

## Quick Reference Cheat Sheet

```
Launch Instance:   aws ec2 run-instances --image-id ami-xxx --instance-type t2.micro
List Instances:    aws ec2 describe-instances
Start Instance:    aws ec2 start-instances --instance-ids i-xxx
Stop Instance:     aws ec2 stop-instances --instance-ids i-xxx
Terminate:         aws ec2 terminate-instances --instance-ids i-xxx
SSH Connect:       ssh -i key.pem ec2-user@<public-ip>
Instance Metadata: curl http://169.254.169.254/latest/meta-data/
Create AMI:        aws ec2 create-image --instance-id i-xxx --name "name"
Create Snapshot:   aws ec2 create-snapshot --volume-id vol-xxx
Elastic IP:        aws ec2 allocate-address --domain vpc
```

---

*Document generated: March 2026 | AWS EC2 Complete Reference Guide*
