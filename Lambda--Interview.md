# AWS Lambda Interview Questions & Answers (Top 20)

---

## 1. What is AWS Lambda?

AWS Lambda is a serverless compute service that runs code without provisioning or managing servers. It executes code in response to events.

---

## 2. What is serverless?

Serverless means:
- No server management
- Automatic scaling
- Pay only for execution time

---

## 3. What are Lambda triggers?

Events that invoke Lambda:
- S3
- API Gateway
- DynamoDB
- SNS / SQS
- CloudWatch Events

---

## 4. What is the maximum execution time of Lambda?

15 minutes (900 seconds)

---

## 5. What languages are supported?

- Python
- Node.js
- Java
- Go
- .NET
- Ruby

---

## 6. What is cold start?

Delay when Lambda is invoked after being idle:
- Container needs initialization
- Impacts performance

---

## 7. How to reduce cold start?

- Use Provisioned Concurrency
- Keep function warm
- Reduce package size

---

## 8. What is Lambda concurrency?

Number of instances running simultaneously.

Types:
- Reserved concurrency
- Provisioned concurrency

---

## 9. What is timeout in Lambda?

Maximum time Lambda can run before termination.

---

## 10. What is memory allocation?

- Configurable (128 MB to 10 GB)
- CPU is proportional to memory

---

## 11. What is IAM role in Lambda?

Defines permissions for Lambda to access AWS services.

---

## 12. What is Lambda Layers?

Reusable code or dependencies shared across multiple Lambda functions.

---

## 13. What is environment variable?

Key-value pairs used to configure Lambda without changing code.

---

## 14. What is event-driven architecture?

System where events trigger actions (e.g., S3 upload → Lambda trigger)

---

## 15. How does Lambda scale?

Automatically scales based on incoming requests/events.

---

## 16. What is Lambda pricing?

Charged based on:
- Number of requests
- Execution duration
- Memory used

---

## 17. What is dead-letter queue (DLQ)?

Stores failed events:
- SQS or SNS used

---

## 18. How to monitor Lambda?

- CloudWatch Logs
- CloudWatch Metrics
- X-Ray (for tracing)

---

## 19. What is VPC in Lambda?

Lambda can run inside a VPC to access private resources like RDS.

---

## 20. What is retry behavior in Lambda?

- Asynchronous → retries automatically
- SQS → retries until success or max attempts

---

# Bonus (Important for DevOps Interviews)

---

## 21. Difference between Lambda and EC2?

- Lambda → serverless, auto-scale
- EC2 → manual server management

---

## 22. When NOT to use Lambda?

- Long-running tasks (>15 min)
- Heavy CPU workloads
- Stateful applications

---

## 23. How to deploy Lambda?

- AWS CLI
- Terraform
- CloudFormation
- CI/CD pipelines

---

## 24. How to secure Lambda?

- IAM roles (least privilege)
- VPC configuration
- Encrypt environment variables

---

## 25. What is Lambda + S3 use case?

- Upload file to S3 → trigger Lambda → process file

---

## 26. What is Lambda + API Gateway?

- Build REST APIs without servers

---

## 27. What is throttling in Lambda?

Limiting concurrent executions to prevent overload.

---

## 28. What is idempotency in Lambda?

Ensures same request doesn’t produce duplicate results.

---

## 29. What is packaging limit?

- Direct upload: 50 MB
- With S3: 250 MB (unzipped)

---

## 30. What is Step Functions?

Used to orchestrate multiple Lambda functions into workflows.

---

# AWS Lambda – Top 3 DevOps Use Cases (Interview Ready)

---

## 1. S3 → Lambda (File Processing Automation)

### Scenario
Users upload files (logs/images/data) to an S3 bucket, and the system should automatically process them.

### Solution
- Configure S3 Event Notification
- Trigger Lambda on object upload
- Lambda processes the file:
  - Validation
  - Transformation
  - Image resizing (if applicable)
- Store processed output back in S3 or send to a database

### Key Points
- Fully event-driven architecture
- Eliminates manual processing
- Scales automatically with uploads

### One-Line Explanation (Interview)
"We used Lambda triggered by S3 uploads to automatically process files, reducing manual effort and enabling scalable event-driven workflows."

---

## 2. API Gateway → Lambda (Serverless Backend)

### Scenario
Build a backend API without managing servers.

### Solution
- Use API Gateway as entry point
- Route HTTP requests to Lambda
- Lambda executes business logic
- Return response via API Gateway

### Key Points
- No server management
- Auto scaling based on requests
- Pay-per-use cost model

### One-Line Explanation (Interview)
"We replaced traditional backend servers with Lambda behind API Gateway to reduce infrastructure overhead and enable automatic scaling."

---

## 3. Scheduled Jobs (Cron Replacement)

### Scenario
Run periodic tasks (daily/weekly jobs) without maintaining servers.

### Solution
- Use CloudWatch Events / EventBridge scheduler
- Configure time-based trigger (cron expression)
- Trigger Lambda at scheduled intervals
- Lambda executes task (cleanup, backup, reporting)

### Key Points
- No always-running servers
- More reliable than cron jobs on EC2
- Easy to manage and scale

### One-Line Explanation (Interview)
"We replaced cron jobs with scheduled Lambda functions using EventBridge, eliminating the need for dedicated servers."

---

# Final Notes

## What Interviewer Expects
- Clear problem → solution explanation
- Why Lambda was chosen
- Real impact (automation, cost, scalability)

## Common Mistakes
- Giving only definitions without use case
- Not mentioning trigger (S3/API Gateway/EventBridge)
- No real-world context

