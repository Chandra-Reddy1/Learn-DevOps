# Jenkins Pipeline Commands for Windows Applications

## Basic Pipeline Structure

### Declarative Pipeline
```groovy
pipeline {
    agent {
        label 'windows'
    }
    stages {
        stage('Build') {
            steps {
                // Your commands here
            }
        }
    }
}
```

### Scripted Pipeline
```groovy
node('windows') {
    stage('Checkout') {
        // Your commands here
    }
}
```

## Source Control Commands

### Git Checkout
```groovy
checkout scm
// Or specific branch
git branch: 'main', url: 'https://github.com/user/repo.git'
```

## Windows Batch Commands

### Execute Batch Commands
```groovy
bat 'command'
bat '''
    echo Multi-line
    echo batch commands
'''
```

### Common Batch Examples
```groovy
// Build .NET application
bat 'msbuild MyProject.sln /p:Configuration=Release'

// Restore NuGet packages
bat 'nuget restore MyProject.sln'

// Run tests
bat 'vstest.console.exe Tests.dll'

// Clean build
bat 'msbuild MyProject.sln /t:Clean'
```

## PowerShell Commands

### Execute PowerShell
```groovy
powershell 'command'
powershell '''
    Write-Host "Multi-line"
    Write-Host "PowerShell commands"
'''
```

### Common PowerShell Examples
```groovy
// Build .NET Core application
powershell 'dotnet build MyProject.sln --configuration Release'

// Run tests
powershell 'dotnet test MyProject.Tests.csproj'

// Publish application
powershell 'dotnet publish -c Release -o ./publish'

// Archive files
powershell 'Compress-Archive -Path ./publish/* -DestinationPath app.zip'
```

## .NET Framework Commands

### MSBuild
```groovy
bat 'msbuild MyProject.sln /p:Configuration=Release /p:Platform="Any CPU"'
bat 'msbuild MyProject.sln /t:Rebuild /p:Configuration=Debug'
```

### NuGet
```groovy
bat 'nuget restore MyProject.sln'
bat 'nuget pack MyProject.nuspec'
```

## .NET Core / .NET 5+ Commands

### Build
```groovy
bat 'dotnet build MyProject.sln --configuration Release'
```

### Restore
```groovy
bat 'dotnet restore MyProject.sln'
```

### Test
```groovy
bat 'dotnet test MyProject.Tests.csproj --logger:trx'
```

### Publish
```groovy
bat 'dotnet publish -c Release -o ./output'
```

### Clean
```groovy
bat 'dotnet clean MyProject.sln'
```

## Node.js / JavaScript Commands

```groovy
bat 'npm install'
bat 'npm run build'
bat 'npm test'
bat 'npm run lint'
```

## Environment Variables

### Set Environment Variables
```groovy
environment {
    PATH = "C:\\Program Files\\nodejs;${env.PATH}"
    BUILD_CONFIG = 'Release'
}
```

### Use in Commands
```groovy
bat "msbuild MyProject.sln /p:Configuration=${BUILD_CONFIG}"
```

## File Operations

### Copy Files
```groovy
bat 'xcopy /s /y source\\* destination\\'
powershell 'Copy-Item -Path source\\* -Destination destination\\ -Recurse'
```

### Delete Files
```groovy
bat 'del /q /s build\\*'
powershell 'Remove-Item -Path build\\* -Recurse -Force'
```

### Create Directory
```groovy
bat 'mkdir output'
powershell 'New-Item -Path output -ItemType Directory -Force'
```

## Archive and Artifacts

### Archive Artifacts
```groovy
archiveArtifacts artifacts: 'output/**/*', fingerprint: true
archiveArtifacts artifacts: '**/*.exe, **/*.dll', excludes: '**/debug/**'
```

### Publish Test Results
```groovy
// MSTest/VSTest
step([$class: 'MSTestPublisher', testResultsFile: '**/*.trx'])

// NUnit
nunit testResultsPattern: '**/TestResult.xml'

// JUnit format
junit '**/test-results/*.xml'
```

## Docker Commands (Windows Containers)

```groovy
bat 'docker build -t myapp:latest .'
bat 'docker run -d -p 8080:80 myapp:latest'
bat 'docker push myapp:latest'
```

## Error Handling

### Try-Catch
```groovy
try {
    bat 'command-that-might-fail'
} catch (Exception e) {
    echo "Build failed: ${e.message}"
    currentBuild.result = 'FAILURE'
}
```

### Return Status
```groovy
def status = bat(returnStatus: true, script: 'command')
if (status != 0) {
    error("Command failed with status ${status}")
}
```

## Parallel Execution

