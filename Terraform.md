# Top 70 Terraform Interview Questions — DevOps / AWS

> Interview-focused question bank for a 4+ year DevOps Engineer.  
> Each question includes the key points an interviewer expects you to understand.

---

## 1. Terraform Fundamentals

### 1. What is Terraform?
**Key points:** Terraform is an Infrastructure as Code (IaC) tool from HashiCorp that lets you define, provision, and manage infrastructure using declarative configuration files.

### 2. Why do we use Terraform?
**Key points:** To automate infrastructure provisioning, make infrastructure repeatable/version-controlled, reduce manual work, and manage infrastructure consistently across environments.

### 3. What is Infrastructure as Code (IaC)?
**Key points:** IaC means defining infrastructure such as VPCs, EC2, EKS, databases, and IAM in code instead of creating them manually.

### 4. What is the difference between declarative and imperative approaches?
**Key points:** Terraform is declarative: you define the desired end state and Terraform determines the required actions. Imperative tools generally specify the exact sequence of operations.

### 5. What is the Terraform workflow?
**Key points:** The common workflow is `terraform init` → `terraform validate` → `terraform plan` → `terraform apply` → `terraform destroy` when resources need to be removed.

### 6. What are Terraform configuration files?
**Key points:** Terraform normally uses `.tf` files written in HCL (HashiCorp Configuration Language) to define providers, resources, variables, outputs, modules, and other configuration.

### 7. What is HCL?
**Key points:** HCL is HashiCorp Configuration Language, a human-readable configuration language used by Terraform to define infrastructure.

### 8. Terraform vs CloudFormation — what is the difference?
**Key points:** Terraform is multi-cloud and uses providers; CloudFormation is AWS-native. Discuss portability, ecosystem, state handling, and team standards rather than claiming one is universally better.

### 9. Terraform vs Ansible — what is the difference?
**Key points:** Terraform is primarily used for provisioning and managing infrastructure; Ansible is commonly used for configuration management and application/task automation. They can be used together.

### 10. What is the difference between Terraform and scripting?
**Key points:** Terraform maintains a model of desired infrastructure and tracks managed resources through state, while scripts generally execute procedural commands without Terraform's dependency graph and state management.

---

## 2. Providers and Resources

### 11. What is a Terraform provider?
**Key points:** A provider is a plugin that allows Terraform to communicate with an external API such as AWS, Azure, GCP, Kubernetes, GitHub, or Datadog.

### 12. How do you configure the AWS provider?
**Key points:** Define the `aws` provider and specify settings such as region. Credentials should normally come from secure mechanisms such as IAM roles, environment variables, or configured credential providers rather than hardcoding secrets.

### 13. What is a Terraform resource?
**Key points:** A resource represents infrastructure or an object Terraform manages, such as `aws_instance`, `aws_vpc`, `aws_s3_bucket`, or `aws_eks_cluster`.

### 14. What is a data source?
**Key points:** A data source reads existing information from a provider without necessarily creating the object. Example: retrieving an existing VPC, AMI, subnet, or availability zones.

### 15. Resource vs data source?
**Key points:** A resource manages lifecycle—create, update, and destroy. A data source reads existing information for use by other Terraform configuration.

### 16. What is a provider alias?
**Key points:** Provider aliases allow multiple configurations of the same provider, such as deploying resources into multiple AWS regions or accounts.

### 17. What is provider versioning?
**Key points:** Provider versions can be constrained in `required_providers` so that Terraform uses compatible provider versions and upgrades can be controlled.

### 18. What is the Terraform Registry?
**Key points:** It is a repository/catalog for providers and reusable Terraform modules. Public providers and modules can be discovered and consumed from it.

---

## 3. Terraform State

### 19. What is the Terraform state file?
**Key points:** State records Terraform's view of managed infrastructure and maps configuration resources to real-world resource instances. It is commonly stored as `terraform.tfstate`.

### 20. Why is Terraform state important?
**Key points:** Terraform uses state to determine what already exists, detect changes, calculate dependencies, and decide what actions are required during planning and apply.

### 21. What happens if the state file is deleted?
**Key points:** Terraform loses its recorded mapping to managed resources. It may try to create resources again or behave as though resources are unmanaged. Existing infrastructure is not automatically deleted just because the state file disappeared.

### 22. What is remote state?
**Key points:** Remote state stores Terraform state in a shared backend such as an S3 bucket, enabling team collaboration and centralized state management.

### 23. Why should we use remote state?
**Key points:** It provides centralized access, better collaboration, controlled storage, backup/versioning options, and enables state locking when supported by the backend.

### 24. How do you secure a Terraform state file?
**Key points:** Use a secure remote backend, restrict IAM access, enable encryption at rest, enable versioning where appropriate, and avoid exposing state publicly because state can contain sensitive information.

### 25. What is state locking?
**Key points:** State locking prevents multiple Terraform operations from modifying the same state simultaneously, reducing the risk of state corruption or conflicting changes.

