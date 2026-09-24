TASK 9 — Test Persistence
1. Start the complete application

From the project root:

cd devops-docker-final-project

Start everything:

docker compose up -d

Check:

docker compose ps

Expected:

NAME               STATUS
devops-app         Up
devops-database    Up (healthy)
2. Verify the database

Run:

docker exec -it devops-database pg_isready -U devopsuser -d devopsdb

Expected:

accepting connections
3. Create test database data

Connect to PostgreSQL:

docker exec -it devops-database psql -U devopsuser -d devopsdb

You should see:

devopsdb=#

Create a table:

CREATE TABLE training_data (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    topic VARCHAR(100)
);

Insert some test data:

INSERT INTO training_data (name, topic)
VALUES
('Sathvik', 'Docker'),
('DevOps Training', 'Docker Compose'),
('Persistence Test', 'PostgreSQL');

Check the data:

SELECT * FROM training_data;

Expected:

 id |       name        |       topic
----+-------------------+------------------
  1 | Sathvik           | Docker
  2 | DevOps Training   | Docker Compose
  3 | Persistence Test  | PostgreSQL

Your exact IDs may differ.

Exit PostgreSQL:

\q
4. Verify the named volume

Run:

docker volume ls

You should see something similar to:

DRIVER    VOLUME NAME
local     devops-docker-final-project_postgres_data

The volume contains PostgreSQL's data.

The architecture is:

PostgreSQL Container
        │
        ▼
/var/lib/postgresql/data
        │
        ▼
postgres_data
        │
        ▼
Named Docker Volume
5. Verify the data before container removal

Run:

docker exec -it devops-database psql -U devopsuser -d devopsdb -c "SELECT * FROM training_data;"

Expected:

 id |       name        |       topic
----+-------------------+------------------
  1 | Sathvik           | Docker
  2 | DevOps Training   | Docker Compose
  3 | Persistence Test  | PostgreSQL
Take a screenshot here

This is your Before Container Removal evidence.

6. Stop the application

Stop only the application:

docker compose stop app

Check:

docker compose ps

The application should be stopped while PostgreSQL can remain running.

7. Stop the containers

Now stop the Compose services:

docker compose stop

Check:

docker compose ps

The containers should no longer be running.

8. Remove the containers

Now remove the containers:

docker compose rm -f

You can verify:

docker ps -a

The devops-app and devops-database containers should be removed.

Important

We removed the containers, but we did not remove:

postgres_data

The named volume still exists.

Verify:

docker volume ls

You should still see:

devops-docker-final-project_postgres_data

This is the key point of the exercise.

9. Start Compose again

Now recreate the containers:

docker compose up -d

Docker Compose will:

Read docker-compose.yml
        ↓
Create new containers
        ↓
Find existing postgres_data volume
        ↓
Mount volume
        ↓
Start PostgreSQL

Check:

docker compose ps

Expected:

NAME               STATUS
devops-app         Up
devops-database    Up (healthy)
10. Check the database again

Run:

docker exec -it devops-database psql -U devopsuser -d devopsdb -c "SELECT * FROM training_data;"

You should still see:

 id |       name        |       topic
----+-------------------+------------------
  1 | Sathvik           | Docker
  2 | DevOps Training   | Docker Compose
  3 | Persistence Test  | PostgreSQL

🎉 Persistence has been successfully demonstrated.

The database container was removed, recreated, and the data remained because the data was stored in the named Docker volume.

11. Verify the volume is mounted again

Run:

docker inspect devops-database

Look for:

Mounts

You should find:

Source: ...postgres_data
Destination: /var/lib/postgresql/data

This proves the new PostgreSQL container is using the existing volume.

12. Complete persistence flow

Your training architecture should be documented as:

          BEFORE REMOVAL
               │
               ▼
       PostgreSQL Container
               │
               ▼
        training_data table
               │
               ▼
          postgres_data
          Named Volume
               │
               │
               ▼
       Container Removed
               │
               │
        Volume remains
               │
               ▼
       docker compose up
               │
               ▼
    New PostgreSQL Container
               │
               ▼
     Existing postgres_data
               │
               ▼
       training_data
       STILL AVAILABLE
Why did the data survive?

Because the data was stored in:

postgres_data

rather than only inside the PostgreSQL container's writable filesystem.

A simplified comparison:

Resource	Container removed?	Data remains?
Container filesystem	Yes	❌
Named Docker volume	No	✅
PostgreSQL data in named volume	No	✅
13. Very important command difference
Safe for this exercise
docker compose rm -f

This removes containers.

Dangerous for this exercise
docker compose down -v

The -v removes the Compose volumes.

If you run:

docker compose down -v

you can lose the PostgreSQL data you are trying to preserve.

14. Final verification commands

Run these at the end:

docker compose ps
docker volume ls
docker exec -it devops-database pg_isready -U devopsuser -d devopsdb
docker exec -it devops-database psql -U devopsuser -d devopsdb -c "SELECT * FROM training_data;"

Expected final result:

Application       → Running
Database          → Running
Database Health   → Healthy
Volume            → Exists
Data              → Still Available
Persistence       → SUCCESS