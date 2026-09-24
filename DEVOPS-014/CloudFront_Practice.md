TASK 12 — AWS CLI CloudFront Practice

Time: 45 Minutes
Topic: AWS CLI CloudFront Practice
Training: DevOps Engineer Training

1. Objective

The objective of this task is to practice using the AWS CLI to inspect an Amazon CloudFront environment.

The trainee must be able to:

List CloudFront distributions
Retrieve details of a specific distribution
Identify the distribution status
Identify the CloudFront domain name
Use AWS CLI query options to display only required information

AWS provides the cloudfront list-distributions command for listing distributions and cloudfront get-distribution for retrieving details about a specific distribution.

2. Prerequisites

Before starting, verify that AWS CLI is installed and that you have authorized AWS credentials.

Check AWS CLI:

aws --version

Example:

aws-cli/2.x.x Python/3.x ...

Verify the current AWS identity:

aws sts get-caller-identity

Example:

{
    "UserId": "AIDAXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/training-user"
}

Security Note: Never place real AWS access keys, secret keys, passwords, or other credentials inside this document or screenshots.

3. CloudFront CLI Architecture

The basic CLI interaction is:

Developer
    │
    ↓
AWS CLI
    │
    ↓
AWS CloudFront API
    │
    ├── List Distributions
    ├── Get Distribution
    ├── Check Status
    └── Get Domain Name
4. Command 1 — List CloudFront Distributions
Command
aws cloudfront list-distributions

AWS documents this command as the CLI operation for listing CloudFront distributions.

Purpose

This command retrieves the CloudFront distributions available in the AWS account.

It can show information such as:

Distribution ID
ARN
Status
Last modified time
Domain name
Aliases
Enabled status
Other distribution configuration information
Example Output
{
    "DistributionList": {
        "Items": [
            {
                "Id": "E123EXAMPLE",
                "ARN": "arn:aws:cloudfront::123456789012:distribution/E123EXAMPLE",
                "Status": "Deployed",
                "LastModifiedTime": "2026-09-13T10:30:00.000Z",
                "DomainName": "d123example.cloudfront.net"
            }
        ],
        "Quantity": 1
    }
}

Important: The values shown above are examples. Your account will return different values.

5. Command 2 — Display Only Distribution IDs

The complete output can be large.

AWS CLI supports the --query option to select specific fields from the response.

Command
aws cloudfront list-distributions --query "DistributionList.Items[].Id"
Purpose

Displays only the CloudFront distribution IDs.

Example Output
[
    "E123EXAMPLE",
    "E456EXAMPLE"
]

This is useful when an AWS account contains multiple distributions.

6. Command 3 — Display Distribution IDs and Status
Command
aws cloudfront list-distributions \
  --query "DistributionList.Items[].{ID:Id,Status:Status}"
Purpose

Displays the distribution ID and current status.

Example Output
[
    {
        "ID": "E123EXAMPLE",
        "Status": "Deployed"
    }
]

This makes it easier to quickly identify whether a distribution has reached the deployed state.

7. Command 4 — Display Distribution Domain Name
Command
aws cloudfront list-distributions \
  --query "DistributionList.Items[].DomainName"
Purpose

Displays the CloudFront domain name associated with each distribution.

Example Output
[
    "d123example.cloudfront.net"
]

A CloudFront distribution normally has a domain name similar to:

d123example.cloudfront.net

The exact value will be different for your distribution. AWS's documentation shows DomainName as a field returned by list-distributions.

8. Command 5 — Display ID, Status, and Domain Name

This is a useful practical command because it displays the main information needed for this task.

Command
aws cloudfront list-distributions \
  --query "DistributionList.Items[].{ID:Id,Status:Status,Domain:DomainName}"
Example Output
[
    {
        "ID": "E123EXAMPLE",
        "Status": "Deployed",
        "Domain": "d123example.cloudfront.net"
    }
]
9. Command 6 — Retrieve Distribution Details

Once you know the distribution ID, use:

aws cloudfront get-distribution --id DISTRIBUTION_ID

AWS documents --id as a required parameter for get-distribution.

Example:

aws cloudfront get-distribution --id E123EXAMPLE
Purpose

Retrieves detailed information about one specific CloudFront distribution.

The response can contain:

Distribution ID
ARN
Status
Last modified time
Domain name
Origins
Cache behaviors
Viewer protocol policy
SSL/TLS configuration
Restrictions
Enabled status
Other distribution configuration details
10. Command 7 — Retrieve Only Distribution Status

