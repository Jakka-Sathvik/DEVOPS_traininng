TASK 9 — Create ECS Service
Objective

Create an ECS Service named:

devops-training-service

The ECS Service ensures your application keeps running continuously.

Without a Service:

Task
 │
 ▼
Runs Once
 │
 ▼
Stops

With a Service:

Service
 │
 ▼
Monitors Task
 │
 ▼
Restarts if Failed
What is an ECS Service?

An ECS Service manages one or more running tasks.

It ensures the number of running tasks always matches the:

Desired Count

Example:

Desired Count = 1

ECS ensures:

1 Running Task

always exists.

Why Do We Need a Service?

Suppose you manually run a task.

Task
 │
 ▼
Container

If the container crashes:

Container
   ❌

Nothing restarts automatically.

With an ECS Service:

Task Stops
     │
     ▼
ECS Detects Failure
     │
     ▼
New Task Created

This provides self-healing.

Architecture
ECS Service
      │
      ▼
Desired Count = 1
      │
      ▼
Running Task
      │
      ▼
Container
Components Required Before Creating Service

You should already have:

ECS Cluster
devops-training-cluster

Verify:

aws ecs list-clusters
Task Definition
devops-training-task

Verify:

aws ecs list-task-definitions
ECR Image
devops-training-app:v1.0.0
Step 1 — Understand Desired Count

Desired Count tells ECS how many copies of the task should run.

Example:

Desired Count = 1

Architecture:

ECS Service
      │
      ▼
Task 1

Example:

Desired Count = 3

Architecture:

Task 1
Task 2
Task 3

ECS keeps all three running.

Step 2 — Create Service Using AWS Console

Open:

AWS Console
     │
     ▼
Amazon ECS

Select:

Clusters

Open:

devops-training-cluster

Click:

Create Service
Step 3 — Configure Service

Enter:

Launch Type
FARGATE
Task Definition
devops-training-task

Select latest revision.

Example:

devops-training-task:1
Service Name
devops-training-service
Desired Tasks
1
Networking

Choose:

VPC

Example:

training-vpc

Choose subnets:

Private Subnet A
Private Subnet B

Example:

10.0.11.0/24
10.0.12.0/24

Choose security group:

ecs-app-sg

Assign public IP:

Training:

Enabled

Production:

Disabled

when using an ALB.

Step 4 — Create Service

Click:

Create

AWS creates:

devops-training-service
CLI Method

You can also create the service using AWS CLI.

Run:

aws ecs create-service \
  --cluster devops-training-cluster \
  --service-name devops-training-service \
  --task-definition devops-training-task \
  --desired-count 1 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-12345678],securityGroups=[sg-12345678],assignPublicIp=ENABLED}"

Replace:

subnet-12345678

with your subnet ID.

Replace:

sg-12345678

with your security group ID.

Step 5 — Verify Service

Open:

ECS
 │
 ▼
Cluster
 │
 ▼
Services

Expected:

devops-training-service

Status:

ACTIVE

CLI verification:

aws ecs describe-services \
  --cluster devops-training-cluster \
  --services devops-training-service

Expected:

Status:
ACTIVE
Step 6 — Verify Running Task

Open:

Cluster
 │
 ▼
Tasks

Expected:

1 Running Task

Architecture:

ECS Service
      │
      ▼
Running Task
      │
      ▼
Container
Self-Healing Demonstration

Suppose the container crashes.

Before:

Desired Count = 1

Task 1

Task fails:

Task 1
   ❌

ECS detects:

Running Tasks = 0

Desired Tasks = 1

Mismatch detected.

ECS automatically creates:

Task 2

Result:

Desired Count = 1

Task 2

Application remains available.

Verify Self-Healing

Stop a task manually.

Console:

Cluster
 │
 ▼
Tasks
 │
 ▼
Stop Task

Observe:

Stopped Task

Within a few moments:

New Task Created

because the service maintains:

Desired Count = 1
ECS Service Architecture
Amazon ECR
      │
      ▼
Task Definition
      │
      ▼
ECS Service
(devops-training-service)
      │
      ▼
Desired Count = 1
      │
      ▼
Fargate Task
      │
      ▼
Container
Service vs Task
Feature	Task	Service
Runs Container	Yes	Yes
Self-Healing	No	Yes
Maintains Desired Count	No	Yes
Automatic Recovery	No	Yes
Long-Running Application	No	Yes
Production Recommended	No	Yes
Deliverable Verification

Run:

aws ecs describe-services \
  --cluster devops-training-cluster \
  --services devops-training-service

You have successfully completed the task when:

✓ Service Name:
devops-training-service

✓ Cluster:
devops-training-cluster

✓ Task Definition:
devops-training-task

✓ Launch Type:
FARGATE

✓ Desired Count:
1

✓ Running Tasks:
1

✓ Status:
ACTIVE
Key Takeaways
An ECS Service manages and maintains running tasks.
Desired Count specifies how many task copies ECS should keep running.
If a task stops unexpectedly, ECS automatically launches a replacement.
ECS Services provide self-healing and high availability.
Fargate tasks are commonly run through ECS Services rather than standalone tasks.
ECS Services are the foundation for production container deployments on AWS.