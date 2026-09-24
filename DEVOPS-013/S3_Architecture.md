TASK 2 — Understand S3 Architecture

Training Area: AWS / DevOps
Time: 45 Minutes
Deliverable: S3_Architecture.md
Status: Completed

1. Objective

The objective of this task is to understand the basic architecture of Amazon S3 (Simple Storage Service) and how the following components work together:

Bucket
Object
Object Key
Region
Storage
Permissions

The main S3 architecture is:

                       AWS
                        │
                       S3
                        │
                     Bucket
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
       Object         Object        Object
       image.jpg      index.html    file.pdf
2. What is Amazon S3?

Amazon S3 is an AWS service used to store and retrieve data as objects.

S3 is an object storage service.

Instead of managing a traditional server disk, applications can store files in S3 buckets and access them through AWS APIs, the AWS Console, AWS CLI, SDKs, or automation tools such as Terraform.

Basic architecture:

AWS
 │
 └── S3
      │
      └── Bucket
           │
           ├── Object
           ├── Object
           └── Object
3. S3 Architecture Diagram

The basic architecture required for this task is:

                       AWS
                        │
                       S3
                        │
                     Bucket
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
       Object         Object        Object
       image.jpg      index.html    file.pdf

For example, a bucket might contain:

devops-training-bucket
│
├── image.jpg
├── index.html
└── file.pdf

Each file is stored as an object.

4. Bucket
Definition

A bucket is a container in Amazon S3 that stores objects.

Think of a bucket as the top-level storage container for S3 data.

Example:

S3
 │
 └── devops-training-bucket

The bucket can contain many objects:

devops-training-bucket
│
├── image.jpg
├── index.html
├── file.pdf
├── logs/app.log
└── backups/database.sql
Important Points

A bucket:

Stores objects
Has a bucket name
Is created in an AWS Region
Can have security and access controls
Can have lifecycle and versioning configurations
Can contain objects with different keys

Example bucket name:

devops-training-s3-bucket

Actual bucket names must follow S3 naming requirements and should follow the organization's naming standards.

5. Object
Definition

An object is the actual data stored inside an S3 bucket.

Examples:

image.jpg
index.html
file.pdf
application.zip
backup.sql
app.log

Architecture:

Bucket
   │
   ├── Object
   │    └── image.jpg
   │
   ├── Object
   │    └── index.html
   │
   └── Object
        └── file.pdf

An S3 object conceptually contains:

Object
│
├── Data
├── Key
└── Metadata

The data is the actual content.

The key identifies the object within the bucket.

The metadata contains information associated with the object.

6. Object Key
Definition

An object key is the name that identifies an object within an S3 bucket.

For example:

image.jpg

can be an object key.

Another example:

images/company-logo.png

can also be an object key.

Example:

Bucket:
devops-training-bucket

Object Key:
images/company-logo.png

The complete S3 location can be represented as:

s3://devops-training-bucket/images/company-logo.png
Folder-Like Structure

S3 does not use traditional filesystem directories in the same way as an operating system.

The / character is part of the object key and creates a folder-like organization in tools such as the S3 Console.

Example:

logs/production/application.log

The complete key is:

logs/production/application.log

Conceptually, it appears as:

logs/
└── production/
    └── application.log
7. Bucket vs Object vs Object Key

These concepts are closely related but different.

Component	Meaning	Example
Bucket	Container for objects	devops-training-bucket
Object	Actual stored data	image.jpg
Object Key	Identifier/name of the object	images/image.jpg

Example:

Bucket:
devops-training-bucket

Object Key:
images/image.jpg

Object:
Actual image.jpg data

Complete location:

s3://devops-training-bucket/images/image.jpg
8. Region
Definition

An AWS Region is a geographic area where AWS provides infrastructure and services.

Examples include:

ap-south-1

which represents the AWS Mumbai Region.

Another example is:

us-east-1

which represents the US East (N. Virginia) Region.

For this training environment, an example S3 bucket may be created in:

ap-south-1
S3 and Region

When creating an S3 bucket, a Region is selected.

Architecture:

AWS
 │
 └── Region
      │
      └── S3
           │
           └── Bucket
                │
                ├── Object
                ├── Object
                └── Object

The selected Region can affect:

Data location
Network latency
Regulatory requirements
Service integration
Cost considerations
Disaster recovery architecture
9. Storage

S3 provides object storage.

Instead of attaching a disk to a server, applications store objects in S3.

Example:

Application
     │
     ↓
     S3
     │
     ↓
   Bucket
     │
     ├── image.jpg
     ├── report.pdf
     └── backup.zip

S3 is designed to scale to very large amounts of object data.

S3 Storage Classes

S3 provides different storage classes designed for different access patterns.

Examples include:

S3 Standard
S3 Intelligent-Tiering
S3 Standard-IA
S3 One Zone-IA
S3 Glacier Instant Retrieval
S3 Glacier Flexible Retrieval
S3 Glacier Deep Archive

The appropriate storage class depends on how frequently data is accessed and how quickly it needs to be retrieved.

For basic training, the important concept is:

S3
 │
 └── Object Storage
       │
       ├── Files
       ├── Backups
       ├── Logs
       ├── Artifacts
       └── Media
10. Permissions

S3 permissions determine who or what is allowed to access S3 resources and perform actions.

Examples of S3 actions include:

s3:ListBucket
s3:GetObject
s3:PutObject
s3:DeleteObject

For example:

Developer
    │
    ↓
