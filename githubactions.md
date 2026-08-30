# GitHub Actions — Top 70 Interview Questions
## DevOps Interview Preparation

> **Focus:** Basics → Core Concepts → CI/CD → Security → Reusable Workflows → Troubleshooting → Production Scenarios  
> **How to use:** First understand the short answer. Then practice explaining each answer in your own words.

---

## 1. GitHub Actions Fundamentals

### 1. What is GitHub Actions?
**Answer:** GitHub Actions is GitHub's built-in CI/CD and automation platform. It allows us to automatically build, test, package, and deploy applications based on repository events or schedules.

### 2. What are the main components of GitHub Actions?
**Answer:** The main components are:
- Workflow
- Events/triggers
- Jobs
- Steps
- Actions
- Runners
- Secrets and variables

### 3. What is a GitHub Actions workflow?
**Answer:** A workflow is a YAML file that defines an automated process. It is stored under `.github/workflows/` and contains triggers, jobs, steps, permissions, and other configuration.

### 4. Where do you create GitHub Actions workflow files?
**Answer:** Inside:
```text
.github/workflows/
```
For example:
```text
.github/workflows/ci.yml
.github/workflows/deploy.yml
```

### 5. What is the difference between a workflow, job, and step?
**Answer:**
- **Workflow:** Complete automation process.
- **Job:** A group of steps executed on a runner.
- **Step:** An individual command or action inside a job.

Example:
```text
Workflow
 ├── Job: Build
 │    ├── Step: Checkout
 │    ├── Step: Install dependencies
 │    └── Step: Build
 └── Job: Test
      ├── Step: Run tests
      └── Step: Publish results
```

### 6. What is an event in GitHub Actions?
**Answer:** An event is something that triggers a workflow. Examples include:
- `push`
- `pull_request`
- `workflow_dispatch`
- `schedule`
- `release`
- `workflow_call`

### 7. What is `on:` in a workflow?
**Answer:** `on:` defines the event or events that trigger the workflow.

Example:
```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

### 8. How do you manually trigger a workflow?
**Answer:** Use `workflow_dispatch`.

```yaml
on:
  workflow_dispatch:
```

This allows the workflow to be started manually from GitHub, and inputs can also be defined.

### 9. How do you trigger a workflow only when code is pushed to `main`?
**Answer:**
```yaml
on:
  push:
    branches:
      - main
```

### 10. How do you trigger a workflow only when specific files change?
**Answer:** Use `paths`.

```yaml
on:
  push:
    paths:
      - 'src/**'
      - 'Dockerfile'
```

---

# 2. Jobs, Steps, and Runners

### 11. What is a job?
**Answer:** A job is a collection of steps that runs on the same runner. Jobs run independently and, by default, can execute in parallel.

### 12. What is a step?
**Answer:** A step is an individual unit inside a job. It can either execute a shell command using `run:` or execute a reusable GitHub Action using `uses:`.

Example:
```yaml
steps:
  - uses: actions/checkout@v4
  - run: npm install
  - run: npm test
```

### 13. What is a runner?
**Answer:** A runner is the machine or execution environment that runs GitHub Actions jobs.

Common options:
- GitHub-hosted runners
- Self-hosted runners

Example:
```yaml
runs-on: ubuntu-latest
```

### 14. What is the difference between GitHub-hosted and self-hosted runners?
**Answer:**

**GitHub-hosted:**
- Managed by GitHub
- Easy to configure
- Usually starts from a clean environment
- Infrastructure maintenance is handled by GitHub

**Self-hosted:**
- Managed by the organization
- More control over network and software
- Useful for private infrastructure or special requirements
- Requires maintenance, patching, and security management

### 15. What does `runs-on` do?
**Answer:** `runs-on` specifies which runner environment should execute the job.

Example:
```yaml
runs-on: ubuntu-latest
```

### 16. Can jobs run in parallel?
**Answer:** Yes. Independent jobs run in parallel by default.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Build"

  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Test"
```

