# 🚀 Jenkins Interview Questions & Answers
### Complete Preparation Guide — Top 30 Questions

---

## 🟢 BASIC LEVEL

---

### Q1. What is Jenkins? Why is it used?

**Answer:**
Jenkins is an open-source **Continuous Integration / Continuous Delivery (CI/CD)** automation server written in Java. It helps automate the parts of software development related to building, testing, and deploying — facilitating CI/CD pipelines.

**Why Jenkins?**
- Free & open-source with a huge plugin ecosystem (1800+ plugins)
- Supports any language, SCM tool, and deployment target
- Distributed builds via master-agent architecture
- Large community support

```
Developer pushes code
       ↓
Jenkins detects change (via webhook/poll)
       ↓
Build → Test → Package → Deploy
       ↓
Notify team of results
```

---

### Q2. What is Continuous Integration (CI) and Continuous Delivery (CD)?

**Answer:**

| Term | Definition |
|---|---|
| **CI** | Automatically build and test code every time a developer pushes changes |
| **CD (Delivery)** | CI + automatically prepare the release for deployment (manual approval to deploy) |
| **CD (Deployment)** | CI + automatically deploy to production without manual intervention |

**Benefits of CI/CD:**
- Detect bugs early
- Faster release cycles
- Reduced manual effort
- Consistent, repeatable deployments

---

### Q3. What is a Jenkins Pipeline? What are the types?

**Answer:**

A **Jenkins Pipeline** is a suite of plugins that supports implementing and integrating CI/CD pipelines into Jenkins using code (as a `Jenkinsfile`).

**Two types:**

| Type | Description |
|---|---|
| **Declarative Pipeline** | Newer, simpler, opinionated syntax. Recommended for most use cases. |
| **Scripted Pipeline** | Older, Groovy-based, more flexible and powerful but complex. |

```groovy
// Declarative Pipeline (Recommended)
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
```

---

### Q4. What is a Jenkinsfile?

**Answer:**

A **Jenkinsfile** is a text file that contains the definition of a Jenkins Pipeline and is checked into source control (Git). This is called **"Pipeline as Code."**

**Benefits:**
- Pipeline reviewed and versioned with the application code
- Audit trail for the pipeline itself
- Can be reused across branches
- Single source of truth

```groovy
// Jenkinsfile lives at root of your repo
// Jenkinsfile (Declarative)
pipeline {
    agent any
    environment {
        APP_NAME = 'my-app'
        VERSION   = '1.0.0'
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build') {
            steps { sh 'mvn clean package -DskipTests' }
        }
        stage('Test') {
            steps { sh 'mvn test' }
            post {
                always { junit 'target/surefire-reports/*.xml' }
            }
        }
    }
}
```

---

### Q5. What is a Jenkins Job / Project? What are the types?

**Answer:**

A **Jenkins Job** (also called a Project) is a runnable task configured in Jenkins. 

**Types of Jenkins Jobs:**

| Job Type | Use Case |
|---|---|
| **Freestyle Project** | Simple, GUI-configured builds. Good for beginners. |
| **Pipeline** | Code-based pipelines using Jenkinsfile |
| **Multibranch Pipeline** | Automatically creates pipelines for each branch in a repo |
| **Folder** | Organizes jobs into groups |
| **Multi-configuration (Matrix)** | Run same job across multiple configurations (OS, JDK version) |
| **Organization Folder** | Scans all repos in a GitHub Org / Bitbucket Project |

---

### Q6. What is a Jenkins Agent / Node? What is the difference between Master and Agent?

**Answer:**

| Component | Role |
|---|---|
| **Master (Controller)** | Central Jenkins server. Schedules jobs, monitors agents, serves the UI, stores configs. |
| **Agent (Node/Slave)** | Machine that actually executes the build steps delegated by the master. |

