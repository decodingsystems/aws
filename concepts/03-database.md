# AWS Database Services

## Introduction to Cloud Databases
AWS offers a broad portfolio of database services covering relational, NoSQL, in-memory, graph, ledger, and time-series use cases. The key advantage is **fully managed** infrastructure: AWS handles provisioning, patching, backups, replication, and failover, so teams focus on their data and application logic rather than DBA-level infrastructure work.

---

## Amazon RDS (Relational Database Service)
Amazon RDS makes it easy to set up, operate, and scale relational databases in the cloud. It automates time-consuming tasks like hardware provisioning, OS patching, database setup, and automated backups.

### Supported Engines
MySQL, PostgreSQL, MariaDB, Oracle, Microsoft SQL Server, and Amazon Aurora.

### Key Concepts
- **Amazon Aurora:** AWS's own MySQL/PostgreSQL-compatible database engine, designed for cloud-native performance. Up to 5× faster than standard MySQL at the same cost.
- **Multi-AZ Deployments:** RDS automatically provisions a synchronous standby replica in a different Availability Zone. Automatic failover if the primary instance fails (typically < 60 seconds).
- **Read Replicas:** Asynchronous copies of the primary database used to offload read traffic. Can be promoted to standalone DB instances.
- **Automated Backups:** Daily snapshots + transaction logs enable point-in-time recovery within the retention window (1–35 days).
- **Parameter Groups / Option Groups:** Configure database engine settings (e.g., max connections, slow query log).
- **Official Docs:** [Amazon RDS Documentation](https://docs.aws.amazon.com/rds/)

---

## Amazon DynamoDB
Amazon DynamoDB is a fully managed, serverless **key-value and document NoSQL** database designed for single-digit millisecond performance at any scale. There are no servers to provision, and capacity scales automatically.

### Key Concepts
- **Tables, Items, Attributes:** Similar to rows and columns, but schema-flexible.
- **Primary Key:** Every table must have one. Two options:
  - **Partition Key only:** Simple key. DynamoDB distributes items across partitions based on this key's hash.
  - **Partition Key + Sort Key (Composite Key):** Enables range queries within a partition.
- **Capacity Modes:**
  - **Provisioned:** You specify read/write capacity units (RCU/WCU). More predictable cost.
  - **On-Demand:** Pay per request with no capacity planning. Best for unpredictable workloads.
- **Global Tables:** Multi-region, multi-master replication for globally distributed applications.
- **DynamoDB Streams:** Change data capture. Every write is published to a stream, enabling real-time triggers (e.g., Lambda).
- **Official Docs:** [Amazon DynamoDB Documentation](https://docs.aws.amazon.com/dynamodb/)

---

## Amazon ElastiCache
Amazon ElastiCache provides fully managed **in-memory caching** using Redis or Memcached. It dramatically reduces database load by serving frequently accessed data from memory (sub-millisecond latency) instead of disk.

### Use Cases
- **Session stores:** Fast read/write for user session data.
- **Database query caching:** Cache expensive DB results.
- **Leaderboards:** Redis sorted sets for real-time rankings.
- **Official Docs:** [Amazon ElastiCache Documentation](https://docs.aws.amazon.com/elasticache/)

---

## Hands-on Lab: Create a DynamoDB Table and Run CRUD Operations

### Objective
Create a DynamoDB table using the AWS Console and perform basic Create, Read, Update, and Delete operations.

### Steps
1. **Navigate to DynamoDB** in the AWS Console → Click "Create table".
2. **Configure:**
   - Table name: `Users`.
   - Partition key: `userId` (String).
   - Leave Sort key empty.
   - Table settings: Default settings (On-Demand mode). Click Create.
3. **Create an Item (PUT):**
   - Open the table → Click "Explore table items" → "Create item".
   - Add attributes:
     - `userId`: `user-001`
     - Add Attribute → `name`: `Abhijeet`
     - Add Attribute → `email`: `abhijeet@example.com`
   - Click "Create item".
4. **Read (GET):** The item appears in the table view. Click it to see all attributes.
5. **Update:** Click on the item → Edit →  change `email` to `updated@example.com` → Save.
6. **Delete:** Check the box next to the item → Actions → Delete items.

### Bonus: Use AWS CLI
```bash
aws dynamodb put-item \
  --table-name Users \
  --item '{"userId": {"S": "user-002"}, "name": {"S": "Test User"}}'

aws dynamodb get-item \
  --table-name Users \
  --key '{"userId": {"S": "user-002"}}'
```

### Cleanup
Go to the DynamoDB console → Select the `Users` table → Delete table.
