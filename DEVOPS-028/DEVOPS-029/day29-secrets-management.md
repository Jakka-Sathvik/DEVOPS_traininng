TASK 8 — AWS Secrets Manager Concept

Training Duration: 45 Minutes
Topic: AWS Secrets Manager and Secure Database Credentials
Environment: AWS + Terraform + Amazon RDS PostgreSQL

1. Objective

The objective of this task is to understand how AWS Secrets Manager can securely store and manage sensitive information such as:

Database usernames
Database passwords
API keys
Application credentials
Tokens
Other secrets

The main principle is:

Secrets should not be hard-coded in application source code, configuration files, or Git repositories.

Instead, the application can retrieve the secret securely from AWS Secrets Manager when it needs it.

2. Basic Architecture

The basic architecture for this task is:

                 AWS Secrets Manager
                         │
                         │ Secret
                         ▼
                  Application
                         │
                         │ Database Connection
                         ▼
                        RDS

For our three-tier architecture:

                         Internet
                            │
                            ▼
                           ALB
                            │
                            ▼
                    Application EC2 / ASG
                            │
                 ┌──────────┴──────────┐
                 │                     │
                 │ Retrieve Secret     │
                 ▼                     │
          AWS Secrets Manager          │
                 │                     │
                 └─────────────────────┘
                            │
                            ▼
                    PostgreSQL RDS

The application does not need the database password hard-coded into its source code.

3. What Is AWS Secrets Manager?

AWS Secrets Manager is an AWS service used to securely store and manage secrets.

A secret can contain information such as:

Username
Password
Database endpoint
API key
Token
Credentials

For example, a database secret could conceptually contain:

{
  "username": "dbadmin",
  "password": "********",
  "engine": "postgres",
  "host": "training-postgres.xxxxx.ap-south-1.rds.amazonaws.com",
  "port": 5432,
  "dbname": "trainingdb"
}

The actual password should not be displayed or copied unnecessarily.

4. Why Do We Need Secrets Manager?

Consider an application with this configuration:

application
     │
     └── config.py
            │
            └── DB_PASSWORD = "MyPassword123"

This is unsafe.

The code may be stored in:

GitHub
GitLab
Bitbucket
Developer laptop
CI/CD workspace
Container image

Anyone who obtains the source code may potentially obtain the database password.

5. Unsafe Architecture
Developer
    │
    ▼
Application Source Code
    │
    ├── DB_USERNAME
    ├── DB_PASSWORD
    └── DB_HOST
    │
    ▼
Git Repository

The password is now part of the application's configuration.

This creates unnecessary exposure.

6. Secure Architecture

Instead:

Developer
    │
    ▼
Application Code
    │
    │ No password
    │
    ▼
Git Repository


AWS Secrets Manager
        │
        │ Database credentials
        ▼
   Application
        │
        │ Database connection
        ▼
       RDS

The application knows how to retrieve the secret, rather than having the secret embedded in its source code.

7. Secret Storage

Secrets Manager stores the secret in AWS.

Conceptually:

AWS Account
│
└── Secrets Manager
       │
       └── training/database
              │
              ├── Username
              ├── Password
              ├── Host
              ├── Port
              └── Database name

The secret can be protected using AWS access controls.

8. Secret Name

A secret can have a meaningful name.

Example:

training/database

or:

dev/rds/postgres

or:

production/application/database

A useful naming convention makes secrets easier to identify.

Example:

Environment / Application / Purpose
training / restaurant-api / database
9. Secret Value

The secret value can contain database connection information.

Example:

{
  "username": "dbadmin",
  "password": "TEMPORARY-PASSWORD",
  "engine": "postgres",
  "host": "RDS-ENDPOINT",
  "port": 5432,
  "dbname": "trainingdb"
}

For training, understand the structure.

Do not copy real credentials into documentation.

10. Secret Retrieval

The application needs permission to retrieve the secret.

The general flow is:

Application EC2
      │
      │ IAM permission
      ▼
Secrets Manager
      │
      │ Secret value
      ▼
Application
      │
      │ Username + Password
      ▼
RDS

The application does not need to be given unrestricted AWS permissions.

