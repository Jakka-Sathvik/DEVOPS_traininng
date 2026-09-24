Create a Reusable Terraform Module
1. What is a Terraform Module?

A module is a reusable collection of Terraform configuration files.

Instead of keeping everything in one large main.tf, we can organize infrastructure into separate modules.

For example:

Root Module
    │
    ▼
Network Module
    │
    ├── VPC
    ├── Subnet
    ├── Internet Gateway
    └── Route Table

The advantage is that you can reuse the same network module for:

Development
Production
Testing
Staging

without copying all the network code.

2. Create the Project Structure

Your project should look like this:

terraform/
│
├── main.tf
├── variables.tf
├── outputs.tf
│
└── modules/
    └── network/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf

If your current project already has files such as:

provider.tf
network.tf
security-group.tf
ec2.tf
outputs.tf
variables.tf

that's okay. For this task, we'll specifically move the network resources into the module.

3. Create the Module Folder

From your Terraform project directory:

mkdir modules
mkdir modules/network

Then check:

ls

On Windows PowerShell:

Get-ChildItem

You should have:

modules

Then:

Get-ChildItem modules\network
4. Create modules/network/main.tf

Inside:

modules/network/main.tf

put:

resource "aws_vpc" "training_vpc" {
  cidr_block = var.vpc_cidr

  tags = {
    Name = var.vpc_name
  }
}

resource "aws_subnet" "training_subnet" {
  vpc_id     = aws_vpc.training_vpc.id
  cidr_block = var.subnet_cidr

  tags = {
    Name = var.subnet_name
  }
}

resource "aws_internet_gateway" "training_igw" {
  vpc_id = aws_vpc.training_vpc.id

  tags = {
    Name = var.internet_gateway_name
  }
}

resource "aws_route_table" "training_route_table" {
  vpc_id = aws_vpc.training_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.training_igw.id
  }

  tags = {
    Name = var.route_table_name
  }
}

resource "aws_route_table_association" "training_association" {
  subnet_id      = aws_subnet.training_subnet.id
  route_table_id = aws_route_table.training_route_table.id
}
What happened?

Previously your root project directly contained:

resource "aws_vpc" ...
resource "aws_subnet" ...
resource "aws_internet_gateway" ...
resource "aws_route_table" ...

Now these resources are inside:

modules/network/main.tf

This makes the network reusable.

5. Create modules/network/variables.tf

Create:

modules/network/variables.tf

Add:

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

variable "vpc_name" {
  description = "Name of the VPC"
  type        = string
  default     = "training-vpc"
}

variable "subnet_name" {
  description = "Name of the subnet"
  type        = string
  default     = "training-subnet"
}

variable "internet_gateway_name" {
  description = "Name of the Internet Gateway"
  type        = string
  default     = "training-igw"
}

variable "route_table_name" {
  description = "Name of the route table"
  type        = string
  default     = "training-route-table"
}

These variables allow the root module to customize the network.

For example:

vpc_cidr
    ↓
10.0.0.0/16

and:

subnet_cidr
    ↓
10.0.1.0/24
6. Create modules/network/outputs.tf

Create:

modules/network/outputs.tf

Add:

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

These outputs allow the root module to access resources created by the network module.

7. Create the Root main.tf

Now we need to tell the root Terraform project:

"Use my network module."

In:

terraform/main.tf

add:

module "network" {
  source = "./modules/network"

  vpc_cidr              = var.vpc_cidr
  subnet_cidr           = var.subnet_cidr
  vpc_name              = var.vpc_name
  subnet_name           = var.subnet_name
  internet_gateway_name = var.internet_gateway_name
  route_table_name      = var.route_table_name
}

This is the most important part of the task.

8. Understand the module Block

Look at:

module "network" {
  source = "./modules/network"
}

Terraform understands:

module
  ↓
network
  ↓
./modules/network

So:

source = "./modules/network"

means:

Load the Terraform module from the local modules/network directory.

9. Root variables.tf

Now create/update:

terraform/variables.tf

Use:

variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
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

variable "vpc_name" {
  description = "Name of the VPC"
  type        = string
  default     = "training-vpc"
}

variable "subnet_name" {
  description = "Name of the subnet"
  type        = string
  default     = "training-subnet"
}

variable "internet_gateway_name" {
  description = "Name of the Internet Gateway"
  type        = string
  default     = "training-igw"
}

variable "route_table_name" {
  description = "Name of the route table"
  type        = string
  default     = "training-route-table"
}
10. Root Provider Configuration

If you already have your provider.tf from the previous task, keep it.

It should look similar to:

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

You don't need another provider inside modules/network.

The module will use the provider inherited from the root module.

11. Root outputs.tf

Now expose the module's outputs to the root project.

Create/update:

terraform/outputs.tf

Add:

output "vpc_id" {
  description = "ID of the VPC created by the network module"
  value       = module.network.vpc_id
}

output "subnet_id" {
  description = "ID of the subnet created by the network module"
  value       = module.network.subnet_id
}

