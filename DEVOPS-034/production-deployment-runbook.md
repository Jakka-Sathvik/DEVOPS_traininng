Production Deployment Runbook
ECS Production Deployment

Purpose:
Provide a repeatable and controlled procedure for deploying a new application version to Amazon ECS while minimizing deployment risk and ensuring that rollback is possible.

1. Deployment Overview

The production deployment flow is:

Developer
    │
    ▼
Source Code
    │
    ▼
Docker Build
    │
    ▼
Image Scan
    │
    ▼
Amazon ECR
    │
    ▼
ECS Task Definition
    │
    ▼
ECS Deployment
    │
    ▼
ALB
    │
    ▼
Health Checks
    │
    ▼
Application Validation
    │
    ▼
Production

For a Blue/Green deployment:

                    ALB
                     │
             ┌───────┴───────┐
             ▼               ▼
           BLUE            GREEN
         Current          New Version
        Production         Testing
             │               │
             │          Health Checks
             │               │
             │          Validation
             │               │
             │          ┌──────┴──────┐
             │          │             │
             │        PASS           FAIL
             │          │             │
             │     Traffic Shift    Stop
             │          │
             └──────────┴───► GREEN = Production
2. Deployment Information

Complete the following before starting the deployment.

Item	Value
AWS Region	ap-south-1
ECS Cluster	<cluster-name>
ECS Service	<service-name>
ECR Repository	<repository-name>
New Image Tag	<image-tag>
Previous Image Tag	<previous-image-tag>
New Task Definition	<new-revision>
Previous Task Definition	<previous-revision>
ALB	<alb-name>
Target Group	<target-group-name>
Health Check Path	/health
Deployment Type	Blue/Green or Rolling
Deployment Date	<date>
Deployment Owner	<name>

Important: Replace placeholders with the actual values from the environment. Do not invent or hard-code AWS resource IDs.

3. Before Deployment

The deployment must not begin until the pre-deployment checks are complete.

Pre-Deployment Checklist
☐ Image available in ECR
☐ Image scanned
☐ Task definition created
☐ Secrets available
☐ IAM permissions verified
☐ Health check verified
☐ ALB healthy
4. Check Image Availability in ECR

Confirm that the new Docker image exists in Amazon ECR.

Expected:

ECR
 │
 └── Repository
       │
       └── New Image
             │
             └── <new-version>

Example:

devops-training:v1.1.0

Check:

AWS Console
   ↓
ECR
   ↓
Repositories
   ↓
Application Repository
   ↓
Images

Verify:

Repository exists.
Image exists.
Correct tag is present.
Image digest is available.
Image was pushed successfully.
Checklist
☐ ECR repository exists
☐ New image exists
☐ Correct image tag confirmed
☐ Image digest confirmed
5. Image Scanning

The container image should be scanned for known vulnerabilities before deployment.

Architecture:

Docker Image
     │
     ▼
Amazon ECR
     │
     ▼
Image Scan
     │
     ├── Vulnerabilities
     └── Security Findings

Review the scan results.

Pay particular attention to:

CRITICAL
HIGH
MEDIUM
LOW

Follow the organization's vulnerability policy before continuing.

Checklist
☐ Image scan completed
☐ Findings reviewed
☐ Critical findings addressed or approved
☐ High-severity findings reviewed
6. Create Task Definition Revision

Create a new ECS task-definition revision for the new application image.

Example:

Current:

devops-training-task:9
        ↓
v1.0.0

New:

devops-training-task:10
        ↓
v1.1.0

Verify:

Container Image
CPU
Memory
Container Port
Environment Variables
Secrets
Logging Configuration
IAM Configuration
Health Check

The new task definition should reference the intended ECR image.

Example:

Image:
<account>.dkr.ecr.ap-south-1.amazonaws.com/<repository>:v1.1.0
Checklist
☐ New task definition created
☐ Correct image configured
☐ CPU verified
☐ Memory verified
☐ Container port verified
☐ Environment variables verified
☐ Secrets verified
☐ Logging configured
☐ Health check configured
7. Verify Secrets

