TASK 7 — S3 Security & Permissions

Time: 1 Hour
Topic: S3 Security & Permissions
Deliverable: S3_Security_Permissions.md

1. Objective

The objective of this task is to understand how security and permissions work with Amazon S3.

The main areas covered are:

IAM access to S3
Bucket policies
Object permissions
Public access controls
Private data protection
IAM roles for applications
Verifying that an S3 bucket does not accidentally expose private files

The basic security architecture is:

                    S3
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
 Bucket Permissions  Object      Public Access
                     Permissions  Controls
        │            │            │
        └────────────┼────────────┘
                     ↓
                IAM Policies
                     │
                     ↓
              Authorized Access
2. What is IAM Access to S3?

IAM (Identity and Access Management) controls who is allowed to perform actions on AWS resources.

For S3, IAM permissions can allow a user or role to perform operations such as:

s3:ListBucket
s3:GetObject
s3:PutObject
s3:DeleteObject

For example:

IAM User / IAM Role
        ↓
    IAM Policy
        ↓
    S3 Permission
        ↓
   S3 Bucket/Object
Example

Suppose a training IAM role has:

{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject"
  ],
  "Resource": "arn:aws:s3:::my-training-bucket/*"
}

This allows the role to read objects from the bucket.

It does not automatically mean that the role can:

delete objects
upload objects
modify bucket settings
change permissions

Those actions require appropriate permissions.

Important

IAM permissions are an important part of S3 security.

The goal should always be:

Minimum Required Permission
            ↓
       Required Action
            ↓
       Required Resource

This is called the principle of least privilege.

3. What is a Bucket Policy?

A bucket policy is a resource-based policy attached directly to an S3 bucket.

It can define who can access the bucket and what actions they can perform.

Basic architecture:

Principal
    ↓
Bucket Policy
    ↓
S3 Bucket
    ↓
Objects

A bucket policy can specify:

Effect
Principal
Action
Resource
Condition

Example structure:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/MyApplicationRole"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-training-bucket/*"
    }
  ]
}

This example allows a specified IAM role to read objects.

Training note: Do not copy this example directly into your bucket. The account ID, role name, bucket name, and required permissions must match the approved environment.

4. IAM Policy vs Bucket Policy
Feature	IAM Policy	Bucket Policy
Attached to	User, group, or role	S3 bucket
Policy type	Identity-based	Resource-based
Controls	What identity can do	Who can access the bucket/resource and what they can do
Common use	Grant permissions to IAM identities	Control access to a specific bucket
Example	Role can read S3 objects	Specific principal can access bucket

Both can participate in AWS authorization decisions.

A permission should not be granted simply because it is technically possible.

5. What are Object Permissions?

S3 stores data as objects inside buckets.

Example:

Bucket
 │
 ├── documents/report.pdf
 ├── images/logo.png
 └── backups/database.zip

An object can be accessed through permissions granted by the applicable AWS authorization mechanisms.

Typical actions include:

GetObject
PutObject
DeleteObject

For example:

Application
     ↓
IAM Role
     ↓
s3:GetObject
     ↓
Bucket
     ↓
Object

The application can retrieve an object only when the applicable permissions allow the operation.

6. What is Public Access Control?

S3 provides Block Public Access controls to help prevent buckets and objects from becoming publicly accessible.

For a private training bucket, the recommended approach is to keep public access blocked unless there is a specific, reviewed requirement for public access.

Architecture:

Internet
   │
   X
   │
Block Public Access
   │
   ↓
Private S3 Bucket
   │
   └── Private Objects

This reduces the risk of accidentally exposing sensitive information.

7. Why is Public Access Dangerous?

Making an S3 bucket or its objects public can allow unauthorized internet users to access data.

For example:

Internet User
      │
      ↓
Public S3 Bucket
      │
      ↓
Private Data

If sensitive files are accidentally made public, they may be copied or downloaded by unauthorized users.

Possible consequences include:

Data leakage
Exposure of company information
Exposure of customer information
Credential leakage
Compliance violations
Reputation damage
Financial loss
Security incidents

Therefore:

Private Data
     ↓
Private Bucket
     ↓
Least-Privilege Access

should be the default security approach.

8. Why Should Private Data Remain Private?

Private information should only be accessible to authorized identities.

Examples of private information include:

