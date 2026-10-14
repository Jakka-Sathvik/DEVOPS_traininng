Review CloudWatch Alarms and Operational Controls
For your ALB → ECS Fargate → RDS PostgreSQL architecture, CloudWatch should provide early warning, automated notifications, and operational visibility.
CloudWatch Alarm Review
1. ECS CPU Utilization Alarm
Purpose:
Detect high application load

Recommended:
Metric: ECS Service CPUUtilization
Threshold: > 80%
Period: 5 minutes
Evaluation Periods: 2

Action:
Alarm
  ↓
SNS Notification
  ↓
Operations Team

Validation:
☐ Alarm configured
☐ SNS notification attached
☐ Alarm tested

2. ECS Memory Utilization Alarm
Purpose:
Detect memory leaks
Prevent task crashes

Recommended:
Metric: MemoryUtilization
Threshold: > 80%
Period: 5 minutes

Validation:
☐ Memory alarm configured
☐ Notification tested

3. ALB 5XX Error Alarm
Purpose:
Detect application failures
Detect backend issues

Recommended:
Metric:
HTTPCode_Target_5XX_Count

Threshold:
> 5 in 5 minutes

Expected Flow:
Application Error
      ↓
ALB 5XX
      ↓
CloudWatch Alarm
      ↓
SNS Alert

Validation:
☐ Alarm exists
☐ Alert delivered

4. ALB Response Time Alarm
Purpose:
Detect performance degradation

Recommended:
Metric:
TargetResponseTime

Threshold:
> 2 seconds

Validation:
☐ Latency alarm configured
☐ Threshold documented

5. RDS CPU Alarm
Purpose:
Detect database stress

Recommended:
Metric:
CPUUtilization

Threshold:
> 80%

Validation:
☐ Alarm configured
☐ Notification configured

6. RDS Free Storage Alarm
Purpose:
Prevent database outage

Recommended:
Metric:
FreeStorageSpace

Threshold:
< 20%

Validation:
☐ Storage alarm configured
☐ Tested

Operational Controls Review
Monitoring Dashboard
Create a dashboard containing:
ECS CPU
ECS Memory
ALB Requests
ALB Response Time
ALB 5XX Errors
RDS CPU
RDS Connections

Example:
CloudWatch Dashboard
        ↓
 ALB Metrics
 ECS Metrics
 RDS Metrics

Validation:
☐ Dashboard exists
☐ Team has access
☐ Dashboard reviewed regularly

SNS Notification Flow
Recommended:
CloudWatch Alarm
        ↓
SNS Topic
        ↓
Email
        ↓
Operations Team

Validation:
☐ Topic configured
☐ Subscription confirmed
☐ Test notification received

Log Monitoring
CloudWatch Logs Insights queries should be available for:
ERROR
WARNING
Exception
Timeout
Connection

Example:
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc

Validation:
☐ Logs retained
☐ Queries documented
☐ Error search tested

Incident Response Control
Recommended workflow:
Alert
  ↓
Acknowledge
  ↓
Investigate
  ↓
Identify Cause
  ↓
Fix
  ↓
Verify
  ↓
Document

Use the same process from your Day 38 Incident Challenge.
Validation:
☐ Incident procedure documented
☐ Team knows escalation path

Backup and Recovery Control
Monitor:
RDS Backups
RDS Failovers
Snapshot Creation
Restore Testing

Validation:
☐ Backup monitoring enabled
☐ Restore tested quarterly

Recommended Alarm Priority
Priority	Alarm
Critical	ALB 5XX Errors
Critical	ECS Service Unavailable
Critical	RDS Unavailable
High	ECS CPU > 80%
High	ECS Memory > 80%
High	RDS Storage Low
Medium	ALB Response Time High
Medium	RDS Connections High


Operational Readiness Checklist
☐ ECS CPU Alarm
☐ ECS Memory Alarm
☐ ALB 5XX Alarm
☐ ALB Response Time Alarm
☐ RDS CPU Alarm
☐ RDS Storage Alarm
☐ SNS Notifications Working
☐ CloudWatch Dashboard Created
☐ Logs Insights Queries Saved
☐ Incident Runbook Available
☐ Backup Monitoring Enabled
☐ Restore Procedure Tested
☐ Alarm Testing Completed

Overall Assessment
Monitoring
     ↓
CloudWatch Metrics
     ↓
CloudWatch Alarms
     ↓
SNS Notifications
     ↓
Operations Team
     ↓
Incident Response

A well-operated environment is not just monitored—it has tested alarms, clear notification paths, dashboards, runbooks, and regular recovery exercises. For your training project, ensure every alarm can be triggered and verified at least once before considering it production-ready.