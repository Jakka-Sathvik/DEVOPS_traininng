TASK 11 — Security Group Architecture

Time: 45 Minutes

The goal of this task is to create separate Security Groups for the ALB and EC2 instances and understand how traffic should flow between them.

1. Understand the Architecture

The recommended architecture is:

                         INTERNET
                            │
                            │ HTTP :80
                            ▼
                  ┌──────────────────┐
                  │       ALB        │
                  │                  │
                  │    ALB-SG        │
                  └────────┬─────────┘
                           │
                           │ HTTP :80
                           │
                           ▼
                  ┌──────────────────┐
                  │      EC2         │
                  │                  │
                  │     EC2-SG       │
                  └──────────────────┘

There are two separate Security Groups:

ALB-SG
   │
   └── Controls traffic entering the ALB


EC2-SG
   │
   └── Controls traffic entering EC2

The important security rule is:

EC2 should accept application traffic from the ALB Security Group, not directly from the entire Internet.

2. Why Use Two Security Groups?

Avoid this architecture:

Internet
   │
   ├──────────────► ALB
   │
   └──────────────► EC2

Because users could bypass the ALB and access EC2 directly.

Instead:

Internet
   │
   ▼
   ALB
   │
   ▼
   EC2

This gives us:

Controlled entry point
Better security
Easier management
ALB-based health checks
Centralized public access
Backend isolation
3. ALB Security Group

Create an ALB Security Group.

The ALB is public, so it can accept HTTP traffic from the Internet.

resource "aws_security_group" "alb" {
  name        = "day28-alb-sg"
  description = "Security group for Application Load Balancer"
  vpc_id      = var.vpc_id

  ingress {
    description = "HTTP from Internet"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "Allow outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "day28-alb-sg"
    Environment = "Training"
  }
}
Meaning
Internet
   │
   │ TCP 80
   ▼
ALB-SG

The ALB accepts:

HTTP
Port 80
Source: Internet
4. EC2 Security Group

Now create a different Security Group for the EC2 instances.

The important part is:

security_groups = [aws_security_group.alb.id]

or, for an ingress rule, reference the ALB Security Group as the source.

Example:

