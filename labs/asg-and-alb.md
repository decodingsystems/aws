# Lab 02: Auto Scaling Group with Application Load Balancer

## Objective
Create a highly available, auto-scaling web tier — the backbone of any production AWS architecture.

## Prerequisites
- A VPC with at least 2 public subnets in different AZs.
- A Security Group allowing HTTP (80) inbound.

## Part 1: Create a Launch Template
1. EC2 → **Launch Templates** → Create Launch Template
   - Name: `web-server-lt`
   - AMI: Amazon Linux 2023
   - Instance type: `t2.micro`
   - Security group: Allow HTTP (80) and SSH (22)
   - **Advanced → User Data:**
     ```bash
     #!/bin/bash
     yum install httpd -y && systemctl start httpd && systemctl enable httpd
     echo "<h1>Response from $(hostname -f)</h1>" > /var/www/html/index.html
     ```
   - Click Create.

## Part 2: Create Application Load Balancer
1. EC2 → **Load Balancers** → Create → Application Load Balancer
   - Name: `web-alb`
   - Scheme: Internet-facing
   - VPC + select **2 different AZ subnets**
   - Security group: Allow HTTP (80) from `0.0.0.0/0`
2. **Target Group:**
   - Name: `web-tg`, Type: Instances, Port: 80
   - Health check: Path `/`, Healthy threshold: 2
3. Listener: HTTP:80 → Forward to `web-tg`
4. Create Load Balancer.

## Part 3: Create the Auto Scaling Group
1. EC2 → **Auto Scaling Groups** → Create
   - Name: `web-asg`
   - Launch Template: `web-server-lt`
   - VPC + both subnets
   - **Attach to Load Balancer**: Choose `web-tg`
   - Health check type: **ELB**
   - Desired: 2, Minimum: 1, Maximum: 4
2. **Scaling Policy**: Target Tracking → CPU Utilization → target 50%
3. Create.

## Verify
- Wait 2-3 mins. Check EC2 console → 2 instances running.
- Copy the ALB DNS name from Load Balancers list.
- Refresh `http://<ALB-DNS>` multiple times → notice different hostnames (different instances!).
- Terminate one instance manually → ASG automatically replaces it!

## Cleanup
Delete ASG → Delete ALB → Delete Target Group → Terminate remaining instances.
