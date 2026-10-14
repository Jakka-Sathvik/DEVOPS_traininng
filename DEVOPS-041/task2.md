Target CI/CD pipeline
Developer
   │
   ▼
Git Push / Pull Request
   │
   ▼
┌──────────────────────────────┐
│       GitHub Actions         │
└──────────────┬───────────────┘
               │
               ▼
        1. Linting Gate
               │
          PASS │ FAIL → STOP
               ▼
        2. Validation Gate
               │
          PASS │ FAIL → STOP
               ▼
        3. Security Gate
               │
          PASS │ FAIL → STOP
               ▼
        4. Terraform Plan
               │
          PASS │ FAIL → STOP
               ▼
        5. Deployment Approval
               │
          APPROVED
               ▼
        6. Docker Build
               │
               ▼
        7. Push to ECR
               │
               ▼
        8. Deploy to ECS
               │
               ▼
        9. Health Check
               │
          PASS │ FAIL
               ▼
            SUCCESS

1. Linting gate
For Terraform:
terraform fmt -check -recursive

You can also add a Terraform linter such as tflint.
For application code, use the appropriate linter for your project.
Example Python:
- name: Install linting tools
  run: |
    pip install ruff

- name: Run lint
  run: ruff check .

If linting fails:
❌ Linting failed
       ↓
Pipeline stops
       ↓
No deployment

2. Terraform validation gate
Add:
- name: Terraform Init
  run: terraform init -backend=false

- name: Terraform Format Check
  run: terraform fmt -check -recursive

- name: Terraform Validate
  run: terraform validate

This catches problems such as:
Invalid Terraform syntax
Missing arguments
Invalid resource configuration
Invalid variable references

3. Terraform security scanning
Add a security scanner such as Checkov.
Example:
- name: Run Checkov
  uses: bridgecrewio/checkov-action@v12
  with:
    directory: terraform
    framework: terraform
    soft_fail: false

Now Terraform has two security layers:
Terraform
   │
   ├── fmt
   ├── validate
   └── Checkov

For example, Checkov can detect risky configurations such as:
Public S3 bucket
Overly permissive security group
Unencrypted resource
Weak IAM policy

4. Docker security scanning
After building your image:
- name: Build Docker image
  run: |
    docker build \
      -t devops-training-app:${{ github.sha }} .

Run Trivy:
- name: Trivy scan
  uses: aquasecurity/trivy-action@0.28.0
  with:
    image-ref: devops-training-app:${{ github.sha }}
    format: table
    severity: CRITICAL,HIGH
    exit-code: 1

The important setting is:
exit-code: 1

That means:
Critical/High vulnerability
        ↓
Trivy returns failure
        ↓
GitHub Actions fails
        ↓
No ECR push
        ↓
No ECS deployment

5. Terraform plan gate
After validation/security checks:
- name: Terraform Plan
  run: terraform plan -out=tfplan

You should review the plan before production deployment.
For example:
Terraform will perform:

+ create ECS task definition
~ update ECS service
+ create security group

But if you see:
- destroy RDS
- destroy VPC
- destroy production ALB

you should stop the deployment and investigate.
6. Deployment approval gate
For production, don't automatically run:
terraform apply -auto-approve

immediately after plan.
Use GitHub Environments.
Create:
dev
staging
production

For production, configure a required reviewer.
Then your workflow becomes:
PR
 ↓
Lint
 ↓
Validate
 ↓
Security Scan
 ↓
Terraform Plan
 ↓
Merge
 ↓
Production Approval
 ↓
Terraform Apply
 ↓
ECS Deployment

This is an important deployment gate.
7. ECS deployment gate
After the Docker image passes security scanning:
Docker
  ↓
Build
  ↓
Trivy
  ↓
ECR
  ↓
ECS

Use:
- name: Deploy ECS
  uses: aws-actions/amazon-ecs-deploy-task-definition@v2
  with:
    task-definition: ${{ steps.task-def.outputs.task-definition }}
    service: devops-training-service
    cluster: devops-training-cluster
    wait-for-service-stability: true

This is important:
wait-for-service-stability: true

GitHub Actions waits for ECS to become stable instead of simply saying "deployment submitted."
8. Health-check gate
Your ALB already has:
/health

with the expected response:
{
  "status": "healthy"
}

After ECS deployment:
curl -f https://YOUR-ALB-DNS/health

Using -f means curl returns a failure status for HTTP errors.
Example:
- name: Application health check
  run: |
    curl --fail --retry 10 --retry-delay 10 \
      https://${{ vars.APP_DOMAIN }}/health

Pipeline:
ECS deployment
      ↓
Health check
      │
   ┌──┴──┐
   │     │
 PASS   FAIL
   │     │
   ▼     ▼
Success  Stop

9. Recommended workflow separation
I recommend three workflows rather than putting everything into one huge YAML file.
terraform-ci.yml
PR
 │
 ├── Terraform fmt
 ├── Terraform validate
 ├── TFLint
 ├── Checkov
 └── Terraform plan

docker-ci.yml
PR / Push
 │
 ├── Application lint
 ├── Unit tests
 ├── Docker build
 ├── Trivy
 └── Docker image validation

deploy.yml
main
 │
 ├── Authenticate with AWS
 ├── Push image to ECR
 ├── Update ECS task definition
 ├── Deploy ECS
 ├── Wait for stability
 └── Health check

This makes troubleshooting much easier.
10. Final gate matrix
Gate	Tool	Failure action
Code lint	Ruff/ESLint/etc.	Stop
Terraform format	terraform fmt	Stop
Terraform validation	terraform validate	Stop
Terraform lint	TFLint	Stop
IaC security	Checkov	Stop
Unit tests	Test framework	Stop
Docker build	Docker	Stop
Container security	Trivy	Stop
Terraform plan	Terraform	Stop/review
Production approval	GitHub Environment	Wait
ECS deployment	ECS	Stop if failed
ECS stability	ECS	Stop if unhealthy
Application health	/health	Stop/rollback


Final architecture
                 GitHub
                    │
                    ▼
              Pull Request
                    │
             ┌──────┴──────┐
             ▼             ▼
          Linting      Unit Tests
             │             │
             └──────┬──────┘
                    ▼
             Terraform Validate
                    │
                    ▼
              TFLint + Checkov
                    │
                    ▼
             Terraform Plan
                    │
             ┌──────┴──────┐
             │             │
           FAIL           PASS
             │             │
            STOP           ▼
                    Approval Gate
                           │
                           ▼
                     Docker Build
                           │
                           ▼
                      Trivy Scan
                           │
                           ▼
                          ECR
                           │
                           ▼
                     ECS Deploy
                           │
                           ▼
                  ECS Stability Check
                           │
                           ▼
                    /health Check
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                  PASS           FAIL
                    │             │
                    ▼             ▼
                 SUCCESS       ROLLBACK

Your key rule should be: no gate passes → no deployment. This gives your GitHub Actions + Terraform + ECS pipeline proper CI/CD quality and production safety.