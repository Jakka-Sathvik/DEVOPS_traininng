TASK 3 — Create Your First S3 Bucket

Training Area: AWS / DevOps
Time: 1 Hour
Deliverable: S3 Bucket Creation
Status: Practical Task

1. Objective

The objective of this task is to create the first Amazon S3 bucket using the AWS Management Console and understand the important security and configuration settings before creating it.

The following settings must be identified:

Bucket name
AWS Region
Object Ownership
Block Public Access
Versioning
Encryption

The bucket should remain private for this training exercise.

2. S3 Bucket Architecture

The bucket will have the following basic structure:

AWS Account
     │
     ↓
   Amazon S3
     │
     ↓
   Region
     │
     ↓
   S3 Bucket
     │
     ├── Object
     ├── Object
     └── Object

Example:

AWS Account
     │
     ↓
S3
     │
     ↓
ap-south-1
     │
     ↓
sathvik-devops-training-2026
     │
     ├── test.txt
     ├── application.zip
     └── README.md
3. Before Creating the Bucket

Before opening the bucket creation form, decide the following values.

Bucket Name

The bucket name must be globally unique.

Example:

sathvik-devops-training-2026

If this name is already taken, use another unique name, for example:

sathvik-devops-s3-training-2026

Do not use spaces or uppercase letters.

A good training naming pattern is:

<name>-<purpose>-<year>

Example:

sathvik-devops-training-2026
4. AWS Region

For this training example, use:

Asia Pacific (Mumbai)
ap-south-1

The Region should normally be selected based on factors such as:

User/application location
Latency
Data residency requirements
Compliance requirements
Service availability
Cost
Disaster recovery strategy

For a training environment, using one consistent Region makes the exercises easier to manage.

5. Open the S3 Console
Sign in to the AWS Management Console using your approved training identity.
Search for:
S3
Open Amazon S3.
Select Create bucket.

Do not use the root account for normal daily administration.

6. Enter the Bucket Name

In the bucket creation page, find:

Bucket name

Enter your globally unique name.

Example:

sathvik-devops-training-2026

Remember:

Bucket Name
      ↓
Must be globally unique

If AWS reports that the name is already in use, change the name.

7. Select the AWS Region

Under the Region setting, select:

Asia Pacific (Mumbai)
ap-south-1

Architecture:

AWS
 │
 └── ap-south-1
       │
       └── S3
            │
            └── Your Bucket
8. Object Ownership

The S3 bucket creation page contains an Object Ownership setting.

For a normal private training bucket, use the recommended modern setting:

ACLs disabled
Bucket owner enforced
What does Object Ownership mean?

Object Ownership determines who owns objects uploaded to the bucket and whether S3 access control lists (ACLs) are used.

With:

Bucket owner enforced

ACLs are disabled for the bucket, and the bucket owner owns the objects.

This simplifies access management because permissions are primarily managed using policies rather than legacy object ACLs.

Architecture:

Uploader
    │
    ↓
S3 Bucket
    │
    ↓
Bucket Owner Owns Object

For this training exercise:

Object Ownership:
Bucket owner enforced
9. Block Public Access

This is one of the most important settings in the exercise.

You should see the Block Public Access section.

For this training bucket:

☑ Block all public access

Keep all public access blocking options enabled.

Do not disable them simply to make testing easier.

10. Why Does Block Public Access Exist?

S3 can store sensitive information such as:

Database backups
Application logs
Source packages
Internal documents
Configuration files
Business data
Customer information

Accidentally making a bucket or objects public can expose this information to the internet.

Therefore AWS provides S3 Block Public Access controls to help prevent unintended public exposure.

The security model should be:

Default
   ↓
Private
   ↓
Grant required access
   ↓
Review
   ↓
Make public only when specifically required

Not:

Make public
    ↓
Try to secure it later
11. Should the Training Bucket Be Public?

No.

There is no requirement for this training bucket to be publicly accessible.

Use:

Block all public access = ON

The bucket can still be used by authorized IAM identities and AWS services.