```groovy
// Assigning a pipeline to a specific agent
pipeline {
    agent { label 'linux-agent' }  // run on node with this label
    stages { ... }
}

// Different stages on different agents
pipeline {
    agent none
    stages {
        stage('Build') {
            agent { label 'maven-agent' }
            steps { sh 'mvn package' }
        }
        stage('Deploy') {
            agent { label 'docker-agent' }
            steps { sh 'docker build .' }
        }
    }
}
```

> 💡 **Best Practice:** Keep Master lean — don't run builds on master in production.

---

### Q7. What are Jenkins Plugins? Name some important ones.

**Answer:**

Plugins extend Jenkins functionality. Jenkins has **1800+** plugins available.

**Must-know plugins:**

| Plugin | Purpose |
|---|---|
| **Git Plugin** | Integrates with Git repositories |
| **Pipeline** | Enables Jenkinsfile pipelines |
| **Blue Ocean** | Modern UI for pipelines |
| **Maven Integration** | Maven build support |
| **Docker Pipeline** | Build/run Docker containers in pipelines |
| **Kubernetes** | Run agents as Kubernetes pods |
| **Credentials Binding** | Inject secrets/credentials into builds |
| **Email Extension** | Send rich email notifications |
| **SonarQube Scanner** | Code quality analysis integration |
| **Slack Notification** | Send build notifications to Slack |
| **JUnit** | Publish test results |
| **Artifactory / Nexus** | Artifact repository integration |

```bash
# Plugins can be managed via:
# Jenkins UI → Manage Jenkins → Manage Plugins
# Or via Jenkins CLI / Configuration as Code
```

---

### Q8. What are Jenkins Build Triggers? How can you trigger a Jenkins build?

**Answer:**

**Ways to trigger a Jenkins build:**

| Trigger | Description |
|---|---|
| **SCM Polling** | Jenkins periodically checks SCM for changes |
| **Webhook (Push trigger)** | GitHub/GitLab sends a webhook to Jenkins on push |
| **Scheduled (Cron)** | Time-based trigger using cron syntax |
| **Manual** | User clicks "Build Now" in UI |
| **Upstream job** | Another job triggers this one on completion |
| **Remote trigger** | Trigger via URL/API call |
| **Parameterized** | Trigger with input parameters |

```groovy
pipeline {
    agent any
    triggers {
        // Poll SCM every 5 minutes
        pollSCM('H/5 * * * *')

        // Schedule: run daily at midnight
        cron('0 0 * * *')

        // Webhook is configured on GitHub side, not here
    }
    stages { ... }
}
```

---

### Q9. What is the Jenkins Build Lifecycle / Post section?

**Answer:**

