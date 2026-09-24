Day 31 — GitHub Actions Workflow Architecture
Introduction

GitHub Actions is GitHub's built-in CI/CD platform.

It allows developers to automate:

Build processes
Testing
Code validation
Docker image creation
Terraform execution
Application deployment

Instead of manually running commands, GitHub Actions automatically executes workflows when specific events occur.

What is GitHub Actions?

GitHub Actions is an automation service inside GitHub repositories.

It listens for events such as:

Push
Pull Request
Tag Creation
Manual Trigger
Schedule

and automatically executes predefined workflows.

Why Use GitHub Actions?

Without automation:

Developer
   │
   ▼
Build Application
   │
   ▼
Run Tests
   │
   ▼
Deploy

Everything is done manually.

With GitHub Actions:

Developer Pushes Code
        │
        ▼
GitHub Actions
        │
        ▼
Build
        │
        ▼
Test
        │
        ▼
Deploy

Automation improves:

Speed
Reliability
Consistency
Quality
Workflow Location

GitHub Actions workflows are stored inside:

.github/
└── workflows/

Example:

.github/
└── workflows/
    └── terraform.yml

or

.github/
└── workflows/
    └── ci.yml

GitHub automatically detects workflow files in this location.

GitHub Actions Architecture
High-Level Structure
Workflow
   │
   └── Job
        │
        ├── Step
        ├── Step
        └── Step
Detailed Architecture
Workflow
    │
    ▼
Jobs
    │
    ├─────────────┐
    ▼             ▼
Build Job      Test Job
    │             │
    ▼             ▼
 Steps         Steps

A workflow contains one or more jobs.

Each job contains one or more steps.

Understanding Workflow Components
1. name

The name field defines the workflow name shown in GitHub.

Example:

name: Simple CI Pipeline

GitHub displays:

Actions
└── Simple CI Pipeline
2. on

The on section defines when the workflow should run.

Example:

on:
  push:

Meaning:

Developer Pushes Code
         │
         ▼
Workflow Starts
Common Triggers
Push
on:
  push:

Runs whenever code is pushed.

Pull Request
on:
  pull_request:

Runs when a PR is opened or updated.

Manual Trigger
on:
  workflow_dispatch:

Allows manual execution.

Multiple Triggers
on:
  push:
  pull_request:

Runs on both events.

3. jobs

A workflow contains jobs.

Example:

jobs:
  build:

Job name:

build

A job is a collection of related tasks.

Job Architecture
Job
 │
 ├── Checkout Code
 ├── Install Dependencies
 ├── Build
 └── Test
4. runs-on

Defines the runner machine.

Example:

runs-on: ubuntu-latest

GitHub creates:

Ubuntu Virtual Machine

for workflow execution.

Common Runners
runs-on: ubuntu-latest

Linux Runner

runs-on: windows-latest

Windows Runner

runs-on: macos-latest

macOS Runner

5. steps

Steps are individual tasks inside a job.

Example:

steps:

Each step performs one action.

Step Architecture
Job
 │
 ├── Step 1
 ├── Step 2
 ├── Step 3
 └── Step 4

Steps execute sequentially.

Common Steps
Checkout Repository

Downloads repository code.

- name: Checkout Repository
  uses: actions/checkout@v4
Run Commands
- name: Show Files
  run: ls -la

Executes shell commands.

Print Message
- name: Hello
  run: echo "Workflow Running"

Output:

Workflow Running
Workflow Execution Flow
Developer
    │
    ▼
Git Push
    │
    ▼
GitHub Detects Event
    │
    ▼
Workflow Starts
    │
    ▼
Job Starts
    │
    ▼
Step 1
    │
    ▼
Step 2
    │
    ▼
Step 3
    │
    ▼
Workflow Complete
Basic Workflow Example

Create:

.github/workflows/basic-ci.yml
Workflow Code
name: Basic CI Workflow

on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Display Message
        run: echo "Code push detected"

      - name: Show Current Directory
        run: pwd

      - name: List Files
        run: ls -la

      - name: Workflow Complete
        run: echo "CI Workflow Finished Successfully"
Workflow Explanation
Trigger
on:
  push:

Runs when code is pushed.

Job
jobs:
  build:

Creates a job named:

build
Runner
runs-on: ubuntu-latest

Uses a GitHub-hosted Ubuntu VM.

Step 1
uses: actions/checkout@v4

Downloads repository code.

Step 2
run: echo "Code push detected"

Prints a message.

Step 3
run: pwd

Shows the current working directory.

Step 4
run: ls -la

Lists repository files.

How to Test This Workflow
Step 1

Create:

.github/workflows/basic-ci.yml
Step 2

Commit changes:

git add .
git commit -m "Add basic GitHub Actions workflow"
Step 3

Push code:

git push origin main
Step 4

Open GitHub Repository.

Navigate to:

Repository
   │
   ▼
Actions Tab
Step 5

Observe workflow execution:

Basic CI Workflow
        │
        ▼
build
        │
        ▼
Success
Real DevOps Example

In production environments:

Push Code
    │
    ▼
GitHub Actions
    │
    ▼
Build Application
    │
    ▼
Run Tests
    │
    ▼
Build Docker Image
    │
    ▼
Push to ECR
    │
    ▼
Deploy to AWS

This is the foundation of a CI/CD pipeline.

Key Revision Points
Workflow
Complete automation definition
Job
Group of related tasks
Step
Single action within a job
name
Workflow name
on
Trigger event
jobs
Collection of jobs
steps
Tasks executed sequentially
Quick Interview Questions
Q1. Where are GitHub Actions workflows stored?

Answer:

.github/workflows/
Q2. What does on: do?

Answer: It defines the event that triggers the workflow.

Q3. What is a Job?

Answer: A job is a collection of steps executed on a runner.

Q4. What is a Step?

Answer: A step is an individual task within a job.

Q5. What happens when code is pushed?

Answer: GitHub detects the push event and automatically executes the workflow defined in .github/workflows/*.yml.