TASK 4 — User Data & Automatic Server Configuration

The objective is to make your EC2 instance automatically configure itself when it launches.

Instead of:

Launch EC2
   ↓
SSH into EC2
   ↓
Install packages
   ↓
Install web server
   ↓
Create website
   ↓
Start service

we want:

Launch EC2
   ↓
User Data
   ↓
Install packages
   ↓
Install web server
   ↓
Create web page
   ↓
Start web server
   ↓
Application Ready
1. What Is EC2 User Data?

User Data is a script that can be provided when an EC2 instance is launched.

For a Linux EC2 instance, a common approach is to provide a shell script:

#!/bin/bash

AWS runs the user-data script during the instance's initial startup.

Example:

EC2 Launch
    │
    ▼
User Data Script
    │
    ├── Update packages
    ├── Install NGINX
    ├── Create HTML page
    └── Start NGINX

This allows you to automate initial server configuration.

2. Why Use User Data?

Without User Data:

EC2
 │
 ▼
Manual SSH
 │
 ├── yum update
 ├── install nginx
 ├── create website
 └── start nginx

With User Data:

Terraform
   │
   ▼
EC2
   │
   ▼
User Data
   │
   ├── Install NGINX
   ├── Configure website
   └── Start NGINX

This is useful because the server becomes configured automatically.

3. User Data in Your Compute Module

You already created the variable:

variable "user_data" {
  description = "EC2 startup script"
  type        = string
  default     = null
}

Your EC2 resource should use it:

resource "aws_instance" "this" {
  ami                    = var.ami
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids
  user_data              = var.user_data

  root_block_device {
    volume_type = "gp3"
    volume_size = var.root_volume_size
  }

  tags = var.tags
}

The important line is:

user_data = var.user_data

This allows the root module to provide the startup script.

4. Create the User Data Script

A clean approach is to create:

terraform/
├── main.tf
├── data.tf
├── outputs.tf
├── user-data.sh
└── modules/
    └── compute/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf

Create:

user-data.sh

with:

#!/bin/bash

# Update packages
dnf update -y

# Install NGINX
dnf install -y nginx

# Create application page
cat > /usr/share/nginx/html/index.html <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>DevOps Training Application</title>
</head>
<body>
    <h1>DevOps Training Application</h1>
    <p>Server: EC2</p>
    <p>Environment: Training</p>
    <p>Status: Application is running</p>
</body>
</html>
EOF

# Enable NGINX at boot
systemctl enable nginx

# Start NGINX
systemctl start nginx
5. Understand the Script
Step 1 — Shell Interpreter
#!/bin/bash

This tells Linux to execute the script using Bash.

Step 2 — Update Packages
dnf update -y

This updates installed packages.

Amazon Linux 2023 uses dnf.

Step 3 — Install NGINX
dnf install -y nginx

This installs the NGINX web server.

After installation:

EC2
 │
 └── NGINX
Step 4 — Create the Website
cat > /usr/share/nginx/html/index.html <<'EOF'

This creates the HTML file used by NGINX.

The page contains:

<h1>DevOps Training Application</h1>
<p>Server: EC2</p>
<p>Environment: Training</p>
Step 5 — Enable NGINX
systemctl enable nginx

This configures NGINX to start automatically when the server boots.

Step 6 — Start NGINX
systemctl start nginx

The web server starts immediately.

6. Pass User Data to the Module

In the root main.tf:

module "compute" {
  source = "./modules/compute"

  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  subnet_id = "subnet-xxxxxxxxxxxxxxxxx"

  security_group_ids = [
    "sg-xxxxxxxxxxxxxxxxx"
  ]

  root_volume_size = 8

  tags = {
    Name        = "day28-web-server"
    Environment = "Training"
  }

  user_data = file("${path.module}/user-data.sh")
}

The important part is:

user_data = file("${path.module}/user-data.sh")

Terraform reads the contents of:

user-data.sh

and passes them to EC2.

7. Complete Architecture

Your Terraform architecture is now:

                    Terraform
                        │
                        ▼
                 Compute Module
                        │
             ┌──────────┴──────────┐
             │                     │
             ▼                     ▼
        EC2 Configuration      User Data
                                   │
                                   ▼
                              EC2 Launch
                                   │
                  ┌────────────────┼────────────────┐
                  ▼                ▼                ▼
             Update OS         Install NGINX    Create HTML
                                                     │
                                                     ▼
                                               Start NGINX
                                                     │
                                                     ▼
                                              Web Application
