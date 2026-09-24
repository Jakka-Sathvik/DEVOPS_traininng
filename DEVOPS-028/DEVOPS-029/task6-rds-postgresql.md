TASK 6 — Create RDS PostgreSQL Instance
Objective

Create a small training Amazon RDS PostgreSQL database using Terraform.

The RDS instance will use:

PostgreSQL
Small training instance class
Small amount of storage
Private subnets
DB Subnet Group
Database Security Group
PostgreSQL port 5432
Encryption
No public accessibility
1. Final Architecture
                         Internet
                            │
                            ▼
                           ALB
                            │
                            ▼
                    Application EC2
                    Application SG
                            │
                            │ TCP 5432
                            ▼
                    Database Security Group
                            │
                            ▼
                    DB Subnet Group
                     │              │
                     ▼              ▼
                Private AZ-1    Private AZ-2
                     │              │
                     └──────┬───────┘
                            │
                            ▼
                           RDS
                       PostgreSQL

The important traffic flow is:

Internet
   │
   ▼
ALB
   │
   ▼
EC2
   │
   │ PostgreSQL : 5432
   ▼
RDS

The internet does not directly connect to RDS.

2. What Are We Creating?

Terraform will create:

RDS PostgreSQL Instance
        │
        ├── PostgreSQL Engine
        ├── Training Instance
        ├── Storage
        ├── Database Name
        ├── Port 5432
        ├── DB Subnet Group
        └── Database Security Group
3. Training Configuration

A suitable small training configuration can be:

Engine:              PostgreSQL
Instance Class:      db.t3.micro
Storage:             20 GB
Database Name:       trainingdb
Port:                5432
Public Access:       No
Multi-AZ:            No
Encryption:          Yes

For a short-lived lab, Multi-AZ is intentionally disabled to avoid unnecessary cost.

4. Important Cost Awareness

RDS is a billable AWS service.

Before creating the instance:

Check AWS pricing
       ↓
Create training database
       ↓
Perform the exercise
       ↓
Destroy the database

For a short training lab:

multi_az = false

can avoid the additional standby infrastructure associated with Multi-AZ deployment.

Also avoid enabling optional features that are not required for this exercise.

Important: even a small RDS instance can incur charges, and storage, backups, data transfer, and other resources may also contribute to cost.

5. Project Structure

Your project can look like:

terraform/
│
├── main.tf
├── variables.tf
├── outputs.tf
│
└── modules/
    │
    ├── network/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    ├── compute/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    └── database/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf

The RDS instance belongs in the database module.

6. Database Module Variables

Open:

modules/database/variables.tf

Use:

variable "db_identifier" {
  description = "RDS database identifier"
  type        = string
}

variable "engine" {
  description = "Database engine"
  type        = string
  default     = "postgres"
}

variable "engine_version" {
  description = "Supported PostgreSQL engine version"
  type        = string
}

variable "instance_class" {
  description = "RDS instance class"
  type        = string
  default     = "db.t3.micro"
}

variable "allocated_storage" {
  description = "Allocated storage in GB"
  type        = number
  default     = 20
}

variable "db_name" {
  description = "Initial database name"
  type        = string
  default     = "trainingdb"
}

variable "username" {
  description = "Master username"
  type        = string
}

variable "password" {
  description = "Master password"
  type        = string
  sensitive   = true
}

variable "port" {
  description = "PostgreSQL port"
  type        = number
  default     = 5432
}

variable "subnet_ids" {
  description = "Private subnet IDs"
  type        = list(string)
}

variable "security_group_ids" {
  description = "Database security group IDs"
  type        = list(string)
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "training"
}
7. Create DB Subnet Group

The DB Subnet Group from Task 4 is required.

In:

modules/database/main.tf

add:

resource "aws_db_subnet_group" "database" {

  name       = "${var.environment}-db-subnet-group"
  subnet_ids = var.subnet_ids

  tags = {
    Name        = "${var.environment}-db-subnet-group"
    Environment = var.environment
  }
}

This connects RDS to the private subnets.

8. Create PostgreSQL RDS Instance

Add:

resource "aws_db_instance" "database" {

  identifier = var.db_identifier

  engine         = var.engine
  engine_version = var.engine_version

  instance_class = var.instance_class

  allocated_storage = var.allocated_storage

  db_name  = var.db_name
  username = var.username
  password = var.password

  port = var.port

  db_subnet_group_name = aws_db_subnet_group.database.name

  vpc_security_group_ids = var.security_group_ids

  publicly_accessible = false

  multi_az = false

  storage_encrypted = true

  skip_final_snapshot = true

  deletion_protection = false

  tags = {
    Name        = "${var.environment}-postgres"
    Environment = var.environment
  }
}
9. Understand Each Configuration
Identifier
identifier = var.db_identifier

