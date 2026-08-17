# EKS Scenario-Based Interview Questions & Troubleshooting

## How to use this document

Use these questions as an interview drill. For each scenario:

1.  Identify the layer where the failure is occurring.
2.  State the commands you would run.
3.  Explain what the output would tell you.
4.  Give the likely root cause.
5.  Explain the fix.
6.  Mention how you would prevent the issue from happening again.

A strong EKS troubleshooting answer usually follows this path:

``` text
DNS
  ↓
Load Balancer
  ↓
Ingress / Service
  ↓
Endpoints / EndpointSlices
  ↓
Pod
  ↓
Container
  ↓
Node
  ↓
EKS / AWS networking
  ↓
IAM / Security
```

------------------------------------------------------------------------

# 1. EKS Architecture Scenarios

## Scenario 1 --- Explain the complete request flow

### Interview question

A user opens `https://app.example.com`. Walk me through the request from
the browser until it reaches the application pod running on EKS.

### Expected answer

A strong answer:

> Route 53 resolves the application domain to the load balancer
> endpoint. The AWS Load Balancer Controller manages the AWS load
> balancer based on Kubernetes resources such as Ingress. The ALB
> receives the request and routes it to a healthy target. Depending on
> the configuration, the target can be a pod IP or a node/NodePort.
> Kubernetes Services provide stable discovery of pods, while the AWS
> Load Balancer Controller keeps AWS target groups synchronized with
> Kubernetes endpoints. The Amazon VPC CNI provides pod networking and
> IP addresses.

### Important correction

Do **not** say:

> Route 53 creates a public IP.

Route 53 is DNS. It resolves names to records/targets.

------------------------------------------------------------------------

## Scenario 2 --- A new pod has a new IP

### Interview question

A Deployment creates a new pod. The pod gets a completely new IP. How
does traffic eventually reach that new pod?

### Expected answer

> The VPC CNI provides networking and an IP address for the pod. Once
> the pod becomes ready, Kubernetes updates its endpoint information.
> The AWS Load Balancer Controller watches Kubernetes resources and
> reconciles the AWS target group, registering the appropriate target.
> The load balancer health-checks the target before sending traffic.

### Key distinction

``` text
VPC CNI
  ↓
Provides pod networking/IP

Kubernetes
  ↓
Maintains endpoint information

AWS Load Balancer Controller
  ↓
Synchronizes AWS load balancer targets

ALB/NLB
  ↓
Health checks and routes traffic
```

------------------------------------------------------------------------

# 2. EKS Access and Authentication

## Scenario 3 --- kubectl cannot access EKS

### Interview question

You run:

``` bash
kubectl get nodes
```

and receive:

``` text
You must be logged in to the server
```

How do you troubleshoot?

### Check

``` bash
aws sts get-caller-identity

aws eks describe-cluster \
  --region <region> \
  --name <cluster>

kubectl config current-context

kubectl config get-contexts

aws eks update-kubeconfig \
  --region <region> \
  --name <cluster>
```

Then:

``` bash
kubectl get nodes
```

### Possible causes

-   Wrong AWS profile
-   Wrong IAM role
-   Wrong cluster
-   Wrong region
-   Missing EKS permissions
-   IAM identity not authorized for Kubernetes access
-   Expired or invalid AWS credentials

### Interview answer

> First I verify the AWS identity with `aws sts get-caller-identity`,
> then verify the cluster and kubeconfig context. If AWS authentication
> works but Kubernetes authorization fails, I check EKS access entries
> or the relevant Kubernetes RBAC configuration.

------------------------------------------------------------------------

## Scenario 4 --- Wrong AWS account

### Interview question

`kubectl` is pointing to an EKS cluster, but you cannot authenticate.
What would you check first?

### Answer

Check:

``` bash
aws sts get-caller-identity
aws eks list-clusters --region <region>
kubectl config current-context
```

A common mistake is using credentials from the wrong AWS account or
profile.

------------------------------------------------------------------------

## Scenario 5 --- Multiple AWS profiles

### Interview question

Your company has multiple AWS accounts. How do you connect to the
correct EKS cluster?

### Answer

Use the appropriate profile:

``` bash
aws sts get-caller-identity --profile production

aws eks update-kubeconfig \
  --region <region> \
  --name <cluster> \
  --profile production
```

You can also use role assumption depending on the organization's IAM
design.

------------------------------------------------------------------------

# 3. EKS API Server Connectivity

## Scenario 6 --- EKS API endpoint is unreachable

### Interview question

`kubectl get nodes` returns:

``` text
Unable to connect to the server
```