The `post` section defines actions to run **after** pipeline stages complete, based on the build result.

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { sh 'mvn package' }
        }
    }
    post {
        always {
            echo 'This always runs (cleanup, reports)'
            cleanWs()  // clean workspace
        }
        success {
            echo 'Build succeeded!'
            slackSend color: 'good', message: "✅ Build passed: ${env.JOB_NAME}"
        }
        failure {
            echo 'Build FAILED!'
            mail to: 'team@company.com',
                 subject: "❌ FAILED: ${env.JOB_NAME}",
                 body: "Check: ${env.BUILD_URL}"
        }
        unstable {
            echo 'Build is unstable (test failures)'
        }
        changed {
            echo 'Build status changed from last run'
        }
    }
}
```

---

### Q10. What are Environment Variables in Jenkins?

**Answer:**

Jenkins provides built-in environment variables and allows custom ones. They are accessible in `sh` steps and Groovy code.

**Important built-in variables:**

| Variable | Value |
|---|---|
| `BUILD_NUMBER` | Current build number |
| `BUILD_URL` | URL of this build |
| `JOB_NAME` | Name of the job |
| `WORKSPACE` | Path to the build workspace |
| `GIT_BRANCH` | Current Git branch |
| `GIT_COMMIT` | Current commit SHA |
| `JENKINS_URL` | URL of Jenkins server |

```groovy
pipeline {
    agent any
    environment {
        // Custom environment variables
        APP_ENV  = 'production'
        DB_URL   = credentials('db-connection-string')  // from credentials store
    }
    stages {
        stage('Info') {
            steps {
                echo "Job: ${env.JOB_NAME}"
                echo "Build: ${env.BUILD_NUMBER}"
                echo "Branch: ${env.GIT_BRANCH}"
                sh "echo Running on workspace: $WORKSPACE"
            }
        }
    }
}
```

---

## 🟡 INTERMEDIATE LEVEL

---

### Q11. How do you manage credentials/secrets in Jenkins?

**Answer:**

Jenkins has a built-in **Credentials Store** to securely manage secrets. Never hardcode secrets in Jenkinsfile.

**Credential Types:**
- Username + Password
- Secret Text (API tokens)
- SSH Private Key
- Certificate
- Secret File

```groovy
pipeline {
    agent any
    environment {
        // Bind credentials to env vars
        DOCKER_CREDS = credentials('dockerhub-credentials')  // username:password
        API_TOKEN    = credentials('my-api-token')           // secret text
    }
    stages {
        stage('Docker Login') {
            steps {
                sh 'echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin'
            }
        }
        stage('Use Token') {
            steps {
                withCredentials([string(credentialsId: 'my-api-token', variable: 'TOKEN')]) {
                    sh 'curl -H "Authorization: Bearer $TOKEN" https://api.example.com'
                }
            }
        }
    }
}
```

> 💡 Jenkins automatically masks credential values in build logs.

---

### Q12. What is a Multibranch Pipeline? Why is it useful?

**Answer:**

A **Multibranch Pipeline** automatically discovers branches in a repository and creates a separate pipeline for each branch that contains a `Jenkinsfile`.

**Benefits:**
- Automatic pipeline creation/deletion as branches are created/deleted
- Each branch gets its own build history
- Perfect for feature branch workflows and PRs
- Branch-specific behavior using `when` conditions

```groovy
// Jenkinsfile with branch-specific logic
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { sh 'mvn package' }
        }
        stage('Deploy to Staging') {
            when { branch 'develop' }
            steps { echo 'Deploying to staging...' }
        }
        stage('Deploy to Production') {
            when { branch 'main' }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                echo 'Deploying to production...'
            }
        }
    }
}
```

---

### Q13. What is the `when` directive in Jenkins Pipeline?

**Answer:**

The `when` directive controls **whether a stage should be executed** based on conditions.

```groovy
pipeline {
    agent any
    stages {
        // Run only on main branch
        stage('Deploy Prod') {
            when { branch 'main' }
            steps { echo 'Deploying to prod' }
        }

        // Run only when environment variable matches
        stage('Integration Test') {
            when { environment name: 'RUN_INTEGRATION', value: 'true' }
            steps { sh 'mvn verify' }
        }

        // Multiple conditions (AND logic)
        stage('Release') {
            when {
                allOf {
                    branch 'main'
                    not { changeRequest() }   // not a PR
                }
            }
            steps { echo 'Releasing...' }
        }

        // Any condition matches (OR logic)
        stage('Notify') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps { echo 'Notifying team' }
        }

        // Based on file changes
        stage('Frontend Build') {
            when { changeset '**/frontend/**' }
            steps { sh 'npm run build' }
        }
    }
}
```

---

### Q14. What are Shared Libraries in Jenkins?

**Answer:**

A **Shared Library** is a way to share common pipeline code across multiple Jenkinsfiles in different repositories. Stored in a separate Git repo and loaded into pipelines.

**Structure of a Shared Library repo:**
```
(root)
├── vars/
│   ├── buildMaven.groovy       ← global functions
│   └── deployToKubernetes.groovy
├── src/
│   └── org/example/
│       └── GitUtils.groovy     ← utility classes
└── resources/
    └── templates/
        └── email-template.html
```

```groovy
// vars/buildMaven.groovy
def call(String version = '3.8') {
    sh "mvn -v ${version} clean package"
    junit 'target/surefire-reports/*.xml'
}

