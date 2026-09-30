AWS Secrets Manager Fundamentals
Objective

Learn how AWS Secrets Manager securely stores sensitive application information and understand how secrets are used in modern ECS production environments.

What is AWS Secrets Manager?

AWS Secrets Manager is a managed AWS service used to securely store and manage sensitive information.

Examples:

Database Passwords
API Keys
JWT Secrets
OAuth Tokens
Application Credentials
Third-Party Service Tokens

Instead of storing secrets inside source code or configuration files, applications retrieve them securely from Secrets Manager.

Why Secrets Manager Is Important

Without Secrets Manager:

Source Code
     ↓
Password Stored
     ↓
Git Repository
     ↓
Security Risk

Example:

const password = "mypassword";

Problems:

Password exposed in source code
Password stored in Git history
Difficult to rotate credentials
Security compliance issues
Secure Architecture

Using AWS Secrets Manager:

Application
      ↓
ECS Task
      ↓
IAM Role
      ↓
Secrets Manager
      ↓
Secret Value

Benefits:

Centralized secret storage
Encryption
Access control
Auditing
Secret rotation
Training Secret

Secret Name:

devops-training/database

Secret Value:

{
  "username": "appuser",
  "password": "TRAINING_PASSWORD",
  "database": "trainingdb"
}

This single secret stores multiple related values.

Creating the Secret
Step 1: Open AWS Secrets Manager

Navigate:

AWS Console
    ↓
Secrets Manager
Step 2: Store a New Secret

Select:

Store a New Secret
Step 3: Choose Secret Type

Select:

Other Type of Secret
Step 4: Enter Secret Values

Example:

{
  "username": "appuser",
  "password": "TRAINING_PASSWORD",
  "database": "trainingdb"
}
Step 5: Configure Secret Name

Enter:

devops-training/database

Description:

Training database credentials for ECS application
Step 6: Save Secret

Select:

Next
 ↓
Next
 ↓
Store

AWS creates the secret.

Secret Architecture
AWS Secrets Manager
        │
        ▼
devops-training/database
        │
        ├── username
        ├── password
        └── database
Understanding Secret ARN

Every secret receives a unique Amazon Resource Name (ARN).

Example:

arn:aws:secretsmanager:ap-south-1:123456789012:secret:devops-training/database-AbCdEf

ARN components:

arn
 │
 ├── aws
 ├── secretsmanager
 ├── region
 ├── account-id
 └── secret-name

Purpose:

Unique identification
IAM permission control
ECS integration
API access
Understanding Secret Versions

Secrets can have multiple versions.

Example:

Version 1
     ↓
Password = TRAINING_PASSWORD

Later:

Version 2
     ↓
Password = NEW_PASSWORD

Architecture:

Secret
   │
   ├── Version 1
   │
   ├── Version 2
   │
   └── Version 3

Benefits:

Credential updates
Rollback support
Audit history
Version Labels

AWS uses labels to identify active versions.

Example:

AWSCURRENT

Current production secret.

Previous version:

AWSPREVIOUS

Architecture:

Version 1
   ↓
AWSPREVIOUS

Version 2
   ↓
AWSCURRENT
Secret Rotation Concept

Secret rotation means changing credentials automatically.

Example:

Old Password
      ↓
Rotation
      ↓
New Password

Benefits:

Reduced exposure risk
Better security
Compliance support
Automated credential management

Architecture:

Secrets Manager
        ↓
Rotate Secret
        ↓
Update Password
        ↓
Application Uses New Secret
Secret Access Permissions

Not every AWS service should access every secret.

Access should follow the principle of least privilege.

Architecture:

ECS Task
     ↓
IAM Role
     ↓
Permission
     ↓
Secrets Manager
Example IAM Permission
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

Purpose:

Read secret value
Restrict access
Improve security
ECS Integration Example

Instead of:

{
  "environment": [
    {
      "name": "DATABASE_PASSWORD",
      "value": "TRAINING_PASSWORD"
    }
  ]
}

Use:

{
  "secrets": [
    {
      "name": "DATABASE_SECRET",
      "valueFrom": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:devops-training/database-AbCdEf"
    }
  ]
}

ECS retrieves the secret securely during startup.

Verifying the Secret

Navigate:

AWS Console
 ↓
Secrets Manager

Expected:

devops-training/database

Open the secret and verify:

Secret Name
Secret ARN
Current Version
Last Modified Date

You should see:

{
  "username": "appuser",
  "password": "TRAINING_PASSWORD",
  "database": "trainingdb"
}
Monitoring and Auditing

AWS CloudTrail records secret access.

Example:

Who Accessed Secret?
When?
Which Service?

Benefits:

Security monitoring
Compliance auditing
Investigation support
Production Best Practices
Do

✅ Store all credentials in Secrets Manager

✅ Use IAM roles

✅ Enable rotation where appropriate

✅ Restrict access using least privilege

✅ Audit access through CloudTrail

✅ Separate development and production secrets

Do Not

❌ Store secrets in source code

❌ Store secrets in Dockerfiles

❌ Commit secrets to Git repositories

❌ Share secrets through email or chat

❌ Grant broad access to all AWS services

Training Outcome

After completing this task, you should have:

AWS Console
    ↓
Secrets Manager
    ↓
devops-training/database

Containing:

{
  "username": "appuser",
  "password": "TRAINING_PASSWORD",
  "database": "trainingdb"
}

And understand:

Secret
Secret Version
Secret ARN
Secret Rotation
Secret Access Permissions
Conclusion

AWS Secrets Manager provides a secure and centralized solution for managing sensitive information. By storing database credentials, API keys, and application secrets outside of source code and container images, organizations improve security, simplify secret rotation, enable auditing, and follow production-grade cloud security best practices.