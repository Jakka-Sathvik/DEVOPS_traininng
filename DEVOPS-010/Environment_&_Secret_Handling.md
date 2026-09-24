1. Go to your project
cd devops-docker-final-project

Check the files:

ls -la

You should have:

.env
.env.example
.gitignore
Dockerfile
docker-compose.yml
2. Create the real .env

If it doesn't already exist:

nano .env

For your training environment, use:

DATABASE_NAME=devopsdb
DATABASE_USER=devopsuser
DATABASE_PASSWORD=devopspass
DATABASE_HOST=database
DATABASE_PORT=5432
What these mean
Variable	Purpose
DATABASE_NAME	PostgreSQL database name
DATABASE_USER	PostgreSQL username
DATABASE_PASSWORD	Database password
DATABASE_HOST	Docker Compose database service
DATABASE_PORT	PostgreSQL port

The important value is:

DATABASE_HOST=database

because your PostgreSQL service in docker-compose.yml is named:

database:
3. Create .env.example

This file can be committed to Git because it contains no real password.

nano .env.example

Put:

DATABASE_NAME=
DATABASE_USER=
DATABASE_PASSWORD=
DATABASE_HOST=
DATABASE_PORT=

Or you can provide example values without exposing the actual password:

DATABASE_NAME=your_database_name
DATABASE_USER=your_database_user
DATABASE_PASSWORD=your_database_password
DATABASE_HOST=database
DATABASE_PORT=5432

For your assignment, I recommend the first version because it clearly demonstrates that .env.example contains only the required variable names.

4. Add .env to .gitignore

Open:

nano .gitignore

Make sure it contains:

.env

A good .gitignore for this project is:

.env

__pycache__/
*.py[cod]

.venv/
venv/

.pytest_cache/

.git/

Do not add .env.example to .gitignore.

You want:

.env          → IGNORE
.env.example  → TRACK
5. Verify Git's ignore rule

Run:

git status --short

If .env is correctly ignored, you should not see:

?? .env

You may see:

?? .env.example

if .env.example has not been committed yet.

That's correct.

6. Explicitly check .env

Run:

git check-ignore -v .env

Expected output will look similar to:

.gitignore:1:.env    .env

This confirms Git is ignoring .env.

7. Check .env.example

Run:

git status

You should see .env.example as an untracked file if it hasn't been committed.

For example:

Untracked files:
  .env.example

You should not see:

.env
8. Check what Git will commit

Run:

git add .env.example .gitignore

Then:

git status

You should see something similar to:

Changes to be committed:

    new file:   .env.example
    modified:   .gitignore

And importantly:

.env

should not be listed.

9. Verify .env is actually ignored

Run:

git status --short --ignored

You should see something similar to:

!! .env

!! means Git is ignoring the file.

This is an excellent screenshot for your training evidence.

10. Commit the safe files

Only commit:

git add .env.example .gitignore

Then:

git commit -m "Added environment configuration and secret handling"

Push:

git push

The result should be:

.env
   ↓
NOT committed

.env.example
   ↓
Committed

.gitignore
   ↓
Committed
11. Important security verification

Run:

git status

You should have a clean working tree or only unrelated changes.

Then check whether .env is tracked:

git ls-files .env

Expected: no output.

This means .env is not tracked by Git.

Check .env.example:

git ls-files .env.example

Expected:

.env.example

This confirms the correct setup.

12. Final expected structure
devops-docker-final-project/
│
├── app/
│   ├── app.py
│   └── requirements.txt
│
├── Dockerfile
├── docker-compose.yml
│
├── .env              ← REAL CONFIG, NOT COMMITTED
├── .env.example      ← SAFE TEMPLATE, COMMITTED
├── .gitignore        ← contains .env
│
├── README.md
├── scripts/
└── docs/
TASK 6 checklist
Requirement	Status
Create .env	✅
Store development configuration	✅
Create .env.example	✅
No real password in .env.example	✅
Add .env to .gitignore	✅
Verify git status	✅
.env does not appear as commit candidate	✅
.env.example can be committed	✅
Verify with git check-ignore	✅
Best screenshots for submission

Take screenshots of these three commands:

git status
git check-ignore -v .env
git status --short --ignored

The most important result is that .env does not appear as a file ready to commit, while .env.example does.