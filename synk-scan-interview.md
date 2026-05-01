# Snyk Interview Questions (Concept + Scenario Based)

---

## 1. What is Snyk?
- Security tool for scanning vulnerabilities in code, dependencies, containers, and IaC.

---

## 2. What are the types of scans in Snyk?
- Snyk Open Source (dependencies)
- Snyk Code (SAST)
- Snyk Container (image scan)
- Snyk IaC (Terraform, Kubernetes)

---

## 3. What is Snyk Open Source?
- Scans third-party dependencies for vulnerabilities.

---

## 4. What is Snyk Container?
- Scans Docker images for OS/package vulnerabilities.

---

## 5. What is Snyk Code?
- Static code analysis (SAST) for security issues.

---

## 6. What is Snyk IaC?
- Scans infrastructure code for misconfigurations.

---

## 7. What is a vulnerability severity?
- Low, Medium, High, Critical.

---

## 8. What is a fixable vulnerability?
- A vulnerability with an available patch/version fix.

---

## 9. What is `snyk test`?
- Runs scan locally or in CI.

---

## 10. What is `snyk monitor`?
- Sends results to Snyk dashboard for continuous monitoring.

---

# 🔥 Scenario-Based Questions

---

## 11. Build fails due to high vulnerabilities — what will you do?
- Identify vulnerable dependency
- Upgrade version or apply patch
- Ignore only if justified

---

## 12. Too many vulnerabilities in report
- Prioritize critical/high
- Focus on fixable issues
- Use ignore rules carefully

---

## 13. Snyk scan is slow
- Optimize project size
- Exclude unnecessary files
- Use caching

---

## 14. Container image has vulnerabilities
- Use smaller base image (alpine)
- Update packages
- Rebuild image

---

## 15. Snyk not detecting dependencies
- Check manifest files (package.json, pom.xml)
- Ensure proper build context

---

## 16. False positives in Snyk
- Verify vulnerability details
- Use `.snyk` ignore file

---

## 17. Pipeline not failing on vulnerabilities
- Configure threshold:
  snyk test --severity-threshold=high

---

## 18. Snyk authentication failure
- Check API token
- Run: snyk auth

---

## 19. IaC misconfiguration detected
- Fix Terraform/K8s config
- Re-run scan

---

## 20. Need to enforce security in CI/CD
- Integrate Snyk in pipeline
- Fail build on critical issues

---

# 🔥 Important Commands

- snyk test
- snyk monitor
- snyk container test
- snyk code test
- snyk iac test

---

# 🚀 Interview One-Liner

"Snyk is a developer-first security tool that scans code, dependencies, containers, and infrastructure for vulnerabilities and integrates directly into CI/CD pipelines to enforce security."
