AWS Security and Cost Audit Report

Project: DevOps Training Environment Audit
Date: 08-Oct-2026
Prepared By: Sathvik Jakka

1. Executive Summary

A comprehensive review was conducted on the AWS infrastructure consisting of:

Application Load Balancer (ALB)

Amazon ECS Fargate

Amazon RDS PostgreSQL

Amazon ECR

CloudWatch

IAM

Secrets Manager

VPC Networking

The audit focused on:

Security posture

Access controls

Secrets management

Monitoring and operational controls

Resource utilization

Cost optimization opportunities

Overall, the environment follows AWS best practices with Multi-AZ deployment, least-privilege principles, centralized monitoring, and managed secret storage. Several optimization opportunities were identified to improve operational efficiency and reduce costs.

2. Infrastructure Overview

Internet
    ↓
Application Load Balancer
    ↓
ECS Fargate Service
    ↓
Amazon RDS PostgreSQL (Multi-AZ)

Monitoring:
CloudWatch + SNS

Secrets:
AWS Secrets Manager

3. Security Audit Findings

3.1 IAM Review

Reviewed Components

ECS Task Role

ECS Execution Role

GitHub Actions OIDC Role

Secrets Manager Access Policies

Findings

Control

Status

Least Privilege Access

PASS

OIDC Authentication

PASS

AdministratorAccess Usage

NOT DETECTED

Secrets Manager Integration

PASS

Resource-Level Permissions

PASS

Recommendations

Continue avoiding AdministratorAccess.

Restrict permissions to specific resources whenever possible.

Perform quarterly IAM permission reviews.

3.2 Security Groups Review

ALB Security Group

Allowed:

HTTP (80)

HTTPS (443)

ECS Security Group

Allowed:

Application Port from ALB Security Group only

RDS Security Group

Allowed:

PostgreSQL (5432) from ECS Security Group only

Findings

Control

Status

Public ALB Access

PASS

ECS Protected Behind ALB

PASS

Database Private Access

PASS

Security Group Segmentation

PASS

Recommendations

Continue restricting access by Security Group references.

Remove unused security group rules during monthly reviews.

3.3 Secrets Management Audit

Reviewed Areas

AWS Secrets Manager

ECS Secret Injection

GitHub Repository

Docker Images

CloudWatch Logs

Findings

Control

Status

Secrets Stored in Secrets Manager

PASS

Secrets Hardcoded in Source Code

NOT DETECTED

Secrets in Docker Images

NOT DETECTED

Secrets in Logs

NOT DETECTED

Encryption Enabled

PASS

Recommendations

Implement periodic secret rotation.

Review secret access permissions quarterly.

Monitor Secrets Manager access through CloudTrail.

4. Monitoring and Operational Controls Review

CloudWatch Alarms Reviewed

Alarm

Status

ECS CPU Utilization

PASS

ECS Memory Utilization

PASS

ALB 5XX Errors

PASS

ALB Response Time

PASS

RDS CPU Utilization

PASS

RDS Storage Monitoring

PASS

Notification Flow

CloudWatch Alarm
        ↓
SNS Topic
        ↓
Email Notification
        ↓
Operations Team

Findings

Control

Status

CloudWatch Monitoring

PASS

Alarming

PASS

Notification Flow

PASS

Dashboard Visibility

PASS

Recommendations

Perform monthly alarm validation.

Conduct quarterly incident response exercises.

5. Backup and Disaster Recovery Review

RDS Backup Strategy

Control

Status

Automated Backups

PASS

Point-in-Time Recovery

PASS

Multi-AZ Deployment

PASS

Manual Snapshots

PASS

Recovery Objectives

Metric

Target

RTO

< 30 Minutes

RPO

< 5 Minutes

Recommendations

Test database restoration quarterly.

Perform annual disaster recovery simulations.

6. Cost Optimization Audit

Resource Review

ECS

Observation:

Resource reservations may exceed actual utilization.

Recommendation:

Right-size CPU and memory allocations based on CloudWatch metrics.

RDS

Observation:

Instance sizing should be periodically reviewed.

Recommendation:

Downsize when CPU and connection metrics remain consistently low.

ECR

Observation:

Old container images accumulate over time.

Recommendation:

Enable lifecycle policies to retain only recent images.

CloudWatch Logs

Observation:

Unlimited retention increases storage costs.

Recommendation:

Configure retention periods:

Development: 14 Days

Test: 30 Days

Production: 90 Days

EBS Volumes

Observation:

Unattached volumes may generate unnecessary charges.

Recommendation:

Remove unused volumes after verification.

Elastic IP Addresses

Observation:

Unassociated Elastic IPs may incur charges.

Recommendation:

Release unused addresses.

Snapshots

Observation:

Old snapshots increase storage costs.

Recommendation:

Apply retention and cleanup policies.

7. Failure and Resilience Validation

ECS Resilience Test

Test:

ECS task termination

Result:

ECS automatically replaced failed task.

ALB maintained service availability.

Status: PASS

ALB Health Validation

Test:

Target health monitoring

Result:

Unhealthy targets removed automatically.

Traffic routed to healthy targets.

Status: PASS

RDS Failover Validation

Test:

Multi-AZ failover review

Result:

Standby database available for automatic failover.

Status: PASS

8. Risk Assessment

Risk Area

Risk Level

IAM Permissions

Low

Secrets Exposure

Low

Security Groups

Low

Monitoring Coverage

Low

Backup Strategy

Low

Cost Management

Medium

Resource Oversizing

Medium

9. Recommendations Summary

Security

Continue least-privilege IAM enforcement.

Enable periodic secrets rotation.

Review security groups monthly.

Audit IAM roles quarterly.

Operations

Validate alarms monthly.

Test disaster recovery procedures quarterly.

Maintain incident response runbooks.

Cost Optimization

Right-size ECS resources.

Review RDS sizing.

Enable ECR lifecycle policies.

Configure CloudWatch log retention.

Remove unused EBS volumes and snapshots.

Release unused Elastic IPs.