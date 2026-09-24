TASK 11 — Create Application Load Balancer

This task connects your ECS Fargate service to the Internet through an Application Load Balancer (ALB).

The final architecture will be:

                         Internet
                            │
                            ▼
                    ┌───────────────┐
                    │      ALB      │
                    │ Public Subnets│
                    └───────┬───────┘
                            │
                       Listener :80
                            │
                            ▼
                    ┌───────────────┐
                    │ Target Group  │
                    │   HTTP :8000  │
                    └───────┬───────┘
                            │
                 ┌──────────┴──────────┐
                 ▼                     ▼
          ┌─────────────┐       ┌─────────────┐
          │ ECS Task 1  │       │ ECS Task 2  │
          │   :8000     │       │   :8000     │
          └─────────────┘       └─────────────┘
                 │                     │
                 └──────────┬──────────┘
                            ▼
                       ECS Fargate
1. What is an Application Load Balancer?

An Application Load Balancer (ALB) is an AWS service that receives HTTP/HTTPS traffic and forwards it to healthy application targets.

Instead of users accessing ECS tasks directly:

Internet
   │
   ├──► ECS Task 1
   ├──► ECS Task 2
   └──► ECS Task 3

we use:

Internet
   │
   ▼
  ALB
   │
   ▼
Target Group
   │
   ├──► Task 1
   ├──► Task 2
   └──► Task 3

The ALB provides:

Traffic distribution
Health checking
High availability
HTTP/HTTPS routing
Integration with ECS
A stable public endpoint
2. Understand the Four Components

Before creating anything, understand the relationship between these components.

ALB
 │
 │ Listener
 ▼
Target Group
 │
 │ Registered Targets
 ▼
ECS Tasks
ALB

The ALB is the front door.

Example:

http://my-alb-123456.ap-south-1.elb.amazonaws.com
Listener

A listener waits for incoming traffic.

For example:

ALB
 │
 └── Listener :80

This means:

When HTTP traffic arrives on port 80, process it according to the listener rules.

Target Group

The target group tells the ALB where to send traffic.

For ECS:

Target Group
Protocol: HTTP
Port: 8000
Health Check: /health

The target group contains the ECS task IP addresses.

ECS Service

The ECS service connects the running Fargate tasks to the target group.

ECS Service
     │
     ▼
Target Group
     │
     ├── Task 1
     └── Task 2

This integration is important because ECS automatically manages task registration/deregistration as tasks start and stop.

3. Prerequisites

Before starting Task 11, you should have:

ECS cluster
devops-training-cluster
ECS service
devops-training-service
ECS task definition
devops-training-task
VPC

Example:

10.0.0.0/16
Public subnets

For the ALB:

Public Subnet 1
Public Subnet 2
Private subnets

For Fargate:

Private Subnet 1
Private Subnet 2
Security groups

Ideally:

ALB Security Group
        │
        ▼
ECS Security Group
4. Recommended Architecture

For this training project, use:

                         Internet
                            │
                            │ HTTP :80
                            ▼
                  ┌──────────────────┐
                  │       ALB        │
                  │  Public Subnets  │
                  └────────┬─────────┘
                           │
                           │ HTTP :8000
                           ▼
                  ┌──────────────────┐
                  │   Target Group   │
                  │      :8000       │
                  └────────┬─────────┘
                           │
               ┌───────────┴───────────┐
               ▼                       ▼
        ┌──────────────┐        ┌──────────────┐
        │ Fargate Task │        │ Fargate Task │
        │   Private    │        │   Private    │
        │    :8000     │        │    :8000     │
        └──────────────┘        └──────────────┘

Notice the important difference:

ALB → PUBLIC
ECS → PRIVATE

You don't need to expose every ECS task directly to the Internet.

5. Step 1 — Create ALB Security Group

Go to:

AWS Console
   ↓
EC2
   ↓
Security Groups
   ↓
Create security group

Name:

devops-alb-sg

Description:

Security group for DevOps training ALB

Select your VPC.

Inbound Rules

Add:

Type	Protocol	Port	Source
HTTP	TCP	80	0.0.0.0/0

