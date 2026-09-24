File Structure
terraform/
└── modules/
    └── vpc/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
modules/vpc/variables.tf
variable "environment" {
  description = "Environment Name"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
}

variable "availability_zones" {
  description = "List of Availability Zones"
  type        = list(string)
}

variable "public_subnet_cidrs" {
  description = "Public Subnet CIDRs"
  type        = list(string)
}

variable "private_subnet_cidrs" {
  description = "Private Subnet CIDRs"
  type        = list(string)
}
modules/vpc/main.tf
Create VPC
resource "aws_vpc" "main" {

  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}
Create Internet Gateway
resource "aws_internet_gateway" "igw" {

  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${var.environment}-igw"
  }
}
Create Public Subnets
resource "aws_subnet" "public" {

  count = length(var.public_subnet_cidrs)

  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.environment}-public-${count.index + 1}"
  }
}
Create Private Subnets
resource "aws_subnet" "private" {

  count = length(var.private_subnet_cidrs)

  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name = "${var.environment}-private-${count.index + 1}"
  }
}
Create Elastic IP for NAT Gateway
resource "aws_eip" "nat" {

  domain = "vpc"

  tags = {
    Name = "${var.environment}-nat-eip"
  }
}
Create NAT Gateway
resource "aws_nat_gateway" "nat" {

  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id

  tags = {
    Name = "${var.environment}-nat-gateway"
  }

  depends_on = [
    aws_internet_gateway.igw
  ]
}
Public Route Table
resource "aws_route_table" "public" {

  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "${var.environment}-public-rt"
  }
}
Associate Public Route Table
resource "aws_route_table_association" "public" {

  count = length(aws_subnet.public)

  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}
Private Route Table
resource "aws_route_table" "private" {

  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }

  tags = {
    Name = "${var.environment}-private-rt"
  }
}
Associate Private Route Table
resource "aws_route_table_association" "private" {

  count = length(aws_subnet.private)

  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}
modules/vpc/outputs.tf
VPC ID
output "vpc_id" {

  description = "VPC ID"

  value = aws_vpc.main.id
}
Public Subnet IDs
output "public_subnet_ids" {

  description = "Public Subnet IDs"

  value = aws_subnet.public[*].id
}
Private Subnet IDs
output "private_subnet_ids" {

  description = "Private Subnet IDs"

  value = aws_subnet.private[*].id
}
How This Module Works
VPC
│
├── Internet Gateway
│
├── Public Subnet AZ-1
├── Public Subnet AZ-2
│
├── NAT Gateway
│
├── Private Subnet AZ-1
├── Private Subnet AZ-2
│
├── Public Route Table
└── Private Route Table
Example Module Usage
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
}
Deliverable Summary

This VPC module creates:

1 VPC
2 Public Subnets
2 Private Subnets
1 Internet Gateway
1 NAT Gateway
1 Public Route Table
1 Private Route Table
Route Table Associations

Outputs returned:

vpc_id
public_subnet_ids
private_subnet_ids

This design follows AWS networking best practices and is suitable for Day 27 Terraform Networking training.