# AWS Networking & Content Delivery

## Introduction to Cloud Networking
When you deploy resources on AWS, each service doesn't just exist in isolation—it communicates with other services, users, and the Internet. AWS networking services let you define your own isolated network topology in the cloud, control traffic routing, protect resources behind firewalls, and distribute content globally with low latency. Understanding AWS networking is foundational to building secure and scalable architectures.

---

## Amazon VPC (Virtual Private Cloud)
Amazon VPC is the networking layer of AWS. A VPC is your own **logically isolated section of the AWS Cloud** where you can launch resources in a virtual network you define—specifying the IP address range, subnets, routing, and security.

### Key Concepts
- **CIDR Block:** The IP address range of your VPC (e.g., `10.0.0.0/16` = 65,536 IP addresses).
- **Subnets:** Subdivisions of your VPC within a single Availability Zone.
  - **Public Subnet:** Traffic can flow to/from the Internet via an Internet Gateway.
  - **Private Subnet:** No direct Internet access. Resources communicate out via a NAT Gateway.
- **Internet Gateway (IGW):** Allows resources in public subnets to reach the Internet.
- **NAT Gateway:** Allows resources in private subnets to initiate outbound Internet connections (e.g., for software updates), without allowing inbound connections.
- **Route Tables:** Rules determining where network traffic is directed. Each subnet is associated with one.
- **Security Groups:** Stateful, instance-level virtual firewalls. Rules apply to inbound and outbound traffic. "Stateful" means if you allow inbound traffic, the response is automatically allowed outbound.
- **Network ACLs:** Stateless, subnet-level firewalls. You must explicitly allow both inbound and outbound.
- **VPC Peering:** Private connection between two VPCs without going over the Internet.
- **AWS PrivateLink:** Private connectivity to AWS services without Internet exposure.
- **Official Docs:** [Amazon VPC Documentation](https://docs.aws.amazon.com/vpc/)

---

## Amazon Route 53
Route 53 is AWS's highly available and scalable DNS service. It routes end users to your applications by translating domain names to IP addresses (or AWS resource names). It also provides **health checks** to detect failures and route traffic away from unhealthy endpoints.

### Routing Policies
- **Simple:** One record, one resource.
- **Weighted:** Split traffic by percentage (e.g., 90% to v1, 10% to v2 for A/B testing).
- **Latency-Based:** Route users to the region with the lowest latency.
- **Failover:** Route to a backup resource when primary health check fails.
- **Geolocation:** Route based on where the DNS query originates.
- **Official Docs:** [Amazon Route 53 Documentation](https://docs.aws.amazon.com/route53/)

---

## Amazon CloudFront
CloudFront is AWS's global **Content Delivery Network (CDN)**. It caches content at over 400+ **Edge Locations** worldwide, serving it from the location closest to the end user.

### How it works
1. User requests `https://cdn.example.com/image.jpg`.
2. The request hits the nearest CloudFront Edge Location.
3. If the Edge has the file cached (Cache Hit): It serves it immediately (<1ms).
4. If not (Cache Miss): The Edge fetches from the **Origin** (S3 bucket, ALB, or custom HTTP server), caches it, and serves it.

- **Official Docs:** [Amazon CloudFront Documentation](https://docs.aws.amazon.com/cloudfront/)

---

## Hands-on Lab: Secure a VPC with Security Groups and NACLs

### Objective
Build a basic VPC with public/private subnets and implement two layers of network security: Security Groups and NACLs.

### Steps
1. **Build the VPC:** In the VPC console, click "Create VPC" → Select "VPC and more":
   - CIDR: `10.0.0.0/16`.
   - Select 2 AZs, 2 public subnets, 2 private subnets.
   - Enable NAT Gateway (1 per AZ).
   - AWS will automatically create the route tables and Internet Gateway. Click Create.
2. **Create a Security Group for a Web Server:**
   - VPC Console → Security Groups → Create.
   - Name: `WebServer-SG`, VPC: your new VPC.
   - Inbound rules:
     - Type: HTTP, Port: 80, Source: `0.0.0.0/0`
     - Type: HTTPS, Port: 443, Source: `0.0.0.0/0`
     - Type: SSH, Port: 22, Source: **My IP** (restrict SSH to your IP only!)
3. **Create a NACL for the public subnet:**
   - VPC Console → Network ACLs → Create.
   - Inbound Rule #100: Allow Port 80 from `0.0.0.0/0`.
   - Inbound Rule #110: Allow Port 443 from `0.0.0.0/0`.
   - Inbound Rule #120: Allow Port 1024-65535 (Ephemeral ports for return traffic) from `0.0.0.0/0`.
   - Inbound Rule *: Deny all.
   - Repeat for Outbound rules. Associate NACL with your public subnets.
4. **Verify:** Launch an EC2 in the public subnet with `WebServer-SG`. You can SSH to it, but port 8080 (not in rules) is blocked.

### Cleanup
Delete EC2, NAT Gateway (charges apply), then delete the VPC and all associated resources.
