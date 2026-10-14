Deploy an Application Through GitHub Actions
Let's build on your Terraform and AWS DevOps training by creating a CI/CD pipeline using GitHub Actions.
Goal: Whenever you push application code to GitHub, GitHub Actions will test the code, build a Docker image, and deploy the application to AWS.
1. Understand the deployment architecture
GitHub repository
Developer pushes code to main

GitHub Actions
Checkout → Test → Build → Security checks

AWS authentication
OIDC with least-privilege IAM role

Amazon ECR
Store versioned Docker image

Amazon ECS
Deploy image → health check → verify


We'll use GitHub Actions + Docker + Amazon ECR + Amazon ECS Fargate. This follows the AWS deployment architecture you've been practicing.
2. Prepare your GitHub repository
Use your existing application repository or create a new one with this structure:
devops-github-actions/

app.py

Dockerfile

requirements.txt

task-definition.json

.github/workflows/deploy.yml



For this example, the application runs on port 8000 and provides a /health endpoint, matching your earlier ECS training setup.
Create a sample application
requirements.txt
flask
gunicorn


app.py
from flask import Flask, jsonifyapp = Flask(__name__)@app.get("/")def home():    return jsonify(message="Deployed through GitHub Actions!")@app.get("/health")def health():    return jsonify(status="healthy")if __name__ == "__main__":    app.run(host="0.0.0.0", port=8000)



Dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]


3. Prepare AWS resources
Before the workflow can deploy, configure the following in AWS:
- ECR repository: devops-training-app
- ECS cluster: devops-training-cluster
- ECS service: devops-training-service
- Task definition container name: devops-app
- AWS region: ap-south-1
These names follow your existing training setup. If your actual resources use different names, change the workflow accordingly.
You also need:
1. An ECS task execution role that can pull images from ECR and write application logs.
2. An ECS task role with only the permissions the application needs.
3. A GitHub Actions IAM role configured for GitHub OIDC, restricted to your repository and deployment branch.
4. Network configuration that lets ECS tasks start and pass their load balancer health checks, if an ALB is used.
GitHub Actions should assume the IAM role through OIDC rather than storing permanent AWS access keys in GitHub.
4. Create the ECS task definition
Create task-definition.json. Replace the account ID and role ARNs with your actual AWS values.

{
  "family": "devops-training-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::YOUR_ACCOUNT_ID:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::YOUR_ACCOUNT_ID:role/devops-training-task-role",
  "containerDefinitions": [
    {
      "name": "devops-app",
      "image": "REPLACE_WITH_ECR_IMAGE",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8000,
          "protocol": "tcp"
        }
      ],
      "healthCheck": {
        "command": [
          "CMD-SHELL",
          "python -c \"import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/health', timeout=3)\" || exit 1"
        ],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 15
      },
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/devops-training-app",
          "awslogs-region": "ap-south-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}



Ensure the CloudWatch log group exists and the task execution role can write to it. The health check requires Python in the container, which this example's image provides.
If your ECS service already uses a task definition with secrets, environment variables, sidecars, or other configuration, preserve those settings rather than replacing the production task definition with this minimal example.
5. Configure GitHub repository settings
Open your repository on GitHub.
Navigate to Settings → Secrets and variables → Actions.
Add this repository secret:
Secret	Value
AWS_ROLE_ARN	ARN of the IAM role GitHub Actions can assume
Add these repository variables:
Variable	Example value
AWS_ACCOUNT_ID	Your 12-digit AWS account ID
The workflow below assumes that the AWS role is already configured with a GitHub OIDC trust policy restricted to your repository and main branch. Don't add AWS access keys as secrets.
6. Create the GitHub Actions workflow
Create .github/workflows/deploy.yml.
This workflow will:
1. Check out your code.
2. Run a basic application health-endpoint test.
3. Authenticate to AWS using OIDC.
4. Build and push a versioned Docker image to ECR.
5. Render a new ECS task definition.
6. Deploy it to ECS and wait for service stability.

name: Deploy Application to ECS

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

concurrency:
  group: ecs-production-deployment
  cancel-in-progress: false

env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: devops-training-app
  ECS_CLUSTER: devops-training-cluster
  ECS_SERVICE: devops-training-service
  CONTAINER_NAME: devops-app

jobs:
  deploy:
    name: Test, Build and Deploy
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Test health endpoint
        run: |
          python -c "from app import app; c = app.test_client(); r = c.get('/health'); assert r.status_code == 200 and r.json['status'] == 'healthy', r.json"

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Verify AWS identity
        run: aws sts get-caller-identity

      - name: Login to Amazon ECR
        id: ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push image
        id: build
        env:
          REGISTRY: ${{ steps.ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t "$REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" .
          docker push "$REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"
          echo "image=$REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> "$GITHUB_OUTPUT"

      - name: Render ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ steps.build.outputs.image }}

      - name: Deploy to Amazon ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v2
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true



Why use ${{ github.sha }}? Every commit receives its own image tag. That makes it easier to identify which code version was deployed and to redeploy an earlier image if needed.
Note that the sample workflow's application test runs before the Docker build. For production, add dependency and image vulnerability scanning, workflow action version pinning, and protected deployment approvals. The deployment job also needs narrowly scoped IAM permissions for ECR uploads, ECS task-definition registration, and updating the intended ECS service.
7. Configure GitHub OIDC permissions
The GitHub Actions IAM role needs permission to push to your specific ECR repository and update your intended ECS service and task definition. Avoid AdministratorAccess.
Its trust policy should restrict the GitHub identity to your repository and deployment branch. For example, replace YOUR_ORG/YOUR_REPO with the exact repository path:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/YOUR_REPO:ref:refs/heads/main"
        }
      }
    }
  ]
}



