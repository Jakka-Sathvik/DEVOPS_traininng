TASK 6 — Create Security Group
1. Objective

Create an AWS Security Group using Terraform.

The architecture is:

Internet
   │
   ▼
Security Group
   │
   ▼
EC2

The Security Group acts like a virtual firewall for AWS resources.

2. What is a Security Group?

A Security Group controls network traffic allowed to and from AWS resources such as EC2 instances.

For this training task, we will allow:

SSH  → Port 22
HTTP → Port 80

We will not allow unnecessary ports.

3. Security Rules
Protocol	Port	Purpose	Source
TCP	22	SSH	Your IP only
TCP	80	HTTP	0.0.0.0/0
All	—	Unnecessary traffic	❌ Not allowed
Why restrict SSH?

SSH provides administrative access to a server. Opening:

0.0.0.0/0

means the entire internet can attempt to connect to port 22.

Instead, use:

YOUR_PUBLIC_IP/32

For example:

203.0.113.10/32

/32 means one specific IPv4 address.

4. Project Structure

You can add the Security Group to your previous network project:

terraform-network-lab/
│
├── provider.tf
├── network.tf
├── security-group.tf
└── outputs.tf
5. Create security-group.tf

Add:

resource "aws_security_group" "training_sg" {
  name        = "training-security-group"
  description = "Security group for Terraform training"
  vpc_id      = aws_vpc.training_vpc.id

  # SSH - restricted to administrator IP
  ingress {
    description = "SSH from administrator IP"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["YOUR_PUBLIC_IP/32"]
  }

  # HTTP - web traffic
  ingress {
    description = "HTTP web traffic"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow outbound traffic
  egress {
    description = "Allow outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "training-security-group"
  }
}
Important

Replace:

YOUR_PUBLIC_IP

with your actual public IP.

For example:

cidr_blocks = ["203.0.113.10/32"]

Do not use the example IP above as your real IP.

If you don't currently need SSH, an even safer training configuration is to omit port 22 entirely and allow only HTTP.

6. Understand the Security Group

This part:

resource "aws_security_group" "training_sg" {

means:

Resource Type
      ↓
aws_security_group

Resource Name
      ↓
training_sg
7. VPC Dependency

This line is important:

vpc_id = aws_vpc.training_vpc.id

The Security Group is associated with the VPC.

Terraform therefore understands:

VPC
 ↓
Security Group

This is an implicit dependency.

8. SSH Rule
ingress {
  from_port   = 22
  to_port     = 22
  protocol    = "tcp"
  cidr_blocks = ["YOUR_PUBLIC_IP/32"]
}

This means:

Port:     22
Protocol: TCP
Purpose:  SSH
Source:   Your IP

Only your specified IP can connect to SSH.

9. HTTP Rule
ingress {
  from_port   = 80
  to_port     = 80
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}

This means:

Port:     80
Protocol: TCP
Purpose:  HTTP
Source:   Internet

HTTP is intentionally public because a web server normally needs users to access it.

10. Outbound Rule
egress {
  from_port   = 0
  to_port     = 0
  protocol    = "-1"
  cidr_blocks = ["0.0.0.0/0"]
}

This allows outbound connections.

For this basic training exercise, that's acceptable because the main security requirement is to avoid unnecessarily exposing inbound sensitive ports.

11. Add Security Group Output

Open:

outputs.tf

Add:

output "security_group_id" {
  value = aws_security_group.training_sg.id
}

You can also output the name:

output "security_group_name" {
  value = aws_security_group.training_sg.name
}
12. Format the Code

Run:

terraform fmt
13. Validate

Run:

terraform validate

Expected:

Success! The configuration is valid.
14. Check Terraform Plan

Run:

terraform plan

Look for:

aws_security_group.training_sg

If you are using the previous network configuration, Terraform should understand the dependency:

VPC
 ↓
Security Group
15. Apply

Run:

terraform apply

Terraform will ask:

Do you want to perform these actions?

Enter:

yes

After successful creation, you should see something similar to:

Outputs:

security_group_id = "sg-xxxxxxxx"
security_group_name = "training-security-group"

The actual Security Group ID will be different.

16. Final Architecture
                    Internet
                       │
              ┌────────┴────────┐
              │                 │
            SSH 22            HTTP 80
         Your IP only        Public
              │                 │
              └────────┬────────┘
                       ▼
              Security Group
                       │
                       ▼
                      EC2
17. Deliverable
Security Group
Name:
training-security-group
Allowed inbound traffic
SSH  → 22 → Your IP only
HTTP → 80 → Internet
Not allowed
❌ Random ports
❌ All inbound traffic
❌ SSH from 0.0.0.0/0
Terraform resource
resource "aws_security_group" "training_sg"
Dependency
aws_vpc.training_vpc
        ↓
aws_security_group.training_sg
Verification
terraform fmt
terraform validate
terraform plan
terraform apply
✅ TASK 6 Complete

You have now created a Terraform-managed AWS Security Group with controlled inbound access and demonstrated how the Security Group depends on the VPC.

One important cleanup step: when you're finished with the lab, run:

terraform destroy

This removes the training resources and helps prevent unexpected AWS charges.