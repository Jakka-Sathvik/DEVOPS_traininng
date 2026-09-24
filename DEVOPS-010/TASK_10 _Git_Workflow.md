TASK 10 — Git Workflow
1. Check your current branch

From your project directory:

cd devops-docker-final-project

git status
git branch

You should ideally start from main.

If you are not on main:

git switch main

Then get the latest changes:

git pull origin main
2. Create the develop branch

Create and switch to develop:

git switch -c develop

Push it to GitHub:

git push -u origin develop

Check:

git branch

Expected:

* develop
  main

Your GitHub repository should now contain:

main
develop
3. Create feature/docker-app

This branch will contain the Docker application/Dockerfile work.

Make sure you are on develop:

git switch develop

Create the feature branch:

git switch -c feature/docker-app

Check:

git branch

Expected:

* feature/docker-app
  develop
  main
4. Make a meaningful change

For example, verify your Dockerfile:

cat Dockerfile

Your Dockerfile should contain something similar to:

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

Check the project:

docker build -t devops-final:v1 .

If successful, commit:

git add Dockerfile .dockerignore
git commit -m "feat: add dockerfile"

Push:

git push -u origin feature/docker-app
5. Create feature/database

Switch back to develop:

git switch develop

Create the branch:

git switch -c feature/database

Check:

git branch

Now make your database-related changes in:

docker-compose.yml

For example, your PostgreSQL service should include:

database:
  image: postgres:16-alpine
  container_name: devops-database

  environment:
    POSTGRES_DB: ${POSTGRES_DB}
    POSTGRES_USER: ${POSTGRES_USER}
    POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

  volumes:
    - postgres_data:/var/lib/postgresql/data

  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
    interval: 10s
    timeout: 5s
    retries: 5

Test:

docker compose config --quiet

Then commit:

git add docker-compose.yml
git commit -m "feat: add database service"

Push:

git push -u origin feature/database
6. Create bugfix/health-check

Switch to develop:

git switch develop

Create the bugfix branch:

git switch -c bugfix/health-check

Your Flask application already has:

/health

Test it:

curl http://localhost:5000/health

Expected:

{
  "application": "running",
  "database": "connected",
  "status": "success"
}

If you make an improvement/fix to the health-check implementation, commit it:

git add app/app.py
git commit -m "fix: resolve database connection issue"

Push:

git push -u origin bugfix/health-check

Note: your current /health endpoint reports "database": "connected" as demo/static output; it does not actually connect Flask to PostgreSQL. If you have not implemented a real DB connection, use a more accurate commit such as fix: improve application health check rather than claiming a real database connection was fixed.

7. Your branches

At this point you should have:

main
develop
feature/docker-app
feature/database
bugfix/health-check

Check them:

git branch -a

You should see something similar to:

* bugfix/health-check
  develop
  feature/database
  feature/docker-app
  main

  remotes/origin/develop
  remotes/origin/feature/database
  remotes/origin/feature/docker-app
  remotes/origin/bugfix/health-check
  remotes/origin/main
8. Meaningful commit history

Check your commits:

git log --oneline --all --decorate --graph

A good history could look like:

* fix: improve application health check
* feat: add database service
* feat: add dockerfile
* docs: add deployment instructions
* Initial project setup

You can also add documentation:

git switch develop

Edit your README.md and add Docker deployment instructions.

Then:

git add README.md
git commit -m "docs: add deployment instructions"
git push origin develop
9. Merge feature branches into develop

A common workflow is:

feature branch
      ↓
   Pull Request
      ↓
   develop
      ↓
   testing
      ↓
     main

For example:

git switch develop
git pull origin develop

Then merge your Docker feature:

git merge feature/docker-app

Then:

git push origin develop

Merge database:

git merge feature/database
git push origin develop

Merge health-check:

git merge bugfix/health-check
git push origin develop
10. Create a Pull Request on GitHub

For the assignment, you need at least one Pull Request.

The easiest option is:

feature/docker-app
        ↓
      develop

Go to your GitHub repository.

You should see a notification such as:

feature/docker-app had recent pushes
Compare & pull request

Click:

Compare & pull request

Set:

base repository: your repository
base: develop

compare: feature/docker-app

Title:

feat: add production Docker configuration

Description:

## Changes

- Added production-style Dockerfile
- Added non-root application user
- Added Docker ignore configuration
- Configured Flask application container
- Exposed application port 5000

## Testing

- Docker image builds successfully
- Application container starts successfully
- /health endpoint tested successfully

Click:

Create pull request

11. Review the Pull Request

On the Pull Request page, check:

Files changed
Commits
Conversation
Checks

Make sure there are no problems.

For your training demonstration, you can review your own PR if you are the repository owner.

12. Merge the Pull Request

Click:

Merge pull request

Then:

Confirm merge

Your workflow is now:

feature/docker-app
        │
        │ Pull Request
        ↓
     develop
        │
        │ testing
        ↓
       main
13. Update local repository

After merging:

git switch develop
git pull origin develop

Then:

git log --oneline --graph --decorate --all
14. Merge develop into main

Once everything has been tested:

git switch main
git pull origin main

Merge:

git merge develop

Push:

git push origin main
15. Final GitHub structure

Your GitHub repository should now have:

main
│
├── Docker application
├── PostgreSQL
├── Docker Compose
├── Health check
├── Persistent volume
└── Documentation


develop
│
├── feature/docker-app
├── feature/database
└── bugfix/health-check

Conceptually:

                    GitHub
                       │
              ┌────────┴────────┐
              │                 │
             main            develop
              │                 │
              │        ┌────────┼────────┐
              │        │        │        │
              │   feature/  feature/  bugfix/
              │   docker    database  health-check
              │
              └──── Production
16. Final verification

Run:

git status

Then:

git branch -a

Then:

git log --oneline --all --decorate --graph

Finally verify GitHub:

✓ main branch
✓ develop branch
✓ feature/docker-app
✓ feature/database
✓ bugfix/health-check
✓ Meaningful commits
✓ Project pushed to GitHub
✓ Pull Request created
✓ Pull Request reviewed
✓ Pull Request merged