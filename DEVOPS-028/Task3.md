TASK 3 — EC2 AMI & Instance Configuration

For this task, the main goal is to understand how an AMI becomes an EC2 instance and then configure the AMI in Terraform without hard-coding a region-specific AMI ID.

1. AMI → EC2 Relationship

The basic relationship is:

AMI
 │
 │ EC2 uses AMI as the source image
 ▼
EC2 Instance
 │
 ├── Operating System
 ├── Root Volume
 ├── Network Interface
 └── Security Group

Think of an AMI as a template/image used to create an EC2 instance.

For example:

Amazon Linux AMI
       │
       ▼
   EC2 Instance
       │
       ├── Amazon Linux
       ├── t3.micro
       ├── EBS Root Volume
       ├── Network Interface
       └── Security Group
2. What Is an AMI?

AMI = Amazon Machine Image

An AMI contains the information required to launch an EC2 instance.

It can include:

Operating system
Pre-installed software
Application configuration
Root volume configuration
Permissions
Architecture information

Examples:

Amazon Linux
Ubuntu
Red Hat Enterprise Linux
Windows Server

For this training, Amazon Linux is a suitable choice.

3. Why Is an AMI Required?

When you create an EC2 instance, AWS needs to know:

"What operating system and base software should this server start with?"

The AMI provides that starting point.

For example:

Amazon Linux AMI
       │
       ▼
EC2 Instance
       │
       ▼
Linux Operating System

Without an AMI, Terraform does not know which image should be used to create the instance.

4. AMI Is Region-Specific

This is very important.

An AMI ID can be different in different AWS regions.

For example:

ap-south-1
    │
    └── AMI A

us-east-1
    │
    └── AMI B

Therefore, you should not blindly copy an AMI ID from an internet tutorial.

For your training environment, if your provider is:

provider "aws" {
  region = "ap-south-1"
}

you should select an AMI available in:

ap-south-1
5. Best Terraform Approach — Use an AMI Data Source

Instead of hard-coding:

ami = "ami-123456789"

you can ask AWS for a suitable AMI.

Create:

data.tf

with:

data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }

  filter {
    name   = "state"
    values = ["available"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }
}

This allows Terraform to find a suitable Amazon Linux 2023 AMI in the configured AWS region.

6. Understand the Data Source

This part:

data "aws_ami" "amazon_linux"

means:

Ask AWS for information about an existing AMI.

It does not create an AMI.

Then:

most_recent = true

means Terraform should select the most recent matching AMI.

And:

owners = ["amazon"]

restricts the search to AMIs owned by Amazon.

7. Use the AMI in Your Compute Module

Your compute module already has:

variable "ami" {
  description = "AMI ID used to launch the EC2 instance"
  type        = string
}

So your root main.tf can pass the discovered AMI:

module "compute" {
  source = "./modules/compute"

  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  subnet_id = "subnet-xxxxxxxxxxxxxxxxx"

  security_group_ids = [
    "sg-xxxxxxxxxxxxxxxxx"
  ]

  tags = {
    Name        = "day28-ec2"
    Environment = "training"
  }

  user_data = <<-EOF
    #!/bin/bash
    echo "Hello from Terraform" > /tmp/day28.txt
  EOF
}

Now the architecture is:

AWS Region
    │
    ▼
AMI Data Source
    │
    │ Finds suitable Amazon Linux AMI
    ▼
AMI ID
    │
    ▼
Compute Module
    │
    ▼
EC2 Instance
8. Instance Type

The instance type determines the compute resources available to the EC2 instance.

For example:

t3.micro

is commonly suitable for training and small workloads, subject to your AWS account's current pricing/free-tier eligibility.

It determines resources such as:

vCPU
Memory
Network performance
Instance capabilities

Terraform:

instance_type = "t3.micro"

For a reusable module, keep it as a variable:

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}
9. Root Volume

The EC2 instance needs storage for the operating system and files.

This is normally an EBS volume.

Basic architecture:

EC2
 │
 └── Root EBS Volume
       │
       ├── Operating System
       ├── Applications
       └── Files

You can configure the root volume in Terraform:

root_block_device {
  volume_type = "gp3"
  volume_size = 8
}

For example, your module can contain:

resource "aws_instance" "this" {
  ami                    = var.ami
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids
  user_data              = var.user_data

  root_block_device {
    volume_type = "gp3"
    volume_size = var.root_volume_size
  }

  tags = var.tags
}

Then add:

variable "root_volume_size" {
  description = "Root EBS volume size in GB"
  type        = number
  default     = 8
}
10. Network Interface

