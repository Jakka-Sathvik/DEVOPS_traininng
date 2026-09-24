Module Inputs and Outputs
1. Understand the Flow

The complete flow is:

Root variables
      │
      ▼
Module input
      │
      ▼
Child module
      │
      ▼
AWS Infrastructure
      │
      ▼
Module output
      │
      ▼
Root output

Think of the module like a function:

INPUT → MODULE → OUTPUT

For example:

environment = "training"
        ↓
network module
        ↓
VPC + Subnet
        ↓
vpc_id + subnet_id
2. Project Structure

Continue using your TASK 9 project:

terraform/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
│
└── modules/
    └── network/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
3. Create Root Variable — environment

Open:

terraform/variables.tf

Add:

variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "training"
}

Now the root module has:

environment
    ↓
training
4. Add More Useful Inputs

For this task, we need at least 2 inputs.

Let's use:

environment
vpc_cidr

Your root variables.tf can contain:

variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "training"
}

variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "subnet_cidr" {
  description = "CIDR block for the subnet"
  type        = string
  default     = "10.0.1.0/24"
}

You now have four root variables.

5. Add Variables to the Child Module

Open:

terraform/modules/network/variables.tf

Add:

variable "environment" {
  description = "Deployment environment"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
}

variable "subnet_cidr" {
  description = "CIDR block for the subnet"
  type        = string
}

Notice the difference.

Root module:

default = "training"

Child module:

variable "environment" {
  type = string
}

The child doesn't need to decide the value. The root module will provide it.

6. Pass Inputs into the Module

Open:

terraform/main.tf

Your module block should look like:

module "network" {
  source = "./modules/network"

  environment = var.environment
  vpc_cidr    = var.vpc_cidr
  subnet_cidr = var.subnet_cidr
}

This is the most important part of TASK 10.

Look at:

environment = var.environment

The left side:

environment

is the child module input.

The right side:

var.environment

is the root variable.

So:

Root variable
var.environment
      │
      ▼
Module input
environment
7. Use the Inputs Inside the Network Module

Open:

terraform/modules/network/main.tf

Use the variables:

resource "aws_vpc" "training_vpc" {
  cidr_block = var.vpc_cidr

  tags = {
    Name        = "training-vpc"
    Environment = var.environment
  }
}

resource "aws_subnet" "training_subnet" {
  vpc_id     = aws_vpc.training_vpc.id
  cidr_block = var.subnet_cidr

  tags = {
    Name        = "training-subnet"
    Environment = var.environment
  }
}

resource "aws_internet_gateway" "training_igw" {
  vpc_id = aws_vpc.training_vpc.id

  tags = {
    Name        = "training-igw"
    Environment = var.environment
  }
}

resource "aws_route_table" "training_route_table" {
  vpc_id = aws_vpc.training_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.training_igw.id
  }

  tags = {
    Name        = "training-route-table"
    Environment = var.environment
  }
}

resource "aws_route_table_association" "training_association" {
  subnet_id      = aws_subnet.training_subnet.id
  route_table_id = aws_route_table.training_route_table.id
}

Now the child module uses:

vpc_cidr
subnet_cidr
environment
8. Example of the Input Flow

Suppose the root variable says:

environment = "training"

Then:

Root
 │
 │ var.environment
 │
 ▼
Module
 │
 │ environment
 │
 ▼
VPC
 │
 ▼
Tag:
Environment = "training"

Similarly:

var.vpc_cidr
      │
      ▼
module.network.vpc_cidr
      │
      ▼
aws_vpc.training_vpc
      │
      ▼
10.0.0.0/16
9. Create Module Outputs

Now the child module needs to return information.

Open:

terraform/modules/network/outputs.tf

Use:

output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.training_vpc.id
}

output "subnet_id" {
  description = "ID of the subnet"
  value       = aws_subnet.training_subnet.id
}

output "internet_gateway_id" {
  description = "ID of the Internet Gateway"
  value       = aws_internet_gateway.training_igw.id
}

