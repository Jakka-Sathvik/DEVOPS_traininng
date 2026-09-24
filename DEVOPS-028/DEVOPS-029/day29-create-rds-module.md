Day 29 – Create Terraform RDS Module
Objective

Create a reusable Terraform module that provisions an Amazon RDS database instance.

The module should:

Support different database engines
Accept configurable inputs
Produce useful outputs
Follow Infrastructure as Code (IaC) best practices
Be reusable across multiple environments
Architecture
Root Module
     │
     ▼
Database Module
     │
     ▼
Amazon RDS
Project Structure
terraform/
│
├── main.tf
├── variables.tf
├── outputs.tf
│
└── modules/
    └── database/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
Step 1 – Create Database Module

Create directory:

mkdir -p modules/database

Files:

modules/database/main.tf
modules/database/variables.tf
modules/database/outputs.tf
Step 2 – Create Variables
modules/database/variables.tf
variable "db_identifier" {
  description = "Database identifier"
  type        = string
}

variable "engine" {
  description = "Database engine"
  type        = string
}

variable "engine_version" {
  description = "Database engine version"
  type        = string
}

variable "instance_class" {
  description = "Database instance class"
  type        = string
}

variable "allocated_storage" {
  description = "Database storage size"
  type        = number
}

variable "db_name" {
  description = "Database name"
  type        = string
}

variable "username" {
  description = "Database username"
  type        = string
}

variable "password" {
  description = "Database password"
  type        = string
  sensitive   = true
}

variable "port" {
  description = "Database port"
  type        = number
}

variable "subnet_ids" {
  description = "Private subnet IDs"
  type        = list(string)
}

variable "security_group_ids" {
  description = "Database security groups"
  type        = list(string)
}

variable "environment" {
  description = "Environment"
  type        = string
}
Step 3 – Create DB Subnet Group

RDS requires a subnet group.

modules/database/main.tf
resource "aws_db_subnet_group" "database_subnet_group" {

  name = "${var.environment}-db-subnet-group"

  subnet_ids = var.subnet_ids

  tags = {
    Name        = "${var.environment}-db-subnet-group"
    Environment = var.environment
  }
}
Step 4 – Create RDS Instance
modules/database/main.tf
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

  db_subnet_group_name = aws_db_subnet_group.database_subnet_group.name

  vpc_security_group_ids = var.security_group_ids

  publicly_accessible = false

  skip_final_snapshot = true

  multi_az = false

  storage_encrypted = true

  deletion_protection = false

  tags = {
    Name        = "${var.environment}-database"
    Environment = var.environment
  }
}
Step 5 – Create Outputs
modules/database/outputs.tf
output "database_endpoint" {
  description = "Database endpoint"
  value       = aws_db_instance.database.endpoint
}

output "database_port" {
  description = "Database port"
  value       = aws_db_instance.database.port
}

output "database_identifier" {
  description = "Database identifier"
  value       = aws_db_instance.database.identifier
}
Step 6 – Call Module from Root Module
terraform/main.tf
module "database" {

  source = "./modules/database"

  db_identifier = "training-db"

  engine = "postgres"

  engine_version = "16.3"

  instance_class = "db.t3.micro"

  allocated_storage = 20

  db_name = "trainingdb"

  username = "dbadmin"

  password = "ChangeMe123!"

  port = 5432

  subnet_ids = [
    module.network.private_subnet_1_id,
    module.network.private_subnet_2_id
  ]

  security_group_ids = [
    aws_security_group.database_sg.id
  ]

  environment = "training"
}
Step 7 – Root Outputs
terraform/outputs.tf
output "database_endpoint" {
  value = module.database.database_endpoint
}

output "database_port" {
  value = module.database.database_port
}

output "database_identifier" {
  value = module.database.database_identifier
}
Example Security Group

Database should only allow traffic from application servers.

resource "aws_security_group" "database_sg" {

  name   = "database-sg"
  vpc_id = module.network.vpc_id

  ingress {

    from_port = 5432
    to_port   = 5432

    protocol = "tcp"

    security_groups = [
      aws_security_group.application_sg.id
    ]
  }

  egress {

    from_port = 0
    to_port   = 0

    protocol = "-1"

    cidr_blocks = [
      "0.0.0.0/0"
    ]
  }

  tags = {
    Name = "database-sg"
  }
}
Terraform Workflow

Format configuration:

terraform fmt

Validate configuration:

terraform validate

Initialize providers:

terraform init

Review changes:

terraform plan

Create infrastructure:

terraform apply

Destroy infrastructure:

terraform destroy
Expected Architecture
User
 │
 ▼
ALB
 │
 ▼
EC2 Application
 │
 │ PostgreSQL Connection
 ▼
RDS Database
Module Inputs Summary
Input	Purpose
engine	Database engine
engine_version	Database version
instance_class	RDS size
db_name	Database name
username	Database username
password	Database password
port	Database port
subnet_ids	Private subnets
security_group_ids	Database security group
environment	Environment name
Module Outputs Summary
Output	Description
database_endpoint	RDS endpoint
database_port	Database port
database_identifier	RDS identifier
Best Practices
Deploy RDS in private subnets.
Never expose RDS directly to the internet.
Use Security Groups to restrict access.
Enable storage encryption.
Use automated backups in production.
Store passwords in AWS Secrets Manager or Terraform variables, not directly in code.
Use Multi-AZ for production workloads.
Monitor RDS using CloudWatch.
Summary

A Terraform RDS Module allows database infrastructure to be created using reusable code. The module accepts configurable inputs such as database engine, version, instance size, subnets, and security groups, then provisions an Amazon RDS instance and returns useful outputs like endpoint, port, and identifier.

This approach follows Infrastructure as Code best practices and makes deployments consistent, scalable, and easy to manage across development, testing, and production environments.