TASK 10 — Manual Rollback Exercise

Training: DevOps Engineer
Topic: Manual Docker Rollback
Time: 45 Minutes
File: Rollback_Exercise.md

1. Objective

The objective of this task is to practice manually rolling back a failed application release to a previously working Docker image.

Two training versions will be used:

v1.0.1
v1.0.2

The exercise will:

Prepare version v1.0.1
Prepare version v1.0.2
Deploy v1.0.2
Verify the application
Simulate a failed release
Stop the failed version
Deploy v1.0.1
Perform health verification
Confirm the rollback succeeded
2. Rollback Architecture

Normal deployment:

v1.0.1
   ↓
Deploy v1.0.2
   ↓
Health Check
   ↓
PASS

Rollback:

v1.0.2
   ↓
FAIL
   ↓
Stop Failed Release
   ↓
Deploy v1.0.1
   ↓
Health Check
   ↓
PASS
3. Prerequisites

Before starting, verify Docker is installed:

docker --version

Verify Docker Compose:

docker compose version

Verify the project directory:

pwd

The application should contain a health endpoint:

GET /health

Expected response:

{
  "status": "healthy"
}
4. Application Versioning

For this exercise, two Docker image versions are required:

devops-final:v1.0.1
devops-final:v1.0.2

The important idea is that each version represents a specific application release.

Application Code
       ↓
Docker Build
       ↓
Versioned Image
       ↓
Deployment

Versioned images make rollback easier because the previous image can be identified and deployed again.

5. Step 1 — Prepare Version v1.0.1

First, make sure the application is working.

Build the first training version:

docker build -t devops-final:v1.0.1 .

Verify the image:

docker images

Expected:

REPOSITORY     TAG       IMAGE ID
devops-final   v1.0.1    <image-id>

The exact image ID will be different on each machine.

6. Step 2 — Prepare Version v1.0.2

Make a small controlled application change.

For example, update the application version displayed on the home page or add a simple version endpoint.

Example:

@app.route("/version")
def version():
    return {
        "version": "v1.0.2"
    }, 200

Build the second image:

docker build -t devops-final:v1.0.2 .

Verify:

docker images

Expected:

REPOSITORY     TAG
devops-final   v1.0.1
devops-final   v1.0.2

Now both releases are available.

7. Step 3 — Deploy v1.0.2

Stop any old training container using the same name:

docker rm -f devops-app 2>/dev/null || true

Start version v1.0.2:

docker run -d \
  --name devops-app \
  -p 5000:5000 \
  devops-final:v1.0.2

Check the container:

docker ps

Expected:

CONTAINER ID   IMAGE                  STATUS
<id>           devops-final:v1.0.2   Up ...
8. Step 4 — Verify Container

Check the running container:

docker ps --filter name=devops-app

Inspect the container:

docker inspect devops-app

Check which image is being used:

docker inspect devops-app --format '{{.Config.Image}}'

Expected:

devops-final:v1.0.2

This confirms that the container is running version v1.0.2.

9. Step 5 — Verify /health

Test the application:

curl -i http://localhost:5000/health

Expected:

HTTP/1.1 200 OK

Response:

{
  "status": "healthy"
}

The exact formatting may vary.

The important result is:

HTTP 200
status = healthy
10. Step 6 — Verify Version

If the application contains the /version endpoint:

curl http://localhost:5000/version

Expected:

{
  "version": "v1.0.2"
}

This confirms that the expected release is running.

11. Step 7 — Verify All Three Items

The task requires verification of:

/health
Version
Container
Health
curl http://localhost:5000/health

Expected:

{
  "status": "healthy"
}
Version
curl http://localhost:5000/version

Expected:

{
  "version": "v1.0.2"
}
Container
docker ps

Expected image:

devops-final:v1.0.2

Verification table:

Check	Expected Result	Status
/health	200 OK	PASS
Version	v1.0.2	PASS
Container	Running	PASS
Image	devops-final:v1.0.2	PASS
12. Step 8 — Record the Working Release

Before simulating failure, record the known-good release.

Current Version: v1.0.2
Health: PASS
Container: Running
Image: devops-final:v1.0.2

The previous working version is:

v1.0.1

This version will be used for rollback.

13. Step 9 — Simulate a Failed Release

For this training exercise, simulate a failed v1.0.2 release.

The failure can be simulated by stopping the container:

docker stop devops-app

Verify:

docker ps

The application should no longer appear as a running container.

Test the health endpoint:

curl -i http://localhost:5000/health

Expected result:

Connection refused

or another connection failure.

This represents an unavailable release.

14. Step 10 — Identify the Failed Release

Check all containers:

docker ps -a

Expected state:

devops-app
IMAGE: devops-final:v1.0.2
STATUS: Exited

Check the image:

docker inspect devops-app --format '{{.Config.Image}}'

Expected:

devops-final:v1.0.2

The failed release is therefore:

v1.0.2
15. Step 11 — Stop and Remove the Failed Container

Remove the failed training container:

docker rm devops-app

Verify:

docker ps -a

The failed container should no longer exist.

