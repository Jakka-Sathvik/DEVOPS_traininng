TASK 8 — NAT Gateway
Objective

In this task, we will create and configure an AWS NAT Gateway using Terraform.

The NAT Gateway allows resources in private subnets to make outbound internet connections without giving those private resources direct internet access.

For this training environment, we will use one NAT Gateway to keep the architecture simple and reduce unnecessary cost.

1. Understand the Architecture

The traffic flow is:

Private EC2
     │
     ▼
Private Subnet
     │
     ▼
Private Route Table
     │
     │ 0.0.0.0/0
     ▼
NAT Gateway
     │
     ▼
Elastic IP
     │
     ▼
Internet Gateway
     │
     ▼
Internet

The important concept is:

Private Subnet
       ↓
NAT Gateway
       ↓
Internet Gateway
       ↓
Internet

The private subnet does not directly use the Internet Gateway.

2. Why Do We Need a NAT Gateway?

Suppose an application server is running in a private subnet.

Private EC2
10.0.11.x

The server may need to:

Download Linux packages
Install security updates
Download application dependencies
Access external APIs
Connect to external services

However, we don't want the server to be directly accessible from the internet.

Therefore:

Internet
   X
   │
Private EC2

Instead, outbound traffic goes through the NAT Gateway:

Private EC2
     │
     ▼
NAT Gateway
     │
     ▼
Internet
3. NAT Gateway Must Be in a Public Subnet

A NAT Gateway should be placed in a public subnet.

Our architecture:

VPC
│
├── Public Subnet
│     │
│     └── NAT Gateway
│
└── Private Subnet
      │
      └── Private EC2

The NAT Gateway uses the public subnet's route to reach the Internet Gateway.

4. Create an Elastic IP

A NAT Gateway requires a public Elastic IP address for internet communication.

In:

terraform/modules/vpc/main.tf

add:

resource "aws_eip" "nat" {
  domain = "vpc"

  tags = {
    Name        = "${var.environment}-nat-eip"
    Environment = var.environment
  }
}
Explanation
domain = "vpc"

associates the Elastic IP with the VPC networking model.

5. Create the NAT Gateway

Add:

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id

  tags = {
    Name        = "${var.environment}-nat-gateway"
    Environment = var.environment
  }

  depends_on = [
    aws_internet_gateway.igw
  ]
}
Important Lines
allocation_id = aws_eip.nat.id

Associates the Elastic IP with the NAT Gateway.

And:

subnet_id = aws_subnet.public[0].id

places the NAT Gateway in the first public subnet.

6. Why Use One NAT Gateway?

For a production architecture, you may deploy a NAT Gateway in each Availability Zone.

Example:

AZ-1                    AZ-2
 │                       │
NAT Gateway             NAT Gateway

However, NAT Gateways incur charges.

For this training environment, we can use:

AZ-1
 │
 └── One NAT Gateway

and allow both private subnets to use it.

This keeps the training architecture smaller and easier to manage.

Production note: A single NAT Gateway creates a dependency on one AZ. For higher availability, production architectures often use a NAT Gateway per AZ.

7. Create the Private Route

The private route table should send internet-bound traffic to the NAT Gateway.

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }

  tags = {
    Name        = "${var.environment}-private-route-table"
    Environment = var.environment
  }
}

The important configuration is:

0.0.0.0/0
     ↓
NAT Gateway
8. Associate Private Subnets

Now associate both private subnets with the private route table.

resource "aws_route_table_association" "private" {
  count = length(aws_subnet.private)

  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}

Our two private subnets are:

10.0.11.0/24
10.0.12.0/24

Both use the private route table.

9. Complete NAT Architecture
                         INTERNET
                            ▲
                            │
                     Internet Gateway
                            ▲
                            │
                       Elastic IP
                            ▲
                            │
                      NAT Gateway
                            ▲
                            │
                    Public Subnet AZ-1
                            ▲
                            │
                    Public Route Table
                            │
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
       Private Subnet AZ-1         Private Subnet AZ-2
        10.0.11.0/24                10.0.12.0/24
              │                           │
              └─────────────┬─────────────┘
                            │
                    Private Route Table
                            │
                         0.0.0.0/0
                            │
                            ▼
                       NAT Gateway
10. Check Terraform Configuration

Your VPC module should now contain these major components:

modules/vpc/
│
├── main.tf
├── variables.tf
└── outputs.tf

main.tf should contain resources for:

VPC
Internet Gateway
Public Subnets
Private Subnets
Elastic IP
NAT Gateway
Public Route Table
Private Route Table
Public Route Associations
Private Route Associations
11. Format the Code

From the root Terraform directory:

terraform fmt

This automatically formats the Terraform files.

12. Validate the Configuration

Run:

terraform validate

Expected result:

Success! The configuration is valid.

If you receive an error, check the resource name and line number shown by Terraform.

13. Initialize Terraform

If required:

terraform init

Terraform will initialize the AWS provider and modules.

14. Generate the Plan

Run:

terraform plan

Look for:

+ aws_eip.nat
+ aws_nat_gateway.nat
+ aws_route_table.private
+ aws_route_table_association.private[0]
+ aws_route_table_association.private[1]

You should also see the VPC, subnets, Internet Gateway, and public route table if they have not already been created.

15. Verify the Route

The private route table should look conceptually like:

Destination       Target
--------------------------------
10.0.0.0/16       local
0.0.0.0/0         NAT Gateway

It should not look like:

0.0.0.0/0 → Internet Gateway

That would provide a direct internet route from the private subnet.

16. Cost Warning

NAT Gateway is not free.

AWS charges can include NAT Gateway usage and data processing.

Therefore, because this is a training environment:

Use a small architecture
1 VPC
2 Public Subnets
2 Private Subnets
1 NAT Gateway
Do not create unnecessary NAT Gateways

For this exercise:

1 NAT Gateway

is sufficient.

Destroy resources after training

When you have finished the exercise, run:

terraform destroy

Terraform will ask for confirmation.

Enter:

yes

Only do this after you have completed all required training tasks and no longer need the AWS resources.

17. Recommended Training Workflow
Write Terraform
      │
      ▼
terraform fmt
      │
      ▼
terraform validate
      │
      ▼
terraform plan
      │
      ▼
Review resources
      │
      ▼
terraform apply
      │
      ▼
Test connectivity
      │
      ▼
terraform destroy

For this particular task, don't apply the infrastructure unless your trainer/task requires actual AWS resources.

Deliverable
NAT Gateway
Created:
✓ Elastic IP
✓ NAT Gateway
✓ NAT Gateway placed in Public Subnet
Private Routing
Private Subnet
      │
      ▼
Private Route Table
      │
      ▼
0.0.0.0/0
      │
      ▼
NAT Gateway
      │
      ▼
Internet Gateway
      │
      ▼
Internet
Commands
terraform fmt
terraform validate
terraform plan
Final Checklist
✅ NAT Gateway configured
✅ Elastic IP configured
✅ NAT Gateway placed in a public subnet
✅ Private route table points to NAT Gateway
✅ Both private subnets associated with private route table
✅ Private subnets do not directly route to Internet Gateway
✅ Terraform configuration validated
✅ Terraform plan reviewed
⚠️ Destroy training resources when finished to avoid unnecessary AWS charges