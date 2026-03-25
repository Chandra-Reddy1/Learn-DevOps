# SonarQube Interview Questions & Answers (Top 20 + Real-Time Setup)

---

# Top 20 Questions

---

## 1. What is SonarQube?

SonarQube is a code quality and security analysis tool used to detect bugs, vulnerabilities, and code smells.

---

## 2. What is a Sonar scan?

Process of analyzing source code using SonarQube to identify issues.

---

## 3. What is SonarQube architecture?

- SonarQube Server
- Database (PostgreSQL)
- Scanner (CLI / Maven / Gradle)

---

## 4. What is a Quality Gate?

A set of conditions that code must meet (e.g., no critical bugs) before passing.

---

## 5. What happens if Quality Gate fails?

- Build should fail in CI/CD
- Deployment is blocked

---

## 6. What is Sonar Scanner?

Tool used to send code analysis results to SonarQube server.

---

## 7. What is code smell?

Bad coding practice that may not break code but reduces maintainability.

---

## 8. What is bug vs vulnerability?

- Bug → affects functionality  
- Vulnerability → security issue  

---

## 9. What is coverage in SonarQube?

Percentage of code covered by tests.

---

## 10. What is duplication in SonarQube?

Repeated code blocks detected by analysis.

---

## 11. What is technical debt?

Effort required to fix code quality issues.

---

## 12. What is Sonar token?

Authentication token used to connect scanner with server.

---

## 13. What is branch analysis?

Analyzing different branches separately.

---

## 14. What is pull request analysis?

Analyzing code changes before merging.

---

## 15. What is rule in SonarQube?

Predefined coding standards used for analysis.

---

## 16. What is Quality Profile?

Collection of rules applied to a project.

---

## 17. What is issue severity?

- Blocker
- Critical
- Major
- Minor

---

## 18. What is incremental analysis?

Analyzing only changed code instead of full project.

---

## 19. What is SonarQube plugin?

Extends functionality (language support, integrations).

---

## 20. What is SonarCloud?

Cloud version of SonarQube.

---

# Bonus (High-Impact DevOps Questions)

---

## 21. How does SonarQube integrate with CI/CD?

- Code pushed → pipeline triggers
- Run Sonar scan
- Check Quality Gate
- Continue or fail pipeline

---

## 22. How to fail build if quality gate fails?

Use:
- SonarQube webhook
- Jenkins plugin / GitHub Action

---

## 23. What is sonar-project.properties?

Configuration file for scan:
- project key
- source path
- server URL

---

## 24. What is coverage tool integration?

JaCoCo, Istanbul, etc. provide coverage reports.

---

## 25. What is difference between SonarQube and lint tools?

- SonarQube → deep analysis + reporting  
- Lint → basic syntax checks  

---

## 26. How to secure SonarQube?

- Use authentication
- Restrict access
- Use HTTPS

---

## 27. What is webhook in SonarQube?

Notifies CI about Quality Gate result.

---

## 28. What is background task in SonarQube?

Processes analysis results asynchronously.

---

## 29. What is project key?

Unique identifier for project in SonarQube.

---

## 30. What is real DevOps use case?

- Code pushed → scan → quality gate → deploy

---

# Real-Time Integration (VERY IMPORTANT)

---

## Scenario: Integrating SonarQube in CI/CD (Jenkins Example)

### Step 1: Setup SonarQube Server
- Install SonarQube
- Configure database (PostgreSQL)
- Start server

---

### Step 2: Generate Token
- Login to SonarQube
- Create authentication token

---

### Step 3: Configure Jenkins
- Install SonarQube plugin
- Add SonarQube server in Jenkins config

---

### Step 4: Add Pipeline Stage

```groovy id="p8q2zb"
stage('SonarQube Scan') {
  steps {
    withSonarQubeEnv('SonarQube') {
      sh 'sonar-scanner'
    }
  }
}
