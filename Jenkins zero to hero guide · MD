# Jenkins Zero to Hero - Complete Learning Guide

## Table of Contents
1. [Introduction to CI/CD](#introduction-to-cicd)
2. [What is Jenkins?](#what-is-jenkins)
3. [Jenkins Installation](#jenkins-installation)
4. [Jenkins Basics](#jenkins-basics)
5. [Your First Job](#your-first-job)
6. [Understanding Build Agents](#understanding-build-agents)
7. [Pipeline Fundamentals](#pipeline-fundamentals)
8. [Windows Jobs](#windows-jobs)
9. [PowerShell Jobs](#powershell-jobs)
10. [Linux Jobs](#linux-jobs)
11. [Advanced Concepts](#advanced-concepts)
12. [Real-World Projects](#real-world-projects)
13. [Best Practices](#best-practices)
14. [Troubleshooting](#troubleshooting)

---

## Introduction to CI/CD

### What is CI/CD?

**CI (Continuous Integration)**
- Developers merge code changes frequently (multiple times a day)
- Automated builds and tests run on every commit
- Catches bugs early and improves code quality

**CD (Continuous Delivery/Deployment)**
- Automated deployment to staging/production
- Reduces manual deployment errors
- Faster time to market

### Why Do We Need It?

**Without CI/CD:**
- Manual testing → slow and error-prone
- Integration happens at the end → big problems
- Deployment takes hours/days
- High risk of production failures

**With CI/CD:**
- Automated testing → fast and reliable
- Continuous integration → small, manageable changes
- Deployment in minutes
- Lower risk, faster feedback

---

## What is Jenkins?

### Overview
Jenkins is an open-source automation server that helps automate:
- Building applications
- Running tests
- Deploying code
- Scheduling tasks

### Key Features
- **Free and Open Source** - No licensing costs
- **Extensible** - 1800+ plugins available
- **Multi-platform** - Works on Windows, Linux, macOS
- **Distributed Builds** - Can use multiple machines
- **Easy Configuration** - Web-based interface

### Jenkins Architecture

```
┌─────────────────────────────────────────────┐
│           Jenkins Master/Controller         │
│  - Schedules builds                         │
│  - Dispatches builds to agents              │
│  - Monitors agents                          │
│  - Records and presents build results       │
└─────────────────┬───────────────────────────┘
                  │
    ┌─────────────┼─────────────┐
    │             │             │
┌───▼────┐   ┌───▼────┐   ┌───▼────┐
│Windows │   │ Linux  │   │ macOS  │
│ Agent  │   │ Agent  │   │ Agent  │
└────────┘   └────────┘   └────────┘
```

---

## Jenkins Installation

### Prerequisites
- Java 11 or Java 17 (LTS versions)
- Minimum 256 MB RAM (1+ GB recommended)
- 1 GB+ disk space

### Windows Installation

#### Method 1: Windows Installer (Easiest)

1. **Download Jenkins**
   - Go to: https://www.jenkins.io/download/
   - Download Windows installer (.msi)

2. **Run Installer**
   ```
   - Double-click jenkins.msi
   - Follow installation wizard
   - Choose installation directory
   - Jenkins installs as Windows Service
   ```

3. **Access Jenkins**
   - Open browser: http://localhost:8080
   - Find initial password at:
     ```
     C:\Program Files\Jenkins\secrets\initialAdminPassword
     ```

#### Method 2: WAR File

```powershell
# Download Jenkins WAR
Invoke-WebRequest -Uri "https://get.jenkins.io/war-stable/latest/jenkins.war" -OutFile "jenkins.war"

# Run Jenkins
java -jar jenkins.war --httpPort=8080
```

### Linux Installation (Ubuntu/Debian)

```bash
# Update package list
sudo apt update

# Install Java
sudo apt install openjdk-11-jdk -y

# Add Jenkins repository
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt update
sudo apt install jenkins -y

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Check status
sudo systemctl status jenkins

# Get initial password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### Linux Installation (CentOS/RHEL)

```bash
# Install Java
sudo yum install java-11-openjdk -y

# Add Jenkins repository
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# Install Jenkins
sudo yum install jenkins -y

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Get initial password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### Initial Setup

1. **Unlock Jenkins**
   - Paste initial admin password
   - Click "Continue"

2. **Install Plugins**
   - Choose "Install suggested plugins" (recommended for beginners)
   - Or select specific plugins

3. **Create Admin User**
   - Username: admin
   - Password: (your secure password)
   - Email: your@email.com

4. **Jenkins URL**
   - Keep default: http://localhost:8080
   - Or set your domain

5. **Start Using Jenkins!**

---

## Jenkins Basics

### Dashboard Overview

```
┌──────────────────────────────────────────────┐
│  Jenkins Dashboard                     [New] │
├──────────────────────────────────────────────┤
│  Build Queue                                 │
│  - (empty)                                   │
├──────────────────────────────────────────────┤
│  Build Executor Status                       │
│  - Master: Idle                              │
├──────────────────────────────────────────────┤
│  All Jobs                                    │
│  ┌────────────────────────────────────────┐ │
│  │ Job Name  │ Status │ Last Success     │ │
│  │ My-Job    │   ✓    │ 5 mins ago       │ │
│  └────────────────────────────────────────┘ │
└──────────────────────────────────────────────┘
```

### Key Terminology

**Job/Project** - A runnable task in Jenkins
**Build** - A single execution of a job
**Workspace** - Directory where builds happen
**Node/Agent** - Machine that executes builds
**Executor** - A slot for running builds
**Plugin** - Extension that adds functionality

### Navigation

- **Dashboard** - Main page showing all jobs
- **New Item** - Create new job
- **People** - User management
- **Build History** - All build records
- **Manage Jenkins** - Configuration and settings
- **Credentials** - Manage passwords, keys, tokens

---

## Your First Job

### Level 1: Hello World - Freestyle Project

#### Step 1: Create Job
1. Click "New Item" on Dashboard
2. Enter name: `hello-world`
3. Select "Freestyle project"
4. Click "OK"

#### Step 2: Configure Job
1. **Description**: "My first Jenkins job"
2. Scroll to **Build Steps**
3. Click "Add build step"
   - Windows: Choose "Execute Windows batch command"
   - Linux: Choose "Execute shell"

#### Step 3: Add Build Command

**For Windows:**
```batch
echo Hello, Jenkins!
echo Build Number: %BUILD_NUMBER%
echo Workspace: %WORKSPACE%
echo Job Name: %JOB_NAME%
```

**For Linux:**
```bash
echo "Hello, Jenkins!"
echo "Build Number: $BUILD_NUMBER"
echo "Workspace: $WORKSPACE"
echo "Job Name: $JOB_NAME"
```

#### Step 4: Save and Build
1. Click "Save"
2. Click "Build Now"
3. Watch build in "Build History"
4. Click build number (e.g., #1)
5. Click "Console Output" to see results

**🎉 Congratulations! You've run your first Jenkins job!**

### Level 2: Build with Parameters

#### Step 1: Create Parameterized Job
1. New Item → `parameterized-hello`
2. Freestyle project

#### Step 2: Add Parameters
1. Check "This project is parameterized"
2. Click "Add Parameter" → "String Parameter"
   - Name: `USERNAME`
   - Default Value: `World`
   - Description: `Enter your name`
3. Click "Add Parameter" → "Choice Parameter"
   - Name: `GREETING`
   - Choices: (one per line)
     ```
     Hello
     Hi
     Hey
     Greetings
     ```

#### Step 3: Use Parameters in Build

**Windows:**
```batch
echo %GREETING%, %USERNAME%!
echo This is build number %BUILD_NUMBER%
```

**Linux:**
```bash
echo "$GREETING, $USERNAME!"
echo "This is build number $BUILD_NUMBER"
```

#### Step 4: Run with Parameters
1. Save
2. Click "Build with Parameters"
3. Enter name and select greeting
4. Click "Build"
5. Check console output

---

## Understanding Build Agents

### What are Agents?

**Master (Controller)**
- Manages Jenkins
- Schedules jobs
- Monitors agents
- Stores configurations

**Agent (Node/Slave)**
- Executes actual builds
- Can be on different machines
- Can run different operating systems
- Reduces load on master

### Why Use Agents?

1. **Resource Distribution** - Don't overload master
2. **Platform Diversity** - Build on Windows, Linux, macOS
3. **Parallel Builds** - Run multiple builds simultaneously
4. **Security** - Isolate build environments

### Agent Types

**Permanent Agent**
- Dedicated machine
- Always available
- Best for production

**Cloud Agent**
- On-demand (AWS, Azure, Docker)
- Scales automatically
- Cost-effective

### Setting Up Windows Agent

#### On Master (Jenkins Web UI)

1. **Manage Jenkins** → **Nodes**
2. Click "New Node"
3. Enter node name: `windows-agent-01`
4. Select "Permanent Agent"
5. Configure:
   ```
   Name: windows-agent-01
   Description: Windows Build Agent
   Number of executors: 2
   Remote root directory: C:\Jenkins
   Labels: windows win
   Usage: Use this node as much as possible
   Launch method: Launch agent via Java Web Start
   ```
6. Save

#### On Windows Agent Machine

1. **Install Java**
   ```powershell
   # Download and install Java JDK 11 or 17
   ```

2. **Download agent.jar**
   - Go to Master Jenkins
   - Click on your new node
   - Copy download command or download `agent.jar`

3. **Run Agent**
   ```powershell
   # Create Jenkins directory
   New-Item -Path "C:\Jenkins" -ItemType Directory
   
   # Navigate to Jenkins directory
   cd C:\Jenkins
   
   # Run agent (replace URL with your master URL)
   java -jar agent.jar -jnlpUrl http://master-ip:8080/computer/windows-agent-01/jenkins-agent.jnlp -secret your-secret-here -workDir "C:\Jenkins"
   ```

4. **Install as Windows Service** (Optional)
   ```powershell
   # Download winsw.exe
   # Rename to jenkins-agent.exe
   # Create jenkins-agent.xml configuration
   # Install service
   jenkins-agent.exe install
   jenkins-agent.exe start
   ```

### Setting Up Linux Agent

#### On Master

Same as Windows, but:
- Remote root directory: `/home/jenkins`
- Labels: `linux ubuntu`
- Launch method: Launch agent via SSH

#### On Linux Agent Machine

1. **Create Jenkins User**
   ```bash
   sudo useradd -m -s /bin/bash jenkins
   sudo su - jenkins
   mkdir -p /home/jenkins
   ```

2. **Install Java**
   ```bash
   sudo apt update
   sudo apt install openjdk-11-jdk -y
   ```

3. **Configure SSH**
   ```bash
   # Generate SSH key on master
   ssh-keygen -t rsa -b 4096
   
   # Copy public key to agent
   ssh-copy-id jenkins@agent-ip
   ```

4. **In Jenkins UI**
   - Add SSH credentials (username + private key)
   - Select "Launch agents via SSH"
   - Enter host, credentials
   - Save

---

## Pipeline Fundamentals

### Why Pipelines?

**Freestyle Jobs:**
- ✓ Easy for beginners
- ✓ GUI configuration
- ✗ Hard to version control
- ✗ Limited reusability
- ✗ Complex workflows are messy

**Pipelines:**
- ✓ Code as configuration (Jenkinsfile)
- ✓ Version controlled
- ✓ Reusable and shareable
- ✓ Complex workflows are manageable
- ✓ Better visualization

### Pipeline Types

#### 1. Declarative Pipeline (Recommended for Beginners)
- Simpler syntax
- More opinionated
- Easier to read
- Better error messages

#### 2. Scripted Pipeline
- More flexible
- Groovy-based
- More programming control
- Steeper learning curve

### Your First Pipeline

#### Create Pipeline Job
1. New Item → `my-first-pipeline`
2. Select "Pipeline"
3. Click OK

#### Write Pipeline Script

```groovy
pipeline {
    agent any
    
    stages {
        stage('Hello') {
            steps {
                echo 'Hello, Pipeline!'
            }
        }
        
        stage('Build Info') {
            steps {
                echo "Build Number: ${env.BUILD_NUMBER}"
                echo "Job Name: ${env.JOB_NAME}"
            }
        }
        
        stage('Goodbye') {
            steps {
                echo 'Pipeline Complete!'
            }
        }
    }
}
```

Save and click "Build Now"

### Pipeline Structure Explained

```groovy
pipeline {                    // Declarative pipeline starts here
    agent any                 // Run on any available agent
    
    stages {                  // Contains all stages
        stage('Stage Name') { // A stage (visible in UI)
            steps {           // Steps to execute
                // Your commands here
            }
        }
    }
}
```

### Pipeline from SCM (Git)

#### Create Jenkinsfile in Your Repository

**File: Jenkinsfile**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Code checked out automatically'
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building application...'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying application...'
            }
        }
    }
}
```

#### Configure Pipeline to Use SCM

1. In Pipeline configuration
2. Definition: "Pipeline script from SCM"
3. SCM: Git
4. Repository URL: `https://github.com/yourusername/your-repo.git`
5. Branch: `*/main`
6. Script Path: `Jenkinsfile`
7. Save

---

## Windows Jobs

### Level 1: Basic Batch Commands

```groovy
pipeline {
    agent {
        label 'windows'
    }
    
    stages {
        stage('System Info') {
            steps {
                bat '''
                    echo Current Directory:
                    cd
                    
                    echo System Info:
                    systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
                    
                    echo Java Version:
                    java -version
                '''
            }
        }
    }
}
```

### Level 2: Building .NET Application

#### Create .NET Console App
```powershell
# On your development machine
dotnet new console -n HelloApp
cd HelloApp
git init
git add .
git commit -m "Initial commit"
git remote add origin YOUR_REPO_URL
git push -u origin main
```

#### Jenkins Pipeline

```groovy
pipeline {
    agent {
        label 'windows'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/yourusername/HelloApp.git'
            }
        }
        
        stage('Restore') {
            steps {
                bat 'dotnet restore'
            }
        }
        
        stage('Build') {
            steps {
                bat 'dotnet build --configuration Release'
            }
        }
        
        stage('Test') {
            steps {
                bat 'dotnet test'
            }
        }
        
        stage('Publish') {
            steps {
                bat 'dotnet publish -c Release -o ./publish'
            }
        }
        
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'publish/**/*',
                                 fingerprint: true
            }
        }
    }
    
    post {
        success {
            echo 'Build succeeded!'
        }
        failure {
            echo 'Build failed!'
        }
        always {
            cleanWs()
        }
    }
}
```

### Level 3: Building .NET Web Application

```groovy
pipeline {
    agent {
        label 'windows'
    }
    
    environment {
        BUILD_CONFIG = 'Release'
        PUBLISH_DIR = 'publish'
    }
    
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['Development', 'Staging', 'Production'],
            description: 'Target environment'
        )
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Restore NuGet Packages') {
            steps {
                bat 'dotnet restore MyWebApp.sln'
            }
        }
        
        stage('Build') {
            steps {
                bat """
                    dotnet build MyWebApp.sln ^
                    --configuration ${BUILD_CONFIG} ^
                    --no-restore
                """
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                bat '''
                    dotnet test MyWebApp.Tests.csproj ^
                    --configuration Release ^
                    --logger "trx;LogFileName=TestResults.trx" ^
                    --no-build
                '''
            }
            post {
                always {
                    mstest testResultsFile: '**/*.trx'
                }
            }
        }
        
        stage('Publish') {
            steps {
                bat """
                    dotnet publish MyWebApp.csproj ^
                    -c ${BUILD_CONFIG} ^
                    -o ${PUBLISH_DIR} ^
                    --no-build
                """
            }
        }
        
        stage('Deploy to IIS') {
            when {
                expression { params.ENVIRONMENT == 'Staging' || params.ENVIRONMENT == 'Production' }
            }
            steps {
                script {
                    def iisPath = params.ENVIRONMENT == 'Production' ? 
                                  'C:\\inetpub\\wwwroot\\MyApp' : 
                                  'C:\\inetpub\\wwwroot\\MyApp-Staging'
                    
                    bat """
                        powershell -Command "Stop-WebSite -Name 'MyApp-${params.ENVIRONMENT}'"
                        
                        xcopy /s /y ${PUBLISH_DIR}\\* ${iisPath}\\
                        
                        powershell -Command "Start-WebSite -Name 'MyApp-${params.ENVIRONMENT}'"
                    """
                }
            }
        }
    }
    
    post {
        success {
            emailext (
                subject: "SUCCESS: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
                    Build successful for environment: ${params.ENVIRONMENT}
                    
                    Build URL: ${env.BUILD_URL}
                """,
                to: 'team@example.com'
            )
        }
        failure {
            emailext (
                subject: "FAILED: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: "Build failed. Check console output.",
                to: 'team@example.com'
            )
        }
    }
}
```

### Level 4: MSBuild (.NET Framework)

```groovy
pipeline {
    agent {
        label 'windows'
    }
    
    environment {
        MSBUILD = 'C:\\Program Files\\Microsoft Visual Studio\\2022\\Community\\MSBuild\\Current\\Bin\\MSBuild.exe'
        NUGET = 'C:\\Tools\\nuget.exe'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Restore NuGet') {
            steps {
                bat "\"${NUGET}\" restore MyApp.sln"
            }
        }
        
        stage('Build') {
            steps {
                bat """
                    "${MSBUILD}" MyApp.sln ^
                    /p:Configuration=Release ^
                    /p:Platform="Any CPU" ^
                    /t:Rebuild ^
                    /m
                """
            }
        }
        
        stage('Run Tests') {
            steps {
                bat '''
                    vstest.console.exe ^
                    MyApp.Tests\\bin\\Release\\MyApp.Tests.dll ^
                    /Logger:trx
                '''
            }
        }
        
        stage('Package') {
            steps {
                bat """
                    "${MSBUILD}" MyApp\\MyApp.csproj ^
                    /p:Configuration=Release ^
                    /p:DeployOnBuild=true ^
                    /p:PublishProfile=FolderProfile
                """
            }
        }
    }
}
```

---

## PowerShell Jobs

### Level 1: Basic PowerShell

```groovy
pipeline {
    agent {
        label 'windows'
    }
    
    stages {
        stage('PowerShell Basics') {
            steps {
                powershell '''
                    Write-Host "Hello from PowerShell!"
                    Write-Host "PowerShell Version: $($PSVersionTable.PSVersion)"
                    Write-Host "Current Location: $(Get-Location)"
                    
                    # Get system info
                    Get-ComputerInfo | Select-Object CsName, OsName, OsVersion
                '''
            }
        }
    }
}
```

### Level 2: PowerShell with Variables

```groovy
pipeline {
    agent {
        label 'windows'
    }
    
    environment {
        APP_NAME = 'MyApplication'
        VERSION = '1.0.0'
    }
    
    stages {
        stage('Build Info') {
            steps {
                powershell """
                    \$appName = '${env.APP_NAME}'
                    \$version = '${env.VERSION}'
                    \$buildNum = '${env.BUILD_NUMBER}'
                    
                    Write-Host "Building \$appName version \$version (Build #\$buildNum)"
                    
                    # Create version file
                    @{
                        AppName = \$appName
                        Version = \$version
                        BuildNumber = \$buildNum
                        BuildDate = Get-Date -Format 'yyyy-MM-dd HH:mm:ss'
                    } | ConvertTo-Json | Out-File version.json
                    
                    # Display content
                    Get-Content version.json
                """
            }
        }
    }
}
```

### Level 3: PowerShell Deployment Script

```groovy
pipeline {
    agent {
        label 'windows'
    }
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['DEV', 'QA', 'PROD'])
        string(name: 'VERSION', defaultValue: '1.0.0')
    }
    
    stages {
        stage('Deploy Application') {
            steps {
                powershell """
                    # Configuration based on environment
                    \$envConfig = @{
                        'DEV' = @{
                            Server = 'dev-server'
                            Path = 'C:\\Apps\\Dev'
                            ServiceName = 'MyApp-Dev'
                        }
                        'QA' = @{
                            Server = 'qa-server'
                            Path = 'C:\\Apps\\QA'
                            ServiceName = 'MyApp-QA'
                        }
                        'PROD' = @{
                            Server = 'prod-server'
                            Path = 'C:\\Apps\\Prod'
                            ServiceName = 'MyApp-Prod'
                        }
                    }
                    
                    \$config = \$envConfig['${params.ENVIRONMENT}']
                    
                    Write-Host "Deploying to ${params.ENVIRONMENT}"
                    Write-Host "Server: \$(\$config.Server)"
                    Write-Host "Path: \$(\$config.Path)"
                    
                    # Stop service
                    Write-Host "Stopping service..."
                    Stop-Service -Name \$config.ServiceName -Force -ErrorAction SilentlyContinue
                    Start-Sleep -Seconds 3
                    
                    # Backup existing deployment
                    \$backupPath = "\$(\$config.Path)_backup_\$(Get-Date -Format 'yyyyMMdd_HHmmss')"
                    if (Test-Path \$config.Path) {
                        Write-Host "Creating backup at \$backupPath"
                        Copy-Item -Path \$config.Path -Destination \$backupPath -Recurse
                    }
                    
                    # Deploy new version
                    Write-Host "Deploying version ${params.VERSION}"
                    if (-not (Test-Path \$config.Path)) {
                        New-Item -Path \$config.Path -ItemType Directory -Force
                    }
                    
                    Copy-Item -Path ".\\publish\\*" -Destination \$config.Path -Recurse -Force
                    
                    # Update config
                    \$configFile = Join-Path \$config.Path "appsettings.json"
                    if (Test-Path \$configFile) {
                        \$json = Get-Content \$configFile | ConvertFrom-Json
                        \$json.Environment = '${params.ENVIRONMENT}'
                        \$json.Version = '${params.VERSION}'
                        \$json | ConvertTo-Json -Depth 10 | Set-Content \$configFile
                    }
                    
                    # Start service
                    Write-Host "Starting service..."
                    Start-Service -Name \$config.ServiceName
                    
                    # Verify service is running
                    \$service = Get-Service -Name \$config.ServiceName
                    if (\$service.Status -eq 'Running') {
                        Write-Host "✓ Deployment successful! Service is running."
                    } else {
                        Write-Error "✗ Service failed to start!"
                        exit 1
                    }
                """
            }
        }
    }
}
```

### Level 4: Advanced PowerShell with Error Handling

```groovy
pipeline {
    agent {
        label 'windows'
    }
    
    stages {
        stage('Complex Operations') {
            steps {
                powershell '''
                    # Set error action preference
                    $ErrorActionPreference = "Stop"
                    
                    function Write-Log {
                        param([string]$Message, [string]$Level = "INFO")
                        $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
                        Write-Host "[$timestamp] [$Level] $Message"
                    }
                    
                    try {
                        Write-Log "Starting deployment process"
                        
                        # Check prerequisites
                        Write-Log "Checking prerequisites..."
                        
                        $requiredModules = @('WebAdministration')
                        foreach ($module in $requiredModules) {
                            if (-not (Get-Module -ListAvailable -Name $module)) {
                                throw "Required module '$module' is not installed"
                            }
                            Import-Module $module
                            Write-Log "Module $module loaded" "SUCCESS"
                        }
                        
                        # Perform health check
                        Write-Log "Performing health check..."
                        $response = Invoke-WebRequest -Uri "http://localhost/health" -UseBasicParsing -TimeoutSec 10
                        if ($response.StatusCode -ne 200) {
                            throw "Health check failed with status code $($response.StatusCode)"
                        }
                        Write-Log "Health check passed" "SUCCESS"
                        
                        # Clean up old files
                        Write-Log "Cleaning up old files..."
                        $daysToKeep = 30
                        $path = "C:\\Backups"
                        Get-ChildItem -Path $path -Recurse -File | 
                            Where-Object { $_.CreationTime -lt (Get-Date).AddDays(-$daysToKeep) } | 
                            ForEach-Object {
                                Write-Log "Deleting old file: $($_.FullName)"
                                Remove-Item $_.FullName -Force
                            }
                        
                        Write-Log "Process completed successfully" "SUCCESS"
                        
                    } catch {
                        Write-Log "Error occurred: $($_.Exception.Message)" "ERROR"
                        Write-Log "Stack trace: $($_.ScriptStackTrace)" "ERROR"
                        throw
                    } finally {
                        Write-Log "Cleanup operations"
                    }
                '''
            }
        }
    }
}
```

---

## Linux Jobs

### Level 1: Basic Shell Commands

```groovy
pipeline {
    agent {
        label 'linux'
    }
    
    stages {
        stage('System Info') {
            steps {
                sh '''
                    echo "===== System Information ====="
                    uname -a
                    
                    echo ""
                    echo "===== OS Release ====="
                    cat /etc/os-release
                    
                    echo ""
                    echo "===== Disk Usage ====="
                    df -h
                    
                    echo ""
                    echo "===== Memory Usage ====="
                    free -h
                    
                    echo ""
                    echo "===== Current User ====="
                    whoami
                    
                    echo ""
                    echo "===== Working Directory ====="
                    pwd
                '''
            }
        }
    }
}
```

### Level 2: Building Node.js Application

```groovy
pipeline {
    agent {
        label 'linux'
    }
    
    tools {
        nodejs 'NodeJS 18'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/yourusername/nodejs-app.git'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Run Linting') {
            steps {
                sh 'npm run lint'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'dist/**/*',
                                 fingerprint: true
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
```

### Level 3: Building Python Application

```groovy
pipeline {
    agent {
        label 'linux'
    }
    
    environment {
        PYTHON_VERSION = '3.9'
        VENV_DIR = 'venv'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Setup Virtual Environment') {
            steps {
                sh '''
                    python3 -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                '''
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh '''
                    . ${VENV_DIR}/bin/activate
                    pip install -r requirements.txt
                '''
            }
        }
        
        stage('Run Linting') {
            steps {
                sh '''
                    . ${VENV_DIR}/bin/activate
                    pip install pylint
                    pylint src/ || true
                '''
            }
        }
        
        stage('Run Tests') {
            steps {
                sh '''
                    . ${VENV_DIR}/bin/activate
                    pip install pytest pytest-cov
                    pytest --cov=src --cov-report=xml --cov-report=html
                '''
            }
        }
        
        stage('Package') {
            steps {
                sh '''
                    . ${VENV_DIR}/bin/activate
                    python setup.py sdist bdist_wheel
                '''
            }
        }
    }
    
    post {
        always {
            junit 'test-results/*.xml'
            publishCoverage adapters: [coberturaAdapter('coverage.xml')]
        }
    }
}
```

### Level 4: Docker Build and Push

```groovy
pipeline {
    agent {
        label 'linux'
    }
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_IMAGE = 'myusername/myapp'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
        stage('Test Image') {
            steps {
                sh """
                    docker run --rm ${DOCKER_IMAGE}:${DOCKER_TAG} npm test
                """
            }
        }
        
        stage('Push to Registry') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin ${DOCKER_REGISTRY}
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
        stage('Clean Up') {
            steps {
                sh """
                    docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true
                    docker rmi ${DOCKER_IMAGE}:latest || true
                """
            }
        }
    }
    
    post {
        always {
            sh 'docker logout ${DOCKER_REGISTRY} || true'
        }
    }
}
```

### Level 5: Kubernetes Deployment

```groovy
pipeline {
    agent {
        label 'linux'
    }
    
    environment {
        KUBE_NAMESPACE = 'production'
        APP_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t myregistry.com/${APP_NAME}:${IMAGE_TAG} .
                        docker push myregistry.com/${APP_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
        
        stage('Update Kubernetes Manifests') {
            steps {
                sh """
                    sed -i 's|IMAGE_TAG|${IMAGE_TAG}|g' k8s/deployment.yaml
                """
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl apply -f k8s/deployment.yaml -n ${KUBE_NAMESPACE}
                        kubectl apply -f k8s/service.yaml -n ${KUBE_NAMESPACE}
                        
                        # Wait for rollout
                        kubectl rollout status deployment/${APP_NAME} -n ${KUBE_NAMESPACE} --timeout=5m
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                        # Check pod status
                        kubectl get pods -n ${KUBE_NAMESPACE} -l app=${APP_NAME}
                        
                        # Check service
                        kubectl get svc -n ${KUBE_NAMESPACE} -l app=${APP_NAME}
                        
                        # Run health check
                        POD_NAME=\$(kubectl get pods -n ${KUBE_NAMESPACE} -l app=${APP_NAME} -o jsonpath='{.items[0].metadata.name}')
                        kubectl exec \$POD_NAME -n ${KUBE_NAMESPACE} -- curl -f http://localhost:8080/health || exit 1
                    """
                }
            }
        }
    }
    
    post {
        failure {
            withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                sh """
                    echo "Deployment failed, rolling back..."
                    kubectl rollout undo deployment/${APP_NAME} -n ${KUBE_NAMESPACE}
                """
            }
        }
    }
}
```

---

## Advanced Concepts

### 1. Shared Libraries

Create reusable code across pipelines.

**File Structure:**
```
my-jenkins-library/
├── vars/
│   ├── buildAndTest.groovy
│   └── deployToK8s.groovy
└── src/
    └── com/
        └── mycompany/
            └── Utilities.groovy
```

**vars/buildAndTest.groovy:**
```groovy
def call(Map config) {
    pipeline {
        agent any
        
        stages {
            stage('Build') {
                steps {
                    sh "${config.buildCommand}"
                }
            }
            
            stage('Test') {
                steps {
                    sh "${config.testCommand}"
                }
            }
        }
    }
}
```

**Using in Jenkinsfile:**
```groovy
@Library('my-jenkins-library') _

buildAndTest(
    buildCommand: 'npm run build',
    testCommand: 'npm test'
)
```

### 2. Parallel Execution

```groovy
pipeline {
    agent any
    
    stages {
        stage('Parallel Tests') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
                
                stage('E2E Tests') {
                    steps {
                        sh 'npm run test:e2e'
                    }
                }
            }
        }
    }
}
```

### 3. Matrix Builds

Test on multiple configurations:

```groovy
pipeline {
    agent none
    
    stages {
        stage('Build and Test') {
            matrix {
                agent any
                
                axes {
                    axis {
                        name 'OS'
                        values 'windows', 'linux', 'macos'
                    }
                    axis {
                        name 'NODE_VERSION'
                        values '14', '16', '18'
                    }
                }
                
                stages {
                    stage('Build') {
                        steps {
                            echo "Building on ${OS} with Node ${NODE_VERSION}"
                            sh "node --version"
                        }
                    }
                }
            }
        }
    }
}
```

### 4. Blue Ocean UI

Better visualization for pipelines:

1. Install Blue Ocean plugin
2. Access at: `http://localhost:8080/blue`
3. Visual pipeline editor
4. Better logs and visualization

### 5. Credentials Management

```groovy
pipeline {
    agent any
    
    stages {
        stage('Use Credentials') {
            steps {
                // Username/Password
                withCredentials([usernamePassword(
                    credentialsId: 'my-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh 'echo $USER'
                }
                
                // Secret text
                withCredentials([string(
                    credentialsId: 'api-token',
                    variable: 'TOKEN'
                )]) {
                    sh 'curl -H "Authorization: Bearer $TOKEN" api.example.com'
                }
                
                // SSH key
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ssh-key',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh 'ssh -i $SSH_KEY $SSH_USER@server.com'
                }
            }
        }
    }
}
```

### 6. Environment-Specific Deployments

```groovy
pipeline {
    agent any
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'production'])
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        stage('Deploy to Dev') {
            when {
                expression { params.ENVIRONMENT == 'dev' }
            }
            steps {
                sh './deploy.sh dev'
            }
        }
        
        stage('Deploy to Staging') {
            when {
                expression { params.ENVIRONMENT == 'staging' }
            }
            steps {
                input message: 'Deploy to staging?'
                sh './deploy.sh staging'
            }
        }
        
        stage('Deploy to Production') {
            when {
                expression { params.ENVIRONMENT == 'production' }
            }
            steps {
                input message: 'Deploy to PRODUCTION?', submitter: 'admin'
                sh './deploy.sh production'
            }
        }
    }
}
```

---

## Real-World Projects

### Project 1: Full-Stack Application CI/CD

**Application Stack:**
- Frontend: React (Node.js)
- Backend: .NET Core API
- Database: SQL Server
- Deployment: Docker + Kubernetes

**Jenkinsfile:**
```groovy
pipeline {
    agent none
    
    environment {
        DOCKER_REGISTRY = 'myregistry.com'
        APP_VERSION = "1.0.${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            agent any
            steps {
                checkout scm
            }
        }
        
        stage('Build Frontend') {
            agent {
                label 'linux'
            }
            steps {
                dir('frontend') {
                    sh '''
                        npm install
                        npm run lint
                        npm run test
                        npm run build
                    '''
                    
                    // Build Docker image
                    sh """
                        docker build -t ${DOCKER_REGISTRY}/myapp-frontend:${APP_VERSION} .
                        docker push ${DOCKER_REGISTRY}/myapp-frontend:${APP_VERSION}
                    """
                }
            }
        }
        
        stage('Build Backend') {
            agent {
                label 'windows'
            }
            steps {
                dir('backend') {
                    bat '''
                        dotnet restore
                        dotnet build --configuration Release
                        dotnet test --configuration Release --logger "trx"
                        dotnet publish -c Release -o ./publish
                    '''
                    
                    // Build Docker image (Windows containers)
                    bat """
                        docker build -t ${DOCKER_REGISTRY}/myapp-backend:${APP_VERSION} .
                        docker push ${DOCKER_REGISTRY}/myapp-backend:${APP_VERSION}
                    """
                }
            }
        }
        
        stage('Integration Tests') {
            agent {
                label 'linux'
            }
            steps {
                sh '''
                    docker-compose up -d
                    npm run test:integration
                    docker-compose down
                '''
            }
        }
        
        stage('Deploy to Staging') {
            agent {
                label 'linux'
            }
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-staging', variable: 'KUBECONFIG')]) {
                    sh """
                        # Update image tags
                        sed -i 's|FRONTEND_IMAGE|${DOCKER_REGISTRY}/myapp-frontend:${APP_VERSION}|g' k8s/staging/frontend.yaml
                        sed -i 's|BACKEND_IMAGE|${DOCKER_REGISTRY}/myapp-backend:${APP_VERSION}|g' k8s/staging/backend.yaml
                        
                        # Deploy
                        kubectl apply -f k8s/staging/
                        
                        # Wait for rollout
                        kubectl rollout status deployment/frontend -n staging
                        kubectl rollout status deployment/backend -n staging
                    """
                }
            }
        }
        
        stage('Smoke Tests') {
            agent {
                label 'linux'
            }
            steps {
                sh '''
                    # Run smoke tests against staging
                    npm run test:smoke -- --env staging
                '''
            }
        }
        
        stage('Deploy to Production') {
            agent {
                label 'linux'
            }
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to Production?', submitter: 'admin,devops-lead'
                
                withCredentials([file(credentialsId: 'kubeconfig-prod', variable: 'KUBECONFIG')]) {
                    sh """
                        # Update image tags
                        sed -i 's|FRONTEND_IMAGE|${DOCKER_REGISTRY}/myapp-frontend:${APP_VERSION}|g' k8s/production/frontend.yaml
                        sed -i 's|BACKEND_IMAGE|${DOCKER_REGISTRY}/myapp-backend:${APP_VERSION}|g' k8s/production/backend.yaml
                        
                        # Blue-Green deployment
                        kubectl apply -f k8s/production/
                        kubectl rollout status deployment/frontend -n production
                        kubectl rollout status deployment/backend -n production
                    """
                }
            }
        }
    }
    
    post {
        success {
            emailext (
                subject: "✓ Deployment Successful - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    Version ${APP_VERSION} deployed successfully!
                    
                    Frontend: ${DOCKER_REGISTRY}/myapp-frontend:${APP_VERSION}
                    Backend: ${DOCKER_REGISTRY}/myapp-backend:${APP_VERSION}
                    
                    Build URL: ${env.BUILD_URL}
                """,
                to: 'team@example.com'
            )
        }
        
        failure {
            emailext (
                subject: "✗ Deployment Failed - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Deployment failed. Check logs: ${env.BUILD_URL}",
                to: 'team@example.com'
            )
        }
    }
}
```

### Project 2: Microservices CI/CD

```groovy
def services = ['auth', 'api', 'frontend', 'notification']

pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Services') {
            steps {
                script {
                    def builds = [:]
                    
                    services.each { service ->
                        builds[service] = {
                            stage("Build ${service}") {
                                sh """
                                    cd services/${service}
                                    docker build -t myregistry.com/${service}:${env.BUILD_NUMBER} .
                                    docker push myregistry.com/${service}:${env.BUILD_NUMBER}
                                """
                            }
                        }
                    }
                    
                    parallel builds
                }
            }
        }
        
        stage('Deploy Services') {
            steps {
                script {
                    services.each { service ->
                        sh """
                            kubectl set image deployment/${service} \
                                ${service}=myregistry.com/${service}:${env.BUILD_NUMBER} \
                                -n production
                            
                            kubectl rollout status deployment/${service} -n production
                        """
                    }
                }
            }
        }
    }
}
```

---

## Best Practices

### 1. Pipeline Design

✅ **DO:**
- Keep Jenkinsfiles in version control
- Use declarative pipeline for clarity
- Break down into small, reusable stages
- Use shared libraries for common code
- Implement proper error handling
- Add meaningful stage names

❌ **DON'T:**
- Hardcode credentials
- Put too much logic in Jenkinsfile
- Create monolithic pipelines
- Skip error handling
- Ignore failed tests

### 2. Security

✅ **DO:**
- Use Jenkins credentials store
- Rotate credentials regularly
- Limit agent access
- Use RBAC (Role-Based Access Control)
- Scan for vulnerabilities
- Keep Jenkins updated

❌ **DON'T:**
- Store passwords in code
- Use root/admin for everything
- Expose Jenkins to internet without security
- Share credentials between teams
- Skip security updates

### 3. Performance

✅ **DO:**
- Use agents for builds (don't run on master)
- Implement parallel execution
- Clean up workspaces
- Cache dependencies
- Use incremental builds
- Archive only necessary artifacts

❌ **DON'T:**
- Run all builds on master
- Keep old builds forever
- Archive entire workspace
- Download dependencies every time
- Keep unused agents running

### 4. Maintenance

✅ **DO:**
- Regular backups of JENKINS_HOME
- Monitor disk space
- Review and clean old jobs
- Update plugins regularly
- Document pipeline configurations
- Use Blue Ocean for better UI

❌ **DON'T:**
- Ignore warnings
- Let disk fill up
- Keep unused plugins
- Skip backups
- Forget to document changes

### 5. Code Quality

✅ **DO:**
```groovy
// Good: Clear, readable pipeline
pipeline {
    agent any
    
    environment {
        APP_NAME = 'myapp'
        VERSION = '1.0.0'
    }
    
    stages {
        stage('Build') {
            steps {
                echo "Building ${APP_NAME} v${VERSION}"
                sh 'make build'
            }
        }
    }
}
```

❌ **DON'T:**
```groovy
// Bad: Unclear, hard to maintain
node {
    sh 'make build && make test && make deploy'
}
```

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: "Agent went offline"

**Symptoms:**
- Build fails with agent disconnection
- "Agent went offline during the build"

**Solutions:**
```groovy
// Add timeout to catch hung builds
timeout(time: 30, unit: 'MINUTES') {
    sh 'long-running-command'
}

// Retry on failure
retry(3) {
    sh 'flaky-command'
}
```

#### Issue 2: "Workspace Cleanup Failed"

**Symptoms:**
- Cannot delete workspace
- Permission denied errors

**Solutions:**

Windows:
```groovy
// Use PowerShell to force cleanup
powershell '''
    Get-ChildItem -Path . -Recurse | Remove-Item -Force -Recurse -ErrorAction SilentlyContinue
'''
```

Linux:
```groovy
// Use sudo if needed (configure sudoers)
sh 'sudo rm -rf ${WORKSPACE}/*'
```

#### Issue 3: "Out of Memory"

**Symptoms:**
- Java heap space errors
- Build crashes

**Solutions:**
```bash
# Increase Jenkins memory (Linux)
sudo systemctl edit jenkins

# Add:
[Service]
Environment="JAVA_OPTS=-Xmx2048m -XX:MaxPermSize=512m"

# Restart
sudo systemctl restart jenkins
```

Windows:
```xml
<!-- Edit jenkins.xml -->
<arguments>-Xmx2048m -jar jenkins.war</arguments>
```

#### Issue 4: "Plugin Conflicts"

**Symptoms:**
- Jenkins won't start
- Plugin errors

**Solutions:**
```bash
# Safe restart (Linux)
sudo systemctl stop jenkins
cd /var/lib/jenkins/plugins
# Remove problematic plugin
rm -rf problematic-plugin*
sudo systemctl start jenkins
```

#### Issue 5: "Build Stuck in Queue"

**Symptoms:**
- Builds waiting forever
- "Waiting for executor"

**Solutions:**
1. Check available executors
2. Add more agents
3. Increase executors on existing agents
4. Check agent labels match job

```groovy
// In Jenkinsfile, specify any available agent
agent {
    label 'windows || linux'
}
```

### Debugging Tips

#### Enable Debug Logging

```groovy
pipeline {
    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    stages {
        stage('Debug') {
            steps {
                script {
                    // Print all environment variables
                    sh 'printenv'
                    
                    // Print specific variables
                    echo "Workspace: ${env.WORKSPACE}"
                    echo "Build Number: ${env.BUILD_NUMBER}"
                    echo "Job Name: ${env.JOB_NAME}"
                }
            }
        }
    }
}
```

#### Check Logs

```bash
# Jenkins main log (Linux)
tail -f /var/lib/jenkins/jenkins.log

# Specific job log
tail -f /var/lib/jenkins/jobs/my-job/builds/lastStableBuild/log

# Windows
# C:\Program Files\Jenkins\jenkins.log
```

---

## Practice Exercises

### Exercise 1: Basic Pipeline
Create a pipeline that:
1. Prints "Hello, Jenkins!"
2. Shows current date and time
3. Lists files in workspace

### Exercise 2: Parameterized Build
Create a pipeline that:
1. Accepts name as parameter
2. Accepts environment (dev/qa/prod) as choice
3. Prints personalized greeting

### Exercise 3: Multi-Stage Pipeline
Create a pipeline with stages:
1. Checkout code from Git
2. Build application
3. Run tests
4. Archive artifacts

### Exercise 4: Windows and Linux Pipeline
Create a pipeline that:
1. Runs on both Windows and Linux
2. Uses appropriate commands for each OS
3. Produces same output

### Exercise 5: Real Application
Build a complete CI/CD pipeline for:
1. Your favorite language (Node.js, Python, .NET, Java)
2. Include: build, test, package, deploy
3. Add notifications on success/failure

---

## Next Steps

### Further Learning Resources

**Official Documentation:**
- Jenkins User Documentation: https://www.jenkins.io/doc/
- Pipeline Syntax Reference: https://www.jenkins.io/doc/book/pipeline/syntax/

**Tutorials:**
- Jenkins Tutorial for Beginners (YouTube)
- CloudBees Jenkins Certification

**Books:**
- "Jenkins 2: Up and Running" by Brent Laster
- "Learning Continuous Integration with Jenkins" by Nikhil Pathania

**Practice:**
- Set up Jenkins locally
- Build real projects
- Contribute to open source
- Get Jenkins certification

### Jenkins Certifications

1. **Certified Jenkins Engineer (CJE)**
   - Entry-level certification
   - Covers basics and pipeline

2. **CloudBees Jenkins Certified Engineer**
   - Advanced certification
   - Includes CloudBees features

---

## Summary

You've learned:

✅ CI/CD fundamentals
✅ Jenkins installation and setup
✅ Creating freestyle and pipeline jobs
✅ Working with agents
✅ Windows, PowerShell, and Linux jobs
✅ Advanced concepts (parallel, matrix, libraries)
✅ Real-world project examples
✅ Best practices and troubleshooting

**You're now ready to:**
- Set up Jenkins for your projects
- Create complex CI/CD pipelines
- Deploy applications automatically
- Manage multi-platform builds
- Troubleshoot common issues

**Keep Learning:**
- Practice with real projects
- Explore plugins
- Join Jenkins community
- Stay updated with new features

---

## Quick Reference Card

### Common Commands

**Pipeline Structure:**
```groovy
pipeline {
    agent any
    stages {
        stage('Name') {
            steps {
                // commands
            }
        }
    }
}
```

**Windows:**
```groovy
bat 'command'
powershell 'command'
```

**Linux:**
```groovy
sh 'command'
```

**Credentials:**
```groovy
withCredentials([...]) {
    // use credentials
}
```

**Parameters:**
```groovy
parameters {
    string(name: 'NAME', defaultValue: 'value')
    choice(name: 'ENV', choices: ['dev', 'prod'])
}
```

**Conditionals:**
```groovy
when {
    branch 'main'
    environment name: 'DEPLOY', value: 'true'
}
```

---

**Happy Building! 🚀**
