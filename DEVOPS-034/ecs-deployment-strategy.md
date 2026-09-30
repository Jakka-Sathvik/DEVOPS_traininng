ECS Deployment Strategies
Objective

Understand how Amazon ECS deploys new application versions and learn the differences between:

Rolling Deployment
Blue/Green Deployment

These deployment strategies help reduce downtime, improve reliability, and support safer releases.

Why Deployment Strategy Matters

Consider a running application:

Application
     ↓
Version 1.0.0

A new version is released:

Version 1.1.0

Without a deployment strategy:

Stop Old Version
      ↓
Start New Version

Problems:

Downtime
Failed deployments
Service interruptions
Difficult recovery

Deployment strategies solve these problems.

Deployment Architecture

Typical ECS deployment flow:

Developer
     ↓
Docker Image
     ↓
Amazon ECR
     ↓
Task Definition Revision
     ↓
ECS Service
     ↓
Deployment
Rolling Deployment
What Is Rolling Deployment?

Rolling deployment gradually replaces old tasks with new tasks.

Architecture:

Old Tasks
    ↓
New Tasks
    ↓
Health Check
    ↓
Old Tasks Removed

The service remains available during deployment.

Rolling Deployment Example

Current state:

v1
v1

Deployment starts:

v1
v2

After health checks succeed:

v2
v2

Deployment complete.

Rolling Deployment Process

Step 1

Current environment:

Task 1 = v1
Task 2 = v1

Step 2

ECS launches a new task:

Task 1 = v1
Task 2 = v1
Task 3 = v2

Step 3

ALB health check runs:

/health

Expected:

HTTP 200

Step 4

New task becomes healthy:

v2 Healthy

Step 5

Old task removed:

Task 1 = v1
Task 3 = v2

Step 6

Repeat until deployment finishes:

v2
v2
Rolling Deployment Visualization
Phase 1

v1  v1

Phase 2

v1  v2

Phase 3

v2  v2
Rolling Deployment Advantages

Benefits:

No Downtime
Simple Configuration
Fast Deployment
Native ECS Support
Rolling Deployment Limitations

Potential challenges:

Old And New Versions Run Together
Database Compatibility Must Be Managed
Partial Deployment State Exists
ECS Rolling Deployment Settings

Important parameters:

Minimum Healthy Percent

Example:

100%

Meaning:

Keep All Existing Tasks Healthy

during deployment.

Maximum Percent

Example:

200%

Meaning:

Allow Temporary Extra Tasks

during deployment.

Example Rolling Deployment

Current:

Desired Tasks = 2

Configuration:

Minimum Healthy = 100%
Maximum = 200%

Deployment:

2 Tasks
 ↓
4 Tasks Allowed
 ↓
Health Checks
 ↓
Old Tasks Removed
Blue/Green Deployment
What Is Blue/Green?

Blue/Green deployment creates two complete environments.

Architecture:

BLUE
v1
 │
 │ Live
 ▼
ALB
 │
 ▼
Users


GREEN
v2
 │
 │ Test
 ▼
ALB
Blue Environment

Current production:

BLUE
Version 1

Users access:

Users
  ↓
ALB
  ↓
BLUE
Green Environment

New release:

GREEN
Version 2

Environment exists separately:

Users
   X
GREEN

No production traffic yet.

Blue/Green Testing

Before release:

GREEN Environment
       ↓
Functional Testing
       ↓
Health Checks
       ↓
Validation

Example tests:

/health
/login
/api
/database
Traffic Shift

After validation:

BLUE → OLD
GREEN → NEW

Traffic changes:

Users
  ↓
ALB
  ↓
GREEN
Blue/Green Visualization

Before deployment:

Users
  ↓
BLUE (v1)

After deployment:

Users
  ↓
GREEN (v2)
Blue/Green Rollback

Suppose:

GREEN = Failure

Rollback:

Users
  ↓
BLUE (v1)

Traffic returns immediately.

Benefits:

Fast Recovery
Low Risk
Simple Rollback
Blue/Green Advantages

Benefits:

Near-Zero Downtime
Production Validation
Easy Rollback
Safer Releases
Blue/Green Challenges

Trade-offs:

Higher Cost
Duplicate Infrastructure
More Complex Setup

Because both environments exist simultaneously.

Rolling vs Blue/Green
Feature	Rolling Deployment	Blue/Green Deployment
Infrastructure	One Environment	Two Environments
Cost	Lower	Higher
Complexity	Lower	Higher
Rollback Speed	Moderate	Very Fast
Resource Usage	Lower	Higher
Production Validation	Limited	Extensive
Deployment Risk	Medium	Lower
Downtime	Very Low	Near Zero
ECS Deployment Controller

Amazon ECS supports:

Rolling Deployment

natively through ECS Service updates.

For Blue/Green deployments, AWS commonly uses:

ECS
   ↓
CodeDeploy
   ↓
Blue/Green Deployment

Architecture:

CodeDeploy
      ↓
Traffic Shift
      ↓
Blue → Green
Health Checks During Deployment

Health checks protect deployments.

Example:

/health

Expected:

HTTP 200

Deployment flow:

Deploy New Version
        ↓
Health Check
        ↓
Healthy?
      /     \
    Yes      No
     │        │
     ▼        ▼
 Continue   Stop Deployment
Production Recommendation

Typical usage:

Small Applications
Rolling Deployment

Advantages:

Simpler
Lower cost
Faster setup
Critical Production Systems
Blue/Green Deployment

Advantages:

Safer releases
Better validation
Faster rollback
Deployment Timeline Example

Rolling:

v1 v1
 ↓
v1 v2
 ↓
v2 v2

Blue/Green:

BLUE v1
     │
     ▼
 Users

GREEN v2
     │
Validation
     ▼

Traffic Shift

BLUE → OLD
GREEN → NEW
Success Criteria

The trainee should be able to explain:

What is Rolling Deployment?

What is Blue/Green Deployment?

How ECS Performs Rolling Deployments

How CodeDeploy Enables Blue/Green

How Health Checks Protect Deployments

How Rollback Works

And understand:

v1 v1
 ↓
v1 v2
 ↓
v2 v2

as well as:

BLUE → OLD
GREEN → NEW

traffic shifting.

Conclusion

Deployment strategies are critical for safe production releases. Rolling deployments gradually replace old tasks with new tasks while maintaining service availability and are the default ECS deployment method. Blue/Green deployments maintain separate production environments, allowing complete validation before shifting traffic, providing safer releases and faster rollback capabilities at the cost of additional infrastructure.