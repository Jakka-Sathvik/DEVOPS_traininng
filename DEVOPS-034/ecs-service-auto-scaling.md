ECS Service Auto Scaling
Objective

Learn how Amazon ECS Service Auto Scaling automatically adjusts the number of running tasks based on application demand.

The goal is to start with:

Desired Count = 1
Minimum Capacity = 1
Maximum Capacity = 3

and automatically scale the service based on CPU utilization.

1. Why Auto Scaling Is Needed

A fixed number of tasks works only when traffic remains predictable.

Example:

Users = 10
Tasks = 1

The application performs well.

Later:

Users = 1000
Tasks = 1

Problems:

Increased response time
High CPU utilization
Poor user experience
Potential application failure

Auto Scaling solves this problem.

2. What Is ECS Service Auto Scaling?

ECS Service Auto Scaling automatically changes the number of running ECS tasks based on metrics such as:

CPU Utilization
Memory Utilization
Request Count
Custom Metrics

Architecture:

CloudWatch Metric
        │
        ▼
Auto Scaling Policy
        │
        ▼
ECS Service
        │
        ▼
Task Count Changes
3. Basic Training Architecture

Initial configuration:

Minimum Capacity = 1
Desired Capacity = 1
Maximum Capacity = 3

Architecture:

             ECS Service
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
     Task 1     Task 2     Task 3

Initially:

Running Tasks = 1

Additional tasks are created only when needed.

4. Understanding Desired Capacity

Desired Capacity represents the number of tasks ECS attempts to keep running.

Example:

Desired Capacity = 1

ECS maintains:

Task 1

If the task crashes:

Task 1 Failed

ECS automatically launches:

Replacement Task

to maintain the desired count.

5. Understanding Minimum Capacity

Minimum Capacity is the lowest number of tasks that Auto Scaling allows.

Example:

Minimum Capacity = 1

Even if traffic becomes:

0 Requests

ECS keeps:

1 Running Task

This ensures the service remains available.

6. Understanding Maximum Capacity

Maximum Capacity is the highest number of tasks that Auto Scaling can create.

Example:

Maximum Capacity = 3

Even if CPU becomes:

95%

Auto Scaling cannot exceed:

3 Tasks

This helps control cost and resource usage.

7. Capacity Example

Configuration:

Minimum = 1
Desired = 1
Maximum = 3

Possible task counts:

1 Task
2 Tasks
3 Tasks

Not allowed:

0 Tasks
4 Tasks
5 Tasks

because they fall outside the configured limits.

8. Target Tracking Scaling

Target Tracking is the most common ECS scaling strategy.

Example:

CPU Target = 60%

Goal:

Keep Average CPU
Near 60%

Architecture:

CPU Metric
     │
     ▼
Target Tracking
     │
     ▼
Adjust Task Count
9. Scale-Out

Scale-Out means adding more ECS tasks.

Condition:

CPU > 60%

Example:

Task Count = 1
CPU = 80%

Auto Scaling action:

Create Task 2

Result:

Task 1
Task 2

Workload is distributed across more tasks.

10. Additional Scale-Out Example

Traffic continues to increase.

Example:

Task Count = 2
CPU = 75%

Auto Scaling performs:

Create Task 3

Result:

Task 1
Task 2
Task 3

Maximum capacity reached.

11. Scale-In

Scale-In means removing unnecessary tasks.

Condition:

CPU < 60%

Example:

Task Count = 3
CPU = 15%

Auto Scaling action:

Remove One Task

Result:

Task 1
Task 2
12. Additional Scale-In Example

Traffic continues to decrease.

Example:

Task Count = 2
CPU = 10%

Auto Scaling action:

Remove One Task

Result:

Task 1

ECS stops at:

Minimum Capacity = 1
13. CloudWatch Integration

Auto Scaling relies on CloudWatch metrics.

Architecture:

ECS Tasks
     │
     ▼
CloudWatch Metrics
     │
     ▼
Auto Scaling Policy
     │
     ▼
Scale Decision

Common metrics:

CPUUtilization
MemoryUtilization
14. Cooldown Behavior

Cooldown prevents rapid scaling actions.

Without cooldown:

Scale Out
Scale In
Scale Out
Scale In

This creates instability.

Scale-Out Cooldown

Example:

Scale Out
      ↓
Wait
      ↓
Evaluate Again

Typical value:

60 Seconds
Scale-In Cooldown

Example:

Scale In
      ↓
Wait
      ↓
Evaluate Again

Typical value:

300 Seconds

Scale-in cooldown is usually longer to avoid removing capacity too quickly.

15. ECS Console Configuration

Navigate:

ECS
 ↓
Cluster
 ↓
Service
 ↓
Auto Scaling

Enable:

Service Auto Scaling
16. Capacity Settings

Configure:

Minimum Tasks = 1
Desired Tasks = 1
Maximum Tasks = 3

Save.

17. Create Target Tracking Policy

Policy Type:

Target Tracking

Metric:

ECS Service Average CPU Utilization

Target Value:

60%

Example:

Target CPU = 60%
18. Expected Scaling Flow
CPU = 30%
      │
      ▼
Keep 1 Task

CPU = 70%
      │
      ▼
Scale To 2 Tasks

CPU = 85%
      │
      ▼
Scale To 3 Tasks

CPU = 20%
      │
      ▼
Scale Back To 2 Tasks

CPU = 10%
      │
      ▼
Scale Back To 1 Task
19. Monitoring Scaling Events

Navigate:

ECS
 ↓
Cluster
 ↓
Service
 ↓
Events

Examples:

Started Task
Stopped Task
Scale-Out Event
Scale-In Event
20. CloudWatch Verification

Navigate:

CloudWatch
 ↓
Metrics
 ↓
ECS

Review:

CPUUtilization
MemoryUtilization

Verify that scaling decisions match observed metrics.

21. Example Production Architecture
                     Users
                       │
                       ▼
                      ALB
                       │
                       ▼
                ECS Service
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Task 1       Task 2       Task 3
                       │
                       ▼
                 CloudWatch
                       │
                       ▼
                Auto Scaling
22. Troubleshooting
Service Does Not Scale

Check:

Auto Scaling Enabled
Target Tracking Policy Exists
CloudWatch Metrics Available
Scaling Stops At One Task

Verify:

Maximum Capacity > 1
Excessive Scaling

Review:

Target CPU
Cooldown Values
Traffic Pattern
23. Success Criteria

The task is complete when:

☑ Desired Count = 1

☑ Minimum Capacity = 1

☑ Maximum Capacity = 3

☑ Target Tracking Configured

☑ CPU Target = 60%

☑ Scale-Out Under High Load

☑ Scale-In Under Low Load

☑ Cooldown Understood

☑ CloudWatch Metrics Verified
Key Concepts Summary
Concept	Meaning
Desired Capacity	Number of tasks ECS attempts to keep running
Minimum Capacity	Lowest allowed task count
Maximum Capacity	Highest allowed task count
Target Tracking	Automatically maintains a target metric value
Scale-Out	Add tasks when utilization increases
Scale-In	Remove tasks when utilization decreases
Cooldown	Wait period between scaling actions
Conclusion

ECS Service Auto Scaling automatically adjusts application capacity based on workload demand. By configuring a minimum capacity of 1, a maximum capacity of 3, and a target tracking policy with a CPU utilization target of 60%, the ECS service can scale out during periods of high demand and scale in when demand decreases. This improves application availability, performance, and cost efficiency while reducing manual operational effort.