TASK 5 — Understand AWS Fargate
Objective

Understand how AWS Fargate allows you to run containers without managing EC2 servers while still requiring proper networking and security configuration.

Why Fargate?

Before Fargate, running containers required managing EC2 instances.

EC2 Launch Model
ECS
 ↓
EC2
 ↓
Docker
 ↓
Container

In this model, you manage:

EC2 instances
Operating System updates
Docker installation
Docker upgrades
Server capacity
Auto Scaling Groups
Infrastructure maintenance
Fargate Launch Model
ECS
 ↓
Fargate
 ↓
Container

With Fargate:

No EC2 management
No OS patching
No Docker installation
No server capacity planning
No infrastructure maintenance

AWS manages the underlying infrastructure.

What Fargate Actually Does

When you deploy a task:

Task Definition
      │
      ▼
Fargate
      │
      ▼
AWS Creates Compute Resources
      │
      ▼
Container Starts

You provide:

Container image
CPU
Memory
Networking configuration

AWS provides:

Compute resources
Host operating system
Container runtime
Infrastructure management
CPU in Fargate

CPU defines how much processing power your container receives.

Think of CPU as the number of workers available to perform tasks.

Example
CPU = 256

Equals:

0.25 vCPU

Common CPU values:

CPU Units	Approximate vCPU
256	0.25
512	0.5
1024	1
2048	2
4096	4
Example

Small training application:

CPU = 256

Production API:

CPU = 1024
Memory in Fargate

Memory determines how much RAM is available.

Think of memory as the workspace available to the application.

Example
Memory = 512

Equals:

512 MB RAM

Common values:

Memory	Usage
512 MB	Small applications
1024 MB	APIs
2048 MB	Medium workloads
4096 MB	Large applications

Example:

CPU = 256
Memory = 512

Suitable for:

Training Projects
Demo Applications
Small APIs
Task Networking

Every Fargate task gets its own networking configuration.

Unlike traditional Docker:

Multiple Containers
        │
        ▼
Single Host Network

Fargate uses:

Task
 │
 ▼
Dedicated Network Interface

Each task receives:

Private IP
Security Group
Network routing
What is an ENI?

ENI = Elastic Network Interface

Think of an ENI as:

Virtual Network Card

When ECS starts a Fargate task:

Fargate Task
      │
      ▼
Creates ENI
      │
      ▼
Assigns Private IP

Example:

Task A
10.0.1.25

Task B
10.0.1.37

Each task has its own network identity.

Why ENI Matters

The ENI allows AWS to attach:

Security Groups
Private IP addresses
Network policies
Monitoring

Architecture:

Container
    │
    ▼
Task
    │
    ▼
ENI
    │
    ▼
Private IP

Without an ENI:

No Network Access
Understanding Subnets

A subnet is a portion of a VPC network.

Example VPC:

10.0.0.0/16

Subnets:

10.0.1.0/24
10.0.2.0/24
10.0.11.0/24
10.0.12.0/24

Architecture:

VPC
│
├── Public Subnet
│
├── Public Subnet
│
├── Private Subnet
│
└── Private Subnet
Public vs Private Subnets
Public Subnet

Can communicate with the internet.

Example:

Internet
    │
    ▼
Public Subnet

Usually contains:

Load Balancers
Bastion Hosts
Private Subnet

Not directly accessible from the internet.

Example:

Internet
    │
    ▼
ALB
    │
    ▼
Private Subnet

Usually contains:

ECS Tasks
Databases
Internal Services
Security Groups

Security Groups act as firewalls.

They control:

Who Can Connect
What Port Can Be Used

Example:

Allow:

TCP 80
TCP 443

Block:

Everything Else
Example Security Group

Application receives traffic only from an ALB.

ALB
 │
 ▼
Security Group
 │
 ▼
Fargate Task

Rule:

Port 80
Source = ALB Security Group

Recommended:

ALB → ECS Tasks

Avoid:

Internet → ECS Tasks

unless specifically required.

IAM Roles

Containers often need access to AWS services.

Examples:

S3
Secrets Manager
CloudWatch
DynamoDB

Do not store AWS credentials inside containers.

Bad:

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY

Good:

Container
     │
     ▼
IAM Role
     │
     ▼
AWS Services
ECS Task Execution Role

Used by ECS itself.

Responsibilities:

Pull Images from ECR
Write Logs to CloudWatch

Architecture:

ECS
 │
 ▼
Execution Role
 │
 ▼
ECR
CloudWatch
ECS Task Role

Used by the application running inside the container.

Example:

Application
     │
     ▼
Task Role
     │
     ▼
S3 Bucket

The application receives temporary credentials automatically.

What Fargate Does NOT Manage

Many new engineers think Fargate manages everything.

It does not.

You still need:

VPC
Subnets
Route Tables
Security Groups
IAM Roles
Load Balancer
DNS
Application Configuration
Real Production Architecture
Internet
    │
    ▼
Application Load Balancer
    │
    ▼
Security Group
    │
    ▼
Fargate Task
    │
    ▼
Private Subnet
    │
    ▼
VPC

Image Source:

Amazon ECR
      │
      ▼
ECS Task Definition
      │
      ▼
Fargate Task
EC2 vs Fargate Summary
Component	ECS on EC2	ECS on Fargate
Manage EC2	Yes	No
Install Docker	Yes	No
Patch OS	Yes	No
Capacity Planning	Yes	No
Configure CPU	Yes	Yes
Configure Memory	Yes	Yes
Configure Networking	Yes	Yes
Configure IAM	Yes	Yes
Run Containers	Yes	Yes
Key Takeaways
AWS Fargate is a serverless compute platform for containers.
You do not manage EC2 servers or Docker hosts.
Every task requires CPU and Memory allocation.
Each Fargate task gets its own ENI and private IP.
Tasks run inside VPC subnets.
Security Groups control inbound and outbound traffic.
IAM Roles provide secure AWS service access.
Fargate manages infrastructure, but you must still configure VPC, subnets, security groups, IAM roles, and networking correctly.
Fargate is commonly used with ECS, ECR, Application Load Balancers, and private subnets in production environments.