Deploy an Application to Amazon ECS
Sathvik, let's continue your DevOps CI/CD project by deploying the immutable Docker image from Amazon ECR to Amazon ECS using Fargate.
We'll use the resources from your previous exercises:
- ECS cluster: devops-training-cluster
- ECS service: devops-training-service
- Task definition family: devops-training-task
- ECR repository: devops-training-app
- Container name: devops-app
- AWS region: ap-south-1
- Container port: 8000
1. Understand the deployment architecture
GitHub Actions
Test application and build image

Amazon ECR
Immutable, versioned Docker image

Amazon ECS + Fargate
Register task definition and update service

Running application
Container listens on port 8000

Health checks and CloudWatch
Verify tasks, logs, and application health


2. Verify your AWS resources
Run these commands in a terminal where the AWS CLI is configured:
aws sts get-caller-identity

aws ecs describe-clusters \
  --clusters devops-training-cluster \
  --region ap-south-1

aws ecs describe-services \
  --cluster devops-training-cluster \
  --services devops-training-service \
  --region ap-south-1


Check that the cluster exists and the service is associated with it. If the service does not exist yet, you must create it with the appropriate VPC, subnets, security groups, and Fargate configuration before deploying.
3. Select the immutable image
Use the image you pushed to ECR in the previous task.
In Bash, set the image URI using your actual AWS account and commit tag:
export AWS_REGION=ap-south-1
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export ECR_REPOSITORY=devops-training-app
export IMAGE_TAG=<YOUR_COMMIT_TAG>
export IMAGE_URI="$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY:$IMAGE_TAG"


Replace <YOUR_COMMIT_TAG> with your real tag, without the angle brackets.
Verify that the image exists:
aws ecr describe-images \
  --repository-name "$ECR_REPOSITORY" \
  --image-ids imageTag="$IMAGE_TAG" \
  --region "$AWS_REGION"


For a production release, consider using the verified image digest instead of relying only on a tag.
4. Register a new ECS task definition
Your task definition must reference the correct image, container name, execution role, task role, port, and CloudWatch log configuration.
If you already have task-definition.json from the previous task, update its image reference to the image you verified in ECR. Preserve the other settings your application needs.
For a file containing the minimal example from the previous task, you can use jq to create a deployment copy:
jq --arg image "$IMAGE_URI" \
  '.containerDefinitions |= map(
    if .name == "devops-app"
    then .image = $image
    else .
    end
  )' task-definition.json > task-definition-deploy.json


Check the result:
jq '.containerDefinitions[] | {name, image, portMappings}' \
  task-definition-deploy.json


Register the new revision:
aws ecs register-task-definition \
  --cli-input-json file://task-definition-deploy.json \
  --region "$AWS_REGION"


Important: The example task definition must contain valid role ARNs and valid log settings. If you have an existing production task definition, use its complete configuration and update only the intended image field.
5. Deploy the new revision to ECS
Get the latest task-definition revision ARN:
export TASK_DEF_ARN=$(aws ecs list-task-definitions \
  --family-prefix devops-training-task \
  --status ACTIVE \
  --sort DESC \
  --max-items 1 \
  --region "$AWS_REGION" \
  --query 'taskDefinitionArns[0]' \
  --output text)

echo "$TASK_DEF_ARN"


Confirm that the revision contains the expected image before updating the service.
Deploy it:
aws ecs update-service \
  --cluster devops-training-cluster \
  --service devops-training-service \
  --task-definition "$TASK_DEF_ARN" \
  --region "$AWS_REGION"


This tells ECS to update the service to use the new task-definition revision. ECS then replaces tasks according to the service's deployment configuration.
6. Wait for deployment stability
Run:
aws ecs wait services-stable \
  --cluster devops-training-cluster \
  --services devops-training-service \
  --region "$AWS_REGION"


If the command returns without an error, the ECS service reached the waiter's stable state. This alone does not prove the application is responding correctly, so verify task health and the application endpoint next.
Check service status:
aws ecs describe-services \
  --cluster devops-training-cluster \
  --services devops-training-service \
  --region "$AWS_REGION" \
  --query 'services[0].{Status:status,Desired:desiredCount,Running:runningCount,Pending:pendingCount,TaskDefinition:taskDefinition}'


For your training service with desired count 1, you would normally expect one running task and zero pending tasks after a successful deployment.
7. Verify application health
Check the running tasks:
aws ecs list-tasks \
  --cluster devops-training-cluster \
  --service-name devops-training-service \
  --region "$AWS_REGION"


Inspect the task details and container health:
aws ecs describe-tasks \
  --cluster devops-training-cluster \
  --tasks <TASK_ARN> \
  --region "$AWS_REGION"


Replace <TASK_ARN> with an actual task ARN from the preceding command.
If your service is behind an Application Load Balancer, retrieve its DNS name from the load balancer configuration and test:
curl -i http://YOUR_ALB_DNS_NAME/health


Expected application response:
{
  "status": "healthy"
}


The ALB listener, target group, security groups, and health-check path must be configured correctly for this test to succeed.
8. Troubleshoot common failures
Problem	What to check
CannotPullContainerError	ECR image URI, execution role permissions, network connectivity
Task exits immediately	CloudWatch logs, application startup command
Target group reports unhealthy	Port 8000, /health, ALB-to-ECS security-group rules
AccessDeniedException	IAM permissions for task registration and service updates
ECS service does not stabilize	ECS service events, failed tasks, deployment configuration
To inspect service events:
aws ecs describe-services \
  --cluster devops-training-cluster \
  --services devops-training-service \
  --region ap-south-1 \
  --query 'services[0].events[:10].[createdAt,message]' \
  --output table