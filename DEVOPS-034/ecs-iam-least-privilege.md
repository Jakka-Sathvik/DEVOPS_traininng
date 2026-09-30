ECS IAM Least Privilege
Objective

Understand how IAM least privilege applies to ECS workloads and configure restricted access to AWS Secrets Manager.

The main principle is:

Application
     │
     ▼
Required Secret Only
     │
     ▼
Least Privilege
1. What Is Least Privilege?

Least privilege means giving a user, application, or AWS service only the permissions required to perform its job.

For an ECS application that needs one database secret:

ECS Task
   │
   ▼
Database Secret

It should not automatically receive access to:

Other Application Secrets
Production Secrets
Unrelated Secrets
All AWS Secrets
2. Which IAM Role Needs Access?

For ECS secret injection, the important distinction is between the:

Task Execution Role

and:

Task Role
Task Execution Role

The task execution role is used by the ECS/Fargate agent for operations required to start the task, including retrieving secrets referenced through the ECS task definition.

For the Secrets Manager integration in this exercise, the execution role needs permission such as:

secretsmanager:GetSecretValue

on the specific secret.

Architecture:

ECS/Fargate
     │
     ▼
Task Execution Role
     │
     ▼
Secrets Manager
     │
     ▼
Training Secret
Task Role

The task role is different.

It provides AWS permissions to the application running inside the container.

Example:

Application Container
        │
        ▼
Task Role
        │
        ├── S3 access
        ├── DynamoDB access
        └── Other required AWS APIs

If the application itself calls Secrets Manager directly at runtime using the AWS SDK, then the task role would need secretsmanager:GetSecretValue.

For this exercise, where ECS injects the secret through the task definition, configure the required Secrets Manager permission on the task execution role.

3. Does Every ECS Task Need Every Secret?

No.

Each workload should receive only the secrets it requires.

Incorrect:

ECS Task
   │
   ▼
All Secrets

Correct:

ECS Task
   │
   ▼
Required Secret Only

Example:

devops-training-app
        │
        ▼
devops-training/database

Another application:

payment-service
        │
        ▼
payment/database

The training application should not automatically access the payment secret.

4. Broad Permission vs Restricted Permission
Broad Permission

Avoid this when a specific secret ARN can be used:

{
  "Effect": "Allow",
  "Action": "secretsmanager:GetSecretValue",
  "Resource": "*"
}

This means the role can request secret values from any Secrets Manager secret allowed by that policy.

The permission is broader than necessary for this training application.

5. Restricted Permission

Instead, specify the particular secret.

Example:

{
  "Effect": "Allow",
  "Action": "secretsmanager:GetSecretValue",
  "Resource": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:devops-training/database-AbCdEf"
}

Now the permission is scoped to the training secret.

Architecture:

ECS Task
   │
   ▼
Task Execution Role
   │
   ▼
GetSecretValue
   │
   ▼
devops-training/database
6. Recommended Training Policy

Use a policy similar to:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadTrainingDatabaseSecret",
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "arn:aws:secretsmanager:ap-south-1:123456789012:secret:devops-training/database-AbCdEf"
    }
  ]
}

Replace:

123456789012

with your AWS account ID and use the actual ARN of your secret.

7. Why Specify the Secret ARN?

Consider two secrets:

devops-training/database
devops-training/api-key

The application only needs:

devops-training/database

The policy should therefore allow:

GetSecretValue
        ↓
database secret only

and not:

GetSecretValue
        ↓
all secrets

This limits the potential impact of an incorrectly configured or compromised workload.

8. IAM Policy Components

Example:

{
  "Effect": "Allow",
  "Action": "secretsmanager:GetSecretValue",
  "Resource": "SECRET_ARN"
}

Understand each part.

Effect
Allow

The action is permitted.

Action
secretsmanager:GetSecretValue

The role can retrieve a secret value.

Resource
SECRET_ARN

The permission applies to a specific secret.

9. AWS Console Configuration

Navigate to:

AWS Console
   ↓
IAM
   ↓
Roles

Find the ECS task execution role used by your service.

