###
###EKS CLUSTER
│
├── CONTROL PLANE
│   │
│   ├── API Server
│   ├── etcd
│   ├── Scheduler
│   └── Controller Manager
│
├── WORKER NODES
│   │
│   ├── kubelet
│   ├── kube-proxy
│   ├── Container Runtime
│   ├── CNI
│   └── Pods
│
├── KUBERNETES OBJECTS
│   │
│   ├── Deployment
│   ├── ReplicaSet
│   ├── Service
│   ├── Ingress
│   ├── ConfigMap
│   ├── Secret
│   ├── Namespace
│   ├── HPA
│   ├── DaemonSet
│   ├── StatefulSet
│   ├── PV/PVC
│   └── RBAC
│
└── AWS INTEGRATION
    │
    ├── VPC
    ├── AWS VPC CNI
    ├── ALB/NLB
    ├── Load Balancer Controller
    ├── IAM
    ├── ECR
    ├── EBS/EFS
    └── CloudWatch / other monitoring


    | Component              | Remember this                              |
| ---------------------- | ------------------------------------------ |
| **EKS**                | Managed Kubernetes service                 |
| **Control Plane**      | Brain of Kubernetes                        |
| **API Server**         | Entry point/API of Kubernetes              |
| **etcd**               | Stores cluster state                       |
| **Scheduler**          | Chooses node for Pod                       |
| **Controller Manager** | Maintains desired state                    |
| **Worker Node**        | Runs workloads                             |
| **kubelet**            | Manages Pods on a node                     |
| **kube-proxy**         | Implements Service networking rules        |
| **Container Runtime**  | Runs containers                            |
| **CNI**                | Provides Pod networking                    |
| **Pod**                | Smallest deployable unit                   |
| **Deployment**         | Manages stateless Pods                     |
| **ReplicaSet**         | Maintains Pod replicas                     |
| **Service**            | Stable access to Pods                      |
| **Ingress**            | HTTP/HTTPS routing                         |
| **CoreDNS**            | Kubernetes DNS/service discovery           |
| **ConfigMap**          | Non-sensitive configuration                |
| **Secret**             | Sensitive configuration                    |
| **Namespace**          | Logical resource isolation                 |
| **RBAC**               | Kubernetes authorization                   |
| **HPA**                | Automatically changes Pod count            |
| **DaemonSet**          | Runs Pods on eligible nodes                |
| **StatefulSet**        | Manages stateful workloads                 |
| **PVC/PV**             | Persistent storage                         |
| **VPC CNI**            | AWS Pod networking                         |
| **AWS LB Controller**  | Manages AWS load balancers from Kubernetes |
| **IAM**                | AWS permissions                            |
| **ECR**                | Container image registry                   |

| Component                        | 1–2 Line Explanation                                                                                                                                             | Main Use                                     |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- |
| **EKS**                          | Amazon Elastic Kubernetes Service is AWS's managed Kubernetes service. AWS manages the Kubernetes control plane, while you manage workloads and worker capacity. | Run containerized applications on AWS        |
| **Control Plane**                | The control plane is the brain of Kubernetes that manages the cluster and maintains the desired state. In EKS, AWS manages it.                                   | Manage and control the Kubernetes cluster    |
| **API Server**                   | The `kube-apiserver` is the central API endpoint through which `kubectl`, controllers, scheduler, and other components communicate with Kubernetes.              | Receive and process Kubernetes API requests  |
| **etcd**                         | `etcd` is Kubernetes' distributed key-value database that stores cluster state and configuration.                                                                | Store Kubernetes cluster state               |
| **Scheduler**                    | The scheduler finds suitable worker nodes for Pods that haven't been assigned to a node yet.                                                                     | Decide where Pods should run                 |
| **Controller Manager**           | Controllers continuously compare the desired state with the actual state and take corrective action.                                                             | Maintain the desired state                   |
| **Worker Node**                  | A worker node is a machine, such as an EC2 instance, where Kubernetes Pods run.                                                                                  | Run application workloads                    |
| **kubelet**                      | kubelet is the agent running on each worker node that communicates with the API server and ensures assigned Pods/containers are running.                         | Manage Pods on a node                        |
| **kube-proxy**                   | kube-proxy helps implement Kubernetes Service networking and routes Service traffic toward backend Pods.                                                         | Enable Service-to-Pod networking             |
| **Container Runtime**            | The container runtime is the software that actually creates and runs containers. EKS commonly uses `containerd`.                                                 | Run containers                               |
| **CNI**                          | CNI (Container Network Interface) provides networking connectivity and IP addresses to Pods.                                                                     | Provide Pod networking                       |
| **Pod**                          | A Pod is Kubernetes' smallest deployable unit and contains one or more containers that share network/storage resources.                                          | Run application containers                   |
| **Deployment**                   | A Deployment manages the desired number and versions of stateless Pods and supports rolling updates and rollbacks.                                               | Deploy and update applications               |
| **ReplicaSet**                   | A ReplicaSet ensures the specified number of Pod replicas are running. It is normally managed by a Deployment.                                                   | Maintain Pod replicas                        |
| **Service**                      | A Service provides a stable network endpoint for accessing a group of Pods, even when Pod IPs change.                                                            | Service discovery and load balancing         |
| **Ingress**                      | Ingress defines HTTP/HTTPS routing rules, typically routing requests based on hostname or URL path.                                                              | Route external web traffic to Services       |
| **CoreDNS**                      | CoreDNS provides DNS-based service discovery inside the Kubernetes cluster.                                                                                      | Resolve Service names to IP addresses        |
| **ConfigMap**                    | A ConfigMap stores non-sensitive application configuration separately from the container image.                                                                  | Store environment/configuration values       |
| **Secret**                       | A Secret stores sensitive configuration such as passwords, tokens, or credentials.                                                                               | Store sensitive data                         |
| **Namespace**                    | A Namespace logically separates Kubernetes resources within the same cluster.                                                                                    | Environment/team/resource isolation          |
| **RBAC**                         | Role-Based Access Control defines what users, groups, or service accounts are allowed to do in Kubernetes.                                                       | Control Kubernetes permissions               |
| **HPA**                          | Horizontal Pod Autoscaler automatically increases or decreases the number of Pod replicas based on metrics such as CPU or memory.                                | Automatically scale Pods                     |
| **DaemonSet**                    | A DaemonSet ensures a Pod runs on each eligible node, commonly one Pod per node.                                                                                 | Node-level agents such as logging/monitoring |
| **StatefulSet**                  | StatefulSet manages stateful applications that require stable Pod identities and persistent storage.                                                             | Run databases/stateful workloads             |
| **PVC/PV**                       | A PersistentVolume (PV) represents storage available to Kubernetes, while a PersistentVolumeClaim (PVC) requests storage for a workload.                         | Provide persistent storage                   |
| **VPC CNI**                      | AWS VPC CNI integrates Kubernetes Pod networking with the AWS VPC and assigns Pods VPC networking/IP addresses.                                                  | Connect Pods to AWS VPC networking           |
| **AWS Load Balancer Controller** | This controller watches Kubernetes resources such as Ingress and Service and creates/configures AWS load balancers accordingly.                                  | Integrate Kubernetes with AWS ALB/NLB        |
| **IAM**                          | AWS Identity and Access Management controls access to AWS resources through users, roles, and policies.                                                          | Control AWS permissions                      |
| **ECR**                          | Amazon Elastic Container Registry is AWS's private container image registry. Kubernetes nodes pull application images from it.                                   | Store and distribute Docker/container images |

