TASK 5 — IAM User Setup for Training

File: IAM_User_Setup.md
Training Topic: AWS Identity and Access Management (IAM)
Purpose: Create a dedicated IAM identity for training without using the AWS root account for normal administration.

1. Objective

The objective of this task is to create a dedicated IAM user for AWS training and assign permissions through an IAM group.

The basic permission structure is:

IAM User
    ↓
IAM Group
    ↓
IAM Policy
    ↓
AWS Permissions

This approach helps organize permissions and makes it easier to manage access.

Security Rule: Do not use the AWS root account for normal daily administration. Use a dedicated IAM identity or an organization-approved identity-management solution.

2. IAM User Setup Architecture
                    AWS Account
                         │
                         ▼
                  IAM Training User
                         │
                         ▼
                  Training Group
                         │
                         ▼
                  IAM Policy
                         │
                         ▼
              AWS Service Permissions
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
            S3                     EC2
       (if permitted)          (if permitted)

The exact AWS services and permissions should follow the organization's AWS account policy.

3. Before Creating the User

Before creating an IAM user, verify:

AWS account access is available.
You are authorized to create IAM identities.
You are not using the root account for routine administration.
Your organization does not require AWS IAM Identity Center instead of IAM users.
The required training permissions are known.
No passwords, access keys, secret keys, or other credentials will be included in screenshots or documentation.
4. Create the IAM Group

A group allows permissions to be managed centrally for multiple users.

Step 1 — Open AWS IAM

Sign in to the AWS Management Console using your authorized administrative identity.

Open:

AWS Console
    ↓
IAM
    ↓
User groups
Step 2 — Create a Group

Select:

Create group

Use a clear training-related name, for example:

DevOpsTrainingGroup

or:

AWS-Training-Users

Choose a name according to your organization's naming standard.

Step 3 — Assign Permissions

Attach only the policies required for the training.

For example, if the training specifically requires S3 access, an organization-approved S3 policy may be assigned.

For a controlled training environment, permissions should be limited to the services and actions actually required.

Avoid automatically assigning broad administrator permissions unless your organization explicitly requires it.

Example:

DevOpsTrainingGroup
        │
        └── Training Policy
                │
                ├── Required AWS Service
                └── Required Actions

Click:

Create group
5. Create the IAM User

Navigate to:

IAM
 ↓
Users
 ↓
Create user

Enter a training-specific username.

Example:

sathvik-training

Use your organization's naming convention if one exists.

6. Configure User Access

AWS may provide different access options depending on the account configuration.

For training, configure only the access required by the assignment.

Possible access types include:

AWS Management Console access

or programmatic access where specifically required.

If console access is required, configure the sign-in credentials according to the organization's security policy.

Important

Never document or screenshot:

Password
Access Key ID
Secret Access Key
Session Token
MFA recovery information

These are credentials or sensitive authentication information.

7. Add the User to the Training Group

During user creation, select:

Add user to group

Select:

DevOpsTrainingGroup

The resulting relationship is:

sathvik-training
       │
       ▼
DevOpsTrainingGroup
       │
       ▼
Training Policy
       │
       ▼
AWS Permissions

Complete the user creation process.

8. Verify the User

After creating the user, open:

IAM
 ↓
Users
 ↓
sathvik-training

Verify:

User exists.
Correct group membership is shown.
Expected policies are attached through the group.
No unnecessary policies are attached directly to the user.

Example:

User:
sathvik-training

Groups:
DevOpsTrainingGroup

Permissions:
Training Policy
9. Understanding the Permission Flow

The permission model created in this task is:

User
 ↓
Group
 ↓
Policy
 ↓
Permission
User

The IAM user represents a specific identity.

Example:

sathvik-training
Group

The group organizes users with similar permissions.

Example:

DevOpsTrainingGroup
Policy

The policy defines what actions are allowed or denied.

Example:

Allow:
    Required training actions

Resource:
    Required AWS resources
Permission

The effective permission determines what AWS operations the user can perform.

10. Why Use a Group?

Using a group makes permission management easier.

For example:

DevOpsTrainingGroup
        │
        ├── User A
        ├── User B
        └── User C

A policy can be attached to the group instead of repeatedly attaching the same policy to individual users.

If the training permission needs to change:

Change Group Policy
        ↓
Users receive updated permissions

This improves consistency and simplifies administration.

11. Least Privilege

Permissions should follow the principle of least privilege.

Least privilege means:

Give an identity only the permissions required to perform its intended task.

