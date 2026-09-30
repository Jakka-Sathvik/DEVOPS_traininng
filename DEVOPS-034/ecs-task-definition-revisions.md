ECS Task Definition Revisions
Objective

Understand how Amazon ECS Task Definitions are versioned and how new application deployments create new task definition revisions.

What is a Task Definition?

A Task Definition is a blueprint that tells ECS how to run a container.

It contains:

Docker image
CPU allocation
Memory allocation
Environment variables
Port mappings
Logging configuration
IAM role configuration

Example:

Task Definition
      ↓
Instructions for ECS
      ↓
Run Container
Why Revisions Exist

Applications change over time.

Example:

Version 1.0.0
      ↓
Version 1.1.0
      ↓
Version 1.2.0

Instead of modifying an existing task definition, ECS creates a new revision.

This provides:

Deployment history
Rollback capability
Version tracking
Safer deployments
ECS Revision Architecture
Task Definition Family

devops-training-task

        │
        ├── Revision 1
        │
        ├── Revision 2
        │
        └── Revision 3

Each revision has its own configuration.

Example Revision History
Revision 1
Image: devops-training:v1.0.0

Revision 2
Image: devops-training:v1.1.0

Revision 3
Image: devops-training:v1.2.0
Deployment Flow

Whenever a new application version is released:

Developer
      ↓
Docker Build
      ↓
Docker Image
      ↓
Amazon ECR
      ↓
New Task Definition Revision
      ↓
ECS Service Update
      ↓
New ECS Tasks
Revision 1

Task Definition Family:

devops-training-task

Revision:

devops-training-task:1

Container Image:

123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training:v1.0.0

Example Configuration:

{
  "family": "devops-training-task",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training:v1.0.0",
      "cpu": 256,
      "memory": 512,
      "essential": true,
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ]
    }
  ]
}

Purpose:

Initial Production Release
Revision 2

A new application version is released.

Docker Image:

devops-training:v1.1.0

New Task Definition Revision:

devops-training-task:2

Example Configuration:

{
  "family": "devops-training-task",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training:v1.1.0",
      "cpu": 256,
      "memory": 512,
      "essential": true,
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ]
    }
  ]
}

Changes:

Revision 1
      ↓
Image v1.0.0

Revision 2
      ↓
Image v1.1.0
ECS Service Update

After Revision 2 is created:

Old Service
      ↓
Revision 1

Update Service:

ECS Service
      ↓
Use Revision 2

ECS performs deployment:

Start New Tasks
      ↓
Health Check
      ↓
Healthy
      ↓
Remove Old Tasks
Running Deployment Example

Before Update:

Service
    ↓
Task Definition:1
    ↓
Image:v1.0.0

After Update:

Service
    ↓
Task Definition:2
    ↓
Image:v1.1.0
Rollback Example

Suppose version 1.1.0 fails.

Current:

Revision 2
Image:v1.1.0

Rollback:

Service
      ↓
Switch Back
      ↓
Revision 1
      ↓
Image:v1.0.0

Result:

Application Restored
Real Production Revision Timeline
devops-training-task:1
      ↓
Image:v1.0.0

devops-training-task:2
      ↓
Image:v1.1.0

devops-training-task:3
      ↓
Image:v1.2.0

devops-training-task:4
      ↓
Image:v1.3.0

Each revision remains available for future rollback if needed.

AWS Console Steps
Create Revision 1
ECS
 ↓
Task Definitions
 ↓
Create New Task Definition
 ↓
Family:
devops-training-task
 ↓
Image:
v1.0.0
 ↓
Create
Create Revision 2
ECS
 ↓
Task Definitions
 ↓
devops-training-task
 ↓
Create New Revision
 ↓
Update Image
 ↓
v1.1.0
 ↓
Create

AWS automatically creates:

devops-training-task:2
Verification

Navigate to:

ECS
 ↓
Task Definitions
 ↓
devops-training-task

Expected:

devops-training-task:1

devops-training-task:2

Minimum requirement achieved:

✅ Revision 1 Created

✅ Revision 2 Created

Key Learning

A Task Definition is versioned through revisions. Every application release should create a new revision instead of modifying an existing one. ECS Services deploy new revisions, allowing controlled deployments, version tracking, and easy rollback
