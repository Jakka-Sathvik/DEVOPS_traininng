Day 30 – AWS Systems Manager Fundamentals
Introduction

AWS Systems Manager (SSM) is a service that helps administrators securely manage AWS resources. One of the most widely used features of Systems Manager is Session Manager, which allows secure access to EC2 instances without using traditional SSH access.

In many organizations, managing SSH keys, opening network ports, and tracking administrator activity can become difficult. Session Manager addresses these challenges by providing centralized, secure, and auditable access to EC2 instances.

What is AWS Systems Manager?

AWS Systems Manager is a management service that helps administrators:

Manage EC2 instances
Run commands remotely
Access servers securely
Automate operational tasks
Track configuration changes
Monitor infrastructure

Systems Manager provides a central location for managing AWS resources.

What is Session Manager?

Session Manager is a capability of AWS Systems Manager that allows administrators to connect to EC2 instances securely through the AWS Console, AWS CLI, or AWS SDK.

Instead of connecting through SSH, administrators establish a secure session through AWS Systems Manager.

Traditional EC2 Access Using SSH

For many years, administrators connected to Linux servers using SSH.

Architecture:

Developer/Admin
      │
      ▼
 SSH Client
      │
      ▼
 Internet
      │
      ▼
EC2 Instance

Example Command:

ssh -i my-key.pem ec2-user@10.0.1.15
SSH Connection Process
1. Create SSH Key Pair
2. Download Private Key (.pem)
3. Launch EC2 with Key Pair
4. Open Port 22 in Security Group
5. Connect Using SSH
Problems with SSH
1. SSH Key Management

Administrators must manage private keys.

Example:

developer1.pem
developer2.pem
admin.pem
production.pem

Problems:

Lost keys
Shared keys
Expired keys
Uncontrolled distribution
2. Port 22 Must Be Open

Security Group:

Inbound Rule

Port: 22
Protocol: TCP
Source: Admin IP

Risks:

Brute-force attacks
Port scanning
Unauthorized access attempts
3. Difficult Auditing

Questions become difficult to answer:

Who logged in?
What commands were executed?
When did they connect?

SSH alone does not provide centralized auditing.

4. Administrative Overhead

Administrators must:

Manage keys
Rotate keys
Distribute keys
Remove old keys
Track access manually

This becomes difficult in large organizations.

AWS Systems Manager Session Manager

Session Manager provides secure shell access without traditional SSH requirements.

Architecture:

Developer
      │
      ▼
AWS Console / AWS CLI
      │
      ▼
AWS Systems Manager
      │
      ▼
SSM Agent
      │
      ▼
EC2 Instance
How Session Manager Works

Session Manager uses:

IAM
Systems Manager
SSM Agent
Secure AWS APIs

Instead of:

SSH Keys
Port 22
Public IP
Session Manager Connection Flow
Administrator
      │
      ▼
IAM Authentication
      │
      ▼
AWS Systems Manager
      │
      ▼
SSM Agent
      │
      ▼
EC2 Instance

The connection remains inside AWS-managed secure channels.

SSM Agent

The SSM Agent runs on the EC2 instance.

Responsibilities:

Receive commands
Create sessions
Send logs
Communicate with AWS Systems Manager

Most modern Amazon Linux AMIs already include the SSM Agent.

IAM-Controlled Access

Session Manager uses IAM permissions.

Example:

Developer
      │
      ▼
IAM Policy
      │
      ▼
Session Manager Access

Instead of:

SSH Key

Access decisions are controlled centrally through IAM.

Example IAM Permission
{
  "Effect": "Allow",
  "Action": [
    "ssm:StartSession"
  ],
  "Resource": "*"
}

Meaning:

User can start Session Manager sessions.
No SSH Keys Required

Traditional SSH:

Private Key Required

Session Manager:

No Private Key Required

Benefits:

No key storage
No key sharing
No key rotation
No key distribution
No Public IP Required

Traditional SSH often requires:

Public IP
Internet Access

Session Manager can work with:

Private EC2 Instances
Private Subnets

This improves security.

No Port 22 Required

SSH Architecture:

Security Group

22/TCP Open

Session Manager Architecture:

Port 22 Closed

Benefits:

Smaller attack surface
Improved security
Reduced exposure
Auditing and Logging

One major advantage of Session Manager is auditing.

AWS can record:

Who connected
When they connected
How long the session lasted
Commands executed
CloudTrail Integration

Session Manager integrates with CloudTrail.

Example:

User: DevOpsAdmin
Action: StartSession
Instance: i-123456
Time: 10:15 AM

CloudTrail records the event.

Session Logging

Session logs can be stored in:

CloudWatch Logs
Amazon S3

This allows organizations to review activity later.

Security Comparison
Feature	SSH	Session Manager
SSH Key Required	Yes	No
Port 22 Required	Yes	No
Public IP Required	Often Yes	No
IAM Integration	No	Yes
Centralized Access	Limited	Yes
Session Auditing	Limited	Yes
CloudTrail Logging	No	Yes
Easier Compliance	Moderate	High
Key Rotation Required	Yes	No
Attack Surface	Larger	Smaller
Real-World Example

Traditional Environment:

Developer
     │
     ▼
SSH Key
     │
     ▼
Port 22
     │
     ▼
EC2

Problems:

Lost key
Open SSH port
Manual auditing
Key management overhead

SSM Environment:

Developer
     │
     ▼
IAM Login
     │
     ▼
Systems Manager
     │
     ▼
EC2

Benefits:

No SSH keys
No open port 22
Centralized permissions
Auditable access
Best Practices
Use IAM Roles

Attach IAM roles to EC2 instances.

EC2
 │
 ▼
IAM Role
 │
 ▼
Systems Manager
Keep SSM Agent Updated

Ensure:

Latest SSM Agent Version

for security and reliability.

Enable CloudTrail

Record:

Session Start
Session End
User Activity
Store Session Logs

Send logs to:

CloudWatch Logs
or
Amazon S3

for auditing.

Follow Least Privilege

Grant only required permissions.

Example:

ssm:StartSession

instead of:

AdministratorAccess
Key Learning Summary
Systems Manager
Centralized AWS management service.
Session Manager
Secure access to EC2 without SSH.
IAM
Controls who can start sessions.
SSM Agent
Runs on EC2 and enables communication with Systems Manager.
CloudTrail
Records Session Manager activity.
CloudWatch Logs
Stores session logs for auditing.
Conclusion

AWS Systems Manager Session Manager provides a modern, secure alternative to traditional SSH access. Instead of managing SSH keys and opening port 22, organizations can use IAM-controlled access through Systems Manager. This approach improves security, simplifies administration, enables centralized access control, and provides comprehensive auditing through CloudTrail and CloudWatch Logs. For production AWS environments, Session Manager is considered a best practice for secure EC2 administration.