Confirm that all required application secrets are available.

Example:

ECS Task
   │
   ▼
Secrets Manager
   │
   ▼
Application Secret

Verify:

Secret exists
Secret ARN is correct
Secret value is available
ECS task has permission to retrieve it

Do not place sensitive values directly into:

Dockerfile
Git repository
GitHub workflow
Plain-text configuration
Checklist
☐ Required secrets exist
☐ Secret references are correct
☐ Secret ARN verified
☐ Required IAM access exists
☐ No secrets hard-coded in image/configuration
8. Verify IAM Permissions

Confirm that the ECS task has only the permissions it requires.

For Secrets Manager, the application should have access to the specific secret it needs.

Example:

Application
     │
     ▼
Required Secret
     │
     ▼
secretsmanager:GetSecretValue

Avoid unnecessarily broad permissions such as:

Resource: "*"

when access can be restricted to a specific resource.

Also verify the difference between:

Task Execution Role

and:

Task Role

Use the appropriate role for the required operation.

Checklist
☐ Task execution role verified
☐ Task role verified
☐ Secrets permissions verified
☐ Resource scope reviewed
☐ No unnecessary AdministratorAccess
9. Verify Health Check

Confirm that the application health endpoint works.

Expected endpoint:

/health

Expected response:

{
  "status": "healthy"
}

Expected status:

HTTP 200 OK

Architecture:

ALB
 │
 ▼
/health
 │
 ▼
Application
 │
 ▼
HTTP 200
 │
 ▼
Healthy
Checklist
☐ Health endpoint exists
☐ Health endpoint tested
☐ HTTP 200 confirmed
☐ Expected response confirmed
10. Verify ALB Health

Before deployment, the existing production targets must be healthy.

Open:

EC2
 ↓
Load Balancers
 ↓
Target Groups
 ↓
Targets

Verify:

Target Status = Healthy

Expected:

ALB
 ↓
Target Group
 ↓
Healthy ECS Tasks

Do not begin a production deployment while the existing production environment has unexplained unhealthy targets.

Checklist
☐ ALB available
☐ Listener verified
☐ Target group verified
☐ Existing targets healthy
☐ Health checks passing
11. Pre-Deployment Approval

Before proceeding, review the complete checklist.

PRE-DEPLOYMENT

☐ Image available in ECR
☐ Image scanned
☐ Task definition created
☐ Secrets available
☐ IAM permissions verified
☐ Health check verified
☐ ALB healthy

If a critical check fails:

STOP DEPLOYMENT

Investigate and correct the issue before continuing.

12. During Deployment

Once all pre-deployment checks pass:

☐ Deploy new revision
☐ Monitor ECS
☐ Monitor CloudWatch
☐ Check ALB
☐ Check application
13. Deploy New Revision

Update the ECS service to use the new task-definition revision.

Example:

Old:

devops-training-task:9
       ↓
v1.0.0

Deploy:

devops-training-task:10
       ↓
v1.1.0

For Blue/Green deployment:

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
Health Checks

Do not shift traffic until the new version has passed validation.

Checklist
☐ New revision deployed
☐ Deployment started successfully
☐ Green environment created/updated
☐ Traffic shift not performed prematurely
14. Monitor ECS

Open:

ECS
 ↓
Cluster
 ↓
Service
 ↓
Deployments

Monitor:

Desired Tasks
Running Tasks
Pending Tasks
Stopped Tasks
Task Health
Deployment Status

Expected:

New Tasks
    ↓
Starting
    ↓
Running
    ↓
Healthy

Investigate immediately if tasks repeatedly stop or fail.

Checklist
☐ ECS deployment active
☐ Tasks starting
☐ Tasks running
☐ Tasks healthy
☐ No unexpected task failures
15. Monitor CloudWatch

Check application logs during deployment.

Architecture:

ECS Container
     │
     ▼
CloudWatch Logs
     │
     ▼
