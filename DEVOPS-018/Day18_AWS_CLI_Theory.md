# Day 18 — AWS CLI Theory (Revision)

**Task:** Answer these from memory, no notes allowed. Let's see how much actually stuck 😅
**Time:** 45 mins

---

## 1. What is AWS CLI?

AWS CLI (Command Line Interface) is a tool that lets you manage AWS services straight from the terminal instead of logging into the AWS Console and clicking around. Once it's installed and configured, you can create, update, or delete resources — EC2 instances, S3 buckets, IAM users, whatever — just by typing commands. Under the hood it's calling the same AWS APIs the console uses; it's just a different, faster front door.

## 2. Why do DevOps engineers use AWS CLI?

Because manually clicking through the console doesn't scale once you're managing real infrastructure. A few reasons:

- **Automation** — commands can be put into scripts, cron jobs, or pipelines instead of repeated manually.
- **CI/CD integration** — Jenkins, GitHub Actions, etc. call AWS CLI (or the SDKs) behind the scenes to deploy things.
- **Speed** — one command beats five console pages, especially for repetitive tasks.
- **Consistency** — no "oops I forgot to check that box" mistakes; a script does the same thing every time.
- **Works headless** — most servers you SSH into don't have a browser, so CLI is often the only option.

## 3. AWS Console vs AWS CLI?

| | Console | CLI |
|---|---|---|
| Type | Web GUI | Terminal/text |
| Good for | Learning, exploring, one-off changes | Automation, repetitive or bulk work |
| Speed | Slower, manual clicks | Faster once you know the commands |
| Scriptable | No | Yes |
| Learning curve | Easy to start | Steeper at first |

Both are really just two different doors into the same AWS APIs.

## 4. What is an AWS CLI profile?

A profile is a named set of credentials + config (access key, secret key, default region, output format) saved in `~/.aws/credentials` and `~/.aws/config`. It's handy when you deal with more than one AWS account — like a personal sandbox and a work account — since you can just switch with `--profile` instead of overwriting your keys every time:

```bash
aws s3 ls --profile personal
aws s3 ls --profile work
```

## 5. What is authentication?

Authentication is proving **who you are**. When the CLI sends a request, AWS checks the access key/secret key (or temporary/SSO credentials) against its records to confirm you're really the IAM user or role you claim to be. No valid credentials, no access — full stop.

## 6. What is authorization?

Authorization is about **what you're allowed to do** once AWS already knows who you are, and it's controlled by IAM policies attached to your user or role. So it's totally possible to authenticate successfully and still get an "Access Denied" error if your permissions don't cover that action. Simple way to remember it: authentication gets you through the front door, authorization decides which rooms you're allowed into.

## 7. Why should credentials not be hard-coded?

Because it's basically writing your password on a sticky note and taping it to your laptop:

- If keys get hardcoded into a script and accidentally pushed to GitHub, there are bots scanning public repos 24/7 that will find and abuse them — sometimes within minutes.
- Rotating keys becomes painful if they're scattered across a dozen files instead of one place.
- The proper way is to use CLI profiles, environment variables, IAM roles, or a secrets manager (like AWS Secrets Manager) — anything that keeps the actual keys out of your source code.

## 8. What is an AWS Region?

A region is a specific geographic location where AWS runs a cluster of data centers — for example `us-east-1` (N. Virginia) or `ap-south-1` (Mumbai). Regions are isolated from each other, so a resource created in one (say an EC2 instance in Mumbai) won't show up if you switch to N. Virginia. You generally pick a region based on where your users are (lower latency), data residency/compliance requirements, and whether the AWS service you need is even offered there.

## 9. What does `aws sts get-caller-identity` do?

It tells you exactly who you currently are, as far as AWS is concerned — it returns your **Account ID**, **User ID**, and **ARN**. I run this constantly as a sanity check before doing anything risky, just to confirm I'm on the right profile/account and not about to run a command against prod by accident.

---

*Revised from memory on Day 18 — will cross-check against original notes after this.*
