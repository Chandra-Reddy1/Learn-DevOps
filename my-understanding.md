# Kubernetes Quick Notes (Minimal + Flows)

---

## 1. What is Kubernetes and why is it used?
- Container orchestration platform  
- Manages deployment, scaling, networking, and availability  

---

## 2. Core components of Kubernetes architecture
- Control Plane: API Server, Scheduler, Controller Manager, etcd  
- Worker Node: Kubelet, Kube Proxy, Pods  

---

## 3. What is a Pod?
- Smallest deployable unit  
- Runs one or more containers  

---

## 4. What is a Deployment and how does it work?
- Manages stateless apps  
- Handles scaling + rolling updates  

Flow:
Deployment → ReplicaSet → Pods

---

## 5. What is a Service?
- Stable endpoint to access Pods  
- Load balances traffic  

Flow:
Client → Service → Pod

---

## 6. Types of Services
- ClusterIP → internal access  
- NodePort → node IP access  
- LoadBalancer → external LB  

---

## 7. What is a Namespace?
- Logical isolation in cluster  
- Used for multi-tenancy  

---

## 8. Deployment vs StatefulSet
- Deployment → stateless  
- StatefulSet → stateful (identity + storage + order)  

---

## 9. What is a DaemonSet?
- Runs 1 pod per node  
- Used for agents (logging, monitoring)  

---

## 10. What is Ingress and how does it work?
- Manages external HTTP/HTTPS routing  

Flow:
User → LoadBalancer → Ingress → Service → Pod

---

## 11. ConfigMaps and Secrets
- ConfigMap → non-sensitive config  
- Secret → sensitive data (base64 encoded)  

---

## 12. Liveness & Readiness Probes
- Liveness → restart container  
- Readiness → control traffic  

---

## 13. Horizontal Pod Autoscaler (HPA)
- Auto scales pods based on CPU/memory  

---

## 14. RBAC in Kubernetes
- Role-based access control  
- Defines permissions  

---

## 15. Role vs ClusterRole
- Role → namespace level  
- ClusterRole → cluster-wide  

---

## 16. Kubernetes networking (internal)
- Every pod gets unique IP  
- Flat network (no NAT between pods)  

---

## 17. What is CNI?
- Container Network Interface  
- Handles pod networking  

---

## 18. Service discovery
- DNS-based  

Example:  
service-name.namespace.svc.cluster.local  

---

## 19. What is etcd?
- Key-value store for cluster state  

Backup:  
etcdctl snapshot save  

Restore:  
etcdctl snapshot restore  

---

## 20. Taints and Tolerations
- Taint → restrict node  
- Toleration → allow pod  

---

## 21. Affinity / Anti-affinity
- Affinity → schedule together  
- Anti-affinity → avoid scheduling together  

---

## 22. Securing Kubernetes cluster
- RBAC  
- Network policies  
- TLS  
- Secrets management  

---

## 23. What is Helm?
- Package manager for Kubernetes  
- Uses charts for deployment  

---

## 24. CrashLoopBackOff troubleshooting
- kubectl logs <pod>  
- kubectl describe pod <pod>  

---

## 25. Service not accessible troubleshooting
- kubectl get svc  
- kubectl get endpoints  
- Verify labels/selectors  

---

## 26. Zero-downtime deployment
- Rolling update  
- Use readiness probes  

Config:  
maxUnavailable: 0  
maxSurge: 1  

---

## 27. Node not ready troubleshooting
- kubectl describe node <node>  
- Check kubelet  
- Check resources  

---

## 28. High CPU/memory handling
- Scale pods (HPA)  
- Optimize requests/limits  

---

## 29. Auto scaling applications
- Use HPA  

---

## 30. Monitoring and logging
- Monitoring: Prometheus, Grafana  
- Logging: EFK stack  

---

## Full Request Flow

User → DNS → LoadBalancer → Ingress → Service → Pod