What do you check?

### Troubleshooting

Check cluster endpoint configuration:

``` bash
aws eks describe-cluster \
  --region <region> \
  --name <cluster> \
  --query "cluster.resourcesVpcConfig"
```

Check whether the endpoint is public, private, or both.

Then check:

-   VPN connectivity
-   VPC routing
-   security groups
-   network ACLs
-   DNS
-   corporate proxy/firewall
-   private endpoint access

### Important point

If the EKS API endpoint is private-only, your laptop must have network
connectivity into the VPC.

------------------------------------------------------------------------

# 4. Nodes

## Scenario 7 --- Node is NotReady

### Interview question

One EKS node shows:

``` text
NotReady
```

How do you troubleshoot?

### Commands

``` bash
kubectl get nodes

kubectl describe node <node>

kubectl get pods -A -o wide
```

Check:

-   kubelet health
-   node conditions
-   disk pressure
-   memory pressure
-   PID pressure
-   networking
-   CNI
-   container runtime
-   node IAM role
-   EC2 instance status

### Look at:

``` text
Conditions:
  MemoryPressure
  DiskPressure
  PIDPressure
  Ready
```

------------------------------------------------------------------------

## Scenario 8 --- Node has DiskPressure

### Interview question

A node is `NotReady` because of `DiskPressure`. What do you do?

### Check

``` bash
kubectl describe node <node>
```

Look for:

``` text
DiskPressure=True
```

Then investigate:

-   container images
-   container logs
-   ephemeral storage
-   unused images
-   application temporary files

### Fix

Clean unnecessary data, increase node disk capacity, tune log retention,
or replace/scale nodes depending on the cause.

------------------------------------------------------------------------

## Scenario 9 --- Node has MemoryPressure

### Interview question

A node has `MemoryPressure=True`. What could cause it?

### Possible causes

-   Pods requesting too little memory
-   Memory leaks
-   Too many workloads
-   Missing resource limits
-   System processes consuming memory

Check:

``` bash
kubectl describe node <node>

kubectl top nodes
kubectl top pods -A
```

If metrics are available.

------------------------------------------------------------------------

# 5. Pod Pending

## Scenario 10 --- Pod stuck in Pending

### Interview question

A Deployment has five replicas, but two pods are stuck in `Pending`.
What do you check?

### Commands

``` bash
kubectl get pods

kubectl describe pod <pod>
```

Look at the Events section.

Also:

``` bash
kubectl get nodes
kubectl describe nodes
```

### Common causes

-   Insufficient CPU
-   Insufficient memory
-   Taints
-   Missing tolerations
-   Node selectors
-   Affinity rules
-   Topology constraints
-   PVC problems
-   No suitable node
-   Autoscaling configuration problems

------------------------------------------------------------------------

## Scenario 11 --- Pending pod says insufficient CPU

### Interview question

Events show:

``` text
0/X nodes are available: Insufficient cpu
```

What happens next?

### Answer

The scheduler cannot place the pod because its CPU request cannot be
satisfied.

If node autoscaling is configured:

``` text
Pod Pending
   ↓
Karpenter / Cluster Autoscaler
   ↓
Additional capacity
   ↓
Pod scheduled
```

Without node autoscaling, you need to increase node capacity manually or
change workload/resource requirements.

------------------------------------------------------------------------

# 6. CrashLoopBackOff

## Scenario 12 --- Pod is CrashLoopBackOff

### Interview question

A pod is continuously restarting and shows:

``` text
CrashLoopBackOff
```

How do you troubleshoot?

### Commands

``` bash
kubectl get pod <pod>

kubectl describe pod <pod>

kubectl logs <pod>

kubectl logs <pod> --previous
```

Check:

-   application crash
-   incorrect configuration
-   missing environment variable
-   missing Secret
-   missing ConfigMap
-   dependency unavailable
-   incorrect command/entrypoint
-   permissions
-   OOMKilled
-   failed liveness probe

### Strong interview answer

> I would first inspect the current and previous container logs, then
> describe the pod and check Events. I would determine whether the
> container is actually crashing or whether kubelet is restarting it
> because of a failing liveness probe.

------------------------------------------------------------------------

## Scenario 13 --- Pod is OOMKilled

### Interview question

A container repeatedly gets:

``` text
OOMKilled
```

What does that mean?

### Answer

The container exceeded its memory limit or the node experienced memory
pressure and the process was killed.

Check:

``` bash
kubectl describe pod <pod>
kubectl top pod <pod>
```

Then review:

-   memory request
-   memory limit
-   actual usage
-   application memory behavior