For example:

IAM User / Role
       │
       ↓
   IAM Policy
       │
       ↓
   Private S3 Bucket

Public access is not required for normal S3 upload/download training.

12. When Would Public Access Actually Be Required?

There are legitimate use cases for public content.

For example, a company may intentionally publish publicly accessible static content.

Possible example:

Internet User
      │
      ↓
Public Website Content
      │
      ↓
S3

Examples could include:

Public website assets
Public images
Public documentation
Public downloads

However, even in these scenarios, the architecture should be intentionally designed and reviewed.

A better modern architecture may use:

Internet
    │
    ↓
CloudFront
    │
    ↓
Private S3 Origin

rather than simply making an entire S3 bucket publicly accessible.

The important lesson is:

Public access should be an intentional business and architecture requirement, not a shortcut for testing.

13. Versioning

Find the Bucket Versioning setting.

For this training bucket, enable:

Enable
What is Versioning?

S3 Versioning allows multiple versions of an object to be maintained.

For example:

application.zip
     │
     ├── Version 1
     ├── Version 2
     └── Version 3

Suppose:

config.json

is uploaded.

Later it is modified and uploaded again.

Without versioning:

config.json
   ↓
New version replaces current object

With versioning:

config.json
   │
   ├── Version 1
   └── Version 2

Versioning can help with:

Accidental overwrites
Accidental deletions
Recovery of previous object versions
Release management
Data protection
14. Encryption

S3 provides encryption for objects stored in the bucket.

For a standard training bucket, keep the recommended server-side encryption setting.

A common choice is:

Server-side encryption
SSE-S3

SSE-S3 uses Amazon S3-managed encryption keys.

The basic concept is:

Upload Object
      │
      ↓
S3 Encryption
      │
      ↓
Encrypted Object

Example:

application.zip
       ↓
Server-side encryption
       ↓
Stored encrypted in S3

Encryption at rest helps protect stored data.

15. Review the Configuration

Before selecting Create bucket, review the configuration.

Recommended training configuration:

Setting	Training Value
Bucket Name	Globally unique name
Region	ap-south-1
Object Ownership	Bucket owner enforced
ACLs	Disabled
Block Public Access	Enabled
Versioning	Enabled
Encryption	SSE-S3

Example:

Bucket Name:
sathvik-devops-training-2026

Region:
ap-south-1

Object Ownership:
Bucket owner enforced

Block Public Access:
Enabled

Versioning:
Enabled

Encryption:
SSE-S3
16. Create the Bucket

After reviewing all settings:

Scroll to the bottom of the page.
Select:
Create bucket

AWS should create the bucket.

You should then see the bucket in the S3 bucket list.

17. Verify the Bucket

After creation, open the bucket.

Check:

Bucket
 │
 ├── Properties
 ├── Permissions
 └── Objects

Verify the following.

Bucket Name

Confirm the name is correct.

Region

Confirm:

ap-south-1
Object Ownership

Confirm:

Bucket owner enforced
Block Public Access

Confirm public access blocking remains enabled.

Versioning

Confirm:

Enabled
Encryption

Confirm server-side encryption is enabled according to your selected configuration.

18. Upload a Test Object

After the bucket is created, upload a simple test file.

Create a local file:

test.txt

Example content:

S3 Training Test File
Created during AWS DevOps training.

Then:

Open the S3 bucket.
Select Upload.
Add test.txt.
Review the upload.
Select Upload.

The bucket should now contain:

sathvik-devops-training-2026
│
└── test.txt
19. Understand the Object Key

If the uploaded file is:

test.txt

then:

Bucket:
sathvik-devops-training-2026

Object Key:
test.txt

The S3 URI can be represented as:

s3://sathvik-devops-training-2026/test.txt

If the object was uploaded under a prefix:

training/test.txt

then the object key is:

training/test.txt
20. Test Private Access

Because Block Public Access is enabled, an anonymous internet user should not be able to access the object simply because its URL is known.

Authorized AWS identities can access it if they have appropriate permissions.

