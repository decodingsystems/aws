# AWS Security, Identity, & Compliance

## Introduction to Cloud Security
Security in AWS follows the **Shared Responsibility Model**: AWS is responsible for the security *of* the cloud (the underlying hardware, software, networking, and facilities), while you are responsible for security *in* the cloud (your data, applications, configuration, and access control). Understanding this distinction is the starting point for all cloud security decisions. A well-secured AWS environment combines identity management, data encryption, network protection, and continuous monitoring.

---

## AWS IAM (Identity and Access Management)
IAM is the core service for controlling **who** can do **what** in your AWS account.

### Key Concepts
- **Root User:** The account owner with absolute privileges. Should only be used for initial setup and billing. Enable MFA immediately.
- **IAM Users:** Long-term identities for individual people or applications. Avoid using root user credentials in code.
- **IAM Groups:** Collections of users sharing the same permissions (e.g., a "Developers" group with EC2 and S3 read access).
- **IAM Roles:** Temporary identities assumed by AWS services (e.g., Lambda needing to write to DynamoDB), EC2 instances, or federated users. No username/password—uses short-lived tokens.
- **IAM Policies:** JSON documents defining permissions using `Effect` (Allow/Deny), `Action` (which API calls), and `Resource` (which ARN). Attached to users, groups, or roles.
- **Principle of Least Privilege:** Grant only the permissions needed to do the job—nothing more.
- **Official Docs:** [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)

### Policy Example
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:ListBucket"],
    "Resource": "arn:aws:s3:::my-data-bucket/*"
  }]
}
```

---

## AWS KMS (Key Management Service)
AWS KMS is a managed service for creating and controlling **encryption keys** used to protect data.

### Key Concepts
- **Customer Managed Keys (CMKs):** Encryption keys you create and control, fully managed in KMS. You can rotate, enable/disable, and add policies.
- **AWS Managed Keys:** Automatically created by AWS services like S3, RDS, EBS. Less control.
- **Envelope Encryption:** KMS encrypts a small **Data Encryption Key (DEK)**. The DEK encrypts your actual data. Only the wrapped DEK is stored alongside the data.
- **Integration:** KMS integrates natively with S3, RDS, EBS, Lambda, Secrets Manager, and many more.
- **Official Docs:** [AWS KMS Documentation](https://docs.aws.amazon.com/kms/)

---

## AWS WAF (Web Application Firewall)
AWS WAF protects your web applications from common web exploits like SQL injection, cross-site scripting (XSS), and bot traffic. It runs at Layer 7 (the Application layer) and integrates with CloudFront, ALB, and API Gateway.

### Key Concepts
- **Web ACLs:** Sets of rules applied to incoming requests. Rules run in order by priority.
- **Rule Types:** IP set rules (block/allow specific IPs), Rate-based rules (throttle IPs sending too many requests), Managed Rule Groups (pre-built rules from AWS or AWS Marketplace security partners).

---

## Hands-on Lab: Create a Least-Privilege IAM Policy

### Objective
Create a custom IAM policy granting only S3 read access to a specific bucket, and verify the restriction works.

### Steps
1. **Create a Custom Policy:**
   - IAM Console → Policies → Create policy.
   - Switch to JSON editor and paste:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [{
         "Effect": "Allow",
         "Action": ["s3:GetObject", "s3:ListBucket"],
         "Resource": [
           "arn:aws:s3:::YOUR_BUCKET_NAME",
           "arn:aws:s3:::YOUR_BUCKET_NAME/*"
         ]
       }]
     }
     ```
   - Replace `YOUR_BUCKET_NAME`. Click Next, name it `S3ReadOnly-MyBucket`, click Create.

2. **Create an IAM User and Attach the Policy:**
   - IAM → Users → Create user.
   - Username: `s3-readonly-user`. Enable Console access.
   - On permissions, choose "Attach policies directly" → attach `S3ReadOnly-MyBucket`. Create user.

3. **Test as the Restricted User:**
   - Copy the console sign-in URL and log in as `s3-readonly-user` (incognito browser).
   - Navigate to S3 → open your target bucket. You can see and download objects ✅.
   - Try to upload a file → `Access Denied` ✅.
   - Navigate to EC2 → `You are not authorized to perform this operation` ✅.

### Cleanup
Log back in as Admin → IAM → Delete `s3-readonly-user` → Delete `S3ReadOnly-MyBucket` policy.
