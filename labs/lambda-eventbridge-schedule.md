# Lab 04: Scheduled Lambda with EventBridge

## Objective
Create a Lambda function that runs automatically on a schedule (like a cron job), using Amazon EventBridge as the scheduler.

## Steps

### Part 1: Create Lambda Function
1. Lambda → Create function → Author from scratch
   - Name: `ScheduledHealthCheck`
   - Runtime: Python 3.12
2. Paste code:
   ```python
   import datetime
   import json

   def lambda_handler(event, context):
       now = datetime.datetime.utcnow().strftime("%Y-%m-%d %H:%M:%S UTC")
       print(f"[HEALTH CHECK] Ran at: {now}")
       print(f"[HEALTH CHECK] Event source: {event.get('source', 'manual')}")
       
       # Simulate checking service health
       services = {"api": "healthy", "db": "healthy", "cache": "healthy"}
       for svc, status in services.items():
           print(f"  - {svc}: {status}")
       
       return {"statusCode": 200, "body": json.dumps({"status": "all_healthy", "timestamp": now})}
   ```
3. Click Deploy.

### Part 2: Create EventBridge Schedule (every 5 minutes)
1. Lambda → Add trigger → **EventBridge (CloudWatch Events)**
   - Rule: Create a new rule
   - Rule name: `every-5-minutes`
   - Rule type: **Schedule expression**
   - Schedule: `rate(5 minutes)`
   - Click Add.

### Part 3: Verify Execution
1. Wait 5 minutes.
2. Lambda → Monitor → View CloudWatch Logs → Latest log stream.
3. You'll see automatic invocations every 5 minutes.

### Bonus: Cron Expression
For more control, use a cron expression instead of rate:
```
# Runs at 9:00 AM UTC every Monday-Friday
cron(0 9 ? * MON-FRI *)

# Runs at midnight on the 1st of every month
cron(0 0 1 * ? *)
```

## Cleanup
Lambda → Configuration → Triggers → Disable and delete the EventBridge rule → Delete function.
