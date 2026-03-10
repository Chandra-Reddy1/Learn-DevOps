# Python Sign-In App — Complete CI/CD Pipeline Guide

---

## Table of Contents

1. [Project Structure on Local Machine](#1-project-structure-on-local-machine)
2. [CI/CD Workflow with Scans](#2-cicd-workflow-with-scans)
3. [Docker Image Build & Push](#3-docker-image-build--push)
4. [Deploy to Kubernetes](#4-deploy-to-kubernetes)
5. [Configure Load Balancer & Endpoint URL](#5-configure-load-balancer--endpoint-url)
6. [Access the Web App as End User](#6-access-the-web-app-as-end-user)

---

## 1. Project Structure on Local Machine

Organize your Python sign-in app with this recommended structure before pushing to Git:

```
python-signin-app/
│
├── app/
│   ├── __init__.py
│   ├── main.py              # Flask/FastAPI sign-in app entry point
│   ├── routes.py            # Sign-in routes
│   └── templates/
│       └── signin.html      # Sign-in HTML page
│
├── tests/
│   ├── __init__.py
│   └── test_signin.py       # Unit tests
│
├── Dockerfile               # Docker build instructions
├── requirements.txt         # Python dependencies
├── .github/
│   └── workflows/
│       └── cicd.yml         # GitHub Actions CI/CD pipeline
└── k8s/
    ├── deployment.yaml      # Kubernetes Deployment
    ├── service.yaml         # Kubernetes Service (LoadBalancer)
    └── ingress.yaml         # (Optional) Ingress config
```

### Sample `main.py` (Flask)

```python
from flask import Flask, render_template, request

app = Flask(__name__)

@app.route("/", methods=["GET", "POST"])
def signin():
    if request.method == "POST":
        username = request.form.get("username")
        password = request.form.get("password")
        # Add authentication logic here
        return f"Welcome, {username}!"
    return render_template("signin.html")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### Sample `requirements.txt`

```
flask==3.0.0
pytest==8.0.0
flake8==7.0.0
bandit==1.7.8
```

### Push Code to GitHub

```bash
cd python-signin-app
git init
git add .
git commit -m "Initial commit: Python sign-in app"
git remote add origin https://github.com/<your-username>/python-signin-app.git
git push -u origin main
```

---

## 2. CI/CD Workflow with Scans

The pipeline runs **4 key stages** automatically on every push:

```
Code Push → Code Scan → Vulnerability Scan → Unit Tests → Build & Push Docker Image → Deploy to Kubernetes
```

### GitHub Actions Pipeline — `.github/workflows/cicd.yml`

```yaml
name: Python Sign-In App CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  DOCKER_IMAGE: ${{ secrets.DOCKERHUB_USERNAME }}/python-signin-app
  K8S_NAMESPACE: default

jobs:

  # ─────────────────────────────────────────
  # STAGE 1: Code Quality Scan (Flake8)
  # ─────────────────────────────────────────
  code-scan:
    name: 🔍 Code Quality Scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install Dependencies
        run: pip install flake8

      - name: Run Flake8 Code Scan
        run: flake8 app/ --max-line-length=120 --statistics

  # ─────────────────────────────────────────
  # STAGE 2: Security / Vulnerability Scan (Bandit + Safety)
  # ─────────────────────────────────────────
  vulnerability-scan:
    name: 🛡️ Security & Vulnerability Scan
    runs-on: ubuntu-latest
    needs: code-scan
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install Scanners
        run: pip install bandit safety

      - name: Run Bandit (SAST - Static Application Security Testing)
        run: bandit -r app/ -ll -ii

      - name: Run Safety (Dependency Vulnerability Check)
        run: safety check -r requirements.txt

  # ─────────────────────────────────────────
  # STAGE 3: Unit Tests (Pytest)
  # ─────────────────────────────────────────
  unit-tests:
    name: ✅ Unit Tests
    runs-on: ubuntu-latest
    needs: vulnerability-scan
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install Dependencies
        run: pip install -r requirements.txt

      - name: Run Unit Tests with Coverage
        run: |
          pip install pytest pytest-cov
          pytest tests/ -v --cov=app --cov-report=xml

      - name: Upload Coverage Report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage.xml

  # ─────────────────────────────────────────
  # STAGE 4: Build & Push Docker Image
  # ─────────────────────────────────────────
  docker-build-push:
    name: 🐳 Build & Push Docker Image
    runs-on: ubuntu-latest
    needs: unit-tests
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build Docker Image
        run: |
          docker build -t $DOCKER_IMAGE:${{ github.sha }} .
          docker tag $DOCKER_IMAGE:${{ github.sha }} $DOCKER_IMAGE:latest

      - name: Scan Docker Image (Trivy)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: "${{ env.DOCKER_IMAGE }}:latest"
          format: "table"
          exit-code: "1"
          severity: "CRITICAL,HIGH"

      - name: Push Docker Image
        run: |
          docker push $DOCKER_IMAGE:${{ github.sha }}
          docker push $DOCKER_IMAGE:latest

  # ─────────────────────────────────────────
  # STAGE 5: Deploy to Kubernetes
  # ─────────────────────────────────────────
  deploy:
    name: 🚀 Deploy to Kubernetes
    runs-on: ubuntu-latest
    needs: docker-build-push
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          method: kubeconfig
          kubeconfig: ${{ secrets.KUBECONFIG }}

      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f k8s/deployment.yaml
          kubectl apply -f k8s/service.yaml
          kubectl set image deployment/signin-app signin-app=$DOCKER_IMAGE:${{ github.sha }}
          kubectl rollout status deployment/signin-app
```

### GitHub Secrets to Configure

Go to: **GitHub Repo → Settings → Secrets and Variables → Actions**

| Secret Name          | Value                          |
|----------------------|-------------------------------|
| `DOCKERHUB_USERNAME` | Your Docker Hub username       |
| `DOCKERHUB_TOKEN`    | Docker Hub access token        |
| `KUBECONFIG`         | Base64-encoded kubeconfig file |

---

## 3. Docker Image Build & Push

### `Dockerfile`

```dockerfile
# Use official Python base image
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Copy dependencies
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY app/ .

# Expose port
EXPOSE 5000

# Run the application
CMD ["python", "main.py"]
```

### Build & Push Manually (Local Test)

```bash
# Build image
docker build -t <your-dockerhub-username>/python-signin-app:latest .

# Login to Docker Hub
docker login

# Push image
docker push <your-dockerhub-username>/python-signin-app:latest

# Test locally
docker run -p 5000:5000 <your-dockerhub-username>/python-signin-app:latest
# Open: http://localhost:5000
```

---

## 4. Deploy Image to Kubernetes

### `k8s/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: signin-app
  namespace: default
  labels:
    app: signin-app
spec:
  replicas: 2                        # Run 2 pods for high availability
  selector:
    matchLabels:
      app: signin-app
  template:
    metadata:
      labels:
        app: signin-app
    spec:
      containers:
        - name: signin-app
          image: <your-dockerhub-username>/python-signin-app:latest
          ports:
            - containerPort: 5000
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /
              port: 5000
            initialDelaySeconds: 10
            periodSeconds: 15
          readinessProbe:
            httpGet:
              path: /
              port: 5000
            initialDelaySeconds: 5
            periodSeconds: 10
```

### Apply Deployment

```bash
kubectl apply -f k8s/deployment.yaml
kubectl get pods                      # Verify pods are running
kubectl get deployments               # Check deployment status
```

---

## 5. Configure Load Balancer & Endpoint URL

### `k8s/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: signin-app-service
  namespace: default
spec:
  selector:
    app: signin-app
  type: LoadBalancer          # Provisions an external cloud Load Balancer
  ports:
    - protocol: TCP
      port: 80                # External port (users access on port 80)
      targetPort: 5000        # Internal container port
```

### Apply the Service

```bash
kubectl apply -f k8s/service.yaml

# Get the external endpoint URL
kubectl get service signin-app-service
```

### Sample Output

```
NAME                  TYPE           CLUSTER-IP     EXTERNAL-IP        PORT(S)        AGE
signin-app-service    LoadBalancer   10.96.45.123   203.0.113.45       80:31234/TCP   2m
```

> **Your app endpoint URL:** `http://203.0.113.45`

### On Cloud Providers

| Cloud        | Load Balancer Type                      | Notes                               |
|--------------|-----------------------------------------|-------------------------------------|
| **AWS EKS**  | AWS Classic / ALB via annotations       | Use `aws-load-balancer-controller`  |
| **GCP GKE**  | Google Cloud Load Balancer (auto)       | Automatically provisioned           |
| **Azure AKS**| Azure Load Balancer (auto)              | Automatically provisioned           |

### (Optional) Custom Domain with Ingress

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: signin-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: signin.yourdomain.com       # Replace with your domain
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: signin-app-service
                port:
                  number: 80
```

```bash
kubectl apply -f k8s/ingress.yaml
# Then point your domain DNS → EXTERNAL-IP of the Load Balancer
```

---

## 6. Access the Web App as End User

Once deployed, end users can access the sign-in page as follows:

### Step-by-Step Access

```bash
# 1. Get the external IP of your Load Balancer
kubectl get service signin-app-service

# 2. Copy the EXTERNAL-IP value from the output
# Example: 203.0.113.45
```

| Access Method         | URL Format                          |
|-----------------------|-------------------------------------|
| **Via IP (default)**  | `http://<EXTERNAL-IP>`              |
| **Via custom domain** | `http://signin.yourdomain.com`      |
| **With HTTPS/TLS**    | `https://signin.yourdomain.com`     |

### Verify Everything is Running

```bash
# Check all pods are healthy
kubectl get pods

# Check service and endpoint
kubectl get service signin-app-service
kubectl get endpoints signin-app-service

# View application logs
kubectl logs -l app=signin-app --tail=50

# Describe deployment for troubleshooting
kubectl describe deployment signin-app
```

---

## Complete Pipeline Flow Summary

```
┌─────────────────────────────────────────────────────────────────────┐
│                     CI/CD PIPELINE FLOW                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Developer        GitHub          CI/CD Pipeline                   │
│  Local PC    ──►  Push Code  ──►  ┌─────────────────┐             │
│                                   │ 1. Code Scan     │ (Flake8)    │
│                                   │ 2. Vuln Scan     │ (Bandit +   │
│                                   │                  │  Safety +   │
│                                   │                  │  Trivy)     │
│                                   │ 3. Unit Tests    │ (Pytest)    │
│                                   │ 4. Docker Build  │             │
│                                   │ 5. Docker Push   │ (DockerHub) │
│                                   │ 6. K8s Deploy    │             │
│                                   └────────┬────────┘             │
│                                            │                       │
│                                   Kubernetes Cluster               │
│                                   ┌────────▼────────┐             │
│                                   │  Deployment     │             │
│                                   │  (2 Pods)       │             │
│                                   └────────┬────────┘             │
│                                            │                       │
│                                   ┌────────▼────────┐             │
│                                   │ LoadBalancer    │             │
│                                   │ Service         │             │
│                                   └────────┬────────┘             │
│                                            │                       │
│                                   End User Browser                 │
│                                   http://<EXTERNAL-IP>             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Tools & Technologies Used

| Category             | Tool/Technology                        |
|----------------------|----------------------------------------|
| Code Language        | Python (Flask)                         |
| Version Control      | GitHub                                 |
| CI/CD                | GitHub Actions                         |
| Code Quality Scan    | Flake8                                 |
| Security SAST Scan   | Bandit                                 |
| Dependency Scan      | Safety                                 |
| Container Image Scan | Trivy (Aqua Security)                  |
| Unit Testing         | Pytest + pytest-cov                    |
| Containerization     | Docker + Docker Hub                    |
| Orchestration        | Kubernetes (K8s)                       |
| Load Balancer        | Kubernetes LoadBalancer Service        |
| Optional Domain      | NGINX Ingress Controller               |
