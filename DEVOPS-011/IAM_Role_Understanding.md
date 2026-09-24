TASK 8 — IAM Role Understanding

File: IAM_Role_Understanding.md
Training Topic: AWS IAM Roles
Time: 45 Minutes

1. Objective

The objective of this task is to understand the difference between an IAM User and an IAM Role.

The main concept is:

IAM User
     ↓
Person / Application Identity


IAM Role
     ↓
Temporary / Assumable Identity

IAM roles are especially important for AWS services such as EC2, Lambda, and CI/CD systems.

2. What is an IAM User?

An IAM User represents a specific identity in an AWS account.

It can represent a person or, in some cases, an application identity.

Example:

Sathvik
   ↓
IAM User
   ↓
AWS Permissions

An IAM user can have permissions through:

Group membership
Identity-based policies
Other applicable IAM permission mechanisms

For human users, organizations should follow their approved identity-management approach and security policies.

3. What is an IAM Role?

An IAM Role is an identity that can be assumed by a trusted principal.

Unlike a traditional IAM user, a role does not have a permanent username/password combination for signing in.

A role can provide temporary credentials when it is assumed.

Example:

EC2 Instance
      ↓
Assumes IAM Role
      ↓
Temporary Credentials
      ↓
AWS Permissions

Roles are commonly used by:

EC2
Lambda
ECS
CI/CD systems
Applications
Federated users
Other AWS services
4. User vs Role
Feature	IAM User	IAM Role
Represents	Specific identity	Assumable identity
Typical use	Human identity	AWS workload/service access
Long-lived credentials	May have credentials depending on access method	Does not require permanent credentials
Temporary credentials	Not the defining model	Commonly provided when assumed
Password login	Can be configured for console access	No direct password login
Access keys	Can exist	Temporary credentials are issued when assumed
EC2 workload access	Not preferred	Recommended approach
Example	Training user	EC2 instance role
5. Why Use IAM Roles?

IAM roles allow AWS services and applications to obtain temporary credentials instead of storing long-lived AWS access keys.

For example:

EC2
 ↓
IAM Role
 ↓
Temporary Credentials
 ↓
S3

The EC2 application can then access the AWS service according to the permissions assigned to the role.

6. EC2 + IAM Role Architecture

A common AWS architecture is:

                  AWS Account
                       │
                       ▼
                IAM Role
             EC2-S3-Access-Role
                       │
                IAM Permissions
                       │
                       ▼
                      S3
                       ▲
                       │
                  EC2 Instance
                       │
                       └── Assumes Role

More simply:

EC2
 ↓
IAM Role
 ↓
AWS Permissions
 ↓
AWS Service
7. Practical Example

Suppose an EC2 server needs to read files from an S3 bucket.

Poor approach

An administrator creates an AWS access key and stores it on the EC2 server.

EC2
 │
 ├── Access Key ID
 └── Secret Access Key
          │
          ▼
          S3

For example, credentials might accidentally be placed in:

/home/ec2-user/.aws/credentials

or:

application configuration
environment variables
source code

This creates unnecessary credential-management risk.

8. Better Approach — IAM Role

Instead, create a role for EC2.

EC2
 │
 ▼
EC2 IAM Role
 │
 ▼
S3 Read Policy
 │
 ▼
S3 Bucket

The EC2 instance receives temporary credentials through the AWS instance metadata mechanism.

The application can use those credentials through the AWS SDK or AWS CLI without storing a long-lived access key in the application.

9. Why Long-Lived Access Keys Are Risky

Long-lived access keys create additional security risks.

For example:

Access Key
     ↓
Stored on EC2
     ↓
Server compromised
     ↓
Credential potentially exposed
     ↓
Unauthorized AWS access

Possible exposure locations include:

Source code
Configuration files
Shell history
Logs
Backups
Images
Git repositories
Environment files

If a long-lived credential is leaked, it may remain valid until it is disabled, rotated, or otherwise invalidated.

10. IAM Role Approach

With an IAM role:

EC2
 ↓
IAM Role
 ↓
Temporary Credentials
 ↓
AWS Service

The credentials are temporary and are automatically provided to the workload through the EC2 role mechanism.

This reduces the need for developers or administrators to manually place permanent credentials on the server.

11. Trust Policy vs Permissions Policy

An important IAM role concept is that a role has two different security questions.

Question 1 — Who can assume the role?

This is controlled by the trust policy.

Example concept:

Who can assume this role?
        ↓
EC2 service
Question 2 — What can the role do?

This is controlled by the role's permissions policies.

Example:

What can the role do?
        ↓
Read objects from S3

Therefore:

IAM Role
   │
   ├── Trust Policy
   │       ↓
   │   Who can assume?
   │
   └── Permissions Policy
           ↓
       What can it do?
12. Example Trust Policy

A simplified EC2 role trust policy conceptually allows the EC2 service to assume the role.

Example:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

Important elements:

Principal
     ↓
ec2.amazonaws.com

Action
     ↓
sts:AssumeRole

This establishes that EC2 is trusted to assume the role.

13. Example Permissions Policy

Suppose the EC2 instance only needs to read objects from a training bucket.

