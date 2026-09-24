TASK 6 — Auto Scaling Group

Time: 60 Minutes

The goal of this task is to create an AWS Auto Scaling Group (ASG) using the Launch Template from Task 5.

The important idea is:

Launch Template
      │
      │ tells ASG HOW to create EC2
      ▼
Auto Scaling Group
      │
      ├── EC2-1
      ├── EC2-2
      └── EC2-3

The Launch Template defines the EC2 configuration, while the Auto Scaling Group controls how many EC2 instances should run.

1. What is an Auto Scaling Group?

An Auto Scaling Group is an AWS service that automatically manages a group of EC2 instances.

For example:

Minimum = 1
Desired = 2
Maximum = 3

This means:

Setting	Meaning
min_size	Minimum number of EC2 instances
desired_capacity	Number of EC2 instances we want normally
max_size	Maximum number of EC2 instances

So normally:

             ASG
              │
       Desired = 2
          /       \
         ▼         ▼
       EC2-1     EC2-2

If one EC2 fails:

EC2-1 ❌
   │
   ▼
ASG detects failure
   │
   ▼
Creates replacement EC2
   │
   ▼
EC2-3 ✅

The ASG tries to return to the desired capacity.

2. Why Do We Need an ASG?

Without Auto Scaling:

Terraform
   │
   ▼
EC2

If the EC2 fails, you may need to manually create another instance.

With Auto Scaling:

Terraform
   │
   ▼
Launch Template
   │
   ▼
Auto Scaling Group
   │
   ├── EC2
   ├── EC2
   └── EC2

AWS can automatically maintain the required number of instances.

3. Understand the Three Important Values
min_size

Minimum number of instances that the ASG should maintain.

Example:

min_size = 1

The ASG should not normally scale below 1 instance.

desired_capacity

The number of instances you want running initially/normally.

desired_capacity = 2

The ASG attempts to maintain 2 instances.

max_size

Maximum number of instances the ASG can create.

max_size = 3

The ASG cannot scale beyond 3 instances.

4. Training Configuration

For this exercise:

Minimum  = 1
Desired  = 2
Maximum  = 3

Architecture:

                  Auto Scaling Group
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
            EC2-1                 EC2-2
          AZ-1/Subnet            AZ-2/Subnet
Cost awareness

For a training environment, 2 running EC2 instances cost more than 1, so you should avoid leaving resources running unnecessarily.

For a low-cost practice setup, you can temporarily use:

min_size         = 1
desired_capacity = 1
max_size         = 2

For demonstrating multi-instance behavior, use:

min_size         = 1
desired_capacity = 2
max_size         = 3

After testing, run:

terraform destroy

if the environment is no longer needed.

5. Project Structure

Your Day 28 Terraform project can now look like:

day28/
└── terraform/
    │
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    ├── user-data.sh
    │
    └── modules/
        └── compute/
            ├── main.tf
            ├── variables.tf
            └── outputs.tf

We will modify the compute module to include the ASG.

6. Step 1 — Understand the ASG Components

An ASG needs:

Auto Scaling Group
       │
       ├── Launch Template
       │
       ├── Subnet/AZ
       │
       ├── Minimum capacity
       │
       ├── Desired capacity
       │
       └── Maximum capacity

Terraform resource:

resource "aws_autoscaling_group" "this" {
}
7. Step 2 — Add ASG Variables

Open:

modules/compute/variables.tf

Add:

variable "subnet_ids" {
  description = "Subnet IDs where the Auto Scaling Group can launch EC2 instances"
  type        = list(string)
}

variable "min_size" {
  description = "Minimum number of EC2 instances"
  type        = number
  default     = 1
}

variable "desired_capacity" {
  description = "Desired number of EC2 instances"
  type        = number
  default     = 2
}

variable "max_size" {
  description = "Maximum number of EC2 instances"
  type        = number
  default     = 3
}
8. Step 3 — Create the Auto Scaling Group

Open:

modules/compute/main.tf

Keep your Launch Template:

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

Now add the ASG below it:

resource "aws_autoscaling_group" "this" {
  name = "day28-compute-asg"

  min_size         = var.min_size
  desired_capacity = var.desired_capacity
  max_size         = var.max_size

  vpc_zone_identifier = var.subnet_ids

  launch_template {
    id      = aws_launch_template.this.id
    version = "$Latest"
  }

  health_check_type = "EC2"

  tag {
    key                 = "Name"
    value               = "day28-compute"
    propagate_at_launch = true
  }

  tag {
    key                 = "Environment"
    value               = "Training"
    propagate_at_launch = true
  }
}
9. Understand vpc_zone_identifier

