TASK 6 — Create an IAM Group

File: IAM_Group_Setup.md
Training Topic: AWS IAM Group and Least Privilege
Example Group: devops-trainees

1. Objective

The objective of this task is to create an IAM group for the AWS training environment and understand how permissions can be managed through groups.

The permission structure is:

IAM Group
     ↓
Permissions
     ↓
Users

Instead of assigning the same permissions individually to every user, permissions can be attached to a group and users can be added to that group.

2. IAM Group Architecture
                         AWS IAM
                            │
                            ▼
                    devops-trainees
                         GROUP
                            │
                            ▼
                    IAM Permissions
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
         Training User A             Training User B

For example:

devops-trainees
       │
       ├── sathvik-training
       └── another-trainee

Both users can receive the permissions assigned to the group.

3. What is an IAM Group?

An IAM group is a collection of IAM users.

Groups make permission management easier because permissions can be assigned to the group instead of repeatedly assigning the same permissions to individual users.

Example:

Group:
devops-trainees

Users:
    ├── trainee-1
    ├── trainee-2
    └── trainee-3

If a training policy is attached to devops-trainees, the users in that group can receive the permissions provided by that policy.

4. Why Use an IAM Group?

Using groups provides several benefits:

Centralized permission management
Consistent permissions
Easier user onboarding
Easier permission updates
Easier access reviews
Supports least privilege
Reduces repetitive configuration

Example:

Without a group:

User A → Policy
User B → Policy
User C → Policy

With a group:

                  Training Policy
                       │
                       ▼
               devops-trainees
                 /      |      \
                /       |       \
             User A   User B   User C

The second approach is easier to manage.

5. Least Privilege

The most important security principle for this task is least privilege.

Definition

Least privilege means giving a user or group only the permissions required to perform its intended tasks.

For example, if today's training exercise only requires access to S3, there is no reason to provide permissions to every AWS service.

Incorrect Approach
devops-trainees
       │
       ▼
AdministratorAccess
       │
       ▼
All AWS Services

This provides much more access than may be necessary.

Better Approach
devops-trainees
       │
       ▼
Training Policy
       │
       ▼
Required Service
       │
       ▼
Required Actions

This limits unnecessary access.

6. Create the IAM Group
Step 1 — Open IAM

Sign in to the AWS Management Console using your authorized administrative identity.

Navigate to:

AWS Console
      ↓
IAM
      ↓
User groups
Step 2 — Select Create Group

Click:

Create group

Enter the group name:

devops-trainees

Use the organization's naming convention if a different naming standard is required.

7. Add Users

If the training user created in the previous task is available, add it to the group.

Example:

Group:
devops-trainees

User:
sathvik-training

The relationship becomes:

sathvik-training
       │
       ▼
devops-trainees
       │
       ▼
Training Permissions
8. Attach Permissions

The group should receive only the permissions required for the day's exercise.

For example, if the training exercise requires limited S3 access:

devops-trainees
       │
       ▼
S3 Training Policy
       │
       ├── Required S3 actions
       └── Required training resources

Do not automatically attach:

AdministratorAccess

just because it makes the exercise easier.

The exact policy should be based on the actual AWS exercise and your organization's security policy.

9. Example of Least-Privilege Design

Suppose today's exercise requires users to inspect objects in a specific training S3 bucket.

A least-privilege design could be:

Group:
devops-trainees

        ↓

Policy:
Training-S3-ReadOnly

        ↓

Actions:
s3:ListBucket
s3:GetObject

        ↓

Resource:
Training S3 Bucket

The users do not need permission to:

Delete users
Create IAM policies
Modify billing
Change account settings
Modify unrelated AWS services

unless a specific training exercise requires those actions and they are authorized.

10. Add the Policy

During group creation:

Create group
      ↓
Select required policy
      ↓
Create group

For a training environment, use the narrowest suitable policy.

If the exercise requires a custom policy, the policy should explicitly define:

Effect
Action
Resource
Condition (when applicable)

Example structure:

Allow
  ↓
Specific Actions
  ↓
Specific Resources
11. Verify the Group

After creating the group, navigate to:

IAM
 ↓
