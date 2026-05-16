# Project 4: Static Website with Global CDN and Custom Domain

## Architecture
```
Browser → Route 53 (DNS) → CloudFront (HTTPS, CDN Edge)
                                │
                        S3 Bucket (Private)
                    (Origin, OAC protected)
```

## What You'll Build
A production-grade static website with:
- Custom domain (or free CloudFront URL)
- HTTPS enforced
- Global content delivery via 400+ edge locations
- Zero server management

## Services Used
- **S3** (origin storage)
- **CloudFront** (CDN + HTTPS)
- **ACM** (SSL Certificate if using custom domain)
- **Route 53** (DNS if using custom domain)

---

## Step 1: Create the S3 Bucket
1. Create bucket: `my-portfolio-site-<unique>`
2. **Block all public access** = enabled (CloudFront will access it via OAC)
3. Enable static website hosting (for error document only):
   - Index: `index.html`, Error: `404.html`

## Step 2: Build Your Site
Create these local files:
```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Portfolio</title>
  <style>
    body { font-family: sans-serif; max-width: 800px; margin: 50px auto; }
    h1 { color: #232f3e; }
  </style>
</head>
<body>
  <h1>Hello, I'm deploying on AWS! 🚀</h1>
  <p>This site is served from S3 via CloudFront CDN.</p>
</body>
</html>
```
Upload to S3:
```bash
aws s3 sync ./site s3://my-portfolio-site-<unique>/ --delete
```

## Step 3: Request ACM SSL Certificate (us-east-1 — required for CloudFront)
1. ACM (Certificate Manager) → **Region: us-east-1** → Request certificate
2. Domain: `yourdomain.com` and `www.yourdomain.com`
3. Validation: DNS (add CNAMEs to your DNS provider or Route 53)
4. Wait for status = **Issued**
- *Skip this step if you don't have a custom domain; CloudFront provides a free HTTPS URL.*

## Step 4: Create CloudFront Distribution
1. CloudFront → Create distribution
   - Origin: your S3 bucket (select from dropdown)
   - **Origin access: OAC** → Create new OAC → Use it
   - Viewer protocol: **Redirect HTTP to HTTPS**
   - Default root object: `index.html`
   - If using custom domain: Alternate domain names → add your domain, SSL cert → your ACM cert
2. Create. Get the bucket policy from the banner and paste into S3.

## Step 5: Route 53 (Custom Domain Only)
1. Route 53 → Hosted zone for your domain → Create record
   - Type: **A**, Alias: Yes
   - Route traffic to: CloudFront distribution → select yours
   - Create.

## Step 6: Test
```bash
# Without custom domain
curl -I https://d1234abcd.cloudfront.net

# With custom domain
curl -I https://yourdomain.com

# Check cache headers
curl -I https://yourdomain.com/index.html | grep -E "x-cache|cache-control|age"
# x-cache: Hit from cloudfront  (second request is served from edge!)
```

## Step 7: Cache Invalidation (after updates)
```bash
# After updating files, invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/*"
```

## Cleanup
Disable → Delete CloudFront distribution → Delete ACM cert → Delete S3 bucket.
