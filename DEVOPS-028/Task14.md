TASK 14 — Auto Scaling Failure Simulation

The goal of this exercise is to safely remove one EC2 instance managed by the Auto Scaling Group (ASG) and observe how the ASG automatically launches a replacement.

Architecture
                    Internet
                       │
                       ▼
                  Application
                 Load Balancer
                       │
                       ▼
                 Target Group
                  /         \
                 ▼           ▼
              EC2-1        EC2-2
                │             │
                └─────┬───────┘
                      │
                      ▼
              Auto Scaling Group
                      │
              Desired Capacity = 2
1. Verify the Healthy State

Before causing any failure, make sure both instances are healthy.

Run:

cd /workspaces/terraform-lab/day28/terraform

terraform output

Check the AWS Console:

EC2 → Auto Scaling Groups → day28-compute-asg

Verify:

Desired capacity: 2
Minimum capacity: 1
Maximum capacity: 3

Then go to:

EC2 → Target Groups → day28-alb-tg → Targets

Expected:

EC2-1 → Healthy
EC2-2 → Healthy

You should have:

Healthy State

EC2-1 → Running → Healthy
EC2-2 → Running → Healthy
2. Test the Application Before Failure

Get the ALB DNS name:

terraform output alb_dns_name

Example:

day28-alb-123456789.ap-south-1.elb.amazonaws.com

Test:

curl http://<ALB-DNS-NAME>

Expected:

DevOps Training Application
Server: EC2
Environment: Training
Status: Application is running

This confirms that the application is working before the failure simulation.

3. Identify an EC2 Instance Managed by the ASG

In AWS Console:

EC2 → Instances

Find the two instances created by:

day28-compute-asg

For example:

EC2-1
Instance ID: i-0123456789abcdef0

EC2-2
Instance ID: i-0abcdef123456789

Important: Do not terminate an unrelated EC2 instance.

4. Safely Simulate Instance Failure

For this training exercise, the easiest method is to terminate one instance that is managed by the ASG.

For example:

Terminate EC2-1

In AWS Console:

EC2
 ↓
Instances
 ↓
Select EC2-1
 ↓
Instance state
 ↓
Terminate instance

Confirm termination.

Why is this safe for the exercise?

The instance is controlled by the Auto Scaling Group.

The ASG knows:

Desired Capacity = 2

After one instance is terminated:

Running instances = 1
Desired instances = 2

Therefore, the ASG should launch a replacement.

5. Observe the Failure

Immediately after termination, you may see:

EC2-1 → Terminating
EC2-2 → Running

Then:

EC2-1 → Terminated
EC2-2 → Running

The ASG detects that the number of running instances is below the desired capacity.

Architecture
Before failure:

ASG
 │
 ├── EC2-1 ✓
 └── EC2-2 ✓

Desired = 2
Running = 2

After terminating EC2-1:

ASG
 │
 ├── EC2-1 ✗
 └── EC2-2 ✓

Desired = 2
Running = 1
6. ASG Launches Replacement

The Auto Scaling Group now attempts to restore:

Desired Capacity = 2

You should see a new instance being launched.

ASG
 │
 ├── EC2-2 ✓
 └── EC2-3 → Launching

Eventually:

ASG
 │
 ├── EC2-2 ✓
 └── EC2-3 ✓

The replacement instance receives its configuration from the Launch Template:

Launch Template
      │
      ├── AMI
      ├── Instance Type
      ├── Security Group
      ├── User Data
      └── Storage
             │
             ▼
       Replacement EC2
7. Verify User Data

Because the Launch Template contains your User Data, the new EC2 should automatically install and start NGINX.

On the replacement instance:

sudo systemctl status nginx

Expected:

Active: active (running)

Test:

curl http://localhost

Expected:

DevOps Training Application

If troubleshooting is required:

sudo cat /var/log/cloud-init-output.log
8. Target Group Health Check

