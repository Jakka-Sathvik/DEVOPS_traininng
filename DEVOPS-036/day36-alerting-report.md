Day 36 — CloudWatch Alerting & Notification Report

## Objective

Implement monitoring and alerting for the application environment using:

- CloudWatch Alarms
- CloudWatch Dashboard
- SNS Notification Flow

The goal is to proactively detect infrastructure and application issues before they impact users.

---

# Environment

| Component | Service |
|------------|------------|
| Compute | ECS Fargate |
| Load Balancer | Application Load Balancer (ALB) |
| Database | Amazon RDS PostgreSQL |
| Monitoring | Amazon CloudWatch |
| Notifications | Amazon SNS |

---

# Alarm 1 — ECS CPU Utilization

## Purpose

Detect high CPU usage on ECS tasks.

### Metric

```text
AWS/ECS
CPUUtilization
Configuration
Threshold:
> 80%

Period:
5 Minutes

Evaluation Periods:
2

Statistic:
Average
Alert Flow
ECS Service
      ↓
CPU > 80%
      ↓
CloudWatch Alarm
      ↓
SNS Notification
      ↓
Email
Status
Created Successfully
Alarm 2 — ALB 5XX Errors
Purpose

Detect application failures resulting in server-side errors.

Metric
AWS/ApplicationELB
HTTPCode_Target_5XX_Count
Configuration
Threshold:
> 10 Errors

Period:
5 Minutes

Evaluation Periods:
1

Statistic:
Sum
Alert Flow
Application Error
      ↓
ALB Detects 5XX
      ↓
CloudWatch Alarm
      ↓
SNS Notification
      ↓
Operations Team
Status
Created Successfully
Alarm 3 — RDS CPU Utilization
Purpose

Monitor database resource consumption.

Metric
AWS/RDS
CPUUtilization
Configuration
Threshold:
> 75%

Period:
5 Minutes

Evaluation Periods:
2

Statistic:
Average
Alert Flow
RDS Database
      ↓
CPU > 75%
      ↓
CloudWatch Alarm
      ↓
SNS Notification
      ↓
Database Team
Status
Created Successfully
CloudWatch Dashboard
Dashboard Name
devops-training-dashboard
Widgets Added
ECS Metrics
CPU Utilization
Memory Utilization
Running Task Count
ALB Metrics
Request Count
Target Response Time
HTTP 5XX Errors
RDS Metrics
CPU Utilization
Database Connections
Free Storage Space
Dashboard Architecture
CloudWatch Dashboard
│
├── ECS Metrics
│    ├── CPU
│    ├── Memory
│    └── Tasks
│
├── ALB Metrics
│    ├── Requests
│    ├── Latency
│    └── 5XX Errors
│
└── RDS Metrics
     ├── CPU
     ├── Connections
     └── Storage
Status
Dashboard Created Successfully
Notification Flow
SNS Topic
devops-training-alerts
Subscribers
Email:
alerts@example.com
Notification Workflow
CloudWatch Alarm
        ↓
SNS Topic
        ↓
Email Notification
        ↓
Operations Team
End-to-End Alerting Architecture
ECS
 │
 ├── CPU Alarm
 │
 ▼
CloudWatch
 │
 ├── ECS Alarm
 ├── ALB Alarm
 └── RDS Alarm
 │
 ▼
SNS Topic
 │
 ▼
Email Notification
 │
 ▼
DevOps Team
Alarm Testing
ECS CPU Alarm Test

Generated high workload on ECS tasks.

Result
Alarm State:
OK → ALARM

Notification:
Received Successfully
ALB 5XX Alarm Test

Simulated application error response.

Result
Alarm State:
OK → ALARM

Notification:
Received Successfully
RDS CPU Alarm Test

Generated database workload.

Result
Alarm State:
OK → ALARM

Notification:
Received Successfully
Monitoring Benefits
Faster incident detection
Reduced downtime
Real-time operational visibility
Automated alert notifications
Improved application reliability
Better production monitoring practices
Deliverables Completed
CloudWatch Alarms
ECS CPU Utilization Alarm
ALB 5XX Error Alarm
RDS CPU Utilization Alarm
Dashboard
devops-training-dashboard
Notification Flow
SNS Topic Created
Email Subscription Configured
Alarm Notifications Verified
Conclusion

Successfully implemented a basic production-style monitoring and alerting solution using Amazon CloudWatch and Amazon SNS. Three critical alarms were configured, a centralized monitoring dashboard was created, and automated notifications were established to ensure rapid detection and response to infrastructure or application issues.