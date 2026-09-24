TASK 7 — Application Load Balancer

Time: 60 Minutes

In Task 6, you created:

Launch Template
      │
      ▼
Auto Scaling Group
      │
      ├── EC2-1
      └── EC2-2

Now we add an Application Load Balancer (ALB) in front of those EC2 instances.

The final architecture will be:

                         INTERNET
                            │
                            ▼
                  Application Load Balancer
                            │
                            ▼
                       Target Group
                       /          \
                      ▼            ▼
                    EC2-1        EC2-2
1. What is an Application Load Balancer?

An Application Load Balancer distributes incoming HTTP/HTTPS traffic across multiple targets such as EC2 instances.

Instead of users directly accessing:

EC2-1
EC2-2

they access the ALB:

User
  │
  ▼
ALB
  │
  ├── EC2-1
  └── EC2-2

This provides a single entry point for the application.

2. Why Do We Need an ALB?

Without an ALB:

User
 ├──────────► EC2-1
 │
 └──────────► EC2-2

The user needs to know individual EC2 addresses.

With an ALB:

User
  │
  ▼
ALB
  │
  ├──► EC2-1
  │
  └──► EC2-2

The user only needs the ALB endpoint.

3. Important ALB Components

For this task, understand these five concepts:

ALB
 │
 ├── Scheme
 ├── Subnets
 ├── Security Group
 ├── Listener
 └── Target Group
4. ALB Scheme

The ALB has two common schemes:

Internet-facing
scheme = "internet-facing"

The ALB receives traffic from the internet.

For this training architecture, use:

scheme = "internet-facing"

Architecture:

INTERNET
    │
    ▼
Public ALB
    │
    ▼
Target Group
    │
    ├── EC2
    └── EC2
Internal
scheme = "internal"

An internal ALB is intended for private/internal application traffic.

For this task:

internet-facing ✅
internal         ❌
5. ALB Subnets

An internet-facing ALB should be placed in public subnets.

For training, use at least two public subnets in different Availability Zones:

VPC
 │
 ├── Public Subnet A
 │      └── AZ-A
 │
 └── Public Subnet B
        └── AZ-B

Then:

subnets = [
  "subnet-public-A",
  "subnet-public-B"
]

The ALB can use both subnets.

6. What Makes a Subnet Public?

A public subnet normally has a route to an Internet Gateway:

Public Subnet
      │
      ▼
Route Table
      │
      ▼
Internet Gateway
      │
      ▼
Internet

For this task, make sure the subnets you provide to the ALB are actually your public subnets.

7. ALB Security Group

Create a Security Group for the ALB.

For HTTP training traffic:

Internet
   │
   │ TCP 80
   ▼
ALB Security Group

Example:

resource "aws_security_group" "alb" {
  name        = "day28-alb-sg"
  description = "Security group for Application Load Balancer"
  vpc_id      = var.vpc_id

  ingress {
    description = "HTTP from internet"
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
Why allow 0.0.0.0/0 for port 80?

Because this is an internet-facing web application and we want users on the internet to access the ALB.

For production environments, security rules should be designed according to the actual application requirements.

8. Important Security Group Architecture

Don't think of the ALB and EC2 security groups as the same thing.

A better architecture is:

Internet
   │
   │ TCP 80
   ▼
ALB Security Group
   │
   │ TCP 80
   ▼
EC2 Security Group
   │
   ▼
EC2

Therefore:

Internet
   │
   ▼
ALB-SG
   │
   ▼
EC2-SG
   │
   ▼
EC2

Later, you should restrict the EC2 Security Group so that HTTP traffic comes from the ALB Security Group, rather than allowing HTTP from the whole internet.

9. Step 1 — Create ALB Module Structure

Your project can now look like:

day28/
└── terraform/
    │
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    ├── user-data.sh
    │
    └── modules/
        ├── compute/
        │   ├── main.tf
        │   ├── variables.tf
        │   └── outputs.tf
        │
        └── alb/
            ├── main.tf
            ├── variables.tf
            └── outputs.tf

We'll create a reusable alb module.

10. Step 2 — ALB Variables

Create:

modules/alb/variables.tf

Add:

variable "name" {
  description = "Name of the Application Load Balancer"
  type        = string
  default     = "day28-alb"
}

variable "vpc_id" {
  description = "VPC ID where the ALB will be deployed"
  type        = string
}

variable "subnet_ids" {
  description = "Public subnet IDs for the ALB"
  type        = list(string)
}

variable "security_group_ids" {
  description = "Security Group IDs for the ALB"
  type        = list(string)
}

variable "target_port" {
  description = "Port used by the application targets"
  type        = number
  default     = 80
}
11. Step 3 — Create the Application Load Balancer

Create:

modules/alb/main.tf

Add:

resource "aws_lb" "this" {
  name               = var.name
  internal           = false
  load_balancer_type = "application"
  security_groups    = var.security_group_ids
  subnets            = var.subnet_ids

  enable_deletion_protection = false

  tags = {
    Name        = var.name
    Environment = "Training"
  }
}
12. Understand This Configuration

This line:

internal = false

means the ALB is internet-facing.

This:

load_balancer_type = "application"

creates an Application Load Balancer.

This:

security_groups = var.security_group_ids

attaches the ALB Security Group.

This:

subnets = var.subnet_ids

places the ALB into the specified public subnets.

13. Step 4 — Create Target Group

The ALB needs somewhere to send traffic.

Create:

resource "aws_lb_target_group" "this" {
  name     = "${var.name}-tg"
  port     = var.target_port
  protocol = "HTTP"
  vpc_id   = var.vpc_id

  target_type = "instance"

  health_check {
    enabled             = true
    path                = "/"
    protocol            = "HTTP"
    port                = "traffic-port"
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
14. What is a Target Group?

A Target Group is a collection of backend targets.

For example:

Target Group
     │
     ├── EC2-1
     │
     └── EC2-2

The ALB sends traffic to targets registered in the target group.

15. Target Group Health Check

The target group checks:

http://EC2-IP/

because we configured:

path = "/"

If the server responds successfully:

HTTP 200

the target is considered healthy.

Example:

EC2-1 → 200 → HEALTHY
EC2-2 → 200 → HEALTHY

If an EC2 doesn't respond correctly:

EC2-1 → timeout → UNHEALTHY

The ALB stops sending normal traffic to that unhealthy target.

16. Step 5 — Create ALB Listener

The listener tells the ALB:

What port should I listen on, and where should I send the request?

Add:

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.this.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.this.arn
  }
}

Architecture:

Internet
   │
   │ HTTP :80
   ▼
ALB
   │
   │ Listener :80
   ▼
Target Group
   │
   ├── EC2-1
   └── EC2-2
17. Complete modules/alb/main.tf

Your file should now contain:

resource "aws_lb" "this" {
  name               = var.name
  internal           = false
  load_balancer_type = "application"
  security_groups    = var.security_group_ids
  subnets            = var.subnet_ids

  enable_deletion_protection = false

  tags = {
    Name        = var.name
    Environment = "Training"
  }
}

resource "aws_lb_target_group" "this" {
  name     = "${var.name}-tg"
  port     = var.target_port
  protocol = "HTTP"
  vpc_id   = var.vpc_id

  target_type = "instance"

  health_check {
    enabled             = true
    path                = "/"
    protocol            = "HTTP"
    port                = "traffic-port"
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

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.this.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.this.arn
  }
}
18. Step 6 — Outputs

Create:

modules/alb/outputs.tf

Add:

output "alb_id" {
  description = "Application Load Balancer ID"
  value       = aws_lb.this.id
}

output "alb_arn" {
  description = "Application Load Balancer ARN"
  value       = aws_lb.this.arn
}

output "alb_dns_name" {
  description = "Application Load Balancer DNS name"
  value       = aws_lb.this.dns_name
}

output "target_group_arn" {
  description = "Target Group ARN"
  value       = aws_lb_target_group.this.arn
}

output "listener_arn" {
  description = "ALB HTTP listener ARN"
  value       = aws_lb_listener.http.arn
}

The most useful output for testing is:

aws_lb.this.dns_name
19. Step 7 — Root Module

In your root main.tf, call the ALB module.

Example:

module "alb" {
  source = "./modules/alb"

  name = "day28-alb"

  vpc_id = "vpc-xxxxxxxxxxxxxxxxx"

  subnet_ids = [
    "subnet-public-aaaa",
    "subnet-public-bbbb"
  ]

  security_group_ids = [
    "sg-alb-xxxxxxxxxxxxxxxxx"
  ]

  target_port = 80
}

Replace the example values with your actual resources.

20. Important Problem — ALB Doesn't Automatically Know About Your ASG

At this point:

ALB
 │
 ▼
Target Group

But we need:

Target Group
 │
 ├── EC2-1
 └── EC2-2

Because your EC2s are managed by an Auto Scaling Group, the best approach is to attach the Target Group to the ASG.

That is the connection between Task 6 and Task 7.

21. Add Target Group to the Auto Scaling Group

Modify your ASG:

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

  target_group_arns = [
    var.target_group_arn
  ]

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

There are two important changes:

target_group_arns = [
  var.target_group_arn
]

and:

health_check_type = "ELB"
22. Add Target Group Variable to Compute Module

In:

modules/compute/variables.tf

add:

variable "target_group_arn" {
  description = "ARN of the ALB Target Group"
  type        = string
  default     = null
}

Then you have:

ALB Module
    │
    ▼
Target Group ARN
    │
    ▼
Compute Module
    │
    ▼
Auto Scaling Group
23. Important Terraform Dependency

There is a dependency:

ALB
 │
 ▼
Target Group
 │
 ▼
Auto Scaling Group

But if both modules are defined in the root module, Terraform needs the target group ARN before configuring the ASG.

A cleaner architecture for this training is:

                    Root Module
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
          ALB Module          Compute Module
              │                   │
              ▼                   ▼
         Target Group          ASG
              │                   │
              └────── ARN ────────┘

However, because the compute module also creates the Launch Template and ASG, you need to avoid a circular dependency.

For this task, the simplest approach is to keep the ALB and Compute resources in the root module or pass the target group ARN into the compute module, while ensuring the ALB module itself does not depend on the ASG.

24. Root main.tf Example

Conceptually:

module "alb" {
  source = "./modules/alb"

  name = "day28-alb"

  vpc_id = var.vpc_id

  subnet_ids = var.public_subnet_ids

  security_group_ids = [
    aws_security_group.alb.id
  ]

  target_port = 80
}

module "compute" {
  source = "./modules/compute"

  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  security_group_ids = [
    var.ec2_security_group_id
  ]

  subnet_ids = var.private_subnet_ids

  min_size         = 1
  desired_capacity = 2
  max_size         = 3

  target_group_arn = module.alb.target_group_arn

  root_volume_size = 8

  user_data = file("${path.module}/user-data.sh")
}

This gives:

                     Internet
                        │
                        ▼
                       ALB
                        │
                        ▼
                   Target Group
                        │
                        ▼
                       ASG
                        │
                ┌───────┴───────┐
                ▼               ▼
              EC2-1           EC2-2
25. Public vs Private Subnets

For a more realistic architecture:

                    INTERNET
                        │
                        ▼
                Public Subnets
                        │
                        ▼
                       ALB
                        │
                        ▼
                Private Subnets
                        │
                ┌───────┴───────┐
                ▼               ▼
              EC2-1           EC2-2

For your specific task:

ALB → Public Subnets

Your EC2 instances can be in private subnets in a production-style architecture, provided they have the required outbound connectivity.

For training, follow the subnet architecture your trainer has provided.

26. Step 8 — Terraform Commands

Go to:

cd /workspaces/terraform-lab/day28/terraform

Run:

terraform fmt

Then:

terraform init

Then:

terraform validate

Expected:

Success! The configuration is valid.

Now:

terraform plan

Review the resources Terraform wants to create.

Then:

terraform apply

Type:

yes
27. Step 9 — Get the ALB DNS Name

Run:

terraform output

You should get something similar to:

alb_dns_name = "day28-alb-123456789.ap-south-1.elb.amazonaws.com"

You can also output it from the root module:

output "alb_dns_name" {
  value = module.alb.alb_dns_name
}

Then:

terraform output alb_dns_name
28. Test the Application

Open the ALB DNS name in your browser:

http://<ALB-DNS-NAME>

For example:

http://day28-alb-123456789.ap-south-1.elb.amazonaws.com

The request flow is:

Browser
   │
   ▼
ALB
   │
   ▼
Listener :80
   │
   ▼
Target Group
   │
   ▼
EC2
   │
   ▼
NGINX
   │
   ▼
index.html

You should see your page:

DevOps Training Application

Server: EC2
Environment: Training
Status: Application is running
29. Check Target Health

In AWS, check:

EC2
  ↓
Target Groups
  ↓
day28-alb-tg
  ↓
Targets

You want:

EC2-1    Healthy
EC2-2    Healthy

If you see:

Unhealthy

check:

NGINX is running.
EC2 Security Group allows HTTP from the ALB Security Group.
Target Group port is 80.
Health check path / works.
EC2 instances are in the correct VPC.
User Data completed successfully.
The application is actually listening on port 80.
30. Useful EC2 Troubleshooting

If you can access an EC2:

sudo systemctl status nginx

Check:

curl http://localhost

You should get the HTML response.

Check User Data:

sudo cat /var/log/cloud-init-output.log

Check whether port 80 is listening:

sudo ss -tulpn | grep :80
31. Final Architecture

After Task 7, your architecture should conceptually be:

                           INTERNET
                              │
                              │ HTTP :80
                              ▼
                 ┌────────────────────────┐
                 │ Application Load        │
                 │ Balancer                │
                 │ Internet-facing         │
                 └───────────┬────────────┘
                             │
                             │ Listener :80
                             ▼
                     ┌───────────────┐
                     │ Target Group  │
                     │ HTTP :80      │
                     └───────┬───────┘
                             │
                    ┌────────┴────────┐
                    ▼                 ▼
                  EC2-1             EC2-2
                    ▲                 ▲
                    └────── ASG ──────┘
32. Task 7 Checklist
[ ] ALB module created
[ ] ALB created
[ ] ALB is internet-facing
[ ] ALB uses public subnets
[ ] ALB Security Group created
[ ] HTTP port 80 allowed
[ ] Target Group created
[ ] Target Group uses HTTP
[ ] Health check configured
[ ] Listener created
[ ] Listener uses port 80
[ ] Listener forwards to Target Group
[ ] Target Group connected to ASG
[ ] EC2 instances registered as targets
[ ] Targets become Healthy
[ ] ALB DNS name obtained
[ ] Application tested through ALB
[ ] terraform fmt completed
[ ] terraform validate passed
[ ] terraform plan reviewed
[ ] terraform apply completed
The key concept for Task 7

Remember this flow:

ALB
 │
 │ Listener
 ▼
Target Group
 │
 │
 ▼
Auto Scaling Group
 │
 ├── EC2-1
 └── EC2-2

ALB = receives traffic
Listener = listens on a port
Target Group = identifies backend targets
ASG = maintains EC2 instances
EC2 = runs the application