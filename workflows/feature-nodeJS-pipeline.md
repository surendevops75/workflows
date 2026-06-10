# Feature Node.js Pipeline

This reusable GitHub Actions workflow implements a complete CI pipeline for Node.js applications.

The workflow is designed to be called from other repositories using `workflow_call` and performs:

- Source Code Checkout
- Dependency Installation
- Unit Testing
- Docker Image Build
- Security Scanning using Trivy
- Push Docker Image to Amazon ECR
- Update GitHub Commit Status Checks

Unlike the DEV pipeline, this workflow is intended for feature branches and pull request validation. It ensures code quality, security, and build verification before changes are merged into the main branch.

---

# CI Pipeline Flow

```text
Developer Pushes Code
         │
         ▼
Reusable Workflow Triggered
         │
         ▼
Checkout Source Code
         │
         ▼
Generate Application Version
         │
         ▼
Install Dependencies
         │
         ▼
Run Unit Tests
         │
         ▼
Update GitHub Status
         │
         ▼
Build Docker Image
         │
         ▼
Update GitHub Status
         │
         ▼
Trivy Security Scan
         │
         ▼
Update GitHub Status
         │
         ▼
Authenticate to AWS
         │
         ▼
Push Image to ECR
```

---

# Workflow Trigger

```yaml
on:
  workflow_call
```

This workflow is a reusable workflow.

It is not triggered directly by:

```yaml
push
pull_request
workflow_dispatch
```

Instead, another workflow calls it.

Example:

```yaml
jobs:
  feature-pipeline:
    uses: org/workflows/.github/workflows/feature-nodejs-pipeline.yaml@main
```

Benefits:

- Centralized pipeline management
- Reusable across multiple repositories
- Consistent CI standards
- Easier maintenance

---

# Input Parameters

The workflow receives values from the calling workflow.

---

## Project

```yaml
project
```

Example:

```yaml
project: roboshop
```

Used for:

- ECR Repository Path
- Kubernetes Namespace
- Deployment Configuration

---

## Component

```yaml
component
```

Example:

```yaml
component: catalogue
```

Represents the microservice being built.

Examples:

```text
catalogue
cart
user
payment
shipping
frontend
```

---

## Environment

```yaml
environment
```

Example:

```yaml
environment: dev
```

Used later during deployments.

---

# Global Environment Variables

```yaml
env:
  ACC_ID: 160885265516
```

Stores AWS Account ID.

Used for:

```text
Amazon ECR Login
Docker Image Tagging
AWS Resource Access
```

---

# Self-Hosted Runner

```yaml
runs-on: self-hosted
```

Uses a self-hosted runner instead of GitHub-hosted runners.

Why?

Because the pipeline requires:

- Docker
- AWS CLI
- Trivy
- kubectl
- Helm

Pre-installed tools reduce execution time and provide more control.

---

# Checkout Source Code

```yaml
actions/checkout
```

Downloads repository contents into the runner.

Required before:

```text
npm install
docker build
testing
```

---

# Generate Application Version

```bash
APP_VERSION=${GITHUB_SHA::9}
```

Creates a version using the first 9 characters of the Git commit SHA.

Example:

```text
Full SHA:
7d9b4a1c5d8ef1234

Generated Version:
7d9b4a1c5
```

Benefits:

- Unique image tags
- Traceable releases
- Easy rollbacks

---

# Install Dependencies

```bash
npm install
```

Installs all packages defined in:

```text
package.json
```

Examples:

```text
express
axios
mongoose
jest
```

Without this step:

```text
Tests cannot run
Application cannot build
```

---

# Unit Testing

```yaml
id: unit_tests
```

Executes application test cases.

Current example:

```bash
echo "run unit tests"
```

In production:

```bash
npm test
```

or

```bash
npm run test
```

Purpose:

- Verify application logic
- Detect regressions
- Validate new features

---

# Update Unit Test Status

After testing, the workflow updates GitHub commit status.

Context:

```text
UNIT_TESTS
```

Possible results:

```text
SUCCESS
FAILURE
CANCELLED
```

