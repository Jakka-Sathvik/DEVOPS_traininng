1. Go to the project
cd devops-docker-final-project

Check:

ls -la

You should have:

app/
Dockerfile
docker-compose.yml
.env
.env.example
.gitignore
2. Verify the environment file

Run:

cat .env

For training, it should contain something like:

DATABASE_NAME=devopsdb
DATABASE_USER=devopsuser
DATABASE_PASSWORD=devopspass
DATABASE_HOST=database
DATABASE_PORT=5432

⚠️ Don't share the actual password in screenshots or commit it to Git.

3. Build the application

Because your docker-compose.yml contains:

app:
  build:
    context: .
    dockerfile: Dockerfile

run:

docker compose build

Or, to make sure everything is rebuilt:

docker compose build --no-cache

You should eventually see something similar to:

Successfully built ...
Successfully tagged devops-final:v1

Verify:

docker images

You should see:

REPOSITORY      TAG
devops-final    v1
4. Start all services

Run:

docker compose up -d

Or combine build + start:

docker compose up -d --build

For this task, I recommend:

docker compose up -d --build

This ensures your latest Dockerfile/application changes are included.

5. Check container status

Run:

docker compose ps

Expected:

NAME               IMAGE                 STATUS
devops-app         devops-final:v1      Up
devops-database    postgres:16-alpine   Up (healthy)

You want:

devops-app       → Running
devops-database  → Running

And PostgreSQL should ideally show:

Up (healthy)
6. Check application logs

Run:

docker compose logs app

For live logs:

docker compose logs -f app

You should see Flask starting, something similar to:

* Serving Flask app 'app'
* Running on all addresses (0.0.0.0)
* Running on http://127.0.0.1:5000

Press:

Ctrl + C

to stop following the logs.

7. Check database logs

Run:

docker compose logs database

You should see PostgreSQL startup information.

Look for a message indicating that PostgreSQL is:

ready to accept connections

You can also use:

docker compose logs --tail=50 database

to see only the latest 50 lines.

8. Test /

Run:

curl http://localhost:5000/

You should receive the HTML from your Flask application.

It should contain:

DevOps Training Application
Application Status: Running
Environment: Development
Database: Connected

If you're using GitHub Codespaces, open the forwarded port 5000 in the Ports tab and test the application in your browser.

9. Test /health

Run:

curl http://localhost:5000/health

Expected:

{
  "application": "running",
  "database": "connected",
  "status": "success"
}

This confirms that the Flask application is responding.

10. Verify the Docker network

First list networks:

docker network ls

You should find something similar to:

devops-docker-final-project_devops-network

Inspect it:

docker network inspect devops-docker-final-project_devops-network

Look for both containers:

devops-app
devops-database

This proves both services are connected to the same Docker network.

11. Verify database connectivity

There are two levels here.

Level 1 — Verify PostgreSQL itself

Run:

docker exec -it devops-database pg_isready -U devopsuser -d devopsdb

Expected:

/var/run/postgresql:5432 - accepting connections

This confirms PostgreSQL is accepting connections.

Level 2 — Connect using PostgreSQL client

Run:

docker exec -it devops-database psql -U devopsuser -d devopsdb

You should get:

devopsdb=#

Then:

\conninfo

You should see connection information.

Exit:

\q
12. Verify the named volume

Run:

docker volume ls

You should see something similar to:

DRIVER    VOLUME NAME
local     devops-docker-final-project_postgres_data

Inspect it:

docker volume inspect devops-docker-final-project_postgres_data

This verifies that the PostgreSQL data is stored in a named Docker volume.

13. Verify the volume is mounted

Run:

docker inspect devops-database

Find the Mounts section.

You should see something similar to:

"Destination": "/var/lib/postgresql/data"

and:

"Name": "...postgres_data"

That proves:

PostgreSQL
    ↓
/var/lib/postgresql/data
    ↓
postgres_data volume
14. Verify health check

Run:

docker inspect --format='{{.State.Health.Status}}' devops-database

Expected:

healthy

This is one of the most important screenshots for Task 7.

15. Run one complete verification

Now run:

docker compose ps

Then:

curl http://localhost:5000/

Then:

curl http://localhost:5000/health

Then:

docker exec -it devops-database pg_isready -U devopsuser -d devopsdb

Then:

docker inspect --format='{{.State.Health.Status}}' devops-database
16. Expected final architecture
                    Docker Compose
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
        ┌───────────┐         ┌────────────┐
        │ Flask App │         │ PostgreSQL │
        │ Container │         │ Container  │
        └─────┬─────┘         └─────┬──────┘
              │                     │
              │   Docker Network    │
              └──────────┬──────────┘
                         │
                         ▼
                  postgres_data
                  Named Volume
Final result
Requirement	Verification	Expected
Application	docker compose ps	Running
Database	docker compose ps	Running
Network	docker network inspect	Both containers connected
Volume	docker volume ls	postgres_data exists
Volume mount	docker inspect	/var/lib/postgresql/data
DB health	docker inspect ... Health.Status	healthy
/	curl localhost:5000/	Application page
/health	curl localhost:5000/health	HTTP 200 / JSON
PostgreSQL	pg_isready	Accepting connections
Best screenshots for Task 7

Capture these:

docker compose ps
docker compose logs app
docker compose logs database
curl http://localhost:5000/
curl http://localhost:5000/health
docker network inspect devops-docker-final-project_devops-network
docker volume ls
docker inspect --format='{{.State.Health.Status}}' devops-database