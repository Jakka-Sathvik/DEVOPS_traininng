Day 29 – Amazon RDS Fundamentals
Objective

Learn the fundamentals of Amazon Relational Database Service (RDS), understand how managed databases work in AWS, and explore the key concepts required to deploy and maintain relational databases in cloud environments.

What is Amazon RDS?

Amazon RDS (Relational Database Service) is a fully managed database service provided by AWS.

It allows organizations to create, operate, scale, and manage relational databases without manually handling the underlying infrastructure.

Instead of installing a database server on an EC2 instance and managing the operating system, storage, backups, patches, and monitoring, AWS manages these tasks automatically.

Traditional Database Deployment

Before cloud-managed databases, organizations typically installed databases on physical servers or virtual machines.

Example:

Application
     │
     ▼
Database Server
     │
     ▼
Operating System
     │
     ▼
Storage

Responsibilities included:

OS installation
Database installation
Patching
Backups
Monitoring
High availability
Storage management
Disaster recovery

This required significant administrative effort.

Amazon RDS Architecture
EC2 Application Server
          │
          │ Database Connection
          ▼
      Amazon RDS
          │
          ▼
     Database Engine

The application communicates directly with the RDS endpoint.

AWS manages:

Database software
Operating system
Storage
Backups
Maintenance
Monitoring
Supported Database Engines

Amazon RDS supports multiple relational database engines.

PostgreSQL

Popular open-source relational database.

Features:

Advanced SQL support
High performance
JSON support
Extensibility
MySQL

One of the most widely used relational databases.

Features:

Easy to use
Reliable
Popular for web applications
Other Supported Engines
PostgreSQL
MySQL
MariaDB
Oracle Database
Microsoft SQL Server
Amazon Aurora
RDS Concept 1 – Managed Relational Database

RDS is a managed service.

AWS performs infrastructure management tasks such as:

Provisioning
Patching
Monitoring
Backups
Recovery

Benefits:

Reduced operational burden
Faster deployment
Improved reliability
RDS Concept 2 – Database Instance

An RDS Database Instance is the database server created inside AWS.

Example:

Database Instance
 ├── CPU
 ├── Memory
 ├── Storage
 └── Database Engine

Example instance classes:

db.t3.micro
db.t3.small
db.t3.medium

The instance size determines database performance.

RDS Concept 3 – Database Storage

RDS stores database files on AWS-managed storage.

Storage options include:

General Purpose SSD (gp3)
Provisioned IOPS SSD
Magnetic Storage (Legacy)

Storage can often be increased without rebuilding the database.

Benefits:

High durability
Reliable performance
Scalability
RDS Concept 4 – Automated Backups

RDS automatically creates backups.

Backup contents:

Database data
Transaction logs
Configuration

Example:

Daily Backup
     +
Transaction Logs
     =
Point-in-Time Recovery

Benefits:

Data protection
Recovery from accidental deletion
Disaster recovery
RDS Concept 5 – Point-in-Time Recovery (PITR)

Allows restoring the database to a specific point in time.

Example:

09:00 AM
10:00 AM
11:00 AM
11:15 AM Accident

Restore database to:

11:14 AM

This minimizes data loss.

RDS Concept 6 – Maintenance Windows

AWS performs maintenance activities during scheduled windows.

Examples:

Minor version upgrades
Security updates
System improvements

Benefits:

Predictable updates
Reduced operational effort
Improved security
RDS Concept 7 – High Availability (Multi-AZ)

RDS supports Multi-AZ deployments.

Architecture:

Primary RDS
      │
      │ Replication
      ▼
Standby RDS

If the primary database fails:

Primary Failure
       ↓
Automatic Failover
       ↓
Standby Becomes Primary

Benefits:

Improved uptime
Automatic failover
Better reliability
RDS Concept 8 – Security Groups

Security Groups act as virtual firewalls.

Example:

EC2 Security Group
         │
         ▼
RDS Security Group

Recommended rule:

Allow PostgreSQL (5432)
Only from EC2 Security Group

or

Allow MySQL (3306)
Only from EC2 Security Group

Never expose the database to the internet unless absolutely required.

RDS Concept 9 – Private Subnets

Best practice is to place RDS inside private subnets.

Architecture:

Public Subnet
 ├── ALB
 └── EC2

Private Subnet
 └── RDS

Benefits:

No public IP
Better security
Reduced attack surface
RDS Concept 10 – Database Endpoint

Applications connect to RDS using an endpoint.

Example:

mydb.xxxxx.ap-south-1.rds.amazonaws.com

Application configuration:

Host     = RDS Endpoint
Port     = 5432
Database = trainingdb
User     = admin

Benefits:

Simplified connectivity
Consistent access point
RDS Concept 11 – Monitoring

Amazon RDS integrates with Amazon CloudWatch.

Metrics include:

CPU Utilization
Memory Usage
Database Connections
Storage Usage
Read Operations
Write Operations

Benefits:

Performance visibility
Alerting
Troubleshooting
RDS Concept 12 – Scaling

RDS supports vertical scaling.

Example:

db.t3.micro
      ↓
db.t3.small
      ↓
db.t3.medium

Storage can also be increased as database requirements grow.

Benefits:

Better performance
Support for growing applications
Self-Managed Database vs Managed RDS
Feature	Self-Managed Database	Amazon RDS
Server Setup	Manual	AWS Managed
OS Management	Manual	AWS Managed
Database Installation	Manual	AWS Managed
Backups	Manual	Automated
Monitoring	Manual Setup	Built-in
Patching	Manual	Managed
High Availability	Complex Setup	Multi-AZ
Scaling	Manual	Easier
Maintenance	Manual	AWS Managed
Operational Effort	High	Low
Application-to-Database Communication

Example Architecture:

User
  │
  ▼
ALB
  │
  ▼
EC2 Application
  │
  │ PostgreSQL/MySQL Connection
  ▼
Amazon RDS

Flow:

User sends request.
Application receives request.
Application queries database.
RDS returns data.
Application sends response to user.

The user never communicates directly with the database.

Why Use Amazon RDS?
Simplicity

AWS manages database infrastructure.

Reliability

Supports backups and failover.

Security

Works with VPCs, private subnets, and Security Groups.

Availability

Supports Multi-AZ deployments.

Scalability

Supports storage and compute scaling.

Automation

Reduces administrative overhead.

Summary

Amazon RDS is a fully managed relational database service that allows organizations to run PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, and Aurora databases without managing the underlying infrastructure.

Key RDS Concepts Covered:

Managed Relational Database
Database Instance
Storage
Automated Backups
Point-in-Time Recovery
Maintenance Windows
Multi-AZ High Availability
Security Groups
Private Subnets
Database Endpoints
Monitoring
Scaling

Amazon RDS simplifies database administration while providing security, availability, scalability, and reliability for modern cloud applications.