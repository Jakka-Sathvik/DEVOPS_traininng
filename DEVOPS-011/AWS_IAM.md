Task 4 — AWS IAM

Training: DevOps Engineer Training
Topic: Identity and Access Management (IAM)
Day: 11
Trainee: Sathvik Jakka

1. What is IAM?

IAM (Identity and Access Management) is an AWS service used to securely control access to AWS resources.

IAM answers two important questions:

Who are you?
     ↓
Authentication

What are you allowed to do?
     ↓
Authorization

For example, an organization may have:

Developers
DevOps engineers
Administrators
Applications
Automated deployment systems

Each of them may require different permissions.

IAM allows an organization to control these permissions.

The main IAM concepts are:

IAM
 │
 ├── Users
 │
 ├── Groups
 │
 ├── Roles
 │
 └── Policies
2. IAM Users

An IAM User represents a specific identity that can interact with AWS.

A user can represent an individual person or, in some cases, a specific long-lived identity.

For example:

AWS Account
     │
     ├── Sathvik
     ├── Developer
     └── DevOps Engineer

An IAM user can have permissions through policies or group membership.

Depending on the use case, an IAM user may have:

Console access
Programmatic access
Permissions assigned directly
Membership in IAM groups
Example

Suppose a developer needs permission to work with S3.

The organization could create an IAM user and grant the appropriate S3 permissions.

IAM User
   │
   └── S3 Permissions
          │
          ├── List
          └── Read

The permissions should only be as broad as required.

3. IAM Groups

An IAM Group is a collection of IAM users.

Groups make permission management easier because permissions can be assigned to the group rather than individually configuring every user.

Example
Developers Group
       │
       ├── User A
       ├── User B
       └── User C

The organization can assign appropriate policies to the Developers group.

All users who belong to that group can receive the permissions associated with the group's policies.

Example
Developers Group
       │
       └── S3 Development Policy
                │
        ┌───────┼───────┐
        │       │       │
      User A  User B  User C

This is easier to manage than individually attaching the same permissions to every developer.

4. IAM Roles

An IAM Role is an identity with permissions that can be assumed by a trusted principal.

Roles are commonly used by:

AWS services
Applications
EC2 instances
Lambda functions
CI/CD systems
Federated users
Other AWS accounts

A role does not represent a permanent individual user in the same way an IAM user does.

Instead, an authorized principal can assume the role and obtain temporary credentials.

Example: EC2 Role

Suppose an EC2 instance needs to read objects from S3.

Instead of putting long-lived AWS credentials inside the server, an IAM role can be attached to the EC2 instance.

EC2 Instance
      │
      ↓
IAM Role
      │
      ↓
S3 Read Permission
      │
      ↓
S3 Bucket

This is a common and safer pattern.

5. IAM Policies

An IAM Policy is a document that defines permissions.

Policies specify what actions are allowed or denied on which resources.

A policy can define concepts such as:

Effect
Action
Resource
Condition

A simplified example:

