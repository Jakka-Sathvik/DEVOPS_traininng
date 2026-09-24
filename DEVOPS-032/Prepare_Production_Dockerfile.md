TASK 2 — Prepare Production Dockerfile (Step-by-Step Guide)
Objective

Create a production-ready Docker image, build it locally, run it as a container, and verify that the application is working.

Step 1: Understand the Project Structure

Assume your application looks like this:

devops-app/
│
├── app.py
├── requirements.txt
└── Dockerfile

Example:

devops-app/
│
├── app.py
├── requirements.txt
└── Dockerfile
Step 2: Understand Each Dockerfile Instruction

Create a file named:

Dockerfile

Content:

FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
1. FROM (Base Image)
FROM python:3.12-slim
What is it?

Every Docker image starts from another image.

Architecture:

Python Runtime
      │
      ▼
python:3.12-slim
      │
      ▼
Our Application
Why use slim?

Smaller image size.

Example:

python:3.12         → Large
python:3.12-slim    → Smaller

Smaller images:

Download faster
Start faster
Use less storage
2. WORKDIR
WORKDIR /app
What is it?

Creates and sets the working directory inside the container.

Without WORKDIR:

/

With WORKDIR:

/app

Architecture:

Container
    │
    ▼
/app

All future commands run from:

/app
3. COPY requirements.txt
COPY requirements.txt .
What does it do?

Copies the file from:

Local Machine
      │
      ▼
requirements.txt

To:

Container
      │
      ▼
/app/requirements.txt

Example:

Flask==3.0.0
4. RUN
RUN pip install --no-cache-dir -r requirements.txt
What does it do?

Installs required Python packages.

Architecture:

requirements.txt
         │
         ▼
pip install
         │
         ▼
Packages Installed

Example:

Flask
Requests
Pytest
Why --no-cache-dir?

Reduces image size.

Without:

Package + Cache Files

With:

Package Only
5. COPY Application Files
COPY . .
What does it do?

Copies all application files.

From:

Local Project

To:

/app

Architecture:

app.py
Dockerfile
requirements.txt
      │
      ▼
Container /app
6. EXPOSE
EXPOSE 8000
What does it do?

Documents that the application uses port:

8000

Architecture:

Container
     │
     ▼
Port 8000

Example:

Flask App

Listening on:

app.run(host="0.0.0.0", port=8000)
7. CMD
CMD ["python", "app.py"]
What does it do?

Starts the application when container starts.

Architecture:

Container Starts
       │
       ▼
python app.py
       │
       ▼
Application Running
Step 3: Understand Docker Layer Caching

Docker builds images layer by layer.

Architecture:

Layer 1 → FROM
Layer 2 → WORKDIR
Layer 3 → COPY requirements.txt
Layer 4 → RUN pip install
Layer 5 → COPY .
Layer 6 → CMD
Why copy requirements first?

Good:

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

If only code changes:

Reuse Layer 4
Skip Reinstall Packages

Build becomes faster.

Step 4: Create .dockerignore

Create:

.dockerignore

Example:

.git
__pycache__
*.pyc
.env
venv
Why?

Avoid sending unnecessary files.

Benefits:

Faster builds
Smaller images
Better security
Step 5: Open Terminal

Move to project folder:

cd devops-app

Verify:

pwd

Linux Example:

/home/ubuntu/devops-app

List files:

ls

Expected:

app.py
requirements.txt
Dockerfile
Step 6: Build Docker Image

Run:

docker build -t devops-app:v1 .

Breakdown:

docker build

Build image.

-t

Tag image.

devops-app:v1

Image name and version.

.

Current directory.

Architecture:

Dockerfile
      │
      ▼
Docker Build
      │
      ▼
devops-app:v1
Step 7: Verify Image

Check image:

docker images

Example:

REPOSITORY      TAG
devops-app      v1
Step 8: Run Container

Start container:

docker run -d -p 8000:8000 devops-app:v1

Breakdown:

-d
Detached Mode

Container runs in background.

-p
Host:Container

Meaning:

8000:8000

Architecture:

Browser
   │
localhost:8000
   │
   ▼
Container Port 8000
Step 9: Verify Running Container

Check:

docker ps

Expected:

CONTAINER ID
IMAGE
PORTS
STATUS

Example:

0.0.0.0:8000->8000/tcp
Step 10: Test Application

Open browser:

http://localhost:8000

Expected:

Hello World

or your application page.

Architecture:

Browser
    │
    ▼
localhost:8000
    │
    ▼
Docker Container
    │
    ▼
Application
Step 11: Check Logs

If application does not work:

docker logs <container-id>

Find container:

docker ps

Example:

docker logs a1b2c3d4
Step 12: Stop Container

List running containers:

docker ps

Stop:

docker stop <container-id>

Example:

docker stop a1b2c3d4
Step 13: Remove Container
docker rm <container-id>

Example:

docker rm a1b2c3d4
Production Dockerfile (Recommended Version)
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
Complete Flow
Application Code
       │
       ▼
Dockerfile
       │
       ▼
docker build
       │
       ▼
Docker Image
(devops-app:v1)
       │
       ▼
docker run
       │
       ▼
Container
       │
       ▼
Port 8000
       │
       ▼
Browser
       │
       ▼
http://localhost:8000