------------------------------------------------------------------------

# 7. ImagePullBackOff

## Scenario 14 --- EKS cannot pull an ECR image

### Interview question

A pod shows:

``` text
ImagePullBackOff
```

The image is stored in ECR. What do you check?

### Commands

``` bash
kubectl describe pod <pod>
```

Look at Events.

Check:

-   repository name
-   image tag
-   ECR repository
-   node/workload IAM permissions
-   network connectivity
-   image architecture
-   registry authentication

### Typical flow

``` text
EKS node
   ↓
ECR authentication
   ↓
ECR repository
   ↓
Container image
```

------------------------------------------------------------------------

# 8. Readiness and Liveness

## Scenario 15 --- Application starts slowly

### Interview question

Your application takes two minutes to start. You configured only a
liveness probe, and the pod keeps restarting. Why?

### Answer

The liveness probe may start checking before the application is ready,
causing Kubernetes to restart the container repeatedly.

### Better design

Use:

-   startup probe
-   readiness probe
-   appropriately configured liveness probe

------------------------------------------------------------------------

## Scenario 16 --- Pod is Running but receives no traffic

### Interview question

`kubectl get pods` says `Running`, but users cannot reach the pod. What
do you check?

### Important point

`Running` does not mean `Ready`.

Run:

``` bash
kubectl get pods
kubectl describe pod <pod>
```

Check:

``` text
READY
```

Then:

``` bash
kubectl get endpoints <service>
kubectl get endpointslices
```

If the pod isn't ready, it may not appear as a ready endpoint.

------------------------------------------------------------------------

# 9. Kubernetes Service

## Scenario 17 --- Service has no endpoints

### Interview question

You run:

``` bash
kubectl get endpoints <service>
```

and see no endpoints. Why?

### Check

``` bash
kubectl get svc <service> -o yaml

kubectl get pods --show-labels

kubectl describe svc <service>
```

The most common cause is a label/selector mismatch.

Example:

``` yaml
selector:
  app: web
```

But the pod has:

``` yaml
labels:
  app: frontend
```

Then the Service selects nothing.

------------------------------------------------------------------------

## Scenario 18 --- Service selector is correct but still no endpoints

### Check

-   Pod readiness
-   EndpointSlices
-   namespace
-   Service selector
-   pod labels
-   readiness gates
-   whether pods are actually running

Commands:

``` bash
kubectl get endpointslices

kubectl describe pod <pod>

kubectl get pods -l app=<label>
```

------------------------------------------------------------------------

# 10. ALB / Ingress

## Scenario 19 --- Ingress exists but ALB is not created

### Interview question

You apply an Ingress, but no ALB appears in AWS. What do you check?

### Commands

``` bash
kubectl get ingress

kubectl describe ingress <ingress>

kubectl get pods -n kube-system
```

Check AWS Load Balancer Controller:

``` bash
kubectl get deployment -n kube-system
```

Then inspect controller logs:

``` bash
kubectl logs -n kube-system deployment/aws-load-balancer-controller
```

### Common causes

-   Controller not installed
-   Controller not running
-   Incorrect IngressClass
-   IAM permissions missing
-   subnet discovery/tagging problems
-   incorrect annotations
-   security group problems

------------------------------------------------------------------------

## Scenario 20 --- ALB exists but target is unhealthy

### Interview question

The ALB exists, but its targets are unhealthy. What do you check?

### Trace the entire path

``` text
ALB
 ↓
Target Group
 ↓
Target
 ↓
Pod
 ↓
Container
```

Check:

``` bash
kubectl get ingress

kubectl describe ingress <ingress>

kubectl get svc

kubectl describe svc <service>

kubectl get endpoints <service>

kubectl get pods -o wide
```

Then check the AWS target group's:

-   health-check path
-   health-check port
-   target type
-   target status

Also check:

-   security groups
-   application listening port
-   readiness
-   Service targetPort
-   containerPort

------------------------------------------------------------------------

# 11. ALB 502 / 503

## Scenario 21 --- ALB returns 502

### Possible causes

-   backend connection failure
-   wrong target port
-   application not listening
-   security group issue
-   unhealthy pod
-   protocol mismatch

### Commands

``` bash
kubectl describe ingress <ingress>
kubectl describe svc <service>
kubectl get endpoints <service>
kubectl logs <pod>
```

------------------------------------------------------------------------

## Scenario 22 --- ALB returns 503

### Likely issue

There may be no healthy backend targets.

Check:

``` bash
kubectl get pods
kubectl get endpoints
kubectl describe ingress
```

Then inspect target group health in AWS.