11. IAM Permissions

IAM controls who or what can access the secret.

For example:

EC2 Application
      │
      │ IAM Role
      ▼
Permission:
secretsmanager:GetSecretValue
      │
      ▼
Specific Secret

The application should receive only the permissions it needs.

12. Least Privilege

The application should not receive:

AdministratorAccess

just because it needs one database secret.

Instead, give it permission to retrieve the required secret.

Conceptually:

EC2 Application Role
        │
        └── secretsmanager:GetSecretValue
                │
                └── training/database

This follows the principle of:

Least privilege

13. Example IAM Policy Concept

A simplified example is:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "arn:aws:secretsmanager:REGION:ACCOUNT_ID:secret:training/database-*"
    }
  ]
}

This is an example for understanding the concept.

In an actual environment, replace the placeholders with the appropriate resource information and use the narrowest possible resource scope.

14. Why IAM Role Is Preferred for EC2

Suppose an EC2 instance needs access to Secrets Manager.

Avoid putting AWS access keys directly into:

Application source code
Environment files
Git
User data
Configuration files

Instead:

EC2
 │
 └── IAM Instance Role
          │
          ▼
   Secrets Manager

The EC2 instance can obtain temporary AWS credentials through its attached IAM role.

15. Application Authentication

The application must authenticate to AWS before retrieving the secret.

For an EC2-based application:

EC2
 │
 ▼
IAM Instance Role
 │
 ▼
Temporary AWS credentials
 │
 ▼
Secrets Manager

The application SDK can use the credentials provided through the AWS environment.

16. Example Application Flow

Suppose the application needs to connect to PostgreSQL.

The application starts:

Application starts
       │
       ▼
Needs database credentials
       │
       ▼
Calls Secrets Manager
       │
       ▼
AWS verifies IAM permissions
       │
       ▼
Secret returned
       │
       ▼
Application creates DB connection
       │
       ▼
PostgreSQL RDS
17. Python Example

A Python application could use the AWS SDK for Python, commonly called boto3.

Conceptually:

import boto3
import json

client = boto3.client("secretsmanager")

response = client.get_secret_value(
    SecretId="training/database"
)

secret = json.loads(response["SecretString"])

username = secret["username"]
password = secret["password"]
host = secret["host"]
port = secret["port"]
database = secret["dbname"]

The important concept is:

Application
    ↓
Secrets Manager API
    ↓
Secret
    ↓
Database credentials

The application source code does not contain the actual password.

18. Important Security Consideration

The example contains:

password = secret["password"]

This does not mean the password is now safe to print.

Never do:

print(password)

or:

logger.info(password)

Secrets should not be written to application logs.

19. Application Database Connection

After retrieving the secret:

Secret
 │
 ├── username
 ├── password
 ├── host
 ├── port
 └── dbname
        │
        ▼
Application Database Driver
        │
        ▼
PostgreSQL RDS

For example, a PostgreSQL application may construct a connection using:

Host     → RDS endpoint
Port     → 5432
Database → trainingdb
Username → retrieved username
Password → retrieved password
20. Network Security Still Matters

Secrets Manager does not replace network security.

The architecture should still use:

ALB
 │
 ▼
Application EC2
 │
 ▼
RDS

The database should remain protected by its security group.

Example:

Application SG
      │
      │ TCP 5432
      ▼
Database SG
      │
      ▼
RDS

The database should not be opened to the entire Internet.

21. Secrets Manager vs Security Group

These solve different problems.

Security Group

Controls:

WHO can communicate
WITH WHICH resource
ON WHICH port

Example:

Application SG
      │
      │ TCP 5432
      ▼
Database SG
Secrets Manager

Controls:

WHERE credentials are stored
WHO can retrieve them
HOW applications obtain them

Therefore:

Security Group
       +
Secrets Manager
       +
IAM
       +
Private Networking

work together to protect the application and database.

22. Secret Retrieval Is Not Database Authorization

Another important distinction:

Secrets Manager

does not itself grant PostgreSQL access.

Instead:

Secrets Manager
       │
       ▼
Database username/password
       │
       ▼
