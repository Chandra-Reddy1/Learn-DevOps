# Terraform Interview Questions & Answers (Top 20)

---

## 1. What is Terraform?

Terraform is an Infrastructure as Code (IaC) tool used to provision and manage infrastructure using declarative configuration files.

---

## 2. What is Infrastructure as Code (IaC)?

Managing infrastructure using code instead of manual setup.

---

## 3. What is a provider in Terraform?

A plugin that allows Terraform to interact with APIs (e.g., AWS, Azure).

---

## 4. What is a resource?

A component of infrastructure (e.g., EC2, S3 bucket).

---

## 5. What is a variable?

Used to parameterize Terraform configuration.

---

## 6. What is output?

Used to display values after execution.

---

## 7. What is state file (terraform.tfstate)?

Stores the current state of infrastructure managed by Terraform.

---

## 8. Why is state file important?

- Tracks resources  
- Maps real infra to config  
- Required for updates  

---

## 9. What is terraform init?

Initializes Terraform:
- Downloads providers
- Sets up backend

---

## 10. What is terraform plan?

Shows execution plan (what will be created/changed).

---

## 11. What is terraform apply?

Applies changes to create/update infrastructure.

---

## 12. What is terraform destroy?

Deletes all managed resources.

---

## 13. What is backend in Terraform?

Defines where state file is stored.

Types:
- Local
- Remote (S3, Terraform Cloud)

---

## 14. Why use remote backend?

- Team collaboration
- State locking
- Security

---

## 15. What is state locking?

Prevents multiple users from modifying state simultaneously.

---

## 16. What is module in Terraform?

Reusable configuration block.

---

## 17. What is data source?

Fetches existing resources (read-only).

---

## 18. What is for_each?

Creates multiple resources dynamically using map/set.

---

## 19. What is count?

Creates multiple instances using index.

---

## 20. Difference between count and for_each?

- count → index-based  
- for_each → key-based (better for dynamic resources)  

---

# Bonus (High-Impact DevOps Questions)

---

## 21. How do you manage Terraform state in real projects?

- Store in S3 backend
- Use DynamoDB for locking

---

## 22. How to secure Terraform?

- Use IAM roles
- Encrypt state file (S3 + KMS)
- Avoid hardcoding secrets

---

## 23. What is terraform workspace?

Used to manage multiple environments (dev, prod).

---

## 24. What is terraform import?

Import existing infrastructure into Terraform state.

---

## 25. What is terraform refresh?

Updates state file with real infrastructure.

---

## 26. What is dependency in Terraform?

Managed automatically via references.

---

## 27. What is provisioner?

Executes scripts (local-exec, remote-exec).

---

## 28. What is lifecycle block?

Controls resource behavior:
- prevent_destroy
- ignore_changes

---

## 29. How Terraform fits in CI/CD?

- Code commit → pipeline runs plan/apply → infra updated

---

## 30. What are best practices?

- Use modules  
- Use remote backend  
- Follow least privilege  
- Separate environments  

---
# Terraform Tricky Interview Questions (High-Impact)

---

## 1. count vs for_each

### Question
When should you use count vs for_each?

### Answer
- count → for identical resources (index-based)
- for_each → for unique resources (key-based)

### Trap
Changing list order:
- count → resources recreated ❌
- for_each → stable mapping ✅

---

## 2. Removing element in count

### Question
What happens if count reduces from 3 to 2?

### Answer
Terraform deletes the last indexed resource.

### Trap
Index shift can delete wrong resource.

---

## 3. Changing key in for_each

### Question
What happens if key changes?

### Answer
- Old resource destroyed
- New resource created

Reason: key = identity

---

## 4. What is Terraform state really?

### Question
What does state file store?

### Answer
- Resource mapping (config ↔ real infra)
- Metadata
- Dependency tracking

---

## 5. What happens if state file is lost?

### Question
State file deleted?

### Answer
- Terraform loses tracking
- May recreate resources
- Risk of duplicates

Fix:
- terraform import

---

## 6. State locking

### Question
Why is state locking needed?

### Answer
Prevents concurrent changes → avoids corruption

---

## 7. Concurrent terraform apply

### Question
Two users run apply simultaneously?

### Answer
- Without locking → corruption risk
- With locking → safe

---

## 8. Terraform null behavior

### Question
Does Terraform send null to provider?

### Answer
No — Terraform ignores the argument completely.

---

## 9. variable vs local

### Question
Difference between variable and local?

### Answer
- variable → external input
- local → internal computation

---

## 10. Implicit dependency

### Question
How Terraform handles dependency automatically?

### Answer
Using resource references:
vpc_id = aws_vpc.main.id

---

## 11. Explicit dependency

### Question
When to use depends_on?

### Answer
When dependency is not obvious to Terraform

---

## 12. Manual deletion outside Terraform

### Question
Resource deleted manually?

### Answer
Terraform recreates it on next apply

---

## 13. terraform taint

### Question
What does taint do?

### Answer
Marks resource for recreation

---

## 14. terraform import limitation

### Question
Does import generate config?

### Answer
No — only updates state

---

## 15. lifecycle ignore_changes

### Question
Why use ignore_changes?

### Answer
To ignore updates on specific attributes

---

## 16. Backend change

### Question
What happens when backend changes?

### Answer
Need to migrate state:
terraform init -migrate-state

---

## 17. Drift in Terraform

### Question
What is drift?

### Answer
Difference between actual infra and state

---

## 18. Drift detection

### Question
How to detect drift?

### Answer
terraform plan

---

## 19. Module design issue

### Question
What if module is not reusable?

### Answer
- Use variables
- Avoid hardcoding
- Make flexible modules

---

## 20. Biggest Terraform mistakes

### Answer
- No remote backend
- No state locking
- Hardcoded values

---

# Most Important Concepts

- count vs for_each  
- state file importance  
- null behavior  

---

# Final Tip

Focus on:
- Behavior (what happens internally)
- Not just syntax

