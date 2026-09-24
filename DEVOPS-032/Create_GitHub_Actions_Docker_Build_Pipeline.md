TASK 4 – Create GitHub Actions Docker Build Pipeline
Objective

Create a GitHub Actions workflow that automatically builds a Docker image whenever code is pushed to the main branch.

This is the first step toward a complete CI/CD pipeline.

What is GitHub Actions?

GitHub Actions is GitHub's automation platform.

It allows us to automatically run tasks when events occur.

Examples:

Push code
Create Pull Request
Merge code
Release application
CI/CD Architecture
Developer
    │
    ▼
Git Push
    │
    ▼
GitHub Repository
    │
    ▼
GitHub Actions Workflow
    │
    ▼
Checkout Source Code
    │
    ▼
Docker Build
    │
    ▼
Docker Image Created
Project Structure

Create the following folder structure:

project/
│
├── app.py
├── Dockerfile
│
└── .github/
    └── workflows/
        └── docker-build.yml
Create Workflow File

Location:

.github/workflows/docker-build.yml

Content:

name: Docker Build

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Build Docker Image
        run: |
          docker build -t devops-app:${{ github.sha }} .
Understanding the Workflow
name
name: Docker Build

Workflow name shown in GitHub Actions.

Example:

Actions
 └── Docker Build
on
on:
  push:

Defines when workflow starts.

Architecture:

Git Push
    │
    ▼
Workflow Triggered
branches
branches:
  - main

Workflow runs only when code is pushed to:

main

Example:

feature/login
      │
      ▼
No Build

main
      │
      ▼
Build Starts
jobs
jobs:

A workflow contains one or more jobs.

Example:

Workflow
   │
   └── Build Job
runs-on
runs-on: ubuntu-latest

GitHub creates a temporary Ubuntu runner.

Architecture:

GitHub
   │
   ▼
Ubuntu Runner
   │
   ▼
Execute Pipeline
steps
steps:

A job is executed step-by-step.

Example:

Step 1
Step 2
Step 3
Step 1 – Checkout Source Code
- name: Checkout Source Code
  uses: actions/checkout@v4

Purpose:

Downloads repository contents into the GitHub runner.

Architecture:

GitHub Repository
        │
        ▼
GitHub Runner

Without checkout:

Dockerfile Not Found
Source Code Not Found

Build will fail.

Step 2 – Build Docker Image
- name: Build Docker Image
  run: |
    docker build -t devops-app:${{ github.sha }} .

Purpose:

Build Docker image from Dockerfile.

Architecture:

Dockerfile
     │
     ▼
docker build
     │
     ▼
Docker Image
Understanding github.sha
${{ github.sha }}

Represents the Git commit ID.

Example:

a7c4e12f5d6b8a9

Image created:

devops-app:a7c4e12f5d6b8a9

Benefits:

Unique image version
Traceable deployment
Easy rollback
Better auditing
Build Flow
Developer Pushes Code
          │
          ▼
GitHub Repository
          │
          ▼
Workflow Trigger
          │
          ▼
Checkout Code
          │
          ▼
Docker Build
          │
          ▼
Docker Image Created
Push Workflow to GitHub

Add workflow file:

mkdir -p .github/workflows

Save:

.github/workflows/docker-build.yml

Commit changes:

git add .

git commit -m "Added Docker build workflow"

git push origin main
Verify Pipeline Execution

Open:

GitHub Repository
      │
      ▼
Actions Tab

Expected:

Docker Build
   ✔ Success

or

Docker Build
   ✖ Failed
Successful Execution Example
Run actions/checkout@v4
✔ Completed

Run docker build
✔ Completed

Successfully built image:
devops-app:a7c4e12f5d6b8a9
Common Errors
Dockerfile Missing

Error:

failed to read dockerfile

Solution:

Ensure Dockerfile exists in project root.
Build Failure

Error:

pip install failed

Solution:

Check requirements.txt
Wrong Branch

Workflow:

branches:
  - main

If code is pushed to:

develop

Workflow will not start.

End-to-End Workflow Diagram
Code
 │
 ▼
Git Push
 │
 ▼
GitHub
 │
 ▼
GitHub Actions
 │
 ▼
Checkout Source
 │
 ▼
Docker Build
 │
 ▼
Docker Image
 │
 ▼
Build Success
Deliverable
Workflow File
.github/workflows/docker-build.yml
Expected Outcome

✅ Code pushed to main

✅ GitHub Actions starts automatically

✅ Repository checked out successfully

✅ Docker image built successfully

✅ Build visible in GitHub Actions tab

✅ Foundation ready for future steps:

Docker Image Scanning
Amazon ECR Push
ECS/EC2 Deployment
Health Checks
Full CI/CD Pipeline