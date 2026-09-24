TASK 11 — Understand Private EC2 Architecture

For this task, do not create another EC2 instance. The objective is to understand why EC2 instances are sometimes placed in a private subnet and how they access the internet when necessary.

1. Architecture
                         INTERNET
                            │
                            ▼
                    ┌───────────────┐
                    │ Internet      │
                    │ Gateway       │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │ Public Subnet │
                    │               │
                    │ NAT Gateway   │
                    └───────┬───────┘
                            │
                            ▼
                  ┌───────────────────┐
                  │ Private Subnet    │
                  │                   │
                  │ Private EC2       │
                  │ Application       │
                  │ Server            │
                  └───────────────────┘
2. How the traffic works

A private EC2 instance does not have a direct route to the Internet Gateway.

Instead:

Private EC2
     │
     ▼
Private Route Table
     │
     │ 0.0.0.0/0 → NAT Gateway
     ▼
NAT Gateway
     │
     ▼
Internet Gateway
     │
     ▼
Internet

The important point is:

NAT Gateway allows private resources to initiate outbound internet connections without making those resources directly reachable from the internet.

For example, a private application server might need to download OS updates or access an external API.

3. Why put an application server in a private subnet?

The main reason is security.

Suppose an application server is directly in a public subnet:

Internet
   │
   ▼
Public IP
   │
   ▼
EC2

The server can potentially receive traffic directly from the internet, depending on its security-group rules.

With a private subnet:

Internet
   │
   X
   │
Private EC2

There is no direct internet route to the private EC2.

Instead, users normally reach the application through a controlled public-facing component such as:

Internet
    │
    ▼
Application Load Balancer
    │
    ▼
Private EC2

This gives you an additional security layer.

4. Common use cases
Application servers

For example:

Internet
   │
   ▼
Load Balancer
   │
   ├──────────┐
   ▼          ▼
Private EC2  Private EC2
App Server   App Server

The load balancer is public, while the application servers remain private.

Internal services

An organization might have services that should only be accessed by other internal applications.

Example:

Frontend
   │
   ▼
Private API Server
   │
   ▼
Private Internal Service

There is no reason for the internal service to have a public IP.

Databases

Databases are commonly placed in private subnets.

Example:

Application Server
        │
        ▼
   Private Database

A database normally should not be directly accessible from the public internet.

Examples include:

PostgreSQL
MySQL
SQL Server
Amazon RDS
Backend services

Backend workers and supporting services can also run privately:

Application
    │
    ▼
Backend Service
    │
    ▼
Database

These components generally don't need public IP addresses.

5. Public vs Private EC2
Feature	Public EC2	Private EC2
Public subnet	✅	❌
Public IP	Usually	Usually no
Direct route to IGW	Yes	No
Direct internet inbound access	Possible	No
Outbound internet	Through IGW	Through NAT Gateway
Typical use	Web/public-facing	Application/backend/database
Security exposure	Higher	Lower
6. Your Terraform architecture

Your current Day 27 network looks like:

                         VPC
                          │
             ┌────────────┴────────────┐
             │                         │
             ▼                         ▼
       PUBLIC SUBNET             PRIVATE SUBNET
       10.0.1.0/24               10.0.11.0/24
             │                         │
             ▼                         ▼
        EC2 / ALB                 Private EC2
             │                         │
             │                    Private Route
             │                         │
             ▼                         ▼
       Internet Gateway          NAT Gateway
                                       │
                                       ▼
                                Internet Gateway
                                       │
                                       ▼
                                   INTERNET

You currently have:

Public:
10.0.1.0/24
10.0.2.0/24

and:

Private:
10.0.11.0/24
10.0.12.0/24

So the next logical architecture is to place backend/application workloads in:

10.0.11.0/24

or:

10.0.12.0/24

rather than directly exposing them in a public subnet.

Deliverable — Notes

You can save the following as:

notes/day27-private-ec2-architecture.md

Here is a clean trainee-style explanation:

Day 27 — Private EC2 Architecture
TASK 11 — Understand Private EC2 Architecture
Objective

The objective of this task is to understand why application servers and backend services are commonly deployed in private subnets instead of public subnets.

Architecture
                     INTERNET
                        │
                        ▼
                 Internet Gateway
                        │
                        ▼
                  Public Subnet
                        │
                  NAT Gateway
                        │
                        ▼
                 Private Subnet
                        │
                        ▼
                  Private EC2
How It Works

A private EC2 instance does not have a direct route to the Internet Gateway.

For outbound internet communication, the traffic follows this path:

Private EC2
     │
     ▼
Private Route Table
     │
     ▼
NAT Gateway
     │
     ▼
Internet Gateway
     │
     ▼
Internet

The NAT Gateway is deployed in a public subnet. The private subnet uses a route such as:

0.0.0.0/0 → NAT Gateway

This allows the private EC2 instance to initiate outbound connections while keeping the instance from being directly reachable from the internet.

Why Use a Private Subnet?

Application servers are often placed in private subnets because they do not need to be directly accessible from the public internet.

A typical production architecture is:

Internet
   │
   ▼
Application Load Balancer
   │
   ▼
Private Application Servers
   │
   ▼
Private Database

The load balancer receives public traffic and forwards approved requests to application servers in private subnets.

This reduces the direct exposure of the application servers.

Common Use Cases
1. Application Servers

Application servers can run in private subnets behind a public load balancer.

Internet
   │
   ▼
Load Balancer
   │
   ├─────────────┐
   ▼             ▼
Private EC2   Private EC2
2. Internal Services

Services that are only required by other applications can remain private.

Application
     │
     ▼
Internal Service
3. Databases

Databases are commonly deployed in private subnets because they should not be directly exposed to the public internet.

Application Server
        │
        ▼
   Private Database
4. Backend Services

Backend APIs, workers and internal processing services can run in private subnets.

Frontend
   │
   ▼
Backend Service
   │
   ▼
Database
Public EC2 vs Private EC2
Feature	Public EC2	Private EC2
Public subnet	Yes	No
Public IP	Usually	Usually no
Direct IGW route	Yes	No
Direct internet exposure	Possible	No
Outbound internet	Through IGW	Through NAT Gateway
Typical use	Public-facing workloads	Applications, backend and internal services
Key Learning

A private subnet provides an additional network isolation layer.

The important concept is:

Public Subnet
     │
     └── Internet Gateway
              │
           Internet


Private Subnet
     │
     └── NAT Gateway
              │
       Internet Gateway
              │
           Internet

The private EC2 can initiate outbound internet communication through the NAT Gateway, but it does not have a direct route from the internet to the instance.

Conclusion

Private subnets are useful for workloads that should not be directly exposed to the internet. Common examples include application servers, internal services, backend services and databases.

For a production application, a common design is to keep public-facing components such as a load balancer in public subnets while keeping application servers and databases in private subnets.

This provides better network isolation and reduces unnecessary public exposure.