TASK 8 — Configure GitHub Actions → AWS Authentication

Time: 50 Minutes

Objective

Configure GitHub Actions to securely authenticate with AWS using OIDC (OpenID Connect).

This approach eliminates the need to store long-lived AWS access keys inside GitHub.

Why Not Use AWS Access Keys?

Avoid storing:

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY

inside:

GitHub Secrets
Docker Images
Application Code
Git Repositories

Problems:

Key Rotation
Secret Leakage
Credential Theft
Long-Lived Access

Traditional architecture:

GitHub Actions
      │
      ▼
Stored AWS Keys
      │
      ▼
AWS

Modern architecture:

GitHub Actions
      │
      ▼
OIDC Token
      │
      ▼
IAM Role
      │
      ▼
Temporary Credentials
      │
      ▼
AWS
What is OIDC?

OIDC (OpenID Connect) allows GitHub Actions to prove its identity to AWS.

Instead of sending permanent credentials:

Username
Password
Access Key
Secret Key

GitHub provides:

Temporary Identity Token

AWS verifies that token and issues temporary credentials.

Authentication Flow
GitHub Actions
      │
      ▼
OIDC Token
      │
      ▼
AWS IAM
      │
      ▼
Assume IAM Role
      │
      ▼
Temporary AWS Credentials
      │
      ▼
Amazon ECR

Benefits:

No stored AWS keys
Short-lived credentials
Improved security
Easier management
Least privilege access
Architecture Overview
GitHub Repository
       │
       ▼
GitHub Actions
       │
       ▼
OIDC Provider
       │
       ▼
AWS IAM Role
       │
       ▼
Temporary Credentials
       │
       ▼
Amazon ECR
Step 1 — Create AWS OIDC Provider

AWS must trust GitHub's OIDC service.

Open:

AWS Console
   │
   ▼
IAM
   │
   ▼
Identity Providers

Click:

Add Provider

Provider Type:

OpenID Connect

Provider URL:

https://token.actions.githubusercontent.com

Audience:

sts.amazonaws.com

Result:

AWS
 │
 ▼
GitHub OIDC Provider
Step 2 — Create IAM Role

Navigate:

IAM
 │
 ▼
Roles
 │
 ▼
Create Role

Select:

Web Identity

Identity Provider:

token.actions.githubusercontent.com

Audience:

sts.amazonaws.com

Role Name:

GitHubActionsECRRole

Architecture:

GitHub Actions
      │
      ▼
GitHubActionsECRRole
Step 3 — Configure Trust Relationship

This determines which GitHub repositories may assume the role.

Example trust policy:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_GITHUB_USERNAME/YOUR_REPOSITORY:*"
        }
      }
    }
  ]
}
Understanding Repository Restriction

Important section:

repo:YOUR_GITHUB_USERNAME/YOUR_REPOSITORY:*

Example:

repo:sathvik/devops-training:*

Meaning:

Only This Repository
Can Assume This Role

Without restrictions:

Any GitHub Repository

could potentially attempt authentication.

Step 4 — Create ECR Permission Policy

The role requires ECR access.

Create a policy similar to:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:CompleteLayerUpload",
        "ecr:InitiateLayerUpload",
        "ecr:PutImage",
        "ecr:UploadLayerPart",
        "ecr:BatchGetImage"
      ],
      "Resource": "*"
    }
  ]
}

Purpose:

Push Docker Images
To Amazon ECR
Step 5 — Restrict Access to Repository

A better production approach is restricting access to a specific ECR repository.

Example:

devops-training-app

Instead of:

Resource: *

use:

arn:aws:ecr:ap-south-1:ACCOUNT_ID:repository/devops-training-app

Architecture:

IAM Role
    │
    ▼
Only One ECR Repository

This follows least privilege principles.

Step 6 — Attach Policy to Role

Attach the ECR policy to:

GitHubActionsECRRole

Result:

GitHub Actions
      │
      ▼
OIDC
      │
      ▼
GitHubActionsECRRole
      │
      ▼
ECR Permissions
Step 7 — Copy Role ARN

After role creation, copy:

Role ARN

Example:

arn:aws:iam::123456789012:role/GitHubActionsECRRole

This value will be used in GitHub Actions.

Step 8 — Configure GitHub Actions Permissions

The workflow must allow OIDC token generation.

Add:

permissions:
  id-token: write
  contents: read

Why?

id-token: write

allows GitHub to request an OIDC token.

Step 9 — Configure AWS Credentials Action

Add:

- name: Configure AWS Credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsECRRole
    aws-region: ap-south-1

Architecture:

GitHub Actions
      │
      ▼
OIDC Token
      │
      ▼
AWS IAM Role
      │
      ▼
Temporary Credentials
Step 10 — Verify Authentication

Add a test step:

- name: Verify AWS Identity
  run: |
    aws sts get-caller-identity

Expected output:

{
  "Account": "123456789012",
  "Arn": "arn:aws:sts::123456789012:assumed-role/GitHubActionsECRRole/..."
}

Important:

assumed-role

confirms OIDC authentication is working.

Complete Workflow Example
name: Docker Build

on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsECRRole
          aws-region: ap-south-1

      - name: Verify AWS Identity
        run: |
          aws sts get-caller-identity
Authentication Sequence
Git Push
    │
    ▼
GitHub Actions
    │
    ▼
Request OIDC Token
    │
    ▼
AWS IAM
    │
    ▼
Assume Role
    │
    ▼
Temporary AWS Credentials
    │
    ▼
Amazon ECR
Security Benefits
No Long-Lived Keys

Avoid:

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
Automatic Expiration
Temporary Credentials

expire automatically.

Repository Restriction

Only approved repositories can use the role.

Least Privilege

Role receives only required ECR permissions.

Troubleshooting
Error
Not authorized to perform sts:AssumeRoleWithWebIdentity

Check:

OIDC provider exists
Trust relationship is correct
Repository name matches
Error
Could not load credentials

Check:

permissions:
  id-token: write
Error
AccessDeniedException

Check:

ECR permissions
Repository ARN
IAM policy attachment
Final Architecture
GitHub
   │
   ▼
GitHub Actions
   │
   ▼
OIDC Token
   │
   ▼
AWS IAM Role
   │
   ▼
Temporary AWS Credentials
   │
   ▼
Amazon ECR
Deliverable

GitHub Actions successfully authenticates to AWS using:

OIDC

without storing:

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY

Verification:

aws sts get-caller-identity

returns:

assumed-role/GitHubActionsECRRole

which confirms that GitHub Actions is using temporary AWS credentials obtained through OIDC.