output "internet_gateway_id" {
  description = "ID of the Internet Gateway"
  value       = module.network.internet_gateway_id
}

output "route_table_id" {
  description = "ID of the route table"
  value       = module.network.route_table_id
}

Notice this:

module.network.vpc_id

It means:

module
  ↓
network
  ↓
vpc_id
12. Your Final Structure

You should now have:

terraform/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
│
└── modules/
    │
    └── network/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf

The architecture is:

                 ROOT MODULE
                     │
                     │
             module "network"
                     │
                     ▼
              NETWORK MODULE
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
       VPC         Subnet        IGW
                     │
                     ▼
                Route Table
13. Important: Remove Duplicate Network Resources

This is very important.

If your old root project has something like:

network.tf

containing:

resource "aws_vpc" ...
resource "aws_subnet" ...
resource "aws_internet_gateway" ...
resource "aws_route_table" ...

do not keep those same resources in the root module.

Otherwise Terraform will try to manage duplicate infrastructure.

Your network resources should now be inside:

modules/network/main.tf

You can delete the old network.tf after moving its contents into the module.

14. Initialize Terraform

Go to the root directory:

cd terraform

Run:

terraform init

You should see something similar to:

Initializing the backend...
Initializing modules...
- network in modules/network

Initializing provider plugins...

Terraform has been successfully initialized!

The important part is:

Initializing modules...
- network in modules/network

That tells you Terraform found your module.

15. Format the Code

Run:

terraform fmt -recursive

This formats both:

terraform/

and:

terraform/modules/network/
16. Validate

Run:

terraform validate

Expected:

Success! The configuration is valid.

If you see this, your module configuration is syntactically valid.

17. Run Terraform Plan

Now:

terraform plan

Terraform should show resources such as:

aws_vpc.training_vpc
aws_subnet.training_subnet
aws_internet_gateway.training_igw
aws_route_table.training_route_table
aws_route_table_association.training_association

Because they are inside a module, the Terraform addresses may appear like:

module.network.aws_vpc.training_vpc

This is important.

It means:

module.network
    ↓
aws_vpc.training_vpc
18. Apply

If the plan looks correct:

terraform apply

Terraform will ask:

Do you want to perform these actions?
  Enter a value:

Type:

yes

Terraform will create the network.

19. Check the Outputs

Run:

terraform output

You should see something similar to:

internet_gateway_id = "igw-0123456789"
route_table_id      = "rtb-0123456789"
subnet_id           = "subnet-0123456789"
vpc_id              = "vpc-0123456789"

Your actual IDs will be different.

You can also check one output:

terraform output vpc_id

Example:

"vpc-0123456789abcdef"
20. Understand How Data Flows

This is the key learning point for TASK 9.

Root module sends variables
Root variables
      │
      ▼
module "network"
      │
      ▼
Network module variables

For example:

vpc_cidr = var.vpc_cidr

Then inside the module:

cidr_block = var.vpc_cidr

So:

Root
10.0.0.0/16
   │
   ▼
Network Module
10.0.0.0/16
   │
   ▼
AWS VPC
21. Understand Module Outputs

Inside the module:

output "vpc_id" {
  value = aws_vpc.training_vpc.id
}

The module creates the VPC and exposes its ID.

The root module receives it:

output "vpc_id" {
  value = module.network.vpc_id
}

So:

AWS VPC
   │
   ▼
module/network output
   │
   ▼
module.network.vpc_id
   │
   ▼
Root output
   │
   ▼
terraform output
22. Why Modules Are Useful

Without modules:

project1
  └── lots of network code

project2
  └── copied network code

project3
  └── copied network code

With modules:

Reusable Network Module
          │
     ┌────┼────┐
     ▼    ▼    ▼
   Dev   Test  Prod

You can reuse the same module with different values.

For example:

Development
VPC = 10.0.0.0/16

Testing
VPC = 10.1.0.0/16

Production
VPC = 10.2.0.0/16

The module code doesn't need to be copied.

23. TASK 9 Command Checklist

Run these commands in order:

cd terraform
terraform init
terraform fmt -recursive
terraform validate
terraform plan
terraform apply

Then:

terraform output
24. Deliverable

For your training submission, you should be able to show:

Project structure
terraform/
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
└── modules/
    └── network/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
Module call
module "network" {
  source = "./modules/network"

  vpc_cidr              = var.vpc_cidr
  subnet_cidr            = var.subnet_cidr
  vpc_name               = var.vpc_name
  subnet_name            = var.subnet_name
  internet_gateway_name  = var.internet_gateway_name
  route_table_name       = var.route_table_name
}
Successful commands
terraform init       ✓
terraform fmt        ✓
terraform validate   ✓
terraform plan       ✓
terraform apply      ✓
terraform output     ✓
What you learned
✓ What a Terraform module is
✓ Root module
✓ Child module
✓ Module source
✓ Module variables
✓ Module outputs
✓ Calling a local module
✓ Reusing infrastructure code
✓ Passing values into modules
✓ Getting values out of modules

Important: Since this is AWS infrastructure, when you're finished practicing, run:

terraform destroy