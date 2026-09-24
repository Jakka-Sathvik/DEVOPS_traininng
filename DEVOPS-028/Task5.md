TASK 5 — Launch Template

The goal of this task is to replace the direct aws_instance approach with an AWS Launch Template that acts as a reusable blueprint for EC2 instances.

The important architecture is:

                    Launch Template
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
         AMI         Instance Type    User Data
          │               │               │
          └───────────────┼───────────────┘
                          ▼
                 Auto Scaling Group
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
            EC2-1       EC2-2       EC2-3
1. What Is a Launch Template?

An AWS Launch Template is a blueprint that defines how EC2 instances should be launched.

Instead of configuring every EC2 instance separately, you define the configuration once.

For example:

Launch Template
│
├── AMI
├── Instance Type
├── Security Group
├── User Data
├── Storage
└── Instance Configuration

The Auto Scaling Group then uses this blueprint to create EC2 instances.

2. Launch Template vs EC2

This distinction is important.

EC2
EC2
↓
Actual running virtual server
Launch Template
Launch Template
↓
Blueprint for creating EC2 servers

For example:

Launch Template
       │
       ├── Amazon Linux
       ├── t3.micro
       ├── Security Group
       └── User Data
              │
              ▼
          EC2 Instance

The Launch Template itself is not the running server.

3. Why Use a Launch Template?

Without a Launch Template:

EC2-1 → manually configured
EC2-2 → manually configured
EC2-3 → manually configured

This can lead to configuration differences.

With a Launch Template:

             Launch Template
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      EC2-1       EC2-2       EC2-3

All instances can be launched using the same configuration.

This is especially useful with an Auto Scaling Group.

4. Create the Launch Template

Your project can now look like:

day28/
└── terraform/
    ├── main.tf
    ├── data.tf
    ├── outputs.tf
    ├── user-data.sh
    │
    └── modules/
        └── compute/
            ├── main.tf
            ├── variables.tf
            └── outputs.tf

For this task, we'll change the compute module from aws_instance to aws_launch_template.

5. Create modules/compute/main.tf

Use:

resource "aws_launch_template" "this" {
  name_prefix   = "day28-compute-"
  image_id      = var.ami
  instance_type = var.instance_type

  user_data = var.user_data != null ? base64encode(var.user_data) : null

  vpc_security_group_ids = var.security_group_ids

  block_device_mappings {
    device_name = "/dev/xvda"

    ebs {
      volume_size           = var.root_volume_size
      volume_type           = "gp3"
      delete_on_termination = true
    }
  }

  tag_specifications {
    resource_type = "instance"

    tags = var.tags
  }
}

This creates the Launch Template.

6. Understand Each Configuration
AMI
image_id = var.ami

This tells the Launch Template which operating system image to use.

Example:

Amazon Linux 2023
       ↓
Launch Template
       ↓
EC2
Instance Type
instance_type = var.instance_type

Example:

t3.micro

This controls the compute capacity of the launched instances.

Security Group
vpc_security_group_ids = var.security_group_ids

This associates the configured Security Group with instances launched from the template.

User Data
user_data = var.user_data != null ? base64encode(var.user_data) : null

This passes the startup script to the EC2 instances.

Your previous script:

user-data.sh

can install NGINX and create your training application automatically.

7. Why base64encode()?

For the Terraform AWS Launch Template resource, User Data is supplied as Base64-encoded data.

Therefore:

base64encode(var.user_data)

converts:

#!/bin/bash
dnf install -y nginx

into Base64 data before sending it to AWS.

This allows AWS to correctly process the User Data.

8. Root Volume

The Launch Template can define the root EBS volume:

block_device_mappings {
  device_name = "/dev/xvda"

  ebs {
    volume_size           = var.root_volume_size
    volume_type           = "gp3"
    delete_on_termination = true
  }
}

Here:

Volume Type = gp3
Volume Size = variable

and:

delete_on_termination = true

means the root volume is deleted when the EC2 instance is terminated.

9. Tags

The Launch Template needs to specify that the tags apply to EC2 instances:

tag_specifications {
  resource_type = "instance"

  tags = var.tags
}

Example:

tags = {
  Name        = "day28-web-server"
  Environment = "Training"
}

The resulting EC2 instance receives:

Name = day28-web-server
Environment = Training
10. Create modules/compute/variables.tf

Use:

variable "ami" {
  description = "AMI ID used by the Launch Template"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "security_group_ids" {
  description = "Security Group IDs for EC2 instances"
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
  description = "Tags applied to EC2 instances"
  type        = map(string)
  default     = {}
}

Notice that we don't need subnet_id in the Launch Template for this task.

The Auto Scaling Group will later decide which subnets the instances should be launched into.

11. Create modules/compute/outputs.tf

The Launch Template has an ID that can be useful to the Auto Scaling Group.

output "launch_template_id" {
  description = "ID of the Launch Template"
  value       = aws_launch_template.this.id
}

output "launch_template_name" {
  description = "Name of the Launch Template"
  value       = aws_launch_template.this.name
}

You can also output the latest version:

output "launch_template_latest_version" {
  description = "Latest version of the Launch Template"
  value       = aws_launch_template.this.latest_version
}
12. Find the AMI Automatically

Continue using the AMI data source from Task 3.

Create or keep:

data.tf
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

This avoids blindly hard-coding an AMI ID.

The AMI is selected within the AWS region configured in your provider.

13. Create user-data.sh

Continue using the User Data from Task 4:

#!/bin/bash

dnf update -y

dnf install -y nginx

cat > /usr/share/nginx/html/index.html <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>DevOps Training Application</title>
</head>
<body>
    <h1>DevOps Training Application</h1>
    <p>Server: EC2</p>
    <p>Environment: Training</p>
    <p>Status: Application is running</p>
</body>
</html>
EOF

systemctl enable nginx
systemctl start nginx

Every EC2 instance launched from the Launch Template can automatically execute this configuration during initialization.

14. Root main.tf

Your root module can now call the compute module:

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

  security_group_ids = [
    "sg-xxxxxxxxxxxxxxxxx"
  ]

  root_volume_size = 8

  tags = {
    Name        = "day28-compute"
    Environment = "Training"
  }

  user_data = file("${path.module}/user-data.sh")
}

Replace:

sg-xxxxxxxxxxxxxxxxx

with the Security Group from your training environment.

15. Important Difference From Task 2

In Task 2, you created:

resource "aws_instance" "this"

That creates an actual EC2 instance.

Now you are creating:

resource "aws_launch_template" "this"

That creates a blueprint.

Task 2
Terraform
   │
   ▼
aws_instance
   │
   ▼
EC2
Task 5
Terraform
   │
   ▼
aws_launch_template
   │
   ▼
Blueprint

Later:

Launch Template
       │
       ▼
Auto Scaling Group
       │
       ├── EC2
       ├── EC2
       └── EC2
16. Validate the Terraform

From your project directory:

cd /workspaces/terraform-lab/day28/terraform

Run:

terraform fmt

Then:

terraform init

Then:

terraform validate

Expected:

Success! The configuration is valid.
17. Run Terraform Plan

Run:

terraform plan

You should see something similar to:

Terraform will perform the following actions:

  # module.compute.aws_launch_template.this will be created
  + resource "aws_launch_template" "this" {
      + image_id      = ...
      + instance_type = "t3.micro"
      ...
    }

Plan: 1 to add, 0 to change, 0 to destroy.

At this point, you have created the Launch Template, not the EC2 instances.

18. Apply

If the plan is correct:

terraform apply

Enter:

yes

Terraform creates:

AWS
 │
 └── Launch Template
       │
       ├── AMI
       ├── t3.micro
       ├── Security Group
       ├── User Data
       └── EBS configuration
19. Verify the Launch Template

Run:

terraform output

You should get something similar to:

launch_template_id = "lt-0123456789abcdef"
launch_template_name = "day28-compute-..."
launch_template_latest_version = 1

You can also check it in the AWS EC2 console:

AWS Console
   ↓
EC2
   ↓
Launch Templates
   ↓
day28-compute-...
20. Launch Template Versioning

Launch Templates support versions.

For example:

Launch Template
      │
      ├── Version 1
      │      └── t3.micro + old User Data
      │
      ├── Version 2
      │      └── t3.micro + updated User Data
      │
      └── Version 3
             └── t3.small + updated configuration

This becomes very useful when an Auto Scaling Group needs to launch instances using a particular configuration version.

21. Launch Template → ASG Flow

This task stops at the Launch Template, but the next logical architecture is:

                  Launch Template
                         │
                         │
                         ▼
                Auto Scaling Group
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
            EC2-1      EC2-2      EC2-3

The Launch Template answers:

"How should each EC2 instance be configured?"

The Auto Scaling Group answers:

"How many EC2 instances should be running, and where should they be launched?"

22. Complete Architecture
                         Terraform
                            │
                            ▼
                    Compute Module
                            │
                            ▼
                  Launch Template
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
         AMI          Instance Type       User Data
          │                 │                 │
          └─────────────────┼─────────────────┘
                            │
                            ▼
                  Auto Scaling Group
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
           EC2-1          EC2-2          EC2-3
23. Task 5 Deliverable Checklist
[✓] Understand Launch Template
[✓] Understand AMI configuration
[✓] Understand instance type
[✓] Understand Security Group configuration
[✓] Understand User Data
[✓] Understand root volume configuration
[✓] Create aws_launch_template
[✓] Make Launch Template reusable
[✓] Use AMI data source
[✓] Add tags
[✓] Add User Data
[✓] Create Launch Template outputs
[✓] Run terraform fmt
[✓] Run terraform init
[✓] Run terraform validate
[✓] Run terraform plan
[✓] Apply Terraform
[✓] Verify Launch Template
Key takeaway
Launch Template
      │
      ├── AMI
      ├── Instance Type
      ├── Security Group
      ├── User Data
      ├── Storage
      └── Tags
             │
             ▼
     Blueprint for EC2
             │
             ▼
    Auto Scaling Group
             │
       ┌─────┼─────┐
       ▼     ▼     ▼
      EC2   EC2   EC2

Remember: The Launch Template defines the configuration of an EC2 instance, while the Auto Scaling Group uses that configuration to manage multiple EC2 instances.