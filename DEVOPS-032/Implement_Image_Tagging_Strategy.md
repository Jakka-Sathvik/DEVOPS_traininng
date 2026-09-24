TASK 5 — Docker Image Tagging Strategy
Objective

Implement a Docker image tagging strategy that provides:

Unique image identification
Immutable deployment references
Easy rollback
Release version tracking
Human-friendly image names

The recommended production approach is to use the Git commit SHA as the primary deployment tag and optionally use semantic version and latest tags for additional convenience.

Why Docker Image Tags Matter

A Docker image tag identifies a particular version of an image.

Example:

devops-app:v1.0.0

Here:

devops-app
     │
     └── Image name

v1.0.0
     │
     └── Image tag

A good tagging strategy makes it possible to answer:

"Exactly which version of the source code is running in production?"

Problem with Only Using latest

Example:

devops-app:latest

Suppose today's image is:

devops-app:latest

Tomorrow a new image is pushed with the same tag:

devops-app:latest

The tag now points to the newer image.

Therefore, latest does not clearly identify the exact application version by itself.

Architecture:

Day 1

latest
  │
  ▼
Image A


Day 2

latest
  │
  ▼
Image B

The old image may still exist, but latest now points somewhere different.

Immutable Image Tags

An immutable tag should identify one specific build.

Git commit SHA is useful for this purpose.

Example:

devops-app:a81c92d

The tag represents a specific Git commit.

Architecture:

Git Commit
    │
    ▼
a81c92d
    │
    ▼
Docker Image
    │
    ▼
devops-app:a81c92d

This provides a direct connection between source code and Docker image.

Recommended Tagging Strategy

Use three types of tags:

devops-app:latest
devops-app:v1.0.0
devops-app:<commit-sha>

Each serves a different purpose.

1. latest

Example:

devops-app:latest

Purpose:

Human-friendly reference

It can represent the current default/recommended image.

However, deployments should preferably use an immutable version such as a commit SHA.

2. Semantic Version

Example:

devops-app:v1.0.0

Semantic versioning generally follows:

MAJOR.MINOR.PATCH

Example:

v1.0.0
v1.0.1
v1.1.0
v2.0.0

Meaning:

v1.0.0
 │ │ │
 │ │ └── PATCH
 │ └──── MINOR
 └────── MAJOR

A release can therefore be associated with a meaningful version.

3. Git Commit SHA

Example:

devops-app:a81c92d

This is the preferred deployment reference for this exercise.

The image can be traced directly to the Git commit that produced it.

Architecture:

Developer
    │
    ▼
Git Commit
    │
    ▼
Commit SHA
    │
    ▼
Docker Build
    │
    ▼
devops-app:a81c92d
Comparison
Tag	Example	Purpose
latest	devops-app:latest	Human-friendly reference
Semantic version	devops-app:v1.0.0	Release/version reference
Commit SHA	devops-app:a81c92d	Immutable deployment reference
Recommended Production Approach
latest
   │
   └── Human-friendly reference

v1.0.0
   │
   └── Release version

Commit SHA
   │
   └── Immutable deployment reference

For an actual deployment:

Production
    │
    ▼
devops-app:a81c92d

This makes it clear which Git commit is deployed.

Configure GitHub Actions

The existing workflow can create the commit-based image tag.

File:

.github/workflows/docker-build.yml

Use:

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

      - name: Set Image Tag
        run: |
          echo "IMAGE_TAG=${GITHUB_SHA}" >> "$GITHUB_ENV"

      - name: Build Docker Image
        run: |
          docker build -t devops-app:${IMAGE_TAG} .
Understanding GITHUB_SHA

GitHub automatically provides:

GITHUB_SHA

It represents the commit SHA associated with the workflow run.

Example:

a81c92d123456789...

The workflow creates:

devops-app:a81c92d123456789...

For this task, the full SHA is acceptable.

Why Use $GITHUB_ENV?

This command:

echo "IMAGE_TAG=${GITHUB_SHA}" >> "$GITHUB_ENV"

creates an environment variable that subsequent GitHub Actions steps can use.

Flow:

GITHUB_SHA
    │
    ▼
IMAGE_TAG
    │
    ▼
Docker Build

Then:

docker build -t devops-app:${IMAGE_TAG} .

uses that value.

Short SHA Option

A full Git SHA can be long.

For example:

a81c92d123456789abcdef...

A shorter identifier can be easier to read.

GitHub Actions can generate a short SHA with:

- name: Set Image Tag
  run: |
    echo "IMAGE_TAG=$(git rev-parse --short HEAD)" >> "$GITHUB_ENV"

Then the image becomes:

devops-app:a81c92d

For this training task, this is convenient for demonstrating the strategy.

Recommended Workflow for This Task
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

      - name: Set Image Tag
        run: |
          echo "IMAGE_TAG=$(git rev-parse --short HEAD)" >> "$GITHUB_ENV"

      - name: Build Docker Image
        run: |
          docker build -t devops-app:${IMAGE_TAG} .
Example Pipeline Execution

Suppose the latest commit is:

a81c92d

GitHub Actions performs:

Git Push
   │
   ▼
GitHub Actions
   │
   ▼
Checkout
   │
   ▼
Get Commit SHA
   │
   ▼
a81c92d
   │
   ▼
Docker Build
   │
   ▼
devops-app:a81c92d
Multiple Tags

A production pipeline may eventually create:

devops-app:a81c92d
devops-app:v1.0.0
devops-app:latest

Architecture:

                    ┌── devops-app:latest
                    │
Docker Image ────────┼── devops-app:v1.0.0
                    │
                    └── devops-app:a81c92d

The tags can refer to the same image digest while serving different identification purposes.

Rollback Example

Suppose production is running:

devops-app:a81c92d

A new release is deployed:

devops-app:b72e41f

If the new release needs to be rolled back, the deployment can explicitly reference:

devops-app:a81c92d

Instead of relying on:

devops-app:latest

Architecture:

Version 1
a81c92d
   │
   ▼
Production
   │
   ▼
Version 2
b72e41f
   │
   ▼
New Deployment

Rollback:

b72e41f
   │
   ▼
Rollback
   │
   ▼
a81c92d
Image Verification

After building locally:

docker images

Example:

REPOSITORY    TAG       IMAGE ID
devops-app    a81c92d   abc123...

You can inspect the image:

docker inspect devops-app:a81c92d
CI/CD Tagging Flow
Developer
    │
    ▼
Git Commit
    │
    ▼
GitHub
    │
    ▼
GITHUB_SHA
    │
    ▼
IMAGE_TAG
    │
    ▼
Docker Build
    │
    ▼
devops-app:<commit-sha>
    │
    ▼
Security Scan
    │
    ▼
Amazon ECR
    │
    ▼
Deployment
Best Practices
Use
devops-app:<commit-sha>

for precise deployment references.

Use:

devops-app:v1.0.0

for release versions.

Optionally use:

devops-app:latest

for a convenient human-friendly reference.

Avoid

Using only:

devops-app:latest

as the production deployment identifier.

Task 5 Deliverable

The GitHub Actions workflow should create an image using the Git commit SHA:

devops-app:<commit-sha>

Example:

devops-app:a81c92d

Expected architecture:

Code
 ↓
GitHub
 ↓
GitHub Actions
 ↓
GITHUB_SHA
 ↓
IMAGE_TAG
 ↓
Docker Build
 ↓
devops-app:a81c92d

This tagging strategy provides traceability between the Git commit, Docker image, and deployed application version.