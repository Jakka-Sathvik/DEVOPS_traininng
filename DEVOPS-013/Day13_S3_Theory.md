Day 13 — Amazon S3 Theory
TASK 1 — Understand Amazon S3

Training Area: AWS / DevOps
Time: 45 Minutes
Deliverable: Day13_S3_Theory.md
Status: Completed

1. Objective

The objective of this task is to understand the fundamentals of Amazon Simple Storage Service (Amazon S3) before performing practical operations in the AWS Console.

The following concepts are covered:

What is Amazon S3?
What problem does S3 solve?
What is a bucket?
What is an object?
What is an object key?
What is object storage?
Difference between S3 and EC2 storage
Why S3 is useful for DevOps
Types of files that can be stored in S3
2. What is Amazon S3?

Amazon S3 (Simple Storage Service) is an AWS cloud storage service designed to store and retrieve large amounts of data from anywhere through APIs, the AWS Console, or other AWS services.

S3 is an example of object storage.

Instead of storing files inside a traditional server's filesystem, S3 stores data as objects inside containers called buckets.

Basic architecture:

                    Amazon S3
                       │
             ┌─────────┴─────────┐
             │                   │
          Bucket A            Bucket B
             │                   │
       ┌─────┼─────┐        ┌────┼────┐
       │     │     │        │    │    │
     File1 File2 File3    File1 File2 File3

For example, a company could use S3 to store:

Application backups
Log files
Images
Videos
Documents
Software packages
Build artifacts
Configuration files
Reports
Database backups
Static website files

S3 is designed for highly durable storage and can scale without the user having to manage physical storage servers.

3. What Problem Does S3 Solve?

Before cloud object storage, organizations commonly stored files on:

Local hard drives
File servers
NAS systems
Application servers
Backup servers
Physical storage appliances

These approaches can create operational problems.

For example:

Traditional Environment

Application
     │
     ↓
Application Server
     │
     ↓
Local Disk
     │
     ↓
Limited Storage

If the server's storage becomes full, administrators may need to add additional disks or storage systems.

With S3:

Application
     │
     ↓
    S3
     │
     ├── Object
     ├── Object
     ├── Object
     └── Object

S3 provides scalable object storage without requiring the DevOps team to manage physical storage infrastructure.

Problems S3 Helps Solve
3.1 Storage Scalability

Applications may generate large amounts of data.

S3 can store very large quantities of objects without the DevOps engineer manually expanding a server disk.

3.2 Backup Storage

Organizations can store backups in S3.

Example:

Application
     │
     ↓
Database Backup
     │
     ↓
     S3
3.3 Build Artifact Storage

CI/CD pipelines can store generated artifacts in S3.

Example:

Developer
    ↓
Git
    ↓
CI/CD Pipeline
    ↓
Build
    ↓
Application Artifact
    ↓
S3
3.4 Log Storage

Application and infrastructure logs can be stored or archived in S3.

Application
     │
     ↓
Logs
     │
     ↓
S3
3.5 Static Website Storage

S3 can also be used to store static website assets such as:

HTML
CSS
JavaScript
Images
4. What is a Bucket?

A bucket is a container used to store objects in Amazon S3.

The relationship is:

AWS Account
     │
     ↓
    S3
     │
     ↓
   Bucket
     │
     ├── Object
     ├── Object
     └── Object

For example:

Bucket: devops-training-storage

Objects:
├── application.zip
├── backup.sql
├── logs/app.log
├── images/logo.png
└── reports/monthly-report.pdf

The bucket provides the top-level namespace for stored objects.

Bucket Name

S3 bucket names must follow AWS naming requirements and must be unique within the applicable S3 namespace.

Example:

my-devops-training-bucket

A more organization-specific naming approach might be:

company-devops-training-artifacts

For training, always follow your organization's naming rules.

5. What is an Object?

An object is the actual data stored in an S3 bucket.

An object consists conceptually of:

Object
  │
  ├── Data
  ├── Key
  └── Metadata

The data can be a file or other byte content.

Examples:

application.zip
backup.sql
logo.png
report.pdf
app.log
index.html

If we have:

Bucket:
devops-training-storage

and upload:

application.zip

then:

S3
 │
 └── devops-training-storage
          │
          └── application.zip

The object is the stored data, while the bucket is the container holding it.

6. What is an Object Key?

An object key is the name used to identify an object within an S3 bucket.

For example:

application.zip

can be an object key.

Another example:

backups/database/backup-2026-09-13.sql

is also an object key.

The structure can look like:

Bucket
 │
 ├── application.zip
 │
 ├── logs/app.log
 │
 ├── backups/database/backup.sql
 │
 └── reports/monthly/report.pdf

Important point:

S3 does not use traditional filesystem directories in the same way an operating system does.

The / characters in keys create a folder-like naming structure in the console.

For example:

logs/application/app.log

is an object key.

Conceptually:

logs/
   └── application/
          └── app.log

The complete key is:

logs/application/app.log
7. Bucket vs Object vs Object Key

These three concepts should not be confused.

Concept	Meaning	Example
Bucket	Container for S3 objects	devops-training-bucket
Object	Actual stored data	application.zip
Object Key	Name identifying the object	builds/application.zip

Example:

Bucket:
devops-training-bucket

Object Key:
builds/application.zip

Object:
The actual application.zip data

Architecture:

S3
 │
 └── Bucket
      │
      └── Object
           │
           ├── Key
           │    └── builds/application.zip
           │
           └── Data
                └── application.zip
8. What is Object Storage?

Object storage is a storage architecture where data is stored as independent objects.

Each object contains data along with information such as metadata and an identifier/key.

S3 is an object storage service.

A simplified model is:

Object Storage

Object
 ├── Data
 ├── Metadata
 └── Identifier / Key

This is different from traditional filesystem storage.

Object Storage Example

Suppose we upload:

photo.jpg

S3 stores it as an object.

Another object might be:

backup.zip

Another:

application.log

All these objects can exist in the same bucket.

Bucket
 │
 ├── photo.jpg
 ├── backup.zip
 ├── application.log
 └── application.tar.gz

Object storage is especially useful for large-scale unstructured data.

9. S3 and EC2 Storage Difference

S3 and EC2 storage serve different purposes.

Amazon S3

S3 is primarily used for object storage.

Examples:

Backups
Images
Videos
Logs
Documents
Build artifacts
Static files
EC2 Storage

EC2 instances commonly use attached storage such as Amazon EBS (Elastic Block Store) for operating system and application files.

Example:

EC2 Instance
     │
     ↓
   EBS Volume
     │
     ├── OS files
     ├── Application files
     └── Runtime data

S3:

Application
     │
     ↓
     S3 Bucket
     │
     └── Objects
Comparison
Feature	S3	EC2/EBS Storage
Storage type	Object storage	Block storage
Main use	Files/objects	OS and application disks
Attached to EC2	Not as a normal disk	Yes, EBS attaches to EC2
Access model	API/object operations	Filesystem/block device
Scaling	Designed for massive object storage	Volume capacity must be managed
Typical use	Backup, artifacts, media, logs	OS, databases, application disk
Server dependency	Independent of a specific EC2 instance	Commonly associated with EC2
DevOps use	Artifact/backup/log storage	Application runtime storage
Simple Rule
Need object/file storage?
        ↓
       S3

Need a disk for an EC2 server?
        ↓
       EBS

S3 should not be thought of as simply a replacement for an EC2 filesystem.

10. Why is S3 Useful for DevOps?

S3 is very useful in DevOps because DevOps environments continuously generate artifacts, logs, backups, configuration packages, and other files.

10.1 CI/CD Artifacts

A CI/CD pipeline may build an application package.

Example:

Developer
    ↓
GitHub
    ↓
GitHub Actions
    ↓
Build
    ↓
