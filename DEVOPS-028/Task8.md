TASK 8 — Target Group

Time: 45 Minutes

In Task 7, you created the Application Load Balancer. Now we focus specifically on the Target Group.

The Target Group is the component that tells the ALB:

Which backend servers should receive the requests, on which port, and whether those servers are healthy.

The overall flow is:

                    INTERNET
                       │
                       ▼
                     ALB
                       │
                       │ Listener :80
                       ▼
                ┌───────────────┐
                │ Target Group  │
                │ HTTP :80      │
                └───────┬───────┘
                        │
                  ┌─────┴─────┐
                  ▼           ▼
                EC2-1       EC2-2
1. What is a Target Group?

A Target Group is a logical group of backend targets.

For this task:

Target Group
     │
     ├── EC2-1
     └── EC2-2

The ALB receives the request and forwards it to a healthy target in the Target Group.

Simple example

A user requests:

http://my-alb.example.com

The request travels:

User
 │
 ▼
ALB
 │
 ▼
Target Group
 │
 ├── EC2-1
 │
 └── EC2-2
2. Important Target Group Settings

You need to understand four important settings:

Protocol
Port
Target Type
Health Check
3. Protocol

For our web application:

protocol = "HTTP"

This means the ALB communicates with the EC2 application using HTTP.

Architecture:

User
 │
 │ HTTP
 ▼
ALB
 │
 │ HTTP
 ▼
Target Group
 │
 ▼
EC2
4. Port

Our NGINX application is running on:

Port 80

Therefore:

port = 80

The Target Group sends traffic to:

EC2:80
5. Target Type

For EC2 instances:

target_type = "instance"

This means the Target Group registers EC2 instances as targets.

Example:

Target Group
     │
     ├── i-012345
     └── i-067890

Other target types exist, such as IP-based targets, but for this training architecture use:

target_type = "instance"
6. Health Check

The ALB needs to know:

Is the application running correctly?

The Target Group performs health checks.

For our application:

HTTP
Port 80
Path /

Terraform:

health_check {
  enabled  = true
  protocol = "HTTP"
  port     = "traffic-port"
  path     = "/"
}

The ALB effectively checks:

http://EC2-IP/

If the EC2 returns a successful response such as:

HTTP 200

the target becomes:

HEALTHY
7. Healthy vs Unhealthy

Suppose you have:

Target Group
 │
 ├── EC2-1 → Healthy
 │
 └── EC2-2 → Healthy

The ALB can send traffic to both.

If EC2-2 fails:

Target Group
 │
 ├── EC2-1 → Healthy
 │
 └── EC2-2 → Unhealthy

The ALB stops sending normal traffic to EC2-2.

This is extremely important for high availability.

8. Step 1 — Create the Target Group Module

Your project can look like:

day28/
└── terraform/
    │
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
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

Because the Target Group is part of the ALB architecture, we'll keep it inside the alb module.

9. Step 2 — Target Group Variables

Open:

modules/alb/variables.tf

Use:

variable "name" {
  description = "Name of the Application Load Balancer"
  type        = string
  default     = "day28-alb"
}

variable "vpc_id" {
  description = "VPC ID for the Target Group"
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
  description = "Port used by the backend application"
  type        = number
  default     = 80
}
10. Step 3 — Create the Target Group

Open:

modules/alb/main.tf

Add:

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
11. Understand Every Setting
Name
name = "${var.name}-tg"

If:

var.name = "day28-alb"

the Target Group becomes:

day28-alb-tg
Port
port = var.target_port

If:

target_port = 80

then:

Target Group → EC2:80
Protocol
protocol = "HTTP"

The backend communication uses HTTP.

VPC
vpc_id = var.vpc_id

The Target Group must belong to the same VPC as the EC2 targets.

Target Type
target_type = "instance"

The Target Group registers EC2 instances.

12. Health Check Configuration

The complete health check is:

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

Let's understand it.

path
path = "/"

The ALB checks:

http://EC2-IP/
interval
interval = 30

The health check runs approximately every 30 seconds.

timeout
timeout = 5

The target must respond within the configured timeout.

healthy_threshold
healthy_threshold = 2

The target needs successful health checks to become healthy.

unhealthy_threshold
unhealthy_threshold = 2

Consecutive failed checks can cause the target to be marked unhealthy.

matcher
matcher = "200"

We expect an HTTP 200 response.

13. Step 4 — Create the ALB Listener

The Target Group alone doesn't receive user traffic directly.

The ALB listener connects the ALB to the Target Group.

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.this.arn

  port     = 80
  protocol = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.this.arn
  }
}

The flow is:

ALB
 │
 │ Listener :80
 ▼
Target Group
 │
 ▼
EC2 :80
14. Step 5 — Connect Target Group to ASG

This is the most important part of Task 8.

Your ASG from Task 6 should use:

target_group_arns = [
  var.target_group_arn
]

For example:

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

Now the architecture becomes:

                    ALB
                     │
                     ▼
                Target Group
                     │
             ┌───────┴───────┐
             ▼               ▼
           EC2-1           EC2-2
             ▲               ▲
             └───────┬───────┘
                     │
                     ASG

