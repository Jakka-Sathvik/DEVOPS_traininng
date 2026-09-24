TASK 6 — Create ECS Task Definition
Objective

Create an ECS Task Definition named:

devops-training-task

A Task Definition is the blueprint that tells ECS exactly how to run your container.

Think of it like a Docker Compose file for ECS.

Without a Task Definition:

ECS
 │
 ▼
❌ Doesn't Know

Which Image?
How Much CPU?
How Much Memory?
Which Port?
Which Logs?

With a Task Definition:

Task Definition
      │
      ▼
Everything ECS Needs
What is a Task Definition?

A Task Definition is a JSON document that contains:

Container Image
CPU
Memory
Ports
Environment Variables
Logging
IAM Roles
Networking

Architecture:

Task Definition
│
├── Container Image
├── CPU
├── Memory
├── Port Mapping
├── Logging
├── IAM Roles
└── Environment Variables
Step 1 — Gather Information

Before creating the Task Definition, collect:

ECS Cluster

Created earlier:

devops-training-cluster
ECR Image

Verify image exists:

aws ecr list-images \
  --repository-name devops-training-app

Example:

v1.0.0

Image URI:

123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app:v1.0.0

You will use this inside the Task Definition.

Step 2 — Understand Each Configuration
Family

Task name.

{
  "family": "devops-training-task"
}

Think of it as:

Task Definition Name
Fargate Compatibility

Because we're using Fargate:

{
  "requiresCompatibilities": [
    "FARGATE"
  ]
}

Meaning:

ECS
 │
 ▼
Fargate

not:

ECS
 │
 ▼
EC2
Network Mode

Use:

{
  "networkMode": "awsvpc"
}

Why?

Because Fargate requires:

Task
 │
 ▼
ENI
 │
 ▼
Private IP

and awsvpc enables that.

Step 3 — Configure CPU

Example:

{
  "cpu": "512"
}

Meaning:

0.5 vCPU

Good for training applications.

Step 4 — Configure Memory

Example:

{
  "memory": "1024"
}

Meaning:

1 GB RAM

Suitable for:

Python App
NodeJS App
Training API
Step 5 — Configure Container Image

Container section:

{
  "name": "devops-app",
  "image": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app:v1.0.0"
}

Architecture:

Task Definition
       │
       ▼
ECR Image
       │
       ▼
Container
Step 6 — Configure Port Mapping

Suppose your application runs:

Port 8000

Configure:

{
  "portMappings": [
    {
      "containerPort": 8000,
      "protocol": "tcp"
    }
  ]
}

Meaning:

Internet
     │
     ▼
ALB
     │
     ▼
Container Port 8000
Step 7 — Configure Environment Variables

Applications often need configuration.

Example:

{
  "environment": [
    {
      "name": "ENVIRONMENT",
      "value": "training"
    }
  ]
}

Application receives:

ENVIRONMENT=training

Inside the container.

Step 8 — Configure Logging

Without logging:

Application
 │
 ▼
Logs Lost

With CloudWatch:

Application
      │
      ▼
CloudWatch Logs

Configuration:

{
  "logConfiguration": {
    "logDriver": "awslogs",
    "options": {
      "awslogs-group": "/ecs/devops-training-app",
      "awslogs-region": "ap-south-1",
      "awslogs-stream-prefix": "ecs"
    }
  }
}
Step 9 — IAM Configuration
Execution Role

Used by ECS.

Responsibilities:

Pull Image From ECR
Send Logs To CloudWatch

Example:

{
  "executionRoleArn":
  "arn:aws:iam::123456789012:role/ecsTaskExecutionRole"
}

Architecture:

ECS
 │
 ▼
Execution Role
 │
 ├── ECR
 └── CloudWatch
Task Role

Used by the application.

Example:

{
  "taskRoleArn":
  "arn:aws:iam::123456789012:role/devops-training-task-role"
}

Architecture:

Application
      │
      ▼
Task Role
      │
      ▼
AWS Services
Complete Task Definition Example

Create file:

task-definition.json
{
  "family": "devops-training-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": [
    "FARGATE"
  ],
  "cpu": "512",
  "memory": "1024",

  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",

  "containerDefinitions": [
    {
      "name": "devops-app",

      "image": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app:v1.0.0",

      "essential": true,

      "portMappings": [
        {
          "containerPort": 8000,
          "protocol": "tcp"
        }
      ],

      "environment": [
        {
          "name": "ENVIRONMENT",
          "value": "training"
        }
      ],

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
Step 10 — Register Task Definition

Save the JSON.

Run:

aws ecs register-task-definition \
  --cli-input-json file://task-definition.json

Expected:

Task Definition Registered
Step 11 — Verify Registration

Run:

aws ecs list-task-definitions

Expected:

devops-training-task

Detailed verification:

aws ecs describe-task-definition \
  --task-definition devops-training-task

Expected:

Family:
devops-training-task

CPU:
512

Memory:
1024

Launch Type:
FARGATE
Visual Architecture
Amazon ECR
      │
      ▼
Task Definition
(devops-training-task)
      │
      ├── Image
      ├── CPU
      ├── Memory
      ├── Port
      ├── Logs
      ├── IAM
      └── Environment Variables
      │
      ▼
ECS Service
      │
      ▼
Fargate Task
Deliverable Verification

Run:

aws ecs describe-task-definition \
  --task-definition devops-training-task

You have successfully completed the task when:

✓ Task Definition Exists

✓ Family:
devops-training-task

✓ Launch Type:
FARGATE

✓ CPU:
512

✓ Memory:
1024

✓ Container Image Configured

✓ Port Mapping Configured

✓ Logging Configured

This Task Definition is now ready to be used by an ECS Service, which will launch and maintain running Fargate tasks.