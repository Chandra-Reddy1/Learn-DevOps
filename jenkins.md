# Jenkins — Top 70 Interview Questions
## DevOps Interview Preparation

> **Focus:** Jenkins basics → architecture → jobs/pipelines → Jenkinsfile → agents → credentials → webhooks → CI/CD → security → troubleshooting → production scenarios.
>
> **Goal:** Be able to explain not only *what* Jenkins does, but *why*, *how*, and how you would troubleshoot it in a real DevOps environment.

---

# 1. Jenkins Fundamentals

### 1. What is Jenkins?
**Answer:** Jenkins is an open-source automation server primarily used to implement CI/CD pipelines. It can automate activities such as building code, running tests, creating artifacts/images, and deploying applications.

### 2. Why is Jenkins used in DevOps?
**Answer:** Jenkins automates repetitive software delivery tasks and connects different tools in a CI/CD process.

A typical flow is:

```text
Developer
   |
   v
Git Repository
   |
   v
Jenkins
   |
   +--> Build
   +--> Test
   +--> Code Quality
   +--> Package
   +--> Docker Build
   +--> Push Image
   +--> Deploy
```

### 3. What is Continuous Integration?
**Answer:** Continuous Integration means developers frequently integrate code into a shared repository, with automated builds and tests validating the changes.

### 4. What is Continuous Delivery?
**Answer:** Continuous Delivery means software is automatically built, tested, and prepared for release, while production deployment may require a manual approval or release decision.

### 5. What is Continuous Deployment?
**Answer:** Continuous Deployment goes one step further: successfully validated changes are automatically deployed to production without a manual deployment approval.

### 6. What is the difference between CI, Continuous Delivery, and Continuous Deployment?
**Answer:**

```text
CI
 |
 +--> Build + Test

Continuous Delivery
 |
 +--> Build + Test + Release-ready package
 |
 +--> Production deployment may require approval

Continuous Deployment
 |
 +--> Build + Test + Automatically deploy to Production
```

### 7. What are the main components of Jenkins?
**Answer:** Important components include:
- Jenkins controller
- Jenkins agents/nodes
- Jobs
- Builds
- Pipelines
- Plugins
- Credentials
- Workspace
- Executors
- Build queue

### 8. What is a Jenkins controller?
**Answer:** The controller is the central Jenkins component responsible for managing Jenkins configuration, scheduling work, managing jobs/pipelines, maintaining build metadata, and communicating with agents.

### 9. What is a Jenkins agent?
**Answer:** An agent is a machine or execution environment that performs work assigned by the Jenkins controller.

### 10. What is an executor?
**Answer:** An executor is a slot on a Jenkins node/agent that can execute one task at a time. If all executors are busy, new jobs wait in the queue.

---

# 2. Jenkins Architecture

### 11. What is the difference between a controller and an agent?
**Answer:**

**Controller:**
- Manages Jenkins
- Schedules jobs
- Stores job/build configuration and metadata
- Coordinates agents

**Agent:**
- Executes build/test/deployment work
- Provides compute resources
- Can have specific tools installed

A production Jenkins setup generally avoids using the controller as the primary build machine.

### 12. Why do we use Jenkins agents?
**Answer:** Agents provide scalability and isolation. Different agents can have different operating systems, tools, CPU/memory resources, Docker environments, or network access.

### 13. Can Jenkins have multiple agents?
**Answer:** Yes. A Jenkins controller can manage multiple agents and distribute workloads among them.

### 14. What is a Jenkins node?
**Answer:** A node is a machine/environment that participates in Jenkins execution. The controller itself can be a node, and agents are additional nodes that execute workloads.

### 15. What is a Jenkins label?
**Answer:** A label identifies a node or group of nodes with particular capabilities.

Example:

```text
linux
docker
kubernetes
java
```

A pipeline can request a matching agent.

### 16. What happens when all Jenkins executors are busy?
**Answer:** New jobs enter the Jenkins build queue and wait until an executor becomes available.

### 17. How do you scale Jenkins?
**Answer:** Common approaches include:
- Add more agents
- Increase agent capacity
- Use dynamic agents
- Use Kubernetes-based agents
- Separate workloads by labels
- Optimize slow pipelines
- Manage concurrency carefully

### 18. What is a static agent?
**Answer:** A static agent is a persistent machine configured as a Jenkins agent. It remains available until it is deliberately stopped or removed.

