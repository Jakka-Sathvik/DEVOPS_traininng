TASK 6 — Internet Gateway & Public Routing
Objective

In this task, we will configure an Internet Gateway (IGW) and a Public Route Table using Terraform.

The goal is to allow resources in the public subnets to communicate with the internet.

Important: A subnet is effectively public when its route table has a route to an Internet Gateway, and the resource also has appropriate public addressing. Creating an Internet Gateway alone does not make a subnet public.

Step 1 — Understand the Traffic Flow

The traffic flow will be:

                    Internet
                       ▲
                       │
                       │
              Internet Gateway
                       ▲
                       │
                0.0.0.0/0
                       ▲
                       │
                Public Route Table
                       ▲
                       │
                Public Subnet

For example:

Public Subnet
10.0.1.0/24
     │
     ▼
Public Route Table
     │
     ├── 10.0.0.0/16 → local
     │
     └── 0.0.0.0/0 → Internet Gateway
Step 2 — Create the Internet Gateway

Inside:

terraform/modules/vpc/main.tf

add:

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name        = "${var.environment}-igw"
    Environment = var.environment
  }
}
Explanation
vpc_id = aws_vpc.main.id

This attaches the Internet Gateway to the VPC created by the module.

Terraform automatically understands that the Internet Gateway depends on the VPC.

Step 3 — Create the Public Route Table

Add:

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name        = "${var.environment}-public-route-table"
    Environment = var.environment
  }
}
What does 0.0.0.0/0 mean?
0.0.0.0/0

means all IPv4 destinations.

Therefore:

0.0.0.0/0 → Internet Gateway

means traffic destined for the internet should be sent to the Internet Gateway.

Step 4 — Associate Public Subnets with the Route Table

Creating a route table is not enough.

We must associate the public subnets with it.

Add:

resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)

  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

Because we have two public subnets:

Public Subnet 1
10.0.1.0/24
       │
       ▼
Public Route Table

Public Subnet 2
10.0.2.0/24
       │
       ▼
Public Route Table
Step 5 — Make Sure Public Subnets Can Receive Public IPs

Your public subnet resource from the previous task should contain:

resource "aws_subnet" "public" {
  count = length(var.public_subnet_cidrs)

  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.environment}-public-${count.index + 1}"
  }
}

The important setting is:

map_public_ip_on_launch = true

This allows eligible instances launched into the subnet to receive a public IPv4 address automatically.

Step 6 — Check the Complete Public Network

Your architecture should now look like this:

                         INTERNET
                            ▲
                            │
                            │
                    Internet Gateway
                            ▲
                            │
                     0.0.0.0/0
                            │
                 ┌──────────┴──────────┐
                 │                     │
          Public Route Table      Public Route Table
                 │                     │
                 ▼                     ▼
       Public Subnet AZ-1     Public Subnet AZ-2
          10.0.1.0/24            10.0.2.0/24
                 │                     │
              EC2/Web              EC2/Web

In our Terraform implementation, both public subnets can share the same public route table.

Step 7 — Check Your Files

Your module should now contain something similar to:

terraform/
│
├── main.tf
├── outputs.tf
│
└── modules/
    └── vpc/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf

The important resources in modules/vpc/main.tf are:

aws_vpc.main
aws_internet_gateway.igw

aws_subnet.public

aws_route_table.public

aws_route_table_association.public
Step 8 — Format Terraform

From the root Terraform directory:

terraform fmt

Expected behavior:

Terraform files are formatted.

If Terraform changes files, it may display their names.

Step 9 — Validate

Run:

terraform validate

Expected result:

Success! The configuration is valid.

If you get an error, read the file name and line number shown by Terraform before making changes.

Step 10 — Initialize Terraform

If you have not initialized the project after adding the module:

terraform init

You should see Terraform initialize the AWS provider and module.

Step 11 — Generate the Terraform Plan

Run:

terraform plan

Look for resources such as:

+ aws_internet_gateway.igw
+ aws_route_table.public
+ aws_route_table_association.public[0]
+ aws_route_table_association.public[1]

The plan should show that the public routing infrastructure will be created.

Step 12 — Understand the Route

The important route is:

Destination       Target
--------------------------------
10.0.0.0/16       local
0.0.0.0/0         Internet Gateway

The local route allows communication within the VPC.

The default route:

0.0.0.0/0 → Internet Gateway

provides the path toward the internet.

Final Architecture
                         INTERNET
                            ▲
                            │
                            ▼
                 ┌───────────────────┐
                 │ Internet Gateway  │
                 └─────────┬─────────┘
                           │
                     0.0.0.0/0
                           │
                 ┌─────────▼─────────┐
                 │ Public Route      │
                 │ Table             │
                 └─────────┬─────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
       Public Subnet 1          Public Subnet 2
        10.0.1.0/24              10.0.2.0/24
              │                         │
              ▼                         ▼
          EC2/Web                   EC2/Web
Task 6 Deliverable

You have completed the task when:

✅ Internet Gateway is attached to the VPC
✅ Public Route Table is created
✅ 0.0.0.0/0 points to the Internet Gateway
✅ Both public subnets are associated with the route table
✅ Public subnets enable public IP assignment where appropriate
✅ terraform fmt succeeds
✅ terraform validate succeeds
✅ terraform plan shows the expected public networking resources