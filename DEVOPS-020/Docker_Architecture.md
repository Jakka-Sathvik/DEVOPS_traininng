# Docker: Workflow & Architecture

## Overview

Docker packages an application together with everything it needs to run — code, runtime, libraries, settings — into a portable unit that behaves the same on any machine. This document covers two views of how that works: the **workflow** a developer follows to go from source code to a running app, and the **architecture** — the components that actually carry that workflow out.

---

## 1. Developer Workflow

```
Developer
    ↓
Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
docker run
    ↓
Container
    ↓
Application
```

**Developer** — writes the application code and decides how it should be packaged: what base environment it needs, what dependencies to install, how it should start.

**Dockerfile** — a plain-text set of instructions describing how to build an image. Typical lines:
- `FROM` — the base image to start from (e.g. `node:20-alpine`)
- `WORKDIR` — sets the working directory inside the image
- `COPY` — copies files from the project into the image
- `RUN` — runs a command at build time (e.g. installing dependencies)
- `CMD` / `ENTRYPOINT` — the command that runs when a container starts

**docker build** — reads the Dockerfile top to bottom and executes each instruction, producing an image.
```bash
docker build -t myapp:1.0 .
```
Each instruction generally creates a cached layer, so a small code change rebuilds fast — Docker reuses every layer above until it hits the one that changed.

**Docker Image** — a read-only, portable snapshot containing the app, its dependencies, a minimal OS layer, and configuration (env vars, exposed ports, default command). Images don't run by themselves and don't change once built. They can be stored and shared through a registry such as Docker Hub.

**docker run** — takes an image and starts a container from it.
```bash
docker run -d -p 8080:80 myapp:1.0
```

**Container** — a running instance of an image: the image's read-only layers plus a thin writable layer on top, isolated from the host and other containers with its own filesystem and process space. Many containers can run from the same image at once, each isolated from the others.

**Application** — the actual process executing inside the container: the web server, API, worker, or script the whole pipeline exists to run.

> **In short:** describe how to build the app (Dockerfile) → build it into a reusable artifact (Image) → run instances of that artifact (Containers) that execute the Application.

---

## 2. Docker Architecture

```
Docker Client
      ↓
Docker Engine
      ↓
    Images
      ↓
  Containers
```

**Docker Client** — the `docker` CLI (or a GUI like Docker Desktop) you actually type commands into. The client doesn't do the work itself: when you run `docker build` or `docker run`, it sends that command to the Docker Engine over a REST API, usually via a local Unix socket.

**Docker Engine** — the background service (`dockerd`, the Docker daemon) that does the real work: building images, starting and stopping containers, managing networks and volumes, and pulling or pushing images to a registry. It listens for requests from the client and carries them out.

**Images** — built and stored by the Engine on disk, as stacked read-only layers. This layer structure is what makes builds and storage efficient — identical layers are shared and cached rather than duplicated.

**Containers** — also created and managed by the Engine, at runtime, from an image. The Engine sets up the isolated namespaces (filesystem, process, network) and resource limits that make a container behave like a lightweight, self-contained machine.

> **Note on the diagram:** it's drawn as a straight chain, but Images and Containers are really two things the Engine manages side by side, with Containers always created *from* an Image rather than simply following after it. Read the arrows as "the Client talks to the Engine, and the Engine operates on Images and Containers," rather than four strictly separate, sequential stages.

---

## 3. How the Two Diagrams Fit Together

The first diagram is the workflow you follow. The second is the system carrying it out underneath. Every `docker` command in the workflow actually resolves to Client → Engine:

```
docker build   ──►  Docker Client  ──►  Docker Engine  ──►  builds and stores an  Image
docker run     ──►  Docker Client  ──►  Docker Engine  ──►  creates and starts a   Container
```

So the full picture reads:

```
Developer writes a Dockerfile
        ↓
"docker build" is typed into the Docker Client
        ↓
Docker Client sends the request to the Docker Engine
        ↓
Docker Engine executes the Dockerfile and produces a Docker Image
        ↓
"docker run" is typed into the Docker Client
        ↓
Docker Client sends the request to the Docker Engine
        ↓
Docker Engine starts a Container from that Image
        ↓
The Application runs inside the Container
```

---

## 4. Image vs. Container, at a Glance

| | Image | Container |
|---|---|---|
| What it is | A blueprint / template | A running instance of that blueprint |
| State | Read-only | Read-only layers + a writable layer |
| Created by | `docker build` | `docker run` (from an image) |
| Good analogy | A class, or a recipe | An object, or the dish you cooked |
| Can there be many? | One image, reused everywhere | Many containers from one image, each isolated |

---

## 5. Command Quick Reference

| Command | What it does |
|---|---|
| `docker build -t name:tag .` | Build an image from a Dockerfile in the current directory |
| `docker images` | List images stored locally |
| `docker run image` | Create and start a container from an image |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers, including stopped ones |
| `docker stop container` | Stop a running container |
| `docker rm container` | Remove a stopped container |
| `docker rmi image` | Remove an image |
