
Docker Knowledge Assessment

Prepared By: Sathvik Jakka
Date: 13-09-2026

Basic Docker
1. What is Docker?

Docker is a containerization platform that allows developers to package applications along with their dependencies, libraries, and configurations into containers. This ensures the application runs consistently across different environments such as development, testing, and production.

2. What is a Container?

A container is a lightweight, isolated environment that runs an application and its dependencies. Containers share the host operating system kernel but operate independently from one another.

3. What is an Image?

A Docker image is a read-only template used to create containers. It contains the application code, runtime, libraries, dependencies, and configuration required to run an application.

4. What is Docker Hub?

Docker Hub is a cloud-based repository where Docker images are stored and shared. Users can download existing images or upload their own custom images for others to use.

5. What is the Difference Between an Image and a Container?
Image	Container
Blueprint or template	Running instance of an image
Read-only	Read-write while running
Used to create containers	Executes the application
Stored on disk	Runs in memory and CPU

Example:

Docker Image → nginx
Docker Container → Running Nginx web server created from that image
Dockerfile
1. What is Dockerfile?

A Dockerfile is a text file that contains instructions used by Docker to build an image automatically. It defines how the image should be created.

2. What Does FROM Do?

The FROM instruction specifies the base image that will be used to build a new Docker image.

Example:

FROM ubuntu:22.04

This starts the image build process using Ubuntu 22.04.

3. What Does COPY Do?

The COPY instruction copies files and directories from the local machine into the Docker image.

Example:

COPY app.py /app/

This copies the file app.py into the /app directory inside the image.

4. What Does RUN Do?

The RUN instruction executes commands during the image build process.

Example:

RUN apt-get update

This command runs while building the image and becomes part of the image layer.

5. What Does CMD Do?

The CMD instruction specifies the default command that runs when a container starts.

Example:

CMD ["python", "app.py"]

When the container starts, it automatically executes the Python application.

Docker Compose
1. What is Docker Compose?

Docker Compose is a tool used to define and manage multi-container Docker applications using a YAML configuration file.

It allows multiple services to be started, stopped, and managed together.

2. What is a Service?

A service in Docker Compose represents a container definition.

Examples:

Web application
Database
Redis cache
API server

Each service has its own configuration such as image, ports, volumes, and environment variables.

3. What is a Compose Network?

A Compose network is an automatically created network that allows services within the same Compose project to communicate with each other.

Example:

Web container can connect to Database container using the service name.
4. Why Use .env?

The .env file stores environment variables separately from application code.

Benefits:

Improves security
Makes configuration easier
Avoids hardcoding values
Supports multiple environments

Example:

DB_HOST=mysql
DB_USER=admin
DB_PASSWORD=password123
5. Why Use .dockerignore?

The .dockerignore file specifies files and folders that should not be copied into the Docker build context.

Benefits:

Smaller image size
Faster image builds
Improved security
Reduced unnecessary files

Example:

.git
node_modules
*.log
Storage
1. What is a Docker Volume?

A Docker volume is a persistent storage mechanism managed by Docker. Volumes allow data to remain available even when containers stop or are recreated.

2. Why Do We Need Volumes?

Volumes are used because container storage is temporary.

Benefits:

Persistent data storage
Data backup capability
Data sharing between containers
Easier container replacement

Common examples:

Database files
Application uploads
Log files
3. What Happens to Container Data When the Container is Removed?

If data is stored only inside the container filesystem, it is deleted when the container is removed.

If a Docker volume is attached, the data remains available even after the container is deleted.

Troubleshooting
1. How Do You Check Container Logs?

Docker provides the docker logs command to view container logs.

Example:

docker logs container_name

Additional examples:

docker logs -f container_name

The -f option continuously streams logs in real time.

2. How Do You Find Why a Container Stopped?

Several methods can be used:

Check Container Status
docker ps -a

This shows whether the container exited and displays the exit code.

Check Logs
docker logs container_name

Logs often reveal application errors or startup failures.

Inspect Container
docker inspect container_name

Provides detailed information about container configuration and state.

Check Exit Code
docker ps -a

Common exit codes:

Exit Code 0 → Successful completion
Exit Code 1 → Application error
Exit Code 137 → Container killed due to memory/resource issues
Summary

This assessment helped reinforce the core Docker concepts including containers, images, Dockerfiles, Docker Compose, storage management, and troubleshooting techniques. Understanding these concepts is important for DevOps Engineers because Docker is widely used for application deployment, scalability, and environment consistency. Practical knowledge of Docker enables efficient application packaging, monitoring, troubleshooting, and management in modern cloud-native environments.