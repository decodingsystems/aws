# Lab 05: S3 Cross-Region Replication

## Objective
Set up automatic replication of all objects from a source S3 bucket to a destination bucket in a different AWS region for disaster recovery.

## Steps

### Part 1: Create Source Bucket (us-east-1)
1. S3 → Create bucket
   - Name: `source-lab-<unique>`, Region: **us-east-1**
   - **Enable Bucket Versioning** (required for replication)
   - Create.

### Part 2: Create Destination Bucket (us-west-2)
1. S3 → Create bucket
   - Name: `dest-lab-<unique>`, Region: **us-west-2**
   - **Enable Bucket Versioning**
   - Create.

### Part 3: Configure Replication Rule
1. Go to the **source** bucket → **Management** → Replication rules → Create rule
   - Rule name: `replicate-all`
   - Status: Enabled
   - Source: All objects in bucket
   - Destination: Choose `dest-lab-<unique>` (different account → No)
   - **IAM role**: Create new role (AWS creates permissions automatically)
   - Click Save.

### Part 4: Test Replication
1. Upload a file to the **source** bucket.
2. Wait ~30-60 seconds.
3. Navigate to the **destination** bucket in `us-west-2`.
4. The file appears automatically! Check its **Replication status** = `REPLICA`.

### Notes
- Replication is **async** (near real-time, not instant).
- Only **new** objects after enabling replication are replicated. Existing objects require a **Batch Operations** job.
- Delete markers are NOT replicated by default (optional setting).

## Cleanup
Empty and delete both buckets.