An EC2 instance needs network connectivity.

AWS provides a network interface, commonly called an ENI.

Basic architecture:

EC2
 │
 ▼
Network Interface
 │
 ├── Private IP
 ├── Subnet
 └── Security Group

When you specify:

subnet_id = var.subnet_id

AWS creates/attaches the primary network interface in that subnet for the instance.

11. Security Group

A Security Group controls allowed network traffic to and from the EC2 instance.

Example:

Internet
   │
   │ HTTP : 80
   ▼
Security Group
   │
   ▼
EC2

For example, a web server might allow:

HTTP  → TCP 80
HTTPS → TCP 443

SSH should generally be restricted to a trusted source rather than opening port 22 to the entire internet.

Your EC2 resource receives the security group IDs through:

vpc_security_group_ids = var.security_group_ids
12. Complete EC2 Configuration

Your compute module can now look like this.

modules/compute/main.tf
resource "aws_instance" "this" {
  ami                    = var.ami
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids
  user_data              = var.user_data

  root_block_device {
    volume_type = "gp3"
    volume_size = var.root_volume_size
  }

  tags = var.tags
}
modules/compute/variables.tf
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
  description = "Subnet where the EC2 instance will be launched"
  type        = string
}

variable "security_group_ids" {
  description = "Security group IDs attached to the EC2 instance"
  type        = list(string)
}

variable "root_volume_size" {
  description = "Root EBS volume size in GB"
  type        = number
  default     = 8
}

variable "user_data" {
  description = "EC2 startup script"
  type        = string
  default     = null
}

variable "tags" {
  description = "EC2 instance tags"
  type        = map(string)
  default     = {}
}
13. Root data.tf

At the root level:

data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }

  filter {
    name   = "state"
    values = ["available"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }
}
14. Root main.tf
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

  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  subnet_id = "subnet-xxxxxxxxxxxxxxxxx"

  security_group_ids = [
    "sg-xxxxxxxxxxxxxxxxx"
  ]

  root_volume_size = 8

  tags = {
    Name        = "day28-ec2"
    Environment = "training"
  }

  user_data = <<-EOF
    #!/bin/bash
    echo "Day 28 EC2 Training" > /tmp/day28.txt
  EOF
}

Replace the example subnet and security-group IDs with resources/IDs from your actual training environment.

15. Validate the Configuration

From your Terraform directory:

terraform fmt

Then:

terraform init

Then:

terraform validate

Expected:

Success! The configuration is valid.

Now run:

terraform plan

Terraform should first find the AMI and then show the EC2 instance that would be created.

16. Verify Which AMI Terraform Selected

You can temporarily add this to root outputs.tf:

output "selected_ami_id" {
  description = "AMI selected by the data source"
  value       = data.aws_ami.amazon_linux.id
}

output "selected_ami_name" {
  description = "AMI name selected by the data source"
  value       = data.aws_ami.amazon_linux.name
}

Then:

terraform plan

or, after apply:

terraform output

You can see which AMI Terraform selected.

17. Important Learning Point
Bad approach
ami = "ami-0123456789abcdef"

This can become a problem because the AMI may not exist in another region.

Better approach
ami = data.aws_ami.amazon_linux.id

Terraform discovers an appropriate AMI for the configured AWS region.

18. Final Architecture
                 AWS Region
                ap-south-1
                     │
                     ▼
              AMI Data Source
                     │
                     ▼
              Amazon Linux AMI
                     │
                     ▼
              Compute Module
                     │
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼
       EC2 Type    Subnet   Security Group
       t3.micro       │          │
          │           │          │
          └───────────┼──────────┘
                      ▼
                EC2 Instance
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Root EBS    Network     Application
       Volume      Interface
Task 3 Deliverable Checklist
[✓] Understand AMI
[✓] Understand AMI → EC2 relationship
[✓] Understand region-specific AMIs
[✓] Understand instance type
[✓] Understand root volume
[✓] Understand network interface
[✓] Understand security groups
[✓] Configure AMI through Terraform
[✓] Use reusable compute module
[✓] Avoid blindly hard-coding an AMI ID
[✓] Run terraform fmt
[✓] Run terraform init
[✓] Run terraform validate
[✓] Run terraform plan

Key takeaway:

AMI = OS/server image
       ↓
EC2 = Running virtual server
       ↓
Root Volume + Network Interface + Security Group

And for reusable Terraform:

AWS Region
    ↓
AMI Data Source
    ↓
AMI ID
    ↓
Reusable Compute Module
    ↓
EC2 Instance