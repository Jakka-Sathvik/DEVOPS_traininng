
DevOps Training Program – Day 4

Topic: Git Fundamentals, Repository Management, Commits, Branching and Collaboration

Prepared By: Sathvik Jakka

Date: 13-09-2026

1. Introduction

On Day 4 of the DevOps Training Program, I learned the fundamentals of Git and Version Control Systems. Git is one of the most widely used tools in software development and DevOps because it helps teams track source code changes, collaborate efficiently, and maintain project history.

The training covered Git concepts, repository creation, commit management, branch workflows, merging, and team collaboration practices. Practical exercises were performed using a local Git repository to understand real-world development workflows.

2. Git Concepts Learned
What is Version Control?

Version Control is a system that records changes made to files over time so that previous versions can be recovered whenever required.

Benefits:

Tracks source code changes
Maintains project history
Supports collaboration
Enables rollback to previous versions
Prevents accidental data loss
What is Git?

Git is a distributed Version Control System developed by Linus Torvalds.

Git allows developers to:

Track file changes
Create multiple development branches
Merge code safely
Collaborate with team members
Manage software releases
Git vs GitHub
Git	GitHub
Version Control System	Cloud Hosting Platform
Installed locally	Hosted online
Tracks changes	Stores repositories
Works offline	Primarily online

Git manages code history while GitHub provides remote repository hosting and collaboration features.

Repository

A Repository is a storage location containing:

Project files
Commit history
Branches
Git configuration

Repositories can be:

Local Repository
Remote Repository
Commit

A Commit is a saved snapshot of project changes.

Benefits:

Records project progress
Tracks modifications
Enables rollback
Provides historical records

Example:

git commit -m "Added login feature"
Branch

A Branch is an independent line of development.

Benefits:

Isolated development
Parallel work
Safe experimentation
Easier feature management

Example:

git branch feature-login
Merge

Merge combines changes from one branch into another.

Example:

git merge feature-login
Pull Request

A Pull Request is a request to merge code after review and testing.

Typical workflow:

Feature Branch
↓
Pull Request
↓
Code Review
↓
Approval
↓
Merge

3. Git Commands Used
Git Installation

Verify installation:

git --version

Example Output:

git version 2.x.x
Git Configuration

Configure username:

git config --global user.name "Sathvik Jakka"

Configure email:

git config --global user.email "your_email@example.com"

View configuration:

git config --list
Repository Initialization

Create project:

mkdir DevOps-Git
cd DevOps-Git

Initialize repository:

git init
Repository Status
git status

Used to check:

Tracked files
Untracked files
Staged files
Modified files
Staging Files
git add .

Moves files to the staging area.

Creating Commits
git commit -m "Initial commit"

Creates a snapshot of project changes.

Viewing History
git log

Short history:

git log --oneline

Detailed commit information:

git show
Branch Commands

Create branch:

git branch feature-login

Switch branch:

git checkout feature-login

or

git switch feature-login

View branches:

git branch
Merge Commands

Merge branch:

git merge feature-login

View graphical history:

git log --oneline --graph --all
4. Repository Screenshots

The following screenshots were captured during practical execution:

Screenshot 1

Git Installation Successful

Command:

git --version
Screenshot 2

Git Configuration

Command:

git config --list
Screenshot 3

Repository Initialization

Command:

git init
Screenshot 4

Repository Status

Command:

git status
Screenshot 5

Commit History

Command:

git log --oneline
Screenshot 6

Branch Creation

Command:

git branch
Screenshot 7

Merge Workflow

Command:

git log --oneline --graph --all
5. Commit History

During the exercise, multiple commits were created.

Example commit history:

a1b2c3d Initial commit
b2c3d4e Added project title to README
c3d4e5f Added Git theory notes
d4e5f6g Added Git commands reference
e5f6g7h Updated notes with Git benefits

Purpose of commits:

Track progress
Save project checkpoints
Record changes
Support rollback
Improve collaboration
6. Branch Workflow

The following branching workflow was practiced.

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
Workflow Explanation
Main Branch

Contains production-ready code.

Develop Branch

Integration branch where completed features are merged.

Feature Branches

Used to develop new functionality independently.

Examples:

feature/auth
feature/profile
Bug Fix Branch

Used to resolve issues.

Example:

bugfix/login
Release Branch

Prepared software release versions.

Example:

release/v1.0
7. Merge Conflict Explanation

A Merge Conflict occurs when Git cannot automatically combine changes from different branches.

Example Scenario:

Developer A modifies:

Login Button = Blue

Developer B modifies:

Login Button = Green

When both branches are merged, Git does not know which change should be kept.

Conflict Example:

<<<<<<< HEAD
Login Button = Blue
=======
Login Button = Green
>>>>>>> feature-login

Resolution Steps:

Open conflicting file.
Choose correct content.
Remove conflict markers.
Save file.
Stage changes.
Commit merge resolution.

Commands:

git add .
git commit -m "Resolved merge conflict"
8. Problems Faced

During the practical exercises, several common challenges were encountered.

Issue 1

Forgot to stage files before committing.

Solution:

git add .

before running commit.

Issue 2

Switched to the wrong branch.

Solution:

git branch

to verify the active branch.

Issue 3

Confusion between Git and GitHub.

Solution:

Understood that Git manages code history while GitHub hosts repositories online.

Issue 4

Understanding merge workflow.

Solution:

Practiced branch creation, switching, and merging multiple times.

9. Summary

Day 4 provided a strong foundation in Git and Version Control concepts. I learned repository management, commit creation, branch workflows, merging, and collaboration practices. Practical exercises helped me understand how software teams manage source code and coordinate development activities.

The hands-on tasks improved my understanding of Git commands, commit history tracking, branching strategies, and merge operations. These skills are essential for DevOps Engineers, Cloud Engineers, Software Developers, and Site Reliability Engineers.

Overall, Day 4 was highly beneficial in understanding modern source code management and collaborative software development workflows.