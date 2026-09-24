# Day 21 - Continuous Deployment (CD) Architecture

## Architecture Diagram

```text
                    Developer
                        │
                        ↓
                     GitHub
                        │
                        ↓
                 GitHub Actions
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
            Tests            Docker Build
                                  │
                                  ↓
                         Docker Registry
                                  │
                                  ↓
                                EC2
                                  │
                                  ↓
                          Docker Container
                                  │
                                  ↓
                             Application
```

---

# Component Explanation

## 1. Developer

The developer writes code, fixes bugs, and adds new features to the application. After completing the changes, the code is pushed to a GitHub repository.

**Responsibilities:**
- Write application code
- Fix bugs and implement features
- Commit and push changes to GitHub

---

## 2. GitHub

GitHub is the source code management platform where the application code is stored.

**Responsibilities:**
- Store source code
- Maintain version control
- Trigger CI/CD workflows when code changes occur

**Benefits:**
- Collaboration among team members
- Code history tracking
- Branch management

---

## 3. GitHub Actions

GitHub Actions is the automation service that executes the CI/CD pipeline whenever code is pushed.

**Responsibilities:**
- Run automated workflows
- Execute tests
- Build Docker images
- Deploy applications automatically

**Benefits:**
- Reduces manual deployment effort
- Ensures consistent deployments
- Improves development speed

---

## 4. Tests

Automated tests validate the application before deployment.

**Types of Tests:**
- Unit Tests
- Integration Tests
- Security Checks
- Linting

**Purpose:**
- Detect bugs early
- Ensure application quality
- Prevent faulty deployments

---

## 5. Docker Build

After successful testing, GitHub Actions builds a Docker image containing the application and its dependencies.

**Responsibilities:**
- Package application code
- Include runtime dependencies
- Create a portable deployment artifact

**Benefits:**
- Consistent environment across systems
- Easy deployment
- Faster scaling

---

## 6. Docker Registry

The Docker Registry stores Docker images for deployment.

**Examples:**
- Docker Hub
- Amazon ECR
- GitHub Container Registry

**Responsibilities:**
- Store versioned images
- Provide secure image distribution
- Maintain deployment artifacts

---

## 7. Amazon EC2

Amazon EC2 (Elastic Compute Cloud) provides virtual servers for hosting applications.

**Responsibilities:**
- Host application workloads
- Pull Docker images from registry
- Run containers

**Benefits:**
- Scalable infrastructure
- High availability
- Flexible resource allocation

---

## 8. Docker Container

The Docker container is a running instance of the Docker image.

**Responsibilities:**
- Execute application processes
- Isolate application environment
- Provide runtime consistency

**Benefits:**
- Lightweight deployment
- Faster startup times
- Easy portability

---

## 9. Application

The final deployed application that users access.

**Responsibilities:**
- Serve user requests
- Provide business functionality
- Deliver application services

**Outcome:**
Users can access the latest version of the application after every successful deployment.

---

# Deployment Flow

1. Developer pushes code to GitHub.
2. GitHub triggers GitHub Actions workflow.
3. Automated tests are executed.
4. Docker image is built.
5. Docker image is pushed to Docker Registry.
6. EC2 server pulls the latest image.
7. Docker container is started or updated.
8. Updated application becomes available to users.

---

# Summary

This Continuous Deployment (CD) architecture automates the software delivery process using GitHub Actions, Docker, Docker Registry, and Amazon EC2. The pipeline ensures that tested code is automatically packaged, deployed, and made available to users with minimal manual intervention.