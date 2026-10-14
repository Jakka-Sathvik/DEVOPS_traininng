RTO and RPO
These are two important Disaster Recovery (DR) metrics.
1. RTO (Recovery Time Objective)
Definition:
The maximum acceptable time to restore a service after an outage.
Question it answers:
"How quickly must the system be back online?"

Example:
Application goes down at 10:00 AM
Application restored at 10:20 AM

RTO = 20 minutes

If your business requires recovery within 30 minutes:
Target RTO = 30 minutes
Actual Recovery = 20 minutes
Result = PASS

2. RPO (Recovery Point Objective)
Definition:
The maximum acceptable amount of data loss measured in time.
Question it answers:
"How much data can we afford to lose?"

Example:
Last backup: 9:55 AM
Failure occurs: 10:00 AM
Recovery completed

Data between:
9:55 AM → 10:00 AM

RPO = 5 minutes

This means up to 5 minutes of data may be lost.
Simple Difference
Metric	Focus
RTO	How fast the system recovers
RPO	How much data can be lost


AWS Example
ECS Application
      ↓
RDS Multi-AZ
      ↓
Automated Backups

Scenario:
Database failure at 2:00 PM
Failover completed at 2:03 PM

RTO = 3 minutes

Because Multi-AZ uses synchronous replication:
RPO ≈ 0

Very little or no data is lost.
DevOps Interview Answer
RTO (Recovery Time Objective) is the maximum acceptable time required to restore a service after a failure.

RPO (Recovery Point Objective) is the maximum acceptable amount of data loss measured in time.

Example:
If an application must be restored within 30 minutes and can lose at most 5 minutes of data, then:
RTO = 30 minutes
RPO = 5 minutes.














