TASK 2 — Create Final Project Repository
Step 1: Create Repository on GitHub
Login to GitHub.
Click New Repository.
Repository Name:
devops-docker-final-project
Select:
Public

or

Private
Click Create Repository.
Step 2: Clone Repository

Copy the repository URL from GitHub.

Example:

git clone https://github.com/<username>/devops-docker-final-project.git

Move into the project directory:

cd devops-docker-final-project

Verify:

pwd
Step 3: Create Project Structure

Create folders:

mkdir app
mkdir scripts
mkdir docs

Create files:

touch Dockerfile

touch docker-compose.yml

touch .dockerignore

touch .gitignore

touch .env.example

touch README.md
Step 4: Verify Structure

Run:

tree

If tree is not installed:

sudo apt install tree -y

Expected Output:

devops-docker-final-project/
├── app/
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── .gitignore
├── .env.example
├── README.md
├── scripts/
└── docs/
Step 5: Add Initial README Content

Open README:

nano README.md

Add:

# DevOps Docker Final Project

This repository contains the final Docker project created during DevOps training.

## Project Structure

- app/ - Application source code
- scripts/ - Utility scripts
- docs/ - Documentation
- Dockerfile - Image build instructions
- docker-compose.yml - Multi-container configuration
- .env.example - Environment variable template

Save and exit.

Step 6: Initial Commit

Check status:

git status

Stage files:

git add .

Commit:

git commit -m "Initial project structure created"
Step 7: Push to GitHub

Rename branch if required:

git branch -M main

Push:

git push -u origin main
Complete Commands
git clone https://github.com/<username>/devops-docker-final-project.git

cd devops-docker-final-project

mkdir app scripts docs

touch Dockerfile

touch docker-compose.yml

touch .dockerignore

touch .gitignore

touch .env.example

touch README.md

tree

git status

git add .

git commit -m "Initial project structure created"

git push -u origin main