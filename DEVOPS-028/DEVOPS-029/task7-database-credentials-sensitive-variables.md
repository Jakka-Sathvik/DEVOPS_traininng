TASK 7 — Database Credentials & Sensitive Variables

Training Duration: 60 Minutes
Topic: Terraform Security — Database Credentials and Sensitive Variables
Environment: AWS + Terraform + Amazon RDS PostgreSQL

1. Objective

The objective of this task is to understand how database credentials should be handled securely when creating AWS RDS resources using Terraform.

The trainee must understand why storing a password directly inside Terraform source code is unsafe.

Unsafe example
resource "aws_db_instance" "database" {
  engine   = "postgres"
  username = "dbadmin"
  password = "MyPassword123"
}

This is dangerous because Terraform configuration is normally stored in a Git repository.

If the password is committed to Git, anyone with access to the repository may be able to obtain the database credentials.

2. Secure Credential Flow

A basic secure Terraform credential flow is:

             Secure Secret
                  │
                  ▼
          Terraform Variable
                  │
                  ▼
             RDS Resource
                  │
                  ▼
        PostgreSQL Database

A more realistic production architecture is:

              Secret Manager
                    │
                    │ Secret
                    ▼
             Deployment System
                    │
                    │ Secure Input
                    ▼
                Terraform
                    │
                    ▼
             Amazon RDS

The important principle is:

Database credentials should not be hardcoded into Terraform source files or committed to Git.

3. Why Hardcoding Passwords Is Unsafe

Consider:

password = "MyPassword123"

This creates several security problems.

Problem 1 — Git exposure

Terraform files are often stored in:

GitHub
GitLab
Bitbucket
Azure Repos

If the password is committed:

Developer
   ↓
terraform code
   ↓
Git repository
   ↓
Password exposed

Even deleting the password later may not completely remove it from Git history.

4. Problem 2 — Multiple Developers

Suppose a team has:

Developer A
Developer B
Developer C
DevOps Engineer
Security Team

If the password exists in the Terraform source code, everyone who can read the file may potentially obtain the credential.

Secrets should instead be restricted to the systems and people that actually need them.

5. Problem 3 — Password Reuse

A hardcoded password can accidentally be reused for:

Development
Testing
Production

This creates a larger security risk.

A better approach is to maintain separate credentials for separate environments.

Development → Development DB password
Testing     → Testing DB password
Production  → Production DB password
6. Terraform Sensitive Variables

Terraform supports sensitive variables.

Example:

variable "db_password" {
  type      = string
  sensitive = true
}

The important part is:

sensitive = true

This tells Terraform that the value is sensitive.

7. What Does sensitive = true Actually Do?

Consider:

variable "db_password" {
  type      = string
  sensitive = true
}

Terraform will generally prevent the value from being displayed in normal command output.

For example, Terraform may display:

password = (sensitive value)

instead of:

password = MyPassword123

This reduces accidental exposure in:

terraform plan
terraform apply
Terraform outputs
CLI logs

However:

sensitive = true does NOT automatically encrypt, remove, or securely store the secret everywhere.

This distinction is extremely important.

8. Sensitive Does Not Mean Encrypted Everywhere

A common beginner misunderstanding is:

sensitive = true
        ↓
Secret is completely secure

That is incorrect.

The actual idea is:

sensitive = true
        ↓
Terraform knows the value is sensitive
        ↓
Terraform hides it from normal display

But the secret may still be stored in Terraform state.

9. Terraform State and Secrets

Terraform maintains state describing infrastructure.

Example:

terraform.tfstate

Depending on the resource and provider behavior, sensitive values may be stored in state.

For example:

Terraform Configuration
        │
        ▼
     Terraform
        │
        ├──────────────► AWS RDS
        │
        ▼
 Terraform State

Therefore:

Sensitive variable
       ≠
Secret completely absent from state

This is one of the most important concepts in Terraform security.

10. Why Terraform State Must Be Protected

Terraform state can contain sensitive infrastructure information.

For example:

Database information
Resource IDs
Network information
Passwords or other sensitive values
Connection information
Provider-managed attributes

Therefore, never treat:

terraform.tfstate

as an ordinary file.

11. Never Commit Terraform State to Git

Do NOT commit:

terraform.tfstate
terraform.tfstate.backup

Add them to .gitignore.

Example:

# Terraform
.terraform/
*.tfstate
*.tfstate.*
.terraform.lock.hcl

