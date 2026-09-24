TASK 9 — Safe Deployment Strategy

Training: DevOps Engineer
Topic: Safe Deployment Flow
Time: 45 Minutes
File: Safe_Deployment_Strategy.md

1. Objective

The objective of this task is to understand how to deploy a new application version safely.

A deployment should not be considered successful immediately after the Docker image is deployed. The application must be:

Built successfully
Tested
Published as a Docker image
Deployed
Health checked
Version checked
Monitored after deployment

If the new release fails the health check, the deployment process should stop the failed release and restore the previous known-good version.

2. Safe Deployment Architecture

The deployment process is:

New Code
   ↓
Tests
   ↓
Docker Build
   ↓
Push Image
   ↓
Deploy
   ↓
Wait
   ↓
Health Check
   ↓
Version Check
   ↓
SUCCESS

If the health check fails:

Deploy
   ↓
Health Check
   ↓
FAIL
   ↓
Stop Failed Release
   ↓
Restore Previous Version
   ↓
Health Check
   ↓
SUCCESS
3. Why Safe Deployment Is Important

A deployment changes the running application.

If the new version contains a bug, configuration problem, dependency problem, or runtime failure, users may experience:

Application errors
HTTP 500 responses
Failed requests
Incorrect application behavior
Container crashes
Increased downtime
Loss of availability

A safe deployment process reduces the chance that a bad release remains in production.

The basic principle is:

Deploy
  ↓
Verify
  ↓
Continue if Healthy
  ↓
Rollback if Unhealthy
4. Step 1 — New Code

The process starts when developers create new application code.

Example:

Developer
    ↓
New Code
    ↓
Git Repository

Example change:

@app.route("/health")
def health():
    return {
        "status": "healthy"
    }, 200

The code should be committed to the appropriate Git branch according to the team's workflow.

Example:

feature → develop → main
5. Step 2 — Run Tests

Before creating a production image, tests should be executed.

Example:

New Code
   ↓
Unit Tests
   ↓
Integration Tests
   ↓
Application Validation

Possible checks include:

Unit tests
API tests
Health endpoint tests
Configuration validation
Dependency checks
Dockerfile validation

Example health test:

curl -i http://localhost:5000/health

Expected:

HTTP/1.1 200 OK

Response:

{
  "status": "healthy"
}

If tests fail:

Tests
  ↓
FAIL
  ↓
STOP DEPLOYMENT

The failed code should not continue to deployment.

6. Step 3 — Build Docker Image

After tests pass, build the Docker image.

Example:

docker build -t devops-final:v1.1.0 .

The version tag identifies the release.

For example:

devops-final:v1.0.0
devops-final:v1.1.0
devops-final:v1.2.0

Avoid relying only on:

latest

for production release identification.

Versioned tags make it easier to identify and restore a previous release.

7. Step 4 — Push Docker Image

After the image is successfully built and validated, push it to the approved container registry.

Architecture:

Developer / CI
      ↓
Docker Build
      ↓
Docker Image
      ↓
Container Registry

Example:

devops-final:v1.1.0

The registry provides a known location from which the deployment environment can obtain the release image.

8. Step 5 — Deploy

The deployment system starts the new application version.

Example:

Registry
   ↓
Pull Image
   ↓
Start Container
   ↓
Application

Example:

devops-final:v1.1.0

The deployment system should know which version was previously running.

Example:

Previous Version = v1.0.0
New Version      = v1.1.0

This information is important if rollback becomes necessary.

9. Step 6 — Wait for Application Startup

The deployment should not immediately declare success.

The application may need time to:

Start the process
Load dependencies
Initialize configuration
Connect to required services
Start listening on the application port

Therefore:

Deploy
  ↓
Wait for Startup
  ↓
Health Check

The waiting period should be appropriate for the application rather than an arbitrary long delay.

10. Step 7 — Health Check

After the new version starts, perform a health check.

For the training application:

GET /health

Example:

curl -i http://localhost:5000/health

Expected:

HTTP/1.1 200 OK

Response:

{
  "status": "healthy"
}

The health endpoint should remain simple.

It must not expose:

Passwords
Access tokens
API keys
Database credentials
Private keys
Internal secrets
11. Health Check Decision

The deployment system evaluates the result.

Healthy
Health Check
     ↓
HTTP 200
     ↓
Healthy Response
     ↓
Continue
Failed
Health Check
     ↓
Failure
     ↓
STOP FAILED RELEASE
     ↓
ROLLBACK

A health check failure should prevent the failed release from being accepted as a successful deployment.

12. Step 8 — Version Check

A health check confirms that the application is responding.

A version check confirms that the expected release is actually running.

For example:

Expected Version
      ↓
v1.1.0

The running application should report:

Running Version
      ↓
v1.1.0

Then:

Expected Version = Running Version
              ↓
             YES
              ↓
           Continue

A version endpoint could return information such as:

{
  "version": "v1.1.0"
}

Only expose information that is appropriate for the application's security requirements.

13. Successful Deployment

The complete successful flow is:

New Code
   ↓
Tests
   ↓
PASS
   ↓
Docker Build
   ↓
Image Created
   ↓
Push Image
   ↓
Deploy
   ↓
Wait
   ↓
Health Check
   ↓
PASS
   ↓
Version Check
   ↓
PASS
   ↓
SUCCESS

The release can now be considered successfully deployed.

14. Failed Deployment

If the new release fails:

Deploy
   ↓
Health Check
   ↓
FAIL
   ↓
Stop Failed Release
   ↓
Restore Previous Version
   ↓
Health Check
   ↓
PASS
   ↓
SUCCESS

The goal is to restore service using the previous known-good version.

15. Rollback Strategy

Assume the production application is currently running:

v1.0.0

A new version is released:

v1.1.0

The deployment starts:

v1.0.0
   ↓
Deploy v1.1.0

The health check fails:

v1.1.0
   ↓
Health Check
   ↓
FAIL

The deployment should stop accepting the failed version and restore:

v1.1.0
   ↓
FAILED
   ↓
Rollback
   ↓
v1.0.0

Then verify:

v1.0.0
   ↓
Health Check
   ↓
200 OK
   ↓
Service Restored
16. Complete Rollback Architecture
                 New Code
                    │
                    ↓
                  Tests
                    │
                    ↓
              Docker Build
                    │
                    ↓
               Push Image
                    │
                    ↓
                 Deploy
                    │
                    ↓
                  Wait
                    │
                    ↓
              Health Check
               ┌────┴────┐
               │         │
              PASS       FAIL
               │         │
               ↓         ↓
          Version      Stop Failed
           Check        Release
               │         │
               ↓         ↓
            SUCCESS   Restore Previous
                          Version
                             │
                             ↓
                       Health Check
                        ┌────┴────┐
                        │         │
                       PASS      FAIL
                        │         │
                        ↓         ↓
                   Service     Escalate /
                   Restored    Investigate
17. Example Docker Deployment

Suppose the current image is:

devops-final:v1.0.0

The new image is:

devops-final:v1.1.0

Deployment process:

v1.0.0
  ↓
Build v1.1.0
  ↓
Test v1.1.0
  ↓
Push v1.1.0
  ↓
Deploy v1.1.0
  ↓
Health Check

If successful:

v1.1.0
  ↓
Healthy
  ↓
Version Verified
  ↓
Deployment Successful

If failed:

v1.1.0
  ↓
Health Check Failed
  ↓
Stop v1.1.0
  ↓
Restore v1.0.0
  ↓
Health Check
  ↓
Healthy
18. Important Rollback Principle

Rollback should restore a known-good version.

For example:

Current:
v1.0.0

Failed:
v1.1.0

Rollback:
v1.0.0

Do not blindly deploy an unknown image.

