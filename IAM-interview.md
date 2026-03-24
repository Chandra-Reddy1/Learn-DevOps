# AWS IAM Interview Questions & Answers (Top 20)

---

## 1. What is IAM?

IAM (Identity and Access Management) is a service used to securely control access to AWS resources.

---

## 2. What are IAM users?

Individual identities created for people or applications to access AWS.

---

## 3. What are IAM roles?

Roles are temporary identities with permissions that can be assumed by users, services, or applications.

---

## 4. What is an IAM policy?

A JSON document that defines permissions (Allow/Deny) for AWS resources.

---

## 5. What are IAM groups?

A collection of users with shared permissions.

---

## 6. Difference between IAM user and role?

- User → permanent credentials  
- Role → temporary credentials  

---

## 7. What is least privilege principle?

Grant only the minimum permissions required to perform a task.

---

## 8. What is AWS managed policy?

Predefined policy created by AWS.

---

## 9. What is customer managed policy?

Custom policy created by user.

---

## 10. What is inline policy?

Policy directly attached to a user/role (not reusable).

---

## 11. What is IAM role assumption?

Temporary access obtained by assuming a role using STS.

---

## 12. What is STS (Security Token Service)?

Provides temporary security credentials.

---

## 13. What is trust policy?

Defines who can assume a role.

---

## 14. What is permission policy?

Defines what actions are allowed or denied.

---

## 15. What is MFA?

Multi-Factor Authentication for extra security.

---

## 16. What is access key?

Used for programmatic access (CLI/SDK).

---

## 17. What is policy evaluation logic?

AWS checks:
- Explicit Deny → highest priority  
- Allow → if no deny  
- Default → deny  

---

## 18. What is IAM condition?

Adds conditions to policies (e.g., IP address, time).

---

## 19. What is resource-based policy?

Policy attached directly to a resource (e.g., S3 bucket policy).

---

## 20. What is cross-account access?

Access resources in another AWS account using roles.

---

# Bonus (High-Impact DevOps Questions)

---

## 21. How does IAM work with EC2?

- Attach IAM role to EC2
- Applications use role instead of access keys

---

## 22. What is IRSA in Kubernetes (EKS)?

IAM Roles for Service Accounts:
- Allows pods to access AWS services securely

---

## 23. How to secure IAM?

- Enable MFA
- Use least privilege
- Rotate access keys
- Avoid root account usage

---

## 24. Difference between role and policy?

- Role → identity  
- Policy → permissions  

---

## 25. What is permission boundary?

Limits maximum permissions an IAM entity can have.

---

## 26. What is session duration in role?

Time limit for temporary credentials.

---

## 27. What is federation in IAM?

Allows external users (e.g., Google/AD) to access AWS.

---

## 28. What is SCP (Service Control Policy)?

Used in AWS Organizations to control permissions across accounts.

---

## 29. What is identity-based vs resource-based policy?

- Identity-based → attached to user/role  
- Resource-based → attached to resource  

---

## 30. What is IAM best practice?

- Use roles instead of users  
- Avoid long-term credentials  
- Apply least privilege  
- Use MFA  

---

# Final Interview Tips

- Always mention:
  - Least privilege
  - Roles over users
  - Temporary credentials (STS)
- Use real examples:
  - EC2 accessing S3 using role
  - Lambda accessing DynamoDB
