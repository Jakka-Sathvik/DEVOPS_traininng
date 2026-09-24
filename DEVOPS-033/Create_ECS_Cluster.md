creating the logical environment where ECS will run your containers.

Think of it like this:

AWS Account
      │
      ▼
ECS Cluster
      │
      ▼
Services
      │
      ▼
Tasks
      │
      ▼
Containers
What is an ECS Cluster?

An ECS Cluster is a logical grouping of resources where ECS runs your applications.

Example:

devops-training-cluster
│
├── ECS Service
├── ECS Tasks
└── Containers

Without a cluster:

Task
 ❌
 Cannot Run

Because ECS needs a cluster to place tasks.

Method 1: Create ECS Cluster Using AWS Console
Step 1: Open ECS

Login to AWS Console.

Search:

ECS

Open:

Amazon Elastic Container Service
Step 2: Open Clusters

Left Menu:

Clusters

Click:

Create Cluster
Step 3: Configure Cluster

Enter:

Cluster Name:
devops-training-cluster

Infrastructure:

AWS Fargate

Leave defaults.

Click:

Create

AWS will create:

devops-training-cluster
Step 4: Verify in Console

You should see:

Clusters

devops-training-cluster

Open it.

Expected:

Services = 0
Tasks = 0
Capacity Providers = FARGATE

This is normal because nothing has been deployed yet.

Method 2: Create ECS Cluster Using AWS CLI
Step 1: Verify AWS Login

Run:

aws sts get-caller-identity

Expected:

{
  "Account": "123456789012"
}
Step 2: Create Cluster

Run:

aws ecs create-cluster \
  --cluster-name devops-training-cluster

Expected output:

{
  "cluster": {
    "clusterName": "devops-training-cluster",
    "status": "ACTIVE"
  }
}
Understanding What Happened

Before:

AWS Account

After:

AWS Account
      │
      ▼
devops-training-cluster

The cluster is now available to host:

Services
Tasks
Containers
Step 3: Verify Cluster

Run:

aws ecs describe-clusters \
  --clusters devops-training-cluster

Expected:

{
  "clusters": [
    {
      "clusterName": "devops-training-cluster",
      "status": "ACTIVE"
    }
  ]
}

Important value:

status = ACTIVE

Meaning:

Cluster Ready
Step 4: List All Clusters

Run:

aws ecs list-clusters

Expected:

arn:aws:ecs:ap-south-1:123456789012:cluster/devops-training-cluster

This shows all clusters in your account.

Visual Architecture

Current environment:

Amazon ECR
     │
     ▼
devops-training-app:v1.0.0

After cluster creation:

Amazon ECR
     │
     ▼
ECS Cluster
(devops-training-cluster)

Next task will connect:

Amazon ECR
      │
      ▼
Task Definition
      │
      ▼
ECS Cluster
      │
      ▼
Fargate
      │
      ▼
Container
Useful Commands
Create Cluster
aws ecs create-cluster \
  --cluster-name devops-training-cluster
Verify Cluster
aws ecs describe-clusters \
  --clusters devops-training-cluster
List Clusters
aws ecs list-clusters
Delete Cluster (Only for Cleanup)
aws ecs delete-cluster \
  --cluster devops-training-cluster
Deliverable Verification

Run:

aws ecs describe-clusters \
  --clusters devops-training-cluster

You have completed the task when:

Cluster Name:
devops-training-cluster

Status:
ACTIVE

Final architecture after Task 4:

AWS Account
      │
      ▼
ECS Cluster
(devops-training-cluster)
      │
      └── Ready for ECS Services