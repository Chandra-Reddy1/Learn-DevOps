# 🚀 Jenkins Interview Questions & Answers
### Complete Guide for DevOps Engineers — Basics to Troubleshooting

---

## 📑 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Jenkins Architecture](#2-jenkins-architecture)
3. [Pipeline & Jenkinsfile](#3-pipeline--jenkinsfile)
4. [Plugins & Integrations](#4-plugins--integrations)
5. [Security & Access Control](#5-security--access-control)
6. [CI/CD Best Practices](#6-cicd-best-practices)
7. [Jenkins Agents & Distributed Builds](#7-jenkins-agents--distributed-builds)
8. [Advanced Topics](#8-advanced-topics)
9. [Troubleshooting — Common Issues & Fixes](#9-troubleshooting--common-issues--fixes)

---

## 1. Core Concepts

---

### Q1. What is Jenkins and why is it used?

**Answer:**
Jenkins is an open-source automation server written in Java. It is used to automate the building, testing, and deployment of software — the core of a CI/CD pipeline.

**Key benefits:**
- Automates repetitive tasks (build, test, deploy)
- Supports hundreds of plugins for integration with tools like Git, Docker, Kubernetes, AWS
- Provides real-time feedback on code quality
- Supports distributed builds across multiple agents

---

### Q2. What is Continuous Integration (CI) and Continuous Delivery (CD)?

**Answer:**

| Term | Meaning |
|------|---------|
| **CI** | Automatically build and test code on every commit |
| **CD (Delivery)** | Automatically prepare a release after CI passes |
| **CD (Deployment)** | Automatically deploy to production without manual approval |

Jenkins supports all three stages through pipelines.

---

### Q3. What are the types of Jenkins jobs?

**Answer:**

| Job Type | Description |
|----------|-------------|
| **Freestyle Project** | Basic GUI-configured job; suitable for simple builds |
| **Pipeline** | Code-based pipeline using Groovy/Jenkinsfile |
| **Multibranch Pipeline** | Auto-discovers branches in a repo and creates pipelines for each |
| **Folder** | Organizes jobs into groups |
| **Multi-configuration (Matrix)** | Runs same job on multiple environments |
| **GitHub Organization** | Scans an entire GitHub org for repos with Jenkinsfiles |

---

### Q4. What is the difference between Freestyle and Pipeline jobs?

**Answer:**

| Feature | Freestyle | Pipeline |
|---------|-----------|---------|
| Configuration | GUI-based | Code-based (Jenkinsfile) |
| Version control | Not natively | Yes — stored in SCM |
| Complexity | Simple tasks | Complex workflows |
| Restart from stage | ❌ | ✅ |
| Parallelism | Limited | Full support |
| Reusability | Low | High (shared libraries) |

---

### Q5. What is a Jenkinsfile?

**Answer:**
A Jenkinsfile is a text file written in Groovy DSL that defines a Jenkins Pipeline. It lives in the root of your source code repository and is version-controlled.

There are two syntax styles:

**Declarative (recommended):**
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```

**Scripted (more flexibility):**
```groovy
node {
    stage('Build') {
        sh 'mvn clean package'
    }
}
```

---

## 2. Jenkins Architecture

---

### Q6. Explain the Jenkins Master-Agent architecture.

**Answer:**

- **Master (Controller):** The central Jenkins server. It schedules builds, dispatches jobs to agents, monitors agents, and presents the UI.
- **Agent (Node/Slave):** A machine that executes the actual build steps delegated by the master.

```
          Jenkins Master
         /       |       \
      Agent1   Agent2   Agent3
    (Linux)  (Windows)  (Docker)
```

**Benefits:**
- Offloads build workload from master
- Supports heterogeneous environments
- Scales horizontally

---

### Q7. How does Jenkins communicate with agents?

**Answer:**
Jenkins supports multiple agent connection methods:

| Method | Description |
|--------|-------------|
| **SSH** | Master SSHes into the agent (Linux/Mac) |
| **JNLP / WebSocket** | Agent initiates connection to master (useful behind firewalls) |
| **Docker Agent** | Spins up a container as an agent per build |
| **Kubernetes Pod** | Creates a pod per build using the Kubernetes plugin |

---

### Q8. What is the Jenkins home directory and what does it contain?

**Answer:**
Default path: `/var/lib/jenkins` (Linux) or `C:\ProgramData\Jenkins` (Windows)

Key contents:

| Path | Purpose |
|------|---------|
| `jobs/` | All job configurations and build histories |
| `workspace/` | Checked-out source code for builds |
| `plugins/` | Installed plugins |
| `config.xml` | Global Jenkins configuration |
| `secrets/` | Credentials and secret files |
| `logs/` | Jenkins log files |

---

## 3. Pipeline & Jenkinsfile

---

### Q9. What are the main sections of a Declarative Pipeline?

**Answer:**

```groovy
pipeline {
    agent any                    // Where to run

    environment {                // Environment variables
        APP_ENV = 'staging'
    }

    options {                    // Pipeline-level options
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/example/repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
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

    post {                       // Post-build actions
        success { echo 'Build succeeded!' }
        failure { mail to: 'team@example.com', subject: 'Build Failed' }
        always  { cleanWs() }
    }
}
```

---

### Q10. How do you run stages in parallel in Jenkins?

**Answer:**
```groovy
stage('Parallel Tests') {
    parallel {
        stage('Unit Tests') {
            steps { sh 'mvn test -Punit' }
        }
        stage('Integration Tests') {
            steps { sh 'mvn test -Pintegration' }
        }
        stage('Security Scan') {
            steps { sh './run-security-scan.sh' }
        }
    }
}
```

---

### Q11. What is a Shared Library in Jenkins and why is it used?

**Answer:**
A Shared Library is a reusable collection of Groovy scripts stored in a separate Git repository that can be imported across multiple Jenkinsfiles. It prevents code duplication.

**Directory structure:**
```
shared-library/
├── vars/
│   └── deployApp.groovy     # Global variable/function
├── src/
│   └── org/example/Utils.groovy  # Groovy classes
└── resources/
    └── scripts/deploy.sh    # Static resources
```

**Usage in Jenkinsfile:**
```groovy
@Library('my-shared-library') _

pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                deployApp('production')
            }
        }
    }
}
```

---

### Q12. How do you pass parameters to a Jenkins pipeline?

**Answer:**
```groovy
pipeline {
    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to build')
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run test suite?')
        password(name: 'SECRET_KEY', defaultValue: '', description: 'API Secret')
    }

    stages {
        stage('Build') {
            steps {
                echo "Building branch: ${params.BRANCH}"
                echo "Deploying to: ${params.ENV}"
            }
        }
    }
}
```

---

### Q13. How do you use environment variables and credentials in Jenkins?

**Answer:**
```groovy
pipeline {
    agent any

    environment {
        // Plain environment variable
        APP_NAME = 'my-app'

        // Inject credentials from Jenkins credential store
        AWS_CREDS    = credentials('aws-access-key-id')
        DOCKER_PASS  = credentials('dockerhub-password')
        GIT_TOKEN    = credentials('github-token')  // username:password or token
    }

    stages {
        stage('Deploy') {
            steps {
                sh '''
                    echo "App: $APP_NAME"
                    aws configure set aws_access_key_id $AWS_CREDS_USR
                    aws configure set aws_secret_access_key $AWS_CREDS_PSW
                '''
            }
        }
    }
}
```

---

### Q14. What is the `when` directive in Jenkins Pipeline?

**Answer:**
The `when` directive allows conditional execution of stages based on conditions:

```groovy
stage('Deploy to Production') {
    when {
        allOf {
            branch 'main'
            environment name: 'DEPLOY_ENV', value: 'production'
        }
    }
    steps {
        sh './deploy-prod.sh'
    }
}

stage('Run on Tag') {
    when {
        tag "v*"   // Only on version tags
    }
    steps { sh './release.sh' }
}
```

---

## 4. Plugins & Integrations

---

### Q15. What are the most essential Jenkins plugins for a DevOps engineer?

**Answer:**

| Plugin | Purpose |
|--------|---------|
| **Git / GitHub** | SCM integration |
| **Pipeline** | Jenkinsfile support |
| **Blue Ocean** | Modern pipeline UI |
| **Docker Pipeline** | Build/push Docker images |
| **Kubernetes** | Dynamic Kubernetes agents |
| **Credentials Binding** | Secure secret injection |
| **SonarQube Scanner** | Code quality analysis |
| **Slack Notification** | Build alerts to Slack |
| **Email Extension** | Custom email notifications |
| **JUnit** | Test result publishing |
| **Artifactory / Nexus** | Artifact management |
| **Ansible** | Run Ansible playbooks |
| **Parameterized Trigger** | Trigger downstream jobs |
| **Build Timeout** | Prevent hung builds |
| **Role-based Authorization** | Fine-grained access control |

---

### Q16. How do you integrate Jenkins with GitHub for automated builds?

**Answer:**

**Steps:**
1. Install the **GitHub plugin** and **GitHub Integration plugin**
2. In GitHub repo → Settings → Webhooks → Add webhook
   - Payload URL: `http://<jenkins-url>/github-webhook/`
   - Content type: `application/json`
   - Event: `push` (and optionally `pull_request`)
3. In Jenkins job → Build Triggers → ✅ **GitHub hook trigger for GITScm polling**

**Result:** Every `git push` triggers a Jenkins build automatically.

---

### Q17. How do you integrate Jenkins with Docker?

**Answer:**
```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.8.6-openjdk-11'
            args '-v /root/.m2:/root/.m2'   // Cache Maven dependencies
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def image = docker.build("myapp:${BUILD_NUMBER}")
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {
                        image.push()
                        image.push('latest')
                    }
                }
            }
        }
    }
}
```

---

## 5. Security & Access Control

---

### Q18. How do you secure Jenkins?

**Answer:**

**Authentication:**
- Enable security: Manage Jenkins → Configure Global Security
- Use Matrix-based or Role-Based Access Control (RBAC)
- Integrate with LDAP, Active Directory, or GitHub OAuth

**Authorization levels:**
- Admin: Full access
- Developer: Build trigger, read logs
- Read-only: View dashboards

**Other hardening steps:**
- Disable CLI over HTTP
- Enable CSRF protection (enabled by default in newer versions)
- Use HTTPS (SSL/TLS) for Jenkins URL
- Keep Jenkins and plugins updated
- Restrict agent-to-master communication using the `Approved Scripts` in Script Security
- Store secrets in the Jenkins Credentials Manager — never hardcode

---

### Q19. What is the Jenkins Credentials store and how do you use it?

**Answer:**
Jenkins has a built-in encrypted credentials manager.

**Types of credentials:**
- Username + password
- SSH private key
- Secret text / Secret file
- Certificate (PKCS#12)
- AWS / cloud credentials (via plugins)

**Access in Jenkinsfile:**
```groovy
withCredentials([
    usernamePassword(credentialsId: 'my-creds', usernameVariable: 'USER', passwordVariable: 'PASS'),
    sshUserPrivateKey(credentialsId: 'my-ssh-key', keyFileVariable: 'SSH_KEY')
]) {
    sh 'ssh -i $SSH_KEY user@server "deploy.sh"'
    sh 'curl -u $USER:$PASS https://api.example.com'
}
```

---

## 6. CI/CD Best Practices

---

### Q20. What are best practices for writing Jenkinsfiles?

**Answer:**
- Store Jenkinsfile in source code repository (GitOps)
- Use Declarative syntax for readability
- Extract reusable logic into Shared Libraries
- Use `environment` block for all environment variables
- Store secrets in Jenkins Credentials — never in code
- Add `timeout` and `buildDiscarder` options
- Use `post` blocks for cleanup (`cleanWs()`)
- Add `retry` for flaky network steps
- Use parallel stages to reduce pipeline execution time
- Validate pipeline syntax using the **Pipeline Syntax Generator** in Jenkins UI

---

### Q21. How do you implement a Blue-Green deployment in Jenkins?

**Answer:**
```groovy
stage('Blue-Green Deploy') {
    steps {
        script {
            def current = sh(script: 'get_active_env.sh', returnStdout: true).trim()
            def target  = (current == 'blue') ? 'green' : 'blue'

            sh "deploy.sh ${target}"
            sh "health-check.sh ${target}"
            sh "switch-traffic.sh ${target}"

            echo "Traffic switched from ${current} to ${target}"
        }
    }
}
```

---

### Q22. How do you trigger a downstream job from Jenkins?

**Answer:**
```groovy
// Method 1: build step
stage('Trigger Deploy') {
    steps {
        build job: 'deploy-to-staging',
              parameters: [string(name: 'VERSION', value: "${BUILD_NUMBER}")],
              wait: true
    }
}

// Method 2: Post-build trigger
post {
    success {
        build job: 'integration-tests', wait: false
    }
}
```

---

## 7. Jenkins Agents & Distributed Builds

---

### Q23. How do you configure a dynamic Kubernetes agent in Jenkins?

**Answer:**
Using the **Kubernetes plugin:**
```groovy
pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.8.6-openjdk-11
    command: ['cat']
    tty: true
  - name: docker
    image: docker:20.10
    command: ['cat']
    tty: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
"""
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
    }
}
```

---

### Q24. How do you label and assign jobs to specific agents?

**Answer:**
```groovy
// Assign to specific label
pipeline {
    agent { label 'linux && docker' }
    ...
}

// Assign stage to different agent
stage('Windows Tests') {
    agent { label 'windows' }
    steps {
        bat 'run-tests.bat'
    }
}
```

In Jenkins UI: Manage Jenkins → Nodes → Agent → Labels → assign label strings.

---

## 8. Advanced Topics

---

### Q25. What is Jenkins Configuration as Code (JCasC)?

**Answer:**
JCasC (Jenkins Configuration as Code plugin) allows defining the entire Jenkins configuration in a YAML file — tools, credentials, plugins, security settings — enabling reproducible setups.

**Example `jenkins.yaml`:**
```yaml
jenkins:
  systemMessage: "Jenkins configured by JCasC"
  securityRealm:
    local:
      allowsSignup: false
      users:
        - id: admin
          password: "${ADMIN_PASSWORD}"
  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: admin
            permissions:
              - Overall/Administer

credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              id: github-creds
              username: myuser
              password: "${GITHUB_TOKEN}"
```

---

### Q26. How do you implement approval gates in Jenkins?

**Answer:**
```groovy
stage('Deploy to Production') {
    steps {
        script {
            // Pause and wait for manual approval
            def approval = input(
                message: 'Approve deployment to PRODUCTION?',
                ok: 'Deploy',
                submitter: 'devops-team,release-manager',
                parameters: [
                    choice(name: 'CONFIRM', choices: ['Yes', 'No'], description: 'Confirm?')
                ]
            )
            if (approval == 'No') {
                error('Deployment aborted by user')
            }
        }
        sh './deploy-production.sh'
    }
    timeout(time: 30, unit: 'MINUTES')   // Auto-abort if no response
}
```

---

### Q27. How do you set up Jenkins backup and restore?

**Answer:**

**Backup using ThinBackup plugin:**
- Install ThinBackup plugin
- Set backup directory and schedule

**Manual backup (recommended for production):**
```bash
# Backup Jenkins home
tar -czf jenkins-backup-$(date +%Y%m%d).tar.gz /var/lib/jenkins

# Exclude workspace to save space
tar -czf jenkins-backup.tar.gz \
  --exclude='/var/lib/jenkins/workspace' \
  /var/lib/jenkins
```

**Restore:**
```bash
systemctl stop jenkins
tar -xzf jenkins-backup.tar.gz -C /
systemctl start jenkins
```

**Key files to backup:**
- `config.xml` (global config)
- `jobs/` (all job configs and history)
- `plugins/` (installed plugins)
- `secrets/` (encrypted credentials)

---

## 9. Troubleshooting — Common Issues & Fixes

---

### T1. Jenkins build is stuck / hanging — how do you troubleshoot?

**Answer:**

**Symptoms:** Build shows `Running` indefinitely, no progress in console.

**Root causes and fixes:**

| Cause | Fix |
|-------|-----|
| Waiting for user input with no timeout | Add `timeout` wrapper around `input` step |
| Deadlock in scripted pipeline | Review `lock()` usage, avoid nested locks |
| Infinite loop in shell script | Add `timeout` to the stage or shell command |
| Hung agent/node | Restart the agent; check agent logs |

```groovy
// Always wrap input with timeout
timeout(time: 15, unit: 'MINUTES') {
    input 'Approve deployment?'
}

// Set global pipeline timeout
options {
    timeout(time: 1, unit: 'HOURS')
}
```

**Quick fix:** Abort the build → Manage Jenkins → Thread Dump to diagnose.

---

### T2. "Out of memory" / Jenkins is slow — how do you fix it?

**Answer:**

**Check current heap:**
```bash
ps aux | grep jenkins | grep -o 'Xmx[^ ]*'
```

**Increase JVM heap size:**
```bash
# /etc/default/jenkins (Debian/Ubuntu)
JAVA_ARGS="-Xms512m -Xmx4g -XX:+UseG1GC"

# /etc/sysconfig/jenkins (RHEL/CentOS)
JENKINS_JAVA_OPTIONS="-Xms512m -Xmx4g -XX:+UseG1GC"
```

**Additional fixes:**
- Set `buildDiscarder` to discard old builds and limit stored artifacts
- Move workspace to a faster disk
- Offload builds to agents; master should not run builds
- Regularly clean up old builds: Manage Jenkins → System → Build History

```groovy
options {
    buildDiscarder(logRotator(
        numToKeepStr: '10',
        artifactNumToKeepStr: '5',
        daysToKeepStr: '30'
    ))
}
```

---

### T3. Build failing with "No space left on device" — how to fix?

**Answer:**

```bash
# Check disk usage
df -h
du -sh /var/lib/jenkins/*

# Clean workspace
find /var/lib/jenkins/workspace -type d -mtime +30 -exec rm -rf {} +

# Clean old builds via Jenkins script console
# Manage Jenkins → Script Console
Jenkins.instance.getAllItems(Job.class).each { job ->
    job.builds.findAll { it.number < job.lastBuild.number - 10 }.each { it.delete() }
}
```

**Preventive measures:**
- Use `cleanWs()` in post block of each pipeline
- Enable workspace cleanup plugin
- Set build discard policies on every job
- Mount `/var/lib/jenkins` on a separate large volume

---

### T4. Agent goes offline during a build — how do you troubleshoot?

**Answer:**

**Check agent logs:**
```bash
# On the agent machine
tail -f /var/log/jenkins/agent.log

# Or on master
# Manage Jenkins → Nodes → <agent-name> → Log
```

**Common causes and fixes:**

| Cause | Fix |
|-------|-----|
| Agent JVM crashed (OOM) | Increase heap on agent: `-Xmx2g` |
| SSH connection timeout | Increase `ServerAliveInterval` in SSH config |
| Agent disconnected due to network | Use JNLP with auto-reconnect |
| Disk full on agent | Clean up workspace on agent |
| Firewall blocked connection | Check ports 50000 (JNLP) or 22 (SSH) |

**Auto-restart agent on failure (JNLP):**
```bash
# Run as a systemd service on agent
[Service]
ExecStart=/usr/bin/java -jar agent.jar -jnlpUrl http://master/computer/agent1/jenkins-agent.jnlp
Restart=always
RestartSec=10
```

---

### T5. Jenkins pipeline stage shows "script not approved" error — how to fix?

**Answer:**

**Error:** `org.jenkinsci.plugins.scriptsecurity.sandbox.RejectedAccessException: Scripts not permitted to use method...`

**Cause:** Jenkins Groovy Sandbox blocks certain methods for security.

**Fixes:**
1. Approve the script: Manage Jenkins → In-process Script Approval → Approve
2. Use approved alternatives in your Jenkinsfile
3. Move restricted logic into a Shared Library (libraries run outside sandbox if marked `@NonCPS`)

```groovy
// Use @NonCPS for non-serializable operations
@NonCPS
def getVersion(text) {
    def matcher = text =~ /version=(\S+)/
    return matcher ? matcher[0][1] : 'unknown'
}
```

---

### T6. Jenkins webhook is not triggering builds — how to troubleshoot?

**Answer:**

**Checklist:**

```
1. Verify webhook is configured in GitHub/GitLab
   → Repo Settings → Webhooks → Check "Recent Deliveries"
   → Look for 200 OK; if 403/404, check Jenkins URL and CSRF token

2. Check Jenkins is reachable from the internet
   → curl -I http://<jenkins-url>/github-webhook/

3. Verify plugin is installed
   → GitHub Integration plugin (for GitHub)

4. Check Jenkins job build trigger
   → Job → Configure → Build Triggers → ✅ GitHub hook trigger for GITScm polling

5. Check Jenkins security settings
   → Manage Jenkins → Configure Global Security
   → Disable CSRF for webhooks OR use a crumb
```

**Test webhook manually:**
```bash
curl -X POST http://<jenkins-url>/github-webhook/ \
  -H "Content-Type: application/json" \
  -H "X-GitHub-Event: push" \
  -d @payload.json
```

---

### T7. "Failed to connect to repository" / Git checkout error — how to fix?

**Answer:**

**Common error messages:**
- `Error performing git command`
- `stderr: Permission denied (publickey)`
- `SSL certificate problem: unable to get local issuer certificate`

**Fixes:**

```bash
# SSH key issue — test connectivity on Jenkins server
ssh -i /var/lib/jenkins/.ssh/id_rsa git@github.com

# Fix permissions
chmod 600 /var/lib/jenkins/.ssh/id_rsa
chown jenkins:jenkins /var/lib/jenkins/.ssh/id_rsa

# SSL issue — bypass (dev only, not production!)
git config --global http.sslVerify false

# Add GitHub to known_hosts
ssh-keyscan github.com >> /var/lib/jenkins/.ssh/known_hosts
```

**In Pipeline:**
```groovy
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    userRemoteConfigs: [[
        url: 'git@github.com:org/repo.git',
        credentialsId: 'github-ssh-key'
    ]]
])
```

---

### T8. Jenkins builds are queuing but not running — how to fix?

**Answer:**

**Causes:**
- All executors are busy
- No agents with matching label available
- Agent is offline

**Diagnostic steps:**
```
1. Check build queue
   → Jenkins Dashboard → Build Queue (left sidebar)

2. Check executor count
   → Manage Jenkins → Nodes → Set executors (default is 2 on master)

3. Check agent availability
   → Manage Jenkins → Nodes → Verify agents are online

4. Check label mismatch
   → Compare job's required label vs agent's configured label

5. Check "Quiet Period" setting on the job
```

**Quick fix — increase executors (not recommended for master):**
```
Manage Jenkins → Nodes → master → Configure → Number of executors: 5
```

**Right fix — add more agents or use dynamic Kubernetes agents.**

---

### T9. How do you debug a Jenkins pipeline locally before pushing?

**Answer:**

**Option 1: Use Jenkins Pipeline Linter (Declarative)**
```bash
# Using Jenkins API
curl --user admin:token -X POST \
  -F "jenkinsfile=<Jenkinsfile" \
  http://localhost:8080/pipeline-model-converter/validate
```

**Option 2: Use the Replay feature**
- Open a completed build → Replay → Edit Jenkinsfile → Run

**Option 3: Local testing with `act` (GitHub Actions-style)**
```bash
# For simulating Jenkins locally — use Docker
docker run --rm -p 8080:8080 jenkins/jenkins:lts
```

**Option 4: Pipeline Syntax Generator in UI**
- Jenkins → New Item → Pipeline → Pipeline Syntax → Generate declarative snippet

---

### T10. Jenkins shows "Disk Usage is Critical" warning — how to remediate?

**Answer:**

```groovy
// Script Console — delete all old builds
Jenkins.instance.getAllItems(AbstractProject.class).each { project ->
    project.builds.each { build ->
        if (build.getTimestamp().before(Calendar.getInstance() - 30)) {
            build.delete()
        }
    }
}

// Cleanup workspaces
Jenkins.instance.getAllItems(Job.class).each { job ->
    job.builds.each { build ->
        build.deleteArtifacts()
    }
}
```

**System-level cleanup:**
```bash
# Remove Docker images on agents
docker system prune -af

# Remove old log files
find /var/log/jenkins -name "*.log.*" -mtime +7 -delete

# Remove old workspace
find /var/lib/jenkins/workspace -maxdepth 1 -mtime +14 -exec rm -rf {} +
```

---

## 🎯 Quick Reference Cheat Sheet

| Topic | Key Command / Concept |
|-------|----------------------|
| Restart Jenkins | `http://jenkins-url/restart` or `systemctl restart jenkins` |
| View logs | `tail -f /var/log/jenkins/jenkins.log` |
| Install plugin via CLI | `java -jar jenkins-cli.jar -s http://url install-plugin <name>` |
| List all jobs | Jenkins Script Console: `Jenkins.instance.getAllItems(Job.class)*.name` |
| Reload config | `http://jenkins-url/reload` |
| Pipeline timeout | `options { timeout(time: 30, unit: 'MINUTES') }` |
| Workspace cleanup | `post { always { cleanWs() } }` |
| Skip stage on failure | `stage { when { expression { currentBuild.result != 'FAILURE' } } }` |
| Archive artifacts | `archiveArtifacts artifacts: '**/target/*.jar'` |
| Publish test results | `junit '**/target/surefire-reports/*.xml'` |

---

## 📌 Tips to Crack Jenkins Interviews

1. **Know the difference** between Declarative and Scripted pipelines — interviewers love this.
2. **Practice Jenkinsfile writing** from scratch — build, test, dockerize, deploy stages.
3. **Understand agent types** — especially Kubernetes dynamic agents (very common in cloud shops).
4. **Be ready with troubleshooting stories** — "How did you fix a broken pipeline?" scenarios.
5. **Know at least 10 plugins** by name and purpose.
6. **Understand Jenkins + Git + Docker + Kubernetes** integration end-to-end.
7. **Security questions** are increasingly common — credentials, RBAC, HTTPS, CSRF.

---

*Good luck with your DevOps interview! 🚀*