# Local variable files containing secrets
terraform.tfvars
*.auto.tfvars
Important note

The .terraform.lock.hcl file is normally useful to commit because it locks provider versions.

Therefore, a better .gitignore is:

.terraform/
*.tfstate
*.tfstate.*
terraform.tfvars
*.auto.tfvars

Do not automatically ignore .terraform.lock.hcl unless your organization's workflow specifically requires it.

12. Terraform Variable Definition

Create:

variables.tf

Example:

variable "db_password" {
  description = "Password for the RDS database"
  type        = string
  sensitive   = true
}

Explanation:

variable
   ↓
Defines an input variable

type = string
   ↓
Password is text

sensitive = true
   ↓
Terraform treats the value as sensitive
13. Database Username Variable

The username does not normally need to be marked sensitive in the same way as a password.

Example:

variable "db_username" {
  description = "RDS database administrator username"
  type        = string
  default     = "dbadmin"
}

Password:

variable "db_password" {
  description = "RDS database administrator password"
  type        = string
  sensitive   = true
}
14. Using the Variable in RDS

Instead of:

password = "MyPassword123"

use:

password = var.db_password

Example:

resource "aws_db_instance" "database" {
  identifier = var.db_identifier

  engine         = "postgres"
  engine_version = var.engine_version

  instance_class    = var.instance_class
  allocated_storage = var.allocated_storage

  db_name  = var.db_name
  username = var.db_username
  password = var.db_password

  port = 5432

  db_subnet_group_name   = aws_db_subnet_group.database.name
  vpc_security_group_ids = var.security_group_ids

  publicly_accessible = false
  multi_az            = false

  storage_encrypted = true

  tags = {
    Name        = "${var.environment}-postgres"
    Environment = var.environment
  }
}

The password is no longer written directly into the Terraform resource.

15. Supplying the Password with terraform.tfvars

For a training environment, one simple method is:

terraform.tfvars

Example:

db_username = "dbadmin"
db_password = "YOUR_TEMPORARY_PASSWORD"

The file should NOT be committed to Git.

Add:

terraform.tfvars
16. Why terraform.tfvars Is Better Than Hardcoding

Compare the two approaches.

Unsafe
resource "aws_db_instance" "database" {
  password = "MyPassword123"
}

Password exists directly in:

main.tf
↓
Git
↓
Repository history
Better for a training lab
resource "aws_db_instance" "database" {
  password = var.db_password
}

and:

terraform.tfvars

contains:

db_password = "YOUR_TEMPORARY_PASSWORD"

while:

terraform.tfvars

prevents the file from being committed.

This is better source-code hygiene, but it is still not a complete production secret-management strategy.

17. Environment Variables

Terraform can also receive variable values through environment variables.

For a variable:

variable "db_password" {
  type      = string
  sensitive = true
}

Terraform can read:

TF_VAR_db_password

The naming convention is:

TF_VAR_<variable_name>

Therefore:

TF_VAR_db_password

corresponds to:

var.db_password
18. Windows PowerShell Example

Since this training can be performed on Windows PowerShell:

$env:TF_VAR_db_password="YOUR_TEMPORARY_PASSWORD"

Then run:

terraform plan

Terraform can obtain:

TF_VAR_db_password

and use it as:

var.db_password
Check that the variable is set

Be careful when displaying environment variables containing secrets.

Avoid commands that expose the password unnecessarily.

19. Linux / macOS Example

A shell environment variable can be set using:

export TF_VAR_db_password="YOUR_TEMPORARY_PASSWORD"

Then:

terraform plan

Terraform reads the variable automatically.

20. Important Environment Variable Warning

Environment variables are not magically secure.

For example:

Shell
 ↓
Environment variable
 ↓
Terraform

The password can still potentially be exposed through:

Shell history
CI/CD configuration
Debug logs
Process/environment inspection
Pipeline configuration

Therefore, environment variables are useful for passing secrets, but they should still be handled carefully.

21. Variable Precedence

Terraform can obtain variable values from several sources.

Common sources include:

1. Default value
2. Environment variables
3. terraform.tfvars
4. *.auto.tfvars
5. -var
6. -var-file

The exact precedence should be understood when multiple sources define the same variable.

For security-sensitive values, avoid putting passwords directly into command-line arguments.

For example, avoid:

terraform apply -var="db_password=MyPassword123"

because command-line arguments can be visible in shell history, logs, or process information depending on the environment.