The Docker image itself can remain available:

devops-final:v1.0.2

This is useful for later investigation.

16. Step 12 — Roll Back to v1.0.1

Deploy the previously known-good version:

docker run -d \
  --name devops-app \
  -p 5000:5000 \
  devops-final:v1.0.1

Check:

docker ps

Expected:

IMAGE                  STATUS
devops-final:v1.0.1   Up ...

The rollback is now deployed.

17. Step 13 — Verify Rollback Container

Run:

docker inspect devops-app --format '{{.Config.Image}}'

Expected:

devops-final:v1.0.1

This confirms that the application is running the previous version.

18. Step 14 — Health Check After Rollback

Run:

curl -i http://localhost:5000/health

Expected:

HTTP/1.1 200 OK

Response:

{
  "status": "healthy"
}

The rollback health check has passed.

19. Step 15 — Version Check After Rollback

Run:

curl http://localhost:5000/version

Expected:

{
  "version": "v1.0.1"
}

This confirms that the previous version has been restored.

20. Complete Rollback Flow

The complete exercise is:

             v1.0.1
                │
                ↓
          Build v1.0.2
                │
                ↓
          Deploy v1.0.2
                │
                ↓
          Health Check
                │
                ↓
          Version Check
                │
                ↓
              PASS
                │
                ↓
        Simulate Failure
                │
                ↓
              FAIL
                │
                ↓
        Stop Failed Release
                │
                ↓
          Deploy v1.0.1
                │
                ↓
          Health Check
                │
                ↓
              PASS
                │
                ↓
        Rollback Successful
21. Before and After Rollback
Before Failure
Application
    ↓
Container
    ↓
devops-final:v1.0.2
    ↓
Healthy
After Failure
Application
    ↓
v1.0.2
    ↓
FAIL
After Rollback
Application
    ↓
Container
    ↓
devops-final:v1.0.1
    ↓
Healthy
22. Verification Table
Verification	v1.0.2	After Rollback
Container	Running	Running
Image	v1.0.2	v1.0.1
/health	PASS	PASS
Version	v1.0.2	v1.0.1
Final Status	Failed/Stopped	Restored
23. Useful Troubleshooting Commands
Check running containers
docker ps
Check all containers
docker ps -a
Check logs
docker logs devops-app
Follow logs
docker logs -f devops-app
Inspect container
docker inspect devops-app
Check image
docker inspect devops-app --format '{{.Config.Image}}'
Check images
docker images
Test health
curl -i http://localhost:5000/health
Test version
curl http://localhost:5000/version
24. Troubleshooting Example

If the rollback container does not start:

docker ps
     ↓
Container missing
     ↓
docker ps -a
     ↓
Check container status
     ↓
docker logs devops-app
     ↓
Identify error
     ↓
Fix problem
     ↓
Restart
     ↓
Health Check

Possible causes include:

Port already in use
Incorrect Docker image
Application startup failure
Missing environment variable
Incorrect configuration
Dependency failure
25. Important Rollback Rules
Rule 1 — Keep Previous Versions

Do not immediately delete the previous known-good image.

v1.0.1 ← Known Good
v1.0.2 ← New Release

Keeping the previous version makes rollback faster.

Rule 2 — Use Versioned Images

Prefer:

devops-final:v1.0.1
devops-final:v1.0.2

instead of relying only on:

devops-final:latest
Rule 3 — Verify After Rollback

Rollback is not complete just because the old container starts.

Always verify:

Container
   ↓
Health
   ↓
Version
Rule 4 — Investigate the Failed Release

After service restoration, investigate why v1.0.2 failed.

Useful information includes:

Container logs
Application logs
Deployment logs
Configuration
Image details
Recent code changes
26. Production Consideration

This exercise uses manual Docker commands for learning.

In a real production environment, rollback may be performed using deployment platforms such as:

CI/CD
Container Platform
Kubernetes
Amazon ECS
Other approved deployment systems

The principle remains the same:

New Version
     ↓
Verify
     ↓
FAIL
     ↓
Restore Known-Good Version
     ↓
Verify Again

Never perform a rollback or failure simulation against a production system without authorization.

27. Exercise Result

Complete the following after performing the exercise:

Application:
DevOps Training Application

Initial Version:
v1.0.2

Initial Health:
PASS

Initial Container:
Running

Simulated Failure:
YES

Failed Version:
v1.0.2

Rollback Version:
v1.0.1

Rollback Health:
PASS

Rollback Container:
Running

Final Version:
v1.0.1

Final Status:
ROLLBACK SUCCESSFUL
28. What I Learned

From this exercise, I learned how to perform a manual Docker rollback using two versioned application images.

I learned that a previous known-good version should be available before deploying a new release.

The rollback process is:

New Version
    ↓
Failure
    ↓
Stop Failed Version
    ↓
Deploy Previous Version
    ↓
Health Check
    ↓
Version Check
    ↓
Service Restored

I also learned that rollback verification is important. The previous version must not only be deployed, but also checked for:

Running container
Correct image version
Healthy application
Correct application version