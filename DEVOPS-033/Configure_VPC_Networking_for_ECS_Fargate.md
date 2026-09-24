TASK 10 — Configure VPC Networking for ECS Fargate

This task is important because Fargate does not remove networking responsibilities. AWS manages the underlying compute host, but you still decide where the task runs and who can communicate with it.

We'll build this architecture:

                    Internet
                       │
                       ▼
                ┌─────────────┐
                │     ALB     │
                │   ALB SG    │
                └──────┬──────┘
                       │
                       │ TCP 8000
                       ▼
              ┌─────────────────┐
              │ Private Subnet  │
              │                 │
              │  Fargate Task   │
              │   ECS SG        │
              │   :8000         │
              └─────────────────┘
                       │
                       ▼
                     VPC

Important: If you are using the ECS service you created in Task 9, changing its networking configuration can require updating/recreating the service. For this task, first understand the network design, then apply it carefully.

1. Understand the VPC
What is a VPC?

VPC = Virtual Private Cloud.

It is your private network inside AWS.

Example:

VPC
10.0.0.0/16

This gives you a large private IP address space.

Think of it as:

AWS
 │
 └── Your VPC
       │
       ├── Subnet A
       ├── Subnet B
       ├── Security Groups
       └── Route Tables
2. Understand Subnets

A subnet is a smaller network inside the VPC.

For your training architecture, you can use:

VPC
10.0.0.0/16
│
├── Public Subnet A
│   10.0.1.0/24
│
├── Public Subnet B
│   10.0.2.0/24
│
├── Private Subnet A
│   10.0.11.0/24
│
└── Private Subnet B
    10.0.12.0/24

Why two subnets of each type?

Because production architectures normally spread resources across multiple Availability Zones.

3. Public vs Private Subnet
Public Subnet

A subnet is considered public when its route table has a route to an Internet Gateway.

Example:

Public Subnet
      │
      ▼
Internet Gateway
      │
      ▼
Internet

The ALB can be placed here.

Private Subnet

A private subnet does not have a direct route to an Internet Gateway.

Private Subnet
      │
      ▼
NAT Gateway
      │
      ▼
Internet

A Fargate application can run here.

4. Recommended Architecture

For an internet-facing application:

                         Internet
                            │
                            ▼
                    ┌──────────────┐
                    │     ALB      │
                    │   Public     │
                    │   Subnets    │
                    └──────┬───────┘
                           │
                           │ :8000
                           ▼
                  ┌─────────────────┐
                  │ Fargate Task    │
                  │ Private Subnet  │
                  │                 │
                  │ Container :8000 │
                  └─────────────────┘

This is preferable to:

Internet
   │
   ▼
Fargate Task :8000

because the ALB becomes the intended public entry point.

5. Step 1 — Find Your VPC

Using AWS Console:

AWS Console
   ↓
VPC
   ↓
Your VPCs

Find the VPC you want to use.

You might see:

Name: training-vpc
IPv4 CIDR: 10.0.0.0/16

Record:

VPC ID

For example:

vpc-0123456789abcdef

Your ID will be different.

6. Step 2 — Check Your Subnets

Go to:

VPC
 ↓
Subnets

You should identify:

Public Subnet A
Public Subnet B

Private Subnet A
Private Subnet B

For example:

Public A:
10.0.1.0/24

Public B:
10.0.2.0/24

Private A:
10.0.11.0/24

Private B:
10.0.12.0/24

Also check the Availability Zone.

Example:

Public A  → ap-south-1a
Public B  → ap-south-1b

Private A → ap-south-1a
Private B → ap-south-1b
7. Step 3 — Check Route Tables

Go to:

VPC
 ↓
Route Tables
Public Route Table

A public route table normally contains:

Destination       Target

10.0.0.0/16       local
0.0.0.0/0        Internet Gateway

Architecture:

Public Subnet
      │
      ▼
Route Table
      │
      ├── VPC local
      │
      └── 0.0.0.0/0
              │
              ▼
         Internet Gateway
8. Private Route Table

A private subnet should not have:

0.0.0.0/0 → Internet Gateway

For internet access from private resources, you can use:

Private Subnet
      │
      ▼
Private Route Table
      │
      ▼
NAT Gateway
      │
      ▼
