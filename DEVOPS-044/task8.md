Final DevOps Project Documentation
For your final DevOps project, prepare four professional documents covering the architecture, operational procedures, incident investigation, and overall project results.
The documents will use your existing training architecture: Application Load Balancer → ECS Fargate → application containers, with RDS for the database and CloudWatch for logs, metrics, alarms, dashboards, and SNS notifications. Terraform manages infrastructure, and GitHub Actions handles CI/CD using OIDC and least-privilege IAM.
1. Final architecture
GitHub repository
Application code · Tests · Terraform · Workflow


GitHub Actions CI/CD
Lint · Test · Security scan · Terraform checks · OIDC


Amazon ECR
Versioned, immutable application image


AWS application infrastructure — multi-AZ design
Application Load Balancer
HTTP/HTTPS listener · Target group · Health checks


Availability Zone A
ECS Fargate task


Availability Zone B
ECS Fargate task




Amazon RDS
Private networking · Backups · Restore testing



CloudWatch + SNS
Logs · Metrics · Alarms · Dashboard · Notifications


Security and reliability controls
- Public access terminates at the ALB; ECS tasks and RDS should use private subnets.
- Allow application traffic to ECS only from the ALB security group.
- Allow database traffic only from the authorized application security group.
- Store secrets in Secrets Manager; use IAM roles rather than long-lived AWS keys.
- Use health checks, ECS service recovery, deployment verification, and rollback procedures.
- Configure RDS backups and test restoration; define recovery time objective (RTO) and recovery point objective (RPO).
- Use Terraform state protection, environment-specific configuration, and reviewed deployment changes.
This is the target architecture for your documentation, not confirmation that every resource or control has been deployed and verified in your AWS account.
2. Operations runbook
Suggested file: docs/operations-runbook.md
DevOps Operations Runbook
Project: DevOps Application Deployment and Monitoring
AWS Region: ap-south-1
ECS Cluster: devops-training-cluster
ECS Service: devops-training-service
ECR Repository: devops-training-app
CloudWatch Log Group: /ecs/devops-training-app
1. Purpose
This runbook describes how to check application health, investigate failures, recover ECS tasks, validate deployments, and respond to monitoring alerts.
2. Pre-incident health checks
Run these commands using an authenticated AWS CLI session with appropriate permissions.
aws sts get-caller-identity
aws ecs describe-services \
  --cluster devops-training-cluster \
  --services devops-training-service \
  --region ap-south-1
Review:
- Desired, running, and pending task counts.
- ECS service events and recent deployment failures.
- Task health and task-definition revision.
- Load balancer target health.
- Recent application errors and CloudWatch alarms.
Check target health using your actual target group ARN:
aws elbv2 describe-target-health \
  --target-group-arn YOUR_TARGET_GROUP_ARN \
  --region ap-south-1
3. Incident response procedure
Step 1 — Detect
Record the alert, timestamp, affected environment, deployment version, and user-visible impact.
Step 2 — Investigate
Check ECS service events, task stopped reasons, container exit codes, ALB 5XX metrics, unhealthy targets, application logs, and database connectivity.
aws logs tail /ecs/devops-training-app \
  --since 30m \
  --region ap-south-1
Step 3 — Identify the cause
Determine whether the issue is caused by application code, container startup, configuration, IAM permissions, networking, database connectivity, resource pressure, or a failed deployment.
Step 4 — Recover
- For a stopped task, confirm ECS replaces it and reaches the desired count.
- For a bad release, roll back to a known-good task-definition revision or image digest using the approved deployment procedure.
- For database issues, check connectivity, available capacity, and database health before making changes.
- For infrastructure issues, review Terraform plans before applying any correction.
Avoid deleting resources or changing production infrastructure without approval.
Step 5 — Verify
Confirm:
- ECS tasks are running and healthy.
- ALB targets are healthy.
- The /health endpoint returns the expected response.
- Application errors and latency have returned to acceptable levels.
- Database access works where required.
- Monitoring alarms return to normal.
Example application test:
curl -i http://YOUR_ALB_DNS_NAME/health
Expected application response:
{"status":"healthy"}
Use HTTPS if configured for the environment.
Step 6 — Document
Record the incident timeline, root cause, impact, recovery actions, evidence, and preventive actions.
4. Recovery and escalation
If automatic recovery fails:
1. Preserve logs and relevant task details.
2. Identify the last known-good release.
3. Escalate to the responsible application, cloud, or database owner.
4. Obtain approval for high-impact changes.
5. Verify recovery before closing the incident.
5. Operational safety
- Run failure simulations only in a dedicated test environment.
- Do not stop tasks in production just to test recovery.
- Do not expose database credentials, access keys, or secret values in logs.
- Review Terraform plans before applying changes.
- Confirm backups and recovery requirements before database restore operations.
6. Closure checklist
- Service is stable.
- Application health checks pass.
- ALB targets are healthy.
- Logs and alarms have been reviewed.
- Root cause and evidence are documented.
- Follow-up actions have owners and due dates.




