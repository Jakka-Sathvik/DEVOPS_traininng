Validate Application Load Balancer (ALB) Health
Sathvik, let's verify that your AWS Application Load Balancer can reach your ECS application and that the application is responding correctly.
For your setup, we'll validate this path:
Client / Browser
Sends an HTTP request

Application Load Balancer
Listener :80 or :443

Target group
Health check: HTTP :8000/health

ECS Fargate task
Container: devops-app · Port 8000


We'll check the target health first, then verify networking, application responses, and CloudWatch evidence.
1. Find your ALB and target group
First, list your load balancers:
aws elbv2 describe-load-balancers \
  --region ap-south-1 \
  --query 'LoadBalancers[*].[LoadBalancerName,DNSName,State.Code,LoadBalancerArn]' \
  --output table


Find your ALB name and copy its ARN. Then list the target groups:
aws elbv2 describe-target-groups \
  --region ap-south-1 \
  --query 'TargetGroups[*].[TargetGroupName,Port,Protocol,HealthCheckPath,TargetGroupArn]' \
  --output table


Choose the target group associated with your ECS service. For our example, it should use HTTP on port 8000 with the health-check path /health.
2. Check target health
Set the target group ARN you found:
export TG_ARN="YOUR_TARGET_GROUP_ARN"


Now run:
aws elbv2 describe-target-health \
  --target-group-arn "$TG_ARN" \
  --region ap-south-1 \
  --output table


Interpret the result:
Target state	Meaning
healthy	The target passed its health check
initial	Registration or initial health checks are in progress
unhealthy	The target failed a health check
unused	The target is not currently eligible for load balancing
draining	The target is being removed from service
AWS reports reason codes that help explain unhealthy targets. 

Elastic Load Balancing
+1



3. Verify the health-check configuration
Run:
aws elbv2 describe-target-groups \
  --target-group-arns "$TG_ARN" \
  --region ap-south-1 \
  --query 'TargetGroups[0].{Port:Port,Protocol:Protocol,Path:HealthCheckPath,HealthPort:HealthCheckPort,Matcher:Matcher.HttpCode,Interval:HealthCheckIntervalSeconds,Timeout:HealthCheckTimeoutSeconds}' \
  --output table


For our sample Flask application, verify:
- Protocol: HTTP
- Health-check port: traffic-port or the configured port 8000
- Path: /health
- Expected response: HTTP 200
If your ALB health check uses / instead, either configure it to use /health or verify that / also returns a successful status code.
4. Troubleshoot unhealthy targets
Run this command to display the health-check reason and description:
aws elbv2 describe-target-health \
  --target-group-arn "$TG_ARN" \
  --region ap-south-1 \
  --query 'TargetHealthDescriptions[*].[Target.Id,Target.Port,TargetHealth.State,TargetHealth.Reason,TargetHealth.Description]' \
  --output table


Use the result to select the next action:
Target.ResponseCodeMismatch
The application returned a status code outside the target group's accepted range. Verify /health and the configured success codes.


Target.Timeout
Check security groups, network ACLs, application availability, and whether the application responds within the timeout.


Target.FailedHealthChecks
Confirm the application is running and listening on the expected container port.


Target.NotInUse
Check whether the target group is associated with an ALB listener rule and whether the target's Availability Zone is enabled.


These are common ALB health-check failure causes documented by AWS. 

Elastic Load Balancing
+1



For Fargate, the ECS task security group should allow inbound traffic on port 8000 from the ALB security group, and the ALB security group must allow outbound traffic to the task on that port. Avoid opening the task port to the entire internet. 

AWS re:Post
+1



5. Test the application through the ALB
Get the ALB DNS name:
aws elbv2 describe-load-balancers \
  --region ap-south-1 \
  --query 'LoadBalancers[*].[LoadBalancerName,DNSName]' \
  --output table


Then test it from your terminal:
curl -i http://YOUR_ALB_DNS_NAME/health


Expected response:
HTTP/1.1 200 OK
Content-Type: application/json


{"status":"healthy"}


Use HTTPS instead if your application is configured with an HTTPS listener and certificate.
6. Verify ECS service health
Finally, inspect the service:
aws ecs describe-services \
  --cluster devops-training-cluster \
  --services devops-training-service \
  --region ap-south-1 \
  --query 'services[0].{Desired:desiredCount,Running:runningCount,Pending:pendingCount,Deployments:deployments[*].rolloutState,Events:events[:5].message}' \
  --output json


Check the target group in the AWS Console too: EC2 → Target Groups → select your target group → Targets.
Your validation is complete when the expected ECS tasks are running, the ALB targets show healthy, and the application health endpoint returns HTTP 200.