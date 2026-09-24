# Terraform and Infrastructure as Code (IaC) Fundamentals

## Introduction

Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure through code instead of manually configuring resources through a graphical user interface such as the AWS Management Console.

### Manual Infrastructure Approach

```
Developer
   ↓
AWS Console
   ↓
Create EC2
   ↓
Configure Security Group
   ↓
Configure Server
```

This approach is time-consuming, error-prone, and difficult to reproduce consistently.

### Infrastructure as Code Approach

```
Terraform Code
      ↓
terraform plan
      ↓
terraform apply
      ↓
AWS Infrastructure
```

Using Terraform, infrastructure is defined in code and can be created, modified, and deleted automatically.

---

# What is Infrastructure as Code?

Infrastructure as Code (IaC) is a methodology that uses configuration files and automation tools to provision and manage infrastructure resources such as virtual machines, networks, storage, databases, and security settings.

Instead of manually creating resources, engineers write code that describes the desired infrastructure state.

---

# Why IaC is Required

* Eliminates manual configuration errors
* Improves deployment speed
* Ensures consistent environments
* Enables automation
* Supports version control
* Simplifies disaster recovery
* Improves collaboration among teams
* Makes infrastructure reproducible

---

# Terraform vs Manual AWS Configuration

| Manual AWS Configuration       | Terraform                 |
| ------------------------------ | ------------------------- |
| Performed through AWS Console  | Performed through code    |
| Error-prone                    | Consistent and repeatable |
| Difficult to track changes     | Changes tracked in Git    |
| Time-consuming                 | Automated                 |
| Hard to replicate environments | Easily reproducible       |
| Limited collaboration          | Team-friendly workflow    |

---

# Declarative Infrastructure

Terraform uses a declarative approach.

Instead of specifying every step, engineers define the desired end state.

Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```

Terraform determines how to create the resource to match the declared configuration.

---

# Infrastructure Reproducibility

The same Terraform code can be used to create identical environments multiple times.

Benefits:

* Development environment matches production
* Easier testing
* Predictable deployments

---

# Infrastructure Version Control

Terraform files can be stored in Git repositories.

Benefits:

* Track infrastructure changes
* Review changes before deployment
* Roll back to previous versions
* Collaborate using pull requests

---

# Infrastructure Automation

Terraform automates infrastructure provisioning.

Common automated tasks:

* Creating EC2 instances
* Configuring VPCs
* Creating Security Groups
* Deploying Load Balancers
* Managing Databases

---

# Infrastructure Consistency

Terraform ensures every environment is built from the same code.

Benefits:

* Reduced configuration drift
* Standardized deployments
* Improved reliability

---

# 10 Important Terraform and IaC Concepts

## 1. Infrastructure as Code (IaC)

Managing infrastructure through code instead of manual processes.

## 2. Terraform Configuration Files

Terraform uses `.tf` files to define infrastructure resources.

## 3. Providers

Providers enable Terraform to interact with platforms such as AWS, Azure, and Google Cloud.

Example:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

## 4. Resources

Resources represent infrastructure components.

Examples:

* EC2 Instances
* VPCs
* Security Groups
* S3 Buckets

## 5. State File

Terraform stores infrastructure information in a state file (`terraform.tfstate`) to track managed resources.

## 6. Terraform Init

Initializes a Terraform project and downloads required providers.

```bash
terraform init
```

## 7. Terraform Plan

Shows what changes Terraform will make before applying them.

```bash
terraform plan
```

## 8. Terraform Apply

Creates or updates infrastructure based on configuration files.

```bash
terraform apply
```

## 9. Terraform Destroy

Removes infrastructure managed by Terraform.

```bash
terraform destroy
```

## 10. Idempotency

Running Terraform multiple times produces the same desired infrastructure state without unnecessary changes.

---

# Key Benefits of Terraform

* Infrastructure automation
* Version-controlled infrastructure
* Consistent deployments
* Reduced human errors
* Multi-cloud support
* Faster provisioning
* Easy scalability
* Reusable configurations

---

# Conclusion

Terraform is one of the most widely used Infrastructure as Code tools. It allows organizations to define infrastructure using code, automate deployments, maintain consistency across environments, and manage infrastructure changes efficiently through version control systems.