------------------------------------------------------------------------

# 12. ALB Target Type

## Scenario 23 --- IP target vs Instance target

### Interview question

What is the difference?

### Answer

**IP mode:**

``` text
ALB
 ↓
Pod IP
```

**Instance mode:**

``` text
ALB
 ↓
Node
 ↓
NodePort
 ↓
Service
 ↓
Pod
```

Do not mix these two flows.

------------------------------------------------------------------------

# 13. AWS Load Balancer Controller

## Scenario 24 --- Controller is crashing

### Interview question

AWS Load Balancer Controller is in `CrashLoopBackOff`. What do you do?

### Commands

``` bash
kubectl get pods -n kube-system

kubectl describe pod <controller-pod> -n kube-system

kubectl logs <controller-pod> -n kube-system
```

Check:

-   IAM permissions
-   ServiceAccount
-   OIDC/Pod Identity configuration
-   webhook
-   controller arguments
-   Kubernetes compatibility
-   networking

------------------------------------------------------------------------

## Scenario 25 --- Controller has IAM AccessDenied

### Interview question

Controller logs show:

``` text
AccessDenied
```

What do you check?

### Answer

The controller needs an AWS IAM role with the appropriate permissions.

Check its ServiceAccount:

``` bash
kubectl get serviceaccount -n kube-system aws-load-balancer-controller -o yaml
```

Then verify the IAM association mechanism and role permissions.

------------------------------------------------------------------------

# 14. VPC CNI

## Scenario 26 --- Pods cannot get IP addresses

### Interview question

New pods stay Pending and events suggest networking/IP allocation
problems. What do you check?

### Commands

``` bash
kubectl get pods -n kube-system

kubectl get daemonset -n kube-system aws-node

kubectl logs -n kube-system daemonset/aws-node
```

Check:

-   VPC CNI health
-   subnet available IPs
-   node ENI/IP capacity
-   security groups
-   IAM permissions
-   prefix delegation configuration
-   custom networking configuration

------------------------------------------------------------------------

## Scenario 27 --- Pod IP exhaustion

### Interview question

The cluster has enough CPU and memory, but new pods cannot get IP
addresses. Why?

### Answer

The subnet or node ENI/IP capacity may be exhausted.

This is a classic EKS-specific issue.

Check:

-   subnet free IPs
-   ENI limits
-   pod density
-   VPC CNI configuration
-   prefix delegation

------------------------------------------------------------------------

# 15. Security Groups

## Scenario 28 --- ALB cannot reach pods

### Interview question

Pods are healthy locally, but the ALB cannot connect to them. What do
you check?

### Check

-   ALB security group
-   node security group
-   pod security group if configured
-   inbound rules
-   outbound rules
-   target port
-   network ACLs
-   routing

------------------------------------------------------------------------

## Scenario 29 --- Pod cannot access RDS

### Interview question

Your application pod cannot connect to RDS. What do you troubleshoot?

### Path

``` text
Pod
 ↓
Node/ENI networking
 ↓
VPC routing
 ↓
RDS security group
 ↓
RDS
```

Check:

-   RDS endpoint
-   DNS
-   port
-   RDS security group
-   pod/node security group
-   subnet routes
-   Network ACLs
-   database availability
-   credentials

------------------------------------------------------------------------

# 16. NAT Gateway / Internet Access

## Scenario 30 --- Private pod cannot reach the internet

### Interview question

A pod in a private subnet cannot download an external dependency. What
do you check?

### Typical path

``` text
Pod
 ↓
Private subnet
 ↓
Route table
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

Check:

-   route table
-   NAT Gateway
-   NAT subnet
-   Internet Gateway
-   security groups
-   NACLs
-   DNS

------------------------------------------------------------------------

# 17. DNS Inside EKS

## Scenario 31 --- Pod cannot resolve a domain

### Interview question

An application pod cannot resolve `api.example.com`. What do you check?

### Commands

``` bash
kubectl get pods -n kube-system

kubectl get svc -n kube-system
```

Check CoreDNS:

``` bash
kubectl get deployment coredns -n kube-system

kubectl logs -n kube-system deployment/coredns
```

Then test DNS from a pod:

``` bash
nslookup api.example.com
```

or:

``` bash
dig api.example.com
```

if available.

------------------------------------------------------------------------

## Scenario 32 --- Kubernetes Service DNS fails

### Example

The application cannot resolve:

``` text
my-service.default.svc.cluster.local
```

### Check

-   CoreDNS
-   Service existence
-   namespace
-   Service name
-   DNS configuration
-   network policies

------------------------------------------------------------------------

# 18. IAM / Pod Identity / IRSA

## Scenario 33 --- Pod cannot access S3

### Interview question

The application receives:

``` text
AccessDenied
```

when accessing S3. What do you check?

### Check

-   IAM role
-   Pod Identity or IRSA configuration
-   trust relationship
-   IAM policy
-   bucket policy
-   correct AWS account
-   correct region

### Important distinction

``` text
Kubernetes RBAC
   ↓
