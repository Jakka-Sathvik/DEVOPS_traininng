TASK 12 — Connect ALB + ASG + Target Group

Time: 45 Minutes

This task connects everything you built in Tasks 1–11 into one complete AWS compute architecture.

The final result will be:

                         INTERNET
                            │
                            │ HTTP :80
                            ▼
                    ┌───────────────┐
                    │      ALB      │
                    │   Port 80     │
                    │    ALB-SG     │
                    └───────┬───────┘
                            │
                            │ Listener
                            ▼
                    ┌───────────────┐
                    │ Target Group  │
                    │ Health Check /│
                    └───────┬───────┘
                            │
                   ┌────────┴────────┐
                   ▼                 ▼
                EC2-1             EC2-2
              Healthy            Healthy
                   │                 │
                   └────────┬────────┘
                            │
                    Auto Scaling Group

Important: The Auto Scaling Group does not sit "after" the EC2 instances. The ASG manages the EC2 instances. The Target Group receives the instances launched by the ASG.

1. Understand the Complete Architecture

Your infrastructure has these major components:

VPC
 │
 ├── Public Subnets
 │      │
 │      └── ALB
 │
 └── Private/Application Subnets
        │
        └── EC2 instances

Security:

Internet
   │
   ▼
ALB-SG
   │
   │ HTTP :80
   ▼
EC2-SG
   │
   ▼
EC2

Load balancing:

ALB
 │
 ▼
Listener :80
 │
 ▼
Target Group
 │
 ├── EC2-1
 └── EC2-2

Scaling:

Auto Scaling Group
 │
 ├── EC2-1
 └── EC2-2

Therefore, the same EC2 instances are:

launched by the ASG
registered in the Target Group
checked by the Target Group
reached through the ALB
2. Understand the Dependencies

The practical dependency chain is:

VPC
 │
 ├──────────────┐
 ▼              ▼
Subnets      Security Groups
 │              │
 │              ├── ALB-SG
 │              └── EC2-SG
 │
 ▼
Launch Template
 │
 │ AMI
 │ Instance Type
 │ Security Group
 │ User Data
 │ Storage
 │
 ▼
Auto Scaling Group
 │
 │ launches EC2
 ▼
EC2 Instances
 │
 ▼
Target Group
 │
 │ health checks
 ▼
ALB Listener
 │
 ▼
ALB

There is an important correction to the simplified sequence in the task:

The Target Group and ALB can be created before the ASG instances exist. The ASG then registers its instances with the Target Group.

A better logical relationship is:

                 VPC
              /       \
             ▼         ▼
         Subnets       SGs
             │
             ▼
      Launch Template
             │
             ▼
      Auto Scaling Group ───────► Target Group
             │                       ▲
             ▼                       │
           EC2s ── health check ─────┘
                                     │
                                     ▼
                                    ALB
                                     │
                                  Listener
3. Final Terraform Structure

Your Day 28 project should look approximately like this:

day28/
└── terraform/
    │
    ├── provider.tf
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    ├── user-data.sh
    │
    └── modules/
        │
        ├── security-groups/
        │   ├── main.tf
        │   ├── variables.tf
        │   └── outputs.tf
        │
        ├── compute/
        │   ├── main.tf
        │   ├── variables.tf
        │   └── outputs.tf
        │
        └── alb/
            ├── main.tf
            ├── variables.tf
            └── outputs.tf
4. Step 1 — VPC and Subnets

You need:

One VPC
At least two subnets for the ALB
Ideally two application/private subnets for EC2

Example:

VPC: 10.0.0.0/16

Public Subnet 1:
10.0.1.0/24

Public Subnet 2:
10.0.2.0/24

Private Subnet 1:
10.0.11.0/24

Private Subnet 2:
10.0.12.0/24

Architecture:

                    VPC
              10.0.0.0/16
                    │
        ┌───────────┴───────────┐
        │                       │
        ▼                       ▼
   Public Subnets         Private Subnets
        │                       │
        ▼                       ▼
       ALB                  EC2/ASG

For an internet-facing ALB, its subnets should have appropriate connectivity to an Internet Gateway.

5. Step 2 — Security Groups

From Task 11:

ALB-SG
Inbound:
TCP 80
Source: 0.0.0.0/0
EC2-SG
Inbound:
TCP 80
Source: ALB-SG

Architecture:

Internet
    │
    │ TCP 80
    ▼
 ALB-SG
    │
    │ TCP 80
    ▼
 EC2-SG
    │
    ▼
   EC2
6. Step 3 — Launch Template

The Launch Template is the blueprint for EC2 instances.

Your modules/compute/main.tf should contain:

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