```groovy
stage('Parallel Stage') {
    parallel {
        stage('Build') {
            steps {
                bat 'msbuild Project1.sln'
            }
        }
        stage('Test') {
            steps {
                bat 'dotnet test Project2.Tests.csproj'
            }
        }
    }
}
```

## Post Actions

```groovy
post {
    always {
        echo 'This runs always'
        cleanWs()
    }
    success {
        echo 'Build succeeded'
    }
    failure {
        echo 'Build failed'
        mail to: 'team@example.com', subject: 'Build Failed'
    }
}
```

## Complete Example Pipeline

```groovy
pipeline {
    agent {
        label 'windows'
    }
    
    environment {
        BUILD_CONFIG = 'Release'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Restore') {
            steps {
                bat 'dotnet restore MyApp.sln'
            }
        }
        
        stage('Build') {
            steps {
                bat "dotnet build MyApp.sln --configuration ${BUILD_CONFIG}"
            }
        }
        
        stage('Test') {
            steps {
                bat 'dotnet test MyApp.Tests.csproj --logger:trx'
            }
        }
        
        stage('Publish') {
            steps {
                bat "dotnet publish MyApp.csproj -c ${BUILD_CONFIG} -o ./publish"
            }
        }
        
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'publish/**/*', fingerprint: true
            }
        }
    }
    
    post {
        always {
            junit '**/TestResults/*.trx'
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

## Credentials and Secrets

### Using Credentials
```groovy
withCredentials([usernamePassword(credentialsId: 'my-credentials', 
                                   usernameVariable: 'USERNAME', 
                                   passwordVariable: 'PASSWORD')]) {
    bat "echo %USERNAME%"
    // Use credentials in your commands
}

// String credential
withCredentials([string(credentialsId: 'api-key', variable: 'API_KEY')]) {
    bat "curl -H \"Authorization: Bearer %API_KEY%\" https://api.example.com"
}

// SSH credentials
withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'SSH_KEY')]) {
    bat "ssh -i %SSH_KEY% user@server"
}
```

## Notifications

### Email Notifications
```groovy
emailext (
    subject: "Build ${currentBuild.result}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
    body: "Build details: ${env.BUILD_URL}",
    to: 'team@example.com',
    attachLog: true
)
```

### Slack Notifications
```groovy
slackSend (
    channel: '#builds',
    color: 'good',
    message: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
)
```

### Teams Notifications
```groovy
office365ConnectorSend (
    webhookUrl: 'https://outlook.office.com/webhook/...',
    message: "Build completed: ${env.JOB_NAME}"
)
```

## Parameters

### Build Parameters
```groovy
parameters {
    string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build')
    choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
    booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
    password(name: 'DEPLOY_PASSWORD', description: 'Deployment password')
}

// Use parameters
stages {
    stage('Build') {
        steps {
            echo "Building branch: ${params.BRANCH_NAME}"
            bat "dotnet build --configuration ${params.ENVIRONMENT}"
        }
    }
}
```

## Workspace Management

### Clean Workspace
```groovy
cleanWs()  // Clean entire workspace

// Clean before build
cleanWs deleteDirs: true, disableDeferredWipeout: true

// Clean specific patterns
cleanWs patterns: [[pattern: 'bin/**', type: 'INCLUDE']]
```

### Change Directory
```groovy
dir('subfolder') {
    bat 'dir'  // Executes in subfolder
}
```

### Workspace Path
```groovy
echo "Workspace: ${env.WORKSPACE}"
bat "cd %WORKSPACE%"
```

## Version Control

### Git Operations
```groovy
// Get commit info
def commitHash = bat(returnStdout: true, script: 'git rev-parse HEAD').trim()
def commitMessage = bat(returnStdout: true, script: 'git log -1 --pretty=%%B').trim()

// Tag creation
bat "git tag -a v1.0.0 -m 'Release 1.0.0'"
bat "git push origin v1.0.0"

// Branch operations
bat 'git checkout -b feature-branch'
bat 'git merge main'
```

## Conditional Execution

### When Conditions
```groovy
stage('Deploy to Production') {
    when {
        branch 'main'
        environment name: 'DEPLOY', value: 'true'
    }
    steps {
        bat 'deploy.bat'
    }
}

// Multiple conditions
stage('Test') {
    when {
        anyOf {
            branch 'main'
            branch 'develop'
        }
    }
    steps {
        bat 'run-tests.bat'
    }
}
```

### If-Else in Script
```groovy
script {
    if (env.BRANCH_NAME == 'main') {
        bat 'deploy-production.bat'
    } else {
        bat 'deploy-staging.bat'
    }
}
```

## Timeout and Retry

### Timeout
```groovy
timeout(time: 30, unit: 'MINUTES') {
    bat 'long-running-command.bat'
}
```

### Retry
```groovy
retry(3) {
    bat 'flaky-command.bat'
}
```

## Input and Approval

### Manual Approval
```groovy
stage('Deploy') {
    steps {
        input message: 'Deploy to production?', ok: 'Deploy'
        bat 'deploy.bat'
    }
}

