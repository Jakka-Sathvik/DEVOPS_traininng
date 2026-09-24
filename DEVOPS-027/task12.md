TASK 12 — Terraform Dependencies & Outputs

This task is about understanding how Terraform modules communicate with each other.

You already have the VPC module from the previous tasks. Now we will connect its outputs to an EC2 module.

The final flow will be:

VPC Module
    │
    ├── VPC ID
    ├── Public Subnet IDs
    └── Private Subnet IDs
             │
             ▼
        EC2 Module
             │
             ▼
        EC2 Instance

The important Terraform concept is:

Module Input
      ↓
Module Resource
      ↓
Module Output
      ↓
Another Module/Resource
Step 1 — Understand the dependency

Your VPC module creates:

VPC
Public Subnets
Private Subnets
Security Group

It then exposes IDs through outputs.

For example:

output "vpc_id" {
  value = aws_vpc.main.id
}

output "public_subnet_ids" {
  value = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}

The root module can consume these values using:

module.vpc.vpc_id

and:

module.vpc.public_subnet_ids

and:

module.vpc.private_subnet_ids
Step 2 — Create the EC2 module

Change your structure to:

terraform/
│
├── main.tf
├── outputs.tf
├── variables.tf
│
└── modules/
    │
    ├── vpc/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    └── ec2/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf

The responsibility of the EC2 module will be:

Receive:
    AMI
    Instance Type
    Subnet ID
    Security Group ID
    Environment

Create:
    EC2 Instance

Return:
    Instance ID
    Public IP
Step 3 — Create EC2 module variables

Create:

modules/ec2/variables.tf

Add:

variable "ami_id" {
  description = "AMI ID for the EC2 instance"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "subnet_id" {
  description = "Subnet ID where EC2 will be deployed"
  type        = string
}

variable "security_group_id" {
  description = "Security group ID for the EC2 instance"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
}

These are the inputs to the EC2 module.

Step 4 — Create the EC2 resource

Open:

modules/ec2/main.tf

Add:

resource "aws_instance" "application" {
  ami           = var.ami_id
  instance_type = var.instance_type

  subnet_id = var.subnet_id

  vpc_security_group_ids = [
    var.security_group_id
  ]

  associate_public_ip_address = true

  tags = {
    Name        = "${var.environment}-application-server"
    Environment = var.environment
    Project     = "terraform-training"
  }
}

Notice something important.

The EC2 module does not create the subnet.

It receives:

var.subnet_id

from the parent module.

This is good module design because the networking module owns networking and the EC2 module owns EC2.

Step 5 — Create EC2 module outputs

Open:

modules/ec2/outputs.tf

Add:

output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.application.id
}

output "public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.application.public_ip
}

output "private_ip" {
  description = "Private IP address of the EC2 instance"
  value       = aws_instance.application.private_ip
}

Now the EC2 module produces outputs.

Step 6 — Make sure the VPC module has outputs

Open:

modules/vpc/outputs.tf

You should have:

output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "IDs of public subnets"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "IDs of private subnets"
  value       = aws_subnet.private[*].id
}

output "application_security_group_id" {
  description = "ID of the application security group"
  value       = aws_security_group.application.id
}

So the VPC module gives the parent module:

VPC ID
Public Subnet IDs
Private Subnet IDs
Security Group ID
Step 7 — Get an AMI dynamically

In the root main.tf, use an Amazon Linux AMI data source:

data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }

  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

Terraform will find an appropriate Amazon Linux AMI in your configured ap-south-1 region.

Step 8 — Call the VPC module

Your root main.tf should contain your existing VPC module:

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

  admin_cidr = "YOUR_PUBLIC_IP/32"
}
Step 9 — Connect VPC → EC2

Now comes the most important part of TASK 12.

Add:

module "ec2" {
  source = "./modules/ec2"

  ami_id = data.aws_ami.amazon_linux.id

  instance_type = "t3.micro"

  subnet_id = module.vpc.public_subnet_ids[0]

  security_group_id = module.vpc.application_security_group_id

  environment = "dev"
}

Look carefully at:

subnet_id = module.vpc.public_subnet_ids[0]

This means:

VPC Module
     │
     │ public_subnet_ids
     ▼