output "route_table_id" {
  description = "ID of the route table"
  value       = aws_route_table.training_route_table.id
}

output "environment" {
  description = "Environment used by the network module"
  value       = var.environment
}

Now the child module returns:

vpc_id
subnet_id
internet_gateway_id
route_table_id
environment
10. Receive Outputs in the Root Module

Open:

terraform/outputs.tf

Add:

output "vpc_id" {
  description = "VPC ID returned by the network module"
  value       = module.network.vpc_id
}

output "subnet_id" {
  description = "Subnet ID returned by the network module"
  value       = module.network.subnet_id
}

output "internet_gateway_id" {
  description = "Internet Gateway ID returned by the network module"
  value       = module.network.internet_gateway_id
}

output "route_table_id" {
  description = "Route table ID returned by the network module"
  value       = module.network.route_table_id
}

output "environment" {
  description = "Environment returned by the network module"
  value       = module.network.environment
}

Notice:

module.network.vpc_id

This means:

module
  ↓
network
  ↓
vpc_id
11. Complete Communication Flow

Now your project works like this:

                    ROOT MODULE
                        │
             ┌──────────┴──────────┐
             │                     │
             ▼                     ▼
      var.environment         var.vpc_cidr
             │                     │
             └──────────┬──────────┘
                        ▼
                module "network"
                        │
                        ▼
                NETWORK MODULE
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
             VPC      Subnet      IGW
              │
              ▼
         Route Table
              │
              ▼
           Outputs
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
     vpc_id subnet_id route_table_id
       │      │      │
       └──────┼──────┘
              ▼
          ROOT OUTPUT
              │
              ▼
      terraform output
12. Test the Configuration

From the root directory:

terraform fmt -recursive

Then:

terraform validate

Expected:

Success! The configuration is valid.
13. Initialize

Run:

terraform init

If you already initialized the project in TASK 9, Terraform may simply confirm that initialization is complete.

14. Run Plan

Run:

terraform plan

Look for:

module.network.aws_vpc.training_vpc
module.network.aws_subnet.training_subnet
module.network.aws_internet_gateway.training_igw
module.network.aws_route_table.training_route_table

The important thing is that the resources are now under:

module.network
15. Apply

If everything looks correct:

terraform apply

Enter:

yes

Terraform creates the infrastructure.

16. Check Outputs

Run:

terraform output

You should get something similar to:

environment = "training"
internet_gateway_id = "igw-0123456789abcdef"
route_table_id = "rtb-0123456789abcdef"
subnet_id = "subnet-0123456789abcdef"
vpc_id = "vpc-0123456789abcdef"

The actual AWS IDs will be different.

17. Test Individual Outputs

You can check only the VPC:

terraform output vpc_id

Subnet:

terraform output subnet_id

Environment:

terraform output environment
18. Optional — Change the Input

This is a good way to demonstrate that module inputs actually work.

For example, create:

terraform.tfvars

with:

environment = "development"
vpc_cidr    = "10.10.0.0/16"
subnet_cidr = "10.10.1.0/24"

Then run:

terraform plan

Terraform will use:

environment
    ↓
development

vpc_cidr
    ↓
10.10.0.0/16

subnet_cidr
    ↓
10.10.1.0/24

This demonstrates the real purpose of module inputs.

19. Your TASK 10 Deliverable

You need at least 2 inputs.

You now have:

Module inputs
✓ environment
✓ vpc_cidr
✓ subnet_cidr
Module outputs
✓ vpc_id
✓ subnet_id
✓ internet_gateway_id
✓ route_table_id
✓ environment
Commands completed
✓ terraform init
✓ terraform fmt -recursive
✓ terraform validate
✓ terraform plan
✓ terraform apply
✓ terraform output
Final concept

Remember this simple rule:

ROOT
 │
 │ inputs
 ▼
CHILD MODULE
 │
 │ creates infrastructure
 ▼
AWS
 │
 │ outputs
 ▼
ROOT