TASK 10 — ALB Health Checks

Time: 45 Minutes

The goal of this task is to configure and verify an Application Load Balancer (ALB) health check so that the ALB sends normal traffic only to healthy EC2 instances.

1. Understand the Architecture
                         User
                           │
                           │ HTTP :80
                           ▼
                  ┌─────────────────┐
                  │       ALB       │
                  │   Listener :80  │
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │  Target Group   │
                  │ Health Check: / │
                  └───────┬─────────┘
                          │
                 ┌────────┴────────┐
                 ▼                 ▼
             EC2-1             EC2-2
             :80               :80
           Healthy            Healthy

If one instance becomes unhealthy:

                  ALB
                   │
                   ▼
             Target Group
              /          \
             /            \
            ▼              ▼
         EC2-1           EC2-2
        Healthy         Unhealthy
            │
            │
            └── Normal traffic

                     X
                 No normal
                  traffic

The ALB continues using the healthy target.

2. What Is a Health Check?

A health check is a test performed by the Target Group to determine whether an EC2 application is working.

For our application:

Protocol : HTTP
Port     : 80
Path     : /

The ALB sends a request similar to:

GET /

to the EC2 instance.

If the application returns:

HTTP 200 OK

the target is considered healthy.

Example:

ALB
 │
 │ GET /
 ▼
EC2
 │
 │ HTTP 200
 ▼
Healthy
3. Configure the Target Group Health Check

You already created the Target Group in:

day28/
└── terraform/
    └── modules/
        └── alb/
            └── main.tf

Open:

modules/alb/main.tf

Use:

resource "aws_lb_target_group" "this" {
  name     = "${var.name}-tg"
  port     = var.target_port
  protocol = "HTTP"

  vpc_id = var.vpc_id

  target_type = "instance"

  health_check {
    enabled = true

    protocol = "HTTP"
    port     = "traffic-port"
    path     = "/"

    healthy_threshold = 2

    unhealthy_threshold = 2

    timeout = 5

    interval = 30

    matcher = "200"
  }

  tags = {
    Name        = "${var.name}-target-group"
    Environment = "Training"
  }
}
4. Understand Each Health Check Setting
enabled
enabled = true

Enables health checking.

protocol
protocol = "HTTP"

The ALB uses HTTP to check the application.

port
port = "traffic-port"

This means the health check uses the target group's configured traffic port.

Our target group uses:

Port = 80

Therefore:

Health Check Port = 80
path
path = "/"

The ALB checks:

http://EC2-IP/

For example:

http://10.0.2.15/

Your NGINX application should return a successful response.

5. Healthy Threshold

We configured:

healthy_threshold = 2

This means the target needs 2 consecutive successful health checks before it becomes healthy.

Example:

Check 1 → 200 OK
Check 2 → 200 OK

        ↓

     HEALTHY
6. Unhealthy Threshold

We configured:

unhealthy_threshold = 2

This means the target needs 2 consecutive failed health checks before being marked unhealthy.

Example:

Check 1 → Failed
Check 2 → Failed

        ↓

    UNHEALTHY

The ALB then stops sending normal requests to that unhealthy target.

7. Health Check Interval

We configured:

interval = 30

The ALB performs the health check approximately every:

30 seconds

Example:

00 sec → Health Check
30 sec → Health Check
60 sec → Health Check
90 sec → Health Check
8. Timeout

We configured:

timeout = 5

The ALB waits up to approximately 5 seconds for the target to respond to the health check.

If the target doesn't respond within the timeout, that check fails.

9. Matcher

We configured:

matcher = "200"

This means:

HTTP 200

is considered successful.

Your NGINX page should therefore return:

HTTP/1.1 200 OK
10. Verify NGINX on EC2

Your EC2 instances are configured through User Data.

The script installs NGINX:

dnf update -y
dnf install -y nginx

and starts it:

systemctl enable nginx
systemctl start nginx

On an EC2 instance, check:

sudo systemctl status nginx

You want:

Active: active (running)
11. Test the Application Locally

On the EC2 instance:

curl http://localhost

You should receive your HTML page.

For example:

<h1>DevOps Training Application</h1>

You can also check only the HTTP status:

curl -I http://localhost

Expected:

HTTP/1.1 200 OK

This is important because the ALB health check is also requesting:

/

on port:

