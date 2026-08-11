# Terraform Scenario-Based Troubleshooting --- Interview + Hands-On Lab

## Goal

This guide is designed for **production-style Terraform troubleshooting
interviews**, not definition-based questions.

The core interview pattern is:

> **Observe → isolate → verify → identify root cause → fix safely →
> validate → prevent recurrence**

Do not jump straight to `terraform apply -auto-approve`,
`terraform destroy`, deleting `.terraform`, or `-lock=false`. Those are
often signs that the engineer is treating symptoms instead of diagnosing
the failure.

------------------------------------------------------------------------

# 1. Terraform Troubleshooting Framework

When Terraform fails, classify the failure first:

1.  **Configuration problem** --- HCL, syntax, types, expressions.
2.  **Provider problem** --- provider version, authentication, region,
    plugin.
3.  **Plan problem** --- dependency graph, unknown values, replacement.
4.  **State problem** --- stale state, drift, lock, corruption, wrong
    backend.
5.  **AWS/API problem** --- permissions, quota, invalid resource
    configuration.
6.  **Networking problem** --- route tables, SG, NACL, DNS,
    public/private path.
7.  **Lifecycle problem** --- replacement, dependency,
    `create_before_destroy`.
8.  **CI/CD problem** --- concurrent runs, credentials,
    workspace/backend selection.
9.  **Security problem** --- secrets in code/state, excessive IAM
    permissions.

A strong answer should name the **failure layer** before proposing
commands.

------------------------------------------------------------------------

# 2. Scenario Questions and Answers

## Scenario 1 --- `terraform init` fails

### Question

`terraform init` is failing immediately. How do you troubleshoot?

### Answer

I would first inspect the exact error instead of assuming the
configuration is wrong.

``` bash
terraform version
terraform init
```

Then check:

``` bash
terraform providers
```

I would verify:

-   Terraform version.
-   Required provider versions.
-   Provider registry/network access.
-   Backend configuration.
-   Credentials if the backend requires cloud authentication.
-   Proxy/firewall restrictions.
-   `.terraform` and lock-file consistency.

If provider installation is corrupted, I would normally remove
`.terraform/` and rerun `terraform init`, but I would **not casually
delete `.terraform.lock.hcl`** because it pins provider
selections/checksums.

### Interview answer

> "`init` is a bootstrap failure, so I first determine whether the
> failure is provider installation, backend initialization, or
> authentication. I check the exact error, Terraform/provider versions,
> backend configuration and network access. I only clean `.terraform`
> when there is evidence of a local plugin/cache problem."

------------------------------------------------------------------------

## Scenario 2 --- Provider authentication fails

### Question

Terraform says AWS credentials are invalid. What do you check?

### Answer

First verify the identity outside Terraform:

``` bash
aws sts get-caller-identity
```

Then check:

``` bash
echo $AWS_PROFILE
aws configure list
```

Depending on the environment, I check:

-   Environment variables.
-   AWS profile.
-   IAM role attached to the CI runner/EC2 instance.
-   OIDC role assumption in CI/CD.
-   Credential expiration.
-   Account and region.
-   Whether Terraform is running under a different user/profile than the
    AWS CLI.

### Important

Do not put access keys directly into Terraform files.

------------------------------------------------------------------------

## Scenario 3 --- `terraform plan` fails with an authentication error

### Question

Why can `terraform plan` fail even though `terraform init` succeeded?

### Answer

`init` primarily initializes Terraform and providers. `plan` actually
evaluates configuration, data sources, provider operations and state
interactions.

So valid provider installation does **not** prove that AWS API access
works.

I would run:

``` bash
aws sts get-caller-identity
terraform plan
```

Then determine which AWS API call is failing.

------------------------------------------------------------------------

## Scenario 4 --- `terraform validate` fails

### Question

What is the difference between `validate` and `plan`?

### Answer

``` bash
terraform fmt -check
terraform validate
terraform plan
```

`validate` checks whether the configuration is structurally valid and
can be evaluated with the available provider schemas.

`plan` goes further: it evaluates the actual infrastructure changes,
data sources, state and provider/API interactions.

So:

-   `validate` = configuration correctness.
-   `plan` = proposed infrastructure change.

------------------------------------------------------------------------

## Scenario 5 --- Terraform creates half the infrastructure and then fails

### Question

`terraform apply` created 8 resources and failed on resource 9. What do
you do?

### Answer

I **do not immediately destroy everything**.

First:

``` bash
terraform plan
terraform state list
```

Then inspect the failed resource and AWS console/API.

Terraform records successfully created resources in state. The next plan
should normally identify what remains to be created or corrected.

I would:

1.  Read the exact failure.
2.  Confirm which resources were created.
3.  Check state.
4.  Fix the root cause.
5.  Run `terraform plan`.
6.  Apply again.

### Interview answer

> "A partial apply is not automatically a disaster. Terraform can
> continue from the recorded state. I inspect the failed resource,
> verify state consistency, fix the root cause and run a new plan before
> applying again."

------------------------------------------------------------------------

## Scenario 6 --- Terraform says resource exists, but AWS says it does not

### Question

Terraform believes an EC2 instance exists, but the instance was manually
deleted. What do you do?

### Answer

This is usually **state drift**.

Run:

``` bash
terraform plan
```

Terraform should detect that the remote object is missing and propose
recreation if the resource remains configured.

If necessary:

``` bash
terraform refresh
```