application.zip
    ↓
S3

The artifact can then be retrieved by a later deployment stage.

10.2 Backup Storage

DevOps teams can use S3 as a destination for backup workflows.

Example:

Database
   │
   ↓
Backup
   │
   ↓
S3

Backups can be organized using object keys.

Example:

backups/
├── database/
│   ├── 2026-09-11.sql
│   ├── 2026-09-12.sql
│   └── 2026-09-13.sql
10.3 Log Archiving

Logs can be archived to S3.

Example:

Application
     ↓
Logs
     ↓
Log Management
     ↓
S3

This can help organizations retain historical logs according to their operational or compliance requirements.

10.4 Deployment Packages

Applications may be packaged into files such as:

application.zip
application.tar.gz
deployment-package.zip

These packages can be stored in S3.

Example:

S3 Bucket
 │
 └── releases/
      ├── application-1.0.0.zip
      ├── application-1.0.1.zip
      └── application-1.1.0.zip
10.5 Static Website Assets

S3 can store static website content such as:

website/
├── index.html
├── style.css
├── app.js
└── images/

S3 can be combined with other AWS services to build scalable website delivery architectures.

10.6 Infrastructure Automation

DevOps teams can interact with S3 through:

AWS CLI
Terraform
CloudFormation
CI/CD pipelines
AWS SDKs
AWS Console

Example Terraform concept:

Terraform
    ↓
AWS Provider
    ↓
S3 Bucket

This allows infrastructure to be defined as code.

11. What Type of Files Can Be Stored in S3?

S3 can store many types of data.

Examples include:

Documents
.pdf
.docx
.xlsx
.txt
Images
.jpg
.jpeg
.png
.gif
.webp
.svg
Videos
.mp4
.mov
.mkv
Audio
.mp3
.wav
Application Packages
.zip
.tar
.gz
.tar.gz
Source/Configuration Files
.json
.yaml
.yml
.xml
.csv
.conf
Database Backups
.sql
.dump
.bak
Logs
.log
Static Website Files
.html
.css
.js

S3 is not limited to a specific file extension. It can store arbitrary object data, subject to S3 service limits and the application's requirements.

12. Example DevOps S3 Architecture

A common DevOps workflow can look like:

                 Developer
                     │
                     ↓
                  GitHub
                     │
                     ↓
                CI/CD Pipeline
                     │
             ┌───────┴────────┐
             │                │
           Build             Test
             │                │
             └───────┬────────┘
                     ↓
              Application
                 Artifact
                     │
                     ↓
                    S3
                     │
             ┌───────┴────────┐
             │                │
          Backup          Deployment
         Storage           Package

S3 acts as durable object storage within the workflow.

13. S3 Security

S3 data can contain sensitive information, so security is important.

Important security practices include:

13.1 Least Privilege

Grant users and applications only the permissions they need.

For example:

Application
    ↓
IAM Role
    ↓
s3:GetObject
    ↓
Specific S3 Objects

Avoid granting unnecessary permissions such as unrestricted access to every S3 bucket.

13.2 Block Unintended Public Access

S3 buckets should not be made public unless there is a specific, reviewed business requirement.

For private application data, access should normally be controlled through appropriate IAM/resource policies and other security controls.

13.3 Encryption

Sensitive data should be protected using appropriate encryption controls.

S3 supports encryption at rest, including AWS-managed and customer-managed key options.

13.4 Avoid Hard-Coded Credentials

Do not put AWS credentials inside:

Python files
Shell scripts
Git repositories
Dockerfiles
README files
Screenshots

For AWS workloads, use appropriate IAM roles or another approved authentication mechanism.

14. S3 Object Organization

Objects can be organized using key naming conventions.

Example:

company-data/
│
├── applications/
│   ├── app1/
│   └── app2/
│
├── backups/
│   ├── database/
│   └── application/
│
├── logs/
│   ├── production/
│   └── development/
│
└── releases/
    ├── v1.0.0/
    └── v1.1.0/

