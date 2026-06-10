# Feature Node.js Pipeline (Reusable GitHub Actions Workflow)

This workflow implements a complete CI/CD pipeline for Node.js applications.

It is designed as a **reusable workflow** and can be called by multiple repositories using `workflow_call`.

The pipeline performs:

- Source Code Checkout
- Dependency Installation
- Unit Testing
- Docker Image Build
- Security Scanning (Trivy)
- Docker Image Push to ECR
- EKS Authentication
- Helm Deployment
- GitHub Commit Status Updates

This workflow follows modern DevOps and GitOps practices.

---

# CI/CD Pipeline Flow

```text
Code Push
    │
    ▼
Reusable Workflow Triggered
    │
    ▼
Checkout Source Code
    │
    ▼
Install Dependencies
    │
    ▼
Unit Tests
    │
    ▼
Docker Build
    │
    ▼
Trivy Security Scan
    │
    ▼
Push Image to ECR
    │
    ▼
Connect to EKS
    │
    ▼
Helm Deployment
    │
    ▼
Update GitHub Status Checks
```

---

# Workflow Type

```yaml
on:
  workflow_call
```

This workflow is not triggered directly.

Instead it is called by another workflow.

Example:

```yaml
jobs:
  feature:
    uses: company/workflows/.github/workflows/feature-nodejs-pipeline.yaml@main
```

Benefits:

- Centralized CI/CD
- Reusable across repositories
- Easier maintenance
- Consistent deployments

---

# Input Parameters

```yaml
inputs:
```

Inputs are passed by the calling workflow.

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

```text
ECR Repository
Kubernetes Namespace
Helm Deployment
```

---

## Component

```yaml
component
```

Example:

```yaml
component: catalogue
```

Represents the application being deployed.

---

## Environment

```yaml
environment
```

Example:

```yaml
environment: dev
```

Determines:

```text
values-dev.yaml
values-qa.yaml
values-uat.yaml
```

---

# Global Environment Variables

```yaml
env:
```

---

## AWS Account ID

```yaml
ACC_ID: 160885265516
```

Used for:

```text
Amazon ECR
AWS Authentication
Docker Push
```

---

# Runner

```yaml
runs-on: self-hosted
```

Uses a self-hosted runner.

Why?

Because pipeline requires:

- Docker
- Helm
- kubectl
- AWS CLI
- Trivy

which are often unavailable on GitHub-hosted runners.

---

# Source Code Checkout

```yaml
actions/checkout
```

Downloads repository source code.

Required before:

```text
npm install
docker build
helm deployment
```

---

# Version Generation

```bash
echo "APP_VERSION=${GITHUB_SHA::9}"
```

Creates application version using commit SHA.

Example:

```text
Commit SHA:
7d9b4a1c5d8ef123

Version:
7d9b4a1c5
```

Benefits:

- Unique image tags
- Easy rollback
- Traceability

---

# Install Dependencies

```bash
npm install
```

Installs:

```text
package.json dependencies
```

Required before:

```text
Testing
Building
Packaging
```

---

# Unit Testing

```yaml
id: unit_tests
```

Runs application tests.

Current example:

```bash
echo "run unit tests"
```

Real-world examples:

```bash
npm test
```

or

```bash
jest
```

or

```bash
npm run test
```

---

# Commit Status Update

After testing:

```yaml
Update unit tests status
```

Uses your custom Composite Action.

Possible statuses:

```text
SUCCESS
FAILURE
CANCELLED
```

Visible in GitHub:

```text
UNIT_TESTS ✅
```

---

# Docker Build

Builds Docker image.

```bash
docker build
```

Generated image:

```text
160885265516.dkr.ecr.us-east-1.amazonaws.com/roboshop/catalogue:7d9b4a1c5
```

Benefits:

- Immutable artifact
- Versioned deployment
- Easy rollback

---

# Docker Status Check

After build:

```yaml
DOCKER_BUILD
```

Status appears in commit checks.

Example:

```text
DOCKER_BUILD ✅
```

---

# Security Scan (Trivy)

```yaml
Security Check
```

Uses:

:contentReference[oaicite:0]{index=0}

to scan container images.

---

## What Is Scanned?

```text
Operating System Packages
Known Vulnerabilities
Security Issues
```