### 17. How do you make one job depend on another?
**Answer:** Use `needs`.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Build"

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploy"
```

The deploy job starts only after the build job succeeds.

### 18. What happens if a job fails?
**Answer:** Normally, dependent jobs using `needs` will not run. Independent jobs can continue unless the workflow structure or conditions prevent them.

### 19. How do you continue a step even if it fails?
**Answer:** Use:

```yaml
- run: ./script.sh
  continue-on-error: true
```

Use this carefully because it can hide genuine failures.

### 20. What is `if:` used for?
**Answer:** `if:` defines a condition under which a job or step should run.

Example:
```yaml
- name: Deploy
  if: github.ref == 'refs/heads/main'
  run: ./deploy.sh
```

---

# 3. Actions and Common CI/CD Tasks

### 21. What is an Action?
**Answer:** An Action is a reusable unit of automation. It can perform a specific task such as checking out code, configuring cloud credentials, uploading artifacts, or setting up a programming language.

### 22. What is the difference between `run:` and `uses:`?
**Answer:**
- `run:` executes shell commands.
- `uses:` invokes an existing action.

Example:
```yaml
- run: npm test
- uses: actions/checkout@v4
```

### 23. Why do we use `actions/checkout`?
**Answer:** It checks out the repository code onto the runner so subsequent build, test, or deployment commands can access it.

Example:
```yaml
- uses: actions/checkout@v4
```

### 24. Why do we use `actions/setup-*` actions?
**Answer:** They install/configure a required runtime on the runner.

Example:
```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
```

### 25. How do you run shell commands in GitHub Actions?
**Answer:**
```yaml
- name: Build
  run: |
    npm install
    npm run build
```

### 26. What is an artifact in GitHub Actions?
**Answer:** An artifact is a file or collection of files produced by a workflow and stored so they can be downloaded or used by later jobs/workflows.

Examples:
- Build packages
- Test reports
- Logs
- Deployment bundles

### 27. What is the difference between artifacts and the Git repository?
**Answer:** The repository contains source-controlled code. Artifacts are generated outputs from workflow execution, such as compiled packages or reports.

### 28. How do you upload an artifact?
**Answer:**
```yaml
- uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/
```

### 29. How do you download an artifact in another job?
**Answer:**
```yaml
- uses: actions/download-artifact@v4
  with:
    name: build-output
```

The consuming job should normally depend on the producing job using `needs`.

### 30. What is caching in GitHub Actions?
**Answer:** Caching stores reusable dependencies or build data between workflow runs to reduce execution time.

Example:
```yaml
- uses: actions/cache@v4
```

Caching dependencies is different from storing final build artifacts.

---

# 4. Variables, Secrets, and Expressions

### 31. What are GitHub Actions variables?
**Answer:** Variables store configuration values that are not necessarily secret, such as environment names, application names, or region names.

They can be defined at organization, repository, or environment scope.

### 32. What are GitHub Actions secrets?
**Answer:** Secrets are encrypted values used for sensitive information such as:
- Cloud credentials
- API tokens
- Passwords
- Private keys

Example:
```yaml
env:
  API_TOKEN: ${{ secrets.API_TOKEN }}
```

### 33. What is the difference between variables and secrets?
**Answer:** Variables are intended for non-sensitive configuration. Secrets are intended for sensitive values and receive additional protection and masking in workflow logs.

### 34. Where can secrets be configured?
**Answer:** Secrets can be configured at:
- Organization level
- Repository level
- Environment level

Environment-level secrets are particularly useful for deployment environments such as staging and production.

### 35. How do you access a secret in a workflow?
**Answer:**
```yaml
${{ secrets.MY_SECRET }}
```

Example:
```yaml
- run: ./deploy.sh
  env:
    AWS_TOKEN: ${{ secrets.AWS_TOKEN }}
```

### 36. What is an expression in GitHub Actions?
**Answer:** Expressions allow dynamic values and conditional logic.

Example:
```yaml
if: ${{ github.ref == 'refs/heads/main' }}
```

### 37. What is the `github` context?
**Answer:** The `github` context provides information about the workflow execution and repository event, such as:
- Repository
- Branch/ref
- Commit SHA
- Actor
- Event name
- Pull request information

Example:
```yaml
${{ github.sha }}
```

### 38. What is the `env` context?
**Answer:** The `env` context provides environment variables defined at workflow, job, or step level.

Example:
```yaml
env:
  APP_ENV: production