resource "aws_security_group" "ec2" {
  name        = "day28-ec2-sg"
  description = "Security group for EC2 application servers"
  vpc_id      = var.vpc_id

  ingress {
    description     = "HTTP from ALB"
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  egress {
    description = "Allow outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "day28-ec2-sg"
    Environment = "Training"
  }
}

The important rule is:

security_groups = [aws_security_group.alb.id]

This means:

Allow HTTP traffic to EC2 only when the source is associated with the ALB Security Group.

5. Understand the Difference
ALB Security Group
Source:
0.0.0.0/0

Port:
80

Protocol:
TCP

Meaning:

Internet ──HTTP──► ALB
EC2 Security Group
Source:
ALB Security Group

Port:
80

Protocol:
TCP

Meaning:

ALB ──HTTP──► EC2

Not:

Internet ──HTTP──► EC2
6. Complete Traffic Flow

The final traffic flow is:

                         INTERNET
                            │
                            │
                            │ HTTP :80
                            ▼
                 ┌─────────────────────┐
                 │         ALB         │
                 │                     │
                 │       ALB-SG        │
                 │                     │
                 │ Allow TCP 80        │
                 │ from 0.0.0.0/0      │
                 └──────────┬──────────┘
                            │
                            │ HTTP :80
                            │
                            ▼
                 ┌─────────────────────┐
                 │        EC2          │
                 │                     │
                 │       EC2-SG        │
                 │                     │
                 │ Allow TCP 80        │
                 │ from ALB-SG         │
                 └─────────────────────┘
                            │
                            ▼
                         NGINX
                          :80
7. Where Does the Target Group Fit?

Remember your previous task.

The complete architecture is:

Internet
    │
    │ HTTP :80
    ▼
┌───────────────┐
│      ALB      │
│    ALB-SG     │
└───────┬───────┘
        │
        │ Listener :80
        ▼
┌───────────────┐
│ Target Group  │
│ Health Check  │
│      /        │
└───────┬───────┘
        │
        │ HTTP :80
        ▼
┌────────────────────────┐
│         EC2            │
│        EC2-SG           │
│                        │
│       NGINX :80        │
└────────────────────────┘
8. Terraform Module Structure

For your Day 28 project, a clean structure can be:

day28/
└── terraform/
    │
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    ├── provider.tf
    │
    └── modules/
        │
        ├── compute/
        │   ├── main.tf
        │   ├── variables.tf
        │   └── outputs.tf
        │
        ├── alb/
        │   ├── main.tf
        │   ├── variables.tf
        │   └── outputs.tf
        │
        └── security-groups/
            ├── main.tf
            ├── variables.tf
            └── outputs.tf

For training, keeping Security Groups in their own module makes the architecture easier to understand.

9. Create Security Group Module

Create:

modules/security-groups/main.tf

Add:

resource "aws_security_group" "alb" {
  name        = "day28-alb-sg"
  description = "Security group for Application Load Balancer"
  vpc_id      = var.vpc_id

  ingress {
    description = "HTTP from Internet"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "Allow outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "day28-alb-sg"
    Environment = "Training"
  }
}

resource "aws_security_group" "ec2" {
  name        = "day28-ec2-sg"
  description = "Security group for EC2 application servers"
  vpc_id      = var.vpc_id

  ingress {
    description     = "HTTP from ALB"
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  egress {
    description = "Allow outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "day28-ec2-sg"
    Environment = "Training"
  }
}
10. Variables

Create:

modules/security-groups/variables.tf

Use:

variable "vpc_id" {
  description = "VPC ID where Security Groups will be created"
  type        = string
}
11. Outputs

Create:

modules/security-groups/outputs.tf

Use:

output "alb_security_group_id" {
  description = "Security Group ID for the ALB"
  value       = aws_security_group.alb.id
}

output "ec2_security_group_id" {
  description = "Security Group ID for EC2 instances"
  value       = aws_security_group.ec2.id
}
12. Call the Module from Root

In your root main.tf:

module "security_groups" {
  source = "./modules/security-groups"

  vpc_id = var.vpc_id
}

Now you can use:

module.security_groups.alb_security_group_id

and:

module.security_groups.ec2_security_group_id
13. Connect ALB to ALB-SG

Your ALB module should receive:

security_group_ids = [
  module.security_groups.alb_security_group_id
]

Example:

module "alb" {
  source = "./modules/alb"

  name = "day28-alb"

  vpc_id = var.vpc_id

  subnet_ids = var.public_subnet_ids

  security_group_ids = [
    module.security_groups.alb_security_group_id
  ]

  target_port = 80
}
14. Connect EC2 to EC2-SG

Your compute module should receive:

security_group_ids = [
  module.security_groups.ec2_security_group_id
]

Example:

module "compute" {
  source = "./modules/compute"

  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  security_group_ids = [
    module.security_groups.ec2_security_group_id
  ]

  subnet_ids = var.private_subnet_ids

  min_size         = 1
  desired_capacity = 2
  max_size         = 3

  target_group_arn = module.alb.target_group_arn

  root_volume_size = 8

  user_data = file("${path.module}/user-data.sh")
}
15. Important Security Concept
Bad architecture
Internet
   │
   ├──────────────► ALB
   │
   │
   └──────────────► EC2

EC2 is directly exposed.

A rule like this on EC2:

cidr_blocks = ["0.0.0.0/0"]

for port 80 allows the entire Internet to reach the application directly.

Better architecture
Internet
   │
   ▼
  ALB
   │
   │ ALB-SG
   ▼
  EC2
   │
   │ EC2-SG
   ▼
Application

Only the ALB is intended to be publicly accessible.

16. Why Reference a Security Group Instead of an IP?

This is an important AWS concept.

Instead of:

cidr_blocks = ["10.0.1.0/24"]

you can say:

security_groups = [
  aws_security_group.alb.id
]

This means:

Allow traffic from resources associated with the ALB Security Group.

This is easier to maintain when ALB IP addresses change.

17. Run Terraform

Go to your project:

cd /workspaces/terraform-lab/day28/terraform

Format:

terraform fmt

Initialize:

terraform init

Validate:

terraform validate

Create the plan:

terraform plan

If everything looks correct:

terraform apply

Enter:

yes
18. Verify in AWS

Go to:

AWS Console
   ↓
EC2
   ↓
Security Groups

You should see:

day28-alb-sg
day28-ec2-sg
ALB-SG

Expected:

Inbound

HTTP
TCP
80
0.0.0.0/0
EC2-SG

Expected:

Inbound

HTTP
TCP
80
Source: day28-alb-sg

The important thing is that EC2-SG does not need:

0.0.0.0/0 → TCP 80
19. Verify the Application

Get the ALB DNS:

terraform output alb_dns_name

Then open:

http://<ALB-DNS-NAME>

Traffic should flow:

Browser
   │
   │ HTTP :80
   ▼
ALB
   │
   │ ALB-SG allows
   ▼
Target Group
   │
   │
   ▼
EC2
   │
   │ EC2-SG allows ALB
   ▼
NGINX
20. Final Security Group Flow

This is the main diagram you should document for the task:

                       INTERNET
                           │
                           │ TCP 80
                           ▼
                 ┌───────────────────┐
                 │       ALB         │
                 │                   │
                 │      ALB-SG       │
                 │                   │
                 │ TCP 80            │
                 │ Source: 0.0.0.0/0 │
                 └─────────┬─────────┘
                           │
                           │ TCP 80
                           │
                           │ Source: ALB-SG
                           ▼
                 ┌───────────────────┐
                 │       EC2         │
                 │                   │
                 │      EC2-SG       │
                 │                   │
                 │ TCP 80            │
                 │ Source: ALB-SG    │
                 └─────────┬─────────┘
                           │
                           ▼
                         NGINX
                           │
                           ▼
                       Application