The replacement instance should automatically be registered with the Target Group because the ASG is configured with:

target_group_arns = [
  var.target_group_arn
]

Go to:

EC2
 ↓
Target Groups
 ↓
day28-alb-tg
 ↓
Targets

Initially you may see:

EC2-2 → Healthy
EC2-3 → Initial

After the health checks succeed:

EC2-2 → Healthy
EC2-3 → Healthy

The health check is:

HTTP
Port: 80
Path: /
Expected: HTTP 200
9. Verify Application Recovery

Once the replacement becomes healthy:

curl http://<ALB-DNS-NAME>

The ALB should successfully serve the application again.

Flow:

Client
   │
   ▼
ALB
   │
   ▼
Target Group
   │
   ├── EC2-2 ✓
   │
   └── EC2-3 ✓
           │
           ▼
         NGINX

The terminated instance is no longer receiving traffic.

10. Complete Failure-Recovery Flow

The complete exercise should look like this:

             HEALTHY
                │
        ┌───────┴───────┐
        ▼               ▼
      EC2-1           EC2-2
    Healthy          Healthy
        │
        │ Terminate EC2-1
        ▼
      FAILURE
        │
        ▼
ASG detects capacity = 1
        │
        ▼
Desired capacity = 2
        │
        ▼
Replacement EC2 launched
        │
        ▼
User Data executes
        │
        ▼
NGINX starts
        │
        ▼
EC2 registered with Target Group
        │
        ▼
Health Check
        │
        ▼
     Healthy
        │
        ▼
Traffic resumes
11. What You Should Observe
Stage	Expected Result
Initial state	2 healthy EC2 instances
Failure	One EC2 terminated
Detection	ASG sees capacity below desired
Replacement	New EC2 launched
Configuration	Launch Template used
Application	User Data installs/starts NGINX
Registration	New EC2 added to Target Group
Health Check	/ returns HTTP 200
Recovery	Replacement becomes Healthy
Traffic	ALB sends traffic to healthy instances
12. Deliverable

Record your actual results in your training notes:

TASK 14 — AUTO SCALING FAILURE SIMULATION

Date:
15-09-2026

Auto Scaling Group:
day28-compute-asg

Initial Desired Capacity:
2

Initial Instances:
EC2-1 → Healthy
EC2-2 → Healthy


FAILURE
--------------------------------

Instance terminated:
____________________________

Instance ID:
____________________________


DETECTION
--------------------------------

ASG detected capacity reduction:
Yes / No

Running instances after failure:
____________________________

Desired capacity:
____________________________


REPLACEMENT
--------------------------------

Replacement instance launched:
Yes / No

Replacement Instance ID:
____________________________

Launch Template used:
Yes / No


HEALTH CHECK
--------------------------------

Target Group:
day28-alb-tg

Health Check Path:
/ 

Health Check Port:
80

Replacement status:
Initial → Healthy


RECOVERY
--------------------------------

Final running instances:
____________________________

Final healthy targets:
____________________________

Application accessible through ALB:
Yes / No

ALB DNS:
____________________________


RESULT
--------------------------------

Failure:
One ASG-managed EC2 instance was terminated.

Detection:
The Auto Scaling Group detected that running capacity
was below the desired capacity.

Replacement:
The ASG launched a replacement EC2 instance using the
Launch Template.

Health Check:
The replacement instance passed the Target Group HTTP
health check on port 80.

Recovery:
The replacement instance became healthy and the ALB
continued serving application traffic.
Key concept to remember

Auto Scaling does not prevent an EC2 failure. It automatically helps recover from the failure by maintaining the desired number of instances.

EC2 Failure
     ↓
ASG Detects
     ↓
Launch Replacement
     ↓
Target Group Registration
     ↓
Health Check
     ↓
Healthy
     ↓
ALB Sends Traffic

For this exercise, do not manually create the replacement EC2. The important learning objective is to observe the ASG automatically replacing the failed instance.