EC2 Module
     │
     │ subnet_id
     ▼
EC2 Instance

And:

security_group_id = module.vpc.application_security_group_id

means the security group created by the VPC module is passed to the EC2 module.

Step 10 — Understand [0]

Your VPC module creates two public subnets:

public_subnet_ids = [
    subnet-A,
    subnet-B
]

Terraform indexes them:

[0] → first subnet
[1] → second subnet

Therefore:

module.vpc.public_subnet_ids[0]

means:

Use the first public subnet.

In your architecture, that corresponds to approximately:

10.0.1.0/24

in:

ap-south-1a
Step 11 — Root outputs

Open:

outputs.tf

Add:

output "vpc_id" {
  description = "VPC ID"
  value       = module.vpc.vpc_id
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = module.vpc.public_subnet_ids
}

output "private_subnet_ids" {
  description = "Private subnet IDs"
  value       = module.vpc.private_subnet_ids
}

output "ec2_instance_id" {
  description = "EC2 instance ID"
  value       = module.ec2.instance_id
}

output "ec2_public_ip" {
  description = "EC2 public IP"
  value       = module.ec2.public_ip
}

output "ec2_private_ip" {
  description = "EC2 private IP"
  value       = module.ec2.private_ip
}

Now the root module exposes information from both modules.

Step 12 — Format the code

Run:

terraform fmt
Step 13 — Initialize

Because we created a new module, run:

terraform init

You should see Terraform recognize:

Initializing modules...
- ec2 in modules/ec2
- vpc in modules/vpc
Step 14 — Validate

Run:

terraform validate

Expected:

Success! The configuration is valid.
Step 15 — Create a plan

Run:

terraform plan

Terraform should understand the dependency automatically.

The logical dependency is:

VPC
 │
 ├── Public Subnet
 │
 └── Security Group
          │
          ▼
      EC2 Module
          │
          ▼
      EC2 Instance

You don't need to manually write:

depends_on = [module.vpc]

in this case.

Why?

Because Terraform sees:

module.vpc.public_subnet_ids[0]

and:

module.vpc.application_security_group_id

The EC2 module depends on values produced by the VPC module.

Terraform therefore builds the dependency graph automatically.

Step 16 — Understand the complete data flow

This is the main deliverable.

Input

Root module sends:

environment
vpc_cidr
availability_zones
subnet CIDRs
admin CIDR

to the VPC module.

Root Module
     │
     ▼
VPC Module
Resource creation

The VPC module creates:

VPC
Public Subnets
Private Subnets
Security Group
Outputs

The VPC module returns:

vpc_id
public_subnet_ids
private_subnet_ids
security_group_id
EC2 module input

The root module takes:

public_subnet_ids[0]
security_group_id

and sends them to the EC2 module.

VPC Module
    │
    ├── public_subnet_ids ──────┐
    │                           │
    └── security_group_id ──────┤
                                ▼
                           EC2 Module
                                │
                                ▼
                           EC2 Instance
Final dependency diagram
                    ROOT MODULE
                         │
             ┌───────────┴───────────┐
             │                       │
             ▼                       │
        VPC MODULE                   │
             │                       │
     ┌───────┼────────┐              │
     ▼       ▼        ▼              │
    VPC   Public    Private           │
          Subnets   Subnets           │
             │                        │
             │ public_subnet_ids      │
             │ security_group_id      │
             └───────────────┐        │
                             ▼        │
                        EC2 MODULE ◄──┘
                             │
                             ▼
                       EC2 INSTANCE
The key Terraform pattern

Remember this:

Module Input
     ↓
Resource
     ↓
Module Output
     ↓
Another Module Input
     ↓
Another Resource

In your project:

VPC Module
     ↓
Creates VPC/Subnets/SG
     ↓
Outputs subnet IDs + SG ID
     ↓
Root Module
     ↓
Passes IDs to EC2 Module
     ↓
EC2 Module
     ↓
Creates EC2
TASK 12 checklist
Requirement	Status
VPC module created	✅
VPC outputs created	✅
EC2 module created	✅
EC2 module inputs created	✅
EC2 module outputs created	✅
VPC → EC2 connection	✅
Automatic dependency	✅
terraform init	✅
terraform validate	✅
terraform plan	✅