For example, if a training exercise only requires reading objects from a specific S3 bucket, the user should not automatically receive permissions to administer every AWS service.

Poor approach:

Training User
      ↓
AdministratorAccess
      ↓
All AWS Services

Better approach:

Training User
      ↓
Training Group
      ↓
Limited Training Policy
      ↓
Required Service + Required Actions

This reduces the potential impact of accidental or unauthorized actions.

12. Example Permission Design

Suppose the training requires access to a specific S3 training bucket.

A controlled design could look like:

IAM User
    ↓
DevOpsTrainingGroup
    ↓
S3 Training Policy
    ↓
Specific S3 Actions
    ↓
Specific Training Bucket

For example, permissions might be limited to actions such as:

s3:GetObject
s3:ListBucket

on the required training resources.

The exact policy should be determined by the training requirements and organization policy.

13. Security Considerations

Follow these security practices:

1. Do not use root for daily administration

The root account has extensive privileges and should be protected carefully.

2. Use least privilege

Do not grant permissions that are not required.

3. Protect credentials

Never share:

Passwords
Access keys
Secret keys
Session tokens
4. Use MFA where required

Follow the organization's MFA requirements.

5. Avoid unnecessary access keys

For human users, use the organization's approved authentication method. Do not create long-lived access keys unless they are specifically required and permitted.

6. Review permissions

Regularly verify that the training identity has only the permissions it needs.

7. Delete unused training identities

After training is completed, remove or disable resources according to organizational policy.

14. Screenshot Evidence

Take screenshots of the relevant IAM configuration.

Screenshot 1 — IAM User

Capture the IAM user page showing:

User name
User ARN/identifier if appropriate
Group membership

Do not expose credentials.

Insert screenshot here:

[INSERT SCREENSHOT — IAM USER]
Screenshot 2 — IAM Group

Capture:

IAM
 ↓
User groups
 ↓
DevOpsTrainingGroup

The screenshot should show the group name and associated permission configuration.

Insert screenshot here:

[INSERT SCREENSHOT — IAM GROUP]
Screenshot 3 — Permissions

Capture the permissions/policies associated with the training group.

Insert screenshot here:

[INSERT SCREENSHOT — IAM POLICY/PERMISSIONS]
Screenshot 4 — User Group Membership

Capture the user's group membership.

Example:

User:
sathvik-training

Group:
DevOpsTrainingGroup

Insert screenshot here:

[INSERT SCREENSHOT — GROUP MEMBERSHIP]
15. Credential Safety Checklist

Before submitting screenshots, verify:

[ ] No password is visible
[ ] No Access Key ID is visible
[ ] No Secret Access Key is visible
[ ] No session token is visible
[ ] No MFA recovery information is visible
[ ] No unnecessary account secrets are visible
[ ] Only relevant IAM configuration is shown

If sensitive information accidentally appears in a screenshot, remove or redact it before submitting the document.

16. Verification

Verify the final configuration:

IAM User
   ↓
Training Group
   ↓
Training Policy
   ↓
Required AWS Permissions

Checklist:

[✓] Dedicated training identity created
[✓] Training group created
[✓] User added to training group
[✓] Required policy attached
[✓] Permissions reviewed
[✓] Least privilege considered
[✓] Credentials protected
[✓] Screenshots captured safely
17. Final Architecture
                    AWS ACCOUNT
                        │
                        ▼
                IAM Training User
                        │
                        │ Member of
                        ▼
              DevOpsTrainingGroup
                        │
                        │ Attached Policy
                        ▼
                Training IAM Policy
                        │
                        ▼
              Required Permissions
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
        AWS Service A          AWS Service B
       (if required)          (if required)
18. What I Learned

Through this task, I learned how IAM users, groups, policies, and permissions work together.

The main permission relationship is:

User
 ↓
Group
 ↓
Policy
 ↓
Permission

I also learned that permissions should follow the principle of least privilege. A training identity should receive only the permissions necessary for the assigned AWS exercises.

I also learned that AWS credentials and authentication information must never be exposed in screenshots, documentation, Git repositories, or shared messages.

19. Conclusion

A dedicated IAM training identity provides a safer way to perform AWS training activities without using the root account for normal administration.

The configuration created in this task follows:

Dedicated User
      ↓
Training Group
      ↓
Limited Policy
      ↓
Required Permissions

This structure makes permissions easier to manage, improves security, and supports the principle of least privilege.

Deliverable: IAM_User_Setup.md

Status: Completed