### 19. What is a dynamic agent?
**Answer:** A dynamic agent is created on demand for a job and then removed after the workload finishes. Kubernetes-based Jenkins agents are a common example.

### 20. What is Jenkins distributed build architecture?
**Answer:** In distributed architecture, the controller schedules workloads while multiple agents execute them.

```text
                 Jenkins Controller
                 /       |        \
                /        |         \
          Agent-1     Agent-2    Agent-3
           Linux       Docker     Windows
```

---

# 3. Jenkins Jobs and Builds

### 21. What is a Jenkins job?
**Answer:** A job is a configured unit of work in Jenkins. It defines what Jenkins should execute and under what conditions.

### 22. What is a Jenkins build?
**Answer:** A build is an execution of a Jenkins job or pipeline. It has a build number, logs, status, duration, and other metadata.

### 23. What are common Jenkins job types?
**Answer:** Common types include:
- Freestyle project
- Pipeline
- Multibranch Pipeline
- Organization Folder
- Maven project and other specialized job types depending on installed plugins

### 24. What is a Freestyle job?
**Answer:** A Freestyle project is a traditional Jenkins job configured largely through the Jenkins UI. It can define source control, build steps, post-build actions, and triggers.

### 25. What is a Pipeline job?
**Answer:** A Pipeline job executes a Jenkins Pipeline, usually defined as code in a `Jenkinsfile`.

### 26. What is a Multibranch Pipeline?
**Answer:** A Multibranch Pipeline automatically discovers branches containing a `Jenkinsfile` and creates/manages pipeline executions for those branches.

### 27. What is a Jenkins workspace?
**Answer:** A workspace is a directory on the agent where Jenkins checks out source code and performs build/test operations.

### 28. What happens to workspace files between builds?
**Answer:** Workspace behavior depends on the agent and pipeline configuration. A workspace can persist on a static agent, while dynamic agents often start with a fresh filesystem. Pipelines should not assume that a workspace always contains data from previous builds.

### 29. What is the Jenkins build queue?
**Answer:** The build queue contains tasks waiting for an executor or an appropriate node/agent.

### 30. How do you manually trigger a Jenkins job?
**Answer:** If the job permits it, click **Build Now** or **Build with Parameters** from the Jenkins UI. Pipelines can also be triggered through APIs or webhooks.

---

# 4. Jenkins Pipeline and Jenkinsfile

### 31. What is Jenkins Pipeline?
**Answer:** Jenkins Pipeline is a suite of plugins that supports implementing CI/CD workflows as code.

Example:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
    }
}
```

### 32. What is a Jenkinsfile?
**Answer:** A Jenkinsfile is a text file containing the Jenkins Pipeline definition. It is commonly stored in source control with the application code.

### 33. Why should we store the Jenkinsfile in Git?
**Answer:** Pipeline-as-code provides:
- Version control
- Code review
- Audit history
- Reproducibility
- Easier rollback
- Collaboration

### 34. What is Declarative Pipeline?
**Answer:** Declarative Pipeline is a structured Jenkins Pipeline syntax designed to make pipeline definitions easier to read and maintain.

Example:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
    }
}
```

### 35. What is Scripted Pipeline?
**Answer:** Scripted Pipeline uses Groovy-based scripting and provides more programming flexibility, but can become harder to maintain if the pipeline becomes overly complex.

### 36. Declarative vs Scripted Pipeline — what is the difference?
**Answer:**

**Declarative:**
- Structured syntax
- Easier to read
- Easier to enforce conventions
- Good default choice for most CI/CD pipelines

**Scripted:**
- More flexible
- More programmatic control
- Useful for complex logic
- Easier to create difficult-to-maintain pipelines if overused

### 37. What are stages and steps?
**Answer:**
- **Stage:** Logical section of a pipeline, such as Build, Test, or Deploy.
- **Step:** Individual operation executed within a stage.

Example:

```text
Pipeline
  |
  +-- Build stage
  |     +-- Checkout
  |     +-- Compile
  |
  +-- Test stage
  |     +-- Unit tests
  |
  +-- Deploy stage
        +-- Deploy application
```

### 38. What does `agent` mean in a Jenkins Pipeline?
**Answer:** `agent` specifies where the pipeline or a stage should execute.

Example:

```groovy
agent any
```

Or:

```groovy
agent {
    label 'docker'
}
```

### 39. What is `post` in a Jenkins Pipeline?
**Answer:** `post` defines actions that run after a pipeline or stage, depending on conditions such as:
- `always`
- `success`
- `failure`
- `unstable`
- `changed`

Example:

```groovy
post {
    always {
        echo 'Pipeline completed'
    }
}
```

### 40. What is `environment` in Jenkins Pipeline?
**Answer:** `environment` defines environment variables for the whole pipeline or a specific stage.

Example:

```groovy
environment {
    APP_ENV = 'staging'
}
```

---

# 5. Pipeline Control and Advanced Basics

### 41. What is `parameters` in Jenkins?
**Answer:** Parameters allow users or external triggers to provide input when starting a build.

Example:

```groovy
parameters {
    choice(
        name: 'ENVIRONMENT',
        choices: ['dev', 'staging', 'prod'],
        description: 'Deployment environment'
    )
}
```

### 42. How do you conditionally execute a stage?
**Answer:** Declarative Pipeline supports `when`.

Example:

```groovy
stage('Deploy') {
    when {
        branch 'main'
    }
    steps {
        sh './deploy.sh'
    }
}
```

### 43. How do you run stages in parallel?
**Answer:** Declarative Pipeline supports parallel stages.

Example:

```groovy
stage('Tests') {
    parallel {
        stage('Unit') {
            steps {
                sh 'npm test'
            }
        }

        stage('Security') {
            steps {
                sh './security-scan.sh'
            }
        }
    }
}
```

### 44. How do you make a pipeline fail when a shell command fails?
**Answer:** Jenkins normally treats a non-zero exit code from `sh` as a step failure.

Example:

```groovy
sh './deploy.sh'
```

If the script returns a non-zero exit status, the step normally fails.

### 45. How do you intentionally ignore a command failure?
**Answer:** You can explicitly handle the return status.

Example:

```groovy
def status = sh(
    script: './check.sh',
    returnStatus: true
)

echo "Exit code: ${status}"
```

Use this carefully. Ignoring failures can hide real problems.

### 46. What is a shared library in Jenkins?
**Answer:** A Jenkins Shared Library is reusable pipeline code stored separately and imported into pipelines. It helps standardize CI/CD logic across many repositories.

### 47. Why use Jenkins Shared Libraries?
**Answer:** They reduce duplicated pipeline code and allow organizations to centralize common practices such as:
- Build logic
- Security scanning
- Deployment
- Notifications
- Release procedures

### 48. What is a Jenkins plugin?
**Answer:** A plugin extends Jenkins functionality. Examples include integrations with Git, Docker, Kubernetes, credentials providers, artifact repositories, and cloud platforms.

### 49. Why should Jenkins plugins be managed carefully?
**Answer:** Plugins can introduce:
- Security vulnerabilities
- Compatibility problems
- Dependency conflicts
- Unexpected behavior

Use controlled versions, test upgrades, and avoid installing unnecessary plugins.

### 50. What is Blue Ocean?
**Answer:** Blue Ocean is a Jenkins user interface/project that provides a more visual Pipeline-oriented experience. It is not the core Jenkins execution engine.

---

# 6. Git Integration and Triggers

### 51. How does Jenkins integrate with Git?
**Answer:** Jenkins can check out source code from Git repositories using Git-related plugins and credentials. A pipeline can then build, test, package, and deploy that code.

Example:

```groovy
stage('Checkout') {
    steps {
        git branch: 'main',
            url: 'https://github.com/example/app.git'
    }
}
```

### 52. What is a webhook?
**Answer:** A webhook allows an external system such as GitHub to send an HTTP request to Jenkins when an event occurs.

Typical flow:

```text
Developer
   |
   v
Git Push
   |
   v
GitHub
   |
   | Webhook
   v
Jenkins
   |
   v
Pipeline
```

### 53. Why are webhooks preferred over frequent polling?
**Answer:** Webhooks provide event-driven triggering. Jenkins does not need to repeatedly ask the Git server whether something changed, which can reduce unnecessary traffic and trigger builds faster.

### 54. What is polling SCM?
**Answer:** Jenkins periodically checks the source-control repository for changes. If changes are detected, it can trigger a build.

### 55. Webhook is configured but Jenkins is not triggering. How would you troubleshoot it?
**Answer:**

