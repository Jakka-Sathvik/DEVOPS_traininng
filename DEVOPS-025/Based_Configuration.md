Environment-Based Configuration

This task teaches you how to use the same Terraform module for different environments, such as Development (dev) and Testing (test).

The main idea is:

                 Root Terraform
                       │
              Reusable Network Module
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
            DEV                 TEST
             │                   │
        Small resources     Different settings

You don't copy the network module. Both environments reuse the same module.

1. Why Do We Need Environments?

Imagine you have two environments:

Development

You are developing and testing your application.

You may want:

Small EC2
Small network
Lower cost
Simple naming
Test

You may want:

Different CIDR
Different names
Different instance size
Separate resources

You should not blindly use production-sized infrastructure for development.

For example:

DEV
EC2 → t3.micro

TEST
EC2 → t3.small

The exact instance types depend on your workload and AWS pricing/eligibility.

2. Create the Environment Structure

Starting from your TASK 10 project:

terraform/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
│
├── modules/
│   └── network/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
└── environments/
    ├── dev/
    └── test/

The important part is:

environments/
├── dev/
└── test/
3. What Is the Idea?

Instead of doing this:

Same configuration
       │
       ├── Dev
       ├── Test
       └── Production

we do:

                Reusable Module
                      │
            ┌─────────┴─────────┐
            ▼                   ▼
           DEV                 TEST
            │                   │
      Different inputs    Different inputs

For example:

DEV
environment = "dev"
vpc_cidr    = "10.10.0.0/16"

TEST
environment = "test"
vpc_cidr    = "10.20.0.0/16"

The module code remains the same.

4. Create the Dev Environment

Create:

terraform/environments/dev/

Inside it create:

main.tf
variables.tf
outputs.tf
provider.tf
terraform.tfvars

So:

environments/
└── dev/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    ├── provider.tf
    └── terraform.tfvars
5. Dev provider.tf

Create:

environments/dev/provider.tf

Add:

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
6. Dev variables.tf

Create:

environments/dev/variables.tf

Add:

variable "aws_region" {
  description = "AWS region for development"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for development VPC"
  type        = string
}

variable "subnet_cidr" {
  description = "CIDR block for development subnet"
  type        = string
}
7. Dev terraform.tfvars

Create:

environments/dev/terraform.tfvars

Add:

aws_region  = "us-east-1"
environment = "dev"

vpc_cidr    = "10.10.0.0/16"
subnet_cidr = "10.10.1.0/24"

Now Dev has its own configuration.

DEV

Environment
    ↓
dev

VPC
    ↓
10.10.0.0/16

Subnet
    ↓
10.10.1.0/24
8. Dev main.tf

This is where Dev calls the reusable network module.

Create:

environments/dev/main.tf

Add:

module "network" {
  source = "../../modules/network"

  environment = var.environment
  vpc_cidr    = var.vpc_cidr
  subnet_cidr = var.subnet_cidr
}

Look carefully at:

source = "../../modules/network"

Why ../../?

Because:

environments/dev/
       │
       ├── ../ → environments/
       │
       └── ../ → terraform/
                         │
                         └── modules/network

So:

../../modules/network

points to the reusable module.

9. Dev outputs.tf

Create:

environments/dev/outputs.tf

Add:

output "vpc_id" {
  description = "Development VPC ID"
  value       = module.network.vpc_id
}

output "subnet_id" {
  description = "Development subnet ID"
  value       = module.network.subnet_id
}

output "environment" {
  description = "Environment name"
  value       = module.network.environment
}
10. Create the Test Environment

Now create:

environments/test/

Structure:

environments/
├── dev/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── provider.tf
│   └── terraform.tfvars
│
└── test/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    ├── provider.tf
    └── terraform.tfvars
11. Test provider.tf

Create:

environments/test/provider.tf

Use:

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
12. Test variables.tf

Create:

environments/test/variables.tf

Add:

variable "aws_region" {
  description = "AWS region for testing"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for test VPC"
  type        = string
}

variable "subnet_cidr" {
  description = "CIDR block for test subnet"
  type        = string
}
13. Test terraform.tfvars

Create:

environments/test/terraform.tfvars

Add:

aws_region  = "us-east-1"
environment = "test"

vpc_cidr    = "10.20.0.0/16"
subnet_cidr = "10.20.1.0/24"

Notice the difference:

Dev
10.10.0.0/16
Test
10.20.0.0/16

This prevents the two environments from using the same network range.

14. Test main.tf

Create:

environments/test/main.tf

Add:

module "network" {
  source = "../../modules/network"

  environment = var.environment
  vpc_cidr    = var.vpc_cidr
  subnet_cidr = var.subnet_cidr
}

Look at this carefully.

The module code is exactly the same as Dev.

