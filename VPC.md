# AWS VPC — Complete Guide from Scratch

> **Amazon Virtual Private Cloud (VPC)** lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define.

---

## Table of Contents

1. [What is AWS VPC?](#1-what-is-aws-vpc)
2. [Core Components](#2-core-components)
3. [VPC Architecture Overview](#3-vpc-architecture-overview)
4. [Creating a VPC — Step by Step](#4-creating-a-vpc--step-by-step)
5. [Subnets](#5-subnets)
6. [Internet Gateway (IGW)](#6-internet-gateway-igw)
7. [Route Tables](#7-route-tables)
8. [Security Groups](#8-security-groups)
9. [Network ACLs (NACLs)](#9-network-acls-nacls)
10. [NAT Gateway & NAT Instance](#10-nat-gateway--nat-instance)
11. [VPC Peering](#11-vpc-peering)
12. [VPC Endpoints](#12-vpc-endpoints)
13. [Flow Logs](#13-vpc-flow-logs)
14. [Transit Gateway](#14-transit-gateway)
15. [AWS CLI Examples with Sample Output](#15-aws-cli-examples-with-sample-output)
16. [Terraform Example](#16-terraform-example)
17. [Most Asked Interview & Exam Questions](#17-most-asked-interview--exam-questions)

---

## 1. What is AWS VPC?

AWS VPC is a **virtual network** dedicated to your AWS account. It is logically isolated from other virtual networks in the AWS Cloud. You have complete control over:

- IP address range (CIDR block)
- Subnets (public and private)
- Route tables
- Network gateways
- Security settings (Security Groups + NACLs)

**Key Facts:**
- Each AWS account gets a **default VPC** in every region (CIDR: `172.31.0.0/16`)
- You can create up to **5 VPCs per region** (soft limit, can be increased)
- VPCs span **all Availability Zones** in a region
- Subnets reside in **a single AZ**

---

## 2. Core Components

| Component | Description |
|-----------|-------------|
| **VPC** | The isolated virtual network itself |
| **Subnet** | A range of IP addresses within the VPC |
| **Internet Gateway (IGW)** | Allows communication between VPC and the internet |
| **Route Table** | Rules that determine where network traffic is directed |
| **Security Group** | Stateful firewall at the instance/ENI level |
| **Network ACL (NACL)** | Stateless firewall at the subnet level |
| **NAT Gateway** | Allows private subnet instances to reach the internet |
| **VPC Endpoint** | Private connection to AWS services without internet |
| **VPC Peering** | Connect two VPCs privately |
| **Transit Gateway** | Hub-and-spoke network for multiple VPCs |
| **Elastic IP (EIP)** | Static public IPv4 address |
| **ENI** | Elastic Network Interface — virtual network card |

---

## 3. VPC Architecture Overview

```
Region: us-east-1
┌──────────────────────────────────────────────────────────────────┐
│                          VPC (10.0.0.0/16)                       │
│                                                                  │
│  ┌───────────── AZ: us-east-1a ──────────────┐                  │
│  │                                           │                  │
│  │  Public Subnet (10.0.1.0/24)              │                  │
│  │  ┌─────────────────────────────────┐      │                  │
│  │  │  EC2 (Web Server)  [EIP]        │      │                  │
│  │  │  NAT Gateway       [EIP]        │      │                  │
│  │  └─────────────────────────────────┘      │                  │
│  │                                           │                  │
│  │  Private Subnet (10.0.2.0/24)             │                  │
│  │  ┌─────────────────────────────────┐      │                  │
│  │  │  EC2 (App Server)               │      │                  │
│  │  │  RDS (Database)                 │      │                  │
│  │  └─────────────────────────────────┘      │                  │
│  └───────────────────────────────────────────┘                  │
│                          │                                       │
│                    Internet Gateway                              │
└──────────────────────────│───────────────────────────────────────┘
                           │
                        Internet
```

---

## 4. Creating a VPC — Step by Step

### Via AWS Console

1. Go to **VPC Dashboard** → **Your VPCs** → **Create VPC**
2. Choose **VPC only** or **VPC and more** (wizard)
3. Enter:
   - Name: `my-custom-vpc`
   - IPv4 CIDR block: `10.0.0.0/16`
   - IPv6: Optional
   - Tenancy: `Default`
4. Click **Create VPC**

### CIDR Block Guidelines

| CIDR | IP Range | Total IPs |
|------|----------|-----------|
| `10.0.0.0/16` | 10.0.0.1 – 10.0.255.254 | 65,536 |
| `10.0.0.0/24` | 10.0.0.1 – 10.0.0.254 | 256 |
| `192.168.0.0/16` | 192.168.0.0 – 192.168.255.255 | 65,536 |

> ⚠️ AWS **reserves 5 IP addresses** in every subnet:
> - `.0` — Network address
> - `.1` — VPC router
> - `.2` — DNS server
> - `.3` — Reserved for future use
> - `.255` — Broadcast address

---

## 5. Subnets

Subnets are subdivisions of your VPC CIDR block tied to a **single AZ**.

### Public vs Private Subnet

| Feature | Public Subnet | Private Subnet |
|---------|--------------|----------------|
| Route to IGW | ✅ Yes | ❌ No |
| Direct internet access | ✅ Yes (with public IP) | ❌ No |
| Use case | Web servers, load balancers | Databases, app servers |
| Internet outbound via | IGW | NAT Gateway |

### Creating a Subnet (Console)

1. VPC Dashboard → **Subnets** → **Create subnet**
2. Select your VPC
3. Name: `public-subnet-1a`
4. AZ: `us-east-1a`
5. IPv4 CIDR: `10.0.1.0/24`
6. Repeat for private subnet with `10.0.2.0/24`

---

## 6. Internet Gateway (IGW)

An IGW enables internet communication. It is:
- **Horizontally scaled**, redundant, and highly available
- **Free** (you pay for data transfer, not the gateway itself)
- Only **one IGW per VPC**

### Setup Steps

1. VPC Dashboard → **Internet Gateways** → **Create internet gateway**
2. Name: `my-vpc-igw`
3. **Attach to VPC** → select your VPC

> Without attaching the IGW to a route table entry, instances won't reach the internet even if the IGW exists.

---

## 7. Route Tables

Route tables contain rules (routes) that determine where traffic is directed.

### Default Route Table Behavior

| Destination | Target | Meaning |
|-------------|--------|---------|
| `10.0.0.0/16` | local | All VPC-internal traffic |
| `0.0.0.0/0` | igw-xxxxxxxx | All internet traffic → IGW |

### Public Subnet Route Table (example)

```
Destination      Target
-----------      ------
10.0.0.0/16      local
0.0.0.0/0        igw-0a1b2c3d4e5f
```

### Private Subnet Route Table (example)

```
Destination      Target
-----------      ------
10.0.0.0/16      local
0.0.0.0/0        nat-0a1b2c3d4e5f   ← Routes through NAT Gateway
```

> 💡 Each subnet must be **explicitly associated** with a route table. If not, it uses the **main (default)** route table of the VPC.

---

## 8. Security Groups

Security Groups act as **virtual firewalls** at the **instance/ENI level**. They are **stateful** — if you allow inbound traffic, the response is automatically allowed outbound.

### Key Characteristics

- **Stateful** (return traffic automatically allowed)
- Only **ALLOW** rules (no explicit deny)
- Evaluated **all rules** before deciding
- Applied at **instance level**
- Can reference other Security Groups as sources

### Example: Web Server Security Group

**Inbound Rules:**

| Type | Protocol | Port | Source | Purpose |
|------|----------|------|--------|---------|
| HTTP | TCP | 80 | 0.0.0.0/0 | Public web traffic |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Secure web traffic |
| SSH | TCP | 22 | 10.0.0.0/16 | Internal SSH only |

**Outbound Rules:**

| Type | Protocol | Port | Destination |
|------|----------|------|-------------|
| All traffic | All | All | 0.0.0.0/0 |

### Example: Database Security Group

**Inbound Rules:**

| Type | Protocol | Port | Source | Purpose |
|------|----------|------|--------|---------|
| MySQL/Aurora | TCP | 3306 | sg-webserver | Only from web SG |

---

## 9. Network ACLs (NACLs)

NACLs operate at the **subnet level** and are **stateless** — both inbound AND outbound rules must be configured explicitly.

### Security Group vs NACL Comparison

| Feature | Security Group | NACL |
|---------|---------------|------|
| Level | Instance/ENI | Subnet |
| Stateful/Stateless | Stateful ✅ | Stateless ❌ |
| Rules | Allow only | Allow + Deny |
| Rule evaluation | All rules | Lowest number first |
| Default | Deny all inbound | Allow all |

### Example NACL Rules

**Inbound:**

| Rule # | Type | Protocol | Port | Source | Action |
|--------|------|----------|------|--------|--------|
| 100 | HTTP | TCP | 80 | 0.0.0.0/0 | ALLOW |
| 110 | HTTPS | TCP | 443 | 0.0.0.0/0 | ALLOW |
| 120 | Custom TCP | TCP | 1024-65535 | 0.0.0.0/0 | ALLOW |
| * | All traffic | All | All | 0.0.0.0/0 | DENY |

**Outbound:**

| Rule # | Type | Protocol | Port | Dest | Action |
|--------|------|----------|------|------|--------|
| 100 | HTTP | TCP | 80 | 0.0.0.0/0 | ALLOW |
| 110 | HTTPS | TCP | 443 | 0.0.0.0/0 | ALLOW |
| 120 | Custom TCP | TCP | 1024-65535 | 0.0.0.0/0 | ALLOW |
| * | All traffic | All | All | 0.0.0.0/0 | DENY |

> ⚠️ **Ephemeral ports** (1024–65535) must be allowed for return traffic since NACLs are stateless!

---

## 10. NAT Gateway & NAT Instance

### NAT Gateway

Allows **private subnet** instances to initiate outbound internet traffic (software updates, patches) without being directly reachable from the internet.

**Key Properties:**
- Managed service (AWS handles availability)
- Must be placed in a **public subnet**
- Requires an **Elastic IP**
- Supports up to **45 Gbps** bandwidth
- **Not free** — billed per hour + per GB processed

**Setup Flow:**
```
Private EC2 → Private Route Table → NAT Gateway (public subnet) → IGW → Internet
```

### NAT Gateway vs NAT Instance

| Feature | NAT Gateway | NAT Instance |
|---------|-------------|--------------|
| Managed | ✅ AWS managed | ❌ Self-managed EC2 |
| Availability | Highly available | Manual failover needed |
| Bandwidth | Up to 45 Gbps | Depends on instance type |
| Cost | Higher | Lower (but more work) |
| Security Groups | Cannot attach | Can attach |
| Bastion host | ❌ No | ✅ Yes |

---

## 11. VPC Peering

VPC Peering creates a **direct network route** between two VPCs using private IPs. Traffic stays on the AWS backbone — it never traverses the public internet.

### Rules & Limitations

- **No transitive peering** — if A↔B and B↔C, then A cannot reach C via B
- **No overlapping CIDR blocks** allowed
- Can peer across **accounts** and **regions** (inter-region peering)
- Both sides must **accept** the peering request
- Route tables on **both sides** must be updated

### Peering Setup

1. Requester VPC → **Peering Connections** → **Create Peering Connection**
2. Select requester VPC and accepter VPC
3. Accepter goes to Peering Connections → **Accept Request**
4. Update route tables on **both VPCs**:

**VPC-A Route Table:**
```
Destination        Target
10.0.0.0/16        local
10.1.0.0/16        pcx-0123456789abcdef0   ← Peer VPC CIDR
```

**VPC-B Route Table:**
```
Destination        Target
10.1.0.0/16        local
10.0.0.0/16        pcx-0123456789abcdef0   ← Peer VPC CIDR
```

---

## 12. VPC Endpoints

VPC Endpoints allow **private connectivity** to AWS services (S3, DynamoDB, etc.) **without requiring IGW, NAT, or VPN**.

### Types

| Type | Services | How it Works |
|------|----------|--------------|
| **Gateway Endpoint** | S3, DynamoDB | Added as route table entry |
| **Interface Endpoint** | Most AWS services | Creates ENI with private IP in subnet |

### Gateway Endpoint for S3

1. VPC Dashboard → **Endpoints** → **Create Endpoint**
2. Service category: AWS services
3. Search: `com.amazonaws.us-east-1.s3`
4. Select **Gateway** type
5. Select VPC and route table(s)
6. Policy: Full access or custom

**Route table after endpoint:**
```
Destination              Target
10.0.0.0/16              local
pl-68a54001 (S3)         vpce-0a1b2c3d4e5f6g7h
```

---

## 13. VPC Flow Logs

Flow Logs **capture information about IP traffic** going to and from network interfaces in your VPC. Used for security analysis, troubleshooting, and compliance.

### Log Destinations

- **CloudWatch Logs**
- **S3 Bucket**
- **Kinesis Data Firehose**

### Flow Log Record Format

```
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
```

### Sample Flow Log Output

```
2 123456789012 eni-0a1b2c3d 10.0.1.23 10.0.2.45 12345 443 6 20 4000 1620000000 1620000060 ACCEPT OK
2 123456789012 eni-0a1b2c3d 203.0.113.99 10.0.1.23 45678 22 6 5 300 1620000010 1620000020 REJECT OK
```

**Field explanation:**
- `ACCEPT` = traffic was allowed
- `REJECT` = traffic was blocked by SG or NACL
- Protocol `6` = TCP, `17` = UDP, `1` = ICMP

---

## 14. Transit Gateway

Transit Gateway is a **hub-and-spoke** network transit hub that connects VPCs and on-premises networks.

### Without Transit Gateway (N VPCs = N*(N-1)/2 peering connections)
```
VPC-A ↔ VPC-B
VPC-A ↔ VPC-C
VPC-B ↔ VPC-C   (3 peering connections for 3 VPCs)
```

### With Transit Gateway
```
VPC-A ──┐
VPC-B ──┤── Transit Gateway ── On-premises (via VPN/Direct Connect)
VPC-C ──┘
```

**Benefits:**
- Simplifies complex peering architectures
- Supports **multicast**
- Supports **inter-region** peering
- Centralized routing policy

---

## 15. AWS CLI Examples with Sample Output

### Create a VPC

```bash
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=my-vpc}]'
```

**Sample Output:**
```json
{
    "Vpc": {
        "CidrBlock": "10.0.0.0/16",
        "DhcpOptionsId": "dopt-0a1b2c3d4e5f6g7h",
        "State": "pending",
        "VpcId": "vpc-0a1b2c3d4e5f6a7b8",
        "OwnerId": "123456789012",
        "InstanceTenancy": "default",
        "Ipv6CidrBlockAssociationSet": [],
        "CidrBlockAssociationSet": [
            {
                "AssociationId": "vpc-cidr-assoc-0abc123",
                "CidrBlock": "10.0.0.0/16",
                "CidrBlockState": { "State": "associated" }
            }
        ],
        "IsDefault": false,
        "Tags": [{ "Key": "Name", "Value": "my-vpc" }]
    }
}
```

---

### Create a Subnet

```bash
aws ec2 create-subnet \
  --vpc-id vpc-0a1b2c3d4e5f6a7b8 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=public-subnet-1a}]'
```

**Sample Output:**
```json
{
    "Subnet": {
        "AvailabilityZone": "us-east-1a",
        "AvailableIpAddressCount": 251,
        "CidrBlock": "10.0.1.0/24",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-0abc123def456gh78",
        "VpcId": "vpc-0a1b2c3d4e5f6a7b8",
        "Tags": [{ "Key": "Name", "Value": "public-subnet-1a" }]
    }
}
```

---

### Create and Attach Internet Gateway

```bash
# Create IGW
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=my-igw}]'
```

**Sample Output:**
```json
{
    "InternetGateway": {
        "Attachments": [],
        "InternetGatewayId": "igw-0a1b2c3d4e5f6g7h",
        "OwnerId": "123456789012",
        "Tags": [{ "Key": "Name", "Value": "my-igw" }]
    }
}
```

```bash
# Attach to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-0a1b2c3d4e5f6g7h \
  --vpc-id vpc-0a1b2c3d4e5f6a7b8
```

---

### Create a Route Table and Add Route

```bash
# Create route table
aws ec2 create-route-table \
  --vpc-id vpc-0a1b2c3d4e5f6a7b8 \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=public-rt}]'
```

**Sample Output:**
```json
{
    "RouteTable": {
        "Associations": [],
        "PropagatingVgws": [],
        "RouteTableId": "rtb-0abc123def456gh78",
        "Routes": [
            {
                "DestinationCidrBlock": "10.0.0.0/16",
                "GatewayId": "local",
                "Origin": "CreateRouteTable",
                "State": "active"
            }
        ],
        "VpcId": "vpc-0a1b2c3d4e5f6a7b8"
    }
}
```

```bash
# Add internet route
aws ec2 create-route \
  --route-table-id rtb-0abc123def456gh78 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-0a1b2c3d4e5f6g7h
```

**Sample Output:**
```json
{
    "Return": true
}
```

---

### Create a Security Group

```bash
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web Server Security Group" \
  --vpc-id vpc-0a1b2c3d4e5f6a7b8
```

**Sample Output:**
```json
{
    "GroupId": "sg-0abc123def456gh78"
}
```

```bash
# Add inbound HTTP rule
aws ec2 authorize-security-group-ingress \
  --group-id sg-0abc123def456gh78 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

---

### Describe VPCs

```bash
aws ec2 describe-vpcs --filters "Name=isDefault,Values=false"
```

**Sample Output:**
```json
{
    "Vpcs": [
        {
            "CidrBlock": "10.0.0.0/16",
            "DhcpOptionsId": "dopt-0a1b2c3d",
            "State": "available",
            "VpcId": "vpc-0a1b2c3d4e5f6a7b8",
            "OwnerId": "123456789012",
            "InstanceTenancy": "default",
            "IsDefault": false,
            "Tags": [{ "Key": "Name", "Value": "my-vpc" }]
        }
    ]
}
```

---

### Create NAT Gateway

```bash
# First allocate an Elastic IP
aws ec2 allocate-address --domain vpc
```

**Sample Output:**
```json
{
    "PublicIp": "54.210.10.220",
    "AllocationId": "eipalloc-0abc123def456gh78",
    "PublicIpv4Pool": "amazon",
    "NetworkBorderGroup": "us-east-1",
    "Domain": "vpc"
}
```

```bash
# Create NAT Gateway in public subnet
aws ec2 create-nat-gateway \
  --subnet-id subnet-0abc123def456gh78 \
  --allocation-id eipalloc-0abc123def456gh78
```

**Sample Output:**
```json
{
    "NatGateway": {
        "NatGatewayId": "nat-0abc123def456gh789",
        "State": "pending",
        "SubnetId": "subnet-0abc123def456gh78",
        "VpcId": "vpc-0a1b2c3d4e5f6a7b8",
        "NatGatewayAddresses": [
            {
                "AllocationId": "eipalloc-0abc123def456gh78",
                "PublicIp": "54.210.10.220"
            }
        ]
    }
}
```

---

## 16. Terraform Example

A complete Terraform configuration for a production-grade VPC with public and private subnets:

```hcl
# main.tf — AWS VPC with Public and Private Subnets

provider "aws" {
  region = "us-east-1"
}

# ─── VPC ─────────────────────────────────────────────
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "main-vpc"
    Environment = "production"
  }
}

# ─── Internet Gateway ────────────────────────────────
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  tags   = { Name = "main-igw" }
}

# ─── Public Subnets ──────────────────────────────────
resource "aws_subnet" "public" {
  count                   = 2
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 1}.0/24"
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-${count.index + 1}"
    Type = "public"
  }
}

# ─── Private Subnets ─────────────────────────────────
resource "aws_subnet" "private" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 10}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "private-subnet-${count.index + 1}"
    Type = "private"
  }
}

# ─── Elastic IP for NAT ──────────────────────────────
resource "aws_eip" "nat" {
  domain = "vpc"
  tags   = { Name = "nat-eip" }
}

# ─── NAT Gateway ─────────────────────────────────────
resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id
  tags          = { Name = "main-nat-gw" }
  depends_on    = [aws_internet_gateway.main]
}

# ─── Public Route Table ───────────────────────────────
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = { Name = "public-rt" }
}

resource "aws_route_table_association" "public" {
  count          = 2
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# ─── Private Route Table ─────────────────────────────
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }

  tags = { Name = "private-rt" }
}

resource "aws_route_table_association" "private" {
  count          = 2
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}

# ─── Data Sources ─────────────────────────────────────
data "aws_availability_zones" "available" {
  state = "available"
}

# ─── Outputs ─────────────────────────────────────────
output "vpc_id"              { value = aws_vpc.main.id }
output "public_subnet_ids"   { value = aws_subnet.public[*].id }
output "private_subnet_ids"  { value = aws_subnet.private[*].id }
```

---

## 17. Most Asked Interview & Exam Questions

### Q1: What is the difference between Security Groups and NACLs?

| | Security Group | NACL |
|--|---------------|------|
| Level | Instance | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow & Deny |
| Order | All evaluated | Evaluated in number order |
| Default | Deny all inbound | Allow all |

---

### Q2: Can you peer two VPCs with overlapping CIDR blocks?

**No.** VPC Peering requires **non-overlapping CIDR blocks**. If CIDRs overlap, routing becomes ambiguous and AWS will not allow the peering connection.

---

### Q3: What is transitive peering, and is it supported?

Transitive peering means routing through a middle VPC (A → B → C). **AWS does NOT support transitive VPC peering.** Each VPC must be directly peered with the other. Use **Transit Gateway** for hub-and-spoke architectures.

---

### Q4: How many IP addresses does AWS reserve in a subnet?

AWS reserves **5 IP addresses** per subnet:
- First IP (network address)
- Second IP (VPC router)
- Third IP (DNS)
- Fourth IP (future use)
- Last IP (broadcast)

So a `/24` subnet gives you **251 usable IPs**, not 256.

---

### Q5: What is the difference between a NAT Gateway and a NAT Instance?

| | NAT Gateway | NAT Instance |
|--|-------------|--------------|
| Managed | Yes (AWS) | No (you manage) |
| High Availability | Built-in per AZ | Manual with scripts |
| Bandwidth | Up to 45 Gbps | Depends on EC2 type |
| Security Groups | Cannot apply | Can apply |
| Bastion | No | Can be used as one |
| Cost | More expensive | Cheaper |

---

### Q6: What is an Egress-Only Internet Gateway?

An Egress-Only Internet Gateway allows **IPv6** traffic from your VPC to the internet, while **preventing the internet from initiating connections** to your instances. It is the IPv6 equivalent of a NAT Gateway.

---

### Q7: How does VPC Flow Logs help with security?

- Captures **accepted and rejected** traffic
- Useful for identifying unauthorized access attempts
- Helps in **compliance auditing** (PCI DSS, HIPAA)
- Can detect **port scanning** or unusual traffic patterns
- Log data sent to CloudWatch Logs or S3

---

### Q8: What is the difference between a Gateway Endpoint and an Interface Endpoint?

| | Gateway Endpoint | Interface Endpoint |
|--|----------------|--------------------|
| Services | S3, DynamoDB | 100+ AWS services |
| Implementation | Route table entry | ENI in subnet |
| Cost | **Free** | Hourly + data charges |
| DNS needed | No | Yes (private DNS) |
| Works across AZs | Yes | Per-AZ |

---

### Q9: Can you change the CIDR block of a VPC after creation?

You **cannot modify** the primary CIDR block. However, you can **add secondary CIDR blocks** (up to 5 total). The original CIDR cannot be removed or resized.

---

### Q10: What happens to traffic if a NACL has no matching rule?

NACLs have a default **DENY ALL** rule (rule number `*`) at the bottom. If no earlier rule matches the traffic, it is **denied**. This is why you must explicitly allow ephemeral/return ports.

---

### Q11: What is VPC Sharing (AWS RAM)?

Using **AWS Resource Access Manager (RAM)**, you can share subnets from a central (owner) VPC with other AWS accounts in the same AWS Organization. Participant accounts can launch resources into the shared subnets but **cannot modify** the VPC or subnet configurations.

---

### Q12: What is the difference between "Auto-assign public IP" and Elastic IP?

| | Auto-Assign Public IP | Elastic IP |
|--|----------------------|------------|
| Persistent | ❌ Changes on stop/start | ✅ Static |
| Cost | Free | Free if attached, charged if idle |
| Reassignment | No | Yes (can move between instances) |
| Use case | Dev/test instances | Production servers, DNS records |

---

### Q13: How do you connect an on-premises network to a VPC?

Two main options:
1. **AWS Site-to-Site VPN** — IPSec VPN over public internet; quick setup, lower cost
2. **AWS Direct Connect** — Dedicated private fiber connection; higher cost, lower latency, more bandwidth

For both, you attach a **Virtual Private Gateway (VGW)** to your VPC and update route tables to route on-premises CIDRs through the VGW.

---

### Q14: What is the default VPC and should you use it in production?

The **default VPC** (172.31.0.0/16) is created automatically in each region. It has:
- A default subnet in each AZ
- An attached IGW
- A route table routing all traffic to the IGW
- Public IPs assigned by default

**Should you use it in production? Generally NO** — use a custom VPC with:
- Proper public/private subnet separation
- Restricted routing
- Controlled security groups and NACLs

---

### Q15: What is a Bastion Host?

A **Bastion Host** (also called a Jump Server) is an EC2 instance in a **public subnet** that provides secure SSH/RDP access to instances in **private subnets**.

```
Internet → Bastion (public subnet) → Private EC2 (private subnet)
```

Best practices:
- Use a small, hardened instance
- Restrict SSH to your IP only
- Use **AWS Systems Manager Session Manager** as a modern alternative (no bastion needed)

---

## Summary Cheat Sheet

```
VPC                → Isolated virtual network (region-wide)
Subnet             → IP range in one AZ (public or private)
IGW                → Internet access for public subnets
NAT Gateway        → Outbound internet for private subnets
Route Table        → Traffic routing rules
Security Group     → Stateful firewall (instance level)
NACL               → Stateless firewall (subnet level)
VPC Peering        → Connect two VPCs (no transitive routing)
Transit Gateway    → Hub for many VPCs (transitive OK)
VPC Endpoint       → Private access to AWS services
Flow Logs          → Network traffic capture for auditing
Bastion Host       → Secure jump server to private resources
```

---

*Last updated: March 2026 | AWS CLI v2 | Terraform v1.7+*