User groups
 ↓
devops-trainees

Verify:

Group Name:
devops-trainees

Users:
sathvik-training

Permissions:
Training Policy

The exact user and policy names may differ according to the training environment.

12. Verify User Membership

Open:

IAM
 ↓
Users
 ↓
sathvik-training

Check the Groups section.

Expected:

Groups
 └── devops-trainees

This confirms that the training user belongs to the group.

13. Permission Flow

The final permission flow should be:

IAM User
    │
    ▼
devops-trainees
    │
    ▼
IAM Policy
    │
    ▼
AWS Permissions
    │
    ▼
Required Training Resources

For multiple users:

                   devops-trainees
                         │
                 Training Policy
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       User A          User B         User C
          │              │              │
          └──────────────┴──────────────┘
                         │
                         ▼
                Allowed AWS Actions
14. Why AdministratorAccess Should Not Be Used Automatically

It may be tempting to attach:

AdministratorAccess

because it avoids permission errors.

However, this is not a good security practice.

Administrator-level access can allow extensive changes across the AWS account.

Potential consequences include:

Accidental resource deletion
Unintended infrastructure changes
IAM permission changes
Security configuration changes
Increased impact if credentials are compromised

Therefore:

Training Requirement
        ↓
Required Permission
        ↓
Required AWS Service
        ↓
Required Resource

should be the preferred approach.

15. Example Comparison
Approach	Permission Level	Security
AdministratorAccess	Very broad	Higher risk
Service-level access	Limited to required service	Better
Action-level access	Limited actions	Better
Resource-level access	Limited resources	Best when practical
Least-privilege policy	Minimum required access	Recommended
16. Screenshot Evidence

Capture screenshots of the IAM group configuration.

Screenshot 1 — Group

Capture:

IAM
 ↓
User groups
 ↓
devops-trainees

Insert screenshot:

[INSERT SCREENSHOT — IAM GROUP]
Screenshot 2 — Group Users

Show the users belonging to:

devops-trainees

Insert screenshot:

[INSERT SCREENSHOT — GROUP USERS]
Screenshot 3 — Group Permissions

Show the policies attached to the group.

Insert screenshot:

[INSERT SCREENSHOT — GROUP PERMISSIONS]
Screenshot 4 — User Membership

Show the training user and its group membership.

Insert screenshot:

[INSERT SCREENSHOT — USER GROUP MEMBERSHIP]
Security Reminder

Before submitting screenshots, verify that they do not expose:

[ ] Passwords
[ ] Secret Access Keys
[ ] Access tokens
[ ] Session tokens
[ ] MFA recovery information
[ ] Other credentials or secrets
17. Verification Checklist
[✓] IAM group created
[✓] Group name is devops-trainees
[✓] Required training user added
[✓] Required policy attached
[✓] Permissions reviewed
[✓] Least privilege considered
[✓] AdministratorAccess not added unnecessarily
[✓] Screenshots captured
[✓] No credentials exposed
18. What I Learned

Through this task, I learned that an IAM group is used to organize users and manage their permissions centrally.

The basic relationship is:

IAM Group
     ↓
Permissions
     ↓
Users

I also learned that permissions should follow the principle of least privilege.

Instead of giving users administrator permissions, I should provide only the AWS services, actions, and resources required for the training exercise.

19. Final Architecture
                         AWS IAM
                            │
                            ▼
                    devops-trainees
                       IAM GROUP
                            │
                            ▼
                    Training Policy
                            │
                            ▼
                  Required Permissions
                            │
             ┌──────────────┴──────────────┐
             ▼                             ▼
       sathvik-training                User B
             │                             │
             └──────────────┬──────────────┘
                            ▼
                   Training AWS Resources
20. Conclusion

The devops-trainees IAM group provides a centralized way to manage permissions for users in the training environment.

The recommended approach is:

Create Group
     ↓
Add Training Users
     ↓
Attach Required Policy
     ↓
Review Permissions
     ↓
Apply Least Privilege
     ↓
Verify Access

The group should receive only the permissions required for the day's exercise.

Deliverable: IAM_Group_Setup.md

Status: Completed