// Using the shared library in Jenkinsfile
@Library('my-shared-library@main') _

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                buildMaven()         // call shared library function
            }
        }
        stage('Deploy') {
            steps {
                deployToKubernetes(env: 'staging', image: 'myapp:latest')
            }
        }
    }
}
```

> 💡 Configure shared libraries in: **Manage Jenkins → Configure System → Global Pipeline Libraries**

---

### Q15. What is Jenkins Blue Ocean?

**Answer:**

**Blue Ocean** is a modern UI plugin for Jenkins that provides a better visual experience for pipelines.

**Features:**
- Visual pipeline editor (no need to write Jenkinsfile manually)
- Clear visualization of pipeline stages
- Better visualization of parallel stages
- Integrated GitHub/Bitbucket PR view
- Per-step log viewing

```bash
# Install Blue Ocean plugin
# Manage Jenkins → Manage Plugins → Available → Search "Blue Ocean"

# Access Blue Ocean
# http://your-jenkins/blue
```

---

### Q16. How do you implement Parallel stages in Jenkins Pipeline?

**Answer:**

`parallel` lets you run multiple stages **simultaneously**, reducing total build time.

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { sh 'mvn package -DskipTests' }
        }

        stage('Run Tests in Parallel') {
            parallel {
                stage('Unit Tests') {
                    agent { label 'agent-1' }
                    steps { sh 'mvn test -Dtest=UnitTests' }
                }
                stage('Integration Tests') {
                    agent { label 'agent-2' }
                    steps { sh 'mvn test -Dtest=IntegrationTests' }
                }
                stage('Security Scan') {
                    agent { label 'agent-3' }
                    steps { sh 'snyk test' }
                }
            }
        }

        stage('Deploy') {
            steps { echo 'All tests passed, deploying...' }
        }
    }
}
```

> 💡 **`failFast: true`** — Abort remaining parallel stages if any one fails:
```groovy
parallel(failFast: true, stages: { ... })
```

---

### Q17. What is the `input` step in Jenkins? How do you implement manual approval?

**Answer:**

The `input` step **pauses the pipeline** and waits for a human to approve before proceeding. Essential for production deployments.

```groovy
pipeline {
    agent any
    stages {
        stage('Build & Test') {
            steps { sh 'mvn clean verify' }
        }

        stage('Approval Gate') {
            steps {
                script {
                    def userInput = input(
                        id: 'deploy-approval',
                        message: 'Deploy to Production?',
                        submitter: 'admin,ops-team',   // only these users can approve
                        parameters: [
                            choice(
                                name: 'TARGET_ENV',
                                choices: ['prod-us', 'prod-eu'],
                                description: 'Select deployment region'
                            ),
                            booleanParam(
                                name: 'RUN_SMOKE_TEST',
                                defaultValue: true,
                                description: 'Run smoke tests after deploy?'
                            )
                        ]
                    )
                    echo "Deploying to: ${userInput.TARGET_ENV}"
                }
            }
        }

        stage('Deploy to Production') {
            steps { echo 'Deploying...' }
        }
    }
}
```

---

### Q18. What is Jenkins Configuration as Code (JCasC)?

**Answer:**

**JCasC** allows you to define and manage Jenkins configuration in a **YAML file** instead of clicking through the UI. The config can be stored in Git.

```yaml
# jenkins.yaml — JCasC configuration file
jenkins:
  systemMessage: "Jenkins configured via JCasC"
  numExecutors: 5
  
  securityRealm:
    local:
      allowsSignup: false
      users:
        - id: "admin"
          password: "${ADMIN_PASSWORD}"

  authorizationStrategy:
    loggedInUsersCanDoAnything:
      allowAnonymousRead: false

  nodes:
    - permanent:
        name: "build-agent-1"
        remoteFS: "/home/jenkins"
        labelString: "linux docker"
        launcher:
          ssh:
            host: "192.168.1.10"
            credentialsId: "agent-ssh-key"

credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              id: "dockerhub"
              username: "myuser"
              password: "${DOCKERHUB_TOKEN}"
```