### 26. How do you handle Terraform state in a team?
**Key points:** Use a shared remote backend, locking, version control for `.tf` code, controlled CI/CD execution, and clear ownership/workspace/environment practices.

### 27. What is state drift?
**Key points:** Drift occurs when infrastructure changes outside Terraform, causing the real infrastructure to differ from the configuration/state Terraform expects.

### 28. How do you detect and handle drift?
**Key points:** Run `terraform plan` to identify differences, determine whether the manual change should be retained or reverted, then update Terraform configuration/state appropriately.

---

## 4. Terraform Commands

### 29. What does `terraform init` do?
**Key points:** It initializes the working directory, downloads required providers/modules, and configures the backend.

### 30. What does `terraform validate` do?
**Key points:** It checks whether Terraform configuration is syntactically valid and internally consistent. It does not provision infrastructure.

### 31. What does `terraform plan` do?
**Key points:** It compares the desired configuration with Terraform's current knowledge of infrastructure and produces a proposed set of changes.

### 32. What does `terraform apply` do?
**Key points:** It executes the planned changes and provisions, updates, or removes resources according to the configuration.

### 33. What does `terraform destroy` do?
**Key points:** It plans and executes removal of resources managed by the configuration/state. It should be used carefully, especially in production.

### 34. What is `terraform refresh`?
**Key points:** Modern Terraform refreshes state as part of normal planning operations. The older standalone `terraform refresh` workflow is generally discouraged; use `terraform plan`/`apply` and appropriate refresh options when needed.

### 35. What is `terraform fmt`?
**Key points:** It formats Terraform files into standard Terraform style, improving consistency across a team.

### 36. What is `terraform show`?
**Key points:** It displays the current state or details of a saved plan in a human-readable form.

### 37. What is `terraform output`?
**Key points:** It displays values declared in Terraform `output` blocks, such as load balancer DNS names, VPC IDs, or resource identifiers.

### 38. What is `terraform import`?
**Key points:** Import associates an existing infrastructure resource with a Terraform resource address so Terraform can manage it. Import does not automatically generate a complete configuration for the resource.

---

## 5. Variables, Outputs and Locals

### 39. What are Terraform variables?
**Key points:** Input variables parameterize Terraform configurations so the same code can be reused with different values across environments.

### 40. What are variable types in Terraform?
**Key points:** Common types include `string`, `number`, `bool`, `list`, `set`, `map`, `object`, and `tuple`.

### 41. What is a variable default value?
**Key points:** A default provides a fallback value when no other value is supplied. Variables without defaults can require an input value.

### 42. How do you pass variables to Terraform?
**Key points:** Common methods include `-var`, `-var-file`, `.tfvars` files, environment variables such as `TF_VAR_name`, and CI/CD pipeline variables.

### 43. What is a `.tfvars` file?
**Key points:** It contains variable values separately from Terraform resource definitions. Environment-specific files such as `dev.tfvars` and `prod.tfvars` are commonly used.

### 44. What are output variables?
**Key points:** Outputs expose useful values from a Terraform configuration, such as resource IDs, DNS names, IP addresses, or module outputs.

### 45. What are local values?
**Key points:** `locals` create reusable named expressions inside a module, helping avoid repeated calculations or strings and making configuration easier to maintain.

---

## 6. Modules

### 46. What is a Terraform module?
**Key points:** A module is a reusable collection of Terraform configuration. The root module is the current configuration; child modules package reusable infrastructure patterns.

### 47. Why do we use modules?
**Key points:** Modules reduce duplication, standardize infrastructure, improve maintainability, and allow teams to reuse approved infrastructure patterns.

### 48. How do you create a reusable AWS module?
**Key points:** Define resources and variables inside a module, expose important values through outputs, and call the module from environment-specific/root configurations.

### 49. How do you pass variables to a module?
**Key points:** Define input variables inside the child module and pass values through the module block from the calling/root module.

### 50. What are module inputs and outputs?
**Key points:** Inputs customize a module; outputs expose values produced by the module to its caller or other configuration.

### 51. How do you version Terraform modules?
**Key points:** Use Git tags/releases or a module registry and pin module versions in consuming configurations to make changes controlled and reproducible.

---

## 7. Dependencies and Resource Lifecycle

### 52. What is Terraform's dependency graph?
**Key points:** Terraform builds a graph of resource dependencies and uses it to determine the order in which resources can be created, updated, or destroyed.

### 53. What is an implicit dependency?
**Key points:** Terraform automatically detects a dependency when one resource references an attribute of another resource.

### 54. What is an explicit dependency?
**Key points:** `depends_on` explicitly tells Terraform that one resource or module must wait for another even when Terraform cannot infer the dependency from references.

### 55. What is `count`?
**Key points:** `count` creates multiple instances of a resource based on a numeric value and uses numeric indexes such as `resource.example[0]`.

### 56. What is `for_each`?
**Key points:** `for_each` creates multiple resource instances from a map or set and provides stable keys, which is often useful when each instance has a distinct identity.

### 57. `count` vs `for_each`?
**Key points:** Use `count` for simple indexed repetition; use `for_each` when resources are naturally identified by stable keys. Changing list ordering can make `count` less stable.

