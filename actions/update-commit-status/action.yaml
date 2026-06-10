# GitHub Composite Action – Update Commit Status

This Composite Action updates the status of a Git commit using the GitHub Statuses API.

It is commonly used in CI/CD pipelines to report the status of different stages such as:

- Build
- Test
- Security Scan
- Docker Build
- Deployment

The status appears directly in GitHub commits and pull requests, making it easy to track pipeline progress.

---

# Purpose

Instead of relying only on workflow success or failure, this action allows you to create custom status checks.

Example:

```text
Commit SHA
    │
    ├── UNIT_TESTS      ✅ Success
    ├── DOCKER_BUILD    ✅ Success
    ├── SONAR_SCAN      ⏳ Pending
    └── DEPLOYMENT      ❌ Failed
```

These statuses become visible in:

- Pull Requests
- Commit Details
- Branch Protection Rules

---

# Composite Action

```yaml
name: "Update Commit Status"

description: "Updates GitHub commit status checks using the GitHub Statuses API"

inputs:

  sha:
    description: "Commit SHA to update"
    required: true

  state:
    description: "Status: success, failure, pending, error"
    required: true

  context:
    description: "Status check context name (ex: DOCKER_BUILD)"
    required: true

  description:
    description: "Optional description for the status"
    required: false
    default: ""

runs:

  using: "composite"

  steps:

    - name: Update commit status

      shell: bash

      run: |

        curl -X POST \
          -H "Authorization: Bearer ${{ github.token }}" \
          -H "Content-Type: application/json" \
          --data "{
            \"state\": \"${{ inputs.state }}\",
            \"context\": \"${{ inputs.context }}\",
            \"description\": \"${{ inputs.description }}\"
          }" \
          "https://api.github.com/repos/${{ github.repository }}/statuses/${{ inputs.sha }}"
```

---

# What Is a Composite Action?

```yaml
using: composite
```

A Composite Action allows you to package reusable workflow logic.

Benefits:

- Reusable across repositories
- Reduces duplicated code
- Easier maintenance
- Standardized CI/CD practices

Example:

```text
Repository A
      ↓
Uses Action

Repository B
      ↓
Uses Same Action

Repository C
      ↓
Uses Same Action
```

---

# Action Name

```yaml
name: "Update Commit Status"
```

Human-readable name shown in GitHub Actions logs.

Purpose:

```text
Update commit status checks
```

---

# Description

```yaml
description:
```

Explains what the action does.

Used when publishing actions internally or publicly.

---

# Inputs

```yaml
inputs:
```

Inputs allow workflows to pass values into the action.

---

# SHA Input

```yaml
sha:
```

Commit SHA that should receive the status update.

Example:

```text
7d9b4a1c5d8e
```

Usage:

```yaml
sha: ${{ github.sha }}
```

---

# State Input

```yaml
state:
```

Defines the status result.

Supported values:

```text
success
failure
pending
error
```

---

## Success

```yaml
state: success
```

Displayed as:

```text
✅ Success
```

Example:

```text
Docker Build Completed
```

---

## Failure

```yaml
state: failure
```

Displayed as:

```text
❌ Failure
```

Example:

```text
Unit Tests Failed
```

---

## Pending

```yaml
state: pending
```

Displayed as:

```text
⏳ Pending
```

Example:

```text
Security Scan Running
```

---

## Error

```yaml
state: error
```

Displayed as:

```text
⚠ Error
```

Example:

```text
Status API Failure
```

---

# Context Input

```yaml
context:
```

Defines the name of the status check.

Examples:

```text
UNIT_TESTS
DOCKER_BUILD
SONAR_SCAN
TRIVY_SCAN
DEV_DEPLOY
PROD_DEPLOY
```

Displayed in GitHub UI.

---

# Description Input

```yaml
description:
```

Optional human-readable message.

Example:

```yaml
description: Docker image pushed successfully
```

Displayed below the status check.

---

