TASK 13 — AWS Security Challenge

Time: 30 Minutes
Deliverable: AWS_Security_Challenge.md

1. Objective

The objective of this challenge is to understand common AWS security mistakes involving credentials, IAM roles, and excessive permissions.

The key security principles are:

Protect Credentials
       ↓
Use Temporary Credentials Where Possible
       ↓
Follow Least Privilege
       ↓
Rotate/Revoke Exposed Credentials
       ↓
Use IAM Roles for AWS Workloads
2. Scenario 1 — Access Key Stored in config.py
Situation

A developer puts an AWS access key inside:

config.py
What is wrong?

This is a security risk because credentials are being hard-coded inside application source code.

For example, storing credentials directly in source code can result in:

Source Code
     ↓
Git Repository
     ↓
GitHub
     ↓
Credentials Exposed
     ↓
Unauthorized AWS Access

The problem becomes especially serious if the repository is shared or public.

Access keys can potentially be used by an unauthorized person to perform AWS operations according to the permissions assigned to those credentials.

Why is this bad?

Hard-coded credentials can:

Be accidentally committed to Git.
Be copied to other systems.
Appear in source-code backups.
Be exposed through logs or screenshots.
Be difficult to rotate safely.
Remain in Git history even after the code is deleted.
Better approach

Use an approved authentication and credential-management mechanism instead of hard-coding long-lived credentials in application code.

Depending on the environment, this could involve:

Environment Variables
       ↓
Approved Secret Management
       ↓
Temporary Credentials
       ↓
IAM Roles

For AWS workloads, IAM roles are generally preferred where applicable.

Answer

Putting AWS access keys directly into config.py is unsafe because credentials can be exposed through source code, version control, logs, or other copies of the application. Credentials should not be hard-coded. An approved credential-management mechanism or IAM role should be used instead.

3. Scenario 2 — Access Key Pushed to GitHub
Situation

A developer accidentally pushes an AWS access key to GitHub.

What should happen immediately?

The exposed credential should be treated as compromised.

The first priority is to revoke or deactivate the exposed credential and rotate it according to the organization's AWS security process.

Do not assume that deleting the file from the latest Git commit makes the credential safe.

Git repositories can retain information in:

Commit History
Branches
Forks
Clones
Caches
Immediate Response
Credential Exposed
       ↓
Stop Using Credential
       ↓
Revoke / Deactivate Credential
       ↓
Rotate / Replace Credential
       ↓
Investigate Usage
       ↓
Review AWS Logs
       ↓
Remove Secret From Repository/History
       ↓
Update Application Authentication
       ↓
Verify Secure Access
Important

Do not simply do this:

Delete config.py
       ↓
Push new commit

That does not necessarily remove the secret from Git history or prevent someone who already obtained it from using it.

Security Investigation

The organization should determine whether the credential was used by an unauthorized party.

Depending on the environment, security personnel may review relevant AWS logging and monitoring information to identify suspicious activity.

Answer

If an AWS access key is pushed to GitHub, it must be treated as compromised immediately. The credential should be revoked/deactivated and replaced according to the organization's security procedure. AWS activity should be reviewed for possible misuse, and the secret should be removed from the repository and its history where appropriate. Simply deleting the file is not sufficient.

4. Scenario 3 — EC2 Application Needs S3 Access
Situation

An application running on EC2 needs permission to access S3.

Should the developer put an access key inside the EC2 server?

No.

The developer should generally not store long-lived AWS access keys inside the EC2 server.

This creates unnecessary credential-management and security risks.

Bad architecture:

EC2
 │
 ├── Application
 │
 └── Hard-coded AWS Access Key
             │
             ▼
          S3

Problems include:

Credential exposure.
Difficult rotation.
Credential theft if the server is compromised.
Accidental disclosure through configuration files.
Long-lived credentials remaining on the server.
5. Better Approach — IAM Role

The preferred approach for an EC2 workload is generally to attach an appropriate IAM role to the EC2 instance.

Architecture:

EC2 Instance
      ↓
IAM Role
      ↓
Temporary Credentials
      ↓
AWS SDK / AWS CLI
      ↓
