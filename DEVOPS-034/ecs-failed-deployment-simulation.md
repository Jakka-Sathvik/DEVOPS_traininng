Simulate Failed Deployment

Time: 35 Minutes

1. Objective

The purpose of this exercise is to deliberately introduce a failure into the Green ECS environment and verify that the deployment process detects the failure before production traffic is shifted.

The expected behavior is:

Blue = Production v1.0.0
Green = New Version v1.1.0

If Green fails validation:

Green
  ↓
Deployment
  ↓
Health Check
  ↓
FAIL
  ↓
Traffic NOT Shifted
  ↓
Blue Remains Production

This demonstrates an important production deployment principle:

A new application version should not receive production traffic until it has passed the required health and validation checks.

2. Current Architecture

Before introducing the failure, the environment should look like this:

                         ALB
                          │
                 ┌────────┴────────┐
                 │                 │
                 ▼                 ▼
          Blue Target Group   Green Target Group
                 │                 │
                 ▼                 ▼
            ECS v1.0.0         ECS v1.1.0
                 │                 │
                 │                 ▼
                 │            Validation
                 │                 │
                 │              FAIL
                 │
                 ▼
             Production

Blue is the current production environment.

Green is the new release being tested.

3. Failure Injection

Choose one controlled failure.

Possible failures include:

Wrong environment variable
Invalid application configuration
Broken application endpoint
Incorrect container port
Invalid secret reference

For this exercise, use a broken /health endpoint because it clearly demonstrates how an unhealthy deployment can be detected.

4. Expected Healthy Application

Before introducing the failure, confirm that the existing application works.

The health endpoint should return:

GET /health

Expected response:

{
  "status": "healthy"
}

Expected HTTP status:

200 OK

Architecture:

User
 │
 ▼
ALB
 │
 ▼
ECS Task
 │
 ▼
/health
 │
 ▼
HTTP 200
 │
 ▼
Healthy
5. Create the Deliberate Failure

There are several ways to simulate failure.

For this training exercise, modify the new application version so that /health does not return the expected successful response.

For example, the new version could return:

HTTP 500

with:

{
  "status": "unhealthy"
}

The important point is that only the Green release should contain the failure.

Do not intentionally break the currently running Blue production version.

6. Example Failed Application

For example, the new application version might contain:

/health

but return:

{
  "status": "unhealthy"
}

Instead of:

{
  "status": "healthy"
}

The deployment now looks like:

Blue
v1.0.0
 │
 └── /health → 200 OK


Green
v1.1.0
 │
 └── /health → 500 ERROR

This creates a controlled failure.

7. Build the Failed Docker Image

Build the deliberately broken version using a new tag.

Example:

docker build -t devops-training:v1.1.0-failed .

Verify the image:

docker images

You should see something similar to:

devops-training    v1.1.0-failed
8. Test the Failure Locally

Before deploying the failed image to ECS, test it locally.

Run:

docker run -p 8080:8080 devops-training:v1.1.0-failed

The port must match the port used by your application.

Then test:

curl http://localhost:8080/health

Expected failure example:

HTTP 500

or:

{
  "status": "unhealthy"
}

This confirms that the failure was intentionally introduced.

9. Push Failed Image to ECR

Tag the image for your ECR repository.

Example:

docker tag devops-training:v1.1.0-failed \
<ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/<REPOSITORY>:v1.1.0-failed

Authenticate Docker with ECR:

aws ecr get-login-password --region ap-south-1 \
| docker login \
--username AWS \
--password-stdin \
<ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com

Push the image:

docker push \
<ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/<REPOSITORY>:v1.1.0-failed

Verify the image exists in ECR.

10. Create a New Task Definition Revision

Create a new ECS task definition revision.

Example:

devops-training-task:10

The exact revision number depends on the revisions already created in your environment.

Change the container image:

v1.1.0

to:

v1.1.0-failed

Do not modify the production Blue task definition.

Architecture:

Task Definition Revision
          │
          ▼
Failed Image
v1.1.0-failed
          │
          ▼
       Green
11. Deploy the Failed Version to Green

Deploy the new task definition to the Green environment.

The important rule is:

Blue = Production
Green = Test Deployment

Do not immediately shift production traffic.

Expected architecture:

                         ALB
                          │
             ┌────────────┴────────────┐
             │                         │
             ▼                         ▼
       Blue Target Group         Green Target Group
             │                         │
             ▼                         ▼
        v1.0.0 Production        v1.1.0 Failed
                                       │
                                       ▼
                                   /health
                                       │
                                       ▼
                                      FAIL
