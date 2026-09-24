TASK 9 — AWS CLI Introduction

File: AWS_CLI_Introduction.md
Training Topic: AWS Command Line Interface (AWS CLI)
Time: 1 Hour

1. Objective

The objective of this task is to understand and use the AWS Command Line Interface (AWS CLI).

AWS provides two common ways to interact with AWS resources:

AWS Console
      ↓
   Browser
      ↓
AWS Services

and:

AWS CLI
      ↓
   Terminal
      ↓
AWS Services

The AWS CLI is useful for DevOps because AWS operations can be performed from a terminal and incorporated into scripts and automation.

2. What is AWS CLI?

AWS CLI stands for AWS Command Line Interface.

It is a command-line tool that allows users to interact with AWS services from a terminal.

For example:

aws s3 ls

can be used to list S3 buckets available to the authenticated identity.

Another example:

aws ec2 describe-instances

can retrieve information about EC2 instances that the authenticated identity is authorized to see.

3. AWS Console vs AWS CLI
Feature	AWS Console	AWS CLI
Interface	Web browser	Terminal
Interaction	Click-based	Command-based
Automation	Limited/manual	Excellent
Scripts	Not normally used	Supported
DevOps usage	Useful	Very common
Repeatability	Manual	High
Learning curve	Beginner-friendly	Requires commands
4. AWS CLI Architecture
                  AWS
                   │
          ┌────────┴────────┐
          │                 │
          ▼                 ▼
     AWS Console         AWS CLI
          │                 │
       Browser           Terminal
          │                 │
          └────────┬────────┘
                   ▼
              AWS Services

The AWS CLI sends API requests to AWS services using the permissions of the authenticated identity.

5. Check Whether AWS CLI Is Installed

Open a terminal.

Run:

aws --version
Example output
aws-cli/2.x.x Python/3.x.x Linux/x86_64

The exact version will depend on the installed AWS CLI version and operating system.

If you see:

aws: command not found

or on Windows:

'aws' is not recognized as an internal or external command

then AWS CLI is either not installed or is not available in the system PATH.

6. Windows Installation

For Windows, install the official AWS CLI package according to your organization's approved software-installation process.

After installation, close and reopen PowerShell or Command Prompt.

Then run:

aws --version

Expected result:

aws-cli/2.x.x ...
7. Linux Installation

On a Linux development machine, use the organization's approved installation method.

After installation:

aws --version

Then verify that the command is available.

For a GitHub Codespace or other Linux development environment, first check:

which aws
aws --version

If AWS CLI is missing, install it using the approved method for that environment rather than copying credentials or binaries from an untrusted source.

8. macOS Installation

On macOS, install the AWS CLI using the organization's approved installation method.

Then open a new terminal and run:

aws --version
9. Verify the Installation

Run:

aws --version

Then:

aws help

You can also inspect command help:

aws s3 help

and:

aws ec2 help

This helps you understand the available AWS CLI commands.

10. AWS CLI Command Structure

A common AWS CLI command has this structure:

aws
 ↓
service
 ↓
operation
 ↓
options

Example:

aws s3 ls

Breakdown:

aws
 ↓
s3
 ↓
ls

Another example:

aws ec2 describe-instances

Breakdown:

aws
 ↓
ec2
 ↓
describe-instances
11. Configure AWS CLI

AWS CLI must authenticate with AWS before it can perform operations that require authentication.

Use only an approved authentication method.

Do not copy credentials from another person or use unauthorized credentials.

For environments where aws configure is approved, the command is:

aws configure

It may request:

AWS Access Key ID
AWS Secret Access Key
Default region name
Default output format

Example region:

ap-south-1

Example output format:

json
Important

Never place real credentials in:

GitHub
README files
Screenshots
Chat messages
Source code
12. Approved Authentication Methods

The authentication method depends on the organization's AWS account configuration.

Possible approaches include:

AWS IAM Identity Center / SSO
IAM role
Temporary credentials
Approved environment credentials
Approved IAM user credentials

For human users, organizations may prefer centralized identity systems such as AWS IAM Identity Center.

For AWS workloads such as EC2, IAM roles are generally preferred over storing long-lived access keys.

13. IAM Role Authentication

For an EC2 instance with an attached IAM role, the AWS CLI can use the instance's role credentials automatically.

Architecture:

EC2
 ↓
IAM Role
 ↓
Temporary Credentials
 ↓
AWS CLI
 ↓
AWS Services

This avoids manually storing long-lived access keys on the server.

14. Verify AWS Authentication

After configuring an approved authentication method, run:

aws sts get-caller-identity

This is one of the most useful AWS CLI verification commands.

It returns information about the identity currently being used.

Example structure:

{
    "UserId": "....",
    "Account": "....",
    "Arn": "arn:aws:iam::....:user/...."
}

The exact values depend on the authenticated identity.

Security note

Do not publish account identifiers or sensitive authentication information unnecessarily in public repositories or screenshots.

15. Check the Current AWS Region

You can inspect the configured region with:

aws configure get region

Example:

ap-south-1

A region can also be specified for individual commands:

aws s3 ls --region ap-south-1
16. AWS CLI Output Formats

AWS CLI supports different output formats.

Common examples include:

json
text
table
yaml
yaml-stream

For example:

aws ec2 describe-instances --output table

JSON is useful for automation because it is structured and can be processed by scripts.

17. Example S3 Commands
List S3 buckets
aws s3 ls
List objects in a bucket
aws s3 ls s3://training-bucket
Copy a local file to S3

Only perform this if the training identity has the required permission:

aws s3 cp test.txt s3://training-bucket/
Download an object
aws s3 cp s3://training-bucket/test.txt .

Permissions determine whether these commands succeed.

18. Example EC2 Commands

List EC2 instances:

aws ec2 describe-instances

Get a concise table:

aws ec2 describe-instances --output table

The command only returns information that the authenticated identity is authorized to access.

19. AWS CLI and IAM Permissions

AWS CLI does not bypass IAM.

For example:

AWS CLI
   │
   ▼
Authenticated Identity
   │
   ▼
IAM Permissions
   │
   ▼
AWS Service

Suppose a user has:

s3:GetObject

but does not have:

s3:DeleteObject

The CLI may successfully execute:

aws s3 cp s3://bucket/file.txt .

but a delete operation can fail because the required permission is missing.

20. Permission Error Example

If the identity does not have permission, AWS may return an error similar to:

An error occurred (AccessDenied) when calling the ...
operation: Access Denied

The troubleshooting process is:

Command fails
    ↓
Read error
    ↓
Identify AWS service
    ↓
Identify operation
    ↓
Check IAM permissions
    ↓
Check resource
    ↓
Check authentication
    ↓
Fix authorized configuration

Do not solve an AccessDenied problem by automatically granting AdministratorAccess.

First determine the exact permission required.

21. Useful AWS CLI Commands
Check CLI version
aws --version
Show help
aws help
Check identity
aws sts get-caller-identity
Check configured region
aws configure get region
List S3 buckets
aws s3 ls
List EC2 instances
aws ec2 describe-instances
22. AWS CLI in DevOps

AWS CLI is widely useful in DevOps workflows.

For example:

Developer
    ↓
GitHub
    ↓
CI/CD Pipeline
    ↓
AWS CLI / SDK
    ↓
AWS Services

A pipeline may use AWS CLI commands to:

Deploy applications
Upload artifacts
Interact with S3
Query AWS resources
Publish deployment information
Trigger AWS operations

Authentication should use the organization's approved role-based or temporary-credential mechanism whenever possible.

23. Credential Security

AWS credentials are sensitive.

Never place real credentials in:

❌ GitHub
❌ README.md
❌ Source code
❌ Dockerfiles
❌ Screenshots
❌ Chat messages
❌ Public documentation
❌ Hard-coded scripts

Do not commit files containing credentials.

For example, if a local configuration file contains secrets, ensure it is excluded from Git according to the organization's policy.

24. Example .gitignore

A project may use:

.env
.aws/
*.pem

However, .gitignore is not a substitute for secret management.

If a credential has already been committed to a repository, simply adding it to .gitignore does not remove it from Git history. The exposed credential should be revoked/rotated according to the organization's security process.

25. Safe Screenshot Practice

For this training task, screenshots can show:

AWS CLI version
CLI commands
Non-sensitive command output

Example:

$ aws --version
aws-cli/2.x.x ...

Avoid showing:

Passwords
Secret Access Keys
Session Tokens
Credential files
Private keys

If sensitive information appears, redact it before submitting the screenshot.

26. Practical Training Exercise

Perform the following steps.

Step 1 — Check installation
aws --version

Record:

AWS CLI Version:
____________________
Step 2 — Check help
aws help
Step 3 — Configure using an approved authentication method

Use your organization's approved method.

Do not paste credentials into this document.

Step 4 — Verify identity
aws sts get-caller-identity
Step 5 — Check region
aws configure get region
Step 6 — Test an authorized AWS service

For example:

aws s3 ls

Only perform operations that your training identity is authorized to perform.

27. Troubleshooting
Problem 1 — aws command not found

Check:

which aws

On Windows:

where.exe aws

Then verify the AWS CLI installation and PATH.

Problem 2 — AccessDenied

Check:

Authentication
      ↓
Correct identity?
      ↓
IAM policy
      ↓
Required action
      ↓
Resource

Do not immediately request administrator permissions.

Problem 3 — Wrong AWS Account

Run:

aws sts get-caller-identity

Confirm that the authenticated identity is the intended training identity.

Problem 4 — Wrong Region

Check:

aws configure get region

Specify the intended region when required:

aws ec2 describe-instances --region ap-south-1
28. AWS Console vs AWS CLI Example
Console
Browser
  ↓
AWS Console
  ↓
S3
  ↓
Buckets
  ↓
View Bucket
CLI
aws s3 ls

Both ultimately interact with AWS services, but the CLI is better suited for repeatable command-line workflows and automation.

29. What I Learned

Through this task, I learned that the AWS CLI provides command-line access to AWS services.

The basic model is:

AWS Console
      ↓
Browser


AWS CLI
      ↓
Terminal

I also learned that AWS CLI commands operate using the permissions of the authenticated identity.

For example:

AWS CLI
   ↓
Authentication
   ↓
IAM Permissions
   ↓
AWS Service

I learned that credentials must never be exposed in GitHub repositories, README files, screenshots, chat messages, or source code.