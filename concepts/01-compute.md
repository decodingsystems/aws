# AWS Compute Services

## Introduction to Cloud Compute
Cloud compute refers to the on-demand delivery of computing power—virtual servers, containers, or serverless functions—over the Internet, managed by a cloud provider. Instead of procuring and maintaining physical servers in a data center, you rent compute capacity from AWS, paying only for what you use. AWS compute services span a wide spectrum, from full virtual machines to completely managed serverless runtimes.

---

## Amazon EC2 (Elastic Compute Cloud)
**Amazon EC2** is the foundation of AWS compute. It provides resizable, on-demand virtual machines called **instances**. You have full administrative control over the OS, networking, and software stack.

### Key Concepts
- **AMI (Amazon Machine Image):** A template containing the OS and pre-installed software. The blueprint from which instances are launched.
- **Instance Types:** Different combinations of CPU, RAM, storage, and network capacity optimized for different workloads:
  - `t3.micro` – General purpose, burstable (Free-Tier eligible).
  - `c6i.large` – Compute optimized (ML inference, HPC).
  - `r6g.xlarge` – Memory optimized (in-memory databases).
- **Security Groups:** Stateful virtual firewall rules controlling inbound/outbound traffic.
- **Elastic IP:** A static public IPv4 address you can attach to any instance.
- **Key Pairs:** RSA key pairs used for SSH access. AWS stores the public key; you keep the private key.
- **Pricing Models:** On-Demand (pay-per-hour), Reserved (1-3 year commitment, 72% savings), Spot (bid for spare capacity, up to 90% savings, can be interrupted).
- **Official Docs:** [Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2/)

---

## AWS Lambda
**AWS Lambda** is AWS's serverless compute service. You upload code (a function), define a trigger, and Lambda executes it in a fully managed, auto-scaling environment. You are billed only for the milliseconds your code runs—there are no idle costs.

### Key Concepts
- **Event-Driven:** Triggers include API Gateway, S3 events, DynamoDB Streams, SQS queues, EventBridge schedules.
- **Runtimes:** Node.js, Python, Java, Go, Ruby, .NET are all supported.
- **Execution Limits:** Max 15-minute duration, 10GB memory, 512MB–10GB ephemeral storage.
- **Cold Starts:** First invocation may be slow as the runtime environment is initialized. Mitigated by Provisioned Concurrency.
- **Official Docs:** [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)

---

## Amazon ECS (Elastic Container Service)
**Amazon ECS** is a fully managed container orchestration service for Docker containers. It runs containers in a cluster, managing placement, scaling, and health checking.
- **Fargate:** A serverless compute engine for ECS. No EC2 servers to manage at all—you define CPU/RAM per task and AWS handles the rest.
- **Official Docs:** [Amazon ECS Documentation](https://docs.aws.amazon.com/ecs/)

## Amazon EKS (Elastic Kubernetes Service)
Managed Kubernetes as a service. AWS manages the Kubernetes control plane; you manage the worker nodes or use Fargate for fully serverless pods.
- **Official Docs:** [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/)

---

## Hands-on Lab: Deploy a Python Lambda Function

### Objective
Write, deploy, and test an AWS Lambda function triggered by a custom event.

### Steps
1. **Navigate to Lambda:** In the AWS Console, search for "Lambda" → Click "Create function".
2. **Configure:**
   - Select "Author from scratch".
   - Function name: `HelloWorldFunction`.
   - Runtime: `Python 3.12`.
   - Leave execution role as "Create a new role with basic Lambda permissions".
3. **Write the function:** In the inline code editor, replace the default code:
   ```python
   import json

   def lambda_handler(event, context):
       name = event.get("name", "World")
       return {
           "statusCode": 200,
           "body": json.dumps(f"Hello, {name}! From Lambda.")
       }
   ```
4. **Deploy:** Click the "Deploy" button.
5. **Test:**
   - Click "Test" → Configure test event.
   - Name it `TestEvent`, use this JSON: `{"name": "Abhijeet"}`
   - Click Save, then run the test.
6. **Observe:** Check the execution result for the response body and the "Duration" (billed milliseconds).
7. **View Logs:** Click "Monitor" → "View CloudWatch Logs" to see invocation logs.

### Cleanup
Go to the Lambda function page → Actions → Delete function.