For a training HTTP application, this allows Internet users to reach the ALB.

You can later use HTTPS:

TCP 443

with an ACM certificate.

Outbound Rules

For training, the default outbound rule can remain enabled.

6. Step 2 — Configure ECS Security Group

Your ECS task security group should not allow Internet traffic directly.

Bad design:

ECS SG

TCP 8000
Source: 0.0.0.0/0

Better:

ALB SG
   │
   │ TCP 8000
   ▼
ECS SG

Add this inbound rule to the ECS security group:

Type	Protocol	Port	Source
Custom TCP	TCP	8000	devops-alb-sg

The important part is:

Source = ALB Security Group

not:

0.0.0.0/0

This means only the ALB can send traffic to port 8000 on the ECS tasks.

7. Step 3 — Create Target Group

Go to:

EC2
   ↓
Load Balancing
   ↓
Target Groups
   ↓
Create target group

Choose:

Target type:
IP addresses
Why IP addresses?

Fargate tasks using:

networkMode = awsvpc

have their own network interface and IP address.

Therefore:

ALB
 │
 ▼
Target Group
 │
 ▼
Task private IP

For ECS Fargate, use IP targets rather than instance targets.

8. Target Group Configuration

Example:

Target group name:

devops-training-tg

Protocol:

HTTP

Port:

8000

IP address type:

IPv4

Select your VPC.

9. Configure Health Check

Set:

Health check protocol:
HTTP

Path:

/health

Port:

Traffic port

So the ALB will effectively check:

http://TASK-IP:8000/health

Expected response:

HTTP 200 OK
10. Why Health Checks Matter

Suppose you have two tasks:

Task 1 → Healthy
Task 2 → Unhealthy

The ALB can route traffic to:

Task 1

and stop sending normal traffic to:

Task 2

until it becomes healthy.

Conceptually:

                ALB
                 │
          ┌──────┴──────┐
          ▼             ▼
       Task 1         Task 2
      HEALTHY       UNHEALTHY
          │
          ▼
       Traffic

This is one of the major benefits of using an ALB.

11. Important Application Requirement

Your application must listen on:

0.0.0.0:8000

and not only:

127.0.0.1:8000

Why?

Inside the container:

127.0.0.1

means:

Only this container itself.

But:

0.0.0.0

means:

Listen on all network interfaces.

For example, a Python application might use:

app.run(host="0.0.0.0", port=8000)
12. Step 4 — Create the ALB

Go to:

EC2
   ↓
Load Balancers
   ↓
Create Load Balancer

Select:

Application Load Balancer

Click:

Create
13. Basic Configuration

Load balancer name:

devops-training-alb

Scheme:

Internet-facing

IP address type:

IPv4

Why Internet-facing?

Because users need to access the application from the Internet.

14. Network Mapping

Select your VPC.

For example:

VPC
10.0.0.0/16

Select two public subnets in different Availability Zones.

Example:

ap-south-1a
   │
   └── Public Subnet 1

ap-south-1b
   │
   └── Public Subnet 2

Architecture:

                 Internet
                     │
                     ▼
              ┌─────────────┐
              │     ALB     │
              └──────┬──────┘
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
     Public AZ-1           Public AZ-2

Using multiple Availability Zones improves availability.

15. Select ALB Security Group

Choose:

devops-alb-sg

The important inbound rule is:

HTTP :80
Source: 0.0.0.0/0
16. Configure Listener

Create listener:

Protocol: HTTP
Port: 80

Default action:

Forward to
devops-training-tg

So the complete flow becomes:

Internet
   │
   │ HTTP :80
   ▼
ALB
   │
   │ Forward
   ▼
Target Group
   │
   │ HTTP :8000
   ▼
ECS Task
17. Create the ALB

Review:

Load balancer name
VPC
Availability Zones
Security Group
Listener
Target Group

Then click:

Create load balancer

AWS will provision the ALB.

Wait until its state becomes:

Active
18. Step 5 — Attach ECS Service to Target Group

This is a very important part.

The ECS service needs to know:

"When my tasks start, register them in this target group."

Go to:

ECS
   ↓
Clusters
   ↓
devops-training-cluster
   ↓