When the ASG creates a new EC2, that instance can automatically become a target of the Target Group.

15. Step 6 — Add Target Group Variable to Compute Module

In:

modules/compute/variables.tf

add:

variable "target_group_arn" {
  description = "ARN of the ALB Target Group"
  type        = string
}

Then the ASG can use:

target_group_arns = [
  var.target_group_arn
]
16. Step 7 — Pass the Target Group ARN

In your root main.tf:

module "alb" {
  source = "./modules/alb"

  name = "day28-alb"

  vpc_id = var.vpc_id

  subnet_ids = var.public_subnet_ids

  security_group_ids = [
    var.alb_security_group_id
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

The important line is:

target_group_arn = module.alb.target_group_arn

This connects the two modules.

17. Step 8 — Target Group Output

In:

modules/alb/outputs.tf

add:

output "target_group_arn" {
  description = "ARN of the Target Group"
  value       = aws_lb_target_group.this.arn
}

output "target_group_name" {
  description = "Name of the Target Group"
  value       = aws_lb_target_group.this.name
}

Root output:

output "target_group_name" {
  value = module.alb.target_group_name
}
18. Step 9 — Terraform Commands

Go to your project:

cd /workspaces/terraform-lab/day28/terraform

Format:

terraform fmt

Initialize:

terraform init

Validate:

terraform validate

You want:

Success! The configuration is valid.

Then:

terraform plan

Review carefully.

If everything is correct:

terraform apply

Enter:

yes
19. Step 10 — Verify Target Group

After deployment, check:

terraform output

You should see:

target_group_name = "day28-alb-tg"

In AWS, navigate conceptually to:

EC2
  ↓
Target Groups
  ↓
day28-alb-tg
  ↓
Targets

You should see:

Target ID       Port       Health
------------------------------------
EC2-1           80         Healthy
EC2-2           80         Healthy
20. If Targets Are Unhealthy

This is a very common troubleshooting exercise.

Check the following in order.

1. Is NGINX running?
sudo systemctl status nginx
2. Does the application respond locally?
curl http://localhost

You should receive your HTML page.

3. Is port 80 listening?
sudo ss -tulpn | grep :80
4. Is the health check path correct?

Target Group:

/

Test:

http://EC2-IP/
5. Is the EC2 Security Group allowing traffic?

The EC2 Security Group should allow:

TCP 80
Source = ALB Security Group

Prefer this:

ALB-SG
   │
   │ TCP 80
   ▼
EC2-SG

rather than unnecessarily allowing:

0.0.0.0/0 → EC2:80
21. Complete Request Flow

When the user opens the ALB URL:

                    USER
                     │
                     │ HTTP :80
                     ▼
              Application LB
                     │
                     │ Listener
                     ▼
                Target Group
                     │
              ┌──────┴──────┐
              ▼             ▼
            EC2-1         EC2-2
              │             │
              ▼             ▼
            NGINX         NGINX
              │             │
              └──────┬──────┘
                     ▼
                  RESPONSE
22. Important Difference

Make sure you understand the difference between these components:

Component	Responsibility
ALB	Receives incoming application traffic
Listener	Listens on port/protocol such as HTTP :80
Target Group	Defines backend targets
Health Check	Determines whether targets are healthy
ASG	Creates and maintains EC2 instances
EC2	Runs the application

The complete relationship is:

ALB
 │
 │ Listener :80
 ▼
Target Group
 │
 │ Health Check
 ▼
EC2 Instances
 │
 ▼
Application
23. Final Task 8 Architecture
                         INTERNET
                            │
                            │ HTTP :80
                            ▼
                 ┌─────────────────────┐
                 │ Application Load    │
                 │ Balancer            │
                 └──────────┬──────────┘
                            │
                       Listener :80
                            │
                            ▼
                 ┌─────────────────────┐
                 │ Target Group        │
                 │ HTTP :80            │
                 │ Health Check: /     │
                 └──────────┬──────────┘
                            │
                 ┌──────────┴──────────┐
                 ▼                     ▼
              EC2-1                  EC2-2
             Healthy                Healthy
                 ▲                     ▲
                 └──────────┬──────────┘
                            │
                    Auto Scaling Group
24. Task 8 Checklist
[ ] Target Group created
[ ] Target Group belongs to correct VPC
[ ] Protocol = HTTP
[ ] Port = 80
[ ] Target Type = instance
[ ] Health Check enabled
[ ] Health Check Path = /
[ ] Health Check Port = traffic-port
[ ] Matcher = 200
[ ] ALB Listener forwards to Target Group
[ ] Target Group connected to ASG
[ ] EC2 instances registered
[ ] EC2-1 is Healthy
[ ] EC2-2 is Healthy
[ ] terraform fmt completed
[ ] terraform validate passed
[ ] terraform plan reviewed
[ ] terraform apply completed
Key concept to remember

Target Group = backend pool + health checks.

ALB
 │
 ▼
Target Group
 │
 ├── EC2-1 → Healthy
 └── EC2-2 → Healthy

The ALB decides where traffic enters, while the Target Group defines where that traffic goes.