However, modern Terraform workflows generally rely on planning/refresh
behavior rather than using `refresh` as a routine standalone recovery
mechanism.

I would not manually edit `terraform.tfstate`.

------------------------------------------------------------------------

## Scenario 7 --- Someone changed infrastructure manually

### Question

An engineer changed an AWS security group manually. Terraform now shows
a difference. What do you do?

### Answer

This is drift.

Run:

``` bash
terraform plan
```

Then decide whether the manual change is:

-   Intended → update Terraform code and make Terraform the source of
    truth.
-   Unintended → allow Terraform to restore the declared configuration.
-   Emergency production change → document it, then reconcile
    code/state.

The important principle is:

> Terraform configuration should eventually represent the desired
> production state.

------------------------------------------------------------------------

## Scenario 8 --- Terraform wants to recreate a resource unexpectedly

### Question

A tiny configuration change causes Terraform to destroy and recreate an
important resource. How do you troubleshoot?

### Answer

Use:

``` bash
terraform plan
```

Look for:

``` text
-/+ destroy and then create replacement
```

Then identify which attribute is forcing replacement.

I would check:

-   Provider documentation.
-   Whether the attribute is immutable.
-   Resource lifecycle rules.
-   Renames of resource addresses.
-   Module changes.
-   `count`/`for_each` changes.
-   Provider version changes.

If the resource was renamed in code but is actually the same
infrastructure, consider a `moved` block rather than recreation.

------------------------------------------------------------------------

## Scenario 9 --- Resource was renamed and Terraform wants destroy/create

### Question

You renamed:

``` hcl
aws_instance.web
```

to:

``` hcl
aws_instance.application
```

Terraform wants to destroy and recreate it. Why?

### Answer

The Terraform resource address changed.

Terraform sees:

``` text
aws_instance.web
```

and:

``` text
aws_instance.application
```

as different addresses.

Use a `moved` block:

``` hcl
moved {
  from = aws_instance.web
  to   = aws_instance.application
}
```

Then:

``` bash
terraform plan
```

The plan should recognize that the existing object is being moved in
Terraform's configuration/state model rather than recreated.

------------------------------------------------------------------------

## Scenario 10 --- Terraform state is locked

### Question

You receive a state lock error during deployment. What do you do?

### Answer

First determine whether another Terraform operation is genuinely
running.

Check:

-   CI/CD pipelines.
-   Other engineers.
-   Remote Terraform runs.
-   Previous crashed jobs.

Do **not** immediately use:

``` bash
terraform force-unlock
```

If I confirm the lock is stale and belongs to my failed operation, I can
use the lock ID from the error:

``` bash
terraform force-unlock <LOCK_ID>
```

Terraform state locking exists to prevent concurrent state writers.
Disabling locking with `-lock=false` is generally unsafe.

------------------------------------------------------------------------

## Scenario 11 --- Two CI pipelines run Terraform simultaneously

### Question

Two GitHub Actions jobs execute `terraform apply` against the same
environment. What can happen?

### Answer

Both jobs may attempt to modify the same state.

With a locking backend, one should obtain the lock and the other should
wait/fail according to configuration.

The correct solution is **pipeline concurrency control**, not disabling
locking.

For example, use CI concurrency so only one production Terraform apply
runs at a time.

------------------------------------------------------------------------

## Scenario 12 --- Terraform state file is missing

### Question

The local `terraform.tfstate` file disappeared. What do you do?

### Answer

First determine whether the project should use local or remote state.

For production, I prefer remote state with appropriate
locking/versioning/recovery mechanisms.

If remote state is configured:

``` bash
terraform init
terraform state list
```

If the state is genuinely lost, recovery depends on backend
backups/versioning and the infrastructure.

I would **not run `terraform apply` blindly** because Terraform may
believe resources do not exist and attempt to recreate them.

------------------------------------------------------------------------

## Scenario 13 --- Terraform state is in the wrong AWS account

### Question

Terraform plan shows infrastructure from another environment/account.
What do you investigate?

### Answer

I verify identity:

``` bash
aws sts get-caller-identity
```

Then inspect:

-   AWS profile.
-   Assumed role.
-   Provider `alias`.
-   Backend configuration.
-   Workspace.
-   CI/CD environment variables.
-   Region/account variables.

A common production mistake is correctly configured Terraform code
running with the **wrong credentials**.

------------------------------------------------------------------------

## Scenario 14 --- Wrong region

### Question

Terraform says a resource cannot be found, but you can see it in AWS
Console.

### Answer

The first thing I check is region.

``` bash
aws configure get region
aws sts get-caller-identity
```

Then verify the provider:

``` hcl
provider "aws" {
  region = var.aws_region
}
```

Also verify that the console is viewing the same region/account.

------------------------------------------------------------------------

## Scenario 15 --- `data` source cannot find an AMI

### Question

Your EC2 data source returns no AMI. What do you check?

### Answer

I check:

-   Region.
-   AMI owner.
-   AMI architecture.
-   Name filter.
-   AMI availability.
-   Whether the AMI is public/private.
-   Account permissions.

For example:

``` bash
aws ec2 describe-images --region ap-south-1
```

I avoid hardcoding an AMI ID unless there is a strong reason.

------------------------------------------------------------------------

## Scenario 16 --- Terraform says "Invalid AMI ID"

### Question

`aws_instance` fails with an invalid AMI ID. How do you troubleshoot?

### Answer

Check:

``` bash
aws ec2 describe-images \
  --image-ids <ami-id> \
  --region <region>
```

Then verify:

-   Correct account.
-   Correct region.
-   AMI exists.
-   AMI architecture matches instance type.
-   AMI is available.

An AMI ID is region-specific.

------------------------------------------------------------------------

## Scenario 17 --- EC2 is running but application is unreachable

### Question

Terraform completed successfully, EC2 is running, but the application
cannot be reached.

### Answer

Terraform success proves the **resource API calls succeeded**. It does
not prove application connectivity.

I troubleshoot:

1.  Instance state.
2.  Public/private IP.
3.  Route table.
4.  Internet Gateway/NAT.
5.  Security Group.
6.  NACL.
7.  OS firewall.
8.  Application process.
9.  Listening port.
10. DNS.

For example:

``` bash
ss -lntp
curl localhost:80
```

Then test externally.

------------------------------------------------------------------------

## Scenario 18 --- ALB exists but returns 503

### Question

Terraform successfully created an ALB, but users receive 503.

### Answer

I separate infrastructure creation from application health.

Check:

-   ALB target group.
-   Target registration.
-   Target health.
-   Health-check path.
-   Health-check port.
-   Security group from ALB → target.
-   Application listening port.
-   Route table/network path.

AWS CLI example:

``` bash
aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn>
```

If targets are unhealthy, Terraform itself may be completely healthy
while the application path is broken.

------------------------------------------------------------------------

## Scenario 19 --- ALB target is unhealthy

### Question

What is your troubleshooting order?

### Answer

I check:

``` text
ALB
 ↓
Target Group
 ↓
Target IP/Instance
 ↓
Target Port
 ↓
Application
 ↓
Health-check path
```

For example, if the target group checks port `8080` but Nginx listens on
`80`, the target will be unhealthy.

------------------------------------------------------------------------

## Scenario 20 --- Security group looks correct but traffic still fails

### Question

The SG allows port 80. Why can traffic still fail?

### Answer

Because SG is only one part of the network path.

Check:

``` text
Client
 ↓
DNS
 ↓
ALB/NLB
 ↓
Route
 ↓
Subnet
 ↓
Security Group
 ↓
Target
 ↓
OS firewall
 ↓
Application
```

For private instances also check NAT or internal routing depending on
the traffic direction.

------------------------------------------------------------------------

## Scenario 21 --- Private EC2 cannot download packages

### Question

A private EC2 instance cannot run `yum update`. What do you check?

### Answer

If it needs internet egress, check:

``` text
Private subnet
   ↓
Route table
   ↓
NAT Gateway
   ↓
Public subnet route
   ↓
Internet Gateway
   ↓
Internet
```

Then check:

-   NAT Gateway exists.
-   NAT is in a public subnet.
-   Public subnet routes to IGW.
-   Private route table routes `0.0.0.0/0` to NAT.
-   Security groups.
-   NACL.
-   DNS.

------------------------------------------------------------------------

## Scenario 22 --- Terraform dependency is wrong

### Question

Terraform creates resource B before resource A even though B requires A.
What do you inspect?

### Answer

Terraform builds a dependency graph.

Prefer an implicit dependency:

``` hcl
subnet_id = aws_subnet.private.id
```

rather than manually forcing dependencies everywhere.

If the dependency is real but not represented by an attribute reference:

``` hcl
depends_on = [aws_iam_role_policy.example]
```

Use `depends_on` only when necessary. Excessive explicit dependencies
make the graph unnecessarily rigid.

------------------------------------------------------------------------

## Scenario 23 --- Terraform reports a dependency cycle

### Question

Terraform says there is a dependency cycle. What do you do?

### Answer

I inspect the dependency graph and resource references.

Typical causes:

``` text
A depends on B
B depends on A
```

I look for:

-   Mutual references.
-   Incorrect `depends_on`.
-   Module-level circular dependencies.
-   Outputs feeding inputs in a loop.

I remove the unnecessary dependency instead of trying to force Terraform
through the cycle.

------------------------------------------------------------------------

## Scenario 24 --- `count` caused the wrong resource to be replaced

### Question

You removed the first item from a list and Terraform wants to replace
several resources.

### Answer

This is a common `count` problem because resources use numeric indexes:

``` text
resource[0]
resource[1]
resource[2]
```

If the list changes positionally, addresses shift.

For identity-based resources, prefer:

``` hcl
for_each = toset(...)
```

or a map with stable keys.

------------------------------------------------------------------------

## Scenario 25 --- `for_each` key changed

### Question

Terraform wants to destroy and recreate resources after a map key
changes.

### Answer

The resource address changed.

For example:

``` text
aws_instance.app["web"]
```

is different from:

``` text
aws_instance.app["frontend"]
```

If the underlying object is the same, use a `moved` block to preserve
identity.

------------------------------------------------------------------------

## Scenario 26 --- Terraform plan contains many "known after apply" values

### Question

Is that automatically a problem?

### Answer

No.

Terraform cannot always know values until resource creation occurs.

For example:

``` text
id = (known after apply)
```

is normal for generated AWS IDs.

The important question is whether an unknown value creates an invalid
dependency or causes an unexpected replacement.

------------------------------------------------------------------------

## Scenario 27 --- Terraform keeps changing the same resource every plan

### Question

Every `terraform plan` shows a change even though nothing was
intentionally changed.

### Answer

This is a classic persistent diff.

Investigate:

-   Provider behavior.
-   API defaults.
-   Computed attributes.
-   Normalization.
-   External automation.
-   Manual changes.
-   Incorrect data types.
-   `ignore_changes`.