Kubernetes API permissions

IAM
   ↓
AWS API permissions
```

------------------------------------------------------------------------

## Scenario 34 --- Why not put AWS access keys in Kubernetes Secrets?

### Answer

> Long-lived access keys increase credential-management risk. For
> workloads, I prefer EKS Pod Identity or IRSA so AWS credentials are
> associated with the workload through IAM roles.

------------------------------------------------------------------------

# 19. Kubernetes RBAC

## Scenario 35 --- User can authenticate but cannot list pods

### Interview question

The user can access EKS but receives:

``` text
Forbidden
```

when running:

``` bash
kubectl get pods
```

What does that mean?

### Answer

Authentication succeeded, but authorization failed.

Check:

-   Kubernetes Role
-   ClusterRole
-   RoleBinding
-   ClusterRoleBinding
-   namespace
-   IAM-to-Kubernetes access configuration

------------------------------------------------------------------------

# 20. Autoscaling

## Scenario 36 --- HPA is not scaling

### Interview question

CPU is high, but HPA isn't creating more pods.

### Commands

``` bash
kubectl get hpa

kubectl describe hpa <hpa>

kubectl top pods
```

Check:

-   Metrics Server
-   resource requests
-   HPA configuration
-   metric availability
-   min/max replicas

------------------------------------------------------------------------

## Scenario 37 --- HPA creates pods but they remain Pending

### Answer

This is usually a node-capacity problem, not an HPA problem.

Flow:

``` text
High CPU
 ↓
HPA increases replicas
 ↓
Pods Pending
 ↓
No node capacity
 ↓
Karpenter / Cluster Autoscaler should add capacity
```

Check:

``` bash
kubectl describe pod <pending-pod>
```

and autoscaler logs/status.

------------------------------------------------------------------------

# 21. Karpenter / Cluster Autoscaler

## Scenario 38 --- Karpenter does not provision a node

### Check

-   Pod requirements
-   NodePool/Provisioner configuration
-   instance constraints
-   subnet discovery
-   security group discovery
-   IAM permissions
-   EC2 capacity
-   taints/tolerations
-   architecture constraints
-   resource requests

------------------------------------------------------------------------

## Scenario 39 --- Nodes are created but pods still don't schedule

### Possible causes

-   node taint
-   node selector mismatch
-   affinity
-   topology constraints
-   incompatible architecture
-   insufficient resources
-   PVC constraints

------------------------------------------------------------------------

# 22. Storage

## Scenario 40 --- PVC is Pending

### Interview question

A PVC remains:

``` text
Pending
```

What do you check?

### Commands

``` bash
kubectl get pvc
kubectl describe pvc <pvc>
kubectl get storageclass
```

Check:

-   StorageClass
-   CSI driver
-   IAM permissions
-   availability zones
-   volume limits
-   topology
-   provisioning errors

------------------------------------------------------------------------

## Scenario 41 --- Pod cannot mount EBS volume

### Check

``` bash
kubectl describe pod <pod>
kubectl describe pvc <pvc>
kubectl get pods -n kube-system
```

Investigate the EBS CSI driver.

Possible causes:

-   CSI driver unavailable
-   IAM problem
-   volume attachment issue
-   AZ mismatch
-   node issue
-   volume already attached elsewhere depending on access mode

------------------------------------------------------------------------

# 23. Secrets and ConfigMaps

## Scenario 42 --- Application starts but configuration is missing

### Check

``` bash
kubectl get configmap
kubectl get secret
kubectl describe pod <pod>
```

Verify:

-   correct namespace
-   correct key name
-   correct environment variable
-   volume mount
-   Secret/ConfigMap exists before deployment

------------------------------------------------------------------------

## Scenario 43 --- Secret changed but application still uses old value

### Answer

If the Secret is injected as an environment variable, changing the
Secret doesn't automatically update the existing process environment.

You may need to restart/roll the workload.

For mounted Secret volumes, Kubernetes can update mounted content, but
application behavior depends on whether and how the application reloads
it.

------------------------------------------------------------------------

# 24. Deployment and Rollouts

## Scenario 44 --- Deployment rollout is stuck

### Commands

``` bash
kubectl rollout status deployment/<deployment>