> 💡 Install the **"Configuration as Code"** plugin and point it to your YAML via environment variable `CASC_JENKINS_CONFIG`.

---

### Q19. How do you integrate Jenkins with Docker?

**Answer:**

Docker + Jenkins enables consistent, isolated build environments.

**Two approaches:**

**1. Build Docker images in pipeline:**
```groovy
pipeline {
    agent any
    environment {
        IMAGE_NAME = "myapp:${env.BUILD_NUMBER}"
    }
    stages {
        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }
        stage('Push to Registry') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker push ${IMAGE_NAME}"
                }
            }
        }
    }
    post {
        always { sh "docker rmi ${IMAGE_NAME} || true" }
    }
}
```

**2. Run build inside a Docker container (agent):**
```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.9-openjdk-17'
            args '-v $HOME/.m2:/root/.m2'  // cache Maven deps
        }
    }
    stages {
        stage('Build') {
            steps { sh 'mvn clean package' }
        }
    }
}
```

---

### Q20. What are Jenkins Parameters? How do you create a parameterized build?

**Answer:**

Parameters allow users to **pass input values** to a build at trigger time, making jobs reusable.

```groovy
pipeline {
    agent any
    parameters {
        string(
            name: 'APP_VERSION',
            defaultValue: '1.0.0',
            description: 'Version to deploy'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'production'],
            description: 'Target deployment environment'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Skip test execution?'
        )
        password(
            name: 'DEPLOY_TOKEN',
            defaultValue: '',
            description: 'Deployment API token'
        )
    }
    stages {
        stage('Deploy') {
            steps {
                echo "Deploying version ${params.APP_VERSION} to ${params.ENVIRONMENT}"
                script {
                    if (!params.SKIP_TESTS) {
                        sh 'mvn test'
                    }
                }
            }
        }
    }
}
```

---

## 🔴 ADVANCED LEVEL

---

### Q21. How does Jenkins integrate with Kubernetes?

**Answer:**

The **Kubernetes plugin** dynamically provisions Jenkins agents as **Kubernetes Pods** — created on demand and destroyed after the build.

```groovy
pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: maven
                    image: maven:3.9-openjdk-17
                    command: ['sleep', '99d']
                  - name: docker
                    image: docker:24-dind
                    securityContext:
                      privileged: true
            '''
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
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp:latest .'
                }
            }
        }
    }
}
```

**Benefits:**
- Auto-scaling agents (no idle agents)
- Resource isolation per build
- Clean environment every time
- Cost-efficient on cloud

---

### Q22. How do you implement a complete CI/CD pipeline in Jenkins?

**Answer:**

```groovy
pipeline {
    agent { label 'linux' }

    environment {
        DOCKER_REGISTRY = 'registry.company.com'
        IMAGE_NAME      = "${DOCKER_REGISTRY}/myapp:${env.BUILD_NUMBER}"
        KUBE_NAMESPACE  = 'production'
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build') {
            steps { sh 'mvn clean package -DskipTests' }
        }

        stage('Unit Tests') {
            steps { sh 'mvn test' }
            post { always { junit 'target/surefire-reports/*.xml' } }
        }

        stage('Code Quality') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
                withCredentials([usernamePassword(credentialsId: 'registry-creds',
                        usernameVariable: 'U', passwordVariable: 'P')]) {
                    sh "echo $P | docker login ${DOCKER_REGISTRY} -u $U --password-stdin"
                    sh "docker push ${IMAGE_NAME}"
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh "kubectl set image deployment/myapp myapp=${IMAGE_NAME} -n staging"
                sh "kubectl rollout status deployment/myapp -n staging"
            }
        }

        stage('Smoke Tests') {
            steps { sh 'curl -f https://staging.company.com/health' }
        }

        stage('Approve Production Deploy') {
            steps {
                input message: '🚀 Deploy to Production?', submitter: 'ops-team'
            }
        }

        stage('Deploy to Production') {
            steps {
                sh "kubectl set image deployment/myapp myapp=${IMAGE_NAME} -n ${KUBE_NAMESPACE}"
                sh "kubectl rollout status deployment/myapp -n ${KUBE_NAMESPACE}"
            }
        }
    }

    post {
        success {
            slackSend color: 'good',
                      message: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} deployed successfully!"
        }
        failure {
            slackSend color: 'danger',
                      message: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} FAILED! ${env.BUILD_URL}"
        }
        always { cleanWs() }
    }
}
```

