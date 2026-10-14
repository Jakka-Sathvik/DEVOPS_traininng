Audit Secrets Management
For your ECS → RDS PostgreSQL architecture, the goal is to make sure database credentials and other sensitive values are never exposed in code, Docker images, GitHub, or logs.
1. Check AWS Secrets Manager
You previously defined:
Secret:
devops-training/database

Verify:
AWS Console
 → Secrets Manager
 → Secrets
 → devops-training/database

Check:
☐ Secret exists
☐ Secret is encrypted
☐ Database username/password stored there
☐ No plaintext credentials in the secret name/description

2. Check ECS Task Definition
Your ECS task should retrieve the secret through Secrets Manager.
Recommended flow:
ECS Task
   ↓
Task Role
   ↓
Secrets Manager
   ↓
Database Credentials
   ↓
RDS PostgreSQL

Do not put this directly in the task definition:
DB_PASSWORD=mypassword123

Instead, reference the Secrets Manager secret.
3. Audit IAM Permissions
The ECS task role should have only the permission required to read the specific secret.
Example:
{
  "Effect": "Allow",
  "Action": "secretsmanager:GetSecretValue",
  "Resource": "arn:aws:secretsmanager:<region>:<account-id>:secret:devops-training/database-*"
}

Check:
☐ GetSecretValue allowed
☐ Specific secret ARN used
☐ No secretsmanager:* permission
☐ No Resource: "*"
☐ No AdministratorAccess

Important: The task role needs access to application secrets. The execution role is primarily for ECS startup operations such as pulling images and sending logs.
4. Check GitHub Repository
Search the repository for:
password
passwd
secret
api_key
access_key
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
DB_PASSWORD

Check:
☐ No passwords committed
☐ No AWS access keys committed
☐ No .env file committed
☐ .env added to .gitignore
☐ No secrets in GitHub Actions YAML

Your .gitignore should include:
.env
.env.*
!.env.example

Your repository can contain:
.env.example

with dummy values:
DB_HOST=example
DB_NAME=example
DB_USER=example
DB_PASSWORD=example

5. Check Docker Image
Make sure secrets are not baked into the Docker image.
Bad:
ENV DB_PASSWORD="mypassword"

Also avoid:
COPY .env /app/.env

Better:
Docker Image
     ↓
No credentials
     ↓
ECS Task
     ↓
Secrets Manager

6. Check Logs
Search CloudWatch Logs for:
password
secret
token
api_key
authorization

Your application should never log:
DB_PASSWORD=MySecretPassword

or:
Authorization: Bearer <token>

Expected:
Database connection successful

without exposing credentials.
7. Encryption
Verify the secret is encrypted using AWS KMS.
Recommended:
Application
     ↓
Secrets Manager
     ↓
KMS Encryption

For higher-security environments, use a customer-managed KMS key and restrict who can use/decrypt it.
8. Rotation
For production, establish a credential rotation strategy.
Current Password
       ↓
Rotation
       ↓
New Password
       ↓
Application Uses New Secret

At minimum, document:
☐ Who can rotate secrets
☐ How secrets are rotated
☐ How applications receive updated values
☐ What happens if rotation fails

Final Secrets Audit Checklist
☐ Secrets stored in AWS Secrets Manager
☐ No credentials in GitHub
☐ No credentials in Dockerfile
☐ No credentials in Docker images
☐ No credentials in .env committed to Git
☐ ECS uses Secrets Manager references
☐ ECS task role has least-privilege access
☐ Secret encrypted
☐ RDS credentials not publicly exposed
☐ Secrets not written to CloudWatch logs
☐ Rotation strategy documented
☐ Secret access audited

Secure Architecture
                 GitHub
                   │
             OIDC Authentication
                   │
                   ▼
             GitHub Actions
                   │
                   ▼
                  ECR
                   │
                   ▼
             ECS Fargate
                   │
          ┌────────┴────────┐
          │                 │
          ▼                 ▼
   Secrets Manager        RDS
   DB credentials       PostgreSQL

Audit result: Your target state should be zero hard-coded secrets, least-privilege IAM, Secrets Manager for credentials, and no sensitive values in source code, Docker images, task definitions, or logs.