Instead of displaying the entire distribution configuration, use --query.

Command
aws cloudfront get-distribution \
  --id E123EXAMPLE \
  --query "Distribution.Status"
Example Output
"Deployed"
Meaning

The CloudFront distribution status tells us its current deployment state.

A commonly expected status is:

Deployed

AWS states that when a distribution's status is Deployed, its information has been propagated to CloudFront edge locations.

11. Command 8 — Retrieve CloudFront Domain Name
Command
aws cloudfront get-distribution \
  --id E123EXAMPLE \
  --query "Distribution.DomainName"
Example Output
"d123example.cloudfront.net"

This identifies the domain name assigned to the CloudFront distribution.

12. Command 9 — Retrieve Important Information Together

A useful command for daily DevOps work is:

aws cloudfront get-distribution \
  --id E123EXAMPLE \
  --query "Distribution.{ID:Id,Status:Status,Domain:DomainName}"
Example Output
{
    "ID": "E123EXAMPLE",
    "Status": "Deployed",
    "Domain": "d123example.cloudfront.net"
}

This is easier to read than the complete JSON response.

13. Command 10 — Use Text Output

AWS CLI also supports output formatting.

For example:

aws cloudfront list-distributions \
  --query "DistributionList.Items[].{ID:Id,Status:Status,Domain:DomainName}" \
  --output table
Example Output
----------------------------------------------------------
|                 ListDistributions                      |
+--------------+-----------+-----------------------------+
| ID           | Status    | Domain                      |
+--------------+-----------+-----------------------------+
| E123EXAMPLE  | Deployed  | d123example.cloudfront.net |
+--------------+-----------+-----------------------------+

This format can be useful when reviewing infrastructure from a terminal.

14. Main Command Reference
Command	Purpose	Important Output
aws cloudfront list-distributions	List distributions	ID, status, domain
aws cloudfront list-distributions --query "DistributionList.Items[].Id"	List IDs	Distribution IDs
aws cloudfront list-distributions --query "DistributionList.Items[].DomainName"	List domains	CloudFront domains
aws cloudfront list-distributions --query "DistributionList.Items[].{ID:Id,Status:Status}"	List IDs and status	ID + status
aws cloudfront get-distribution --id ID	Get distribution details	Complete distribution information
aws cloudfront get-distribution --id ID --query "Distribution.Status"	Get status	Deployed, etc.
aws cloudfront get-distribution --id ID --query "Distribution.DomainName"	Get domain	CloudFront domain
aws cloudfront get-distribution --id ID --query "Distribution.{ID:Id,Status:Status,Domain:DomainName}"	Get key details	ID + status + domain
15. Practical Exercise
Step 1 — List distributions

Run:

aws cloudfront list-distributions

Record:

Distribution ID:
Status:
Domain Name:
Step 2 — Get the Distribution ID

Run:

aws cloudfront list-distributions \
  --query "DistributionList.Items[].Id"

Copy the appropriate distribution ID.

Example:

E123EXAMPLE

Do not use the example ID in your environment.

Step 3 — Retrieve Distribution Details

Run:

aws cloudfront get-distribution --id YOUR_DISTRIBUTION_ID

Example:

aws cloudfront get-distribution --id E123EXAMPLE

Review the response.

Look for:

Id
Status
DomainName
Origins
Enabled
Step 4 — Check Status

Run:

aws cloudfront get-distribution \
  --id YOUR_DISTRIBUTION_ID \
  --query "Distribution.Status"

Expected example:

"Deployed"
Step 5 — Check Domain Name

Run:

aws cloudfront get-distribution \
  --id YOUR_DISTRIBUTION_ID \
  --query "Distribution.DomainName"

Expected example:

"d123example.cloudfront.net"
Step 6 — Display All Required Information

Run:

aws cloudfront get-distribution \
  --id YOUR_DISTRIBUTION_ID \
  --query "Distribution.{ID:Id,Status:Status,Domain:DomainName}" \
  --output table

Example:

----------------------------------------------
|              GetDistribution               |
+--------------+-----------+-----------------+
| ID           | Status    | Domain          |
+--------------+-----------+-----------------+
| E123EXAMPLE  | Deployed  | d123example...  |
+--------------+-----------+-----------------+
16. Required Documentation

