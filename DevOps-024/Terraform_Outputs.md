 Terraform Outputs
1. Objective

Learn how Terraform outputs make important infrastructure information available after deployment.

We will create at least 5 meaningful outputs:

VPC ID
Subnet ID
Security Group ID
EC2 Instance ID
EC2 Private IP
2. What is a Terraform Output?

An output displays information about infrastructure after Terraform creates or reads it.

For example:

output "vpc_id" {
  value = aws_vpc.training_vpc.id
}

After:

terraform apply

Terraform can display:

vpc_id = "vpc-xxxxxxxx"
3. Create outputs.tf

Your project should already contain:

terraform-network-lab/
│
├── provider.tf
├── variables.tf
├── terraform.tfvars
├── network.tf
├── security-group.tf
├── ec2.tf
└── outputs.tf

Open:

outputs.tf

Replace/add the following outputs.

Output 1 — VPC ID
output "vpc_id" {
  description = "ID of the training VPC"
  value       = aws_vpc.training_vpc.id
}

This displays the unique ID of the VPC.

Example:

vpc_id = "vpc-0123456789abcdef0"
4. Output 2 — Subnet ID
output "subnet_id" {
  description = "ID of the training subnet"
  value       = aws_subnet.training_subnet.id
}

Example:

subnet_id = "subnet-0123456789abcdef0"
5. Output 3 — Security Group ID
output "security_group_id" {
  description = "ID of the training security group"
  value       = aws_security_group.training_sg.id
}

Example:

security_group_id = "sg-0123456789abcdef0"
6. Output 4 — EC2 Instance ID
output "instance_id" {
  description = "ID of the training EC2 instance"
  value       = aws_instance.training_server.id
}

Example:

instance_id = "i-0123456789abcdef0"
7. Output 5 — EC2 Private IP
output "instance_private_ip" {
  description = "Private IP address of the EC2 instance"
  value       = aws_instance.training_server.private_ip
}

Example:

instance_private_ip = "10.0.1.25"
8. Add More Useful Outputs

You can also add these optional outputs.

Instance Name
output "instance_name" {
  description = "Name of the EC2 instance"
  value       = aws_instance.training_server.tags["Name"]
}
Instance Type
output "instance_type" {
  description = "Type of the EC2 instance"
  value       = aws_instance.training_server.instance_type
}
AWS Region
output "aws_region" {
  description = "AWS region used for the deployment"
  value       = var.aws_region
}
9. Complete outputs.tf

Your complete file can look like this:

output "vpc_id" {
  description = "ID of the training VPC"
  value       = aws_vpc.training_vpc.id
}

output "subnet_id" {
  description = "ID of the training subnet"
  value       = aws_subnet.training_subnet.id
}

output "security_group_id" {
  description = "ID of the training security group"
  value       = aws_security_group.training_sg.id
}

output "instance_id" {
  description = "ID of the training EC2 instance"
  value       = aws_instance.training_server.id
}

output "instance_private_ip" {
  description = "Private IP address of the EC2 instance"
  value       = aws_instance.training_server.private_ip
}

output "instance_name" {
  description = "Name of the EC2 instance"
  value       = aws_instance.training_server.tags["Name"]
}

output "instance_type" {
  description = "Type of the EC2 instance"
  value       = aws_instance.training_server.instance_type
}

output "aws_region" {
  description = "AWS region used for the deployment"
  value       = var.aws_region
}

You now have 8 outputs.

10. Format the Configuration

Run:

terraform fmt
11. Validate

Run:

terraform validate

Expected:

Success! The configuration is valid.
12. Apply

If your infrastructure has not been created yet:

terraform apply

Enter:

yes

After deployment, Terraform will display your outputs.

You should see something similar to:

Outputs:

aws_region = "us-east-1"

instance_id = "i-0123456789abcdef0"

instance_name = "terraform-training-server"

instance_private_ip = "10.0.1.25"

instance_type = "t3.micro"

security_group_id = "sg-0123456789abcdef0"

subnet_id = "subnet-0123456789abcdef0"

vpc_id = "vpc-0123456789abcdef0"

Your actual IDs and IP address will be different.

13. Run terraform output

The task specifically asks you to use:

terraform output

Run it after deployment.

You should get:

aws_region = "us-east-1"
instance_id = "i-xxxxxxxx"
instance_name = "terraform-training-server"
instance_private_ip = "10.0.1.x"
instance_type = "t3.micro"
security_group_id = "sg-xxxxxxxx"
subnet_id = "subnet-xxxxxxxx"
vpc_id = "vpc-xxxxxxxx"
14. Get One Specific Output

You can also request a single output.

For example:

terraform output vpc_id

Result:

"vpc-0123456789abcdef0"

For the EC2 private IP:

terraform output instance_private_ip

Result:

"10.0.1.25"
15. Understand the Output Reference

Look at:

value = aws_vpc.training_vpc.id

Breakdown:

aws_vpc
   │
   ▼
training_vpc
   │
   ▼
id

Terraform retrieves the VPC's ID and exposes it as an output.

Similarly:

value = aws_instance.training_server.private_ip

means:

aws_instance
      │
      ▼
training_server
      │
      ▼
private_ip
16. Architecture With Outputs
                 VPC
                  │
              ┌───┴───┐
              │       │
              ▼       ▼
           Subnet   Security
                      Group
              │       │
              └───┬───┘
                  ▼
                 EC2
                  │
                  ▼
             Private IP


Terraform Outputs
       │
       ├── VPC ID
       ├── Subnet ID
       ├── Security Group ID
       ├── EC2 Instance ID
       └── Private IP
17. Why Are Outputs Useful?

Outputs are useful because they make important infrastructure information easy to access.

For example:

VPC ID

Useful when another Terraform configuration needs to use the VPC.

Subnet ID

Useful when deploying another resource into the subnet.

Security Group ID

Useful when attaching the Security Group to another EC2 instance.

EC2 Instance ID

Useful for identifying the server.

Private IP

Useful for communication with the EC2 instance inside the VPC.

18. Deliverable
Minimum required outputs
Output	Terraform Value	Purpose
VPC ID	aws_vpc.training_vpc.id	Identify VPC
Subnet ID	aws_subnet.training_subnet.id	Identify subnet
Security Group ID	aws_security_group.training_sg.id	Identify firewall
EC2 Instance ID	aws_instance.training_server.id	Identify EC2
Private IP	aws_instance.training_server.private_ip	Identify internal EC2 IP
Additional outputs
✅ Instance Name
✅ Instance Type
✅ AWS Region
19. Commands to Complete TASK 8

Run these in order:

terraform fmt
terraform validate
terraform plan
terraform apply

Then:

terraform output

And test an individual output:

terraform output vpc_id