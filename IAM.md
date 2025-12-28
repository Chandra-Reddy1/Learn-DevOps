# Complete AWS Guide: Top 10 Most-Used Services
## From Zero to Production-Ready Knowledge

**Target Audience:** Basic AWS Learners with IT fundamentals  
**Approach:** Real-world, industry-focused, beginner-friendly

---

# Table of Contents
1. [IAM (Identity and Access Management)](#1-iam)
2. [EC2 (Elastic Compute Cloud)](#2-ec2)
3. [S3 (Simple Storage Service)](#3-s3)
4. [VPC (Virtual Private Cloud)](#4-vpc)
5. [RDS (Relational Database Service)](#5-rds)
6. [ELB (Elastic Load Balancer)](#6-elb)
7. [Auto Scaling](#7-auto-scaling)
8. [CloudWatch](#8-cloudwatch)
9. [AWS CodePipeline](#9-codepipeline)
10. [AWS EKS (Elastic Kubernetes Service)](#10-eks)

---

# 1. IAM (Identity and Access Management) {#1-iam}

## 1.1 WHY THIS SERVICE EXISTS (DEEP BUT SIMPLE)

### What exact problem it solves:

IAM solves: **"Who can do what in my AWS account?"**

**Real-world analogy:**
- Your AWS account = Office building with 100 rooms
- IAM = Security badge system
- Each badge opens specific doors only
- Security logs track who entered where and when

### What pain companies had BEFORE IAM:

**Pain 1: Everyone shared one password**
- All 50 employees used same AWS login
- Junior dev accidentally deleted production database
- Can't identify who did it (everyone shared credentials)
- Employee leaves → Must change password → Notify all 49 others

**Pain 2: No access control**
- Intern needs to upload test files to S3
- Only option: Give full AWS access
- Intern accidentally terminates production servers
- Company loses $100,000 in downtime

**Pain 3: No audit trail**
- Database deleted at 3 AM
- Who did it? Unknown
- Why? Unknown
- Compliance audit fails

### What breaks if IAM is NOT used:

1. **Security disasters:**
   - Ex-employees still have access
   - One compromised password = entire infrastructure at risk
   - Can't limit damage from mistakes

2. **Compliance failures:**
   - HIPAA, SOC 2, PCI-DSS all require access controls
   - Must prove "who accessed patient data"
   - Without IAM: Impossible to pass audits

3. **Financial disasters:**
   - Someone launches 500 expensive servers by mistake
   - No way to prevent it (everyone has full access)
   - Surprise bill: $50,000

---

## 1.2 REAL-WORLD USE CASES (DEEP EXPLANATION)

### Startup (5-20 people):

**Scenario:** Food delivery app startup

**Team:**
- 1 CEO (Sarah) → Needs: Billing access only
- 1 CTO (Mike) → Needs: Full admin access
- 2 Backend Devs → Need: EC2, RDS, Lambda access
- 1 Frontend Dev → Needs: S3, CloudFront access
- 2 Interns (summer) → Need: Dev environment only

**IAM Setup:**
```
IAM Users: 7
├── sarah-ceo → Policy: ViewBilling
├── mike-cto → Policy: AdministratorAccess
├── anna-backend → Group: Backend-Developers
├── john-backend → Group: Backend-Developers
├── lisa-frontend → Group: Frontend-Developers
├── intern-1 → Group: Interns (expires Sept 1)
└── intern-2 → Group: Interns (expires Sept 1)

IAM Groups: 3
├── Backend-Developers
│   ├── AmazonEC2FullAccess
│   ├── AmazonRDSFullAccess
│   └── AWSLambdaFullAccess
├── Frontend-Developers
│   ├── AmazonS3FullAccess
│   └── CloudFrontFullAccess
└── Interns
    ├── DevEnvironmentAccess (custom policy)
    └── Condition: Only 9 AM - 6 PM access
```

**Real incident they prevented:**
- Intern tried to terminate production EC2
- IAM denied: "User has no permission for production resources"
- Production stayed safe

---

### Mid-size Company (200 people, $10M revenue):

**Scenario:** E-commerce platform

**Complex requirements:**
```
Engineering (60 people)
├── Frontend Team (15) → Deploy web servers, no database access
├── Backend Team (20) → Database access, can't delete backups
├── Mobile Team (15) → Upload to app stores, limited AWS
└── DevOps Team (10) → Infrastructure, no customer data access

Data Team (20)
└── Read-only database access for reports

Finance (10)
└── Billing dashboard access only

Customer Support (110)
└── NO AWS access (use separate support tools)
```

**IAM Architecture:**
```
AWS Accounts: 3 (Organization structure)
├── Production Account
│   ├── MFA required for all access
│   ├── Access only from office IPs
│   ├── All actions logged to CloudTrail
│   └── 50+ different IAM roles
├── Staging Account
│   └── Mirror of production for testing
└── Development Account
    └── Developers can experiment freely

Cross-Account Access:
- DevOps can assume role in Production (with approval)
- Data Team reads Production DB (read-only role)
```

**Applications also need IAM:**
```
Mobile App:
├── IAM Role: Upload user photos to S3
├── Can: PutObject to bucket "user-uploads"
└── Cannot: Delete, access other buckets, touch databases

Backend API:
├── IAM Role: Access user database
├── Can: Read/Write user tables
└── Cannot: Delete databases, access payment data

Payment Processor:
├── IAM Role: Access payment database only
├── Can: Process payments
├── Restricted: Specific IP addresses, business hours only
└── Cannot: Access user data, other services
```

---

### Large Enterprise (5,000+ people):

**Scenario:** Global bank

**Enterprise complexity:**
```
Multi-Account Strategy: 50+ AWS accounts
├── Master/Billing Account
├── Production US (10 accounts by business unit)
├── Production EU (10 accounts)
├── Production Asia (10 accounts)
├── Staging/QA (10 accounts)
└── Development (10 accounts)

Identity Federation:
├── Employees use corporate Windows login
├── SSO (Single Sign-On) to AWS
├── When employee leaves company → AWS access auto-revoked
└── No separate AWS passwords to manage

Service Control Policies (SCPs):
├── No one can launch servers outside approved regions
├── All data must be encrypted
├── CloudTrail logging cannot be disabled
└── Maximum EC2 instance size: r5.4xlarge
```

**Advanced IAM features:**
```
Permission Boundaries:
- DevOps can create IAM roles for services
- But those roles can never exceed predefined limits
- Prevents privilege escalation

Time-based Access:
- Production database access: 9 AM - 6 PM only
- Weekend access requires VP approval
- Emergency access available (fully logged)

Automated Compliance:
- AWS Config checks IAM policies every hour
- Alerts if any policy violates standards
- Auto-remediation for critical violations
```

---

### REALISTIC ARCHITECTURE EXAMPLE:

**Instagram-like Photo App (1M users)**

```
USER UPLOADS PHOTO:

[Mobile App]
├── IAM Credentials (temporary, expire hourly)
├── Can: s3:PutObject to "raw-photos/user-{id}/"
└── Cannot: Read other users' photos, delete anything
    ↓
[S3 Bucket: raw-photos]
    ↓ (triggers)
[Lambda Function: ProcessPhoto]
├── IAM Role: "PhotoProcessorRole"
├── Can: Read "raw-photos", Write "processed-photos"
├── Can: Invoke Rekognition (image analysis)
└── Cannot: Access databases, billing, other services
    ↓
[S3 Bucket: processed-photos]
    ↓
[API Server - EC2 Instances]
├── IAM Role: "APIServerRole"
├── Can: Read "processed-photos"
├── Can: Query user database (RDS)
├── Can: Write to cache (ElastiCache)
└── Cannot: Delete photos, modify billing
    ↓
[User sees photo in feed]

MODERATOR REVIEWS FLAGGED PHOTOS:

[Moderator Dashboard]
├── IAM User: "moderator-jane"
├── Can: View flagged photos, approve/reject
├── Can: Suspend user accounts
└── Cannot: See user passwords, payment info, modify code
    ↓ (all actions logged)
[CloudTrail: Audit log]
└── "moderator-jane removed photo xyz at 2:30 PM"
```

**Security principle:** Least Privilege
- Each component gets ONLY minimum permissions needed
- If one part is compromised, damage is limited

---

## 1.3 WHERE THIS SERVICE FITS IN AWS (BIG PICTURE)

### Category: **SECURITY & ACCESS CONTROL**

IAM is the **FOUNDATION** of everything in AWS. It's not optional - every action in AWS goes through IAM.

### How it interacts with other AWS services:

```
EVERY AWS REQUEST FLOW:

[User/Application makes request]
    ↓
[IAM Authentication]
├── Who are you?
├── Valid credentials?
└── MFA verified?
    ↓ (if authenticated)
[IAM Authorization]
├── What are you trying to do?
├── Do you have permission?
└── Any deny policies?
    ↓ (if authorized)
[AWS Service] (EC2, S3, RDS, etc.)
├── Executes the action
└── Logs result to CloudTrail
    ↓
[CloudWatch/CloudTrail]
└── Records: Who, What, When, Result
```

### What comes BEFORE IAM:

**NOTHING.** IAM is step 1.

```
AWS Account Creation:
1. Create AWS account → Get root user
2. Set up IAM (secure root, create users)
3. NOW you can use other services
```

### What comes AFTER IAM:

**EVERYTHING.** All services require IAM.

```
Typical Setup Order:

1. IAM
   ├── Create admin user (with MFA)
   ├── Create roles for services
   └── Create policies

2. VPC (Networking)
   ├── Uses IAM to control who can create networks
   └── IAM role needed to manage VPC

3. EC2 (Servers)
   ├── IAM role attached to each EC2 instance
   └── Defines what EC2 can access

4. S3 (Storage)
   ├── IAM policies control bucket access
   └── IAM roles for applications uploading files

5. RDS (Database)
   ├── IAM controls who can create/modify databases
   └── IAM authentication for database connections

... and so on for ALL AWS services
```

### Complete Application Stack with IAM:

```
[Route 53 - DNS]
├── IAM: Who can modify DNS records?
    ↓
[CloudFront - CDN]
├── IAM: Who can invalidate cache?
    ↓
[Application Load Balancer]
├── IAM: Who can modify routing rules?
    ↓
[EC2 Instances]
├── IAM Role attached (for accessing other services)
├── IAM: Who can SSH into these servers?
    ↓
[RDS Database]
├── IAM: Who can query/modify database?
├── IAM authentication option (instead of password)
    ↓
[S3 Storage]
├── IAM: Who can upload/download files?
    ↓
[CloudWatch Monitoring]
├── IAM: Who can view metrics/logs?
    ↓
[CloudTrail Audit Logs]
└── IAM: Who can view/export audit logs?

IAM controls access to EVERY layer.
```

---

## 1.4 PREREQUISITES (BEGINNER-FRIENDLY)

### Concept 1: Authentication vs Authorization

**Authentication** = Proving WHO you are
```
Examples:
- Username + Password
- Access Key ID + Secret Access Key
- MFA token
- Fingerprint scan

Like showing your ID at airport security.
```

**Authorization** = Proving WHAT you can do
```
Examples:
- IAM Policy says you can read S3
- IAM Policy says you cannot delete databases

Like your boarding pass showing which gate you can enter.
```

**Remember:**
- Authentication: "Are you really John?"
- Authorization: "Is John allowed to delete this?"

---

### Concept 2: Principle of Least Privilege

**Definition:** Give minimum permissions needed, nothing more.

**Bad approach:**
```
❌ Give everyone AdministratorAccess
Why it's bad:
- Junior dev can accidentally delete everything
- If one account compromised, attacker has full access
- Can't restrict damage
```

**Good approach:**
```
✅ Give specific permissions only

Frontend Dev:
- Needs: S3 access for website files
- Gets: AmazonS3FullAccess only
- Cannot: Touch databases, servers, billing

Backend Dev:
- Needs: EC2 and RDS access
- Gets: Custom policy for specific EC2/RDS permissions
- Cannot: Delete backups, access billing

Contractor:
- Needs: Temporary access to dev environment
- Gets: Time-limited role (expires in 30 days)
- Cannot: Access production
```

---

### Concept 3: IAM Components

**1. IAM Users** = Individual people
```
Example:
- john@company.com → IAM User "john-backend-dev"
- Has: Username, password, optional MFA
- Uses: To log into AWS Console
```

**2. IAM Groups** = Collection of users
```
Example:
- Group: "Developers" (contains 20 users)
- Attach policy to group once
- All 20 users get those permissions
- Easier than managing 20 users individually
```

**3. IAM Roles** = Permissions for services/applications
```
Example:
- Role: "EC2-S3-Access-Role"
- EC2 instance "assumes" this role
- Gets temporary credentials (expire after hours)
- No password stored on server (secure!)

Key difference: Users have passwords, Roles don't.
```

**4. IAM Policies** = JSON documents defining permissions
```
Example:
{
  "Effect": "Allow",
  "Action": "s3:PutObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}

Translation: "Allow uploading files to my-bucket"
```

---

### Concept 4: Policy Structure (JSON)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3Upload",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

**Line-by-line explanation:**
```
"Version": "2012-10-17"
→ Policy language version (always use this)

"Statement": [...]
→ List of permission rules (can have multiple)

"Sid": "AllowS3Upload"
→ Statement ID (optional, for humans to read)

"Effect": "Allow" or "Deny"
→ Allow grants permission, Deny blocks it

"Action": ["s3:PutObject", "s3:GetObject"]
→ What actions are allowed
→ s3:PutObject = upload files
→ s3:GetObject = download files

"Resource": "arn:aws:s3:::my-bucket/*"
→ WHICH bucket/files
→ /* means all files in bucket

"Condition": {...}
→ Optional: Additional restrictions
→ This example: Only from specific IP addresses
```

---

### Concept 5: ARN (Amazon Resource Name)

**Format:**
```
arn:aws:service:region:account-id:resource

Examples:

S3 bucket:
arn:aws:s3:::my-website-bucket
    ↑   ↑           ↑
   AWS  S3      bucket name

EC2 instance:
arn:aws:ec2:us-east-1:123456789012:instance/i-1234567890abcdef0
    ↑   ↑      ↑           ↑              ↑
   AWS EC2   region    account ID    instance ID

IAM user:
arn:aws:iam::123456789012:user/john-admin
    ↑   ↑        ↑            ↑
   AWS IAM  account ID    username
```

**Think of ARN as:**  
The complete "address" of something in AWS, like a postal address for a house.

---

### Permissions Required (To Learn IAM):

**If you're using your own AWS account:**
- You have full access (root user)
- Can do everything

**If using company AWS account:**
Ask your admin for these permissions:
```
iam:CreateUser
iam:CreateGroup
iam:CreateRole
iam:CreatePolicy
iam:AttachUserPolicy
iam:AttachRolePolicy
iam:ListUsers
iam:ListRoles
iam:ListPolicies
```

Or simply ask for: **IAMFullAccess** policy

---

## 1.5 STEP-BY-STEP CONFIGURATION (AWS CONSOLE)

### ⚠️ CRITICAL FIRST STEP: Secure Your Root Account

**Current situation:**
- You created AWS account with email + password
- This is "root user" (has unlimited power)
- Using root daily = DANGEROUS

**Industry best practice:**
1. Secure root account (add MFA)
2. Create admin IAM user for yourself
3. **NEVER use root again** (except billing/emergency)

---

### PHASE 1: Secure Root Account

#### Step 1: Enable MFA on Root

**What is MFA?**
- Multi-Factor Authentication
- Need TWO things to log in:
  1. Password (something you know)
  2. 6-digit code from phone (something you have)

**Why critical:**
If hacker steals your password, they still can't log in without your phone.

**Steps:**

1. Log into AWS Console: https://console.aws.amazon.com
2. Click your account name (top-right)
3. Click "Security credentials"
4. Find "Multi-factor authentication (MFA)" section
5. Click "Activate MFA"

6. Choose: **"Virtual MFA device"**
   - ❌ Don't choose: "U2F security key" (costs money)
   - ❌ Don't choose: "Other hardware MFA" (old)

7. Install authenticator app on phone:
   - iPhone: Google Authenticator or Authy
   - Android: Google Authenticator or Authy

8. Click "Show QR code"
9. Scan with your authenticator app
10. App shows 6-digit code
11. Enter code in "MFA code 1"
12. Wait 30 seconds for code to change
13. Enter NEW code in "MFA code 2"
14. Click "Assign MFA"

**Success:** You see "MFA device assigned"

**Test it:**
- Log out
- Log back in
- Now asks for MFA code (check your phone app)

---

### PHASE 2: Create Admin IAM User

#### Step 2: Navigate to IAM

1. AWS Console search bar → Type "IAM"
2. Click "IAM" service
3. You see IAM Dashboard

**You'll see:**
```
Users: 0 (none yet)
Groups: 0
Roles: Some (AWS creates defaults)
Policies: Many (AWS-managed)
```

---

#### Step 3: Create SSH Key Pair First

**Why now?** You'll need this for EC2 later. Better to create it once.

1. Search "EC2" → Click EC2
2. Left sidebar → "Key Pairs"
3. Click "Create key pair"
4. Name: "my-first-key"
5. Type: **RSA**
6. Format: **.pem** (Mac/Linux) or **.ppk** (Windows/PuTTY)
7. Click "Create"
8. File downloads → **SAVE IT SECURELY**

Move to safe location:
```bash
# Mac/Linux
mkdir -p ~/.ssh
mv ~/Downloads/my-first-key.pem ~/.ssh/
chmod 400 ~/.ssh/my-first-key.pem
```

---

#### Step 4: Create Your Admin IAM User

Back to IAM Dashboard:

1. Left sidebar → "Users"
2. Click "Create user"

**Screen 1: User details**

3. Username: "your-name-admin" (example: "john-admin")
   - Use: firstname-purpose format
   - ❌ No spaces or special characters

4. ✅ Check: "Provide user access to AWS Management Console"
5. Select: "I want to create an IAM user"
6. Password option: "Custom password"
7. Enter strong password (12+ characters)
8. ❌ Uncheck: "Users must create new password at next sign-in"
   - Why: This is YOUR account

9. Click "Next"

---

**Screen 2: Permissions**

10. Select: **"Attach policies directly"**

11. Search: "AdministratorAccess"
12. ✅ Check the box next to "AdministratorAccess"
    - This gives full AWS access
    - OK for learning, not for production

13. Click "Next"

---

**Screen 3: Review**

14. Review:
```
Username: john-admin
Console access: Enabled
Permissions: AdministratorAccess
```

15. Click "Create user"

---

**Screen 4: Success!**

16. **CRITICAL:** Save this info NOW!

```
Console sign-in URL: https://123456789012.signin.aws.amazon.com/console
Username: john-admin
Password: [your password]
```

17. **Bookmark the Console sign-in URL**
18. Click "Download .csv" (backup)
19. Click "Return to users list"

---

#### Step 5: Test IAM User Login

**Log out of root account:**
1. Top-right → Account name → "Sign out"

**Log in as IAM user:**
1. Go to Console sign-in URL you saved
2. Account ID: (pre-filled or enter 12-digit number)
3. IAM user name: "john-admin"
4. Password: [your password]
5. Click "Sign in"

**Success indicator:**
Top-right shows: "john-admin @ your-account-name"
(NOT your email address)

---

#### Step 6: Add MFA to IAM User

Even though this is your admin user, add MFA for security.

1. IAM Dashboard
2. Users → Click "john-admin"
3. "Security credentials" tab
4. MFA section → "Assign MFA device"
5. Repeat same process as root MFA

---

### PHASE 3: Create IAM Groups (Best Practice)

#### Step 7: Create Developer Group

1. IAM → Left sidebar → "User groups"
2. Click "Create group"

**Screen: Create group**

3. Group name: "Developers"
4. Search policies: "AmazonEC2FullAccess"
5. ✅ Check box
6. Search: "AmazonS3FullAccess"
7. ✅ Check box
8. Click "Create group"

**What you just created:**
Group with permissions to EC2 and S3 (common developer needs)

---

#### Step 8: Create More Groups

Repeat for other common groups:

**Data Analysts Group:**
```
Name: DataAnalysts
Policies:
- AmazonS3ReadOnlyAccess
- AmazonAthenaFullAccess
```

**Finance Group:**
```
Name: Finance
Policies:
- Billing (AWS managed policy)
```

---

### PHASE 4: Create IAM Role (For Services)

#### Step 9: Create EC2 Role for S3 Access

**Scenario:** EC2 server needs to upload files to S3

1. IAM → "Roles" → "Create role"

**Screen 1: Trusted entity**

2. Select: **"AWS service"**
3. Use case: **"EC2"**
   - Means: EC2 instances can use this role
4. Click "Next"

---

**Screen 2: Permissions**

5. Search: "AmazonS3FullAccess"
6. ✅ Check box
7. Click "Next"

---

**Screen 3: Name**

8. Role name: "EC2-S3-Access-Role"
   - Convention: [WhoUsesIt]-[WhatItAccesses]-Role
9. Description: "Allows EC2 instances to read/write S3 buckets"
10. Click "Create role"

**Success!** Role created.

**How to use:**
When launching EC2, attach this role → EC2 automatically gets S3 permissions.

---

### PHASE 5: Create Custom Policy

#### Step 10: S3 Upload-Only Policy

**Scenario:** Allow uploading but NOT deleting files.

1. IAM → "Policies" → "Create policy"
2. Click "JSON" tab
3. Delete everything
4. Paste this:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowS3Upload",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::my-upload-bucket/*"
        },
        {
            "Sid": "AllowListBucket",
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::my-upload-bucket"
        }
    ]
}
```

**What this policy does:**
```
✅ Allows: Upload files to my-upload-bucket
✅ Allows: List files in bucket
❌ Blocks: Delete files
❌ Blocks: Access to other buckets
```

5. Click "Next"
6. Policy name: "S3-Upload-Only-Policy"
7. Description: "Upload to my-upload-bucket, no delete"
8. Click "Create policy"

---

### COMMON BEGINNER MISTAKES

❌ **Mistake 1:** Using root account daily
```
Danger: Unlimited power, no safety net
Fix: Create admin IAM user, use that
```

❌ **Mistake 2:** No MFA enabled
```
Danger: Password stolen = full account access
Fix: Enable MFA on root AND all IAM users
```

❌ **Mistake 3:** Everyone gets AdministratorAccess
```
Danger: Junior dev can delete production
Fix: Use groups with specific permissions
```

❌ **Mistake 4:** Hardcoding credentials in code
```python
# ❌ NEVER DO THIS:
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"
AWS_SECRET_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCY"
```
Fix: Use IAM roles for EC2/Lambda

❌ **Mistake 5:** Lost .pem key file
```
Danger: Can't SSH into EC2, locked out forever
Fix: Keep backup in secure location
```

❌ **Mistake 6:** SSH security group allows 0.0.0.0/0
```
Danger: Everyone on internet can try to hack your server
Fix: SSH should only allow your IP
```

---

## 1.6 MINIMAL REALISTIC HANDS-ON EXAMPLE

### Scenario: 3-Person Startup Team Setup

**Team:**
- Alice (CTO) → Full admin access
- Bob (Backend Dev) → EC2, RDS only
- Charlie (Frontend Dev) → S3, CloudFront only

---

### Implementation:

#### Step 1: Create Users

1. IAM → Users → Create user
2. Username: "alice-cto"
3. Console access: ✅
4. Password: [strong password]
5. Permissions: AdministratorAccess
6. Create user

Repeat for:
- "bob-backend"
- "charlie-frontend"

---

#### Step 2: Create Groups

**Backend Developers Group:**
1. User groups → Create group
2. Name: "Backend-Developers"
3. Attach policies:
   - AmazonEC2FullAccess
   - AmazonRDSFullAccess
4. Create group

**Frontend Developers Group:**
1. Create group
2. Name: "Frontend-Developers"
3. Attach policies:
   - AmazonS3FullAccess
   - CloudFrontFullAccess
4. Create group

---

#### Step 3: Add Users to Groups

1. IAM → Users → Click "bob-backend"
2. "Groups" tab → "Add user to groups"
3. Select "Backend-Developers"
4. Click "Add to groups"

Repeat for Charlie → Frontend-Developers

---

#### Step 4: Verify Permissions

1. Log in as Bob
2. Try to access EC2 → ✅ Works
3. Try to access S3 → ❌ Access Denied (correct!)

1. Log in as Charlie
2. Try to access S3 → ✅ Works
3. Try to access EC2 → ❌ Access Denied (correct!)

---

### Behind the Scenes:

```
Bob tries to view EC2 instances:

1. Bob logs in → IAM authenticates
2. Bob clicks "EC2 Dashboard"
3. IAM checks: Does bob-backend have ec2:DescribeInstances?
4. Bob is in "Backend-Developers" group
5. Group has "AmazonEC2FullAccess" policy
6. Policy includes ec2:DescribeInstances
7. IAM: ✅ Allowed
8. Bob sees EC2 dashboard

Bob tries to view S3 buckets:

1. Bob clicks "S3"
2. IAM checks: Does bob-backend have s3:ListAllMyBuckets?
3. Bob is in "Backend-Developers" group
4. Group policies: EC2, RDS only (no S3)
5. IAM: ❌ Denied
6. Bob sees "Access Denied" error
```

---

## 1.7 COST UNDERSTANDING (NO SURPRISES)

### What causes billing:

**IAM itself: 100% FREE**

```
✅ Free forever:
- Creating IAM users (unlimited)
- Creating IAM roles (unlimited)
- Creating IAM policies (unlimited)
- Creating IAM groups (unlimited)
- Using IAM to control access (unlimited)
```

**The ONLY way IAM costs money (indirectly):**

Wrong permissions → Someone launches expensive resources

**Example 1:**
```
Gave intern AdministratorAccess
Intern launches 100 p3.8xlarge instances (GPU servers)
Cost: $30,000/day
Problem: Not IAM cost, but IAM ALLOWED it
```

**Example 2:**
```
IAM credentials leaked on GitHub
Hacker launches crypto mining servers
Cost: $50,000 before you notice
Problem: IAM credentials not rotated/secured
```

---

### Free Tier:

**IAM has NO free tier because it's ALWAYS free.**

**Service limits (all free):**
```
IAM users per account: 5,000
IAM roles per account: 1,000
IAM groups per account: 300
IAM policies per user: 10 managed policies
Policy size: 6,144 characters
```

These limits are HUGE - most companies never hit them.

---

### Mistakes that cause unexpected bills:

❌ **Mistake 1:** Overly permissive policies
```
Policy allows launching ANY instance type
Someone launches 50× r5.24xlarge instances
Cost: $2,000/day
```

**Prevention:**
```json
{
    "Effect": "Deny",
    "Action": "ec2:RunInstances",
    "Resource": "*",
    "Condition": {
        "StringNotEquals": {
            "ec2:InstanceType": ["t2.micro", "t3.micro"]
        }
    }
}
```
This blocks launching anything except tiny free-tier instances.

---

❌ **Mistake 2:** Not rotating access keys
```
Old employee's access keys still valid
Keys leaked online
Attacker uses them to launch resources
Cost: Could be $10,000+ before noticed
```

**Prevention:**
- Delete access keys when employee leaves
- Rotate keys every 90 days
- Use IAM Access Analyzer (free tool to detect issues)

---

❌ **Mistake 3:** Root account compromised
```
Root account has no MFA
Password stolen
Attacker has unlimited access
Cost: EVERYTHING (could delete/steal all data)
```

**Prevention:**
- MFA on root account (REQUIRED)
- Never use root account daily
- Store root credentials in physical safe

---

### Stay Safe as Beginner:

**1. Set billing alert:**
```
CloudWatch → Billing → Create alert
Alert when: Estimated charges > $10
Send email notification
```

**2. Create learning policy:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:*",
                "ec2:DescribeInstances",
                "ec2:RunInstances"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "ec2:InstanceType": "t2.micro"
                }
            }
        }
    ]
}
```
Allows only tiny free-tier instances.

**3. Use tagging:**
```
Tag all resources: Environment=Learning
Check billing: Filter by tag
Spot expensive mistakes early
```

**4. Clean up regularly:**
```
Stop EC2 instances when not using (free while stopped)
Delete S3 buckets after learning (pay per GB stored)
Terminate resources you don't need
```

---

## 1.8 COMMON INTERVIEW QUESTIONS

### Question 1: "What is IAM and why is it important?"

❌ **Bad answer:** "IAM is AWS's authentication and authorization service."

✅ **Good answer:**
"IAM solves the problem of access control in AWS. Without it, everyone would share one account with unlimited access, which is a security nightmare.

For example, in my last project, we had 20 engineers. With IAM, we could:
- Give frontend devs S3 access only
- Give backend devs EC2 and RDS access
- Ensure junior engineers couldn't touch production
- Track who made changes when things broke

IAM is critical because one wrong permission can lead to data breaches or someone accidentally deleting the production database. It's the foundation of AWS security."

**Why this is better:** Shows understanding of real problems, gives concrete examples.

---

### Question 2: "What's the difference between IAM users, groups, and roles?"

✅ **Good answer:**
"Think of them like this:

**IAM Users** are for people. Each person gets their own account with username and password. Like giving each employee their own key card.

**IAM Groups** are collections of users needing the same permissions. Instead of giving permissions to 20 developers individually, I create a 'Developers' group and add them all. Much easier to manage.

**IAM Roles** are for AWS services and applications - they don't have passwords because services can't type. For example, if my EC2 server needs to read from S3, I create a role with S3 permissions and attach it to EC2. The instance automatically gets temporary credentials.

The key difference: Users are for people, roles are for things."

---

### Question 3: "Explain least privilege with a real example."

✅ **Good answer:**
"Least privilege means giving users ONLY the minimum permissions needed for their job - nothing more.

Real example from e-commerce project:

**Bad approach:** Give mobile app full S3 access
- App can upload photos ✓ (needed)
- App can delete any photo ✗ (not needed)
- App can list all buckets ✗ (not needed)
- If app is compromised, attacker deletes everything

**Good approach:** Custom policy
```json
{
    "Effect": "Allow",
    "Action": "s3:PutObject",
    "Resource": "arn:aws:s3:::user-photos/user-{userid}/*"
}
```
- App can ONLY upload to its own folder
- Cannot delete anything
- Cannot access other users' photos
- If compromised, damage limited to one user

You start with zero access and only add what's absolutely necessary."

---

### Question 4: "How would you secure a new AWS account?"

✅ **Good answer:**
"I'd follow this checklist:

**1. Secure root account immediately:**
- Enable MFA (Google Authenticator)
- Lock away root credentials
- Never use root for daily work

**2. Create admin IAM user for myself:**
- With MFA enabled
- Use this for daily work

**3. Set up role-based access:**
- 'Developers' group → EC2, S3, Lambda
- 'DevOps' group → Infrastructure access
- 'Finance' group → Billing read-only

**4. Enable CloudTrail:**
- Logs every action in AWS
- Essential for auditing: "Who deleted the database?"

**5. Set billing alerts:**
- Alert at $50, $100, $500
- Catch expensive mistakes early

**6. Create password policy:**
- 12+ characters minimum
- Require MFA for console access
- Rotate every 90 days

**7. Regular access reviews:**
- Audit unused IAM users monthly
- Remove access for people who left company
- Delete old access keys

These steps prevent 90% of security incidents I've seen."

---

### Question 5: "A developer needs temporary production access for debugging. What do you do?"

✅ **Good answer:**
"I would NEVER give direct credentials or permanent access. Here's the secure approach:

**1. Create temporary IAM role:**
```
Role: Production-DB-Read-Only
Permissions: 
- Read database only (no write/delete)
- Only during business hours (9 AM - 6 PM)
- Auto-expires after 4 hours
```

**2. Use AWS STS (Security Token Service):**
```bash
# Developer runs:
aws sts assume-role --role-arn arn:aws:iam::123:role/Production-DB-Read-Only

# Gets temporary credentials valid for 4 hours
# Credentials expire automatically
```

**3. Enable extra monitoring:**
- All queries logged to CloudWatch
- Security team gets notified when role assumed
- Full audit trail: who accessed what, when

**4. Alternative: RDS Query Editor**
- Developer uses AWS Console Query Editor
- No direct database credentials needed
- Every query logged automatically

Benefits:
- Access is temporary (auto-expires)
- Actions are tracked (full audit)
- Damage is limited (read-only)
- Security has visibility

Much better than giving permanent database password or IAM credentials."

---

## 1.9 PRACTICE TASK

### Task: Set Up 3-Person Team with Different Access

**Scenario:**  
You're setting up AWS for a startup with 3 people:
- Alice (CTO) → Full admin access
- Bob (Backend Dev) → EC2, RDS only
- Charlie (Frontend Dev) → S3, CloudFront only

---

### Your Steps:

**1. Create three IAM users:**
- alice-cto
- bob-backend
- charlie-frontend

**2. Create two groups:**
- Backend-Developers
- Frontend-Developers

**3. Assign policies:**
- Alice: AdministratorAccess (direct)
- Backend-Developers: AmazonEC2FullAccess, AmazonRDSFullAccess
- Frontend-Developers: AmazonS3FullAccess, CloudFrontFullAccess

**4. Add users to groups:**
- Bob → Backend-Developers
- Charlie → Frontend-Developers

---

### Success Criteria:

✅ **You've succeeded when:**

1. All 3 IAM users exist
2. Bob is in Backend-Developers group
3. Charlie is in Frontend-Developers group
4. Bob's permissions show:
   - AmazonEC2FullAccess ✓
   - AmazonRDSFullAccess ✓
   - (No S3 access)
5. Charlie's permissions show:
   - AmazonS3FullAccess ✓
   - CloudFrontFullAccess ✓
   - (No EC2 access)

---

### What to Observe:

**In IAM Dashboard:**
```
Users: 3
├── alice-cto (Policy: AdministratorAccess)
├── bob-backend (Group: Backend-Developers)
└── charlie-frontend (Group: Frontend-Developers)

Groups: 2
├── Backend-Developers (1 user, 2 policies)
└── Frontend-Developers (1 user, 2 policies)
```

**Test access (optional):**
1. Log in as Bob
2. Navigate to EC2 → ✅ Works
3. Navigate to S3 → ❌ "Access Denied"
4. Log out

5. Log in as Charlie
6. Navigate to S3 → ✅ Works
7. Navigate to EC2 → ❌ "Access Denied"

---

### Bonus Challenge:

Create a custom policy that allows Bob to:
- Start/stop EC2 instances ✓
- Describe EC2 instances ✓
- But NOT create new instances ✗
- And NOT terminate instances ✗

**Policy should allow:**
- `ec2:StartInstances`
- `ec2:StopInstances`
- `ec2:DescribeInstances`

**Policy should deny:**
- `ec2:RunInstances` (create new)
- `ec2:TerminateInstances` (delete)

---

## Summary: IAM Key Takeaways

### Core Principles:

1. **IAM is FREE** (always)
2. **IAM is the foundation** of AWS security
3. **Never use root account** for daily work
4. **Always enable MFA** on accounts with console access
5. **Follow least privilege** (minimum permissions needed)
6. **Use groups** to manage permissions (not individual users)
7. **Use roles** for services/applications (not users)
8. **IAM doesn't cost money**, but wrong permissions cause expensive mistakes

---

### Before Moving to EC2:

**You should have:**
- ✅ Admin IAM user with MFA (not using root)
- ✅ Root account secured and locked away
- ✅ Understanding of users vs groups vs roles
- ✅ Can create custom policies in JSON
- ✅ Completed the 3-person team practice task

**You should understand:**
- Authentication vs Authorization
- Principle of Least Privilege
- How IAM integrates with all AWS services
- IAM policy structure (JSON)

---

**Next Service: EC2 (Elastic Compute Cloud)**

Once you've completed the IAM practice task and feel comfortable with these concepts, we'll move to EC2 where you'll:
- Launch your first server
- Attach IAM roles to EC2 instances
- Configure security groups
- Deploy a real application

---

*End of IAM Module*

---

# Note About This Guide

This IAM module follows the exact template you requested:
1. Why service exists (deep but simple)
2. Real-world use cases (startups to enterprises)
3. Where it fits in AWS
4. Prerequisites (beginner-friendly)
5. Step-by-step configuration (AWS Console)
6. Realistic hands-on example
7. Cost understanding
8. Interview questions
9. Practice task

**The remaining 9 services (EC2, S3, VPC, RDS, ELB, Auto Scaling, CloudWatch, CodePipeline, EKS) will follow this EXACT same structure with the same level of detail.**

Due to the length (each service is 40-50 pages), this complete guide would be 400-500 pages total.

**Would you like me to:**
1. Continue with EC2 in the same detail?
2. Create all 10 services in a condensed version?
3. Provide the template and you replicate for other services?

Let me know how you'd like to proceed!
