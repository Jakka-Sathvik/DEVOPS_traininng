Disaster Recovery Runbook

Purpose

Provide a standardized process for recovering the production environment after infrastructure, application, or database failures.

Environment

ECS Cluster: devops-training-cluster

ECS Service: devops-training-service

Application Load Balancer (ALB)

Amazon RDS PostgreSQL (Multi-AZ)

CloudWatch Alarms

SNS Notifications

Incident Detection

Sources:

CloudWatch Alarm

ECS Service Events

ALB 5XX Errors

Application Health Checks

User Reports

Initial Assessment

Confirm alert details.

Determine impacted component:

ECS

ALB

RDS

Network

Estimate business impact.

Start incident timeline documentation.

ECS Recovery Procedure

Symptoms

Tasks stopped unexpectedly

Application unavailable

High error rate

Actions

Open ECS Cluster.

Check service events.

Verify running task count.

Review CloudWatch logs.

Confirm image availability.

Force new deployment if required.

Validation

Desired task count restored.

All targets healthy.

Application accessible.

ALB Recovery Procedure

Symptoms

Increased 5XX errors

Unhealthy targets

Routing failures

Actions

Check target group health.

Verify ECS task health.

Review ALB metrics:

Request Count

Target Response Time

HTTPCode_ELB_5XX_Count

Replace failed targets if required.

Validation

Healthy targets available.

No active routing failures.

RDS Failover Procedure

Symptoms

Database connectivity failures

Increased database latency

Actions

Verify RDS status.

Confirm Multi-AZ failover event.

Monitor application reconnection.

Validate database availability.

Validation

Database available.

Application transactions successful.

No data inconsistency.

Database Restore Procedure

Use When

Data corruption

Accidental deletion

Failed upgrade

Actions

Identify recovery point.

Initiate Point-in-Time Restore.

Restore database instance.

Validate restored data.

Update application connection if required.

Perform application testing.

Validation

Data verified.

Application functioning normally.

Communication

Notify stakeholders.

Provide status updates every 30 minutes during major incidents.

Record all actions taken.

Recovery Objectives

RTO: Less than 30 minutes

RPO: Less than 5 minutes

Post-Incident Review

Document root cause.

Complete 5 Whys analysis.

Identify corrective actions.

Update runbooks.

Schedule follow-up testing.

Disaster Recovery Testing Frequency

ECS Recovery Test: Monthly

RDS Failover Test: Quarterly

Backup Restore Test: Quarterly

Full DR Exercise: Twice per year