A good naming strategy makes objects easier to locate and manage.

Example key:

releases/myapp/1.0.0/application.zip
15. S3 and DevOps Automation

S3 can be managed manually or automatically.

Manual
Developer
    ↓
AWS Console
    ↓
S3
    ↓
Upload Object
CLI
Developer
    ↓
Terminal
    ↓
AWS CLI
    ↓
S3
Infrastructure as Code
Developer
    ↓
Terraform
    ↓
AWS Provider
    ↓
S3
CI/CD
Git Push
   ↓
CI/CD
   ↓
Build
   ↓
Artifact
   ↓
S3

Automation reduces repetitive manual work and supports consistent DevOps workflows.

16. Important S3 Terminology
Term	Meaning
S3	Amazon Simple Storage Service
Bucket	Container for S3 objects
Object	Data stored in S3
Object Key	Name/identifier of an object within a bucket
Object Storage	Storage model based on independent objects
Metadata	Information describing an object
S3 URI	URI-style reference to an S3 location
Prefix	Leading portion of an object key used to organize/list objects

Example:

s3://devops-training-bucket/releases/app.zip

Here:

Bucket:
devops-training-bucket

Key:
releases/app.zip
17. S3 vs Traditional Server Storage
Traditional approach
User
 ↓
Application Server
 ↓
Server Disk
 ↓
Files

Problems may include:

Limited disk capacity
Server dependency
Manual storage management
Backup management
Hardware maintenance
S3 approach
User/Application
       ↓
      S3
       ↓
    Bucket
       ↓
    Objects

Benefits include:

Scalable object storage
API-based access
AWS-managed infrastructure
Integration with many AWS services
Useful storage classes and lifecycle capabilities
Automation support
18. Real-World Example

Consider an online shopping application.

The application stores product images.

Without object storage:

Web Server
    ↓
Local Disk
    ↓
product-images/

If several application servers are running, managing the same files across servers becomes more complicated.

With S3:

             Web Application
                    │
                    ↓
                   S3
                    │
             Product Images

Example keys:

products/
├── phones/
│   ├── phone-001.jpg
│   └── phone-002.jpg
├── laptops/
│   ├── laptop-001.jpg
│   └── laptop-002.jpg
└── accessories/
    └── mouse-001.jpg

The application can access the required objects through appropriate AWS permissions.

19. Key Differences to Remember
Bucket
Container
Object
Actual stored data
Object Key
Name used to identify the object
S3
AWS object storage service
EC2/EBS
Compute + block storage for server workloads

Simple memory trick:

S3
 │
 └── Bucket
       │
       └── Object
             │
             └── Object Key
20. What I Learned

After completing this task, I understand:

Amazon S3 is an AWS object storage service.
S3 stores data as objects.
Objects are stored inside buckets.
An object key identifies an object within a bucket.
S3 is useful for backups, logs, artifacts, documents, media, and static files.
S3 is different from EC2/EBS storage.
S3 provides scalable object storage without managing physical storage servers.
S3 is highly useful in CI/CD and DevOps workflows.
S3 can be accessed through the AWS Console, AWS CLI, SDKs, and infrastructure-as-code tools.
IAM should be used to control access to S3.
Least privilege should be applied to S3 permissions.
AWS credentials should never be hard-coded into application code or repositories.
21. Final Architecture

The complete S3 concept can be summarized as:

                         AWS
                          │
                          ↓
                         S3
                          │
                    ┌─────┴─────┐
                    │           │
                 Bucket A    Bucket B
                    │           │
              ┌─────┼─────┐     │
              │     │     │     │
           Object Object Object Object
              │
              ↓
         Object Key

DevOps integration:

Developer
    │
    ↓
Git / GitHub
    │
    ↓
CI/CD Pipeline
    │
    ↓
Build Artifact
    │
    ↓
   S3
    │
    ├── Release Packages
    ├── Backups
    ├── Logs
    └── Static Files