source = "../../modules/network"

The difference is the input values.

15. Test outputs.tf

Create:

environments/test/outputs.tf

Add:

output "vpc_id" {
  description = "Test VPC ID"
  value       = module.network.vpc_id
}

output "subnet_id" {
  description = "Test subnet ID"
  value       = module.network.subnet_id
}

output "environment" {
  description = "Environment name"
  value       = module.network.environment
}
16. Final Project Structure

Your project should now look like:

terraform/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
│
├── modules/
│   └── network/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
└── environments/
    │
    ├── dev/
    │   ├── main.tf
    │   ├── variables.tf
    │   ├── outputs.tf
    │   ├── provider.tf
    │   └── terraform.tfvars
    │
    └── test/
        ├── main.tf
        ├── variables.tf
        ├── outputs.tf
        ├── provider.tf
        └── terraform.tfvars
17. Understand the Reuse

This is the most important part of TASK 11.

Both environments use:

source = "../../modules/network"

Therefore:

             modules/network
             REUSABLE MODULE
                    │
             ┌──────┴──────┐
             ▼             ▼
            DEV           TEST
             │             │
             ▼             ▼
         10.10.0.0/16  10.20.0.0/16

One module → multiple environments.

18. Test the Dev Environment

Open your terminal and go to:

cd environments/dev

Then:

terraform init

You should see Terraform loading:

../../modules/network

Then:

terraform fmt -recursive

Then:

terraform validate

Expected:

Success! The configuration is valid.

Now:

terraform plan

Check that Terraform plans to create the Dev network.

If you want to actually create it:

terraform apply

Enter:

yes

Then:

terraform output

You should see something similar to:

environment = "dev"
subnet_id = "subnet-xxxxxxxx"
vpc_id = "vpc-xxxxxxxx"
19. Test the Test Environment

Now go back:

cd ..

Then:

cd test

Run:

terraform init

Then:

terraform validate

Then:

terraform plan

You should see a different VPC configuration because Test uses:

10.20.0.0/16

instead of Dev:

10.10.0.0/16

If you want to create it:

terraform apply

Then:

terraform output

Expected conceptually:

environment = "test"
subnet_id = "subnet-yyyyyyyy"
vpc_id = "vpc-yyyyyyyy"
20. Dev vs Test
Setting	Dev	Test
Environment	dev	test
VPC CIDR	10.10.0.0/16	10.20.0.0/16
Subnet CIDR	10.10.1.0/24	10.20.1.0/24
Module	Same	Same
Configuration	Development	Testing
Resources	Can be smaller	Can be different

The key point:

The module is shared, but the environment-specific inputs are different.

21. Why This Is Better

Without environment separation:

One configuration
      │
      ├── Dev
      ├── Test
      └── Production

This can become difficult to manage.

With environment separation:

Reusable Module
      │
      ├── environments/dev
      │
      └── environments/test

Each environment can have different:

CIDR ranges
instance sizes
names
tags
scaling settings
resource counts
configuration values

while still using the same reusable Terraform module.

22. Very Important: Separate Terraform State

Because dev and test are separate Terraform root configurations, each directory gets its own state:

environments/dev/terraform.tfstate

and:

environments/test/terraform.tfstate

Conceptually:

DEV
 │
 └── dev state
      │
      └── Dev infrastructure

TEST
 │
 └── test state
      │
      └── Test infrastructure

This is another reason environment separation is useful.

For real team/production setups, remote state and state locking should be considered rather than relying on local state files.

23. What You Should Demonstrate to Your Trainer

Show this:

Same module
source = "../../modules/network"

in both:

environments/dev/main.tf

and:

environments/test/main.tf
Different inputs

Dev:

environment = "dev"
vpc_cidr    = "10.10.0.0/16"

Test:

environment = "test"
vpc_cidr    = "10.20.0.0/16"
Same infrastructure module
modules/network/

is reused by both.

24. TASK 11 Deliverable

Your final deliverable can be summarized as:

TASK 11 — Environment-Based Configuration

✓ Created environments/dev
✓ Created environments/test
✓ Reused the same network Terraform module
✓ Configured different environment variables
✓ Used different VPC CIDR ranges
✓ Created separate Terraform configurations
✓ Verified Dev with terraform plan
✓ Verified Test with terraform plan
✓ Understood environment-specific configuration
One-line explanation for your report

The same reusable Terraform network module is used by both Dev and Test environments, while each environment provides its own configuration values such as environment name, VPC CIDR, and subnet CIDR. This provides consistency through code reuse while allowing environments to have different infrastructure settings.

Cost reminder: if you actually ran terraform apply in both environments, clean them up when finished:

cd environments/dev
terraform destroy

and then:

cd ../test
terraform destroy