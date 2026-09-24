TASK 9 — ALB Listener

Time: 30 Minutes

In this task, we create an HTTP Listener for the Application Load Balancer.

You already have:

Internet
   │
   ▼
   ALB
   │
   ▼
Target Group
   │
   ├── EC2-1
   └── EC2-2

Now the Listener connects the incoming HTTP request to the Target Group.

1. What is an ALB Listener?

An ALB Listener waits for incoming requests on a specific:

Port
Protocol

For this task:

Protocol = HTTP
Port     = 80

The listener receives the request and forwards it to the Target Group.

Client
  │
  │ HTTP :80
  ▼
ALB
  │
  │ Listener
  ▼
Target Group
  │
  ▼
EC2 :80
Simple definition

Listener = the ALB component that listens for incoming traffic and defines what action to take with that traffic.

2. Understand the Traffic Flow

When a user opens:

http://your-alb-dns-name

the request flows like this:

Client
  │
  │ HTTP
  ▼
ALB :80
  │
  │ Listener :80
  ▼
Target Group
  │
  ▼
Healthy EC2
  │
  ▼
NGINX :80

The response travels back:

EC2
 │
 ▼
Target Group
 │
 ▼
ALB
 │
 ▼
Client
3. Listener Components

Our listener needs:

Load Balancer
Port
Protocol
Default Action
Target Group

Terraform resource:

resource "aws_lb_listener" "http" {
}
4. Step 1 — Open Your ALB Module

Your structure should be:

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

Open:

modules/alb/main.tf
5. Step 2 — Create HTTP Listener

Add:

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.this.arn

  port     = 80
  protocol = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.this.arn
  }
}

This is the main configuration for Task 9.

6. Understand the Configuration
Load Balancer
load_balancer_arn = aws_lb.this.arn

This tells Terraform:

Attach this listener to the ALB.

Architecture:

ALB
 │
 └── HTTP Listener
Port
port = 80

The listener waits for HTTP requests on port 80.

ALB :80
Protocol
protocol = "HTTP"

The listener accepts HTTP traffic.

Default Action
default_action {
  type             = "forward"
  target_group_arn = aws_lb_target_group.this.arn
}

This means:

When a request arrives, forward it to the Target Group.

7. Complete Traffic Architecture

After creating the listener:

                         INTERNET
                            │
                            │ HTTP :80
                            ▼
                  ┌───────────────────┐
                  │       ALB         │
                  │                   │
                  │ Listener :80      │
                  └─────────┬─────────┘
                            │
                            │ Forward
                            ▼
                  ┌───────────────────┐
                  │   Target Group    │
                  │     HTTP :80      │
                  └─────────┬─────────┘
                            │
                    ┌───────┴───────┐
                    ▼               ▼
                  EC2-1           EC2-2
                    │               │
                    ▼               ▼
                  NGINX           NGINX
8. Step 3 — Target Group Must Exist

The listener references:

aws_lb_target_group.this.arn

Therefore your Target Group should already be defined.

Example:

resource "aws_lb_target_group" "this" {
  name     = "${var.name}-tg"
  port     = 80
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

    timeout  = 5
    interval = 30

    matcher = "200"
  }

  tags = {
    Name        = "${var.name}-target-group"
    Environment = "Training"
  }
}

Terraform understands the dependency:

ALB
 │
 ▼
Target Group
 │
 ▼
Listener

because the listener references the Target Group ARN.

9. Step 4 — Add Listener Output

Open:

modules/alb/outputs.tf

Add:

output "listener_arn" {
  description = "ARN of the HTTP listener"
  value       = aws_lb_listener.http.arn
}

You can also output the listener port:

output "listener_port" {
  description = "HTTP listener port"
  value       = aws_lb_listener.http.port
}

And protocol:

output "listener_protocol" {
  description = "Listener protocol"
  value       = aws_lb_listener.http.protocol
}
10. Step 5 — Root Output

