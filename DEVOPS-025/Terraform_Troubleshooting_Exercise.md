TASK 13 — Terraform Troubleshooting Exercise

This task is about learning how to read Terraform errors, find the problem, fix it, and verify the fix.

You don't just need to make Terraform work—you need to document why it failed and how you fixed it.

The troubleshooting workflow is:

Terraform Error
      ↓
Read Error
      ↓
Find File
      ↓
Find Resource / Variable
      ↓
Understand Root Cause
      ↓
Fix Configuration
      ↓
terraform fmt
      ↓
terraform validate
      ↓
terraform plan
      ↓
terraform apply (if required)
1. Create the Notes Folder

From your Terraform project root:

mkdir notes

Your structure should eventually look like:

terraform/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
│
├── modules/
│   └── network/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
├── environments/
│   ├── dev/
│   └── test/
│
└── notes/
    └── day25-troubleshooting.md
2. Understand the Types of Problems

Your trainer may intentionally introduce one of these.

Problem 1 — Incorrect Variable

Example:

cidr_block = var.vpc_cidr_block

But the variable is actually:

variable "vpc_cidr" {
  type = string
}

Terraform will complain because:

var.vpc_cidr_block

doesn't exist.

Problem 2 — Invalid Resource Reference

Example:

subnet_id = aws_subnet.application_subnet.id

But the actual resource is:

resource "aws_subnet" "training_subnet" {

The correct reference is:

aws_subnet.training_subnet.id
Problem 3 — Missing Required Argument

Example:

resource "aws_subnet" "training_subnet" {
  vpc_id = aws_vpc.training_vpc.id
}

The subnet also needs a CIDR block.

Correct:

resource "aws_subnet" "training_subnet" {
  vpc_id     = aws_vpc.training_vpc.id
  cidr_block = var.subnet_cidr
}
3. Example Troubleshooting Exercise

Let's intentionally create a simple error.

Suppose your module contains:

resource "aws_vpc" "training_vpc" {
  cidr_block = var.vpc_cidr
}

But someone accidentally changes it to:

resource "aws_vpc" "training_vpc" {
  cidr_block = var.vpc_network
}

However, your variables.tf only contains:

variable "vpc_cidr" {
  type = string
}
4. Run Terraform

Go to your environment:

cd environments\dev

Run:

terraform validate

Terraform may return an error similar to:

Error: Reference to undeclared input variable

A variable named "vpc_network" has not been declared.
5. Step 1 — Read the Error

Don't immediately change code.

First look at:

Error
File
Line number
Resource
Variable/reference

For example:

Error: Reference to undeclared input variable

on ../../modules/network/main.tf line 2:

This tells you:

File:
modules/network/main.tf

Problem:
vpc_network

Type:
Undeclared variable
6. Step 2 — Identify the Affected File

Terraform tells you:

modules/network/main.tf

Open that file.

Find:

cidr_block = var.vpc_network
7. Step 3 — Identify the Affected Resource

Look above the line:

resource "aws_vpc" "training_vpc" {

Therefore the affected resource is:

aws_vpc.training_vpc
8. Step 4 — Understand the Root Cause

Now inspect:

modules/network/variables.tf

You find:

variable "vpc_cidr" {
  type = string
}

There is no:

vpc_network

Therefore:

Code expects:
var.vpc_network

Variable actually exists:
var.vpc_cidr
Root Cause

The Terraform configuration references a variable that has not been declared.

9. Step 5 — Fix the Configuration

Change:

cidr_block = var.vpc_network

to:

cidr_block = var.vpc_cidr

Save the file.

10. Step 6 — Run terraform fmt

Run:

terraform fmt -recursive

This formats the Terraform files.

11. Step 7 — Run terraform validate

Run:

terraform validate

Expected:

Success! The configuration is valid.

This confirms the configuration is structurally valid.

12. Step 8 — Run terraform plan

Now run:

terraform plan

If the configuration is correct, Terraform should produce a plan without the previous variable error.

You may see:

Plan: 5 to add, 0 to change, 0 to destroy.

The exact number depends on your project.

13. Step 9 — Apply If Required

If your trainer asks you to actually deploy the corrected configuration:

terraform apply

Then:

yes

If the exercise only requires troubleshooting and validation, you don't need to apply.

14. Document the Problem

Now create:

notes/day25-troubleshooting.md

You can use this structure:

# Terraform Troubleshooting Exercise

## Problem

The Terraform configuration failed because the network module referenced
an input variable named `vpc_network`.

## Root Cause

The variable `vpc_network` was not declared in the module's
`variables.tf` file.

The correct variable was `vpc_cidr`.

## Investigation

1. Ran `terraform validate`.
2. Terraform reported an undeclared input variable.
3. Checked the error message and identified:
   `modules/network/main.tf`.
4. Identified the affected resource:
   `aws_vpc.training_vpc`.
5. Checked `modules/network/variables.tf`.
6. Confirmed that `vpc_cidr` was declared but `vpc_network` was not.

## Solution

Changed:

```hcl
cidr_block = var.vpc_network

to:

cidr_block = var.vpc_cidr
Validation

Ran:

terraform fmt -recursive
terraform validate
terraform plan

Terraform validation completed successfully.

Result

The invalid variable reference was fixed successfully.
Terraform was able to validate the configuration and generate
a deployment plan.


---

# 15. Better Troubleshooting Example — Module Output

You may also get an **incorrect module output** problem.

Suppose the module has:

```hcl
output "vpc_id" {
  value = aws_vpc.training_vpc.id
}

But the root module incorrectly uses:

output "network_id" {
  value = module.network.network_id
}

Terraform will complain because the child module doesn't expose:

network_id

It exposes:

vpc_id
Fix

Change:

value = module.network.network_id

to:

value = module.network.vpc_id

Then:

terraform fmt -recursive
terraform validate
terraform plan
16. Another Example — Invalid Resource Reference

Suppose you have:

resource "aws_subnet" "training_subnet" {
  vpc_id     = aws_vpc.training_vpc.id
  cidr_block = var.subnet_cidr
}

But another resource says:

subnet_id = aws_subnet.app_subnet.id

There is no:

aws_subnet.app_subnet

The actual resource is:

aws_subnet.training_subnet

Therefore change:

subnet_id = aws_subnet.app_subnet.id

to:

subnet_id = aws_subnet.training_subnet.id

Then verify:

terraform fmt -recursive
terraform validate
terraform plan
17. Troubleshooting Checklist

Whenever Terraform gives you an error, follow this checklist:

1. Read the error
What exactly is Terraform complaining about?
2. Find the file
Which .tf file?
3. Find the line
Which line number?
4. Find the resource
Which resource/module/variable is affected?
5. Check related configuration

For example:

Variable error
    ↓
Check variables.tf

Module output error
    ↓
Check module outputs.tf

Resource reference error
    ↓
Check resource name

Missing argument
    ↓
Check resource documentation/configuration
6. Fix it

Make the smallest correct change.

7. Format
terraform fmt -recursive
8. Validate
terraform validate
9. Plan
terraform plan
10. Apply if necessary
terraform apply
18. Final Deliverable

Your final file must be:

notes/
└── day25-troubleshooting.md

It must contain:

Problem
   ↓
Root Cause
   ↓
Investigation
   ↓
Solution
   ↓
Validation
   ↓
Result

A good submission demonstrates that you didn't just fix the error—you understood why Terraform produced it.

Commands to document
terraform fmt -recursive
terraform validate
terraform plan
terraform apply    ← if required