Services
   ↓
devops-training-service
   ↓
Update

Find:

Load balancing

Choose:

Application Load Balancer

Select:

Load balancer:
devops-training-alb

Then configure:

Listener:
HTTP :80

Target group:

devops-training-tg

Container:

devops-app

Container port:

8000

The relationship becomes:

ALB
 │
 └── HTTP :80
       │
       ▼
Target Group
 │
 └── HTTP :8000
       │
       ├── devops-app Task 1
       └── devops-app Task 2
19. Why ECS Needs the Container Name and Port

Suppose your task definition contains:

Container name:
devops-app

Container port:
8000

ECS needs to know:

ALB
  ↓
Target Group
  ↓
Which container?
  ↓
devops-app
  ↓
Which port?
  ↓
8000

Without the correct container/port mapping, the ALB cannot correctly route traffic to your application.

20. Update Desired Count

For testing, you can use:

Desired count = 2

Then ECS attempts to maintain:

Task 1
Task 2

Architecture:

                    ALB
                     │
                     ▼
               Target Group
                /         \
               /           \
              ▼             ▼
          Task 1          Task 2
          :8000           :8000

The ALB distributes requests between healthy targets.

21. Verify ECS Tasks

Go to:

ECS
 ↓
Cluster
 ↓
devops-training-cluster
 ↓
Services
 ↓
devops-training-service

Check:

Desired tasks: 2
Running tasks: 2

You want:

Desired = 2
Running = 2
22. Verify Target Group

Go to:

EC2
 ↓
Target Groups
 ↓
devops-training-tg
 ↓
Targets

You should eventually see something like:

Target                         Status
------------------------------------------------
10.0.x.x:8000                  Healthy
10.0.x.x:8000                  Healthy

The exact IP addresses will be your Fargate task private IPs.

23. If Targets Say "Unhealthy"

Don't immediately recreate everything.

Troubleshoot systematically.

Check 1 — Is the ECS task running?
ECS
 ↓
Service
 ↓
Tasks

Status should be:

RUNNING
Check 2 — Is port 8000 correct?

Task definition:

Container port = 8000

Target group:

Port = 8000

Application:

Listening = 8000

All three need to agree.

Task Definition
      │
      ├── 8000
      │
      ▼
Target Group
      │
      ├── 8000
      │
      ▼
Application
      │
      └── 8000
Check 3 — Check health endpoint

Open your application's health endpoint through the ALB:

/health

It should return:

HTTP 200
Check 4 — Check ECS Security Group

You need:

ECS SG
Inbound TCP 8000
Source = ALB SG

Not:

Source = 0.0.0.0/0
Check 5 — Check ALB Security Group

ALB needs:

Inbound TCP 80
Source = 0.0.0.0/0

for public HTTP access.

Check 6 — Check Application Binding

Make sure the application listens on:

0.0.0.0:8000

rather than:

127.0.0.1:8000
24. Test the ALB

Go to:

EC2
 ↓
Load Balancers
 ↓
devops-training-alb

Find:

DNS name

It will look approximately like:

devops-training-alb-123456789.ap-south-1.elb.amazonaws.com

Open:

http://ALB-DNS-NAME

For example:

http://devops-training-alb-123456789.ap-south-1.elb.amazonaws.com

The request should follow:

Browser
   │
   ▼
Internet
   │
   ▼
ALB :80
   │
   ▼
Target Group :8000
   │
   ▼
Fargate Task
   │
   ▼
Application
25. Test the Health Endpoint

If your application has:

GET /health

test:

http://ALB-DNS-NAME/health

Expected:

HTTP 200

For example:

{
  "status": "healthy"
}
26. Understand Traffic Distribution

Suppose you have:

Task 1
10.0.11.20:8000

Task 2
10.0.12.30:8000

The target group knows:

Targets
├── 10.0.11.20:8000
└── 10.0.12.30:8000

Traffic enters:

Client
  │
  ▼
ALB
  │
  ▼
Target Group
  │
  ├──────► Task 1
  │
  └──────► Task 2

The ALB only uses healthy targets for normal routing.

