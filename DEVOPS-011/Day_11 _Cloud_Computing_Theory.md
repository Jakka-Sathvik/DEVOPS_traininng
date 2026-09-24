Day 11 — Cloud Computing Theory

Training: DevOps Engineer Training
Topic: Cloud Computing Fundamentals
Day: 11
Trainee: Sathvik Jakka

1. What is Cloud Computing?

Cloud computing is the delivery of computing resources and IT services over the internet instead of requiring an organization to purchase and maintain all computing infrastructure physically.

These resources can include:

Virtual machines
Servers
Storage
Databases
Networking
Applications
Security services
Monitoring
Development platforms
Data processing services

Traditionally, a company had to purchase physical servers, install them in a data center, configure networking, install operating systems, and maintain the hardware.

With cloud computing, companies can obtain these resources from cloud providers and use them whenever required.

Simple Example

Suppose a company needs a server to host a website.

Traditional approach
Company
   ↓
Purchase Physical Server
   ↓
Install Server in Data Center
   ↓
Install Operating System
   ↓
Configure Network
   ↓
Deploy Application
Cloud approach
Company
   ↓
Cloud Provider
   ↓
Create Virtual Server
   ↓
Configure Network
   ↓
Deploy Application

The cloud approach allows organizations to provision infrastructure much faster.

Cloud computing commonly follows a pay-as-you-go model. Instead of purchasing a large amount of hardware in advance, an organization can consume resources according to its requirements.

2. Why Do Companies Use Cloud?

Companies use cloud infrastructure because it provides flexibility, scalability, availability, and access to a large range of IT services.

2.1 Reduced Infrastructure Investment

Companies do not always need to purchase large amounts of physical hardware.

For example, instead of purchasing ten physical servers, a company can provision virtual servers in the cloud according to its requirements.

This can reduce the initial capital expenditure associated with building a data center.

2.2 Scalability

Cloud infrastructure allows organizations to increase or decrease resources according to workload.

For example:

Normal Traffic
      ↓
2 Servers

High Traffic
      ↓
10 Servers

After the traffic decreases, resources can be reduced again.

This is particularly useful for applications with unpredictable workloads.

2.3 Faster Deployment

Cloud resources can often be created within minutes.

A DevOps engineer can use tools such as:

Terraform
CloudFormation
AWS CLI
AWS Management Console
CI/CD pipelines

to provision infrastructure.

For example:

Terraform Code
      ↓
AWS API
      ↓
Infrastructure
      ↓
Application Deployment
2.4 Global Availability

Major cloud providers operate infrastructure in multiple geographical regions.

Organizations can deploy applications closer to their users.

For example:

Users in India
      ↓
AWS Region in India

Users in Europe
      ↓
European AWS Region

This can help reduce network latency and support global applications.

2.5 Managed Services

Cloud providers offer many managed services.

Examples include:

Managed databases
Object storage
Load balancers
Monitoring
DNS
Queues
Serverless computing
Container platforms

The company can use these services without managing every underlying hardware component itself.

3. What is AWS?

AWS stands for Amazon Web Services.

AWS is a cloud computing platform provided by Amazon. It offers a large collection of infrastructure and managed services that organizations can use to build, deploy, and operate applications.

Some important AWS services include:

AWS Service	Purpose
EC2	Virtual servers
S3	Object storage
VPC	Virtual networking
RDS	Managed relational databases
IAM	Identity and access management
CloudWatch	Monitoring and logging
Route 53	DNS service
Elastic Load Balancing	Load balancing
Lambda	Serverless computing
ECR	Container image registry
AWS EC2 Example

Amazon EC2 provides virtual servers.

A simplified architecture can be:

Developer
    ↓
AWS
    ↓
EC2 Instance
    ↓
Operating System
    ↓
Docker
    ↓
Application

A DevOps engineer can configure an EC2 instance and deploy a Dockerized application on it.

AWS S3 Example

Amazon S3 is an object storage service.

It can be used to store:

Images
Documents
Backups
Logs
Application files
Static website content

Example:

Application
     ↓
AWS S3
     ↓
Object Storage
4. What is Infrastructure as a Service?

Infrastructure as a Service, commonly called IaaS, provides fundamental computing infrastructure through the cloud.

IaaS generally includes resources such as:

Virtual machines
Storage
Networking
Firewalls/security controls
IP addresses

The cloud provider manages the underlying physical infrastructure while the customer manages much of the software stack.

IaaS Responsibility Model
Cloud Provider
      │
      ├── Physical Data Center
      ├── Physical Servers
      ├── Networking Hardware
      └── Storage Hardware
               │
               ↓
Customer
      │
      ├── Operating System
      ├── Applications
      ├── Runtime
      └── Data
Example

Amazon EC2 is an example of an IaaS-oriented service.

