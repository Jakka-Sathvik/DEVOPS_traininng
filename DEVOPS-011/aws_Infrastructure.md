AWS Global Infrastructure

Training: DevOps Engineer Training
Day: 11
Task: Task 2 — Understand AWS Global Infrastructure
Trainee: Sathvik Jakka

1. AWS Global Infrastructure

AWS operates a large global infrastructure that is organized into geographic Regions and isolated Availability Zones.

The basic hierarchy can be understood as:

AWS
 ↓
Region
 ↓
Availability Zone
 ↓
Data Center

Understanding this structure is important for designing highly available and reliable cloud applications.

2. AWS

AWS (Amazon Web Services) is a cloud computing platform that provides services for:

Compute
Storage
Databases
Networking
Security
Monitoring
Containers
Serverless applications
Machine learning

AWS operates infrastructure across multiple geographic locations around the world.

3. AWS Region

An AWS Region is a geographic area where AWS has infrastructure for running cloud services.

Examples of AWS Regions include:

Asia Pacific (Mumbai)
Asia Pacific (Hyderabad)
US East (N. Virginia)
Europe (Ireland)
Asia Pacific (Singapore)

A Region is designed to be separate from other AWS Regions.

For example:

AWS
 │
 ├── India Region
 │
 ├── US Region
 │
 ├── Europe Region
 │
 └── Asia Pacific Region

Organizations select a Region based on requirements such as:

User location
Latency
Data residency
Compliance
Service availability
Cost
Disaster recovery
4. Availability Zone

An Availability Zone (AZ) is an isolated location within an AWS Region.

A Region normally contains multiple Availability Zones.

For example:

AWS Region
 │
 ├── Availability Zone A
 ├── Availability Zone B
 └── Availability Zone C

Availability Zones are designed to provide isolation from failures in other Availability Zones.

This allows organizations to distribute applications across multiple AZs.

5. Data Center

A data center is a physical facility containing computing infrastructure such as:

Servers
Storage systems
Networking equipment
Power systems
Cooling systems
Physical security infrastructure

AWS Availability Zones consist of one or more data centers.

A simplified relationship is:

Region
   │
   ├── Availability Zone A
   │       └── Data Center(s)
   │
   ├── Availability Zone B
   │       └── Data Center(s)
   │
   └── Availability Zone C
           └── Data Center(s)

The exact physical architecture is managed by AWS.

6. AWS Infrastructure Hierarchy

The relationship can be summarized as:

AWS
 │
 ├── Region
 │     │
 │     ├── Availability Zone A
 │     │       └── Data Center(s)
 │     │
 │     ├── Availability Zone B
 │     │       └── Data Center(s)
 │     │
 │     └── Availability Zone C
 │             └── Data Center(s)
 │
 └── Another Region
       │
       ├── Availability Zone A
       │       └── Data Center(s)
       │
       └── Availability Zone B
               └── Data Center(s)
7. Required AWS Infrastructure Diagram
                         AWS
                          │
             ┌────────────┴────────────┐
             │                         │
        Region 1                  Another Region
             │                         │
      ┌──────┼──────┐             ┌────┴────┐
      │      │      │             │         │
     AZ-A   AZ-B   AZ-C          AZ-A      AZ-B
      │      │      │             │         │
   Data    Data    Data         Data      Data
   Center  Center  Center       Center    Center
Simple Version
AWS
 │
 ├── Region
 │     ├── Availability Zone A
 │     ├── Availability Zone B
 │     └── Availability Zone C
 │
 └── Another Region
       ├── Availability Zone A
       └── Availability Zone B
8. Why Multiple Availability Zones?

Using multiple Availability Zones helps improve application availability.

For example, suppose an application runs only in AZ-A:

User
  ↓
AZ-A
  ↓
Application

If AZ-A experiences a problem, the application could become unavailable.

A more resilient architecture distributes resources:

                 Load Balancer
                       │
                ┌──────┴──────┐
                │             │
              AZ-A           AZ-B
                │             │
            Application   Application

If one Availability Zone has a problem, traffic can potentially be served by resources in another AZ, depending on the architecture.

9. Region vs Availability Zone
Feature	Region	Availability Zone
Meaning	Geographic AWS location	Isolated location inside a Region
Scope	Larger	Smaller
Contains	Multiple AZs	One or more data centers
Purpose	Geographic deployment	Availability and fault isolation
Example	Mumbai Region	AZ within Mumbai Region
10. Example DevOps Architecture

A DevOps engineer might deploy an application like this:

                    AWS Region
                        │
             ┌──────────┴──────────┐
             │                     │
            AZ-A                  AZ-B
             │                     │
          EC2 App               EC2 App
             │                     │
             └──────────┬──────────┘
                        │
                  Load Balancer
                        │
                       User

The application is distributed across multiple Availability Zones to improve resilience.

11. Region Selection

When selecting an AWS Region, organizations should consider:

1. Latency

Choose a Region geographically close to users when low latency is important.

2. Compliance

Some applications have requirements concerning where data can be stored.

3. Service Availability

Not every AWS service or feature is necessarily available in every Region.

4. Cost

Pricing can vary between Regions.

5. Disaster Recovery

Organizations can use multiple Regions for disaster recovery and business continuity when required.

12. Important Points to Remember
AWS
 ↓
Global cloud platform

Region
 ↓
Geographic AWS location

Availability Zone
 ↓
Isolated location inside a Region

Data Center
 ↓
Physical infrastructure facility

The key relationship is:

AWS
  ↓
Region
  ↓
Availability Zone
  ↓
Data Center

Multiple Availability Zones within a Region help organizations design applications with better availability and fault isolation.

13. Real-World DevOps Example

Consider an application serving users in India.

A company could select an AWS Region in India and deploy application infrastructure across multiple Availability Zones.

                  AWS
                   │
            India Region
                   │
          ┌────────┴────────┐
          │                 │
        AZ-A              AZ-B
          │                 │
       EC2 App           EC2 App
          │                 │
          └────────┬────────┘
                   │
             Load Balancer
                   │
                Users

If the application is deployed across multiple Availability Zones, the architecture can be more resilient than a single-AZ deployment.

14. Task Summary
Region

A geographic AWS location containing multiple Availability Zones.

Availability Zone

An isolated location within an AWS Region, consisting of one or more data centers.

Data Center

A physical facility containing computing, storage, networking, power, cooling, and other infrastructure.

Architecture
AWS
 │
 ├── Region
 │     ├── Availability Zone A
 │     ├── Availability Zone B
 │     └── Availability Zone C
 │
 └── Another Region
       ├── Availability Zone A
       └── Availability Zone B
15. Deliverable

File name:

AWS_Infrastructure.md

Task status:

✓ AWS understood
✓ Region understood
✓ Availability Zone understood
✓ Data Center understood
✓ Region vs AZ understood
✓ Multi-AZ concept understood
✓ AWS infrastructure diagram created
✓ Region selection factors documented

Status: Completed