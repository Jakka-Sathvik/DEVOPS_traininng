TASK 8 — Failure Simulation
The troubleshooting workflow

Always follow this sequence:

Problem
   ↓
Observe
   ↓
Check Status
   ↓
Check Logs
   ↓
Inspect Configuration
   ↓
Find Root Cause
   ↓
Fix
   ↓
Restart
   ↓
Test
1. Before intentionally breaking anything

First make sure your application works normally.

docker compose up -d --build

Check:

docker compose ps

You should have:

devops-app         Up
devops-database    Up (healthy)

Test:

curl http://localhost:5000/health

Also verify PostgreSQL:

docker exec devops-database pg_isready -U devopsuser -d devopsdb

Expected:

accepting connections

Only after the normal setup works should you simulate a failure.

FAILURE 1 — Change the database password
Step 1 — Introduce the problem

Open .env:

nano .env

Change:

DATABASE_PASSWORD=devopspass

to something incorrect, for example:

DATABASE_PASSWORD=wrongpassword

Save the file.

Step 2 — Observe

Restart the services:

docker compose down
docker compose up -d

Check:

docker compose ps

At this point, observe whether the services are healthy.

Step 3 — Check logs

Check database logs:

docker compose logs database

Then application logs:

docker compose logs app

Look for authentication-related errors such as:

password authentication failed
Step 4 — Inspect configuration

Check .env:

cat .env

Check the Compose configuration:

docker compose config

Compare:

Application configuration
        ↓
DATABASE_PASSWORD

PostgreSQL configuration
        ↓
POSTGRES_PASSWORD
Important

There is a subtle point here: PostgreSQL initialization variables are used when the database is first initialized. Changing POSTGRES_PASSWORD in .env does not necessarily change the password of an already-initialized PostgreSQL database.

For your exercise, the important troubleshooting lesson is:

Environment variables and actual database credentials must be consistent.

Step 5 — Fix

Restore the correct value:

DATABASE_PASSWORD=devopspass

If your Compose file uses POSTGRES_PASSWORD, make sure that value is also correct.

Then:

docker compose down
docker compose up -d

Verify:

docker compose ps
FAILURE 2 — Change the database hostname

This is a very good networking troubleshooting exercise.

Step 1 — Break it

Open:

nano .env

Change:

DATABASE_HOST=database

to:

DATABASE_HOST=wrong-database
Step 2 — Restart
docker compose down
docker compose up -d
Step 3 — Observe

Check:

docker compose ps

Then:

docker compose logs app

You may see an error related to:

could not translate host name

or:

connection refused
Step 4 — Inspect configuration

Check:

cat .env

Then:

docker compose config

Check the actual network:

docker network ls

Then:

docker network inspect devops-docker-final-project_devops-network

You should find:

devops-app
devops-database

The PostgreSQL service is called:

database

Therefore the application should use:

database:5432

not:

wrong-database:5432
Step 5 — Fix

Change back:

DATABASE_HOST=database

Restart:

docker compose down
docker compose up -d

Test:

docker compose ps
FAILURE 3 — Change the application port

There are two common ways to create this failure.

For example, change the host mapping in docker-compose.yml:

ports:
  - "8080:5000"

Now the application is available on:

localhost:8080

not:

localhost:5000
Diagnose

Run:

docker compose ps

Look at the port mapping.

Then:

docker compose logs app

The Flask application may still be listening internally on:

5000

The host mapping is:

8080 → 5000

So:

curl http://localhost:5000

may fail.

But:

curl http://localhost:8080

should work.

Fix

Restore:

ports:
  - "5000:5000"

Then:

docker compose down
docker compose up -d

Test:

curl http://localhost:5000/health
FAILURE 4 — Break the Dockerfile startup command

This is a very common Docker troubleshooting problem.

Step 1 — Break it

Open:

nano Dockerfile

Change:

CMD ["python", "app.py"]

to something incorrect:

CMD ["python", "wrong.py"]
Step 2 — Rebuild

This is important.

A Dockerfile change requires a rebuild:

docker compose up -d --build
Step 3 — Check status
docker compose ps

The application may repeatedly stop or show an exited state.

Check:

docker ps -a
Step 4 — Check logs
docker compose logs app

You should find something similar to:

python: can't open file '/app/wrong.py'

Now you know:

Container failed
      ↓
Application process failed
      ↓
Startup command is incorrect
Step 5 — Fix

Restore:

CMD ["python", "app.py"]

Rebuild:

docker compose up -d --build

Check:

docker compose ps

Then:

curl http://localhost:5000/health
FAILURE 5 — Remove database volume configuration

This exercise demonstrates data persistence.

Your current configuration contains:

volumes:
  - postgres_data:/var/lib/postgresql/data

Temporarily remove/comment it:

# volumes:
#   - postgres_data:/var/lib/postgresql/data
Before removing it

First understand the current setup:

PostgreSQL
     ↓
/var/lib/postgresql/data
     ↓
postgres_data
     ↓
Named Docker Volume

The volume provides persistent storage.

Diagnose

Check:

docker compose config

Then:

docker volume ls

Inspect the database container:

docker inspect devops-database

Look for:

Mounts

You should notice that the expected named volume is no longer mounted if you have actually removed the volume mapping and recreated the container.

Important warning

Don't immediately run:

docker compose down -v

during this exercise.

The -v option removes Compose-managed volumes and can permanently delete the training database data.

Fix

Restore:

volumes:
  - postgres_data:/var/lib/postgresql/data

and make sure the bottom of your Compose file contains:

volumes:
  postgres_data:

Then:

docker compose up -d

Verify:

docker volume ls

and:

docker inspect devops-database
Your troubleshooting command toolkit

When the trainer gives you an unknown failure, don't immediately edit files.

Start with:

1. Check containers
docker compose ps
2. Check all containers
docker ps -a
3. Check application logs
docker compose logs app
4. Check database logs
docker compose logs database
5. Follow logs live
docker compose logs -f app
6. Check Compose configuration
docker compose config
7. Check networks
docker network ls
8. Inspect network
docker network inspect devops-docker-final-project_devops-network
9. Check volumes
docker volume ls
10. Inspect container
docker inspect devops-app

or:

docker inspect devops-database
11. Test application
curl http://localhost:5000/
curl http://localhost:5000/health
12. Test PostgreSQL
docker exec devops-database pg_isready -U devopsuser -d devopsdb
How to think during the trainer's test

If the application is not working, don't immediately guess.

Ask yourself:

Is the container running?
        ↓
        No → Check container logs
        │
        Yes
        ↓
Is the application responding?
        ↓
        No → Check application logs/port
        │
        Yes
        ↓
Is PostgreSQL healthy?
        ↓
        No → Check database logs
        │
        Yes
        ↓
Is the network correct?
        ↓
        No → Check service names/network
        │
        Yes
        ↓
Is the volume mounted?
        ↓
        No → Check Compose volume
Final troubleshooting report format

For each failure, record:

Problem:
What was intentionally changed?

Observation:
What happened after restarting?

Status:
What did docker compose ps show?

Logs:
What error appeared?

Configuration:
Which file/setting was inspected?

Root Cause:
Why did the application fail?

Fix:
What configuration was corrected?

Restart:
What command was used?

Test:
How was the fix verified?

Result:
Application → Running
Database → Running
Network → Connected
Volume → Mounted
Health Check → Healthy