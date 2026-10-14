CI/CD Pipeline Controls

Project: DevOps Training Application
Platform: GitHub Actions + AWS
Deployment: Amazon ECS
Region: ap-south-1

1. Purpose

This document defines the controls used to secure, validate, deploy, verify, and roll back application and infrastructure changes.

The pipeline follows this principle:

Commit
  ↓
Lint
  ↓
Test
  ↓
Validate
  ↓
Security Scan
  ↓
Terraform Plan
  ↓
Approval
  ↓
Build
  ↓
ECR
  ↓
ECS Deployment
  ↓
Verification
  ↓
Success / Rollback

2. Pipeline Control Summary

Control

Tool / Mechanism

Failure Action

Source control

GitHub

Unauthorized changes prevented

Branch protection

GitHub

Direct production changes restricted

Code linting

Ruff / ESLint

Pipeline stops

Unit testing

Application test framework

Pipeline stops

Terraform formatting

terraform fmt

Pipeline stops

Terraform validation

terraform validate

Pipeline stops

Terraform linting

TFLint

Pipeline stops

IaC security

Checkov

Pipeline stops

Docker security

Trivy

Pipeline stops

Terraform plan

Terraform

Review required

AWS authentication

GitHub OIDC

No static AWS keys

IAM security

Least privilege

Unauthorized actions denied

Artifact tracking

Git SHA + digest

Exact artifact identified

Deployment approval

GitHub Environment

Manual approval required

ECS verification

ECS health/stability

Deployment stops on failure

Application verification

ALB /health

Deployment fails

Monitoring

CloudWatch

Errors investigated

Rollback

ECS previous revision

Previous version restored

3. Source Control Controls

All application and infrastructure code is stored in GitHub.

The repository contains:

.github/workflows/
terraform/
app/
docs/

Controls

Pull requests are used for changes.

Production changes require review.

Direct changes to the production branch should be restricted.

CI checks must pass before merging.

Commit history is retained for traceability.

4. Linting Control

Linting is executed before deployment.

Example:

terraform fmt -check -recursive

Application linting is also performed using the project's appropriate linter.

Objective

Detect:

Formatting problems

Coding mistakes

Unused code

Invalid style

Potential maintainability problems

Gate

Lint PASS → Continue
Lint FAIL → Stop

5. Testing Control

Application tests run before building and deployment.

Source Code
    ↓
Tests
    ↓
PASS → Continue
FAIL → Stop

A failed test prevents the application from reaching ECR or ECS.

6. Terraform Validation Controls

Terraform configuration is checked using:

terraform fmt -check -recursive
terraform validate

TFLint is also used where applicable.

Control objective

Prevent invalid infrastructure configuration from reaching AWS.

7. Infrastructure Security Control

Terraform configurations are scanned using Checkov.

The scan checks for security problems such as:

Publicly exposed resources

Weak security groups

Missing encryption

Excessive IAM permissions

Insecure infrastructure configuration

Critical findings should fail the pipeline unless explicitly reviewed and accepted.

Checkov
   │
   ├── PASS → Continue
   │
   └── FAIL → Stop

8. Docker Security Control

Docker images are scanned with Trivy before being pushed to ECR.

Example policy:

CRITICAL vulnerabilities → Fail
HIGH vulnerabilities → Fail

Pipeline:

Docker Build
     ↓
Trivy Scan
     ↓
   PASS
     ↓
    ECR

A vulnerable image must not be deployed.

9. AWS Authentication Control

GitHub Actions uses GitHub OIDC to authenticate with AWS.

The pipeline does not use permanent AWS access keys.

GitHub Actions
      ↓
GitHub OIDC
      ↓
AWS STS
      ↓
IAM Role
      ↓
Temporary Credentials

Workflow permissions are restricted:

permissions:
  id-token: write
  contents: read

10. IAM Least-Privilege Control

AWS IAM permissions are restricted to the actions required by each workflow.

Separate roles are recommended:

github-terraform-role
github-ecs-deploy-role

The Terraform role manages infrastructure.

The ECS deployment role manages application deployment.

AdministratorAccess is not permitted.

Sensitive permissions such as:

iam:PassRole

must be restricted to specific ECS roles.

11. Artifact Version Control

Every Docker image is associated with a Git commit.

Example:

Git commit:
abc123456

Docker image:
devops-training-app:abc123456

Image digest:
sha256:8f9a...

The image is therefore traceable to the exact source code that produced it.

Production deployments should not rely only on:

latest

Immutable commit-based tags and image digests are preferred.

12. Terraform Version Control

The Terraform dependency lock file is committed:

.terraform.lock.hcl

Terraform configuration, provider versions, Git commit, and Terraform plan are tracked.

Plans can be saved as:

terraform plan -out=tfplan

This provides a record of the infrastructure changes being reviewed.

13. Deployment Approval Control

Production deployments require an approval gate.

Recommended environments:

dev
staging
production

Production should use a protected GitHub Environment with required reviewers.

Deployment flow:

CI Checks
    ↓
Terraform Plan
    ↓
Review
    ↓
Production Approval
    ↓
Deployment

14. ECS Deployment Control

The ECS service is deployed using an immutable task definition.

Example:

devops-training-task:41

New deployment:

devops-training-task:42

The previous revision remains available for rollback.

