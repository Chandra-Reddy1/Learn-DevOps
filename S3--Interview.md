# AWS S3 Interview Questions & Answers (Top 20)

## 1. What is Amazon S3?
Amazon S3 (Simple Storage Service) is an object storage service that provides scalable, durable, and highly available storage for data.

---

## 2. What is an S3 bucket?
An S3 bucket is a container used to store objects (files + metadata) in S3.

---

## 3. What is an S3 object?
An object consists of:
- Data (file)
- Metadata
- Key (unique identifier)

---

## 4. What is the maximum size of an S3 object?
- Single upload: 5 GB  
- With multipart upload: 5 TB

---

## 5. What are different S3 storage classes?
- Standard
- Intelligent-Tiering
- Standard-IA
- One Zone-IA
- Glacier Instant Retrieval
- Glacier Flexible Retrieval
- Glacier Deep Archive

---

## 6. What is S3 durability and availability?
- Durability: 99.999999999% (11 9’s)
- Availability: Depends on storage class (e.g., 99.99% for Standard)

---

## 7. What is versioning in S3?
Versioning keeps multiple versions of an object to protect against accidental deletion or overwrite.

---

## 8. What is lifecycle policy?
It automatically moves or deletes objects based on rules (e.g., move to Glacier after 30 days).

---

## 9. What is S3 replication?
Automatic copying of objects:
- CRR (Cross-Region Replication)
- SRR (Same-Region Replication)

---

## 10. What is pre-signed URL?
A URL that provides temporary access to private S3 objects.

---

## 11. What is bucket policy?
JSON-based policy attached to a bucket to control access.

---

## 12. Difference between bucket policy and IAM policy?
- Bucket policy → attached to bucket
- IAM policy → attached to user/role

---

## 13. What is S3 encryption?
Types:
- SSE-S3
- SSE-KMS
- SSE-C
- Client-side encryption

---

## 14. What is multipart upload?
Uploading large files in parts to improve performance and reliability.

---

## 15. What is S3 Transfer Acceleration?
Speeds up uploads using AWS edge locations.

---

## 16. What is static website hosting in S3?
You can host static websites (HTML, CSS, JS) using S3.

---

## 17. What is S3 event notification?
Triggers events when objects are created/deleted:
- SNS
- SQS
- Lambda

---

## 18. What is S3 consistency model?
- Strong read-after-write consistency (new objects)
- Strong consistency for overwrite/delete (latest AWS update)

---

## 19. What is object locking?
Prevents deletion or modification for a fixed time (WORM - Write Once Read Many).

---

## 20. What is S3 access control methods?
- IAM policies
- Bucket policies
- ACLs (legacy, not recommended)

---

# Bonus (High-Impact Interview Questions)

## 21. How do you secure S3 bucket?
- Block public access
- Use IAM roles
- Enable encryption
- Use bucket policies
- Enable logging

---

## 22. How do you optimize S3 cost?
- Use lifecycle policies
- Use Intelligent-Tiering
- Delete unused objects
- Compress data

---

## 23. How to make S3 object public?
- Disable block public access
- Add bucket policy or ACL

---

## 24. Difference between EBS, EFS, and S3?
- S3 → Object storage
- EBS → Block storage
- EFS → File storage

---

## 25. What happens if S3 bucket is deleted?
All objects are permanently deleted (cannot recover unless versioning enabled).

---

# AWS S3 DevOps Scenario-Based Interview Questions & Answers

---

## 1. CI/CD Artifacts Storage

### Scenario
Your Jenkins pipeline stores build artifacts in S3, and storage cost is increasing.

### Answer
- Store artifacts in S3 Standard initially
- Apply lifecycle policy:
  - Move to Standard-IA after 30 days
  - Move to Glacier after 90 days
  - Delete after 180 days
- Compress artifacts before upload
- Use proper naming/versioning

---

## 2. Secure Private Data Access

### Scenario
Sensitive files in S3 should only be accessed by your application.

### Answer
- Enable Block Public Access
- Use IAM Role (EC2/EKS)
- Apply bucket policy restrictions
- Enable SSE-KMS encryption

---

## 3. Temporary Access to Private Files

### Scenario
Users need temporary access to private files.

### Answer
- Generate Pre-Signed URL with expiry time

---

## 4. Multi-Region Disaster Recovery

### Scenario
You must protect data from region failure.

### Answer
- Enable Cross-Region Replication (CRR)
- Enable versioning (mandatory)
- Store in secondary region

---

## 5. Large File Upload Optimization

### Scenario
Uploading large files (10GB+) fails frequently.

### Answer
- Use Multipart Upload
- Retry failed parts
- Use Transfer Acceleration (optional)

---

## 6. Static Website Hosting

### Scenario
Host a frontend application.

### Answer
- Enable S3 static website hosting
- Use CloudFront for CDN
- Restrict public access to only required files

---

## 7. Log Storage and Analysis

### Scenario
Store and analyze application logs.

### Answer
- Store logs in S3
- Use lifecycle to move to Glacier
- Organize logs with date-based prefixes
- Query using Athena

---

## 8. Prevent Accidental Deletion

### Scenario
Critical data should not be deleted accidentally.

### Answer
- Enable Versioning
- Enable Object Lock (WORM)
- Use MFA Delete

---

## 9. High Request Performance

### Scenario
Application receives very high request traffic.

### Answer
- Avoid hot partitions (use proper key naming)
- Use CloudFront caching
- Leverage S3 scalability

---

## 10. Data Security Compliance

### Scenario
Company requires encryption and auditing.

### Answer
- Enable SSE-KMS encryption
- Enable CloudTrail / access logging
- Restrict IAM access

---

## 11. Sharing Data Between Services

### Scenario
Multiple services need access to S3.

### Answer
- Use IAM roles with least privilege
- Control access via bucket policy

---

## 12. Backup Strategy

### Scenario
Daily database backups need to be stored.

### Answer
- Store backups in S3
- Enable lifecycle policies
- Enable versioning
- Use CRR for disaster recovery

---

## 13. Event-Driven Processing

### Scenario
Trigger processing when file is uploaded.

### Answer
- Configure S3 Event Notifications
- Trigger Lambda / SQS / SNS

---

## 14. Cost Optimization

### Scenario
S3 bill is high.

### Answer
- Use lifecycle policies
- Use Intelligent-Tiering
- Delete unused objects
- Analyze with S3 Storage Lens

---

## 15. Access Issue Debugging

### Scenario
Application cannot access S3.

### Answer
1. Check IAM role permissions
2. Check bucket policy
3. Check object permissions
4. Verify region
5. Check KMS permissions

---

## 16. Data Migration to S3

### Scenario
Migrate large data (TBs) to S3.

### Answer
- Use AWS DataSync or Snowball
- Use multipart upload

---

## 17. Public Content Delivery

### Scenario
Serve public images globally.

### Answer
- Use S3 + CloudFront
- Keep bucket private, expose via CDN

---

## 18. Handling Duplicate Uploads

### Scenario
Same files uploaded multiple times.

### Answer
- Use hashing (MD5) to detect duplicates
- Implement unique object key strategy

---

## 19. Environment Separation

### Scenario
Separate Dev, QA, Prod environments.

### Answer
- Use separate buckets or prefixes
- Apply IAM restrictions per environment

---

## 20. Terraform + S3

### Scenario
Manage S3 using Terraform.

### Answer
- Use aws_s3_bucket resource
- Enable versioning and lifecycle rules
- Store Terraform state in S3 backend
- Enable DynamoDB locking

---

# Final Tip

In interviews, always explain:
- What problem you faced
- Why you used S3 feature
- What impact it created (cost, performance, security)

