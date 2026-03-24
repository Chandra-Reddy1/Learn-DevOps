# AWS VPC Interview Questions & Answers (Top 20)

---

## 1. What is a VPC?

A Virtual Private Cloud (VPC) is a logically isolated network in AWS where you can launch resources like EC2, RDS, etc.

---

## 2. What is CIDR in VPC?

CIDR (Classless Inter-Domain Routing) defines the IP address range of the VPC (e.g., 10.0.0.0/16).

---

## 3. What is a subnet?

A subnet is a subdivision of a VPC’s IP range.

Types:
- Public subnet
- Private subnet

---

## 4. Difference between public and private subnet?

- Public subnet → has route to Internet Gateway
- Private subnet → no direct internet access

---

## 5. What is an Internet Gateway (IGW)?

A component that allows communication between VPC and the internet.

---

## 6. What is a route table?

Defines how traffic is routed within the VPC.

---

## 7. What is NAT Gateway?

Allows private subnet resources to access the internet (outbound only).

---

## 8. What is Security Group?

- Acts as a firewall at instance level
- Stateful (response traffic automatically allowed)

---

## 9. What is Network ACL (NACL)?

- Firewall at subnet level
- Stateless (rules must be defined for both directions)

---

## 10. Difference between Security Group and NACL?

- SG → stateful, instance level
- NACL → stateless, subnet level

---

## 11. What is VPC Peering?

Connects two VPCs privately.

---

## 12. What is VPC Endpoint?

Private connection to AWS services without internet.

Types:
- Gateway Endpoint (S3, DynamoDB)
- Interface Endpoint (PrivateLink)

---

## 13. What is Elastic IP?

Static public IP address for EC2.

---

## 14. What is Bastion Host?

A server in public subnet used to access private instances.

---

## 15. What is flow log?

Captures IP traffic information in VPC for monitoring/debugging.

---

## 16. What is VPC Flow Logs used for?

- Troubleshooting network issues
- Monitoring traffic
- Security analysis

---

## 17. What is Transit Gateway?

Connects multiple VPCs and on-prem networks centrally.

---

## 18. What is DHCP option set?

Defines domain name servers and settings for VPC.

---

## 19. What is DNS resolution in VPC?

Allows instances to resolve domain names to IP addresses.

---

## 20. What is default VPC?

Pre-configured VPC created by AWS with public subnets.

---

# Bonus (High-Impact Interview Questions)

---

## 21. How does internet access work in VPC?

Flow:
EC2 → Route Table → Internet Gateway → Internet

---

## 22. How does private subnet access internet?

Flow:
Private EC2 → NAT Gateway → Internet Gateway → Internet

---

## 23. How to secure VPC?

- Use private subnets
- Restrict Security Groups
- Use NACLs
- Enable flow logs
- Use VPC endpoints

---

## 24. Difference between NAT Gateway and NAT Instance?

- NAT Gateway → managed, scalable
- NAT Instance → manual, less reliable

---

## 25. Can private subnet access S3 without internet?

Yes, using VPC Endpoint (Gateway endpoint for S3).

---

## 26. What happens if route table is misconfigured?

Traffic will fail (no connectivity).

---

## 27. How do you design highly available VPC?

- Use multiple AZs
- Separate subnets per AZ
- Use load balancer

---

## 28. What is AWS PrivateLink?

Provides private connectivity to services via interface endpoints.

---

## 29. What is subnet CIDR planning?

Dividing VPC CIDR into smaller ranges for proper resource allocation.

---

## 30. What is egress-only Internet Gateway?

Used for IPv6 outbound internet access only.

---

# Final Interview Tips

- Always explain traffic flow (very important)
- Use real examples:
  - Public subnet → load balancer
  - Private subnet → application servers
  - NAT → outbound access
- Know difference:
  - SG vs NACL
  - Public vs Private subnet
  - NAT vs IGW
