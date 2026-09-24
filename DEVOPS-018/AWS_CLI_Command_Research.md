# AWS CLI Command Research — Task 3 (Help System)

**Time:** 45 mins
**Goal:** Learn to navigate the AWS CLI's built-in help system instead of relying on the trainer (or Google) every time I forget a flag.

---

## Exploring the help system

Started at the top:

```bash
aws help
```

This is the entry point into literally everything — shows global options (`--region`, `--profile`, `--output`, etc.) and then lists every AWS service the CLI can talk to (`ec2`, `s3`, `iam`, `lambda`...). Long list, but good to know it's all searchable from one place.

Narrowed it down to the two services from the practice section:

```bash
aws ec2 help
```

Lists every EC2 subcommand — `describe-instances`, `run-instances`, `stop-instances`, and a lot more than I expected. There's basically a command for every action you'd otherwise do by clicking around the console.

```bash
aws s3 help
```

Same idea, but for S3. Shows the simplified "s3" commands (`cp`, `ls`, `mb`, `rb`, `sync`, etc.). Worth noting — there's also a separate, lower-level `s3api` command set with way more granular control, but plain `s3` covers everyday stuff like uploading/downloading files.

From here the pattern is just `aws <service> <command> help` to drill into any individual command and see its required/optional flags plus usage examples (e.g. `aws ec2 describe-instances help`, `aws s3 cp help`). Used that to fill in the table below.

---

## Commands documented

| Command | Purpose | Required Parameters | Example Use |
|---|---|---|---|
| `aws ec2 describe-instances` | Lists your EC2 instances and their details (state, type, IPs, etc.) | None — runs fine with no flags, returns everything in the region | `aws ec2 describe-instances --instance-ids i-0abcd1234efgh5678` |
| `aws ec2 run-instances` | Launches new EC2 instance(s) | `--image-id` (and realistically `--instance-type` too, even though it technically has a default) | `aws ec2 run-instances --image-id ami-0abcdef1234567890 --instance-type t2.micro --key-name MyKeyPair` |
| `aws ec2 start-instances` | Starts a stopped EC2 instance | `--instance-ids` | `aws ec2 start-instances --instance-ids i-1234567890abcdef0` |
| `aws ec2 stop-instances` | Stops a running instance without deleting it | `--instance-ids` | `aws ec2 stop-instances --instance-ids i-1234567890abcdef0` |
| `aws ec2 terminate-instances` | Permanently deletes an instance — can't be undone | `--instance-ids` | `aws ec2 terminate-instances --instance-ids i-1234567890abcdef0` |
| `aws s3 ls` | Lists buckets, or objects inside a bucket | None | `aws s3 ls s3://my-bucket-name` |
| `aws s3 mb` | Creates a new bucket ("make bucket") | bucket path | `aws s3 mb s3://my-new-bucket-2026` |
| `aws s3 rb` | Deletes a bucket ("remove bucket") | bucket path (needs `--force` if it's not empty) | `aws s3 rb s3://my-new-bucket-2026 --force` |
| `aws s3 cp` | Copies a file to, from, or between S3 locations | source, destination | `aws s3 cp myfile.txt s3://my-bucket-name/` |
| `aws s3 sync` | Syncs an entire folder with a bucket — only transfers what changed | source, destination | `aws s3 sync ./local-folder s3://my-bucket-name/folder` |

---

## Takeaway

Biggest thing I noticed: almost every "specific resource" or "destructive" command (start, stop, terminate, cp, sync) needs you to explicitly name the resource — instance ID, bucket path, file path. The "read-only" commands like `describe-instances` and `s3 ls` are the only ones happy to run with zero arguments. Also — `help` opens in a pager, so if the terminal looks "stuck," press `q` to get back out. Tripped me up the first time.
