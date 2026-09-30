ECS Auto Scaling Testing and Validation
Objective

Validate that ECS Service Auto Scaling is functioning correctly by generating application traffic, observing CPU utilization increases, and verifying automatic scale-out and scale-in events.

Training configuration:

Minimum Capacity = 1
Desired Capacity = 1
Maximum Capacity = 3

CPU Target = 60%

Expected behavior:

1 Task
  ↓
2 Tasks
  ↓
3 Tasks

Then:

3 Tasks
  ↓
2 Tasks
  ↓
1 Task
1. Test Architecture
Application Traffic
        │
        ▼
      ECS Task
        │
        ▼
  CPU Utilization
        │
        ▼
   CloudWatch
        │
        ▼
Auto Scaling Policy
        │
        ▼
   ECS Service
        │
        ▼
 Task Count Changes
2. Verify Initial State

Navigate:

ECS
 ↓
Cluster
 ↓
Service

Expected:

Running Tasks = 1
Desired Count = 1

Architecture:

ECS Service
     │
     ▼
   Task 1

Take Screenshot:

Initial Task Count

Evidence #1

3. Verify Auto Scaling Configuration

Navigate:

ECS
 ↓
Cluster
 ↓
Service
 ↓
Auto Scaling

Verify:

Minimum = 1
Maximum = 3
Target CPU = 60%

Take Screenshot:

Auto Scaling Configuration

Evidence #2

4. Generate Application Traffic

The objective is to increase CPU utilization.

Possible methods:

Method 1: Browser Refresh

Open application URL.

Continuously refresh:

F5

multiple times.

Method 2: Load Testing Tool

Example:

hey
ab
curl loop
Method 3: Shell Loop

Example:

while true
do
  curl http://YOUR-ALB-DNS
done

Stop with:

CTRL + C
5. Observe CPU Utilization

Navigate:

CloudWatch
 ↓
Metrics
 ↓
ECS

Monitor:

CPUUtilization

Expected:

CPU > 60%

Architecture:

Traffic
   ↓
CPU Increase
   ↓
CloudWatch Metric

Take Screenshot:

CloudWatch CPU Graph

Evidence #3

6. Scale-Out Event

When CPU exceeds target:

CPU > 60%

Auto Scaling evaluates:

Need More Capacity

ECS creates another task.

Expected:

1 Task
  ↓
2 Tasks

Architecture:

Task 1
   │
CPU High
   ▼
Task 1
Task 2

Take Screenshot:

Two Running Tasks

Evidence #4

7. Continue Load Generation

Keep traffic running.

Expected:

CPU Remains High

Auto Scaling triggers again.

Expected:

2 Tasks
   ↓
3 Tasks

Architecture:

Task 1
Task 2
      │
CPU High
      ▼
Task 1
Task 2
Task 3

Take Screenshot:

Three Running Tasks

Evidence #5

8. Verify Maximum Capacity

Configured:

Maximum Capacity = 3

Expected:

No Additional Tasks

Even if CPU remains high:

Task Count = 3

This confirms maximum capacity enforcement.

Take Screenshot:

Maximum Capacity Reached

Evidence #6

9. Review ECS Service Events

Navigate:

ECS
 ↓
Cluster
 ↓
Service
 ↓
Events

Expected events:

Service scaled from 1 to 2 tasks

Service scaled from 2 to 3 tasks

Take Screenshot:

Scale-Out Events

Evidence #7

10. Stop Traffic

Terminate load generation.

Example:

CTRL + C

Traffic becomes:

Low

CPU begins decreasing.

11. Observe CPU Reduction

CloudWatch should show:

CPU < 60%

Architecture:

Traffic Stops
      │
      ▼
CPU Drops

Take Screenshot:

CPU Decrease Graph

Evidence #8

12. Scale-In Event

Auto Scaling evaluates:

Excess Capacity

Expected:

3 Tasks
   ↓
2 Tasks

Architecture:

Task 1
Task 2
Task 3
      │
CPU Low
      ▼
Task 1
Task 2

Take Screenshot:

Two Running Tasks

Evidence #9

13. Final Scale-In

After cooldown period:

Expected:

2 Tasks
   ↓
1 Task

Architecture:

Task 1
Task 2
      │
CPU Low
      ▼
Task 1

Minimum capacity reached.

Take Screenshot:

Single Running Task

Evidence #10

14. Verify Minimum Capacity

Configured:

Minimum Capacity = 1

Expected:

Task Count Never Drops Below 1

This confirms proper minimum capacity enforcement.

15. Cooldown Behavior Observation

Observe delay between scaling events.

Example:

Scale Out
     ↓
Cooldown
     ↓
Reevaluate

And:

Scale In
     ↓
Cooldown
     ↓
Reevaluate

Purpose:

Prevent Rapid Scaling

Expected learning:

Scaling is not immediate.
ECS waits before additional actions.
Cooldown prevents instability.
16. CloudWatch Verification

Review CPU graph.

Expected pattern:

CPU Rise
    ↓
Scale Out
    ↓
CPU Drop

Then:

Traffic Stops
      ↓
CPU Drop
      ↓
Scale In

Take Screenshot:

Complete CPU Timeline

Evidence #11

17. Evidence Checklist

Capture screenshots of:

☑ Initial Task Count = 1

☑ Auto Scaling Configuration

☑ CloudWatch CPU Increase

☑ Scale Out To 2 Tasks

☑ Scale Out To 3 Tasks

☑ ECS Scaling Events

☑ CPU Reduction

☑ Scale In To 2 Tasks

☑ Scale In To 1 Task

☑ CloudWatch Timeline
18. Expected Demonstration Flow
Initial State
      │
      ▼
1 Task
      │
Generate Traffic
      ▼
CPU > 60%
      │
      ▼
2 Tasks
      │
CPU Still High
      ▼
3 Tasks
      │
Stop Traffic
      ▼
CPU Drops
      │
      ▼
2 Tasks
      │
Cooldown
      ▼
1 Task
19. Success Criteria

The test is successful when:

☑ Initial task count = 1

☑ CPU exceeds 60%

☑ ECS scales to 2 tasks

☑ ECS scales to 3 tasks

☑ ECS service events confirm scale-out

☑ Traffic removed

☑ CPU decreases

☑ ECS scales to 2 tasks

☑ ECS scales to 1 task

☑ ECS service events confirm scale-in
Conclusion

ECS Service Auto Scaling should automatically increase task count when CPU utilization exceeds the configured target of 60% and decrease task count when utilization falls below the target. By generating traffic, monitoring CloudWatch metrics, observing ECS service events, and capturing screenshots of scale-out and scale-in activities, the trainee can verify that Auto Scaling is functioning correctly and that minimum and maximum capacity limits are being enforced.
