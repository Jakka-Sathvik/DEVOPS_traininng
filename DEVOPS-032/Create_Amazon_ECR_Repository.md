TASK 7 — Create Amazon ECR Repository

Time: 35 Minutes

1. Objective

The goal of this task is to create an Amazon Elastic Container Registry (ECR) repository where Docker images produced by our CI/CD pipeline can be stored.

Example repository:

devops-training-app

The overall flow will eventually be:

Developer
    │
    ▼
GitHub
    │
    ▼
GitHub Actions
    │
    ├── Docker Build
    ├── Security Scan
    │
    ▼
Amazon ECR
    │
    ├── Image v1
    ├── Image v2
    └── Image SHA
2. What is Amazon ECR?

Amazon Elastic Container Registry (ECR) is an AWS service used to store and manage Docker/container images.

Think of ECR as a private Docker image repository inside AWS.

For example:

Docker Hub
    │
    └── Public/Private Docker Images


AWS ECR
    │
    └── AWS Container Images

If our GitHub Actions pipeline builds:

devops-training-app:1.0.0

we can push that image to ECR.

ECR can then store:

devops-training-app
│
├── 1.0.0
├── 1.0.1
├── latest
└── a81c92d
3. Why Do We Need ECR?

In a CI/CD pipeline, building an image is only one part of the process.

For example:

Source Code
     │
     ▼
Docker Build
     │
     ▼
Docker Image
     │
     ▼
Security Scan
     │
     ▼
ECR
     │
     ▼
Deployment

The ECR repository provides a central location for storing images that can later be deployed to services such as:

ECS
EKS
EC2
AWS App Runner
4. ECR Architecture

Our repository will look conceptually like this:

AWS Account
     │
     ▼
Amazon ECR
     │
     ▼
Repository
devops-training-app
     │
     ├── v1.0.0
     │
     ├── v1.0.1
     │
     ├── latest
     │
     └── Git Commit SHA

Important:

Repository

The repository is the container that holds Docker images.

Example:

devops-training-app
Image

The actual Docker image stored inside the repository.

Example:

devops-training-app:v1.0.0
Tag

A tag identifies a particular image version.

Examples:

v1.0.0
v1.0.1
latest
a81c92d
5. Prerequisites

Before creating the repository, verify that the AWS CLI is installed.

Run:

aws --version

Example:

aws-cli/2.x.x ...

If the command is not found, install/configure the AWS CLI before continuing.

6. Verify AWS Authentication

Before creating an ECR repository, confirm that the AWS CLI is connected to the correct AWS account.

Run:

aws sts get-caller-identity

You should receive information similar to:

{
    "UserId": "AIDXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/devops-user"
}
What does this command do?
aws sts get-caller-identity

It asks AWS:

"Who am I authenticated as?"

This is an important troubleshooting step.

If this command fails, do not continue with ECR creation until AWS authentication is fixed.

7. Check the AWS Region

Our training environment uses:

ap-south-1

which is the Mumbai AWS region.

You can explicitly specify the region in commands:

--region ap-south-1

Using an explicit region helps avoid accidentally creating resources in the wrong AWS region.

8. Create the ECR Repository

Create the repository using:

aws ecr create-repository \
  --repository-name devops-training-app \
  --region ap-south-1

The important part is:

repository-name
        │
        ▼
devops-training-app

AWS will create the repository.

9. Understand the Command

The command:

aws ecr create-repository \
  --repository-name devops-training-app \
  --region ap-south-1

has several parts.

aws

AWS CLI command.

ecr

Specifies that we are working with Amazon Elastic Container Registry.

create-repository

Tells AWS to create a new ECR repository.

--repository-name

Specifies the repository name.

devops-training-app
--region

Specifies where the repository should be created.

ap-south-1
10. Expected Output

AWS should return information similar to:

{
    "repository": {
        "repositoryArn": "arn:aws:ecr:ap-south-1:123456789012:repository/devops-training-app",
        "registryId": "123456789012",
        "repositoryName": "devops-training-app",
        "repositoryUri": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app"
    }
}

The most important value is:

repositoryUri

For example:

123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app

This URI will later be used when tagging and pushing Docker images.

11. Verify the Repository

The task specifically requires:

aws ecr describe-repositories

Run:

aws ecr describe-repositories \
  --region ap-south-1

You should see:

devops-training-app

in the output.

12. Verify Only Our Repository

You can also check the specific repository:

aws ecr describe-repositories \
  --repository-names devops-training-app \
  --region ap-south-1

This is useful because it directly verifies that the expected repository exists.

13. List ECR Repositories

Another useful command is:

aws ecr describe-repositories \
  --query 'repositories[*].repositoryName' \
  --output table \
  --region ap-south-1

Example:

-------------------------
|   RepositoryName      |
-------------------------
| devops-training-app   |
-------------------------

This gives a cleaner view.

14. Check the Repository from AWS Console

You can also verify the repository through the AWS Console.