Do **not** blindly add:

``` hcl
lifecycle {
  ignore_changes = all
}
```

That can hide real drift.

Use `ignore_changes` only when the field is intentionally managed
elsewhere.

------------------------------------------------------------------------

## Scenario 28 --- Terraform says a resource must be replaced

### Question

How do you safely handle an important resource that requires
replacement?

### Answer

First understand why replacement is required.

Then evaluate:

-   Downtime.
-   Data loss.
-   Dependency impact.
-   Backup/recovery.
-   `create_before_destroy`.
-   Blue/green strategy.
-   Maintenance window.

For replaceable infrastructure:

``` hcl
lifecycle {
  create_before_destroy = true
}
```

But this is not universally safe. Some resources have uniqueness
constraints or dependencies that make parallel creation impossible.

------------------------------------------------------------------------

## Scenario 29 --- `create_before_destroy` fails

### Question

You enabled `create_before_destroy`, but Terraform cannot create the new
resource.

### Answer

Check whether the new resource can coexist with the old one.

Typical blockers:

-   Unique name.
-   Fixed IP.
-   Limited quota.
-   AWS resource constraints.
-   Dependency relationships.
-   Port/address conflicts.

`create_before_destroy` is a lifecycle strategy, not a magic
zero-downtime guarantee.

------------------------------------------------------------------------

## Scenario 30 --- `terraform destroy` fails

### Question

Terraform cannot destroy a resource.

### Answer

I identify the exact AWS dependency blocking deletion.

Common causes:

-   ENIs.
-   Load balancers.
-   Security group dependencies.
-   Route table associations.
-   NAT Gateway.
-   VPC endpoints.
-   S3 bucket contents.
-   Attached volumes.
-   IAM dependencies.

I inspect AWS dependencies first, then Terraform state/configuration.

I do not manually delete random resources unless I understand how that
will affect state.

------------------------------------------------------------------------

## Scenario 31 --- S3 bucket cannot be destroyed

### Question

Why does Terraform fail to destroy an S3 bucket?

### Answer

A non-empty bucket may prevent deletion.

For disposable lab environments, you can use:

``` hcl
force_destroy = true
```

But this is dangerous for production because it allows Terraform to
delete objects as part of bucket destruction.

------------------------------------------------------------------------

## Scenario 32 --- Resource was manually created and Terraform needs to manage it

### Question

How do you bring an existing AWS resource under Terraform management?

### Answer

Write the Terraform resource configuration first, then import it.

Modern Terraform supports import blocks, for example:

``` hcl
import {
  to = aws_subnet.example
  id = "subnet-xxxxxxxx"
}
```

Then:

``` bash
terraform plan
```

The goal is not merely to import the ID. The configuration must
accurately represent the real resource.

------------------------------------------------------------------------

## Scenario 33 --- Import succeeded but plan shows many changes

### Question

Why?

### Answer

Import places the existing object into Terraform's management/state
model. It does not automatically guarantee that the configuration
exactly matches every remote attribute.

I inspect:

``` bash
terraform plan
```

Then reconcile:

``` text
Remote infrastructure
        ↕
Terraform configuration
        ↕
Terraform state
```

The final goal is a clean plan.

------------------------------------------------------------------------

## Scenario 34 --- State has a resource but Terraform code does not

### Question

What do you do?

### Answer

First determine whether the resource should continue to exist.

If it should not exist:

``` bash
terraform plan
```

may propose destruction depending on configuration.

If it should exist but should no longer be managed by this state:

``` bash
terraform state rm <address>
```

`state rm` does **not** delete the AWS resource. It only removes
Terraform's management reference.

This is why it must be used carefully.

------------------------------------------------------------------------

## Scenario 35 --- Resource exists in AWS but is missing from state

### Question

What happens?

### Answer

Terraform may attempt to create another resource because it has no state
association.

Before applying, I verify whether the resource should be imported.

This is one reason state loss is dangerous.

------------------------------------------------------------------------

## Scenario 36 --- `terraform state list` shows unexpected resources

### Question

What does that tell you?

### Answer

The current state contains resources managed by that state.

I verify:

``` bash
terraform workspace show
terraform state list
terraform state show <resource>
```

Then check the backend and environment.

A wrong workspace/backend can make a perfectly valid state look "wrong."

------------------------------------------------------------------------

## Scenario 37 --- Wrong workspace

### Question

You intended to deploy production but Terraform is using staging.

### Answer

Check:

``` bash
terraform workspace show
terraform workspace list
```

Then inspect variables/backend configuration.

But I would not rely on workspaces alone as the security boundary
between production and non-production. Separate state/backend/account
structures are often safer.

------------------------------------------------------------------------

## Scenario 38 --- Backend configuration changed

### Question

The backend configuration was changed from one bucket/key to another.
What do you do?

### Answer

Run:

``` bash
terraform init
```

Terraform may ask whether state should be migrated.

I verify the old and new backend locations before approving migration.

I do not blindly answer "yes" when moving production state.

------------------------------------------------------------------------

## Scenario 39 --- Remote state is unavailable

### Question

Terraform cannot access the remote backend.

### Answer

I check:

-   Backend credentials.
-   Bucket/storage availability.
-   Network access.
-   IAM permissions.
-   Locking mechanism.
-   Backend configuration.
-   Region.
-   Key/path.

Then confirm whether the problem affects one engineer or the whole team.

------------------------------------------------------------------------

