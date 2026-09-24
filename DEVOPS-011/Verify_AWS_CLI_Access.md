TASK 10 — Verify AWS CLI Access

Time: 30 Minutes
Topic: AWS CLI Access Verification

1. Objective

The objective of this task is to verify that the AWS CLI is correctly authenticated and determine:

Which AWS identity is currently being used.
Which AWS account the identity belongs to.
Which AWS region is configured.
Whether the CLI is ready to communicate with AWS services.

The main commands used are:

aws sts get-caller-identity

and:

aws configure get region
2. Verify AWS CLI Installation

First, check that AWS CLI is installed.

Run:

aws --version

Example output:

aws-cli/2.x.x Python/3.x.x Windows/10 exe/AMD64

The exact version and operating-system information may be different.

If the command is not recognized, AWS CLI may not be installed or may not be available in the system PATH.

3. Verify AWS Identity

Run:

aws sts get-caller-identity

This command uses AWS Security Token Service (STS) to identify the credentials currently being used by the AWS CLI.

Example:

{
    "UserId": "AIDAEXAMPLE123456789",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/sathvik-training"
}

Your actual output will be different.

4. Understand the Output

The command normally returns three important fields.

UserId

Example:

AIDAEXAMPLE123456789

This identifies the AWS principal associated with the credentials.

Depending on the authentication method, this value can represent a user or another type of AWS principal/session.

Account

Example:

123456789012

This is the 12-digit AWS account ID associated with the identity.

It helps verify that the CLI is connected to the expected AWS account.

Arn

Example:

arn:aws:iam::123456789012:user/sathvik-training

The ARN identifies the AWS principal being used.

For example, an IAM user may appear as:

arn:aws:iam::123456789012:user/sathvik-training

A role session may appear differently, for example:

arn:aws:sts::123456789012:assumed-role/DevOpsTrainingRole/session-name

Therefore, the ARN also helps determine whether authentication is occurring through an IAM user, role, or another supported identity mechanism.

5. Why get-caller-identity Is Important

Before performing AWS CLI operations, it is good practice to verify the current identity.

Architecture:

AWS CLI
   |
   | Credentials / Authentication
   ↓
AWS STS
   |
   ↓
get-caller-identity
   |
   ├── UserId
   ├── Account
   └── Arn

This can prevent mistakes such as accidentally working in the wrong AWS account.

For example:

Developer
    ↓
AWS CLI
    ↓
Wrong AWS Profile
    ↓
Wrong AWS Account
    ↓
Potentially modify wrong resources

Running:

aws sts get-caller-identity

before important operations helps detect this problem.

6. Identify the Configured AWS Region

Run:

aws configure get region

Example:

ap-south-1

This means the configured default region is:

ap-south-1

which is the AWS Asia Pacific (Mumbai) Region.

Your configured region may be different.

7. Understand AWS Region

A region determines the default geographic AWS location used by many AWS CLI commands.

Examples:

ap-south-1       → Mumbai
us-east-1        → N. Virginia
us-west-2        → Oregon
eu-west-1        → Ireland
ap-southeast-1   → Singapore

For example:

aws ec2 describe-instances

without specifying another region generally operates against the CLI's configured/default region.

8. Check Region Using the Current Configuration

Run:

aws configure get region

If the command returns:

ap-south-1

you can understand the configuration as:

AWS CLI
   ↓
Default Region
   ↓
ap-south-1
   ↓
Mumbai Region

If nothing is returned, a default region may not be configured in that CLI configuration.

A region can also be explicitly supplied to a command:

aws ec2 describe-instances --region ap-south-1

This allows the command to use ap-south-1 for that operation.

9. Complete Verification Procedure

Follow these steps in order.

Step 1 — Check AWS CLI
aws --version

Expected:

AWS CLI version information
Step 2 — Check AWS Identity
aws sts get-caller-identity

Expected:

{
    "UserId": "...",
    "Account": "...",
    "Arn": "..."
}
Step 3 — Check Configured Region
aws configure get region

Expected:

ap-south-1

or another configured AWS region.

Step 4 — Record the Result

Create a small verification record:

AWS CLI Version : <your output>
AWS Account     : <12-digit account ID>
AWS Identity    : <ARN>
AWS Region      : <configured region>
Status          : Verified

Do not publish sensitive authentication information.

10. Example Verification

Example only:

$ aws --version

aws-cli/2.x.x Python/3.x.x Windows/10 exe/AMD64

Then:

$ aws sts get-caller-identity

Example:

{
    "UserId": "AIDAEXAMPLE123456789",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/sathvik-training"
}

Then:

$ aws configure get region

ap-south-1

Verification result:

AWS CLI Installation : Verified
AWS Identity         : Verified
AWS Account          : Verified
AWS Region           : ap-south-1
CLI Access           : Working
11. Important Security Notes

Never put the following information into GitHub, README files, screenshots, or chat:

AWS Access Key ID
AWS Secret Access Key
AWS Session Token
Passwords
Private keys
Temporary credentials

The output of:

aws sts get-caller-identity

does not reveal your secret access key, but account/identity information should still not be unnecessarily published publicly.

If credentials are accidentally exposed, follow your organization's credential-revocation and rotation procedure immediately.

12. Troubleshooting
Problem 1 — AWS Command Not Found

If you see:

aws: command not found

or on Windows:

'aws' is not recognized as an internal or external command

Check whether AWS CLI is installed and available in PATH.

Windows:

where.exe aws

Linux/macOS:

which aws
Problem 2 — Unable to Locate Credentials

If you see an error indicating that credentials cannot be found, the CLI has not been authenticated through an approved method.

Do not copy credentials from another person.

Use the authentication method approved by your organization, such as IAM Identity Center/SSO, an approved role, or another authorized method.

Problem 3 — Access Denied

If you receive:

AccessDenied

the identity may be authenticated successfully but does not have permission for the requested AWS operation.

Remember:

Authentication
      ↓
Who are you?
      ↓
Authorization
      ↓
What are you allowed to do?

get-caller-identity can help verify the first part.

Problem 4 — Wrong AWS Account

Run:

aws sts get-caller-identity

Check the:

Account
Arn

If they are unexpected, stop before making changes to AWS resources and check which approved profile or authentication method is being used.

13. Useful Commands
Check AWS CLI version
aws --version
Check current identity
aws sts get-caller-identity
Check configured region
aws configure get region
Display AWS CLI help
aws help
Check EC2 help
aws ec2 help
Check S3 help
aws s3 help
14. Architecture
                AWS CLI
                   |
                   ↓
          Authentication Method
                   |
                   ↓
             AWS STS
                   |
          ┌────────┴────────┐
          ↓                 ↓
    Caller Identity      AWS Account
          |
          ↓
         ARN


AWS CLI Configuration
          |
          ↓
    Default Region
          |
          ↓
      AWS Region
15. What I Learned

After completing this task, I understand:

How to verify AWS CLI installation.
How aws sts get-caller-identity verifies the current AWS identity.
What UserId, Account, and Arn represent.
How to identify the configured AWS region.
How AWS CLI authentication differs from authorization.
Why identity verification is important before performing AWS operations.
Why AWS credentials must never be exposed in source code, GitHub, screenshots, or chat.