# Kubernetes API Server — 360° Deep Dive
### Complete Architecture, Workflow, Storage & Internals

---

## Table of Contents

1. [What is the kube-apiserver?](#1-what-is-the-kube-apiserver)
2. [Architecture Diagram — Full 360° View](#2-architecture-diagram--full-360-view)
3. [End-to-End Request Flow](#3-end-to-end-request-flow)
4. [What Does the API Server Actually Do?](#4-what-does-the-api-server-actually-do)
5. [What Does It Store?](#5-what-does-it-store)
6. [Internal Components Breakdown](#6-internal-components-breakdown)
7. [Authentication — Who Are You?](#7-authentication--who-are-you)
8. [Authorization — Can You Do This?](#8-authorization--can-you-do-this)
9. [Admission Controllers — Should This Happen?](#9-admission-controllers--should-this-happen)
10. [The Watch Mechanism — How Controllers Stay Notified](#10-the-watch-mechanism--how-controllers-stay-notified)
11. [API Groups & Versioning](#11-api-groups--versioning)
12. [etcd: What Gets Stored & How](#12-etcd-what-gets-stored--how)
13. [High Availability Setup](#13-high-availability-setup)
14. [Security Surface](#14-security-surface)
15. [Key Config Flags](#15-key-config-flags)
16. [Q&A — 360° Questions Answered](#16-qa--360-questions-answered)

---

## 1. What is the kube-apiserver?

The `kube-apiserver` is the **central nervous system** of a Kubernetes cluster. It is the only component that talks directly to `etcd`. Every other component — the scheduler, controller-manager, kubelet, kube-proxy, and any human using `kubectl` — communicates with the cluster exclusively through the API server.

It is a stateless HTTP/2 REST server that:
- Exposes the Kubernetes API (REST + Watch + gRPC)
- Validates and processes every object mutation
- Is the sole writer and reader of `etcd`
- Broadcasts state changes to all watching components in real time

> **One rule**: Nothing in Kubernetes changes state without going through the API server. It is the single source of truth gateway.

---

## 2. Architecture Diagram — Full 360° View

```
╔══════════════════════════════════════════════════════════════════════════════════════════════╗
║                          KUBE-APISERVER — 360° ARCHITECTURE                                 ║
╚══════════════════════════════════════════════════════════════════════════════════════════════╝

  ┌──────────────────────────────────────────────────────────────────────────────────────────┐
  │                              CLIENTS (Who talks to the API server)                       │
  │                                                                                          │
  │   kubectl        CI/CD Pipelines      Operators/CRDs       Helm / ArgoCD / Flux         │
  │      │                │                    │                      │                      │
  │      └────────────────┴────────────────────┴──────────────────────┘                      │
  │                                       │                                                  │
  │                          HTTPS :6443 (TLS + mTLS)                                        │
  └───────────────────────────────────────┼──────────────────────────────────────────────────┘
                                          │
  ┌───────────────────────────────────────▼──────────────────────────────────────────────────┐
  │                              KUBE-APISERVER PROCESS                                      │
  │                                                                                          │
  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐ │
  │  │  LAYER 1 — TRANSPORT & CONNECTION                                                   │ │
  │  │  ┌─────────────────┐  ┌──────────────────┐  ┌────────────────────────────────────┐ │ │
  │  │  │  TLS Termination│  │  HTTP/2 Mux       │  │  Rate Limiter                      │ │ │
  │  │  │  (cert + CA)    │  │  (path routing)   │  │  (--max-requests-inflight)         │ │ │
  │  │  └─────────────────┘  └──────────────────┘  └────────────────────────────────────┘ │ │
  │  └─────────────────────────────────────────────────────────────────────────────────────┘ │
  │                                          │                                               │
  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐ │
  │  │  LAYER 2 — AUTHENTICATION (AuthN)                                                   │ │
  │  │                                                                                     │ │
  │  │  ┌──────────────┐ ┌───────────────┐ ┌─────────────┐ ┌──────────────────────────┐  │ │
  │  │  │ X.509 Client │ │ Bearer Token  │ │ OIDC Token  │ │ Webhook TokenReview      │  │ │
  │  │  │ Certificates │ │ (ServiceAcct) │ │ (OIDC IdP)  │ │ (external auth provider) │  │ │
  │  │  └──────────────┘ └───────────────┘ └─────────────┘ └──────────────────────────┘  │ │
  │  │                                                                                     │ │
  │  │  Result: UserInfo { Username, UID, Groups, Extra }                                  │ │
  │  └─────────────────────────────────────────────────────────────────────────────────────┘ │
  │                                          │                                               │
  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐ │
  │  │  LAYER 3 — AUTHORIZATION (AuthZ)                                                    │ │
  │  │                                                                                     │ │
  │  │  ┌──────────────────┐  ┌──────────────┐  ┌─────────────────┐  ┌──────────────┐   │ │
  │  │  │ RBAC             │  │ ABAC         │  │ Node Authorizer │  │ Webhook AuthZ│   │ │
  │  │  │ (Roles +         │  │ (policy file)│  │ (kubelet-only)  │  │ (OPA etc.)  │   │ │
  │  │  │  Bindings)       │  │              │  │                 │  │             │   │ │
  │  │  └──────────────────┘  └──────────────┘  └─────────────────┘  └──────────────┘   │ │
  │  │                                                                                     │ │
  │  │  Decision: ALLOW / DENY for (user, verb, resource, namespace)                       │ │
  │  └─────────────────────────────────────────────────────────────────────────────────────┘ │
  │                                          │                                               │
  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐ │
  │  │  LAYER 4 — ADMISSION CONTROL                                                        │ │
  │  │                                                                                     │ │
  │  │  ┌────────────────────────────┐          ┌───────────────────────────────────────┐ │ │
  │  │  │ MUTATING ADMISSION WEBHOOKS│          │ VALIDATING ADMISSION WEBHOOKS         │ │ │
  │  │  │                            │          │                                       │ │ │
  │  │  │ • Inject sidecars          │          │ • OPA / Gatekeeper policies           │ │ │
  │  │  │ • Set default limits       │   ──►    │ • PodSecurity admission               │ │ │
  │  │  │ • Add labels/annotations   │          │ • ResourceQuota enforcement           │ │ │
  │  │  │ • LimitRanger defaults     │          │ • ImagePolicy validation              │ │ │
  │  │  └────────────────────────────┘          └───────────────────────────────────────┘ │ │
  │  │                                                                                     │ │
  │  │  Built-in Controllers: NamespaceLifecycle, ServiceAccount, PodSecurity, etc.        │ │
  │  └─────────────────────────────────────────────────────────────────────────────────────┘ │
  │                                          │                                               │
  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐ │
  │  │  LAYER 5 — VALIDATION & SCHEMA CHECK                                               │ │
  │  │                                                                                     │ │
  │  │  OpenAPI v3 schema validation • Field type checking • Required fields              │ │
  │  │  Resource version conflict detection (optimistic locking via resourceVersion)       │ │
  │  └─────────────────────────────────────────────────────────────────────────────────────┘ │
  │                                          │                                               │
  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐ │
  │  │  LAYER 6 — SERIALIZATION & STORAGE                                                  │ │
  │  │                                                                                     │ │
  │  │  JSON (external) ◄──► Protobuf (internal/etcd)                                     │ │
  │  │  Object stored under key: /registry/<group>/<resource>/<namespace>/<name>           │ │
  │  └─────────────────────────────────────────────────────────────────────────────────────┘ │
  └───────────────────────────────────────┬──────────────────────────────────────────────────┘
                                          │
              ┌───────────────────────────┼───────────────────────────┐
              ▼                           ▼                           ▼
  ┌─────────────────────┐    ┌────────────────────────┐   ┌────────────────────────────┐
  │        etcd          │    │   Watch Event Stream   │   │  Aggregation Layer         │
  │  (Key-Value Store)   │    │                        │   │  (Extension API Servers)   │
  │                      │    │ kube-scheduler         │   │                            │
  │ /registry/pods/...   │    │ kube-ctrl-manager      │   │ metrics-server             │
  │ /registry/secrets/.. │    │ kubelet (each node)    │   │ custom CRD servers         │
  │ /registry/services/. │    │ kube-proxy             │   │ service catalog            │
  │                      │    │ Ingress controllers    │   └────────────────────────────┘
  │ Protobuf encoded     │    │ Custom operators       │
  │ Raft consensus       │    │ kubectl watches        │
  └─────────────────────┘    └────────────────────────┘
```

---

## 3. End-to-End Request Flow

Every request — whether a `kubectl get`, `kubectl apply`, or a controller update — passes through the exact same pipeline inside the API server.

```
  CLIENT REQUEST
  (kubectl / controller / kubelet)
         │
         │ HTTPS POST/GET/PATCH/DELETE to :6443
         ▼
  ┌─────────────────────────────────────────────────────────┐
  │ STEP 1: TLS Handshake                                   │
  │  • Client presents cert or token                        │
  │  • Server presents its certificate                      │
  │  • Mutual TLS established                               │
  └─────────────────────────┬───────────────────────────────┘
                            │
                            ▼
  ┌─────────────────────────────────────────────────────────┐
  │ STEP 2: Authentication (AuthN)                          │
  │  • Which authenticator handles this request?            │
  │    - X.509 cert? → extract CN=username, O=groups        │
  │    - Bearer token? → ServiceAccount JWT validation      │
  │    - OIDC? → decode + verify JWT from IdP               │
  │    - Webhook? → external TokenReview call               │
  │  • Returns: UserInfo{username, groups}                  │
  │  • Fail → 401 Unauthorized                              │
  └─────────────────────────┬───────────────────────────────┘
                            │
                            ▼
  ┌─────────────────────────────────────────────────────────┐
  │ STEP 3: Authorization (AuthZ)                           │
  │  • Builds SubjectAccessReview:                          │
  │    { user, verb, resource, namespace, name }            │
  │  • Runs authorizer chain (RBAC → Node → Webhook)        │
  │  • First ALLOW wins, default is DENY                    │
  │  • Fail → 403 Forbidden                                 │
  └─────────────────────────┬───────────────────────────────┘
                            │
                            ▼
  ┌─────────────────────────────────────────────────────────┐
  │ STEP 4: Mutating Admission                              │
  │  • Runs each enabled MutatingWebhookConfiguration       │
  │  • Each webhook receives the object, may return patches │
  │  • Patches are applied to the object (JSON Patch)       │
  │  • Example: Istio injects envoy sidecar here            │
  └─────────────────────────┬───────────────────────────────┘
                            │
                            ▼
  ┌─────────────────────────────────────────────────────────┐
  │ STEP 5: Object Schema Validation                        │
  │  • Validates against OpenAPI v3 schema                  │
  │  • Required fields present?                             │
  │  • Field types correct?                                 │
  │  • Enum values valid?                                   │
  │  • Fail → 422 Unprocessable Entity                      │
  └─────────────────────────┬───────────────────────────────┘
                            │
                            ▼
  ┌─────────────────────────────────────────────────────────┐
  │ STEP 6: Validating Admission                            │
  │  • Runs each ValidatingWebhookConfiguration             │
  │  • Cannot modify object — can only ALLOW or DENY        │
  │  • OPA Gatekeeper, Kyverno run here                     │
  │  • Example: deny images not from approved registry      │
  │  • Fail → 422 Unprocessable Entity                      │
  └─────────────────────────┬───────────────────────────────┘
                            │
                            ▼
  ┌─────────────────────────────────────────────────────────┐
  │ STEP 7: Persist to etcd                                 │
  │  • Serialize object to Protobuf                         │
  │  • Write to etcd under correct key                      │
  │  • Set resourceVersion (monotonic counter from etcd)    │
  │  • Set creationTimestamp, uid (UUID)                    │
  │  • etcd confirms via Raft quorum                        │
  └─────────────────────────┬───────────────────────────────┘
                            │
                            ▼
  ┌─────────────────────────────────────────────────────────┐
  │ STEP 8: Return Response + Broadcast Watch Events        │
  │  • Returns 200 OK / 201 Created / 204 No Content        │
  │  • Watch event (ADDED/MODIFIED/DELETED) sent to all     │
  │    components watching that resource type               │
  │  • Scheduler, controllers, kubelets all notified        │
  └─────────────────────────────────────────────────────────┘
```

---

## 4. What Does the API Server Actually Do?

### Q: What is its primary function?

The API server does **six core jobs**:

| Job | Description |
|---|---|
| **Gateway** | Single entry point for all cluster interactions — no component bypasses it |
| **Validator** | Enforces schema correctness, field constraints, and admission policies |
| **Authenticator** | Verifies the identity of every caller |
| **Authorizer** | Enforces RBAC/ABAC permissions on every operation |
| **Serializer** | Converts between external JSON ↔ internal types ↔ etcd protobuf |
| **Event broadcaster** | Streams watch events to all registered listeners in real time |

### Q: What HTTP verbs does it support?

| HTTP Verb | Kubernetes Action | Example |
|---|---|---|
| `GET` | Read single object or list | `kubectl get pod nginx` |
| `POST` | Create new object | `kubectl apply -f pod.yaml` (new) |
| `PUT` | Replace full object | `kubectl replace` |
| `PATCH` | Partial update | `kubectl patch` / controller reconcile |
| `DELETE` | Delete object | `kubectl delete pod nginx` |
| `WATCH` (GET + `?watch=true`) | Stream change events | `kubectl get pods -w` |

### Q: What ports does it listen on?

| Port | Purpose |
|---|---|
| `6443` (default) | Secure HTTPS port — all external traffic |
| `8080` (deprecated) | Insecure localhost-only port — disabled in modern clusters |

### Q: How does it handle concurrent updates?

Using **optimistic concurrency** with `resourceVersion`. Every etcd object has a `resourceVersion` (a string encoding the etcd revision). When you PATCH or PUT an object, you must send the current `resourceVersion`. If another writer updated the object in between, your `resourceVersion` is stale → the API server returns `409 Conflict` → you must re-read and retry.

---

## 5. What Does It Store?

The API server itself is **stateless** — it stores nothing on disk. All state lives in `etcd`.

### etcd Key Structure

```
/registry/
│
├── core/
│   ├── pods/
│   │   ├── default/nginx-pod-abc12
│   │   └── dev/signin-portal-xyz99
│   │
│   ├── services/
│   │   ├── default/kubernetes          ← The cluster-internal API service itself
│   │   └── dev/signin-portal-svc
│   │
│   ├── endpoints/
│   │   └── dev/signin-portal-svc       ← Live pod IPs backing a Service
│   │
│   ├── namespaces/
│   │   ├── default
│   │   ├── dev
│   │   └── kube-system
│   │
│   ├── secrets/
│   │   └── dev/db-secret               ← Encrypted at rest (AES-CBC / AES-GCM)
│   │
│   ├── configmaps/
│   │   └── dev/app-config
│   │
│   ├── serviceaccounts/
│   │   └── dev/signin-sa
│   │
│   └── persistentvolumes/              ← Cluster-scoped (no namespace)
│
├── apps/
│   ├── deployments/
│   │   └── dev/signin-portal
│   │
│   ├── replicasets/
│   │   └── dev/signin-portal-7d9f8b
│   │
│   ├── statefulsets/
│   └── daemonsets/
│
├── batch/
│   ├── jobs/
│   └── cronjobs/
│
├── networking.k8s.io/
│   ├── ingresses/
│   │   └── dev/signin-portal-ingress
│   └── networkpolicies/
│
├── rbac.authorization.k8s.io/
│   ├── clusterroles/
│   ├── clusterrolebindings/
│   ├── roles/
│   └── rolebindings/
│
├── apiregistration.k8s.io/             ← Aggregated API server registrations
│
└── events/
    └── dev/signin-portal.pod.scheduled ← Short TTL, auto-expired
```

### What Each Object Type Stores

| Object | Key Fields Stored |
|---|---|
| **Pod** | Spec (containers, volumes, probes), Status (phase, conditions, podIP, containerStatuses) |
| **Deployment** | Spec (replicas, selector, template), Status (readyReplicas, updatedReplicas), annotations (rollout revision) |
| **Secret** | `data` map (base64 + optionally encrypted), `type` (Opaque / kubernetes.io/tls / etc.) |
| **ConfigMap** | `data` map of key-value strings, `binaryData` for binary |
| **Service** | `spec.clusterIP`, `spec.ports`, `spec.selector`, `spec.type` |
| **Endpoints** | `subsets[].addresses[].ip` — live list of pod IPs ready to receive traffic |
| **Node** | Capacity (CPU/RAM), allocatable, conditions (Ready/MemPressure), addresses |
| **Event** | InvolvedObject, reason, message, count, firstTimestamp — TTL 1 hour by default |
| **RBAC objects** | rules (verbs + resources), subjects (users/groups/serviceaccounts) |

### What Gets Encrypted at Rest?

Kubernetes supports **encryption at rest** for selected resource types. By default, nothing is encrypted. With an `EncryptionConfiguration`, these types are encrypted before writing to etcd:

- `Secrets` (most critical — contains passwords, tokens, keys)
- `ConfigMaps` (if they contain sensitive config)
- Custom Resources (CRDs)

Encryption providers: `aescbc`, `aesgcm`, `secretbox`, `kms` (external KMS like AWS KMS, HashiCorp Vault).

---

## 6. Internal Components Breakdown

```
  INSIDE kube-apiserver BINARY
  ┌─────────────────────────────────────────────────────────────┐
  │                                                             │
  │  ┌──────────────────────────────────────────────────────┐   │
  │  │  API Handler Registry (go-restful)                   │   │
  │  │  Maps URL paths → handler functions                  │   │
  │  │  /api/v1/pods → PodHandler                           │   │
  │  │  /apis/apps/v1/deployments → DeploymentHandler       │   │
  │  └──────────────────────────────────────────────────────┘   │
  │                         │                                   │
  │  ┌──────────────────────▼───────────────────────────────┐   │
  │  │  API Machinery (k8s.io/apiserver)                    │   │
  │  │                                                      │   │
  │  │  • Scheme: maps GVK (group/version/kind) to Go types │   │
  │  │  • Codec: JSON ↔ YAML ↔ Protobuf conversion          │   │
  │  │  • Defaulting: fills in unset fields with defaults   │   │
  │  │  • Conversion: v1beta1 → v1 object conversion        │   │
  │  └──────────────────────────────────────────────────────┘   │
  │                         │                                   │
  │  ┌──────────────────────▼───────────────────────────────┐   │
  │  │  Storage Layer (etcd3 driver)                        │   │
  │  │                                                      │   │
  │  │  • Prefix: /registry                                 │   │
  │  │  • Compaction: runs periodically to reclaim space    │   │
  │  │  • WatchCache: in-memory cache of recent objects     │   │
  │  │    for efficient List and Watch without hitting etcd │   │
  │  └──────────────────────────────────────────────────────┘   │
  │                         │                                   │
  │  ┌──────────────────────▼───────────────────────────────┐   │
  │  │  Watch Cache & Caching Layer                         │   │
  │  │                                                      │   │
  │  │  • Maintains in-memory snapshot of all objects       │   │
  │  │  • Serves LIST requests from cache (not etcd)        │   │
  │  │  • Fans out single etcd watch to N client watches    │   │
  │  │    (prevents thundering herd on etcd)                │   │
  │  └──────────────────────────────────────────────────────┘   │
  │                                                             │
  └─────────────────────────────────────────────────────────────┘
```

---

## 7. Authentication — Who Are You?

### Q: How does the API server know who is calling?

It runs a **chain of authenticators** — the first one that succeeds wins. If all fail → 401.

```
  Incoming Request
        │
        ▼
  ┌─────────────────────────────────────────────────────────────────┐
  │              AUTHENTICATOR CHAIN                                │
  │                                                                 │
  │  ① X.509 Client Certificates                                   │
  │    ─────────────────────────────────────────────────────────   │
  │    • Client presents a TLS cert signed by the cluster CA       │
  │    • CN field → username                                        │
  │    • O fields  → groups                                         │
  │    • Example: CN=alice, O=dev-team                              │
  │                                                                 │
  │  ② ServiceAccount Tokens (JWT)                                  │
  │    ─────────────────────────────────────────────────────────   │
  │    • Pods get auto-mounted token at                             │
  │      /var/run/secrets/kubernetes.io/serviceaccount/token        │
  │    • API server validates signature with its own signing key    │
  │    • Token contains: namespace, serviceaccount name, uid        │
  │                                                                 │
  │  ③ OIDC Tokens (OpenID Connect)                                 │
  │    ─────────────────────────────────────────────────────────   │
  │    • JWT issued by external IdP (Okta, Dex, Auth0, Keycloak)   │
  │    • API server validates signature against IdP's JWKS endpoint │
  │    • Claims: sub → username, groups → groups claim              │
  │                                                                 │
  │  ④ Webhook Token Authentication                                 │
  │    ─────────────────────────────────────────────────────────   │
  │    • API server forwards token to external webhook              │
  │    • Webhook returns TokenReview.Status.Authenticated=true/false│
  │                                                                 │
  │  ⑤ Bootstrap Tokens                                             │
  │    ─────────────────────────────────────────────────────────   │
  │    • Used during node bootstrap (kubeadm join)                  │
  │    • Stored as Secret in kube-system namespace                  │
  │                                                                 │
  └─────────────────────────────────────────────────────────────────┘
        │
        ▼
  UserInfo { Username, UID, Groups, Extra }
  Passed to the Authorization layer
```

---

## 8. Authorization — Can You Do This?

### Q: How does RBAC work inside the API server?

```
  UserInfo from AuthN
        │
        ▼
  ┌───────────────────────────────────────────────────────────────┐
  │               AUTHORIZATION CHAIN                             │
  │                                                               │
  │  ① Node Authorizer                                            │
  │     • Only for system:node:<nodename> principals              │
  │     • Allows kubelets to read Pods, Secrets, ConfigMaps       │
  │       bound to their own node only                            │
  │                                                               │
  │  ② RBAC Authorizer                                            │
  │     Input: SubjectAccessReview {                              │
  │       user: "alice",                                          │
  │       verb: "create",                                         │
  │       resource: "deployments",                                │
  │       namespace: "dev"                                        │
  │     }                                                         │
  │                                                               │
  │     Logic:                                                    │
  │     1. Find all RoleBindings/ClusterRoleBindings for "alice"  │
  │     2. Collect all referenced Roles/ClusterRoles              │
  │     3. Check if any rule matches:                             │
  │        { verbs: ["create"], resources: ["deployments"] }      │
  │     4. If match found → ALLOW                                 │
  │     5. No match → move to next authorizer                     │
  │                                                               │
  │  ③ Webhook Authorizer                                         │
  │     • Sends SubjectAccessReview to external endpoint           │
  │     • Used for OPA, custom policy engines                     │
  │                                                               │
  │  ④ ABAC Authorizer (legacy, rarely used)                      │
  │     • Static policy file on disk                              │
  │                                                               │
  └───────────────────────────────────────────────────────────────┘
        │
        ▼
  ALLOW → proceed to Admission
  DENY  → 403 Forbidden returned to client
```

### RBAC Object Relationships

```
  ClusterRole "deployment-admin"          Role "pod-reader" (namespace: dev)
  ┌────────────────────────────────┐      ┌────────────────────────────────┐
  │ rules:                         │      │ rules:                         │
  │  - apiGroups: ["apps"]         │      │  - apiGroups: [""]             │
  │    resources: ["deployments"]  │      │    resources: ["pods"]         │
  │    verbs: ["*"]                │      │    verbs: ["get","list"]       │
  └────────────────────────────────┘      └────────────────────────────────┘
            │                                        │
            │ ClusterRoleBinding                     │ RoleBinding (ns: dev)
            ▼                                        ▼
  subject: User "alice"                   subject: ServiceAccount "ci-bot"
  → alice can manage deployments          → ci-bot can list pods in dev
    cluster-wide                            namespace only
```

---

## 9. Admission Controllers — Should This Happen?

### Q: What are admission controllers and why do they matter?

Admission controllers are the last line of defense before an object is written to etcd. They are **plugins compiled into the API server** or called as **external webhooks**.

```
  MUTATING PHASE (runs first)                 VALIDATING PHASE (runs after)
  ┌──────────────────────────────┐            ┌──────────────────────────────┐
  │ Built-in Mutating:           │            │ Built-in Validating:         │
  │                              │            │                              │
  │ • DefaultStorageClass        │            │ • LimitRanger                │
  │   Adds default StorageClass  │            │   Ensures limits are set     │
  │   to PVC if not specified    │            │                              │
  │                              │            │ • ResourceQuota              │
  │ • DefaultTolerationSeconds   │            │   Namespace cannot exceed    │
  │   Adds tolerations for       │            │   CPU/memory/pod quota       │
  │   node unreachable/notready  │            │                              │
  │                              │            │ • NamespaceLifecycle         │
  │ • ServiceAccount             │  ──────►   │   Reject creates in          │
  │   Auto-mounts token to pods  │            │   terminating namespace      │
  │                              │            │                              │
  │ • MutatingWebhookConfig      │            │ • PodSecurity                │
  │   (external webhooks)        │            │   Enforces security profile  │
  │   e.g. Istio sidecar inject  │            │   (baseline/restricted)      │
  │   e.g. Vault agent inject    │            │                              │
  │                              │            │ • ValidatingWebhookConfig    │
  │                              │            │   OPA Gatekeeper policies    │
  │                              │            │   Kyverno policies           │
  └──────────────────────────────┘            └──────────────────────────────┘
```

### How Webhook Admission Works

```
  API Server                    External Webhook Server
       │                              (OPA / custom)
       │  AdmissionReview {           │
       │    request: {                │
       │      uid: "abc-123",         │
       │      kind: Deployment,       │
       │      operation: CREATE,      │
       │      object: <manifest>,     │
       │      userInfo: {...}         │
       │    }                         │
       │  }                           │
       │ ─────────────── HTTPS ──────►│
       │                              │  Run policy / logic
       │◄────────────────────────────│
       │  AdmissionReview {           │
       │    response: {               │
       │      uid: "abc-123",         │
       │      allowed: true/false,    │
       │      status: { message: "" } │
       │      patchType: "JSONPatch", │
       │      patch: [...]            │
       │    }                         │
       │  }                           │
```

---

## 10. The Watch Mechanism — How Controllers Stay Notified

### Q: How does the scheduler know when a new Pod appears? How does kubelet know it has work to do?

The API server provides a **long-lived HTTP watch** endpoint. Every controller opens a persistent connection: `GET /api/v1/pods?watch=true`. The API server streams newline-delimited JSON events back as state changes occur.

```
  API Server Watch Architecture
  ┌─────────────────────────────────────────────────────────────────┐
  │                                                                 │
  │   etcd Watch Stream ──► Watch Cache (in-memory)                │
  │                                │                               │
  │                         ┌──────▼───────┐                       │
  │                         │  Event Queue │                       │
  │                         └──────┬───────┘                       │
  │                                │                               │
  │              ┌─────────────────┼─────────────────┐             │
  │              ▼                 ▼                 ▼             │
  │    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
  │    │  Scheduler   │  │  Controller  │  │  Kubelet     │        │
  │    │  Watch       │  │  Manager     │  │  Watch       │        │
  │    │  (Pods with  │  │  Watch       │  │  (Pods on    │        │
  │    │   no node)   │  │  (all types) │  │   this node) │        │
  │    └──────────────┘  └──────────────┘  └──────────────┘        │
  │                                                                 │
  └─────────────────────────────────────────────────────────────────┘

  Watch Event format (newline-delimited JSON over HTTP):
  {"type":"ADDED","object":{"kind":"Pod","metadata":{"name":"signin-portal-abc"},...}}
  {"type":"MODIFIED","object":{"kind":"Pod","metadata":{"name":"signin-portal-abc"},...}}
  {"type":"DELETED","object":{"kind":"Pod","metadata":{"name":"signin-portal-abc"},...}}
```

### Why the Watch Cache Matters

Without the watch cache, every `LIST` from every controller would hit etcd directly. With thousands of Pods across hundreds of nodes, that would destroy etcd. The watch cache:
- Stores a full in-memory snapshot of all objects per resource type
- Serves `LIST` and `WATCH` requests from memory
- Only the API server itself maintains a single watch to etcd
- Fans this single etcd watch out to **N client watches** — this is how the API server protects etcd from overload

---

## 11. API Groups & Versioning

### Q: What is `/api` vs `/apis`?

```
  /api
  └── /v1                     ← Core group (legacy)
      ├── /pods
      ├── /services
      ├── /endpoints
      ├── /namespaces
      ├── /secrets
      ├── /configmaps
      ├── /nodes
      ├── /persistentvolumes
      └── /persistentvolumeclaims

  /apis
  ├── /apps
  │   └── /v1
  │       ├── /deployments
  │       ├── /replicasets
  │       ├── /statefulsets
  │       └── /daemonsets
  │
  ├── /batch
  │   └── /v1
  │       ├── /jobs
  │       └── /cronjobs
  │
  ├── /networking.k8s.io
  │   └── /v1
  │       ├── /ingresses
  │       └── /networkpolicies
  │
  ├── /rbac.authorization.k8s.io
  │   └── /v1
  │       ├── /clusterroles
  │       ├── /clusterrolebindings
  │       ├── /roles
  │       └── /rolebindings
  │
  └── /apiextensions.k8s.io      ← CRD definitions live here
      └── /v1
          └── /customresourcedefinitions
```

### API Version Maturity

| Version | Meaning |
|---|---|
| `v1alpha1` | Experimental — may be removed without notice |
| `v1beta1` | Near-stable — deprecated 9 months before removal |
| `v1` | Stable — long-term support guaranteed |

---

## 12. etcd: What Gets Stored & How

### Q: What is the exact format of data in etcd?

The API server serializes objects to **Protobuf** before writing to etcd (not JSON). This is more compact and faster to decode. A Deployment stored in etcd is a binary protobuf blob, prefixed with a magic header `k8s\x00`.

The human-readable equivalent of what is stored:

```json
{
  "kind": "Deployment",
  "apiVersion": "apps/v1",
  "metadata": {
    "name": "signin-portal",
    "namespace": "dev",
    "uid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "resourceVersion": "487234",
    "generation": 1,
    "creationTimestamp": "2025-08-15T09:00:00Z",
    "labels": { "app": "signin-portal" },
    "annotations": {
      "deployment.kubernetes.io/revision": "1"
    }
  },
  "spec": {
    "replicas": 2,
    "selector": { "matchLabels": { "app": "signin-portal" } },
    "template": { ... }
  },
  "status": {
    "replicas": 2,
    "readyReplicas": 2,
    "availableReplicas": 2,
    "updatedReplicas": 2
  }
}
```

### The `resourceVersion` Field

This is the etcd revision at which the object was last modified. It enables:
- **Optimistic concurrency**: updates must send the current `resourceVersion` or get `409 Conflict`
- **Watch resumption**: `GET /pods?watch=true&resourceVersion=487234` — resumes watch from that exact point, no events missed

### Secrets Encryption at Rest

```
  etcd stores:
  Key:   /registry/secrets/dev/db-secret
  Value: k8s:enc:aescbc:v1:key1:<encrypted-bytes>
               ↑       ↑      ↑
               prefix  algo   key-id

  Decrypted only inside the API server process.
  etcd never sees plaintext secret values.
```

---

## 13. High Availability Setup

### Q: What does the API server look like in a production HA cluster?

```
  ┌─────────────────────────────────────────────────────────────────────┐
  │                    PRODUCTION HA CONTROL PLANE                      │
  │                                                                     │
  │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
  │  │  Master Node 1   │  │  Master Node 2   │  │  Master Node 3   │  │
  │  │                  │  │                  │  │                  │  │
  │  │  kube-apiserver  │  │  kube-apiserver  │  │  kube-apiserver  │  │
  │  │  :6443           │  │  :6443           │  │  :6443           │  │
  │  │                  │  │                  │  │                  │  │
  │  │  kube-scheduler  │  │  kube-scheduler  │  │  kube-scheduler  │  │
  │  │  (standby)       │  │  (active/leader) │  │  (standby)       │  │
  │  │                  │  │                  │  │                  │  │
  │  │  kube-ctrl-mgr   │  │  kube-ctrl-mgr   │  │  kube-ctrl-mgr   │  │
  │  │  (standby)       │  │  (active/leader) │  │  (standby)       │  │
  │  └────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘  │
  │           │                     │                      │            │
  │           └─────────────────────┼──────────────────────┘            │
  │                                 │                                   │
  │  ┌──────────────────────────────▼──────────────────────────────┐    │
  │  │                    etcd cluster (3 or 5 members)            │    │
  │  │    etcd-1 (leader)    etcd-2 (follower)    etcd-3 (follower)│    │
  │  │                    Raft consensus                           │    │
  │  └─────────────────────────────────────────────────────────────┘    │
  │                                                                     │
  │  ┌──────────────────────────────────────────────────────────────┐   │
  │  │           Load Balancer (HAProxy / AWS NLB / etc.)           │   │
  │  │           Single endpoint for all kubectl clients            │   │
  │  │           Round-robin across all 3 API server replicas       │   │
  │  └──────────────────────────────────────────────────────────────┘   │
  └─────────────────────────────────────────────────────────────────────┘

  Key HA facts:
  • All API server replicas are ACTIVE (no leader election needed)
  • Scheduler and Controller Manager run in active/standby (leader election via etcd lease)
  • Any API server can serve any request
  • etcd handles consistency across all replicas via Raft
```

---

## 14. Security Surface

### Q: What are the main security concerns around the API server?

| Attack Surface | Risk | Mitigation |
|---|---|---|
| Unauthenticated access | Anyone on network reads/writes cluster state | Disable `--insecure-port`, enforce mTLS |
| Overly broad RBAC | Service accounts with cluster-admin | Principle of least privilege — minimal verbs/resources |
| Admission bypass | Malicious pods deployed without policy checks | Always enable ValidatingWebhookConfiguration |
| Secrets in etcd plaintext | etcd breach exposes all secrets | Enable EncryptionConfiguration with KMS provider |
| API server to etcd communication | MITM → full cluster compromise | Dedicated CA for etcd, not shared with Kubernetes CA |
| Audit log gaps | No trail of who did what | Enable audit logging at RequestResponse level |
| Token lifetime | Long-lived ServiceAccount tokens can be exfiltrated | Use projected volumes with short-lived tokens (1h TTL) |

### Audit Logging

The API server can log every request to an audit backend (file / webhook):

```yaml
# Audit policy — log everything at RequestResponse level for sensitive resources
rules:
  - level: RequestResponse
    resources:
      - group: ""
        resources: ["secrets", "configmaps"]
  - level: Metadata
    resources:
      - group: "apps"
        resources: ["deployments"]
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
      - group: ""
        resources: ["endpoints", "services"]
```

---

## 15. Key Config Flags

These are the most important `kube-apiserver` startup flags you will encounter:

```bash
kube-apiserver \
  # etcd connection
  --etcd-servers=https://etcd-1:2379,https://etcd-2:2379,https://etcd-3:2379 \
  --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt \
  --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt \
  --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key \

  # TLS / serving
  --tls-cert-file=/etc/kubernetes/pki/apiserver.crt \
  --tls-private-key-file=/etc/kubernetes/pki/apiserver.key \
  --client-ca-file=/etc/kubernetes/pki/ca.crt \
  --secure-port=6443 \

  # AuthN
  --service-account-key-file=/etc/kubernetes/pki/sa.pub \
  --service-account-issuer=https://kubernetes.default.svc.cluster.local \
  --oidc-issuer-url=https://auth.mycompany.com \
  --oidc-client-id=kubectl \

  # AuthZ
  --authorization-mode=Node,RBAC \

  # Admission
  --enable-admission-plugins=NodeRestriction,PodSecurity,ResourceQuota \

  # Encryption at rest
  --encryption-provider-config=/etc/kubernetes/enc/encryption.yaml \

  # Audit
  --audit-policy-file=/etc/kubernetes/audit-policy.yaml \
  --audit-log-path=/var/log/kubernetes/audit.log \
  --audit-log-maxage=30 \

  # Performance
  --max-requests-inflight=400 \
  --max-mutating-requests-inflight=200 \
  --watch-cache-sizes=pods#500,nodes#100
```

---

## 16. Q&A — 360° Questions Answered

---

**Q: Is the API server a single point of failure?**

In a single-master cluster — yes. That is why production clusters always run 3 or 5 master nodes with a load balancer in front. The API server is stateless, so any replica can serve any request.

---

**Q: What happens if etcd goes down?**

The API server stops accepting write requests (POST/PUT/PATCH/DELETE). Read requests may still be served from the watch cache depending on configuration. The cluster enters a read-only degraded mode — running workloads continue, but no new deployments, scaling, or scheduling can occur.

---

**Q: Can two controllers conflict by updating the same object simultaneously?**

Yes — and optimistic concurrency handles this. If two controllers both try to update a Pod at the same `resourceVersion`, the second write gets a `409 Conflict`. The losing controller must re-read the object and retry. This is why all Kubernetes controllers are designed to be **idempotent** — retrying the same update is always safe.

---

**Q: How does a new worker node register itself?**

The kubelet on the new node sends a POST to `/api/v1/nodes` with a `Node` object containing its hostname, capacity (CPU/RAM), and labels. The API server validates the request (the node must have a valid bootstrap token or client cert), creates the Node object in etcd, and the node is now part of the cluster and eligible for scheduling.

---

**Q: What is the difference between a List and a Watch?**

A `List` (`GET /api/v1/pods`) returns a point-in-time snapshot of all matching objects, along with the current `resourceVersion`. A `Watch` (`GET /api/v1/pods?watch=true`) keeps the HTTP connection open and streams events as objects change. Most controllers do a List first (to load current state), then immediately open a Watch (to stay updated) — this pattern is called **List-Watch**.

---

**Q: What is an Aggregated API Server?**

The main API server can delegate certain API paths to external API servers via the **Aggregation Layer**. For example, `metrics-server` registers itself to handle `/apis/metrics.k8s.io/v1beta1`. When kubectl calls `kubectl top pods`, the main API server proxies the request to `metrics-server`. CRDs work differently — they are handled by the main API server itself with a generic handler.

---

**Q: How does kubectl get autocompletion for resource fields?**

The API server exposes its full OpenAPI v3 schema at `/openapi/v3`. kubectl downloads this schema and uses it to validate manifests client-side (`kubectl apply --dry-run=client`) and to provide field completion in IDEs and terminals.

---

**Q: What is a SubjectAccessReview?**

It's an API object you can `POST` to `/apis/authorization.k8s.io/v1/subjectaccessreviews` to ask the API server "can user X do action Y?". This is how admission webhooks, CI systems, and tools like `kubectl auth can-i` check permissions without attempting the actual operation.

```bash
# Check if the current user can create deployments in namespace dev
kubectl auth can-i create deployments --namespace dev

# Check as a specific service account
kubectl auth can-i list pods --as=system:serviceaccount:dev:ci-bot -n dev
```

---

**Q: What is the difference between `/status` and the main resource endpoint?**

Kubernetes objects split their data into `spec` (desired state, set by users/controllers) and `status` (actual state, set by controllers). The `/status` subresource endpoint allows controllers to update the `status` field without needing permission to update the full object. For example, the Deployment controller updates `deployment.status.readyReplicas` via `PATCH /apis/apps/v1/namespaces/dev/deployments/signin-portal/status` — it does not need write permission on the Deployment spec itself.

---

**Q: How are CRDs handled by the API server?**

When you create a `CustomResourceDefinition`, the API server's built-in `apiextensions` server:
1. Validates and stores the CRD in etcd under `/registry/apiextensions.k8s.io/customresourcedefinitions/`
2. Dynamically registers a new REST endpoint (e.g. `/apis/mygroup.io/v1/myresources`)
3. Generates an OpenAPI schema from the CRD validation spec
4. All CRUD and watch operations on custom resources work identically to built-in resources

---

## Summary — The API Server in One Paragraph

The `kube-apiserver` is the **sole gateway** between the outside world and the cluster's state. Every request travels through a layered pipeline: TLS transport → authentication (who are you?) → authorization (are you allowed?) → mutating admission (transform the object) → schema validation → validating admission (policy check) → etcd persistence. It is stateless itself — etcd holds all state. It protects etcd by serving most reads from an in-memory watch cache and fanning a single etcd watch stream out to all connected controllers. In production it runs in 3+ replicas behind a load balancer. Its audit log, RBAC engine, and admission webhook system together form the security and governance backbone of the entire cluster.

---

*Document generated for: Kubernetes API Server 360° Deep Dive | Version: Kubernetes 1.29+*
