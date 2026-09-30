ECS Green Environment Validation
Objective

Before shifting production traffic to a new ECS application version, validate that the Green environment is healthy and functioning correctly.

The purpose of validation is to identify deployment issues before users are affected.

Target principle:

Validate First
      │
      ▼
Shift Traffic Later

Production traffic should only be shifted after all validation checks pass successfully.

1. Why Validation Is Required

Consider the deployment flow:

Deploy New Version
        │
        ▼
Green Environment

Without validation:

Deploy
   │
   ▼
Immediate Traffic Shift

Potential problems:

Application startup failure
Database connection failure
Missing secrets
Missing environment variables
API errors
Dependency failures

Validation helps detect these issues before production users are affected.

2. Validation Architecture
Green Environment
       │
       ├── Container Healthy
       ├── ALB Healthy
       ├── Application Healthy
       └── Dependencies Healthy

Every component must pass validation before traffic shifting.

3. Green Environment Overview

Example:

Blue = v1.0.0

Green = v1.1.0

Current state:

Users
  │
  ▼
Blue Environment

Green is isolated for testing.

4. Validation Flow
Deploy v1.1.0
       │
       ▼
Container Startup
       │
       ▼
Health Check
       │
       ▼
Functional Testing
       │
       ▼
Dependency Validation
       │
       ▼
Traffic Shift Decision
5. Container Health Validation

First verify that the ECS task starts correctly.

Navigate:

ECS
 ↓
Cluster
 ↓
Service
 ↓
Tasks

Expected:

Task Status = RUNNING

Not:

STOPPED
PENDING
FAILED
6. Container Startup Verification

Review task events.

Expected:

Task Started Successfully

Check CloudWatch Logs.

Expected:

Application Started
Server Listening

Example:

Listening on Port 3000
7. Health Endpoint Validation

The first validation endpoint should be:

/health

Expected response:

{
  "status": "healthy"
}

Expected HTTP status:

200 OK
8. Health Check Architecture
ALB
 │
 ▼
/health
 │
 ▼
Container
 │
 ▼
HTTP 200

A healthy response indicates that the application is running.

9. Validate Root Endpoint

Test:

GET /

Expected:

HTTP 200

Verify:

Response received
Correct content
No server errors
10. Validate Health Endpoint

Test:

GET /health

Expected:

HTTP 200

Response:

{
  "status": "healthy"
}
11. Validate Application APIs

Test all important APIs.

Examples:

/api/users

/api/products

/api/orders

/api/auth

Verify:

Successful Responses
Correct Data
Expected Behavior
12. API Validation Flow
API Request
      │
      ▼
Application
      │
      ▼
Expected Response

Check:

HTTP status codes
Response structure
Business logic
13. Database Connectivity Validation

Many applications depend on a database.

Architecture:

Application
      │
      ▼
Database

Verify:

Connection Successful
Queries Successful
14. Database Validation Example

Application startup logs should show:

Database Connected

Not:

Connection Timeout

Authentication Failed

Connection Refused
15. Secrets Validation

Verify that required secrets were injected.

Examples:

DATABASE_PASSWORD

JWT_SECRET

API_SECRET

Architecture:

Secrets Manager
       │
       ▼
ECS Task
       │
       ▼
Application
16. Secret Validation

Expected:

Application Starts Successfully

Common failures:

Secret Missing

Permission Denied

Incorrect ARN

Review CloudWatch Logs for secret-related errors.

17. Environment Variable Validation

Verify required configuration values.

Examples:

ENVIRONMENT

PORT

APP_NAME

LOG_LEVEL

Expected:

Values Loaded Successfully
18. Dependency Validation

Many applications depend on external services.

Examples:

Database

Secrets Manager

External APIs

Authentication Services

Architecture:

Application
      │
      ▼
Dependencies

All dependencies must be reachable.

19. ALB Health Validation

Navigate:

EC2
 ↓
Target Groups

Select:

Green Target Group

Expected:

Healthy Targets

Not:

Unhealthy
20. Target Group Validation

Expected flow:

ALB
 │
 ▼
Target Group
 │
 ▼
Healthy Target

Example:

1/1 Healthy

or:

2/2 Healthy
21. CloudWatch Log Validation

Navigate:

CloudWatch
 ↓
Log Groups

Review:

Application Logs

Verify absence of:

Exceptions

Fatal Errors

Connection Failures
22. Complete Validation Checklist
Container
☑ Task Running

☑ Container Started

☑ No Startup Failures
ALB
☑ Target Healthy

☑ Health Check Passing
Application
☑ GET /

☑ GET /health

☑ APIs Working
Database
☑ Connection Successful

☑ Queries Successful
Configuration
☑ Environment Variables Present

☑ Secrets Available
Dependencies
☑ External Services Reachable
23. Validation Decision

Decision process:

Validation Complete
        │
        ▼
All Checks Pass?
      /     \
    Yes      No
     │        │
     ▼        ▼
Shift      Fix Issue
Traffic
24. Traffic Shift Prerequisite

Traffic should not move until:

Container Healthy

ALB Healthy

Application Healthy

Dependencies Healthy

Architecture:

Green Environment
       │
       ├── Container Healthy
       ├── ALB Healthy
       ├── Application Healthy
       └── Dependencies Healthy
25. Expected Deployment Flow
Deploy v1.1.0
       │
       ▼
Green Environment
       │
       ▼
Validate
       │
       ▼
Pass?
    /      \
  Yes       No
   │         │
   ▼         ▼
Shift     Fix Issue
Traffic
26. Success Criteria

The Green environment is validated when:

☑ ECS Task Running

☑ ALB Health Checks Passing

☑ GET / Successful

☑ GET /health Successful

☑ APIs Working

☑ Database Connectivity Verified

☑ Environment Variables Loaded

☑ Secrets Retrieved Successfully

☑ Dependencies Healthy

☑ CloudWatch Logs Clean
Conclusion

Green environment validation is a critical step in Blue/Green deployments. Before shifting production traffic, the new application version must be verified at multiple levels including container health, ALB health checks, application functionality, API behavior, database connectivity, configuration loading, secret retrieval, and dependency availability. Only after all validation checks pass should traffic be shifted from the Blue environment to the Green environment.