```

### 39. What is the difference between environment variables and secrets?
**Answer:** Environment variables are general configuration values. Secrets are specifically designed to protect sensitive information.

Do not put passwords, tokens, or private keys directly into normal environment variables in workflow YAML.

### 40. What is the safest way to handle cloud credentials?
**Answer:** Prefer short-lived, federated credentials such as OIDC where supported instead of storing long-lived cloud access keys as GitHub secrets.

---

# 5. Environments and Deployment

### 41. What is a GitHub Environment?
**Answer:** An environment represents a deployment target such as:
- development
- staging
- production

It can contain environment-specific secrets, variables, and deployment protection rules.

### 42. Why are environments useful for production deployments?
**Answer:** They allow us to separate production configuration and add controls such as required approvals before deployment.

### 43. How do you specify an environment for a job?
**Answer:**
```yaml
jobs:
  deploy:
    environment: production
```

### 44. How can you require approval before production deployment?
**Answer:** Configure protection rules for the production environment, such as required reviewers. The deployment job pauses until the required approval is provided.

### 45. How would you design a CI/CD workflow for an application?
**Answer:**

A common design is:

```text
Developer
   |
   v
Git Push / Pull Request
   |
   v
GitHub Actions
   |
   +--> Checkout
   |
   +--> Install Dependencies
   |
   +--> Lint
   |
   +--> Unit Tests
   |
   +--> Build
   |
   +--> Security Scan
   |
   +--> Package / Docker Build
   |
   +--> Push Artifact/Image
   |
   +--> Deploy to Staging
   |
   +--> Approval
   |
   +--> Deploy to Production
```

---

# 6. Reusable and Advanced Workflows

### 46. What is a reusable workflow?
**Answer:** A reusable workflow is a workflow that another workflow can call. It reduces duplication and standardizes CI/CD processes across repositories.

It uses:
```yaml
on:
  workflow_call:
```

### 47. What is the difference between a reusable workflow and a composite action?
**Answer:**

**Reusable workflow:**
- Reuses complete workflows/jobs
- Can contain multiple jobs
- Called using `workflow_call`

**Composite action:**
- Packages multiple steps into one reusable action
- Primarily used to reuse step logic

### 48. How do you call a reusable workflow?
**Answer:**
```yaml
jobs:
  deploy:
    uses: organization/repository/.github/workflows/deploy.yml@main
```

Inputs and secrets can be passed when required.

### 49. What is a matrix strategy?
**Answer:** A matrix allows the same job to run with multiple combinations of parameters.

Example:
```yaml
strategy:
  matrix:
    node: [18, 20, 22]
```

This can test an application against multiple Node.js versions.

### 50. What is `strategy.fail-fast`?
**Answer:** `fail-fast` controls whether other matrix jobs are cancelled when one matrix job fails.

Example:
```yaml
strategy:
  fail-fast: false
  matrix:
    node: [18, 20, 22]
```

### 51. What is `concurrency` in GitHub Actions?
**Answer:** Concurrency controls simultaneous workflow/job executions. It is useful when we want to prevent multiple deployments to the same environment at the same time.

Example:
```yaml
concurrency:
  group: production
  cancel-in-progress: true
```

### 52. What is a workflow dependency?
**Answer:** A workflow dependency is a relationship where one workflow starts based on another workflow's completion or event. GitHub Actions provides mechanisms such as `workflow_run` and reusable workflows for these patterns.

### 53. What is `workflow_run`?
**Answer:** `workflow_run` triggers a workflow based on the completion of another workflow.

It can be useful when separating CI and deployment workflows.

### 54. How can you pass data between jobs?
**Answer:** Common approaches include:
- Job outputs
- Artifacts
- Environment files such as `$GITHUB_OUTPUT`
- External storage when appropriate

Example:
```yaml
- id: version
  run: echo "version=1.2.3" >> "$GITHUB_OUTPUT"
```

---

# 7. Security

### 55. What is `GITHUB_TOKEN`?
**Answer:** `GITHUB_TOKEN` is an automatically provided token that allows a workflow to authenticate with GitHub APIs and repository resources according to its configured permissions.

### 56. How do you restrict `GITHUB_TOKEN` permissions?
**Answer:** Define explicit permissions.

```yaml
permissions:
  contents: read
