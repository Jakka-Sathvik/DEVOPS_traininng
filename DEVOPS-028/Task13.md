TASK 13 — Test Application Through ALB

Time: 30 Minutes

In this task, you will verify that the complete architecture is working:

Client
  ↓
ALB
  ↓
Listener :80
  ↓
Target Group
  ↓
Healthy EC2
  ↓
NGINX
  ↓
Web Application
1. Make Sure Terraform Is Applied

Go to your Day 28 Terraform directory:

cd /workspaces/terraform-lab/day28/terraform

Run:

terraform fmt
terraform validate
terraform plan

If the plan is correct:

terraform apply

Enter:

yes

Wait until Terraform finishes successfully.

2. Get the ALB DNS Name

Run:

terraform output alb_dns_name

Example:

day28-alb-123456789.ap-south-1.elb.amazonaws.com

This is your ALB DNS name.

You can also check:

terraform output

Look for:

alb_dns_name
3. Test the ALB from the Browser

Open:

http://<ALB-DNS-NAME>

For example:

http://day28-alb-123456789.ap-south-1.elb.amazonaws.com

You should see your application:

DevOps Training Application

Server: EC2
Environment: Training
Status: Application is running

The important point is that you are not accessing the EC2 public IP directly.

You are accessing:

Browser
   ↓
ALB DNS
   ↓
ALB
4. Test Using curl

You can also test from your terminal.

curl http://<ALB-DNS-NAME>

Example:

curl http://day28-alb-123456789.ap-south-1.elb.amazonaws.com

Expected response:

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
5. Check HTTP Status

Run:

curl -I http://<ALB-DNS-NAME>

Expected:

HTTP/1.1 200 OK

This confirms that:

ALB
 ↓
Target Group
 ↓
Healthy EC2
 ↓
NGINX

is working.

6. Verify the Target Group

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

You should see something like:

Instance       Port       Health
------------------------------------
EC2-1          80         Healthy
EC2-2          80         Healthy

The expected result is:

EC2-1 → Healthy
EC2-2 → Healthy
7. Verify the Health Check

Your Target Group should have:

Protocol: HTTP
Port: 80
Path: /

The ALB performs:

GET /

against the EC2 instances.

Expected:

EC2-1 → HTTP 200 → Healthy
EC2-2 → HTTP 200 → Healthy
8. Verify the EC2 Instances

Go to:

AWS Console
   ↓
EC2
   ↓
Instances

You should see the instances created by your ASG.

Example:

Instance       State       ASG
----------------------------------------
EC2-1          Running     day28-compute-asg
EC2-2          Running     day28-compute-asg

Check that:

State = Running

and that they belong to:

day28-compute-asg
9. Verify the Auto Scaling Group

Go to:

EC2
   ↓
Auto Scaling Groups
   ↓
day28-compute-asg

Check:

Minimum capacity: 1
Desired capacity: 2
Maximum capacity: 3

You should currently have:

Desired = 2
Running = 2
10. Verify the ALB Listener

Go to:

EC2
   ↓
Load Balancers
   ↓
day28-alb
   ↓
Listeners

You should see:

Protocol: HTTP
Port: 80
Default action: Forward to Target Group

The traffic flow is:

Client
  │
  │ HTTP :80
  ▼
ALB
  │
  │ Listener :80
  ▼
Target Group
11. Test Multiple Requests

Run:

for i in {1..10}; do
  curl -s http://<ALB-DNS-NAME>
  echo
done

If your application page identifies the backend instance, you may see different responses as the ALB distributes requests.

However, don't expect every refresh to alternate perfectly between EC2-1 and EC2-2. ALB routing behavior can involve connection reuse and other factors.

For a clear demonstration, make each server return a unique hostname.

12. Better Way to Identify Different EC2 Instances

Modify your User Data so the application displays the EC2 hostname.

For example:

#!/bin/bash

dnf update -y
dnf install -y nginx

HOSTNAME=$(hostname)

cat > /usr/share/nginx/html/index.html <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>DevOps Training Application</title>
</head>
<body>
    <h1>DevOps Training Application</h1>
    <p>Server: EC2</p>
    <p>Hostname: $HOSTNAME</p>
    <p>Environment: Training</p>
    <p>Status: Application is running</p>
</body>
</html>
EOF

systemctl enable nginx
systemctl start nginx

Then the response could show:

DevOps Training Application

Server: EC2
Hostname: ip-10-0-11-25
Environment: Training
Status: Application is running

Another instance might show:

Hostname: ip-10-0-12-37

This makes backend distribution easier to demonstrate.

Remember: changing User Data doesn't normally rerun it on an already-created EC2 instance. For this training change, you may need to replace the instances through the ASG/Launch Template process.

13. Test the Complete Flow

Your final test should prove this:

                         CLIENT
                           │
                           │ HTTP
                           ▼
                    ┌──────────────┐
                    │     ALB      │
                    │    :80       │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Target Group │
                    │              │
                    │ Health: /    │
                    └──────┬───────┘
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                  EC2-1         EC2-2
                Healthy        Healthy
                    │             │
                    └──────┬──────┘
                           ▼
                         NGINX
                           │
                           ▼
                     APPLICATION
14. Troubleshooting — If ALB Doesn't Open
Problem 1 — Target is Unhealthy

Check:

Target Group
   ↓
Targets

If:

EC2 → Unhealthy

check NGINX:

sudo systemctl status nginx

Then:

curl -I http://localhost

Expected:

HTTP/1.1 200 OK
Problem 2 — Security Group

Check:

ALB-SG

It should allow:

TCP
Port 80
Source: 0.0.0.0/0

Check EC2-SG:

TCP
Port 80
Source: ALB-SG
Problem 3 — Wrong Target Port

Check Target Group:

Target Port = 80

Check NGINX:

Port = 80

They must match.

Problem 4 — Wrong Health Check Path

Your health check should be:

Protocol: HTTP
Port: 80
Path: /

Test:

curl -I http://localhost/
Problem 5 — ALB Has No Internet Connectivity

For an internet-facing ALB, verify its subnets have the appropriate route through an Internet Gateway.

Conceptually:

Internet
   │
   ▼
Internet Gateway
   │
   ▼
Public Subnet
   │
   ▼
ALB
15. Capture the Required Deliverables

Create a simple record like this:

TASK 13 — ALB APPLICATION TEST

ALB DNS Name:
____________________________________________

Target Group:
____________________________________________

Target Health:

EC2-1 → Healthy
EC2-2 → Healthy

EC2 Instances:

EC2-1 → Running
EC2-2 → Running

HTTP Test:

URL:
http://<ALB-DNS-NAME>

HTTP Status:
200 OK

Application Response:
DevOps Training Application
16. Final Verification Checklist
ALB
 ALB is running
 ALB is internet-facing
 Listener is HTTP :80
 ALB DNS name obtained
Target Group
 Target Group exists
 Health check path is /
 Health check port is 80
 EC2-1 is Healthy
 EC2-2 is Healthy
EC2
 EC2-1 is Running
 EC2-2 is Running
 NGINX is running
 Application responds on port 80
Application
 ALB DNS opens successfully
 curl returns HTTP 200
 Application page is displayed
 Backend instances can be identified if required
Final Result

The successful test proves:

Client
  ↓
ALB DNS
  ↓
Application Load Balancer
  ↓
Listener :80
  ↓
Target Group
  ↓
Healthy EC2
  ↓
NGINX
  ↓
Web Application

Deliverable: capture the ALB DNS name, Target Group health status, running EC2 instances, and successful HTTP 200 response as evidence for Task 13.