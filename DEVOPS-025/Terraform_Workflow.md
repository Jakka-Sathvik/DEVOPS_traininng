TASK 12 — Terraform Workflow

This task is a hands-on practice task. You need to perform the complete Terraform lifecycle yourself and capture the terminal output as your deliverable.

The workflow is:

terraform fmt
      ↓
terraform init
      ↓
terraform validate
      ↓
terraform plan
      ↓
terraform apply
      ↓
Verify Infrastructure
      ↓
terraform destroy

We'll do it step by step.

1. Go to Your Terraform Environment

Because TASK 11 created separate environments, I recommend using Dev for this workflow.

Open PowerShell or your terminal and go to:

cd terraform\environments\dev

Check that you're in the correct directory:

Get-ChildItem

You should see something like:

main.tf
variables.tf
outputs.tf
provider.tf
terraform.tfvars
2. Step 1 — terraform fmt

Run:

terraform fmt -recursive
What does it do?

It automatically formats your Terraform code.

For example, poorly formatted code:

resource "aws_vpc" "training_vpc" {
cidr_block="10.10.0.0/16"
}

Terraform formats it:

resource "aws_vpc" "training_vpc" {
  cidr_block = "10.10.0.0/16"
}
Expected result

You may see filenames if Terraform changed them.

If everything is already formatted, you may see no output.

Screenshot 1: Capture this command/result.

3. Step 2 — terraform init

Run:

terraform init

Terraform initializes the working directory.

It will:

Initialize Terraform
Download the AWS provider
Initialize your modules
Create/update .terraform
Create/update .terraform.lock.hcl

You should see something similar to:

Initializing the backend...
Initializing modules...
- network in ../../modules/network

Initializing provider plugins...

Terraform has been successfully initialized!
Important

You should see your module:

- network in ../../modules/network

Screenshot 2: Capture the successful terraform init.

4. Step 3 — terraform validate

Run:

terraform validate

Expected:

Success! The configuration is valid.

This checks whether your Terraform configuration is structurally and syntactically valid.

It does not create AWS resources.

Screenshot 3: Capture:

terraform validate
Success! The configuration is valid.
5. Step 4 — terraform plan

Run:

terraform plan

Terraform now compares your configuration with the current state and determines what it would create/change.

You should see resources related to your network module.

For example:

Plan: 5 to add, 0 to change, 0 to destroy.

Your number could be different depending on your existing state/configuration.

You may see addresses like:

module.network.aws_vpc.training_vpc
module.network.aws_subnet.training_subnet
module.network.aws_internet_gateway.training_igw
module.network.aws_route_table.training_route_table
module.network.aws_route_table_association.training_association

This demonstrates that your reusable module is being used.

Important

terraform plan does not create the resources.

It only shows what Terraform intends to do.

Screenshot 4: Capture the plan summary.

6. Step 5 — terraform apply

Now create the infrastructure.

Run:

terraform apply

Terraform will show the plan again and ask:

Do you want to perform these actions?
  Enter a value:

Type:

yes

Then press Enter.

Terraform will create the infrastructure.

You should eventually see:

Apply complete! Resources: 5 added, 0 changed, 0 destroyed.

Again, your number may differ.

You should also see your outputs, for example:

Outputs:

environment = "dev"
subnet_id = "subnet-xxxxxxxx"
vpc_id = "vpc-xxxxxxxx"

Screenshot 5: Capture the successful terraform apply.

7. Step 6 — Verify the Infrastructure

Now verify your Terraform outputs.

Run:

terraform output

Example:

environment = "dev"
subnet_id = "subnet-0123456789"
vpc_id = "vpc-0123456789"

You can also check the VPC specifically:

terraform output vpc_id

Then open the AWS Console and verify the resources.

You should find your:

VPC
 │
 ├── Subnet
 │
 ├── Internet Gateway
 │
 └── Route Table

Check that the resources actually exist.

Screenshot 6: Capture terraform output and/or the AWS Console showing the created infrastructure.

8. Step 7 — terraform destroy

After verification, clean up the resources.

Run:

terraform destroy

Terraform will show what it is going to remove.

It will ask:

Do you really want to destroy all resources?

Enter:

yes

Terraform will delete the infrastructure.

Expected result:

Destroy complete! Resources: 5 destroyed.

Your number may be different.

Screenshot 7: Capture the successful destroy.

9. Complete Lifecycle

Your terminal history should demonstrate:

┌─────────────────────────────┐
│ terraform fmt               │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ terraform init              │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ terraform validate          │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ terraform plan              │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ terraform apply             │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ Verify AWS Infrastructure   │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ terraform destroy           │
└─────────────────────────────┘
10. Quick Command List

For your practical execution, simply follow this:

cd terraform/environments/dev
terraform fmt -recursive
terraform init
terraform validate
terraform plan
terraform apply

Type:

yes

Then verify:

terraform output

Finally:

terraform destroy

Type:

yes
11. What to Put in Your Screenshot/Log

Your submission should demonstrate these results:

Step	Command	Expected Result
1	terraform fmt	Code formatted
2	terraform init	Successfully initialized
3	terraform validate	Configuration is valid
4	terraform plan	Resources planned
5	terraform apply	Resources created
6	terraform output	Infrastructure IDs displayed
7	AWS Console	Resources verified
8	terraform destroy	Resources removed