This is very important.

vpc_zone_identifier = var.subnet_ids

It tells the ASG which subnets it can use.

For example:

VPC
 │
 ├── Subnet-A
 │     └── AZ-1
 │
 └── Subnet-B
       └── AZ-2

Terraform:

subnet_ids = [
  "subnet-AAAA",
  "subnet-BBBB"
]

The ASG can launch instances across these subnets.

10. Multi-AZ Architecture

The desired architecture is:

                    Auto Scaling Group
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
        Availability Zone A        Availability Zone B
             │                           │
             ▼                           ▼
          Subnet-A                    Subnet-B
             │                           │
             ▼                           ▼
           EC2-1                       EC2-2

This is better than putting all instances into one subnet/AZ because it improves availability.

11. Step 4 — Root main.tf

Your root main.tf can look like:

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
    name   = "state"
    values = ["available"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }
}

module "compute" {
  source = "./modules/compute"

  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  security_group_ids = [
    "sg-xxxxxxxxxxxxxxxxx"
  ]

  subnet_ids = [
    "subnet-xxxxxxxxxxxxxxxxx",
    "subnet-yyyyyyyyyyyyyyyyy"
  ]

  min_size         = 1
  desired_capacity = 2
  max_size         = 3

  root_volume_size = 8

  tags = {
    Name        = "day28-compute"
    Environment = "Training"
  }

  user_data = file("${path.module}/user-data.sh")
}

Replace:

sg-xxxxxxxxxxxxxxxxx

and:

subnet-xxxxxxxxxxxxxxxxx
subnet-yyyyyyyyyyyyyyyyy

with your actual training Security Group and subnet IDs.

12. Important — Subnets Must Belong to the Same VPC

Suppose your Security Group belongs to:

VPC-A

Your subnets must also belong to:

VPC-A

Don't mix:

Security Group → VPC-A

Subnet-1 → VPC-A
Subnet-2 → VPC-B ❌

Correct:

Security Group → VPC-A
                    │
              ┌─────┴─────┐
              ▼           ▼
           Subnet-1    Subnet-2
           VPC-A       VPC-A
13. Step 5 — Add Outputs

Open:

modules/compute/outputs.tf

Add:

output "autoscaling_group_id" {
  description = "ID of the Auto Scaling Group"
  value       = aws_autoscaling_group.this.id
}

output "autoscaling_group_name" {
  description = "Name of the Auto Scaling Group"
  value       = aws_autoscaling_group.this.name
}

output "autoscaling_group_min_size" {
  description = "Minimum ASG capacity"
  value       = aws_autoscaling_group.this.min_size
}

output "autoscaling_group_desired_capacity" {
  description = "Desired ASG capacity"
  value       = aws_autoscaling_group.this.desired_capacity
}

output "autoscaling_group_max_size" {
  description = "Maximum ASG capacity"
  value       = aws_autoscaling_group.this.max_size
}
14. Step 6 — Root Outputs

In root outputs.tf:

output "autoscaling_group_name" {
  value = module.compute.autoscaling_group_name
}

output "autoscaling_group_min_size" {
  value = module.compute.autoscaling_group_min_size
}

output "autoscaling_group_desired_capacity" {
  value = module.compute.autoscaling_group_desired_capacity
}

output "autoscaling_group_max_size" {
  value = module.compute.autoscaling_group_max_size
}

output "launch_template_id" {
  value = module.compute.launch_template_id
}
15. Step 7 — Format Terraform

From your Codespace:

cd /workspaces/terraform-lab/day28/terraform

Run:

terraform fmt

Expected:

modules/compute/main.tf
modules/compute/variables.tf
modules/compute/outputs.tf
main.tf
outputs.tf
16. Step 8 — Initialize Terraform

Run:

terraform init

Terraform will initialize the AWS provider and module.

17. Step 9 — Validate

Run:

terraform validate

Expected:

Success! The configuration is valid.

If there is an error, don't run apply yet. Read the error and correct the configuration.

18. Step 10 — Review the Plan

Run:

terraform plan

You should see Terraform planning to create resources such as:

aws_launch_template.this
aws_autoscaling_group.this

Conceptually:

