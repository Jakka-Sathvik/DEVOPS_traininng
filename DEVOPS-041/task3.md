1. Target architecture
GitHub Repository
       │
       │ OIDC token
       ▼
GitHub Actions
       │
       ▼
AWS IAM OIDC Provider
       │
       ▼
┌─────────────────────────────┐
│ GitHub Actions IAM Roles    │
├─────────────────────────────┤
│ Terraform Role              │
│ Deployment Role             │
└──────────────┬──────────────┘
               │
       Least-privilege access
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
     ECR      ECS      Terraform
                      resources

Important
Do not use:
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY

as permanent GitHub secrets.
Instead:
GitHub OIDC → AssumeRole → Temporary AWS credentials

2. Create GitHub OIDC provider
In AWS:
IAM → Identity providers → Add provider
Use:
Provider type:
OpenID Connect

Provider URL:
https://token.actions.githubusercontent.com

Audience:
sts.amazonaws.com

The GitHub Actions workflow then requests an OIDC token and AWS STS exchanges it for temporary credentials.
3. Create separate IAM roles
Don't give one role every permission.
For your project, use:
github-terraform-role
github-ecs-deploy-role

Optionally later:
github-readonly-plan-role
github-production-deploy-role

Architecture:
                 GitHub
                   │
             OIDC authentication
                   │
          ┌────────┴────────┐
          ▼                 ▼
 Terraform Role        ECS Deploy Role
          │                 │
          ▼                 ▼
 Terraform AWS        ECR + ECS
 resources            deployment

This limits the blast radius if one workflow is compromised.
4. Restrict the trust policy
This is one of the most important parts.
Don't create a trust policy that effectively says:
Any GitHub repository can assume this role

Instead restrict it to your repository.
Example:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/YOUR_REPO:ref:refs/heads/main"
        }
      }
    }
  ]
}

Replace:
YOUR_ORG/YOUR_REPO

with your actual GitHub repository.
For example:
repo:mycompany/devops-training:ref:refs/heads/main

Now only the specified repository's main branch can assume the role.
5. Terraform role
Terraform needs permissions for the infrastructure it manages.
For your architecture this may include:
VPC
ALB
ECS
IAM
Security Groups
CloudWatch
RDS
Secrets Manager
ECR

But don't simply attach:
AdministratorAccess

Your previous IAM requirement of no AdministratorAccess is exactly right.
Instead, create permissions based on the actual resources Terraform manages.
For example:
github-terraform-role
       │
       ├── EC2/VPC permissions
       ├── ECS permissions
       ├── ELB permissions
       ├── IAM limited permissions
       ├── CloudWatch permissions
       ├── RDS permissions
       └── Secrets Manager limited permissions

6. Be especially careful with IAM permissions
IAM is more sensitive than normal AWS resources.
Avoid:
{
  "Effect": "Allow",
  "Action": "iam:*",
  "Resource": "*"
}

Instead allow only the IAM resources Terraform actually creates or manages.
For example, if Terraform creates an ECS task execution role, restrict permissions around that specific role/resource.
Also restrict:
iam:PassRole

This is very important.
Don't give:
iam:PassRole → *

Instead:
iam:PassRole
Resource:
arn:aws:iam::<ACCOUNT_ID>:role/devops-training-ecs-task-role

Conceptually:
GitHub Actions
      │
      │ iam:PassRole
      ▼
Specific ECS task role

not:
GitHub Actions
      │
      │ iam:PassRole *
      ▼
Any IAM role

7. ECS deployment role
The application deployment workflow doesn't need permission to modify your entire AWS account.
It primarily needs:
ECR
ECS
iam:PassRole

For example:
ECR:
  GetAuthorizationToken
  BatchCheckLayerAvailability
  InitiateLayerUpload
  UploadLayerPart
  CompleteLayerUpload
  PutImage

ECS:
  DescribeServices
  DescribeTaskDefinition
  RegisterTaskDefinition
  UpdateService
  DescribeTasks

IAM:
  PassRole

And restrict the resources wherever AWS supports resource-level permissions.
8. GitHub Actions permissions
Your workflow should explicitly request only what it needs.
For OIDC:
permissions:
  id-token: write
  contents: read

This means:
id-token: write
    ↓
GitHub can request OIDC token

contents: read
    ↓
GitHub can checkout repository

Don't use:
permissions: write-all

unless there is a specific reason.
9. Configure AWS credentials
Your workflow can use:
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
    aws-region: ap-south-1

The workflow receives temporary credentials.
There is no:
AWS secret key stored in GitHub

10. Better: use GitHub Environment
For production, create:
GitHub
 └── Settings
      └── Environments
           ├── dev
           ├── staging
           └── production

For production:
Required reviewer
Deployment branch restriction
Environment-specific variables

Then:
Developer
   ↓
Pull Request
   ↓
Lint
   ↓
Terraform Validate
   ↓
Security Scan
   ↓
Terraform Plan
   ↓
Merge
   ↓
Production Approval
   ↓
OIDC
   ↓
AWS IAM Role
   ↓
Terraform/ECS

11. Don't use the same role everywhere
A good production design is:
                    GitHub
                       │
                     OIDC
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
     Terraform Role        ECS Deploy Role
             │                   │
             │                   ├── ECR
             │                   ├── ECS
             │                   └── PassRole
             │
             ├── VPC
             ├── ALB
             ├── ECS
             ├── RDS
             ├── CloudWatch
             ├── Secrets Manager
             └── Limited IAM

This follows least privilege and separation of duties.
12. Security gates + OIDC together
Your complete pipeline is now:
                 GitHub
                    │
                    ▼
               Pull Request
                    │
                    ▼
                 Linting
                    │
                    ▼
              Unit Testing
                    │
                    ▼
          Terraform Validate
                    │
                    ▼
            TFLint + Checkov
                    │
                    ▼
             Terraform Plan
                    │
                    ▼
             Security Review
                    │
                    ▼
             Approval Gate
                    │
                    ▼
             GitHub OIDC
                    │
                    ▼
             AWS IAM Role
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
   Terraform Role       ECS Deploy Role
          │                   │
          ▼                   ▼
    AWS Infrastructure       ECR
                              │
                              ▼
                             ECS
                              │
                              ▼
                       Health Check
