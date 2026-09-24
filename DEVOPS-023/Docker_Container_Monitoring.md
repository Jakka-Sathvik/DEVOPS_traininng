# Docker Container Monitoring

## Objective

Learn how to monitor Docker containers running on an EC2 instance and understand how application logs are generated and accessed through Docker.

---

# Step 1: Connect to EC2

Use SSH to connect to the EC2 instance.

```bash
ssh -i my-key.pem ec2-user@<EC2-Public-IP>
```

Verify connection:

```bash
hostname
```

---

# Step 2: Check Running Containers

List all running containers:

```bash
docker ps
```

Example Output:

```text
CONTAINER ID   IMAGE            COMMAND       CREATED
a1b2c3d4e5f6   myapp:1.0.0      "npm start"   2 days ago

STATUS          PORTS                     NAMES
Up 2 days       0.0.0.0:3000->3000/tcp    restaurant-app
```

---

# Container Information to Inspect

## 1. Container ID

### What is it?

A unique identifier assigned to every Docker container.

Example:

```text
a1b2c3d4e5f6
```

### Why is it important?

- Uniquely identifies a container.
- Used for inspection and troubleshooting.
- Helpful when multiple containers are running.

---

## 2. Container Name

### What is it?

A human-readable name assigned to a container.

Example:

```text
restaurant-app
```

### Why is it important?

- Easier to remember than Container IDs.
- Used in Docker commands.
- Simplifies monitoring and management.

---

## 3. Image

### What is it?

The Docker image from which the container was created.

Example:

```text
myapp
```

### Why is it important?

- Identifies the application package.
- Helps determine what software is running.
- Useful during deployments and rollbacks.

---

## 4. Image Tag

### What is it?

The version associated with a Docker image.

Example:

```text
myapp:1.0.0
```

### Why is it important?

- Tracks application versions.
- Supports controlled deployments.
- Helps identify production releases.

Examples:

```text
myapp:1.0.0
myapp:1.0.1
myapp:2.0.0
myapp:a4c8d9f
```

---

## 5. Status

### What is it?

Shows the current state of the container.

Examples:

```text
Up 2 hours
Up 5 days
Exited (1)
Restarting
```

### Why is it important?

- Indicates whether the application is healthy.
- Helps identify failures quickly.
- Useful for incident troubleshooting.

---

## 6. Ports

### What is it?

Shows how container ports are mapped to host machine ports.

Example:

```text
0.0.0.0:3000->3000/tcp
```

Meaning:

```text
EC2 Port 3000
      ↓
Container Port 3000
```

### Why is it important?

- Confirms application accessibility.
- Helps troubleshoot connectivity issues.
- Verifies deployment configuration.

---

# Step 3: Inspect Container Logs

View container logs:

```bash
docker logs <container-name>
```

Example:

```bash
docker logs restaurant-app
```

Or:

```bash
docker logs a1b2c3d4e5f6
```

---

# Follow Logs in Real Time

```bash
docker logs -f restaurant-app
```

Example Output:

```text
Server started on port 3000
Database connected successfully
GET /api/restaurants 200
GET /api/orders 200
```

---

# Understanding Docker Logging

```text
Application
     ↓
Container
     ↓
stdout / stderr
     ↓
Docker Logs
```

### Explanation

#### Application

The application generates messages during execution.

Examples:

```text
Server Started
User Login Successful
Database Connected
```

#### Container

The application runs inside a Docker container.

#### stdout / stderr

Docker captures output streams:

- stdout → Standard output
- stderr → Error output

Examples:

```text
stdout: Application started
stderr: Database connection failed
```

#### Docker Logs

Docker stores these outputs and makes them available using:

```bash
docker logs <container>
```

---

# Why DevOps Engineers Monitor Docker Logs

- Detect application errors
- Troubleshoot incidents
- Verify successful deployments
- Monitor application health
- Track user requests
- Investigate crashes
- Perform root-cause analysis

---

# Example Incident Investigation

Issue:

```text
Users cannot access the application.
```

Check container:

```bash
docker ps
```

Result:

```text
Container is running.
```

Check logs:

```bash
docker logs restaurant-app
```

Result:

```text
Database connection failed.
```

Root Cause:

```text
Database service unavailable.
```

Resolution:

```text
Restart database service and verify connectivity.
```

---

# Useful Commands

## Running Containers

```bash
docker ps
```

## All Containers

```bash
docker ps -a
```

## Container Details

```bash
docker inspect <container>
```

## View Logs

```bash
docker logs <container>
```

## Follow Logs

```bash
docker logs -f <container>
```

## Last 100 Log Lines

```bash
docker logs --tail 100 <container>
```

---

# Summary

Docker container monitoring helps DevOps engineers verify that applications are running correctly. By inspecting Container ID, Container Name, Image, Image Tag, Status, Ports, and Docker Logs, engineers can quickly identify issues, troubleshoot incidents, and maintain production reliability. Docker logs capture application output from stdout and stderr, making them one of the most important tools for debugging and monitoring containerized applications.