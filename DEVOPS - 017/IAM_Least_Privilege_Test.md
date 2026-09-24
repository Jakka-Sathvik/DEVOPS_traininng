## Objective

Create a training IAM identity that can **only** read objects from a specific S3 bucket, and prove — through testing — that everything else (deleting objects, creating users, modifying IAM, terminating EC2) is blocked.

```
S3
 ↓
Read objects
```

Not allowed:
```
Delete objects
Create users
Modify IAM
Terminate EC2
```

---

## IAM Policy — S3 Read-Only (Least Privilege)

Replace `your-training-bucket` with the real bucket name. Save as `s3-readonly-policy.json`.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowListTrainingBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::your-training-bucket"
    },
    {
      "Sid": "AllowReadObjectsInTrainingBucket",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-training-bucket/*"
    }
  ]
}
```

This is the whole mechanism: only `ListBucket` and `GetObject` are granted. Delete, IAM, and EC2 actions are never mentioned — so by IAM's default-deny rule, they're already blocked. Nothing extra needs to be written to "turn them off."

**Optional hardening:** if this identity might ever sit in a group with broader permissions, add an explicit `Deny` as a safety net — explicit Deny always wins over any Allow, from any source:

```json
{
  "Sid": "ExplicitDenyDefenseInDepth",
  "Effect": "Deny",
  "Action": ["s3:Delete*", "iam:*", "ec2:TerminateInstances"],
  "Resource": "*"
}
```
(append as an extra item in the `Statement` array above)

---

## Setup Steps

### 0. Create the training bucket + a sample object (if it doesn't exist yet)
```bash
aws s3 mb s3://your-training-bucket --region us-east-1
echo "hello trainee" > sample.txt
aws s3 cp sample.txt s3://your-training-bucket/
```

### 1. Create the policy
```bash
aws iam create-policy \
  --policy-name TrainingS3ReadOnly \
  --policy-document file://s3-readonly-policy.json
```

### 2. Create a group and attach the policy
(Groups > individual user policies — matches Day 17's IAM Group best practice)
```bash
aws iam create-group --group-name S3-ReadOnly-Trainees

aws iam attach-group-policy \
  --group-name S3-ReadOnly-Trainees \
  --policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/TrainingS3ReadOnly
```
Find `<ACCOUNT_ID>` with `aws sts get-caller-identity`.

### 3. Create the trainee user and add to the group
```bash
aws iam create-user --user-name trainee-user

aws iam add-user-to-group \
  --user-name trainee-user \
  --group-name S3-ReadOnly-Trainees

aws iam create-access-key --user-name trainee-user
```
Save the returned Access Key ID + Secret Access Key securely — never commit them anywhere (see Day 17, Q10).

### 4. Configure a CLI profile for the trainee
```bash
aws configure --profile trainee
# paste the access key + secret from step 3
```

> **Console note:** if the trainee will click around the S3 console instead of using the CLI, add `s3:ListAllMyBuckets` so bucket names are visible in the UI. That permission only reveals bucket *names*, not contents — but for strictest least privilege, prefer testing via CLI (`aws s3 ls s3://your-training-bucket --profile trainee`), which doesn't need it.

---

## Test Plan & Expected Results

### Test 1 — List / read the training bucket
```bash
aws s3 ls s3://your-training-bucket --profile trainee
aws s3 cp s3://your-training-bucket/sample.txt . --profile trainee
```
**Expected:** `SUCCESS` — bucket contents list, file downloads.
**Why:** `s3:ListBucket` and `s3:GetObject` are explicitly allowed.

### Test 2 — Try to delete an object
```bash
aws s3 rm s3://your-training-bucket/sample.txt --profile trainee
```
**Expected:** `ACCESS DENIED`
**Why:** `s3:DeleteObject` was never granted — implicit deny.

### Test 3 — Try unrelated administrative operations
```bash
# 3a: create a user
aws iam create-user --user-name test-escalation --profile trainee

# 3b: modify IAM (attach an admin policy to itself)
aws iam attach-user-policy \
  --user-name trainee-user \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess \
  --profile trainee

# 3c: terminate an EC2 instance
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0 --profile trainee
```
**Expected:** `ACCESS DENIED` on all three.
**Why:** none of `iam:CreateUser`, `iam:AttachUserPolicy`, or `ec2:TerminateInstances` appear in the policy. Note EC2 typically returns the error code `UnauthorizedOperation` rather than `AccessDenied` — same outcome, different label.

---

## Results Log

Fill in after running the tests above against the real account.

| # | Test | Expected | Actual | Pass/Fail |
|---|------|----------|--------|-----------|
| 1 | List / read training bucket | SUCCESS | | |
| 2 | Delete an object | ACCESS DENIED | | |
| 3a | Create IAM user | ACCESS DENIED | | |
| 3b | Modify IAM (attach policy) | ACCESS DENIED | | |
| 3c | Terminate EC2 instance | ACCESS DENIED | | |

---

## Cleanup

Remove the training identity once the exercise is done — don't leave unused users/keys lying around.
```bash
aws iam delete-access-key --user-name trainee-user --access-key-id <ACCESS_KEY_ID>
aws iam remove-user-from-group --user-name trainee-user --group-name S3-ReadOnly-Trainees
aws iam delete-user --user-name trainee-user
aws iam detach-group-policy --group-name S3-ReadOnly-Trainees --policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/TrainingS3ReadOnly
aws iam delete-group --group-name S3-ReadOnly-Trainees
aws iam delete-policy --policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/TrainingS3ReadOnly
```

---

## Key Takeaway

Least privilege isn't enforced by listing everything you want to block — it's enforced by only listing what you want to allow. Two lines of policy (`ListBucket`, `GetObject`) are all it took to make deleting objects, creating users, modifying IAM, and terminating EC2 fail automatically.
