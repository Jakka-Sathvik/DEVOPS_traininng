TASK 4 — Create Database Subnet Group
Objective

Create an Amazon RDS DB Subnet Group using Terraform.

The DB Subnet Group will use private subnets in multiple Availability Zones so that RDS has the required network placement for a highly available database architecture.

1. Architecture
                         VPC
                          │
             ┌────────────┴────────────┐
             │                         │
             ▼                         ▼
       Private Subnet AZ-1       Private Subnet AZ-2
       10.0.11.0/24              10.0.12.0/24
             │                         │
             └────────────┬────────────┘
                          │
                          ▼
                  DB Subnet Group
                          │
                          ▼
                     Amazon RDS

The relationship is:

VPC
 ↓
Private Subnets
 ↓
DB Subnet Group
 ↓
RDS
2. What is an RDS DB Subnet Group?

An RDS DB Subnet Group is a collection of subnets that you designate for your RDS database.

The subnets belong to a VPC.

For a typical production architecture, the DB Subnet Group contains private subnets in at least two Availability Zones.

Example:

DB Subnet Group
│
├── Private Subnet AZ-1
│
└── Private Subnet AZ-2

RDS can then use the network configuration provided by these subnets.

3. Why Use Private Subnets?

The database normally should not be directly accessible from the public internet.

A common architecture is:

Internet
   │
   ▼
ALB
   │
   ▼
EC2
   │
   ▼
RDS

Network placement:

Public Subnets
│
└── ALB

Private Application Subnets
│
└── EC2

Private Database Subnets
│
└── RDS

This provides network isolation between the public-facing application and the database.

4. Why Use Multiple Availability Zones?

Using subnets in different Availability Zones provides network placement across separate AZs.

Example:

VPC
│
├── AZ-1
│   └── Private DB Subnet
│
└── AZ-2
    └── Private DB Subnet

This is important for architectures that require higher availability.

If a database deployment uses Multi-AZ capabilities, AWS can use resources across Availability Zones to provide failover.

Therefore, the DB Subnet Group should not contain only one subnet in one Availability Zone for a typical highly available design.

5. Prerequisites

Before creating the DB Subnet Group, you should already have:

VPC
│
├── Private Subnet AZ-1
│
└── Private Subnet AZ-2

For this training example:

VPC CIDR:
10.0.0.0/16

Private Subnet AZ-1:
10.0.11.0/24

Private Subnet AZ-2:
10.0.12.0/24

The subnets must belong to the same VPC.

6. Project Structure

Use the following structure:

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
    └── database/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf

The DB Subnet Group can be managed inside the existing database module.

7. Create Database Subnet Group

Open:

modules/database/main.tf

Add:

resource "aws_db_subnet_group" "database" {

  name = "${var.environment}-db-subnet-group"

  subnet_ids = var.subnet_ids

  tags = {
    Name        = "${var.environment}-db-subnet-group"
    Environment = var.environment
  }
}
8. Understand the Resource

The Terraform resource is:

resource "aws_db_subnet_group" "database"

This tells Terraform to manage an AWS RDS DB Subnet Group.

Name
name = "${var.environment}-db-subnet-group"

If:

environment = training

Terraform creates:

training-db-subnet-group
Subnet IDs
subnet_ids = var.subnet_ids

This receives the private subnet IDs from the root module.

Example:

subnet-abc123
subnet-def456
9. Create Variables

Open:

modules/database/variables.tf

Add:

variable "subnet_ids" {
  description = "Private subnet IDs for the database subnet group"
  type        = list(string)
}

variable "environment" {
  description = "Environment name"
  type        = string
}

The module now expects:

subnet_ids
environment
10. Pass Subnets from Root Module

Open:

main.tf

Example:

module "database" {

  source = "./modules/database"

  subnet_ids = [
    module.network.private_subnet_1_id,
    module.network.private_subnet_2_id
  ]

  environment = "training"
}

The flow is:

Root Module
     │
     │ private subnet IDs
     ▼
Database Module
     │
     ▼
DB Subnet Group
11. If Your Network Module Has Different Output Names

Your existing network module might use names such as:

module.network.private_subnet_ids

instead of:

module.network.private_subnet_1_id
module.network.private_subnet_2_id

For example:

module "database" {

  source = "./modules/database"

  subnet_ids = module.network.private_subnet_ids

  environment = "training"
}

Use the actual output names defined by your network module.

12. Network Module Outputs

For example, your network module can expose private subnet IDs.

In:

modules/network/outputs.tf

you could have:

output "private_subnet_1_id" {
  value = aws_subnet.private_1.id
}

output "private_subnet_2_id" {
  value = aws_subnet.private_2.id
}

Then the root module can consume them:

module.network.private_subnet_1_id
module.network.private_subnet_2_id
13. Complete Database Subnet Group Module
modules/database/variables.tf
variable "subnet_ids" {
  description = "Private subnet IDs for the database subnet group"
  type        = list(string)
}

variable "environment" {
  description = "Environment name"
  type        = string
}
modules/database/main.tf
resource "aws_db_subnet_group" "database" {

  name = "${var.environment}-db-subnet-group"

  subnet_ids = var.subnet_ids

  tags = {
    Name        = "${var.environment}-db-subnet-group"
    Environment = var.environment
  }
}
modules/database/outputs.tf
output "db_subnet_group_name" {
  description = "Name of the RDS DB subnet group"

  value = aws_db_subnet_group.database.name
}
14. Root Module

Example:

module "database" {

  source = "./modules/database"