In the root outputs.tf:

output "alb_listener_arn" {
  value = module.alb.listener_arn
}

output "alb_listener_port" {
  value = module.alb.listener_port
}

output "alb_listener_protocol" {
  value = module.alb.listener_protocol
}
11. Step 6 — Format Terraform

Go to your Terraform project:

cd /workspaces/terraform-lab/day28/terraform

Run:

terraform fmt
12. Step 7 — Initialize

Run:

terraform init

If you already initialized the project during Tasks 6–8, Terraform will simply verify/update the configuration.

13. Step 8 — Validate

Run:

terraform validate

Expected:

Success! The configuration is valid.
14. Step 9 — Review Plan

Run:

terraform plan

Look for:

aws_lb_listener.http

Terraform should show the listener being created or updated.

15. Step 10 — Apply

Run:

terraform apply

When prompted:

Do you want to perform these actions?

Enter:

yes
16. Step 11 — Verify the Listener

Run:

terraform output

You should see something similar to:

alb_listener_port = 80
alb_listener_protocol = "HTTP"
alb_listener_arn = "arn:aws:elasticloadbalancing:..."
17. Test Through the ALB

Get your ALB DNS:

terraform output alb_dns_name

You might get:

day28-alb-123456.ap-south-1.elb.amazonaws.com

Open:

http://day28-alb-123456.ap-south-1.elb.amazonaws.com

The request should travel:

Browser
   │
   │ HTTP :80
   ▼
ALB
   │
   │ Listener :80
   ▼
Target Group
   │
   ▼
Healthy EC2
   │
   ▼
NGINX

You should see your application page.

18. Listener vs Target Group

This is an important interview concept.

Listener

The listener answers:

What incoming traffic should the ALB accept, and what should it do with it?

Example:

HTTP :80
     │
     ▼
Forward
Target Group

The Target Group answers:

Which backend servers should receive the traffic?

Example:

Target Group
     │
     ├── EC2-1
     └── EC2-2

Together:

ALB
 │
 │ Listener :80
 ▼
Target Group
 │
 ├── EC2-1
 └── EC2-2
19. Listener Request Flow

Suppose the user requests:

GET /

The process is:

1. Client sends HTTP request
          │
          ▼
2. ALB receives request
          │
          ▼
3. Listener receives request on :80
          │
          ▼
4. Listener executes default action
          │
          ▼
5. Request forwarded to Target Group
          │
          ▼
6. Target Group selects healthy EC2
          │
          ▼
7. EC2/Nginx returns response
          │
          ▼
8. ALB returns response to client
20. Final Architecture

After Task 9:

                         CLIENT
                            │
                            │ HTTP
                            ▼
                  ┌──────────────────┐
                  │       ALB        │
                  │                  │
                  │   Listener :80   │
                  └────────┬─────────┘
                           │
                           │ Forward
                           ▼
                  ┌──────────────────┐
                  │  Target Group    │
                  │    HTTP :80      │
                  └────────┬─────────┘
                           │
                 ┌─────────┴─────────┐
                 ▼                   ▼
              EC2-1               EC2-2
              NGINX               NGINX
21. Task 9 Checklist
[ ] HTTP listener created
[ ] Listener attached to ALB
[ ] Protocol = HTTP
[ ] Port = 80
[ ] Default action = forward
[ ] Target Group ARN configured
[ ] Target Group exists
[ ] Target Group has healthy EC2 targets
[ ] Listener output configured
[ ] terraform fmt completed
[ ] terraform validate passed
[ ] terraform plan reviewed
[ ] terraform apply completed
[ ] ALB DNS tested
[ ] Application page accessible
Key concept
Listener
    │
    │ Receives HTTP :80
    ▼
Target Group
    │
    │ Selects healthy backend
    ▼
EC2 :80

Listener = receives and forwards traffic.
Target Group = identifies and health-checks the backend targets.