---

### Q23. What is SonarQube integration in Jenkins?

**Answer:**

SonarQube performs **static code analysis** to detect bugs, vulnerabilities, and code smells. Jenkins integrates via the SonarQube Scanner plugin.

```groovy
pipeline {
    agent any
    stages {
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {  // configured in Manage Jenkins
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=my-project \
                          -Dsonar.projectName="My Project" \
                          -Dsonar.java.coveragePlugin=jacoco \
                          -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    // Abort pipeline if quality gate fails
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
```

**Quality Gate criteria examples:**
- Code coverage > 80%
- No new critical bugs
- No new security vulnerabilities
- Technical debt < 5%

---

### Q24. What is Jenkins DSL (Job DSL Plugin)?

**Answer:**

**Job DSL** lets you define Jenkins jobs **programmatically using Groovy** instead of configuring them through the UI. Enables "Jenkins jobs as code."

```groovy
// seed-job.groovy — creates other jobs programmatically
pipelineJob('my-app-pipeline') {
    description('CI/CD pipeline for my-app')

    triggers {
        githubPush()
        scm('H/5 * * * *')   // poll every 5 min as fallback
    }

    definition {
        cpsScm {
            scm {
                git {
                    remote {
                        url('https://github.com/company/my-app.git')
                        credentials('github-token')
                    }
                    branch('*/main')
                }
            }
            scriptPath('Jenkinsfile')
        }
    }
}

// Create a folder and multiple jobs
folder('microservices') {
    description('All microservice pipelines')
}

['user-service', 'order-service', 'payment-service'].each { service ->
    pipelineJob("microservices/${service}") {
        definition {
            cpsScm {
                scm {
                    git { remote { url("https://github.com/company/${service}.git") } }
                }
                scriptPath('Jenkinsfile')
            }
        }
    }
}
```

---

### Q25. How do you handle Jenkins pipeline failures and implement retry logic?

**Answer:**

```groovy
pipeline {
    agent any
    stages {
        stage('Flaky Network Call') {
            steps {
                // Retry up to 3 times on failure
                retry(3) {
                    sh 'curl https://unreliable-api.example.com/data'
                }
            }
        }

        stage('Build with Timeout') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Deploy with Error Handling') {
            steps {
                script {
                    try {
                        sh './deploy.sh'
                    } catch (Exception e) {
                        echo "Deploy failed: ${e.getMessage()}"
                        // Attempt rollback
                        sh './rollback.sh'
                        // Re-throw to fail the build
                        throw e
                    }
                }
            }
        }

        stage('Optional Step') {
            steps {
                // Don't fail the build if this step fails
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    sh './optional-check.sh'
                }
            }
        }
    }
}
```

---

### Q26. What is Jenkins X? How does it differ from Jenkins?

**Answer:**

| Feature | Jenkins | Jenkins X |
|---|---|---|
| **Focus** | General CI/CD for any environment | Kubernetes-native CI/CD |
| **Configuration** | Jenkinsfile + manual setup | Automated, convention-over-config |
| **Preview Environments** | Manual setup | Built-in per PR |
| **GitOps** | Plugin-based | First-class support |
| **Complexity** | High flexibility, complex config | Opinionated, easier to start |
| **Agent Management** | Manual or Kubernetes plugin | Always Kubernetes pods |

> Jenkins X is purpose-built for cloud-native, Kubernetes-based workflows with built-in GitOps.

