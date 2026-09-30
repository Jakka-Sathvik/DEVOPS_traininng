ECS Controlled Blue Green Deployment
Objective

Perform a controlled deployment of the application from:

v1.0.0 → v1.1.0

The new version must be deployed to the Green environment, validated, and only then receive production traffic.

The deployment must minimize disruption and provide a clear rollback path.

1. Deployment Architecture
                         ALB
                          │
             ┌────────────┴────────────┐
             ▼                         ▼
       Blue Target Group        Green Target Group
             │                         │
             ▼                         ▼
        ECS v1.0.0                ECS v1.1.0
        Production                Validation
             │
             ▼
           Users
2. Initial Production State

Before deployment:

Blue = v1.0.0
Green = Not Production

Traffic flow:

Users
  │
  ▼
 ALB
  │
  ▼
Blue
  │
  ▼
ECS v1.0.0

Verify that the current production version is healthy before beginning.

3. Pre-Deployment Checklist

Before deploying v1.1.0, verify:

☑ Blue environment is healthy

☑ ECS service is running

☑ Current tasks are healthy

☑ ALB is healthy

☑ Target group is healthy

☑ /health returns HTTP 200

☑ Application is responding

☑ CloudWatch logs show no critical errors

☑ v1.1.0 image is available in ECR
4. Verify Current Version

Open:

ECS
 ↓
Cluster
 ↓
Service
 ↓
Tasks

Confirm that the running application uses:

v1.0.0

Example:

Task Definition:
devops-training-task:3

Container Image:
devops-training:v1.0.0
5. Verify Blue Target Group

Navigate to:

EC2
 ↓
Target Groups
 ↓
Blue Target Group

Check target status.

Expected:

Healthy

Example:

Targets
----------------
ECS Task 1    Healthy
ECS Task 2    Healthy
6. Verify ALB

Navigate to:

EC2
 ↓
Load Balancers
 ↓
Application Load Balancer

Verify:

ALB Status = Active

Check the listener:

HTTP : 80

Verify that the listener currently routes production traffic to Blue.

7. Prepare v1.1.0

Build the new application version.

Example:

v1.0.0
   ↓
Code Changes
   ↓
Build
   ↓
Docker Image
   ↓
v1.1.0

Push the image to Amazon ECR.

Example:

devops-training:v1.1.0
8. Create New Task Definition Revision

Create a new task definition revision containing:

Image:
devops-training:v1.1.0

Keep the required configuration:

Environment Variables
Secrets Manager References
CPU
Memory
Port
Health Check
IAM Roles

Example:

Revision 3 → v1.0.0
Revision 4 → v1.1.0
9. Deploy v1.1.0 to Green

The new application version should run in the Green environment.

Architecture:

Blue
v1.0.0
 │
 │ Production
 ▼
Users


Green
v1.1.0
 │
 │ Validation
 ▼
Test Requests

At this point:

Users → Blue
Test → Green

Do not shift production traffic yet.

10. Verify Green ECS Tasks

Navigate:

ECS
 ↓
Cluster
 ↓
Service / Deployment
 ↓
Tasks

Verify the Green tasks are:

RUNNING

Check:

Task Definition
Container Image
CPU
Memory
Port

Expected:

Image = v1.1.0
Status = RUNNING
11. Verify Green Target Health

Navigate:

EC2
 ↓
Target Groups
 ↓
Green Target Group

Verify:

Target Status = Healthy

Health check:

Protocol = HTTP
Port = traffic-port
Path = /health

Expected:

HTTP 200

Architecture:

Green ECS Task
      │
      ▼
ALB Health Check
      │
      ▼
/health
      │
      ▼
HTTP 200
      │
      ▼
Healthy
12. Validate Application

Test:

GET /

Expected:

HTTP 200

Then:

GET /health

Expected:

{
  "status": "healthy"
}
13. Validate Application APIs

Test important application APIs.

Examples:

GET /api/users
GET /api/products
GET /api/orders

Verify:

HTTP Status
Response Body
Application Behavior

Only test endpoints that actually exist in your application.

14. Validate Database Connectivity

The Green application must successfully communicate with its database.

Architecture:

Green ECS
    │
    ▼
Application
    │
    ▼
Database

Verify:

Database Connection = Successful

Check CloudWatch logs for connection errors.

15. Validate Secrets

Verify that the new task can access the required Secrets Manager values.

Architecture:

Green ECS Task
      │
      ▼
Task Execution Role
      │
      ▼
Secrets Manager
      │
      ▼
Required Secret

Do not print the actual secret value in application logs.

Verify only that the application starts and performs the operation requiring the secret successfully.

16. Validate Environment Variables

Check that required non-sensitive configuration exists.

Examples:

ENVIRONMENT=production
PORT=3000
APP_NAME=devops-training

Sensitive values should continue to come from Secrets Manager.

17. CloudWatch Log Validation

Navigate:

CloudWatch
 ↓
Log Groups
 ↓
ECS Application Logs

Review the Green deployment logs.

Look for:

Application Started
Server Listening
Database Connected
Health Check Successful

Check for:

ERROR
Exception
Connection Refused
Authentication Failed
Timeout

Do not expose passwords, tokens, or secret values in logs.

18. Green Environment Validation

