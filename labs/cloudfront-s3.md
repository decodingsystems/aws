# Lab 09: CloudFront CDN for S3 Static Website

## Objective
Accelerate a static website by placing Amazon CloudFront in front of an S3 bucket, serving content from edge locations globally.

## Steps

### Part 1: Create and Populate S3 Bucket (private)
1. Create S3 bucket: `cdn-origin-<unique>`. **Keep "Block all public access" ENABLED.**
2. Create `index.html`:
   ```html
   <!DOCTYPE html>
   <html>
   <head><title>My CDN Site</title></head>
   <body><h1>Hello from CloudFront Edge!</h1><p>This content is cached at the edge location nearest to you.</p></body>
   </html>
   ```
3. Upload `index.html` to the bucket.

### Part 2: Create CloudFront Distribution
1. CloudFront → **Create distribution**
2. **Origin:**
   - Origin domain: Select your S3 bucket from the dropdown.
   - **Origin access: Origin access control settings (OAC)**
   - Click "Create new OAC" → Create. (This lets CloudFront access private S3)
3. **Default cache behavior:**
   - Viewer protocol policy: **Redirect HTTP to HTTPS**
   - Cache policy: `CachingOptimized`
4. **Settings:**
   - Default root object: `index.html`
5. Click **Create distribution** (takes ~5-10 mins to deploy globally).

### Part 3: Update S3 Bucket Policy (for OAC)
After creating, CloudFront will show a banner: **"Copy policy"**. Click it, then:
1. Go to S3 bucket → Permissions → Bucket policy → Edit → paste the copied policy → Save.

### Part 4: Test
1. Copy the **Distribution domain name** from CloudFront (e.g., `d1234abcd.cloudfront.net`)
2. Visit `https://d1234abcd.cloudfront.net` → Your HTML page loads over HTTPS from the nearest edge!
3. Try curl with timing to see how fast it is:
   ```bash
   curl -o /dev/null -s -w "Total time: %{time_total}s\n" https://d1234abcd.cloudfront.net
   ```
4. Check response headers for `X-Cache: Hit from cloudfront` on second request.

## Cleanup
CloudFront → Disable distribution (wait ~5 mins) → Delete → Empty + delete S3 bucket.