# Runs Section

```yaml
runs:
```

Defines how the action executes.

---

# Composite Execution

```yaml
using: composite
```

Tells GitHub:

```text
Execute a series of workflow steps
```

instead of:

```text
Docker Action
JavaScript Action
```

---

# Step: Update Commit Status

```yaml
- name: Update commit status
```

Single step responsible for calling GitHub API.

---

# Shell

```yaml
shell: bash
```

Executes commands using Bash.

---

# GitHub Status API

The action sends a request to:

```text
https://api.github.com/repos/{owner}/{repo}/statuses/{sha}
```

Purpose:

```text
Create or update commit status
```

---

# Authentication

```bash
Authorization: Bearer ${{ github.token }}
```

Uses GitHub's built-in token.

Benefits:

- No Personal Access Token required
- Secure
- Automatically managed by GitHub

---

# Content Type

```bash
Content-Type: application/json
```

Tells GitHub API:

```text
Request body is JSON
```

---

# JSON Payload

```json
{
  "state": "success",
  "context": "DOCKER_BUILD",
  "description": "Docker image pushed successfully"
}
```

Fields:

| Field | Purpose |
|---------|---------|
| state | Success / Failure |
| context | Check name |
| description | Status message |

---

# API URL

```bash
https://api.github.com/repos/${{ github.repository }}/statuses/${{ inputs.sha }}
```

Example:

```text
https://api.github.com/repos/daws-86s/catalogue/statuses/7d9b4a1
```

This attaches the status to a specific commit.

---

# Example Usage

```yaml
- name: Mark Build Pending

  uses: ./.github/actions/update-status

  with:
    sha: ${{ github.sha }}
    state: pending
    context: DOCKER_BUILD
    description: Docker build started
```

---

# Success Example

```yaml
- name: Mark Build Success

  uses: ./.github/actions/update-status

  with:
    sha: ${{ github.sha }}
    state: success
    context: DOCKER_BUILD
    description: Docker image pushed successfully
```

GitHub UI:

```text
DOCKER_BUILD  ✅
Docker image pushed successfully
```

---

# Failure Example

```yaml
- name: Mark Build Failure

  uses: ./.github/actions/update-status

  with:
    sha: ${{ github.sha }}
    state: failure
    context: DOCKER_BUILD
    description: Docker build failed
```

GitHub UI:

```text
DOCKER_BUILD ❌
Docker build failed
```

---

# Real DevOps Use Cases

## Docker Build Status

```text
DOCKER_BUILD
```

Track image build progress.

---

## SonarQube Status

```text
SONAR_SCAN
```

Track code quality checks.

---

## Security Scan Status

```text
TRIVY_SCAN
```

Track vulnerability scanning.

---

## Deployment Status

```text
DEV_DEPLOY
UAT_DEPLOY
PROD_DEPLOY
```

Track environment deployments.

---

# CI/CD Flow Example

```text
Code Push
    ↓
Build Started
    ↓
Status = Pending
    ↓
Build Completed
    ↓
Status = Success
    ↓
Deployment Started
    ↓
Status = Pending
    ↓
Deployment Completed
    ↓
Status = Success
```

---

# Best Practices

✅ Use meaningful context names

✅ Update status at every major stage

✅ Use pending before execution

✅ Use success or failure after completion

✅ Reuse through composite actions

✅ Integrate with branch protection rules

---

# Benefits

- Better CI/CD visibility
- Custom status checks
- Pull Request integration
- Branch protection support
- Reusable action design
- Enterprise-grade pipeline reporting

---

# Why This Action Is Important

This Composite Action demonstrates advanced GitHub Actions concepts:

- Reusable Actions
- GitHub Status API
- Commit Status Checks
- CI/CD Visibility
- Pipeline Governance

These concepts are widely used in:

- DevOps Engineering
- Platform Engineering
- GitOps Workflows
- Enterprise CI/CD Pipelines
- Production Software Delivery