Monitoring

Look for:

Application startup
Connection errors
Exceptions
Authentication errors
Database errors
Secret errors
HTTP 5xx errors

Normal example:

Application started
Server listening on port 8080
Database connection successful

Problem example:

ERROR: Database connection failed
Checklist
☐ Logs available
☐ Application startup successful
☐ No repeated critical errors
☐ No unexpected exceptions
☐ Dependencies connected successfully
16. Check ALB

Verify the new targets through the target group.

Check:

Target Status
Health Check
Registered Port
Availability Zone

Expected:

New ECS Task
      │
      ▼
ALB Target Group
      │
      ▼
/health
      │
      ▼
HTTP 200
      │
      ▼
Healthy

If targets remain unhealthy:

STOP
   ↓
Investigate
   ↓
Do Not Shift Traffic
17. Check Application

Test the application through the appropriate endpoint.

Health Check
GET /health

Expected:

HTTP 200

Response:

{
  "status": "healthy"
}
Main Application
GET /

Verify that the application responds correctly.

API Endpoints

If the application provides APIs, test important endpoints.

Example:

GET /api/...
POST /api/...

Use the actual endpoints available in the application.

18. Green Environment Validation

Before traffic is shifted:

Green
 │
 ├── ECS Task Healthy
 │
 ├── ALB Target Healthy
 │
 ├── /health = 200
 │
 ├── Application Working
 │
 ├── Database Connected
 │
 ├── Secrets Available
 │
 └── Logs Normal

Only after successful validation should the deployment proceed to the traffic-shift stage.

19. Traffic Shift

For a Blue/Green deployment:

Before:

Users
  │
  ▼
ALB
  │
  ▼
BLUE v1.0.0


GREEN v1.1.0
  │
  └── Validation

After successful validation:

Users
  │
  ▼
ALB
  │
  ▼
GREEN v1.1.0

Green becomes the production version after the configured traffic shift.

Blue should remain available long enough to support rollback according to the deployment strategy.

20. After Deployment

Complete the post-deployment checklist:

☐ All tasks healthy
☐ Application responding
☐ Logs clean
☐ No critical errors
☐ Previous version available for rollback
21. Verify All Tasks Are Healthy

Open:

ECS
 ↓
Cluster
 ↓
Service
 ↓
Tasks

Confirm:

Desired Count = Running Count

Example:

Desired: 2
Running: 2
Healthy: 2

No unexpected stopped or unhealthy tasks should remain.

22. Verify Application Response

Test:

GET /

and:

GET /health

Expected:

/health
   ↓
HTTP 200

Response:

{
  "status": "healthy"
}

If APIs are available, test the critical application paths.

23. Verify Logs

Check CloudWatch after deployment.

Confirm:

No critical application errors
No repeated exceptions
No authentication failures
No database connection failures
No secret retrieval errors

A small number of expected informational messages is normal.

Focus on errors that affect application functionality.

24. Verify Production Target Health

The final architecture should be:

Users
  │
  ▼
ALB
  │
  ▼
Production Target Group
  │
  ├── ECS Task
  │      └── Healthy
  │
  └── ECS Task
         └── Healthy

All production targets should pass their configured health checks.

25. Confirm Previous Version Is Available

Rollback readiness is part of deployment completion.

Verify that the previous stable version remains available.

Example:

Previous:
v1.0.0

Current:
v1.1.0

And the previous task-definition revision is known:

devops-training-task:9

This allows the team to quickly return to the previous version if a problem appears after deployment.

Checklist
☐ Previous image still available
☐ Previous task definition still available
☐ Previous revision identified
☐ Rollback procedure known
26. Rollback Decision

If serious problems are detected:

Production
    ↓
Problem
    ↓
Investigate
    ↓
Rollback Required

Rollback:

v1.1.0
   ↓
Rollback
   ↓
v1.0.0
   ↓
Health Check
   ↓
Healthy

Verify:

ALB
 ↓
Healthy Target
 ↓
v1.0.0
 ↓