{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::example-bucket/*"
}

This means the identity can be allowed to perform the s3:GetObject action against objects in the specified bucket.

Policy Structure
Policy
 │
 ├── Effect
 │     ├── Allow
 │     └── Deny
 │
 ├── Action
 │
 ├── Resource
 │
 └── Condition

Policies are the mechanism used to define what actions identities can perform.

6. IAM Architecture

The overall relationship can be understood as:

                         IAM
                          │
          ┌───────────────┼───────────────┐
          │               │               │
        Users          Groups           Roles
          │               │               │
          │               │               │
          └───────────────┼───────────────┘
                          │
                       Policies
                          │
                          ↓
                   AWS Permissions
                          │
            ┌─────────────┼─────────────┐
            │             │             │
           EC2           S3            RDS
7. Difference Between IAM User and IAM Role

IAM Users and IAM Roles are both identities in AWS, but they are used differently.

Feature	IAM User	IAM Role
Represents	A specific identity	An assumable identity
Credentials	Can have long-term credentials	Uses temporary credentials when assumed
Common use	Individual identity	Applications/services/temporary access
Used by EC2	Generally not recommended	Recommended
Used by Lambda	Not normally needed	Common
Access method	User authenticates	Principal assumes role
Permissions	Policies	Policies
Typical use	Human identity	AWS service/application access
8. IAM User Example

Imagine a company has a developer named Developer A.

Developer A
     ↓
IAM User
     ↓
Developer Group
     ↓
S3 Development Policy
     ↓
S3

The developer authenticates as their identity and receives the permissions associated with their user/group.

9. IAM Role Example

Now suppose an EC2 server needs to access an S3 bucket.

A poor design would be:

EC2
 ↓
Hard-coded AWS Access Key
 ↓
S3

A better design is:

EC2
 ↓
IAM Role
 ↓
Temporary Credentials
 ↓
S3

This avoids placing long-lived access keys directly in the application or server.

10. Why Should Permissions Follow Least Privilege?

Least privilege means giving an identity only the permissions required to perform its job, and no more.

For example, suppose an application only needs to read objects from one S3 bucket.

It should not receive:

AdministratorAccess

Instead, it should receive only the required S3 read permission.

Too Much Permission
Application
    ↓
AdministratorAccess
    ↓
Entire AWS Account

This creates unnecessary security risk.

Least Privilege
Application
    ↓
IAM Role
    ↓
S3 Read Policy
    ↓
Specific Bucket

The second approach limits the potential impact of compromised credentials or applications.

11. Example of Least Privilege

Suppose a DevOps application needs to read deployment artifacts from:

deployment-artifacts

It may only need:

s3:GetObject

on objects in that bucket.

It does not necessarily need:

s3:DeleteObject
s3:PutObject
s3:ListAllMyBuckets
ec2:TerminateInstances
iam:CreateUser

Granting unrelated permissions violates the principle of least privilege.

12. Benefits of Least Privilege
12.1 Reduces Security Risk

If credentials are compromised, the attacker has fewer permissions available.

12.2 Limits Accidental Changes

A user with only required permissions is less likely to accidentally modify unrelated resources.

12.3 Limits Blast Radius

If an application or identity is compromised, restricting its permissions limits the potential damage.

Limited Permissions
       ↓
Limited Access
       ↓
Limited Impact
12.4 Easier Auditing

Specific permissions make it easier to understand what an identity can access.

13. IAM in DevOps

IAM is extremely important in DevOps because automated systems frequently interact with AWS.

For example:

Developer
    ↓
GitHub
    ↓
CI/CD Pipeline
    ↓
AWS
    ↓
Deployment

The pipeline may need permissions to:

Upload artifacts
Push container images
Deploy applications
Update infrastructure

Instead of giving the pipeline full administrator access, a dedicated IAM role or appropriately scoped identity should be used.

14. IAM with Terraform

Terraform also interacts with AWS APIs.

A simplified workflow is:

Terraform
    ↓
AWS Authentication
    ↓
IAM Permissions
    ↓
AWS API
    ↓
Infrastructure

Terraform may need permissions to create resources such as:

VPC
Subnet
Security Group
EC2
S3
IAM

The permissions should be carefully controlled according to the infrastructure Terraform is responsible for managing.

15. IAM Security Best Practices

Important IAM practices include:

1. Follow Least Privilege

Give only required permissions.

2. Prefer Roles for AWS Workloads

Use IAM roles for EC2, Lambda, and other AWS services where appropriate.

3. Avoid Hard-Coded Credentials

Do not put AWS access keys directly inside:

Source code
Dockerfiles
Git repositories
Terraform files
4. Protect the Root User

The AWS account root user has extensive privileges and should not be used for routine tasks.

5. Use MFA

Multi-factor authentication adds an additional authentication factor.

6. Review Permissions

Regularly review unused and excessive permissions.

7. Monitor Access

Use appropriate AWS logging and security services to identify suspicious activity.

16. IAM Example Architecture

A practical organization might have:

                         AWS Account
                              │
                             IAM
                              │
             ┌────────────────┼────────────────┐
             │                │                │
          Users            Groups            Roles
             │                │                │
             │                │                │
       Developer A       Developers       EC2 Role
       Developer B       DevOps           Lambda Role
       Developer C                        CI/CD Role
             │                │                │
             └────────────────┼────────────────┘
                              │
                           Policies
                              │
                    ┌─────────┼─────────┐
                    │         │         │
                   S3        EC2       RDS
17. Key Differences
User
Specific identity
       ↓
Usually represents a person or long-lived identity
Group
Collection of users
       ↓
Simplifies permission management
Role
Assumable identity
       ↓
Commonly used by AWS services and applications
Policy
Permission document
       ↓
Defines allowed/denied actions
18. Easy Way to Remember IAM
USER
Who is the identity?

GROUP
Which users belong together?

ROLE
Who/what can temporarily assume this identity?

POLICY
What is the identity allowed to do?

Or:

IAM
 │
 ├── User  → Identity
 │
 ├── Group → Collection of Users
 │
 ├── Role  → Assumable Identity
 │
 └── Policy → Permissions
19. Final Answers
1. What is IAM?

IAM is AWS's identity and access management service used to control authentication and authorization for AWS resources.

2. What is an IAM User?

An IAM User is an AWS identity representing a specific user or long-lived identity that can authenticate and receive permissions.

3. What is an IAM Group?

An IAM Group is a collection of IAM users used to manage common permissions efficiently.

4. What is an IAM Role?

An IAM Role is an identity that can be assumed by an authorized principal and provides temporary credentials and permissions.

5. What is an IAM Policy?

An IAM Policy is a permission document that defines which actions are allowed or denied on specified AWS resources.

6. Difference Between User and Role
IAM User
   ↓
Specific identity
   ↓
Commonly used for human identities

IAM Role
   ↓
Assumable identity
   ↓
Commonly used by AWS services,
applications, and temporary access
7. Why Least Privilege?

Least privilege ensures that users, applications, and services receive only the permissions they actually need. This reduces security risk, limits accidental changes, and reduces the potential impact if an identity is compromised.

20. Task Deliverable

Suggested file name:

AWS_IAM.md
Completed Topics
✓ IAM
✓ IAM Users
✓ IAM Groups
✓ IAM Roles
✓ IAM Policies
✓ User vs Role
✓ Least Privilege
✓ IAM Architecture
✓ IAM Security Best Practices
✓ IAM in DevOps
✓ IAM with Terraform

Status: Completed