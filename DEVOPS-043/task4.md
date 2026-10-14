Identify Unused/Oversized AWS Resources
The goal is to reduce costs, improve security, and eliminate waste without affecting application availability.
1. EC2 Instances
Check Utilization
Open:
CloudWatch
 → Metrics
 → EC2
 → Per-Instance Metrics

Review:
CPUUtilization
NetworkIn
NetworkOut
DiskReadOps
DiskWriteOps

Possible Oversized Instance:
Instance Type: t3.large
Average CPU: 3%
Average Memory: 20%

Recommendation:
t3.large
    ↓
t3.micro or t3.small

Validation:
☐ CPU consistently below 20%
☐ Instance sizing reviewed
☐ Rightsizing opportunity identified

2. ECS Services and Tasks
Review:
ECS
 → Clusters
 → Services

Check:
Desired Count
Running Count
CPU Usage
Memory Usage

Example:
Task CPU Reserved: 1024
Actual Usage: 150

Recommendation:
1024 CPU
   ↓
512 CPU

Validation:
☐ CPU reservation appropriate
☐ Memory reservation appropriate
☐ No idle services

3. Load Balancers
Review:
EC2
 → Load Balancers

Look for:
Unused ALBs
Unused Target Groups
Zero Request Count

CloudWatch Metric:
RequestCount

Potential Issue:
ALB exists
No traffic for 30+ days

Validation:
☐ ALBs actively serving traffic
☐ Unused target groups removed

4. RDS Databases
Check:
RDS
 → Databases

Review:
CPUUtilization
DatabaseConnections
FreeStorageSpace

Example:
db.t3.medium
CPU average: 5%
Connections: 2

Potential Recommendation:
db.t3.medium
      ↓
db.t3.micro

Validation:
☐ Database size justified
☐ Storage utilization reviewed

5. EBS Volumes
Open:
EC2
 → Elastic Block Store
 → Volumes

Look for:
Available

Status:
Volume State = Available

This usually means:
Volume not attached to any instance

Validation:
☐ Unattached volumes identified
☐ Required volumes documented
☐ Unused volumes removed

6. Elastic IP Addresses
Open:
EC2
 → Elastic IPs

Look for:
Unassociated

Unused Elastic IPs may incur charges.
Validation:
☐ All Elastic IPs associated
☐ Unused Elastic IPs released

7. ECR Repositories
Review:
ECR
 → Repositories

Common issue:
v1
v2
v3
...
v100

Old images consume storage.
Recommendation:
Lifecycle Policy
    ↓
Keep latest 10 images
Delete older images

Validation:
☐ Lifecycle policy configured
☐ Old images cleaned up

8. CloudWatch Logs
Review:
CloudWatch
 → Log Groups

Check:
Retention = Never Expire

Potential issue:
Large log storage costs

Recommendation:
Development = 14 days
Testing = 30 days
Production = 90–365 days

Validation:
☐ Retention policies configured
☐ Old logs removed automatically

9. NAT Gateways
Review:
VPC
 → NAT Gateways

Common issue:
NAT Gateway exists
No outbound traffic

Check CloudWatch metrics:
BytesOutToDestination
PacketsOutToDestination

Validation:
☐ NAT Gateway required
☐ Usage verified

10. Snapshots and Backups
Review:
EBS Snapshots
RDS Snapshots

Look for:
Old snapshots
Unused snapshots
Duplicate snapshots

Validation:
☐ Snapshot retention policy defined
☐ Obsolete snapshots removed

AWS Cost Optimization Checklist
☐ Idle EC2 instances identified
☐ ECS CPU/Memory rightsized
☐ Unused ALBs removed
☐ Unused Target Groups removed
☐ RDS sizing reviewed
☐ Unattached EBS volumes removed
☐ Unused Elastic IPs released
☐ ECR lifecycle policies enabled
☐ CloudWatch log retention configured
☐ NAT Gateway usage reviewed
☐ Old snapshots cleaned up

High-Impact Savings Areas
For a training environment like yours, the most common cost sources are:
1. NAT Gateway (often highest unexpected cost)
2. Idle RDS instances
3. Oversized ECS task definitions
4. Unattached EBS volumes
5. CloudWatch log retention
6. Old ECR images
7. Forgotten snapshots
8. Unused Load Balancers

Recommended Audit Commands
If you use AWS CLI:
aws ec2 describe-volumes --filters Name=status,Values=available

aws ec2 describe-addresses

aws ecs list-services --cluster devops-training-cluster

aws rds describe-db-instances

aws ecr describe-repositories

Deliverable Summary
Resource Type      Status      Action
EC2                Review      Right-size if CPU <20%
ECS                Review      Reduce CPU/Memory reservations
ALB                Review      Remove unused ALBs
RDS                Review      Right-size if consistently underutilized
EBS                Review      Delete unattached volumes
EIP                Review      Release unused addresses
ECR                Review      Enable lifecycle policy
CloudWatch Logs    Review      Configure retention
NAT Gateway        Review      Validate actual usage
Snapshots          Review      Apply retention policy

A monthly audit of these resources is a good operational control and often finds the majority of avoidable AWS costs.