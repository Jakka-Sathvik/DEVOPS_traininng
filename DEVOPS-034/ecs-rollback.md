Objective

Simulate a production problem in v1.1.0, identify the failure, roll back to the previously stable v1.0.0, and verify that production is healthy again.

Rollback Flow
Production
   ↓
v1.1.0
   ↓
Problem Detected
   ↓
Rollback
   ↓
v1.0.0
   ↓
Health Check
   ↓
Application Working
1. Failure

Assume the application is currently running:

Production
    ↓
v1.1.0

After deployment, simulate a production problem.

Possible failures:

Wrong environment variable
Invalid application configuration
Broken API
Database connection problem
Invalid secret reference
Application code error

For this exercise, assume:

v1.1.0
   ↓
Application Problem

Example:

GET /health
     ↓
HTTP 500

Expected healthy response:

{
  "status": "healthy"
}

Expected HTTP status:

200 OK

The important point is that v1.1.0 is no longer considered reliable.

2. Detection

The problem can be detected using multiple AWS components.

2.1 Check the Application

Test:

GET /health

Expected healthy response:

{
  "status": "healthy"
}

If the deployment has failed:

HTTP 500

or another unsuccessful response may be returned.

2.2 Check ALB Target Health

Open:

AWS Console
   ↓
EC2
   ↓
Target Groups
   ↓
Production Target Group
   ↓
Targets

Check the target status.

Possible result:

v1.1.0
   ↓
Unhealthy

The ALB health check provides an important indication that the application is not functioning correctly.

2.3 Check ECS Service

Open:

AWS Console
   ↓
ECS
   ↓
Cluster
   ↓
Service

Check:

Running Tasks
Task Definition
Deployment
Task Health

Confirm that production is running:

v1.1.0
2.4 Check CloudWatch Logs

Open:

CloudWatch
   ↓
Log Groups
   ↓
ECS Application Logs

Look for:

ERROR
Exception
Configuration error
Database error
Application error

Example:

ERROR: Application failed to connect to database

The actual error will depend on the failure simulated during the exercise.

3. Rollback Procedure

The objective is:

v1.1.0
   ↓
Rollback
   ↓
v1.0.0
Step 1 — Confirm the Problem

Before rolling back, confirm:

Application → Problem
ALB → Target unhealthy
ECS → v1.1.0
CloudWatch → Error
Step 2 — Identify the Previous Stable Version

Find the ECS task-definition revision associated with v1.0.0.

Example:

devops-training-task:9
        ↓
v1.0.0

The revision number is only an example.

Use the actual revision from your ECS environment.

Verify:

Task Definition
      ↓
Container Image
      ↓
v1.0.0
Step 3 — Update the ECS Service

Open:

ECS
 ↓
Cluster
 ↓
Service
 ↓
Update Service

Select the previous stable task-definition revision.

For example:

Current:

Task Definition
     ↓
v1.1.0

Change to:

Task Definition
     ↓
v1.0.0

Then deploy/update the service.

Step 4 — ECS Starts New Tasks

ECS will start tasks using the previous stable version.

v1.1.0
   ↓
Rollback
   ↓
Task Definition for v1.0.0
   ↓
New ECS Tasks

Monitor the service until the desired number of healthy tasks is running.

Step 5 — Monitor Task Status

Open:

ECS
 ↓
Cluster
 ↓
Service
 ↓
Tasks

Check:

Task Status
Container Status
Health Status
Task Definition

Expected:

Task
 ↓
RUNNING
 ↓
v1.0.0
Step 6 — Monitor ALB Targets

Open:

EC2
 ↓
Target Groups
 ↓
Targets

Expected:

v1.0.0
   ↓
Health Check
   ↓
Healthy
4. Verification

Rollback is not complete just because the ECS service changed versions.

Verify the entire application path.

ALB
 ↓
Healthy Target
 ↓
ECS Task
 ↓
v1.0.0
 ↓
Application
4.1 Verify ALB

Open the ALB and confirm:

Listener
   ↓
Production Target Group

is functioning correctly.

4.2 Verify Target Health

Target status should be:

Healthy

Expected:

ALB
 ↓
Target
 ↓
Healthy
4.3 Verify Old Version

Confirm that the running ECS task uses:

v1.0.0

Check:

ECS Task
   ↓
Task Definition
   ↓
Container
   ↓
Image
   ↓
v1.0.0

Do not assume the rollback succeeded solely from the service status.

4.4 Verify /health

Send:

GET /health

Expected:

HTTP 200

Response:

{
  "status": "healthy"
}
4.5 Verify Application

Test:

GET /

Expected:

Application working

Also test important application APIs if available.

4.6 Verify CloudWatch Logs

Confirm that the v1.0.0 tasks are producing normal application logs.

Check for:

Application started
Server listening
Successful requests
No repeated application errors
5. Root Cause

After restoring production, identify why v1.1.0 failed.

Use the following format:

Root Cause:

The v1.1.0 release introduced
<actual problem>.

The problem caused
<actual impact>.

The issue was detected through
<health check / ALB / ECS / CloudWatch>.
Example
Root Cause:

v1.1.0 contained an incorrect environment
configuration value.

The incorrect configuration caused the
application dependency to fail.

The problem was detected through the
application health check and CloudWatch logs.

The ECS service was rolled back to the
previous stable v1.0.0 task definition.
Important

Do not document only:

Deployment failed.

That is the symptom.

The root cause should explain:

What changed?
       ↓
Why did it cause the problem?
       ↓
What component was affected?
6. Prevention

Rollback restores the previous working version, but prevention reduces the possibility of repeating the same failure.

6.1 Automated Testing

Use:

Code
 ↓
Build
 ↓
Unit Tests
 ↓
Docker Build
 ↓
Container Tests
 ↓
Integration Tests
 ↓
Deployment
6.2 Health Checks

Configure an application health endpoint:

/health

Expected:

HTTP 200

Architecture:

ALB
 ↓
/health
 ↓
HTTP 200
 ↓
Healthy
6.3 Green Environment Validation

Before production traffic is shifted:

New Version
    ↓
Green
    ↓
Health Check
    ↓
API Test
    ↓
Database Test
    ↓
Secret Test
    ↓
Functional Test
    ↓
Traffic Shift

A release should not be considered ready simply because its container started.

6.4 Configuration Validation

Validate:

Environment Variables
Secrets
Container Port
Application Configuration
Database Configuration

before deployment.

6.5 Monitoring

Use CloudWatch to monitor:

CPU
Memory
ALB 4xx
ALB 5xx
Application Errors
Task Health
Latency

Monitoring helps detect problems after deployment.

6.6 Maintain Previous Versions

Keep the previous known-good image and task-definition revision available.

Example:

v1.0.0 ← Previous Stable
v1.1.0 ← Failed Release
v1.1.1 ← Corrected Release

This makes rollback faster.

Complete Rollback Flow
                 PRODUCTION
                     │
                     ▼
                   v1.1.0
                     │
                     ▼
              Problem Detected
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
         ALB        ECS      CloudWatch
       Health      Status       Logs
          │          │          │
          └──────────┼──────────┘
                     ▼
                Confirm Failure
                     │
                     ▼
                  ROLLBACK
                     │
                     ▼
            Previous Task Definition
                     │
                     ▼
                   v1.0.0
                     │
                     ▼
                 ECS Tasks
                     │
                     ▼
                ALB Health Check
                     │
                     ▼
                  Healthy
                     │
                     ▼
              Application Working
Rollback Verification Checklist

Production was running v1.1.0.

A production problem was detected.

The failure was confirmed using appropriate monitoring.

ALB target health was checked.

ECS task status was checked.

CloudWatch logs were inspected.

The previous stable task-definition revision was identified.

ECS service was updated to the previous revision.

v1.0.0 tasks became RUNNING.

ALB target became Healthy.

/health returned HTTP 200.

The main application endpoint worked.

The running container image was confirmed as v1.0.0.

Root cause was documented.

Prevention measures were documented.

Final Trainee Explanation

The trainee should be able to explain the exercise:

Production was running v1.1.0 when an application
problem was detected.

The problem was identified using application health
checks, ALB target health, ECS status, and CloudWatch
logs.

The previous stable ECS task-definition revision
running v1.0.0 was identified and deployed to the
service.

After the rollback, the ECS tasks were verified,
the ALB target became healthy, /health returned HTTP
200, and the application was confirmed to be working.

The root cause of the v1.1.0 failure was documented,
along with preventive measures such as automated
testing, health checks, monitoring, and Green
environment validation.
Key Takeaway
v1.1.0
   ↓
Problem
   ↓
Detect
   ↓
Rollback
   ↓
v1.0.0
   ↓
ALB Healthy
   ↓
Application Healthy

A rollback provides a controlled path back to a previously known-good application version when a new production release causes a problem.