```text
1. Check GitHub webhook delivery status
2. Verify Jenkins webhook URL
3. Check Jenkins job trigger configuration
4. Check GitHub/Jenkins authentication if applicable
5. Check network/firewall/proxy configuration
6. Check Jenkins logs
7. Verify the event and branch filters
8. Check whether the job is disabled
9. Test webhook delivery again
```

---

# 7. Credentials and Security

### 56. What is Jenkins Credentials Store?
**Answer:** Jenkins Credentials Store securely manages credentials used by jobs, such as:
- Username/password
- SSH keys
- API tokens
- Secret text
- Certificates
- Cloud credentials through supported integrations

### 57. How do you use credentials in a Jenkins Pipeline?
**Answer:** Use Jenkins credential bindings rather than hardcoding secrets.

Example:

```groovy
withCredentials([
    string(
        credentialsId: 'api-token',
        variable: 'API_TOKEN'
    )
]) {
    sh './deploy.sh'
}
```

### 58. Why should credentials not be hardcoded in a Jenkinsfile?
**Answer:** A Jenkinsfile is normally stored in source control. Hardcoded credentials can therefore be exposed through Git history, pull requests, forks, logs, or backups.

### 59. What is credential binding?
**Answer:** Credential binding temporarily exposes a stored Jenkins credential to a build step through variables or files without putting the credential directly in the Jenkinsfile.

### 60. How do you secure Jenkins?
**Answer:** Important practices include:
- Use authentication and authorization
- Apply least privilege
- Keep Jenkins and plugins patched
- Restrict network exposure
- Protect credentials
- Use HTTPS
- Limit administrative access
- Secure agents
- Avoid running builds with unnecessary privileges
- Audit users and credentials
- Back up Jenkins configuration appropriately

---

# 8. Artifacts, Docker, and Deployment

### 61. What is an artifact in Jenkins?
**Answer:** An artifact is a file generated by a build and stored by Jenkins or an external artifact repository for later use.

Examples:

```text
JAR
WAR
ZIP
Test reports
Deployment packages
```

### 62. How do you publish artifacts?
**Answer:** Jenkins can archive artifacts or publish them to external repositories such as Nexus, Artifactory, or cloud/container registries, depending on the organization's architecture.

### 63. How would you build a Docker image using Jenkins?
**Answer:** A typical pipeline would:

```text
Checkout
   |
Build/Test
   |
Docker Build
   |
Security Scan
   |
Tag Image
   |
Push to Registry
```

Example:

```groovy
sh 'docker build -t myapp:${BUILD_NUMBER} .'
sh 'docker push myregistry/myapp:${BUILD_NUMBER}'
```

The actual authentication should use Jenkins credentials or a secure identity mechanism rather than hardcoded passwords.

### 64. How would Jenkins deploy an application to Kubernetes?
**Answer:** A common flow is:

```text
Jenkins
   |
   +--> Build application
   +--> Build Docker image
   +--> Push image to registry
   +--> Authenticate to cluster
   +--> Update deployment
   +--> Wait for rollout
   +--> Verify application
```

Example command:

```bash
kubectl set image deployment/myapp \
  myapp=myregistry/myapp:${BUILD_NUMBER}
```

The exact authentication and deployment mechanism depends on the Kubernetes environment.

### 65. How would you implement rollback in a Jenkins deployment pipeline?
**Answer:** The rollback strategy depends on the deployment technology. For Kubernetes, for example, you can use deployment history and rollout mechanisms.

Conceptually:

```text
Deploy
  |
Smoke Test
  |
  +--> Pass --> Continue
  |
  +--> Fail --> Rollback
```

A good pipeline should define what constitutes failure and how the previous known-good version is restored.

---

# 9. Troubleshooting Scenarios

### 66. A Jenkins job is stuck in the queue. What would you check?
**Answer:**

Check systematically:

1. Is an executor available?
2. Is the required agent online?
3. Does the job's label match an available agent?
4. Is the agent temporarily offline?
5. Is the agent at capacity?
6. Is the job blocked by throttling/concurrency configuration?
7. Is it waiting for an input/approval?
8. Is there a node provisioning problem?
9. Check Jenkins logs and queue information.

Do not immediately restart Jenkins without understanding why the job is queued.

### 67. A Jenkins pipeline fails during checkout. How would you troubleshoot it?
**Answer:**