Displayed directly in GitHub commits and pull requests.

Example:

```text
UNIT_TESTS ✅
```

---

# Docker Build

Creates a Docker image for the application.

```bash
docker build
```

Generated image:

```text
160885265516.dkr.ecr.us-east-1.amazonaws.com/roboshop/catalogue:7d9b4a1c5
```

Image structure:

```text
AWS_ACCOUNT_ID
        │
        ▼
ECR Repository
        │
        ▼
Project
        │
        ▼
Component
        │
        ▼
Version Tag
```

Benefits:

- Immutable deployments
- Version control
- Consistent runtime environments

---

# Docker Build Status

After image creation, the workflow updates GitHub status.

Context:

```text
DOCKER_BUILD
```

Example:

```text
DOCKER_BUILD ✅
```

---

# Security Scan (Trivy)

Uses:

- :contentReference[oaicite:0]{index=0}

to scan the generated Docker image.

Purpose:

- Detect vulnerabilities before deployment
- Improve security posture
- Shift security left

---

# Severity Levels

Current scan checks:

```text
MEDIUM
HIGH
CRITICAL
```

Configuration:

```bash
--severity HIGH,CRITICAL,MEDIUM
```

---

# Fail Pipeline on Vulnerabilities

```bash
--exit-code 1
```

Behavior:

```text
Vulnerabilities Found
        ↓
Pipeline Fails
```

Benefits:

- Prevent insecure deployments
- Enforce security standards
- Reduce production risks

---

# Security Status Update

Updates GitHub commit status.

Context:

```text
SECURITY_CHECK
```

Example:

```text
SECURITY_CHECK ✅
```

---

# AWS Authentication

Uses:

```yaml
aws-actions/configure-aws-credentials
```

Credentials stored securely in:

```text
GitHub Secrets
```

Examples:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

Benefits:

- Secure authentication
- No hardcoded credentials
- Centralized secret management

---

# Amazon ECR Login

Authenticates Docker to:

- :contentReference[oaicite:1]{index=1}

Command:

```bash
aws ecr get-login-password
```

Flow:

```text
AWS
    ↓
Generate Token
    ↓
Docker Login
    ↓
ECR Access Granted
```

---

# Push Docker Image

Uploads image to Amazon ECR.

```bash
docker push
```

Example:

```text
160885265516.dkr.ecr.us-east-1.amazonaws.com/roboshop/catalogue:7d9b4a1c5
```

Benefits:

- Centralized image storage
- Versioned artifacts
- Ready for Kubernetes deployment

---

# Commit Status Checks Created

This workflow creates:

```text
UNIT_TESTS
DOCKER_BUILD
SECURITY_CHECK
```

GitHub UI:

```text
UNIT_TESTS       ✅
DOCKER_BUILD     ✅
SECURITY_CHECK   ✅
```

These checks help developers quickly identify failures.

---

# Real DevOps Use Cases

## Pull Request Validation

Verify code quality before merge.

---

## Security Compliance

Block vulnerable images from progressing.

---

## Containerized Applications

Build and store Docker images.

---

## Microservices

Standardized CI process across multiple services.

Examples:

```text
catalogue
cart
user
payment
shipping
```

---

# Best Practices Used

✅ Reusable Workflows

✅ Self-Hosted Runners

✅ SHA-Based Versioning

✅ Container Security Scanning

✅ GitHub Status Checks

✅ Secure AWS Authentication

✅ Centralized Artifact Storage

---

# Benefits of This Workflow

- Automated CI Pipeline
- Consistent Build Process
- Security Integrated Early
- Reusable Across Projects
- Better Developer Feedback
- Production-Ready Design

---

# Why This Workflow Is Important

This workflow demonstrates modern CI and DevSecOps practices using:

- :contentReference[oaicite:2]{index=2}
- :contentReference[oaicite:3]{index=3}
- :contentReference[oaicite:4]{index=4}
- :contentReference[oaicite:5]{index=5}

These concepts are fundamental for:

- DevOps Engineering
- DevSecOps
- Platform Engineering
- Cloud-Native Applications
- Enterprise CI/CD Pipelines