27. Complete Architecture for Task 11
                         INTERNET
                            │
                            │ HTTP :80
                            ▼
              ┌────────────────────────┐
              │  Application Load      │
              │      Balancer          │
              │                        │
              │  Public Subnet AZ-1    │
              │  Public Subnet AZ-2    │
              └───────────┬────────────┘
                          │
                          │ Listener :80
                          ▼
              ┌────────────────────────┐
              │      Target Group      │
              │                        │
              │ HTTP :8000             │
              │ Health: /health        │
              └───────────┬────────────┘
                          │
             ┌────────────┴────────────┐
             │                         │
             ▼                         ▼
      ┌──────────────┐          ┌──────────────┐
      │ ECS Fargate  │          │ ECS Fargate  │
      │    Task 1    │          │    Task 2    │
      │   :8000      │          │   :8000      │
      │ Private      │          │ Private      │
      └──────────────┘          └──────────────┘
             │                         │
             └────────────┬────────────┘
                          ▼
                     ECS Service
28. Security Group Architecture

Keep this relationship in mind:

             Internet
                 │
                 │ TCP 80
                 ▼
          ┌─────────────┐
          │   ALB SG    │
          └──────┬──────┘
                 │
                 │ TCP 8000
                 │ Source = ALB SG
                 ▼
          ┌─────────────┐
          │   ECS SG    │
          └──────┬──────┘
                 │
                 ▼
             Fargate

This is better than:

Internet
   │
   │ TCP 8000
   ▼
Fargate

because the application tasks don't need to be directly exposed.

29. CLI Alternative

If you want to practice the AWS CLI, first identify your resources:

aws ec2 describe-vpcs

Find your subnets:

aws ec2 describe-subnets

Find security groups:

aws ec2 describe-security-groups

Create the target group:

aws elbv2 create-target-group \
  --name devops-training-tg \
  --protocol HTTP \
  --port 8000 \
  --target-type ip \
  --vpc-id <VPC_ID> \
  --health-check-path /health

The command returns a target group ARN.

Create ALB
aws elbv2 create-load-balancer \
  --name devops-training-alb \
  --subnets <PUBLIC_SUBNET_1> <PUBLIC_SUBNET_2> \
  --security-groups <ALB_SECURITY_GROUP_ID> \
  --scheme internet-facing \
  --type application
Create Listener

After getting the ALB ARN:

aws elbv2 create-listener \
  --load-balancer-arn <ALB_ARN> \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=<TARGET_GROUP_ARN>

Verify:

aws elbv2 describe-load-balancers \
  --names devops-training-alb

Check target group:

aws elbv2 describe-target-groups \
  --names devops-training-tg

Check target health:

aws elbv2 describe-target-health \
  --target-group-arn <TARGET_GROUP_ARN>
30. Important Fargate Difference

With traditional EC2-based ECS:

ALB
 │
 ▼
EC2 Instance
 │
 ▼
Container

With Fargate:

ALB
 │
 ▼
Target Group
 │
 ▼
Task ENI / Private IP
 │
 ▼
Container

This is why your target group uses:

Target type = IP

for this Fargate architecture.

31. Final Verification Checklist

Before marking TASK 11 complete, verify:

☐ ALB created

☐ ALB is Internet-facing

☐ ALB uses two public subnets

☐ ALB security group created

☐ HTTP port 80 allowed to ALB

☐ Target group created

☐ Target type = IP

☐ Target group port = 8000

☐ Health check = /health

☐ ECS service attached to target group

☐ ECS tasks are RUNNING

☐ ECS SG allows port 8000 from ALB SG

☐ Targets show HEALTHY

☐ ALB DNS name obtained

☐ Application accessible through ALB

☐ /health returns HTTP 200
Final expected result
Browser
   │
   │ http://ALB-DNS
   ▼
┌─────────────────┐
│       ALB       │
│      :80        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Target Group   │
│      :8000      │
│   /health       │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
 Task 1     Task 2
 :8000      :8000
    │         │
    └────┬────┘
         ▼
    ECS Fargate

Deliverable: ALB successfully created, connected to the ECS service through an IP-based target group, with healthy Fargate targets and HTTP traffic flowing through p