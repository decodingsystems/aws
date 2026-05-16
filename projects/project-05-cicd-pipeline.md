# Project 5: CI/CD Pipeline — CodePipeline + CodeBuild + S3

## Architecture
```
GitHub Push (main branch)
        │  webhook trigger
        ▼
 AWS CodePipeline
    │         │         │
    ▼         ▼         ▼
 Source    Build      Deploy
(GitHub) (CodeBuild) (S3/EB)
              │
              ▼
        Run tests, build
        artifacts, push
        to S3 bucket
```

## What You'll Build
A fully automated CI/CD pipeline: push code to GitHub → AWS automatically builds and tests it → deploys the updated website/app to production S3.

## Services Used
- **CodePipeline** (orchestrate the pipeline)
- **CodeBuild** (build and test)
- **S3** (artifact storage + deployment target)
- **GitHub** (source)
- **IAM** (service roles)

---

## Step 1: Prepare the Source Repo on GitHub
Create a repo with this structure:
```
├── src/
│   └── index.html
├── buildspec.yml         ← CodeBuild instructions
└── tests/
    └── test_index.sh
```

`buildspec.yml`:
```yaml
version: 0.2
phases:
  install:
    commands:
      - echo "Setting up environment..."
  pre_build:
    commands:
      - echo "Running tests..."
      - bash tests/test_index.sh
  build:
    commands:
      - echo "Building..."
      - mkdir -p dist
      - cp -r src/* dist/
      - echo "Build timestamp: $(date)" >> dist/build-info.txt
  post_build:
    commands:
      - echo "Build completed at $(date)"
artifacts:
  files:
    - '**/*'
  base-directory: dist
```

`tests/test_index.sh`:
```bash
#!/bin/bash
if grep -q "<title>" src/index.html; then
    echo "✅ Title tag found"
else
    echo "❌ Title tag missing" && exit 1
fi
echo "All tests passed!"
```

## Step 2: Create S3 Buckets
```bash
aws s3 mb s3://my-pipeline-artifacts-<unique>   # CodePipeline artifacts
aws s3 mb s3://my-pipeline-website-<unique>     # Deployment target
```
Enable static website hosting on the website bucket.

## Step 3: Create CodeBuild Project
1. CodeBuild → Create project
   - Name: `my-app-build`
   - Source: **GitHub** → Connect → Select your repo
   - Environment: Managed image → Ubuntu → Standard → Latest
   - Buildspec: Use a buildspec file in the project
   - Artifacts: **No artifacts** (pipeline manages this)
   - Create service role automatically.

## Step 4: Create CodePipeline
1. CodePipeline → Create pipeline
   - Name: `my-app-pipeline`
   - **Source stage**: GitHub (Version 2) → Connect GitHub → Select repo, branch: `main`
   - **Build stage**: CodeBuild → select `my-app-build`
   - **Deploy stage**: Amazon S3 → select website bucket → Extract file before deploy: ✅
   - Create.

## Step 5: Test the Pipeline
1. Make a code change in GitHub, commit + push to `main`.
2. CodePipeline console → Watch each stage progress: Source ✅ → Build ✅ → Deploy ✅
3. Open the S3 website URL → Your changes are live!

## Step 6: Add Notifications
1. Pipeline → Settings → Notifications → Create rule
2. Events: Pipeline execution started/failed/succeeded
3. Target: SNS topic (email) → You'll get emailed on every deployment!

## Cleanup
Delete CodePipeline → CodeBuild project → S3 buckets → IAM service roles.