Application Working

Refer to:

docs/ecs-rollback.md

for the detailed rollback procedure.

27. Emergency Stop Conditions

Stop the deployment if any of the following occur:

☐ Critical vulnerability discovered
☐ ECS tasks repeatedly fail
☐ ALB targets remain unhealthy
☐ Application returns unexpected 5xx errors
☐ Database connectivity fails
☐ Required secret cannot be retrieved
☐ Critical application functionality fails
☐ Unexpected infrastructure behavior occurs

The deployment should remain stopped until the issue is understood.

28. Deployment Evidence

Record evidence for every production deployment.

Recommended evidence:

ECR image/tag
Task definition revision
ECR scan result
ECS deployment status
ECS task health
ALB target health
/health response
Application response
CloudWatch logs
Production version
Previous rollback version

Example deployment record:

Deployment:
v1.0.0 → v1.1.0

ECR:
<repository>:v1.1.0

Task Definition:
<task-definition>:<revision>

Health Check:
/health → HTTP 200

ECS:
All tasks healthy

ALB:
All production targets healthy

Application:
Working

Rollback Version:
v1.0.0
29. Final Production Checklist
Before Deployment
☐ Image available in ECR
☐ Image scanned
☐ Task definition created
☐ Secrets available
☐ IAM permissions verified
☐ Health check verified
☐ ALB healthy
During Deployment
☐ Deploy new revision
☐ Monitor ECS
☐ Monitor CloudWatch
☐ Check ALB
☐ Check application
After Deployment
☐ All tasks healthy
☐ Application responding
☐ Logs clean
☐ No critical errors
☐ Previous version available for rollback
30. Complete Deployment Flow
                    SOURCE CODE
                         │
                         ▼
                   DOCKER BUILD
                         │
                         ▼
                    IMAGE SCAN
                         │
                         ▼
                       ECR
                         │
                         ▼
                TASK DEFINITION
                         │
                         ▼
                  ECS DEPLOYMENT
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
            BLUE                  GREEN
          Current               New Version
       Production                Testing
              │                     │
              │                Health Check
              │                     │
              │                Application
              │                 Validation
              │                     │
              │              ┌──────┴──────┐
              │              ▼             ▼
              │            PASS          FAIL
              │              │             │
              │        Traffic Shift      STOP
              │              │
              └──────────────┤
                             ▼
                         Production
                             │
                             ▼
                       Monitor ECS
                             │
                             ▼
                      Monitor CloudWatch
                             │
                             ▼
                         Check ALB
                             │
                             ▼
                    Check Application
                             │
                             ▼
                    Deployment Complete
31. Deployment Completion Criteria

The deployment is considered complete when:

☐ New version deployed successfully
☐ ECS tasks are healthy
☐ ALB targets are healthy
☐ /health returns HTTP 200
☐ Application endpoints work
☐ CloudWatch logs show no critical errors
☐ Required secrets are available
☐ IAM permissions work correctly
☐ Production traffic is reaching the intended version
☐ Previous stable version remains available
☐ Rollback procedure is ready
32. Deployment Sign-Off

Record the final deployment status.

Field	Result
Deployment Version	<version>
Previous Version	<version>
ECS Service	<service>
Task Definition	<revision>
ALB	<ALB>
Health Check	PASS / FAIL
Target Health	PASS / FAIL
Application Test	PASS / FAIL
CloudWatch Logs	PASS / FAIL
Rollback Available	YES / NO
Deployment Status	SUCCESS / STOPPED / ROLLED BACK
Notes	<deployment notes>
Key Principle

A production deployment should follow:

CHECK
  ↓
DEPLOY
  ↓
MONITOR
  ↓
VALIDATE
  ↓
SHIFT TRAFFIC
  ↓
VERIFY
  ↓
KEEP ROLLBACK AVAILABLE

The objective is not simply to start a new ECS task.

The objective is to prove that the new application version is healthy, correctly configured, observable, reachable through the ALB, and recoverable if a problem occurs.