kubectl get rs

kubectl get pods

kubectl describe deployment <deployment>
```

Check:

-   image
-   readiness
-   resource requests
-   scheduling
-   probes
-   quota
-   PDB
-   admission webhook

------------------------------------------------------------------------

## Scenario 45 --- New version is broken

### Interview question

You deployed version 2 and production errors started. What do you do?

### Answer

First stop further rollout if necessary:

``` bash
kubectl rollout pause deployment/<deployment>
```

Then inspect:

``` bash
kubectl rollout history deployment/<deployment>
```

Rollback:

``` bash
kubectl rollout undo deployment/<deployment>
```

Then verify:

``` bash
kubectl rollout status deployment/<deployment>
```

------------------------------------------------------------------------

# 25. Zero Downtime

## Scenario 46 --- Deployment causes downtime

### Check

-   replicas
-   readiness probe
-   rolling update configuration
-   `maxUnavailable`
-   `maxSurge`
-   PodDisruptionBudget
-   graceful shutdown
-   termination grace period

A good answer:

> I would ensure the new pod becomes Ready before receiving traffic and
> keep sufficient old replicas available during the rollout.

------------------------------------------------------------------------

# 26. Node Maintenance

## Scenario 47 --- You need to terminate an EKS node

### Interview question

How do you safely remove it?

### Answer

Drain it:

``` bash
kubectl drain <node> \
  --ignore-daemonsets \
  --delete-emptydir-data
```

Then terminate/replace the node according to the node group's lifecycle.

Afterward:

``` bash
kubectl get nodes
```

------------------------------------------------------------------------

## Scenario 48 --- Drain fails because of a PodDisruptionBudget

### Answer

A PDB is preventing too many replicas from being unavailable.

Check:

``` bash
kubectl get pdb -A
kubectl describe pdb <pdb>
```

You should understand the availability requirement before modifying the
PDB.

------------------------------------------------------------------------

# 27. EKS Upgrade

## Scenario 49 --- How would you upgrade an EKS cluster?

### Answer

I would:

1.  Check supported upgrade paths.
2.  Review deprecated APIs.
3.  Check EKS upgrade insights.
4.  Check add-on compatibility.
5.  Upgrade the control plane.
6.  Upgrade managed node groups or replace self-managed nodes.
7.  Update EKS add-ons.
8.  Validate workloads.
9.  Monitor application health.

------------------------------------------------------------------------

## Scenario 50 --- Application breaks after Kubernetes upgrade

### Investigate

-   deprecated APIs
-   admission webhooks
-   CRDs
-   controllers
-   CNI
-   CoreDNS
-   kube-proxy
-   ingress controller
-   storage CSI driver
-   application manifests

------------------------------------------------------------------------

# 28. EKS Add-ons

## Scenario 51 --- CoreDNS is failing

### Commands

``` bash
kubectl get pods -n kube-system
kubectl describe pod -n kube-system <coredns-pod>
kubectl logs -n kube-system <coredns-pod>
```

Check:

-   scheduling
-   resource limits
-   DNS configuration
-   ConfigMap
-   network connectivity
-   version compatibility

------------------------------------------------------------------------

## Scenario 52 --- kube-proxy is unhealthy

### Check

``` bash
kubectl get daemonset kube-proxy -n kube-system

kubectl get pods -n kube-system -l k8s-app=kube-proxy

kubectl logs -n kube-system <pod>
```

Investigate node networking and compatibility.

------------------------------------------------------------------------

# 29. NetworkPolicy

## Scenario 53 --- Pod-to-pod communication suddenly stops

### Check

-   NetworkPolicy
-   security groups
-   CNI
-   routes
-   DNS
-   application ports

Commands:

``` bash
kubectl get networkpolicy -A

kubectl describe networkpolicy <policy>
```

Remember that Kubernetes NetworkPolicy and AWS security groups operate
at different layers.

------------------------------------------------------------------------

# 30. GitOps / Argo CD with EKS

## Scenario 54 --- Argo CD application is OutOfSync

### Check

``` bash
argocd app get <app>
```

Or in the UI inspect:

-   desired state
-   live state
-   diff
-   sync status
-   health

Possible causes:

-   Git changed
-   manual cluster change
-   unsupported field/defaulting
-   sync failure
-   wrong namespace
-   wrong cluster destination

------------------------------------------------------------------------

## Scenario 55 --- Argo CD cannot connect to EKS

### Troubleshooting

First verify from the machine/environment running Argo CD:

``` bash
kubectl get nodes
```

Then inspect the registered cluster:

``` bash
argocd cluster list
```

Check:

-   EKS API endpoint
-   network connectivity
-   credentials
-   IAM
-   Kubernetes permissions
-   cluster registration
-   private endpoint connectivity

### Critical point

If Argo CD is running locally, **Argo CD itself must be able to reach
the EKS API**, not just your terminal.

------------------------------------------------------------------------

## Scenario 56 --- Argo CD says cluster is unreachable

### Answer

I would distinguish:

``` text
Authentication problem
        vs
