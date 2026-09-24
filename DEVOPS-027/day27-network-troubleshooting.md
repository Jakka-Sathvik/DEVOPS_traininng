TASK 14 — Networking Troubleshooting
1. Objective

The objective of this task is to troubleshoot a controlled AWS networking problem created in the Terraform infrastructure.

The trainee must identify the networking symptom, investigate each networking component, find the root cause, fix the Terraform configuration, and verify the infrastructure after applying the fix.

The troubleshooting process is:

Identify Symptom
       ↓
Check EC2
       ↓
Check Subnet
       ↓
Check Route Table
       ↓
Check Internet Gateway
       ↓
Check NAT Gateway
       ↓
Check Security Group
       ↓
Identify Root Cause
       ↓
Fix Terraform
       ↓
terraform plan
       ↓
terraform apply
       ↓
Verify
2. Possible Networking Problems

The trainer may intentionally introduce one of the following problems:

Wrong subnet association
Incorrect route table
Missing route
Incorrect Security Group rule
Incorrect subnet CIDR
EC2 launched into wrong subnet
NAT Gateway route missing

The trainee should not immediately change Terraform configuration.

First, the symptom must be understood and the infrastructure must be investigated.

3. Training Architecture

The expected training architecture is:

                    INTERNET
                       |
                       |
               Internet Gateway
                       |
              +--------+--------+
              |                 |
        Public Subnet      Public Subnet
        10.0.1.0/24        10.0.2.0/24
              |
          NAT Gateway
              |
       Private Route Table
              |
       +------+------+
       |             |
 Private Subnet   Private Subnet
 10.0.11.0/24     10.0.12.0/24

The public EC2 architecture is:

Internet
   |
Internet Gateway
   |
Public Route Table
   |
Public Subnet
   |
Security Group
   |
EC2

The private EC2 architecture is:

Private EC2
    |
Private Route Table
    |
NAT Gateway
    |
Public Subnet
    |
Internet Gateway
    |
Internet
4. Problem

For this troubleshooting exercise, assume the trainer introduces the following controlled problem:

EC2 instance is running,
but the application cannot be reached from the internet.

The EC2 instance is expected to be in a public subnet.

However, the EC2 instance has been associated with an incorrect route table.

Example:

EC2
 |
Public Subnet
 |
Incorrect Route Table
 |
No route to Internet Gateway

Because the subnet does not have the correct public route, internet connectivity does not work as expected.

5. Symptom

The first step is to identify what the user or application is experiencing.

Possible symptoms include:

Application is not reachable
HTTP request fails
Browser cannot connect
EC2 has no working internet connectivity
Public IP exists but application is unavailable

For example:

curl http://<EC2_PUBLIC_IP>

Possible result:

Connection timed out

The important point is:

A running EC2 instance does not automatically mean that network connectivity is configured correctly.

6. Step 1 — Check EC2

First verify the EC2 instance.

Check:

Instance State
Instance Type
VPC
Subnet
Private IP
Public IP
Security Group

Expected example:

Instance:
dev-application-server

State:
Running

Instance Type:
t3.micro

VPC:
10.0.0.0/16

Subnet:
10.0.1.0/24

Public IP:
Present

Terraform command:

terraform output

Also check Terraform state:

terraform state list

The EC2 resource should appear.

7. Step 2 — Check Subnet

Next check which subnet the EC2 instance is using.

Expected public subnet:

10.0.1.0/24

or:

10.0.2.0/24

The EC2 should not accidentally be placed in:

10.0.11.0/24

or:

10.0.12.0/24

because those are private subnets.

Terraform configuration should contain something similar to:

subnet_id = module.vpc.public_subnet_ids[0]

This means the EC2 is using the first public subnet returned by the VPC module.

8. Step 3 — Check Route Table

This is one of the most important troubleshooting steps.

A public subnet should have a route:

Destination:
0.0.0.0/0

Target:
Internet Gateway

Expected architecture:

Public Subnet
      |
Public Route Table
      |
0.0.0.0/0
      |
Internet Gateway

If the route is missing:

Public Subnet
      |
Public Route Table
      |
No Internet Route

the EC2 instance cannot properly communicate with the internet.

9. Check Public Route Table Terraform Configuration

The public route table should contain a route similar to:

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id  = aws_internet_gateway.igw.id
  }

  tags = {
    Name        = "${var.environment}-public-route-table"
    Environment = var.environment
  }
}

The public subnet must also be associated with this route table:

resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)

  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}
10. Step 4 — Check Internet Gateway

