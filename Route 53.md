# Top 70 AWS Route 53 Interview Questions

## Basic / Fundamentals

### 1. What is Amazon Route 53?
Route 53 is AWS's highly available and scalable DNS and domain-name service. It provides DNS resolution, domain registration, health checks, and traffic-routing capabilities.

### 2. Why is it called Route 53?
The name refers to DNS, which traditionally uses port 53.

### 3. What are the main functions of Route 53?
- Domain registration
- DNS hosting and resolution
- Health checks
- Traffic routing
- Failover and availability management

### 4. What is DNS?
DNS (Domain Name System) translates human-readable domain names such as `www.example.com` into IP addresses or other DNS records.

### 5. What is a hosted zone?
A hosted zone is a container for DNS records for a domain. Route 53 supports public and private hosted zones.

### 6. What is a public hosted zone?
A public hosted zone contains DNS records that can be resolved from the public internet.

### 7. What is a private hosted zone?
A private hosted zone contains DNS records that are resolvable only within associated VPCs.

### 8. What is the difference between a public and private hosted zone?
A public hosted zone serves public DNS resolution. A private hosted zone is associated with VPCs and is intended for internal DNS resolution.

### 9. What is a DNS record?
A DNS record defines how a DNS name should resolve. Examples include A, AAAA, CNAME, MX, NS, TXT, and Alias records.

### 10. What is an A record?
An A record maps a hostname to an IPv4 address.

### 11. What is an AAAA record?
An AAAA record maps a hostname to an IPv6 address.

### 12. What is a CNAME record?
A CNAME maps one DNS name to another DNS name.

Example:

```text
app.example.com -> myapp.example.net
```

### 13. What is an MX record?
An MX record identifies mail servers responsible for receiving email for a domain.

### 14. What is a TXT record?
TXT records store text data and are commonly used for domain verification, SPF-related information, and other application-specific purposes.

### 15. What is an NS record?
An NS record identifies the authoritative name servers for a DNS zone.

### 16. What is a TTL?
TTL (Time To Live) specifies how long DNS resolvers may cache a DNS response before querying again.

### 17. What happens when TTL is set to a low value?
DNS changes can generally propagate through caches more quickly, but resolvers may need to query the authoritative DNS service more frequently.

### 18. What happens when TTL is set to a high value?
Records may remain cached longer, reducing DNS query frequency but potentially making changes take longer to be observed by clients.

### 19. What are Route 53 name servers?
They are authoritative DNS servers assigned to a hosted zone. They answer DNS queries for that zone.

### 20. What is an authoritative DNS server?
It is the DNS server that contains the authoritative records for a domain or zone and provides the definitive answer for those records.

---

## Routing Policies

### 21. What are Route 53 routing policies?
Routing policies determine how Route 53 responds to DNS queries. Common policies include Simple, Weighted, Latency-based, Failover, Geolocation, Geoproximity, and Multivalue Answer.

### 22. What is Simple routing?
Simple routing returns a single resource or a set of values without sophisticated traffic distribution logic.

### 23. What is Weighted routing?
Weighted routing distributes DNS traffic between resources according to assigned weights.

Example:

```text
Application A -> 90
Application B -> 10
```

This can be useful for gradual deployments.

### 24. What is a common use case for weighted routing?
Blue/green deployments, canary releases, and controlled traffic distribution.

### 25. What is latency-based routing?
Latency-based routing directs users to the AWS region that Route 53 determines can provide the lowest latency.

### 26. When would you use latency-based routing?
When the application is deployed in multiple regions and you want users routed based on network latency.

### 27. What is failover routing?
Failover routing sends traffic to a primary resource while it is healthy and directs traffic to a secondary resource when the primary is considered unhealthy.

### 28. What is an active-passive architecture using Route 53?
The primary environment serves traffic normally, while a secondary environment is available for failover.

### 29. What is an active-active architecture?
Multiple environments actively serve traffic, often using latency-based, weighted, or other routing policies.

### 30. What is geolocation routing?
Geolocation routing routes users based on the geographic location associated with their DNS query.