Authorization problem
        vs
Network connectivity problem
```

Then test each independently.

------------------------------------------------------------------------

# 31. ECR + Argo CD

## Scenario 57 --- Argo CD sync succeeds but pod fails to start

### Answer

Argo CD being healthy doesn't guarantee the workload is healthy.

Flow:

``` text
Git
 ↓
Argo CD
 ↓
Kubernetes manifest
 ↓
Deployment
 ↓
Pod
 ↓
Image pull
 ↓
Container
```

If image pull fails, troubleshoot ECR/IAM/networking separately.

------------------------------------------------------------------------

# 32. Production Incident Scenarios

## Scenario 58 --- All pods suddenly become unavailable

### Approach

Don't immediately restart everything.

Check:

``` bash
kubectl get nodes
kubectl get pods -A
kubectl get events -A --sort-by=.lastTimestamp
```

Then identify whether the failure is:

-   node-wide
-   namespace-wide
-   deployment-specific
-   networking
-   DNS
-   AWS infrastructure
-   application

------------------------------------------------------------------------

## Scenario 59 --- Application latency suddenly increases

### Investigate layers

``` text
ALB
 ↓
Target health
 ↓
Pod latency
 ↓
CPU/memory
 ↓
Node capacity
 ↓
Database
 ↓
External dependencies
```

Check:

-   ALB metrics
-   application metrics
-   pod CPU/memory
-   node CPU/memory
-   HPA
-   database latency
-   network errors

------------------------------------------------------------------------

## Scenario 60 --- Traffic increased 10x

### Interview question

What happens to the EKS application?

### Strong answer

> First the ALB distributes traffic across healthy targets. HPA may
> increase pod replicas based on configured metrics. If the existing
> nodes lack capacity, Karpenter or Cluster Autoscaler may provision
> additional nodes. New pods are scheduled onto available capacity,
> receive networking from the CNI, become Ready, and then become
> eligible for traffic.

------------------------------------------------------------------------

# 33. Fast Troubleshooting Commands

## Cluster

``` bash
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide
kubectl describe node <node>
```

## Pods

``` bash
kubectl get pods -A
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
```

## Services

``` bash
kubectl get svc -A
kubectl describe svc <service>
kubectl get endpoints <service>
kubectl get endpointslices
```

## Ingress

``` bash
kubectl get ingress -A
kubectl describe ingress <ingress>
```

## Deployments

``` bash
kubectl get deployment -A
kubectl describe deployment <deployment>
kubectl rollout status deployment/<deployment>
kubectl rollout history deployment/<deployment>
```

## Events

``` bash
kubectl get events -A --sort-by=.lastTimestamp
```

## Resource usage

``` bash
kubectl top nodes
kubectl top pods -A
```

## EKS

``` bash
aws sts get-caller-identity

aws eks describe-cluster \
  --region <region> \
  --name <cluster>

aws eks update-kubeconfig \
  --region <region> \
  --name <cluster>
```

------------------------------------------------------------------------

# 34. The Interviewer's Favorite "Why?" Questions

## Why does a pod get a new IP?

Because pods are ephemeral. The VPC CNI provides pod networking/IP
allocation.

## Why do we need a Service?

Because pod IPs are not stable. A Service provides stable service
discovery and load balancing.

## Why do we need an Ingress?

For HTTP/HTTPS routing and external load-balancer integration.

## Why do we need the AWS Load Balancer Controller?

To translate Kubernetes resources into AWS load-balancer configuration
and reconcile AWS resources with Kubernetes state.

## Why do we need readiness probes?

To prevent traffic from reaching an application that is running but not
ready to serve requests.

## Why do we need liveness probes?

To detect applications that are alive from the container-runtime
perspective but unhealthy and need restarting.

## Why do we need HPA?

To scale pod replicas based on workload metrics.

## Why do we need Karpenter/Cluster Autoscaler?

To provide additional node capacity when workloads cannot fit on
existing nodes.

## Why use private subnets?

To reduce direct exposure of worker infrastructure and keep application
nodes private.

## Why use IAM roles for pods?

To give workloads AWS permissions without distributing long-lived AWS
access keys.

------------------------------------------------------------------------

# 35. The Ultimate EKS Troubleshooting Decision Tree

When the interviewer gives you a problem, don't randomly run commands.

Use this:

``` text
                    User reports issue
                           │
                           ▼
                    Is DNS working?
                       /       \
                     No         Yes
                     │           │
                  Route 53       ▼
                            Is ALB working?
                             /        \
                           No          Yes
                           │            │
                    LB Controller      ▼
                    / IAM / subnet   Are targets healthy?
                                    /              \
                                  No                Yes
                                  │                  │
                           Target/Ingress          ▼
                           / Service            Is Service
                           / Pod                 healthy?
                                                /      \
                                              No        Yes
                                              │          │
                                           Pod/Service   ▼
                                                     Application
