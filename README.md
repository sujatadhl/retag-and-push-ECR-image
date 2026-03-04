# 🐳 Retag & Push ECR Image

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Retag%20%26%20Push%20ECR%20Image-blue?logo=github)](https://github.com/marketplace/actions/retag-push-ecr-image)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> Pull an existing Amazon ECR image by tag, apply a new tag, and push it back — **no re-build required**.

Perfect for **promotion workflows**: build once, tag it to `dev`, promote to `staging`, then to `production` by simply retagging.

---

## ✨ Features

- ✅ Pull any existing ECR image by tag
- ✅ Apply one or more new tags
- ✅ Optionally push a `latest` tag simultaneously
- ✅ Works with OIDC (keyless) and IAM key-based auth
- ✅ Outputs the full image URI for downstream steps
- ✅ Clean job summary in GitHub UI

---

## 🚀 Usage

### Basic example

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
    aws-region: us-east-1

- name: Retag ECR image
  uses: your-github-username/retag-ecr-action@v1
  with:
    source_tag: "sha-abc1234"
    new_tag:    "v1.2.3"
    repository: "my-app"
```

### Promote to production on Git tag push

```yaml
name: Promote to Production

on:
  push:
    tags:
      - "v*.*.*"

jobs:
  promote:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Retag image for production
        uses: your-github-username/retag-ecr-action@v1
        with:
          source_tag:  "sha-${{ github.sha }}"
          new_tag:     "${{ github.ref_name }}"   # e.g. v1.2.3
          repository:  "my-app"
          push_latest: "true"
```

### Manual retag via workflow_dispatch

```yaml
name: Manual ECR Retag

on:
  workflow_dispatch:
    inputs:
      source_tag:
        description: "Source tag"
        required: true
      new_tag:
        description: "New tag"
        required: true

jobs:
  retag:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - uses: your-github-username/retag-ecr-action@v1
        with:
          source_tag: ${{ inputs.source_tag }}
          new_tag:    ${{ inputs.new_tag }}
          repository: "my-app"
```

---

## 📥 Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `source_tag` | ✅ | — | Existing ECR image tag to pull |
| `new_tag` | ✅ | — | New tag to apply and push |
| `repository` | ✅ | — | ECR repository name (e.g. `my-app`) |
| `aws_region` | ❌ | `us-east-1` | AWS region of the ECR registry |
| `push_latest` | ❌ | `false` | Also push a `latest` tag |

## 📤 Outputs

| Output | Description |
|---|---|
| `image` | Full image URI — `<registry>/<repo>:<new_tag>` |
| `registry` | ECR registry URI (without repo/tag) |

---

## 🔐 AWS IAM Permissions

Your role needs these ECR permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchGetImage",
        "ecr:PutImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload",
        "ecr:BatchCheckLayerAvailability"
      ],
      "Resource": "*"
    }
  ]
}
```

### OIDC trust policy (recommended)

```json
{
  "Effect": "Allow",
  "Principal": {
    "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
  },
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringLike": {
      "token.actions.githubusercontent.com:sub": "repo:<YOUR_ORG>/<YOUR_REPO>:*"
    }
  }
}
```

---

## 📄 License

MIT © sujatadhl