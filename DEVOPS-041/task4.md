1. Version-tracking architecture
Git Commit
   │
   ├── Git SHA: abc123...
   │
   ▼
GitHub Actions
   │
   ├── Application version
   ├── Docker image tag
   ├── Terraform plan
   └── Deployment metadata
          │
          ▼
         ECR
          │
          │ image: abc123
          ▼
         ECS
          │
          ▼
   Running Container
          │
          ▼
     CloudWatch Logs

The key rule is:
Never rely only on latest.
2. Use Git SHA as the immutable version
For every commit:
Git commit
abc123456789

build:
devops-training-app:abc123456789

Then ECS runs exactly that image.
Example:
GitHub
  │
  │ commit abc123
  ▼
Docker build
  │
  ▼
ECR
  │
  └── devops-training-app:abc123
             │
             ▼
            ECS

This makes rollback much easier.
3. Add semantic application versions
You can also maintain releases such as:
v1.0.0
v1.1.0
v1.2.0

A useful combination is:
devops-training-app:v1.2.0
devops-training-app:abc123456

Where:
v1.2.0
   ↓
Human-readable release

abc123456
   ↓
Exact source commit

For production, the commit SHA should remain the immutable deployment identifier.
4. Generate version metadata
In GitHub Actions:
- name: Set version
  id: version
  run: |
    echo "VERSION=v1.0.${GITHUB_RUN_NUMBER}" >> "$GITHUB_OUTPUT"
    echo "COMMIT_SHA=${GITHUB_SHA}" >> "$GITHUB_OUTPUT"

You now have:
VERSION
v1.0.42

COMMIT_SHA
abc123456789...

You can use these throughout the workflow.
5. Tag the Docker image
Example:
- name: Build Docker image
  env:
    IMAGE_NAME: devops-training-app
    VERSION: ${{ steps.version.outputs.VERSION }}
    COMMIT_SHA: ${{ steps.version.outputs.COMMIT_SHA }}

  run: |
    docker build \
      -t ${IMAGE_NAME}:${COMMIT_SHA} \
      -t ${IMAGE_NAME}:${VERSION} \
      .

Now the same image has two useful tags:
devops-training-app:abc123456
devops-training-app:v1.0.42

6. Push both tags to ECR
- name: Push Docker image
  env:
    ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
    ECR_REPOSITORY: devops-training-app
    COMMIT_SHA: ${{ steps.version.outputs.COMMIT_SHA }}
    VERSION: ${{ steps.version.outputs.VERSION }}

  run: |
    docker tag \
      devops-training-app:${COMMIT_SHA} \
      ${ECR_REGISTRY}/${ECR_REPOSITORY}:${COMMIT_SHA}

    docker tag \
      devops-training-app:${COMMIT_SHA} \
      ${ECR_REGISTRY}/${ECR_REPOSITORY}:${VERSION}

    docker push \
      ${ECR_REGISTRY}/${ECR_REPOSITORY}:${COMMIT_SHA}

    docker push \
      ${ECR_REGISTRY}/${ECR_REPOSITORY}:${VERSION}

7. Better: track the image digest
Tags can be moved. An image digest identifies the exact image content.
After pushing:
ECR
 │
 ├── Tag: abc123456
 │
 └── Digest:
     sha256:8f9a...

Your strongest artifact identity becomes:
Repository
+
Commit SHA
+
Image Digest

For example:
devops-training-app
commit: abc123456
digest: sha256:8f9a...

This is much better for auditing.
8. Add OCI image labels
You can also put version information inside the Docker image.
Dockerfile:
FROM python:3.12-slim

ARG VERSION
ARG COMMIT_SHA
ARG BUILD_DATE

LABEL org.opencontainers.image.version="${VERSION}"
LABEL org.opencontainers.image.revision="${COMMIT_SHA}"
LABEL org.opencontainers.image.created="${BUILD_DATE}"

WORKDIR /app

COPY . .

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 8000

CMD ["python", "app.py"]

Build:
docker build \
  --build-arg VERSION=v1.0.42 \
  --build-arg COMMIT_SHA=abc123456 \
  --build-arg BUILD_DATE=2026-10-08T10:00:00Z \
  -t devops-training-app:abc123456 .

Now the artifact carries its own identity.
9. ECS task definition should reference the version
Avoid:
"image": "devops-training-app:latest"

