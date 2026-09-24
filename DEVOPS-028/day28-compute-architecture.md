Day 28 — Compute Architecture Fundamentals
TASK 1 — Compute Architecture Fundamentals

Time: 45 Minutes

1. Objective

The objective of this task is to understand the basic AWS compute and load-balancing components used when building a highly available application architecture.

The main components covered are:

EC2
Launch Template
Auto Scaling Group
Load Balancer
Target Group
Listener
Health Check

The key concept is:

The Load Balancer distributes incoming requests, while the Auto Scaling Group manages the EC2 instances.

2. Basic Architecture

The application architecture can be represented as:

                         User
                           │
                           ▼
                    ┌─────────────┐
                    │     ALB     │
                    │Load Balancer│
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │Target Group │
                    └──────┬──────┘
                           │
                 ┌─────────┴─────────┐
                 ▼                   ▼
             ┌────────┐          ┌────────┐
             │ EC2-1  │          │ EC2-2  │
             │ Server │          │ Server │
             └────────┘          └────────┘

                    ▲
                    │
             Managed by ASG
                    │
                    ▼
             Auto Scaling Group

The Launch Template provides the configuration required to launch EC2 instances.

3. EC2
What is EC2?

Amazon EC2 (Elastic Compute Cloud) provides virtual servers in the AWS cloud.

An EC2 instance can run:

Web applications
APIs
Backend services
Docker containers
Application servers
Development environments

Example:

EC2 Instance
    │
    ├── Operating System
    ├── Application
    ├── Web Server
    └── Application Port
Responsibility

The EC2 instance is responsible for running the application workload.

For example:

EC2-1 → Application Server
EC2-2 → Application Server
Important EC2 Properties

Common properties include:

Instance ID
AMI
Instance type
Security Group
Subnet
Private IP
Public IP
IAM role
Storage

Example:

AMI: Amazon Linux
Instance Type: t3.micro
Subnet: Private Subnet
Security Group: Web-SG
Application Port: 80
4. Launch Template
What is a Launch Template?

A Launch Template defines how new EC2 instances should be created.

Instead of manually configuring every EC2 instance, we define the configuration once.

Example:

Launch Template
      │
      ├── AMI
      ├── Instance Type
      ├── Security Group
      ├── IAM Role
      ├── User Data
      └── Storage Configuration

The Auto Scaling Group can then use this template when launching instances.

Responsibility

The Launch Template is responsible for providing the configuration blueprint for EC2 instances.

For example:

Launch Template
       │
       ├── AMI = Amazon Linux
       ├── Type = t3.micro
       └── Security Group = Web-SG
              │
              ▼
        New EC2 Instance
Important Concept

A Launch Template does not manage the number of EC2 instances.

It only defines how an instance should be launched.

5. Auto Scaling Group
What is an Auto Scaling Group?

An Auto Scaling Group (ASG) manages a group of EC2 instances.

It can:

Launch EC2 instances
Maintain the desired number of instances
Replace unhealthy instances
Scale instances up
Scale instances down

Example:

                 Auto Scaling Group
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
          EC2-1                   EC2-2

Suppose the desired capacity is:

Desired = 2
Minimum = 2
Maximum = 4

The ASG attempts to maintain two healthy instances.

If one instance becomes unhealthy:

Before:

EC2-1    EC2-2
  │        │
  │        X
           Unhealthy


After:

EC2-1    EC2-3
  │        │
Healthy   New Instance

The ASG can replace the unhealthy instance.

Responsibility

The ASG is responsible for instance lifecycle and scaling.

It answers:

"How many EC2 instances should be running?"

6. Load Balancer
What is a Load Balancer?

A Load Balancer receives incoming client requests and distributes them across available backend servers.

For this architecture, we use an Application Load Balancer (ALB).

Example:

User
 │
 ▼
ALB
 │
 ├──────► EC2-1
 │
 └──────► EC2-2

Instead of users directly accessing EC2 instances, they access the Load Balancer.

Responsibility

The Load Balancer is responsible for:

Receiving client requests
Distributing requests
Routing traffic
Checking target health through the target group
Providing a stable application endpoint

Example:

User Request
     │
     ▼
    ALB
     │
     ├── Request 1 ──► EC2-1
     ├── Request 2 ──► EC2-2
     ├── Request 3 ──► EC2-1
     └── Request 4 ──► EC2-2
Important Concept

The Load Balancer does not primarily manage the number of EC2 instances.

It manages traffic distribution.

7. Target Group
What is a Target Group?