Passwords
AWS Credentials
API Keys
.env files
Private Company Documents
Customer Data
Database Backups
Internal Reports

A security mistake can happen if a developer uploads:

.env

containing:

AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
DATABASE_PASSWORD=...
API_KEY=...

This information must never be intentionally uploaded to a public S3 location.

Important Rule
Sensitive Data
      ↓
Never Public
      ↓
Restrict Access
      ↓
Use Least Privilege
9. Why Should Applications Use IAM Roles?

Applications running on AWS should generally use IAM roles rather than storing long-lived AWS access keys inside application code or servers.

Bad architecture:

EC2
 │
 ├── application
 │
 └── hard-coded AWS access key

This creates a risk if the key is exposed through:

Source code
Git repositories
Logs
Configuration files
Screenshots
Backups
Server compromise

Better architecture:

EC2
 │
 ↓
IAM Role
 │
 ↓
Temporary AWS Credentials
 │
 ↓
S3

The role can be given only the permissions the application requires.

For example:

Application
     ↓
EC2 IAM Role
     ↓
s3:GetObject
     ↓
Specific S3 Bucket

This follows least privilege.

10. Security Exercise
Verify That the Training Bucket Is Private

The objective is to confirm that the training bucket does not accidentally expose private files publicly.

Use the training bucket created in the previous S3 task.

Example:

S3
 │
 └── Training Bucket
       │
       ├── test.txt
       └── version-test.txt
Step 1 — Open S3
Sign in to the approved AWS environment.
Open Amazon S3.
Open your training bucket.

Do not use the AWS root account for routine administration.

Step 2 — Check Block Public Access

Open:

S3
 ↓
Training Bucket
 ↓
Permissions
 ↓
Block public access

Verify that public access blocking is enabled according to your organization's training/security requirements.

For a private training bucket, the expected result is:

Block Public Access
        ↓
     Enabled
Expected Result

The training bucket should not be intentionally configured for public access.

11. Step 3 — Check Bucket Policy

Navigate to:

Permissions
    ↓
Bucket policy

Check whether a bucket policy exists.

If there is no bucket policy, that is not automatically a problem.

If there is a bucket policy, carefully inspect:

Principal
Action
Resource
Condition
Effect

Pay particular attention to:

"Principal": "*"

A wildcard principal can be appropriate in some intentionally public architectures, but for a private training bucket it requires careful review.

Do not add a public policy just for testing.

12. Step 4 — Check Object Ownership

Check:

Permissions
     ↓
Object Ownership

For a typical private training bucket, Bucket owner enforced is a common configuration.

This disables the use of legacy ACL-based ownership/access patterns for the bucket.

13. Step 5 — Check Individual Objects

Open:

Objects
   ↓
test.txt

Inspect the object's available access information.

Confirm that you are not intentionally granting public access to the object.

Repeat for:

version-test.txt
14. Step 6 — Understand the Object URL

An S3 object may have a URL similar to:

https://bucket-name.s3.amazonaws.com/test.txt

The existence of an object URL does not automatically mean that the object is public.

A private object requires appropriate authorization.

Therefore:

Object URL
     ≠
Public Access

This is an important S3 concept.

15. Step 7 — Safe Public Access Test

Do not intentionally make the training bucket public just to test security.

Instead, verify the configuration through:

Block Public Access
Bucket Policy
Object Ownership
Object Access Settings
IAM Permissions

The goal is to confirm that private data is protected without creating a real security exposure.

16. Files That Must Never Be Uploaded

The following files must not be uploaded to the training bucket:

Passwords
password.txt
credentials.txt
AWS Credentials
access-keys.txt
aws-credentials.txt
Environment Files
.env
API Keys
api-keys.txt
Private Company Files
internal-report.pdf
customer-data.csv
private-document.docx
Database Credentials
database-password.txt
17. Secure vs Insecure Example
Insecure
Developer
    │
    ↓
.env
    │
    ├── AWS Access Key
    ├── AWS Secret Key
    └── Database Password
    │
    ↓
S3 Public Bucket

This can result in serious credential and data exposure.

Secure
Developer
    │
    ↓
Private S3 Bucket
    │
    ├── Block Public Access
    ├── Encryption
    ├── Least-Privilege IAM
    └── Versioning

For AWS applications:

Application
     ↓
IAM Role
     ↓
Minimum Required Permissions
     ↓
