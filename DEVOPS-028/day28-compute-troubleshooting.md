TASK 15 — Terraform Troubleshooting

Day: 28
Topic: AWS Compute Architecture Troubleshooting
Time: 45 Minutes

1. Objective

The objective of this exercise is to troubleshoot a controlled failure in the Day 28 AWS Compute Architecture.

The infrastructure contains:

Internet
   │
   ▼
Application Load Balancer
   │
   ▼
Listener :80
   │
   ▼
Target Group
   │
   ▼
Auto Scaling Group
   │
   ▼
EC2 Instances
   │
   ▼
NGINX Application

Terraform is used to manage the infrastructure.

The trainee must identify the failure, determine the root cause, fix the Terraform configuration, and verify that the application works again.

2. Possible Failures

The trainer may introduce one controlled failure.

Possible problems include:

Incorrect Target Group port
Wrong Security Group rule
Incorrect subnet
Incorrect health-check path
Incorrect ALB listener configuration
Launch Template error
Auto Scaling Group configuration issue
EC2 User Data failure
3. Troubleshooting Process

Follow this order:

1. Check ALB
2. Check Listener
3. Check Target Group
4. Check Target Health
5. Check EC2
6. Check Security Group
7. Check User Data
8. Check Application
9. Identify root cause
10. Fix Terraform
11. terraform plan
12. terraform apply
13. Verify

The important rule is:

Do not immediately change random Terraform settings. First identify where the failure is occurring.

4. Step 1 — Check the Application Load Balancer

Go to:

AWS Console
    ↓
EC2
    ↓
Load Balancers
    ↓
day28-alb

Check:

State: Active
Scheme: Internet-facing
Type: Application Load Balancer

Check the ALB subnets.

The ALB should normally be deployed into public subnets that have connectivity to an Internet Gateway.

Expected:

Public Subnet 1
       │
       └── ALB

Public Subnet 2
       │
       └── ALB

If the ALB is not active or is deployed incorrectly, investigate the Terraform configuration.

5. Step 2 — Check the Listener

Go to:

ALB
 ↓
Listeners and rules

Expected listener:

Protocol: HTTP
Port: 80
Default Action: Forward
Target Group: day28-alb-tg

Expected architecture:

Client
   │
 HTTP :80
   ▼
ALB
   │
Listener :80
   │
Forward
   ▼
Target Group

Terraform example:

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.this.arn

  port     = 80
  protocol = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.this.arn
  }
}
Possible Failure

For example:

port = 8080

while the application is running on port 80.

This can cause traffic to be sent incorrectly.

Fix

Use:

port = 80

and make sure the target group uses the correct application port.

6. Step 3 — Check the Target Group

Go to:

EC2
 ↓
Target Groups
 ↓
day28-alb-tg

Check:

Protocol: HTTP
Port: 80
Target Type: Instance
VPC: Correct VPC

Expected:

Target Group
     │
     ├── EC2-1
     └── EC2-2

Port: 80
Protocol: HTTP

Terraform example:

resource "aws_lb_target_group" "this" {
  name     = "${var.name}-tg"
  port     = var.target_port
  protocol = "HTTP"

  vpc_id = var.vpc_id

  target_type = "instance"
}
7. Incorrect Target Group Port
Failure

Suppose the EC2 application is running on:

Port 80

but Terraform contains:

target_port = 8080

Then:

ALB
 ↓
Target Group :8080
 ↓
EC2 :80

The ALB cannot successfully reach the application on the expected port.

Fix

Change:

target_port = 80

Then run:

terraform fmt
terraform validate
terraform plan
terraform apply
8. Step 4 — Check Target Health

This is one of the most important troubleshooting steps.

Go to:

Target Groups
 ↓
day28-alb-tg
 ↓
Targets

Check the status.

Expected:

EC2-1 → Healthy
EC2-2 → Healthy

Possible failure:

EC2-1 → Unhealthy
EC2-2 → Unhealthy

or:

Initial

If targets are unhealthy, investigate:

Target Group
     │
     ├── Port
     ├── Protocol
     ├── Health Check Path
     └── Health Check Status
9. Step 5 — Check Health Check Configuration

Expected health check:

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

The ALB effectively checks:

http://EC2-IP:80/

The application should return:

HTTP 200 OK
10. Incorrect Health Check Path
Failure

Suppose Terraform contains:

path = "/health"

but the application only provides:

/

The ALB checks:

http://EC2-IP/health

The application may return:

404 Not Found

Therefore:

Target → Unhealthy
Fix

If the application serves /, use:

path = "/"

Then:

terraform fmt
terraform validate
terraform plan
terraform apply

Wait for the target to become:

Healthy
11. Step 6 — Check EC2 Instances

Go to:

EC2
 ↓
Instances

Check:

Instance State: Running
Status Checks: 2/2 checks passed

Also confirm the instances belong to:

day28-compute-asg

Expected:

EC2-1 → Running
EC2-2 → Running

If instances are missing, check the Auto Scaling Group.

12. Step 7 — Check Auto Scaling Group

Go to:

EC2
 ↓
Auto Scaling Groups
 ↓
day28-compute-asg

Check:

Minimum: 1
Desired: 2
Maximum: 3

For example:

Minimum capacity = 1
Desired capacity = 2
Maximum capacity = 3

Terraform:

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
}

Check whether the ASG is successfully launching instances.

13. Possible ASG Configuration Failure

Example:

Desired Capacity = 2
Maximum Capacity = 1

This is an invalid capacity relationship because:

min <= desired <= max

should normally be maintained.

Correct example:

Minimum = 1
Desired = 2
Maximum = 3

Fix the Terraform variables:

min_size         = 1
desired_capacity = 2
max_size         = 3

Then:

terraform plan
terraform apply
14. Step 8 — Check Security Groups

The security group architecture should be:

Internet
    │
    │ HTTP :80
    ▼
ALB Security Group
    │
    │ HTTP :80
    ▼
EC2 Security Group
    │
    ▼
EC2
ALB Security Group

The ALB should allow HTTP traffic:

ingress {
  description = "HTTP from Internet"

  from_port   = 80
  to_port     = 80
  protocol    = "tcp"

  cidr_blocks = ["0.0.0.0/0"]
}
EC2 Security Group

The EC2 application should allow HTTP from the ALB Security Group:

ingress {
  description     = "HTTP from ALB"

  from_port       = 80
  to_port         = 80
  protocol        = "tcp"

  security_groups = [
    aws_security_group.alb.id
  ]
}

This is preferable to allowing application traffic from the entire Internet.

15. Wrong Security Group Rule
Failure

Suppose the EC2 Security Group contains:

from_port = 8080
to_port   = 8080

but NGINX is listening on:

80

Then:

ALB
 │
 │ Port 80
 ▼
EC2 SG
 │
 X Port 80 blocked
 ▼
EC2

The target can become unhealthy.

Fix

Use:

from_port = 80
to_port   = 80

and allow traffic from the ALB Security Group.

16. Step 9 — Check Subnets

Verify:

ALB → Public Subnets
EC2 → Appropriate Application Subnets

Example:

VPC
│
├── Public Subnet 1
│      └── ALB
│
├── Public Subnet 2
│      └── ALB
│
├── Private Subnet 1
│      └── EC2
│
└── Private Subnet 2
       └── EC2

Check that:

ALB subnets
EC2 subnets
Security Groups
Target Group

all belong to the correct VPC.

17. Incorrect Subnet

A common problem is putting the ALB in the wrong subnet.

For an internet-facing ALB, the selected subnets should have appropriate routing through an Internet Gateway.

Example:

Public Subnet
     │
Route Table
     │
0.0.0.0/0
     │
Internet Gateway
     │
