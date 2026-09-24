Task 3 – Create Your First Local Repository
Objective

Learn how to create a local Git repository, add project files, and check repository status.

Step 1: Create Project Folder

Open Terminal and run:

mkdir DevOps-Git

This creates a new directory named DevOps-Git.

Move into the directory:

cd DevOps-Git

Verify your location:

pwd

Example Output:

/home/sathvik/DevOps-Git
Step 2: Initialize Git Repository

Run:

git init

Example Output:

Initialized empty Git repository in /home/sathvik/DevOps-Git/.git/
Explanation
Creates a hidden .git directory.
Stores Git history, configuration, branches, and commits.
Converts a normal folder into a Git repository.
Screenshot 1

Capture the terminal showing:

git init

and the message:

Initialized empty Git repository...
Step 3: Create Project Files

Create README file:

touch README.md

Create notes file:

touch notes.txt

Create commands file:

touch commands.txt

Check files:

ls

Example Output:

README.md
commands.txt
notes.txt
Explanation

README.md

Project documentation.
Describes the project purpose.

notes.txt

Stores learning notes.

commands.txt

Stores Git commands used during practice.
Screenshot 2

Capture:

ls

showing all three files.

Step 4: Check Repository Status

Run:

git status

Example Output:

On branch main

No commits yet

Untracked files:
  README.md
  commands.txt
  notes.txt

nothing added to commit but untracked files present
Explanation

Git shows:

Current branch
Commit status
Tracked files
Untracked files

Since files were just created, Git recognizes them as Untracked Files.

Understanding Git Status
Untracked Files

Files created but not yet added to Git.

Example:

README.md
notes.txt
commands.txt
Tracked Files

Files that Git is monitoring after using:

git add
Repository Status

git status helps developers:

See modified files
View untracked files
Check commit readiness
Identify repository state
Complete Commands Used
mkdir DevOps-Git

cd DevOps-Git

git init

touch README.md

touch notes.txt

touch commands.txt

ls

git status