TASK 5 — RDS Security Group
Objective

Create a dedicated Security Group for the RDS database.

The database should not be directly accessible from the internet.

The required communication path is:

Internet
   X
   │
   │ NO DIRECT ACCESS
   X
  RDS

Instead:

Internet
   │
   ▼
  ALB
   │
   ▼
Application EC2
   │
   │ TCP 5432
   ▼
   RDS
1. Security Group Architecture
                    Internet
                       │
                       │ HTTP / HTTPS
                       ▼
                ┌─────────────┐
                │    ALB SG   │
                └─────────────┘
                       │
                       ▼
                ┌─────────────┐
                │ Application │
                │     SG      │
                └─────────────┘
                       │
                       │ TCP 5432
                       ▼
                ┌─────────────┐
                │  Database   │
                │     SG      │
                └─────────────┘
                       │
                       ▼
                      RDS

The important relationship is:

Application SG
      │
      │ TCP 5432
      ▼
Database SG
      │
      ▼
RDS
2. What is a Security Group?

An AWS Security Group acts as a virtual firewall for resources inside a VPC.

It controls network traffic using rules such as:

Protocol
Port
Source
Destination

For RDS, the Security Group controls which resources can connect to the database.

3. Why Create a Dedicated Database Security Group?

The database should have its own Security Group.

Example:

ALB SG
Application SG
Database SG

Each tier has different network requirements.

Internet
   │
   ▼
ALB SG
   │
   ▼
Application SG
   │
   ▼
Database SG

This creates controlled communication between application layers.

4. Database Port

This task uses PostgreSQL.

The default PostgreSQL port is:

5432

Therefore, the database Security Group should allow:

TCP 5432
5. Important Security Rule

Do NOT create this rule:

Type: PostgreSQL
Port: 5432
Source: 0.0.0.0/0

That would allow database connection attempts from anywhere on the internet.

Instead:

Type: PostgreSQL
Port: 5432
Source: Application Security Group

This means only resources associated with the Application Security Group can initiate the allowed database traffic.

6. Terraform Project Structure

A possible project structure is:

terraform/
│
├── main.tf
├── variables.tf
├── outputs.tf
│
└── modules/
    │
    ├── network/
    │
    ├── compute/
    │
    └── database/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf

For this task, the Security Group can be managed by the root module or a dedicated security module.

For learning purposes, we will create it in the root module.

7. Create Database Security Group

In:

main.tf

add:

resource "aws_security_group" "database" {

  name        = "${var.environment}-database-sg"
  description = "Security group for RDS database"
  vpc_id      = module.network.vpc_id

  tags = {
    Name        = "${var.environment}-database-sg"
    Environment = var.environment
  }
}

This creates the dedicated database Security Group.

8. Add PostgreSQL Ingress Rule

Now create the database ingress rule.

resource "aws_vpc_security_group_ingress_rule" "database_postgresql" {

  security_group_id = aws_security_group.database.id

  referenced_security_group_id = aws_security_group.application.id

  from_port = 5432
  to_port   = 5432

  ip_protocol = "tcp"

  description = "Allow PostgreSQL traffic from application tier"
}

The important line is:

referenced_security_group_id = aws_security_group.application.id

It means the source is the Application Security Group.

9. Why Reference a Security Group?

Instead of using:

cidr_ipv4 = "0.0.0.0/0"

we use:

referenced_security_group_id = aws_security_group.application.id

Conceptually:

Application SG
       │
       │ TCP 5432
       ▼
Database SG

This is more appropriate for tier-to-tier communication because access follows the application tier rather than depending on fixed IP addresses.

10. Add Egress Rule

You can explicitly define the database Security Group's outbound traffic:

resource "aws_vpc_security_group_egress_rule" "database_all" {

  security_group_id = aws_security_group.database.id

  cidr_ipv4   = "0.0.0.0/0"
  ip_protocol = "-1"

  description = "Allow outbound traffic"
}

The exact egress policy should be reviewed according to your organization's security requirements.

11. Complete Database Security Group

The complete example becomes:

resource "aws_security_group" "database" {

  name        = "${var.environment}-database-sg"
  description = "Security group for RDS database"

  vpc_id = module.network.vpc_id

  tags = {
    Name        = "${var.environment}-database-sg"
    Environment = var.environment
  }
}

resource "aws_vpc_security_group_ingress_rule" "database_postgresql" {

  security_group_id = aws_security_group.database.id

  referenced_security_group_id = aws_security_group.application.id

  from_port = 5432
  to_port   = 5432

  ip_protocol = "tcp"

  description = "Allow PostgreSQL from application tier"
}

resource "aws_vpc_security_group_egress_rule" "database_all" {

  security_group_id = aws_security_group.database.id

  cidr_ipv4   = "0.0.0.0/0"
  ip_protocol = "-1"

  description = "Allow outbound traffic"
}
12. Application Security Group

The Application Security Group must already exist.

For example:

resource "aws_security_group" "application" {

  name        = "${var.environment}-application-sg"
  description = "Security group for application servers"

  vpc_id = module.network.vpc_id

  tags = {
    Name        = "${var.environment}-application-sg"
    Environment = var.environment
  }
}

The database rule references this Security Group.

13. Attach Database SG to RDS

Your RDS resource should reference the database Security Group.

For example:

resource "aws_db_instance" "database" {

  identifier = var.db_identifier

  engine         = var.engine
  engine_version = var.engine_version

  instance_class = var.instance_class

  allocated_storage = var.allocated_storage

  db_name  = var.db_name
  username = var.username
  password = var.password

  port = 5432

  db_subnet_group_name = aws_db_subnet_group.database.name

  vpc_security_group_ids = [
    aws_security_group.database.id
  ]

  publicly_accessible = false

  storage_encrypted = true

  skip_final_snapshot = true
}

The important configuration is:

vpc_security_group_ids = [
  aws_security_group.database.id
]
14. Database Network Flow

The resulting architecture is:

                         Internet
                            │
                            │ HTTP/HTTPS
                            ▼
                     ┌─────────────┐
                     │     ALB     │
                     │    ALB SG   │
                     └─────────────┘
                            │
                            ▼
                     ┌─────────────┐
                     │ Application │
                     │     EC2     │
                     │ Application │
                     │     SG      │
                     └─────────────┘
                            │
                            │ TCP 5432
                            ▼
                     ┌─────────────┐
                     │     RDS     │
                     │ Database SG │
                     └─────────────┘
15. Security Rule Table
Source	Destination	Protocol	Port	Purpose
Internet	ALB SG	TCP	80	HTTP
Internet	ALB SG	TCP	443	HTTPS
ALB SG	Application SG	TCP	80	Application traffic
Application SG	Database SG	TCP	5432	PostgreSQL
Internet	Database SG	TCP	5432	Not allowed

The most important rule is:

Application SG
      │
      │ TCP 5432
      ▼
Database SG
16. Variables

If environment is not already defined, add it to variables.tf:

variable "environment" {

  description = "Deployment environment"

  type = string

  default = "training"
}
17. Terraform Commands

Run these commands from the Terraform root directory.

Format
terraform fmt

This formats the Terraform files.

Validate
terraform validate

Expected:

Success! The configuration is valid.
Plan
terraform plan

Review the resources Terraform intends to create.

Look for:

aws_security_group.database
aws_vpc_security_group_ingress_rule.database_postgresql
aws_vpc_security_group_egress_rule.database_all
Apply
terraform apply

Review the plan and enter:

yes
18. Verify Terraform State

Run:

terraform state list

You should see resources similar to:

aws_security_group.database
aws_vpc_security_group_ingress_rule.database_postgresql
aws_vpc_security_group_egress_rule.database_all
19. Check the Security Group

You can inspect the configuration using the AWS CLI:

aws ec2 describe-security-groups

Or inspect it from:

AWS Console
   ↓
VPC
   ↓
Security Groups
   ↓
Database Security Group

Verify:

Inbound Rule
Protocol: TCP
Port: 5432
Source: Application Security Group
20. Common Mistakes
Mistake 1 – Allowing Everyone

Incorrect:

cidr_ipv4 = "0.0.0.0/0"

for PostgreSQL port 5432.

This allows connection attempts from any IPv4 address.

Mistake 2 – Wrong Port

PostgreSQL:

5432

MySQL:

3306

Make sure the Security Group matches the database engine.

Mistake 3 – Wrong Security Group

The source should be:

Application Security Group

not:

ALB Security Group

for direct database access.

The expected flow is:

ALB
 ↓
Application EC2
 ↓
RDS
Mistake 4 – Public RDS

Avoid:

publicly_accessible = true

for this architecture.

Use:

publicly_accessible = false

and place RDS in the appropriate private subnets.

21. Security Group vs Subnet

These provide different types of protection.

Private Subnet

Controls network placement and routing.

Private Subnet
     │
     ▼
RDS
Security Group

Controls which network traffic can reach the resource.

Application SG
      │
      │ TCP 5432
      ▼
Database SG

Use both:

Private Subnets
       +
Security Groups
       +
RDS Private Access
22. Final Architecture
                         VPC
                          │
          ┌───────────────┴────────────────┐
          │                                │
     Public Subnets                  Private Subnets
          │                                │
          ▼                                ▼
         ALB                              EC2
      ALB Security                    Application SG
          │                                │
          │                                │ TCP 5432
          │                                ▼
          │                         Database SG
          │                                │
          └────────────────────────────────┤
                                           ▼
                                          RDS

More specifically:

Internet
   │
   X───────────────────────────────X
   │                               │
   ▼                               │
  ALB                              │
   │                               │
   ▼                               │
Application EC2                    │
   │                               │
   │ TCP 5432                      │
   ▼                               │
Database SG                        │
   │                               │
   ▼                               │
  RDS                              │
                                   │
                 NO DIRECT INTERNET ACCESS
23. Key Learning

The database Security Group should follow the principle:

Who needs database access?
            │
            ▼
     Application SG
            │
            │ TCP 5432
            ▼
       Database SG
            │
            ▼
           RDS

Do not think of the rule as:

"Allow port 5432 from the internet."

Instead think:

"Allow PostgreSQL connections from my application tier."

This creates a clear security boundary between the application and database tiers.

Deliverable

The completed Terraform configuration should provide:

Application SG
       │
       │ TCP 5432
       ▼
Database SG
       │
       ▼
RDS

And the database should satisfy:

☑ Dedicated Database Security Group
☑ PostgreSQL port 5432
☑ Source is Application Security Group
☑ No direct internet database access
☑ RDS uses Database Security Group
☑ RDS is in private subnets
☑ Terraform manages the configuration
☑ terraform fmt succeeds
☑ terraform validate succeeds
☑ terraform plan succeeds