```

For production workflows, use least privilege instead of giving broad permissions.

### 57. What is OIDC in GitHub Actions?
**Answer:** OpenID Connect allows GitHub Actions to obtain short-lived cloud credentials without storing long-lived cloud access keys in GitHub secrets.

Typical flow:

```text
GitHub Actions
      |
      | OIDC identity token
      v
Cloud IAM
      |
      | Temporary credentials
      v
AWS / Azure / GCP
```

### 58. Why is OIDC better than storing long-lived cloud access keys?
**Answer:** Long-lived keys can remain valid until rotated or revoked. OIDC can provide short-lived credentials based on workload identity and trust policies, reducing credential exposure and operational overhead.

### 59. What is least privilege in GitHub Actions?
**Answer:** Give the workflow, token, identity, and cloud role only the permissions required for its task.

For example, a build job that only reads repository content should not receive write access to the repository.

### 60. What security risks should you consider in GitHub Actions?
**Answer:** Important risks include:
- Exposing secrets in logs
- Overly broad `GITHUB_TOKEN` permissions
- Untrusted pull request code
- Unsafe use of untrusted input in shell commands
- Compromised third-party actions
- Long-lived cloud credentials
- Insecure self-hosted runners

---

# 8. Troubleshooting and Production Scenarios

### 61. A GitHub Actions workflow is not triggering. How do you troubleshoot it?
**Answer:**
1. Check the workflow YAML syntax.
2. Check the `on:` trigger.
3. Check branch/path filters.
4. Check whether the workflow file exists on the relevant branch.
5. Check repository/workflow permissions.
6. Check whether the event actually occurred.
7. Check Actions settings and whether workflows are enabled.
8. Check organization/repository policies that may restrict Actions.

### 62. A workflow is queued and never starts. What would you check?
**Answer:**
1. Check runner availability.
2. Check `runs-on` labels.
3. For self-hosted runners, verify the runner is online and has matching labels.
4. Check whether the runner is busy.
5. Check concurrency restrictions.
6. Check organization/repository runner policies.
7. Check whether the job is waiting for an environment approval.

### 63. A job fails during the checkout step. What would you check?
**Answer:**
1. Inspect the exact checkout error.
2. Check repository/ref information.
3. Check token permissions.
4. Check whether the repository is private and authentication is valid.
5. Check GitHub service status if the failure appears platform-related.
6. Check network restrictions for self-hosted runners.
7. Check the checkout action version.

### 64. A deployment job fails. How would you troubleshoot it?
**Answer:**
Use a structured approach:

```text
Identify failed job
      |
      v
Identify failed step
      |
      v
Read exact error
      |
      v
Check credentials/permissions
      |
      v
Check target environment
      |
      v
Check application/configuration
      |
      v
Check network/connectivity
      |
      v
Check recent code/config changes
      |
      v
Fix -> rerun -> verify
```

Do not immediately rerun blindly. First identify the failure cause.

### 65. The workflow is running slowly. How would you improve it?
**Answer:**
- Use caching for dependencies.
- Run independent jobs in parallel.
- Avoid unnecessary steps.
- Use appropriate runners.
- Use artifacts intelligently.
- Avoid downloading/installing the same dependencies repeatedly.
- Use matrix jobs only where needed.
- Investigate which steps consume the most execution time.

### 66. How would you prevent two production deployments from running simultaneously?
**Answer:** Use `concurrency`.

```yaml
concurrency:
  group: production-deployment
  cancel-in-progress: false