A company can create an EC2 instance and then manage:

Operating system
Installed packages
Docker
Application
Configuration
IaaS Example
AWS EC2
     ↓
Ubuntu
     ↓
Docker
     ↓
Container
     ↓
Application

IaaS provides significant control but also requires more administration.

5. What is Platform as a Service?

Platform as a Service, or PaaS, provides a managed application platform.

The cloud provider manages more of the underlying infrastructure and runtime environment, allowing developers to focus primarily on application development and deployment.

The provider may manage:

Servers
Operating system
Runtime
Infrastructure
Scaling mechanisms

The developer generally focuses on:

Application code
Configuration
Data
Simplified Model
Cloud Provider
      │
      ├── Hardware
      ├── Networking
      ├── Operating System
      └── Runtime
             │
             ↓
        Developer
             │
             └── Application Code
Example

A managed application platform can allow a developer to deploy an application without manually creating and maintaining a server.

The developer may provide:

Application Code
      ↓
Platform
      ↓
Running Application

PaaS can reduce infrastructure management work compared with IaaS.

6. What is Software as a Service?

Software as a Service, or SaaS, provides complete software applications to users over the internet.

The service provider manages almost the entire technology stack.

The user normally only needs to access the application.

Examples include:

Gmail
Microsoft 365
Salesforce
Slack
Online collaboration applications
SaaS Model
Cloud Provider
      │
      ├── Hardware
      ├── Network
      ├── Operating System
      ├── Runtime
      ├── Application
      └── Data Platform
             │
             ↓
           User

Users generally do not need to manage the underlying servers or operating systems.

For example, when using an online email service, the user does not need to install and maintain the email server hardware.

7. IaaS vs PaaS vs SaaS

The three service models can be compared as follows:

Feature	IaaS	PaaS	SaaS
Infrastructure	Provider	Provider	Provider
Operating System	Customer	Provider	Provider
Runtime	Customer	Provider	Provider
Application	Customer	Customer	Provider
Data	Customer	Customer	Usually shared responsibility
Management effort	High	Medium	Low
User control	High	Medium	Low
Simple Memory Trick
IaaS → Infrastructure
PaaS → Platform
SaaS → Software

Another way to understand it:

IaaS
You manage more.

PaaS
Provider manages more.

SaaS
Provider manages almost everything.
8. On-Premise vs Cloud
On-Premise

In an on-premise environment, the organization owns or directly manages its physical infrastructure.

Example:

Company Data Center
       │
       ├── Physical Servers
       ├── Storage
       ├── Network Devices
       ├── Firewalls
       └── Applications

The organization is responsible for:

Hardware
Data center
Electricity
Cooling
Networking
Hardware replacement
Operating systems
Applications
Security controls
Maintenance
Cloud

In a cloud environment, infrastructure is provided through a cloud provider.

Example:

Company
   ↓
Cloud Provider
   ↓
Virtual Infrastructure
   ↓
Application

The provider manages the underlying physical infrastructure.

The customer manages the resources and services according to the selected cloud service model.

On-Premise vs Cloud Comparison
Feature	On-Premise	Cloud
Hardware	Company owns/manages	Cloud provider
Initial investment	Usually high	Usually lower upfront
Scalability	Requires hardware planning	Can scale more easily
Deployment	Usually slower	Usually faster
Maintenance	Company	Shared/provider depending on service
Physical data center	Company	Cloud provider
Global deployment	More difficult	Easier
Resource provisioning	Manual/limited	On-demand
Cost model	Capital expenditure + operations	Usage-based options
Flexibility	Lower	Higher

Cloud does not automatically mean lower cost in every situation. Poorly managed cloud resources can become expensive.

9. Advantages of Cloud Infrastructure
9.1 Scalability

Cloud resources can be increased or decreased based on demand.

Low Demand
   ↓
Few Resources

High Demand
   ↓
More Resources
9.2 Elasticity

Cloud environments can dynamically adapt to workload changes.

For example, an application may use:

2 instances

during normal traffic and scale to:

10 instances

during peak traffic.

9.3 High Availability

Cloud platforms provide features that can help applications remain available even when individual infrastructure components fail.

Organizations can deploy applications across multiple availability zones or regions depending on their requirements.

9.4 Faster Provisioning

Infrastructure can be created quickly.

For example:

Terraform
    ↓
AWS API
    ↓
VPC
    ↓
Subnet
    ↓
EC2
    ↓
Application

This is especially useful in DevOps environments.

9.5 Automation

Cloud infrastructure works well with automation tools.

Examples:

Terraform
Ansible
AWS CLI
CI/CD pipelines
Infrastructure as Code

Automation improves repeatability and reduces manual configuration.

9.6 Global Reach

Cloud providers have infrastructure in multiple geographical locations.