### 58. What is `lifecycle` in Terraform?
**Key points:** The `lifecycle` block changes resource lifecycle behavior. Common arguments include `create_before_destroy`, `prevent_destroy`, and `ignore_changes`.

### 59. What is `create_before_destroy`?
**Key points:** It tells Terraform to create a replacement resource before destroying the old one when possible, which can reduce downtime.

### 60. What is `prevent_destroy`?
**Key points:** It prevents Terraform from destroying a resource through normal Terraform operations and causes an error if a planned action would destroy it.

---

## 8. Terraform Advanced / Production Scenarios

### 61. How do you manage multiple environments such as dev, QA and prod?
**Key points:** Common approaches include separate root configurations/directories with shared modules, separate state/backends, and environment-specific variable files. Keep production state isolated.

### 62. What are Terraform workspaces?
**Key points:** Workspaces allow multiple state instances for one configuration. They can be useful for simple environment separation, but many teams prefer separate configurations/backends for strongly isolated production environments.

### 63. What is a Terraform backend?
**Key points:** A backend defines where Terraform state is stored and, depending on the backend, how state locking and remote operations are handled.

### 64. How would you store Terraform state in AWS?
**Key points:** A common design is an S3 backend with restricted IAM access, encryption, and versioning; use the locking mechanism supported by the Terraform/backend version and architecture in use.

### 65. How do you handle secrets in Terraform?
**Key points:** Never hardcode secrets in `.tf` files or commit them to Git. Use secret managers, CI/CD secret stores, environment variables, or other secure mechanisms, and remember that sensitive values may still exist in state.

### 66. How do you prevent accidental production changes?
**Key points:** Use separate production state, protected branches, pull-request reviews, CI/CD plan approval, restricted IAM permissions, policy checks, and manual approval before production apply.

### 67. How do you perform a Terraform upgrade safely?
**Key points:** Review Terraform/provider/module compatibility, update version constraints deliberately, test in a lower environment, inspect the plan carefully, and then roll out to production with approval.

### 68. What would you do if `terraform apply` fails halfway?
**Key points:** Do not immediately rerun blindly. Inspect the error and current state, verify what was successfully created/changed, fix the underlying issue, then run `terraform plan` before continuing.

### 69. How do you handle a resource that was manually deleted from AWS?
**Key points:** Run `terraform plan` to understand the difference. If the resource is still declared and should exist, Terraform will generally plan to recreate it; verify dependencies and impact before applying.

### 70. How do you integrate Terraform with CI/CD?
**Key points:** A typical pipeline runs formatting/validation, security or policy checks, `terraform plan`, stores the plan for review, and requires controlled approval before `terraform apply`. State access and cloud credentials must be securely managed.

---

# High-Priority Terraform Questions for Interviews

If you have limited preparation time, focus first on these:

1. What is Terraform and why do we use it?
2. Terraform workflow
3. `terraform init`
4. `terraform plan`
5. `terraform apply`
6. Terraform state
7. Remote state
8. State locking
9. State drift
10. Terraform backend
11. Provider
12. Resource
13. Data source
14. Variables
15. `.tfvars`
16. Outputs
17. Modules
18. `count` vs `for_each`
19. Implicit vs explicit dependencies
20. `depends_on`
21. Lifecycle rules
22. `terraform import`
23. Workspaces
24. Secrets
25. Terraform with CI/CD
26. Terraform + AWS
27. Handling failed `terraform apply`
28. Handling manual infrastructure changes
29. Environment management
30. Terraform upgrade strategy

---

# AWS + Terraform Topics You Should Be Ready to Explain

For an AWS DevOps interview, also practice these real-world scenarios:

```text
Terraform
   |
   +--> VPC
   |     +--> Subnets
   |     +--> Route Tables
   |     +--> Internet Gateway
   |     +--> NAT Gateway
   |
   +--> IAM
   |
   +--> Security Groups
   |
   +--> EC2
   |
   +--> S3
   |
   +--> EKS
   |     +--> Cluster
   |     +--> Node Groups
   |     +--> Add-ons
   |
   +--> Load Balancer
   |
   +--> RDS
   |
   +--> CloudWatch
   |
   +--> ECR
```

## Commands to know without hesitation

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform destroy

terraform show
terraform output
terraform state list
terraform state show
terraform state mv
terraform state rm

terraform import
terraform providers
terraform graph
```

## The core Terraform flow to memorize

```text
Write Terraform Code
        |
        v
terraform init
        |
        v
terraform validate
        |
        v
terraform plan
        |
        v
Review Changes
        |
        v
terraform apply
        |
        v
Infrastructure Created/Updated
        |
        v
Terraform State Updated
```

> **Interview tip:** For a 4+ year DevOps role, interviewers usually go beyond definitions. Be prepared to explain **state management, remote backends, modules, dependencies, `count` vs `for_each`, drift, import, secrets, CI/CD integration, and troubleshooting failed applies** with a real AWS example.