The Launch Template defines:

AMI
Instance Type
Security Group
User Data
Storage
Tags

It does not create a running EC2 instance by itself.

7. Step 4 — Create the Target Group

The Target Group is the backend pool.

resource "aws_lb_target_group" "this" {
  name     = "${var.name}-tg"
  port     = var.target_port
  protocol = "HTTP"

  vpc_id = var.vpc_id

  target_type = "instance"

  health_check {
    enabled             = true
    protocol            = "HTTP"
    port                = "traffic-port"
    path                = "/"
    healthy_threshold   = 2
    unhealthy_threshold = 2
    timeout             = 5
    interval            = 30
    matcher             = "200"
  }

  tags = {
    Name        = "${var.name}-target-group"
    Environment = "Training"
  }
}

The Target Group expects:

HTTP
Port 80
Path /
8. Step 5 — Create the ALB

Your ALB should use the ALB Security Group.

resource "aws_lb" "this" {
  name               = var.name
  internal           = false
  load_balancer_type = "application"

  security_groups = var.security_group_ids

  subnets = var.subnet_ids

  enable_deletion_protection = false

  tags = {
    Name        = var.name
    Environment = "Training"
  }
}

Because:

internal = false

the ALB is internet-facing.

9. Step 6 — Create the ALB Listener

The listener accepts client traffic.

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.this.arn

  port     = 80
  protocol = "HTTP"

  default_action {
    type             = "forward"

    target_group_arn = aws_lb_target_group.this.arn
  }
}

Traffic:

Browser
   │
   │ HTTP :80
   ▼
ALB
   │
   │ Listener :80
   ▼
Target Group
10. Step 7 — Connect ASG to Target Group

This is the most important part of Task 12.

In your compute module:

resource "aws_autoscaling_group" "this" {
  name = "day28-compute-asg"

  min_size         = var.min_size
  desired_capacity = var.desired_capacity
  max_size         = var.max_size

  vpc_zone_identifier = var.subnet_ids

  target_group_arns = [
    var.target_group_arn
  ]

  launch_template {
    id      = aws_launch_template.this.id
    version = "$Latest"
  }

  health_check_type = "ELB"

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

This line is extremely important:

target_group_arns = [
  var.target_group_arn
]

It tells the ASG:

Register instances launched by this Auto Scaling Group with this Target Group.

11. Step 8 — ASG Configuration

For your training environment:

min_size         = 1
desired_capacity = 2
max_size         = 3

This means:

Minimum = 1
Desired = 2
Maximum = 3

Initially:

ASG
 │
 ├── EC2-1
 └── EC2-2

If one instance fails:

ASG
 │
 ├── EC2-1 ❌
 └── EC2-2 ✅

ASG launches replacement

 │
 ├── EC2-2 ✅
 └── EC2-3 ✅
12. Step 9 — Root Terraform Configuration

Now connect the modules.

Security Groups
module "security_groups" {
  source = "./modules/security-groups"

  vpc_id = var.vpc_id
}
ALB
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

Notice:

module.security_groups.alb_security_group_id

The ALB receives the ALB Security Group.

Compute / ASG
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

  tags = {
    Name        = "day28-compute"
    Environment = "Training"
  }
}

The most important connection is:

target_group_arn = module.alb.target_group_arn
13. Understand the Module Connections

Now you have:

Security Groups
      │
      ├───────────────► ALB
      │
      └───────────────► EC2

ALB
 │
 ├── Listener
 │
 └── Target Group
          │
          ▲
          │
        ASG
          │
          ▼
         EC2

Or more simply:

              Security Groups
                /         \
               ▼           ▼
             ALB          EC2
              │             ▲
              │             │
              ▼             │
        Target Group ◄──── ASG
                            │
                            ▼
                           EC2
14. Complete Request Flow

A user opens:

http://<ALB-DNS-NAME>

The request travels:

1. Browser
      │
      ▼
2. Internet
      │
      ▼
3. ALB
      │
      ▼
4. Listener :80
      │
      ▼
5. Target Group
      │
      ▼
6. Healthy EC2
      │
      ▼
7. NGINX :80
      │
      ▼
8. Application Response
15. Complete Health Check Flow

The ALB/Target Group periodically checks the EC2 instances:

Target Group
      │
      ├── GET /
      │
      ├────────► EC2-1 :80
      │             │
      │             └── 200 OK
      │
      └────────► EC2-2 :80
                    │
                    └── 200 OK

Result:

EC2-1 → Healthy
EC2-2 → Healthy

If EC2-1 fails:

EC2-1 → Unhealthy
EC2-2 → Healthy

