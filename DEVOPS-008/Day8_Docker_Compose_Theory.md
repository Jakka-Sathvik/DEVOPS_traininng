TASK 1 — Understand Docker Compose

Time: 45 Minutes
Deliverable: Day8_Docker_Compose_Theory.md

1. Objective

The objective of this task is to understand Docker Compose and how it simplifies the management of applications that require multiple Docker containers.

In a real application, we may need multiple services such as:

Web Application
      ↓
Database
      ↓
Redis / Cache

Running each container manually can become difficult.

Without Docker Compose:

docker run web
docker run database
docker run redis
docker network create ...
docker volume create ...

With Docker Compose:

docker compose up

Docker Compose allows us to define the application's containers, networks, volumes, environment variables, ports, and other configuration in a YAML file.

2. What is Docker Compose?

Docker Compose is a tool used to define and run multi-container Docker applications.

Instead of running every container manually, we can describe the complete application architecture in a configuration file.

The configuration is commonly stored in:

compose.yaml

or:

docker-compose.yml

Example:

services:

  web:
    image: nginx:latest
    ports:
      - "8080:80"

  database:
    image: postgres:16

  redis:
    image: redis:7

Then the complete application can be started with:

docker compose up

Docker Compose creates and manages the required containers and their supporting resources according to the configuration.

3. Why Do We Need Docker Compose?

Modern applications often contain multiple components.

For example:

                 Application
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
       Web          API        Database
        │                         │
        └───────────┬─────────────┘
                    ↓
                  Redis

Managing these containers manually can require many commands.

For example:

docker network create app-network

docker volume create database-data

docker run -d \
  --name database \
  --network app-network \
  -v database-data:/var/lib/postgresql/data \
  postgres:16

docker run -d \
  --name redis \
  --network app-network \
  redis:7

docker run -d \
  --name web \
  --network app-network \
  -p 8080:80 \
  nginx

This becomes difficult to maintain.

With Compose, the configuration can be stored in one file:

services:
  web:
    image: nginx:latest

  database:
    image: postgres:16

  redis:
    image: redis:7

Then:

docker compose up -d

This provides a much simpler workflow.

4. Main Benefits of Docker Compose

Docker Compose provides several benefits.

4.1 Multi-Container Management

Compose allows multiple services to be defined in one configuration.

Example:

compose.yaml
     │
     ├── web
     ├── api
     ├── database
     └── redis
4.2 Configuration as Code

The application's infrastructure configuration can be stored as a file.

For example:

Git Repository
      │
      ↓
compose.yaml
      │
      ↓
Docker Compose
      │
      ↓
Application Environment

This makes configuration easier to review and version-control.

4.3 Reproducibility

Another developer can clone the project and use the same Compose configuration.

For example:

git clone <repository>
cd project
docker compose up -d

The same services can be created according to the Compose file.

4.4 Easy Startup and Shutdown

Start the application:

docker compose up -d

Stop the application:

docker compose down

View running services:

docker compose ps

View logs:

docker compose logs

This is much easier than manually managing several containers.

5. What is a Service?

A service in Docker Compose represents a containerized component of an application.

Example:

services:

  web:
    image: nginx:latest

  database:
    image: postgres:16

  redis:
    image: redis:7

Here there are three services:

web
database
redis

Each service describes how a container should be created and run.

A service can define:

Docker image
Build configuration
Ports
Environment variables
Networks
Volumes
Dependencies
Restart policies
Health checks
6. Example of Services

Consider a simple application:

                 Web Application
                       │
                       ↓
                     API
                    /   \
                   ↓     ↓
              PostgreSQL Redis

The Compose file could contain:

services:

  web:
    image: nginx:latest

  api:
    build: ./api

  database:
    image: postgres:16

  redis:
    image: redis:7

The four services are:

web
api
database
redis

Docker Compose manages them as one application environment.

7. What is docker-compose.yml?

docker-compose.yml is a YAML configuration file used to define a multi-container application.

Modern Docker Compose also commonly uses:

compose.yaml

The older and still widely recognized name is:

docker-compose.yml

Example:

services:

  web:
    image: nginx:latest
    ports:
      - "8080:80"

  database:
    image: postgres:16

This file tells Compose:

Create web service
        +
Create database service
        +
