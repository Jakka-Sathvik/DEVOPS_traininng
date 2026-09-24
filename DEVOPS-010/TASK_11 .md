TASK 11 — Final Docker Troubleshooting Test
Objective

Trainer intentionally breaks or stops one service.

You are only told:

“The application is not working. Find the problem and fix it.”

Your job:

Problem
   ↓
Observe
   ↓
Check containers
   ↓
Check status
   ↓
Check logs
   ↓
Inspect configuration
   ↓
Find root cause
   ↓
Fix
   ↓
Restart
   ↓
Verify
Step 1 — Check running containers

Start with:

docker ps

Look for:

devops-app
devops-database
Example
CONTAINER ID   IMAGE                STATUS
abc123         devops-final:v1     Up
def456         postgres:16-alpine  Up

If one container is missing, continue with:

docker ps -a

This is important because docker ps shows running containers, while docker ps -a also shows stopped/exited containers.

Step 2 — Check Compose status

Run:

docker compose ps

Look for:

NAME                STATUS
devops-app          Up
devops-database     Up (healthy)

Possible problems:

devops-app          Exited
devops-database     Exited
devops-database     Up (unhealthy)

This immediately tells you which service may be failing.

Step 3 — Check application logs

If the application looks suspicious:

docker compose logs app

Or:

docker logs devops-app

For the latest messages:

docker compose logs --tail=50 app

Look for errors such as:

ModuleNotFoundError
python: can't open file
Connection refused
Address already in use
Step 4 — Check database logs

If PostgreSQL is the problem:

docker compose logs database

Or:

docker logs devops-database

Look for:

password authentication failed
database system is starting up
database system is ready to accept connections
Step 5 — Check the Compose configuration

Run:

docker compose config

This is one of the most useful commands.

It shows the final configuration after Compose processes:

.env
environment variables
services
networks
volumes
ports
dependencies

For validation only:

docker compose config --quiet

If there is a configuration error, fix it before continuing.

Step 6 — Check the network

Run:

docker network ls

Find the project network.

Then:

docker network inspect <network-name>

You should find both containers:

devops-app
devops-database

The application should communicate with PostgreSQL using:

database:5432

Not:

localhost:5432

Inside a Docker container, localhost means the same container.

Step 7 — Check the volume

Run:

docker volume ls

You should see something similar to:

devops-docker-final-project_postgres_data

Then inspect the database:

docker inspect devops-database

Look for:

Mounts

You should find:

Destination:
/var/lib/postgresql/data

This confirms PostgreSQL is using the persistent volume.

Step 8 — Test PostgreSQL directly

Check PostgreSQL health:

docker exec -it devops-database pg_isready -U devopsuser -d devopsdb

Expected:

/var/run/postgresql:5432 - accepting connections

If this succeeds, PostgreSQL itself is running.

Step 9 — Test the application

Test the main page:

curl http://localhost:5000/

Then:

curl http://localhost:5000/health

Expected:

{
  "application": "running",
  "database": "connected",
  "status": "success"
}
Common Trainer Failures

The trainer may intentionally create one of these problems.

Failure 1 — Application container stopped
Investigation
docker ps -a

You may see:

devops-app    Exited (1)

Check logs:

docker logs devops-app

Possible error:

python: can't open file 'app.py'
Root Cause

Application startup command or application files are incorrect.

Fix

Correct the Dockerfile:

CMD ["python", "app.py"]

Then rebuild:

docker compose up -d --build

Verify:

docker compose ps
curl http://localhost:5000/health
Failure 2 — Database container stopped

Check:

docker compose ps
docker compose logs database

If PostgreSQL is stopped, investigate the log.

For example:

database system is unavailable

or another configuration error.

Fix the configuration and restart:

docker compose up -d

Then:

docker compose ps

Expected:

devops-database    Up (healthy)
Failure 3 — Wrong database hostname

Suppose the configuration contains:

DB_HOST=wrong-database

Check:

docker compose config

Then inspect application logs:

docker compose logs app

You may find a connection error.

Root Cause

The application cannot resolve the PostgreSQL service.

Correct value
DB_HOST=database

Restart:

docker compose up -d

Test:

curl http://localhost:5000/health
Failure 4 — Wrong port mapping

Suppose Compose contains:

ports:
  - "8080:5000"

instead of:

ports:
  - "5000:5000"

Check:

docker compose ps

You may see:

0.0.0.0:8080->5000/tcp

Then:

curl http://localhost:5000/

fails.

But:

curl http://localhost:8080/

works.

Root Cause

The host port was changed.

Fix

Restore:

ports:
  - "5000:5000"

Then:

docker compose up -d
Failure 5 — Database unhealthy

Check:

docker compose ps

You may see:

devops-database    Up (unhealthy)

Check:

docker compose logs database

Then check:

docker inspect --format='{{.State.Health.Status}}' devops-database

Possible issue: health check credentials/configuration don't match the database configuration.

Check:

docker compose config

Make sure the health check is consistent:

healthcheck:
  test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]

Then restart:

docker compose up -d

Verify:

docker compose ps
Failure 6 — Network problem

Check:

docker network ls

Then:

docker network inspect <network-name>

Both should be connected to the same network:

devops-app
devops-database

If they aren't, inspect your docker-compose.yml and make sure both services use:

networks:
  - devops-network
Failure 7 — Volume problem

Check:

docker volume ls

Then:

docker inspect devops-database

Look for:

/var/lib/postgresql/data

The Compose configuration should contain:

volumes:
  - postgres_data:/var/lib/postgresql/data

And at the bottom:

volumes:
  postgres_data:
Important

Do NOT run:

docker compose down -v

during this test.

That removes the named volume and can destroy your PostgreSQL training data.

The Most Important Troubleshooting Method

Don't randomly change files.

Use this sequence:

docker ps

↓

docker ps -a

↓

docker compose ps

↓

docker compose logs app

↓

docker compose logs database

↓

docker compose config

↓

docker network ls

↓

docker network inspect <network>

↓

docker volume ls

↓

docker inspect <container>

↓

Fix the identified problem.

Final Verification

After fixing the problem, run all of these:

docker compose ps
docker ps
docker compose logs --tail=20 app
docker compose logs --tail=20 database
curl http://localhost:5000/
curl http://localhost:5000/health
docker exec -it devops-database pg_isready -U devopsuser -d devopsdb

Expected final state:

Application       → Running
Database          → Running
Database Health   → Healthy
Network           → Connected
Volume            → Mounted
Application URL   → Working
Health Check      → HTTP 200
PostgreSQL        → Accepting connections
Your Task 11 answer format

When the trainer asks you to report the result, use:

Item	Result
Problem	Application was not working
Investigation	Checked containers, Compose status, logs and configuration
Root Cause	State exactly what was broken
Solution	State exactly what you changed
Verification	/health returned HTTP 200 and services became healthy
Example
Root Cause:
The application container was using an incorrect startup command.

Solution:
Corrected the Dockerfile CMD to:
CMD ["python", "app.py"]

Rebuilt and restarted the application using:
docker compose up -d --build

Verification:
docker compose ps showed the application as running.

curl http://localhost:5000/health

Returned HTTP 200 with the expected health-check response.