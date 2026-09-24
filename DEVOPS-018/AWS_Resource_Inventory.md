# AWS Resource Inventory — Task 7

**Time:** 45 mins
**Goal:** Put together a simple inventory of what's actually running in the training environment — service, resource name, purpose, and current status.

---

## How I checked each one

Instead of guessing, the idea is to pull real status from the CLI for each service:

- **EC2** → `aws ec2 describe-instances --query 'Reservations[].Instances[].State.Name' --output text`
- **S3** → `aws s3 ls` (if the bucket lists, it's active)
- **CloudFront** → `aws cloudfront list-distributions`
- **Route 53** → `aws route53 list-hosted-zones`
- **CloudWatch** → `aws cloudwatch describe-alarms`

---

## Inventory

| Service | Resource | Purpose | Status |
|---|---|---|---|
| EC2 | Training Server | Linux Server | Running / Stopped* |
| S3 | Training Bucket | Object Storage | Active |
| CloudFront | Training Distribution | CDN | Active |
| Route 53 | Training Zone | DNS | Active |
| CloudWatch | Training Alarm | Monitoring | Active |

*EC2 is the one status that actually changes day to day depending on whether the instance was left running or stopped after last use — worth re-running `describe-instances` before trusting this if it's been a while.

---

## Note

Kept this deliberately small — just the 5 resources actively used for training. A production account's inventory would also track IAM roles, VPCs/subnets, security groups, etc., but for a training sandbox this covers everything that's actually doing something.