S3

The role should have only the permissions required by the application.

For example, if the application only needs to read objects from a specific S3 location, the role should provide the minimum required S3 read permissions rather than unrestricted access to all AWS services.

6. Why IAM Roles Are Better

IAM roles provide an important security advantage because applications can obtain temporary credentials rather than requiring a developer to place long-lived access keys on the server.

The architecture becomes:

Application
     ↓
EC2
     ↓
IAM Role
     ↓
Temporary Credentials
     ↓
S3

The developer does not need to manually embed an AWS access key in the application.

Answer

No. A developer should generally not store long-lived AWS access keys inside an EC2 server. The better approach is to attach an IAM role to the EC2 instance and grant the role only the S3 permissions required by the application. AWS can then provide temporary credentials to the workload.

7. Scenario 4 — Request for Administrator Permissions
Situation

A trainee requests full Administrator permissions for every AWS service.

Is this good security practice?

No.

This violates the principle of least privilege.

Least privilege means:

An identity should receive only the permissions necessary to perform its required tasks.

8. Why AdministratorAccess Is Risky

Suppose a trainee only needs to work with:

S3
EC2
IAM

Giving them unrestricted administrator permissions across AWS could allow operations on many unrelated services and resources.

For example:

Required:
S3 → Read training data

Requested:
AdministratorAccess → All AWS services

The requested access is much broader than the actual requirement.

9. Risks of Excessive Permissions

Excessive permissions can increase the impact of:

Accidental configuration changes.
Incorrect commands.
Compromised credentials.
Malicious activity.
Unauthorized resource deletion.
Unexpected resource creation.
Security configuration changes.
Excessive AWS costs.

For example:

Compromised Low-Privilege Identity
        ↓
Limited AWS Access
        ↓
Limited Potential Damage

Compared with:

Compromised Administrator Identity
        ↓
Broad AWS Access
        ↓
Much Larger Potential Impact
10. Better Approach

First determine what the trainee actually needs.

Training Task
     ↓
Required AWS Service
     ↓
Required Action
     ↓
Required Resource
     ↓
Minimum Permission

For example:

Training Requirement:
Read S3 objects

Required Permission:
S3 read operation

Resource:
Specific training bucket/object scope where supported

There is no need to automatically grant unrestricted administrator access.

11. Least-Privilege Example
Poor approach
Trainee
   ↓
AdministratorAccess
   ↓
All AWS Services
Better approach
Trainee
   ↓
Training Group / Role
   ↓
Specific IAM Policy
   ↓
Required AWS Actions
   ↓
Required Resources

This reduces unnecessary permissions.

12. Four Scenarios — Quick Answers
Scenario	Correct Security Response
Access key in config.py	Never hard-code credentials; use an approved credential-management mechanism or role
Key pushed to GitHub	Treat it as compromised; immediately revoke/deactivate and rotate/replace it; investigate usage
EC2 needs S3	Use an IAM role with least-privilege S3 permissions instead of storing long-lived keys
Trainee wants AdministratorAccess	Do not grant broad access without a justified requirement; follow least privilege
13. Security Architecture

A secure AWS DevOps approach looks like:

                    AWS
                     │
                     ▼
                    IAM
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      IAM User              IAM Role
          │                     │
          │                AWS Workload
          │                     │
          ▼                     ▼
    Least Privilege       Temporary Credentials
          │                     │
          └──────────┬──────────┘
                     ▼
                AWS Services

The central security principle is:

Identity
   ↓
Minimum Required Permissions
   ↓
Authorized Resource
14. What I Learned

After completing this challenge, I understand:

Why AWS credentials should never be hard-coded into source code.
Why committing credentials to GitHub is a serious security incident.
Why an exposed credential should be revoked/deactivated and replaced immediately.
Why deleting a secret from the latest Git commit is not enough.
Why EC2 applications should generally use IAM roles instead of long-lived access keys.
How IAM roles provide temporary credentials to AWS workloads.
Why least privilege is important.
Why unrestricted AdministratorAccess should not be granted simply for convenience.
How excessive permissions increase the potential impact of mistakes or compromised credentials.