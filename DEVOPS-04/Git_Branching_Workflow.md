Task 6 – Git Branching Workflow
Objective

Learn how to:

Create branches
Switch between branches
Work independently on different features
Commit changes in separate branches
Understand real-world Git workflows
Step 1 – Check Current Branch
git branch

Example:

* main

The * indicates your current branch.

Step 2 – Create New Branches

Create three branches:

git branch feature-login

git branch feature-profile

git branch bugfix-readme

Verify:

git branch

Output:

bugfix-readme
feature-login
feature-profile
* main
Step 3 – Switch to feature-login Branch
git checkout feature-login

or

git switch feature-login

Verify:

git branch

Output:

* feature-login
Step 4 – Make Changes in feature-login

Add login feature notes:

echo "Login page feature added" >> notes.txt

Check:

git status

Stage:

git add .

Commit:

git commit -m "Added login feature notes"

Example:

[feature-login abc123]
Added login feature notes
Step 5 – Switch to feature-profile
git checkout feature-profile

or

git switch feature-profile
Step 6 – Make Changes in feature-profile

Add profile feature information:

echo "User profile feature created" >> README.md

Stage:

git add .

Commit:

git commit -m "Added profile feature documentation"
Step 7 – Switch to bugfix-readme
git checkout bugfix-readme

or

git switch bugfix-readme
Step 8 – Fix README

Add bug-fix information:

echo "README formatting corrected" >> README.md

Stage:

git add .

Commit:

git commit -m "Fixed README formatting issue"
Step 9 – Return to Main Branch
git switch main

Verify:

git branch

Output:

bugfix-readme
feature-login
feature-profile
* main
Verify Branch Commits

View all branches:

git branch

View commit history for current branch:

git log --oneline

View all branch history:

git log --oneline --all --graph

Example:

* a1b2c3d Fixed README formatting issue
* d4e5f6g Added profile feature documentation
* g7h8i9j Added login feature notes
* x1y2z3a Initial commit
Real-World Branching Example
main

Production-ready code.

feature-login

Development of login functionality.

feature-profile

Development of user profile functionality.

bugfix-readme

Fixes documentation issues.

This allows multiple developers to work simultaneously without affecting the main codebase.

Complete Commands
git branch feature-login

git branch feature-profile

git branch bugfix-readme

git branch

git checkout feature-login

echo "Login page feature added" >> notes.txt

git add .

git commit -m "Added login feature notes"

git checkout feature-profile

echo "User profile feature created" >> README.md

git add .

git commit -m "Added profile feature documentation"

git checkout bugfix-readme

echo "README formatting corrected" >> README.md

git add .

git commit -m "Fixed README formatting issue"

git switch main

git log --oneline --all --graph