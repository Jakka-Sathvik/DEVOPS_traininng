TASK 13 — Terraform Plan & Infrastructure Validation

This task is where you validate the complete Terraform configuration, review the dependency graph, deploy the infrastructure, and verify it in AWS.

Your workflow is:

terraform fmt
      ↓
terraform validate
      ↓
terraform plan
      ↓
Review dependency graph
      ↓
terraform apply
      ↓
Verify in AWS
Step 1 — Go to the Terraform project

In your Codespace:

cd /workspaces/terraform-lab

If your actual Terraform project is inside another directory, first enter that directory.

Check:

ls

You should see something similar to:

main.tf
outputs.tf
variables.tf
modules/
Step 2 — Run Terraform Format

Run:

terraform fmt
Why?

terraform fmt automatically formats Terraform files into the standard Terraform style.

You may see:

main.tf
modules/ec2/main.tf
modules/ec2/variables.tf
modules/vpc/main.tf

If nothing is displayed, that is also fine—the files may already be formatted.

Step 3 — Validate Terraform

Run:

terraform validate

Expected result:

Success! The configuration is valid.

This checks the Terraform configuration syntax and internal configuration structure.

It can identify problems such as:

Invalid resource reference
Missing required argument
Invalid module input
Invalid module output
Incorrect variable
Important

terraform validate does not create AWS resources.

It only checks the configuration.

Step 4 — Run Terraform Plan

Now run:

terraform plan

Terraform will calculate what it wants to create, change, or destroy.

You should see resources from your previous tasks.

For example:

Plan: XX to add, 0 to change, 0 to destroy.

The exact number depends on what already exists in your state.

Step 5 — Understand the dependency graph

Your architecture should now be understood as:

                    VPC
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
    Public Subnets        Private Subnets
          │                     │
          ▼                     ▼
   Internet Gateway        NAT Gateway
          │                     │
          ▼                     ▼
   Public Route Table     Private Route Table
          │                     │
          ▼                     ▼
    Security Group       Private Resources
          │
          ▼
         EC2

A simplified dependency chain from the task is:

VPC
 ↓
Subnets
 ↓
Internet Gateway
 ↓
Route Tables
 ↓
NAT Gateway
 ↓
Security Group
 ↓
EC2

However, remember that these aren't always one strictly linear chain.

For example:

VPC
 ├── Public Subnets
 │       ├── Public Route Table
 │       └── NAT Gateway
 │
 ├── Private Subnets
 │       └── Private Route Table
 │                 │
 │                 └── NAT Gateway
 │
 └── Security Group
           │
           ▼
          EC2

Terraform creates the correct ordering based on resource references.

Step 6 — How Terraform knows the dependencies

For example, your EC2 module has:

subnet_id = module.vpc.public_subnet_ids[0]

Terraform understands:

EC2
 ↓
needs public_subnet_ids
 ↓
VPC module
 ↓
creates public subnet

Similarly:

security_group_id = module.vpc.application_security_group_id

creates another dependency:

EC2
 ↓
needs security group
 ↓
VPC module
 ↓
creates security group

Therefore Terraform automatically builds a dependency graph.

You normally do not need to manually add:

depends_on = [module.vpc]

when Terraform can already determine the dependency through references.

Step 7 — Review the Plan carefully

Before applying, look for:

+ create
~ update
- destroy
+

Means Terraform plans to create the resource.

Example:

+ aws_instance.application
~

Means Terraform plans to modify an existing resource.

-

Means Terraform plans to destroy a resource.

For this task, be careful if you see unexpected:

- destroy

or:

~ update

Review the resource before continuing.

Step 8 — Check the EC2 portion

In the plan, confirm that the EC2 has:

AMI
Instance type = t3.micro
Subnet ID
Security Group
Tags

Conceptually:

EC2
├── AMI → Amazon Linux
├── Type → t3.micro
├── Subnet → Public Subnet
├── Security Group → Application SG
└── Tags → dev-application-server
Step 9 — Check the network

Confirm that your plan contains your networking components:

VPC
│
├── Public Subnet 1
├── Public Subnet 2
│
├── Private Subnet 1
├── Private Subnet 2
│
├── Internet Gateway
│
├── Public Route Table
│
├── Private Route Table
│
├── NAT Gateway
│
└── Elastic IP

And your security group:

Security Group
├── HTTP 80
├── HTTPS 443
├── SSH 22 from restricted CIDR
└── Outbound traffic
Step 10 — Apply the infrastructure

Once you have reviewed the plan and there are no unexpected changes, run:

terraform apply

Terraform will show the proposed changes again.

You will see:

Do you want to perform these actions?

Enter:

yes

Terraform will begin creating the infrastructure.

