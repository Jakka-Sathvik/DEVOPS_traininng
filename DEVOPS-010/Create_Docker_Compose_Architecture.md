1. Final architecture
                    Docker Compose
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
        Flask App               PostgreSQL
        Container                Container
              │                     │
              └────── Network ──────┘
                                    │
                                    ▼
                            postgres_data
                             Named Volume

Your project will look like:

devops-docker-final-project/
├── app/
│   ├── app.py
│   └── requirements.txt
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── .gitignore
├── .env.example
├── README.md
├── scripts/
└── docs/
2. Create .env

From your project root:

cd devops-docker-final-project

Create:

nano .env

Add:

POSTGRES_DB=devopsdb
POSTGRES_USER=devopsuser
POSTGRES_PASSWORD=devopspass

DB_HOST=database
DB_PORT=5432
Important

Do not commit .env to Git because it contains a password.

Your .gitignore should contain:

.env

Keep .env.example in Git.

3. Create .env.example
nano .env.example

Add:

POSTGRES_DB=devopsdb
POSTGRES_USER=devopsuser
POSTGRES_PASSWORD=change-me

DB_HOST=database
DB_PORT=5432

This shows other developers which environment variables are required without exposing your real password.

4. Create docker-compose.yml

Create/open:

nano docker-compose.yml

Use this:

services:

  app:
    build:
      context: .
      dockerfile: Dockerfile

    image: devops-final:v1
    container_name: devops-app

    ports:
      - "5000:5000"

    environment:
      DB_HOST: ${DB_HOST}
      DB_PORT: ${DB_PORT}
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

    depends_on:
      database:
        condition: service_healthy

    networks:
      - devops-network

    restart: unless-stopped


  database:
    image: postgres:16-alpine
    container_name: devops-database

    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

    ports:
      - "5432:5432"

    volumes:
      - postgres_data:/var/lib/postgresql/data

    networks:
      - devops-network

    restart: unless-stopped

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5


networks:
  devops-network:
    driver: bridge


volumes:
  postgres_data:
5. Understand the App service

This section:

app:
  build:
    context: .
    dockerfile: Dockerfile

means Docker Compose builds your Flask application using the Dockerfile in the project root.

The image will be:

image: devops-final:v1

So your application image is:

devops-final:v1
Port mapping
ports:
  - "5000:5000"

This means:

Host Port       Container Port
    5000    →       5000

You can access:

http://localhost:5000
6. Application environment variables

The app receives:

environment:
  DB_HOST: ${DB_HOST}
  DB_PORT: ${DB_PORT}
  POSTGRES_DB: ${POSTGRES_DB}
  POSTGRES_USER: ${POSTGRES_USER}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

The values come from .env.

For example:

DB_HOST=database

The important part is:

database

This is the Compose service name.

Inside the Docker Compose network, the application can reach PostgreSQL using:

database:5432

You do not use:

localhost:5432

from the Flask container.

7. Why depends_on is used

We have:

depends_on:
  database:
    condition: service_healthy

This tells Compose:

Start PostgreSQL
       ↓
Wait for PostgreSQL health check
       ↓
Start application

This is better than simply:

depends_on:
  - database

because the latter only controls startup order; it does not guarantee that PostgreSQL is ready to accept connections.

8. PostgreSQL service

Our database uses:

image: postgres:16-alpine

This is a PostgreSQL image based on Alpine Linux.

The database receives:

environment:
  POSTGRES_DB: ${POSTGRES_DB}
  POSTGRES_USER: ${POSTGRES_USER}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

From .env:

Database:  devopsdb
User:      devopsuser
Password:  devopspass
9. PostgreSQL port
ports:
  - "5432:5432"

This means:

Host
 │
 │ 5432
 ▼
PostgreSQL Container
 │
 │ 5432
 ▼
PostgreSQL

For communication between Compose containers, the application uses:

database:5432

The host port mapping is mainly useful if you want to connect to PostgreSQL from your host/development environment.

10. Named volume

This is very important.

volumes:
  - postgres_data:/var/lib/postgresql/data

PostgreSQL stores its database data inside:

/var/lib/postgresql/data

The named volume:

postgres_data

keeps the database data outside the temporary container filesystem.

Architecture:

PostgreSQL Container
        │
        ▼
/var/lib/postgresql/data
        │
        ▼
postgres_data
        │
        ▼
Persistent Docker Volume

If the PostgreSQL container is recreated, the data can remain in the named volume.

11. Docker network

We define:

networks:
  devops-network:
    driver: bridge

Both services use it:

