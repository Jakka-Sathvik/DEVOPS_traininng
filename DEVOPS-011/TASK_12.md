TASK 12 — Day 11 Practical Challenge

Time: 1 Hour
Topic: Safe AWS DevOps Training Identity and CLI Verification
Deliverable: Practical explanation and documentation

1. Challenge Requirement

Set up a safe AWS DevOps training identity, understand its permissions, configure approved CLI access, verify the AWS identity, and document the complete setup without exposing credentials.

The trainee must explain every step of the process.

2. Expected Workflow
AWS Account
     ↓
IAM
     ↓
Training Identity
     ↓
Group / Role
     ↓
Least-Privilege Permissions
     ↓
AWS CLI
     ↓
Identity Verification
     ↓
Documentation
3. Step 1 — AWS Account

First, identify the AWS account being used for training.

The trainee should understand:

Which AWS account is being used.
Whether it is a personal, training, or organizational account.
Whether the trainee has authorization to perform IAM configuration.
Which AWS Region will be used for the training exercises.
Trainee Explanation

The AWS account provides the overall environment in which AWS resources and IAM identities are managed.

The trainee should confirm that they are working in the correct account before creating users, groups, roles, or resources.

4. Step 2 — IAM

Open AWS IAM and understand its purpose.

The trainee should explain:

IAM
 ├── Users
 ├── Groups
 ├── Roles
 └── Policies

IAM controls:

Who can access AWS?
        ↓
What can they do?
        ↓
Which resources can they access?

The trainee must understand the difference between:

Authentication
Authorization
User
Group
Role
Policy
5. Step 3 — Create or Identify a Training Identity

Create or identify a dedicated training identity according to the organization's AWS account policy.

The training identity should not be confused with the AWS root account.

The trainee should explain:

Root Account
     ↓
Avoid for normal daily administration

Training Identity
     ↓
Used for authorized training activities

The trainee must understand why a dedicated training identity is safer and easier to manage than using the root account for everyday work.

6. Step 4 — Group / Role

Determine whether the training identity should receive permissions through an IAM group, role, or another approved access mechanism.

For example:

Training User
      ↓
devops-trainees
      ↓
Training Permissions

or, for a workload:

AWS Workload
      ↓
IAM Role
      ↓
Temporary Credentials
      ↓
AWS Permissions

The trainee must explain why the selected access model is appropriate.

7. Step 5 — Least-Privilege Permissions

Determine exactly what permissions are required for the training activities.

The trainee should ask:

What service is required?
        ↓
What action is required?
        ↓
Which resource is required?
        ↓
What is the minimum permission needed?

For example, if an exercise only requires reading information from an AWS service, the identity should not automatically receive full administrative permissions.

Important Principle

Give an identity only the permissions required to perform its intended task.

Avoid using broad administrator permissions simply because they are easier.

8. Step 6 — Understand the Policy

The trainee must inspect the permissions assigned to the training identity.

For an IAM policy, understand:

Effect
Action
Resource
Condition

The trainee should be able to answer:

Who?
  ↓
can do what?
  ↓
to which resource?

The trainee should also understand:

Allow grants permission.
An applicable explicit Deny overrides an Allow.
Permissions should be restricted to the required actions and resources whenever possible.
Some AWS APIs require broad resource specifications because resource-level permissions are not supported.
9. Step 7 — Configure Approved AWS CLI Access

Configure AWS CLI access using an authentication method approved by the organization.

Possible approved methods may include:

IAM Identity Center / SSO
        ↓
Temporary credentials
        ↓
Approved IAM user credentials
        ↓
IAM role
        ↓
Other organization-approved authentication

The trainee must understand that authentication establishes who the caller is.

Authorization determines what that identity is allowed to do.

10. Step 8 — Verify the AWS Identity

After configuring the CLI, verify the identity being used.

The trainee should use the appropriate AWS STS identity-verification mechanism.

The trainee must explain the information returned, including:

UserId
Account
Arn

The trainee should understand that this verification answers:

Who am I?
        ↓
Which AWS account am I using?
        ↓
Which AWS principal am I using?

This is especially important before performing operations that modify AWS resources.

11. Step 9 — Verify the Configured Region

The trainee must identify the AWS CLI's configured/default Region.

Explain:

AWS CLI
   ↓
Configured Region
   ↓
Default AWS location for applicable commands

The trainee should understand that a Region can also be explicitly selected for an individual AWS CLI operation when appropriate.

12. Step 10 — Test Authorized Access

Perform a safe test using an AWS service for which the training identity has permission.

