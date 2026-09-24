ECS IAM Roles
Objective

Understand the difference between:

Task Execution Role

and

Task Role

These are two of the most important security concepts in Amazon ECS and AWS Fargate.

A common beginner mistake is treating them as the same thing.

They are not.

Why IAM Roles Are Needed

When containers run inside ECS, they often need access to AWS services.

Examples:

Amazon ECR
CloudWatch Logs
Amazon S3
Secrets Manager
DynamoDB
SSM Parameter Store

Instead of storing AWS credentials inside containers:

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY

AWS recommends using IAM Roles.

Architecture:

Container
    │
    ▼
IAM Role
    │
    ▼
AWS Service

Benefits:

No hardcoded credentials
Temporary credentials
Improved security
Easier permission management
Supports least privilege
Two Different Roles in ECS

ECS uses two separate IAM roles:

ECS Task
│
├── Task Execution Role
│
└── Task Role

Each role serves a different purpose.

Task Execution Role
What Is It?

The Task Execution Role is used by ECS itself.

The application inside the container does not use this role.

AWS ECS uses it to perform actions required to start and manage containers.

Responsibilities

Typical responsibilities:

Pull container images from ECR
Send logs to CloudWatch
Retrieve secrets during startup
Retrieve parameters during startup

Architecture:

ECS Service
     │
     ▼
Task Execution Role
     │
     ├── Amazon ECR
     └── CloudWatch Logs
Example Workflow

When a task starts:

ECS Task Start
      │
      ▼
Use Execution Role
      │
      ▼
Pull Image From ECR
      │
      ▼
Container Starts
      │
      ▼
Send Logs To CloudWatch

The application never directly uses this role.

Common AWS Managed Role

AWS often creates:

ecsTaskExecutionRole

Common policy:

AmazonECSTaskExecutionRolePolicy

Provides permissions for:

ECR image pull
CloudWatch logs
ECS startup operations
Task Role
What Is It?

The Task Role is used by the application running inside the container.

This role provides permissions to AWS services that the application needs.

Example

Application:

Python API

Needs:

Read files from S3

Architecture:

Application
      │
      ▼
Task Role
      │
      ▼
Amazon S3

The application receives temporary AWS credentials automatically.

Example Workflow
User Request
      │
      ▼
Application
      │
      ▼
Task Role
      │
      ▼
Amazon S3

No access keys are stored inside the container.

Execution Role vs Task Role
Feature	Task Execution Role	Task Role
Used By	ECS	Application
Pull ECR Images	Yes	No
Send CloudWatch Logs	Yes	No
Access S3	No	Yes
Access DynamoDB	No	Yes
Access Secrets Manager	Application Runtime Only Through Task Role	Yes
Needed For Task Startup	Yes	No
Needed For Application AWS Access	No	Yes
Real-World Example

Suppose your application:

Reads files from S3
Writes logs to CloudWatch
Uses container image from ECR

Architecture:

Amazon ECR
      ▲
      │
Execution Role
      │
      ▼
ECS Starts Task
      │
      ▼
Application
      │
      ▼
Task Role
      │
      ▼
Amazon S3

Responsibilities:

Execution Role
Pull Image
Send Logs
Task Role
Read S3 Objects
Principle of Least Privilege

One of the most important AWS security principles is:

Least Privilege

Meaning:

Only grant permissions that are required

Not:

Give AdministratorAccess
Bad Example

Task role:

AdministratorAccess

Problems:

Full account access
Increased security risk
Accidental resource deletion
Compliance violations

Architecture:

Application
      │
      ▼
Everything In AWS

Not recommended.

Good Example

Application only needs:

Read S3 Objects

Policy:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::training-bucket/*"
      ]
    }
  ]
}

Result:

Application
      │
      ▼
Read Only Access
      │
      ▼
Training Bucket

Much safer.

Example IAM Policy for S3 Read Access
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadTrainingFiles",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::training-bucket/*"
      ]
    }
  ]
}

This policy allows:

Read Files

but does not allow:

Delete Files
Create Buckets
Modify IAM
ECS IAM Architecture
                 ECS Service
                       │
                       ▼
                ECS Task Starts
                       │
        ┌──────────────┴──────────────┐
        ▼                             ▼

Task Execution Role             Task Role
        │                             │
        ▼                             ▼

Amazon ECR                    Application
CloudWatch Logs                    │
                                   ▼
                           AWS Services
                           (S3, DynamoDB,
                            Secrets Manager)
Security Best Practices
Use IAM Roles
✓ Recommended

instead of:

AWS Access Keys
Use Least Privilege
Only Required Permissions
Separate Roles
Execution Role
Task Role

Do not combine responsibilities unnecessarily.

Avoid AdministratorAccess
Never use AdministratorAccess
for ECS tasks unless absolutely required.
Key Takeaways
ECS uses two separate IAM roles.
The Task Execution Role is used by ECS for startup operations such as pulling images from ECR and sending logs to CloudWatch.
The Task Role is used by the application running inside the container.
Applications should never store AWS access keys.
IAM Roles provide temporary credentials automatically.
Follow the Principle of Least Privilege.
Grant only the permissions required by the application.
Avoid using broad permissions such as AdministratorAccess.
Proper IAM configuration is essential for secure ECS and Fargate deployments.