A Target Group contains the backend targets that receive traffic from the Load Balancer.

The targets can include EC2 instances and other supported target types.

Example:

ALB
 │
 ▼
Target Group
 │
 ├── EC2-1
 └── EC2-2
Responsibility

The Target Group defines:

Which targets receive traffic
Target port
Protocol
Health check configuration

Example:

Target Group
    │
    ├── Protocol: HTTP
    ├── Port: 80
    ├── EC2-1
    └── EC2-2
Target Registration

When EC2 instances are registered with the target group:

Target Group
     │
     ├── EC2-1 → Healthy
     └── EC2-2 → Healthy

The ALB can send traffic to healthy targets.

8. Listener
What is a Listener?

A Listener checks for incoming connection requests on a specified protocol and port.

For example:

HTTP → Port 80
HTTPS → Port 443

Example:

User
 │
 │ HTTP Request
 ▼
ALB
 │
 ▼
Listener : 80
 │
 ▼
Target Group
Responsibility

The Listener determines:

"What should the Load Balancer do when a request arrives?"

For example:

HTTP : 80
     │
     ▼
Forward request
     │
     ▼
Target Group

An HTTPS listener may also use rules to route traffic based on conditions such as:

Host name
URL path
HTTP headers

Example:

example.com/api
       │
       ▼
API Target Group

and:

example.com/web
       │
       ▼
Web Target Group
9. Health Check
What is a Health Check?

A health check determines whether a target is healthy and able to receive traffic.

For example:

Health Check
     │
     ▼
GET /health
     │
     ▼
EC2 Application

If the application responds successfully:

EC2-1 → Healthy

If the application fails:

EC2-2 → Unhealthy

The Load Balancer can stop sending normal traffic to unhealthy targets.

Example

Suppose the application provides:

http://EC2/health

Expected response:

HTTP 200 OK

The target can be considered healthy.

If the application returns:

HTTP 500

or fails to respond, the target may become unhealthy.

Responsibility

Health checks help determine:

"Can this backend instance currently receive traffic?"

10. Complete Component Relationship

The components work together as follows:

                    User
                      │
                      ▼
             ┌─────────────────┐
             │       ALB       │
             │ Application     │
             │ Load Balancer   │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │    Listener     │
             │ HTTP : 80       │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │  Target Group   │
             └────────┬────────┘
                      │
              ┌───────┴────────┐
              ▼                ▼
          ┌────────┐       ┌────────┐
          │ EC2-1  │       │ EC2-2  │
          └────────┘       └────────┘
              ▲                ▲
              │                │
              └───────┬────────┘
                      │
             Auto Scaling Group
                      ▲
                      │
              Launch Template
11. How the Components Work Together
Step 1 — User Sends Request

The user accesses the application:

http://example.com

The request reaches the ALB.

Step 2 — ALB Receives Request

The ALB receives the request.

User
 │
 ▼
ALB
Step 3 — Listener Receives Connection

The listener listens on the configured port.

Example:

HTTP : 80
Step 4 — Listener Forwards Request

The listener forwards the request to the configured target group.

Listener
    │
    ▼
Target Group
Step 5 — Target Group Selects Healthy Target

The target group contains EC2 instances.

Target Group
    │
    ├── EC2-1 → Healthy
    └── EC2-2 → Healthy

The ALB distributes requests among healthy targets.

Step 6 — EC2 Processes Request

The selected EC2 instance runs the application.

ALB
 │
 ▼
EC2-1
 │
 ▼
Application
Step 7 — Auto Scaling Group Manages Instances

The ASG ensures that the required number of instances are available.

For example:

Minimum = 2
Desired = 2
Maximum = 4

If additional capacity is required, the ASG can launch additional instances using the Launch Template.

Launch Template
       │
       ▼
Auto Scaling Group
       │
       ├── EC2-1
       ├── EC2-2
       └── EC2-3
12. Load Balancer vs Auto Scaling Group

This is one of the most important concepts in this architecture.

Component	Main Responsibility
Load Balancer	Distributes incoming traffic
Auto Scaling Group	Manages EC2 instances
Launch Template	Defines EC2 launch configuration
Target Group	Defines backend targets
Listener	Receives connections and applies routing rules
Health Check	Determines target health
EC2	Runs the application
Simple Rule
Load Balancer
      ↓
"Where should this request go?"
Auto Scaling Group
      ↓
"How many EC2 instances should be running?"
13. Example Scenario

Assume an online shopping application.

Initially:

Users
  │
  ▼
 ALB
  │
  ▼