Example:

ecsTaskExecutionRole

Open:

Permissions
   ↓
Add permissions
   ↓
Create inline policy

Select:

Secrets Manager

Allow:

GetSecretValue

For the resource, select or specify:

devops-training/database

Avoid selecting all secrets when the application only needs one.

10. Verify the Role

Your architecture should now look like:

ECS Service
     │
     ▼
Fargate Task
     │
     ▼
Task Execution Role
     │
     ▼
secretsmanager:GetSecretValue
     │
     ▼
Specific Secret ARN
     │
     ▼
devops-training/database
11. Test the Permission

First verify that the ECS application can start successfully.

ECS
 ↓
Cluster
 ↓
Service
 ↓
Tasks

Expected:

Task Status: RUNNING

The secret should be successfully injected.

12. Understand Access Denied

If the role does not have the required permission, the task may fail during startup.

A possible error is:

AccessDeniedException

Example cause:

ECS Execution Role
       │
       X
       │
Secrets Manager

The role does not have permission to retrieve the secret.

13. Troubleshooting Process

When secret retrieval fails:

Step 1 — Check Secret ARN

Verify that the task definition references the correct secret.

Task Definition
      ↓
Secret ARN
Step 2 — Check Execution Role

Verify the ECS task execution role.

ECS Task
   ↓
Execution Role
Step 3 — Check IAM Action

Verify:

secretsmanager:GetSecretValue
Step 4 — Check Resource

Verify that the policy references the correct secret ARN.

Step 5 — Check Region

Make sure the secret is in the expected AWS Region.

Example:

ap-south-1
14. Least Privilege Example

Suppose the environment contains:

Secret A
devops-training/database

Secret B
devops-training/api-key

Secret C
production/database

The training application needs only:

Secret A

Therefore:

Allowed:
Secret A

Not required:
Secret B
Secret C

This is least privilege.

15. Important Security Principle

Do not think:

"The ECS application might need more permissions later, so give it everything now."

Instead:

Identify Requirement
       ↓
Grant Required Permission
       ↓
Restrict Resource
       ↓
Test
       ↓
Add Permission Only When Necessary

This makes IAM policies easier to understand and reduces unnecessary access.

16. Task Role vs Task Execution Role
Feature	Task Execution Role	Task Role
Used by	ECS/Fargate agent	Application container
Pull image from ECR	Yes	No
Send logs to CloudWatch	Yes	No
ECS-injected Secrets Manager values	Yes	No
Application calls AWS APIs	No	Yes
Example	Retrieve task startup resources	Application reads S3

The exact permissions should always match the way the application is implemented.

17. Production Architecture
                    AWS
                     │
                     ▼
               ECS Service
                     │
                     ▼
                Fargate Task
                     │
             ┌───────┴────────┐
             │                │
             ▼                ▼
    Execution Role         Task Role
             │                │
             ▼                ▼
      Secrets Manager    Other AWS APIs
             │
             ▼
   Specific Training Secret
18. Success Criteria

The task is complete when:

☑ ECS execution role identified

☑ Required Secrets Manager permission identified

☑ GetSecretValue permission configured

☑ Permission restricted to the training secret ARN

☑ Resource "*" avoided when unnecessary

☑ ECS task starts successfully

☑ Secret is successfully injected

☑ Application runs normally
19. Key Takeaways

Remember these four principles:

1. Give only required permissions.

2. Restrict permissions to specific resources.

3. Do not give every ECS task access to every secret.

4. Use "*" only when broader access is genuinely required.

The target architecture is:

Application
     │
     ▼
Required Secret Only
     │
     ▼
IAM Least Privilege
     │
     ▼
Secure ECS Deployment
Conclusion

IAM least privilege ensures that an ECS workload can access only the AWS resources it actually requires. For ECS secret injection, the task execution role needs permission to retrieve the referenced Secrets Manager secret. Instead of granting secretsmanager:GetSecretValue against Resource: "*", restrict the permission to the specific training secret ARN whenever possible. This reduces unnecessary access and establishes a s