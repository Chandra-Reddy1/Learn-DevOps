# ☸️ Kubernetes — All Resource Types Explained
### Application: `chandra219/welcome-login-app:66` (Flask app on port 5000)

> Each YAML file below is production-ready, fully annotated, and directly tied to **your application**.
> Every concept is explained in plain English before showing the YAML.

---

## 📁 File Index

| # | Resource | File | Purpose |
|---|----------|------|---------|
| 1 | [ConfigMap](#1-configmap) | `01-configmap.yaml` | Store app configuration (env vars, settings) |
| 2 | [Secret](#2-secret) | `02-secret.yaml` | Store sensitive data (passwords, keys) |
| 3 | [Pod](#3-pod) | `03-pod.yaml` | Single instance of your Flask app |
| 4 | [ReplicaSet](#4-replicaset) | `04-replicaset.yaml` | Keep N copies of your Pod always running |
| 5 | [Deployment](#5-deployment) | `05-deployment.yaml` | Manage rolling updates + rollbacks |
| 6 | [StatefulSet](#6-statefulset) | `06-statefulset.yaml` | Run a stateful DB alongside your app |
| 7 | [DaemonSet](#7-daemonset) | `07-daemonset.yaml` | Run a log-collector on every node |
| 8 | [Job](#8-job) | `08-job.yaml` | Run a one-time DB migration task |
| 9 | [Service](#9-service) | `09-service.yaml` | Expose your app inside/outside the cluster |
| 10 | [Ingress](#10-ingress) | `10-ingress.yaml` | Route HTTP traffic via domain names |

---

## Architecture Overview

```
 Internet / Browser
        │
        ▼
  [ Ingress ]  ← routes myapp.local/  →  [ Service: nginx-demo (NodePort) ]
                                                      │
                              ┌───────────────────────┤
                              ▼                       ▼
                         [ Pod 1 ]              [ Pod 2 ]         ← managed by Deployment
                     Flask app:5000          Flask app:5000
                         │                       │
                         └────────┬──────────────┘
                                  ▼
                       [ ConfigMap + Secret ]  ← env vars injected into pods
                                  │
                                  ▼
                     [ StatefulSet: MySQL Pod ]  ← persistent DB
                                  │
                       [ PersistentVolume ]
```

---

## 1. ConfigMap

### 📖 What is it?
A **ConfigMap** stores **non-sensitive configuration** as key-value pairs and injects them into your Pods as environment variables or files. This way you don't hardcode config inside your Docker image.

### 🤔 Why use it for YOUR app?
Your Flask app likely needs to know things like:
- Which environment it's running in (dev/prod)
- The database hostname
- Logging level

Instead of rebuilding the Docker image every time config changes — just update the ConfigMap!

### 📄 `01-configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: flask-app-config          # Name used to reference this ConfigMap in Pods
  namespace: default
  labels:
    app: nginx-demo               # Same label as your app for grouping

data:
  # ── App Settings ──────────────────────────────────────────
  APP_ENV: "production"           # Tells Flask which environment it's running in
  APP_PORT: "5000"                # The port Flask listens on (matches containerPort)
  LOG_LEVEL: "INFO"               # Logging verbosity: DEBUG, INFO, WARNING, ERROR

  # ── Database Settings ─────────────────────────────────────
  DB_HOST: "mysql-0.mysql.default.svc.cluster.local"
  #         └─ pod name ─┘└─ service ─┘└─ namespace + domain ─┘
  DB_PORT: "3306"
  DB_NAME: "welcomeapp"

  # ── Feature Flags ─────────────────────────────────────────
  ENABLE_SIGNUP: "true"           # Turn signup on/off without redeploying
  MAX_UPLOAD_SIZE: "10MB"
```

### ✅ How to apply & verify

```bash
# Apply the ConfigMap
kubectl apply -f 01-configmap.yaml

# View it
kubectl get configmap flask-app-config
kubectl describe configmap flask-app-config

# See all key-value pairs
kubectl get configmap flask-app-config -o yaml
```

---

## 2. Secret

### 📖 What is it?
A **Secret** is like a ConfigMap but for **sensitive data** (passwords, API keys, tokens). Values are **base64-encoded** (not encrypted by default — for real encryption, use Sealed Secrets or Vault). Kubernetes restricts access to Secrets via RBAC.

### 🤔 Why use it for YOUR app?
Your login app needs:
- A database password
- A Flask secret key (for session signing)
- Any API tokens

Never put these in your YAML in plain text — use Secrets!

### 📄 `02-secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: flask-app-secret          # Name referenced by Pods
  namespace: default
  labels:
    app: nginx-demo

# Opaque = generic key-value secret (most common type)
# Other types: kubernetes.io/tls, kubernetes.io/dockerconfigjson
type: Opaque

data:
  # ⚠️  Values MUST be base64-encoded
  # To encode: echo -n "mypassword" | base64
  # To decode: echo "bXlwYXNzd29yZA==" | base64 --decode

  DB_PASSWORD: "bXlzcWxwYXNzMTIz"      # mysqlpass123
  FLASK_SECRET_KEY: "c3VwZXJzZWNyZXRrZXkxMjM="  # supersecretkey123
  ADMIN_PASSWORD: "YWRtaW4xMjM="        # admin123

# 💡 TIP: In production use stringData instead (auto-encodes for you):
# stringData:
#   DB_PASSWORD: "mysqlpass123"          ← plain text, Kubernetes encodes it
#   FLASK_SECRET_KEY: "supersecretkey123"
```

### ✅ How to apply & verify

```bash
# Apply
kubectl apply -f 02-secret.yaml

# List secrets (values are hidden)
kubectl get secrets

# Decode a value to verify
kubectl get secret flask-app-secret -o jsonpath='{.data.DB_PASSWORD}' | base64 --decode
```

---

## 3. Pod

### 📖 What is it?
A **Pod** is the **smallest deployable unit** in Kubernetes. It wraps one or more containers that share the same network and storage. Think of it as a "wrapper around your Docker container."

### 🤔 Why NOT use a raw Pod in production?
Pods are **not self-healing**. If this Pod crashes, it's gone — Kubernetes does NOT restart it. That's why we use Deployments. But understanding Pods is fundamental.

### 📄 `03-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: flask-app-pod             # Unique name for this Pod
  namespace: default
  labels:
    app: nginx-demo               # Label used by Services to find this Pod
    version: "66"                 # Track which image version is running
    tier: frontend

spec:
  # ── Where to pull the image from ──────────────────────────
  containers:
    - name: flask-app             # Container name (used in logs, exec commands)
      image: chandra219/welcome-login-app:66   # YOUR Docker image
      imagePullPolicy: Always     # Always pull latest (use IfNotPresent in prod)

      # ── Port your Flask app listens on ──────────────────
      ports:
        - containerPort: 5000     # Flask port — must match your app
          name: http              # Human-readable port name

      # ── Inject ConfigMap values as environment variables ──
      envFrom:
        - configMapRef:
            name: flask-app-config    # All keys from ConfigMap become env vars
        - secretRef:
            name: flask-app-secret    # All keys from Secret become env vars

      # ── Resource limits: prevent one pod hogging the node ─
      resources:
        requests:                 # Minimum guaranteed resources
          cpu: "100m"             # 100 millicores = 0.1 CPU core
          memory: "128Mi"         # 128 Megabytes RAM
        limits:                   # Maximum allowed resources
          cpu: "500m"             # 0.5 CPU core max
          memory: "256Mi"         # 256 MB RAM max — if exceeded, Pod is OOMKilled

      # ── Health Checks ─────────────────────────────────────
      livenessProbe:              # Is the app alive? If fails → restart container
        httpGet:
          path: /                 # Flask homepage route
          port: 5000
        initialDelaySeconds: 15   # Wait 15s after start before first check
        periodSeconds: 10         # Check every 10 seconds
        failureThreshold: 3       # Restart after 3 consecutive failures

      readinessProbe:             # Is app ready to receive traffic? If fails → remove from Service
        httpGet:
          path: /
          port: 5000
        initialDelaySeconds: 5
        periodSeconds: 5

  # ── Restart policy ────────────────────────────────────────
  restartPolicy: Always           # Always, OnFailure, Never
                                  # Always = restart on any exit (crash or 0)
```

### ✅ How to apply & verify

```bash
kubectl apply -f 03-pod.yaml
kubectl get pods
kubectl describe pod flask-app-pod
kubectl logs flask-app-pod
kubectl exec -it flask-app-pod -- /bin/sh    # Shell into the container
kubectl delete pod flask-app-pod            # Notice: it stays deleted (not self-healing)
```

---

## 4. ReplicaSet

### 📖 What is it?
A **ReplicaSet** ensures a **specified number of Pod replicas are always running**. If a Pod crashes, the ReplicaSet creates a new one. If you manually delete a Pod, it's recreated automatically.

### 🤔 When to use directly?
In practice, you **rarely create ReplicaSets directly** — Deployments manage them for you. But understanding ReplicaSets helps you understand how Deployments work internally.

### 📄 `04-replicaset.yaml`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: flask-app-rs              # ReplicaSet name
  namespace: default
  labels:
    app: nginx-demo

spec:
  replicas: 3                     # Keep EXACTLY 3 Pods running at all times
                                  # Delete 1 → immediately creates a new one
                                  # Scale: kubectl scale rs flask-app-rs --replicas=5

  # ── This selector tells RS which Pods it "owns" ───────────
  selector:
    matchLabels:
      app: nginx-demo             # RS adopts any Pod with this label
      tier: frontend

  # ── Pod template: what each replica looks like ────────────
  template:
    metadata:
      labels:
        app: nginx-demo           # ⚠️ MUST match selector.matchLabels above
        tier: frontend
    spec:
      containers:
        - name: flask-app
          image: chandra219/welcome-login-app:66
          ports:
            - containerPort: 5000
          envFrom:
            - configMapRef:
                name: flask-app-config
            - secretRef:
                name: flask-app-secret
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

### ✅ How to apply & verify

```bash
kubectl apply -f 04-replicaset.yaml

# See all 3 pods created
kubectl get pods -l app=nginx-demo

# Try deleting one — watch RS recreate it immediately!
kubectl delete pod <pod-name>
kubectl get pods -l app=nginx-demo    # New Pod appears within seconds

# Scale up
kubectl scale rs flask-app-rs --replicas=5
kubectl get pods

# ⚠️ Limitation: Change the image tag → RS does NOT roll out the update!
# That's why we use Deployments
```

---

## 5. Deployment

### 📖 What is it?
A **Deployment** is the most common way to run stateless apps in Kubernetes. It manages a ReplicaSet and adds:
- **Rolling updates**: Replace old Pods with new ones gradually (zero downtime)
- **Rollbacks**: Instantly revert to the previous working version
- **History**: Track all past deployments

### 🤔 Why use it for YOUR app?
Your Flask welcome-login app is **stateless** (each request is independent) — perfect for Deployments. When you push a new image like `:67`, you do a rolling update with zero downtime.

### 📄 `05-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-demo                # Same name as your original deployment
  namespace: default
  labels:
    app: nginx-demo
  annotations:
    # Document what this deployment does
    description: "Flask welcome-login app deployment with rolling updates"

spec:
  replicas: 2                     # Run 2 Pods (as in your original yaml)

  # ── Rolling Update Strategy ───────────────────────────────
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1                 # Allow 1 extra Pod during update (total = 3 temporarily)
      maxUnavailable: 0           # Never kill old Pod before new one is ready (ZERO DOWNTIME)
      # Result: starts Pod with :67 → waits for it to pass readiness → kills old :66 Pod

  # ── Which Pods this Deployment manages ────────────────────
  selector:
    matchLabels:
      app: nginx-demo

  # ── Pod Template ──────────────────────────────────────────
  template:
    metadata:
      labels:
        app: nginx-demo
      annotations:
        # Forces pod restart even if only ConfigMap changed (not the image)
        configmap-checksum: "abc123"   # Update this when ConfigMap changes

    spec:
      # ── Graceful shutdown: let in-flight requests finish ──
      terminationGracePeriodSeconds: 30

      containers:
        - name: flask-app
          image: chandra219/welcome-login-app:66   # Change tag here to trigger rolling update

          ports:
            - containerPort: 5000
              name: http

          # ── Load all config and secrets ───────────────────
          envFrom:
            - configMapRef:
                name: flask-app-config
            - secretRef:
                name: flask-app-secret

          # ── Override specific env vars if needed ──────────
          env:
            - name: POD_NAME          # Inject Pod's own name (useful for logging)
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace

          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"

          # ── Readiness: don't send traffic until Flask is up ─
          readinessProbe:
            httpGet:
              path: /
              port: 5000
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 3

          # ── Liveness: restart if Flask is stuck/dead ──────
          livenessProbe:
            httpGet:
              path: /
              port: 5000
            initialDelaySeconds: 15
            periodSeconds: 10
            failureThreshold: 3
```

### ✅ How to apply & verify

```bash
kubectl apply -f 05-deployment.yaml

# Watch rolling update in real time
kubectl rollout status deployment/nginx-demo

# Update to new image version (triggers rolling update)
kubectl set image deployment/nginx-demo flask-app=chandra219/welcome-login-app:67

# Watch pods cycle out one by one
kubectl get pods -w

# View revision history
kubectl rollout history deployment/nginx-demo

# ROLLBACK if something goes wrong!
kubectl rollout undo deployment/nginx-demo

# Rollback to specific revision
kubectl rollout undo deployment/nginx-demo --to-revision=2
```

---

## 6. StatefulSet

### 📖 What is it?
A **StatefulSet** manages stateful applications that need:
- **Stable, unique Pod names**: `mysql-0`, `mysql-1` (not random names)
- **Stable network identity**: `mysql-0.mysql.default.svc.cluster.local`
- **Dedicated storage**: Each Pod gets its own PersistentVolumeClaim (data survives Pod restarts)
- **Ordered startup/shutdown**: `mysql-0` starts before `mysql-1`

### 🤔 Why use it for YOUR app?
Your Flask app **stores login data in a database (MySQL)**. The DB must be a StatefulSet because:
- It needs persistent storage (user data must survive Pod restarts)
- Replicas need stable hostnames (primary vs replica in replication setup)

### 📄 `06-statefulset.yaml`

```yaml
# ── Headless Service (required for StatefulSet DNS) ─────────
# Each Pod gets a DNS: mysql-0.mysql.default.svc.cluster.local
apiVersion: v1
kind: Service
metadata:
  name: mysql                     # Must match StatefulSet's serviceName
  namespace: default
spec:
  clusterIP: None                 # "Headless" — no load balancing, direct Pod DNS
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306

---
# ── StatefulSet for MySQL ────────────────────────────────────
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql                     # Pods will be named: mysql-0, mysql-1, mysql-2
  namespace: default

spec:
  serviceName: "mysql"            # Must match the headless Service above
  replicas: 1                     # 1 MySQL instance (increase for read replicas)

  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql

    spec:
      containers:
        - name: mysql
          image: mysql:8.0

          ports:
            - containerPort: 3306
              name: mysql

          # ── MySQL environment variables ──────────────────
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: flask-app-secret  # Use password from our Secret
                  key: DB_PASSWORD

            - name: MYSQL_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: flask-app-config  # DB name from our ConfigMap
                  key: DB_NAME

          # ── Mount the persistent volume at MySQL data dir ─
          volumeMounts:
            - name: mysql-data          # Must match volumeClaimTemplates name below
              mountPath: /var/lib/mysql  # MySQL stores all data here

          resources:
            requests:
              cpu: "250m"
              memory: "512Mi"
            limits:
              cpu: "1000m"
              memory: "1Gi"

  # ── PVC Template: each Pod gets its OWN dedicated PVC ──────
  # mysql-0 → mysql-data-mysql-0 (10Gi PVC)
  # mysql-1 → mysql-data-mysql-1 (10Gi PVC) — if replicas > 1
  volumeClaimTemplates:
    - metadata:
        name: mysql-data
      spec:
        accessModes:
          - ReadWriteOnce           # Only one Pod can write at a time (standard for DB)
        resources:
          requests:
            storage: 10Gi          # 10 GB per MySQL Pod
        # storageClassName: standard  # Uncomment and set for your cloud provider
```

### ✅ How to apply & verify

```bash
kubectl apply -f 06-statefulset.yaml

# Watch ordered startup (mysql-0 starts first)
kubectl get pods -w -l app=mysql

# Stable DNS — connect from Flask app using:
# DB_HOST = mysql-0.mysql.default.svc.cluster.local

# Check the PVC created for mysql-0
kubectl get pvc

# Shell into MySQL
kubectl exec -it mysql-0 -- mysql -u root -p

# DELETE the pod — StatefulSet recreates it WITH THE SAME DATA
kubectl delete pod mysql-0
kubectl get pvc   # PVC mysql-data-mysql-0 still exists with all data!
```

---

## 7. DaemonSet

### 📖 What is it?
A **DaemonSet** ensures that **exactly one copy of a Pod runs on every node** in the cluster. As nodes are added, Pods are automatically scheduled on them. As nodes are removed, Pods are garbage collected.

### 🤔 Why use it for YOUR app?
Your Flask app runs on multiple nodes. You want to **collect logs from every node** using Fluentd (a log shipper). It must run on ALL nodes — a DaemonSet is perfect for this.

Other use cases: Prometheus Node Exporter (metrics), security agents, network plugins.

### 📄 `07-daemonset.yaml`

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-log-collector     # Runs on EVERY node
  namespace: kube-system          # System namespace for cluster-level services
  labels:
    app: fluentd
    component: logging

spec:
  selector:
    matchLabels:
      app: fluentd

  # ── How to update DaemonSet pods ──────────────────────────
  updateStrategy:
    type: RollingUpdate           # Update one node at a time
    rollingUpdate:
      maxUnavailable: 1           # At most 1 node can be without the daemon at a time

  template:
    metadata:
      labels:
        app: fluentd

    spec:
      # ── Allow running on control-plane nodes too ──────────
      tolerations:
        - key: node-role.kubernetes.io/control-plane
          operator: Exists
          effect: NoSchedule
        - key: node-role.kubernetes.io/master
          operator: Exists
          effect: NoSchedule

      # ── Run as root to access host log files ──────────────
      # (normally you'd avoid root, but log collection needs it)
      containers:
        - name: fluentd
          image: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
          env:
            # Tell Fluentd where to ship logs (your app name for filtering)
            - name: APP_NAME
              value: "nginx-demo"
            - name: FLUENT_ELASTICSEARCH_HOST
              value: "elasticsearch.kube-system.svc.cluster.local"
            - name: FLUENT_ELASTICSEARCH_PORT
              value: "9200"

          resources:
            requests:
              cpu: "100m"
              memory: "200Mi"
            limits:
              cpu: "500m"
              memory: "500Mi"

          # ── Mount host paths to collect node-level logs ───
          volumeMounts:
            - name: varlog
              mountPath: /var/log          # All system logs
              readOnly: true
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers   # Container logs
              readOnly: true
            - name: fluentd-config
              mountPath: /fluentd/etc/     # Fluentd config files

      # ── Host volumes: read node's actual log directories ──
      volumes:
        - name: varlog
          hostPath:
            path: /var/log                 # Node's /var/log directory

        - name: varlibdockercontainers
          hostPath:
            path: /var/lib/docker/containers

        - name: fluentd-config
          configMap:
            name: fluentd-config           # Fluentd config (separate ConfigMap)
```

### ✅ How to apply & verify

```bash
kubectl apply -f 07-daemonset.yaml

# See one Fluentd pod on each node
kubectl get pods -n kube-system -l app=fluentd -o wide
# Output shows each pod on a DIFFERENT NODE

# Check logs from one fluentd pod
kubectl logs -n kube-system <fluentd-pod-name>

# Count: pods should equal number of nodes
kubectl get nodes --no-headers | wc -l      # Node count
kubectl get pods -n kube-system -l app=fluentd --no-headers | wc -l  # Should match!
```

---

## 8. Job

### 📖 What is it?
A **Job** creates one or more Pods and ensures they **run to completion** (exit code 0). Unlike Deployments (which run forever), Jobs are for **one-time or batch tasks**. Once complete, the Job is done — it doesn't restart.

A **CronJob** is a Job that runs on a schedule (like cron in Linux).

### 🤔 Why use it for YOUR app?
Before deploying a new version of your Flask app, you need to:
- **Run database migrations** (`flask db upgrade`)
- **Seed initial data** (create admin user, populate lookup tables)
- **Send batch emails** to all registered users

These are all one-time tasks — perfect for Jobs!

### 📄 `08-job.yaml`

```yaml
# ── One-time Job: Run DB migrations before deployment ────────
apiVersion: batch/v1
kind: Job
metadata:
  name: flask-db-migration        # Unique job name
  namespace: default
  labels:
    app: nginx-demo
    job-type: migration

spec:
  # ── Retry settings ────────────────────────────────────────
  backoffLimit: 3                 # Retry up to 3 times if the job fails
  activeDeadlineSeconds: 300      # Kill the job if it runs > 5 minutes (safety)

  # ── Parallelism (for batch jobs) ──────────────────────────
  completions: 1                  # Run exactly 1 successful completion
  parallelism: 1                  # Run 1 Pod at a time

  template:
    metadata:
      labels:
        app: nginx-demo
        job-type: migration

    spec:
      # ── MUST be OnFailure or Never for Jobs ───────────────
      # Never = don't restart, just mark as failed (Job handles retries)
      restartPolicy: OnFailure

      # ── Run migrations BEFORE the main app starts ─────────
      initContainers:
        - name: wait-for-db
          image: busybox
          # Wait until MySQL is accepting connections
          command:
            - sh
            - -c
            - |
              echo "Waiting for MySQL to be ready..."
              until nc -z mysql 3306; do
                echo "MySQL not ready, sleeping 2s..."
                sleep 2
              done
              echo "MySQL is ready!"

      containers:
        - name: db-migration
          image: chandra219/welcome-login-app:66   # Same Flask image

          # ── Override the default command to run migrations ─
          command:
            - sh
            - -c
            - |
              echo "Starting database migration..."
              flask db upgrade         # Run Flask-Migrate upgrades
              echo "Seeding initial data..."
              python seed.py           # Seed initial data if needed
              echo "Migration completed successfully!"

          # ── Load config so Flask knows how to connect to DB ─
          envFrom:
            - configMapRef:
                name: flask-app-config
            - secretRef:
                name: flask-app-secret

          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"

---
# ── CronJob: Send weekly summary emails to all users ─────────
apiVersion: batch/v1
kind: CronJob
metadata:
  name: flask-weekly-report
  namespace: default

spec:
  schedule: "0 9 * * 1"          # Every Monday at 9:00 AM (cron syntax)
  #           │ │ │ │ └─ Day of week (1=Monday)
  #           │ │ │ └─── Month (any)
  #           │ │ └───── Day of month (any)
  #           │ └─────── Hour (9 AM)
  #           └───────── Minute (0)

  concurrencyPolicy: Forbid       # Don't run new job if previous is still running
  successfulJobsHistoryLimit: 3   # Keep last 3 successful job records
  failedJobsHistoryLimit: 1       # Keep last 1 failed job record

  jobTemplate:
    spec:
      backoffLimit: 2
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: email-sender
              image: chandra219/welcome-login-app:66
              command: ["python", "send_weekly_report.py"]
              envFrom:
                - configMapRef:
                    name: flask-app-config
                - secretRef:
                    name: flask-app-secret
```

### ✅ How to apply & verify

```bash
kubectl apply -f 08-job.yaml

# Watch the migration job run
kubectl get jobs
kubectl get pods -l job-type=migration

# See logs of the migration
kubectl logs -l job-type=migration --follow

# Check if job completed successfully
kubectl describe job flask-db-migration
# Look for: "Succeeded: 1"

# After success, pods stay (but are Completed, not Running)
kubectl get pods -l job-type=migration

# Clean up completed job
kubectl delete job flask-db-migration

# For CronJob: manually trigger
kubectl create job --from=cronjob/flask-weekly-report manual-run-001
```

---

## 9. Service

### 📖 What is it?
A **Service** provides a **stable network endpoint** to access your Pods. Since Pods are ephemeral (IPs change when they restart), Services give you a fixed IP and DNS name that always routes to the right Pods using **label selectors**.

### 🤔 The 4 Service Types for YOUR app

| Type | Access | Use Case |
|------|--------|----------|
| `ClusterIP` | Only inside cluster | Flask → MySQL communication |
| `NodePort` | NodeIP:Port from outside | Minikube/local development |
| `LoadBalancer` | External IP (cloud) | Production on AWS/GCP/Azure |
| `ExternalName` | CNAME to external DNS | Access external services like managed DBs |

### 📄 `09-service.yaml`

```yaml
# ── Service 1: NodePort for your Flask app (Development) ─────
# Access your app at: http://<minikube-ip>:30500
apiVersion: v1
kind: Service
metadata:
  name: nginx-demo                # Same name as your original service.yaml
  namespace: default
  labels:
    app: nginx-demo
  annotations:
    description: "NodePort service for Flask app — works on Minikube"

spec:
  type: NodePort

  # ── Route traffic to Pods with this label ─────────────────
  selector:
    app: nginx-demo               # Matches Pod labels in Deployment

  ports:
    - name: http
      protocol: TCP
      port: 5000                  # Service port (other services call this)
      targetPort: 5000            # Pod's container port (Flask listens here)
      nodePort: 30500             # External port on each node (30000–32767)
      # Access URL: http://$(minikube ip):30500

---
# ── Service 2: ClusterIP for MySQL (Internal only) ───────────
# Flask app connects to MySQL using: mysql-service:3306
# Nobody outside the cluster can reach this!
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  namespace: default

spec:
  type: ClusterIP                 # Default type — internal only

  selector:
    app: mysql                    # Routes to StatefulSet mysql pods

  ports:
    - name: mysql
      port: 3306
      targetPort: 3306

---
# ── Service 3: LoadBalancer (Production on Cloud) ────────────
# Gets an EXTERNAL IP from AWS ELB / GCP / Azure
# ⚠️ Costs money on cloud! Use only for production.
apiVersion: v1
kind: Service
metadata:
  name: nginx-demo-lb
  namespace: default
  annotations:
    # AWS-specific: create an internet-facing load balancer
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"

spec:
  type: LoadBalancer

  selector:
    app: nginx-demo

  ports:
    - name: http
      port: 80                    # External port (standard HTTP)
      targetPort: 5000            # Internal Flask port
    - name: https
      port: 443
      targetPort: 5000

---
# ── Service 4: ExternalName (Access external managed DB) ─────
# Map an in-cluster DNS name to an external AWS RDS hostname
# Pods connect to "rds-service" → gets routed to AWS RDS
apiVersion: v1
kind: Service
metadata:
  name: rds-service
  namespace: default

spec:
  type: ExternalName
  externalName: "mydb.xxxxxx.us-east-1.rds.amazonaws.com"
  # Pods can now connect to: rds-service.default.svc.cluster.local
  # No selector needed — it's just a DNS alias (CNAME)
```

### ✅ How to apply & verify

```bash
kubectl apply -f 09-service.yaml

# View all services
kubectl get services

# Get Minikube IP and access your app
minikube ip
# Then open: http://<minikube-ip>:30500

# Or use minikube service shortcut (opens browser automatically!)
minikube service nginx-demo

# Check endpoints (should show your Pod IPs)
kubectl get endpoints nginx-demo

# Test internal DNS from inside a pod
kubectl run test --image=busybox -it --rm -- wget -qO- http://nginx-demo:5000
```

---

## 10. Ingress

### 📖 What is it?
**Ingress** manages **HTTP/HTTPS traffic from outside the cluster** to Services inside. Instead of exposing each Service with its own LoadBalancer (expensive!), Ingress provides a **single entry point** with:
- **Path-based routing**: `/` → Flask app, `/api` → API service
- **Host-based routing**: `myapp.com` → Flask, `admin.myapp.com` → Admin panel
- **TLS termination**: Handle HTTPS at the Ingress level
- **Rate limiting, auth** (via annotations)

### 🤔 Why use it for YOUR app?
Instead of NodePort (only for dev) or paying for multiple LoadBalancers, use ONE Ingress to route all traffic to your Flask app and future services.

### 📄 `10-ingress.yaml`

```yaml
# ── PREREQUISITE: Install NGINX Ingress Controller first! ────
# Minikube:  minikube addons enable ingress
# General:   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.0/deploy/static/provider/cloud/deploy.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: flask-app-ingress
  namespace: default
  labels:
    app: nginx-demo

  # ── Annotations configure the Ingress Controller behavior ─
  annotations:
    # Specify which Ingress Controller to use
    kubernetes.io/ingress.class: "nginx"

    # Redirect HTTP to HTTPS
    nginx.ingress.kubernetes.io/ssl-redirect: "true"

    # Rate limiting: max 100 requests/minute per IP
    nginx.ingress.kubernetes.io/limit-rps: "100"

    # Request size limit (for file uploads in your app)
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"

    # Timeout settings
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"

spec:
  # ── TLS: HTTPS termination ────────────────────────────────
  # Create the secret first: kubectl create secret tls flask-tls --cert=cert.pem --key=key.pem
  tls:
    - hosts:
        - myapp.local               # Your domain (add to /etc/hosts for local dev)
        - www.myapp.local
      secretName: flask-tls-secret  # Kubernetes Secret with TLS cert+key

  # ── Routing Rules ─────────────────────────────────────────
  rules:
    # ── Rule 1: Main Flask App ───────────────────────────────
    - host: myapp.local            # Traffic for this host
      http:
        paths:
          # All traffic to root → Flask app
          - path: /
            pathType: Prefix       # Prefix = matches / and /anything/else
            backend:
              service:
                name: nginx-demo   # Your Service name
                port:
                  number: 5000

          # API traffic → separate API service (future microservice)
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service  # A future API microservice
                port:
                  number: 8080

    # ── Rule 2: Admin Panel on separate subdomain ────────────
    - host: admin.myapp.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: admin-service
                port:
                  number: 5001

    # ── Rule 3: No host = catch-all (useful as fallback) ─────
    - http:
        paths:
          - path: /health
            pathType: Exact        # Exact = only matches /health (not /health/foo)
            backend:
              service:
                name: nginx-demo
                port:
                  number: 5000
```

### ✅ How to apply & verify

```bash
# Enable ingress on Minikube first!
minikube addons enable ingress

kubectl apply -f 10-ingress.yaml

# Check Ingress status (wait for ADDRESS to appear)
kubectl get ingress
kubectl describe ingress flask-app-ingress

# Add to /etc/hosts so your browser resolves myapp.local
echo "$(minikube ip) myapp.local" | sudo tee -a /etc/hosts

# Now open in browser:
# http://myapp.local      → Your Flask app!
# http://myapp.local/api  → API service

# Test with curl
curl http://myapp.local/
curl -H "Host: myapp.local" http://$(minikube ip)/
```

---

## 🚀 Complete Deployment Order

Apply all files in this order (dependencies first):

```bash
# Step 1: Config and Secrets first (Pods need these)
kubectl apply -f 01-configmap.yaml
kubectl apply -f 02-secret.yaml

# Step 2: StatefulSet for DB (app needs DB to be up)
kubectl apply -f 06-statefulset.yaml
kubectl wait --for=condition=ready pod/mysql-0 --timeout=120s

# Step 3: Run DB migrations (before app starts)
kubectl apply -f 08-job.yaml
kubectl wait --for=condition=complete job/flask-db-migration --timeout=300s

# Step 4: Deploy the Flask app
kubectl apply -f 05-deployment.yaml
kubectl rollout status deployment/nginx-demo

# Step 5: Expose the app via Service
kubectl apply -f 09-service.yaml

# Step 6: Set up Ingress routing
kubectl apply -f 10-ingress.yaml

# Step 7: Set up log collection on all nodes
kubectl apply -f 07-daemonset.yaml

# ✅ Verify everything is running
kubectl get all
minikube service nginx-demo   # Open app in browser!
```

---

## 🔁 Quick Reference Cheat Sheet

```bash
# ── Deployment ────────────────────────────────────────────
kubectl rollout status deployment/nginx-demo
kubectl rollout undo deployment/nginx-demo
kubectl set image deployment/nginx-demo flask-app=chandra219/welcome-login-app:67
kubectl scale deployment nginx-demo --replicas=5

# ── Pods ──────────────────────────────────────────────────
kubectl get pods -o wide
kubectl describe pod <name>
kubectl logs <pod-name> --follow
kubectl exec -it <pod-name> -- /bin/sh

# ── Services ──────────────────────────────────────────────
kubectl get svc
kubectl get endpoints nginx-demo
minikube service nginx-demo

# ── ConfigMap & Secrets ───────────────────────────────────
kubectl get configmap flask-app-config -o yaml
kubectl get secret flask-app-secret -o yaml

# ── Debugging ─────────────────────────────────────────────
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl top pods
kubectl top nodes

# ── Cleanup ───────────────────────────────────────────────
kubectl delete -f 05-deployment.yaml
kubectl delete all -l app=nginx-demo
```

---

## 🧠 Summary — When to Use What?

| Resource | When to use | Your App Example |
|---|---|---|
| **ConfigMap** | Non-sensitive app config | `APP_ENV`, `DB_HOST`, `LOG_LEVEL` |
| **Secret** | Passwords, keys, tokens | `DB_PASSWORD`, `FLASK_SECRET_KEY` |
| **Pod** | Learning / debugging only | Raw Flask container instance |
| **ReplicaSet** | Understanding internals | Managed automatically by Deployment |
| **Deployment** | Stateless apps, rolling updates | Your Flask welcome-login app |
| **StatefulSet** | Databases, message queues | MySQL for storing login data |
| **DaemonSet** | Per-node agents | Fluentd log collector on every node |
| **Job** | One-time tasks, migrations | `flask db upgrade` before deploy |
| **CronJob** | Scheduled recurring tasks | Weekly email reports |
| **Service** | Stable network endpoint | Expose Flask on port 30500 |
| **Ingress** | HTTP routing, TLS, domains | `myapp.local` → Flask app |

---

*All YAML files are built around `chandra219/welcome-login-app:66` running Flask on port 5000.*
*Apply them in order for a fully functional, production-grade Kubernetes setup.*
