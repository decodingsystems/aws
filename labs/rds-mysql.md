# Lab 07: RDS MySQL — Launch, Connect, and Query

## Objective
Launch a managed MySQL database on Amazon RDS, connect to it from an EC2 instance in the same VPC, and run SQL queries.

## Steps

### Part 1: Launch RDS MySQL Instance
1. RDS → **Create database**
   - Engine: **MySQL** 8.0
   - Template: **Free tier**
   - DB identifier: `mydb-lab`
   - Master username: `admin`
   - Master password: `Password123!` (note it)
   - DB instance: `db.t3.micro`
   - **Storage**: 20 GiB gp2
   - **Connectivity**:
     - VPC: Default
     - **Public access: No** (best practice — access only via EC2 in same VPC)
     - Create new security group: `rds-sg`
   - Click Create (takes ~5 mins).

### Part 2: Update RDS Security Group
1. While waiting, find the `rds-sg` → Edit inbound rules:
   - Type: MySQL/Aurora, Port: 3306, Source: **Security group of your EC2** (or `0.0.0.0/0` for lab purposes)

### Part 3: Launch EC2 as DB Client
1. Launch an EC2 (`t2.micro`, Amazon Linux 2023) in the same VPC.
2. SSH into it:
   ```bash
   sudo yum install mysql -y
   ```

### Part 4: Connect from EC2 to RDS
```bash
# Get the RDS endpoint from: RDS → Databases → mydb-lab → Endpoint
mysql -h <rds-endpoint> -u admin -p
# Enter password: Password123!
```

### Part 5: Run SQL Commands
```sql
-- Create a database and table
CREATE DATABASE appdb;
USE appdb;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert records
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');
INSERT INTO users (name, email) VALUES ('Bob', 'bob@example.com');

-- Query
SELECT * FROM users;
SELECT COUNT(*) FROM users;

EXIT;
```

### Part 6: Enable Automated Backups and Verify
1. RDS → Databases → mydb-lab → Maintenance & backups
2. Automated backups: 7 day retention (default)
3. You can restore to any point in time within the retention window!

## Cleanup
RDS → Delete database (disable deletion protection first) → Terminate EC2.
