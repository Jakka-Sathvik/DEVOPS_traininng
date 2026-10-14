Review RDS Backup, Restore, and Recovery Strategy
For a production-grade AWS environment, your RDS recovery strategy should cover backups, restores, failover, and disaster recovery.
1. Automated Backups
Verify that automated backups are enabled.
Recommended:
Backup Retention Period: 7–30 days
Backup Window: Off-peak hours
Storage Encryption: Enabled

Check:
RDS
 ↓
Databases
 ↓
devops-training-db
 ↓
Backups

Expected:
Automated Backups = Enabled
Retention Period > 0 days

2. Point-in-Time Recovery (PITR)
RDS continuously stores transaction logs.
This allows recovery to a specific time.
Example:
Database corrupted at 10:15 AM

Recover to:
10:14:59 AM

Validation:
☐ Automated backups enabled
☐ PITR available
☐ Recovery window verified

3. Manual Snapshots
Take snapshots before major changes:
- Engine upgrades
- Schema changes
- Application deployments
- Data migrations
Naming example:
pre-release-v1
pre-db-upgrade
monthly-backup-oct

Validation:
☐ Manual snapshot process documented
☐ Snapshot naming standard defined
☐ Snapshot restore tested

4. Multi-AZ Recovery
For your PostgreSQL Multi-AZ setup:
Primary DB (AZ-1)
       ↓
Synchronous Replication
       ↓
Standby DB (AZ-2)

Benefits:
AZ failure
     ↓
Automatic failover
     ↓
Minimal downtime

Validation:
☐ Multi-AZ enabled
☐ Standby instance present
☐ Failover tested

5. Restore Testing
A backup is useful only if it can be restored.
Perform periodic tests:
Snapshot
     ↓
Restore New Database
     ↓
Connect
     ↓
Verify Data

Validation:
☐ Restore test completed
☐ Application connectivity verified
☐ Data integrity verified

6. Disaster Recovery Scenario
Example:
Application Bug
      ↓
Data Deleted
      ↓
Identify Incident Time
      ↓
Point-in-Time Restore
      ↓
Create New RDS Instance
      ↓
Validate Data
      ↓
Switch Application Connection

Recovery flow:
Backup
   ↓
Restore
   ↓
Validate
   ↓
Redirect Application
   ↓
Resume Service

7. Security Review
Recommended:
Encryption at Rest = Enabled
Encryption in Transit = Enabled
Secrets Manager = Used
Private Subnets = Used
Public Access = Disabled

Validation:
☐ KMS encryption enabled
☐ Secrets stored in Secrets Manager
☐ Database not publicly accessible
☐ Security groups restricted

8. Monitoring and Alerts
Create CloudWatch alarms for:
RDS CPU Utilization
Free Storage Space
Database Connections
Read/Write Latency
Backup Failures

Validation:
☐ CloudWatch alarms configured
☐ SNS notifications configured
☐ Alarm testing completed

Recovery Objectives
Recommended targets:
Metric	Target
RPO (data loss)	< 5 minutes
RTO (service recovery)	< 30 minutes
Backup Retention	7–30 days
Restore Testing	Monthly
Snapshot Before Changes	Always