This trust policy is only the authentication portion. You must also configure the IAM role's permission policies, create or verify the GitHub OIDC provider in AWS IAM, and ensure the ECS service's deployment configuration is correct.
8. Push your code to GitHub
Run the following commands from your project directory:
git init
git add app.py Dockerfile requirements.txt task-definition.json .github/workflows/deploy.yml
git commit -m "Add GitHub Actions ECS deployment"
git branch -M main
git remote add origin https://github.com/YOUR_ORG/YOUR_REPO.git
git push -u origin main


Replace the remote URL with your real repository URL. If the repository already has a remote, check it with git remote -v instead of adding another one.
9. Verify the deployment
Open your repository on GitHub and select Actions.
1. GitHub Actions
Confirm each workflow step succeeds.



2. Amazon ECR
Verify that the image tagged with your commit SHA exists.



3. Amazon ECS
Verify that the service is stable and its tasks are running.



4. Application health
Open your application's configured URL and verify its health endpoint.



You can also check ECS from your terminal:
aws ecs describe-services \
  --cluster devops-training-cluster \
  --services devops-training-service \
  --region ap-south-1


If you have an Application Load Balancer, test the configured application URL:
curl https://YOUR_APPLICATION_DOMAIN/health


Expected response:
{
  "status": "healthy"
}


Use your actual ALB DNS name or application domain. This test only works after DNS, networking, security groups, and HTTPS configuration are in place.
10. Deployment failure and rollback
If deployment fails:
1. Open the failed run in GitHub Actions and inspect the first failing step.
2. Check the ECS service's events for task startup or health-check failures.
3. Inspect /ecs/devops-training-app in CloudWatch Logs.
4. Verify the IAM permissions, ECR image, container port, and health-check settings.
5. If a new deployment is unhealthy, restore a previously verified task-definition revision or redeploy the previous known-good image through your controlled deployment process.
Do not delete the ECS service or cluster as a troubleshooting shortcut.
Your practical assignment
CI/CD completion tracker
0/7



[ ] Configure the GitHub OIDC provider and least-privilege IAM role
[ ] Create the ECR repository and verify ECS resources
[ ] Commit the application and workflow to GitHub
[ ] Run the pipeline and verify tests pass
[ ] Verify the commit-tagged image in ECR
[ ] Verify the ECS deployment and application health
[ ] Document a rollback procedure and deployment evidence