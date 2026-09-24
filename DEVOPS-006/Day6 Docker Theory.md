# DEVOPS-006 — Docker Fundamentals & Containerization
## Task 1 — Understand Why Docker Exists

---

### 1. What is Docker?

Docker is a platform/tool for building, packaging, and running applications inside **containers**. A container bundles an application together with everything it needs to run — code, runtime, system libraries, and configuration — so it behaves the same way no matter where it's deployed (your laptop, a teammate's machine, a test server, or production in the cloud).

---

### 2. What problem does Docker solve?

Before Docker, developers ran into the classic problem: **"It works on my machine!"**

Reasons this happened:
- Different OS versions, library versions, or missing dependencies between environments.
- Manually installing and configuring software on every server was slow and error-prone.
- Running multiple apps on one server could cause dependency conflicts (e.g., App A needs Python 3.8, App B needs Python 3.11).
- Setting up a new dev environment for a new team member took hours or days.

Docker solves this by **packaging the application with its exact environment**, so it runs identically everywhere — no more "works on my machine."

---

### 3. What is a container?

A container is a **lightweight, isolated, running instance of an application**, created from an image. It packages the app's code + dependencies + environment, and runs as an isolated process on the host OS.

Key traits:
- Shares the host machine's OS kernel (unlike a VM, which has its own OS).
- Starts in seconds.
- Isolated from other containers (separate filesystem, network, process space).
- Lightweight — usually megabytes, not gigabytes.

---

### 4. What is a Docker image?

A Docker image is a **read-only template/blueprint** used to create containers. It contains the application code, runtime, libraries, environment variables, and configuration files needed to run the app.

- Think of the **image** as the "recipe" and the **container** as the "dish" made from that recipe.
- One image can be used to spin up many identical containers.
- Images are built in layers, and each layer is cached, making builds efficient.

---

### 5. What is a Dockerfile?

A Dockerfile is a **plain text file containing step-by-step instructions** on how to build a Docker image.

Example of typical instructions it contains:
- `FROM` – base image to start from (e.g., `node:18`)
- `WORKDIR` – sets the working directory inside the container
- `COPY` – copies files from your machine into the image
- `RUN` – executes commands during the build (e.g., install dependencies)
- `EXPOSE` – documents which port the app uses
- `CMD` – the command that runs when the container starts

Docker reads this file top-to-bottom and builds the image accordingly.

---

### 6. What is Docker Hub?

Docker Hub is a **cloud-based registry** (like GitHub, but for Docker images) where you can:
- Store and share your own Docker images (public or private).
- Pull official/pre-built images (e.g., `nginx`, `mysql`, `python`, `node`) instead of building from scratch.
- Version images using tags (e.g., `python:3.11`, `node:18-alpine`).

It's the default registry Docker uses when you run `docker pull` or `docker push`.

---

### 7. Difference between a Virtual Machine and a Docker Container

| Aspect | Virtual Machine | Docker Container |
|---|---|---|
| **OS** | Each VM has its own full guest OS | Shares the host machine's OS kernel |
| **Size** | Large (GBs) | Small (MBs) |
| **Boot time** | Minutes | Seconds |
| **Resource usage** | Heavy — allocates fixed CPU/RAM | Lightweight — shares host resources |
| **Isolation level** | Full hardware-level isolation (via hypervisor) | Process-level isolation |
| **Portability** | Less portable, tied to hypervisor | Highly portable across environments |
| **Use case** | Running multiple full OS environments | Running multiple lightweight, isolated apps |

**In short:** A VM virtualizes the entire hardware + OS. A container virtualizes only the application layer, sitting on top of a shared OS — making it much faster and lighter.

---

## Diagram: Traditional Application vs Dockerized Application

**Traditional Application Stack:**
```
Application
    ↓
Operating System
    ↓
Server
```

**Dockerized Application Stack:**
```
Docker Application

Application
    ↓
Container
    ↓
Docker Engine
    ↓
Operating System
    ↓
Server
```

**Key insight:** In the traditional model, the application is installed directly onto the OS — tightly coupling it to that specific server's setup. In the Dockerized model, the application runs inside a **Container**, managed by the **Docker Engine**, which sits on top of the OS. This adds a layer of abstraction that makes the app portable and isolated from the underlying server's specific configuration.

---
*Deliverable: Day6 Docker Theory.md — Sprint 9 (29/6 - 12/7), DEVOPS-006*
