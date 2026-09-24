Day 30 – AWS Security Architecture Revision
Introduction

Security is one of the most important parts of any cloud environment. In AWS, security is not handled by a single service. Instead, multiple AWS services work together to protect users, applications, databases, and sensitive data.

In previous training sessions, we built the following application architecture:

Internet
   │
   ▼
ALB
   │
   ▼
Application EC2
   │
   ▼
RDS

This architecture allows users to access an application hosted on EC2 through an Application Load Balancer (ALB), while application data is stored in Amazon RDS.

To make the environment secure, AWS provides additional security services.

Security Architecture
                 IAM
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
       EC2       SSM     Secrets Manager
        │                   │
        │                   ▼
        │                  RDS
        │
        ▼
       KMS

Each service has a specific responsibility for protecting resources and controlling access.

Security Components Overview
Service	Purpose
IAM	Identity and Access Management
EC2 IAM Role	Secure AWS access from EC2
Systems Manager (SSM)	Secure server management
Secrets Manager	Secure storage of credentials
KMS	Encryption management
CloudTrail	Audit and activity logging
Security Groups	Network-level protection
RDS	Secure database storage
1. Who Can Access AWS?

AWS access is controlled using IAM (Identity and Access Management).

IAM determines:

Who can sign in?
Who can use AWS services?
Who can perform specific actions?

Examples:

DevOps Engineer

Can:

Create EC2
Manage IAM Roles
Deploy Infrastructure
View Logs
Developer

Can:

Deploy Application
Read Secrets
View CloudWatch Logs

Cannot:

Delete Production Database
Modify IAM Policies
Read-Only User

Can:

View Resources
View Billing
View Logs

Cannot:

Create Resources
Delete Resources
IAM Authentication Methods

Users can authenticate using:

AWS Management Console
Username
Password
MFA

Example:

sathvik-devops
Password
MFA Code
AWS CLI
aws configure

Provides:

Access Key
Secret Key

Used by administrators and automation systems.

IAM Best Practices
Enable MFA
Password
+
MFA

Example:

Google Authenticator
Microsoft Authenticator
Principle of Least Privilege

Users should only receive permissions they actually need.

Bad:

AdministratorAccess

Good:

EC2 Read Only
CloudWatch Read Only
2. What Can They Access?

IAM Policies define permissions.

Example Policy:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "*"
    }
  ]
}

This policy allows:

Read S3 Objects

But prevents:

Delete S3 Bucket
Create EC2
Modify IAM
IAM Authorization Flow
User
  │
  ▼
IAM Policy Check
  │
  ▼
Allowed ?
  │
 ├── Yes → Access Granted
 │
 └── No → Access Denied
3. How Does EC2 Authenticate With AWS?

Applications running on EC2 often need access to AWS services.

Examples:

S3
Secrets Manager
CloudWatch
SSM
SNS
Bad Practice

Storing credentials inside application code.

ACCESS_KEY="AKIA123"
SECRET_KEY="xyz123"

Problems:

Credential Leakage
Credential Rotation Issues
Security Risk
Recommended Practice

Use IAM Roles.

Architecture:

EC2
 │
 ▼
IAM Role
 │
 ▼
AWS Services
What is an IAM Role?

An IAM Role provides temporary AWS credentials automatically.

Benefits:

No Hardcoded Credentials
Automatic Rotation
More Secure
AWS Recommended
EC2 Role Example
EC2 Instance
      │
      ▼
IAM Role
      │
      ▼
Secrets Manager

Role Permissions:

{
  "Effect": "Allow",
  "Action": [
    "secretsmanager:GetSecretValue"
  ],
  "Resource": "*"
}

Now EC2 can read secrets securely.

Instance Profile

When a role is attached to EC2, AWS creates an Instance Profile.

EC2
 │
 ▼
Instance Profile
 │
 ▼
IAM Role

The EC2 instance receives temporary credentials automatically.

No manual configuration required.

4. Where Should Database Credentials Be Stored?

Applications need:

Database Username
Database Password
Database Endpoint
Bad Practice

Store credentials in:

Source Code
GitHub Repository
Docker Image
Environment Variables

Example:

DB_PASSWORD="admin123"

Security risk:

Anyone with code access can see credentials.
Recommended Practice

Store credentials in AWS Secrets Manager.

Architecture:

Secrets Manager
       │
       ▼
Database Credentials
       │
       ▼
Application EC2
Example Secret
{
  "username": "admin",
  "password": "SecurePassword123",
  "host": "training-db.ap-south-1.rds.amazonaws.com"
}
Secret Retrieval Flow
Application
     │
     ▼
IAM Role
     │
     ▼
Secrets Manager
     │
     ▼
Database Credentials

Benefits:

Secure Storage
Automatic Rotation
Centralized Management
Audit Tracking
5. How Is Data Encrypted?

AWS uses KMS (Key Management Service).

KMS manages encryption keys used by AWS services.

Encryption Overview
Data
 │
 ▼
KMS Key
 │
 ▼
Encrypted Data

Without the correct key:

Data Cannot Be Read
Encryption Types
Data At Rest

Data stored on disk.

Examples:

RDS Storage
EBS Volumes
S3 Buckets
Secrets Manager

Encrypted using:

AWS KMS
Example
RDS Database
      │
      ▼
KMS Key
      │
      ▼
Encrypted Storage

Even if storage is stolen, data remains protected.

Data In Transit

Data moving across networks.

Example:

User
 │
 ▼
HTTPS
 │
 ▼
ALB
 │
 ▼
EC2

Encryption:

SSL/TLS Certificates
HTTPS

Protects data during transmission.

KMS Architecture
Application
      │
      ▼
KMS Key
      │
      ▼
Encrypted Data
      │
      ▼
RDS / EBS / S3

Benefits:

Centralized Key Management
Automatic Key Rotation
Strong Encryption
Compliance Support
6. How Can AWS Activity Be Audited?

AWS uses CloudTrail.

CloudTrail records:

Who performed an action
When it happened
Which resource was affected
Source IP Address
CloudTrail Example

User creates an EC2 instance.

User: DevOpsAdmin
Action: RunInstances
Service: EC2
Time: 10:30 AM
IP: 192.168.1.10

CloudTrail stores this event.

Audit Flow
User
 │
 ▼
AWS API Call
 │
 ▼
CloudTrail
 │
 ▼
Audit Logs
Example Security Investigation

Question:

Who deleted an S3 bucket?

Check:

CloudTrail Logs

Find:

Username
Time
IP Address
API Call
Systems Manager (SSM)

AWS Systems Manager helps manage EC2 instances securely.

Traditional Method:

Internet
   │
   ▼
SSH Port 22 Open

Security Risk:

Brute Force Attacks
Credential Theft
Recommended Method
Administrator
      │
      ▼
AWS Console
      │
      ▼
SSM Session Manager
      │
      ▼
EC2 Instance

Benefits:

No SSH Keys
No Open Port 22
Session Logging
Improved Security
Complete Security Architecture
                     IAM
                      │
      ┌───────────────┼───────────────┐
      ▼               ▼               ▼
   EC2 Role          SSM       Secrets Manager
      │                               │
      ▼                               ▼
     KMS ---------------------------> RDS
      │
      ▼
Encrypted Data

CloudTrail
      │
      ▼
Audit Logs
Real-World Security Flow
User
 │
 ▼
ALB
 │
 ▼
EC2 Application
 │
 ▼
IAM Role
 │
 ▼
Secrets Manager
 │
 ▼
Database Credentials
 │
 ▼
RDS

KMS Encrypts Data

CloudTrail Records Activity

SSM Provides Secure Access
Key Learning Summary
IAM
Controls who can access AWS.
IAM Policies
Control what users can access.
IAM Roles
Allow EC2 to securely access AWS services.
Secrets Manager
Stores database credentials securely.
KMS
Encrypts data at rest and supports key management.
SSM
Provides secure EC2 administration without SSH.
CloudTrail
Records and audits AWS activity.
Conclusion

AWS security is built using multiple layers. IAM controls identities and permissions, IAM Roles provide secure authentication for EC2, Secrets Manager protects sensitive credentials, KMS encrypts data, SSM enables secure administration, and CloudTrail records all important actions. Together, these services create a secure, auditable, and production-ready AWS environment that follows AWS security best practices and the principle of least privilege.