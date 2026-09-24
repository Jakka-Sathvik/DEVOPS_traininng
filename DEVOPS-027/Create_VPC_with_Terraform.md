TASK 4 — Create VPC with Terraform (Step-by-Step)
Objective

Use the VPC module created in Task 3 to create AWS networking infrastructure.

Architecture:

Terraform Root Module
        │
        ▼
      VPC Module
        │
        ▼
       VPC
Step 1 — Create Root Module Structure

Create the following structure:

terraform/
│
├── main.tf
├── variables.tf
├── outputs.tf
│
└── modules/
    └── vpc/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
Step 2 — Configure AWS Provider

Create:

main.tf
terraform {

  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}
Step 3 — Call the VPC Module

Add below the provider block in main.tf:

module "vpc" {

  source = "./modules/vpc"

  environment = "dev"

  vpc_cidr = "10.0.0.0/16"

  availability_zones = [
    "ap-south-1a",
    "ap-south-1b"
  ]

  public_subnet_cidrs = [
    "10.0.1.0/24",
    "10.0.2.0/24"
  ]

  private_subnet_cidrs = [
    "10.0.11.0/24",
    "10.0.12.0/24"
  ]
}
Step 4 — Create Root Outputs

Create:

outputs.tf
output "vpc_id" {
  value = module.vpc.vpc_id
}

output "public_subnets" {
  value = module.vpc.public_subnet_ids
}

output "private_subnets" {
  value = module.vpc.private_subnet_ids
}
Step 5 — Check Terraform Formatting

Run:

terraform fmt

Purpose:

Fix indentation
Format Terraform code
Maintain standard style

Expected Output:

main.tf
outputs.tf

If no output appears:

All files already formatted.
Step 6 — Validate Configuration

Run:

terraform validate

Purpose:

Check Terraform syntax
Verify references
Validate module structure

Expected Output:

Success! The configuration is valid.
Step 7 — Initialize Terraform

If not already initialized:

terraform init

Terraform will:

1. Download AWS Provider
2. Load Modules
3. Create .terraform folder
4. Create lock file

Expected Output:

Terraform has been successfully initialized!
Step 8 — Generate Execution Plan

Run:

terraform plan

Purpose:

Shows what Terraform will create
No resources are created yet
Safe to review before deployment
Expected Infrastructure in Plan

Terraform should identify resources from the VPC module:

VPC
│
├── Internet Gateway
│
├── Public Subnet 1
├── Public Subnet 2
│
├── Private Subnet 1
├── Private Subnet 2
│
├── Elastic IP
│
├── NAT Gateway
│
├── Public Route Table
│
├── Private Route Table
│
├── Public Associations
│
└── Private Associations
Example Plan Output
Terraform will perform the following actions:

+ aws_vpc.main

+ aws_internet_gateway.igw

+ aws_subnet.public[0]

+ aws_subnet.public[1]

+ aws_subnet.private[0]

+ aws_subnet.private[1]

+ aws_eip.nat

+ aws_nat_gateway.nat

+ aws_route_table.public

+ aws_route_table.private

+ aws_route_table_association.public[0]

+ aws_route_table_association.public[1]

+ aws_route_table_association.private[0]

+ aws_route_table_association.private[1]