22. Better Training Flow

A simple training flow is:

                    Password
                       │
                       ▼
             terraform.tfvars
                       │
                       ▼
              Terraform Variable
                       │
                 sensitive=true
                       │
                       ▼
               var.db_password
                       │
                       ▼
                 aws_db_instance
                       │
                       ▼
                     RDS

Remember:

terraform.tfvars
       │
       └── MUST NOT be committed to Git
23. Production-Level Secret Management

For production environments, a dedicated secret-management service is preferable.

AWS provides:

AWS Secrets Manager

A typical architecture is:

                AWS Secrets Manager
                       │
                       │ Secret
                       ▼
                Deployment System
                       │
                       ▼
                   Terraform
                       │
                       ▼
                      RDS

Another common approach is to let the application retrieve the database secret at runtime rather than passing the password through Terraform unnecessarily.

24. Why Secrets Manager Is Different

A secret-management system is designed specifically for credentials and other sensitive values.

It can provide capabilities such as:

Secret storage
Access control
Encryption
IAM permissions
Secret rotation
Auditing
Centralized management

This is different from simply putting:

password = "..."

inside a Terraform file.

25. Terraform + Secrets Manager Important Consideration

There is an important design issue.

If Terraform retrieves a secret and passes the secret value into a resource argument, that value may still end up in Terraform state depending on the resource/provider behavior.

Therefore:

Secrets Manager
      ↓
Terraform
      ↓
RDS

does not automatically mean:

Secret never appears in Terraform state

The exact behavior depends on the Terraform resource/provider and configuration.

The trainee must understand this distinction.

26. State Security

For a team environment, Terraform state should generally use a protected remote backend rather than a local state file.

Example architecture:

Developer A ──┐
Developer B ──┤
CI/CD ────────┤
               ▼
        Secure Remote State
               │
               ▼
        Terraform State

For AWS environments, a common backend design uses an S3 bucket with appropriate access controls and state protection features.

The important security principle is:

Protect the state with the same seriousness as other sensitive infrastructure data.

27. Sensitive Outputs

Suppose Terraform has:

output "database_password" {
  value = var.db_password
}

This is not a good output design.

If an output genuinely needs to expose a secret, Terraform supports:

output "database_password" {
  value     = var.db_password
  sensitive = true
}

Terraform will hide the value in normal CLI output.

However:

Marking an output as sensitive does not make the secret disappear from Terraform state.

Therefore, do not create secret outputs unless there is a specific reason.

28. Better RDS Outputs

For our training database, expose useful non-secret information:

output "database_endpoint" {
  description = "RDS database endpoint"
  value       = aws_db_instance.database.address
}

output "database_port" {
  description = "RDS database port"
  value       = aws_db_instance.database.port
}

output "database_identifier" {
  description = "RDS database identifier"
  value       = aws_db_instance.database.identifier
}

Do NOT create:

output "database_password" {
  value = var.db_password
}

unless there is a strong reason.

29. Complete Example
Directory Structure
terraform/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
├── .gitignore
│
└── modules/
    └── database/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
30. Root variables.tf
variable "db_username" {
  description = "RDS database username"
  type        = string
}

variable "db_password" {
  description = "RDS database password"
  type        = string
  sensitive   = true
}
31. Root main.tf
module "database" {
  source = "./modules/database"

  db_identifier = "training-postgres"

  engine         = "postgres"
  engine_version = var.postgres_engine_version

  instance_class    = "db.t3.micro"
  allocated_storage = 20

  db_name  = "trainingdb"
  username = var.db_username
  password = var.db_password

  port = 5432

  subnet_ids = [
    module.network.private_subnet_1_id,
    module.network.private_subnet_2_id
  ]

  security_group_ids = [
    aws_security_group.database.id
  ]

  environment = "training"
}

The important part is:

password = var.db_password

instead of:

password = "MyPassword123"
32. Root terraform.tfvars

For a temporary training environment:

db_username = "dbadmin"

db_password = "USE-A-TEMPORARY-STRONG-PASSWORD"

Do not commit this file.

33. .gitignore

Use:

# Terraform working directory
.terraform/

# Terraform state
*.tfstate
*.tfstate.*

# Local variable files containing secrets
terraform.tfvars
*.auto.tfvars

Keep .terraform.lock.hcl committed when your team uses it to lock provider versions.

34. Terraform Commands

Initialize:

terraform init

Format:

terraform fmt -recursive

Validate:

terraform validate

Create a plan:

terraform plan

Apply:

terraform apply

After the infrastructure is created:

terraform output

Sensitive values should be handled carefully and should not be unnecessarily printed.

35. Verify Sensitive Variable Behavior

Run:

terraform plan

Look at the output.

The password should not normally be printed as plain text in the plan output.

You may see something similar to:

password = (sensitive value)

This demonstrates that Terraform recognizes the value as sensitive.

36. Verify the RDS Configuration

After deployment:

terraform output

Expected useful information may include:

database_endpoint
database_port
database_identifier

The password should not be part of the normal output.

37. Security Test

The trainee should answer:

Question 1

Is this safe?

password = "MyPassword123"

Answer: No.

Question 2

Is this better?

password = var.db_password

Answer: Yes. The credential is separated from the resource configuration.

Question 3

Does this make the secret completely secure?

sensitive = true

Answer: No.

It mainly controls how Terraform treats/displays the value. The secret may still exist in Terraform state.

Question 4

Should terraform.tfstate be committed to Git?

Answer: No.

Question 5

Should terraform.tfvars containing a database password be committed?

Answer: No.

Question 6

What is a production-oriented secret-management option on AWS?

Answer:

AWS Secrets Manager
38. Common Mistakes
Mistake 1 — Hardcoded password
password = "MyPassword123"
Fix
password = var.db_password
Mistake 2 — No sensitive flag
variable "db_password" {
  type = string
}
Better
variable "db_password" {
  type      = string
  sensitive = true
}
Mistake 3 — Committing terraform.tfvars
git add terraform.tfvars
git commit

Avoid this when it contains secrets.

Mistake 4 — Committing state
terraform.tfstate

Never treat it as ordinary source code.

Mistake 5 — Printing the password

Avoid commands or scripts that unnecessarily display:

DB_PASSWORD
TF_VAR_db_password
Mistake 6 — Putting passwords in command-line arguments

Avoid:

terraform apply -var="db_password=MyPassword123"

Use a safer variable-passing mechanism appropriate to your environment.

Mistake 7 — Assuming sensitive = true encrypts everything

Incorrect:

sensitive = true
        ↓
everything is encrypted
        ↓
secret is completely secure

Correct:

sensitive = true
        ↓
Terraform marks value as sensitive
        ↓
normal display is protected
        ↓
state still requires protection
39. Security Comparison
Method	Secret in Source Code	Hidden from Normal Terraform Output	State Risk	Production Recommendation
Hardcoded password	Yes	No	Yes	Avoid
Variable without sensitive	No	No	Yes	Avoid for passwords
Sensitive variable	No	Yes	Yes	Useful, but not sufficient alone
Gitignored terraform.tfvars	No	Yes, if variable is sensitive	Yes	Training/local use
Environment variable + sensitive variable	No	Yes, if variable is sensitive	Yes	Useful in controlled automation
Secrets Manager + appropriate architecture	No	Depends on implementation	Must still be evaluated	Production-oriented
40. Recommended Training Approach

For this lab:

Step 1
Create sensitive variable
        ↓
Step 2
Remove password from Terraform source
        ↓
Step 3
Use var.db_password
        ↓
Step 4
Provide temporary credential securely
        ↓
Step 5
Add secret files to .gitignore
        ↓
Step 6
Run terraform plan
        ↓
Step 7
Verify password is not displayed normally
        ↓
Step 8
Create RDS
        ↓
Step 9
Verify database
        ↓
Step 10
Destroy training resources
41. Hands-On Exercise
Step 1 — Create the variable

Open:

terraform/variables.tf

Add:

variable "db_password" {
  description = "RDS database password"
  type        = string
  sensitive   = true
}
Step 2 — Update the RDS module

Open:

terraform/modules/database/main.tf

Find:

password = "MyPassword123"

Remove it.

Use:

password = var.db_password
Step 3 — Provide the training value

Create:

terraform/terraform.tfvars

Add:

db_password = "USE-A-TEMPORARY-STRONG-PASSWORD"
Step 4 — Protect the file

Open:

.gitignore

Add:

terraform.tfvars
*.auto.tfvars
Step 5 — Format

Run:

terraform fmt -recursive
Step 6 — Validate

Run:

terraform validate

Expected:

Success! The configuration is valid.
Step 7 — Plan

Run:

terraform plan

Check that the password is not displayed as plain text.

Step 8 — Apply

Run:

terraform apply

