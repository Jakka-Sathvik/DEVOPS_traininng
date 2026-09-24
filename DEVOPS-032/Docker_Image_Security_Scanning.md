TASK 6 — Docker Image Security Scanning
Objective

Add Docker image security scanning to the CI pipeline.

The pipeline should:

Checkout source code
Create the Docker image
Scan the image
Fail when a critical vulnerability is found
Allow the pipeline to continue only when the security check passes
Why Docker Image Scanning Is Important

A Docker image contains more than application code.

It can contain:

Base OS
System Libraries
Python/Node/Java Runtime
Application Dependencies
Application Code

Some of these components may contain known security vulnerabilities.

Without scanning:

Docker Build
     ↓
Vulnerable Image
     ↓
Push to Registry
     ↓
Deploy

With scanning:

Docker Build
     ↓
Security Scan
     ↓
   ┌───┴───┐
   ↓       ↓
 PASS     FAIL
   ↓       ↓
Push     Stop
ECR      Pipeline
What Does a Security Scanner Detect?

A container scanner can identify known vulnerabilities in components such as:

1. Operating System Packages

Example:

Debian
Ubuntu
Alpine

The scanner checks installed OS packages against vulnerability databases.

2. Application Dependencies

For a Python application:

Flask
Requests
Django

For Node.js:

express
react
lodash

The scanner can identify known vulnerabilities in supported dependency ecosystems.

3. Critical CVEs

CVE means Common Vulnerabilities and Exposures.

A CVE identifies a publicly known security vulnerability.

Example concept:

Package
   ↓
Known Vulnerability
   ↓
CVE
   ↓
Severity
Severity Levels

Vulnerability scanners commonly categorize findings by severity.

A simplified model is:

UNKNOWN
LOW
MEDIUM
HIGH
CRITICAL

For this task:

CRITICAL
    ↓
Pipeline FAILED
Tool Used: Trivy

Trivy is a vulnerability scanner commonly used for container images and other software artifacts.

Our pipeline will use the Trivy GitHub Action.

Architecture:

GitHub Actions
      │
      ▼
Docker Build
      │
      ▼
devops-app:<commit-sha>
      │
      ▼
Trivy Scan
      │
 ┌────┴─────┐
 │          │
PASS       FAIL
 │          │
 ▼          ▼
Continue   Stop
 │
 ▼
ECR Push
Step 1 — Existing Docker Build Pipeline

The previous task created:

.github/workflows/docker-build.yml

The basic pipeline looked like:

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

Now we add the security scan.

Step 2 — Add Trivy Scan

Add the following step after Docker image creation:

- name: Scan Docker Image
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: devops-app:${{ env.IMAGE_TAG }}
    format: table
    exit-code: '1'
    ignore-unfixed: true
    severity: CRITICAL
Understanding the Configuration
image-ref
image-ref: devops-app:${{ env.IMAGE_TAG }}

This tells Trivy which image to scan.

Example:

devops-app:a81c92d
format
format: table

Displays scan results in a readable table.

Example concept:

Library     Vulnerability    Severity
openssl     CVE-XXXX         HIGH
package-x   CVE-YYYY         CRITICAL
exit-code
exit-code: '1'

This is extremely important.

When Trivy finds a vulnerability matching the configured severity threshold, it returns exit code 1.

GitHub Actions interprets this as a failed step.

Therefore:

Critical Finding
       ↓
Trivy exit code 1
       ↓
GitHub Actions step fails
       ↓
Job fails
       ↓
Later steps do not run
severity
severity: CRITICAL

For this training task, we are checking for critical vulnerabilities.

The required rule is:

CRITICAL
    ↓
FAIL PIPELINE
ignore-unfixed
ignore-unfixed: true

This tells Trivy to ignore vulnerabilities for which no fix is currently available.

This is a policy choice, not a universal requirement.

For a stricter production security policy, your organization may choose to report or fail on unfixed vulnerabilities as well.

Step 3 — Complete Workflow