Prefer:
"image": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app:abc123456"

Even better, production can pin the image by digest:
123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app@sha256:...

This guarantees ECS runs the exact artifact you tested.
10. Track ECS task-definition revisions
Your ECS history might look like:
Task Definition
────────────────────────
Revision 1 → v1.0.0
Revision 2 → v1.0.1
Revision 3 → v1.0.2
Revision 4 → v1.0.3

Keep the relationship:
ECS revision
      ↓
Docker image
      ↓
Image digest
      ↓
Git commit
      ↓
GitHub Actions run

Example:
ECS revision: 42
       ↓
Image: v1.0.42
       ↓
SHA: abc123456
       ↓
GitHub Run: #128

11. Create a deployment manifest
A very useful practice is generating a small artifact such as:
deployment.json

Example:
{
  "application": "devops-training-app",
  "version": "v1.0.42",
  "git_commit": "abc123456789",
  "docker_image": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app:abc123456789",
  "image_digest": "sha256:8f9a...",
  "ecs_cluster": "devops-training-cluster",
  "ecs_service": "devops-training-service",
  "github_run": "128",
  "deployed_at": "2026-10-08T10:55:00Z"
}

Upload this as a GitHub Actions artifact.
Now every deployment has an audit record.
12. Upload deployment metadata
Add:
- name: Generate deployment metadata
  run: |
    cat > deployment.json <<EOF
    {
      "application": "devops-training-app",
      "version": "${{ steps.version.outputs.VERSION }}",
      "git_commit": "${GITHUB_SHA}",
      "github_run": "${GITHUB_RUN_ID}",
      "deployed_at": "$(date -u +"%Y-%m-%dT%H:%M:%SZ")"
    }
    EOF

- name: Upload deployment metadata
  uses: actions/upload-artifact@v4
  with:
    name: deployment-${{ github.sha }}
    path: deployment.json

GitHub will retain the deployment record as a workflow artifact according to your configured retention policy.
13. Terraform version tracking
Terraform should also be versioned.
Record:
Terraform version
Provider versions
Git commit
Terraform plan
Terraform state

For example:
Terraform
1.14.x

AWS provider
6.x

Git commit
abc123456

Use a Terraform lock file:
.terraform.lock.hcl

Commit it to Git.
This helps ensure that CI uses the same provider selections.
14. Track Terraform plan
Instead of only running:
terraform plan

save the plan:
terraform plan -out=tfplan

Then you can inspect it and, where appropriate, apply the exact reviewed plan:
terraform apply tfplan

This gives you:
Git commit
    ↓
Terraform configuration
    ↓
Terraform plan
    ↓
Approval
    ↓
Exact plan applied

15. Add Terraform outputs
Useful outputs include:
output "ecs_cluster_name" {
  value = aws_ecs_cluster.main.name
}

output "ecs_service_name" {
  value = aws_ecs_service.app.name
}

output "ecr_repository_url" {
  value = aws_ecr_repository.app.repository_url
}

These help your deployment workflow consume infrastructure information without hard-coding values.
16. Your final artifact/version strategy
Item	Version identifier
Source code	Git SHA
Application release	v1.0.42
Docker image	Git SHA tag
Docker artifact	SHA-256 digest
ECS deployment	Task definition revision
GitHub pipeline	Run ID
Terraform	Terraform/provider version
Terraform plan	tfplan artifact
Deployment	deployment.json
Logs	Git SHA/version metadata


Final flow
                    Git Commit
                       │
                 abc123456
                       │
                       ▼
                GitHub Actions
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
       Terraform              Docker
       Plan                    Build
         │                       │
         ▼                       ▼
     tfplan              v1.0.42 + abc123
                                 │
                                 ▼
                                ECR
                                 │
                          sha256:8f9a...
                                 │
                                 ▼
                                ECS
                                 │
                       Task Definition #42
                                 │
                                 ▼
                         Running Container
                                 │
                                 ▼
                         Health Check
                                 │
                                 ▼
                       deployment.json

The key rule
Build once, identify it immutably, promote that exact artifact.
So if production is running:
v1.0.42
commit abc123456
digest sha256:8f9a...
ECS revision 42

you can reproduce exactly what is running and roll back to the previous known-good digest without rebuilding anything.