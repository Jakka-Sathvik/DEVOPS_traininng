Amazon ECR Application Image Verification
Objective

Reuse the Docker application image created previously and verify that it is stored in Amazon Elastic Container Registry (ECR).

The image will later be used by Amazon ECS and AWS Fargate deployments.

Architecture
Developer
    │
    ▼
Docker Build
    │
    ▼
Docker Image
    │
    ▼
Amazon ECR
    │
    ▼
Amazon ECS
    │
    ▼
AWS Fargate
What is Amazon ECR?

Amazon Elastic Container Registry (ECR) is a fully managed AWS service used to store Docker container images.

ECR acts as a private Docker registry inside AWS.

Benefits:

Secure image storage
Version management
Integration with ECS
Integration with Fargate
High availability
IAM-based access control
Verify AWS Authentication

Before checking repositories, verify that AWS CLI is authenticated.

aws sts get-caller-identity

Expected Output:

{
  "UserId": "AIDXXXXXXXXXXXX",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/devops-user"
}

This confirms that AWS CLI can communicate with AWS services.

List Available ECR Repositories

Check whether the application repository exists.

aws ecr describe-repositories

Example Output:

devops-training-app

Expected Architecture:

Amazon ECR
     │
     └── devops-training-app
Verify Available Images

List all image tags stored in the repository.

aws ecr list-images \
  --repository-name devops-training-app

Example Output:

{
  "imageIds": [
    {
      "imageTag": "commit-a12345"
    },
    {
      "imageTag": "commit-b23456"
    },
    {
      "imageTag": "v1.0.0"
    }
  ]
}

Visual Representation:

devops-training-app
│
├── commit-a12345
├── commit-b23456
└── v1.0.0
Understand Image Tags

Container images should be tagged properly.

Example:

commit-a12345

Represents:

Git Commit
     │
     ▼
Container Image

Example:

v1.0.0

Represents:

Application Release Version

Benefits:

Easy rollback
Version tracking
Deployment consistency
CI/CD integration
Verify Image Details

To inspect image information:

aws ecr describe-images \
  --repository-name devops-training-app

This displays:

Image tag
Image size
Push date
Digest
Metadata

Example:

Image Tag: v1.0.0
Size: 180 MB
Pushed: 2026-09-23
ECR Workflow
Developer
     │
     ▼
Docker Build
     │
     ▼
Tag Image
     │
     ▼
Push to ECR
     │
     ▼
Store Image
     │
     ▼
ECS Pulls Image
     │
     ▼
Fargate Runs Container
Why ECS Uses ECR

ECS does not store container images.

Instead:

ECS
 │
 ▼
Pull Image
 │
 ▼
Amazon ECR

During deployment:

Task Definition
       │
       ▼
Image URI
       │
       ▼
ECR Repository
       │
       ▼
Container Starts

Example Image URI:

123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app:v1.0.0
Validation Checklist
Repository Verification
aws ecr describe-repositories

✓ Repository exists

Image Verification
aws ecr list-images \
  --repository-name devops-training-app

✓ Images visible

Image Metadata Verification
aws ecr describe-images \
  --repository-name devops-training-app

✓ Metadata visible

Expected Result
Repository:
devops-training-app

Images:
├── commit-a12345
├── commit-b23456
└── v1.0.0

The application image is successfully stored in Amazon ECR and ready for deployment through Amazon ECS and AWS Fargate.

Key Takeaways
Amazon ECR stores Docker images securely.
ECS retrieves images from ECR during deployments.
Image tags help track releases and commits.
ECR supports version control and rollback strategies.
Proper image verification ensures deployment readiness.
ECR is the standard image registry for ECS and Fargate workloads.