Internet Gateway
      │
      ▼
Internet

Example:

Destination       Target

10.0.0.0/16       local
0.0.0.0/0         nat-xxxxxxxx
9. Why Does Fargate Need Internet Connectivity?

This depends on your application architecture.

For example, your Fargate task may need to:

Pull an image from ECR
Send logs to CloudWatch
Access AWS APIs
Download external dependencies
Access external APIs

A task in a private subnet can reach these services through appropriate AWS networking.

There are two common approaches.

Option A — NAT Gateway
Private Fargate Task
        │
        ▼
NAT Gateway
        │
        ▼
Internet Gateway
        │
        ▼
Internet

NAT Gateway has a cost, so be conscious of it during training.

Option B — VPC Endpoints

For some AWS services, VPC endpoints can provide private connectivity without sending traffic through the public internet.

For example, AWS provides VPC endpoints for services such as S3 and ECR.

The exact endpoint requirements depend on the services your task uses.

10. Step 4 — Create ALB Security Group

Go to:

VPC
 ↓
Security Groups
 ↓
Create Security Group

Name:

alb-sg

Description:

Security group for ECS Application Load Balancer

Select your VPC.

ALB Inbound Rules

For an HTTP application:

Type: HTTP
Protocol: TCP
Port: 80
Source: 0.0.0.0/0

Architecture:

Internet
    │
    ▼
ALB SG
    │
    ▼
ALB :80

For HTTPS, you would also configure port 443 appropriately.

11. Step 5 — Create ECS Security Group

Create another security group:

ecs-task-sg

Description:

Security group for ECS Fargate tasks

Use the same VPC.

12. ECS Security Group Inbound Rule

This is one of the most important parts.

Do not configure:

Port: 8000
Source: 0.0.0.0/0

Instead:

Type: Custom TCP
Port: 8000
Source: alb-sg

Architecture:

Internet
    │
    ▼
 ALB
    │
    ▼
 ALB SG
    │
    │ TCP 8000
    ▼
 ECS SG
    │
    ▼
Fargate Container :8000

This means the application accepts traffic from the ALB rather than directly from everyone on the internet.

13. Understand Security Group Flow

Your security groups should logically work like this:

             Internet
                 │
                 │ TCP 80
                 ▼
             ┌───────┐
             │ ALB   │
             └───┬───┘
                 │
                 │ TCP 8000
                 ▼
            ┌─────────┐
            │ ECS SG  │
            └────┬────┘
                 │
                 ▼
          Container :8000

The important relationship is:

ECS SG
Inbound TCP 8000
Source = ALB SG

Not:

Source = 0.0.0.0/0
14. Step 6 — Understand the ECS Task ENI

When Fargate starts a task using awsvpc networking, the task gets an Elastic Network Interface.

Fargate Task
      │
      ▼
     ENI
      │
      ├── Private IP
      ├── Subnet
      └── Security Group

Example:

Task
Private IP:
10.0.11.25

The ENI is what connects your Fargate task to your VPC.

15. Step 7 — Configure ECS Service Networking

Go to:

AWS Console
 ↓
ECS
 ↓
Clusters
 ↓
devops-training-cluster
 ↓
devops-training-service
 ↓
Update

During service configuration, select:

VPC

Your training VPC:

training-vpc
Subnets

Select private subnets:

Private Subnet A
Private Subnet B

Example:

10.0.11.0/24
10.0.12.0/24
Security Group

Select:

ecs-task-sg
16. Public IP Setting

For the production-style architecture:

ALB
 ↓
Private Fargate Task

the Fargate task does not need a public IP.

Therefore:

Assign Public IP:
Disabled

The ALB is the public entry point.

17. Important: ALB Must Know About the ECS Service

Networking alone is not enough.

You eventually need:

ALB
 │
 ▼
Target Group
 │
 ▼
ECS Service
 │
 ▼
Fargate Task

The target group should use the appropriate container port:

8000

For example:

ALB Listener
Port 80
     │
     ▼
Target Group
Port 8000
     │
     ▼
Fargate Task
Port 8000
18. Health Check

The ALB needs to know whether the container is healthy.

If your application has:

/health

configure:

Protocol: HTTP
Port: 8000
Path: /health