Terraform Plan
      │
      ├── Launch Template
      │
      └── Auto Scaling Group
19. Step 11 — Apply

If the plan is correct:

terraform apply

Terraform asks:

Do you want to perform these actions?

Enter:

yes

AWS will then create the Launch Template and Auto Scaling Group.

20. Step 12 — Check Outputs

Run:

terraform output

You should see something similar to:

autoscaling_group_desired_capacity = 2
autoscaling_group_max_size = 3
autoscaling_group_min_size = 1
autoscaling_group_name = "day28-compute-asg"
launch_template_id = "lt-xxxxxxxxxxxxxxxxx"
21. What Happens Inside AWS?

Terraform creates:

Launch Template
       │
       │
       ▼
Auto Scaling Group
       │
       ├───────────────┐
       ▼               ▼
     EC2-1           EC2-2

Because:

desired_capacity = 2

the ASG attempts to maintain two instances.

22. Automatic Replacement

Suppose:

EC2-1 ✅
EC2-2 ✅

One instance becomes unhealthy or terminates:

EC2-1 ❌
EC2-2 ✅

ASG sees:

Current = 1
Desired = 2

Therefore:

ASG
 │
 └── Create replacement
          │
          ▼
        EC2-3

Final:

EC2-2 ✅
EC2-3 ✅

This is one of the major benefits of an Auto Scaling Group.

23. Understand the Capacity Values

Imagine:

min_size         = 1
desired_capacity = 2
max_size         = 3
Normal condition
EC2-1
EC2-2

Current = 2
Desired = 2

Everything is normal.

One instance fails
EC2-1 ❌
EC2-2 ✅

Current = 1
Desired = 2

ASG creates another:

EC2-2 ✅
EC2-3 ✅
Scaling up

If a scaling policy later determines that more capacity is needed:

2 → 3

The ASG can create:

EC2-1
EC2-2
EC2-3

but cannot exceed:

max_size = 3
24. Important Difference: Launch Template vs ASG

This is an important interview question.

Launch Template

Answers:

How should an EC2 instance be created?

It contains:

AMI
Instance Type
Security Groups
User Data
Storage
Tags
Auto Scaling Group

Answers:

How many EC2 instances should be running and where can they be launched?

It contains:

Minimum
Desired
Maximum
Subnets
Launch Template
Health Check

Therefore:

Launch Template
      │
      │ HOW
      ▼
Auto Scaling Group
      │
      │ HOW MANY + WHERE
      ▼
EC2 Instances
25. Final Architecture

After completing Task 6:

                         User
                           │
                           ▼
                    Auto Scaling Group
                           │
                  Launch Template
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
       Availability Zone A        Availability Zone B
             │                           │
             ▼                           ▼
          Subnet-A                    Subnet-B
             │                           │
             ▼                           ▼
           EC2-1                       EC2-2

Later, in the next stage, the architecture will become:

                         User
                           │
                           ▼
                    Application Load
                       Balancer
                           │
                           ▼
                      Target Group
                           │
                 ┌─────────┴─────────┐
                 ▼                   ▼
               EC2-1               EC2-2
                 ▲                   ▲
                 └──── Auto Scaling ─┘
                         Group
26. Task 6 Checklist

Before marking the task complete, verify:

[ ] Launch Template exists
[ ] Auto Scaling Group created
[ ] min_size configured
[ ] desired_capacity configured
[ ] max_size configured
[ ] Launch Template connected to ASG
[ ] Multiple subnets configured
[ ] Subnets belong to the same VPC
[ ] EC2 instances launched
[ ] User Data configured
[ ] Security Group attached
[ ] terraform fmt completed
[ ] terraform validate passed
[ ] terraform plan reviewed
[ ] terraform apply completed
[ ] Terraform outputs checked
Deliverable

Your final Terraform architecture for Task 6 should be:

Terraform
    │
    ▼
Compute Module
    │
    ├── Launch Template
    │      ├── AMI
    │      ├── Instance Type
    │      ├── Security Group
    │      ├── User Data
    │      └── EBS
    │
    └── Auto Scaling Group
           ├── min = 1
           ├── desired = 2
           ├── max = 3
           ├── Subnet-A
           └── Subnet-B
                    │
              ┌─────┴─────┐
              ▼           ▼
            EC2-1       EC2-2

Key concept to remember:
Launch Template = EC2 blueprint.
Auto Scaling Group = manages the number and placement of EC2 instances.