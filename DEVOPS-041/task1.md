GitHub → GitHub Actions → Terraform → AWS infrastructure → ECS → Docker image → ECR → ECS deployment











1. Understand the flow
Developer
   │
   │ git push
   ▼
GitHub Repository
   │
   ▼
GitHub Actions
   │
   ├── Terraform fmt
   ├── Terraform validate
   ├── Terraform plan
   └── Terraform apply
           │
           ▼
       AWS Infrastructure
           │
           ├── VPC
           ├── ALB
           ├── ECS Cluster
           ├── ECS Service
           ├── IAM
           ├── Security Groups
           └── CloudWatch
           
Then application deployment:

GitHub Actions
      │
      ├── Docker build
      ├── Trivy scan
      ├── Push image
      ▼
     ECR
      │
      ▼
ECS Task Definition
      │
      ▼
ECS Service
      │
      ▼
Running Container

Important distinction
Terraform should primarily create/manage infrastructure:
VPC
ALB
ECS cluster
ECS service
IAM
Security groups
CloudWatch
ECR repository

GitHub Actions should handle application delivery:
Test
 ↓
Docker build
 ↓
Security scan
 ↓
Push image to ECR
 ↓
Update ECS task definition
 ↓
Deploy
 ↓
Health check

Don't make Terraform rebuild the Docker image on every application commit.
2. Recommended project structure
For your existing training project, use:
devops-project/
│
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── providers.tf
│   ├── backend.tf
│   │
│   ├── modules/
│   │   ├── vpc/
│   │   ├── ecs/
│   │   ├── alb/
│   │   ├── iam/
│   │   └── security/
│   │
│   └── environments/
│       ├── dev/
│       │   ├── main.tf
│       │   ├── terraform.tfvars
│       │   └── backend.tf
│       │
│       ├── staging/
│       └── prod/
│
├── app/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── ...
│
├── .github/
│   └── workflows/
│       ├── terraform.yml
│       └── deploy-ecs.yml
│
└── docs/
    └── github-actions-terraform-ecs.md

This fits nicely with the Terraform module/environment structure you've been working on.
3. Connect GitHub Actions to AWS
Don't use this:
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY

as long-lived GitHub secrets.
Instead use GitHub OIDC → AWS IAM role.
The flow becomes:
GitHub Actions
      │
      │ OIDC token
      ▼
AWS IAM
      │
      │ AssumeRole
      ▼
Deployment Role
      │
      ├── Terraform
      └── ECR/ECS deployment

This avoids storing permanent AWS access keys in GitHub.
4. Create GitHub OIDC provider
In AWS IAM, configure the GitHub OIDC provider.
The provider is:
https://token.actions.githubusercontent.com

Then create an IAM role, for example:
github-actions-devops-role

Trust policy should restrict access to your repository rather than allowing every GitHub repository.
Conceptually:
GitHub repository
       ↓
GitHub OIDC
       ↓
github-actions-devops-role
       ↓
AWS resources

5. GitHub Actions Terraform workflow
Create:
.github/workflows/terraform.yml

Example:
name: Terraform

on:
  pull_request:
    paths:
      - "terraform/**"

  push:
    branches:
      - main
    paths:
      - "terraform/**"

permissions:
  id-token: write
  contents: read

jobs:
  terraform:
    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: terraform

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ap-south-1

      - name: Terraform Init
        run: terraform init

      - name: Terraform Format
        run: terraform fmt -check -recursive

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan

For your AWS region:
ap-south-1

6. Why each step exists
Checkout
uses: actions/checkout@v4

Downloads your GitHub repository into the GitHub Actions runner.
Terraform installation
uses: hashicorp/setup-terraform@v3

Installs Terraform.
AWS authentication
uses: aws-actions/configure-aws-credentials@v4

Allows Terraform to communicate with AWS.
The important part is:
role-to-assume: ${{ secrets.AWS_ROLE_ARN }}

Format
terraform fmt -check -recursive

Checks whether Terraform files are correctly formatted.
Validate
terraform validate

Checks Terraform configuration syntax and internal consistency.
Plan
terraform plan

Shows what AWS resources Terraform wants to create/change/destroy.
7. Add Terraform Apply
You can add a separate production-controlled workflow.
For example:
name: Terraform Apply

on:
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  terraform-apply:
    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: terraform

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ap-south-1

      - name: Terraform Init
        run: terraform init

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan

      - name: Terraform Apply
        run: terraform apply -auto-approve

workflow_dispatch means you manually start the deployment from GitHub Actions.
For production, an even better pattern is:
Pull Request
     ↓