Configure their required settings
8. Basic Docker Compose File Structure

A typical Compose file can contain:

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"

  database:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: example

volumes:
  database-data:

networks:
  application-network:

The major sections are:

Compose File
    │
    ├── services
    │
    ├── volumes
    │
    └── networks
9. Service Configuration

Example:

services:

  web:
    image: nginx:latest
    ports:
      - "8080:80"

Explanation:

services

Defines application services.

web

Name of the service.

image

Specifies the Docker image.

image: nginx:latest
ports

Maps a host port to a container port.

ports:
  - "8080:80"

This means:

Host Port 8080
      ↓
Container Port 80
10. How is Docker Compose Different from docker run?

docker run is primarily used to create and start an individual container from an image.

Example:

docker run -d --name web -p 8080:80 nginx

Docker Compose is designed to manage an application consisting of multiple services.

Example:

docker compose up -d

Comparison:

Feature	docker run	Docker Compose
Main purpose	Run individual containers	Manage multi-service applications
Configuration	Command-line arguments	YAML configuration
Multiple services	Manual management	Defined together
Networks	Usually configured manually	Can be defined automatically
Volumes	Manually configured	Defined in Compose
Environment variables	Command options/files	Defined in Compose
Reproducibility	Depends on commands	Configuration stored as code
Startup	Individual commands	One Compose command
Shutdown	Individual container commands	docker compose down
11. Example: Without Compose

Suppose we have:

Web
Database
Redis

We might need:

docker network create app-network

docker volume create db-data

docker run -d \
  --name database \
  --network app-network \
  -v db-data:/var/lib/postgresql/data \
  postgres:16

docker run -d \
  --name redis \
  --network app-network \
  redis:7

docker run -d \
  --name web \
  --network app-network \
  -p 8080:80 \
  nginx

There are many commands and configuration details to remember.

12. Example: With Compose

The same architecture can be represented as:

services:

  web:
    image: nginx:latest
    ports:
      - "8080:80"

  database:
    image: postgres:16
    volumes:
      - db-data:/var/lib/postgresql/data

  redis:
    image: redis:7

volumes:
  db-data:

Then run:

docker compose up -d

This is much easier to maintain.

13. What is a Docker Network?

A Docker network allows containers to communicate with each other.

Example:

Docker Network
      │
      ├── web
      │
      ├── api
      │
      ├── database
      │
      └── redis

Containers attached to the same Docker network can communicate using Docker's networking and service discovery mechanisms.

For example:

api
 │
 └── database:5432

The application can use the Compose service name:

database

instead of trying to use the host's localhost.

14. Important localhost Concept

Inside a container:

localhost

refers to the same container.

For example:

API Container
     │
     └── localhost:5432

This does not normally refer to the PostgreSQL container.

If PostgreSQL is another Compose service:

services:

  api:
    ...

  database:
    image: postgres:16

The API should typically connect to:

database:5432

because database is the Compose service name.

Architecture:

API Container
      │
      │ database:5432
      ↓
Database Container
15. What is a Docker Volume?

A Docker volume provides persistent storage for containers.

Containers are generally considered replaceable.

If a database container is removed, data stored only inside the container can be lost.

A volume allows data to persist outside the container lifecycle.

Example:

PostgreSQL Container
        │
        ↓
Docker Volume
        │
        ↓
Persistent Database Data

Compose example:

services:

  database:
    image: postgres:16
    volumes:
      - database-data:/var/lib/postgresql/data

volumes:
  database-data:

Here:

database-data

is a named Docker volume.

16. Why Volumes are Important

Consider a PostgreSQL database:

Container
   │
   └── Database Files

If the container is deleted:

Container
    X

the data may not survive if it was stored only in the container's writable layer.

With a volume:

PostgreSQL Container
       │
       ↓
database-data
       │
       ↓
Persistent Storage

The container can be recreated while the data remains in the volume.

17. Docker Compose Networking and Volumes Together

A real application can use both.

                     Docker Compose
                           │
              ┌────────────┼────────────┐
              ↓            ↓            ↓
             Web           API       Database
              │             │            │
              └─────────────┼────────────┘
                            │
                     Docker Network
                                         
                                      Database
                                         │
                                         ↓
                                  Docker Volume

This provides:

service communication through networks
persistent data through volumes
18. Example Multi-Container Application

