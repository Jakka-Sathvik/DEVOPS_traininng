Day 27 - VPC CIDR Design

## Objective

The objective of this task is to design a simple AWS VPC network and divide the VPC CIDR block into smaller subnet ranges. This helps organize resources and supports high availability across multiple Availability Zones.

---

# VPC Configuration

VPC CIDR Block:

10.0.0.0/16

This CIDR block provides a large private network range that can be divided into multiple subnets.

Range:

10.0.0.0 - 10.0.255.255

---

# Subnet Design

The VPC is divided into four subnets.

## Public Subnets

### Public Subnet AZ-1

CIDR:

10.0.1.0/24

Purpose:

- Web Servers
- Bastion Hosts
- Load Balancers

Availability Zone:

ap-south-1a

---

### Public Subnet AZ-2

CIDR:

10.0.2.0/24

Purpose:

- Web Servers
- Load Balancers

Availability Zone:

ap-south-1b

---

## Private Subnets

### Private Subnet AZ-1

CIDR:

10.0.11.0/24

Purpose:

- Application Servers
- Internal Services

Availability Zone:

ap-south-1a

---

### Private Subnet AZ-2

CIDR:

10.0.12.0/24

Purpose:

- Databases
- Internal Applications

Availability Zone:

ap-south-1b

---

# CIDR Allocation Diagram

                    VPC
                10.0.0.0/16
                      │
      ┌───────────────┼───────────────┐
      │               │               │
      │               │               │
10.0.1.0/24     10.0.2.0/24    Public Subnets
 Public AZ-1     Public AZ-2

10.0.11.0/24    10.0.12.0/24
 Private AZ-1    Private AZ-2

---

# Network Architecture Diagram

                          Internet
                              │
                              │
                    Internet Gateway
                              │
               ┌──────────────┴──────────────┐
               │                             │
       Public Subnet AZ-1           Public Subnet AZ-2
         10.0.1.0/24                 10.0.2.0/24
               │                             │
               └──────────────┬──────────────┘
                              │
                              │
                      Application Layer
                              │
               ┌──────────────┴──────────────┐
               │                             │
      Private Subnet AZ-1          Private Subnet AZ-2
        10.0.11.0/24                10.0.12.0/24
               │                             │
               └──────────────┬──────────────┘
                              │
                           Database

---

# CIDR Breakdown

| Network Component | CIDR Block | Availability Zone |
|------------------|------------|-------------------|
| VPC | 10.0.0.0/16 | Entire Region |
| Public Subnet 1 | 10.0.1.0/24 | ap-south-1a |
| Public Subnet 2 | 10.0.2.0/24 | ap-south-1b |
| Private Subnet 1 | 10.0.11.0/24 | ap-south-1a |
| Private Subnet 2 | 10.0.12.0/24 | ap-south-1b |

---

# Summary

A VPC with CIDR block 10.0.0.0/16 has been created and divided into four /24 subnets.

Public Subnets:
- 10.0.1.0/24
- 10.0.2.0/24

Private Subnets:
- 10.0.11.0/24
- 10.0.12.0/24

This design supports:
- High Availability
- Network Segmentation
- Secure Infrastructure
- Future Scalability

The CIDR plan will be used in upcoming Terraform networking projects.