3. Root cause analysis (RCA)
Suggested file: docs/root-cause-analysis.md
Use this report when investigating the training scenario where ECS CPU, ALB 5XX errors, and response time increase.
Root Cause Analysis (RCA)
1. Incident summary
Incident title: Increased Application Error Rate
Environment: Test or staging
Affected services: Application, ECS Fargate, ALB; RDS if evidence confirms database involvement
Severity: To be assigned according to actual impact
Status: Investigation template — evidence required
2. Symptoms
The monitoring scenario reports:
- Increased ALB 5XX responses.
- Increased ECS CPU utilization.
- Increased application response time.
- CloudWatch notification indicating an increased application error rate.
These are observed symptoms in the exercise scenario. They do not independently establish the root cause.
3. Incident timeline
Time	Event
T0	CloudWatch alarm detected elevated errors
T1	Engineer checked ALB metrics and target health
T2	Engineer reviewed ECS service events and task logs
T3	Suspected cause was investigated using evidence
T4	Corrective action was applied
T5	Health checks and application metrics were verifiedReplace T0–T5 with actual timestamps and actions from the exercise.
4. Investigation evidence
Collect:
- ALB HTTPCode_ELB_5XX_Count and HTTPCode_Target_5XX_Count.
- ALB TargetResponseTime and UnHealthyHostCount.
- ECS CPU and memory utilization.
- ECS stopped-task reasons and deployment events.
- Application logs around the incident window.
- RDS CPU, connections, storage, and relevant database errors, if applicable.
- Recent application releases, configuration changes, and infrastructure changes.
5. Five Whys
Why 1: Why did users experience errors?
Record the failing request, HTTP status, and application error from logs.
Why 2: Why did that request fail?
Determine whether the cause was an application exception, timeout, unhealthy target, or dependency failure.
Why 3: Why did that condition occur?
Inspect resource utilization, recent code changes, task health, network connectivity, and database behavior.
Why 4: Why was the condition not prevented or recovered from sooner?
Review capacity, health checks, deployment safeguards, retry behavior, and alarms.
Why 5: What underlying control needs improvement?
Document the verified process, design, testing, or configuration weakness.
6. Root cause
Confirmed root cause: Pending investigation.
Do not declare high CPU to be the root cause merely because it occurred alongside errors. Correlate logs, metrics, task events, and changes before reaching a conclusion.
7. Resolution
Document:
- The corrective action.
- The release or configuration involved.
- Who approved the change.
- The time recovery began and completed.
- How the fix was validated.
- Whether rollback was required.
8. Preventive actions
Action	Priority	Owner	Status
Add or refine actionable error-rate alarms	High	Operations	Open
Review ECS CPU/memory sizing and scaling	High	Cloud team	Open
Add deployment health verification and rollback	High	CI/CD owner	Open
Improve application error context in logs	Medium	Application team	Open
Test recovery and document lessons learned	Medium	DevOps team	OpenAdjust the actions to match the confirmed cause.
9. Closure criteria
Close the incident only after service health is restored, the root cause is evidence-based, recovery is verified, and follow-up actions have owners.




