TASK 7 — Private Subnet Routing
Objective

In this task, we will configure private subnet routing using Terraform.

The main goal is to allow resources inside private subnets to access the internet outbound through a NAT Gateway, while preventing the private subnet from having a direct route to the Internet Gateway.

The architecture will be:

                    Internet
                       ▲
                       │
                Internet Gateway
                       ▲
                       │
                 NAT Gateway
                       ▲
                       │
              Private Route Table
                       ▲
                       │
                 Private Subnet
                       ▲
                       │
                  Private EC2
Step 1 — Understand Public vs Private Routing
Public subnet
Public Subnet
     │
     ▼
Public Route Table
     │
     ▼
Internet Gateway
     │
     ▼
Internet

The public route table contains:

0.0.0.0/0 → Internet Gateway
Private subnet
Private Subnet
     │
     ▼
Private Route Table
     │
     ▼
NAT Gateway
     │
     ▼
Internet Gateway
     │
     ▼
Internet

The private route table contains:

0.0.0.0/0 → NAT Gateway

Important: The private route table must not use the Internet Gateway as its default route.

Step 2 — Check the NAT Gateway

From Task 3, we already created the NAT Gateway.

Your modules/vpc/main.tf should have:

resource "aws_eip" "nat" {
  domain = "vpc"

  tags = {
    Name = "${var.environment}-nat-eip"
  }
}

Then:

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id

  tags = {
    Name = "${var.environment}-nat-gateway"
  }

  depends_on = [
    aws_internet_gateway.igw
  ]
}
Why is the NAT Gateway in a public subnet?

A NAT Gateway needs a path to the internet.

Therefore:

NAT Gateway
     │
Public Subnet
     │
Internet Gateway
     │
Internet

The private resources use the NAT Gateway, rather than connecting directly to the Internet Gateway.

Step 3 — Create the Private Route Table

Open:

terraform/modules/vpc/main.tf

Add:

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

The important part is:

nat_gateway_id = aws_nat_gateway.nat.id

This means:

0.0.0.0/0
      ↓
NAT Gateway

and not:

0.0.0.0/0
      ↓
Internet Gateway
Step 4 — Associate Private Subnets

Creating the private route table is not enough.

We need to associate the private subnets with it.

Add:

resource "aws_route_table_association" "private" {
  count = length(aws_subnet.private)

  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}

Because we have two private subnets:

Private Subnet 1
10.0.11.0/24
       │
       ▼
Private Route Table

Private Subnet 2
10.0.12.0/24
       │
       ▼
Private Route Table
Step 5 — Understand the Complete Network

Your architecture should now look like:

                           INTERNET
                              ▲
                              │
                    ┌─────────┴─────────┐
                    │ Internet Gateway  │
                    └─────────▲─────────┘
                              │
                    ┌─────────┴─────────┐
                    │   NAT Gateway     │
                    │   Public Subnet   │
                    └─────────▲─────────┘
                              │
                    0.0.0.0/0
                              │
                    ┌─────────┴─────────┐
                    │ Private Route     │
                    │ Table             │
                    └─────────▲─────────┘
                              │
              ┌───────────────┴───────────────┐
              │                               │
              ▼                               ▼
       Private Subnet 1                Private Subnet 2
        10.0.11.0/24                    10.0.12.0/24
              │                               │
              ▼                               ▼
        Private EC2                    Private EC2
Step 6 — Why Doesn't the Private Subnet Use the IGW?

The private subnet should not contain:

0.0.0.0/0 → Internet Gateway

Instead:

0.0.0.0/0 → NAT Gateway

This prevents resources in the private subnet from being directly reachable from the public internet through the Internet Gateway.

For example:

Internet
   X
   │
   X
Private EC2

But outbound traffic can work:

Private EC2
     │
     ▼
NAT Gateway
     │
     ▼
Internet Gateway
     │
     ▼
Internet

This is useful when a private server needs to:

Download software packages
Install updates
Access external APIs
Download application dependencies
Step 7 — Format the Terraform Code

From the root Terraform directory:

terraform fmt

This formats your Terraform files.

Step 8 — Validate

Run:

terraform validate

Expected:

Success! The configuration is valid.

If there is an error, Terraform will show the file and line where the problem occurred.

Step 9 — Initialize Terraform

If necessary:

terraform init

This ensures Terraform has initialized the provider and module.

Step 10 — Generate the Plan

Run:

terraform plan

Look for:

+ aws_route_table.private
+ aws_route_table_association.private[0]
+ aws_route_table_association.private[1]

You should also see the NAT Gateway from the previous task if it has not already been created.

Step 11 — Verify the Route

The private route table should contain:

Destination       Target
--------------------------------
10.0.0.0/16       local
0.0.0.0/0         NAT Gateway

The important point is:

0.0.0.0/0 → NAT Gateway

NOT:

0.0.0.0/0 → Internet Gateway
Step 12 — Final Terraform Code

Your private routing section should look like this:

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

resource "aws_route_table_association" "private" {
  count = length(aws_subnet.private)

  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}
Final Architecture
                         VPC
                          │
          ┌───────────────┴────────────────┐
          │                                │
       Public                           Private
          │                                │
    Public Subnet                    Private Subnet
          │                                │
    Public Route Table              Private Route Table
          │                                │
     0.0.0.0/0                      0.0.0.0/0
          │                                │
          ▼                                ▼
 Internet Gateway                    NAT Gateway
          │                                │
          └───────────────┬────────────────┘
                          │
                       Internet
Deliverable Checklist
✅ Private Route Table created
✅ Private Subnet 1 associated
✅ Private Subnet 2 associated
✅ Default route points to NAT Gateway
✅ NAT Gateway is located in a public subnet
✅ Private subnet has no direct Internet Gateway route
✅ terraform fmt completed
✅ terraform validate successful
✅ terraform plan shows private routing resources
Commands to capture for your task
terraform fmt
terraform validate
terraform plan