12. Observe ECS

Open:

AWS Console
    ↓
ECS
    ↓
Cluster
    ↓
Service
    ↓
Tasks

Inspect the Green tasks.

Check:

Task Status
Task Definition
Container Status
Stopped Reason
Health Status

Possible outcomes include:

UNHEALTHY

or:

STOPPED

depending on the failure and ECS configuration.

13. Check the ALB Target Group

Open:

EC2
 ↓
Target Groups
 ↓
Green Target Group
 ↓
Targets

Check the target health.

Expected:

Green Target
     ↓
Health Check
     ↓
/health
     ↓
HTTP 500
     ↓
Unhealthy

The target should not become a healthy production target.

14. Understand the Health Check

The ALB health check might be configured as:

Protocol: HTTP

Port: traffic-port

Path: /health

Success Code: 200

The load balancer sends:

GET /health

The failed application responds:

HTTP 500

Therefore:

Expected:
200

Received:
500

Result:
UNHEALTHY

This is how the infrastructure detects the failed deployment.

15. Verify Application Logs

Open:

AWS Console
    ↓
CloudWatch
    ↓
Log Groups
    ↓
ECS Application Logs

Look for:

ERROR
Exception
Configuration error
Health check failure
Application startup failure
Database connection failure

The exact message depends on the failure you introduced.

Example:

ERROR: Health endpoint returned unhealthy status

Logs help answer:

Why did the deployment fail?

16. Verify the Deployment Status

Check the ECS deployment.

You may see information indicating that the new deployment is not healthy or has failed validation.

Conceptually:

Deployment
    │
    ▼
Green Tasks
    │
    ▼
Health Checks
    │
    ▼
UNHEALTHY
    │
    ▼
Deployment Cannot Proceed

The exact status wording depends on whether you are using ECS rolling deployments or ECS blue/green with CodeDeploy.

17. Confirm Traffic Was NOT Shifted

This is one of the most important parts of the exercise.

Production traffic should remain on:

Blue
v1.0.0

The failed:

Green
v1.1.0-failed

must not become the production environment.

Expected:

                 ALB
                  │
                  ▼
            Blue Production
                v1.0.0
                  │
                  ▼
                Users

While:

Green
v1.1.0-failed
     │
     ▼
Health Check
     │
     ▼
FAIL
18. Test the Production Application

Access the normal ALB application URL.

For example:

http://<ALB-DNS-NAME>/

Check:

GET /
GET /health

Expected production result:

{
  "status": "healthy"
}

The production response should still come from:

Blue
v1.0.0
19. Verify Green Separately

If your architecture allows direct testing of the Green target group, test Green independently.

Example:

Green Target Group
        ↓
Green ECS Task
        ↓
/health
        ↓
FAIL

This demonstrates why the Green environment must be validated separately before traffic shifting.

20. What Prevented Users From Receiving the Failed Version?

Several controls can prevent the failed release from reaching users.

Control 1 — Separate Green Environment

The new version was deployed to:

Green

rather than immediately replacing:

Blue
Control 2 — Health Check

The ALB checked:

/health

The failed application returned an unsuccessful response.

Therefore:

Health Check
     ↓
FAIL
Control 3 — Deployment Validation

The Green environment was validated before traffic shifting.

Because validation failed:

Traffic Shift
     ↓
NOT PERFORMED
Control 4 — Blue Remained Production

The existing Blue environment continued serving users:

Blue v1.0.0
     ↓
ALB
     ↓
Users

Therefore the failed Green version was isolated from production traffic.

21. Failure Investigation Process

Use this troubleshooting sequence:

1. Check ECS Service
       ↓
2. Check Green Deployment
       ↓
3. Check ECS Tasks
       ↓
4. Check Task Health
       ↓
5. Check Target Group
       ↓
6. Check ALB Health Check
       ↓
7. Check /health endpoint
       ↓
8. Check CloudWatch Logs
       ↓
9. Identify root cause
       ↓
10. Confirm Blue is still serving traffic

This is the same general investigation pattern used in production environments.

22. Trainee Questions

After completing the exercise, explain the following.

Question 1 — What failed?

Example answer:

The Green deployment contained an intentionally broken
application health endpoint. The /health endpoint did not
return the expected HTTP 200 response.
Question 2 — Why did it fail?

Example:

The failure was deliberately introduced into the new
application version to simulate an unsuccessful release.

The ALB health check expected /health to return HTTP 200,
but the application returned an unsuccessful response.
Question 3 — How was it detected?

Example:

The failure was detected by the health-check mechanism.

The ALB sent a request to:

/health

The Green application returned an unsuccessful response,
so the target was considered unhealthy.

CloudWatch logs can provide additional diagnostic information.

Question 4 — What prevented users from receiving the failed version?

Example:

The Green environment was validated before production
traffic was shifted.

Because the health check failed, the deployment did not
proceed to the traffic-shift stage.

Blue remained the production environment.
23. Expected Failure Flow

The complete flow should be:

              Current Production
                     │
                     ▼
                Blue v1.0.0
                     │
                     ▼
                   Users


              New Deployment
                     │
                     ▼
                Green v1.1.0
                     │
                     ▼
                 /health
                     │
                     ▼
                HTTP 500
                     │
                     ▼
                UNHEALTHY
                     │
                     ▼
             Validation FAILED
                     │
                     ▼
            Traffic NOT Shifted
                     │
                     ▼
             Blue Remains PROD
24. Recovery

After proving the failure, do not leave the training environment broken unnecessarily.

Remove the deliberate failure by deploying the corrected application version.

Example:

v1.1.0-failed
      ↓
Remove failure
      ↓
v1.1.0-corrected
      ↓
Build image
      ↓
Push to ECR
      ↓
Create task-definition revision
      ↓
Deploy to Green
      ↓
Health Check
      ↓
Validation

Expected:

/health
   ↓
HTTP 200
   ↓
Healthy

Only after successful validation should a normal traffic shift be considered.

25. Evidence to Capture

Record evidence for the training exercise.

Evidence 1 — Failed Green Task
ECS
 ↓
Tasks
 ↓
Green
 ↓
Unhealthy / Failed
Evidence 2 — Target Group

Show:

Green Target
     ↓
Unhealthy
Evidence 3 — Health Check

Show:

Path:
/health

Expected:
HTTP 200

Actual:
Failure
Evidence 4 — CloudWatch Logs

Capture the application error.

Evidence 5 — Production

Show that:

Blue v1.0.0

remained production.

Evidence 6 — Application

Verify:

/

and:

/health

still work through the production ALB.

26. Failure Scenario Comparison
Failure	Detection	Expected Result
Wrong environment variable	Application startup/logs	Green fails
Invalid configuration	Application startup/logs	Green fails
Broken /health	ALB health check	Green unhealthy
Incorrect container port	ALB/ECS connectivity	Green unhealthy
Invalid secret reference	ECS task/application logs	Green fails

The exact detection mechanism depends on the failure introduced.

27. Production Principle

A production deployment should not depend only on:

"Container started successfully"

A container can be running while the application itself is broken.

For example:

Container
   │
   ├── RUNNING
   │
   └── Application
           │
           └── Broken

Therefore, deployment validation should check the application itself:

Container
    ↓
Application
    ↓
Health Endpoint
    ↓
Dependencies
    ↓
Functional Validation
28. Final Architecture

After the failed deployment:

                         ALB
                          │
                          ▼
                   Blue Production
                      v1.0.0
                          │
                          ▼
                        Users


                    Green Environment
                       v1.1.0
                          │
                          ▼
                     Health Check
                          │
                          ▼
                         FAIL
                          │
                          ▼
                  Traffic Not Shifted
29. Success Criteria

The task is successful when all of the following are demonstrated:

A controlled failure was intentionally introduced.

The failure affected only Green.

Blue remained the production environment.

Green was deployed with the failed version.

ECS task status was inspected.

ALB target health was inspected.

/health validation failed.

CloudWatch logs were checked.

The failed deployment did not receive production traffic.

Blue continued serving users.

The failure cause was identified.

The environment was restored after testing.

30. Final Trainee Explanation

A trainee should be able to explain the exercise like this:

We intentionally introduced a failure into the Green
application version.

The failed version was deployed to the Green environment
while Blue continued serving production traffic.

The ALB health check tested the /health endpoint and detected
that the Green application was unhealthy.

Because Green failed validation, the deployment did not
proceed with the production traffic shift.

Blue therefore remained the production environment and users
continued receiving the stable v1.0.0 application.

After identifying the problem, we corrected the application
and restored the Green environment.
Key Concept
FAILED RELEASE
      ↓
DETECT
      ↓
STOP
      ↓
DO NOT SHIFT TRAFFIC
      ↓
KEEP BLUE PRODUCTION
      ↓
FIX
      ↓
VALIDATE AGAIN
      ↓
DEPLOY SAFELY

This is the core idea behind failure isolation and safe deployment validation in a Blue/Green ECS deployment.