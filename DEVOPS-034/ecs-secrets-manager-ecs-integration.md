Connecting AWS Secrets Manager to ECS
Objective

Configure Amazon ECS so that sensitive information is securely retrieved from AWS Secrets Manager instead of being stored inside source code, Docker images, task definitions, or Git repositories.

Production Architecture
ECS Task
   │
   ▼
Task IAM Role
   │
   ▼
AWS Secrets Manager
   │
   ▼
Secret Value
   │
   ▼
Application

The application receives secrets securely during task startup.

Why This Is Important

Many applications require configuration values.

Examples:

ENVIRONMENT
APP_NAME
PORT
LOG_LEVEL

These are normal configuration values.

Example:

ENVIRONMENT=production

These values are generally not sensitive.

However, some values are highly sensitive.

Examples:

DATABASE_PASSWORD
API_SECRET
JWT_SECRET
ACCESS_TOKEN
SMTP_PASSWORD

These values must never be hardcoded.

Environment Variable vs Secret Injection
Environment Variable

Normal application configuration.

Example:

{
  "environment": [
    {
      "name": "ENVIRONMENT",
      "value": "production"
    }
  ]
}

Characteristics:

Non-sensitive
Visible in task definition
Safe for application configuration
Stored directly in ECS configuration
Secret Injection

Sensitive application configuration.

Example:

{
  "secrets": [
    {
      "name": "DATABASE_PASSWORD",
      "valueFrom": "SECRET_ARN"
    }
  ]
}

Characteristics:

Stored in Secrets Manager
Retrieved securely at runtime
Not hardcoded
Better security
Architecture Comparison
Incorrect Method
Source Code
     │
     ▼
Password Stored
     │
     ▼
Git Repository
     │
     ▼
Security Risk

Example:

ENV DATABASE_PASSWORD=my-password

Problems:

Secret exposed
Visible in source control
Visible in Docker image
Difficult to rotate
Correct Method
Secrets Manager
       │
       ▼
Secret
       │
       ▼
ECS Task
       │
       ▼
Application

Benefits:

Centralized secret management
Better security
Easier rotation
IAM-controlled access
Step 1: Create Secret

Example secret:

Name:

devops-training/database

Value:

{
  "username": "appuser",
  "password": "TRAINING_PASSWORD",
  "database": "trainingdb"
}

Store the secret inside AWS Secrets Manager.

Step 2: Obtain Secret ARN

Navigate:

AWS Console
 ↓
Secrets Manager
 ↓
devops-training/database

Example ARN:

arn:aws:secretsmanager:ap-south-1:123456789012:secret:devops-training/database-AbCdEf

Save this ARN.

It will be used in the ECS task definition.

Step 3: Configure IAM Permissions

ECS must have permission to read the secret.

Architecture:

ECS Task
     │
     ▼
IAM Role
     │
     ▼
Secrets Manager

Example IAM Policy:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:devops-training/database-*"
    }
  ]
}

Attach the policy to the ECS Task Role.

Step 4: Update ECS Task Definition

Old configuration:

{
  "environment": [
    {
      "name": "DATABASE_PASSWORD",
      "value": "TRAINING_PASSWORD"
    }
  ]
}

Problem:

Password Visible

New configuration:

{
  "secrets": [
    {
      "name": "DATABASE_PASSWORD",
      "valueFrom": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:devops-training/database-AbCdEf"
    }
  ]
}

ECS retrieves the value during task startup.

Example Task Definition
{
  "family": "devops-training-task",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training:v1.0.0",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 3000
        }
      ],
      "environment": [
        {
          "name": "ENVIRONMENT",
          "value": "production"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_SECRET",
          "valueFrom": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:devops-training/database-AbCdEf"
        }
      ]
    }
  ]
}
Step 5: Create New Task Definition Revision

Task definitions are versioned.

Example:

devops-training-task:1
devops-training-task:2
devops-training-task:3

Create a new revision.

Example:

Revision 1
     ↓
Environment Variables Only

Revision 2
     ↓
Secrets Manager Integration

Benefits:

Safe deployments
Rollback support
Version tracking
Step 6: Update ECS Service

Navigate:

ECS
 ↓
Cluster
 ↓
Service
 ↓
Update Service

Select:

Latest Task Definition Revision

Deploy.

Step 7: ECS Startup Process

When the task starts:

Task Starts
      │
      ▼
Assume IAM Role
      │
      ▼
Request Secret
      │
      ▼
Secrets Manager
      │
      ▼
Return Secret
      │
      ▼
Application Starts
Application Access

The application reads the value as a normal environment variable.

Node.js Example:

const dbPassword = process.env.DATABASE_PASSWORD;

Python Example:

import os

db_password = os.getenv("DATABASE_PASSWORD")

The application does not need to know where the secret originated.

Verification

After deployment:

Navigate:

ECS
 ↓
Cluster
 ↓
Tasks
 ↓
Running Task

Verify:

Task Status = RUNNING

Check CloudWatch Logs.

Expected:

Application Started Successfully
Database Connection Successful

Verify:

Secret Retrieved Successfully

No hardcoded passwords should exist in:

Source code
Docker image
Dockerfile
Git repository
GitHub Actions workflow
Troubleshooting
Access Denied

Example:

AccessDeniedException

Cause:

IAM Role Missing Permission

Solution:

Add secretsmanager:GetSecretValue
Secret Not Found

Example:

ResourceNotFoundException

Cause:

Incorrect ARN

Solution:

Verify Secret ARN
Task Fails to Start

Possible causes:

Incorrect Secret ARN
Missing IAM Permission
Deleted Secret
Invalid Task Definition

Review ECS Events and CloudWatch Logs.

Expected Production Flow
Developer
      │
      ▼
GitHub
      │
      ▼
GitHub Actions
      │
      ▼
Amazon ECR
      │
      ▼
ECS Deployment
      │
      ▼
Task Starts
      │
      ▼
Retrieve Secret
      │
      ▼
Application Running

No secrets are stored inside:

Source code
Git repository
Docker image
Dockerfile
CI/CD pipeline
Success Criteria

✅ Secret created in AWS Secrets Manager

✅ Secret ARN obtained

✅ IAM permission configured

✅ ECS task definition updated

✅ New revision created

✅ ECS service updated

✅ Task starts successfully

✅ Secret retrieved securely

✅ Application runs successfully

Conclusion

Amazon ECS integrates directly with AWS Secrets Manager, allowing applications to securely retrieve passwords, API keys, and other sensitive values during startup. By using IAM permissions and secret injection, organizations eliminate hardcoded credentials, improve security, simplify secret rotation, and follow production-grade cloud security best prac