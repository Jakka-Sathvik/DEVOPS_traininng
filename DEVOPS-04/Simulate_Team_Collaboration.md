Task 8 – Simulate Team Collaboration
Objective

Understand how multiple developers work on different features, bug fixes, and releases using Git branches.

Branch Structure
main
│
└── develop
      │
      ├── feature/auth
      │
      ├── feature/profile
      │
      ├── bugfix/login
      │
      └── release/v1.0
Branch Purpose
Branch	Purpose
main	Production-ready code
develop	Integration branch
feature/auth	Authentication feature
feature/profile	User profile feature
bugfix/login	Fix login issues
release/v1.0	Release preparation
Step 1 – Create Project Repository
mkdir Team-Collaboration

cd Team-Collaboration

git init

Create README:

echo "# Team Collaboration Project" > README.md

Stage and Commit:

git add .

git commit -m "Initial project setup"
Step 2 – Create Develop Branch
git branch develop

git switch develop

Verify:

git branch

Output:

* develop
main
Step 3 – Create Feature Branches
Feature Auth
git branch feature/auth

git switch feature/auth

Create file:

echo "Authentication Module" > auth.txt

Commit:

git add .

git commit -m "Added authentication feature"
Feature Profile

Switch:

git switch develop

Create branch:

git branch feature/profile

git switch feature/profile

Create file:

echo "User Profile Module" > profile.txt

Commit:

git add .

git commit -m "Added profile feature"
Bug Fix Branch

Switch:

git switch develop

Create:

git branch bugfix/login

git switch bugfix/login

Create file:

echo "Fixed login validation issue" > login-fix.txt

Commit:

git add .

git commit -m "Fixed login validation bug"
Release Branch

Switch:

git switch develop

Create:

git branch release/v1.0

git switch release/v1.0

Create file:

echo "Release Version 1.0" > release-notes.txt

Commit:

git add .

git commit -m "Prepared release v1.0"
Step 4 – Merge Feature Branches into Develop
Merge Authentication Feature
git switch develop

git merge feature/auth

Expected Output:

Merge made by the 'ort' strategy.
Merge Profile Feature
git merge feature/profile

Expected Output:

Merge made by the 'ort' strategy.
Merge Bug Fix
git merge bugfix/login

Expected Output:

Merge made by the 'ort' strategy.
Step 5 – Verify Develop Branch
ls

Expected Files:

README.md
auth.txt
profile.txt
login-fix.txt

View history:

git log --oneline --graph
Step 6 – Merge Develop into Main

Switch to main:

git switch main

Merge:

git merge develop

Expected Output:

Updating abc123..xyz456
Fast-forward
Step 7 – Merge Release Branch
git merge release/v1.0
Step 8 – Verify Final Repository

Check branches:

git branch

Expected:

bugfix/login
develop
feature/auth
feature/profile
release/v1.0
* main
View Complete Workflow

Run:

git log --oneline --graph --all

Example:

* f6g7h8i Prepared release v1.0
* e5f6g7h Fixed login validation bug
* d4e5f6g Added profile feature
* c3d4e5f Added authentication feature
* b2c3d4e Initial project setup
Team Collaboration Workflow Summary
Developer A
      │
      └── feature/auth
                │
                ▼
             develop

Developer B
      │
      └── feature/profile
                │
                ▼
             develop

Developer C
      │
      └── bugfix/login
                │
                ▼
             develop

Develop Branch
       │
       ▼
Release Branch
       │
       ▼
Main Branch