---

### Q27. How do you back up and restore Jenkins?

**Answer:**

**What to back up:**
- `$JENKINS_HOME` directory (jobs, configs, plugins, credentials)
- Key subdirectories: `jobs/`, `config.xml`, `credentials.xml`, `plugins/`, `users/`

```bash
# Method 1: Backup JENKINS_HOME
tar -czf jenkins-backup-$(date +%Y%m%d).tar.gz $JENKINS_HOME

# Method 2: Using ThinBackup Plugin (UI-based scheduler)
# Manage Jenkins → ThinBackup → Settings → Schedule backup

# Method 3: Script to back up only configs (not workspace)
rsync -av --exclude='workspace' \
          --exclude='builds' \
          $JENKINS_HOME/ \
          /backup/jenkins/

# Restore: stop Jenkins, restore files, restart
systemctl stop jenkins
tar -xzf jenkins-backup-20240101.tar.gz -C /
systemctl start jenkins
```

> 💡 Use **JCasC** + **Job DSL** = most of your Jenkins config is already in Git!

---

### Q28. What is the difference between Declarative and Scripted Pipeline? When to use which?

**Answer:**

| Aspect | Declarative | Scripted |
|---|---|---|
| **Syntax** | Structured, predefined blocks | Full Groovy, flexible |
| **Learning Curve** | Lower | Higher |
| **Error Messages** | Better, clearer | Sometimes cryptic |
| **Validation** | At start of run | At runtime |
| **Restart from Stage** | ✅ Supported | ❌ Not supported |
| **`when` directive** | ✅ Built-in | Manual `if` statements |
| **Use When** | Most CI/CD pipelines | Complex logic, loops, dynamic stages |

```groovy
// Scripted Pipeline example (more flexible)
node('linux') {
    def mvnHome = tool 'Maven-3.9'
    
    stage('Checkout') {
        checkout scm
    }
    
    def services = ['user-service', 'order-service', 'payment-service']
    
    // Dynamic parallel stages (hard in Declarative)
    def parallelStages = services.collectEntries { service ->
        ["Build ${service}": {
            stage("Build ${service}") {
                dir(service) {
                    sh "${mvnHome}/bin/mvn package"
                }
            }
        }]
    }
    
    stage('Build All Services') {
        parallel parallelStages
    }
}
```

---

### Q29. How do you secure Jenkins?

**Answer:**

**Key security practices:**

```groovy
// 1. Use credentials store — never hardcode secrets
withCredentials([string(credentialsId: 'api-token', variable: 'TOKEN')]) {
    sh 'curl -H "Authorization: Bearer $TOKEN" ...'
}
```

**Security Checklist:**

| Area | Best Practice |
|---|---|
| **Authentication** | Use LDAP/SSO, disable anonymous access |
| **Authorization** | Role-Based Access Control (RBAC) via Matrix plugin |
| **Secrets** | Credentials plugin, HashiCorp Vault integration |
| **Network** | Put Jenkins behind reverse proxy (Nginx), use HTTPS |
| **Agents** | Run agents with minimal permissions |
| **Plugins** | Keep plugins updated, remove unused ones |
| **Audit** | Enable audit trail plugin |
| **Scripts** | Use Script Security plugin, approve Groovy scripts |
| **Updates** | Keep Jenkins core updated |
| **Backups** | Regular encrypted backups |

---

### Q30. What is a Jenkins Declarative Pipeline with complete real-world example?

**Answer:**

