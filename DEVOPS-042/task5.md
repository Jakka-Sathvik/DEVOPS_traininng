Failure Simulations
1. ECS Task Failure
Objective: Verify ECS self-healing.
Steps:
ECS Cluster
 → Service
 → Tasks
 → Select Task
 → Stop

Expected:
Task Stops
    ↓
Service Detects Failure
    ↓
New Task Created
    ↓
ALB Health Check Passes
    ↓
Service Restored

Validation:
☐ Desired Count Maintained
☐ New Task Started Automatically
☐ ALB Remained Available
☐ No User Impact

2. Application Failure
Simulate:
Return HTTP 500
OR
Stop Application Process

Expected:
ALB Health Check Fails
      ↓
Target Marked Unhealthy
      ↓
Traffic Sent Only To Healthy Targets

Validation:
☐ Unhealthy Target Removed
☐ Healthy Targets Serve Requests
☐ 5XX Alarm Triggered

3. ECS Instance Failure (EC2 Launch Type)
Stop One EC2 Instance

Expected:
Instance Lost
      ↓
Tasks Rescheduled
      ↓
ASG Launches Replacement

Validation:
☐ ASG Replacement Successful
☐ Service Availability Maintained

4. RDS Failover Test
AWS Console:
RDS
 → Actions
 → Reboot
 → Reboot With Failover

Expected:
Primary DB
      ↓
Standby Promoted
      ↓
Application Reconnects

Validation:
☐ Automatic Failover Completed
☐ Application Recovered
☐ Data Consistent

5. Restore Test
Manual Snapshot
      ↓
Restore New Instance
      ↓
Connect To Database
      ↓
Verify Tables/Data

Validation:
☐ Restore Successful
☐ Data Verified
☐ Recovery Procedure Documented