The trainee should:

Select an authorized operation.
Run the operation through the approved CLI authentication.
Observe the result.
Explain why the operation succeeded or failed.
Relate the result to the IAM permissions.

Do not perform destructive operations simply to test access.

13. Step 11 — Understand AccessDenied

If an operation fails with an authorization error, the trainee should investigate rather than immediately requesting administrator access.

Use this troubleshooting process:

Operation Failed
      ↓
Read Error
      ↓
Identify AWS Service
      ↓
Identify Required Action
      ↓
Check Identity
      ↓
Check IAM Policy
      ↓
Check Resource
      ↓
Check Region / Configuration
      ↓
Correct Authorized Configuration
      ↓
Test Again

The trainee should be able to explain the root cause of the failure.

14. Step 12 — Document the Complete Setup

Create documentation explaining the complete workflow.

The documentation should contain:

AWS Account
Training Account:
________________________

Do not publish sensitive account information unnecessarily.

Training Identity
Identity Type:
________________________

Identity Name:
________________________
Access Model
User / Group / Role:
________________________
Permissions

Explain:

Service:
________________________

Required Actions:
________________________

Resource Scope:
________________________

Reason:
________________________
CLI Authentication

Document the approved authentication method without documenting secret credentials.

Identity Verification

Record that the identity verification was successful.

Region
Configured Region:
________________________
15. Security Requirements

The trainee must never document or share:

AWS Secret Access Keys
Passwords
Session Tokens
Private Keys
Credential Files
MFA recovery information

Do not put credentials into:

GitHub
README files
Source code
Screenshots
Chat messages
Training documents
Public repositories

If screenshots are required, redact sensitive information before submitting them.

16. Screenshot Requirements

If screenshots are included, useful screenshots may show:

1. IAM training identity
2. Group or role
3. Attached permissions
4. Policy details
5. AWS CLI identity verification
6. Configured region
7. Successful authorized test

Do not capture or submit:

Passwords
Secret Access Keys
Session Tokens
Private Keys
Credential files
17. Trainee Explanation

The trainee must be able to explain the following workflow verbally:

AWS Account
     ↓
IAM
     ↓
Training Identity
     ↓
Group / Role
     ↓
Least-Privilege Policy
     ↓
Approved Authentication
     ↓
AWS CLI
     ↓
Identity Verification
     ↓
Authorized AWS Operation
     ↓
Documentation

The trainee should explain why each step exists, not simply state that the step was completed.

18. Questions the Trainee Must Answer
Question 1

Why should the root account not be used for normal daily administration?

Expected understanding: The root account has very broad privileges and should be protected and reserved for tasks that specifically require root access.

Question 2

What is the purpose of IAM?

Expected understanding: IAM controls authentication and authorization for access to AWS resources.

Question 3

Why use a group?

Expected understanding: A group allows permissions to be managed consistently for multiple IAM users.

Question 4

When should a role be used?

Expected understanding: Roles are assumable identities and are commonly used by AWS workloads and users/services that need temporary credentials.

Question 5

What does least privilege mean?

Expected understanding: Grant only the permissions necessary to perform the required task.

Question 6

What does an IAM policy define?

Expected understanding: A policy defines permissions using elements such as Effect, Action, Resource, and optionally Condition.

Question 7

How do you verify which AWS identity the CLI is using?

Expected understanding: Use AWS STS caller-identity verification and understand the returned identity information.

Question 8

How do you identify the configured AWS Region?

Expected understanding: Use the appropriate AWS CLI configuration query for the default Region.

Question 9

What should you do if AWS returns AccessDenied?

Expected understanding: Check the authenticated identity, required action, IAM permissions, resource scope, and configuration instead of automatically granting administrator access.

Question 10

Why should credentials never be included in documentation?

Expected understanding: Exposed credentials can allow unauthorized access to AWS resources.

19. Final Architecture
                         AWS ACCOUNT
                              │
                              ▼
                             IAM
                              │
                              ▼
                     TRAINING IDENTITY
                              │
                       ┌──────┴──────┐
                       ▼             ▼
                     GROUP          ROLE
                       │             │
                       └──────┬──────┘
                              ▼
                    LEAST-PRIVILEGE
                       PERMISSIONS
                              │
                              ▼
                    APPROVED AUTHENTICATION
                              │
                              ▼
                         AWS CLI
                              │
                              ▼
                    IDENTITY VERIFICATION
                              │
                              ▼
                    AUTHORIZED OPERATION
                              │
                              ▼
                       DOCUMENTATION