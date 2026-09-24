                 CI/CD Pipeline Architecture


Introduction

A CI/CD pipeline is an automated process that takes code from a developer and checks whether the application is ready to continue to the next stage.

In this task, I created a simple CI/CD pipeline using Git, GitHub, and GitHub Actions.

The basic pipeline is:

Developer
    ↓
Git Commit
    ↓
Git Push
    ↓
GitHub Repository
    ↓
GitHub Actions
    ↓
Checkout Code
    ↓
Install Dependencies
    ↓
Run Tests
    ↓
Build
    ↓
Success / Failure


Each stage has a specific purpose.

1. Developer

The process starts with the developer.

The developer writes new code, modifies existing code, or fixes a bug in the application.

For example, in my Day 19 project, I created a simple calculator application.

Developer
    ↓
Write or modify code


The developer first tests the changes on their own computer before committing them.

2. Git Commit

After making changes, the developer uses Git to save the changes in a commit.

Example:

git add .
git commit -m "feat: update calculator"


A commit creates a record of the changes made to the project.

The commit message should describe what was changed.

For example:

feat: add calculator tests


The commit is still stored locally until it is pushed to GitHub.

3. Git Push

After creating a commit, the developer pushes the changes to the remote GitHub repository.

Example:

git push origin main


The push transfers the local Git commits to GitHub.

The flow is:

Local Computer
      ↓
Git Commit
      ↓
Git Push
      ↓
GitHub


A push can also act as a trigger for a GitHub Actions workflow.

4. GitHub Repository

The GitHub repository stores the project's source code and Git history.

It can contain:

src/
tests/
README.md
.gitignore
.github/
└── workflows/
    └── ci.yml


The repository is the central place where developers collaborate on the project.

Once the code is pushed, GitHub can detect the configured workflow trigger.

5. GitHub Actions

GitHub Actions is used to automate tasks such as testing and building the application.

The workflow is stored inside:

.github/workflows/


For example:

.github/workflows/ci.yml


When a push occurs on the configured branch, GitHub Actions starts the workflow.

The workflow tells GitHub what tasks need to be performed.

The flow is:

Git Push
    ↓
GitHub
    ↓
GitHub Actions
    ↓
Start CI Workflow

6. Checkout Code

The first major step in the workflow is checking out the repository.

Example:

- name: Checkout Code
  uses: actions/checkout@v4


This step gets the repository code onto the GitHub Actions runner.

The runner needs a copy of the source code before it can run commands or tests.

In simple terms:

Checkout means getting my project code onto the machine that is running the pipeline.

7. Install Dependencies

The application may require additional software or libraries.

For my Python project, I use pytest for automated testing.

The workflow can install it using:

python -m pip install --upgrade pip
pip install pytest


This prepares the runner to execute the tests.

The idea is:

Clean Runner
    ↓
Install Required Tools
    ↓
Ready to Test


Dependencies should be installed automatically so that the pipeline does not depend on my personal computer.

8. Run Tests

After installing the dependencies, the pipeline runs the automated tests.

For the Python project, the command is:

python -m pytest


The tests check whether the application behaves as expected.

For example:

add(2, 2)
    ↓
Expected: 4
    ↓
Actual: 4
    ↓
PASS


If all tests pass, the pipeline can continue.

If a test fails, the pipeline reports a failure.

Example:

Tests
  ↓
PASS ✓


or:

Tests
  ↓
FAIL ✗


Automated testing is important because it helps identify bugs before the application is released.

9. Build

The build stage prepares the application for the next stage.

The exact build process depends on the programming language and application.

For my small Python project, there is no complicated production build process.

As a basic build/validation step, Python files can be compiled using:

python -m compileall src


This checks that the Python source files can be compiled successfully.

For larger applications, the build stage might create:

Packages
Executables
Docker images
Deployment artifacts
Application bundles

Therefore, the build stage can be much more complex in a real production project.

10. Success / Failure

After the pipeline completes, GitHub Actions reports the result.

Successful Pipeline

If all required steps complete successfully:

Developer
    ↓
Git Commit
    ↓
Git Push
    ↓
GitHub Repository
    ↓
GitHub Actions
    ↓
Checkout Code       ✓
    ↓
Install Dependencies ✓
    ↓
Run Tests           ✓
    ↓
Build               ✓
    ↓
SUCCESS             ✓


A successful result means the configured pipeline steps completed without errors.

It does not necessarily mean that the application has zero bugs. It means the checks defined in the pipeline passed.

Failed Pipeline

If an important step fails:

Developer
    ↓
Git Commit
    ↓
Git Push
    ↓
GitHub Repository
    ↓
GitHub Actions
    ↓
Checkout Code       ✓
    ↓
Install Dependencies ✓
    ↓
Run Tests           ✗
    ↓
FAILURE             ✗


For example, if a test contains an incorrect expected value, pytest can fail.

The developer should then inspect the GitHub Actions logs to find the cause.

Complete Pipeline

The complete CI/CD pipeline can be represented as:

                    ┌─────────────┐
                    │  Developer  │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │ Git Commit  │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │  Git Push   │
                    └──────┬──────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ GitHub Repository  │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │   GitHub Actions   │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │   Checkout Code    │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ Install Dependencies│
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │     Run Tests      │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │       Build        │
                 └─────────┬──────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │   Result    │
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                 SUCCESS        FAILURE
                    ✓              ✗

What Happens When a Test Fails?

If the tests fail, the developer should not simply ignore the failure.

The troubleshooting process is:

Pipeline Failure
      ↓
Open GitHub Actions
      ↓
Open Failed Job
      ↓
Find Failed Step
      ↓
Read Error Logs
      ↓
Identify Root Cause
      ↓
Fix Code
      ↓
Run Tests Locally
      ↓
Git Commit
      ↓
Git Push
      ↓
Pipeline Runs Again
      ↓
SUCCESS


This creates a feedback loop between development and automation.

Where Does the Pipeline Execute?

The GitHub Actions workflow executes on a runner.

For example, the workflow may contain:

runs-on: ubuntu-latest


This means the job runs on a GitHub-hosted Ubuntu runner.

My local computer is used to write code, commit changes, and push them.

After the push, GitHub Actions performs the automated pipeline work on the configured runner.

My Computer
     │
     │ Git Push
     ▼
GitHub
     │
     ▼
GitHub Actions
     │
     ▼
GitHub Actions Runner
     │
     ├── Checkout Code
     ├── Install Dependencies
     ├── Run Tests
     └── Build
     │
     ▼
Success / Failure

Why This Pipeline Is Useful

This pipeline reduces the amount of manual work required from developers.

Without CI, I might have to remember to:

Push Code
   ↓
Download/Update Code
   ↓
Install Dependencies
   ↓
Run Tests
   ↓
Build


With CI, these steps happen automatically after the configured Git event.

This provides faster feedback and helps catch problems earlier.

Conclusion

The CI/CD pipeline connects development, Git, GitHub, automated testing, and build processes.

The complete flow is:

Developer
    ↓
Git Commit
    ↓
Git Push
    ↓
GitHub Repository
    ↓
GitHub Actions
    ↓
Checkout Code
    ↓
Install Dependencies
    ↓
Run Tests
    ↓
Build
    ↓
Success / Failure


The main thing I learned from this architecture is that a CI pipeline is not just a green status on GitHub. It is a sequence of automated steps executed by a runner.

If every required step completes successfully, the pipeline passes. If an important step fails, the logs can be used to identify the problem, fix it, and run the pipeline again.