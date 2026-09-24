Dockerized DevOps Training Application
1. Final Architecture

Your project should look like this:

                    GitHub Repository
                           │
                           │
                    Docker Compose
                           │
             ┌─────────────┴─────────────┐
             │                           │
        Flask Application          PostgreSQL Database
          Container                   Container
             │                           │
             │                           │
          Port 5000                  Port 5432
             │                           │
             └───────────┬───────────────┘
                         │
                  Docker Network
                  devops-network
                         │
                    PostgreSQL
                   Named Volume
                  postgres_data
2. Required Project Structure

Your final GitHub repository should contain:

devops-docker-final-project/
│
├── app/
│   ├── app.py
│   └── requirements.txt
│
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── .gitignore
├── .env
├── .env.example
├── README.md
│
├── scripts/
│
└── docs/

Important: .env should remain local and should not be pushed to GitHub.

3. Application

Your Flask application should provide:

/
└── Application information

/health
└── Health check

Test:

curl http://localhost:5000/

and:

curl http://localhost:5000/health

Expected health response:

{
  "status": "success",
  "application": "running",
  "database": "connected"
}

Remember: in your current implementation, "database": "connected" is a demo/static value; the Flask application isn't actually opening a PostgreSQL connection.

4. Dockerfile

Your production-style Dockerfile:

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

Important points:

✓ Python slim image
✓ Non-root user
✓ Working directory
✓ Dependency installation
✓ Application copied
✓ Port 5000
✓ Application startup command

Build:

docker build -t devops-final:v1 .
5. PostgreSQL

Docker Compose should create PostgreSQL:

database:
  image: postgres:16-alpine
  container_name: devops-database

The database should use environment variables:

environment:
  POSTGRES_DB: ${POSTGRES_DB}
  POSTGRES_USER: ${POSTGRES_USER}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
6. Docker Compose

Your complete Compose architecture should include:

Application
     │
     ├── Build from Dockerfile
     ├── Port 5000
     ├── Environment variables
     ├── Depends on PostgreSQL
     └── devops-network

PostgreSQL
     │
     ├── postgres:16-alpine
     ├── Port 5432
     ├── Health check
     ├── postgres_data volume
     └── devops-network

Start everything:

docker compose up -d --build

Check:

docker compose ps

Expected:

NAME                 STATUS
devops-app           Up
devops-database      Up (healthy)
7. Docker Network

Your Compose file should define:

networks:
  devops-network:
    driver: bridge

Both services should use it:

networks:
  - devops-network

Check:

docker network ls

Then:

docker network inspect <network-name>

You should find:

devops-app
devops-database

on the same network.

The application communicates with PostgreSQL using:

database:5432

not:

localhost:5432
8. Docker Volume

PostgreSQL must use a named volume:

volumes:
  - postgres_data:/var/lib/postgresql/data

And define:

volumes:
  postgres_data:

Check:

docker volume ls

You should see something similar to:

devops-docker-final-project_postgres_data

This ensures PostgreSQL data survives container removal.

Never use docker compose down -v when testing persistence, because that removes the volume.

9. Environment Configuration

.env:

POSTGRES_DB=devopsdb
POSTGRES_USER=devopsuser
POSTGRES_PASSWORD=devopspass

DB_HOST=database
DB_PORT=5432

.env.example:

POSTGRES_DB=
POSTGRES_USER=
POSTGRES_PASSWORD=

DB_HOST=
DB_PORT=

Your .gitignore must contain:

.env

Verify:

git check-ignore -v .env

And:

git ls-files .env

The second command should produce no output.

10. Health Check

PostgreSQL should have:

healthcheck:
  test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
  interval: 10s
  timeout: 5s
  retries: 5

Check:

docker compose ps

Expected:

devops-database    Up (healthy)

You can also verify directly:

docker inspect --format='{{.State.Health.Status}}' devops-database

Expected:

healthy
11. PostgreSQL Verification

Run:

docker exec -it devops-database pg_isready -U devopsuser -d devopsdb

Expected:

accepting connections

Connect:

docker exec -it devops-database psql -U devopsuser -d devopsdb

Then:

SELECT current_database();

Expected:

devopsdb

Exit:

\q
12. Persistence Test

Create a test table:

CREATE TABLE training_data (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    topic VARCHAR(100)
);

Insert data:

INSERT INTO training_data (name, topic)
VALUES
('Sathvik', 'Docker'),
('DevOps Training', 'Docker Compose'),
('Persistence Test', 'PostgreSQL');

Check:

SELECT * FROM training_data;

Then stop/remove containers:

docker compose stop
docker compose rm -f

Do not remove the volume.

Check:

docker volume ls

Start again:

docker compose up -d

Query:

docker exec -it devops-database \
psql -U devopsuser -d devopsdb \
-c "SELECT * FROM training_data;"

If the same records appear, your Docker Volume Persistence requirement is complete.

13. GitHub Repository

Your repository should contain:

✓ Application
✓ Dockerfile
✓ Docker Compose
✓ PostgreSQL
✓ Network
✓ Volume
✓ Environment configuration
✓ Health check
✓ README
✓ Documentation

Useful Git commands:

git status
git branch -a
git log --oneline --graph --all

Push:

git push origin main

Your Task 10 workflow should also be visible:

main
develop
feature/docker-app
feature/database
bugfix/health-check

And at least one Pull Request should have been created and merged.

14. Final Testing Checklist

Run this complete checklist before submitting the project:

docker compose config --quiet
docker compose up -d --build
docker compose ps
docker ps
docker compose logs app
docker compose logs database
curl http://localhost:5000/
curl http://localhost:5000/health
docker exec -it devops-database pg_isready -U devopsuser -d devopsdb
docker network ls
docker volume ls
docker inspect devops-database
Final Requirement Matrix
Requirement	Implementation	Verification
Application	Flask	curl localhost:5000
PostgreSQL	postgres:16-alpine	pg_isready
Dockerfile	Production-style Dockerfile	docker build
Docker Compose	docker-compose.yml	docker compose config
Docker Network	devops-network	docker network inspect
Docker Volume	postgres_data	docker volume ls
Environment	.env + .env.example	git check-ignore
Health Check	/health + PostgreSQL healthcheck	curl + docker compose ps
GitHub	Git repository	git remote -v
Git Workflow	main/develop/features	git branch -a
Pull Request	Feature → develop	GitHub PR
Persistence	PostgreSQL named volume	Data survives container recreation
Troubleshooting	Task 11	Root cause + fix + verification
Final success condition

Your project is complete when you can demonstrate:

             DOCKERIZED DEVOPS APPLICATION
                         │
          ┌──────────────┴──────────────┐
          │                             │
     Flask App                     PostgreSQL
     Port 5000                     Port 5432
          │                             │
          └──────────┬──────────────────┘
                     │
              Docker Network
                     │
              Named Volume
                     │
              Persistent Data
                     │
              Health Checks
                     │
                  GitHub
                     │
             Git + Pull Request