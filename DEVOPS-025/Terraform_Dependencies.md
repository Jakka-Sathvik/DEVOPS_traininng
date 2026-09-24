TASK 3 — Terraform Dependencies
Objective

Understand how Terraform determines the order in which resources are created and how dependencies work.

What is a Dependency?

A dependency exists when one resource requires another resource to be created first.

Example:

Network
   ↓
Security Group
   ↓
EC2 Instance

The EC2 instance cannot be created until the Security Group exists, and the Security Group cannot be created until the Network exists.

Implicit Dependency

Terraform automatically detects dependencies when one resource references another.

Example:

resource "aws_vpc" "main_vpc" {
  cidr_block = "10.0.0.0/16"
}
resource "aws_subnet" "public_subnet" {
  vpc_id     = aws_vpc.main_vpc.id
  cidr_block = "10.0.1.0/24"
}
resource "aws_instance" "web_server" {
  ami           = "ami-1234567890abcdef0"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public_subnet.id
}

Terraform automatically understands:

VPC
 ↓
Subnet
 ↓
EC2 Instance

No depends_on is needed.

Explicit Dependency

Sometimes resources do not directly reference each other, but one still depends on another.

In such cases, use:

depends_on = [aws_vpc.main_vpc]

Example:

resource "aws_security_group" "web_sg" {
  name   = "web-sg"
  vpc_id = aws_vpc.main_vpc.id
}
resource "aws_instance" "web_server" {
  ami           = "ami-1234567890abcdef0"
  instance_type = "t2.micro"

  depends_on = [
    aws_security_group.web_sg
  ]
}

Terraform ensures the Security Group is created before the EC2 instance.

Exercise: Resource A → Resource B → Resource C
Resource A – VPC
resource "aws_vpc" "project_vpc" {
  cidr_block = "10.0.0.0/16"
}
Resource B – Subnet
resource "aws_subnet" "project_subnet" {
  vpc_id     = aws_vpc.project_vpc.id
  cidr_block = "10.0.1.0/24"
}
Resource C – EC2 Instance
resource "aws_instance" "project_server" {
  ami           = "ami-1234567890abcdef0"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.project_subnet.id
}
Dependency Flow
Resource A (VPC)
        ↓
Resource B (Subnet)
        ↓
Resource C (EC2 Instance)

Terraform recognizes this automatically through resource references.

Verify Using Terraform Plan

Initialize Terraform:

terraform init

Validate Configuration:

terraform validate

Generate Execution Plan:

terraform plan

Expected creation order:

1. aws_vpc.project_vpc
2. aws_subnet.project_subnet
3. aws_instance.project_server
Deliverable
Resource	Dependency	Reason	Creation Order
aws_vpc.project_vpc	None	Base network resource	1
aws_subnet.project_subnet	aws_vpc.project_vpc	Subnet requires a VPC ID	2
aws_instance.project_server	aws_subnet.project_subnet	EC2 requires a Subnet ID	3
Summary
Implicit Dependency
vpc_id = aws_vpc.project_vpc.id

Terraform automatically determines the dependency.

Explicit Dependency
depends_on = [aws_security_group.web_sg]

Used when resources do not directly reference each other.

Infrastructure Flow
VPC
 ↓
Subnet
 ↓
EC2 Instance