Architecture:

ALB
 │
 ├── Health Check
 │
 ▼
Fargate
 │
 ▼
GET /health

Expected:

HTTP 200
19. Verify ECS Task Networking

After the service starts, go to:

ECS
 ↓
Clusters
 ↓
devops-training-cluster
 ↓
Tasks

Open the running task.

Look for:

Networking

You should see:

VPC
Subnet
Security Group
Private IP
ENI

Example:

VPC:
vpc-xxxxxxxx

Subnet:
subnet-xxxxxxxx

Private IP:
10.0.11.25

Security Group:
ecs-task-sg
20. Verify From AWS CLI

List running tasks:

aws ecs list-tasks \
  --cluster devops-training-cluster

Then inspect the task:

aws ecs describe-tasks \
  --cluster devops-training-cluster \
  --tasks TASK_ARN

Look for networking information:

networkInterfaces
privateIPv4Address
subnetId
securityGroups
21. Test the Application

Once the ALB is configured:

Internet
   │
   ▼
ALB DNS Name
   │
   ▼
ALB :80
   │
   ▼
Target Group
   │
   ▼
Fargate :8000

Open the ALB DNS name:

http://ALB-DNS-NAME

Your application should respond.

22. Test the Health Endpoint

If your application provides:

/health

test:

http://ALB-DNS-NAME/health

Expected:

HTTP 200
23. Troubleshooting

If the ALB cannot reach the Fargate task, check in this order.

Check 1 — Task Running
ECS
 ↓
Tasks
 ↓
Status = RUNNING
Check 2 — Container Port

Task Definition:

containerPort = 8000

Application:

listening on 8000

These must match.

Check 3 — ECS Security Group

Check:

TCP 8000
Source = ALB SG
Check 4 — ALB Security Group

Check:

TCP 80
Source = 0.0.0.0/0

for an internet-facing HTTP ALB.

Check 5 — Target Group

Check:

Target = Fargate Task
Port = 8000
Health = Healthy
Check 6 — Application Binding

Your application should normally listen on:

0.0.0.0:8000

rather than only:

127.0.0.1:8000

Otherwise the application may not accept traffic arriving through the task network interface.

24. Final Architecture

Your production-style ECS networking should look like:

                              Internet
                                  │
                                  ▼
                         ┌────────────────┐
                         │      ALB       │
                         │  Public Subnet │
                         └───────┬────────┘
                                 │
                           TCP 8000
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │    Private Subnet A    │
                    │                        │
                    │    Fargate Task        │
                    │    ECS Security Group  │
                    │         :8000          │
                    └────────────────────────┘
                                 │
                                 ▼
                               VPC

For high availability:

                         Internet
                             │
                             ▼
                            ALB
                         /       \
                        /         \
                       ▼           ▼
                Private A      Private B
                    │               │
                    ▼               ▼
               Fargate A       Fargate B
25. What Each Component Does
Component	Purpose
VPC	Provides the private AWS network
Public Subnet	Hosts internet-facing resources such as ALB
Private Subnet	Hosts application tasks
Route Table	Controls network routing
Internet Gateway	Provides internet connectivity for public resources
NAT Gateway	Allows private resources to initiate outbound internet connections
Security Group	Acts as a virtual firewall
ENI	Connects Fargate task to the VPC
ALB	Receives public application traffic
ECS Service	Maintains the desired Fargate tasks
Fargate	Runs the container without you managing the host
26. Final Verification Checklist

Before considering Task 10 complete:

✓ VPC identified

✓ Public subnets identified

✓ Private subnets identified

✓ Route tables checked

✓ Internet Gateway understood/configured

✓ NAT connectivity configured if required

✓ ALB Security Group created

✓ ECS Security Group created

✓ ECS SG allows TCP 8000 from ALB SG

✓ ECS task uses private subnet

✓ ECS task uses ecs-task-sg

✓ Fargate task receives an ENI

✓ Fargate task receives a private IP

✓ Application listens on port 8000

The key security principle is:

Internet
   │
   ▼
ALB SG
   │
   ▼
ECS SG
   │
   ▼
Fargate :8000

Do not expose Fargate :8000 directly to 0.0.0.0/0 when the ALB is intended to be the application's public entry point.