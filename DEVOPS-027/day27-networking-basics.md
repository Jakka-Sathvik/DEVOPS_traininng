Day 27 - AWS Networking Basics

## Introduction

Before creating infrastructure using Terraform, it is important to understand the basic AWS networking components. These networking services help us design secure, scalable, and highly available cloud environments.

AWS networking mainly consists of VPCs, Subnets, Route Tables, Internet Gateways, NAT Gateways, Security Groups, and Availability Zones.

---

# 1. VPC (Virtual Private Cloud)

A VPC is a logically isolated virtual network inside AWS where we can launch and manage AWS resources.

Think of a VPC as our own private data center inside AWS.

Example:

VPC CIDR Block:

10.0.0.0/16

This means the VPC can contain IP addresses from:

10.0.0.0 to 10.0.255.255

Benefits of VPC:

- Network isolation
- Better security
- Full control over IP addressing
- Supports public and private networks
- Can host multiple AWS resources

Diagram:

Internet
    |
    |
+------------------+
|       VPC        |
|   10.0.0.0/16    |
+------------------+

---

# 2. CIDR (Classless Inter-Domain Routing)

CIDR defines the IP address range available within a network.

Format:

IP Address / Prefix

Example:

10.0.0.0/16

Here:

- Network Address = 10.0.0.0
- Prefix Length = 16

Common CIDR Examples:

| CIDR | Number of IPs |
|--------|-------------|
| /16 | 65,536 |
| /24 | 256 |
| /28 | 16 |

CIDR helps divide large networks into smaller subnetworks.

---

# 3. Subnet

A subnet is a smaller network created inside a VPC.

Example:

VPC:

10.0.0.0/16

Subnets:

10.0.1.0/24

10.0.2.0/24

10.0.3.0/24

Benefits:

- Better organization
- Improved security
- Network segmentation
- High availability

Diagram:

VPC 10.0.0.0/16
|
|-- Subnet A 10.0.1.0/24
|
|-- Subnet B 10.0.2.0/24
|
|-- Subnet C 10.0.3.0/24

---

# 4. Public Subnet

A public subnet is a subnet that has internet access through an Internet Gateway.

Resources inside a public subnet can:

- Access the internet
- Receive traffic from the internet

Examples:

- Web Servers
- Load Balancers
- Bastion Hosts

Diagram:

Internet
   |
Internet Gateway
   |
Public Subnet
   |
EC2 Web Server

Example CIDR:

10.0.1.0/24

---

# 5. Private Subnet

A private subnet does not allow direct inbound access from the internet.

Resources inside private subnets are protected from external users.

Examples:

- Application Servers
- Databases
- Internal Services

Benefits:

- Enhanced security
- Reduced attack surface
- Better compliance

Diagram:

Private Subnet
      |
Database Server

Example CIDR:

10.0.2.0/24

---

# 6. Internet Gateway (IGW)

An Internet Gateway enables communication between resources in a VPC and the internet.

Functions:

- Allows inbound internet traffic
- Allows outbound internet traffic
- Attached directly to a VPC

Diagram:

Internet
    |
Internet Gateway
    |
VPC

Without an Internet Gateway, resources inside the VPC cannot communicate with the internet.

---

# 7. Route Table

A Route Table contains rules that determine where network traffic should be directed.

Example Route Table:

| Destination | Target |
|------------|---------|
| 10.0.0.0/16 | Local |
| 0.0.0.0/0 | Internet Gateway |

Explanation:

- Local route allows communication inside the VPC.
- 0.0.0.0/0 sends traffic to the internet.

Diagram:

Subnet
   |
Route Table
   |
Internet Gateway

Each subnet must be associated with a route table.

---

# 8. NAT Gateway

A NAT Gateway allows resources in private subnets to access the internet without allowing internet users to directly access those resources.

Purpose:

- Software updates
- Package downloads
- API calls

Example:

Private EC2 Instance needs internet access.

Traffic Flow:

Private EC2
      |
Private Subnet
      |
NAT Gateway
      |
Internet Gateway
      |
Internet

Benefits:

- Improved security
- Controlled internet access
- No direct public exposure

---

# 9. Security Group

A Security Group acts as a virtual firewall for AWS resources.

It controls:

- Inbound traffic
- Outbound traffic

Example Rules:

Inbound:

| Port | Protocol | Purpose |
|--------|----------|---------|
| 22 | TCP | SSH |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |

Outbound:

Allow all traffic.

Example:

Web Server Security Group

Allow:
- SSH (22)
- HTTP (80)
- HTTPS (443)

Block:
- All other inbound traffic

Benefits:

- Instance-level security
- Easy rule management
- Stateful firewall

---

# 10. Availability Zone (AZ)

An Availability Zone is an isolated data center within an AWS Region.

Example Region:

ap-south-1 (Mumbai)

Availability Zones:

- ap-south-1a
- ap-south-1b
- ap-south-1c

Benefits:

- High Availability
- Fault Tolerance
- Disaster Recovery

Example:

Web Server → AZ-A

Database → AZ-B

If one Availability Zone fails, resources in another zone can continue working.

Diagram:

AWS Region (Mumbai)

|-- AZ-A
|     |
|     Web Server
|
|-- AZ-B
      |
      Database

---

# Summary

AWS networking provides the foundation for building secure and scalable cloud infrastructure.

Key Components:

- VPC → Private virtual network
- CIDR → IP address range
- Subnet → Smaller network inside VPC
- Public Subnet → Internet accessible
- Private Subnet → No direct internet access
- Internet Gateway → Connects VPC to internet
- Route Table → Controls traffic routing
- NAT Gateway → Outbound internet for private resources
- Security Group → Virtual firewall
- Availability Zone → Isolated data center

Understanding these concepts is essential before creating AWS infrastructure using Terraform.