The complete validation should look like:

                 GREEN v1.1.0
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
    Container        ALB          Application
     Healthy        Healthy         Healthy
        │              │              │
        └──────────────┼──────────────┘
                       ▼
                 Dependencies
                    Healthy

Validation checklist:

☑ Container healthy
☑ Target healthy
☑ ALB healthy
☑ GET / successful
☑ GET /health successful
☑ APIs successful
☑ Database connection successful
☑ Required secrets available
☑ Environment variables correct
☑ CloudWatch logs reviewed
19. Traffic Shift Decision

After Green validation:

All Checks Pass
       │
       ▼
Traffic Shift

If validation fails:

Validation Failed
       │
       ▼
Do NOT Shift Traffic
       │
       ▼
Investigate
20. Traffic Shift

Before:

Users
  │
  ▼
 ALB
  │
  ▼
Blue
v1.0.0

After:

Users
  │
  ▼
 ALB
  │
  ▼
Green
v1.1.0

The Green environment becomes the production environment.

21. Post-Deployment Verification

Immediately after the traffic shift, test the production endpoint.

Test:

GET /

Then:

GET /health

Expected:

HTTP 200

Application response should correspond to:

v1.1.0
22. Verify ALB After Traffic Shift

Check:

ALB
 ↓
Listener
 ↓
Target Group

Confirm that production traffic is directed to the Green target group.

Expected:

Production Traffic
        ↓
       ALB
        ↓
Green Target Group
        ↓
ECS v1.1.0
23. Verify ECS Service

Navigate:

ECS
 ↓
Cluster
 ↓
Service

Check:

Deployment Status
Running Count
Desired Count
Task Definition

Expected:

Running Tasks = Expected Count
Deployment = Successful
Version = v1.1.0
24. Verify Target Health

Check the Green target group again.

Expected:

Healthy Targets

No production target should unexpectedly show:

Unhealthy
Draining
25. Verify Application Response

Send requests through the production ALB.

Example:

GET /

Expected:

Application v1.1.0

Health:

GET /health

Expected:

{
  "status": "healthy"
}
26. Verify CloudWatch Logs

After traffic is shifted, monitor CloudWatch.

Look for:

Successful Requests
Normal Application Logs
No Unexpected Errors

Monitor for:

5xx Errors
Database Errors
Application Exceptions
Timeouts
Container Restarts
27. Complete Deployment Flow
Current Production
      │
      ▼
Blue = v1.0.0
      │
      ├──────────────────┐
      │                  │
      ▼                  ▼
    Users              Green
                       v1.1.0
                          │
                          ▼
                    Health Checks
                          │
                          ▼
                     Validation
                          │
                          ▼
                    Traffic Shift
                          │
                          ▼
                    Green = PROD
28. Deployment Timeline
Stage 1 — Current Production
Blue = v1.0.0

Users:

Users → ALB → Blue
Stage 2 — Deploy New Version
Green = v1.1.0

Users remain on:

Blue
Stage 3 — Validate
Green
 │
 ├── Container
 ├── ALB
 ├── Application
 ├── Database
 ├── Secrets
 └── CloudWatch
Stage 4 — Shift Traffic
ALB
 │
 ▼
Green
Stage 5 — Production
Green = v1.1.0
29. Rollback Preparation

Before the deployment, keep the Blue environment available until the new version has been validated in production.

If a problem occurs:

Green v1.1.0
       │
       ▼
Problem
       │
       ▼
Shift Traffic Back
       │
       ▼
Blue v1.0.0

This provides a clear rollback path.

30. Evidence to Capture

Capture screenshots showing:

☑ Blue v1.0.0 before deployment

☑ Green v1.1.0 running

☑ Green target health

☑ ALB configuration

☑ /health response

☑ Application response

☑ ECS service deployment

☑ CloudWatch logs

☑ Traffic routed to Green

☑ Final production version = v1.1.0
31. Success Criteria

The controlled deployment is successful when:

☑ v1.0.0 is initially serving production traffic

☑ v1.1.0 is available in ECR

☑ New task definition revision created

☑ v1.1.0 deployed to Green

☑ Green ECS tasks are RUNNING

☑ Green targets are HEALTHY

☑ /health returns HTTP 200

☑ Application APIs work

☑ Database connectivity works

☑ Required secrets are available

☑ CloudWatch logs are healthy

☑ Production traffic is shifted to Green

☑ Application responds correctly after traffic shift

☑ Blue v1.0.0 remains available for rollback
Final Architecture
                         Users
                           │
                           ▼
                          ALB
                           │
                           ▼
                  Green Target Group
                           │
                           ▼
                     ECS v1.1.0
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Secrets       Database      CloudWatch
          Manager       Connection       Logs

Previous production environment:

Blue
v1.0.0

New production environment:

Green
v1.1.0
Conclusion

A controlled Blue/Green deployment separates deployment from production traffic. Version v1.1.0 is first deployed to Green and validated through ECS task health, ALB target health, application endpoints, APIs, database connectivity, secrets, environment configuration, and CloudWatch logs. Only after these checks pass is traffic shifted from Blue v1.0.0 to Green v1.1.0.

The final state is:

Blue  = v1.0.0 → Previous Production / Rollback
Green = v1.1.0 → Current Production