AWS Solution Architecture
A highly available, multi-AZ web application stack using AWS CloudFront, ALBs, EC2 Auto Scaling, NAT gateways, and a Multi-AZ RDS deployment.

📖 Table of Contents
Overview
Architecture Diagram
Component Breakdown
Amazon CloudFront
VPC, Subnets & Networking
Security Groups & Bastion Host
Auto Scaling Groups & Load Balancers
RDS Multi-AZ Database
Data Flow
High Availability & Fault Tolerance
Security Considerations
📌 Overview
This architecture delivers a scalable, secure web application by leveraging:

Amazon CloudFront for global content caching
Public and private subnets across two Availability Zones
Auto Scaling for both NGINX (front-end) and Node.js (application) tiers
Elastic Load Balancers (public and internal)
NAT Gateways for controlled outbound internet access
Multi-AZ Amazon RDS for relational data with automated failover
🖼️ Architecture Diagram
AWS Multi-AZ Web Architecture Figure 1: High-level diagram of the solution

🔍 Component Breakdown
Amazon CloudFront
Purpose: Caches static content at edge locations to reduce latency and origin load.
Origin: VPC Internet Gateway → Public ALB
VPC, Subnets & Networking
VPC CIDR: 10.0.0.0/16
AZ1
Public Subnet: 10.0.1.0/24 (NGINX + NAT)
Private Subnet: 10.0.2.0/24 (Node.js + internal ALB)
DB Subnet: 10.0.3.0/24 (RDS primary)
AZ2
Public Subnet: 10.0.4.0/24
Private Subnet: 10.0.5.0/24
DB Subnet: 10.0.6.0/24 (RDS standby)
NAT Gateways: one per AZ, in each public subnet, for controlled outbound internet.
Security Groups & Bastion Host
Bastion Host:
In public subnet, SSH-only access from your office/IP.
Security group allows inbound SSH (TCP 22) from your trusted IPs.
NGINX SG:
In public subnets; inbound HTTP (80) & HTTPS (443) from CloudFront/Internet Gateway.
Outbound to internal ALB.
Node.js SG:
In private subnets; inbound from internal ALB on app port (e.g. 3000).
Outbound to RDS.
Auto Scaling Groups & Load Balancers
Front-end Tier (NGINX) in Public Subnets
Auto Scaling Group:
Minimum instances: 2
Maximum instances: 6
Scaling policies based on ALB request count and CPU utilization.
Internet-facing ALB:
Distributes incoming user requests across NGINX instances in both AZs.
Health checks on HTTP/HTTPS.
App Tier (Node.js) in Private Subnets
Auto Scaling Group:
Minimum instances: 2
Maximum instances: 6
Scaling policies based on ALB request count and CPU utilization.
Internal ALB:
Receives traffic from NGINX tier.
Routes to healthy Node.js instances.
Health checks on the application’s /health endpoint.
Load Balancer Flow Diagram
Users
  ↓
CloudFront
  ↓
Internet-Facing ALB
  ↓
Auto Scaling Group (NGINX)
  ↓
Internal ALB
  ↓
Auto Scaling Group (Node.js)
  ↓
RDS Multi-AZ


RDS Multi-AZ Database
Engine: Amazon Aurora/MySQL/PostgreSQL (as required)
Deployment: Primary in AZ1, Standby in AZ2
Subnet Group: DB subnets in both AZs
Failover: Automatic multi-AZ failover on instance or AZ failure
🔄 Data Flow
Static content (e.g. JS/CSS/images) served via CloudFront edge caches.
Dynamic requests routed from CloudFront to public ALB.
NGINX instances handle SSL termination, caching, routing, and forward to internal ALB.
Node.js instances process business logic, read/write to RDS over secure private subnets.
Outbound traffic (e.g. OS updates, external API calls) flows through NAT gateways.
🛡️ High Availability & Fault Tolerance
Multi-AZ for every tier:
ALBs span AZs; ASGs launch across both AZs.
RDS in Multi-AZ mode.
Health Checks:
ALBs monitor instance health and redistribute traffic.
Auto Scaling Policies:
Scale-out on CPU or request metrics to handle spikes.
Scale-in to minimize cost when idle.
🔐 Security Considerations
Least-privilege Security Groups only permit required traffic.
Bastion Host for SSH—no public SSH on any EC2 except bastion.
IAM Roles & Policies assigned to EC2s for AWS API access (e.g. Secrets Manager).
Encryption at Rest & In Transit:
RDS uses KMS-managed keys.
ALBs enforce HTTPS for user connections.
Logging & Monitoring:
VPC Flow Logs, CloudWatch metrics, ALB access logs.
