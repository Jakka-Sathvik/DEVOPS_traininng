ECS Blue Green Deployment Architecture
Objective

Design a production-style Blue/Green deployment architecture for Amazon ECS that enables safe application releases, deployment validation, and rapid rollback.

Target architecture:

                    ALB
                     │
              ┌──────┴──────┐
              ▼             ▼
        Blue Target     Green Target
           Group            Group
              │             │
              ▼             ▼
          ECS v1          ECS v2
1. What Is Blue/Green Deployment?

Blue/Green deployment is a release strategy that maintains two separate application environments.

Environment names:

Blue
Green

Only one environment receives production traffic at a time.

2. Blue Environment

Blue represents the currently running production application.

Example:

Production
Version 1.0.0

Architecture:

Blue Environment
       │
       ▼
    ECS v1.0.0

Users currently access this environment.

3. Green Environment

Green represents the new application release.

Example:

New Release
Version 1.1.0

Architecture:

Green Environment
        │
        ▼
     ECS v1.1.0

Initially, production users do not access this environment.

4. High-Level Architecture
                    ALB
                     │
              ┌──────┴──────┐
              ▼             ▼
        Blue Target     Green Target
           Group            Group
              │             │
              ▼             ▼
          ECS v1          ECS v2

Components:

Application Load Balancer (ALB)
Blue Target Group
Green Target Group
ECS Service Version 1
ECS Service Version 2
5. Component Overview
Application Load Balancer

The ALB receives user traffic.

Users
   │
   ▼
  ALB

Responsibilities:

Traffic routing
Health checks
Deployment traffic shifting
Blue Target Group

Production target group.

Blue Target Group
        │
        ▼
     ECS v1

Receives live production traffic.

Green Target Group

Deployment target group.

Green Target Group
         │
         ▼
      ECS v2

Used for validation before release.

6. Initial Production State

Current production:

Users
  │
  ▼
 ALB
  │
  ▼
Blue Target Group
  │
  ▼
ECS v1.0.0

Green environment exists but receives no production traffic.

7. Deploy New Version

Development team creates:

Docker Image
v1.1.0

Deployment flow:

Build Image
    │
    ▼
Push To ECR
    │
    ▼
Create Task Definition Revision
    │
    ▼
Deploy To Green
8. Green Environment Deployment

New version runs independently.

Architecture:

Blue Environment
     │
     ▼
 ECS v1.0.0


Green Environment
      │
      ▼
 ECS v1.1.0

Both versions operate simultaneously.

9. Health Check Validation

Before traffic shift:

Green Environment
       │
       ▼
Health Check

Example endpoint:

/health

Expected response:

{
  "status": "healthy"
}

Expected status:

HTTP 200
10. Health Check Flow
ALB
 │
 ▼
/health
 │
 ▼
ECS v1.1.0
 │
 ▼
HTTP 200
 │
 ▼
Healthy

Only healthy tasks can receive traffic.

11. Functional Testing

After health checks succeed:

Green Environment
       │
       ▼
Functional Testing

Example tests:

/health
/login
/api/users
/api/orders

Verify:

Application startup
API functionality
Database connectivity
Authentication
Business logic
12. Functional Test Architecture
Tester
  │
  ▼
Green Environment
  │
  ▼
Application Tests

Production traffic remains unaffected.

13. Traffic Shift

After validation succeeds:

Traffic Shift

Before:

Users
  │
  ▼
Blue

After:

Users
  │
  ▼
Green
14. Traffic Shift Architecture

Before deployment:

                    ALB
                     │
                     ▼
             Blue Target Group
                     │
                     ▼
                ECS v1.0.0

After deployment:

                    ALB
                     │
                     ▼
            Green Target Group
                     │
                     ▼
                ECS v1.1.0
15. Green Becomes Production

Final state:

Production
Version 1.1.0

Architecture:

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

Green is now the production environment.

16. Rollback Scenario

Suppose issues appear after release.

Example:

Application Errors
Performance Problems
Deployment Failure

Rollback:

ALB
 │
 ▼
Blue Target Group
 │
 ▼
ECS v1.0.0

Traffic returns to the previous version.

17. Rollback Flow
Green Failure
      │
      ▼
Traffic Shift Back
      │
      ▼
Blue Environment

Recovery is significantly faster than rebuilding the application.

18. Example Deployment Timeline
Stage 1
Blue = v1.0.0
Production
Stage 2
Green = v1.1.0
Testing
Stage 3
Health Check
Functional Test
Validation
Stage 4
Traffic Shift
Stage 5
Green = Production
19. AWS Service Integration

Typical AWS architecture:

GitHub Actions
        │
        ▼
Amazon ECR
        │
        ▼
ECS
        │
        ▼
CodeDeploy
        │
        ▼
Blue/Green Deployment

AWS CodeDeploy manages:

Traffic shifting
Validation
Rollback
Deployment lifecycle
20. Benefits

Blue/Green deployment provides:

Near-Zero Downtime
Safer Releases
Production Validation
Fast Rollback
Reduced Deployment Risk
21. Trade-Offs

Blue/Green deployment requires:

Additional Infrastructure
Additional ECS Tasks
Additional Target Group
Higher Cost
More Configuration

Because two environments run simultaneously.

22. Production Deployment Flow
Deploy v1.1.0
       │
       ▼
Green Environment
       │
       ▼
Health Check
       │
       ▼
Functional Test
       │
       ▼
Traffic Shift
       │
       ▼
Green Becomes Production
23. Complete Production Architecture
                          Users
                            │
                            ▼
                           ALB
                            │
               ┌────────────┴────────────┐
               ▼                         ▼
      Blue Target Group         Green Target Group
               │                         │
               ▼                         ▼
        ECS Service v1            ECS Service v2
          Version 1.0.0            Version 1.1.0
               │                         │
               ▼                         ▼
          Production                Validation
24. Success Criteria

The task is complete when the trainee can explain:

Blue Environment

Green Environment

Target Groups

Traffic Shifting

Health Checks

Functional Testing

Rollback Process

CodeDeploy Integration

And understand the deployment flow:

Deploy v1.1.0
       ↓
Green Environment
       ↓
Health Check
       ↓
Functional Test
       ↓
Traffic Shift
       ↓
Green becomes Production
Conclusion

Blue/Green deployment is a production-grade deployment strategy that uses separate environments for the current and new application versions. The new release is deployed to the Green environment, validated through health checks and functional testing, and only then receives production traffic. If issues occur, traffic can quickly be shifted back to the Blue environment, providing a reliable deployment and rollback mechanism with minimal user impact.