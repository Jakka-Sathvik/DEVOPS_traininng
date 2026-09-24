1. Create the folder structure

From your Terraform project:

terraform/
├── main.tf
├── variables.tf
├── outputs.tf
└── modules/
    └── compute/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf

If you are using your Codespace:

cd /workspaces/terraform-lab
mkdir -p day28/terraform/modules/compute
cd day28/terraform
2. Create modules/compute/main.tf

This file defines the EC2 resource.

resource "aws_instance" "this" {
  ami                    = var.ami
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids
  user_data              = var.user_data

  tags = var.tags
}
What this does

Terraform will create an EC2 instance using values supplied by the root module.

For example:

var.ami
     ↓
EC2 AMI

var.instance_type
     ↓
EC2 Instance Type

var.subnet_id
     ↓
EC2 Subnet

var.security_group_ids
     ↓
Security Groups

var.user_data
     ↓
Startup Script

var.tags
     ↓
EC2 Tags
3. Create modules/compute/variables.tf

This file defines the inputs required by the module.

variable "ami" {
  description = "AMI ID used to launch the EC2 instance"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "subnet_id" {
  description = "Subnet ID where the EC2 instance will be launched"
  type        = string
}

variable "security_group_ids" {
  description = "List of security group IDs"
  type        = list(string)
}

variable "tags" {
  description = "Tags assigned to the EC2 instance"
  type        = map(string)
  default     = {}
}

variable "user_data" {
  description = "Startup script for the EC2 instance"
  type        = string
  default     = null
}
4. Create modules/compute/outputs.tf

The module should return useful information about the EC2 instance.

output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.this.id
}

output "private_ip" {
  description = "Private IP address of the EC2 instance"
  value       = aws_instance.this.private_ip
}

output "public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.this.public_ip
}

Now the module can expose:

EC2 Instance ID
Private IP
Public IP
5. Create the Root main.tf

The root module calls the compute module.

terraform {
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

module "compute" {
  source = "./modules/compute"

  ami           = "ami-xxxxxxxxxxxxxxxxx"
  instance_type = "t3.micro"

  subnet_id = "subnet-xxxxxxxxxxxxxxxxx"

  security_group_ids = [
    "sg-xxxxxxxxxxxxxxxxx"
  ]

  tags = {
    Name        = "day28-compute-instance"
    Environment = "training"
  }

  user_data = <<-EOF
    #!/bin/bash
    echo "Day 28 Terraform EC2" > /tmp/day28.txt
  EOF
}

Important: Replace the example AMI, subnet, and security-group IDs with IDs from your AWS environment. Don't hard-code AWS access keys into Terraform.

6. Create Root outputs.tf
output "instance_id" {
  description = "EC2 instance ID"
  value       = module.compute.instance_id
}

output "private_ip" {
  description = "EC2 private IP"
  value       = module.compute.private_ip
}

output "public_ip" {
  description = "EC2 public IP"
  value       = module.compute.public_ip
}

The root module receives the outputs from the compute module:

Compute Module
      │
      ├── instance_id
      ├── private_ip
      └── public_ip
             │
             ▼
        Root Module
7. Final Directory Structure

Your project should now look like:

day28/
└── terraform/
    ├── main.tf
    ├── outputs.tf
    ├── variables.tf
    │
    └── modules/
        └── compute/
            ├── main.tf
            ├── variables.tf
            └── outputs.tf

You can leave the root variables.tf empty for this basic exercise, or use it later to make the root configuration fully parameterized.

8. Understand the Module Architecture

The important concept is:

                    ROOT MODULE
                         │
                         │ module "compute"
                         ▼
                 ┌─────────────────┐
                 │ COMPUTE MODULE  │
                 │                 │
                 │ main.tf         │
                 │ variables.tf   │
                 │ outputs.tf     │
                 └────────┬────────┘
                          │
                          ▼
                     AWS EC2

The root module provides values:

Root
 │
 ├── AMI
 ├── Instance Type
 ├── Subnet
 ├── Security Group
 ├── Tags
 └── User Data
       │
       ▼
Compute Module
       │
       ▼
    EC2 Instance
9. Initialize Terraform

From:

cd /workspaces/terraform-lab/day28/terraform

Run:

terraform init

You should see Terraform initialize the AWS provider and the local module.

Then:

terraform fmt

This formats your Terraform files.

Then:

terraform validate

Expected:

Success! The configuration is valid.
10. Review the Plan

Run:

terraform plan

Terraform should show an EC2 instance that it plans to create.

You should see something similar to:

Plan: 1 to add, 0 to change, 0 to destroy.

Do not run terraform apply until your AMI, subnet, security group, and AWS credentials are correctly configured.

11. Test the Module

If you are ready to create the instance:

terraform apply

Terraform asks:

Do you want to perform these actions?

Enter:

yes

After successful creation, check:

terraform output

Example:

instance_id = "i-0123456789abcdef0"
private_ip  = "10.0.1.25"
public_ip   = "3.xx.xx.xx"
12. Verify the Module

Your reusable module now supports all six requested inputs:

Requirement	Implemented
AMI	✅
Instance Type	✅
Subnet	✅
Security Group	✅
Tags	✅
User Data	✅
Key learning

The biggest Terraform concept in this task is separation of responsibility:

Root Module
    │
    │ supplies values
    ▼
Compute Module
    │
    │ creates resources
    ▼
AWS EC2

The same compute module can later be reused:

module "web_server" {
  source = "./modules/compute"
  ...
}

module "app_server" {
  source = "./modules/compute"
  ...
}

That is the main advantage of a Terraform module: write the infrastructure logic once and reuse it with different inputs.