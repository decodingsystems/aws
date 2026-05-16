# Lab 12: EC2 Instance Profile — IAM Role Access Without Keys

## Objective
Attach an IAM role to an EC2 instance so it can access AWS services (S3) without storing any access keys in the application code.

## Steps

### Part 1: Create IAM Role for EC2
1. IAM → Roles → Create role
   - Trusted entity: **AWS service**
   - Use case: **EC2**
   - Next: Permissions → search and attach `AmazonS3ReadOnlyAccess`
   - Role name: `EC2-S3ReadOnly-Role`
   - Create role.

### Part 2: Launch EC2 with the IAM Role
1. EC2 → Launch instance
2. In **Advanced details → IAM instance profile** → select `EC2-S3ReadOnly-Role`
3. Launch the instance.

### Part 3: Test S3 Access Without Keys
SSH into the instance:
```bash
# No aws configure needed! Role credentials are automatically provided.
aws sts get-caller-identity
# Shows: Account, UserId, Arn  (the role!)

# List all S3 buckets in the account
aws s3 ls

# Download a file from S3
aws s3 cp s3://your-bucket-name/somefile.txt /tmp/

# Try a write operation (should fail — ReadOnly role!)
aws s3 cp /tmp/test.txt s3://your-bucket-name/
# Access Denied — correct! Role only allows reading.
```

### Part 4: How Instance Profiles Work Internally
The temporary credentials are served from the **EC2 Instance Metadata Service**:
```bash
# AWS SDKs and CLI automatically call this URL to get credentials
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/EC2-S3ReadOnly-Role
# Returns: AccessKeyId, SecretAccessKey, Token (rotate every ~1 hour automatically)
```
This is why you never hard-code credentials into EC2 application code.

### Part 5: Attach a Role to a Running Instance
You can also attach/replace roles on already-running instances:
```bash
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxxxxxxxx \
  --iam-instance-profile Name=EC2-S3ReadOnly-Role
```

## Cleanup
Terminate EC2. Delete IAM role.