## Scenario 40 --- State lock is stuck after a crashed pipeline

### Question

What is your safe procedure?

### Answer

1.  Confirm no Terraform run is active.
2.  Identify the lock ID.
3.  Confirm it belongs to the failed operation.
4.  Force-unlock only if appropriate.
5.  Run `terraform plan`.
6.  Review the plan carefully.

Never force-unlock because "the pipeline is stuck" without verifying
ownership.

------------------------------------------------------------------------

## Scenario 41 --- Terraform provider version upgrade causes changes

### Question

After upgrading the AWS provider, Terraform wants many changes. What do
you do?

### Answer

I do not apply immediately.

I:

``` bash
terraform version
terraform providers
terraform plan
```

Then inspect provider changelog/release notes and resource behavior.

Provider upgrades can change defaults, schemas, validation and resource
behavior.

For production, pin/test provider versions and upgrade deliberately.

------------------------------------------------------------------------

## Scenario 42 --- Terraform code works locally but fails in CI/CD

### Question

What do you compare?

### Answer

I compare:

-   Terraform version.
-   Provider lock file.
-   AWS credentials/role.
-   AWS account.
-   Region.
-   Environment variables.
-   Workspace.
-   Backend.
-   Working directory.
-   Variable files.
-   Secrets.
-   Network/proxy.
-   CI runner permissions.

A classic mistake is assuming:

> "It works on my machine, therefore Terraform is correct."

The CI execution context is part of the system.

------------------------------------------------------------------------

## Scenario 43 --- CI pipeline uses wrong AWS account

### Question

How do you detect it quickly?

### Answer

Add a non-sensitive identity check before Terraform:

``` bash
aws sts get-caller-identity
```

Then compare the returned account with the expected environment.

I also ensure the Terraform provider assumes the intended role
explicitly when appropriate.

------------------------------------------------------------------------

## Scenario 44 --- Terraform plan is correct but apply fails

### Question

Why can this happen?

### Answer

Plan is not a guarantee that apply will succeed.

Between plan and apply:

-   Resources can change.
-   Quotas can be exhausted.
-   Permissions can change.
-   External resources can disappear.
-   AWS APIs can return errors.
-   Concurrent changes can occur.

Therefore I treat `plan` as a proposal, not a transaction guarantee.

------------------------------------------------------------------------

## Scenario 45 --- Apply times out

### Question

Terraform says the resource creation timed out. What do you do?

### Answer

I check AWS first.

``` bash
aws <service> describe-...
```

The Terraform operation may have timed out while AWS continued or
completed the resource operation.

Then:

``` bash
terraform plan
```

If the resource exists and is correctly represented in state, I avoid
creating duplicates.

If state is inconsistent, I investigate/import/reconcile before
proceeding.

------------------------------------------------------------------------

## Scenario 46 --- Terraform reports an AWS quota error

### Question

How do you troubleshoot?

### Answer

This is an AWS capacity/quota problem, not primarily a Terraform syntax
problem.

I:

1.  Identify the exact quota.
2.  Check current usage.
3.  Check whether the resource can be reduced.
4.  Request a quota increase if appropriate.
5.  Re-run plan/apply after correcting capacity.

------------------------------------------------------------------------

## Scenario 47 --- IAM permission denied

### Question

Terraform gets `AccessDenied`. What do you check?

### Answer

I identify:

``` text
Caller identity
+
API operation
+
Resource ARN
+
IAM policy
+
SCP
+
Permission boundary
+
Session policy
```

I verify the actual assumed role:

``` bash
aws sts get-caller-identity
```

Then identify the denied AWS API action.

Do not simply attach `AdministratorAccess` to make the error disappear.

------------------------------------------------------------------------

## Scenario 48 --- Secret appears in Terraform state

### Question

Why is this dangerous?

### Answer

Terraform state can contain sensitive infrastructure values.

Marking a variable as:

``` hcl
sensitive = true
```

primarily controls display; it does not automatically make the value
absent from state.

Therefore:

-   Secure the backend.
-   Restrict state access.
-   Encrypt remote state.
-   Avoid putting secrets directly into Terraform when possible.
-   Prefer AWS Secrets Manager/SSM Parameter Store or an external
    secret-management system.

Never commit state files to Git.

------------------------------------------------------------------------

## Scenario 49 --- Secret was accidentally committed to Git

### Question

What do you do?

### Answer

Do not merely delete the line in a new commit.

The secret may still exist in Git history.

Immediately:

1.  Revoke/rotate the secret.
2.  Remove exposure.
3.  Audit usage.
4.  Clean repository history if required by policy.
5.  Add secret scanning/prevention.
6.  Check Terraform state and CI logs.

The most important action is **rotation**, because deletion from Git
does not make an exposed credential safe.

------------------------------------------------------------------------

## Scenario 50 --- Terraform is very slow

### Question

`terraform plan` takes 20 minutes. How do you troubleshoot?

### Answer

I determine where the time is spent.

Check:

-   Number of resources.
-   Data sources.
-   Provider API calls.
-   Large modules.
-   Remote backend latency.
-   State size.
-   Refresh behavior.
-   CI network.
-   Excessive dependencies.
-   External provisioners.

I avoid randomly adding timeouts or splitting code without evidence.

------------------------------------------------------------------------

## Scenario 51 --- Terraform hangs during apply

### Question

What is your first move?

### Answer

Do not kill it immediately.

Determine:

-   Which resource is running.
-   Whether AWS is still processing.
-   Whether the provider is waiting for eventual consistency.
-   Whether a dependency is blocked.
-   Whether an API call is retrying.
-   Whether the CI runner is still alive.

Then inspect AWS and Terraform output/logs.

------------------------------------------------------------------------

## Scenario 52 --- Terraform provider says resource does not exist

### Question

The AWS Console shows the resource, but Terraform cannot find it.

### Answer

Check:

1.  AWS account.
2.  AWS region.
3.  Provider alias.
4.  Resource ID.
5.  IAM permissions.
6.  Resource type.
7.  Workspace/backend.
8.  Whether the console user and Terraform role have different
    permissions.

Wrong-account and wrong-region errors are extremely common.

------------------------------------------------------------------------

## Scenario 53 --- Provider alias points to the wrong region/account

### Question

You have multiple AWS providers and the resource appears in the wrong
account.

### Answer

Use explicit aliases:

``` hcl
provider "aws" {
  alias  = "prod"
  region = "ap-south-1"
}
```

Then:

``` hcl
resource "aws_s3_bucket" "example" {
  provider = aws.prod
}
```

I verify provider wiring carefully when using multi-account Terraform.

------------------------------------------------------------------------

## Scenario 54 --- Terraform creates duplicate infrastructure

### Question

Why might this happen?

### Answer

Possible causes:

-   Lost state.
-   Wrong backend.
-   Wrong workspace.
-   Resource was removed from state.
-   Import was never performed.
-   Multiple state files manage the same infrastructure.
-   Different provider/account configuration.

The root problem is often **ownership ambiguity**.

------------------------------------------------------------------------

## Scenario 55 --- Two Terraform projects manage the same resource

### Question

Why is this dangerous?

### Answer

Both states believe they own the same infrastructure.

One project can overwrite or destroy changes made by the other.

The solution is to define clear ownership boundaries and avoid multiple
states managing the same resource unless the design explicitly requires
it.

------------------------------------------------------------------------

## Scenario 56 --- Module output is unknown

### Question

A child module output is not known during plan.

### Answer

This can be normal if the value is generated only during resource
creation.

I inspect whether the consuming resource can accept an unknown value.

If Terraform cannot construct the dependency graph correctly, I
investigate the module interface and resource dependency.

------------------------------------------------------------------------

## Scenario 57 --- Module change causes unexpected replacement

### Question

What do you compare?

### Answer

Compare:

``` bash
terraform plan
terraform state show <resource>
```

Then inspect:

-   Module input changes.
-   Resource address changes.
-   `count`/`for_each`.
-   Provider version.
-   Resource immutable attributes.
-   Renamed resources.
-   Moved blocks.

------------------------------------------------------------------------

## Scenario 58 --- A variable has the wrong value

### Question

Terraform uses a value you didn't expect. How do you find where it came
from?

### Answer

Check precedence and sources:

-   `terraform.tfvars`
-   `*.auto.tfvars`
-   `-var`
-   `-var-file`
-   Environment variables such as `TF_VAR_*`
-   CI/CD injected variables
-   Module inputs

Use:

``` bash
terraform plan
```

and inspect the actual environment/CI configuration.

Do not print secrets into logs while debugging.

------------------------------------------------------------------------

## Scenario 59 --- `terraform fmt` changes many files

### Question

Is that a Terraform failure?

### Answer

No.

Formatting is a code-quality issue.

Use:

``` bash
terraform fmt -recursive
terraform fmt -check -recursive
```

A CI failure on `fmt -check` means the code does not conform to the
expected Terraform formatting.

------------------------------------------------------------------------

## Scenario 60 --- Terraform detects a change but you expected no change

### Question

What is your generic troubleshooting process?

### Answer

I ask four questions:

``` text
What changed?
Who changed it?
Where is the source of truth?
Why does Terraform believe it differs?
```

Then:

``` bash
terraform plan
terraform state show <resource>
```

Compare:

``` text
Configuration
     ↓
State
     ↓
Remote AWS object
```

That three-way comparison is one of the most useful Terraform
troubleshooting models.

------------------------------------------------------------------------

# 3. High-Value Production Scenarios

## Scenario 61 --- Production apply partially failed

### Strong interview answer

> "I would not immediately destroy the environment. First I would
> identify the exact failed resource and inspect Terraform state to
> understand what was successfully created. I would verify the remote
> AWS resources, fix the root cause, run another plan, and only then
> continue the apply. If state and reality diverged, I would reconcile
> them through refresh/plan/import/state operations rather than manually
> editing the state file."

------------------------------------------------------------------------

## Scenario 62 --- Terraform wants to destroy production

### Strong interview answer

> "I stop before apply. A destroy plan is a high-risk signal. I inspect
> why Terraform thinks the resource should be destroyed: configuration
> change, resource address change, provider behavior, state issue,
> workspace/backend mismatch, or drift. I verify the target account and
> workspace, review the plan, and only proceed after confirming the
> replacement/destruction is intentional."

------------------------------------------------------------------------

## Scenario 63 --- Terraform plan shows 500 unexpected changes

### Strong interview answer

> "I would not approve the plan. I would first suspect
> environment/state/provider problems because 500 unexpected changes
> usually indicate a systemic issue rather than 500 independent changes.
> I would verify account, region, backend, workspace, provider version
> and state, then inspect representative resources to determine whether
> the diff is drift, provider behavior, or incorrect configuration."

------------------------------------------------------------------------

## Scenario 64 --- Terraform state is corrupted

### Strong interview answer

