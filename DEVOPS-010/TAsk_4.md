The goal is to create a Docker image that is small, clean, secure, and production-oriented, then build it as:

devops-final:v1
1. Expected project structure

From your project root:

devops-docker-final-project/
├── app/
│   ├── app.py
│   └── requirements.txt
├── Dockerfile
├── .dockerignore
├── .gitignore
├── .env.example
├── docker-compose.yml
├── README.md
├── scripts/
└── docs/
2. Create the Dockerfile

From the project root:

cd devops-docker-final-project
touch Dockerfile

Open it:

nano Dockerfile

Put this inside:

# Use a small official Python image
FROM python:3.12-slim

# Create a non-root user
RUN useradd --create-home --shell /bin/bash appuser

# Set working directory
WORKDIR /app

# Copy only dependency file first
COPY app/requirements.txt .

# Install required Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application files
COPY app/ .

# Change ownership to non-root user
RUN chown -R appuser:appuser /app

# Run application as non-root user
USER appuser

# Application port
EXPOSE 5000

# Start Flask application
CMD ["python", "app.py"]

Save:

Ctrl + O
Enter
Ctrl + X
3. Understand each section
Base Image
FROM python:3.12-slim

We use python:3.12-slim because:

Python is required by our Flask application.
slim is smaller than the full Python image.
It contains the necessary runtime without many unnecessary packages.
Create a non-root user
RUN useradd --create-home --shell /bin/bash appuser

By default, Docker containers often run as root.

Running the application as a dedicated user is better from a security perspective.

Working directory
WORKDIR /app

The application will run from:

/app
Install dependencies
COPY app/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

Our requirements.txt contains:

Flask

The --no-cache-dir option avoids keeping pip's package cache in the image, helping reduce image size.

Copy application
COPY app/ .

This copies:

app/app.py

into:

/app/app.py

We don't copy unnecessary project files into the image.

Non-root ownership
RUN chown -R appuser:appuser /app

This gives our application user ownership of the application directory.

Then:

USER appuser

means the application runs as:

appuser

instead of:

root
Port
EXPOSE 5000

Our Flask application listens on:

5000

Remember: EXPOSE documents the container port; it does not itself publish the port to your host.

Startup command
CMD ["python", "app.py"]

When the container starts, Docker executes:

python app.py

Our Flask application then starts on:

0.0.0.0:5000
4. Create .dockerignore

This is important because the task says:

Avoid unnecessary files.

Create:

nano .dockerignore

Add:

.git
.gitignore
__pycache__
*.pyc
*.pyo
*.pyd
.venv
venv
.env
.pytest_cache
README.md
docs
scripts

This prevents unnecessary files from being sent to Docker during the build.

Check it:

cat .dockerignore
5. Verify the Dockerfile

Run:

cat Dockerfile

You should see:

FROM python:3.12-slim

RUN useradd --create-home --shell /bin/bash appuser

WORKDIR /app

COPY app/requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app/ .

RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 5000

CMD ["python", "app.py"]
6. Build the Docker image

This is the main requirement of ASK 4.

From the project root:

docker build -t devops-final:v1 .
What does this mean?
docker build

Build a Docker image.

-t devops-final:v1

Give the image:

Repository: devops-final
Tag:        v1
.

Use the current directory as the build context.

7. Check the image

Run:

docker images

You should see something similar to:

REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
devops-final    v1        abc123456789   10 seconds ago   ...

Your image is now:

devops-final:v1
8. Run the container

Run:

docker run -d \
  --name devops-final \
  -p 5000:5000 \
  devops-final:v1

Check:

docker ps

Expected:

CONTAINER ID   IMAGE              STATUS         PORTS
xxxxxxxxxxxx   devops-final:v1    Up ...         0.0.0.0:5000->5000/tcp
9. Check application logs

Run:

docker logs devops-final

You should see Flask starting, similar to:

* Serving Flask app 'app'
* Running on all addresses (0.0.0.0)
* Running on http://127.0.0.1:5000
* Running on http://172.x.x.x:5000
10. Test the application

Test the health endpoint:

curl http://localhost:5000/health

Expected:

{
  "application": "running",
  "database": "connected",
  "status": "success"
}

Test the main page:

curl http://localhost:5000/

You should get the HTML containing:

DevOps Training Application
Application Status: Running
Environment: Development
Database: Connected
If you're using GitHub Codespaces

Open the Ports tab and find port:

5000

Then open the forwarded port in your browser.

11. Verify that the container is not running as root

This is a useful production-style check.

Run:

docker exec devops-final whoami

Expected:

appuser

This demonstrates that your application is not running as root.

You can also check:

docker exec devops-final id

You should see a user corresponding to appuser.

12. Check the image history

Run:

docker history devops-final:v1

This helps you understand the Docker image layers.

13. Final verification

Run these commands:

docker images
docker ps
docker logs devops-final
curl http://localhost:5000/health
docker exec devops-final whoami

You should have:

Image:
devops-final:v1

Container:
devops-final

Application:
Running

Port:
5000

Health:
HTTP 200

User:
appuser
14. Commit your work

After successful testing:

git status

Then:

git add Dockerfile .dockerignore

Commit:

git commit -m "Created production-style Dockerfile"

Push:

git push
15. ASK 4 deliverables

For your training submission, capture screenshots of:

Dockerfile
.dockerignore

Successful build:

docker build -t devops-final:v1 .

Image:

docker images

Running container:

docker ps

Application health:

curl http://localhost:5000/health

Non-root verification:

docker exec devops-final whoami
Final architecture
Dockerfile
    │
    ├── Python 3.12 Slim
    │
    ├── Install Flask
    │
    ├── Copy application
    │
    ├── Run as appuser
    │
    └── Port 5000
          │
          ▼
    devops-final:v1
          │
          ▼
    Docker Container
          │
          ▼
    Flask Application
          │
          ├── /
          │
          └── /health

Important: Your current /health endpoint reports "database": "connected" as part of the demo application's static response. It does not mean an actual database is running yet.