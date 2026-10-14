Simulate Failure and Recover an ECS Application
Sathvik, let's perform a controlled production-style incident exercise using your AWS architecture:
ALB → ECS Fargate → CloudWatch → Recovery
The goal is to learn how to detect a failure, investigate the root cause, recover the application, and verify that service health has returned to normal.
We'll start with a safe simulation that doesn't intentionally break your live application. Then we'll cover controlled ECS failure testing in a dedicated training environment.
1. Understand the failure-and-recovery workflow
1. Simulate failure
A test request fails or a test task stops

2. Detect
ALB health checks and CloudWatch alarms

3. Investigate
ECS events, task status, and application logs

4. Recover
Fix the cause or restore a known-good revision

5. Verify
Healthy targets, stable ECS service, successful requests


2. Establish a healthy baseline
Before simulating anything, confirm that your service is healthy.
Run these commands using the AWS CLI configured for ap-south-1.
aws ecs describe-services \
  --cluster devops-training-cluster \
  --services devops-training-service \
  --region ap-south-1 \
  --query 'services[0].{Desired:desiredCount,Running:runningCount,Pending:pendingCount,TaskDefinition:taskDefinition}'


Check the latest service events:
aws ecs describe-services \
  --cluster devops-training-cluster \
  --services devops-training-service \
  --region ap-south-1 \
  --query 'services[0].events[:5].[createdAt,message]' \
  --output table


Also verify your ALB targets using the target-group ARN:
aws elbv2 describe-target-health \
  --target-group-arn YOUR_TARGET_GROUP_ARN \
  --region ap-south-1 \
  --output table


Record the running task count, active task-definition revision, target health, and successful /health response. If the service is already unhealthy, investigate that condition before starting a failure simulation.
3. Simulate a controlled ECS task failure
For this exercise, use a dedicated test service with desired count 2. Do not stop a task in production or in a single-task service, because it can cause an outage.
First, list the running tasks:
aws ecs list-tasks \
  --cluster devops-training-cluster \
  --service-name devops-training-service \
  --desired-status RUNNING \
  --region ap-south-1


Before proceeding, confirm that this is a non-production service, that at least two tasks are running, and that the application remains available if one task stops.
Choose one task ARN from the output and inspect it:
aws ecs describe-tasks \
  --cluster devops-training-cluster \
  --tasks YOUR_TASK_ARN \
  --region ap-south-1


Now simulate the failure by stopping that test task only:
aws ecs stop-task \
  --cluster devops-training-cluster \
  --task YOUR_TASK_ARN \
  --reason "Controlled DevOps failure and recovery exercise" \
  --region ap-south-1


What happens next?
- ECS records the task as stopped.
- The ECS service scheduler should launch a replacement to maintain the desired count.
- The ALB should route traffic to healthy targets, provided another healthy target is available.
Stopping a task is a controlled simulation, not a full infrastructure failure test.
4. Detect and investigate the failure
Immediately inspect ECS service events:
aws ecs describe-services \
  --cluster devops-training-cluster \
  --services devops-training-service \
  --region ap-south-1 \
  --query 'services[0].events[:10].[createdAt,message]' \
  --output table


Check the ALB target health:
aws elbv2 describe-target-health \
  --target-group-arn YOUR_TARGET_GROUP_ARN \
  --region ap-south-1 \
  --output table


Inspect application logs:
aws logs tail /ecs/devops-training-app \
  --since 30m \
  --region ap-south-1


Look for messages related to stopped tasks, failed health checks, image-pull failures, application startup errors, or insufficient capacity.
5. Recover the service
For this task-stop simulation, recovery should happen automatically through the ECS service scheduler.
Check the service again:
aws ecs describe-services \
  --cluster devops-training-cluster \
  --services devops-training-service \
  --region ap-south-1 \
  --query 'services[0].{Desired:desiredCount,Running:runningCount,Pending:pendingCount}'


Wait for the service to stabilize:
aws ecs wait services-stable \
  --cluster devops-training-cluster \
  --services devops-training-service \
  --region ap-south-1


Then confirm that the ALB target group reports healthy targets and test the application:
curl -i http://YOUR_ALB_DNS_NAME/health


Expected response:
{"status":"healthy"}


If the replacement task repeatedly fails, don't keep stopping or restarting tasks. Inspect the ECS events and CloudWatch logs to identify the underlying issue first.
6. Practice a second scenario: failed deployment
In a dedicated test environment, you can also practice recovering from a bad application release.
1. Deploy a test revision that fails its health check.
2. Observe ECS deployment events and ALB target health.
3. Identify the bad task-definition revision.
4. Restore the previously verified task-definition revision.
5. Wait for ECS service stability.
6. Verify healthy ALB targets and a successful application response.
If ECS deployment circuit-breaker rollback is enabled and configured, ECS may automatically roll back an unsuccessful deployment. Otherwise, perform a controlled rollback to a known-good revision. Never replace the active production task definition without first checking the previous working revision and its configuration.
7. Record the incident
Use this template for docs/day-failure-recovery-report.md:
Failure Simulation and Recovery Report
Environment: DevOps training
AWS Region: ap-south-1
ECS Cluster: devops-training-cluster
ECS Service: devops-training-service
1. Baseline
- Initial running task count:
- Active task-definition revision:
- Initial ALB target health:
- Initial application health response:
2. Simulated Failure
- Failure injected:
- Time of failure:
- ECS task affected:
- CloudWatch alarm or event observed:
3. Investigation
- ECS service events:
- CloudWatch log findings:
- ALB target-health findings:
- Root cause:
4. Recovery
- Recovery action:
- Replacement task status:
- Final running task count:
- Final ALB target health:
- Application health-check result:
5. Lessons Learned
- What worked:
- What needs improvement:
- Preventive actions:
- Evidence collected:
Conclusion
Record whether the application recovered successfully and whether monitoring detected the incident.