Review the resources before confirming.

Step 9 — Verify

Check:

RDS
DB subnet group
Database security group
Private subnets
Database endpoint
Step 10 — Cleanup

Because this is a training environment:

terraform destroy

Confirm the resources are removed.

42. Final Secure Architecture

The training architecture is:

                  Developer / CI/CD
                         │
                         │ Secure credential input
                         ▼
                Terraform Variable
                  sensitive = true
                         │
                         ▼
                 Database Module
                         │
                         ▼
                  Amazon RDS
                         │
                         ▼
                   PostgreSQL

The source-code repository contains:

Terraform configuration
        │
        ├── No hardcoded password
        ├── No terraform.tfvars
        └── No terraform.tfstate

Protected separately:

Secret
  │
  └── Secure credential mechanism
43. Important Security Rules

Remember these rules:

1. Never hardcode database passwords in Terraform source code.

2. Mark password variables as sensitive.

3. Never commit terraform.tfvars when it contains secrets.

4. Never commit terraform.tfstate.

5. Protect remote Terraform state.

6. Do not unnecessarily output passwords.

7. Do not unnecessarily print secrets in CI/CD logs.

8. Environment variables are not automatically secure.

9. sensitive = true does not remove secrets from Terraform state.

10. Use a dedicated secret-management solution for production workloads.
44. Interview Questions
Q1. Why should passwords not be hardcoded in Terraform?

Because Terraform source code is commonly stored in version control, which can expose credentials to repository users and Git history.

Q2. What does sensitive = true do?

It tells Terraform to treat the variable as sensitive and prevents normal CLI output from displaying the value in plain text.

Q3. Does sensitive = true encrypt the password everywhere?

No. Sensitive values may still be stored in Terraform state depending on the resource/provider behavior.

Q4. Why is Terraform state sensitive?

Terraform state can contain infrastructure attributes and potentially sensitive values.

Q5. Should Terraform state be committed to Git?

No.

Q6. What does TF_VAR_db_password mean?

It is an environment variable Terraform recognizes as the value for:

var.db_password
Q7. What is AWS Secrets Manager?

It is an AWS service designed to securely store and manage secrets such as database credentials, API keys, and other sensitive information.

Q8. Is an environment variable automatically secure?

No. Environment variables can still be exposed through shell history, CI/CD configuration, logs, or process/environment inspection depending on the environment.

Q9. What is the difference between a secret and a sensitive Terraform variable?

A secret is the actual confidential value. sensitive = true is Terraform metadata that controls how Terraform handles/displays that value. It is not itself a secret-management system.

Q10. What is the most important lesson from this task?
Hiding a secret is not the same as securely managing a secret.

Terraform configuration, variables, state, CI/CD systems, and secret-management services must all be considered when designing secure credential handling.

45. Task 7 Deliverable Checklist

The trainee has completed the task when:

Understands why hardcoded passwords are unsafe

Created db_password Terraform variable

Used sensitive = true

Removed password from Terraform resource source code

Understands var.db_password

Understands terraform.tfvars

Added secret variable files to .gitignore

Understands TF_VAR_db_password

Understands environment-variable limitations

Understands Terraform state can contain sensitive values

Does not commit terraform.tfstate

Does not commit credentials to Git

Understands AWS Secrets Manager

Understands that sensitive variables are not a complete secret-management solution

Successfully ran terraform validate

Successfully ran terraform plan

Tested the RDS configuration

Destroyed temporary training resources

46. Summary

The insecure approach is:

password = "MyPassword123"

The better Terraform approach is:

Secure Credential
       ↓
Terraform Variable
       ↓
sensitive = true
       ↓
var.db_password
       ↓
RDS

But remember:

sensitive = true
       ≠
complete secret security

Sensitive values may still exist in Terraform state.

Therefore, secure infrastructure requires multiple layers:

             Secret Management
                    │
                    ▼
              Secure Input
                    │
                    ▼
               Terraform
                    │
          ┌─────────┴─────────┐
          │                   │
     Configuration          State
          │                   │
          ▼                   ▼
       No secrets       Protected storage
          │
          ▼
         RDS

The key DevOps principle is:

Never put credentials in source code, and never assume that marking a Terraform variable as sensitive makes the secret secure everywhere.

For production systems, combine Terraform's sensitive-variable handling with strong state protection, least-privilege access, secure CI/CD secret handling, and a dedicated secret-management service such as AWS Secrets Manager.