How to investigate the scenario
1. Open CloudWatch and identify the time the error alarm changed state.
2. Compare ALB 5XX and response-time metrics with ECS CPU and memory metrics.
3. Inspect ECS service events and stopped-task reasons.
4. Search application logs for ERROR, Exception, Timeout, and Connection around the same timestamp.
5. Form a hypothesis, test it against evidence, apply the smallest safe fix, and verify recovery.
4. Final operations report
Suggested file: docs/final-operations-report.md
Final DevOps Operations Report
Prepared by: Sathvik Jakka
Project: AWS DevOps Application Deployment, Monitoring and Recovery
Reporting date: 09 October 2026
AWS Region: ap-south-1
1. Executive summary
This project brings together infrastructure as code, automated application delivery, container orchestration, monitoring, security controls, and incident recovery procedures.
The intended architecture uses Terraform-managed AWS infrastructure, GitHub Actions for CI/CD, Amazon ECR for application images, ECS Fargate for container execution, an Application Load Balancer for traffic distribution, RDS for persistence, and CloudWatch with SNS for monitoring and notifications.
Actual deployment status and test results must be recorded from the AWS environment before this report is treated as evidence of successful completion.
2. Scope of work
- Reviewed and organized Terraform infrastructure.
- Structured reusable modules and environment-specific configuration.
- Added CI/CD validation, testing, image versioning, and deployment safeguards.
- Planned OIDC-based AWS authentication and least-privilege permissions.
- Designed multi-AZ application traffic distribution.
- Reviewed ECS health checks, service recovery, and deployment rollback.
- Defined CloudWatch logs, metrics, alarms, dashboards, and notifications.
- Reviewed RDS backup, restore, RTO, and RPO requirements.
- Documented incident response, root cause analysis, and recovery validation.
- Identified security and cost optimization review areas.
Mark each item complete only when supported by implementation or test evidence.
3. Architecture overview
GitHub → GitHub Actions → Amazon ECR → ECS Fargate → Application
User traffic → Application Load Balancer → Healthy ECS targets
Application → Private RDS database
ECS, ALB, RDS, and application telemetry → CloudWatch → Alarms and SNS notifications
Terraform manages the AWS resources and configuration.
4. Security controls
- OIDC-based GitHub Actions authentication where implemented.
- Least-privilege IAM roles and policies.
- Security groups restricting traffic to required sources and ports.
- Private database access.
- Secrets Manager for application secrets.
- No long-lived AWS credentials stored in repository files.
- Image scanning and deployment gates where configured.
- Review of Terraform state access and sensitive outputs.
Record any remaining security gaps and remediation dates.
5. Monitoring and operations
Monitoring should cover:
- ECS CPU and memory.
- ALB request count, response time, 5XX responses, and unhealthy targets.
- RDS CPU, connections, storage, and database health.
- Application errors and health-check failures.
Operational procedures cover alert detection, investigation, recovery, rollback, verification, and incident documentation.
6. Resilience and disaster recovery
The design aims to distribute application tasks across availability zones and use ALB health checks to avoid routing traffic to unhealthy targets.
ECS service scheduling can replace failed tasks when the service and capacity are configured correctly. This does not guarantee zero downtime.
RDS recovery must be validated through backup and restore testing. Document the measured recovery time and data recovery point; do not report an RTO or RPO as achieved until it has been tested.
7. Cost optimization
Review:
- Oversized ECS task CPU and memory allocations.
- Unused load balancers, volumes, snapshots, IP addresses, and other resources.
- CloudWatch log retention and metric costs.
- NAT gateway and data-transfer charges.
- RDS instance sizing and storage utilization.
- Development resources that can be stopped or removed safely.
Record estimated and actual savings only when supported by billing or cost analysis.
8. Validation results
Validation	Result
Terraform formatting and validation	Pending evidence
CI tests and security scans	Pending evidence
ECR image publishing and version tracking	Pending evidence
ECS service stability	Pending evidence
ALB target health and /health endpoint	Pending evidence
CloudWatch alarms and SNS notifications	Pending evidence
Controlled failure and task recovery	Pending evidence
Deployment rollback	Pending evidence
RDS backup and restore test	Pending evidence
Security and cost audit	Pending evidenceReplace each result with Pass, Fail, Not Tested, or Not Applicable and include evidence links.
9. Outstanding actions
1. Run and record the final health and recovery tests.
2. Verify the CI/CD deployment and rollback procedures.
3. Validate database backup restoration against agreed RTO/RPO targets.
4. Resolve outstanding security findings.
5. Review AWS resource utilization and costs.
6. Attach sanitized logs, screenshots, workflow runs, Terraform plans, and test results.
10. Conclusion
The project defines a DevOps operating model that connects infrastructure automation, secure CI/CD, container deployment, monitoring, incident management, and recovery.
Final completion should be confirmed only after the relevant validation tests pass and evidence is attached. Any untested or incomplete control should remain explicitly documented as outstanding.




5. Final verification checklist
Project readiness
0 of 13



[ ] Architecture diagram and resource inventory documented
[ ] Terraform fmt, validate, and plan reviewed
[ ] GitHub Actions tests and security gates verified
[ ] OIDC and least-privilege IAM verified
[ ] Correct image version deployed to ECS
[ ] Desired ECS task count and ALB target health verified
[ ] /health endpoint returns the expected response
[ ] CloudWatch dashboard, alarms, and SNS notifications tested
[ ] Failure recovery tested safely in a non-production environment
[ ] Rollback procedure tested or explicitly marked not tested
[ ] RDS backup and restore procedure documented and tested
[ ] Security findings and cost optimization actions recorded
[ ] RCA and final operations report updated with real evidence
 Copy checklist
Recommended final folder structure
docs/
├── final-architecture.md
├── operations-runbook.md
├── root-cause-analysis.md
└── final-operations-report.md