# Argo CD Interview Questions & Answers (Top 20 + Scenarios)

---

# Top 20 Questions

---

## 1. What is Argo CD?

Argo CD is a GitOps continuous delivery tool for Kubernetes that deploys applications using Git as the source of truth.

---

## 2. What is GitOps?

Managing infrastructure and applications using Git repositories as the single source of truth.

---

## 3. How does Argo CD work?

- Reads manifests from Git
- Compares with cluster state
- Syncs changes automatically or manually

---

## 4. What is an Argo CD Application?

A custom resource that defines:
- Git repo
- Target cluster
- Deployment path

---

## 5. What is sync in Argo CD?

Process of applying Git state to Kubernetes cluster.

---

## 6. What is auto-sync?

Automatically applies changes from Git to cluster without manual intervention.

---

## 7. What is drift in Argo CD?

Difference between:
- Git state
- Actual cluster state

---

## 8. How does Argo CD detect drift?

Continuously compares Git repo with live cluster state.

---

## 9. What is self-healing?

Automatically fixes drift by re-applying Git state.

---

## 10. What is pruning?

Deletes resources that are removed from Git.

---

## 11. What is rollback in Argo CD?

Reverting application to a previous Git commit.

---

## 12. What is Argo CD architecture?

Components:
- API Server
- Repository Server
- Application Controller

---

## 13. What is repo server?

Fetches manifests from Git repository.

---

## 14. What is application controller?

Monitors and syncs application state.

---

## 15. What is sync policy?

Defines behavior:
- Manual sync
- Auto-sync

---

## 16. What is Helm support in Argo CD?

Deploy Helm charts directly from Git.

---

## 17. What is Kustomize support?

Deploy customized Kubernetes manifests.

---

## 18. What is RBAC in Argo CD?

Controls access to applications and actions.

---

## 19. What is Argo CD UI used for?

Visualize applications, sync status, and health.

---

## 20. What is multi-cluster support?

Argo CD can deploy apps to multiple Kubernetes clusters.

---

# Bonus Questions

---

## 21. How do you secure Argo CD?

- Use RBAC
- Restrict Git access
- Enable authentication (SSO)

---

## 22. What happens if someone changes cluster manually?

Argo CD detects drift and:
- Reverts changes (if auto-sync enabled)

---

## 23. How to deploy Argo CD?

- Using Helm
- Using manifests

---

## 24. What is difference between CI and Argo CD?

- CI → builds artifacts
- Argo CD → deploys to Kubernetes

---

## 25. What is sync waves?

Controls order of resource deployment.

---

## 26. What is health status?

Indicates if application is running correctly.

---

## 27. What is application refresh?

Re-checks Git vs cluster state.

---

## 28. What is app-of-apps pattern?

One parent app manages multiple child applications.

---

## 29. What is declarative setup?

Define Argo CD apps using YAML instead of UI.

---

## 30. What is webhook in Argo CD?

Triggers sync when Git repo changes.

---

# Scenario-Based Questions (Very Important)

---

## 1. Drift Handling Scenario

### Question
A developer manually changes a Kubernetes resource. What happens?

### Answer
- Argo CD detects drift
- If auto-sync enabled → reverts to Git state
- Ensures Git remains source of truth

---

## 2. Deployment Failure Scenario

### Question
Deployment fails after sync. What do you do?

### Answer
- Check Argo CD UI (health/status)
- Check pod logs (kubectl logs)
- Rollback to previous version

---

## 3. Multi-Environment Deployment

### Question
How do you manage dev, QA, prod?

### Answer
- Separate Git folders or branches
- Use Argo CD applications per environment

---

## 4. Rollback Scenario

### Question
Production deployment breaks. How to fix?

### Answer
- Revert Git commit
- Argo CD syncs previous stable version

---

## 5. CI/CD Integration

### Question
How does Argo CD fit into pipeline?

### Answer
- CI builds image and pushes to registry
- Updates Git manifests
- Argo CD detects change and deploys

---

## 6. Access Control Scenario

### Question
Different teams need different access.

### Answer
- Use Argo CD RBAC policies
- Restrict access per project/app

---

## 7. Application Not Syncing

### Question
Argo CD not deploying changes.

### Answer
- Check repo connectivity
- Check sync policy
- Check logs

---

## 8. Secret Management

### Question
How to handle secrets in Argo CD?

### Answer
- Use Kubernetes Secrets
- Use external tools (Vault, Sealed Secrets)

---

## 9. High Availability Scenario

### Question
How to ensure Argo CD reliability?

### Answer
- Deploy in HA mode
- Multiple replicas of components

---

## 10. Large Scale Deployment

### Question
Managing many apps?

### Answer
- Use app-of-apps pattern
- Organize repos properly

---

# Final Interview Tips

- Always explain:
  - Git → Argo CD → Kubernetes flow
- Talk about:
  - drift handling
  - rollback
  - automation
- Give real example:
  - CI updates Git → Argo CD deploys
