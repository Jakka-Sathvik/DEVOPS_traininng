Validate ALB + ECS Service Resilience
The goal is to confirm that your ALB continues routing traffic and ECS keeps the application available when a task fails.
1. Check ECS Service
Open ECS → Clusters → devops-training-cluster → devops-training-service.
Verify:
Desired tasks: 2
Running tasks: 2
Pending tasks: 0

Ideally:
Task 1 → AZ-1
Task 2 → AZ-2

2. Check ALB Target Health
Go to:
EC2 → Load Balancers → your ALB → Target Groups → Targets
Expected:
Target 1 → Healthy
Target 2 → Healthy

The /health endpoint should return HTTP 200:
{
  "status": "healthy"
}

3. Test Normal Traffic
Access the ALB DNS name:
http://<ALB-DNS-NAME>

Refresh several times and confirm the application continues responding.
Also check:
ALB
 ↓
Target Group
 ↓
ECS Task 1 / Task 2

4. Simulate ECS Task Failure
Stop one ECS task manually:
ECS → Service → Tasks → select task → Stop
Watch the service.
Expected:
Task stopped
     ↓
ECS detects desired count < 2
     ↓
New task starts
     ↓
Task registers with ALB
     ↓
Health check passes
     ↓
2 healthy tasks restored

The application should remain available through the other healthy task.
5. Check ALB During Failure
While the task is being replaced, verify:
Healthy targets → at least 1
ALB 5XX → no unexpected increase
Application → still accessible

After recovery:
Healthy targets = 2
Running tasks = 2

6. Verify AZ Resilience
Confirm that tasks are distributed across both AZs.
Example:
             ALB
           /     \
        AZ-1     AZ-2
         ↓         ↓
       ECS       ECS
      Task 1    Task 2

If one task/AZ becomes unavailable, the ALB should route traffic to the remaining healthy target.
Validation Checklist
☐ ECS desired count = 2
☐ ECS running count = 2
☐ Tasks distributed across AZs
☐ ALB has healthy targets
☐ /health returns HTTP 200
☐ Application accessible through ALB
☐ One task failure tested
☐ ECS automatically replaced failed task
☐ ALB continued serving traffic
☐ No sustained 5XX errors
☐ Two healthy tasks restored

Pass condition: If you can stop one ECS task and the application remains available while ECS automatically launches a replacement, your ALB + ECS service resilience is working correctly.