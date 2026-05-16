# Lab 01: Launch EC2 with User Data Bootstrap Script

## Objective
Launch an EC2 instance that automatically installs and starts Apache + deploys a sample HTML page using the **User Data** feature — no manual SSH needed.

## Steps
1. **EC2 Console → Launch Instance**
2. Name: `web-bootstrap`. AMI: **Amazon Linux 2023**. Instance type: `t3.micro`
3. Key pair: Create or select existing.
4. Security Group: Allow **SSH (22)** from My IP, **HTTP (80)** from Anywhere.
5. **Expand "Advanced details" → User Data field** → paste:
   ```bash
   #!/bin/bash
   yum update -y
   yum install httpd -y
   systemctl start httpd
   systemctl enable httpd
   echo "<h1>Hello from EC2! Instance ID: $(curl -s http://169.254.169.254/latest/meta-data/instance-id)</h1>" > /var/www/html/index.html
   ```
6. Click **Launch Instance**.
7. Wait ~2 minutes for initialization. Copy the **Public IPv4 address**.
8. Visit `http://<PUBLIC-IP>` in your browser. You'll see the instance ID displayed!
9. **SSH in and verify:**
   ```bash
   ssh -i your-key.pem ec2-user@<PUBLIC-IP>
   systemctl status httpd   # Should be active (running)
   cat /var/www/html/index.html
   ```
10. Inspect the instance metadata from within:
    ```bash
    curl http://169.254.169.254/latest/meta-data/   # List all metadata keys
    curl http://169.254.169.254/latest/meta-data/instance-type
    curl http://169.254.169.254/latest/meta-data/local-ipv4
    ```

## Cleanup
EC2 → Select instance → Instance State → **Terminate**.