8. Security Group Requirement

There is one important networking requirement.

NGINX listens on:

HTTP → TCP 80

Therefore, the EC2 Security Group must allow HTTP traffic from the appropriate source.

For a simple training environment, an HTTP rule might look like:

ingress {
  description = "HTTP"
  from_port   = 80
  to_port     = 80
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}

For production, you should generally avoid unnecessarily exposing services publicly and should place application instances behind an ALB with appropriately restricted security-group rules.

9. Terraform Configuration Example

For this exercise, your root configuration could look like:

provider "aws" {
  region = "ap-south-1"
}

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

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }
}

module "compute" {
  source = "./modules/compute"

  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  subnet_id = "subnet-xxxxxxxxxxxxxxxxx"

  security_group_ids = [
    "sg-xxxxxxxxxxxxxxxxx"
  ]

  root_volume_size = 8

  tags = {
    Name        = "day28-web-server"
    Environment = "Training"
  }

  user_data = file("${path.module}/user-data.sh")
}

Replace the example subnet and security-group IDs with your actual training resources.

10. Run Terraform

Go to your Terraform directory:

cd /workspaces/terraform-lab/day28/terraform

Format:

terraform fmt

Initialize:

terraform init

Validate:

terraform validate

Expected:

Success! The configuration is valid.

Then:

terraform plan

Review the plan.

If everything is correct:

terraform apply

Enter:

yes
11. Get the EC2 Public IP

Your outputs.tf should contain:

output "public_ip" {
  description = "EC2 public IP"
  value       = module.compute.public_ip
}

Run:

terraform output public_ip

Example:

"13.201.xx.xx"

Open:

http://13.201.xx.xx

You should see:

DevOps Training Application

Server: EC2
Environment: Training
Status: Application is running
12. Verify on the EC2 Instance

If you have SSH access, you can verify that User Data worked.

Check NGINX:

sudo systemctl status nginx

Expected:

Active: active (running)

Check the web page:

curl http://localhost

You should see the HTML content.

Check the User Data log:

sudo cat /var/log/cloud-init-output.log

This is especially useful when troubleshooting User Data.

13. Important User Data Troubleshooting

If the web page does not load, check in this order:

1. Is the EC2 running?
EC2 → Running
2. Is NGINX running?
sudo systemctl status nginx
3. Is port 80 allowed?

Check the Security Group.

HTTP
TCP
80
4. Check User Data logs
sudo cat /var/log/cloud-init-output.log
5. Test locally
curl http://localhost

If this works:

curl localhost → Works

but this doesn't:

Browser → EC2 Public IP → Doesn't work

the problem is likely related to networking/Security Group rather than User Data.

14. Important Concept — User Data Usually Runs at First Boot

A common beginner mistake is expecting:

terraform apply

to repeatedly execute the User Data script whenever Terraform runs.

That's not the normal behavior.

The User Data is primarily used during initial instance initialization.

Therefore:

Create EC2
     ↓
First boot
     ↓
User Data executes

If you modify the User Data after the instance already exists, don't assume the script will simply rerun on that existing instance.

For training, if necessary, destroy and recreate the instance:

terraform destroy

then:

terraform apply
15. Final Task 4 Architecture
                         Terraform
                            │
                            ▼
                     Compute Module
                            │
                            ▼
                       EC2 Launch
                            │
                            ▼
                        User Data
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
        Update OS      Install NGINX    Create Page
                                            │
                                            ▼
                                      Start Service
                                            │
                                            ▼
                                      HTTP : 80
                                            │
                                            ▼
                                         User
Task 4 Deliverable Checklist
[✓] Understand EC2 User Data
[✓] Create user-data.sh
[✓] Update packages
[✓] Install NGINX
[✓] Create application page
[✓] Enable NGINX
[✓] Start NGINX
[✓] Pass User Data through Terraform
[✓] Configure HTTP Security Group rule
[✓] Run terraform fmt
[✓] Run terraform validate
[✓] Run terraform plan
[✓] Apply Terraform
[✓] Verify the web page
[✓] Check cloud-init logs if required
Key takeaway
Terraform
    ↓
EC2
    ↓
User Data
    ↓
Automatic Configuration
    ↓
NGINX
    ↓
Application Page
    ↓
Ready to Use

The important DevOps principle is: the server should configure itself automatically instead of requiring manual SSH configuration.