The VPC should have an Internet Gateway.

Example:

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name        = "${var.environment}-igw"
    Environment = var.environment
  }
}

The Internet Gateway must be attached to the correct VPC.

Expected:

VPC
 |
Internet Gateway

Check in AWS:

VPC Console
   ↓
Internet Gateways
   ↓
Check attachment

The Internet Gateway should be attached to:

dev VPC
10.0.0.0/16
11. Step 5 — Check NAT Gateway

NAT Gateway is mainly required for private subnet outbound internet access.

Expected architecture:

Private EC2
     |
Private Route Table
     |
0.0.0.0/0
     |
NAT Gateway
     |
Public Subnet
     |
Internet Gateway
     |
Internet

The private route table should contain:

Destination: 0.0.0.0/0
Target: NAT Gateway

Terraform example:

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }

  tags = {
    Name        = "${var.environment}-private-route-table"
    Environment = var.environment
  }
}

If the NAT route is missing:

Private EC2
     |
Private Route Table
     |
No NAT Route
     X
Internet

The private EC2 will not have normal outbound internet connectivity.

12. Step 6 — Check Security Group

A Security Group can also cause connectivity problems.

For an application server, HTTP should normally be allowed:

resource "aws_vpc_security_group_ingress_rule" "http" {
  security_group_id = aws_security_group.application.id

  cidr_ipv4   = "0.0.0.0/0"
  from_port   = 80
  to_port     = 80
  ip_protocol = "tcp"

  description = "Allow HTTP traffic"
}

HTTPS:

resource "aws_vpc_security_group_ingress_rule" "https" {
  security_group_id = aws_security_group.application.id

  cidr_ipv4   = "0.0.0.0/0"
  from_port   = 443
  to_port     = 443
  ip_protocol = "tcp"

  description = "Allow HTTPS traffic"
}

SSH should be restricted:

resource "aws_vpc_security_group_ingress_rule" "ssh" {
  security_group_id = aws_security_group.application.id

  cidr_ipv4   = var.admin_cidr
  from_port   = 22
  to_port     = 22
  ip_protocol = "tcp"

  description = "Allow SSH from restricted administrative IP"
}

Do not use:

SSH → 0.0.0.0/0

unless it is specifically required for a controlled training scenario.

13. Step 7 — Check Subnet CIDR

Verify that the subnet CIDRs are correct.

Expected configuration:

VPC
10.0.0.0/16

Public:
10.0.1.0/24
10.0.2.0/24

Private:
10.0.11.0/24
10.0.12.0/24

Example:

public_subnet_cidrs = [
  "10.0.1.0/24",
  "10.0.2.0/24"
]

private_subnet_cidrs = [
  "10.0.11.0/24",
  "10.0.12.0/24"
]

A wrong CIDR can cause unexpected routing or subnet placement problems.

14. Step 8 — Identify Root Cause

After checking all components, identify the actual problem.

Example investigation:

EC2:
Running ✓

EC2 Public IP:
Present ✓

Subnet:
Public subnet ✓

Security Group:
HTTP 80 allowed ✓

Internet Gateway:
Attached ✓

Route Table:
Public subnet associated ✓

Default Route:
Missing ✗

Root cause:

The public route table does not contain:

0.0.0.0/0 → Internet Gateway

Therefore:

EC2
 ↓
Public Subnet
 ↓
Public Route Table
 ↓
NO INTERNET ROUTE

The problem is a routing configuration issue.

15. Step 9 — Fix Terraform

Do not manually fix the AWS console and leave Terraform unchanged.

The Terraform configuration must be corrected because Terraform should remain the source of truth.

Add or correct the public route:

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name        = "${var.environment}-public-route-table"
    Environment = var.environment
  }
}

Make sure the public subnet is associated with the public route table:

resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)

  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}
16. Step 10 — Run Terraform Format

After changing Terraform:

terraform fmt

Expected:

Terraform files formatted successfully.

If Terraform prints the changed file names, that is also normal.

17. Step 11 — Run Terraform Validate

Run:

terraform validate

Expected:

Success! The configuration is valid.

If validation fails:

Read the error
   ↓
Identify file
   ↓
Identify resource
   ↓
Fix syntax/configuration
   ↓
Run terraform validate again
18. Step 12 — Run Terraform Plan

Run:

terraform plan

Terraform should show the infrastructure changes required to correct the problem.

Look for changes related to:

aws_route_table
aws_route_table_association
aws_route

The plan should be reviewed before applying.

Important:

terraform plan

does not modify AWS infrastructure.

It only shows what Terraform intends to change.

19. Step 13 — Apply the Fix

Once the plan is correct:

terraform apply

Terraform will ask:

Do you want to perform these actions?

Enter:

yes

Terraform will apply the networking correction.

20. Step 14 — Verify

After the apply completes:

terraform output

Check the EC2 information.

Then verify in AWS Console.

EC2

Check:

Instance State → Running
VPC → Correct VPC
Subnet → Correct public subnet
Public IP → Present
Security Group → Correct SG
Route Table

Check:

Destination:
0.0.0.0/0

Target:
Internet Gateway
Internet Gateway

Check:

IGW → Attached to correct VPC
Security Group

Check:

HTTP 80 → Allowed
HTTPS 443 → Allowed
SSH 22 → Restricted
21. Application Connectivity Test

If a web server is running on the EC2 instance, test:

curl http://<EC2_PUBLIC_IP>

or open:

http://<EC2_PUBLIC_IP>

Expected result:

Application responds successfully

If the application responds, the network troubleshooting fix is successful.

22. Troubleshooting Decision Table
Symptom	Check	Possible Root Cause
EC2 has no public IP	EC2 configuration	Public IP not associated
EC2 is in private subnet	Subnet	Wrong subnet selected
Internet unavailable	Route table	Missing default route
Public subnet cannot reach internet	IGW	IGW missing/not attached
HTTP unavailable	Security Group	Port 80 blocked
HTTPS unavailable	Security Group	Port 443 blocked
SSH unavailable	Security Group	Port 22 not allowed from admin CIDR
Private EC2 cannot access internet	Private route table	NAT route missing
Private EC2 cannot reach NAT	NAT Gateway	NAT unavailable or wrong subnet
Unexpected networking behavior	CIDR	Incorrect subnet CIDR
Public subnet behaves like private	Route association	Wrong route table associated
23. Complete Troubleshooting Checklist

Use this checklist during the exercise:

[ ] Identify symptom

[ ] Check EC2 instance

[ ] Check EC2 subnet

[ ] Check VPC

[ ] Check subnet CIDR

[ ] Check route table association

[ ] Check public route

[ ] Check Internet Gateway

[ ] Check NAT Gateway

[ ] Check private route

[ ] Check Security Group

[ ] Identify root cause

[ ] Fix Terraform configuration

[ ] terraform fmt

[ ] terraform validate

[ ] terraform plan

[ ] terraform apply

[ ] Verify AWS infrastructure

[ ] Test application connectivity
24. Important Terraform Principle

Do not make a manual AWS Console change and consider the task completed.

For example, if a route is missing, do not only add it manually in AWS.

Instead:

Find Problem
     ↓
Fix Terraform Code
     ↓
terraform plan
     ↓
terraform apply
     ↓
AWS Updated

This keeps Terraform as the Infrastructure as Code source of truth.

25. Final Architecture

After the issue is fixed, the expected public architecture is:

                    INTERNET
                       |
                       ↓
              Internet Gateway
                       |
                       ↓
              Public Route Table
                0.0.0.0/0 → IGW
                       |
                       ↓
                Public Subnet
                 10.0.1.0/24
                       |
                       ↓
                Security Group
                  Port 80/443
                       |
                       ↓
                    EC2

Private networking:

                 Private EC2
                      |
                      ↓
              Private Route Table
                0.0.0.0/0 → NAT
                      |
                      ↓
                 NAT Gateway
                      |
                      ↓
                 Public Subnet
                      |
                      ↓
              Internet Gateway
                      |
                      ↓
                   INTERNET
26. Task 14 Result

The networking troubleshooting exercise demonstrates the ability to:

Identify networking symptoms
Inspect EC2 configuration
Verify subnet placement
Check route table associations
Check Internet Gateway configuration
Check NAT Gateway configuration
Verify Security Group rules
Identify networking root causes
Correct Terraform configuration
Run terraform plan
Run terraform apply
Verify AWS infrastructure
Test application connectivity
Final Troubleshooting Flow
PROBLEM
   ↓
SYMPTOM
   ↓
INVESTIGATION
   ↓
ROOT CAUSE
   ↓
TERRAFORM FIX
   ↓
terraform fmt
   ↓
terraform validate
   ↓
terraform plan
   ↓
terraform apply
   ↓
AWS VERIFICATION
   ↓
CONNECTIVITY TEST
   ↓
RESOLVED

Deliverable:

notes/day27-network-troubleshooting.md