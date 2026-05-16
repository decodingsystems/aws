# Project 2: 3-Tier Highly Available Web Application

## Architecture
```
Internet
    │
    ▼
Application Load Balancer (Public)
    │
    ▼
Auto Scaling Group — EC2 Web Servers (Private Subnet)
    │
    ▼
Amazon RDS MySQL — Multi-AZ (Private Subnet)
```

## What You'll Build
A production-grade 3-tier architecture: a highly available web application with automatic scaling and a managed database with automatic failover.

## Services Used
- **VPC** with public (ALB) and private (EC2, RDS) subnets across 2 AZs
- **ALB** (Application Load Balancer)
- **Auto Scaling Group** of EC2 t2.micro instances
- **RDS MySQL** Multi-AZ
- **Secrets Manager** for DB credentials
- **CloudWatch** alarms + SNS alerts

---

## Step 1: Network Setup
1. Create VPC: `10.0.0.0/16`
2. **Public subnets** (for ALB): `10.0.1.0/24` (AZ-a), `10.0.2.0/24` (AZ-b)
3. **Private subnets** (for EC2): `10.0.3.0/24` (AZ-a), `10.0.4.0/24` (AZ-b)
4. **Database subnets** (for RDS): `10.0.5.0/24` (AZ-a), `10.0.6.0/24` (AZ-b)
5. Create IGW → attach to VPC → add to public subnet route tables
6. Create NAT Gateway in public subnet → add to private subnet route tables (EC2 needs outbound Internet for updates)

## Step 2: Security Groups
- **ALB-SG**: Inbound HTTP(80), HTTPS(443) from `0.0.0.0/0`
- **EC2-SG**: Inbound HTTP(80) from `ALB-SG` only. SSH from My IP.
- **RDS-SG**: Inbound MySQL(3306) from `EC2-SG` only.

## Step 3: RDS MySQL Multi-AZ
1. RDS → Create MySQL, `db.t3.micro`
2. **Multi-AZ deployment: Yes**
3. Place in DB subnets → use `RDS-SG`
4. Store password in **AWS Secrets Manager** → `prod/myapp/db`

## Step 4: Launch Template + User Data
```bash
#!/bin/bash
yum install httpd php php-mysqlnd -y
systemctl start httpd && systemctl enable httpd

# Retrieve DB credentials from Secrets Manager
SECRET=$(aws secretsmanager get-secret-value --secret-id prod/myapp/db --query SecretString --output text)
DB_HOST=$(echo $SECRET | python3 -c "import sys,json; print(json.load(sys.stdin)['host'])")
DB_PASS=$(echo $SECRET | python3 -c "import sys,json; print(json.load(sys.stdin)['password'])")

# Create a PHP info page
cat > /var/www/html/index.php << EOF
<?php
\$conn = new mysqli("$DB_HOST", "admin", "$DB_PASS", "appdb");
echo "<h1>Server: " . gethostname() . "</h1>";
echo "<p>DB Status: " . (\$conn->connect_error ? "Error" : "Connected ✅") . "</p>";
?>
EOF
```

## Step 5: Create ALB and Auto Scaling Group
- Follow Lab 02 steps to create ALB and ASG
- Attach ASG to the ALB target group
- Deploy into the **private subnets** (EC2 in private, ALB in public)
- Scaling policy: CPU > 70% → scale out

## Step 6: CloudWatch Dashboard
Create a dashboard with:
- ALB active connections count
- EC2 fleet CPU
- RDS FreeStorageSpace
- Set alarms with SNS email alerts for all critical metrics

## Testing
```bash
# Get ALB DNS
ALB_DNS="<your-alb-dns>.amazonaws.com"
curl http://$ALB_DNS
# Refresh multiple times — notice different hostnames (different AZs)
```

## Cleanup
Delete ASG → ALB → Target Group → RDS → Secrets → NAT Gateway → VPC.