ECS deployment waits for service stability before continuing.

ECS Deployment
      ↓
Wait for Stability
      ↓
Stable → Continue
Failed → Stop / Rollback

15. Deployment Verification Controls

After deployment, the pipeline verifies:

ECS

Desired count = Running count
Pending count = 0
Service stable

ALB

Targets must be healthy.

Application

The health endpoint must return HTTP 200:

/health

Expected response:

{
  "status": "healthy"
}

Smoke Tests

Important application endpoints are tested after deployment.

16. Monitoring Control

CloudWatch is used to monitor the deployed application.

The deployment verification process checks:

ECS CPU
ECS memory
ALB request count
ALB 5XX
ALB response time
Application errors
RDS CPU
CloudWatch logs

Important log patterns include:

ERROR
Exception
Timeout
Connection refused
Database connection failure

17. Deployment Observation Window

A deployment is not considered successful immediately after ECS starts.

A short monitoring period is used to verify:

ECS stability
+
ALB health
+
Application health
+
5XX rate
+
Response time
+
Application logs

If these remain healthy, the deployment is marked successful.

18. Rollback Control

The previous known-good ECS task definition is recorded before deployment.

Example:

Previous:
devops-training-task:41

New:
devops-training-task:42

If deployment 42 fails:

42
 ↓
Failure
 ↓
Rollback
 ↓
41
 ↓
Verify

Rollback conditions include:

ECS tasks repeatedly fail

ALB targets become unhealthy

Health check fails

5XX errors increase significantly

Application errors increase

Response time becomes unacceptable

Database connectivity fails

19. Deployment Metadata

Each deployment should record:

{
  "application": "devops-training-app",
  "version": "v1.0.42",
  "git_commit": "abc123456",
  "image_digest": "sha256:8f9a...",
  "ecs_task_definition": "devops-training-task:42",
  "github_run": "128"
}

This provides deployment traceability.

20. Audit and Traceability

The following relationship must be maintained:

Git Commit
     ↓
GitHub Actions Run
     ↓
Docker Image
     ↓
Image Digest
     ↓
ECS Task Definition
     ↓
ECS Service
     ↓
CloudWatch Logs

This allows the team to determine exactly which source code is running in production.

21. Security Rules

The pipeline must follow these rules:

No long-lived AWS access keys in GitHub.

Use OIDC for AWS authentication.

Do not use AdministratorAccess.

Use least-privilege IAM roles.

Restrict iam:PassRole.

Do not deploy images that fail security scanning.

Do not deploy invalid Terraform.

Do not deploy when required tests fail.

Use immutable artifact versions.

Require production approval.

Maintain rollback capability.

Verify the application after deployment.

22. Failure Handling

Pipeline Failure
       │
       ▼
Identify Failed Gate
       │
       ├── Lint/Test
       │      → Fix code
       │
       ├── Terraform
       │      → Fix infrastructure
       │
       ├── Security
       │      → Remediate vulnerability
       │
       ├── Deployment
       │      → Investigate ECS
       │
       └── Health Check
              → Rollback

23. Control Checklist

Code

Pull request created

Code reviewed

Lint passed

Tests passed

Terraform

Format passed

Validation passed

TFLint passed

Checkov passed

Plan reviewed

Security

OIDC configured

No static AWS credentials

Least-privilege IAM

Trivy scan passed

iam:PassRole restricted

Artifact

Git SHA recorded

Docker image tagged

Image digest recorded

Artifact pushed to ECR

Deployment

Production approval obtained

Previous ECS revision recorded

ECS deployment completed

ECS service stable

ALB targets healthy

/health passed

Smoke tests passed

CloudWatch checked

Rollback

Previous revision available

Rollback procedure documented

Rollback verification tested

24. Final Control Model

             SOURCE CONTROL
                    │
                    ▼
              QUALITY GATES
        ┌───────────┼───────────┐
        │           │           │
      Lint         Test      Validate
        │           │           │
        └───────────┼───────────┘
                    ▼
              SECURITY GATES
             ┌──────┴──────┐
             ▼             ▼
          Checkov        Trivy
             │             │
             └──────┬──────┘
                    ▼
              TERRAFORM PLAN
                    │
                    ▼
             APPROVAL GATE
                    │
                    ▼
             OIDC + IAM
                    │
                    ▼
              ECR ARTIFACT
                    │
                    ▼
                ECS DEPLOY
                    │
                    ▼
          DEPLOYMENT VERIFICATION
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
        PASS                 FAIL
          │                   │
          ▼                   ▼
       SUCCESS             ROLLBACK
                              │
                              ▼
                     PREVIOUS REVISION
                              │
                              ▼
                          VERIFY

25. Success Criteria

The pipeline is considered production-ready when:

✓ Code quality enforced
✓ Tests enforced
✓ Terraform validated
✓ Infrastructure security scanned
✓ Docker security scanned
✓ OIDC authentication enabled
✓ IAM least privilege enforced
✓ Artifacts immutably versioned
✓ Terraform plans reviewed
✓ Production approval required
✓ ECS deployment verified
✓ Application health verified
✓ CloudWatch monitoring enabled
✓ Previous version recorded
✓ Rollback tested
✓ Deployment fully traceable

Result: Every production deployment is validated, secured, approved, traceable, observable, and reversible.