The deployment system should keep track of:

Current Version
Previous Version
Image Tag
Deployment Time
Health Status

This makes troubleshooting easier.

19. Health Check vs Version Check
Check	Purpose
Health Check	Confirms application is responding correctly at a basic level
Version Check	Confirms expected release is running
Both	Provides stronger deployment verification

Example:

Health Check
    ↓
Is application healthy?

Then:

Version Check
    ↓
Is the expected release running?

Both checks provide better deployment confidence.

20. Deployment Failure Scenarios

A deployment may fail because of:

Application Error
Application starts
      ↓
Runtime error
      ↓
Health Check FAIL
Incorrect Configuration
New Version
    ↓
Wrong Environment Variable
    ↓
Application Failure
Dependency Problem
New Image
   ↓
Missing/Incorrect Dependency
   ↓
Application Failure
Port Problem
Application
    ↓
Expected Port Not Available
    ↓
Health Check FAIL
Database Dependency Problem

If the application genuinely requires a database during startup, an unavailable or incompatible database can prevent the application from becoming healthy.

The deployment system should distinguish between application health and deeper dependency diagnostics rather than exposing sensitive information through the health endpoint.

21. Deployment Safety Rules

Follow these rules:

Test before deployment.
Build an identifiable Docker image.
Use meaningful version tags.
Push only approved images to the registry.
Keep track of the previous known-good version.
Wait for application startup.
Run health checks.
Verify the deployed version.
Stop failed releases.
Roll back when required.
Re-check health after rollback.
Investigate the root cause after service restoration.
22. CI/CD Deployment Flow

A CI/CD pipeline can automate the process:

Developer
    ↓
Git Push
    ↓
CI/CD Pipeline
    ↓
Tests
    ↓
Docker Build
    ↓
Security / Validation Checks
    ↓
Push Image
    ↓
Deploy
    ↓
Health Check
    ↓
Version Check
    ↓
 ┌───────────────┐
 │               │
 PASS            FAIL
 │               │
 ↓               ↓
SUCCESS       ROLLBACK
                 ↓
          Health Check
                 ↓
              RESTORED

This reduces manual deployment errors.

23. Observability During Deployment

Deployment verification should not rely only on one health endpoint.

DevOps engineers can also monitor:

Application logs
Container status
Error rates
CPU usage
Memory usage
Request latency
HTTP response codes
Application metrics

Example:

Deployment
    │
    ├── Health Check
    ├── Version Check
    ├── Logs
    ├── Metrics
    └── Error Monitoring

A deployment can technically return HTTP 200 while still having increased error rates or performance problems, so monitoring provides additional context.

24. Practical Exercise
Step 1 — Identify Current Version

Record:

Current Version: v1.0.0
Step 2 — Prepare New Version

Create:

v1.1.0
Step 3 — Run Tests

Verify:

Tests → PASS
Step 4 — Build Image

Build the new Docker image using the approved repository/tagging convention.

Step 5 — Push Image

Push the image to the approved registry.

Step 6 — Deploy

Deploy:

v1.1.0
Step 7 — Wait

Allow the application to start.

Step 8 — Health Check

Test:

curl -i http://localhost:5000/health

Expected:

200 OK
Step 9 — Version Check

Confirm:

Expected: v1.1.0
Running:  v1.1.0
Step 10 — Record Result

Example:

Deployment Version: v1.1.0
Health Check: PASS
Version Check: PASS
Deployment Status: SUCCESS
25. Failure Simulation

For training purposes, a controlled failure can be simulated safely.

Example:

Deploy New Version
       ↓
Health Check
       ↓
FAIL
       ↓
Rollback
       ↓
Previous Version
       ↓
Health Check
       ↓
PASS

Record:

New Version: v1.1.0
Health Check: FAIL
Rollback: v1.0.0
Rollback Health Check: PASS
Final Status: RESTORED

Do not perform failure simulations against an actual production system without authorization.