### 31. What is geoproximity routing?
Geoproximity routing routes traffic based on the geographic location of resources and users. It can also use bias to expand or shrink the geographic area assigned to a resource.

### 32. What is multivalue answer routing?
It allows Route 53 to return multiple healthy records and can be used to improve availability.

### 33. Can Route 53 perform load balancing?
Route 53 can distribute DNS responses using routing policies, but it is not a replacement for a Layer 4/Layer 7 load balancer such as NLB or ALB.

### 34. What is the difference between Route 53 and an AWS Load Balancer?
Route 53 operates at the DNS layer. ALB/NLB operate at the load-balancing layer and distribute network/application traffic to targets.

### 35. Can Route 53 route traffic directly to an EC2 instance?
Yes. For example, an A record can point to an EC2 public IPv4 address, although using a stable DNS target or load balancer is often preferable for production architectures.

---

## Alias, CNAME, and DNS Records

### 36. What is an Alias record in Route 53?
An Alias record is an AWS-specific DNS record capability that can point a name to supported AWS resources such as ALB, NLB, CloudFront distributions, API Gateway, and certain other AWS endpoints.

### 37. Alias vs CNAME?
CNAME points to another DNS name. Route 53 Alias records can point to supported AWS resources and can be used at the zone apex, where a CNAME cannot normally be used.

### 38. Can a CNAME be used for the root domain?
A standard DNS CNAME cannot normally be used at the zone apex. Route 53 Alias records can be used for supported AWS targets at the apex.

### 39. Can an Alias record point to an ALB?
Yes.

Example:

```text
www.example.com
       |
       v
   Alias Record
       |
       v
   Application Load Balancer
```

### 40. Can an Alias record point to CloudFront?
Yes.

### 41. Can an Alias record point to API Gateway?
Yes, for supported API Gateway endpoints.

### 42. What is the advantage of an Alias record over CNAME for AWS resources?
It provides native integration with supported AWS resources and can be used at the zone apex.

### 43. What happens if you delete a DNS record?
DNS resolution for that name stops according to the behavior of remaining records and cached responses. Existing cached responses may continue until their TTL expires.

---

## Health Checks and Failover

### 44. What is a Route 53 health check?
A Route 53 health check monitors the health of an endpoint or evaluates the status of other AWS resources through supported mechanisms.

### 45. Why are health checks useful?
They can be used with routing policies such as failover to help Route 53 avoid routing traffic to unhealthy resources.

### 46. Can Route 53 health checks monitor HTTP endpoints?
Yes. Route 53 supports health checks for supported HTTP, HTTPS, and TCP endpoints.

### 47. Can Route 53 health checks check a specific path?
Yes. HTTP/HTTPS health checks can check a specified endpoint/path.

Example:

```text
https://app.example.com/health
```

### 48. What happens when the primary endpoint fails a health check?
With an appropriate failover configuration, Route 53 can stop returning the primary resource and return the secondary resource.

### 49. Can Route 53 monitor resources inside a private VPC directly using standard endpoint health checks?
Standard Route 53 health checkers operate from outside your VPC, so private endpoints require an architecture appropriate for private health evaluation, such as calculated health checks or CloudWatch-alarm-based mechanisms where supported.

### 50. What is a calculated health check?
It combines the health status of multiple child health checks using configured logic.

---

## Private DNS and VPC

### 51. What is Route 53 private DNS?
Private DNS allows you to create DNS names that resolve inside associated VPCs without making the records publicly resolvable.

### 52. How do you create internal DNS for an application?
Create a private hosted zone, associate it with the required VPC(s), and create records pointing to internal resources.

Example:

```text
argocd.internal.example.com
             |
             v
       Internal ALB
```

### 53. Can one private hosted zone be associated with multiple VPCs?
Yes, subject to Route 53 association requirements and account/region considerations.

### 54. Can private hosted zones work across AWS accounts?
Yes, with the appropriate cross-account VPC association process and permissions.

### 55. Can Route 53 private hosted zones be used for EKS?
Yes. They can be used for internal application DNS names and other private AWS architectures.

### 56. How would you expose an internal Argo CD URL?
A common architecture is:

```text
User on corporate network/VPN
          |
          v
argocd.internal.example.com
          |
          v
Private Route 53 Hosted Zone
          |
          v
Internal ALB
          |
          v
Argo CD Service
```

---

## EKS / DevOps Scenarios

### 57. How would you route a domain to an ALB in EKS?
Use an appropriate Kubernetes ingress/load-balancer integration to create or reference the ALB, then create a Route 53 Alias record pointing the application hostname to the ALB.

### 58. What is the DNS flow for an EKS application?
A simplified flow is:

```text
User
  |
  v
Route 53
  |
  v
ALB
  |
  v
Kubernetes Service
  |
  v
Pod
```

### 59. How would you make an EKS application internal-only?
Use an internal load balancer, private subnets/network controls, private DNS where appropriate, and restrict access through the corporate network/VPN/security controls.

### 60. How would you create a Route 53 record using Terraform?
Example:

```hcl
resource "aws_route53_record" "app" {
  zone_id = var.zone_id
  name    = "app.example.com"
  type    = "A"

  alias {
    name                   = var.alb_dns_name
    zone_id                = var.alb_zone_id
    evaluate_target_health = true
  }
}
```

### 61. How would you create a private hosted zone using Terraform?

```hcl
resource "aws_route53_zone" "private" {
  name = "internal.example.com"

  vpc {
    vpc_id = var.vpc_id
  }
}
```

### 62. How would you troubleshoot a Route 53 record that is not resolving?
Check:
1. Record name
2. Record type
3. Hosted zone
4. Name-server delegation
5. TTL/cache
6. VPC association for private zones
7. Resolver/network connectivity
8. Target resource health
9. Security/network configuration
10. DNS tools such as `dig` or `nslookup`

### 63. What commands can you use to troubleshoot DNS?

```bash
nslookup app.example.com
```

or:

```bash
dig app.example.com
```

You can also inspect authoritative name servers with appropriate `dig` queries.

### 64. What is DNS propagation?
It is commonly used to describe the period during which DNS changes become visible through recursive resolver caches and other DNS infrastructure. TTLs influence caching duration.

### 65. Does Route 53 instantly propagate DNS changes everywhere?
Not necessarily. Recursive resolvers may cache previous responses until their TTL expires.

### 66. How would you implement blue/green deployment using Route 53?
You can use weighted routing to send traffic between blue and green environments, gradually changing the weights as you validate the new environment.

Example:

```text
Blue  -> 90%
Green -> 10%
```

Then gradually increase Green after validation.

### 67. How would you implement disaster recovery using Route 53?
A common approach is to deploy primary and secondary environments in different locations and use health checks with failover routing so DNS responses can move to the secondary environment when the primary is unhealthy.

### 68. How would you migrate a domain to a new application without downtime?
A common approach is to lower TTL ahead of the planned change, prepare and validate the new target, change the DNS record, monitor the migration, and restore an appropriate TTL afterward. Exact behavior depends on existing resolver caches.

### 69. What is split-horizon DNS?
Split-horizon DNS means returning different DNS answers depending on where the query originates. A common example is using public DNS records for external users and private hosted-zone records for internal users.

### 70. Explain a complete Route 53 production architecture.

A common example:

```text
                         Users
                           |
                           v
                    Route 53 Public DNS
                           |
                           v
                    CloudFront / ALB
                           |
                           v
                    Application Layer
                           |
                           v
                     EKS / EC2 / ECS
```

For an internal application:

```text
                  Corporate User
                        |
                    VPN / DX
                        |
                        v
              Route 53 Private DNS
                        |
                        v
                 Internal ALB
                        |
                        v
                EKS Kubernetes
                        |
                        v
                       Pods
```

The key interview concepts to understand are:

- Hosted zones
- Public vs private DNS
- A/AAAA/CNAME/TXT/MX/NS records
- Alias records
- TTL
- Routing policies
- Health checks
- Failover
- Weighted routing
- Latency routing
- Geolocation/geoproximity
- Private DNS
- Route 53 + ALB
- Route 53 + EKS
- Route 53 + Terraform
- DNS troubleshooting
