

# Automated CI/CD Web Deployment Pipeline (GitHub to AWS S3 & CloudFront)

An automated CI/CD pipeline that instantly deploys a static portfolio website to a highly available and secure AWS infrastructure upon every code push.

By leveraging **GitHub Actions** and AWS cloud services (**S3, CloudFront, IAM**), this project eliminates manual deployment overhead, reduces latency globally, and ensures secure, encrypted traffic using modern DevOps practices.

---

## 🏗️ Architecture Overview

The pipeline follows a modern, serverless hosting architecture:

1. **Source**: Developer pushes code changes (HTML/CSS/JS) to the `main` branch of the GitHub repository.
2. **CI/CD Build & Deploy**: **GitHub Actions** triggers a workflow, authenticates securely with AWS using dedicated IAM credentials, and synchronizes the files.
3. **Storage**: **AWS S3** hosts the static web files securely. Direct public access to the bucket is blocked for maximum security.
4. **Content Delivery (CDN)**: **AWS CloudFront** caches and serves the website from edge locations globally to drastically reduce load times.
5. **Security**: CloudFront enforces HTTPS communication, while an **Origin Access Control (OAC)** policy ensures the S3 bucket only accepts requests originating from CloudFront.

---

## 🛠️ Tech Stack & AWS Services

* **Version Control & CI/CD**: GitHub & GitHub Actions
* **Storage & Hosting**: AWS Simple Storage Service (S3)
* **Content Delivery Network**: AWS CloudFront
* **Identity & Access Management**: AWS IAM (Configured using the principle of least privilege)
* **Frontend**: HTML5, CSS3, JavaScript

---

## 🚀 Key Features

* **Zero-Touch Deployment**: Push code to GitHub, and your updates are live globally in under a minute.
* **Production-Grade Security**: S3 Public Access is entirely blocked. The bucket is isolated, relying strictly on CloudFront OAC for asset retrieval.
* **Global Performance**: CloudFront caches assets at AWS edge locations, providing low-latency access to users worldwide.
* **Cache Invalidation**: The deployment pipeline automatically clears the CloudFront edge cache on every deploy, ensuring users see the latest updates instantly without browser-stale issues.

---

## 📖 Step-by-Step Setup & Deployment

### 1. AWS Infrastructure Configuration

#### **AWS S3 Bucket**

1. Create an S3 bucket (e.g., `yourname-portfolio-bucket`).
2. Keep **Block *all* public access** enabled (Secure Best Practice).
3. Enable **Static website hosting** under the Properties tab.

#### **AWS CloudFront Distribution**

1. Create a new CloudFront distribution and set the **Origin Domain** to your S3 bucket's website endpoint.
2. Under **Origin Access**, select **Origin Access Control (OAC)** and create a new control setting to ensure secure bucket access.
3. Once created, copy the generated **S3 Bucket Policy** from CloudFront and paste it into your S3 bucket's **Permissions** tab.

#### **AWS IAM User (CI/CD Credentials)**

1. Create an IAM User named `github-actions-deployer`.
2. Attach a **least privilege inline policy** allowing only `s3:Sync`, `s3:PutObject`, and `cloudfront:CreateInvalidation`.

> ⚠️ **Security Note:** Copy the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` immediately.

---

### 2. Configuring GitHub Secrets

To deploy securely without exposing sensitive cloud credentials in your codebase, add the following **Repository Secrets** under `Settings > Secrets and variables > Actions`:

| Secret Name | Description |
| --- | --- |
| `AWS_ACCESS_KEY_ID` | Your dedicated IAM user access key |
| `AWS_SECRET_ACCESS_KEY` | Your dedicated IAM user secret key |
| `AWS_S3_BUCKET_NAME` | The exact name of your production S3 bucket |
| `AWS_CLOUDFRONT_DIST_ID` | Your CloudFront Distribution ID (for cache invalidation) |

---

### 3. CI/CD Workflow Pipeline (`deploy.yml`)

The automation pipeline is configured in `.github/workflows/deploy.yml`. On every `push` to the `main` branch, the runner spins up an Ubuntu environment, configures AWS credentials, syncs the repository to S3, and invalidates the CloudFront cache.

```yaml
name: Deploy Portfolio to AWS S3 & CloudFront

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    # Step 1: Checkout the repository code
    - name: Checkout Code
      uses: actions/checkout@v4

    # Step 2: Configure AWS Credentials using GitHub Secrets
    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1 # Change to your preferred AWS Region

    # Step 3: Sync static files to the S3 Bucket
    - name: Deploy to S3
      run: |
        aws s3 sync . s3://${{ secrets.AWS_S3_BUCKET_NAME }} --delete --exclude ".git/*" --exclude ".github/*"

    # Step 4: Invalidate CloudFront Cache to force immediate global updates
    - name: Invalidate CloudFront Cache
      run: |
        aws cloudfront create-invalidation --distribution-id ${{ secrets.AWS_CLOUDFRONT_DIST_ID }} --paths "/*"

```

---

## 📈 Verification & Testing

1. Modify a file (e.g., `index.html`) in your local workspace.
2. Commit and push the changes to GitHub:
```bash
git add .
git commit -m "feat: updated portfolio landing page"
git push origin main

```


3. Navigate to the **Actions** tab in your GitHub repository to watch the real-time execution of the deployment pipeline.
4. Once completed, visit your CloudFront URL to verify the changes are live.

---

## 🔒 Security Best Practices Implemented

* **IAM Least Privilege:** The pipeline credentials hold zero administration rights and are confined strictly to the target S3 bucket and CloudFront distribution.
* **Origin Access Control (OAC):** Direct traffic to the S3 bucket is blocked; the site is accessible *only* via the CloudFront CDN, mitigating data exposure risks.
* **Secure Environment Variables:** No hardcoded API keys; all cloud operations utilize encrypted GitHub Secret Stores.