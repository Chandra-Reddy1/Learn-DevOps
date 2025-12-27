# GitHub Actions Zero to Hero - Complete Learning Guide

## Table of Contents
1. [Introduction to GitHub Actions](#introduction-to-github-actions)
2. [What is GitHub Actions?](#what-is-github-actions)
3. [Getting Started](#getting-started)
4. [GitHub Actions Basics](#github-actions-basics)
5. [Your First Workflow](#your-first-workflow)
6. [Workflow Syntax](#workflow-syntax)
7. [Windows Workflows](#windows-workflows)
8. [PowerShell in GitHub Actions](#powershell-in-github-actions)
9. [Linux Workflows](#linux-workflows)
10. [Advanced Concepts](#advanced-concepts)
11. [Real-World Projects](#real-world-projects)
12. [Best Practices](#best-practices)
13. [Troubleshooting](#troubleshooting)

---

## Introduction to GitHub Actions

### What is GitHub Actions?

**GitHub Actions** is a CI/CD platform integrated directly into GitHub that allows you to:
- Automate build, test, and deployment workflows
- Run workflows on GitHub-hosted or self-hosted runners
- Respond to GitHub events (push, pull request, issues, etc.)
- Use pre-built actions from the marketplace

### Why GitHub Actions?

**Advantages:**
- ✅ **Integrated with GitHub** - No external tools needed
- ✅ **Free for public repositories** - Generous free tier for private repos
- ✅ **Matrix builds** - Test across multiple OS and versions easily
- ✅ **Huge marketplace** - 20,000+ pre-built actions
- ✅ **Simple YAML syntax** - Easy to learn and read
- ✅ **Secrets management** - Built-in secure storage
- ✅ **Multi-platform** - Linux, Windows, macOS runners

**Comparison with Jenkins:**

| Feature | GitHub Actions | Jenkins |
|---------|---------------|---------|
| **Setup** | No setup needed | Requires installation |
| **Hosting** | GitHub-hosted or self-hosted | Self-hosted |
| **Configuration** | YAML files in repo | Jenkinsfile or UI |
| **Cost** | Free tier available | Free, but hosting costs |
| **Learning Curve** | Easy | Moderate |
| **Marketplace** | 20,000+ actions | 1,800+ plugins |

### Key Terminology

**Workflow** - Automated process defined in YAML  
**Job** - A set of steps that execute on the same runner  
**Step** - Individual task (run command or action)  
**Action** - Reusable unit of code  
**Runner** - Server that runs your workflows  
**Event** - Activity that triggers a workflow  
**Artifact** - Files produced by a workflow  
**Secret** - Encrypted environment variable  

---

## What is GitHub Actions?

### Architecture

```
┌─────────────────────────────────────────────┐
│           GitHub Repository                 │
│  .github/workflows/                         │
│    ├── ci.yml                               │
│    ├── deploy.yml                           │
│    └── release.yml                          │
└─────────────────┬───────────────────────────┘
                  │
                  │ Event Triggered
                  │
    ┌─────────────▼─────────────┐
    │   GitHub Actions Engine    │
    │  - Parses workflow YAML    │
    │  - Schedules jobs          │
    │  - Manages secrets         │
    └─────────────┬───────────────┘
                  │
    ┌─────────────┼─────────────┐
    │             │             │
┌───▼────┐   ┌───▼────┐   ┌───▼────┐
│Ubuntu  │   │Windows │   │ macOS  │
│Runner  │   │Runner  │   │Runner  │
└────────┘   └────────┘   └────────┘
```

### Workflow Lifecycle

```
1. Event occurs (push, PR, schedule, etc.)
   ↓
2. GitHub Actions reads .github/workflows/*.yml
   ↓
3. Runner is provisioned
   ↓
4. Jobs execute (can be parallel or sequential)
   ↓
5. Steps run within each job
   ↓
6. Artifacts and logs are saved
   ↓
7. Runner is cleaned up
```

---

## Getting Started

### Prerequisites

- ✅ GitHub account (free)
- ✅ A repository (public or private)
- ✅ Basic YAML knowledge (will learn as we go)
- ✅ Basic Git knowledge

### Free Tier Limits

**Public Repositories:**
- ✅ Unlimited minutes
- ✅ Unlimited storage

**Private Repositories (Free tier):**
- ✅ 2,000 minutes/month
- ✅ 500 MB storage
- ✅ Linux: 1x multiplier
- ✅ Windows: 2x multiplier
- ✅ macOS: 10x multiplier

**Example:** Running 1,000 minutes on Windows = 2,000 minutes usage

### Creating Your First Workflow

#### Step 1: Create Workflow Directory

```bash
# In your repository
mkdir -p .github/workflows
cd .github/workflows
```

#### Step 2: Create Workflow File

```bash
# Create your first workflow
touch hello-world.yml
```

#### Step 3: Edit Workflow File

```yaml
# .github/workflows/hello-world.yml
name: Hello World

on: [push]

jobs:
  greet:
    runs-on: ubuntu-latest
    
    steps:
      - name: Say Hello
        run: echo "Hello, GitHub Actions!"
```

#### Step 4: Commit and Push

```bash
git add .github/workflows/hello-world.yml
git commit -m "Add first GitHub Actions workflow"
git push
```

#### Step 5: View Workflow Run

1. Go to your repository on GitHub
2. Click "Actions" tab
3. See your workflow running!
4. Click on the workflow to see details

**🎉 Congratulations! You've created your first GitHub Actions workflow!**

---

## GitHub Actions Basics

### Workflow File Structure

```yaml
name: Workflow Name                    # Display name

on:                                    # Events that trigger workflow
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:                                  # Jobs to run
  job-name:                           # Job ID
    runs-on: ubuntu-latest            # Runner OS
    
    steps:                            # Steps in this job
      - name: Step name               # Step display name
        run: echo "Hello"             # Command to run
```

### Common Triggers (Events)

```yaml
# Trigger on push to specific branches
on:
  push:
    branches:
      - main
      - develop

# Trigger on pull request
on:
  pull_request:
    branches: [ main ]

# Trigger on multiple events
on: [push, pull_request]

# Trigger on schedule (cron)
on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight UTC

# Trigger manually
on:
  workflow_dispatch:

# Trigger on release
on:
  release:
    types: [published]

# Trigger on issue
on:
  issues:
    types: [opened, labeled]
```

### Runner Types

```yaml
# Ubuntu (Linux)
runs-on: ubuntu-latest        # Ubuntu 22.04
runs-on: ubuntu-22.04
runs-on: ubuntu-20.04

# Windows
runs-on: windows-latest       # Windows Server 2022
runs-on: windows-2022
runs-on: windows-2019

# macOS
runs-on: macos-latest         # macOS 12
runs-on: macos-12
runs-on: macos-11

# Self-hosted
runs-on: self-hosted
runs-on: [self-hosted, linux]
```

### Checkout Code

**Always checkout your code first!**

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v4
  
  - name: Now you can access your code
    run: ls -la
```

---

## Your First Workflow

### Level 1: Hello World with Multiple Steps

```yaml
# .github/workflows/hello-steps.yml
name: Hello World Multi-Step

on: [push]

jobs:
  greet:
    runs-on: ubuntu-latest
    
    steps:
      - name: Step 1 - Say Hello
        run: echo "Hello from Step 1!"
      
      - name: Step 2 - Show Date
        run: date
      
      - name: Step 3 - System Info
        run: |
          echo "OS: ${{ runner.os }}"
          echo "Runner: ${{ runner.name }}"
          echo "Repository: ${{ github.repository }}"
      
      - name: Step 4 - Multi-line Script
        run: |
          echo "This is a multi-line script"
          echo "Current directory: $(pwd)"
          echo "Files in directory:"
          ls -la
```

### Level 2: Checkout and Build

```yaml
# .github/workflows/build.yml
name: Build Application

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm install
      
      - name: Run tests
        run: npm test
      
      - name: Build application
        run: npm run build
```

### Level 3: Multiple Jobs

```yaml
# .github/workflows/multi-jobs.yml
name: Multiple Jobs

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Tests
        run: echo "Running tests..."
  
  build:
    needs: test                    # Wait for 'test' job to complete
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Application
        run: echo "Building application..."
  
  deploy:
    needs: build                   # Wait for 'build' job to complete
    runs-on: ubuntu-latest
    steps:
      - name: Deploy Application
        run: echo "Deploying application..."
```

### Level 4: Parallel Jobs

```yaml
# .github/workflows/parallel-jobs.yml
name: Parallel Jobs

on: [push]

jobs:
  test-unit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Running unit tests..."
  
  test-integration:
    runs-on: ubuntu-latest         # Runs in parallel with test-unit
    steps:
      - uses: actions/checkout@v4
      - run: echo "Running integration tests..."
  
  test-e2e:
    runs-on: ubuntu-latest         # Runs in parallel with others
    steps:
      - uses: actions/checkout@v4
      - run: echo "Running E2E tests..."
```

---

## Workflow Syntax

### Environment Variables

```yaml
# Global environment variables
env:
  NODE_ENV: production
  APP_NAME: MyApp

jobs:
  build:
    runs-on: ubuntu-latest
    
    # Job-level environment variables
    env:
      BUILD_CONFIG: Release
    
    steps:
      - name: Use environment variables
        # Step-level environment variables
        env:
          STEP_VAR: "Hello"
        run: |
          echo "App: $APP_NAME"
          echo "Environment: $NODE_ENV"
          echo "Config: $BUILD_CONFIG"
          echo "Step: $STEP_VAR"
```

### Secrets

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Deploy with secrets
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
        run: |
          echo "Deploying with API key..."
          # Use $API_KEY and $DB_PASSWORD securely
```

**Adding Secrets:**
1. Go to Repository Settings
2. Click "Secrets and variables" → "Actions"
3. Click "New repository secret"
4. Add name and value

### Conditionals

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Deploy to Production
        if: github.ref == 'refs/heads/main'
        run: echo "Deploying to production..."
      
      - name: Deploy to Staging
        if: github.ref == 'refs/heads/develop'
        run: echo "Deploying to staging..."
      
      - name: Skip on PR
        if: github.event_name != 'pull_request'
        run: echo "Not a pull request"
```

### Matrix Strategy

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [14, 16, 18]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      
      - name: Run tests
        run: |
          echo "Testing on ${{ matrix.os }} with Node ${{ matrix.node-version }}"
          npm test
```

### Artifacts

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build application
        run: npm run build
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/
          retention-days: 7
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    
    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-output
          path: dist/
      
      - name: Deploy
        run: echo "Deploying artifacts..."
```

### Outputs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.get-version.outputs.version }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Get version
        id: get-version
        run: |
          VERSION=$(cat package.json | jq -r '.version')
          echo "version=$VERSION" >> $GITHUB_OUTPUT
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    
    steps:
      - name: Use version from build job
        run: echo "Deploying version ${{ needs.build.outputs.version }}"
```

---

## Windows Workflows

### Level 1: Basic Windows Commands

```yaml
# .github/workflows/windows-basic.yml
name: Windows Basic

on: [push]

jobs:
  windows-commands:
    runs-on: windows-latest
    
    steps:
      - name: System Information
        run: |
          echo "Computer Name: $env:COMPUTERNAME"
          echo "User: $env:USERNAME"
          echo "OS: $env:OS"
          systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
      
      - name: Directory Listing
        run: dir
      
      - name: Create Directory
        run: |
          mkdir test-folder
          cd test-folder
          echo "Hello" > hello.txt
          type hello.txt
```

### Level 2: Building .NET Application

```yaml
# .github/workflows/dotnet-build.yml
name: .NET Build

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: windows-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      
      - name: Restore dependencies
        run: dotnet restore
      
      - name: Build
        run: dotnet build --configuration Release --no-restore
      
      - name: Test
        run: dotnet test --no-build --verbosity normal --logger "trx"
      
      - name: Publish
        run: dotnet publish -c Release -o ./publish
      
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: dotnet-app
          path: ./publish
```

### Level 3: MSBuild (.NET Framework)

```yaml
# .github/workflows/msbuild.yml
name: MSBuild

on: [push]

jobs:
  build:
    runs-on: windows-2022
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup MSBuild
        uses: microsoft/setup-msbuild@v1.1
      
      - name: Setup NuGet
        uses: NuGet/setup-nuget@v1
      
      - name: Restore NuGet packages
        run: nuget restore MyApp.sln
      
      - name: Build with MSBuild
        run: msbuild MyApp.sln /p:Configuration=Release /p:Platform="Any CPU" /m
      
      - name: Run VSTest
        run: |
          vstest.console.exe MyApp.Tests\bin\Release\MyApp.Tests.dll /Logger:trx
```

### Level 4: Windows with IIS Deployment

```yaml
# .github/workflows/windows-iis.yml
name: Deploy to IIS

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: windows-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      
      - name: Build and Publish
        run: |
          dotnet restore
          dotnet build --configuration Release
          dotnet publish -c Release -o ./publish
      
      - name: Stop IIS Website
        run: |
          Import-Module WebAdministration
          Stop-WebSite -Name "MyWebsite"
        shell: pwsh
      
      - name: Deploy to IIS
        run: |
          $destination = "C:\inetpub\wwwroot\MyApp"
          
          # Backup
          if (Test-Path $destination) {
            $backup = "$destination`_backup_$(Get-Date -Format 'yyyyMMdd_HHmmss')"
            Copy-Item -Path $destination -Destination $backup -Recurse
          }
          
          # Deploy
          Remove-Item -Path "$destination\*" -Recurse -Force
          Copy-Item -Path ".\publish\*" -Destination $destination -Recurse -Force
        shell: pwsh
      
      - name: Start IIS Website
        run: |
          Import-Module WebAdministration
          Start-WebSite -Name "MyWebsite"
        shell: pwsh
```

### Level 5: Multi-Platform .NET

```yaml
# .github/workflows/dotnet-multiplatform.yml
name: .NET Multi-Platform

on: [push]

jobs:
  build:
    strategy:
      matrix:
        os: [windows-latest, ubuntu-latest, macos-latest]
        dotnet-version: ['6.0.x', '7.0.x', '8.0.x']
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ matrix.dotnet-version }}
      
      - name: Build
        run: dotnet build --configuration Release
      
      - name: Test
        run: dotnet test --configuration Release --no-build
```

---

## PowerShell in GitHub Actions

### Level 1: Basic PowerShell

```yaml
# .github/workflows/powershell-basic.yml
name: PowerShell Basics

on: [push]

jobs:
  powershell-job:
    runs-on: windows-latest
    
    steps:
      - name: PowerShell Version
        run: $PSVersionTable
        shell: pwsh
      
      - name: Variables and Output
        run: |
          $name = "GitHub Actions"
          $version = "1.0.0"
          Write-Host "Application: $name"
          Write-Host "Version: $version"
          
          # Get system info
          Get-ComputerInfo | Select-Object CsName, OsName, OsVersion
        shell: pwsh
      
      - name: File Operations
        run: |
          # Create file
          "Hello from PowerShell" | Out-File hello.txt
          
          # Read file
          Get-Content hello.txt
          
          # List files
          Get-ChildItem
        shell: pwsh
```

### Level 2: PowerShell with Environment Variables

```yaml
# .github/workflows/powershell-env.yml
name: PowerShell with Environment

on: [push]

env:
  APP_NAME: MyApplication
  ENVIRONMENT: Production

jobs:
  deploy:
    runs-on: windows-latest
    
    steps:
      - name: Use Environment Variables
        run: |
          Write-Host "Application: $env:APP_NAME"
          Write-Host "Environment: $env:ENVIRONMENT"
          Write-Host "Repository: $env:GITHUB_REPOSITORY"
          Write-Host "Workflow: $env:GITHUB_WORKFLOW"
        shell: pwsh
      
      - name: Set Output Variables
        id: vars
        run: |
          $buildNumber = "${{ github.run_number }}"
          $version = "1.0.$buildNumber"
          echo "version=$version" >> $env:GITHUB_OUTPUT
        shell: pwsh
      
      - name: Use Output Variable
        run: |
          Write-Host "Version: ${{ steps.vars.outputs.version }}"
        shell: pwsh
```

### Level 3: Advanced PowerShell Script

```yaml
# .github/workflows/powershell-advanced.yml
name: PowerShell Advanced

on: [push]

jobs:
  advanced-script:
    runs-on: windows-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Complex Deployment Script
        run: |
          # Configuration
          $config = @{
            AppName = "${{ github.event.repository.name }}"
            Version = "1.0.${{ github.run_number }}"
            Environment = "Production"
            DeployPath = "C:\Deploy"
          }
          
          Write-Host "=== Starting Deployment ==="
          Write-Host "App: $($config.AppName)"
          Write-Host "Version: $($config.Version)"
          
          # Function definitions
          function Write-Log {
              param([string]$Message, [string]$Level = "INFO")
              $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
              Write-Host "[$timestamp] [$Level] $Message"
          }
          
          function Test-Prerequisites {
              Write-Log "Checking prerequisites..."
              
              $checks = @(
                  @{ Name = "PowerShell"; Command = { $PSVersionTable.PSVersion.Major -ge 5 } }
                  @{ Name = "Disk Space"; Command = { (Get-PSDrive C).Free -gt 1GB } }
              )
              
              foreach ($check in $checks) {
                  try {
                      $result = & $check.Command
                      if ($result) {
                          Write-Log "$($check.Name) check passed" "SUCCESS"
                      } else {
                          Write-Log "$($check.Name) check failed" "ERROR"
                          exit 1
                      }
                  } catch {
                      Write-Log "Error checking $($check.Name): $_" "ERROR"
                      exit 1
                  }
              }
          }
          
          try {
              # Run checks
              Test-Prerequisites
              
              # Create deployment directory
              if (-not (Test-Path $config.DeployPath)) {
                  Write-Log "Creating deployment directory..."
                  New-Item -Path $config.DeployPath -ItemType Directory -Force | Out-Null
              }
              
              # Create version file
              $versionInfo = @{
                  AppName = $config.AppName
                  Version = $config.Version
                  DeployedAt = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
                  DeployedBy = $env:GITHUB_ACTOR
                  Commit = "${{ github.sha }}"
              } | ConvertTo-Json
              
              $versionInfo | Out-File "$($config.DeployPath)\version.json"
              Write-Log "Version file created" "SUCCESS"
              
              # Simulate deployment
              Write-Log "Deploying application..."
              Start-Sleep -Seconds 2
              
              Write-Log "Deployment completed successfully!" "SUCCESS"
              
          } catch {
              Write-Log "Deployment failed: $_" "ERROR"
              Write-Log "Stack Trace: $($_.ScriptStackTrace)" "ERROR"
              exit 1
          }
        shell: pwsh
```

### Level 4: PowerShell with Secrets

```yaml
# .github/workflows/powershell-secrets.yml
name: PowerShell with Secrets

on: [workflow_dispatch]

jobs:
  secure-deployment:
    runs-on: windows-latest
    
    steps:
      - name: Deploy with Credentials
        env:
          DB_CONNECTION: ${{ secrets.DB_CONNECTION_STRING }}
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          # Use secrets securely
          Write-Host "Connecting to database..."
          # Don't print secrets!
          
          if ([string]::IsNullOrEmpty($env:DB_CONNECTION)) {
              Write-Error "Database connection string is missing!"
              exit 1
          }
          
          Write-Host "✓ Database connection configured"
          Write-Host "✓ API key configured"
          
          # Use secrets in your deployment
          # $connectionString = $env:DB_CONNECTION
          # $apiKey = $env:API_KEY
        shell: pwsh
```

---

## Linux Workflows

### Level 1: Basic Shell Commands

```yaml
# .github/workflows/linux-basic.yml
name: Linux Basics

on: [push]

jobs:
  linux-commands:
    runs-on: ubuntu-latest
    
    steps:
      - name: System Information
        run: |
          echo "=== System Information ==="
          uname -a
          
          echo ""
          echo "=== OS Release ==="
          cat /etc/os-release
          
          echo ""
          echo "=== Current User ==="
          whoami
          
          echo ""
          echo "=== Working Directory ==="
          pwd
      
      - name: File Operations
        run: |
          echo "Creating test directory..."
          mkdir -p test-dir
          cd test-dir
          
          echo "Hello from GitHub Actions" > hello.txt
          cat hello.txt
          
          ls -la
```

### Level 2: Node.js Application

```yaml
# .github/workflows/nodejs.yml
name: Node.js CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm test
      
      - name: Run build
        run: npm run build
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        if: matrix.node-version == '20.x'
        with:
          file: ./coverage/coverage-final.json
      
      - name: Archive production artifacts
        if: matrix.node-version == '20.x'
        uses: actions/upload-artifact@v4
        with:
          name: dist-files
          path: dist/
```

### Level 3: Python Application

```yaml
# .github/workflows/python.yml
name: Python CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        python-version: ['3.8', '3.9', '3.10', '3.11']
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov flake8
      
      - name: Lint with flake8
        run: |
          # Stop the build if there are Python syntax errors or undefined names
          flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
          # Exit-zero treats all errors as warnings
          flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
      
      - name: Run tests with pytest
        run: |
          pytest --cov=src --cov-report=xml --cov-report=html
      
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
```

### Level 4: Docker Build and Push

```yaml
# .github/workflows/docker.yml
name: Docker Build and Push

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Log in to Container Registry
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.meta.outputs.version }}
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

### Level 5: Kubernetes Deployment

```yaml
# .github/workflows/k8s-deploy.yml
name: Deploy to Kubernetes

on:
  push:
    branches: [ main ]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy to'
        required: true
        type: choice
        options:
          - development
          - staging
          - production

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.environment || 'development' }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          method: kubeconfig
          kubeconfig: ${{ secrets.KUBE_CONFIG }}
      
      - name: Deploy to Kubernetes
        run: |
          # Update image tag
          sed -i 's|IMAGE_TAG|${{ github.sha }}|g' k8s/deployment.yaml
          
          # Apply manifests
          kubectl apply -f k8s/namespace.yaml
          kubectl apply -f k8s/configmap.yaml
          kubectl apply -f k8s/secret.yaml
          kubectl apply -f k8s/deployment.yaml
          kubectl apply -f k8s/service.yaml
          
          # Wait for rollout
          kubectl rollout status deployment/myapp -n ${{ github.event.inputs.environment }}
      
      - name: Verify deployment
        run: |
          kubectl get pods -n ${{ github.event.inputs.environment }}
          kubectl get svc -n ${{ github.event.inputs.environment }}
      
      - name: Run smoke tests
        run: |
          SERVICE_IP=$(kubectl get svc myapp -n ${{ github.event.inputs.environment }} -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
          curl -f http://$SERVICE_IP/health || exit 1
```

### Level 6: Terraform Infrastructure

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  push:
    branches: [ main ]
    paths:
      - 'terraform/**'
  pull_request:
    branches: [ main ]
    paths:
      - 'terraform/**'

jobs:
  terraform:
    runs-on: ubuntu-latest
    
    defaults:
      run:
        working-directory: ./terraform
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0
      
      - name: Terraform Format Check
        run: terraform fmt -check
      
      - name: Terraform Init
        run: terraform init
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      
      - name: Terraform Validate
        run: terraform validate
      
      - name: Terraform Plan
        run: terraform plan -no-color
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      
      - name: Terraform Apply
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: terraform apply -auto-approve
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## Advanced Concepts

### 1. Reusable Workflows

**Create reusable workflow:**

```yaml
# .github/workflows/reusable-build.yml
name: Reusable Build Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
      build-command:
        required: true
        type: string
    outputs:
      artifact-name:
        description: "Name of the uploaded artifact"
        value: ${{ jobs.build.outputs.artifact }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      artifact: ${{ steps.artifact.outputs.name }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      
      - run: npm ci
      - run: ${{ inputs.build-command }}
      
      - name: Upload artifact
        id: artifact
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ inputs.node-version }}
          path: dist/
```

**Use reusable workflow:**

```yaml
# .github/workflows/main.yml
name: Main Workflow

on: [push]

jobs:
  build-18:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '18'
      build-command: 'npm run build'
  
  build-20:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '20'
      build-command: 'npm run build'
```

### 2. Composite Actions

**Create composite action:**

```yaml
# .github/actions/setup-app/action.yml
name: 'Setup Application'
description: 'Setup Node.js and install dependencies'

inputs:
  node-version:
    description: 'Node.js version'
    required: true
    default: '18'

runs:
  using: 'composite'
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'
      shell: bash
    
    - name: Install dependencies
      run: npm ci
      shell: bash
    
    - name: Print versions
      run: |
        node --version
        npm --version
      shell: bash
```

**Use composite action:**

```yaml
name: Use Composite Action

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup application
        uses: ./.github/actions/setup-app
        with:
          node-version: '18'
      
      - name: Build
        run: npm run build
```

### 3. Manual Approval (Environments)

```yaml
# .github/workflows/deploy-with-approval.yml
name: Deploy with Approval

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: echo "Building..."
  
  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Deploy to Staging
        run: echo "Deploying to staging..."
  
  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production  # Requires approval in settings
    steps:
      - name: Deploy to Production
        run: echo "Deploying to production..."
```

**Configure environment protection:**
1. Go to Repository Settings → Environments
2. Click "New environment" or select existing
3. Add "Required reviewers"
4. Set "Wait timer" if needed

### 4. Caching Dependencies

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      # Cache Node modules
      - name: Cache Node modules
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - run: npm ci
      - run: npm run build
```

### 5. Status Checks and Branch Protection

```yaml
# .github/workflows/pr-checks.yml
name: PR Checks

on:
  pull_request:
    branches: [ main ]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run lint
  
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
  
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run security audit
        run: npm audit --audit-level=high
```

**Configure branch protection:**
1. Settings → Branches → Add rule
2. Branch name pattern: `main`
3. ✅ Require status checks to pass
4. Select required checks: lint, test, security

### 6. Scheduled Workflows

```yaml
# .github/workflows/nightly.yml
name: Nightly Build

on:
  schedule:
    # Runs at 00:00 UTC every day
    - cron: '0 0 * * *'
  workflow_dispatch:  # Allow manual trigger

jobs:
  nightly-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run nightly tests
        run: |
          npm ci
          npm run test:integration
          npm run test:e2e
      
      - name: Send notification
        if: failure()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

**Cron syntax:**
```
* * * * *
│ │ │ │ │
│ │ │ │ └─── Day of week (0-6, Sunday = 0)
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)

Examples:
'0 0 * * *'     # Daily at midnight UTC
'0 9 * * 1-5'   # Weekdays at 9 AM UTC
'0 */6 * * *'   # Every 6 hours
'0 0 1 * *'     # First day of month
```

### 7. GitHub Script

```yaml
jobs:
  comment-on-pr:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '👋 Thanks for the PR! Our CI is running...'
            })
```

### 8. Self-Hosted Runners

**Register self-hosted runner:**

```bash
# On your server (Linux)
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz
tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz

# Configure
./config.sh --url https://github.com/YOUR_ORG/YOUR_REPO --token YOUR_TOKEN

# Run
./run.sh

# Or install as service
sudo ./svc.sh install
sudo ./svc.sh start
```

**Use self-hosted runner:**

```yaml
jobs:
  build:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
      - run: echo "Running on self-hosted runner"
```

---

## Real-World Projects

### Project 1: Full-Stack Application (MERN)

```yaml
# .github/workflows/fullstack-cicd.yml
name: Full-Stack CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '18.x'
  DOCKER_REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Frontend job
  frontend:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./frontend
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Test
        run: npm test -- --coverage
      
      - name: Build
        run: npm run build
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: frontend-build
          path: frontend/build/
  
  # Backend job
  backend:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./backend
    
    services:
      mongodb:
        image: mongo:6
        ports:
          - 27017:27017
        env:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: password
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Run tests
        run: npm test
        env:
          MONGODB_URI: mongodb://root:password@localhost:27017/test?authSource=admin
      
      - name: Build
        run: npm run build
  
  # Build and push Docker images
  docker:
    needs: [frontend, backend]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    strategy:
      matrix:
        component: [frontend, backend]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download build artifacts
        if: matrix.component == 'frontend'
        uses: actions/download-artifact@v4
        with:
          name: frontend-build
          path: frontend/build/
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.DOCKER_REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}-${{ matrix.component }}
          tags: |
            type=sha
            type=ref,event=branch
            type=semver,pattern={{version}}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: ./${{ matrix.component }}
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
  
  # Deploy to staging
  deploy-staging:
    needs: docker
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to staging
        run: |
          echo "Deploying to staging..."
          # Add your deployment script here
  
  # Integration tests
  integration-tests:
    needs: deploy-staging
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run integration tests
        run: |
          npm ci
          npm run test:integration -- --env staging
  
  # Deploy to production (with approval)
  deploy-production:
    needs: integration-tests
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to production
        run: |
          echo "Deploying to production..."
          # Add your deployment script here
      
      - name: Send notification
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Production deployment completed!'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### Project 2: Microservices CI/CD

```yaml
# .github/workflows/microservices.yml
name: Microservices CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  # Detect changed services
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      auth: ${{ steps.changes.outputs.auth }}
      api: ${{ steps.changes.outputs.api }}
      frontend: ${{ steps.changes.outputs.frontend }}
      notification: ${{ steps.changes.outputs.notification }}
    steps:
      - uses: actions/checkout@v4
      
      - uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            auth:
              - 'services/auth/**'
            api:
              - 'services/api/**'
            frontend:
              - 'services/frontend/**'
            notification:
              - 'services/notification/**'
  
  # Build changed services
  build:
    needs: detect-changes
    if: |
      needs.detect-changes.outputs.auth == 'true' ||
      needs.detect-changes.outputs.api == 'true' ||
      needs.detect-changes.outputs.frontend == 'true' ||
      needs.detect-changes.outputs.notification == 'true'
    
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        service: [auth, api, frontend, notification]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Check if service changed
        id: check
        run: |
          if [ "${{ needs.detect-changes.outputs[matrix.service] }}" = "true" ]; then
            echo "changed=true" >> $GITHUB_OUTPUT
          fi
      
      - name: Build ${{ matrix.service }}
        if: steps.check.outputs.changed == 'true'
        working-directory: services/${{ matrix.service }}
        run: |
          docker build -t ${{ matrix.service }}:${{ github.sha }} .
          docker save ${{ matrix.service }}:${{ github.sha }} > /tmp/${{ matrix.service }}.tar
      
      - name: Upload image
        if: steps.check.outputs.changed == 'true'
        uses: actions/upload-artifact@v4
        with:
          name: ${{ matrix.service }}-image
          path: /tmp/${{ matrix.service }}.tar
  
  # Deploy services
  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        service: [auth, api, frontend, notification]
    
    steps:
      - name: Download image
        uses: actions/download-artifact@v4
        with:
          name: ${{ matrix.service }}-image
          path: /tmp/
      
      - name: Deploy ${{ matrix.service }}
        run: |
          docker load < /tmp/${{ matrix.service }}.tar
          # Deploy to Kubernetes
          kubectl set image deployment/${{ matrix.service }} \
            ${{ matrix.service }}=${{ matrix.service }}:${{ github.sha }} \
            -n production
```

### Project 3: Monorepo with Multiple Languages

```yaml
# .github/workflows/monorepo.yml
name: Monorepo CI/CD

on: [push, pull_request]

jobs:
  # Detect changes
  changes:
    runs-on: ubuntu-latest
    outputs:
      backend-go: ${{ steps.filter.outputs.backend-go }}
      backend-node: ${{ steps.filter.outputs.backend-node }}
      frontend: ${{ steps.filter.outputs.frontend }}
      mobile: ${{ steps.filter.outputs.mobile }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v2
        id: filter
        with:
          filters: |
            backend-go:
              - 'backend/go/**'
            backend-node:
              - 'backend/node/**'
            frontend:
              - 'frontend/**'
            mobile:
              - 'mobile/**'
  
  # Go backend
  backend-go:
    needs: changes
    if: needs.changes.outputs.backend-go == 'true'
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: backend/go
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      
      - name: Run tests
        run: go test -v ./...
      
      - name: Build
        run: go build -v ./...
  
  # Node.js backend
  backend-node:
    needs: changes
    if: needs.changes.outputs.backend-node == 'true'
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: backend/node
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: backend/node/package-lock.json
      
      - run: npm ci
      - run: npm test
      - run: npm run build
  
  # React frontend
  frontend:
    needs: changes
    if: needs.changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: frontend
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
  
  # React Native mobile
  mobile:
    needs: changes
    if: needs.changes.outputs.mobile == 'true'
    runs-on: macos-latest
    defaults:
      run:
        working-directory: mobile
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - run: npm ci
      - run: npm test
      
      - name: Build iOS
        run: |
          cd ios
          pod install
          xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release
```

---

## Best Practices

### 1. Workflow Organization

✅ **DO:**

```yaml
# Good: Clear naming and structure
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  lint:
    name: Code Linting
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run ESLint
        run: npm run lint
  
  test:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: npm test
```

❌ **DON'T:**

```yaml
# Bad: Unclear naming
name: stuff

on: push

jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - run: npm run lint
      - run: npm test
      - run: npm run build
      - run: deploy.sh
```

### 2. Security Best Practices

✅ **DO:**

```yaml
# Good: Use secrets properly
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
        run: ./deploy.sh
```

❌ **DON'T:**

```yaml
# Bad: Hardcoded secrets
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: |
          API_KEY="sk-1234567890"  # Never do this!
          ./deploy.sh
```

### 3. Caching Strategy

✅ **DO:**

```yaml
# Good: Cache dependencies
steps:
  - uses: actions/checkout@v4
  
  - name: Cache dependencies
    uses: actions/cache@v3
    with:
      path: ~/.npm
      key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
      restore-keys: |
        ${{ runner.os }}-node-
  
  - run: npm ci
  - run: npm run build
```

### 4. Matrix Testing

✅ **DO:**

```yaml
# Good: Test multiple versions
strategy:
  matrix:
    node-version: [16, 18, 20]
    os: [ubuntu-latest, windows-latest, macos-latest]
  fail-fast: false  # Don't cancel other jobs on failure
```

### 5. Use Concurrency Control

```yaml
# Prevent multiple workflows from running simultaneously
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

### 6. Limit Permissions

```yaml
# Use least privilege principle
permissions:
  contents: read
  packages: write
  
jobs:
  build:
    permissions:
      contents: read
      packages: write
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
```

### 7. Timeout Jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 30  # Prevent hung jobs
    steps:
      - run: npm run build
```

### 8. Use Dependency Review

```yaml
# .github/workflows/dependency-review.yml
name: Dependency Review

on: [pull_request]

permissions:
  contents: read

jobs:
  dependency-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/dependency-review-action@v3
```

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: Workflow Not Triggering

**Symptoms:**
- Workflow doesn't run on push
- No workflow runs visible

**Solutions:**

```yaml
# Check trigger configuration
on:
  push:
    branches:
      - main        # Make sure branch name matches
  pull_request:
    branches:
      - main

# Enable manual trigger for debugging
on:
  workflow_dispatch:
  push:
    branches: [ main ]
```

**Debugging steps:**
1. Check workflow file is in `.github/workflows/`
2. Verify YAML syntax (use YAML validator)
3. Check branch protection rules
4. Verify file has `.yml` or `.yaml` extension

#### Issue 2: Authentication Failed

**Symptoms:**
- Cannot push to registry
- Permission denied

**Solutions:**

```yaml
# For GitHub Container Registry
- name: Login to GHCR
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

# For Docker Hub
- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

#### Issue 3: Artifact Not Found

**Symptoms:**
- "Artifact not found" error
- Download fails

**Solutions:**

```yaml
# Upload artifact
- name: Upload artifact
  uses: actions/upload-artifact@v4
  with:
    name: my-artifact        # Must match download name
    path: dist/
    retention-days: 1

# Download in same workflow (different job)
- name: Download artifact
  uses: actions/download-artifact@v4
  with:
    name: my-artifact        # Same name as upload
    path: dist/
```

#### Issue 4: Environment Variables Not Working

**Symptoms:**
- Variables are empty
- "Variable not found"

**Solutions:**

```yaml
# Set environment variable
- name: Set variable
  run: echo "MY_VAR=hello" >> $GITHUB_ENV

# Use in next step
- name: Use variable
  run: echo $MY_VAR

# Or use job outputs
jobs:
  job1:
    outputs:
      my-output: ${{ steps.step1.outputs.value }}
    steps:
      - id: step1
        run: echo "value=hello" >> $GITHUB_OUTPUT
  
  job2:
    needs: job1
    steps:
      - run: echo ${{ needs.job1.outputs.my-output }}
```

#### Issue 5: Path Not Found

**Symptoms:**
- File or directory not found
- Wrong working directory

**Solutions:**

```yaml
# Check current directory
- name: Debug paths
  run: |
    pwd
    ls -la
    echo "GITHUB_WORKSPACE: $GITHUB_WORKSPACE"

# Use working-directory
- name: Build in subdirectory
  working-directory: ./backend
  run: npm run build

# Or use defaults
jobs:
  build:
    defaults:
      run:
        working-directory: ./backend
    steps:
      - run: npm install
      - run: npm run build
```

#### Issue 6: Out of Minutes

**Symptoms:**
- Workflow stops unexpectedly
- "No more minutes available"

**Solutions:**

1. **Check usage:**
   - Settings → Billing → Usage this month

2. **Optimize workflows:**
   ```yaml
   # Use caching
   - uses: actions/cache@v3
     with:
       path: ~/.npm
       key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
   
   # Use concurrency control
   concurrency:
     group: ${{ github.workflow }}-${{ github.ref }}
     cancel-in-progress: true
   
   # Add timeouts
   timeout-minutes: 10
   ```

3. **Use self-hosted runners** (unlimited minutes)

#### Issue 7: Rate Limiting

**Symptoms:**
- API rate limit exceeded
- 403 errors

**Solutions:**

```yaml
# Use GITHUB_TOKEN for API calls
- uses: actions/github-script@v7
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    script: |
      const { data } = await github.rest.repos.get({
        owner: context.repo.owner,
        repo: context.repo.repo
      });

# Add retry logic
- name: Retry on failure
  uses: nick-fields/retry@v2
  with:
    timeout_minutes: 10
    max_attempts: 3
    command: npm install
```

### Debugging Techniques

#### 1. Enable Debug Logging

```yaml
# Add to workflow
steps:
  - name: Enable debug
    run: echo "ACTIONS_STEP_DEBUG=true" >> $GITHUB_ENV
```

Or enable in repository settings:
- Settings → Secrets → Add `ACTIONS_STEP_DEBUG` = `true`

#### 2. Use tmate for SSH Debugging

```yaml
- name: Setup tmate session
  if: ${{ failure() }}
  uses: mxschmitt/action-tmate@v3
  timeout-minutes: 30
```

#### 3. Print All Variables

```yaml
- name: Dump GitHub context
  run: echo '${{ toJSON(github) }}'

- name: Dump job context
  run: echo '${{ toJSON(job) }}'

- name: Dump runner context
  run: echo '${{ toJSON(runner) }}'
```

---

## Practice Exercises

### Exercise 1: Basic CI
Create a workflow that:
1. Runs on push to any branch
2. Checks out code
3. Runs tests
4. Uploads test results

### Exercise 2: Multi-Environment Deploy
Create a workflow with:
1. Build job
2. Deploy to staging (automatic)
3. Deploy to production (manual approval)

### Exercise 3: Matrix Build
Create a workflow that tests:
1. Node.js versions: 16, 18, 20
2. Operating systems: Ubuntu, Windows, macOS
3. Uploads results from all combinations

### Exercise 4: Docker Pipeline
Create a complete Docker workflow:
1. Build image
2. Run tests in container
3. Push to registry
4. Deploy to environment

### Exercise 5: Monorepo
Create a workflow for monorepo:
1. Detect which packages changed
2. Build only changed packages
3. Run tests for changed packages
4. Deploy changed packages

---

## Next Steps

### Learning Resources

**Official Documentation:**
- GitHub Actions Docs: https://docs.github.com/actions
- Workflow Syntax: https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions
- Action Marketplace: https://github.com/marketplace?type=actions

**Tutorials:**
- GitHub Learning Lab
- FreeCodeCamp GitHub Actions Course
- YouTube tutorials

**Community:**
- GitHub Community Forum
- Stack Overflow
- Reddit r/github

### Advanced Topics to Explore

1. **Custom Actions**
   - JavaScript actions
   - Docker actions
   - Composite actions

2. **Self-Hosted Runners**
   - Setup and configuration
   - Autoscaling
   - Security

3. **GitHub Apps**
   - Create bots
   - Automate workflows
   - Integrate with external services

4. **Advanced Security**
   - CodeQL analysis
   - Dependabot
   - Secret scanning

---

## Quick Reference Card

### Common Workflow Patterns

**Basic CI:**
```yaml
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
```

**Conditional:**
```yaml
if: github.ref == 'refs/heads/main'
```

**Matrix:**
```yaml
strategy:
  matrix:
    node: [16, 18, 20]
steps:
  - uses: actions/setup-node@v4
    with:
      node-version: ${{ matrix.node }}
```

**Secrets:**
```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
```

**Artifacts:**
```yaml
- uses: actions/upload-artifact@v4
  with:
    name: build
    path: dist/
```

**Caching:**
```yaml
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

---

## Comparison: GitHub Actions vs Jenkins

| Feature | GitHub Actions | Jenkins |
|---------|---------------|---------|
| **Setup** | No setup required | Install & configure required |
| **Hosting** | Cloud or self-hosted | Self-hosted only |
| **Cost** | Free tier + paid plans | Free (hosting costs apply) |
| **Configuration** | YAML in repository | Jenkinsfile or UI |
| **Marketplace** | 20,000+ actions | 1,800+ plugins |
| **UI** | Modern, integrated | Classic, customizable |
| **Learning Curve** | Easy | Moderate |
| **Runners** | Ubuntu, Windows, macOS | Any platform |
| **Scalability** | Automatic | Manual setup |
| **Integration** | Native GitHub | External integration needed |

**When to use GitHub Actions:**
- ✅ GitHub-hosted projects
- ✅ Quick setup needed
- ✅ Standard CI/CD workflows
- ✅ Small to medium teams

**When to use Jenkins:**
- ✅ Complex custom requirements
- ✅ Multi-platform enterprise needs
- ✅ Existing Jenkins infrastructure
- ✅ Need full control

---

**Happy Automating! 🚀**

## Summary

You've learned:

✅ GitHub Actions fundamentals
✅ YAML workflow syntax
✅ Windows, PowerShell, and Linux workflows
✅ Docker and Kubernetes deployment
✅ Matrix builds and parallel execution
✅ Secrets and environment management
✅ Real-world project examples
✅ Best practices and troubleshooting

**You're now ready to:**
- Create CI/CD pipelines for any project
- Automate testing and deployment
- Use GitHub Actions marketplace
- Debug workflow issues
- Build production-ready workflows
