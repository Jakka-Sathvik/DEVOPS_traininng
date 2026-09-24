# Day 12 – EC2 Theory

## 1. What is Amazon EC2?
Amazon Elastic Compute Cloud (EC2) is a core AWS service that provides resizable virtual servers ("compute capacity") in the cloud. Instead of buying and maintaining physical hardware, you rent virtual machines on demand and pay only for what you use.

## 2. Why Do Companies Use EC2?
- **No upfront hardware cost** – pay-as-you-go pricing
- **Elastic scaling** – add or remove capacity as demand changes
- **Fast provisioning** – new servers ready in minutes, not weeks
- **Global reach** – deploy across multiple AWS regions and availability zones
- **Workload-specific hardware** – instance types tuned for compute, memory, storage, or GPU needs
- **High availability** – build redundant, fault-tolerant architectures
- **Deep AWS integration** – works natively with S3, RDS, VPC, IAM, Load Balancers, etc.

## 3. What is an EC2 Instance?
An EC2 instance is a single virtual server running in AWS. It's the actual compute unit you launch — with CPU, memory, storage, and networking defined by its instance type, running the OS/software from the AMI it was launched with.

## 4. What is an AMI?
An **Amazon Machine Image (AMI)** is a template used to launch an instance. It bundles the operating system plus any pre-installed software or configuration. AWS provides ready-made AMIs (Amazon Linux, Ubuntu, Windows Server, etc.), and you can also create and save your own custom AMIs from a configured instance.

## 5. What is an Instance Type?
The instance type defines an instance's hardware profile — vCPUs, RAM, storage, and network performance. Types are grouped into families by workload:

| Family | Optimized For | Example Types |
|---|---|---|
| General Purpose | Balanced compute/memory/network | t3, m5 |
| Compute Optimized | CPU-intensive workloads | c5 |
| Memory Optimized | Large in-memory datasets | r5 |
| Storage Optimized | High-speed local disk I/O | i3 |
| Accelerated Computing | GPU/ML workloads | p3, g4 |

## 6. What is an EBS Volume?
**Elastic Block Store (EBS)** is persistent block-level storage attached to an EC2 instance — essentially a virtual hard disk. Unlike instance store (temporary, tied to the instance's life), an EBS volume's data survives independently of the instance's power state. EBS offers several volume types (gp3, io2, st1, sc1) balancing performance and cost.

## 7. What is a Security Group?
A **Security Group** is a virtual firewall attached to an instance that controls inbound and outbound traffic.
- Supports **allow rules only** (no explicit "deny")
- **Stateful** – if you allow inbound traffic, the matching outbound response is automatically allowed
- Rules are defined by protocol, port range, and source/destination (IP or another security group)

## 8. What is a Key Pair?
A **Key Pair** is the credential AWS uses to secure access to an instance: a public key (stored on the instance) and a private key (downloaded once, kept by you). It's used to SSH into Linux instances or decrypt the administrator password on Windows instances. If you lose the private key, AWS cannot recover it for you.

## 9. What is a Public IP?
A **Public IP** is an internet-routable address that lets an instance be reached from outside AWS. By default, EC2 assigns a **dynamic** public IP that changes each time the instance stops and restarts. For a fixed, unchanging address, you'd attach an **Elastic IP** instead.

## 10. Stop vs. Reboot vs. Terminate

| Action | Effect | Billing | Data |
|---|---|---|---|
| **Reboot** | OS restarts; instance stays on the same host | Compute billing continues | EBS data, IP, instance ID all unchanged |
| **Stop** | Instance shuts down (EBS-backed instances only) | Compute billing stops; EBS storage still billed | EBS data preserved; public IP released (unless using Elastic IP) |
| **Terminate** | Instance permanently deleted | All billing stops | EBS volumes deleted by default (unless "Delete on Termination" is disabled); cannot be restarted |

---
*Day 12 – EC2 Theory Notes*
