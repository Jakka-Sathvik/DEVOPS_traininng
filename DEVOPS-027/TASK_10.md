TASK 10 — Deploy EC2 into Public Subnet
Step 1 — Go to your Terraform project

Your structure should currently look approximately like:

terraform/
├── main.tf
├── outputs.tf
├── variables.tf
│
└── modules/
    └── vpc/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf

We will add EC2 configuration to the root Terraform project.

Step 2 — Get the latest Amazon Linux AMI

Instead of manually entering an AMI ID, use a Terraform data source.

Add this to your root main.tf:

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
Why use this?

Terraform will find a suitable Amazon Linux 2023 AMI in your configured region.

So you don't need to manually write something like:

ami = "ami-xxxxxxxx"

which could become invalid if the AMI belongs to another region.

Step 3 — Create the EC2 instance

Add:

resource "aws_instance" "application" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  subnet_id = module.vpc.public_subnet_ids[0]

  vpc_security_group_ids = [
    module.vpc.application_security_group_id
  ]

  associate_public_ip_address = true

  tags = {
    Name        = "${var.environment}-application-server"
    Environment = var.environment
    Project     = "terraform-training"
  }
}
Important

This assumes your VPC module outputs the security group ID.

Currently your module outputs include:

vpc_id
public_subnet_ids
private_subnet_ids

So we need to add the security group output.

Step 4 — Add Security Group output

Open:

modules/vpc/outputs.tf

Add:

output "application_security_group_id" {
  description = "ID of the application security group"
  value       = aws_security_group.application.id
}

Your outputs can now look like:

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
Step 5 — Check your security group

Your application security group from TASK 9 should contain:

HTTP
TCP 80
Source: 0.0.0.0/0
HTTPS
TCP 443
Source: 0.0.0.0/0
SSH
TCP 22
Source: YOUR_PUBLIC_IP/32

For example:

203.0.113.25/32

Don't use:

0.0.0.0/0

for SSH in a real environment.

Step 6 — Add EC2 output

Open:

outputs.tf

Add:

output "ec2_instance_id" {
  description = "ID of the application EC2 instance"
  value       = aws_instance.application.id
}

output "ec2_public_ip" {
  description = "Public IP address of the application EC2 instance"
  value       = aws_instance.application.public_ip
}

output "ec2_public_dns" {
  description = "Public DNS name of the application EC2 instance"
  value       = aws_instance.application.public_dns
}

This allows Terraform to show you the instance details after deployment.

Step 7 — Check your root main.tf

Your root configuration should be similar to:

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

resource "aws_instance" "application" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  subnet_id = module.vpc.public_subnet_ids[0]

  vpc_security_group_ids = [
    module.vpc.application_security_group_id
  ]

  associate_public_ip_address = true

  tags = {
    Name        = "dev-application-server"
    Environment = "dev"
    Project     = "terraform-training"
  }
}

Replace:

YOUR_PUBLIC_IP/32

with your actual trusted public IP.

Step 8 — Format Terraform

Run:

terraform fmt

Expected:

main.tf
outputs.tf

or similar files being formatted.

Step 9 — Validate

Run:

terraform validate

Expected:

Success! The configuration is valid.

If you get an error about:

application_security_group_id

check that this output exists in:

modules/vpc/outputs.tf
Step 10 — Initialize Terraform

Run:

terraform init

If you already initialized the project, Terraform may simply report that initialization is complete.

Step 11 — Create the Terraform plan

Run:

terraform plan

Terraform should show the EC2 resource:

+ resource "aws_instance" "application" {
    + ami           = ...
    + instance_type = "t3.micro"
    + subnet_id     = ...
    + public_ip     = (known after apply)
    + tags          = {
        + "Environment" = "dev"
        + "Name"        = "dev-application-server"
        + "Project"     = "terraform-training"
      }
  }

You should also see the networking resources from your previous tasks if they have not already been created.

Step 12 — Apply the configuration

Your task says:

EC2 successfully launched through Terraform.

Therefore, this task requires an actual deployment.

Run:

terraform apply

Terraform will display:

Do you want to perform these actions?

Enter:

yes

Terraform will create the EC2 instance.

Step 13 — Check the output

After successful deployment, you should see something similar to:

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

And your outputs:

ec2_instance_id = "i-xxxxxxxxxxxxxxxxx"

ec2_public_ip = "13.xx.xx.xx"

ec2_public_dns = "ec2-13-xx-xx-xx.ap-south-1.compute.amazonaws.com"

The actual values will be different.

Step 14 — Verify in AWS Console

Go to:

AWS Console → EC2 → Instances

You should see:

Name: dev-application-server
Instance type: t3.micro
State: Running
Subnet: Public subnet

Check:

AMI

Amazon Linux 2023.

Instance type
t3.micro
VPC

Your:

10.0.0.0/16
Subnet

The first public subnet:

10.0.1.0/24
Availability Zone

Likely:

ap-south-1a

because we used:

module.vpc.public_subnet_ids[0]
Security Group

Your application security group.

Public IPv4

A public IP should be assigned.

Step 15 — Understand why this is a public EC2

The traffic path is:

User
 │
 │ Internet
 ▼
Internet Gateway
 │
 ▼
Public Route Table
 │
 │ 0.0.0.0/0 → IGW
 ▼
Public Subnet
 │
 ▼
Security Group
 │
 ▼
EC2

The EC2 is placed into:

10.0.1.0/24

and that subnet has a route:

0.0.0.0/0 → Internet Gateway

The EC2 also gets a public IPv4 address.

Therefore, it can communicate with the internet, subject to the security group's rules.

Step 16 — Verify Terraform state

Run:

terraform state list

You should see your EC2 resource:

aws_instance.application

Because the EC2 was created by Terraform, Terraform tracks it in:

terraform.tfstate

You can also run:

terraform show

to inspect the deployed infrastructure.

Final architecture

Your Day 27 architecture is now becoming:

                         INTERNET
                            │
                            ▼
                    ┌───────────────┐
                    │ Internet GW   │
                    └───────┬───────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │ Public Route Table  │
                 │ 0.0.0.0/0 → IGW    │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │ Public Subnet       │
                 │ 10.0.1.0/24        │
                 │                     │
                 │ ┌─────────────────┐ │
                 │ │ Security Group  │ │
                 │ │ 80 / 443 / SSH  │ │
                 │ └────────┬────────┘ │
                 │          │           │
                 │          ▼           │
                 │   ┌──────────────┐  │
                 │   │ EC2          │  │
                 │   │ t3.micro     │  │
                 │   │ Amazon Linux │  │
                 │   └──────────────┘  │
                 └─────────────────────┘
TASK 10 checklist
Requirement	Status
AMI configured	✅
Instance type configured	✅ t3.micro
Public subnet configured	✅
Security group configured	✅
Tags configured	✅
Public IP enabled	✅
Terraform plan	✅
Terraform apply	✅
EC2 launched	✅
Terraform state	✅
Important cost step

This is a real AWS EC2 deployment, so after completing the training, clean up resources you no longer need:

terraform destroy

Then confirm:

yes