PostgreSQL authentication
       │
       ▼
RDS

The PostgreSQL database still validates the credentials.

23. Rotation Concept

Secrets Manager supports secret rotation.

Rotation means changing credentials periodically.

Conceptually:

Old Password
     │
     ▼
Rotation Process
     │
     ▼
New Password
     │
     ▼
Application retrieves updated secret
     │
     ▼
RDS accepts updated credentials

This reduces the lifetime of a credential.

24. Why Rotation Is Useful

Without rotation:

Password created
      │
      │
      │ Long lifetime
      │
      ▼
Password eventually compromised

With rotation:

Password
   │
   ▼
Rotation
   │
   ▼
New Password
   │
   ▼
Rotation
   │
   ▼
Another Password

Regular credential rotation can reduce the period during which an exposed credential remains useful.

25. Today's Lab Does Not Require Rotation

For this training task:

Automatic rotation
       ↓
NOT REQUIRED

The trainee only needs to understand the concept.

Today's focus is:

Secret Storage
       ↓
IAM Permission
       ↓
Secret Retrieval
       ↓
Application
       ↓
RDS
26. Secrets Manager and Terraform

Terraform can create a Secrets Manager secret.

For example:

resource "aws_secretsmanager_secret" "database" {
  name = "training/database"

  tags = {
    Environment = "training"
  }
}

This creates the secret container.

However, handling the actual secret value through Terraform requires additional consideration.

27. Important Terraform Security Consideration

It may be tempting to write:

resource "aws_secretsmanager_secret_version" "database" {
  secret_id = aws_secretsmanager_secret.database.id

  secret_string = jsonencode({
    username = var.db_username
    password = var.db_password
  })
}

This can be useful for understanding Terraform integration, but it introduces an important issue:

The secret value can be recorded in Terraform state.

Therefore, simply using Secrets Manager through Terraform does not automatically mean the secret will never appear in Terraform state.

This is a critical concept.

28. Terraform vs Secrets Manager Responsibilities

Think of the responsibilities separately.

Terraform

Terraform manages infrastructure such as:

VPC
Subnets
Security Groups
RDS
IAM
Secrets Manager resources
Secrets Manager

Secrets Manager manages:

Secrets
Credential lifecycle
Access control
Rotation capabilities
Application

The application retrieves the secret when needed.

Application
    ↓
Secrets Manager
    ↓
Credentials
    ↓
RDS
29. Recommended Production Pattern

A production-oriented architecture can look like:

                   ┌──────────────────────┐
                   │  AWS Secrets Manager  │
                   │                      │
                   │ DB Credentials       │
                   └──────────┬───────────┘
                              │
                              │ GetSecretValue
                              │
                         IAM Role
                              │
                              ▼
                       Application EC2
                              │
                              │ PostgreSQL
                              │ 5432
                              ▼
                         Private RDS

Security controls:

IAM
 │
 ├── Application can retrieve specific secret
 │
 ▼
Secrets Manager

Security Groups
 │
 ├── ALB → Application
 │
 └── Application → RDS:5432

Network
 │
 └── RDS in private subnets
30. End-to-End Request Flow

Let's follow a real application request.

Step 1 — User sends request
User
 ↓
ALB
Step 2 — ALB forwards request
ALB
 ↓
Application EC2
Step 3 — Application needs database
Application
 ↓
Need database credentials
Step 4 — Application calls Secrets Manager
Application
 ↓
IAM Role
 ↓
Secrets Manager
Step 5 — IAM authorization

AWS checks:

Does this application role have permission?

For example:

secretsmanager:GetSecretValue
Step 6 — Secret returned
Secrets Manager
 ↓
Database credentials
 ↓
Application
Step 7 — Application connects to RDS
Application
 │
 │ TCP 5432
 ▼
RDS PostgreSQL
31. Complete Architecture
                         USER
                           │
                           ▼
                         ALB
                           │
                           ▼
                  ┌────────────────┐
                  │  Application   │
                  │     EC2        │
                  └───────┬────────┘
                          │
             ┌────────────┴────────────┐
             │                         │
             │ IAM Role                │ DB connection
             ▼                         ▼
     AWS Secrets Manager             RDS
             │                     PostgreSQL
             │
             │ Credentials
             └──────────────► Application
