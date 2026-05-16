# AWS Storage Services

## Introduction to Cloud Storage
Cloud storage decouples data persistence from individual compute resources. Instead of storing data on a local server disk, data is stored in scalable, durable, managed services provided by AWS. This enables independent scaling of compute and data, disaster recovery, global access, and pay-as-you-go cost models. AWS offers three fundamental types of cloud storage: **Object Storage (S3)**, **Block Storage (EBS)**, and **File Storage (EFS)**.

---

## Amazon S3 (Simple Storage Service)
Amazon S3 is AWS's flagship object storage service. Data is stored as **objects** inside **buckets**. Objects can be any type of file—images, videos, backups, JSON files, ML datasets—of virtually any size (up to 5TB per object).

### Key Concepts
- **Buckets:** Globally unique containers for objects. Names must be lowercase and unique across all of AWS.
- **Objects:** Files stored in a bucket, identified by a **Key** (the full file path, like `data/2024/report.csv`).
- **Storage Classes** (matching access frequency to cost):
  - `S3 Standard`: Frequently accessed data. Low latency. Highest cost.
  - `S3 Standard-IA (Infrequent Access)`: Lower cost for less-accessed data. Retrieval fee applies.
  - `S3 Glacier / Glacier Deep Archive`: Archival storage. Minutes to hours to retrieve. Very cheap.
- **Versioning:** Keeps multiple versions of an object, protecting against accidental deletes.
- **Lifecycle Policies:** Automate transitioning objects between storage classes or deleting them after a set period.
- **Access Control:** Bucket Policies (JSON) and ACLs control who can access which objects.
- **Official Docs:** [Amazon S3 Documentation](https://docs.aws.amazon.com/s3/)

---

## Amazon EBS (Elastic Block Store)
Amazon EBS provides **block-level storage volumes** that attach to EC2 instances like a virtual hard drive. The OS formats the volume and uses it like any other disk. EBS volumes are tied to a specific Availability Zone.

### Key Concepts
- **Volume Types:**
  - `gp3 / gp2` – General Purpose SSD. Balanced price/performance. Default for most workloads.
  - `io2 / io1` – Provisioned IOPS SSD. High-performance databases.
  - `st1` – Throughput HDD. Large sequential read workloads like big data.
- **Snapshots:** Point-in-time backups stored in S3. Incremental (only changes since last snapshot).
- **Multi-Attach:** `io1/io2` volumes can attach to multiple EC2 instances simultaneously.
- **Official Docs:** [Amazon EBS Documentation](https://docs.aws.amazon.com/ebs/)

---

## Amazon EFS (Elastic File System)
Amazon EFS provides a managed **NFS (Network File System)** that can be mounted by many EC2 instances simultaneously across multiple Availability Zones. Works like a shared network drive.

### Use Cases
- Shared content repositories (e.g., many web servers reading the same static files).
- Development environments (consistent file system across team containers).
- **Official Docs:** [Amazon EFS Documentation](https://docs.aws.amazon.com/efs/)

---

## Hands-on Lab: S3 Versioning and Lifecycle Policies

### Objective
Enable S3 versioning to protect against data loss, and set up a lifecycle rule to move old object versions to Glacier.

### Steps
1. **Create an S3 bucket** with a unique name (e.g., `my-versioning-lab-12345`). Leave all other settings as default.
2. **Enable Versioning:**
   - Select the bucket → **Properties** → Scroll to "Bucket Versioning" → Edit → Enable. Save.
3. **Test versioning:**
   - Create a local file `notes.txt` with the content `Version 1`.
   - Upload it to the bucket.
   - Overwrite your local file with `Version 2`.
   - Upload the same `notes.txt` again.
   - In the S3 Console, click on `notes.txt` → **Versions** tab. You'll see both version IDs!
   - You can download or restore the older version at any time.
4. **Create a Lifecycle Rule:**
   - Go to **Management** tab → Create lifecycle rule.
   - Name: `ArchiveOldVersions`.
   - Apply to: "All objects in the bucket".
   - Under "Lifecycle rule actions" select "Transition noncurrent versions of objects between storage classes".
   - Storage class: `Glacier Flexible Retrieval` after `30 days`.
   - Save the rule. AWS will now automatically archive old versions after 30 days.

### Cleanup
Select the bucket → Empty (deletes all versions) → Delete bucket.