The trainee should document the commands used during the exercise.

Command	Purpose	Output
aws cloudfront list-distributions	List CloudFront distributions	Distribution list
aws cloudfront get-distribution --id YOUR_ID	Retrieve distribution details	Distribution configuration/details
aws cloudfront get-distribution --id YOUR_ID --query "Distribution.Status"	Identify distribution status	Status such as Deployed
aws cloudfront get-distribution --id YOUR_ID --query "Distribution.DomainName"	Identify CloudFront domain	d123example.cloudfront.net
17. Recommended Screenshot Evidence

For the training record, capture screenshots of:

Screenshot 1 — List distributions
aws cloudfront list-distributions

Screenshot:

[Insert Screenshot Here]
Screenshot 2 — Distribution status
aws cloudfront get-distribution \
  --id YOUR_DISTRIBUTION_ID \
  --query "Distribution.Status"

Screenshot:

[Insert Screenshot Here]
Screenshot 3 — CloudFront domain
aws cloudfront get-distribution \
  --id YOUR_DISTRIBUTION_ID \
  --query "Distribution.DomainName"

Screenshot:

[Insert Screenshot Here]
Screenshot 4 — Summary
aws cloudfront get-distribution \
  --id YOUR_DISTRIBUTION_ID \
  --query "Distribution.{ID:Id,Status:Status,Domain:DomainName}" \
  --output table

Screenshot:

[Insert Screenshot Here]
18. Troubleshooting
Problem 1 — AWS CLI not found

Error:

aws: command not found

or Windows:

'aws' is not recognized as an internal or external command
Check:
aws --version

If AWS CLI is not installed, install it according to your organization's approved procedure.

Problem 2 — AccessDenied

Example:

An error occurred (AccessDenied) when calling the ListDistributions operation

This means the AWS identity may be authenticated but does not have sufficient authorization for the requested operation.

Check:

aws sts get-caller-identity

Then review the IAM permissions assigned to the identity.

Problem 3 — No distributions

The command may return an empty distribution list.

Example:

{
    "DistributionList": {
        "Items": [],
        "Quantity": 0
    }
}

This means there are currently no CloudFront distributions visible to that AWS account/identity.

Do not create a distribution unless the training task specifically requires it.

Problem 4 — Wrong AWS account

Always verify the identity before inspecting infrastructure:

aws sts get-caller-identity

Check that you are working in the intended AWS account.

19. Important AWS CLI Concepts Learned
Service
cloudfront

Specifies that the AWS CLI command is interacting with CloudFront.

Operation
list-distributions

Lists distributions.

Operation
get-distribution

Retrieves one distribution.

Parameter
--id

Specifies which distribution should be inspected.

Query
--query

Selects specific fields from the command response.

Output
--output table

Changes how the result is displayed.

20. DevOps Use Case

CloudFront information can be inspected from scripts and CI/CD workflows.

For example:

CI/CD Pipeline
      ↓
AWS CLI
      ↓
CloudFront
      ↓
Distribution Status
      ↓
Domain Name
      ↓
Deployment Verification

Instead of manually opening the AWS Console, a DevOps engineer can retrieve infrastructure information through the CLI.

Example:

aws cloudfront list-distributions \
  --query "DistributionList.Items[].{ID:Id,Status:Status,Domain:DomainName}" \
  --output table

This is useful for automation, troubleshooting, monitoring, and deployment verification.

21. Key Learning

The main commands learned in this exercise are:

List
 ↓
aws cloudfront list-distributions

Get Details
 ↓
aws cloudfront get-distribution --id ID

Check Status
 ↓
--query "Distribution.Status"

Get Domain
 ↓
--query "Distribution.DomainName"

The overall workflow is:

AWS CLI
   ↓
CloudFront
   ↓
List Distribution
   ↓
Get Distribution ID
   ↓
Get Distribution Details
   ↓
Check Status
   ↓
Check Domain Name
22. What I Learned

From this task, I learned:

How to list CloudFront distributions using AWS CLI.
How to retrieve details of a specific distribution.
How to identify the CloudFront distribution ID.
How to identify the distribution status.
How to identify the CloudFront domain name.
How to use --query to extract specific information.
How to use --output table for easier terminal-based viewing.
How AWS CLI can be used instead of the AWS Console for infrastructure inspection.
How IAM permissions affect CloudFront CLI operations.
How CloudFront CLI commands can support DevOps automation.