32. What Happens If the Application Has No IAM Permission?

Suppose:

Application
     │
     ▼
Secrets Manager

but the IAM role does not allow:

secretsmanager:GetSecretValue

The request is denied.

Conceptually:

Application
     │
     ▼
Secrets Manager
     │
     ▼
IAM Authorization
     │
     └── DENIED

The application cannot retrieve the secret.

This demonstrates why IAM is an important part of secret management.

33. What Happens If RDS Security Group Is Wrong?

Suppose the application successfully retrieves the password:

Secrets Manager
      │
      ▼
Application

but the RDS security group does not allow:

Application SG → RDS:5432

Then:

Credentials → SUCCESS
Database Network Connection → FAIL

This demonstrates that:

Credentials and network access are separate security controls.

34. What Happens If the Database Password Is Wrong?

The application may successfully retrieve a secret:

Secrets Manager
      │
      ▼
Application

but PostgreSQL can reject the credentials:

Application
      │
      ▼
RDS PostgreSQL
      │
      ▼
Authentication Failed

Therefore, troubleshooting must distinguish:

IAM problem
Secrets Manager problem
Network problem
Security Group problem
Database authentication problem
35. Common Mistakes
Mistake 1 — Hardcoded password
DB_PASSWORD = "MyPassword123"
Problem

Credential is stored in source code.

Better

Retrieve it from Secrets Manager.

Mistake 2 — Putting secret in .env

Example:

DB_PASSWORD=MyPassword123

A .env file may still expose credentials if accidentally committed or copied.

Mistake 3 — Logging the secret

Bad:

print(secret)

or:

logger.info(secret)

Never intentionally log credentials.

Mistake 4 — Giving AdministratorAccess

Bad architecture:

Application
     │
     └── AdministratorAccess

The application should receive only the permissions it needs.

Mistake 5 — Opening RDS to the Internet

Avoid:

0.0.0.0/0 → TCP 5432 → RDS

The database should be protected by private networking and a restrictive security group.

Mistake 6 — Assuming Secrets Manager solves every security problem

Secrets Manager protects secret storage and retrieval, but you still need:

IAM
Security Groups
Private networking
Application security
State protection
Logging controls
36. Secrets Manager vs Hardcoded Password
Feature	Hardcoded Password	Secrets Manager
Stored in source code	Yes	No
Git exposure risk	High	Reduced
Centralized management	No	Yes
IAM access control	No	Yes
Rotation capability	Manual	Supported
Application retrieval	Hardcoded	API/SDK
Audit capabilities	Limited	AWS-integrated
Suitable for production	No	Production-oriented
37. Secrets Manager vs Terraform Sensitive Variable

These are different concepts.

Terraform Sensitive Variable
variable "db_password" {
  type      = string
  sensitive = true
}

Purpose:

Tell Terraform the value is sensitive.

It helps prevent normal CLI output from exposing the value.

But the value may still be stored in Terraform state.

Secrets Manager

Purpose:

Securely store and manage secrets

It provides dedicated secret-management capabilities such as IAM-controlled retrieval and rotation support.

38. Important Security Principle

Do not think:

sensitive = true
       =
Secrets Manager

They are not the same.

Correct understanding:

Terraform sensitive variable
        │
        └── Terraform handling/display protection

AWS Secrets Manager
        │
        └── Dedicated secret storage and management
39. Trainee Research Exercise

Research and document this question:

How could an application securely retrieve its database credentials from AWS Secrets Manager?

Your answer should include:

1. Application runs on EC2
2. EC2 has an IAM role
3. IAM role allows GetSecretValue
4. Permission is restricted to the required secret
5. Application uses AWS SDK
6. SDK calls Secrets Manager
7. Secrets Manager returns the authorized secret
8. Application uses credentials to connect to RDS
9. Password is not hard-coded
10. Password is not written to logs
40. Research Flow

Document the following architecture:

EC2 Application
      │
      ▼
IAM Instance Role
      │
      ▼
secretsmanager:GetSecretValue
      │
      ▼