IAM Permission
    │
    ↓
S3 Bucket
    │
    ↓
Object

A DevOps engineer might have permission to upload objects:

s3:PutObject

while another application may only need to read objects:

s3:GetObject
11. IAM and S3

S3 access is commonly controlled using AWS identity and resource-based authorization mechanisms.

A simplified architecture is:

IAM User / Role
       │
       ↓
    Policy
       │
       ↓
 S3 Permission
       │
       ↓
     Bucket
       │
       ↓
    Object

For example:

IAM Role
   │
   └── s3:GetObject
            │
            ↓
      S3 Bucket/Object

The exact authorization result depends on applicable policies and other S3 access controls.

12. Least Privilege for S3

S3 permissions should follow the principle of least privilege.

This means giving an identity only the permissions required to perform its task.

For example, if an application only needs to download files, it may only require read access.

Application
    │
    ↓
IAM Role
    │
    ↓
s3:GetObject
    │
    ↓
Required S3 Objects

It should not automatically receive unrestricted permissions such as:

s3:*

unless there is a specific, justified requirement.

13. Complete S3 Architecture

Combining all the concepts:

                              AWS
                               │
                         AWS Region
                               │
                               ↓
                              S3
                               │
                            Bucket
                               │
             ┌─────────────────┼─────────────────┐
             ↓                 ↓                 ↓
          Object             Object             Object
          image.jpg          index.html         file.pdf
             │                 │                 │
             ↓                 ↓                 ↓
        Object Key        Object Key        Object Key
        image.jpg         index.html        file.pdf

Permissions are applied through AWS access-control mechanisms:

                  IAM User / Role
                         │
                         ↓
                       Policy
                         │
                         ↓
                    S3 Permissions
                         │
                         ↓
                       Bucket
                         │
                         ↓
                      Objects
14. Example S3 Architecture for DevOps

Consider a CI/CD pipeline.

Developer
    │
    ↓
GitHub
    │
    ↓
CI/CD Pipeline
    │
    ↓
Build Application
    │
    ↓
Application Artifact
    │
    ↓
S3 Bucket
    │
    ├── application-1.0.0.zip
    ├── application-1.0.1.zip
    └── application-1.1.0.zip

Here:

S3
 │
 └── Bucket
      │
      ├── Object
      ├── Object
      └── Object

The object keys could be:

releases/application-1.0.0.zip
releases/application-1.0.1.zip
releases/application-1.1.0.zip
15. Example with Website Files

Suppose an organization stores a static website in S3.

Bucket:

company-website-bucket

Objects:

index.html
css/style.css
js/app.js
images/logo.png

Architecture:

S3
 │
 └── company-website-bucket
       │
       ├── index.html
       ├── css/style.css
       ├── js/app.js
       └── images/logo.png

Object keys:

index.html
css/style.css
js/app.js
images/logo.png
16. Example with Application Backups

An application can store backups in S3.

Application
     │
     ↓
Backup Process
     │
     ↓
S3 Bucket
     │
     └── backups/
          ├── database-2026-09-11.sql
          ├── database-2026-09-12.sql
          └── database-2026-09-13.sql

The bucket provides the storage container, while each backup is stored as an object.

17. S3 Architecture vs EC2 Storage

S3 and EC2 storage should not be confused.

S3
 │
 └── Object Storage
      │
      └── Objects

EC2 with EBS:

EC2 Instance
     │
     ↓
EBS Volume
     │
     └── Filesystem
S3

Used for:

Backups
Images
Videos
Documents
Logs
Build artifacts
Static files
EC2/EBS

Used for:

Operating system
Application runtime files
Server filesystem
Database storage where appropriate

Simple rule:

Object storage → S3

Block storage attached to EC2 → EBS
18. S3 Architecture Summary
Component	Explanation
AWS	Cloud platform providing S3
Region	Geographic AWS location where the bucket is created
S3	AWS object storage service
Bucket	Container for S3 objects
Object	Actual stored data
Object Key	Name/identifier of the object
Storage	Object storage provided by S3
Permissions	Controls who/what can perform actions on S3 resources
19. Simple Memory Diagram

The complete concept can be remembered as:

                         AWS
                          │
                       Region
                          │
                          ↓
                         S3
                          │
                       Bucket
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
          Object        Object       Object
          image.jpg     index.html   file.pdf
             │            │            │
             ↓            ↓            ↓
            Key          Key          Key

Access control:

IAM User / Role
       │
       ↓
     Policy
       │
       ↓
   Permissions
       │
       ↓
     S3 Bucket
       │
       ↓
    Objects
20. Key Points to Remember
Bucket

A container that stores S3 objects.

Object

The actual data stored in S3.

Object Key

The name/identifier used to locate an object within a bucket.

Region

The AWS geographic Region associated with the bucket.

Storage

S3 provides object storage for files and other data.

Permissions

IAM and S3 access-control mechanisms determine what identities can do with S3 resources.

21. What I Learned

After completing this task, I understand:

Amazon S3 is an object storage service.
S3 stores objects inside buckets.
A bucket is a container for objects.
An object is the actual stored data.
An object key identifies an object within a bucket.
S3 buckets are associated with AWS Regions.
S3 can store documents, images, logs, backups, application packages, and other data.
S3 uses object storage rather than traditional server filesystem storage.
IAM permissions can control access to S3.
Least privilege should be followed when granting S3 permissions.
S3 is useful for DevOps pipelines, backups, artifacts, and application data.