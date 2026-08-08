# Kubernetes `kind` and `apiVersion` — Full Clarity

## 1. What are `apiVersion` and `kind`?

Every Kubernetes YAML object normally starts with:

```yaml
apiVersion: apps/v1
kind: Deployment
```

Think of it as:

- **`apiVersion`** = Which Kubernetes API group and version should handle this object?
- **`kind`** = What type of Kubernetes object is this?

For example:

```yaml
apiVersion: apps/v1
kind: Deployment
```

means:

> This is a `Deployment` object, and Kubernetes should process it using the `apps/v1` API.

---

# 2. The easiest mental model

Think of Kubernetes as having different API departments:

```text
Kubernetes API
│
├── Core API
│   └── v1
│       ├── Pod
│       ├── Service
│       ├── ConfigMap
│       └── Secret
│
├── Apps API
│   └── apps/v1
│       ├── Deployment
│       ├── ReplicaSet
│       ├── StatefulSet
│       └── DaemonSet
│
├── Networking API
│   └── networking.k8s.io/v1
│       └── Ingress
│
├── Batch API
│   └── batch/v1
│       ├── Job
│       └── CronJob
│
└── API Extensions
    └── apiextensions.k8s.io/v1
        └── CustomResourceDefinition
```

The important idea:

> **`apiVersion` tells Kubernetes which API group/version owns the resource.**

---

# 3. What does `v1` mean?

When you see:

```yaml
apiVersion: v1
```

this means:

```text
API Group = Core
Version   = v1
```

The Core API group is special because Kubernetes does not write its group name.

So:

```yaml
apiVersion: v1
```

means:

```text
Core API group + v1
```

Whereas:

```yaml
apiVersion: apps/v1
```

means:

```text
API Group = apps
Version   = v1
```

And:

```yaml
apiVersion: batch/v1
```

means:

```text
API Group = batch
Version   = v1
```

---

# 4. Pod → `v1`

Example:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: my-pod

spec:
  containers:
    - name: nginx
      image: nginx
```

Here:

```text
apiVersion: v1
kind: Pod
```

means:

> This is a Pod, and Pod belongs to the Kubernetes Core API.

---

# 5. Service → `v1`

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: my-service

spec:
  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

Service is also part of the Core API.

Therefore:

```yaml
apiVersion: v1
```

---

# 6. ConfigMap → `v1`

Example:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  ENV: production
```

Again:

```text
Core API
   ↓
  v1
   ↓
ConfigMap
```

---

# 7. Secret → `v1`

Example:

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: db-secret

type: Opaque

data:
  username: YWRtaW4=
```

Secret is also a Core API resource.

Therefore:

```yaml
apiVersion: v1
```

---

# 8. Why Deployment is `apps/v1`

Deployment is **not** part of the Core API.

It belongs to the:

```text
apps API group
```

Therefore:

```yaml
apiVersion: apps/v1
kind: Deployment
```

Example:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx
```

The important relationship is:

```text
apps/v1
  │
  ├── Deployment
  ├── ReplicaSet
  ├── StatefulSet
  └── DaemonSet
```

---

# 9. Why are Deployment, ReplicaSet, StatefulSet and DaemonSet together?

They all belong to the:

```text
apps
```

API group.

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
```

### ReplicaSet

```yaml
apiVersion: apps/v1
kind: ReplicaSet
```

### StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
```

### DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
```

A useful memory rule:

> **Most major application workload controllers → `apps/v1`**

Note: Jobs and CronJobs are also workloads, but Kubernetes places them under the `batch` API group.

---

# 10. Ingress → `networking.k8s.io/v1`

Ingress belongs to the Networking API group.

Therefore:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
```

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: my-ingress

spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

Break it down:

```text
networking.k8s.io
        │
        └── API Group

v1
│
└── API Version
```

So do not confuse:

```yaml
apiVersion: v1
```

with:

```yaml
apiVersion: networking.k8s.io/v1
```

They are different API groups.

---

# 11. Job → `batch/v1`

A Job runs a task to completion.

Example:

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: database-migration

spec:
  template:
    spec:
      containers:
        - name: migration
          image: my-migration:1.0

      restartPolicy: Never
```

The API group is:

```text
batch
```

The version is:

```text
v1
```

Therefore:

```text
batch/v1
```

---

# 12. CronJob → `batch/v1`

CronJob is also under the Batch API.

Example:

```yaml
apiVersion: batch/v1
kind: CronJob

metadata:
  name: backup

spec:
  schedule: "0 2 * * *"

  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: backup
              image: backup:1.0

          restartPolicy: Never
```

So:

```text
batch/v1
   │
   ├── Job
   └── CronJob