Check:

```text
1. Exact checkout error
2. Repository URL
3. Branch/ref
4. Git installation on the agent
5. Jenkins credentials
6. Repository permissions
7. Network/DNS/proxy/firewall
8. SSH known-host configuration if SSH is used
9. Git plugin configuration
10. Agent connectivity
```

The first step is always to identify the exact error rather than assuming it is a credential problem.

### 68. A Jenkins pipeline works manually but fails when triggered by GitHub. What would you check?
**Answer:**

Compare the two execution contexts:

- Trigger configuration
- Branch/ref
- Environment variables
- Credentials available to the build
- Parameters
- Permissions
- Webhook payload
- Agent selection
- Workspace
- SCM checkout behavior

A manual build and a webhook-triggered build may not execute with identical parameters or context.

### 69. A Jenkins build suddenly started failing after working for weeks. How would you troubleshoot it?
**Answer:**

Use a change-focused investigation:

```text
1. Identify the exact failing stage
2. Compare the failing build with the last successful build
3. Check source-code changes
4. Check Jenkinsfile changes
5. Check plugin upgrades
6. Check agent/image changes
7. Check dependency/version changes
8. Check credentials/secret expiration
9. Check external service changes
10. Reproduce and fix the root cause
```

Avoid blindly rerunning the build repeatedly. A successful rerun does not explain why the original build failed.

### 70. Design a production-grade Jenkins CI/CD pipeline for an application deployed to AWS/Kubernetes.
**Answer:**

A strong production architecture could look like:

```text
Developer
   |
   v
GitHub
   |
   | Pull Request / Push
   v
Webhook
   |
   v
Jenkins Controller
   |
   +-------------------------------+
   |                               |
   v                               v
Dynamic Build Agents          Pipeline Metadata
   |
   +--> Checkout
   |
   +--> Dependency Install
   |
   +--> Lint
   |
   +--> Unit Tests
   |
   +--> SAST / Dependency Scan
   |
   +--> Build Application
   |
   +--> Docker Build
   |
   +--> Container Scan
   |
   +--> Push Image
           |
           v
       ECR / Registry
           |
           v
     Deploy to Staging
           |
           v
      Smoke Tests
           |
           v
     Approval / Gate
           |
           v
    Deploy Production
           |
           v
   Health Verification
           |
      +----+----+
      |         |
    Pass      Fail
      |         |
      v         v
   Finish    Rollback
```

Production principles:

- Keep the Jenkins controller focused on orchestration.
- Use agents for builds.
- Prefer ephemeral/dynamic agents where practical.
- Store the Jenkinsfile in Git.
- Use Jenkins Credentials Store or an external secret-management system.
- Use least-privilege cloud identities.
- Avoid long-lived cloud credentials where short-lived identity mechanisms such as AWS IAM/OIDC integration are available.
- Scan dependencies and container images.
- Use immutable image tags where possible.
- Protect production deployment stages.
- Add approval gates when required by the release process.
- Implement deployment health checks.
- Have a defined rollback strategy.
- Monitor Jenkins and build agents.
- Back up critical Jenkins configuration/data.
- Control and regularly review plugins.

---

# High-Priority Questions to Master First

If the interview is only a few days away, prioritize these questions:

## Fundamentals
1. What is Jenkins?
2. Why Jenkins?
3. CI vs Continuous Delivery vs Continuous Deployment
4. Jenkins controller vs agent
5. Executor
6. Node and label
7. Job vs build
8. Workspace

## Pipeline
9. What is Jenkins Pipeline?
10. What is Jenkinsfile?
11. Why Pipeline as Code?
12. Declarative vs Scripted Pipeline
13. Stage vs step
14. `agent`
15. `post`
16. `environment`
17. `parameters`
18. `when`
19. Parallel stages
20. Shared Libraries

## Git/Webhooks
21. Jenkins + Git integration
22. Webhooks
23. Poll SCM
24. Webhook troubleshooting

## Security
25. Jenkins Credentials
26. Credential binding
27. Why never hardcode credentials
28. Jenkins security
29. Least privilege
30. Secure cloud authentication

## Troubleshooting
31. Job stuck in queue
32. Agent offline
33. Checkout failure
34. Pipeline works manually but not from webhook
35. Suddenly failing pipeline

## Production
36. Artifact management
37. Docker build/push
38. Kubernetes deployment
39. Rollback
40. Production-grade Jenkins architecture