```

This ensures only one deployment in that concurrency group proceeds at a time, depending on the chosen behavior.

### 67. A secret is not available inside a workflow. What would you check?
**Answer:**
1. Verify the secret name exactly.
2. Check whether it is repository, organization, or environment scoped.
3. Confirm the job references the correct environment.
4. Check whether organization policies restrict access.
5. Check whether the workflow is running from a context where the secret is intentionally unavailable, such as certain forked pull request scenarios.
6. Ensure the secret is referenced correctly:
```yaml
${{ secrets.SECRET_NAME }}
```

### 68. How would you handle deployments from pull requests securely?
**Answer:** Treat pull request code as potentially untrusted, especially when contributions can come from forks.

Do not expose production secrets or powerful credentials to untrusted code. Separate validation/CI from privileged deployment workflows and use protected environments and appropriate permissions.

### 69. Your self-hosted runner is online but jobs are not being picked up. What would you check?
**Answer:**
1. Runner status.
2. Runner labels.
3. `runs-on` labels in the workflow.
4. Repository/organization runner assignment.
5. Runner group access.
6. Whether another job is consuming the runner.
7. Runner service/process health.
8. Network connectivity to GitHub.
9. Runner logs.
10. Whether the runner software is supported/up to date.

### 70. Design a production-grade GitHub Actions CI/CD pipeline for an application deployed to AWS.
**Answer:**

A strong production design could be:

```text
Developer
   |
   v
GitHub Repository
   |
   +------------------------------+
   | Pull Request                 |
   v                              |
CI Workflow                       |
   |                              |
   +--> Checkout                  |
   +--> Setup Runtime             |
   +--> Dependency Install        |
   +--> Lint                      |
   +--> Unit Tests                |
   +--> SAST / Dependency Scan    |
   +--> Build                     |
   +--> Docker Build              |
   +--> Container Scan            |
   +--> Push Image to ECR         |
                                  |
Merge to main --------------------+
   |
   v
Deployment Workflow
   |
   +--> Authenticate using OIDC
   |
   +--> Deploy to Staging
   |
   +--> Smoke Tests
   |
   +--> Production Environment
   |       |
   |       +--> Required Approval
   |
   +--> Deploy to Production
   |
   +--> Post-deployment Verification
   |
   v
Monitoring / Rollback
```

Key production principles:
- Use OIDC instead of long-lived AWS keys where possible.
- Use least-privilege IAM roles.
- Protect the production environment.
- Separate CI and deployment concerns where useful.
- Scan dependencies and container images.
- Pin or carefully control third-party actions.
- Use artifacts or a container registry for immutable build outputs.
- Add deployment concurrency controls.
- Keep secrets out of source code and logs.
- Provide a rollback strategy.

---

# High-Priority Questions to Master First

If your interview is close, do **not** try to memorize all 70 equally. Master these first:

1. What is GitHub Actions?
2. Workflow vs job vs step
3. Events and `on:`
4. `push` vs `pull_request`
5. `workflow_dispatch`
6. GitHub-hosted vs self-hosted runners
7. `runs-on`
8. `needs`
9. `if`
10. `run` vs `uses`
11. Actions and `actions/checkout`
12. Artifacts
13. Caching
14. Variables vs secrets
15. GitHub contexts
16. Environments
17. Environment approvals
18. Reusable workflows
19. Composite actions
20. Matrix strategy
21. Concurrency
22. `GITHUB_TOKEN`
23. Permissions
24. OIDC
25. Least privilege
26. Workflow not triggering troubleshooting
27. Queued workflow troubleshooting
28. Checkout failure troubleshooting
29. Deployment failure troubleshooting
30. Production-grade AWS CI/CD design

---

# Interview Answer Pattern

For scenario-based questions, avoid giving random troubleshooting steps.

Use this structure:

```text
1. Identify the exact failure
2. Locate the failed job/step
3. Read the error message/logs
4. Check the relevant configuration
5. Check permissions/credentials
6. Check runner/environment/network
7. Check recent changes
8. Fix the root cause
9. Rerun the workflow
10. Verify the deployment/application
```

For architecture questions, use:

```text
Trigger
  ↓
CI
  ↓
Build
  ↓
Test
  ↓
Security Scan
  ↓
Package / Image
  ↓
Registry / Artifact
  ↓
Deploy
  ↓
Approval
  ↓
Production
  ↓
Verification / Rollback
```

## Final Preparation Strategy

For an interview, knowing definitions alone is not enough.

You should be able to explain:

- **What it is**
- **Why it is used**
- **How it works**
- **Where you used it**
- **What can fail**
- **How you troubleshoot it**
- **How you secure it**

The highest-value preparation is to practice these 70 questions verbally rather than simply reading them.