// With parameters
input(
    message: 'Select deployment options',
    parameters: [
        choice(choices: ['blue', 'green'], name: 'DEPLOYMENT_TYPE')
    ]
)
```

## Build Triggers

### Trigger Configuration
```groovy
triggers {
    // Poll SCM every 5 minutes
    pollSCM('H/5 * * * *')
    
    // CRON schedule
    cron('0 0 * * *')  // Daily at midnight
    
    // Upstream trigger
    upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS)
}
```

## SonarQube Analysis

```groovy
stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('SonarQube') {
            bat '''
                dotnet sonarscanner begin /k:"project-key" /d:sonar.host.url="%SONAR_HOST_URL%"
                dotnet build
                dotnet sonarscanner end
            '''
        }
    }
}

// Quality Gate
stage('Quality Gate') {
    steps {
        timeout(time: 1, unit: 'HOURS') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

## Database Operations

```groovy
// SQL Server
bat '''
    sqlcmd -S server -d database -U user -P password -i script.sql
'''

// Entity Framework Migrations
bat 'dotnet ef database update --project MyProject.csproj'
bat 'dotnet ef migrations add InitialCreate'
```

## IIS Deployment

```groovy
// Stop IIS site
powershell 'Stop-WebSite -Name "MySite"'

// Copy files
bat 'xcopy /s /y publish\\* C:\\inetpub\\wwwroot\\myapp\\'

// Start IIS site
powershell 'Start-WebSite -Name "MySite"'

// Recycle App Pool
powershell 'Restart-WebAppPool -Name "MyAppPool"'
```

## Windows Service Management

```groovy
// Stop service
powershell 'Stop-Service -Name "MyService" -Force'

// Install service
bat 'sc create MyService binPath= "C:\\path\\to\\service.exe"'

// Start service
powershell 'Start-Service -Name "MyService"'

// Check service status
powershell 'Get-Service -Name "MyService"'
```

## Code Coverage

```groovy
// Generate coverage report
bat '''
    dotnet test --collect:"XPlat Code Coverage"
    reportgenerator -reports:**/coverage.cobertura.xml -targetdir:coverage
'''

// Publish coverage
publishCoverage adapters: [coberturaAdapter('coverage/Cobertura.xml')]
```

## Azure DevOps / Cloud

```groovy
// Azure CLI
bat 'az login --service-principal -u %CLIENT_ID% -p %CLIENT_SECRET% --tenant %TENANT_ID%'
bat 'az webapp deployment source config-zip --resource-group myRG --name myApp --src app.zip'

// AWS
bat 'aws s3 cp app.zip s3://my-bucket/'
```

## Performance Testing

```groovy
// JMeter
bat 'jmeter -n -t test.jmx -l results.jtl'

// Load test results
perfReport sourceDataFiles: 'results.jtl'
```

## Security Scanning

```groovy
// OWASP Dependency Check
bat 'dependency-check.bat --project MyApp --scan . --format HTML'

// Publish results
dependencyCheckPublisher pattern: 'dependency-check-report.xml'
```

## Registry Operations

```groovy
// Read registry
powershell 'Get-ItemProperty -Path "HKLM:\\Software\\MyApp"'

// Write registry
powershell 'Set-ItemProperty -Path "HKLM:\\Software\\MyApp" -Name Version -Value "1.0"'
```

## Common Build Variables

```groovy
// Jenkins environment variables
echo "Job Name: ${env.JOB_NAME}"
echo "Build Number: ${env.BUILD_NUMBER}"
echo "Build URL: ${env.BUILD_URL}"
echo "Workspace: ${env.WORKSPACE}"
echo "Branch: ${env.BRANCH_NAME}"
echo "Git Commit: ${env.GIT_COMMIT}"
echo "Node Name: ${env.NODE_NAME}"

// Build properties
currentBuild.displayName = "Build-${env.BUILD_NUMBER}"
currentBuild.description = "Built from ${env.BRANCH_NAME}"
```

## Tips

1. Use `bat` for simple Windows commands
2. Use `powershell` for more complex scripting
3. Always specify the Windows agent with `label 'windows'`
4. Use forward slashes or double backslashes in paths: `C:/path` or `C:\\path`
5. Add `returnStatus: true` to handle command failures gracefully
6. Use environment variables for configurations
7. Clean workspace with `cleanWs()` in post actions
8. Store sensitive data in Jenkins credentials, never in code
9. Use parameters for flexible pipeline execution
10. Add timeout to prevent hanging builds
11. Use parallel stages for faster builds
12. Always include proper error handling and notifications