AWS Secrets Manager
      │
      ▼
Database Credentials
      │
      ▼
Application
      │
      ▼
RDS PostgreSQL
41. Questions for the Trainee
Question 1

Why should database passwords not be hard-coded?

Answer:

Because application source code may be stored in version control or shared across developers and systems. Hardcoding creates unnecessary credential exposure.

Question 2

Where should production secrets be stored?

Answer:

A dedicated secret-management service such as AWS Secrets Manager can be used.

Question 3

How does an EC2 application authenticate to AWS?

Answer:

A common approach is to attach an IAM role to the EC2 instance and allow the application to use the role's temporary credentials.

Question 4

What permission does the application need?

Conceptually:

secretsmanager:GetSecretValue

The permission should be scoped to the required secret.

Question 5

Does Secrets Manager automatically give access to RDS?

Answer:

No.

Secrets Manager provides the credential. The application still needs network access to RDS and the database credentials must be valid.

Question 6

Does IAM replace the RDS security group?

Answer:

No.

IAM controls AWS API authorization, while the RDS security group controls network traffic to the database.

Question 7

Does Secrets Manager replace security groups?

Answer:

No.

They protect different parts of the architecture.

Question 8

What is secret rotation?

Answer:

Rotation is the process of periodically changing a secret, such as a database password, and updating the systems that use it.

42. Today's Lab Scope

Today's task does not require:

Automatic secret rotation
Complex application deployment
Production secret lifecycle
Advanced CI/CD integration

Focus on understanding:

Secret Storage
       ↓
IAM
       ↓
Secret Retrieval
       ↓
Application
       ↓
RDS
43. Final Secure Design

The desired design is:

                    AWS
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
Secrets Manager                 RDS
        │                     PostgreSQL
        │                         ▲
        │                         │
        ▼                         │
   Application EC2 ───────────────┘
        │
        │ IAM Role
        ▼
    AWS APIs

Application source code:

NO DATABASE PASSWORD
NO AWS SECRET KEY
NO HARDCODED CREDENTIAL

Instead:

Application
    ↓
IAM Role
    ↓
Secrets Manager
    ↓
Retrieve secret
    ↓
Connect to RDS
44. Security Checklist

Before considering the task complete:

Understand what AWS Secrets Manager is

Understand secret storage

Understand secret retrieval

Understand IAM permissions

Understand EC2 IAM roles

Understand least privilege

Understand application authentication to AWS

Understand secret rotation concept

Understand why passwords must not be hard-coded

Understand why passwords must not be logged

Understand RDS security groups

Understand private database networking

Understand Secrets Manager does not replace security groups

Understand Secrets Manager does not replace IAM

Understand Terraform sensitive is different from Secrets Manager

Understand Terraform can still record secrets in state

Understand today's lab does not require automatic rotation

45. Final Summary

The insecure architecture is:

Application
     │
     └── Hardcoded Password
              │
              ▼
        Git Repository

The improved architecture is:

                 AWS Secrets Manager
                         │
                         │ GetSecretValue
                         ▼
                    Application
                         │
                         │ Username +
                         │ Password
                         ▼
                    RDS PostgreSQL

With IAM:

Application EC2
      │
      ▼
   IAM Role
      │
      ▼
GetSecretValue
      │
      ▼
Secrets Manager

With network security:

ALB
 │
 ▼
Application EC2
 │
 │ TCP 5432
 ▼
Database SG
 │
 ▼
Private RDS

The complete security model is therefore:

              ┌─────────────────────┐
              │   Secrets Manager   │
              │   Secret Storage    │
              └──────────┬──────────┘
                         │
                         ▼
                    IAM Role
                         │
                         ▼
                  Application EC2
                         │
                         │ 5432
                         ▼
                    Private RDS
                         │
                    Database
Key Takeaway

Do not put database credentials in application source code. Store secrets in a dedicated secret-management system and allow the application to retrieve them through tightly scoped IAM permissions.

Also remember:

Secrets Manager, IAM, security groups, private networking, and Terraform state protection solve different security problems. A secure architecture uses them together rather than relying on one mechanism alone.