Architecture:

Authorized IAM Identity
          │
          ↓
     IAM Policy
          │
          ↓
     Private S3 Bucket
          │
          ↓
       Object

This demonstrates the difference between:

Private Access

and:

Public Internet Access
21. S3 Security Architecture

The secure training architecture is:

                    AWS Account
                         │
                         ↓
                       IAM
                         │
                         ↓
                Least-Privilege Policy
                         │
                         ↓
                    Private S3
                         │
             ┌───────────┼───────────┐
             ↓           ↓           ↓
          Object      Object      Object

Public access is blocked:

Internet
   │
   X
   │
Block Public Access
   │
   ↓
Private S3 Bucket
22. Why Not Use AdministratorAccess?

The bucket creation task does not mean that the training user should receive unrestricted AWS permissions.

AWS permissions should follow:

Required Action
      ↓
Required Resource
      ↓
Minimum Permission

For example, an application that only needs to read objects should not automatically receive permission to:

DeleteBucket
PutBucketPolicy
DeleteObject

unless those actions are genuinely required.

23. Important Security Rules

Follow these rules during the exercise.

Rule 1 — Keep the bucket private
Block Public Access = ON
Rule 2 — Do not expose credentials

Never put these in screenshots:

Access Key
Secret Access Key
Session Token
Password
MFA recovery information
Rule 3 — Use approved authentication

Use the training identity and authentication method approved by your organization.

Rule 4 — Follow least privilege

Grant only the permissions required for the exercise.

Rule 5 — Do not make the bucket public for convenience

If an operation fails because the bucket is private, investigate the permissions rather than immediately disabling security controls.

24. Screenshots to Capture

For your training documentation, capture screenshots of:

Screenshot 1 — Bucket Creation Settings

Show:

Bucket Name
Region
Object Ownership

Do not expose credentials.

Screenshot 2 — Block Public Access

Show:

Block all public access

with the setting enabled.

Screenshot 3 — Versioning and Encryption

Show:

Versioning: Enabled
Encryption: Enabled
Screenshot 4 — Created Bucket

Show the bucket in the S3 bucket list.

Screenshot 5 — Uploaded Object

Show:

test.txt

inside the bucket.

25. Final Architecture
                         AWS
                          │
                          ↓
                    ap-south-1
                          │
                          ↓
                         S3
                          │
                          ↓
              sathvik-devops-training-2026
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
          Object        Object       Object
         test.txt     application.zip README.md
             │
             ↓
        Object Key
         test.txt

Security:

                IAM User / Role
                       │
                       ↓
                 IAM Policy
                       │
                       ↓
              Least Privilege
                       │
                       ↓
                Private S3 Bucket
                       │
                       ↓
                    Objects

Internet ──────── X ────────> Bucket
                 Block
              Public Access
26. Task Completion Checklist

Opened Amazon S3 Console

Selected approved AWS Region

Chosen a globally unique bucket name

Understood Object Ownership

Kept ACLs disabled / Bucket owner enforced

Enabled Block Public Access

Understood why public access is blocked

Enabled Versioning

Reviewed encryption settings

Created the S3 bucket

Verified bucket configuration

Uploaded a test object

Verified the object

Did not expose credentials

Did not make the bucket public

27. What I Learned

After completing this task, I understand:

How to create an S3 bucket.
Why an S3 bucket name must be globally unique.
How to select an AWS Region.
What Object Ownership means.
Why ACLs are normally disabled with Bucket owner enforced.
Why S3 Block Public Access is important.
Why an S3 bucket should remain private unless public access is intentionally required.
What S3 Versioning provides.
What server-side encryption provides.
How IAM permissions control access to private S3 resources.
How to upload and verify an S3 object.
Why security settings should not be disabled simply to make testing easier.
Conclusion

The first S3 bucket was created using a security-first configuration.

The recommended training architecture is:

AWS
 │
 └── Region
      │
      └── S3
           │
           └── Private Bucket
                │
                ├── Object
                ├── Object
                └── Object