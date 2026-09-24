Task 4 – Stage, Commit & View History
Objective

Learn how to:

Stage files
Create commits
Track project history
View commit details
Understand Git workflow
Step 1 – Check Repository Status
git status

Example:

On branch main

No commits yet

Untracked files:
 README.md
 notes.txt
 commands.txt
Step 2 – Stage Files

Add all files to the staging area:

git add .

Verify:

git status

Example:

Changes to be committed:
 new file: README.md
 new file: notes.txt
 new file: commands.txt
What is Staging?

The staging area acts like a waiting room where changes are prepared before creating a commit.

Step 3 – Create First Commit
git commit -m "Initial commit"

Example Output:

[main abc1234] Initial commit
 3 files changed
Step 4 – View Commit History

View detailed history:

git log

Example:

commit abc123456789
Author: Sathvik Jakka
Date: Sun Sep 13

Initial commit
Step 5 – View Short History
git log --oneline

Example:

abc1234 Initial commit
Step 6 – View Commit Details
git show

Shows:

Commit ID
Author
Date
Files changed
Added/removed content
Create Additional Commits
Commit 2 – Update README

Add content:

echo "# DevOps Git Practice" >> README.md

Check:

git status

Stage:

git add .

Commit:

git commit -m "Added project title to README"
Commit 3 – Add Notes

Add learning notes:

echo "Git is a Version Control System" >> notes.txt

Stage and commit:

git add .
git commit -m "Added Git theory notes"
Commit 4 – Add Commands

Add commands:

echo "git init" >> commands.txt
echo "git status" >> commands.txt

Commit:

git add .
git commit -m "Added Git commands reference"
Commit 5 – Update Notes

Add more content:

echo "Git helps track source code changes" >> notes.txt

Commit:

git add .
git commit -m "Updated notes with Git benefits"
Verify All 5 Commits

Run:

git log --oneline

Expected Output:

e5f6g7h Updated notes with Git benefits
d4e5f6g Added Git commands reference
c3d4e5f Added Git theory notes
b2c3d4e Added project title to README
a1b2c3d Initial commit
Meaningful Commit Messages Used
Initial commit
Added project title to README
Added Git theory notes
Added Git commands reference
Updated notes with Git benefits
Complete Commands
git add .

git status

git commit -m "Initial commit"

git log

git log --oneline

git show

echo "# DevOps Git Practice" >> README.md

git add .
git commit -m "Added project title to README"

echo "Git is a Version Control System" >> notes.txt

git add .
git commit -m "Added Git theory notes"

echo "git init" >> commands.txt
echo "git status" >> commands.txt

git add .
git commit -m "Added Git commands reference"

echo "Git helps track source code changes" >> notes.txt

git add .
git commit -m "Updated notes with Git benefits"

git log --oneline