ECS Secrets Manager Integration
Objective

Remove sensitive information from the application and securely retrieve secrets from AWS Secrets Manager during runtime.

Why Secrets Must Not Be Stored in Code

Many beginners place secrets directly inside applications.

Example:

DATABASE_PASSWORD=my-password
JWT_SECRET=my-secret-key
API_KEY=my-api-key

Or inside a Dockerfile:

ENV DATABASE_PASSWORD="mypassword"

Or inside source code:

const password = "mypassword";

This is a serious security risk.

If someone gains access to:

GitHub repository
Docker image
CI/CD pipeline logs
Source code
Backup files

they can immediately obtain application credentials.

Never Store Secrets In
Dockerfile

❌ Bad

ENV DATABASE_PASSWORD="mypassword"
Source Code

❌ Bad

const jwtSecret = "super-secret";
Git Repository

❌ Bad

config.env
.env
secrets.txt
credentials.json
GitHub Actions

❌ Bad

env:
  DATABASE_PASSWORD: mypassword
Docker Image

❌ Bad

Docker Image
      ↓
Contains Secrets
      ↓
Anyone With Image Access Can Read Them
Common Application Secrets

Examples:

DATABASE_PASSWORD
DATABASE_USERNAME
API_KEY
JWT_SECRET
SECRET_KEY
ACCESS_TOKEN
SMTP_PASSWORD
THIRD_PARTY_API_TOKEN

All should be protected.

Production Architecture

Instead of storing secrets inside the container:

Application
      ↓
Reads Secret
      ↓
AWS Secrets Manager

Architecture:

ECS Task
   │
   ▼
Secrets Manager
   │
   ▼
Secret Value
   │
   ▼
Application
What is AWS Secrets Manager?

AWS Secrets Manager is a managed service used to securely store sensitive values.

Examples:

Database Passwords
API Keys
OAuth Tokens
JWT Secrets
Certificates

Benefits:

Encryption
Centralized management
Access control
Secret rotation
Audit logging
How Secrets Manager Works

Step 1:

Create secret.

DATABASE_PASSWORD

Stored securely inside AWS.

Step 2:

Grant ECS permission to read the secret.

ECS Task Role
      ↓
Read Secret

Step 3:

ECS injects secret into container.

Secrets Manager
      ↓
ECS Task
      ↓
Environment Variable
      ↓
Application
Architecture Comparison
Insecure Method
Source Code
     ↓
Docker Image
     ↓
GitHub
     ↓
Production

Risk:

Secret Exposed Everywhere
Secure Method
Secrets Manager
       ↓
IAM Role
       ↓
ECS Task
       ↓
Application

Benefit:

Secret Never Stored In Code
Example Secret Creation

Navigate:

AWS Console
 ↓
Secrets Manager
 ↓
Store New Secret

Example:

Secret Name:
prod/database/password

Secret Value:

MyStrongPassword123!

Save.

AWS generates an ARN.

Example:

arn:aws:secretsmanager:ap-south-1:123456789012:secret:prod/database/password
ECS Task Definition Integration

Instead of:

{
  "environment": [
    {
      "name": "DATABASE_PASSWORD",
      "value": "mypassword"
    }
  ]
}

Use:

{
  "secrets": [
    {
      "name": "DATABASE_PASSWORD",
      "valueFrom": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:prod/database/password"
    }
  ]
}

ECS retrieves the value securely during startup.

Example Task Definition
{
  "family": "devops-training-task",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "myapp:v1.0.0",
      "secrets": [
        {
          "name": "DATABASE_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:prod/database/password"
        },
        {
          "name": "JWT_SECRET",
          "valueFrom": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:prod/jwt/secret"
        }
      ]
    }
  ]
}
IAM Permissions Required

ECS must have permission to read secrets.

Example Policy:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "*"
    }
  ]
}

Production environments should restrict access to specific secret ARNs.

Application Usage

The application reads the value like a normal environment variable.

Node.js Example:

const dbPassword = process.env.DATABASE_PASSWORD;

Python Example:

import os

db_password = os.getenv("DATABASE_PASSWORD")

The application does not need to know where the secret came from.

Secret Rotation

Production systems rotate credentials periodically.

Example:

Old Password
      ↓
Rotation
      ↓
New Password

Benefits:

Reduced exposure risk
Better compliance
Improved security posture

Secrets Manager supports automatic rotation for supported services.

Monitoring and Auditing

AWS records secret access through CloudTrail.

Example:

Who Accessed Secret?
When?
From Which Service?

Benefits:

Security visibility
Compliance support
Audit trail
Deployment Flow
Developer
      ↓
GitHub
      ↓
GitHub Actions
      ↓
Amazon ECR
      ↓
ECS Deployment
      ↓
Task Starts
      ↓
Read Secrets
      ↓
Application Running

No secret is stored inside:

Git repository
Docker image
Dockerfile
GitHub workflow
Source code
Verification Checklist

Check application configuration:

DATABASE_PASSWORD
API_KEY
SECRET_KEY
JWT_SECRET

Verify:

✅ Not in source code

✅ Not in Dockerfile

✅ Not in Git repository

✅ Not in GitHub Actions workflow

✅ Stored in Secrets Manager

✅ Retrieved by ECS Task

✅ IAM permissions configured

Production Best Practices
Store all secrets in AWS Secrets Manager.
Use IAM roles instead of hardcoded credentials.
Grant least-privilege access.
Rotate secrets regularly.
Enable CloudTrail auditing.
Never commit secrets to Git.
Never place secrets inside Docker images.
Use separate secrets for dev, test, and production environments.
Conclusion

Production ECS applications should never store passwords, API keys, JWT secrets, or other sensitive values inside source code, Dockerfiles, repositories, or container images. AWS Secrets Manager provides a secure, centralized solution where ECS tasks retrieve secrets at runtime using IAM permissions. This approach improves security, supports auditing and rotation, and follows production-grade cloud security best practices.