  subnet_ids = [
    module.network.private_subnet_1_id,
    module.network.private_subnet_2_id
  ]

  environment = "training"
}
15. Root Output

In:

outputs.tf

add:

output "db_subnet_group_name" {
  description = "RDS DB subnet group name"

  value = module.database.db_subnet_group_name
}

After deployment, Terraform can display:

db_subnet_group_name = "training-db-subnet-group"
16. Initialize Terraform

From the Terraform root directory:

terraform init

This initializes the Terraform project and downloads the required provider.

17. Format the Configuration

Run:

terraform fmt

This formats the Terraform files.

18. Validate Configuration

Run:

terraform validate

Expected result:

Success! The configuration is valid.
19. Review the Plan

Run:

terraform plan

Look for:

aws_db_subnet_group.database

Terraform should show that it plans to create the DB Subnet Group.

20. Apply the Configuration

Run:

terraform apply

Review the proposed changes.

Enter:

yes

Terraform will create the DB Subnet Group.

21. Verify Using Terraform

Run:

terraform output db_subnet_group_name

Expected example:

"training-db-subnet-group"

You can also inspect the state:

terraform state list

You should see something similar to:

module.database.aws_db_subnet_group.database
22. Important: DB Subnet Group Does Not Create RDS

A DB Subnet Group and an RDS instance are different resources.

DB Subnet Group
      │
      │ Provides subnet placement
      ▼
RDS Instance

Creating:

resource "aws_db_subnet_group"

does not automatically create:

resource "aws_db_instance"

The RDS instance must reference the subnet group.

Example:

resource "aws_db_instance" "database" {

  ...

  db_subnet_group_name = aws_db_subnet_group.database.name

  ...
}
23. Complete Architecture

After this task, your architecture becomes:

                         VPC
                          │
          ┌───────────────┴───────────────┐
          │                               │
          ▼                               ▼
   Private Subnet AZ-1             Private Subnet AZ-2
   10.0.11.0/24                    10.0.12.0/24
          │                               │
          └───────────────┬───────────────┘
                          │
                          ▼
                  DB Subnet Group
                          │
                          ▼
                       RDS

With the complete application:

                         Internet
                            │
                            ▼
                           ALB
                            │
                            ▼
                    EC2 / Auto Scaling
                            │
                            ▼
                    DB Subnet Group
                            │
                            ▼
                           RDS
24. Important Security Point

A DB Subnet Group itself does not provide all database security.

Security should be implemented using multiple controls:

Private Subnets
      +
Security Group
      +
No Public Accessibility
      +
Encryption
      +
Backups

For example, the RDS Security Group should allow the database port only from the application Security Group.

For PostgreSQL:

EC2 SG
  │
  │ TCP 5432
  ▼
RDS SG

For MySQL:

EC2 SG
  │
  │ TCP 3306
  ▼
RDS SG

Do not use:

0.0.0.0/0

for database access in a normal application architecture.

25. Troubleshooting
Problem 1 – Subnet IDs are incorrect

Check:

terraform plan

Verify that the subnet IDs belong to the expected VPC.

Problem 2 – Only One Availability Zone

Check the private subnet configuration.

You should have subnets in different AZs:

AZ-1 → Private DB Subnet
AZ-2 → Private DB Subnet
Problem 3 – Invalid Subnet IDs

Make sure the subnet outputs from the network module are correctly referenced.

Check:

module.network.private_subnet_1_id
module.network.private_subnet_2_id
Problem 4 – Wrong VPC

All subnets used by the DB Subnet Group must belong to the same VPC.

Architecture:

VPC A
├── Private Subnet 1
└── Private Subnet 2

Do not mix:

VPC A → Subnet 1
VPC B → Subnet 2
26. Verification Checklist

Before completing the task, verify:

☑ VPC exists
☑ Private subnet AZ-1 exists
☑ Private subnet AZ-2 exists
☑ Both subnets belong to the same VPC
☑ Subnets are in different Availability Zones
☑ DB Subnet Group created
☑ Terraform manages the DB Subnet Group
☑ Terraform validation succeeds
☑ Terraform plan succeeds
☑ Terraform state contains the DB Subnet Group
27. Key Concepts Learned
VPC

Provides the isolated network for AWS resources.

Private Subnet

A subnet intended for resources that should not be directly reachable from the internet.

Availability Zone

A separate infrastructure location within an AWS Region.

DB Subnet Group

A collection of subnets that provides network placement for RDS databases.

RDS

Managed relational database service.

Terraform Module

Reusable Terraform configuration that helps organize infrastructure.

Final Summary

The database layer should be separated from the public application layer.

The network architecture is:

VPC
 ↓
Private Subnets
 ↓
DB Subnet Group
 ↓
RDS

For a multi-AZ architecture:

                 VPC
                  │
       ┌──────────┴──────────┐
       │                     │
    AZ-1                   AZ-2
       │                     │
 Private Subnet          Private Subnet
       │                     │
       └──────────┬──────────┘
                  │
           DB Subnet Group
                  │
                  ▼
                 RDS

The DB Subnet Group organizes the private subnets that RDS can use for database placement. Using subnets across multiple Availability Zones supports architectures designed for higher availability and failover.

Deliverable

Terraform-managed resource:

modules/database/main.tf
modules/database/variables.tf
modules/database/outputs.tf

Main resource:

resource "aws_db_subnet_group" "database" {
  name       = "${var.environment}-db-subnet-group"
  subnet_ids = var.subnet_ids
}

Validation commands:

terraform fmt
terraform validate
terraform plan
terraform apply