networks:
  - devops-network

Therefore:

devops-app
     │
     │ devops-network
     │
     ▼
devops-database

Docker's internal DNS allows the app to find PostgreSQL using:

database

So the database address is:

database:5432
12. Restart configuration

Both services contain:

restart: unless-stopped

This means Docker will attempt to restart the container if it stops unexpectedly.

The container won't automatically keep restarting if you intentionally stop it.

13. PostgreSQL health check

This section:

healthcheck:
  test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
  interval: 10s
  timeout: 5s
  retries: 5

checks whether PostgreSQL is ready.

The command:

pg_isready

checks PostgreSQL availability.

The flow is:

PostgreSQL starts
       ↓
Health check
       ↓
Is PostgreSQL ready?
       │
    ┌──┴───┐
    │      │
   No     Yes
    │      │
 Retry    Healthy
           │
           ▼
      App can start
14. Validate the Compose file

Before starting anything, run:

docker compose config

If your configuration is correct, Docker Compose will print the resolved configuration.

You can also run:

docker compose config --quiet

If there is no output, the Compose configuration is valid.

15. Build and start everything

Run:

docker compose up -d --build

This does two things:

--build
   ↓
Build Flask image

up -d
   ↓
Start containers in background

You should see Compose creating:

devops-app
devops-database

and the network/volume.

16. Check containers

Run:

docker compose ps

You should see something similar to:

NAME               IMAGE                 STATUS
devops-app         devops-final:v1      Up
devops-database    postgres:16-alpine   Up (healthy)

The important part is:

devops-database    Up (healthy)
17. Check application logs
docker compose logs app

Or:

docker logs devops-app

You should see Flask running.

18. Check PostgreSQL logs
docker compose logs database

You should see PostgreSQL startup messages.

A successful startup normally contains a message indicating that the database system is ready to accept connections.

19. Test the Flask application

Run:

curl http://localhost:5000/

Then:

curl http://localhost:5000/health

You should receive the application's current response.

Again, remember that your current Flask code reports:

"database": "connected"

statically. The PostgreSQL container is now running, but your Flask code does not yet perform a real PostgreSQL connection.

20. Check the PostgreSQL volume

Run:

docker volume ls

You should see a volume similar to:

DRIVER    VOLUME NAME
local     devops-docker-final-project_postgres_data

The exact prefix may vary depending on your Compose project name.

Inspect it:

docker volume inspect devops-docker-final-project_postgres_data
21. Check the network

Run:

docker network ls

You should see something similar to:

devops-docker-final-project_devops-network

Inspect it:

docker network inspect devops-docker-final-project_devops-network

You should find both:

devops-app
devops-database

connected to the network.

22. Test PostgreSQL directly

You can enter the PostgreSQL container:

docker exec -it devops-database psql -U devopsuser -d devopsdb

You should get a PostgreSQL prompt:

devopsdb=#

Run:

\conninfo

You should see connection information.

Exit:

\q
23. Test persistence

This is a very good practical demonstration of the named volume.

First create a table:

docker exec -it devops-database psql -U devopsuser -d devopsdb

Inside PostgreSQL:

CREATE TABLE training (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

Insert data:

INSERT INTO training (name) VALUES ('DevOps Training');

Check:

SELECT * FROM training;

Exit:

\q

Now remove the database container:

docker compose rm -sf database

Start it again:

docker compose up -d database

Check the data:

docker exec -it devops-database psql -U devopsuser -d devopsdb

Then:

SELECT * FROM training;

The data should still exist because it is stored in:

postgres_data

rather than only inside the container.

24. Final architecture for your report
                         Docker Compose
                              │
               ┌──────────────┴──────────────┐
               │                             │
               ▼                             ▼
        ┌─────────────┐               ┌──────────────┐
        │     App     │               │  PostgreSQL  │
        │  Flask      │               │   Database   │
        │             │               │              │
        │ Port: 5000  │               │ Port: 5432   │
        └──────┬──────┘               └──────┬───────┘
               │                             │
               │      devops-network         │
               └─────────────┬───────────────┘
                             │
                             ▼
                     ┌──────────────┐
                     │ postgres_data│
                     │ Named Volume │
                     └──────────────┘
25. Commands you should demonstrate

For the training task, these are the key commands:

docker compose config
docker compose up -d --build
docker compose ps
docker compose logs app
docker compose logs database
docker volume ls
docker network ls
curl http://localhost:5000/health

And finally:

git add docker-compose.yml .env.example .gitignore
git commit -m "Added Docker Compose app and PostgreSQL architecture"
git push