80
12. Check the Target Group

After applying Terraform:

terraform apply

Run:

terraform output

You should have your Target Group ARN.

You can also check the AWS Console:

AWS Console
   ↓
EC2
   ↓
Target Groups
   ↓
day28-alb-tg
   ↓
Targets

You should see something similar to:

Instance       Port       Health
------------------------------------
EC2-1          80         Healthy
EC2-2          80         Healthy
13. Expected Result

Your final state should be:

              ALB
               │
               │ Listener :80
               ▼
          Target Group
               │
       ┌───────┴───────┐
       │               │
       ▼               ▼
     EC2-1           EC2-2
      :80             :80
       │               │
       ▼               ▼
   HTTP 200         HTTP 200
       │               │
       ▼               ▼
    HEALTHY         HEALTHY
14. Test an Unhealthy Instance

For the training exercise, you can intentionally stop NGINX on one EC2 instance.

On EC2-1:

sudo systemctl stop nginx

Now NGINX is no longer listening on port 80.

Check:

curl -I http://localhost

It should fail.

After the configured health-check failures, the Target Group should show:

EC2-1 → Unhealthy
EC2-2 → Healthy

The architecture becomes:

                    ALB
                     │
                     ▼
                Target Group
                 /          \
                /            \
               ▼              ▼
            EC2-1           EC2-2
          UNHEALTHY         HEALTHY
               X               │
               │               │
          No normal            │
           traffic             │
                               ▼
                          Normal traffic
15. Restore the Instance

Start NGINX again:

sudo systemctl start nginx

Check:

sudo systemctl status nginx

Then:

curl -I http://localhost

Expected:

HTTP/1.1 200 OK

After successful health checks, EC2-1 should return to:

Healthy

So finally:

ALB
 │
 ▼
Target Group
 │
 ├── EC2-1 → Healthy
 │
 └── EC2-2 → Healthy
16. Run Terraform Validation

From your Terraform directory:

cd /workspaces/terraform-lab/day28/terraform

Run:

terraform fmt

Then:

terraform init

Then:

terraform validate

Then:

terraform plan

Finally:

terraform apply

Type:

yes
17. Test Through the ALB

Get the ALB DNS name:

terraform output alb_dns_name

You may get something like:

day28-alb-123456789.ap-south-1.elb.amazonaws.com

Open:

http://day28-alb-123456789.ap-south-1.elb.amazonaws.com

Expected:

DevOps Training Application

Server: EC2
Environment: Training
Status: Application is running
18. Troubleshooting

If the target shows Unhealthy, check these in order:

Check 1 — NGINX
sudo systemctl status nginx

It should be:

active (running)
Check 2 — Port 80
sudo ss -tlnp | grep :80

You should see NGINX listening on port 80.

Check 3 — Local application
curl -I http://localhost/

Expected:

HTTP/1.1 200 OK
Check 4 — Security Group

EC2 Security Group should allow:

TCP
Port: 80
Source: ALB Security Group

For a training setup, if your EC2 is directly exposed, 0.0.0.0/0 may work, but the preferred architecture is:

Internet
   │
   ▼
ALB SG
   │
   ▼
EC2 SG

The EC2 should not need to accept HTTP from the entire Internet when traffic comes through the ALB.

Check 5 — Health-check path

Make sure:

path = "/"

matches an actual working application endpoint.

Check 6 — Target port

Make sure:

Target Group → Port 80
EC2 → NGINX Port 80
19. Important Concepts to Remember
Concept	Meaning
Health Check Path	URL path checked by ALB
Health Check Port	Port checked on EC2
Healthy	Application responds successfully
Unhealthy	Health check fails repeatedly
Healthy Threshold	Successful checks required to become healthy
Unhealthy Threshold	Failed checks required to become unhealthy
Interval	Time between health checks
Timeout	Maximum time to wait for response
Matcher	HTTP status considered successful
Final architecture
                         INTERNET
                            │
                            ▼
                    ┌──────────────┐
                    │     ALB      │
                    │   :80 HTTP   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Target Group │
                    │ Health: /    │
                    └──────┬───────┘
                           │
                 ┌─────────┴─────────┐
                 ▼                   ▼
              EC2-1                EC2-2
               :80                   :80
             NGINX                  NGINX
                │                     │
             Healthy               Healthy