---

# Jenkins Pipeline Example to Understand

A basic CI/CD pipeline:

```groovy
pipeline {
    agent any

    environment {
        APP_NAME = 'myapp'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh './build.sh'
            }
        }

        stage('Test') {
            steps {
                sh './test.sh'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${APP_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh './deploy.sh'
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed'
        }

        failure {
            echo 'Pipeline failed'
        }
    }
}
```

Understand every line of this example. An interviewer can take almost any line and ask a follow-up question.

---

# Jenkins Troubleshooting Framework

When an interviewer gives you a Jenkins production problem, answer systematically:

```text
1. Identify the failed job/pipeline
          |
          v
2. Identify the exact failed stage
          |
          v
3. Read the console output
          |
          v
4. Identify the error category
          |
          +--> SCM / Git
          +--> Credentials
          +--> Agent
          +--> Network
          +--> Dependency
          +--> Application
          +--> Plugin
          +--> External service
          |
          v
5. Check recent changes
          |
          v
6. Fix root cause
          |
          v
7. Rerun / verify
          |
          v
8. Confirm deployment/application health
```

## Important Interview Principle

Do not say:

> "I will restart Jenkins."

That is not a troubleshooting strategy.

Say what you would **check first**, identify the evidence, isolate the failure, and then take corrective action.

---

# Jenkins vs GitHub Actions — Know the Difference

| Area | Jenkins | GitHub Actions |
|---|---|---|
| Platform | Automation server | GitHub-native automation |
| Infrastructure | Usually self-managed | GitHub-hosted or self-hosted runners |
| Pipeline definition | Jenkinsfile | YAML workflow |
| Extensibility | Large plugin ecosystem | Actions ecosystem |
| Controller | Jenkins controller | GitHub manages orchestration for hosted Actions |
| Agents | Jenkins agents | Runners |
| Credentials | Jenkins Credentials Store | GitHub Secrets / OIDC |
| Triggering | Webhooks, SCM polling, schedules, etc. | Repository events, schedules, dispatch, etc. |
| CI/CD | Yes | Yes |
| Maintenance | Organization manages Jenkins infrastructure | Less infrastructure management with GitHub-hosted runners |

---

# Final Interview Preparation Strategy

For every important Jenkins question, practice answering using this structure:

```text
1. Definition
2. Why we use it
3. How it works
4. Real-world example
5. Common failure
6. Troubleshooting approach
```

For example, if asked:

> "What happens when a Jenkins job is stuck in the queue?"

Do not answer only:

> "The agent is busy."

A stronger answer is:

```text
First I would check why the job is queued.

I would check whether an executor is available,
whether the required agent is online,
whether the job label matches an available agent,
whether the agent has capacity,
and whether concurrency/throttling or an approval
is preventing execution.

Then I would inspect Jenkins queue information
and logs to identify the actual cause.
```

That demonstrates operational understanding rather than command memorization.

---

# Final Must-Know Jenkins Command/Concept Cheat Sheet

```text
Jenkins
  |
  +-- Controller
  +-- Agent
  +-- Node
  +-- Executor
  +-- Label
  +-- Job
  +-- Build
  +-- Workspace
  +-- Pipeline
  +-- Jenkinsfile
  +-- Stage
  +-- Step
  +-- Plugin
  +-- Credential
  +-- Shared Library
  +-- Webhook
  +-- Artifact
```

## Core Pipeline keywords

```groovy
pipeline
agent
stages
stage
steps
post
environment
parameters
when
parallel
input
options
triggers
```

## Core CI/CD flow

```text
Git Push
   ↓
Webhook
   ↓
Jenkins
   ↓
Checkout
   ↓
Build
   ↓
Test
   ↓
Quality/Security Scan
   ↓
Package
   ↓
Docker Build
   ↓
Registry
   ↓
Deploy
   ↓
Smoke Test
   ↓
Production
   ↓
Verification / Rollback
```

## The most important concepts to truly understand

```text
Controller vs Agent
Job vs Build
Node vs Executor
Workspace
Jenkinsfile
Declarative vs Scripted Pipeline
Stage vs Step
Credentials
Webhook
Shared Library
Artifacts
Queue
Agent labels
Parallel execution
Pipeline failure handling
Production deployment
Rollback
Troubleshooting
```