fmt
     ↓
validate
     ↓
plan
     ↓
Review
     ↓
Merge
     ↓
Approval
     ↓
apply

8. ECS application deployment
Now create:
.github/workflows/deploy-ecs.yml

The flow should be:
git push main
     │
     ▼
GitHub Actions
     │
     ├── Checkout
     ├── Test
     ├── Docker build
     ├── Trivy scan
     ├── AWS authentication
     ├── ECR login
     ├── Docker push
     ├── Update ECS task definition
     ├── Deploy ECS
     └── Wait for service stability

9. Build Docker image
Example:
- name: Build Docker image
  run: |
    docker build \
      -t devops-training-app:${{ github.sha }} \
      .

Using:
${{ github.sha }}

is better than only using:
latest

because every commit gets a unique image.
For example:
devops-training-app:8f3a21c

10. Login to ECR
Example:
- name: Login to Amazon ECR
  id: login-ecr
  uses: aws-actions/amazon-ecr-login@v2

Then push:
- name: Push image
  env:
    ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
    ECR_REPOSITORY: devops-training-app
    IMAGE_TAG: ${{ github.sha }}

  run: |
    docker tag \
      devops-training-app:${IMAGE_TAG} \
      ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}

    docker push \
      ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}

11. Update ECS
Your ECS task definition might currently contain:
devops-app

with an image such as:
123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app:v1.0.0

After GitHub Actions builds commit abc123, you want:
123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app:abc123

Then ECS creates a new task revision.
Task Definition :1
       ↓
Task Definition :2
       ↓
Task Definition :3
       ↓
Task Definition :4

12. Deploy the new task definition
A common GitHub Actions approach is:
- name: Render ECS task definition
  id: task-def
  uses: aws-actions/amazon-ecs-render-task-definition@v1
  with:
    task-definition: ecs-task-definition.json
    container-name: devops-app
    image: ${{ steps.build-image.outputs.image }}

- name: Deploy to ECS
  uses: aws-actions/amazon-ecs-deploy-task-definition@v2
  with:
    task-definition: ${{ steps.task-def.outputs.task-definition }}
    service: devops-training-service
    cluster: devops-training-cluster
    wait-for-service-stability: true

The important values match your existing ECS design:
Cluster:
devops-training-cluster

Service:
devops-training-service

Container:
devops-app

13. Complete deployment architecture
Your final architecture should look like this:
                    GitHub
                       │
                       │ Push
                       ▼
              ┌─────────────────┐
              │ GitHub Actions   │
              └────────┬────────┘
                       │
            ┌──────────┴──────────┐
            │                     │
            ▼                     ▼
      Terraform Job         Application Job
            │                     │
            │                     ├── Tests
            │                     ├── Docker Build
            │                     ├── Trivy
            │                     └── Push
            │                          │
            ▼                          ▼
       AWS Infrastructure             ECR
            │                          │
            ├── VPC                    │
            ├── ALB                    │
            ├── ECS ◄─────────────────┘
            ├── IAM
            ├── SG
            └── CloudWatch
                       │
                       ▼
                 ECS Service
                       │
                  ┌────┴────┐
                  ▼         ▼
                Task 1     Task 2
                  │         │
                  └────┬────┘
                       ▼
                     ALB
                       │
                       ▼
                   Users

14. Your CI/CD pipeline
For your current training project, I recommend this final pipeline:
Developer
   │
   │ git push
   ▼
GitHub
   │
   ├──────────────────────────┐
   ▼                          ▼
Terraform workflow       Application workflow
   │                          │
   ├─ fmt                     ├─ Test
   ├─ validate                ├─ Docker build
   ├─ plan                    ├─ Trivy scan
   └─ apply*                  ├─ ECR push
                              ├─ ECS deploy
                              └─ Health check

* Apply should preferably require manual approval for staging/production.
What you should implement next
Since you've already worked through Terraform modules, ECS, ECR, ALB, CloudWatch, IAM and CI/CD, the next practical exercise is:
Day 39 — GitHub Actions + Terraform + ECS Integration
1. Create GitHub OIDC → AWS IAM role.
2. Connect Terraform to GitHub Actions.
3. Run fmt → validate → plan.
4. Apply Terraform.
5. Build your Docker application.
6. Scan with Trivy.
7. Push image to ECR.
8. Register new ECS task definition.
9. Deploy to devops-training-service.
10. Wait for ECS stability.
11. Test ALB /health.
12. Verify CloudWatch logs.
13. Document the complete pipeline and rollback procedure.