Process:

AWS Console
    │
    ▼
Search
    │
    ▼
ECR
    │
    ▼
Elastic Container Registry
    │
    ▼
Repositories
    │
    ▼
devops-training-app

Open:

Amazon ECR
→ Repositories
→ devops-training-app

At this stage, the repository may contain:

Images: 0

That is completely normal.

We have created the repository, but we have not pushed an image yet.

15. Repository vs Image

This distinction is important.

Repository
devops-training-app

is the storage location.

Image
devops-training-app:v1.0.0

is an actual Docker image stored in that repository.

Initially:

ECR
 │
 └── devops-training-app
       │
       └── No images yet

After pushing an image:

ECR
 │
 └── devops-training-app
       │
       ├── v1.0.0
       ├── latest
       └── a81c92d
16. ECR URI

The repository URI usually follows this structure:

AWS_ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/REPOSITORY_NAME

For example:

123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app

Breaking it down:

123456789012
      │
      └── AWS Account ID

dkr.ecr
      │
      └── Amazon ECR registry

ap-south-1
      │
      └── AWS Region

devops-training-app
      │
      └── Repository
17. How Docker Images Will Eventually Be Stored

Suppose our local image is:

devops-app:a81c92d

We can eventually tag it with the ECR repository URI:

123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app:a81c92d

Then:

Docker Image
     │
     ▼
ECR Repository

The complete future flow will be:

Docker Build
     │
     ▼
devops-app:a81c92d
     │
     ▼
Docker Tag
     │
     ▼
ECR Image
123456789012.dkr.ecr.ap-south-1.amazonaws.com/
devops-training-app:a81c92d
     │
     ▼
Docker Push
     │
     ▼
Amazon ECR
18. Do Not Push the Image Yet

This task is only:

Create ECR Repository

The image push will be handled in the next stage.

For now, verify:

aws ecr describe-repositories \
  --repository-names devops-training-app \
  --region ap-south-1

If the repository information is returned successfully, Task 7 is complete.

19. Important Security Consideration

Do not put AWS access keys directly into source code.

Avoid:

AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...

inside:

Dockerfile
Git repository
application source code
GitHub workflow files

For the eventual GitHub Actions → AWS connection, a preferred approach is GitHub Actions OIDC with an IAM role, rather than storing long-lived AWS access keys.

The architecture will eventually become:

GitHub Actions
      │
      │ OIDC
      ▼
AWS IAM Role
      │
      ▼
Amazon ECR

This follows the same least-privilege/security approach used in the earlier AWS training tasks.

20. Troubleshooting
Problem 1 — AWS command not found
aws: command not found

Check:

aws --version

Install/configure the AWS CLI before continuing.

Problem 2 — Authentication error

Example:

Unable to locate credentials

Check:

aws sts get-caller-identity

Your AWS credentials/configuration need to be fixed first.

Problem 3 — AccessDeniedException

Example:

AccessDeniedException

The IAM identity being used may not have permission to create ECR repositories.

The required permission includes:

ecr:CreateRepository

Your IAM permissions should follow least privilege rather than using:

AdministratorAccess

for normal training/application workflows.

Problem 4 — Wrong Region

If you cannot find the repository, check the region.

For this training:

ap-south-1

Use:

aws ecr describe-repositories \
  --region ap-south-1

Remember that ECR repositories are regional resources.

21. Complete Practical Process

Follow this sequence during the practical session:

Step 1 — Check AWS CLI
aws --version
Step 2 — Check AWS identity
aws sts get-caller-identity
Step 3 — Create repository
aws ecr create-repository \
  --repository-name devops-training-app \
  --region ap-south-1
Step 4 — Verify repository
aws ecr describe-repositories \
  --repository-names devops-training-app \
  --region ap-south-1
Step 5 — List repositories
aws ecr describe-repositories \
  --query 'repositories[*].repositoryName' \
  --output table \
  --region ap-south-1
Step 6 — Open AWS Console
AWS Console
 → ECR
 → Repositories
 → devops-training-app
22. Final Architecture
                 AWS
                  │
                  ▼
          Amazon ECR
                  │
                  ▼
       devops-training-app
                  │
          ┌───────┼────────┐
          │       │        │
          ▼       ▼        ▼
        v1.0.0  v1.0.1  Git SHA

The future CI/CD pipeline will be:

Developer
    │
    ▼
GitHub
    │
    ▼
GitHub Actions
    │
    ├── Checkout
    │
    ├── Docker Build
    │
    ├── Image Tag
    │
    ├── Trivy Security Scan
    │
    ▼
AWS Authentication
    │
    ▼
Amazon ECR
    │
    ▼
Docker Push
23. Task 7 Deliverable

The required deliverable is:

Working ECR Repository

Repository name:

devops-training-app

Region:

ap-south-1

Verification command:

aws ecr describe-repositories \
  --repository-names devops-training-app \
  --region ap-south-1

Expected result:

Repository exists successfully