This gives the RDS instance a unique identifier.

Example:

training-postgres
10. Engine
engine = var.engine

For this exercise:

postgres

This tells AWS to create a PostgreSQL database.

11. Engine Version
engine_version = var.engine_version

Do not blindly copy a version from an old tutorial.

First check which PostgreSQL versions are currently offered in your AWS region.

You can use the AWS Console or AWS CLI to check available RDS engine versions.

Example CLI:

aws rds describe-db-engine-versions \
  --engine postgres \
  --query "DBEngineVersions[].EngineVersion" \
  --output text

Then select an appropriate supported version for your training environment.

12. Instance Class
instance_class = "db.t3.micro"

This is intended as a small training configuration.

The available instance classes and pricing can vary by region and over time, so verify availability before deployment.

13. Storage
allocated_storage = 20

This requests:

20 GB

for the database.

For a basic training lab, there is usually no reason to provision a large amount of storage.

14. Database Name
db_name = "trainingdb"

The initial database is:

trainingdb

Your application can later connect to this database.

15. Username

Example:

username = var.username

For training:

dbadmin

Avoid using credentials directly in source-controlled Terraform files.

16. Password

The password is:

password = var.password

The variable is marked:

sensitive = true

However, sensitive does not mean secret is completely absent from Terraform state. RDS credentials can be stored in Terraform state depending on the configuration.

For production, use a proper secret-management approach such as AWS Secrets Manager and secure your Terraform state appropriately.

For this training lab, use a temporary credential and destroy the infrastructure afterward.

17. PostgreSQL Port
port = 5432

PostgreSQL normally uses:

TCP 5432

The database Security Group should allow this port only from the application Security Group.

18. DB Subnet Group
db_subnet_group_name = aws_db_subnet_group.database.name

This connects the RDS instance to the DB Subnet Group.

Flow:

Private Subnet
      │
      ▼
DB Subnet Group
      │
      ▼
RDS
19. Security Group
vpc_security_group_ids = var.security_group_ids

The database Security Group controls which resources can connect to PostgreSQL.

Expected rule:

Application SG
      │
      │ TCP 5432
      ▼
Database SG
      │
      ▼
RDS
20. Disable Public Accessibility

Use:

publicly_accessible = false

This is an important setting for this architecture.

The database should be accessed through the application tier rather than directly from the internet.

21. Disable Multi-AZ for Training

Use:

multi_az = false

This is appropriate for a short-lived basic training exercise where high availability is not being tested.

For production, the availability requirements should be evaluated separately.

22. Enable Storage Encryption

Use:

storage_encrypted = true

This enables encryption for RDS storage.

For training, it also gives you experience with encrypted database storage.

23. Training Cleanup Configuration

Use:

skip_final_snapshot = true

and:

deletion_protection = false

This makes the temporary training database easier to destroy.

Do not blindly carry these settings into production.

For production databases, deletion protection and final snapshots should be considered according to the organization's backup and recovery requirements.

24. Complete Database Module

Your modules/database/main.tf can now look like:

resource "aws_db_subnet_group" "database" {

  name       = "${var.environment}-db-subnet-group"
  subnet_ids = var.subnet_ids

  tags = {
    Name        = "${var.environment}-db-subnet-group"
    Environment = var.environment
  }
}


resource "aws_db_instance" "database" {

  identifier = var.db_identifier

  engine         = var.engine
  engine_version = var.engine_version

  instance_class = var.instance_class

  allocated_storage = var.allocated_storage

  db_name  = var.db_name
  username = var.username
  password = var.password

  port = var.port

  db_subnet_group_name = aws_db_subnet_group.database.name

  vpc_security_group_ids = var.security_group_ids

  publicly_accessible = false

  multi_az = false

  storage_encrypted = true

  skip_final_snapshot = true

  deletion_protection = false

  tags = {
    Name        = "${var.environment}-postgres"
    Environment = var.environment
  }
}
25. Create Module Outputs

Open:

modules/database/outputs.tf

Add:

output "database_endpoint" {
  description = "RDS PostgreSQL endpoint"
  value       = aws_db_instance.database.endpoint
}

output "database_port" {
  description = "RDS PostgreSQL port"
  value       = aws_db_instance.database.port
}

output "database_identifier" {
  description = "RDS database identifier"
  value       = aws_db_instance.database.identifier
}

output "database_name" {
  description = "Database name"
  value       = aws_db_instance.database.db_name
}
26. Call the Database Module