Target Group
  │
  ├── EC2-1
  └── EC2-2

There are two EC2 instances.

Now traffic increases.

Users
Users
Users
Users
Users

The Auto Scaling Group can increase the number of instances:

Before:

EC2-1
EC2-2


After:

EC2-1
EC2-2
EC2-3
EC2-4

The new instances are created using the Launch Template.

The target group can then register the new instances, and the ALB can distribute traffic to healthy targets.

14. Failure Scenario

Suppose EC2-1 becomes unhealthy.

Target Group

EC2-1 → Unhealthy
EC2-2 → Healthy

The Load Balancer stops sending normal traffic to the unhealthy target.

The Auto Scaling Group can detect the unhealthy instance and replace it, depending on the configured health-check behavior.

EC2-1 → Unhealthy
          │
          ▼
     ASG replaces it
          │
          ▼
       EC2-3

The new instance is configured according to the Launch Template.

After it becomes healthy:

Target Group

EC2-2 → Healthy
EC2-3 → Healthy

The ALB can distribute traffic again.

15. Key Architecture Flow

Remember this flow:

User
 │
 ▼
ALB
 │
 ▼
Listener
 │
 ▼
Target Group
 │
 ├── EC2-1
 └── EC2-2

And the EC2 management flow:

Launch Template
       │
       ▼
Auto Scaling Group
       │
       ├── EC2-1
       └── EC2-2

Health monitoring:

Target
  │
  ▼
Health Check
  │
  ├── Healthy
  └── Unhealthy
16. Important Differences
EC2 vs Launch Template
EC2
↓
Actual running server
Launch Template
↓
Blueprint used to create servers
Launch Template vs Auto Scaling Group
Launch Template
↓
Defines HOW to launch an instance
Auto Scaling Group
↓
Defines HOW MANY instances should run
Load Balancer vs Auto Scaling Group
Load Balancer
↓
Manages traffic distribution
Auto Scaling Group
↓
Manages EC2 instance capacity
Target Group vs Load Balancer
Load Balancer
↓
Receives and routes traffic
Target Group
↓
Defines the backend targets
Listener vs Health Check
Listener
↓
Handles incoming connections
Health Check
↓
Checks whether backend targets are healthy
17. Terraform Mapping

These AWS components commonly map to Terraform resources as follows:

AWS Component	Terraform Resource
EC2	aws_instance
Launch Template	aws_launch_template
Auto Scaling Group	aws_autoscaling_group
Application Load Balancer	aws_lb
Target Group	aws_lb_target_group
Listener	aws_lb_listener
Security Group	aws_security_group

A typical Terraform architecture may therefore contain:

VPC
 │
 ├── Public Subnets
 │      │
 │      └── ALB
 │
 └── Private Subnets
        │
        └── Auto Scaling Group
                │
                ├── EC2
                └── EC2

Launch Template
       │
       ▼
Auto Scaling Group

ALB
 │
 ▼
Target Group
 │
 ▼
EC2 Instances
18. Final Summary

The complete architecture can be remembered using the following model:

                 USER
                   │
                   ▼
                  ALB
                   │
                   ▼
               LISTENER
                   │
                   ▼
              TARGET GROUP
               │         │
               ▼         ▼
             EC2-1     EC2-2
               ▲         ▲
               │         │
               └────┬────┘
                    │
                   ASG
                    ▲
                    │
             LAUNCH TEMPLATE
Responsibilities

EC2

Runs the application.

Launch Template

Defines how EC2 instances should be launched.

Auto Scaling Group

Maintains and scales the required number of EC2 instances.

Load Balancer

Distributes incoming application traffic.

Target Group

Contains the backend targets that receive traffic.

Listener

Receives connections on a configured port and forwards requests according to its rules.

Health Check

Determines whether backend targets are healthy enough to receive traffic.

Most Important Concept
                    TRAFFIC
                       │
                       ▼
               LOAD BALANCER
                       │
                       ▼
                TARGET GROUP
                       │
                ┌──────┴──────┐
                ▼             ▼
              EC2-1         EC2-2
                ▲             ▲
                └──────┬──────┘
                       │
                      ASG
                       │
                INSTANCE CAPACITY

             LAUNCH TEMPLATE
                    │
                    ▼
             INSTANCE CONFIG

Remember:

ALB distributes traffic. ASG manages EC2 capacity. Launch Template defines how EC2 instances are created. Target Group connects the ALB to backend instances. Listener controls incoming traffic, and Health Checks determine whether targets are healthy.