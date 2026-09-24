TASK 5 — Create Multiple Availability Zones
Objective

In AWS, production environments should not depend on a single Availability Zone (AZ). If one AZ experiences a failure, resources running in another AZ can continue serving users. This design is known as a Multi-AZ Architecture and is an important part of building highly available and fault-tolerant applications.

1. What is an Availability Zone?

An Availability Zone (AZ) is one or more physically separate data centers within an AWS Region.

Example Region:

ap-south-1 (Mumbai)

Availability Zones:

ap-south-1a
ap-south-1b
ap-south-1c

Each AZ has:

Independent power
Independent networking
Independent cooling systems

This reduces the chance that a single failure impacts the entire region.

2. What is High Availability?

High Availability (HA) means applications remain available even when part of the infrastructure fails.

Without High Availability:

VPC
 │
 AZ-1
 │
 Web Server

AZ-1 Failure
      ↓
Application Down

With High Availability:

VPC
 │
 ├── AZ-1
 │     └── Web Server
 │
 └── AZ-2
       └── Web Server

AZ-1 Failure
      ↓
AZ-2 Continues Serving Traffic

Benefits:

Reduced downtime
Better reliability
Improved user experience
Production-ready architecture
3. Subnet and Availability Zone Relationship

A subnet belongs to only one Availability Zone.

Example:

Subnet	CIDR	Availability Zone
Public Subnet 1	10.0.1.0/24	ap-south-1a
Public Subnet 2	10.0.2.0/24	ap-south-1b
Private Subnet 1	10.0.11.0/24	ap-south-1a
Private Subnet 2	10.0.12.0/24	ap-south-1b
4. Multi-AZ Architecture

Network Design:

                         VPC
                     10.0.0.0/16
                           │
        ┌──────────────────┴──────────────────┐
        │                                     │
        │                                     │
     AZ-1                                 AZ-2
  ap-south-1a                          ap-south-1b
        │                                     │
 ┌──────┴──────┐                     ┌────────┴───────┐
 │             │                     │                │
 │             │                     │                │
Public      Private               Public         Private
Subnet      Subnet                Subnet         Subnet
10.0.1.0    10.0.11.0             10.0.2.0       10.0.12.0
5. Terraform Configuration

The VPC module should receive two Availability Zones.

main.tf
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
6. Public Subnets Across Multiple AZs

Terraform creates:

Public Subnet 1
CIDR: 10.0.1.0/24
AZ: ap-south-1a

Public Subnet 2
CIDR: 10.0.2.0/24
AZ: ap-south-1b

Resource Example:

resource "aws_subnet" "public" {

  count = length(var.public_subnet_cidrs)

  vpc_id            = aws_vpc.main.id
  cidr_block        = var.public_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]

  map_public_ip_on_launch = true
}
7. Private Subnets Across Multiple AZs

Terraform creates:

Private Subnet 1
CIDR: 10.0.11.0/24
AZ: ap-south-1a

Private Subnet 2
CIDR: 10.0.12.0/24
AZ: ap-south-1b

Resource Example:

resource "aws_subnet" "private" {

  count = length(var.private_subnet_cidrs)

  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]
}
8. Verification Commands

Format configuration:

terraform fmt

Validate configuration:

terraform validate

Generate execution plan:

terraform plan

Verify that Terraform shows:

aws_subnet.public[0]
Availability Zone = ap-south-1a

aws_subnet.public[1]
Availability Zone = ap-south-1b

aws_subnet.private[0]
Availability Zone = ap-south-1a

aws_subnet.private[1]
Availability Zone = ap-south-1b