Terraform Resource References
Objective

Learn how Terraform resources can reference other resources and create implicit dependencies.

Key Concepts
1. Resource Type

The type of infrastructure resource Terraform manages.

Example:

aws_s3_bucket

Here:

aws = Provider
s3_bucket = Resource Type
2. Resource Name

A local identifier used inside Terraform code.

Example:

resource "aws_s3_bucket" "application_bucket" {
Resource Type = aws_s3_bucket
Resource Name = application_bucket
3. Resource Attribute

Properties exposed by a resource.

Example:

aws_s3_bucket.application_bucket.id

Attribute:

id

Other common attributes:

bucket
arn
region
4. Resource Reference

A resource can use attributes from another resource.

Syntax:

resource_type.resource_name.attribute

Example:

aws_s3_bucket.application_bucket.id
5. Implicit Dependency

Terraform automatically determines resource creation order when one resource references another.

Example:

bucket = aws_s3_bucket.application_bucket.id

Terraform knows:

Create S3 Bucket
      ↓
Configure Versioning

No depends_on is required.

Example: Two Resources with Resource Reference
Resource 1: S3 Bucket
resource "aws_s3_bucket" "application_bucket" {
  bucket = "terraform-demo-bucket-12345"
}
Resource 2: Bucket Versioning
resource "aws_s3_bucket_versioning" "bucket_versioning" {
  bucket = aws_s3_bucket.application_bucket.id

  versioning_configuration {
    status = "Enabled"
  }
}
Output
output "bucket_name" {
  value = aws_s3_bucket.application_bucket.bucket
}
Reference Flow
aws_s3_bucket
      │
      ▼
application_bucket
      │
      ▼
id
      │
      ▼
aws_s3_bucket_versioning
      │
      ▼
bucket_versioning
Terraform Commands

Initialize Terraform:

terraform init

Validate Configuration:

terraform validate

Preview Changes:

terraform plan

Create Resources:

terraform apply

Destroy Resources:

terraform destroy
Deliverable Summary

✅ Created 2 Terraform resources

aws_s3_bucket.application_bucket
aws_s3_bucket_versioning.bucket_versioning

✅ Demonstrated Resource Reference

bucket = aws_s3_bucket.application_bucket.id

✅ Demonstrated Resource Type

aws_s3_bucket

✅ Demonstrated Resource Name

application_bucket

✅ Demonstrated Resource Attribute

id

✅ Demonstrated Implicit Dependency

Terraform automatically creates the buck