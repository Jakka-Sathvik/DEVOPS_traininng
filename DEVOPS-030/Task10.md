Day 30 – AWS KMS & Encryption Fundamentals
Introduction

Data security is one of the most important responsibilities in cloud computing. Organizations store sensitive information such as:

Customer Data
Database Credentials
API Keys
Financial Records
Application Secrets
Personal Information

If this data is not protected, unauthorized users may gain access to confidential information.

AWS provides AWS Key Management Service (KMS) to help securely encrypt and protect data.

What is Encryption?

Encryption is the process of converting readable data into an unreadable format using a cryptographic key.

Without encryption:

Hello World

Anyone can read it.

After encryption:

X7#@!9Ab$Qz*

The data becomes unreadable.

Only authorized users with the correct key can decrypt and read the original data.

Basic Encryption Architecture
Data
 │
 ▼
Encryption
 │
 ▼
KMS Key
 │
 ▼
Encrypted Data
Why Encryption is Important

Encryption helps protect:

Sensitive Data
Passwords
Secrets
Database Information
Customer Records
Application Configuration

Benefits:

Improved Security
Regulatory Compliance
Data Protection
Reduced Risk of Data Theft
Types of Encryption in AWS

AWS commonly uses two types of encryption:

1. Encryption at Rest
2. Encryption in Transit
1. Encryption at Rest
What is Encryption at Rest?

Encryption at Rest protects data that is stored on disk.

Examples:

RDS Databases
EBS Volumes
S3 Buckets
EFS File Systems
Secrets Manager
CloudWatch Logs

Data remains encrypted while stored.

Architecture
Stored Data
     │
     ▼
KMS Key
     │
     ▼
Encrypted Storage
Example: RDS Encryption
Application
      │
      ▼
RDS Database
      │
      ▼
KMS Key
      │
      ▼
Encrypted Storage

When data is written to the database:

Write Data
      │
      ▼
Encrypt
      │
      ▼
Store on Disk

When data is read:

Read Data
      │
      ▼
Decrypt
      │
      ▼
Return to Application

This process happens automatically.

Example: EBS Encryption

Amazon EBS stores EC2 instance data.

EC2
 │
 ▼
EBS Volume
 │
 ▼
KMS Key

Benefits:

Protects Server Data
Protects Backups
Protects Snapshots
Example: S3 Encryption
Application
     │
     ▼
S3 Bucket
     │
     ▼
KMS Key

Files uploaded to S3 are encrypted before storage.

Benefits of Encryption at Rest
Protects Stored Data
Protects Backups
Protects Snapshots
Compliance Requirements
Automatic Encryption
2. Encryption in Transit
What is Encryption in Transit?

Encryption in Transit protects data while it moves between systems.

Examples:

User → Website
Application → Database
EC2 → AWS Services
Browser → ALB

Without encryption:

Data travels in plain text

Attackers may intercept it.

With encryption:

Data travels securely
Architecture
User
 │
 ▼
Encrypted Connection
 │
 ▼
Application
HTTPS and TLS

AWS typically uses:

HTTPS
TLS Certificates
SSL/TLS Encryption

Example:

User
 │
 ▼
HTTPS
 │
 ▼
Application Load Balancer
 │
 ▼
EC2 Application

All communication is encrypted.

Example: Database Connection
Application
      │
      ▼
TLS Encrypted Connection
      │
      ▼
RDS Database

Data remains protected while traveling across the network.

Benefits of Encryption in Transit
Protects Network Traffic
Prevents Data Snooping
Prevents Data Interception
Improves Security
Supports Compliance
What is AWS KMS?

AWS Key Management Service (KMS) is a managed service that creates, stores, protects, and manages encryption keys.

Instead of creating encryption systems manually, AWS KMS handles:

Key Creation
Key Storage
Key Rotation
Key Security
Access Control
Audit Logging
KMS Architecture
Application
      │
      ▼
Encrypted Secret/Data
      │
      ▼
AWS KMS
      │
      ▼
KMS Key
What is a KMS Key?

A KMS Key is the cryptographic key used to encrypt and decrypt data.

