Day 31 — CI/CD Fundamentals
Introduction

Modern software development requires teams to release applications quickly, safely, and reliably.

Without automation, developers must manually:

Build applications
Run tests
Package software
Deploy to servers
Verify deployments

This process is slow and error-prone.

CI/CD automates these activities and helps teams deliver software faster with better quality.

What is CI/CD?

CI/CD stands for:

Continuous Integration
Continuous Delivery
Continuous Deployment

CI/CD is a software development practice that automates code integration, testing, validation, and deployment.

CI/CD High-Level Architecture
Developer
    │
    ▼
GitHub Repository
    │
    ▼
CI Pipeline
(Build + Test)
    │
    ▼
Validated Artifact
    │
    ▼
CD Pipeline
(Deploy)
    │
    ▼
AWS Environment
    │
    ▼
Users
Continuous Integration (CI)
What is CI?

Continuous Integration is the practice of frequently merging code changes into a shared repository and automatically validating those changes.

Every code change should be:

Built
Tested
Validated

before being merged into the main branch.

CI Goal

The goal of CI is to find problems early.

Instead of discovering bugs during deployment, issues are detected immediately after code is pushed.

CI Workflow
Developer
    │
    ▼
Push Code
    │
    ▼
GitHub
    │
    ▼
Build Application
    │
    ▼
Run Tests
    │
    ▼
Validate Code
    │
    ▼
Pass / Fail
Example CI Process

A developer modifies an application.

Example:

Fix login page bug

The developer pushes code:

git push origin feature/login-fix

GitHub Actions automatically starts:

Build Application
Run Unit Tests
Run Linting
Run Validation Checks

If all checks pass:

CI Success

If any check fails:

CI Failure

Developer fixes the issue and pushes again.

Benefits of Continuous Integration
Early Bug Detection

Problems are found quickly.

Developer pushes code
      ↓
Tests fail
      ↓
Issue fixed immediately
Better Code Quality

Automated testing ensures stable code.

Faster Development

Developers receive rapid feedback.

Reduced Integration Problems

Frequent code merges prevent large conflicts.

CI Pipeline
What is a CI Pipeline?

A CI pipeline is an automated workflow that verifies code changes.

Typical CI Pipeline Stages
Source Code
      │
      ▼
Build
      │
      ▼
Static Analysis
      │
      ▼
Unit Tests
      │
      ▼
Validation
      │
      ▼
Artifact Creation
CI Pipeline Example

GitHub Actions:

Developer Push
      │
      ▼
Checkout Code
      │
      ▼
Install Dependencies
      │
      ▼
Build Application
      │
      ▼
Run Tests
      │
      ▼
Create Docker Image
Continuous Delivery (CD)
What is Continuous Delivery?

Continuous Delivery is the practice of automatically preparing validated software for deployment.

The software is always deployment-ready.

A human approval may still be required before production deployment.

Continuous Delivery Workflow
Validated Build
      │
      ▼
Create Artifact
      │
      ▼
Store Artifact
      │
      ▼
Approval Required
      │
      ▼
Deploy
Example

Application passes CI.

Docker image is created:

myapp:v1.0.0

Image is stored in:

Amazon ECR

Operations team reviews deployment.

After approval:

Deploy to AWS
Continuous Deployment
What is Continuous Deployment?

Continuous Deployment automatically deploys every successful change to production without manual approval.

Continuous Deployment Workflow
Developer Push
      │
      ▼
Build
      │
      ▼
Test
      │
      ▼
Validation
      │
      ▼
Automatic Deployment
      │
      ▼
Production
Continuous Delivery vs Continuous Deployment
Feature	Continuous Delivery	Continuous Deployment
Deployment Ready	Yes	Yes
Human Approval	Required	Not Required
Automation Level	High	Very High
Production Deployment	Manual Trigger	Automatic
Risk Level	Lower	Higher
CD Pipeline
What is a CD Pipeline?

A CD pipeline automates software delivery and deployment.

It takes validated software from the CI pipeline and deploys it to environments.

Typical CD Pipeline
Validated Artifact
        │
        ▼
Development
        │
        ▼
Testing
        │
        ▼
Staging
        │
        ▼
Production
AWS Deployment Example
GitHub Actions
      │
      ▼
Docker Image
      │
      ▼
Amazon ECR
      │
      ▼
EC2 / ECS
      │
      ▼
Application Running
Deployment Approval
What is Deployment Approval?

Deployment approval is a control mechanism that requires human authorization before deployment.

Why Use Approval?

Production systems are critical.

Approval helps prevent:

Accidental Deployments
Incorrect Versions
Unverified Releases
Production Outages
Approval Workflow
Build Success
      │
      ▼
Testing Complete
      │
      ▼
Manager Approval
      │
      ▼
Deploy Production
Example

GitHub Actions Environment Protection:

Deploy Production
      │
      ▼
Waiting for Approval
      │
      ▼
Approved
      │
      ▼
Deployment Starts
Rollback
What is Rollback?

Rollback means returning to a previous stable version when a deployment causes problems.

Why Rollback?

Possible deployment failures:

Application Crash
Database Error
Performance Issue
Configuration Problem
Rollback Architecture
v1.0.1
    │
    ▼
Stable

Deploy v1.0.2
    │
    ▼
Failure

Rollback
    │
    ▼
v1.0.1
Docker Rollback Example

Current deployment:

myapp:v1.0.2

Issue detected.

Rollback to:

myapp:v1.0.1

Application becomes stable again.

CI/CD in AWS
Example End-to-End Flow
Developer
    │
    ▼
GitHub
    │
    ▼
GitHub Actions
    │
    ▼
Build Application
    │
    ▼
Run Tests
    │
    ▼
Build Docker Image
    │
    ▼
Push Image to ECR
    │
    ▼
Deploy to EC2/ECS
    │
    ▼
Health Check
    │
    ▼
Production
Key Revision Points
Continuous Integration (CI)
Verify code changes
Build applications
Run tests
Validate quality
Detect issues early
Continuous Delivery (CD)
Prepare software for deployment
Deployment approval may be required
Always release-ready
Continuous Deployment
Automatic production deployment
No manual approval
Every successful build is released
Deployment Approval
Human review before deployment
Common in production environments
Reduces risk
Rollback
Return to previous stable version
Used when deployment fails
Minimizes downtime
Quick Interview Questions
Q1. What is CI?

Answer: Continuous Integration is the process of automatically building, testing, and validating code whenever developers make changes.

Q2. What is CD?

Answer: Continuous Delivery/Deployment is the process of automatically delivering or deploying validated software to environments.

Q3. What is the main goal of CI?

Answer: To detect issues early and maintain code quality.

Q4. What is the difference between Continuous Delivery and Continuous Deployment?

Answer: Continuous Delivery requires approval before production deployment, while Continuous Deployment deploys automatically.

Q5. What is rollback?

Answer: Rollback is the process of restoring a previously working version when a deployment fails.