 DEVOPS-009 — Docker Production Practices & Container Troubleshooting**

---
 1. What is a Docker image?

A Docker image is a read-only, immutable template containing everything an application needs to run: code, runtime, system libraries, dependencies, and configuration.

- Built in layers — each Dockerfile instruction (`FROM`, `RUN`, `COPY`, …) adds a layer, and layers are cached/reused to speed up builds.
- An image never runs by itself — it's a blueprint. `docker run` creates a container *from* an image.
- Identified by `name:tag` (e.g. `nginx:1.25`) and stored in registries like Docker Hub.

 2. What is a container?

A container is a running instance of an image  an isolated, lightweight process with its own filesystem, network stack, and process space.

-Shares the host's OS kernel (unlike a VM, which virtualizes a whole guest OS), so it starts in milliseconds and uses far fewer resources.
- Isolation comes from Linux namespaces (process/network/mount isolation) and cgroups (CPU/memory limits).
- On top of the image's read-only layers, Docker adds a thin writable layer for anything the running process creates or changes.

3. What is a Dockerfile?

A Dockerfile is a plain-text set of build instructions that `docker build` reads to produce an image.

| Instruction | Purpose |
|---|---|
| `FROM` | Base image to start from |
| `RUN` | Executes a command at build time (e.g. install packages) |
| `COPY` / `ADD` | Copies files from the host into the image |
| `WORKDIR` | Sets the working directory for later instructions |
| `ENV` | Sets environment variables |
| `EXPOSE` | Documents the port the container listens on |
| `CMD` / `ENTRYPOINT` | Default command run when a container starts |

 4. What is Docker Compose?

Docker Compose defines and runs multi-container applications from a single YAML file (`docker-compose.yml`) instead of chaining multiple `docker run` commands by hand.

- Declares services, networks, and volumes together in one place.
- `docker compose up` builds/starts the whole stack; `docker compose down` tears it down.
- Typical use: an app + database + cache (e.g. web server + Postgres + Redis) started as one unit — very common for local dev environments.

 5. What is a Docker network?

A Docker network lets containers talk to each other and to the outside world. Main driver types:

- bridge (default) — private, isolated network on one host; containers on the same *custom* bridge can resolve each other by container name via Docker's built-in DNS.
- host — container shares the host's network stack directly, no isolation.
- none — no networking.
- overlay — connects containers across multiple hosts (used in Swarm/clusters).

 6. What is a Docker volume?

A Docker volume is persistent storage managed by Docker, kept outside any single container's lifecycle.

- Lives in a location Docker manages on the host (`/var/lib/docker/volumes/` on Linux), not inside the container's writable layer.
- Survives container removal, and the same volume can be mounted into multiple containers to share data.
- Created explicitly (`docker volume create`) or via `-v`/`--mount` on `docker run`, or a `volumes:` block in Compose.

7. Difference between image and container

| | Image | Container |
|---|---|---|
| Nature | Static, read-only template | Running (or stopped) instance of an image |
| State | No runtime state | Has a writable layer + runtime state |
| Resource use | Disk space only | CPU / memory / network while running |
| Relationship | One image → many containers | Each container is created from one image |
| Commands | `docker build`, `docker pull` | `docker run`, `docker start` |

In short: the image is the recipe, the container is the meal made from it — one recipe can make many meals.

 8. Difference between container filesystem and volume

| | Container's writable layer | Volume |
|---|---|---|
| Lifecycle | Deleted when the container is removed | Independent of any container |
| Managed by | Storage driver (e.g. overlay2), copy-on-write | Dedicated volume driver/location |
| Sharing | Not easily shared across containers | Can be mounted into multiple containers |
| Best for | Temporary/scratch data | Databases, uploads, logs — anything that must persist |

In short: the container filesystem is scratch space that disappears with the container; a volume is durable storage that outlives it.


 Command reference

Run these locally to see your own machine's current state — actual output will differ from anyone else's:

| Command | Shows |
|---|---|
| `docker images` | Local images: REPOSITORY, TAG, IMAGE ID, CREATED, SIZE |
| `docker ps` | Only running containers |
| `docker ps -a` | All containers, including stopped/exited ones |
| `docker network ls` | All networks on the host (`bridge`, `host`, `none`, + any custom ones) |
| `docker volume ls` | All volumes on the host |



