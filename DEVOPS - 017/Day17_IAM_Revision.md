## 1. What is IAM?

IAM (Identity and Access Management) is a global, free AWS service that lets you securely control access to AWS resources. It manages **who** can sign in (authentication) and **what** they can do (authorization) through users, groups, roles, and policies. Being global means IAM applies across all AWS regions, not just one.

---

## 2. What is an IAM User?

An IAM User is an identity representing a single person or application that interacts with AWS. Each user:
- Has a unique name within the account
- Has **no permissions by default** — must be explicitly granted via policies
- Can have a console password and/or access keys (Access Key ID + Secret Access Key) for CLI/SDK use

---

## 3. What is an IAM Group?

An IAM Group is a collection of IAM Users used to manage permissions in bulk. Instead of attaching a policy to every user individually, you attach it once to the group.
- A user can belong to multiple groups
- Groups **cannot be nested** (no group-within-a-group)
- A group is not an identity itself — it has no credentials and can't be referenced as a "principal"

---

## 4. What is an IAM Role?

An IAM Role is an identity with permissions that is **assumed temporarily** rather than owned permanently. It can be assumed by IAM users, AWS services (e.g., EC2, Lambda), or federated/external identities. When assumed, AWS issues **temporary security credentials** (via STS) instead of long-term keys. Common uses:
- Letting an AWS service act on your behalf (e.g., EC2 reading from S3)
- Cross-account access
- Federated login (SSO, Google, SAML, etc.)

---

## 5. What is an IAM Policy?

An IAM Policy is a JSON document that defines **permissions** — what's allowed or denied, on which resources, under what conditions. Core elements of a statement:
- **Effect** – Allow or Deny
- **Action** – the API call(s), e.g., `s3:GetObject`
- **Resource** – the AWS resource(s) it applies to
- **Condition** *(optional)* – extra restrictions (IP address, MFA, time, etc.)

Policies come in three types: **AWS managed**, **customer managed**, and **inline**. They can be attached to users, groups, or roles.

---

## 6. What is Authentication?

Authentication answers **"Who are you?"** It's the process of verifying identity before granting access — e.g., logging in with a username/password, access keys, or MFA.

---

## 7. What is Authorization?

Authorization answers **"What are you allowed to do?"** Once identity is confirmed (authenticated), IAM policies determine whether that identity can perform a given action on a given resource.

> Order matters: **authenticate first, authorize second.**

---

## 8. What is Least Privilege?

The Principle of Least Privilege means giving a user, group, or role **only the permissions needed to do its job — nothing more**. This limits the damage a compromised, misconfigured, or careless identity can cause, and is a foundational AWS security best practice.

---

## 9. Why should root credentials not be used for daily work?

The root user has **full, unrestricted access** to the entire AWS account — billing, all resources, even account closure — and this access can't be limited by an IAM policy.
- A compromised root account means total account compromise
- Best practice: enable MFA on root, store its credentials securely, and use root only for the few tasks that require it (e.g., changing account settings, closing the account)
- For everyday work, use an IAM user/role with only the permissions actually needed

---

## 10. Why should AWS credentials never be committed to GitHub?

Once pushed to GitHub (even briefly, even in a private repo), credentials can leak or be scraped:
- Automated bots scan public repos continuously and can find exposed AWS keys within minutes
- Leaked keys can be used to launch expensive resources (e.g., crypto-mining), exfiltrate data, or fully compromise the account
- Deleting the file later isn't enough — the keys often remain in the Git commit history

**Prevention:** `.gitignore` credential files, use environment variables or AWS Secrets Manager/Parameter Store, prefer IAM roles over hardcoded long-term keys, and use tools like `git-secrets` or GitHub secret scanning.

---

*End of Day 17 IAM Revision*