Think of it as:

A Secure Digital Lock

Example:

Secret Data
      │
      ▼
KMS Key
      │
      ▼
Encrypted Secret

To access the data:

Encrypted Secret
      │
      ▼
KMS Key
      │
      ▼
Original Secret
Encryption and Decryption Process
Encryption
Plain Text
     │
     ▼
KMS Key
     │
     ▼
Cipher Text

Example:

DatabasePassword123

Becomes:

A9fX!#8KqP2Lm@
Decryption
Cipher Text
     │
     ▼
KMS Key
     │
     ▼
Plain Text

Only authorized users can decrypt the data.

AWS-Managed Keys

AWS automatically creates and manages these keys.

Examples:

aws/ebs
aws/rds
aws/s3
aws/secretsmanager

Characteristics:

Managed by AWS
Automatic Maintenance
Simple Setup
No Administration Required
Architecture
AWS Service
      │
      ▼
AWS Managed Key
      │
      ▼
Encrypted Data
Customer-Managed Keys (CMKs)

Organizations can create their own KMS keys.

Advantages:

Full Control
Custom Policies
Manual Rotation Control
Cross-Account Access
Compliance Requirements
Architecture
Application
      │
      ▼
Customer Managed Key
      │
      ▼
Encrypted Data
AWS-Managed Keys vs Customer-Managed Keys
Feature	AWS-Managed Key	Customer-Managed Key
Created By	AWS	Customer
Managed By	AWS	Customer
Policy Control	Limited	Full Control
Key Rotation	AWS Managed	Configurable
Cross-Account Access	Limited	Supported
Administration Effort	Low	Higher
Best For	Simple Environments	Production Workloads
What are Key Policies?

A Key Policy controls:

Who Can Use the Key
Who Can Encrypt Data
Who Can Decrypt Data
Who Can Manage the Key

Similar to IAM policies, but specifically for KMS keys.

Example Key Policy Concept
{
  "Effect": "Allow",
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt"
  ],
  "Resource": "*"
}

Meaning:

User can encrypt and decrypt data using the KMS key.
Real-World Example

Application stores database credentials in Secrets Manager.

Application
      │
      ▼
Secrets Manager
      │
      ▼
KMS Key
      │
      ▼
Encrypted Secret

Process:

1. Secret Created
2. KMS Encrypts Secret
3. Secret Stored Securely
4. Application Requests Secret
5. KMS Decrypts Secret
6. Application Receives Secret
KMS Integration with AWS Services

AWS KMS integrates with:

Amazon S3
Amazon EBS
Amazon RDS
AWS Secrets Manager
AWS Systems Manager Parameter Store
CloudWatch Logs
Lambda
Security Best Practices
Enable Encryption by Default

For:

S3 Buckets
EBS Volumes
RDS Databases
Secrets Manager
Use Customer-Managed Keys for Production

Benefits:

Better Control
Custom Access Policies
Improved Compliance
Follow Least Privilege

Grant only:

kms:Encrypt
kms:Decrypt

when necessary.

Avoid:

kms:*

unless required.

Monitor Key Usage

Use:

CloudTrail
CloudWatch

to monitor key activity.

Rotate Keys Regularly

Benefits:

Improved Security
Reduced Risk
Compliance Support
Key Learning Summary
Encryption at Rest
Protects stored data on disks and storage services.
Encryption in Transit
Protects data moving across networks.
AWS KMS
Managed service for encryption key management.
KMS Key
Cryptographic key used for encryption and decryption.
AWS-Managed Key
Created and managed by AWS.
Customer-Managed Key
Created and controlled by the customer.
Key Policy
Defines who can use and manage a KMS key.
Conclusion

AWS KMS is the foundation of encryption in AWS. It enables organizations to securely manage encryption keys and protect sensitive information. Encryption at Rest secures stored data, while Encryption in Transit protects data moving across networks. KMS keys perform encryption and decryption operations, and Key Policies control who can use those keys. By combining KMS, proper key management, and encryption best practices, organizations can build secure, compliant, and production-ready AWS environments.