> "I would stop concurrent operations, preserve the current state,
> inspect backend versioning/backups, and recover from the known-good
> state if available. I would not manually edit JSON unless there is a
> controlled recovery procedure. After recovery, I would run a plan and
> reconcile any remaining drift."

------------------------------------------------------------------------

## Scenario 65 --- Terraform is modifying resources outside the intended environment

### Strong interview answer

> "I would immediately stop the deployment and verify AWS caller
> identity, backend, workspace, provider aliases, region and variables.
> I would determine whether the problem is wrong credentials or wrong
> state. The priority is preventing additional changes before
> troubleshooting."

------------------------------------------------------------------------

# 4. Terraform Commands You Should Be Able to Explain

## Initialization

``` bash
terraform init
terraform init -upgrade
```

## Formatting

``` bash
terraform fmt
terraform fmt -recursive
terraform fmt -check -recursive
```

## Validation

``` bash
terraform validate
```

## Planning

``` bash
terraform plan
terraform plan -out=tfplan
terraform show tfplan
```

## Apply

``` bash
terraform apply
terraform apply tfplan
```

## State inspection

``` bash
terraform state list
terraform state show <address>
terraform state pull
```

## State manipulation

``` bash
terraform state mv <old> <new>
terraform state rm <address>
```

Use these carefully.

## Import

``` bash
terraform import <address> <id>
```

Or use an import block where supported:

``` hcl
import {
  to = aws_subnet.example
  id = "subnet-xxxxxxxx"
}
```

## Workspace

``` bash
terraform workspace list
terraform workspace show
terraform workspace select <name>
```

## Dependency graph

``` bash
terraform graph
```

## Provider information

``` bash
terraform providers
terraform version
```

------------------------------------------------------------------------

# 5. What NOT to Do in an Interview

Bad answer:

> "I will run `terraform destroy` and recreate everything."

Why it is bad:

-   Destructive.
-   Causes downtime.
-   Can destroy data.
-   Does not diagnose the root cause.

Bad answer:

> "I will use `terraform force-unlock`."

Why it is bad:

You have not established that the lock is stale.

Bad answer:

> "I will delete the state file and run apply again."

Why it is bad:

It can cause Terraform to attempt recreation of existing infrastructure.

Bad answer:

> "I will add `ignore_changes`."

Why it is bad:

You may be hiding drift instead of fixing it.

Bad answer:

> "I will give the Terraform role AdministratorAccess."

Why it is bad:

It masks IAM design problems and violates least privilege.

------------------------------------------------------------------------

# 6. Golden Troubleshooting Pattern

Use this structure in almost every interview:

``` text
1. Read the exact error.
2. Identify the failing layer.
3. Check Terraform configuration.
4. Check Terraform state.
5. Check AWS remote resource.
6. Check identity/account/region.
7. Check dependencies.
8. Check permissions/networking if applicable.
9. Make the smallest safe change.
10. Run terraform plan.
11. Review the plan.
12. Apply.
13. Validate the remote resource.
14. Add prevention/monitoring if it was a recurring problem.
```

------------------------------------------------------------------------

# 7. Terraform Three-Way Troubleshooting Model

Always think:

``` text
                Terraform
               Configuration
                    |
                    v
                  STATE
                    |
                    v
             AWS Remote Object
```

When something is wrong, determine which relationship is broken.

### Configuration ↔ State

Usually resource rename, `count`, `for_each`, module/address changes.

### State ↔ AWS

Usually drift, manual deletion, import/state issues.

### Configuration ↔ AWS

Usually incorrect desired configuration, wrong account/region, API
constraints.

This model is far more useful than memorizing isolated commands.

------------------------------------------------------------------------

# 8. Hands-On Lab

The accompanying file is:

``` text
terraform_scenario_lab.tf
```

It creates a small AWS environment:

``` text
VPC
├── Public Subnet A
├── Public Subnet B
├── Internet Gateway
├── Route Table
├── Security Group
├── EC2 instances
├── Application Load Balancer
├── Target Group
└── S3 bucket
```

The EC2 instances install Nginx.

This gives you enough infrastructure to troubleshoot:

-   VPC connectivity.
-   Route tables.
-   Security groups.
-   ALB.
-   Target health.
-   EC2.
-   AMI selection.
-   State.
-   Drift.
-   Import.
-   Resource replacement.
-   `count`/`for_each`.
-   Dependency behavior.
-   Lifecycle behavior.
-   S3 destroy behavior.

------------------------------------------------------------------------

# 9. Lab Setup

Requirements:

``` bash
terraform version
aws sts get-caller-identity
```

Create a directory:

``` bash
mkdir terraform-scenario-lab
cd terraform-scenario-lab
```

Put the provided:

``` text
terraform_scenario_lab.tf
```

inside it.

Then:

``` bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

Do not use this against a production AWS account.

The lab creates billable AWS resources.

------------------------------------------------------------------------

# 10. Lab Scenario Exercises

## Exercise A --- Wrong AMI

Change the AMI selection to an invalid value or use a wrong-region AMI.

Observe the error.

Then verify:

``` bash
aws sts get-caller-identity
aws ec2 describe-images --region <region> --image-ids <ami-id>
```

Fix the AMI and run:

``` bash
terraform plan
terraform apply
```

------------------------------------------------------------------------

## Exercise B --- Break the ALB health check

Change the target group health-check path from:

``` text
/
```

to:

``` text
/does-not-exist
```

Apply and inspect target health.

Expected lesson:

> Terraform can successfully create an ALB while the application remains
> unhealthy.

------------------------------------------------------------------------

## Exercise C --- Break target port

Change the target group port from:

``` text
80
```

to:

``` text
8080
```

The EC2 instance still serves Nginx on port 80.

Expected lesson:

``` text
ALB → Target Group :8080 → EC2
```

will fail because Nginx is not listening on 8080.

------------------------------------------------------------------------

## Exercise D --- Break security group access

Remove the ALB ingress rule for port 80.

Terraform may apply successfully.

The application becomes unreachable.

Expected lesson:

> Infrastructure deployment success does not equal application
> availability.

------------------------------------------------------------------------

## Exercise E --- Create drift

After Terraform creates the environment, manually change an AWS
resource.

For example, modify a security group rule in the AWS Console.

Then:

``` bash
terraform plan
```

Observe the drift.

Decide whether the manual change should be preserved or reverted.

------------------------------------------------------------------------

## Exercise F --- Delete EC2 manually

Delete one EC2 instance from AWS.

Then:

``` bash
terraform plan
```

Observe Terraform's proposed recovery.

Expected lesson:

> State says the instance exists, AWS says it does not; Terraform
> reconciles desired state against reality.

------------------------------------------------------------------------

## Exercise G --- Rename a resource

Rename a Terraform resource address.

Observe the destroy/create plan.

Then add:

``` hcl
moved {
  from = aws_instance.web
  to   = aws_instance.application
}
```

Run:

``` bash
terraform plan
```

Observe the difference.

------------------------------------------------------------------------

## Exercise H --- State inspection

Run:

``` bash
terraform state list
```

Then:

``` bash
terraform state show <resource>
```

Compare the state with the AWS Console.

------------------------------------------------------------------------

## Exercise I --- Wrong workspace

Create a workspace:

``` bash
terraform workspace new lab
```

Then:

``` bash
terraform workspace show
```

Understand how the state context changes.

For serious production isolation, do not treat workspaces as the only
security/environment boundary.

------------------------------------------------------------------------

## Exercise J --- Import

Create an AWS resource manually.

Then define its Terraform resource block and import it.

After import:

``` bash
terraform plan
```

Your goal is to make the plan converge toward no unexpected changes.

------------------------------------------------------------------------

## Exercise K --- Count/index problem

Create multiple resources using:

``` hcl
count = length(var.instances)
```

Remove the first list element.

Observe how indexes affect resource identity.

Then compare the behavior with:

``` hcl
for_each
```

using stable keys.

------------------------------------------------------------------------

## Exercise L --- Replacement

Change an attribute that forces resource replacement.

Read the plan carefully:

``` text
-/+ destroy and then create replacement
```

Ask:

1.  Why is replacement required?
2.  Is downtime acceptable?
3.  Can `create_before_destroy` help?
4.  Are there uniqueness constraints?
5.  What happens to dependent resources?

------------------------------------------------------------------------

# 11. Interview Rapid-Fire Questions

### Q: What is Terraform state?

State is Terraform's record of the infrastructure objects it manages and
their relationship to the configuration.

### Q: Why remote state?

Centralized state, collaboration, controlled access, recovery/versioning
options and appropriate locking depending on backend.

### Q: Why state locking?

To prevent concurrent state writers from corrupting or conflicting over
state.

### Q: What is drift?

When remote infrastructure differs from what Terraform
configuration/state expects.

### Q: What does import do?

It associates an existing remote resource with a Terraform resource
address/state so Terraform can manage it.

### Q: What does `state rm` do?

Removes a resource from Terraform state without deleting the remote
object.

### Q: What does `moved` do?

Records a resource-address change so Terraform can preserve resource
identity instead of treating the rename as destroy/create.

### Q: `count` vs `for_each`?

`count` uses numeric indexes; `for_each` uses keys. Stable keys are
often safer for independently identifiable resources.

### Q: What is `depends_on`?

An explicit dependency declaration used when Terraform cannot infer the
dependency from resource references.

### Q: What is `create_before_destroy`?

A lifecycle strategy that attempts to create the replacement before
destroying the old resource.

### Q: What is `ignore_changes`?

A lifecycle rule that tells Terraform to ignore changes to specified
attributes. It should not be used to hide unexplained drift.

### Q: Can plan guarantee apply success?

No.

### Q: Should you manually edit state?

Normally no. Prefer supported state/import/move/recovery operations.

### Q: Should you use `-lock=false`?

Normally no for shared state. It removes an important concurrency
protection.

------------------------------------------------------------------------

# 12. Final Interview Rule

When the interviewer gives you a Terraform production incident, do not
start with a command.

Start with the **failure hypothesis**.

Example:

> "First I would determine whether this is a Terraform configuration
> issue, a state issue, an AWS API issue, or an environment issue. Then
> I would inspect the exact error, verify account and region, inspect
> state and the remote resource, identify the root cause, make the
> smallest safe correction, run a plan, review the blast radius, and
> only then apply."

That answer demonstrates production thinking.

------------------------------------------------------------------------

# Official References

-   HashiCorp AWS provider resources:
    https://registry.terraform.io/providers/hashicorp/aws/latest/docs/list-resources/vpc
-   AWS subnet resource:
    https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet
-   Terraform state locking:
    https://developer.hashicorp.com/terraform/language/state/locking

The AWS provider documentation confirms the current provider resource
model, and HashiCorp documents state locking as protection against
concurrent state writers.