```

Then go one layer deeper:

``` text
Application problem?
        │
        ├── Pod Running?
        │
        ├── Pod Ready?
        │
        ├── Logs?
        │
        ├── Events?
        │
        ├── CPU/Memory?
        │
        ├── Dependencies?
        │
        └── Network?
```

------------------------------------------------------------------------

# 36. A Strong Interview Answer Pattern

When they give you any EKS incident, answer in this format:

### 1. State your hypothesis

> "First, I would determine whether this is a networking, Kubernetes,
> AWS load-balancer, or application issue."

### 2. Check the highest-level symptom

``` bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

### 3. Follow the traffic path

``` text
DNS
 ↓
ALB
 ↓
Ingress
 ↓
Service
 ↓
Endpoint
 ↓
Pod
 ↓
Container
```

### 4. Inspect events

``` bash
kubectl describe ...
kubectl get events -A --sort-by=.lastTimestamp
```

### 5. Check logs

``` bash
kubectl logs ...
```

### 6. Check AWS-specific components

-   ALB/NLB
-   Target groups
-   Security groups
-   VPC
-   subnets
-   IAM
-   ECR
-   CloudWatch

### 7. Fix the root cause

Don't just restart pods.

### 8. Verify

Run the same request/test again and confirm the system has recovered.

------------------------------------------------------------------------

# 37. 20 Rapid-Fire Questions

Practice answering these without looking at the answers.

1.  What is EKS?
2.  What does AWS manage in standard EKS?
3.  What is a managed node group?
4.  What does the VPC CNI do?
5.  What does kubelet do?
6.  What does kube-proxy do?
7.  What does the AWS Load Balancer Controller do?
8.  Route 53 vs ALB?
9.  ALB IP target vs Instance target?
10. Service vs Ingress?
11. Liveness vs readiness?
12. What happens when a pod crashes?
13. Why would a pod be Pending?
14. Why would a pod be CrashLoopBackOff?
15. Why would a pod be ImagePullBackOff?
16. Why would an ALB target be unhealthy?
17. HPA vs Karpenter?
18. IAM vs Kubernetes RBAC?
19. How does a pod access S3?
20. How would you troubleshoot an EKS application that is returning 503?

------------------------------------------------------------------------

# 38. Final Mental Model

If you remember only one architecture, remember this:

``` text
                         INTERNET
                            │
                            ▼
                       Route 53
                         DNS
                            │
                            ▼
                          ALB
                            │
                 AWS Load Balancer
                    Controller
                            │
                            ▼
                       Ingress
                            │
                            ▼
                        Service
                            │
                     Endpoints
                            │
                            ▼
                          Pod
                            │
                       Container
                            │
                            ▼
                         Node
                            │
                            ▼
                          EKS
                            │
                  ┌─────────┴─────────┐
                  │                   │
              VPC CNI             IAM/RBAC
                  │                   │
             Pod networking      Permissions
```

For scaling:

``` text
Traffic
  ↓
HPA
  ↓
More Pods
  ↓
No capacity?
  ↓
Karpenter / Cluster Autoscaler
  ↓
More Nodes
```

For a failing pod:

``` text
Pod failure
   ↓
Kubelet / Kubernetes detects problem
   ↓
Restart or replacement pod
   ↓
VPC CNI gives networking
   ↓
Pod becomes Ready
   ↓
Endpoint updated
   ↓
AWS Load Balancer Controller reconciles target
   ↓
ALB health check passes
   ↓
Traffic resumes
```

For AWS access from a workload:

``` text
Pod
 ↓
EKS Pod Identity / IRSA
 ↓
IAM Role
 ↓
AWS API
 ↓
S3 / Secrets Manager / DynamoDB / etc.
```

That is the core EKS mental model you should be able to explain without
notes.