```groovy
pipeline {
    agent { label 'docker-agent' }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))  // keep last 10 builds
        timeout(time: 30, unit: 'MINUTES')              // global timeout
        disableConcurrentBuilds()                        // one build at a time
        timestamps()                                     // timestamps in logs
    }

    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'staging', 'prod'], description: 'Deploy target')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip tests?')
    }

    environment {
        APP_VERSION   = sh(script: 'cat version.txt', returnStdout: true).trim()
        IMAGE_TAG     = "${APP_VERSION}-${env.BUILD_NUMBER}"
        REGISTRY      = 'gcr.io/my-project'
        SONAR_PROJECT = 'my-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Building branch: ${env.GIT_BRANCH}, commit: ${env.GIT_COMMIT[0..7]}"
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Test') {
            when { not { params.SKIP_TESTS } }
            parallel {
                stage('Unit Tests') {
                    steps { sh 'mvn test' }
                    post { always { junit 'target/surefire-reports/*.xml' } }
                }
                stage('Code Coverage') {
                    steps { sh 'mvn jacoco:report' }
                    post { always { publishHTML([reportDir: 'target/site/jacoco', reportFiles: 'index.html', reportName: 'Coverage']) } }
                }
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') { sh 'mvn sonar:sonar' }
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh "docker build -t ${REGISTRY}/my-app:${IMAGE_TAG} ."
                withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'KEY')]) {
                    sh "gcloud auth activate-service-account --key-file=$KEY"
                    sh "docker push ${REGISTRY}/my-app:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (params.DEPLOY_ENV == 'prod') {
                        input message: "Deploy ${IMAGE_TAG} to PRODUCTION?", submitter: 'ops-team'
                    }
                }
                sh "helm upgrade --install my-app ./helm --set image.tag=${IMAGE_TAG} --namespace ${params.DEPLOY_ENV}"
                sh "kubectl rollout status deployment/my-app -n ${params.DEPLOY_ENV} --timeout=5m"
            }
        }
    }

    post {
        success {
            slackSend channel: '#deployments', color: 'good',
                      message: "✅ *${env.JOB_NAME}* v${APP_VERSION} deployed to *${params.DEPLOY_ENV}* by ${env.BUILD_USER}"
        }
        failure {
            slackSend channel: '#deployments', color: 'danger',
                      message: "❌ *${env.JOB_NAME}* FAILED at stage *${env.STAGE_NAME}* | <${env.BUILD_URL}|View Logs>"
            mail to: 'devops@company.com', subject: "FAILED: ${env.JOB_NAME}", body: "${env.BUILD_URL}"
        }
        always {
            cleanWs()
            sh "docker rmi ${REGISTRY}/my-app:${IMAGE_TAG} || true"
        }
    }
}
```

---

## 🎯 QUICK REVISION CHEATSHEET

| Concept | Key Point |
|---|---|
| **Jenkins** | Open-source CI/CD automation server |
| **Jenkinsfile** | Pipeline as Code — lives in your Git repo |
| **Declarative Pipeline** | Structured syntax, recommended, supports restart from stage |
| **Scripted Pipeline** | Full Groovy, flexible, for complex logic |
| **Master/Controller** | Schedules jobs, serves UI, stores config |
| **Agent/Node** | Executes builds |
| **Multibranch Pipeline** | Auto-creates pipelines per Git branch |
| **Shared Library** | Reusable pipeline code across repos |
| **Credentials Store** | Secure secret management — never hardcode secrets |
| **`post` section** | always / success / failure / unstable / changed |
| **`when` directive** | Conditional stage execution |
| **`parallel`** | Run stages simultaneously |
| **`input` step** | Manual approval gate |
| **`retry(n)`** | Retry a step n times on failure |
| **`timeout`** | Kill step/pipeline after time limit |
| **Blue Ocean** | Modern visual UI for pipelines |
| **JCasC** | Jenkins config as YAML in Git |
| **Job DSL** | Create jobs programmatically with Groovy |
| **Kubernetes Plugin** | Dynamic pod-based agents |
| **SonarQube** | Code quality gate integration |
| **`BUILD_NUMBER`** | Built-in env var — current build number |
| **`GIT_COMMIT`** | Built-in env var — current commit SHA |
| **`cleanWs()`** | Clean workspace after build |

---

> 💪 **Good luck with your interview tomorrow!**
> Remember: Interviewers love hearing **real-world scenarios** — always explain *why* you'd use a feature, not just *what* it does.
