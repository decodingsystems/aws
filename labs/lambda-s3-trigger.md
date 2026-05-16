# Lab 03: Lambda Triggered by S3 Upload

## Objective
Build an event-driven pipeline: whenever a file is uploaded to S3, Lambda automatically processes it and logs the result to CloudWatch.

## Steps

### Part 1: Create the S3 Bucket
1. S3 → Create bucket → Name: `lambda-trigger-demo-<unique>` → Region: `us-east-1` → Create.

### Part 2: Create the Lambda Function
1. Lambda → Create function → Author from scratch
   - Name: `S3FileProcessor`
   - Runtime: Python 3.12
   - Execution role: **Create new role with basic Lambda permissions**
2. In the code editor, paste:
   ```python
   import json
   import urllib.parse

   def lambda_handler(event, context):
       # Extract S3 event details
       bucket = event['Records'][0]['s3']['bucket']['name']
       key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'])
       size = event['Records'][0]['s3']['object']['size']
       
       print(f"[NEW FILE] Bucket: {bucket}")
       print(f"[NEW FILE] Key: {key}")
       print(f"[NEW FILE] Size: {size} bytes")
       
       return {
           'statusCode': 200,
           'body': json.dumps(f'Processed: {key}')
       }
   ```
3. Click **Deploy**.

### Part 3: Add S3 Permissions to Lambda Role
1. Lambda → Configuration → Permissions → Click the execution role name.
2. IAM opens → Add permissions → Attach policy → `AmazonS3ReadOnlyAccess` → Add.

### Part 4: Create S3 Trigger
1. Back in Lambda → **Add trigger** → S3
   - Bucket: your bucket
   - Event type: **PUT** (All object create events)
   - Click Add.

### Part 5: Test It
1. Go to S3 → Upload any file (a text file, image, etc.) to your bucket.
2. Lambda → Monitor → **View CloudWatch Logs**
3. Click the latest log stream → See the filename, bucket, and size logged!

## Cleanup
Delete Lambda function → Remove S3 event notification from bucket → Delete S3 bucket.