Your .github/workflows/docker-build.yml can now be:

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

      - name: Scan Docker Image
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: devops-app:${{ env.IMAGE_TAG }}
          format: table
          exit-code: '1'
          ignore-unfixed: true
          severity: CRITICAL
Step 4 — Understand the Complete Pipeline

The workflow now performs:

Git Push
    │
    ▼
GitHub Actions
    │
    ▼
Checkout Code
    │
    ▼
Generate Image Tag
    │
    ▼
Docker Build
    │
    ▼
Docker Image
    │
    ▼
Trivy Security Scan
    │
 ┌──┴────┐
 │       │
PASS    CRITICAL
 │       │
 ▼       ▼
Continue FAIL
         │
         ▼
     Stop Pipeline
Step 5 — Commit the Workflow

Check your files:

git status

You should see:

.github/workflows/docker-build.yml

Add the change:

git add .github/workflows/docker-build.yml

Commit:

git commit -m "Add Docker image security scanning"

Push:

git push origin main
Step 6 — Check GitHub Actions

Open your GitHub repository.

Go to:

Actions
   ↓
Docker Build

You should see:

✓ Checkout Source Code
✓ Set Image Tag
✓ Build Docker Image
✓ Scan Docker Image

If the scan passes:

SCAN PASSED
     ↓
JOB SUCCESS
Step 7 — Understand a Failed Scan

Suppose the scanner finds:

CRITICAL vulnerability

The process becomes:

Docker Build
     ↓
Trivy Scan
     ↓
CRITICAL vulnerability
     ↓
exit-code = 1
     ↓
GitHub Actions fails
     ↓
Pipeline stops

The image should not proceed to the ECR push step.

This creates a security gate:

Security Scan
      │
      ▼
Security Gate
      │
 ┌────┴────┐
 │         │
 PASS     FAIL
 │         │
 ▼         ▼
ECR       STOP
Step 8 — Why Scan Before ECR Push?

The desired order is:

Build
  ↓
Scan
  ↓
Push ECR

Not:

Build
  ↓
Push ECR
  ↓
Scan

Scanning before pushing prevents the pipeline from intentionally publishing an image that violates the defined security policy.

Important Note About Image Scanning

A scanner does not prove that an image is completely secure.

It primarily identifies known issues based on available vulnerability information and scanner capabilities.

For example:

Scanner
   │
   ├── OS vulnerabilities
   ├── Dependency vulnerabilities
   └── Known CVEs

It does not replace:

Secure coding
Dependency management
Secret management
Access control
Network security
Runtime security
Application security testing
Optional Local Test

If Trivy is installed on your local machine, you can scan the image yourself:

trivy image devops-app:v1

To focus on critical vulnerabilities:

trivy image --severity CRITICAL devops-app:v1

This helps you understand the same security concept before running it through GitHub Actions.

Final CI/CD Architecture

After Task 6, the architecture becomes:

Developer
    │
    ▼
GitHub
    │
    ▼
GitHub Actions
    │
    ▼
Checkout
    │
    ▼
Generate Commit SHA Tag
    │
    ▼
Docker Build
    │
    ▼
Docker Image
    │
    ▼
Trivy Security Scan
    │
    ├───────────────┐
    │               │
    ▼               ▼
  PASS            CRITICAL
    │               │
    ▼               ▼
Continue           FAIL
    │               │
    ▼               ▼
Push ECR          Stop Pipeline
Task 6 Deliverable

The required deliverable is:

.github/workflows/docker-build.yml

The pipeline must contain:

✓ Docker image build
✓ Image tagging
✓ Security scanning
✓ CRITICAL vulnerability failure
✓ Scan before ECR push

The resulting CI flow is:

Code
 ↓
GitHub
 ↓
GitHub Actions
 ↓
Docker Build
 ↓
Docker Image
 ↓
Trivy Security Scan
 ↓
PASS ─────────→ ECR Push
 │
 └─ CRITICAL ─→ Pipeline FAILED