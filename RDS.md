# AWS RDS — Relational Database Service
## Complete Guide from Scratch

> **Amazon RDS (Relational Database Service)** is a managed database service that makes it easy to set up, operate, and scale a relational database in the cloud. AWS handles routine database tasks such as provisioning, patching, backup, recovery, failure detection, and repair.

---

## Table of Contents

1. [What is AWS RDS?](#1-what-is-aws-rds)
2. [Supported Database Engines](#2-supported-database-engines)
3. [Core Components & Concepts](#3-core-components--concepts)
4. [RDS Architecture Overview](#4-rds-architecture-overview)
5. [RDS Instance Classes](#5-rds-instance-classes)
6. [Storage Types](#6-storage-types)
7. [Multi-AZ Deployment](#7-multi-az-deployment)
8. [Read Replicas](#8-read-replicas)
9. [RDS Backups & Snapshots](#9-rds-backups--snapshots)
10. [RDS Security](#10-rds-security)
11. [RDS Parameter Groups & Option Groups](#11-rds-parameter-groups--option-groups)
12. [RDS Monitoring & Performance Insights](#12-rds-monitoring--performance-insights)
13. [RDS Proxy](#13-rds-proxy)
14. [Amazon Aurora (RDS-Compatible)](#14-amazon-aurora-rds-compatible)
15. [AWS CLI Examples with Sample Output](#15-aws-cli-examples-with-sample-output)
16. [Python Code Examples (boto3 + psycopg2)](#16-python-code-examples-boto3--psycopg2)
17. [Terraform Example](#17-terraform-example)
18. [RDS Pricing Model](#18-rds-pricing-model)
19. [Most Asked Interview & Exam Questions](#19-most-asked-interview--exam-questions)
20. [Summary Cheat Sheet](#20-summary-cheat-sheet)

---

## 1. What is AWS RDS?

AWS RDS is a **fully managed relational database service** in the cloud. Instead of installing, patching, and managing a database on an EC2 instance yourself, RDS automates all of that — letting you focus on your application.

### What RDS Manages FOR You

```
┌────────────────────────────────────────────────────────────┐
│              AWS RDS — Managed Responsibilities            │
├────────────────────────────────────────────────────────────┤
│  ✅ OS installation & patching                             │
│  ✅ Database software installation & upgrades              │
│  ✅ Automated backups (daily snapshots + transaction logs)  │
│  ✅ High availability (Multi-AZ failover)                  │
│  ✅ Hardware provisioning & replacement                     │
│  ✅ Storage auto-scaling                                   │
│  ✅ Monitoring & alerting (CloudWatch integration)         │
│  ✅ Encryption at rest & in transit                        │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│              YOU Manage                                    │
├────────────────────────────────────────────────────────────┤
│  🔧 Schema design & queries                                │
│  🔧 Database users & permissions                           │
│  🔧 Application connection strings                         │
│  🔧 Index optimization                                     │
│  🔧 Security Group rules                                   │
└────────────────────────────────────────────────────────────┘
```

### RDS vs Self-Managed DB on EC2

| Feature | RDS (Managed) | EC2 + DB (Self-Managed) |
|---------|--------------|-------------------------|
| OS patching | ✅ AWS | ❌ You |
| DB patching | ✅ AWS | ❌ You |
| Automated backups | ✅ Built-in | ❌ Manual setup |
| Multi-AZ failover | ✅ 1-click | ❌ Complex setup |
| Read replicas | ✅ 1-click | ❌ Manual replication |
| SSH access to OS | ❌ Not available | ✅ Full access |
| Custom OS config | ❌ Not available | ✅ Full control |
| Cost | Higher | Lower (but more work) |
| Use case | Most apps | Special DB configs |

---

## 2. Supported Database Engines

| Engine | Versions | Best For |
|--------|----------|----------|
| **MySQL** | 5.7, 8.0 | General purpose, web apps |
| **PostgreSQL** | 13, 14, 15, 16 | Complex queries, GIS, JSONB |
| **MariaDB** | 10.6, 10.11 | MySQL alternative, open-source |
| **Oracle** | 19c, 21c | Enterprise legacy apps |
| **Microsoft SQL Server** | 2017, 2019, 2022 | .NET apps, Windows ecosystem |
| **Amazon Aurora (MySQL)** | MySQL 5.7/8.0 compat | High-throughput MySQL workloads |
| **Amazon Aurora (PostgreSQL)** | PG 13/14/15/16 compat | High-throughput PG workloads |

> 💡 **Aurora** is AWS's own cloud-optimized engine — up to **5× faster than MySQL** and **3× faster than PostgreSQL** on standard hardware.

---

## 3. Core Components & Concepts

### DB Instance
The basic building block of RDS. A **DB Instance** is an isolated database environment running in the cloud. Each instance runs one DB engine.

### DB Instance Identifier
A unique name for your RDS instance within a region.
```
Example: my-postgres-db
Endpoint: my-postgres-db.cxyz123abc.us-east-1.rds.amazonaws.com
```

### DB Subnet Group
A collection of subnets (typically private) across **multiple AZs** that RDS uses to place DB instances. Always use **at least 2 subnets in different AZs**.

```
DB Subnet Group: my-db-subnet-group
  ├── subnet-0abc123  (10.0.2.0/24) — us-east-1a
  └── subnet-0def456  (10.0.3.0/24) — us-east-1b
```

### DB Parameter Group
A container for DB engine configuration values.
```
Example parameters (PostgreSQL):
  max_connections       = 200
  shared_buffers        = 256MB
  log_min_duration_statement = 1000  (log queries > 1s)
```

### DB Option Group
Additional features for specific engines (mainly MySQL and Oracle).
```
Example options:
  MARIADB_AUDIT_PLUGIN  — audit logging
  MEMCACHED             — in-memory caching layer
```

### Endpoint & Port

| Engine | Default Port |
|--------|-------------|
| MySQL / Aurora MySQL | 3306 |
| PostgreSQL / Aurora PG | 5432 |
| MariaDB | 3306 |
| Oracle | 1521 |
| SQL Server | 1433 |

---

## 4. RDS Architecture Overview

### Single-AZ (Development)

```
┌─────────────────────────────────────────────────────────────┐
│                  VPC (10.0.0.0/16)                          │
│                                                             │
│  ┌── Public Subnet ──────────────────────────────────────┐  │
│  │   EC2 App Server                                      │  │
│  │   (connects to RDS via endpoint)                      │  │
│  └───────────────────────────┬───────────────────────────┘  │
│                              │ Port 5432                    │
│  ┌── Private Subnet ─────────▼───────────────────────────┐  │
│  │                                                        │  │
│  │   ┌─────────────────────────────────────────────────┐  │  │
│  │   │           RDS Instance (Primary)                │  │  │
│  │   │           PostgreSQL 15                         │  │  │
│  │   │           Storage: 100 GB gp3                   │  │  │
│  │   └─────────────────────────────────────────────────┘  │  │
│  │                     AZ: us-east-1a                     │  │
│  └────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Multi-AZ (Production)

```
┌─────────────────────────────────────────────────────────────────────┐
│                      VPC (10.0.0.0/16)                              │
│                                                                     │
│  ┌── Public Subnet ────────────────────────────────────────────┐    │
│  │   EC2 App Server → connects to RDS ENDPOINT (DNS-based)    │    │
│  └────────────────────────────┬────────────────────────────────┘    │
│                               │                                     │
│          ┌────────────────────┴────────────────────┐               │
│          │                                         │               │
│  ┌── Private Subnet (1a) ──┐      ┌── Private Subnet (1b) ──┐      │
│  │                         │      │                         │      │
│  │  ┌─────────────────┐    │      │  ┌─────────────────┐    │      │
│  │  │  RDS PRIMARY    │    │      │  │  RDS STANDBY    │    │      │
│  │  │  (Read + Write) │◄───┼──────┼──│  (Sync Replica) │    │      │
│  │  │  Active         │    │Sync  │  │  Passive        │    │      │
│  │  └─────────────────┘    │Replic│  └─────────────────┘    │      │
│  │   AZ: us-east-1a        │ation │   AZ: us-east-1b        │      │
│  └─────────────────────────┘      └─────────────────────────┘      │
│                                                                     │
│  ⚡ On failure: DNS endpoint automatically flips to Standby (~60s)  │
└─────────────────────────────────────────────────────────────────────┘
```

### Multi-AZ + Read Replicas (Full Production)

```
                    App Server (Writes)
                          │
                          ▼
              ┌─── RDS PRIMARY ───┐
              │   (Read + Write)   │
              │   us-east-1a       │
              └──┬─────────────┬──┘
         Sync    │             │  Async Replication
      Replication│             │
                 ▼             ▼
        ┌── STANDBY ──┐   ┌── READ REPLICA 1 ──┐
        │ us-east-1b  │   │   us-east-1c        │
        │ (failover)  │   │   (read-only)       │
        └─────────────┘   └─────────────────────┘
                                    │
                          App Server (Reads)
                          Reporting / Analytics
```

---

## 5. RDS Instance Classes

### Instance Family Overview

| Family | Use Case | Examples |
|--------|----------|---------|
| **db.t3 / db.t4g** | Dev, test, small workloads | db.t3.micro, db.t4g.small |
| **db.m5 / db.m6g** | General purpose production | db.m5.large, db.m6g.xlarge |
| **db.r5 / db.r6g** | Memory-optimized (large DBs) | db.r5.2xlarge, db.r6g.4xlarge |
| **db.x1e / db.x2g** | Ultra-high memory | db.x2g.16xlarge |

### Common Instance Sizes

| Instance | vCPU | RAM | Network | Best For |
|----------|------|-----|---------|----------|
| db.t3.micro | 2 | 1 GB | Low | Free tier, dev |
| db.t3.small | 2 | 2 GB | Low | Small apps |
| db.t3.medium | 2 | 4 GB | Medium | Medium apps |
| db.m5.large | 2 | 8 GB | Up to 10 Gbps | Production |
| db.m5.xlarge | 4 | 16 GB | Up to 10 Gbps | Busy DBs |
| db.r5.2xlarge | 8 | 64 GB | Up to 10 Gbps | Memory-heavy |
| db.r5.4xlarge | 16 | 128 GB | Up to 10 Gbps | Large DBs |

> 💡 **Free Tier:** `db.t3.micro` — 750 hours/month for 12 months (MySQL, PostgreSQL, MariaDB)

---

## 6. Storage Types

| Storage Type | IOPS | Use Case | Notes |
|--------------|------|----------|-------|
| **gp2** (General Purpose SSD) | 3 IOPS/GB, burst 3000 | General workloads | Legacy; being replaced by gp3 |
| **gp3** (General Purpose SSD v2) | 3000 baseline, up to 16,000 | Most workloads | **Recommended** — cheaper than gp2 for high IOPS |
| **io1 / io2** (Provisioned IOPS) | Up to 64,000 IOPS | I/O-intensive (OLTP) | Most expensive, most consistent |
| **magnetic** (standard) | Low | Legacy only | Not recommended |

### Storage Auto-Scaling

```
Initial: 20 GB
  ↓ (when 10% free space left)
Auto-scales to: 25 GB → 35 GB → ... up to max threshold you set
Max threshold: 1000 GB (example)
```

Enable with:
```bash
aws rds modify-db-instance \
  --db-instance-identifier my-db \
  --max-allocated-storage 500 \
  --apply-immediately
```

---

## 7. Multi-AZ Deployment

Multi-AZ provides **high availability** and **automatic failover** for production databases.

### How It Works

```
Normal Operation:
  App → PRIMARY (AZ-1) ← writes/reads
                │
                │ Synchronous replication
                ▼
           STANDBY (AZ-2) ← receives all changes in real-time

Failure Scenario:
  AZ-1 goes down → AWS detects failure (~30 seconds)
  DNS endpoint flips to STANDBY (now becomes PRIMARY)
  App reconnects using SAME endpoint → zero config change
  Total downtime: ~60–120 seconds
```

### Key Facts

- Standby is **NOT readable** (only for failover)
- Replication is **synchronous** (zero data loss)
- Automatic failover triggered by: AZ failure, DB failure, OS patching, DB instance type change
- Same DNS endpoint — your app connection string **never changes**
- Backups are taken from **standby** (no I/O impact on primary)

### Enable Multi-AZ

```bash
aws rds modify-db-instance \
  --db-instance-identifier my-postgres-db \
  --multi-az \
  --apply-immediately
```

---

## 8. Read Replicas

Read Replicas are **read-only copies** of your primary database used to offload read traffic and improve performance.

### How It Works

```
Primary DB (Read + Write)
      │
      │  Asynchronous replication
      ├──────────────────────────► Read Replica 1 (same region)
      ├──────────────────────────► Read Replica 2 (same region)
      └──────────────────────────► Read Replica 3 (cross-region, DR)
```

### Read Replica vs Multi-AZ

| Feature | Read Replica | Multi-AZ Standby |
|---------|-------------|-----------------|
| Purpose | Scale reads | High availability |
| Replication | **Asynchronous** | **Synchronous** |
| Readable | ✅ Yes | ❌ No |
| Automatic failover | ❌ No (manual promote) | ✅ Yes |
| Separate endpoint | ✅ Yes | ❌ Same endpoint |
| Cross-region | ✅ Yes | ❌ No |
| Max replicas | 5 (MySQL/PG) | 1 standby |

### Use Cases for Read Replicas

- Offload **reporting** or **analytics** queries from primary
- Handle **read-heavy** traffic spikes
- **Cross-region** disaster recovery
- Can be **promoted to primary** in disaster scenarios

### Create a Read Replica

```bash
aws rds create-db-instance-read-replica \
  --db-instance-identifier my-read-replica \
  --source-db-instance-identifier my-postgres-db \
  --db-instance-class db.t3.medium \
  --availability-zone us-east-1b
```

---

## 9. RDS Backups & Snapshots

### Automated Backups

- Enabled by default (retention: 1–35 days)
- Taken daily during **backup window** you define
- Includes **transaction logs** → enables Point-in-Time Recovery (PITR)
- Stored in **S3** (free up to DB size)
- Deleted when DB instance is deleted (unless you keep final snapshot)

```
Backup Window Example:
  Daily snapshot: 03:00–04:00 UTC
  Transaction logs: every 5 minutes
  PITR: restore to ANY second in last 7 days
```

### Manual Snapshots

- User-initiated, persist **indefinitely** (not auto-deleted)
- Can be **shared** across AWS accounts
- Can be **copied** across regions for disaster recovery
- Basis for restoring to a new DB instance

### Backup Flow Diagram

```
DAY 1           DAY 2           DAY 3           NOW
  │               │               │               │
  ▼               ▼               ▼               ▼
[Snapshot]──►[TxLogs]──►[Snapshot]──►[TxLogs]──►[Snapshot]──►[TxLogs]

PITR: "Restore to March 10, 2026 at 14:37:52" ← Any point!
```

### Restore a DB from Snapshot

```bash
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier my-restored-db \
  --db-snapshot-identifier my-db-snapshot-2026-03-10 \
  --db-instance-class db.t3.medium \
  --availability-zone us-east-1a
```

> ⚠️ Restoring creates a **NEW DB instance** — it does not overwrite the existing one.

---

## 10. RDS Security

### Layers of Security

```
Layer 1 — Network Isolation
  └── Place RDS in PRIVATE subnet (no public route)
  └── DB Subnet Group spans multiple AZs

Layer 2 — Security Groups (Stateful Firewall)
  └── Allow ONLY app server's Security Group on DB port
  └── Example: Allow sg-webapp on port 5432

Layer 3 — IAM Authentication
  └── Use IAM roles instead of static DB passwords
  └── Generates temporary auth tokens (15-min expiry)

Layer 4 — Encryption
  └── At Rest: AWS KMS (AES-256) — enable at creation
  └── In Transit: SSL/TLS enforced via parameter group
  └── Encrypted snapshots & read replicas

Layer 5 — Secrets Management
  └── Store DB password in AWS Secrets Manager
  └── Auto-rotation of credentials every N days

Layer 6 — Audit Logging
  └── Enable audit logs → CloudWatch Logs
  └── Track who connected, what queries ran
```

### Security Group Setup

```
App Server Security Group (sg-webapp)
  └── Can initiate connections to sg-database on port 5432

Database Security Group (sg-database)
  └── Inbound: Port 5432, Source = sg-webapp  ✅
  └── Inbound: Port 5432, Source = 0.0.0.0/0  ❌ NEVER DO THIS
```

### Enable SSL for PostgreSQL

```sql
-- Verify SSL is in use
SELECT ssl, version()
FROM pg_stat_ssl, version()
WHERE pid = pg_backend_pid();

-- Force SSL in parameter group
rds.force_ssl = 1
```

### IAM Database Authentication

```python
import boto3
import psycopg2

# Generate auth token
client = boto3.client('rds', region_name='us-east-1')
token = client.generate_db_auth_token(
    DBHostname='my-db.cxyz123abc.us-east-1.rds.amazonaws.com',
    Port=5432,
    DBUsername='iam_user'
)

# Connect using token as password
conn = psycopg2.connect(
    host='my-db.cxyz123abc.us-east-1.rds.amazonaws.com',
    port=5432,
    database='myappdb',
    user='iam_user',
    password=token,
    sslmode='require'
)
```

---

## 11. RDS Parameter Groups & Option Groups

### Parameter Group

Controls **DB engine configuration** (like a config file for the DB).

```
Default parameter group: default.postgres15
Custom parameter group:  my-pg15-params

Key PostgreSQL parameters:
┌─────────────────────────────────────┬──────────────┬──────────────────────┐
│ Parameter                           │ Value        │ Effect               │
├─────────────────────────────────────┼──────────────┼──────────────────────┤
│ max_connections                     │ 200          │ Max concurrent users │
│ shared_buffers                      │ 256MB        │ Memory for caching   │
│ work_mem                            │ 4MB          │ Per-query sort mem   │
│ log_min_duration_statement          │ 1000         │ Log slow queries>1s  │
│ rds.force_ssl                       │ 1            │ Require SSL          │
│ log_connections                     │ 1            │ Log all connections  │
└─────────────────────────────────────┴──────────────┴──────────────────────┘
```

### Static vs Dynamic Parameters

| Type | Requires Reboot? | Example |
|------|-----------------|---------|
| **Static** | ✅ Yes | `max_connections` |
| **Dynamic** | ❌ No | `log_min_duration_statement` |

---

## 12. RDS Monitoring & Performance Insights

### CloudWatch Metrics (Key Ones)

| Metric | Description | Alert Threshold |
|--------|-------------|----------------|
| `CPUUtilization` | DB CPU usage % | > 80% sustained |
| `FreeStorageSpace` | Free disk in bytes | < 10% of total |
| `DatabaseConnections` | Active connections | Close to max_connections |
| `ReadLatency` | Avg read I/O time | > 20ms |
| `WriteLatency` | Avg write I/O time | > 20ms |
| `ReadIOPS` | Read operations/sec | Near provisioned limit |
| `WriteIOPS` | Write operations/sec | Near provisioned limit |
| `FreeableMemory` | Available RAM | < 256 MB |
| `ReplicaLag` | Replica behind primary (ms) | > 1000ms |

### Enhanced Monitoring

Provides **OS-level metrics** at 1–60 second granularity:
- CPU steal, load average
- Memory: active, inactive, free
- Disk: read/write bytes, I/O utilization
- Network: rx/tx bytes

### Performance Insights

Visualizes **database load** — shows which SQL queries are consuming most resources.

```
Performance Insights Dashboard:

Database Load (Average Active Sessions):
─────────────────────────────────────────────────
Max vCPU line: ████████████████████  (2 vCPUs)
─────────────────────────────────────────────────
14:00  █████████░░░░░░░░░░░░  1.2 AAS
14:05  ████████████░░░░░░░░░  1.6 AAS
14:10  ███████████████████░░  2.4 AAS  ← bottleneck!
─────────────────────────────────────────────────
Top SQL:
  SELECT * FROM orders WHERE...    (45% of load)
  UPDATE inventory SET...          (30% of load)
  SELECT COUNT(*) FROM users...    (15% of load)
```

---

## 13. RDS Proxy

RDS Proxy sits **between your application and RDS**, pooling and sharing database connections.

### Why Use RDS Proxy?

```
WITHOUT RDS Proxy:
  1000 Lambda functions → 1000 DB connections → DB overwhelmed ❌

WITH RDS Proxy:
  1000 Lambda functions → RDS Proxy (pool: 50 connections) → DB healthy ✅
```

### Key Benefits

| Benefit | Detail |
|---------|--------|
| **Connection pooling** | Reduces DB connections by up to 99% |
| **Faster failover** | Reduces failover time by up to 66% |
| **IAM Auth** | Enforces IAM-based DB authentication |
| **Serverless friendly** | Perfect for Lambda + RDS |
| **Secrets Manager** | Manages credentials automatically |

### Supported Engines

- MySQL 5.6, 5.7, 8.0
- PostgreSQL 10.11+, 11.5+, 12+, 13+, 14+
- MariaDB 10.3+
- Aurora MySQL & PostgreSQL

---

## 14. Amazon Aurora (RDS-Compatible)

Aurora is AWS's **cloud-native relational database** — compatible with MySQL and PostgreSQL but built from scratch for the cloud.

### Aurora Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    Aurora Cluster                            │
│                                                              │
│   Writer Instance (Primary)    Reader Instance ×1–15         │
│   ┌─────────────────────┐      ┌────────────────────┐        │
│   │  Read + Write       │      │  Read Only         │        │
│   └────────┬────────────┘      └──────────┬─────────┘        │
│            │                              │                  │
│            └──────────────┬───────────────┘                  │
│                           │                                  │
│   ┌───────────────────────▼────────────────────────────────┐  │
│   │              Shared Cluster Storage Volume             │  │
│   │              (6 copies across 3 AZs — automatic)      │  │
│   │              Auto-grows 10 GB increments               │  │
│   │              Max: 128 TB                               │  │
│   └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

### Aurora vs Standard RDS

| Feature | Standard RDS | Aurora |
|---------|-------------|--------|
| Storage | Per-instance | **Shared cluster volume** |
| Replicas | Up to 5 | Up to **15** |
| Failover time | ~60–120 sec | **~30 sec** |
| Storage replication | Multi-AZ only | **6 copies across 3 AZs always** |
| Performance | Baseline | **5× MySQL, 3× PostgreSQL** |
| Storage cost | Pay per GB | Pay per GB used (auto-shrinks) |
| Serverless option | ❌ | ✅ Aurora Serverless v2 |
| Global DB | ❌ | ✅ Aurora Global Database |

### Aurora Serverless v2

```
Traditional Aurora: Fixed instance size
  → You pay for db.r5.large even at 2am with 0 users

Aurora Serverless v2: Auto-scaling compute
  → Scales from 0.5 ACU to 128 ACU in seconds
  → Pay only for what you use
  → Perfect for: dev environments, variable workloads
```

---

## 15. AWS CLI Examples with Sample Output

### Create a PostgreSQL RDS Instance

```bash
aws rds create-db-instance \
  --db-instance-identifier my-postgres-db \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version 15.4 \
  --master-username dbadmin \
  --master-user-password MySecurePass123! \
  --allocated-storage 20 \
  --storage-type gp3 \
  --db-subnet-group-name my-db-subnet-group \
  --vpc-security-group-ids sg-0abc123def456gh78 \
  --backup-retention-period 7 \
  --multi-az \
  --storage-encrypted \
  --no-publicly-accessible \
  --tags Key=Environment,Value=production Key=Project,Value=myapp
```

**Sample Output:**
```json
{
    "DBInstance": {
        "DBInstanceIdentifier": "my-postgres-db",
        "DBInstanceClass": "db.t3.micro",
        "Engine": "postgres",
        "DBInstanceStatus": "creating",
        "MasterUsername": "dbadmin",
        "Endpoint": {
            "Address": "my-postgres-db.cxyz123abc.us-east-1.rds.amazonaws.com",
            "Port": 5432,
            "HostedZoneId": "Z2R2ITUGPM61AM"
        },
        "AllocatedStorage": 20,
        "InstanceCreateTime": "2026-03-10T10:00:00.000Z",
        "PreferredBackupWindow": "03:00-04:00",
        "BackupRetentionPeriod": 7,
        "DBSecurityGroups": [],
        "VpcSecurityGroups": [
            {
                "VpcSecurityGroupId": "sg-0abc123def456gh78",
                "Status": "active"
            }
        ],
        "DBParameterGroups": [
            {
                "DBParameterGroupName": "default.postgres15",
                "ParameterApplyStatus": "in-sync"
            }
        ],
        "AvailabilityZone": "us-east-1a",
        "MultiAZ": true,
        "EngineVersion": "15.4",
        "AutoMinorVersionUpgrade": true,
        "StorageType": "gp3",
        "StorageEncrypted": true,
        "PubliclyAccessible": false,
        "DeletionProtection": false
    }
}
```

---

### Describe DB Instances

```bash
aws rds describe-db-instances \
  --db-instance-identifier my-postgres-db \
  --query 'DBInstances[0].{ID:DBInstanceIdentifier,Status:DBInstanceStatus,Endpoint:Endpoint.Address,Engine:Engine,Class:DBInstanceClass}'
```

**Sample Output:**
```json
{
    "ID": "my-postgres-db",
    "Status": "available",
    "Endpoint": "my-postgres-db.cxyz123abc.us-east-1.rds.amazonaws.com",
    "Engine": "postgres",
    "Class": "db.t3.micro"
}
```

---

### Create Manual Snapshot

```bash
aws rds create-db-snapshot \
  --db-instance-identifier my-postgres-db \
  --db-snapshot-identifier my-postgres-db-snap-20260310
```

**Sample Output:**
```json
{
    "DBSnapshot": {
        "DBSnapshotIdentifier": "my-postgres-db-snap-20260310",
        "DBInstanceIdentifier": "my-postgres-db",
        "SnapshotCreateTime": "2026-03-10T14:30:00.000Z",
        "Engine": "postgres",
        "AllocatedStorage": 20,
        "Status": "creating",
        "Port": 5432,
        "AvailabilityZone": "us-east-1a",
        "VpcId": "vpc-0a1b2c3d4e5f6a7b8",
        "InstanceCreateTime": "2026-03-10T10:00:00.000Z",
        "MasterUsername": "dbadmin",
        "EngineVersion": "15.4",
        "LicenseModel": "postgresql-license",
        "SnapshotType": "manual",
        "PercentProgress": 0,
        "StorageType": "gp3",
        "Encrypted": true,
        "DBSnapshotArn": "arn:aws:rds:us-east-1:123456789012:snapshot:my-postgres-db-snap-20260310"
    }
}
```

---

### Create a Read Replica

```bash
aws rds create-db-instance-read-replica \
  --db-instance-identifier my-postgres-db-replica \
  --source-db-instance-identifier my-postgres-db \
  --db-instance-class db.t3.micro \
  --availability-zone us-east-1b \
  --no-publicly-accessible
```

**Sample Output:**
```json
{
    "DBInstance": {
        "DBInstanceIdentifier": "my-postgres-db-replica",
        "DBInstanceClass": "db.t3.micro",
        "Engine": "postgres",
        "DBInstanceStatus": "creating",
        "ReadReplicaSourceDBInstanceIdentifier": "my-postgres-db",
        "Endpoint": {
            "Address": "my-postgres-db-replica.cxyz123abc.us-east-1.rds.amazonaws.com",
            "Port": 5432
        },
        "MultiAZ": false,
        "StorageType": "gp3",
        "StorageEncrypted": true,
        "PubliclyAccessible": false
    }
}
```

---

### Modify an Existing DB Instance

```bash
aws rds modify-db-instance \
  --db-instance-identifier my-postgres-db \
  --db-instance-class db.t3.small \
  --backup-retention-period 14 \
  --preferred-backup-window "02:00-03:00" \
  --apply-immediately
```

**Sample Output:**
```json
{
    "DBInstance": {
        "DBInstanceIdentifier": "my-postgres-db",
        "DBInstanceClass": "db.t3.small",
        "DBInstanceStatus": "modifying",
        "PendingModifiedValues": {
            "DBInstanceClass": "db.t3.small"
        },
        "BackupRetentionPeriod": 14,
        "PreferredBackupWindow": "02:00-03:00"
    }
}
```

---

### List Snapshots

```bash
aws rds describe-db-snapshots \
  --db-instance-identifier my-postgres-db \
  --query 'DBSnapshots[*].{Snapshot:DBSnapshotIdentifier,Status:Status,Time:SnapshotCreateTime,Type:SnapshotType}' \
  --output table
```

**Sample Output:**
```
-----------------------------------------------------------------------
|                      DescribeDBSnapshots                            |
+------------------------------------+---------+----------+-----------+
|           Snapshot                 | Status  |   Time   |   Type    |
+------------------------------------+---------+----------+-----------+
|  rds:my-postgres-db-2026-03-10...  | available | 2026-03-10T03:00Z | automated |
|  my-postgres-db-snap-20260310      | available | 2026-03-10T14:30Z | manual    |
+------------------------------------+---------+----------+-----------+
```

---

### Delete DB Instance (with final snapshot)

```bash
aws rds delete-db-instance \
  --db-instance-identifier my-postgres-db \
  --final-db-snapshot-identifier my-postgres-db-final-snap \
  --delete-automated-backups false
```

**Sample Output:**
```json
{
    "DBInstance": {
        "DBInstanceIdentifier": "my-postgres-db",
        "DBInstanceStatus": "deleting",
        "DBInstanceClass": "db.t3.micro",
        "Engine": "postgres",
        "FinalDBSnapshotIdentifier": "my-postgres-db-final-snap"
    }
}
```

---

## 16. Python Code Examples (boto3 + psycopg2)

### Connect to RDS PostgreSQL

```python
import psycopg2
import os

def get_db_connection():
    """Connect to RDS PostgreSQL instance."""
    conn = psycopg2.connect(
        host=os.environ['DB_HOST'],       # RDS endpoint
        port=5432,
        database=os.environ['DB_NAME'],
        user=os.environ['DB_USER'],
        password=os.environ['DB_PASS'],
        sslmode='require',                # Always use SSL
        connect_timeout=10
    )
    return conn

# Usage
try:
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("SELECT version();")
    version = cursor.fetchone()
    print(f"PostgreSQL version: {version[0]}")
    cursor.close()
    conn.close()
except psycopg2.Error as e:
    print(f"Database error: {e}")
```

**Sample Output:**
```
PostgreSQL version: PostgreSQL 15.4 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 7.3.1 20180712, 64-bit
```

---

### Create Table and Insert Data

```python
import psycopg2

conn = get_db_connection()
cursor = conn.cursor()

# Create table
cursor.execute("""
    CREATE TABLE IF NOT EXISTS products (
        id          SERIAL PRIMARY KEY,
        name        VARCHAR(100) NOT NULL,
        price       DECIMAL(10,2),
        stock       INTEGER DEFAULT 0,
        created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );
""")

# Insert rows
products = [
    ('Laptop Pro 15', 1299.99, 50),
    ('Wireless Mouse', 29.99, 200),
    ('Mechanical Keyboard', 89.99, 150),
    ('USB-C Hub', 49.99, 300),
]

cursor.executemany(
    "INSERT INTO products (name, price, stock) VALUES (%s, %s, %s)",
    products
)

conn.commit()
print(f"Inserted {cursor.rowcount} rows successfully.")
cursor.close()
conn.close()
```

**Sample Output:**
```
Inserted 4 rows successfully.
```

---

### Query with Parameterized Input

```python
def get_products_by_price(max_price: float):
    conn = get_db_connection()
    cursor = conn.cursor()

    cursor.execute(
        "SELECT id, name, price, stock FROM products WHERE price <= %s ORDER BY price",
        (max_price,)           # Always use parameterized queries!
    )

    rows = cursor.fetchall()
    for row in rows:
        print(f"ID: {row[0]}  Name: {row[1]:<25}  Price: ${row[2]:<8}  Stock: {row[3]}")

    cursor.close()
    conn.close()
    return rows

# Call the function
get_products_by_price(100.00)
```

**Sample Output:**
```
ID: 2  Name: Wireless Mouse           Price: $29.99   Stock: 200
ID: 4  Name: USB-C Hub                Price: $49.99   Stock: 300
ID: 3  Name: Mechanical Keyboard      Price: $89.99   Stock: 150
```

---

### Create RDS Snapshot with boto3

```python
import boto3
from datetime import datetime

def create_rds_snapshot(db_identifier: str):
    """Create a manual RDS snapshot."""
    client = boto3.client('rds', region_name='us-east-1')

    timestamp = datetime.now().strftime('%Y%m%d-%H%M%S')
    snapshot_id = f"{db_identifier}-snap-{timestamp}"

    response = client.create_db_snapshot(
        DBInstanceIdentifier=db_identifier,
        DBSnapshotIdentifier=snapshot_id,
        Tags=[
            {'Key': 'CreatedBy', 'Value': 'boto3-script'},
            {'Key': 'Date', 'Value': timestamp}
        ]
    )

    snap = response['DBSnapshot']
    print(f"Snapshot created: {snap['DBSnapshotIdentifier']}")
    print(f"Status:           {snap['Status']}")
    print(f"Engine:           {snap['Engine']} {snap['EngineVersion']}")
    print(f"Storage:          {snap['AllocatedStorage']} GB")

create_rds_snapshot('my-postgres-db')
```

**Sample Output:**
```
Snapshot created: my-postgres-db-snap-20260310-143052
Status:           creating
Engine:           postgres 15.4
Storage:          20 GB
```

---

### List All RDS Instances

```python
import boto3

def list_rds_instances():
    client = boto3.client('rds', region_name='us-east-1')
    response = client.describe_db_instances()

    print(f"{'Instance ID':<30} {'Engine':<12} {'Class':<15} {'Status':<12} {'Multi-AZ'}")
    print("─" * 85)

    for db in response['DBInstances']:
        print(
            f"{db['DBInstanceIdentifier']:<30} "
            f"{db['Engine']:<12} "
            f"{db['DBInstanceClass']:<15} "
            f"{db['DBInstanceStatus']:<12} "
            f"{db['MultiAZ']}"
        )

list_rds_instances()
```

**Sample Output:**
```
Instance ID                    Engine       Class           Status       Multi-AZ
─────────────────────────────────────────────────────────────────────────────────────
my-postgres-db                 postgres     db.t3.micro     available    True
my-postgres-db-replica         postgres     db.t3.micro     available    False
my-mysql-dev                   mysql        db.t3.micro     available    False
```

---

## 17. Terraform Example

```hcl
# ─── variables.tf ────────────────────────────────────────

variable "db_password" {
  description = "RDS master password"
  type        = string
  sensitive   = true
}

# ─── rds.tf ──────────────────────────────────────────────

# DB Subnet Group
resource "aws_db_subnet_group" "main" {
  name       = "main-db-subnet-group"
  subnet_ids = [aws_subnet.private_1a.id, aws_subnet.private_1b.id]
  tags       = { Name = "main-db-subnet-group" }
}

# DB Parameter Group
resource "aws_db_parameter_group" "postgres15" {
  family = "postgres15"
  name   = "my-postgres15-params"

  parameter {
    name  = "log_min_duration_statement"
    value = "1000"
  }

  parameter {
    name  = "rds.force_ssl"
    value = "1"
  }

  parameter {
    name  = "log_connections"
    value = "1"
  }
}

# Security Group for RDS
resource "aws_security_group" "rds" {
  name        = "rds-sg"
  description = "Allow PostgreSQL from app servers only"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]  # Only from app SG
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "rds-sg" }
}

# RDS PostgreSQL Instance
resource "aws_db_instance" "main" {
  identifier             = "my-postgres-db"
  engine                 = "postgres"
  engine_version         = "15.4"
  instance_class         = "db.t3.micro"
  allocated_storage      = 20
  max_allocated_storage  = 100       # Auto-scaling up to 100 GB
  storage_type           = "gp3"
  storage_encrypted      = true

  db_name  = "myappdb"
  username = "dbadmin"
  password = var.db_password

  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.rds.id]
  parameter_group_name   = aws_db_parameter_group.postgres15.name

  # High Availability
  multi_az = true

  # Backups
  backup_retention_period   = 7
  backup_window             = "03:00-04:00"
  maintenance_window        = "Mon:04:00-Mon:05:00"
  copy_tags_to_snapshot     = true
  delete_automated_backups  = false

  # Access
  publicly_accessible = false

  # Protection
  deletion_protection       = true
  skip_final_snapshot       = false
  final_snapshot_identifier = "my-postgres-db-final"

  # Monitoring
  monitoring_interval             = 60
  monitoring_role_arn             = aws_iam_role.rds_monitoring.arn
  performance_insights_enabled    = true
  performance_insights_retention_period = 7
  enabled_cloudwatch_logs_exports = ["postgresql", "upgrade"]

  tags = {
    Name        = "my-postgres-db"
    Environment = "production"
  }
}

# Read Replica
resource "aws_db_instance" "replica" {
  identifier             = "my-postgres-db-replica"
  replicate_source_db    = aws_db_instance.main.identifier
  instance_class         = "db.t3.micro"
  publicly_accessible    = false
  skip_final_snapshot    = true
  vpc_security_group_ids = [aws_security_group.rds.id]

  tags = { Name = "my-postgres-db-replica" }
}

# ─── outputs.tf ──────────────────────────────────────────

output "rds_endpoint"         { value = aws_db_instance.main.endpoint }
output "rds_replica_endpoint" { value = aws_db_instance.replica.endpoint }
output "rds_port"             { value = aws_db_instance.main.port }
output "rds_db_name"          { value = aws_db_instance.main.db_name }
```

---

## 18. RDS Pricing Model

### Cost Components

| Component | Description |
|-----------|-------------|
| **DB Instance hours** | Charged per hour for the running instance class |
| **Storage (GB/month)** | gp3 cheaper than gp2; io1 most expensive |
| **I/O requests** | Only for magnetic storage (not gp2/gp3/io1) |
| **Backup storage** | Free up to DB size; charged beyond that |
| **Data transfer** | Free into RDS; charged for outbound to internet |
| **Multi-AZ** | ~2× single-AZ cost (standby instance) |
| **Read replicas** | Same instance cost as primary |

### Pricing Modes

| Mode | Best For | Savings |
|------|----------|---------|
| **On-Demand** | Variable / unpredictable workloads | Baseline |
| **Reserved (1yr)** | Steady-state production | ~40% savings |
| **Reserved (3yr)** | Long-term steady workloads | ~60% savings |

### Free Tier (12 months)

```
✅ db.t3.micro — 750 hours/month
✅ 20 GB gp2 storage
✅ 20 GB backup storage
✅ Engines: MySQL, PostgreSQL, MariaDB
```

---

## 19. Most Asked Interview & Exam Questions

### Q1: What is the difference between RDS Multi-AZ and Read Replicas?

| | Multi-AZ | Read Replica |
|--|----------|-------------|
| Purpose | **High Availability** | **Read Scaling** |
| Replication | Synchronous | Asynchronous |
| Standby readable | ❌ No | ✅ Yes |
| Automatic failover | ✅ Yes | ❌ No (manual promote) |
| Separate endpoint | ❌ Same DNS | ✅ Own endpoint |
| Cross-region | ❌ No | ✅ Yes |

---

### Q2: Can you take a snapshot of a Multi-AZ RDS instance?

**Yes.** Snapshots are taken from the **standby replica**, which means **zero performance impact** on the primary (production) database.

---

### Q3: What happens during an RDS Multi-AZ failover?

1. AWS detects failure (primary crash, AZ outage, or maintenance)
2. The **CNAME (DNS endpoint) is flipped** to point to the standby
3. Standby becomes the new primary
4. Total downtime: **~60–120 seconds**
5. Your application reconnects using the **same endpoint** (no config change needed)

---

### Q4: Is data lost during Multi-AZ failover?

**No.** Multi-AZ uses **synchronous replication** — every write to the primary is simultaneously written to the standby before the primary acknowledges success. So there is **zero data loss (RPO = 0)**.

---

### Q5: What is Point-in-Time Recovery (PITR)?

PITR allows you to restore your RDS database to **any specific second** within your backup retention window (1–35 days). It uses daily snapshots + continuous transaction logs stored in S3.

```
Example:
  Retention: 7 days
  Incident: Data deleted at March 10, 2026 14:30:00
  PITR: Restore to March 10, 2026 14:29:59  ← 1 second before!
```

---

### Q6: Can you resize (scale up/down) an RDS instance?

**Yes.** You can change the instance class at any time:
- **Immediately:** Brief downtime (few minutes) for Multi-AZ (failover to standby)
- **During maintenance window:** Applied during the next scheduled window

```bash
aws rds modify-db-instance \
  --db-instance-identifier my-db \
  --db-instance-class db.m5.large \
  --apply-immediately
```

---

### Q7: Can you enable encryption on an existing unencrypted RDS instance?

**No — not directly.** To encrypt an unencrypted instance:
1. Take a snapshot of the unencrypted instance
2. Copy the snapshot with encryption enabled
3. Restore a new instance from the encrypted snapshot
4. Update your app to point to the new instance
5. Delete the old unencrypted instance

---

### Q8: What is the difference between RDS and Aurora?

| Feature | RDS (MySQL/PG) | Aurora |
|---------|---------------|--------|
| Storage | Per-instance EBS | Shared cluster volume |
| Replicas | Up to 5 | Up to 15 |
| Failover | ~60–120 sec | ~30 sec |
| Performance | Standard | 5× MySQL, 3× PG |
| Storage HA | Multi-AZ only | 6 copies/3 AZs always |
| Cost | Lower | Higher |
| Serverless | ❌ | ✅ v2 |

---

### Q9: What is RDS Proxy and when should you use it?

RDS Proxy is a **fully managed connection pool** between your app and RDS. Use it when:
- Running **AWS Lambda** functions (Lambda creates new connections per invocation)
- Application has **connection spikes** that overwhelm the DB
- You want **faster Multi-AZ failover** (cuts failover time by 66%)
- You need **IAM authentication** enforcement

---

### Q10: Can RDS be publicly accessible? Should it be?

RDS **can** be set to publicly accessible (`PubliclyAccessible = true`), which assigns a public IP. However, this is **strongly discouraged** for production. Best practice:
- Place RDS in a **private subnet**
- Allow access **only** from app server Security Group
- Never expose DB port to `0.0.0.0/0`

---

### Q11: What is a DB Subnet Group?

A DB Subnet Group is a **collection of subnets** across multiple AZs within a VPC that you assign to your RDS instance. Requirements:
- Minimum **2 subnets in different AZs**
- Subnets should be **private** (no route to IGW)
- Required before creating any RDS instance in a VPC

---

### Q12: How does RDS handle OS and DB patching?

AWS handles all patching automatically:
- **Minor version upgrades:** Applied during the maintenance window (if auto-upgrade is on)
- **Major version upgrades:** Manual — you initiate this
- **OS patches:** Applied transparently by AWS
- **Multi-AZ:** Standby patched first → failover → primary patched (minimal downtime)

---

### Q13: What is the maximum storage size for RDS?

| Engine | Max Storage |
|--------|------------|
| MySQL / MariaDB | 64 TB |
| PostgreSQL | 64 TB |
| Oracle | 64 TB |
| SQL Server | 16 TB |
| Aurora | **128 TB** (auto-grows) |

---

### Q14: What is the difference between `apply-immediately` and maintenance window?

| Option | Effect | Impact |
|--------|--------|--------|
| `--apply-immediately` | Changes applied NOW | May cause brief downtime |
| Maintenance window | Changes applied next scheduled window | Predictable downtime window |

For Multi-AZ, most changes use a **failover** (minimal impact). For Single-AZ, there's brief downtime.

---

### Q15: How do you secure RDS credentials in an application?

**Never hardcode credentials!** Best practices:

```python
# ✅ Option 1: AWS Secrets Manager (recommended)
import boto3, json

def get_db_credentials():
    client = boto3.client('secretsmanager', region_name='us-east-1')
    secret = client.get_secret_value(SecretId='prod/myapp/rds')
    creds = json.loads(secret['SecretString'])
    return creds['username'], creds['password']

# ✅ Option 2: Environment variables (from ECS task definition or EC2 IAM)
import os
DB_USER = os.environ['DB_USER']
DB_PASS = os.environ['DB_PASS']

# ✅ Option 3: IAM Database Authentication (no password at all)
# Uses temporary auth tokens via boto3 — token expires in 15 minutes
```

---

## 20. Summary Cheat Sheet

```
SERVICE           DESCRIPTION
────────────────────────────────────────────────────────────────────────
RDS               Managed relational DB — no OS/DB patching needed
Multi-AZ          Synchronous standby for automatic failover (HA)
Read Replica      Async read-only copy for scaling reads
Aurora            AWS-native DB — 5× MySQL speed, 6-way storage replication
Aurora Serverless Auto-scaling compute (0.5 → 128 ACU) — pay per use
RDS Proxy         Connection pooler — ideal for Lambda + RDS
Parameter Group   DB engine config file (max_connections, ssl, etc.)
Subnet Group      Private subnets across AZs for RDS placement
Automated Backup  Daily snapshot + tx logs → PITR up to 35 days
Manual Snapshot   User-triggered, persists indefinitely
Encryption        KMS AES-256 at rest; SSL/TLS in transit
Secrets Manager   Store & rotate DB credentials automatically
Performance Insights Visualize slow queries & DB load
Enhanced Monitoring OS-level metrics (CPU steal, memory, disk)

PORT REFERENCE
────────────────────────────────────────────────────────────────────────
MySQL / Aurora MySQL      3306
PostgreSQL / Aurora PG    5432
MariaDB                   3306
Oracle                    1521
SQL Server                1433

INSTANCE QUICK PICK
────────────────────────────────────────────────────────────────────────
Dev / Test        db.t3.micro  (Free tier eligible)
Small Production  db.t3.medium or db.m5.large
High Traffic      db.m5.xlarge or db.r5.2xlarge
Memory-heavy      db.r5.4xlarge / db.r6g.8xlarge
```

---

*Last Updated: March 2026 | AWS CLI v2 | boto3 1.34+ | Terraform v1.7+*