---

## Severity Levels

```text
MEDIUM
HIGH
CRITICAL
```

Current configuration:

```bash
--severity HIGH,CRITICAL,MEDIUM
```

---

## Fail Build on Vulnerabilities

```bash
--exit-code 1
```

Pipeline fails if vulnerabilities exist.

Benefits:

```text
Shift Left Security
DevSecOps
Compliance
```

---

# Security Status Update

Creates GitHub status:

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

Credentials:

```yaml
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

stored in GitHub Secrets.

---

# Push Image to ECR

Login:

```bash
aws ecr get-login-password
```

Push:

```bash
docker push
```

Result:

```text
Docker Image Stored in ECR
```

Repository:

```text
roboshop/catalogue
```

---

# Authenticate to EKS

```bash
aws eks update-kubeconfig
```

Connects runner to:

```text
roboshop-dev
```

cluster.

Verification:

```bash
kubectl get nodes
```

---

# Checkout Deployment Repository

Pipeline switches repositories.

Application repo:

```text
catalogue
```

Deployment repo:

```text
catalogue-deploy
```

Benefits:

```text
GitOps
Separation of Concerns
Independent Releases
```

---

# Helm Deployment

Deploys application using:

:contentReference[oaicite:1]{index=1}

Command:

```bash
helm upgrade --install
```

---

# What Happens?

```text
If Release Exists
      ↓
Upgrade

If Release Missing
      ↓
Install
```

---

# Values File Selection

```yaml
values-${environment}.yaml
```

Examples:

```text
values-dev.yaml
values-qa.yaml
values-uat.yaml
```

Environment-specific configuration.

---

# Image Version Injection

```bash
--set deployment.imageVersion=${APP_VERSION}
```

Injects newly built image.

Example:

```text
catalogue:7d9b4a1c5
```

---

# Atomic Deployment

```bash
--atomic
```

Behavior:

```text
Deployment Success
      ↓
Keep Changes

Deployment Failure
      ↓
Automatic Rollback
```

Benefits:

- Zero broken releases
- Safer deployments

---

# Wait for Deployment

```bash
--wait
```

Pipeline waits until resources become healthy.

Checks:

```text
Pods
Deployments
Services
```

---

# Deployment Timeout

```bash
--timeout=5m
```

Maximum wait time:

```text
5 Minutes
```

---

# Deployment Status Update

Creates GitHub status:

```text
DEV_DEPLOY
```

Example:

```text
DEV_DEPLOY ✅
```

---

# Complete Status Checks

This workflow creates:

```text
UNIT_TESTS
DOCKER_BUILD
SECURITY_CHECK
DEV_DEPLOY
```

GitHub UI:

```text
UNIT_TESTS       ✅
DOCKER_BUILD     ✅
SECURITY_CHECK   ✅
DEV_DEPLOY       ✅
```

---

# DevOps Concepts Demonstrated

## CI

- Checkout
- Dependency Installation
- Testing
- Docker Build

---

## DevSecOps

- Trivy Scanning
- Vulnerability Detection

---

## CD

- ECR Push
- Helm Deployment
- EKS Deployment

---

## GitOps

Separate:

```text
Application Repository
Deployment Repository
```

---

## Observability

GitHub Commit Status API integration.

---

# Best Practices Used

✅ Reusable Workflows

✅ Self-Hosted Runners

✅ SHA-Based Versioning

✅ Container Security Scanning

✅ GitHub Commit Status Checks

✅ Helm Atomic Deployments

✅ GitOps Repository Separation

✅ EKS Deployments

---

# Benefits of This Pipeline

- Fully Automated CI/CD
- Security Integrated
- Reusable Across Projects
- Deployment Traceability
- Easy Rollbacks
- Production-Ready Design

---

# Why This Workflow Is Important

This workflow demonstrates modern enterprise CI/CD practices using:

- :contentReference[oaicite:2]{index=2}
- :contentReference[oaicite:3]{index=3}
- :contentReference[oaicite:4]{index=4}
- :contentReference[oaicite:5]{index=5}
- :contentReference[oaicite:6]{index=6}

These concepts are foundational for:

- DevOps Engineering
- Platform Engineering
- DevSecOps
- Kubernetes Operations
- Enterprise CI/CD Platforms