TASK 7 — Learn AWS IAM Policies

File: IAM_Policies.md
Training Topic: AWS IAM Policies
Time: 1 Hour

1. Objective

The objective of this task is to understand how AWS IAM policies control access to AWS resources.

An IAM policy defines:

Who can do what to which resource?

The three important concepts are:

Effect
Action
Resource

A simplified permission model is:

Principal / Identity
        │
        ▼
      Policy
        │
   ┌────┴────┐
   ▼         ▼
 Action    Resource
   │         │
   └────┬────┘
        ▼
      Effect
   Allow / Deny
2. What is an IAM Policy?

An IAM policy is a document that defines permissions.

It tells AWS whether an identity should be allowed or denied access to specific AWS actions on specific resources.

For example:

User
 ↓
IAM Policy
 ↓
Allow S3 Read
 ↓
S3 Bucket

A policy can control permissions such as:

Read an S3 object
Create an EC2 instance
Describe EC2 instances
Read CloudWatch logs
Access a specific DynamoDB table
3. Basic Policy Structure

A basic IAM policy uses JSON.

Example:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::training-bucket/*"
    }
  ]
}

The important fields are:

Effect
Action
Resource
4. Effect

The Effect specifies whether the requested action is allowed or denied.

There are two possible values:

Allow
Deny
4.1 Allow

Allow grants permission to perform an action.

Example:

{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::training-bucket/*"
}

This means:

Allow
  ↓
Get S3 Objects
  ↓
From the specified bucket
4.2 Deny

Deny explicitly prevents an action.

Example:

{
  "Effect": "Deny",
  "Action": "s3:DeleteObject",
  "Resource": "arn:aws:s3:::training-bucket/*"
}

This means:

Deny
  ↓
Delete S3 Objects
  ↓
From the specified bucket
5. Allow vs Deny
Feature	Allow	Deny
Purpose	Grants permission	Blocks permission
Result	Action can be performed	Action is blocked
Example	s3:GetObject	s3:DeleteObject
Security use	Grant required access	Explicitly prevent access

An important AWS IAM rule is:

Explicit Deny
      ↓
Overrides Allow

For example:

Policy A → Allow s3:DeleteObject
Policy B → Deny s3:DeleteObject

Result → DENIED

Therefore, adding an Allow does not override an applicable explicit Deny.

6. Action

The Action specifies what AWS operation the identity is allowed or denied to perform.

Examples:

s3:GetObject
s3:PutObject
s3:DeleteObject
ec2:DescribeInstances
ec2:StartInstances
ec2:StopInstances

The general format is:

service:operation

Examples:

s3:GetObject
│  │
│  └── Operation
└───── AWS Service

ec2:StartInstances
│   │
│   └── Operation
└────── AWS Service
7. Resource

The Resource specifies which AWS resource the permission applies to.

AWS resources are commonly identified using an ARN:

Amazon Resource Name

Example:

arn:aws:s3:::training-bucket/*

This identifies objects inside the training-bucket.

Another example:

arn:aws:s3:::training-bucket

identifies the S3 bucket itself.

8. The Main Question

When reading an IAM policy, ask:

WHO
 │
 ▼
can do
 │
 ▼
WHAT
 │
 ▼
to WHICH RESOURCE?

In policy terms:

Who?
  ↓
Identity / Principal

What?
  ↓
Action

Which resource?
  ↓
Resource

Allowed or denied?
  ↓
Effect

Example:

User
  │
  └── can
       │
       └── GetObject
             │
             └── from training-bucket

Policy:

{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::training-bucket/*"
}
9. Understanding a Complete Policy

Consider:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::training-bucket",
        "arn:aws:s3:::training-bucket/*"
      ]
    }
  ]
}

Break it down:

Version
2012-10-17

Identifies the IAM policy language version.

Statement

Contains the permission statements.

Effect
Allow

The actions are permitted.

Action
s3:GetObject
s3:ListBucket

The identity can read objects and list the bucket, subject to the resource scope and any other applicable policies.

Resource
training-bucket
training-bucket/*

The permissions apply to the specified bucket and its objects.

10. Policy Statement Structure

A policy can contain multiple statements.

Example:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::training-bucket/*"
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::training-bucket/*"
    }
  ]
}

This means:

Get objects
     ↓
Allowed

Delete objects
     ↓
Denied
11. What is a Wildcard?

The * symbol can represent multiple matching values.

Example:

s3:Get*

can match multiple S3 actions beginning with Get.

A resource such as:

arn:aws:s3:::training-bucket/*

refers to objects under that bucket.

Wildcards should be used carefully because overly broad wildcards can grant more access than intended.

12. Least Privilege

IAM policies should follow the principle of least privilege.

Instead of:

{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}

a better approach is to grant only the required action and resource.

For example:

{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::training-bucket/*"
}

The second policy is much more restricted.

13. AWS Managed Policies

AWS provides AWS managed policies that are created and maintained by AWS.

These policies can provide predefined permissions for common use cases.

Examples include policies for:

Amazon S3
Amazon EC2
AWS Lambda
CloudWatch
Read-only access
Full service access

Examples of AWS managed policy names include:

AmazonS3ReadOnlyAccess
AmazonEC2ReadOnlyAccess
AWSLambdaBasicExecutionRole

The exact permissions should always be reviewed before attaching a managed policy.

14. How to Examine an AWS Managed Policy

To examine a managed policy in the AWS Console:

AWS Console
     ↓
IAM
     ↓
Policies
     ↓
Search policy

For example, search for:

AmazonS3ReadOnlyAccess

Open the policy and examine:

Policy name
Policy type
Permissions
JSON

Select the JSON view to understand what actions and resources are specified.

15. Example: AmazonS3ReadOnlyAccess

When examining an S3 read-only managed policy, look for permissions related to reading or listing S3 resources.

Conceptually:

AmazonS3ReadOnlyAccess
          │
          ▼
     S3 Read Access
          │
     ┌────┴────┐
     ▼         ▼
 List        Read
 Bucket      Objects

The important lesson is not just the policy name.

You should inspect the actual policy document and identify:

Effect
Action
Resource

Then explain what those permissions allow.

16. Managed vs Customer Managed Policies

There are two important policy categories to understand.

AWS Managed Policy

Created and maintained by AWS.

AWS
 ↓
Creates Policy
 ↓
Maintains Policy
Customer Managed Policy

Created and maintained by the AWS customer/organization.

Organization
      ↓
Creates Policy
      ↓
Controls Policy
      ↓
Attaches to Identities

Customer managed policies are useful when an organization needs permissions specifically designed for its environment.

17. Managed vs Inline Policies

Another distinction is:

Managed Policy

A reusable policy that can be attached to multiple identities or resources where supported.

Policy
  │
  ├── User A
  ├── User B
  └── Group
Inline Policy

A policy embedded directly into a single identity or resource.

For training and centralized permission management, managed policies are often easier to manage and review.

18. Policy Evaluation Example

Consider:

User:
sathvik-training

Group:
devops-trainees

Policy:
Training-S3-ReadOnly

Policy:

{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::training-bucket/*"
}

User attempts:

s3:GetObject

Result:

Action matches
      ↓
Resource matches
      ↓
Effect = Allow
      ↓
Access permitted

If the user attempts:

s3:DeleteObject

there is no corresponding Allow in this example.

Therefore, assuming no other applicable policy grants it:

No applicable Allow
      ↓
Access denied
19. Explicit Deny Example

Suppose one policy allows:

{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::training-bucket/*"
}

Another applicable policy contains:

{
  "Effect": "Deny",
  "Action": "s3:DeleteObject",
  "Resource": "arn:aws:s3:::training-bucket/*"
}

The user attempts:

Delete Object

Result:

Allow
  +
Explicit Deny
  ↓
DENIED

This demonstrates the importance of explicit Deny.

20. Policy Analysis Checklist

When examining any IAM policy, ask:

1. What is the Effect?
2. What Actions are allowed or denied?
3. What Resources are affected?
4. Who receives this policy?
5. Is the policy broader than necessary?
6. Are wildcards being used?
7. Is there an explicit Deny?
8. Does the policy follow least privilege?
21. Example Policy Analysis
Policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeVolumes"
      ],
      "Resource": "*"
    }
  ]
}
Analysis

Effect:

Allow

The actions are permitted.

Actions:

ec2:DescribeInstances
ec2:DescribeVolumes

The identity can retrieve information about EC2 instances and EBS volumes.

Resource:

*

The policy uses a wildcard resource. Some AWS read/describe APIs require "Resource": "*" because they do not support resource-level permissions for that action.

Therefore, least privilege is not simply about avoiding every *; it also requires understanding what AWS supports for the specific API action.

22. IAM Policy in DevOps

IAM policies are extremely important in DevOps.

Examples:

Developer
    ↓
IAM Permissions
    ↓
AWS Resources

A CI/CD pipeline might need permission to:

Read source artifacts
Push images
Deploy infrastructure
Read deployment status
Write logs

Instead of giving the pipeline administrator permissions, create a role or policy with only the actions required by the pipeline.

Example:

CI/CD Pipeline
      ↓
IAM Role
      ↓
Limited Policy
      ↓
Required AWS Services

This reduces the potential impact of compromised credentials or incorrect automation.

23. Security Best Practices

Follow these IAM policy practices:

✓ Use least privilege
✓ Grant only required actions
✓ Limit resources where possible
✓ Avoid unnecessary wildcards
✓ Review managed policies before using them
✓ Prefer roles for AWS workloads
✓ Avoid hard-coded credentials
✓ Monitor and review permissions
✓ Use explicit Deny where an organization requires a hard restriction

Do not assume that a policy is safe merely because its name contains words such as:

ReadOnly
PowerUser
FullAccess

Always examine what permissions the policy actually provides.

24. Practical Training Exercise

For this task, examine an AWS managed policy in the IAM Console.

Step 1

Open:

IAM
 ↓
Policies
Step 2

Search for:

AmazonS3ReadOnlyAccess
Step 3

Open the policy.

Step 4

Select the JSON representation.

Step 5

Identify:

Effect
Action
Resource
Step 6

Write your explanation:

Policy Name:
AmazonS3ReadOnlyAccess

Effect:
Allow

Service:
Amazon S3

Purpose:
Provides read-oriented access to S3 resources according to
the actions defined in the policy.

Security consideration:
The policy should be reviewed before use to determine whether
its permissions are appropriate for the training environment.
25. Screenshot Evidence

Capture screenshots of the AWS managed policy you examined.

Screenshot 1 — IAM Policies
[INSERT SCREENSHOT — IAM POLICIES PAGE]
Screenshot 2 — Managed Policy Details

Show:

Policy Name
Policy Type
Permissions
[INSERT SCREENSHOT — POLICY DETAILS]
Screenshot 3 — JSON Policy

Show the policy JSON containing:

Effect
Action
Resource
[INSERT SCREENSHOT — POLICY JSON]

Do not include passwords, access keys, secret keys, session tokens, or other credentials in screenshots.

26. Key Concepts Learned
IAM Policy
    │
    ├── Effect
    │      ├── Allow
    │      └── Deny
    │
    ├── Action
    │      └── What can be done?
    │
    └── Resource
           └── Which resource?

The core question is:

WHO
 │
 ▼
CAN DO WHAT
 │
 ▼
TO WHICH RESOURCE?
27. Final Summary

An IAM policy is a set of rules that controls access to AWS resources.

The three fundamental elements are:

Effect
Action
Resource

Allow grants permission, while an applicable explicit Deny blocks access and overrides an Allow.

When analyzing a policy, always determine:

Who?
 ↓
What action?
 ↓
Which resource?
 ↓
Allowed or denied?

The principle of least privilege should be applied whenever permissions are designed.

For AWS managed policies, do not rely only on the policy name. Open the policy and examine its actual permissions.