A permissions policy could conceptually allow:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::training-bucket/*"
    }
  ]
}

This means:

EC2
 ↓
Assumes Role
 ↓
Role allows s3:GetObject
 ↓
Specified S3 objects

Only the permissions actually required should be granted.

14. EC2 Instance Profile

For EC2, an IAM role is made available to the instance through an instance profile.

Conceptually:

IAM Role
    ↓
Instance Profile
    ↓
EC2 Instance

The EC2 instance can then obtain temporary credentials associated with the role.

For training purposes, remember:

EC2
 ↓
Instance Profile
 ↓
IAM Role
 ↓
Permissions
15. Example Scenario
Requirement

An EC2 application needs to read application configuration files stored in S3.

Design
                   EC2
                    │
                    ▼
              EC2 IAM Role
                    │
                    ▼
             S3 Read Policy
                    │
                    ▼
           Application Bucket
Required permission
s3:GetObject
Unnecessary permissions

The application should not automatically receive:

iam:*
s3:DeleteObject
ec2:TerminateInstances
s3:*

unless they are genuinely required by the application and authorized.

This is an example of least privilege.

16. User Credentials vs Role Credentials
IAM User

A user may have long-term credentials depending on how the user is configured.

IAM User
   ↓
Authentication
   ↓
AWS Permissions
IAM Role

A role is assumed by a trusted principal.

Principal
   ↓
Assume Role
   ↓
Temporary Credentials
   ↓
AWS Permissions

The role-based model is particularly useful for AWS workloads.

17. Why EC2 Should Generally Use a Role

An EC2 instance should generally use an IAM role instead of storing long-lived AWS access keys because:

1. Temporary credentials

The workload can receive temporary credentials rather than relying on permanent credentials stored on disk.

2. Reduced credential exposure

There is less need to place AWS secrets in:

Source code
Configuration files
Environment files
3. Easier credential management

AWS handles the availability and rotation of temporary credentials associated with the role.

4. Better security

If the application only receives the permissions it needs, the potential impact of compromise is reduced.

5. Easier administration

Administrators can change the role's policy instead of distributing new access keys to multiple servers.

18. Bad vs Good Architecture
Bad Architecture
Developer
    │
    ▼
Creates Access Key
    │
    ▼
Stores on EC2
    │
    ▼
Application
    │
    ▼
AWS Service

Problems:

Credential exposure
Credential rotation
Secret management
Potential accidental commits
Larger security risk
Good Architecture
EC2
 │
 ▼
IAM Role
 │
 ▼
Temporary Credentials
 │
 ▼
Limited Permissions
 │
 ▼
AWS Service

Advantages:

Temporary access
No hard-coded long-lived credentials
Centralized permissions
Least privilege
Easier management
19. DevOps Example

IAM roles are commonly used in DevOps environments.

For example:

CI/CD Pipeline
      ↓
IAM Role
      ↓
Temporary Credentials
      ↓
AWS

A deployment pipeline may need permission to:

Push container images
Deploy infrastructure
Update services
Read deployment status
Write logs

Instead of storing AWS access keys inside the pipeline configuration, the pipeline can use an appropriate role-based authentication mechanism.

20. Security Best Practices

Follow these practices when using IAM roles:

✓ Prefer IAM roles for AWS workloads
✓ Use least privilege
✓ Limit permissions to required actions
✓ Limit access to required resources
✓ Review the trust policy
✓ Review the permissions policy
✓ Avoid hard-coded AWS access keys
✓ Do not commit credentials to Git
✓ Remove unnecessary permissions
✓ Monitor role usage
21. Practical Training Exercise

For this task, study the following scenario:

Requirement:
EC2 needs to read files from S3.

Design:

EC2
 ↓
IAM Role
 ↓
S3 Read Policy
 ↓
S3 Bucket

Answer these questions:

Question 1

Why should EC2 use a role?

Answer:

Because the EC2 workload can obtain temporary AWS credentials through the role rather than requiring long-lived access keys to be stored on the server.

Question 2

Who can assume the role?

Answer:

The trust policy determines the trusted principal. For an EC2 instance role, the EC2 service is commonly trusted to assume the role.

Question 3

What can the role do?

Answer:

The attached permissions policies determine what AWS actions and resources the role can access.

Question 4

Should the role have administrator permissions?

Answer:

No, not unless the workload genuinely requires them and the organization explicitly authorizes them. Least privilege should be used.

22. Key Difference

Remember this simple comparison:

IAM USER
    ↓
Represents an identity
    ↓
Typically associated with a person or specific identity


IAM ROLE
    ↓
Assumable identity
    ↓
Provides temporary credentials
    ↓
Commonly used by AWS workloads
23. Final Architecture
                         AWS ACCOUNT
                              │
                              ▼
                         IAM ROLE
                    EC2-S3-Access-Role
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
              Trust Policy       Permissions Policy
                    │                   │
                    ▼                   ▼
                  EC2              S3:GetObject
                    │                   │
                    └─────────┬─────────┘
                              ▼
                         S3 Bucket
24. What I Learned

Through this task, I learned that an IAM user and IAM role serve different purposes.

An IAM user represents a specific identity, while an IAM role is an assumable identity that can provide temporary credentials.

The important EC2 pattern is:

EC2
 ↓
IAM Role
 ↓
Temporary Credentials
 ↓
AWS Permissions
 ↓
AWS Service

I also learned that EC2 instances should generally use IAM roles instead of storing long-lived AWS access keys inside the server.

This reduces credential-management risk and supports least-privilege access.