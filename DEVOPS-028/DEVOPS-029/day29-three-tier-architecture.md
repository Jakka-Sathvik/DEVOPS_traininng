Day 29 – Three-Tier Architecture Fundamentals
Objective

Understand the design and purpose of a Three-Tier Architecture in AWS and why it is widely used in modern cloud applications.

What is a Three-Tier Architecture?

A Three-Tier Architecture is a software architecture pattern that separates an application into three independent layers.

These layers work together to deliver a secure, scalable, and maintainable application.

The three layers are:

Presentation Layer
Application Layer
Database Layer

Each layer has a specific responsibility.

Architecture Overview
                Internet
                    │
                    ▼
        ┌─────────────────────┐
        │ Application Load    │
        │ Balancer (ALB)      │
        └─────────────────────┘
                    │
                    ▼
        ┌─────────────────────┐
        │ Auto Scaling Group  │
        │ EC2 Instances       │
        └─────────────────────┘
                    │
                    ▼
        ┌─────────────────────┐
        │ Amazon RDS          │
        │ Database            │
        └─────────────────────┘
Full User Request Flow
User
 ↓
Internet
 ↓
Application Load Balancer (ALB)
 ↓
EC2 Application Server
 ↓
Amazon RDS Database
 ↓
Response Returned
 ↓
User
Tier 1 – Presentation Layer
Components
Internet
Route 53 (Optional)
Application Load Balancer (ALB)
Purpose

The Presentation Layer is the entry point for users.

Users access the application through a web browser, mobile application, or API request.

The ALB receives incoming requests and distributes traffic across multiple EC2 instances.

Example

When a user visits:

https://myapp.com

The request reaches:

Application Load Balancer

The ALB forwards the request to a healthy application server.

Benefits
High Availability

If one EC2 instance fails, the ALB routes traffic to healthy instances.

Load Distribution

Traffic is distributed evenly.

Scalability

Supports large numbers of users.

SSL Termination

HTTPS certificates can be configured on the ALB.

Tier 2 – Application Layer
Components
EC2 Instances
Auto Scaling Group (ASG)
Application Code
Purpose

The Application Layer contains the business logic.

This layer processes user requests and communicates with the database.

Examples:

Login functionality
Product search
Order processing
User registration
Payment processing
Example Flow

User clicks:

Login

The application server:

Receives credentials
Validates input
Checks database records
Generates response
Auto Scaling Group (ASG)

The ASG automatically manages EC2 instances.

Example:

Minimum: 2
Desired: 2
Maximum: 4

If traffic increases:

2 EC2
 ↓
3 EC2
 ↓
4 EC2

If traffic decreases:

4 EC2
 ↓
3 EC2
 ↓
2 EC2
Benefits
Scalability

Automatically adds servers during heavy traffic.

Fault Tolerance

Replaces failed instances.

Cost Optimization

Removes unused resources.

Tier 3 – Database Layer
Components
Amazon RDS
MySQL
PostgreSQL
MariaDB
SQL Server
Purpose

Stores application data.

Examples:

User accounts
Orders
Products
Payments
Application settings
Example

Application server sends query:

SELECT * FROM users
WHERE email='user@example.com';

Database returns the requested data.

Why Should the Database NOT Be Accessible From the Internet?

This is one of the most important security principles in cloud architecture.

The database should never be exposed directly to public users.

Reason 1 – Security

If the database is public:

Internet
   ↓
Database

Anyone can attempt:

Unauthorized access
Password attacks
Brute force attacks
Data theft

This creates a major security risk.

Reason 2 – Network Isolation

The database should be placed in a private subnet.

Architecture:

Public Subnet
   │
   ├── ALB
   └── EC2

Private Subnet
   └── RDS

Private subnets do not allow direct internet access.

Only approved resources can communicate with the database.

Reason 3 – Reduced Attack Surface

Attack Surface means the number of points attackers can target.

Bad Architecture:

Internet
 ↓
Database

Good Architecture:

Internet
 ↓
ALB
 ↓
EC2
 ↓
RDS

Attackers cannot directly reach the database.

This significantly improves security.

Reason 4 – Application-to-Database Communication

Users should communicate only with the application.

Correct flow:

User
 ↓
Application
 ↓
Database

The application:

Validates input
Performs authentication
Applies authorization
Processes business logic

The database only responds to trusted application servers.

Reason 5 – Private Subnets

RDS should be deployed in private subnets.

Example:

VPC
│
├── Public Subnet AZ-1
│   ├── ALB
│   └── EC2
│
├── Public Subnet AZ-2
│   └── EC2
│
├── Private Subnet AZ-1
│   └── RDS
│
└── Private Subnet AZ-2
    └── RDS Standby

Benefits:

No public IP
No internet access
Better security
Better compliance
Controlled access
Security Group Communication

Recommended Security Group Design:

Internet
   │
   ▼
ALB Security Group
   │ Allow HTTP/HTTPS
   ▼
EC2 Security Group
   │ Allow traffic only from ALB SG
   ▼
RDS Security Group
   │ Allow traffic only from EC2 SG

This ensures secure communication between layers.

Real-World Example

Consider an online shopping application.

Presentation Layer
www.shop.com
ALB
Application Layer
Product Service
Order Service
Payment Service
EC2 Instances
Database Layer
Amazon RDS MySQL

Flow:

Customer
 ↓
ALB
 ↓
Application Server
 ↓
RDS
 ↓
Application Server
 ↓
Customer
Advantages of Three-Tier Architecture
Security

Database remains private.

Scalability

ASG automatically scales EC2 instances.

High Availability

Multiple servers reduce downtime.

Maintainability

Each layer can be managed independently.

Performance

ALB distributes traffic efficiently.

Reliability

Failures in one server do not impact the entire application.

Summary

Three-Tier Architecture is a cloud design pattern that separates the application into Presentation, Application, and Database layers.

User
 ↓
ALB
 ↓
EC2 / ASG
 ↓
RDS

The database should always remain in private subnets and should never be directly accessible from the internet. This improves security, reduces attack surface, enables network isolation, and ensures that all database communication passes through the application layer.

This architecture is considered a best practice for building secure, scalable, and highly available applications in AWS.