Organizations can deploy applications closer to customers.

9.7 Backup and Disaster Recovery

Cloud storage and managed services can be used to implement:

Backups
Replication
Disaster recovery
Data retention

This can improve an organization's ability to recover from failures.

9.8 Managed Services

Organizations can use managed databases, monitoring systems, storage, load balancing, and other services without managing every underlying infrastructure component.

10. Risks of Cloud Infrastructure

Cloud computing provides many benefits, but it also introduces risks.

10.1 Security Risks

Misconfigured cloud resources can expose sensitive information.

Examples:

Public storage buckets
Weak passwords
Excessive permissions
Exposed ports
Poorly configured security groups

Security must therefore be designed into cloud infrastructure.

10.2 Cost Management

Cloud resources are often usage-based.

If resources are not monitored, unexpected costs can occur.

For example:

Unused EC2 Instances
        +
Large Storage
        +
High Data Transfer
        ↓
Unexpected Cloud Bill

Companies should implement:

Cost monitoring
Budgets
Alerts
Resource tagging
Automatic cleanup
10.3 Vendor Lock-In

Organizations may become highly dependent on one cloud provider's services.

For example, an application designed specifically around many provider-specific services can become difficult to migrate.

10.4 Availability Risks

Cloud providers operate highly available infrastructure, but cloud services can still experience outages.

Organizations should design critical systems with appropriate redundancy and disaster recovery strategies.

10.5 Data Privacy and Compliance

Organizations may have legal and regulatory requirements concerning where and how data is stored.

Cloud deployments must consider:

Data location
Encryption
Access control
Retention
Compliance requirements
10.6 Configuration Errors

Human mistakes can cause serious cloud problems.

For example:

Incorrect Security Group
          ↓
Publicly Accessible Service
          ↓
Security Risk

Infrastructure as Code, reviews, automated testing, and policy controls can reduce these risks.

10.7 Internet Dependency

Many cloud services are accessed over networks and the internet.

Network failures or connectivity problems can affect access to cloud applications and management interfaces.

11. Cloud and DevOps

Cloud computing is strongly connected with DevOps.

A typical DevOps workflow can look like:

Developer
    ↓
GitHub
    ↓
CI/CD Pipeline
    ↓
Docker Build
    ↓
Container Registry
    ↓
Cloud Infrastructure
    ↓
Application
    ↓
Monitoring

Infrastructure as Code can further automate the process:

Terraform
    ↓
AWS
    ↓
VPC
    ↓
Subnet
    ↓
Security Group
    ↓
EC2
    ↓
Docker
    ↓
Application

This allows infrastructure to be version-controlled and reproduced consistently.

12. Shared Responsibility Concept

Cloud security is generally a shared responsibility.

The cloud provider is responsible for security of the cloud infrastructure, while customers remain responsible for security tasks related to the services and resources they use.

For example:

Cloud Provider
      │
      ├── Physical infrastructure
      ├── Data centers
      └── Underlying hardware
               │
               ↓
Customer
      │
      ├── IAM permissions
      ├── Application security
      ├── Data protection
      ├── Network configuration
      └── Operating system security

The exact division of responsibility depends on the cloud service being used.

13. Summary

Cloud computing provides computing resources through network-accessible services instead of requiring organizations to own and operate all physical infrastructure themselves.

The major cloud service models are:

IaaS
Infrastructure as a Service

PaaS
Platform as a Service

SaaS
Software as a Service

AWS is one of the major cloud platforms and provides services for computing, storage, networking, databases, security, monitoring, and many other workloads.

Cloud infrastructure provides advantages such as:

Scalability
Elasticity
Faster deployment
Automation
Global availability
Managed services
Backup and disaster recovery capabilities

However, organizations must also manage risks including:

Security misconfiguration
Unexpected costs
Vendor lock-in
Availability problems
Compliance requirements
Data privacy
Configuration mistakes

For DevOps engineers, understanding cloud computing is important because modern application deployment frequently combines:

Git
 +
CI/CD
 +
Docker
 +
Terraform
 +
Cloud
 +
Monitoring

Therefore, cloud computing fundamentals provide the foundation for learning AWS infrastructure, Infrastructure as Code, container deployment, CI/CD, and cloud-based DevOps practices.

14. Key Points to Remember
Cloud Computing
        ↓
Computing Resources over the Internet

AWS
        ↓
Amazon Web Services

IaaS
        ↓
Infrastructure

PaaS
        ↓
Platform

SaaS
        ↓
Software

On-Premise
        ↓
Company manages physical infrastructure

Cloud
        ↓
Cloud provider provides infrastructure/services

Advantages
        ↓
Scalability + Flexibility + Automation + Global Reach

Risks
        ↓
Security + Cost + Compliance + Vendor Lock-in