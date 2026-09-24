TASK 1 — Terraform Advanced Concepts Revision
Review of Previous Concepts
1. Terraform

Terraform is an Infrastructure as Code (IaC) tool developed by HashiCorp. It allows engineers to create, manage, and automate infrastructure using code instead of manual configuration.

Example:

resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
2. Provider

A provider is a plugin that enables Terraform to interact with cloud platforms and services.

Examples:

AWS Provider
Azure Provider
Google Cloud Provider
provider "aws" {
  region = "us-east-1"
}
3. Resource

A resource represents an infrastructure component managed by Terraform.

Examples:

EC2 Instance
S3 Bucket
VPC
resource "aws_s3_bucket" "demo" {
  bucket = "demo-bucket"
}
4. Variable

Variables allow values to be reused and customized.

variable "instance_type" {
  default = "t2.micro"
}
5. Output

Outputs display useful information after Terraform execution.

output "instance_id" {
  value = aws_instance.web.id
}
6. State

Terraform State keeps track of the infrastructure managed by Terraform.

File:

terraform.tfstate

Purpose:

Tracks resources
Maps code to real infrastructure
Stores metadata
7. Plan

Terraform Plan shows what changes Terraform will make.

terraform plan

Purpose:

Preview changes
Validate configuration
Avoid mistakes
8. Apply

Apply executes the planned changes.

terraform apply

Purpose:

Create resources
Modify resources
Update infrastructure
9. Destroy

Destroy removes infrastructure managed by Terraform.

terraform destroy

Purpose:

Clean up resources
Reduce cloud costs
Remove test environments
Advanced Concepts
10. Resource Reference

A resource can reference another resource's attributes.

Example:

resource "aws_instance" "web" {
  subnet_id = aws_subnet.public.id
}

Terraform automatically retrieves the subnet ID.

11. Dependency

Dependencies define the order in which resources are created.

Implicit Dependency

Terraform detects automatically.

subnet_id = aws_subnet.public.id
Explicit Dependency
depends_on = [aws_vpc.main]

Purpose:

Correct creation order
Avoid deployment failures
12. Data Source

Data sources allow Terraform to read existing infrastructure information.

Example:

data "aws_ami" "amazon_linux" {
  most_recent = true
}

Purpose:

Use existing resources
Fetch cloud information
Avoid hardcoding values
13. Module

A module is a reusable collection of Terraform files.

Structure:

modules/
 └── ec2/
      ├── main.tf
      ├── variables.tf
      └── outputs.tf

Benefits:

Reusability
Standardization
Easier maintenance
14. Module Input

Inputs are variables passed to a module.

Example:

module "webserver" {
  source        = "./modules/ec2"
  instance_type = "t2.micro"
}

Purpose:

Customize modules
Reuse code
15. Module Output

Outputs expose information from a module.

Inside module:

output "instance_id" {
  value = aws_instance.web.id
}

Usage:

module.webserver.instance_id
16. Environment

An environment represents a separate deployment stage.

Common Environments:

Development (Dev)
Testing (QA)
Staging
Production (Prod)

Example Structure:

environments/
 ├── dev
 ├── staging
 └── prod

Benefits:

Isolation
Safer deployments
Better testing
17. Reusable Infrastructure

Reusable Infrastructure means writing Terraform code once and using it many times.

Example:

VPC Module
EC2 Module
S3 Module

Benefits:

Faster deployment
Consistency
Reduced errors
Easier maintenance
Summary

Terraform concepts covered:

Terraform
Provider
Resource
Variable
Output
State
Plan
Apply
Destroy
Resource Reference
Dependency
Data Source
Module
Module Input
Module Output
Environment
Reusable Infrastructure