TASK 8 — Configure CloudWatch Container Logging
Objective

Configure your ECS Fargate container to automatically send logs to Amazon CloudWatch Logs.

This allows you to view:

Application logs
Startup logs
Error logs
Debug logs
Container output

without logging into any server.

Why CloudWatch Logging?

In traditional servers:

EC2
 │
 ▼
SSH
 │
 ▼
View Logs

With ECS Fargate:

Container
     │
     ▼
CloudWatch Logs

No SSH access is required.

Logging Architecture
Application
     │
     ▼
stdout / stderr
     │
     ▼
Docker Container
     │
     ▼
awslogs Driver
     │
     ▼
CloudWatch Logs
Step 1 — Create CloudWatch Log Group

Before deploying the task, create a log group.

Run:

aws logs create-log-group \
  --log-group-name /ecs/devops-training-app

Verify:

aws logs describe-log-groups

Expected:

/ecs/devops-training-app

Architecture:

CloudWatch
      │
      ▼
/ecs/devops-training-app
Step 2 — Verify Execution Role Permissions

The ECS Task Execution Role must be able to write logs.

Usually AWS provides:

AmazonECSTaskExecutionRolePolicy

Permissions include:

logs:CreateLogStream
logs:PutLogEvents

Verify:

aws iam get-role \
  --role-name ecsTaskExecutionRole
Step 3 — Update ECS Task Definition

Inside your container definition add:

"logConfiguration": {
  "logDriver": "awslogs",
  "options": {
    "awslogs-group": "/ecs/devops-training-app",
    "awslogs-region": "ap-south-1",
    "awslogs-stream-prefix": "ecs"
  }
}
Understanding Each Option
awslogs-group
"awslogs-group": "/ecs/devops-training-app"

Destination log group.

Architecture:

Container
     │
     ▼
/ecs/devops-training-app
awslogs-region
"awslogs-region": "ap-south-1"

Region where logs are stored.

Example:

Mumbai Region
awslogs-stream-prefix
"awslogs-stream-prefix": "ecs"

CloudWatch creates streams like:

ecs/devops-app/123456
ecs/devops-app/789123
Complete Container Definition Example
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

  "logConfiguration": {
    "logDriver": "awslogs",
    "options": {
      "awslogs-group": "/ecs/devops-training-app",
      "awslogs-region": "ap-south-1",
      "awslogs-stream-prefix": "ecs"
    }
  }
}
Step 4 — Register Updated Task Definition

After adding logging configuration:

aws ecs register-task-definition \
  --cli-input-json file://task-definition.json

Expected:

Task Definition Registered
Step 5 — Deploy ECS Service

When the service launches:

Task Definition
      │
      ▼
Fargate Task
      │
      ▼
Container Starts

CloudWatch log streams are created automatically.

Step 6 — Generate Application Traffic

Open the application.

Example:

http://ALB-DNS

Refresh multiple times.

Generate requests:

GET /
GET /health
GET /status

Each request should create logs.

Step 7 — Verify Logs in AWS Console

Open:

AWS Console
    │
    ▼
CloudWatch
    │
    ▼
Log Groups

Open:

/ecs/devops-training-app

You should see:

Log Streams

Example:

ecs/devops-app/123456
ecs/devops-app/789456

Open a stream.

Expected:

Application Started
Listening on Port 8000
GET /
GET /health
Step 8 — Verify Logs Using AWS CLI

List streams:

aws logs describe-log-streams \
  --log-group-name /ecs/devops-training-app

Expected:

ecs/devops-app/123456

Get log events:

aws logs get-log-events \
  --log-group-name /ecs/devops-training-app \
  --log-stream-name STREAM_NAME

Example:

Application Started
Listening on Port 8000
What Gets Logged?
Startup Logs
Application Starting
Database Connected
Server Listening
Application Logs
GET /
GET /health
GET /products
Error Logs
Database Connection Failed
Timeout Error
Internal Server Error
Real Production Architecture
Internet User
      │
      ▼
Application Load Balancer
      │
      ▼
ECS Service
      │
      ▼
Fargate Task
      │
      ▼
Container Logs
      │
      ▼
CloudWatch Logs
      │
      ├── Startup Logs
      ├── Application Logs
      └── Error Logs
Troubleshooting
No Logs Appearing

Check:

Execution Role

Must include:

AmazonECSTaskExecutionRolePolicy
Wrong Log Group

Verify:

"awslogs-group":
"/ecs/devops-training-app"

matches the actual log group.

Wrong Region

Verify:

"awslogs-region":
"ap-south-1"

matches your deployment region.

Container Not Starting

Check:

ECS
 → Tasks
 → Stopped Tasks

for startup errors.

Deliverable Verification

You have completed the task when:

✓ Log Group Exists

/ecs/devops-training-app

✓ ECS Task Running

✓ Log Stream Created

✓ Startup Logs Visible

✓ Application Logs Visible

✓ Error Logs Visible
Key Takeaways
CloudWatch Logs is the standard logging solution for ECS and Fargate.
Containers send logs through the awslogs log driver.
Logs are stored in a CloudWatch Log Group such as /ecs/devops-training-app.
The Task Execution Role requires permissions to create log streams and write log events.
Startup logs, application logs, and error logs can all be viewed from the CloudWatch console.
CloudWatch logging is critical for troubleshooting, monitoring, and production support of containerized applications.