Private S3 Bucket
18. S3 Security Architecture

A secure training architecture can be represented as:

                    AWS
                     │
                    S3
                     │
              Private Bucket
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
    Objects       Encryption    Versioning
       │
       ↓
 Block Public Access
       │
       ↓
 IAM / Resource Policies
       │
       ↓
 Authorized Users / Roles
19. Five Important Security Principles
1. Use least privilege

Only grant the permissions that are actually required.

Required Access
      ↓
Minimum Permission
2. Keep private data private

Do not make confidential data publicly accessible.

3. Avoid long-lived credentials

Applications should generally use IAM roles and temporary credentials where appropriate.

4. Block unnecessary public access

For private training buckets, keep S3 Block Public Access enabled.

5. Review permissions regularly

Check:

IAM Policies
Bucket Policies
Public Access Settings
Object Access

regularly.

20. Security Verification Checklist

Complete the following checklist:

Training bucket identified

Correct AWS account verified

Correct AWS Region verified

Block Public Access reviewed

Bucket policy reviewed

Object Ownership reviewed

Object access reviewed

No public access intentionally configured

No passwords uploaded

No AWS credentials uploaded

No .env file uploaded

No API keys uploaded

No private company files uploaded

IAM permissions reviewed

Least privilege understood

IAM role usage understood

21. Screenshot Evidence

For the training report, capture screenshots of:

Screenshot 1 — Block Public Access
[Screenshot: S3 → Bucket → Permissions → Block Public Access]
Screenshot 2 — Bucket Policy
[Screenshot: S3 → Bucket → Permissions → Bucket Policy]

If there is no bucket policy, capture the appropriate empty/default state.

Screenshot 3 — Object Ownership
[Screenshot: S3 → Bucket → Permissions → Object Ownership]
Screenshot 4 — Bucket Objects
[Screenshot: S3 → Bucket → Objects]

Security warning: Never include passwords, access keys, secret keys, session tokens, API keys, or other credentials in screenshots.

22. Questions and Answers
Q1. What is IAM access to S3?

IAM access to S3 is the permission system that determines which IAM users, groups, or roles can perform specific S3 actions on specific resources.

Example:

IAM Role
   ↓
s3:GetObject
   ↓
Specific S3 Object
Q2. What is a bucket policy?

A bucket policy is a resource-based JSON policy attached to an S3 bucket. It can define which principals are allowed or denied access and which S3 actions they can perform.

Q3. Why is public access dangerous?

Public access can allow unauthorized internet users to access data.

This may cause:

Data leakage
Credential exposure
Privacy violations
Compliance problems
Financial and reputational damage
Q4. Why should private data remain private?

Private data may contain confidential business information, credentials, customer information, or other sensitive content.

Only authorized identities should be able to access it.

Q5. Why should applications use IAM roles where appropriate?

IAM roles avoid the need to store long-lived AWS access keys inside applications or servers.

Roles can provide temporary credentials and can be restricted to the minimum permissions required by the application.

23. What I Learned

During this task, I learned:

IAM permissions control access to S3.
Bucket policies are resource-based policies attached to S3 buckets.
S3 public access can create serious security risks.
Private data should remain accessible only to authorized identities.
Block Public Access helps prevent accidental public exposure.
Applications should generally use IAM roles rather than hard-coded long-lived AWS credentials.
Least privilege is an important AWS security principle.
Bucket policies and IAM policies must be reviewed carefully.
Credentials and sensitive files should never be uploaded to an S3 training bucket.
S3 security requires both access control and safe configuration.
24. Final Security Architecture
                         AWS
                          │
                          ↓
                         S3
                          │
                  Private Bucket
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
       Block Public   Encryption   Versioning
          Access
             │
             ↓
      IAM / Bucket Policies
             │
             ↓
     Authorized Users/Roles
             │
             ↓
      Minimum Required Access

For applications:

Application
     │
     ↓
  IAM Role
     │
     ↓
Temporary Credentials
     │
     ↓
Least-Privilege S3 Access
     │
     ↓
Private S3 Bucket
Conclusion

S3 security depends on controlling who can access buckets and objects and preventing accidental public exposure.

A secure training bucket should normally use:

Private Bucket
      ↓
Block Public Access
      ↓
Least-Privilege Permissions
      ↓
IAM Roles Where Appropriate
      ↓
Protected Data