In the root main.tf:

module "database" {

  source = "./modules/database"

  db_identifier = "training-postgres"

  engine = "postgres"

  engine_version = var.postgres_engine_version

  instance_class = "db.t3.micro"

  allocated_storage = 20

  db_name = "trainingdb"

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

Adjust the subnet and Security Group output/reference names to match your existing project.

27. Root Variables

In:

variables.tf

add:

variable "postgres_engine_version" {
  description = "Supported PostgreSQL engine version"
  type        = string
}

variable "db_username" {
  description = "RDS master username"
  type        = string
}

variable "db_password" {
  description = "RDS master password"
  type        = string
  sensitive   = true
}
28. Training Values

Create a local terraform.tfvars file:

postgres_engine_version = "YOUR_SUPPORTED_VERSION"

db_username = "dbadmin"

db_password = "USE_A_TEMPORARY_PASSWORD"

Replace:

YOUR_SUPPORTED_VERSION

with a PostgreSQL version currently supported in your AWS region.

Do not commit this file to Git.

Add it to .gitignore:

terraform.tfvars
*.tfstate
*.tfstate.*
.terraform/
29. Root Outputs

In:

outputs.tf

add:

output "database_endpoint" {
  description = "RDS PostgreSQL endpoint"
  value       = module.database.database_endpoint
}

output "database_port" {
  description = "RDS PostgreSQL port"
  value       = module.database.database_port
}

output "database_identifier" {
  description = "RDS database identifier"
  value       = module.database.database_identifier
}

output "database_name" {
  description = "Database name"
  value       = module.database.database_name
}
30. Initialize Terraform

Run:

terraform init

Terraform initializes the AWS provider and module dependencies.

31. Format

Run:

terraform fmt -recursive

This formats the entire Terraform project.

32. Validate

Run:

terraform validate

Expected:

Success! The configuration is valid.
33. Check the Plan

Run:

terraform plan

Review carefully before applying.

You should see resources such as:

aws_db_subnet_group.database
aws_db_instance.database

along with the network and Security Group resources if they are part of the same configuration.

34. Apply

Run:

terraform apply

Terraform will display the resources it plans to create.

Type:

yes

RDS creation can take several minutes.

Do not assume the database is ready immediately after Terraform starts creating it.

35. Get the RDS Endpoint

After successful deployment:

terraform output database_endpoint

Example format:

training-postgres.xxxxxxxxx.ap-south-1.rds.amazonaws.com:5432

The exact endpoint will be generated by AWS.

36. Get Database Port

Run:

terraform output database_port

Expected:

5432
37. Get Database Identifier

Run:

terraform output database_identifier

Expected:

training-postgres
38. Verify RDS in AWS Console

Navigate to:

AWS Console
   ↓
RDS
   ↓
Databases
   ↓
training-postgres

Check:

Engine:
PostgreSQL

Status:
Available

Public access:
No

Port:
5432

DB subnet group:
training-db-subnet-group
39. Verify Networking

Check:

RDS
 ↓
Connectivity & security

Verify that:

VPC
Private subnets
Database Security Group
Port 5432

are configured correctly.

40. Verify Security Group

The database Security Group should contain a rule similar to:

Inbound

Protocol: TCP
Port: 5432
Source: Application Security Group

It should NOT contain:

TCP 5432
0.0.0.0/0
41. Application Connection

Once the application EC2 instance is correctly configured, the application can connect using:

Host:
RDS Endpoint

Port:
5432

Database:
trainingdb

Username:
dbadmin

Password:
Temporary training password

Conceptually:

EC2
 │
 │ TCP 5432
 ▼
RDS
 │
 ▼
trainingdb
42. Important: RDS Endpoint Is Not an IP Address to Hard-Code

Do not write an RDS IP address into your application configuration.

Use the RDS DNS endpoint.

Example:

training-postgres.xxxxx.ap-south-1.rds.amazonaws.com

AWS manages the underlying infrastructure.

43. Test Connectivity

If PostgreSQL client tools are installed on the application server, you can test:

psql \
  -h <RDS-ENDPOINT> \
  -p 5432 \
  -U dbadmin \
  -d trainingdb

You will be prompted for the password.

The connection should succeed only when:

EC2
 │
 ├── Correct VPC
 ├── Correct subnet/routing
 ├── Application SG
 │
 ▼
RDS
 │
 └── Database SG allows TCP 5432 from Application SG
44. Troubleshooting
Problem 1 – RDS Cannot Be Created

Check:

terraform plan

and:

terraform apply

Read the AWS error carefully.

Common causes include:

Unsupported engine version
Invalid instance class
Invalid subnet configuration
Invalid parameter
Invalid Security Group
Incorrect VPC configuration
45. Problem 2 – Unsupported PostgreSQL Version

If AWS reports that the selected PostgreSQL version is unavailable:

Check available versions:

aws rds describe-db-engine-versions \
  --engine postgres \
  --query "DBEngineVersions[].EngineVersion" \
  --output text

Select a currently supported version and update:

postgres_engine_version = "SUPPORTED_VERSION"

Then:

terraform plan
terraform apply
46. Problem 3 – Application Cannot Connect

Check in this order:

1. RDS status
       ↓
2. RDS endpoint
       ↓
3. Port 5432
       ↓
4. EC2 VPC
       ↓
5. EC2 subnet
       ↓
6. Database subnet group
       ↓
7. Application Security Group
       ↓
8. Database Security Group
       ↓
9. PostgreSQL credentials
47. Problem 4 – Connection Timeout

A timeout commonly indicates a networking or Security Group problem.

Check:

Application SG
       │
       │ TCP 5432
       ▼
Database SG

Make sure the database SG references the application SG.

Do not solve the problem by opening:

0.0.0.0/0

on port 5432.

48. Problem 5 – RDS Is Public

Check:

publicly_accessible = false

Also verify that RDS is using the intended private DB subnets and that your overall VPC routing does not unintentionally expose the database.

49. Terraform State

Run:

terraform state list

You should see resources similar to:

module.database.aws_db_subnet_group.database
module.database.aws_db_instance.database

Remember that Terraform state can contain sensitive infrastructure information and, depending on configuration, credentials.

Protect your state file.

Never commit:

terraform.tfstate

to Git.

50. Cleanup

This is a training database.

When the exercise is complete:

terraform destroy

Review the resources that will be removed.

Then confirm:

yes

After destruction, verify in:

AWS Console
   ↓
RDS
   ↓
Databases

and confirm that the training database has been removed.

Also check for other lab resources that may still incur charges.

51. Final Architecture
                         Internet
                            │
                            ▼
                     Application ALB
                            │
                            ▼
                    Application EC2
                    Application SG
                            │
                            │
                       TCP 5432
                            │
                            ▼
                    Database Security SG
                            │
                            ▼
                    DB Subnet Group
                       │         │
                       ▼         ▼
                  Private AZ-1  Private AZ-2
                       │         │
                       └────┬────┘
                            │
                            ▼
                    RDS PostgreSQL
                       trainingdb
52. Complete Learning Flow

The tasks completed so far connect together:

TASK 1
Three-Tier Architecture
        │
        ▼
TASK 2
RDS Fundamentals
        │
        ▼
TASK 3
Database Terraform Module
        │
        ▼
TASK 4
DB Subnet Group
        │
        ▼
TASK 5
RDS Security Group
        │
        ▼
TASK 6
RDS PostgreSQL Instance

Final infrastructure relationship:

VPC
 │
 ├── Public Subnets
 │       └── ALB
 │
 ├── Private Application Subnets
 │       └── EC2 / ASG
 │              │
 │              │ TCP 5432
 │              ▼
 │       Database Security Group
 │              │
 │              ▼
 │       DB Subnet Group
 │              │
 │              ├── Private DB Subnet AZ-1
 │              └── Private DB Subnet AZ-2
 │                       │
 │                       ▼
 │                 RDS PostgreSQL
 │
 └── Internet Gateway / NAT as required
53. Deliverable Checklist

The Terraform-managed RDS instance should satisfy:

☑ PostgreSQL engine
☑ Supported PostgreSQL version
☑ Small training instance class
☑ 20 GB training storage
☑ Database name: trainingdb
☑ PostgreSQL port: 5432
☑ DB Subnet Group configured
☑ Private subnets configured
☑ Database Security Group configured
☑ Application SG allowed on TCP 5432
☑ Public accessibility disabled
☑ Encryption enabled
☑ Multi-AZ disabled for this short training lab
☑ Terraform module used
☑ Terraform outputs created
☑ terraform fmt successful
☑ terraform validate successful
☑ terraform plan reviewed
☑ terraform apply successful
☑ RDS endpoint verified
☑ Training database destroyed after exercise
Key Takeaway

The RDS instance is the database tier of the three-tier architecture.

The secure communication path is:

Internet
   ↓
ALB
   ↓
Application EC2
   ↓
Application Security Group
   ↓
TCP 5432
   ↓
Database Security Group
   ↓
Private DB Subnets
   ↓
RDS PostgreSQL

The database is therefore not directly exposed to the internet. Terraform manages the database infrastructure consistently and allows the same module to be reused for different environments.