Step 11 — Wait for completion

Terraform may take several minutes because a NAT Gateway can take some time to become available.

Eventually you should see:

Apply complete! Resources: XX added, 0 changed, 0 destroyed.

The exact number depends on your current Terraform state.

Step 12 — Check Terraform outputs

Run:

terraform output

You should see outputs such as:

ec2_instance_id = "i-xxxxxxxxxxxxxxxxx"
ec2_public_ip   = "xx.xx.xx.xx"
ec2_private_ip  = "10.0.1.x"
vpc_id          = "vpc-xxxxxxxx"

You can also check individually:

terraform output ec2_instance_id
terraform output ec2_public_ip
Step 13 — Verify EC2 in AWS

Open:

AWS Console → EC2 → Instances

Find:

dev-application-server

Check:

Property	Expected
State	Running
Instance type	t3.micro
VPC	Your training VPC
Subnet	Public subnet
Security group	Application SG
Public IPv4	Assigned
Private IPv4	10.0.1.x or similar
Step 14 — Verify the VPC

Go to:

AWS Console → VPC → Your VPCs

Check your VPC:

CIDR:
10.0.0.0/16
Step 15 — Verify the subnets

Go to:

VPC → Subnets

You should have approximately:

Public:
10.0.1.0/24
10.0.2.0/24

Private:
10.0.11.0/24
10.0.12.0/24

Across:

ap-south-1a
ap-south-1b
Step 16 — Verify Internet Gateway

Go to:

VPC → Internet Gateways

Confirm that your IGW is attached to your training VPC.

The public route table should contain:

Destination       Target
0.0.0.0/0         Internet Gateway
Step 17 — Verify NAT Gateway

Go to:

VPC → NAT Gateways

Your training architecture currently uses one NAT Gateway to reduce training cost.

It should be located in a public subnet.

The private route table should contain:

Destination       Target
0.0.0.0/0         NAT Gateway
Step 18 — Verify Security Group

Go to:

EC2 → Security Groups

Check your application security group.

Expected inbound rules:

HTTP     TCP 80     0.0.0.0/0
HTTPS    TCP 443    0.0.0.0/0
SSH      TCP 22     YOUR_IP/32

Do not accidentally configure:

SSH → 0.0.0.0/0

for your training security group unless there is a specific reason and you understand the security implications.

Step 19 — Verify Terraform state

Run:

terraform state list

You should see resources such as:

module.vpc.aws_vpc.main
module.vpc.aws_subnet.public[0]
module.vpc.aws_subnet.public[1]
module.vpc.aws_subnet.private[0]
module.vpc.aws_subnet.private[1]
module.vpc.aws_internet_gateway.igw
module.vpc.aws_nat_gateway.nat
module.vpc.aws_route_table.public
module.vpc.aws_route_table.private
module.vpc.aws_security_group.application
module.ec2.aws_instance.application

The exact list depends on your implementation.

This proves Terraform is tracking the infrastructure in its state.

Step 20 — Final architecture

Your completed training environment should look like:

                              INTERNET
                                  │
                                  ▼
                         ┌────────────────┐
                         │ Internet GW    │
                         └───────┬────────┘
                                 │
                ┌────────────────┴────────────────┐
                │                                 │
                ▼                                 ▼
        PUBLIC SUBNET                       PUBLIC SUBNET
        10.0.1.0/24                         10.0.2.0/24
                │
                ├── Route Table
                │       │
                │       └── 0.0.0.0/0 → IGW
                │
                ├── NAT Gateway
                │
                └── EC2
                    t3.micro


        PRIVATE SUBNET                     PRIVATE SUBNET
        10.0.11.0/24                       10.0.12.0/24
                │                                 │
                └────────── NAT Gateway ──────────┘
                              │
                              ▼
                         Internet GW
                              │
                              ▼
                           INTERNET
TASK 13 — Final checklist
Task	Command / Verification	Status
Format	terraform fmt	⬜
Validate	terraform validate	⬜
Plan	terraform plan	⬜
Review dependencies	VPC → Subnets → Routes → SG → EC2	⬜
Apply	terraform apply	⬜
Verify EC2	AWS EC2 Console	⬜
Verify VPC	AWS VPC Console	⬜
Verify subnets	AWS VPC Console	⬜
Verify IGW	AWS VPC Console	⬜
Verify NAT	AWS VPC Console	⬜
Verify route tables	AWS VPC Console	⬜
Verify SG	AWS EC2 Console	⬜
Verify state	terraform state list	⬜
Most important commands

Run these in order:

terraform fmt
terraform validate
terraform plan

Review the plan carefully, then:

terraform apply

After deployment:

terraform output
terraform state list

Finally verify the resources in the AWS Console