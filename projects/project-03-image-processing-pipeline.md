# Project 3: Event-Driven Image Processing Pipeline

## Architecture
```
User uploads image to S3
        │  S3:PutObject event
        ▼
  AWS Lambda (Thumbnail Generator)
        │  writes thumbnail
        ▼
  S3 Output Bucket (thumbnails/)
        │  publishes message
        ▼
  Amazon SNS Topic
        │        │
        ▼        ▼
   Email Alert  SQS Queue (audit log)
```

## What You'll Build
A fully automated image processing pipeline: upload a photo → Lambda auto-generates a thumbnail → stores it in a different S3 folder → sends an email notification.

## Services Used
- **S3** (2 buckets: originals + processed)
- **Lambda** (Python, uses Pillow for image resizing)
- **SNS** (email alerts)
- **SQS** (audit log queue)
- **IAM** (Lambda execution role)

---

## Step 1: Create S3 Buckets
```bash
aws s3 mb s3://image-originals-<unique>
aws s3 mb s3://image-thumbnails-<unique>
```

## Step 2: Create SNS Topic + Email Subscription
```bash
aws sns create-topic --name image-processed-alerts
# Note the ARN
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:ACCOUNT:image-processed-alerts \
  --protocol email \
  --notification-endpoint your@email.com
# Confirm subscription in your email
```

## Step 3: Create SQS Queue
```bash
aws sqs create-queue --queue-name image-audit-queue
```

## Step 4: Lambda Function + Layer with Pillow
1. Lambda → Create function: `ImageThumbnailGenerator`, Python 3.12
2. Attach a **Lambda layer** with Pillow:
   - Use ARN from: https://github.com/keithrozario/Klayers (find `Pillow` for your region)
3. IAM Role for Lambda: attach
   - `AmazonS3FullAccess`
   - `AmazonSNSFullAccess`
   - `AmazonSQSFullAccess`
4. Code:
```python
import boto3, os
from PIL import Image
from io import BytesIO

s3 = boto3.client('s3')
sns = boto3.client('sns')
sqs = boto3.client('sqs')

THUMBNAIL_BUCKET = "image-thumbnails-<unique>"
SNS_TOPIC_ARN = "arn:aws:sns:us-east-1:ACCOUNT:image-processed-alerts"
SQS_URL = "https://sqs.us-east-1.amazonaws.com/ACCOUNT/image-audit-queue"

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Download original
    response = s3.get_object(Bucket=bucket, Key=key)
    img = Image.open(BytesIO(response['Body'].read()))
    
    # Resize to 200x200 thumbnail
    img.thumbnail((200, 200))
    buffer = BytesIO()
    img.save(buffer, format=img.format or 'JPEG')
    buffer.seek(0)
    
    # Upload thumbnail
    thumb_key = f"thumbnails/{key}"
    s3.put_object(Bucket=THUMBNAIL_BUCKET, Key=thumb_key, Body=buffer)
    print(f"Thumbnail created: s3://{THUMBNAIL_BUCKET}/{thumb_key}")
    
    # Send SNS notification
    sns.publish(
        TopicArn=SNS_TOPIC_ARN,
        Message=f"Thumbnail generated for {key}. Size: {img.size}",
        Subject="Image Processed"
    )
    
    # Audit log to SQS
    sqs.send_message(
        QueueUrl=SQS_URL,
        MessageBody=f'{{"original": "{key}", "thumbnail": "{thumb_key}"}}'
    )
    return {"statusCode": 200}
```

## Step 5: Add S3 Trigger
Lambda → Add trigger → S3 → originals bucket → Event: PUT

## Step 6: Test
```bash
aws s3 cp ./photo.jpg s3://image-originals-<unique>/photo.jpg
# Wait ~5 seconds
aws s3 ls s3://image-thumbnails-<unique>/thumbnails/
# Check your email for notification!
```

## Cleanup
Delete Lambda → Both S3 buckets → SNS topic → SQS queue.
