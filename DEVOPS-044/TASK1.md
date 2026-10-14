Provisioning means creating cloud resources automatically using code instead of manually creating each resource in the AWS Console.
1. Understand the workflow
Terraform configuration
main.tf · variables.tf · outputs.tf

terraform init
Download required providers

terraform plan
Preview resources to create or change

terraform apply
Create the infrastructure in AWS

AWS infrastructure
For this exercise: one S3 bucket


We'll start with a small, low-complexity example using an Amazon S3 bucket. After that, you can apply the same workflow to your VPC, EC2, ALB, ECS, and RDS infrastructure.
2. Prerequisites
Before starting, make sure you have:
- An AWS account.
- Terraform installed on your computer.
- AWS CLI installed and configured.
- An IAM identity with permission to create and manage the resources in this exercise.
Check the installations in your terminal:
terraform -version
aws --version
aws sts get-caller-identity


The last command verifies which AWS identity your CLI is using. If Terraform or AWS CLI is not recognized, fix that before proceeding.
Security: Do not put AWS access keys directly in Terraform files or commit credentials to GitHub.
3. Create your Terraform project
In your terminal, create a project directory:
mkdir terraform-provisioning
cd terraform-provisioning


Create these files:
terraform-provisioning/

main.tf

variables.tf

outputs.tf



4. Write the Terraform configuration
File 1: main.tf
This file defines the AWS provider and the resources Terraform should create.

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

resource "aws_s3_bucket" "training" {
  bucket = var.bucket_name

  tags = {
    Name        = "DevOps-Training"
    Environment = "training"
    ManagedBy   = "Terraform"
  }
}

resource "aws_s3_bucket_public_access_block" "training" {
  bucket = aws_s3_bucket.training.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_versioning" "training" {
  bucket = aws_s3_bucket.training.id

  versioning_configuration {
    status = "Enabled"
  }
}



Understand the code:
- terraform: declares the required provider.
- provider "aws": tells Terraform which AWS region to use.
- resource "aws_s3_bucket": defines the S3 bucket.
- aws_s3_bucket_public_access_block: prevents public access settings from being enabled.
- aws_s3_bucket_versioning: enables version history for stored objects.
- tags: helps identify and organize resources.
File 2: variables.tf
Variables let you change configuration without rewriting your infrastructure.

variable "aws_region" {
  description = "AWS region for the resources"
  type        = string
  default     = "ap-south-1"
}

variable "bucket_name" {
  description = "Globally unique S3 bucket name"
  type        = string
}



The region is set to Mumbai (ap-south-1). The bucket name must be globally unique across S3.
File 3: terraform.tfvars
Create this file and provide a unique bucket name. Replace the example with your own unique lowercase name.

bucket_name = "devops-training-sathvik-2026-unique123"



Do not use that exact example if it is already taken.
File 4: outputs.tf
Outputs show useful information after Terraform creates the bucket.

output "bucket_name" {
  description = "Created S3 bucket name"
  value       = aws_s3_bucket.training.bucket
}

output "bucket_arn" {
  description = "Created S3 bucket ARN"
  value       = aws_s3_bucket.training.arn
}



5. Provision the infrastructure
Run these commands one at a time, from inside your project directory.

Step 1 — Initialize Terraform
terraform init


Downloads the AWS provider and prepares the working directory.




Step 2 — Format and validate
terraform fmt
terraform validate


Formats your files and checks whether the configuration is valid.




Step 3 — Preview changes
terraform plan


Review the resources Terraform intends to create before making changes.




Step 4 — Apply changes
terraform apply


Terraform displays a plan and asks for confirmation. Review the plan, then enter yes only if the changes are correct and expected.




Step 5 — Verify the result
terraform output
aws s3 ls


Check Terraform's outputs and confirm that the bucket exists in your AWS account.



These commands describe the expected workflow; the infrastructure is not created until you execute terraform apply successfully.
6. Understand the Terraform state
Terraform stores information about managed resources in a file named terraform.tfstate. This lets Terraform compare the resources it knows about with your desired configuration.
- Keep the state file private.
- Never commit state files, .terraform/, or credentials to Git.
- For a team or production environment, configure a secured remote backend with state locking appropriate to your Terraform version and backend.
7. Day's practical assignment
Provisioning checklist
0 of 7



[ ] Create all Terraform configuration files
[ ] Run terraform init successfully
[ ] Run terraform fmt and terraform validate
[ ] Review the terraform plan
[ ] Apply the configuration and verify the S3 bucket
[ ] Capture terraform output and document the result
[ ] Check that state files and credentials are excluded from Git

Important cleanup
If this was only a temporary exercise and you no longer need the bucket, inspect the proposed deletion first:
terraform plan -destroy