The ALB sends normal traffic only to EC2-2.

16. Terraform Dependency

Terraform understands dependencies through references.

For example:

target_group_arn = module.alb.target_group_arn

creates a dependency:

ALB module
     │
     ▼
Target Group
     │
     ▼
Compute module
     │
     ▼
ASG

Similarly:

module.security_groups.alb_security_group_id

creates:

Security Groups
      │
      ▼
ALB

And:

module.security_groups.ec2_security_group_id

creates:

Security Groups
      │
      ▼
Compute / ASG

Terraform uses these references to determine the correct resource creation order.

17. Run Terraform

Go to your project:

cd /workspaces/terraform-lab/day28/terraform
Format
terraform fmt
Initialize
terraform init
Validate
terraform validate

Expected:

Success! The configuration is valid.
Create plan
terraform plan

Carefully review:

aws_lb
aws_lb_target_group
aws_lb_listener
aws_launch_template
aws_autoscaling_group
aws_security_group
Apply
terraform apply

Enter:

yes
18. Check Terraform Outputs

Run:

terraform output

You should have useful outputs such as:

alb_dns_name
alb_arn
target_group_arn
launch_template_id
autoscaling_group_name

Get only the ALB DNS:

terraform output alb_dns_name
19. Test the Application

Open:

http://<ALB-DNS-NAME>

You should see your NGINX application:

DevOps Training Application

Server: EC2
Environment: Training
Status: Application is running

You are now testing the complete architecture, not an individual EC2 instance.

20. Verify Target Group

Go to:

AWS Console
   ↓
EC2
   ↓
Target Groups
   ↓
day28-alb-tg
   ↓
Targets

Expected:

Instance       Port       Health
------------------------------------
EC2-1          80         Healthy
EC2-2          80         Healthy
21. Verify Auto Scaling Group

Go to:

AWS Console
   ↓
EC2
   ↓
Auto Scaling Groups
   ↓
day28-compute-asg

Check:

Minimum capacity: 1
Desired capacity: 2
Maximum capacity: 3

You should see two instances running.

22. Verify the Security Groups
ALB
ALB
 │
 └── ALB-SG
       │
       └── TCP 80 from Internet
EC2
EC2
 │
 └── EC2-SG
       │
       └── TCP 80 from ALB-SG
23. Test Failure Recovery

This is a very useful practical test.

Stop NGINX on one EC2:

sudo systemctl stop nginx

After health checks:

Target Group

EC2-1 → Unhealthy
EC2-2 → Healthy

The ALB should stop sending normal requests to EC2-1.

Then restore:

sudo systemctl start nginx

After successful health checks:

EC2-1 → Healthy
EC2-2 → Healthy
24. Test ASG Recovery

For another training test, terminate one EC2 instance from the AWS Console.

The ASG should detect that the number of instances has fallen below desired capacity.

For example:

Before:

ASG
 ├── EC2-1
 └── EC2-2

Desired = 2

Terminate EC2-1:

ASG
 └── EC2-2

Running = 1
Desired = 2

The ASG should launch a replacement:

ASG
 ├── EC2-2
 └── EC2-3

Then the new instance should:

Launch
  ↓
Run User Data
  ↓
Install NGINX
  ↓
Start NGINX
  ↓
Register in Target Group
  ↓
Health Check
  ↓
Healthy
25. Final Architecture

This is the architecture you should understand and document:

                              INTERNET
                                  │
                                  │ HTTP :80
                                  ▼
                         ┌─────────────────┐
                         │       ALB       │
                         │                 │
                         │     ALB-SG      │
                         └────────┬────────┘
                                  │
                                  │ Listener :80
                                  ▼
                         ┌─────────────────┐
                         │  Target Group   │
                         │                 │
                         │ Health Check /  │
                         └────────┬────────┘
                                  │
                     ┌────────────┴────────────┐
                     │                         │
                     ▼                         ▼
                ┌─────────┐               ┌─────────┐
                │  EC2-1  │               │  EC2-2  │
                │  :80    │               │  :80    │
                │ EC2-SG  │               │ EC2-SG  │
                └────┬────┘               └────┬────┘
                     │                         │
                     └────────────┬────────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ Auto Scaling    │
                         │     Group       │
                         │                 │
                         │ Min: 1          │
                         │ Desired: 2      │
                         │ Max: 3          │
                         └─────────────────┘
The most important relationships
ALB
 ↓
Listener
 ↓
Target Group
 ↓
EC2 instances
 ↑
Auto Scaling Group
 ↑
Launch Template

And security:

Internet
   ↓
ALB-SG
   ↓
ALB
   ↓
EC2-SG
   ↓
EC2