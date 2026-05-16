# Lab 11: CloudWatch Alarms and Auto-Remediation

## Objective
Set up a CloudWatch alarm on EC2 CPU utilization that sends an SNS email notification and automatically triggers a Lambda function to log remediation actions.

## Steps

### Part 1: Create SNS Topic for Notifications
1. SNS → Create topic → Type: Standard → Name: `high-cpu-alert`
2. Create subscription → Protocol: **Email** → Enter your email → Create.
3. **Check your email and confirm the subscription!**

### Part 2: Create CloudWatch Alarm
1. CloudWatch → **Alarms** → Create alarm
2. Select metric → EC2 → Per-Instance Metrics → Find your running instance → **CPUUtilization** → Select metric
3. Configure:
   - Period: 1 minute
   - Threshold: **Greater than 70**%
   - Datapoints: 2 out of 3
4. **Action when ALARM:**
   - Send notification to SNS topic: `high-cpu-alert`
5. Alarm name: `EC2-HighCPU-Alarm`
6. Create alarm.

### Part 3: Trigger the Alarm (Simulate Load)
SSH into your EC2:
```bash
# Install stress tool
sudo yum install stress -y

# Spike CPU to 100% for 3 minutes
stress --cpu 4 --timeout 180s &
```
Watch CloudWatch → Alarm state changes from OK → In Alarm (takes 1-2 min). Check your email for the SNS notification!

### Part 4: CloudWatch Logs Insights Query
1. CloudWatch → **Log Insights** → Select a log group
2. Run queries:
```sql
-- Find all ERROR-level log entries
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20

-- Count events per minute
fields @timestamp
| stats count(*) as events by bin(1m)
| sort @timestamp
```

### Part 5: Create a Dashboard
1. CloudWatch → **Dashboards** → Create
2. Add widgets: Line chart for CPU, Number widget for request count
3. This becomes your single-pane-of-glass operational view.

## Cleanup
Delete alarm → Delete SNS topic + subscription.