```

---

# 13. CustomResourceDefinition → `apiextensions.k8s.io/v1`

This one is different.

A `CustomResourceDefinition` (CRD) is used to extend Kubernetes with your own resource type.

For example, Kubernetes does not natively have:

```yaml
kind: Database
```

You can define a new resource type called `Database` using a CRD.

First, create the CRD:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition

metadata:
  name: databases.example.com

spec:
  group: example.com

  names:
    kind: Database
    plural: databases

  scope: Namespaced

  versions:
    - name: v1
      served: true
      storage: true
```

After the CRD exists, you can create a custom resource:

```yaml
apiVersion: example.com/v1
kind: Database
```

## Important distinction

### The CRD itself

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
```

### A custom resource created using that CRD

```yaml
apiVersion: example.com/v1
kind: Database
```

These are **not the same thing**.

The CRD defines what the custom resource looks like.

---

# 14. The key concept

Suppose you have:

```yaml
apiVersion: apps/v1
kind: Deployment
```

Break it apart:

```text
apiVersion: apps/v1
            │    │
            │    └── Version
            │
            └────── API Group

kind: Deployment
      │
      └── Resource type
```

So:

```text
apps/v1 + Deployment
```

identifies:

> The Deployment resource in version 1 of the `apps` API group.

---

# 15. Think of `apiVersion` like an address

A useful analogy:

```text
apiVersion = API Department + Version
kind       = Specific Resource
```

For example:

```text
apps/v1
  ↓
Apps API Department, version 1

Deployment
  ↓
Deployment resource
```

Together:

```text
apps/v1 + Deployment
```

means:

> Find the Deployment resource in the Apps API version 1.

---

# 16. Complete reference table

| Kind | apiVersion | API Group | What it represents |
|---|---|---|---|
| Pod | `v1` | Core | Runs containers |
| Service | `v1` | Core | Provides stable networking to Pods |
| ConfigMap | `v1` | Core | Non-sensitive configuration |
| Secret | `v1` | Core | Sensitive configuration/data |
| Deployment | `apps/v1` | apps | Manages stateless application Pods |
| ReplicaSet | `apps/v1` | apps | Maintains desired Pod count |
| StatefulSet | `apps/v1` | apps | Manages stateful workloads |
| DaemonSet | `apps/v1` | apps | Runs Pods on nodes according to scheduling rules |
| Ingress | `networking.k8s.io/v1` | networking.k8s.io | HTTP/HTTPS routing into the cluster |
| Job | `batch/v1` | batch | Runs a task to completion |
| CronJob | `batch/v1` | batch | Creates Jobs on a schedule |
| CustomResourceDefinition | `apiextensions.k8s.io/v1` | apiextensions.k8s.io | Defines new Kubernetes resource types |

---

# 17. How Kubernetes uses `apiVersion` and `kind`

When you run:

```bash
kubectl apply -f deployment.yaml
```

and the YAML contains:

```yaml
apiVersion: apps/v1
kind: Deployment
```

conceptually the flow is:

```text
kubectl
   |
   | apiVersion: apps/v1
   | kind: Deployment
   ↓
kube-apiserver
   |
   ↓
Apps API
   |
   ↓
Deployment resource
```

The API server uses the API group/version and resource kind to determine how the object should be handled.

---

# 18. Important interview question

## Q: What is the difference between `v1` and `apps/v1`?

A weak answer:

> They are different versions.

That is incomplete.

A technically correct answer:

> **`v1` is version 1 of Kubernetes' Core API group, whereas `apps/v1` is version 1 of the `apps` API group. The API group determines which Kubernetes resources and API endpoints the resource belongs to.**

---

# 19. Don't memorize random pairs

Avoid memorizing only:

```text
Pod → v1
Deployment → apps/v1
Ingress → networking.k8s.io/v1
Job → batch/v1
```

Instead, memorize the API groups:

```text
Core
  └── v1

Apps
  └── apps/v1

Networking
  └── networking.k8s.io/v1

Batch
  └── batch/v1

API Extensions
  └── apiextensions.k8s.io/v1
```

Then associate resources with each group.

---

# 20. Final mental picture

```text
                    KUBERNETES API
                          |
        +-----------------+------------------+
        |                 |                  |
      Core              apps               batch
       |                  |                  |
      v1                  v1                 v1
       |                  |                  |
   +---+----+        +----+---------+     +--+------+
   |   |    |        |    |    |    |     |         |
  Pod Svc Secret Deployment RS Stateful DaemonSet   Job
                                                        |
                                                     CronJob


             networking.k8s.io
                    |
                    v1
                    |
                 Ingress


             apiextensions.k8s.io
                    |
                    v1
                    |
             CustomResourceDefinition
```

---

# 21. One-line rule to remember

> **`apiVersion` = API group + API version; `kind` = the resource type within that API.**

And the special case:

> **`v1` alone means the Core API group at version 1.**

Once you understand this, you don't need to blindly memorize the table.
