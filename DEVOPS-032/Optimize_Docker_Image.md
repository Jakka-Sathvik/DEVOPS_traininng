Task 3 – Optimize Docker Image
Objective

Docker image optimization is the process of reducing image size, improving build speed, minimizing security risks, and creating production-ready container images.

A smaller image:

Downloads faster
Deploys faster
Uses less storage
Has a smaller attack surface
Improves CI/CD performance
Why Optimize Docker Images?

Without optimization:

Large Image
     │
     ▼
Slow Build
Slow Push
Slow Pull
More Storage
More Vulnerabilities

With optimization:

Optimized Image
      │
      ▼
Fast Build
Fast Deployment
Less Storage
Improved Security
Common Reasons Images Become Large

Examples:

Unnecessary Files
Git Repository
Cache Files
Virtual Environments
Logs
Unused Packages
Large Base Images

These files increase image size without helping the application run.

Docker Optimization Techniques
1. Use Smaller Base Images
Poor Example
FROM python:3.12

Full Python images contain many packages that may not be required.

Better Example
FROM python:3.12-slim
Architecture
python:3.12
      │
      ▼
Large Image
python:3.12-slim
      │
      ▼
Smaller Image
Benefits
Reduced storage
Faster downloads
Faster deployment
2. Use .dockerignore
What is .dockerignore?

.dockerignore prevents unnecessary files from being copied into the Docker build context.

Without .dockerignore:

Project Files
      │
      ▼
Everything Sent To Docker

With .dockerignore:

Project Files
      │
      ▼
Ignore Unnecessary Files
      │
      ▼
Smaller Build Context
Create .dockerignore

File:

.dockerignore

Content:

.git
.github
__pycache__
*.pyc
.env
venv
node_modules
README.md
Explanation of Each Entry
.git
Git History

Not required inside containers.

.github
GitHub Actions Workflows

Used only for CI/CD.

pycache
Python Cache Files

Automatically generated.

*.pyc
Compiled Python Files

Can be recreated when needed.

.env
Environment Variables

Should not be stored inside images.

venv
Local Virtual Environment

Docker installs dependencies separately.

node_modules
Node.js Dependencies

Should be rebuilt during image creation.

README.md
Documentation

Not required for application execution.

3. Dependency Caching

Docker builds images layer by layer.

Example:

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .
Why This Order Matters
Good Practice
COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

Architecture:

requirements.txt
        │
        ▼
Install Dependencies
        │
        ▼
Copy Application

If only source code changes:

Reuse Dependency Layer

Docker skips reinstalling packages.

Bad Practice
COPY . .

RUN pip install -r requirements.txt

Any file change forces:

Reinstall All Packages

Result:

Slower Builds
4. Remove Unnecessary Cache Files

Example:

RUN pip install --no-cache-dir -r requirements.txt

Without:

Package Files
+
Cache Files

With:

Package Files Only

Benefits:

Smaller image
Reduced storage
5. Multi-Stage Builds
What is a Multi-Stage Build?

A multi-stage build separates build tools from the final runtime image.

Architecture:

Build Stage
     │
     ▼
Compile Application
     │
     ▼
Runtime Stage
     │
     ▼
Copy Required Files Only
Example Multi-Stage Dockerfile
# Stage 1 - Build

FROM python:3.12-slim AS builder

WORKDIR /app

COPY requirements.txt .

RUN pip install --prefix=/install \
    --no-cache-dir \
    -r requirements.txt

# Stage 2 - Runtime

FROM python:3.12-slim

WORKDIR /app

COPY --from=builder /install /usr/local

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
Why Multi-Stage Builds?

Without multi-stage:

Build Tools
Dependencies
Temporary Files
Application

Everything remains inside image.

With multi-stage:

Application
Required Libraries
Only Runtime Files

Benefits:

Smaller image
Cleaner image
Improved security
6. Reduce Layers
Poor Example
RUN apt update

RUN apt install -y curl

RUN apt clean

Creates:

Layer 1
Layer 2
Layer 3
Better Example
RUN apt update && \
    apt install -y curl && \
    apt clean

Creates:

Single Layer

Benefits:

Smaller image
Faster build
Production Optimized Dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
Build the Original Image
docker build -t devops-app:v1 .

Check size:

docker images

Example:

REPOSITORY     TAG     SIZE
devops-app     v1      420MB
Apply Optimization
Create .dockerignore
Use slim base image
Enable dependency caching
Remove cache files
Use multi-stage build if required

Build again:

docker build -t devops-app:v2 .
Compare Image Sizes

Check:

docker images

Example:

REPOSITORY     TAG     SIZE
devops-app     v1      420MB
devops-app     v2      185MB
Record Results
Before Optimization
Image Name: devops-app:v1
Size: 420MB
After Optimization
Image Name: devops-app:v2
Size: 185MB
Size Reduction
420MB - 185MB = 235MB

Percentage:

(235 / 420) × 100
= 55.95%
≈ 56%
Optimization Workflow
Application
      │
      ▼
Create .dockerignore
      │
      ▼
Use Slim Base Image
      │
      ▼
Optimize Dependency Installation
      │
      ▼
Remove Cache Files
      │
      ▼
Multi-Stage Build
      │
      ▼
Build Optimized Image
      │
      ▼
Compare Sizes
Best Practices
Always

✅ Use slim or lightweight base images

✅ Create .dockerignore

✅ Install dependencies before copying application code

✅ Use --no-cache-dir

✅ Remove unnecessary files

✅ Scan images for vulnerabilities

✅ Use multi-stage builds for production

Avoid

❌ Copying .git

❌ Copying secrets

❌ Including local virtual environments

❌ Using oversized base images

❌ Installing unnecessary packages

❌ Leaving cache files inside images