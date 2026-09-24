Terraform EC2 Infrastructure
1. Objective

Create an EC2 training instance using Terraform.

Final architecture:

AWS
 │
 ▼
VPC
 │
 ▼
Subnet
 │
 ▼
Security Group
 │
 ▼
EC2 Instance

You will use variables for:

AWS Region
Instance Type
Environment
Instance Name
2. Project Structure

Use the same project from TASK 5 and TASK 6:

terraform-network-lab/
│
├── provider.tf
├── variables.tf
├── network.tf
├── security-group.tf
├── ec2.tf
└── outputs.tf
3. Create Variables

Create:

variables.tf

Add:

variable "aws_region" {
  description = "AWS region where infrastructure will be created"
  type        = string
  default     = "us-east-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "training"
}

variable "instance_name" {
  description = "Name of the EC2 instance"
  type        = string
  default     = "terraform-training-server"
}
Why use variables?

Instead of writing:

instance_type = "t3.micro"

everywhere, we can use:

instance_type = var.instance_type

This makes the Terraform configuration reusable.

4. Configure AWS Provider

Open:

provider.tf

Use:

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

Notice:

region = var.aws_region

The AWS region now comes from the variable.

5. Understand the EC2 AMI

An EC2 instance needs an AMI ID.

Instead of hardcoding an AMI ID that may become outdated, use an AWS data source to find a current Amazon Linux 2023 AMI.

Add this to ec2.tf:

data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }

  filter {
    name   = "state"
    values = ["available"]
  }
}

This tells Terraform:

Find the most recent available Amazon Linux 2023 x86_64 AMI.

6. Create the EC2 Instance

In the same ec2.tf, add:

resource "aws_instance" "training_server" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type

  subnet_id = aws_subnet.training_subnet.id

  vpc_security_group_ids = [
    aws_security_group.training_sg.id
  ]

  tags = {
    Name        = var.instance_name
    Environment = var.environment
  }
}
7. Understand the Resource References

This is the most important part of TASK 7.

AMI
ami = data.aws_ami.amazon_linux.id

EC2 uses the AMI retrieved by the data source.

Data Source
     ↓
Amazon Linux AMI
     ↓
EC2
Subnet
subnet_id = aws_subnet.training_subnet.id

EC2 is placed inside the subnet.

VPC
 ↓
Subnet
 ↓
EC2
Security Group
vpc_security_group_ids = [
  aws_security_group.training_sg.id
]

EC2 uses the Security Group created in TASK 6.

Security Group
      ↓
     EC2

Therefore Terraform understands the infrastructure dependencies automatically.

8. Complete Dependency Chain

Your Terraform configuration now represents:

                VPC
                 │
                 ▼
              Subnet
                 │
                 ▼
          Security Group
                 │
                 ▼
                EC2

More accurately, the EC2 has dependencies on both the subnet and security group:

             VPC
              │
              ▼
           Subnet ──────┐
                        │
                        ▼
                    EC2 Instance
                        ▲
                        │
                 Security Group
                        ▲
                        │
                       VPC
9. Add EC2 Outputs

Open:

outputs.tf

Add:

output "instance_id" {
  description = "EC2 instance ID"
  value       = aws_instance.training_server.id
}

output "instance_name" {
  description = "EC2 instance name"
  value       = aws_instance.training_server.tags["Name"]
}

output "instance_private_ip" {
  description = "Private IP address of the EC2 instance"
  value       = aws_instance.training_server.private_ip
}

output "instance_type" {
  description = "EC2 instance type"
  value       = aws_instance.training_server.instance_type
}
10. Final File Structure

Your project should now look like:

terraform-network-lab/
│
├── provider.tf
│
├── variables.tf
│
├── network.tf
│
├── security-group.tf
│
├── ec2.tf
│
└── outputs.tf
11. Initialize Terraform

From your project directory:

terraform init

Terraform will download the AWS provider if it is not already initialized.

12. Format Terraform

Run:

terraform fmt

This makes your Terraform files consistently formatted.

13. Validate

Run:

terraform validate

Expected:

Success! The configuration is valid.
14. Check the Plan

Run:

terraform plan

Terraform should show the resources it intends to create.

If you are starting from the previous tasks, you should see resources such as:

aws_vpc.training_vpc
aws_subnet.training_subnet
aws_internet_gateway.training_igw
aws_route_table.training_route_table
aws_route_table_association.training_association
aws_security_group.training_sg
aws_instance.training_server

The exact plan depends on what you already created.

15. Apply

When the plan looks correct:

terraform apply

Terraform will ask for confirmation.

Enter:

yes

Terraform will create the infrastructure.

16. Check the Output

After successful creation, you should see something similar to:

instance_id = "i-xxxxxxxxxxxxxxxxx"

instance_name = "terraform-training-server"

instance_private_ip = "10.0.1.x"

instance_type = "t3.micro"

The actual values will be different in your AWS account.

17. Verify in AWS

Open the AWS Console and go to:

EC2 → Instances

You should find:

Name:
terraform-training-server

Instance type:
t3.micro

State:
Running

You can also check the instance's:

VPC
Subnet
Security Group
Private IP
AMI
18. Understand the Variables

Your configuration now uses:

Variable
    │
    ├── aws_region
    │
    ├── instance_type
    │
    ├── environment
    │
    └── instance_name
    │
    ▼
Terraform
    │
    ▼
EC2 Infrastructure

For example, changing:

default = "t3.micro"

to:

default = "t3.small"

changes the EC2 instance type.

For a training lab, t3.micro is a reasonable small choice, but actual cost and free-tier eligibility depend on your AWS account, region, and current AWS pricing.

19. Better Way to Change Variables

You can also use a terraform.tfvars file.

Create:

terraform.tfvars

Add:

aws_region    = "us-east-1"
instance_type = "t3.micro"
environment   = "training"
instance_name = "terraform-training-server"

Then Terraform automatically loads these values.

Your structure becomes:

terraform-network-lab/
│
├── provider.tf
├── variables.tf
├── terraform.tfvars
├── network.tf
├── security-group.tf
├── ec2.tf
└── outputs.tf
20. Security Group Important Note

Your EC2 is using the Security Group from TASK 6:

vpc_security_group_ids = [
  aws_security_group.training_sg.id
]

If your Security Group has:

SSH  → Your IP only
HTTP → 0.0.0.0/0

then the EC2 receives those rules.

Do not change SSH to:

0.0.0.0/0

unless there is a specific, controlled training requirement.

21. Deliverable
EC2 Resource
resource "aws_instance" "training_server"
Variables
✅ AWS Region
✅ Instance Type
✅ Environment
✅ Instance Name
Infrastructure
VPC
 ↓
Subnet
 ↓
Security Group
 ↓
EC2
Data Source
data "aws_ami" "amazon_linux"

Used to retrieve the latest suitable Amazon Linux AMI.

Verification
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
22. Final Task 7 Checklist
☑ Create variables.tf
☑ Configure AWS region variable
☑ Configure instance type variable
☑ Configure environment variable
☑ Configure instance name variable
☑ Create EC2 data source for AMI
☑ Create EC2 instance
☑ Connect EC2 to subnet
☑ Connect EC2 to security group
☑ Add EC2 outputs
☑ Run terraform init
☑ Run terraform fmt
☑ Run terraform validate
☑ Run terraform plan
☑ Run terraform apply
☑ Verify EC2 in AWS
✅ TASK 7 Complete

The key concept you should be able to explain is:

VPC
 │
 ├── Subnet ─────────────┐
 │                       │
 └── Security Group ─────┤
                         ▼
                     EC2 Instance
                         │
                         ▼
                    Amazon Linux