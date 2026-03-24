# Load Balancer Interview Questions & Answers (Top 20)

---

## 1. What is a Load Balancer?

A load balancer distributes incoming traffic across multiple servers to ensure high availability and reliability.

---

## 2. Why do we use a Load Balancer?

- Improve availability
- Distribute traffic
- Prevent server overload
- Enable scaling

---

## 3. What are types of Load Balancers in AWS?

- Application Load Balancer (ALB) → Layer 7 (HTTP/HTTPS)
- Network Load Balancer (NLB) → Layer 4 (TCP/UDP)
- Gateway Load Balancer (GWLB)

---

## 4. What is Layer 4 vs Layer 7?

- Layer 4 → Transport (IP, TCP, UDP)
- Layer 7 → Application (HTTP, HTTPS)

---

## 5. What is ALB?

Application Load Balancer routes traffic based on:
- Path
- Host
- Headers

---

## 6. What is NLB?

Network Load Balancer handles:
- High-performance traffic
- Low latency
- TCP/UDP connections

---

## 7. What is target group?

Group of resources (EC2, IP, Lambda) where traffic is routed.

---

## 8. What is health check?

Checks if backend servers are healthy before routing traffic.

---

## 9. What happens if instance is unhealthy?

Load balancer stops sending traffic to it.

---

## 10. What is listener?

Defines protocol and port (e.g., HTTP:80, HTTPS:443)

---

## 11. What is SSL termination?

Load balancer handles SSL decryption instead of backend servers.

---

## 12. What is sticky session?

Ensures user requests go to same backend server.

---

## 13. What is cross-zone load balancing?

Distributes traffic across all instances in multiple AZs.

---

## 14. What is path-based routing?

Routes traffic based on URL path:
- /api → service1
- /app → service2

---

## 15. What is host-based routing?

Routes based on domain:
- api.example.com
- app.example.com

---

## 16. What is autoscaling with load balancer?

Load balancer distributes traffic to dynamically scaled instances.

---

## 17. What is difference between ALB and NLB?

- ALB → Layer 7, advanced routing  
- NLB → Layer 4, high performance  

---

## 18. What is connection draining?

Allows existing connections to finish before removing instance.

---

## 19. What is idle timeout?

Time before closing inactive connections.

---

## 20. What is load balancing algorithm?

- Round Robin
- Least Connections
- IP Hash

---

# Bonus (High-Impact DevOps Questions)

---

## 21. How does traffic flow in ALB?

User → ALB → Target Group → EC2

---

## 22. How to secure Load Balancer?

- Use HTTPS (SSL)
- Use security groups
- Integrate with WAF

---

## 23. Can ALB work with Kubernetes?

Yes:
- ALB Ingress Controller routes traffic to services

---

## 24. What is difference between ELB and ALB?

ELB = Classic Load Balancer (older)  
ALB = modern, Layer 7

---

## 25. What is failover in load balancer?

Redirect traffic to healthy instances if failure occurs.

---

## 26. What is gateway load balancer?

Used for security appliances (firewalls, inspection)

---

## 27. What is deregistration delay?

Time before instance is removed from target group.

---

## 28. What is scaling impact?

Load balancer distributes traffic automatically to new instances.

---

## 29. How to debug load balancer issues?

- Check health checks
- Check target group
- Check logs (CloudWatch)

---

## 30. Real DevOps use case?

- ALB in public subnet
- EC2/EKS in private subnet
- Traffic routed securely

---

# Load Balancer Core Interview Explanations

---

# 1. ALB Traffic Flow

## Step-by-Step Explanation

### 1. User Request
- Client sends HTTP/HTTPS request from browser

---

### 2. DNS Resolution
- Domain (e.g., app.example.com) resolves to ALB DNS

---

### 3. Application Load Balancer (ALB)
- Receives incoming request
- Works at Layer 7 (HTTP/HTTPS)
- Evaluates listener rules:
  - Path-based routing
  - Host-based routing

---

### 4. Target Group
- ALB forwards request to target group
- Target group contains backend instances:
  - EC2 / IP / Lambda

---

### 5. Backend (Application)
- Request reaches application server
- Application processes request

---

### 6. Response Flow
- Backend → ALB → User

---

## Flow Summary

User → DNS → ALB → Target Group → Backend → Response

---

## One-Line Interview Answer

"User requests reach the ALB, which routes traffic based on rules to target groups, and then forwards it to backend instances."

---

# 2. ALB vs NLB Difference

## ALB (Application Load Balancer)

- Layer: 7 (Application)
- Protocols: HTTP, HTTPS
- Features:
  - Path-based routing
  - Host-based routing
  - SSL termination
- Use Case:
  - Web applications, APIs

---

## NLB (Network Load Balancer)

- Layer: 4 (Transport)
- Protocols: TCP, UDP
- Features:
  - High performance
  - Low latency
  - Static IP support
- Use Case:
  - High-throughput applications
  - Real-time systems

---

## Key Difference

- ALB → intelligent routing (content-based)  
- NLB → fast, simple routing (connection-based)  

---

## One-Line Interview Answer

"ALB operates at Layer 7 and supports advanced routing, while NLB operates at Layer 4 and is optimized for high-performance, low-latency traffic."

---

# 3. Health Check Behavior

## What is Health Check?

- Load balancer periodically checks backend health

---

## How it Works

- Sends request (HTTP/TCP) to backend
- Checks response:
  - Success → healthy
  - Failure → unhealthy

---

## Behavior

- Healthy instance → receives traffic  
- Unhealthy instance → removed from rotation  

---

## Example

- Endpoint: /health
- If returns 200 → healthy  
- If fails → traffic stopped  

---

## Important Points

- Prevents sending traffic to failed instances  
- Automatically recovers when instance becomes healthy  

---

## One-Line Interview Answer

"Health checks ensure that only healthy backend instances receive traffic by continuously monitoring their status."

---

# Common Mistakes

- Not explaining full traffic flow  
- Confusing ALB and NLB layers  
- Ignoring health check importance  
- Not mentioning target groups  

---

# Final Tip

Always explain in order:
1. How request comes (User → DNS → ALB)  
2. How routing happens (Listener → Target Group)  
3. Where app runs (Backend)  
4. How failures handled (Health Checks)  