Internet

If the ALB is incorrectly placed in unsuitable subnets, external access may fail.

18. Step 10 — Check Launch Template

The Launch Template defines how replacement EC2 instances are created.

Check:

AMI
Instance Type
Security Group
User Data
Root Volume
Tags

Terraform:

resource "aws_launch_template" "this" {
  name_prefix   = "day28-compute-"
  image_id      = var.ami
  instance_type = var.instance_type

  user_data = var.user_data != null ? base64encode(var.user_data) : null

  vpc_security_group_ids = var.security_group_ids
}

If the Launch Template is incorrect, newly created instances may fail to start correctly.

19. Step 11 — Check EC2 User Data

User Data automatically configures the application when the EC2 instance starts.

Example:

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
20. User Data Failure

If NGINX is not running, connect to the EC2 instance and check:

sudo systemctl status nginx

Test:

curl http://localhost

If NGINX is not running:

sudo systemctl start nginx

Check User Data logs:

sudo cat /var/log/cloud-init-output.log

Also check:

sudo journalctl -u nginx

Possible causes:

Incorrect package command
Incorrect User Data syntax
NGINX installation failed
Incorrect file path
NGINX failed to start
21. Step 12 — Check the Application

From the EC2 instance:

curl http://localhost

Expected:

DevOps Training Application

Check the listening port:

sudo ss -tulpn | grep :80

Expected:

nginx → port 80

If nothing is listening on port 80, investigate NGINX.

22. Step 13 — Identify the Root Cause

Before changing Terraform, write down the root cause.

Example:

Problem:
Target Group was unhealthy.

Investigation:
ALB was active.
Listener was correct.
EC2 instances were running.
Security Groups were correct.
Target Group health check path was /health.

Application only served /.

Root Cause:
Incorrect Target Group health-check path.

Fix:
Changed health-check path from /health to /.

This demonstrates actual troubleshooting rather than random configuration changes.

23. Step 14 — Fix Terraform

After identifying the problem, modify the appropriate .tf file.

Example:

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

Do not manually fix the AWS Console configuration and leave Terraform incorrect.

The desired configuration should be corrected in Terraform.

24. Step 15 — Run Terraform Format

Run:

terraform fmt

Expected:

Terraform files formatted successfully.
25. Step 16 — Validate Terraform

Run:

terraform validate

Expected:

Success! The configuration is valid.

If validation fails, read the error carefully.

Check:

File
Line
Resource
Argument
Variable
Module
26. Step 17 — Run Terraform Plan

Run:

terraform plan

Review the changes carefully.

You should understand:

What will change?
Which resource will change?
Why is it changing?
Will any resource be destroyed?

Do not automatically approve a plan you do not understand.

27. Step 18 — Apply the Fix

Run:

terraform apply

Review the plan and confirm when prompted.

Terraform will update the AWS infrastructure.

Example:

Apply complete!

Resources:
    changed: 1

The exact output depends on the failure introduced by the trainer.

28. Step 19 — Verify the Infrastructure

Check the ALB:

ALB → Active

Check listener:

HTTP :80

Check Target Group:

Port: 80
Protocol: HTTP

Check targets:

EC2-1 → Healthy
EC2-2 → Healthy

Check ASG:

Desired = 2
Running = 2
29. Step 20 — Verify Application

Get the ALB DNS name:

terraform output alb_dns_name

Test:

curl http://<ALB-DNS-NAME>

Test HTTP headers:

curl -I http://<ALB-DNS-NAME>

Expected:

HTTP/1.1 200 OK

The browser should also display:

DevOps Training Application
30. Troubleshooting Decision Tree

Use this simplified process:

Application not accessible
          │
          ▼
       Check ALB
          │
          ├── Not Active
          │      ↓
          │   Check ALB configuration
          │
          ▼
      Check Listener
          │
          ├── Wrong Port/Rule
          │      ↓
          │   Fix Listener
          │
          ▼
    Check Target Group
          │
          ├── Wrong Port
          │      ↓
          │   Fix Target Port
          │
          ▼
     Check Target Health
          │
          ├── Unhealthy
          │      ↓
          │   Check Health Check
          │
          ▼
       Check EC2
          │
          ├── Not Running
          │      ↓
          │   Check ASG / Launch Template
          │
          ▼
   Check Security Groups
          │
          ├── Port blocked
          │      ↓
          │   Fix SG
          │
          ▼
      Check User Data
          │
          ├── NGINX failed
          │      ↓
          │   Fix User Data
          │
          ▼
     Check Application
          │
          ▼
      curl localhost
          │
          ▼
       Verify ALB
31. Common Failure Reference
Failure	Symptom	Check	Typical Fix
Wrong Target Group port	Targets unhealthy	Target Group port	Use port 80
Wrong SG rule	Targets unhealthy	EC2 SG	Allow HTTP from ALB SG
Wrong subnet	ALB/application inaccessible	Subnet and route table	Correct subnet
Wrong health path	Targets unhealthy	Health Check	Use /
Wrong listener	ALB doesn't forward	Listener	Forward to correct Target Group
Launch Template error	EC2 fails	Launch Template	Correct AMI/configuration
ASG issue	Wrong number of instances	ASG settings	Correct min/desired/max
User Data failure	NGINX not running	Cloud-init logs	Fix startup script
32. Important Troubleshooting Commands
Terraform
terraform fmt
terraform validate
terraform plan
terraform apply
terraform output
EC2
sudo systemctl status nginx
sudo systemctl start nginx
sudo systemctl restart nginx
Application
curl http://localhost
curl -I http://localhost
Port
sudo ss -tulpn | grep :80
User Data
sudo cat /var/log/cloud-init-output.log
NGINX logs
sudo journalctl -u nginx
33. Final Architecture After Fix
                         INTERNET
                            │
                            │ HTTP :80
                            ▼
                 ┌─────────────────────┐
                 │        ALB          │
                 │   Listener :80      │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │    Target Group     │
                 │     HTTP :80        │
                 │   Health Check /    │
                 └──────────┬──────────┘
                            │
                    ┌───────┴───────┐
                    ▼               ▼
                 EC2-1           EC2-2
                Healthy         Healthy
                    │               │
                    └───────┬───────┘
                            │
                       NGINX :80
34. Final Deliverable

Record the actual failure introduced by the trainer.

TASK 15 — TERRAFORM TROUBLESHOOTING

Date:
15-09-2026

Failure Introduced:
____________________________________________

Symptom:
____________________________________________

Step 1 — ALB:
____________________________________________

Step 2 — Listener:
____________________________________________

Step 3 — Target Group:
____________________________________________

Step 4 — Target Health:
____________________________________________

Step 5 — EC2:
____________________________________________

Step 6 — Security Group:
____________________________________________

Step 7 — User Data:
____________________________________________

Step 8 — Application:
____________________________________________

Root Cause:
____________________________________________

Terraform File Changed:
____________________________________________

Terraform Fix:
____________________________________________

terraform validate:
PASS / FAIL

terraform plan:
PASS / FAIL

terraform apply:
PASS / FAIL

Final Target Health:
EC2-1 → Healthy
EC2-2 → Healthy

Final Application Test:
HTTP 200 OK

Application:
Working / Not Working
35. Key Learning

The correct troubleshooting approach is:

Observe
   ↓
Identify symptom
   ↓
Check infrastructure layer-by-layer
   ↓
Find root cause
   ↓
Fix Terraform
   ↓
terraform fmt
   ↓
terraform validate
   ↓
terraform plan
   ↓
terraform apply
   ↓
Verify

The main lesson is:

Troubleshoot from the outside toward the application, identify the root cause before changing anything, and make the permanent fix in Terraform rather than only changing the AWS Console.