Consider an application with:

Frontend
Backend API
PostgreSQL
Redis

Architecture:

                 Docker Compose
                       │
       ┌───────────────┼────────────────┐
       ↓               ↓                ↓
   Frontend          Backend         Database
       │               │                │
       └───────────────┼────────────────┘
                       │
                     Redis
                       │
                Docker Network
                       │
                PostgreSQL Volume

A Compose file could look like:

services:

  frontend:
    image: nginx:latest
    ports:
      - "8080:80"

  backend:
    build: ./backend
    environment:
      DB_HOST: database
      REDIS_HOST: redis

  database:
    image: postgres:16
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:7

volumes:
  postgres-data:

The important relationships are:

Backend → database
Backend → redis
Database → postgres-data
19. Important Docker Compose Commands
Start services
docker compose up

Run in background:

docker compose up -d
Stop and remove Compose resources
docker compose down
Check service status
docker compose ps
View logs
docker compose logs

For a particular service:

docker compose logs backend

Follow logs:

docker compose logs -f
Build images
docker compose build

Build and start:

docker compose up -d --build
Validate the Compose configuration
docker compose config

This is useful for detecting configuration problems before starting the application.

20. Docker Compose vs Dockerfile

These two files have different purposes.

Dockerfile

A Dockerfile describes how to build an image.

Example:

FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]

Architecture:

Dockerfile
    ↓
Docker Build
    ↓
Docker Image
Compose File

Compose describes how to run and connect application services.

services:
  app:
    build: .
    ports:
      - "5000:5000"

Architecture:

Compose File
     ↓
Services
     ↓
Containers
     ↓
Networks / Volumes

They are often used together.

Dockerfile
     ↓
Application Image
     ↓
Docker Compose
     ↓
Application Container
21. Docker Compose in DevOps

Docker Compose is useful during development, testing, and some deployment scenarios.

A developer may have:

Developer
    │
    ↓
Git Repository
    │
    ↓
compose.yaml
    │
    ↓
docker compose up
    │
    ├── Application
    ├── Database
    └── Redis

This provides a repeatable development environment.

Compose can also be useful for:

local integration testing
development environments
application demonstrations
CI test environments
multi-container prototypes
learning environments

For large production environments, organizations may use platforms such as Kubernetes, Amazon ECS, or other orchestrators depending on their requirements.

22. Docker Compose Architecture

The overall architecture can be represented as:

                         Docker Compose
                               │
                 ┌─────────────┼─────────────┐
                 ↓             ↓             ↓
              Service        Service       Service
                 │             │             │
                Web            API         Database
                 │             │             │
                 └─────────────┼─────────────┘
                               ↓
                        Docker Network
                               │
                               ↓
                         Docker Volume
                               │
                               ↓
                       Persistent Data
23. Without Compose vs With Compose
Without Compose
Developer
    │
    ├── docker network create
    ├── docker volume create
    ├── docker run web
    ├── docker run database
    └── docker run redis

The developer must manually manage the configuration.

With Compose
Developer
    │
    ↓
compose.yaml
    │
    ↓
docker compose up
    │
    ├── Web
    ├── Database
    ├── Redis
    ├── Network
    └── Volume

This provides a single application definition.

24. Key Differences Summary
Concept	Meaning
Docker Compose	Tool for defining/running multi-container applications
Service	A containerized component defined in Compose
Compose file	YAML configuration describing the application
Network	Enables container-to-container communication
Volume	Provides persistent storage
docker run	Starts an individual container
docker compose up	Creates/starts the services defined in Compose
Dockerfile	Defines how an image is built
Image	Template used to create containers
Container	Running instance of an image
25. What I Learned

During this task, I learned:

Docker Compose is used to manage multi-container applications.
Compose configuration is stored in a YAML file such as compose.yaml or docker-compose.yml.
A service represents a containerized application component.
Docker networks allow containers to communicate with each other.
Docker volumes provide persistent storage.
docker run is mainly used to run individual containers.
docker compose up can start multiple services from one configuration.
Compose makes application environments easier to reproduce.
Docker Compose can work together with Dockerfiles.
Service names can be used for container-to-container communication on the Compose network.
Database services should generally use persistent volumes.
Compose is especially useful for development and multi-container testing environments.