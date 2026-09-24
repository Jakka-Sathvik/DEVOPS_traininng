TASK 9 — Security Group Design
Objective

In this task, we will create a Security Group using Terraform and define only the network ports required for the application.

A Security Group acts as a virtual firewall for AWS resources such as EC2 instances.

The main security principle for this task is:

Allow only the traffic that is actually required.

We should avoid blindly allowing:

ALL ports
ALL protocols
0.0.0.0/0
1. Understand the Architecture
                    Internet
                       │
                       │
                HTTP / HTTPS
                       │
                       ▼
              ┌─────────────────┐
              │ Security Group  │
              │                 │
              │ 80  HTTP        │
              │ 443 HTTPS       │
              │ 22  Restricted  │
              └────────┬────────┘
                       │
                       ▼
                  Application
                     EC2

The Security Group decides whether the incoming traffic should be allowed.

2. Security Group Rules

For our training application, we need:

Source	Port	Protocol	Purpose
Internet/Application users	80	TCP	HTTP
Internet/Application users	443	TCP	HTTPS
Restricted admin source	22	TCP	SSH administration

The SSH rule should not blindly use:

0.0.0.0/0

Instead, use a trusted administrative IP/CIDR.

Example:

YOUR_ADMIN_IP/32

/32 represents a single IPv4 address.

3. Create Security Group Variables

Open:

terraform/modules/vpc/variables.tf

Add a variable for the administrative CIDR:

variable "admin_cidr" {
  description = "CIDR block allowed to access SSH"
  type        = string
}
4. Pass the Admin CIDR from the Root Module

In the root:

terraform/main.tf

add:

module "vpc" {
  source = "./modules/vpc"

  environment = "dev"

  vpc_cidr = "10.0.0.0/16"

  availability_zones = [
    "ap-south-1a",
    "ap-south-1b"
  ]

  public_subnet_cidrs = [
    "10.0.1.0/24",
    "10.0.2.0/24"
  ]

  private_subnet_cidrs = [
    "10.0.11.0/24",
    "10.0.12.0/24"
  ]

  admin_cidr = "YOUR_PUBLIC_IP/32"
}

Replace:

YOUR_PUBLIC_IP

with your actual administrative public IP.

For example, if your trusted IP is:

203.0.113.10

the rule would be:

203.0.113.10/32

Do not copy the example IP above as your real IP.

5. Create the Security Group

In:

terraform/modules/vpc/main.tf

add:

resource "aws_security_group" "application" {
  name        = "${var.environment}-application-sg"
  description = "Security group for application EC2 instances"
  vpc_id      = aws_vpc.main.id

  tags = {
    Name        = "${var.environment}-application-sg"
    Environment = var.environment
  }
}

This creates the Security Group inside our VPC.

6. Add HTTP Rule

Allow HTTP traffic on port 80.

resource "aws_vpc_security_group_ingress_rule" "http" {
  security_group_id = aws_security_group.application.id

  cidr_ipv4   = "0.0.0.0/0"
  from_port   = 80
  to_port     = 80
  ip_protocol = "tcp"

  description = "Allow HTTP traffic"
}
Why is 0.0.0.0/0 acceptable here?

HTTP is normally a public application service.

Therefore:

Internet
   │
   │ TCP 80
   ▼
Application

can be intentionally exposed.

This is different from exposing SSH.

7. Add HTTPS Rule

Allow HTTPS traffic on port 443.

resource "aws_vpc_security_group_ingress_rule" "https" {
  security_group_id = aws_security_group.application.id

  cidr_ipv4   = "0.0.0.0/0"
  from_port   = 443
  to_port     = 443
  ip_protocol = "tcp"

  description = "Allow HTTPS traffic"
}

HTTPS is commonly public because users need to access the application securely from the internet.

8. Add Restricted SSH Rule

SSH uses port 22.

Do not use:

0.0.0.0/0

Instead:

resource "aws_vpc_security_group_ingress_rule" "ssh" {
  security_group_id = aws_security_group.application.id

  cidr_ipv4   = var.admin_cidr
  from_port   = 22
  to_port     = 22
  ip_protocol = "tcp"

  description = "Allow SSH from restricted administrative IP"
}

Now the architecture is:

Internet
   │
   ├── TCP 80 ────► EC2
   │
   ├── TCP 443 ───► EC2
   │
   └── TCP 22
          │
          ▼
     Admin IP only
9. Configure Outbound Traffic

For a simple training application, we can explicitly allow outbound IPv4 traffic:

resource "aws_vpc_security_group_egress_rule" "all_ipv4" {
  security_group_id = aws_security_group.application.id

  cidr_ipv4   = "0.0.0.0/0"
  ip_protocol = "-1"

  description = "Allow outbound IPv4 traffic"
}

This means the EC2 instance can initiate outbound connections.

Important distinction

The requirement is to avoid blindly allowing inbound access to everything.

This rule is an outbound rule and is common for basic EC2 application environments, although production environments may restrict egress further according to application requirements.

10. Complete Security Group

Your Security Group configuration can now look like:

resource "aws_security_group" "application" {
  name        = "${var.environment}-application-sg"
  description = "Security group for application EC2 instances"
  vpc_id      = aws_vpc.main.id

  tags = {
    Name        = "${var.environment}-application-sg"
    Environment = var.environment
  }
}

resource "aws_vpc_security_group_ingress_rule" "http" {
  security_group_id = aws_security_group.application.id

  cidr_ipv4   = "0.0.0.0/0"
  from_port   = 80
  to_port     = 80
  ip_protocol = "tcp"

  description = "Allow HTTP traffic"
}

resource "aws_vpc_security_group_ingress_rule" "https" {
  security_group_id = aws_security_group.application.id

  cidr_ipv4   = "0.0.0.0/0"
  from_port   = 443
  to_port     = 443
  ip_protocol = "tcp"

  description = "Allow HTTPS traffic"
}

resource "aws_vpc_security_group_ingress_rule" "ssh" {
  security_group_id = aws_security_group.application.id

  cidr_ipv4   = var.admin_cidr
  from_port   = 22
  to_port     = 22
  ip_protocol = "tcp"

  description = "Allow SSH from restricted administrative IP"
}

resource "aws_vpc_security_group_egress_rule" "all_ipv4" {
  security_group_id = aws_security_group.application.id

  cidr_ipv4   = "0.0.0.0/0"
  ip_protocol = "-1"

  description = "Allow outbound IPv4 traffic"
}
11. Understand Each Rule
HTTP
Source:    0.0.0.0/0
Port:      80
Protocol:  TCP
Purpose:   Public web traffic
HTTPS
Source:    0.0.0.0/0
Port:      443
Protocol:  TCP
Purpose:   Secure web traffic
SSH
Source:    Admin IP/32
Port:      22
Protocol:  TCP
Purpose:   Administrative access
Outbound
Destination: 0.0.0.0/0
Protocol:    All IPv4
Purpose:     Outbound application traffic
12. Security Group Diagram
                         INTERNET
                            │
              ┌─────────────┼─────────────┐
              │             │             │
            TCP 80        TCP 443       TCP 22
              │             │             │
              │             │       Restricted
              │             │       Admin IP
              │             │             │
              └─────────────┼─────────────┘
                            ▼
                   ┌─────────────────┐
                   │  Application SG │
                   │                 │
                   │ HTTP  → 80      │
                   │ HTTPS → 443     │
                   │ SSH   → Admin   │
                   └────────┬────────┘
                            │
                            ▼
                       Application
                          EC2
13. Run Terraform Formatting

From the root Terraform directory:

terraform fmt
14. Validate Configuration

Run:

terraform validate

Expected:

Success! The configuration is valid.
15. Generate Terraform Plan

Run:

terraform plan

Look for:

+ aws_security_group.application

+ aws_vpc_security_group_ingress_rule.http

+ aws_vpc_security_group_ingress_rule.https

+ aws_vpc_security_group_ingress_rule.ssh

+ aws_vpc_security_group_egress_rule.all_ipv4
16. Security Checklist

Before applying, check:

✓ HTTP port 80 is allowed
✓ HTTPS port 443 is allowed
✓ SSH port 22 is restricted
✓ No ALL inbound ports
✓ No ALL inbound protocols
✓ SSH is not open to 0.0.0.0/0
✓ Every rule has a clear purpose
✓ Terraform validate succeeds
✓ Terraform plan reviewed
Deliverable — Documented Security Group Rule Set
Security Group: dev-application-sg

Inbound Rules
─────────────────────────────────────────────
Source              Port    Protocol   Purpose
0.0.0.0/0            80     TCP        HTTP
0.0.0.0/0           443     TCP        HTTPS
Admin IP/32           22     TCP        SSH


Outbound Rules
─────────────────────────────────────────────
Destination         Protocol             Purpose
0.0.0.0/0            All IPv4             Application outbound traffic
Final Commands
terraform fmt
terraform validate
terraform plan