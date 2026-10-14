This continues your GitHub Actions → Docker → ECR → ECS deployment pipeline.
1. Understand the workflow
Application source code
Git commit identifies the source version

Docker build
Create a versioned image

Amazon ECR
Push and store an immutable image tag

Pull the image
Retrieve the same image for testing or deployment

Amazon ECS
Deploy the verified image


Important difference: Docker image tags are mutable by default. An immutable ECR repository rejects attempts to overwrite an existing tag, helping prevent a version from silently pointing to different image content.
2. Create an immutable ECR repository
Step 1 — Verify AWS access
Run:
aws sts get-caller-identity
aws configure get region


For this exercise, use ap-south-1 (Mumbai). If your AWS CLI identity or region is incorrect, resolve that before continuing.
Step 2 — Create the repository
Run this command in your terminal:
aws ecr create-repository \
  --repository-name devops-training-app \
  --image-tag-mutability IMMUTABLE \
  --region ap-south-1


If the repository already exists, configure its tag mutability instead:
aws ecr put-image-tag-mutability \
  --repository-name devops-training-app \
  --image-tag-mutability IMMUTABLE \
  --region ap-south-1


Verify the setting:
aws ecr describe-repositories \
  --repository-names devops-training-app \
  --region ap-south-1 \
  --query 'repositories[0].[repositoryUri,imageTagMutability]'


Expected result: your ECR repository URI and IMMUTABLE.
3. Build and tag the Docker image
Run these commands from the directory containing your Dockerfile and application files.
First, obtain your AWS account ID:
aws sts get-caller-identity --query Account --output text


Set the account ID and image variables in Bash (Linux, macOS, or Git Bash on Windows):
export AWS_REGION=ap-south-1
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export ECR_REPOSITORY=devops-training-app
export IMAGE_TAG=$(git rev-parse --short=12 HEAD)
export ECR_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY


The Git commit must exist for the git rev-parse command to work. If you haven't committed your code yet, commit it first.
Build the image:
docker build -t $ECR_REPOSITORY:$IMAGE_TAG .


Tag it with the ECR repository URI:
docker tag $ECR_REPOSITORY:$IMAGE_TAG $ECR_URI:$IMAGE_TAG


Understand the image name:
Example image reference
123456789012.dkr.ecr.ap-south-1.amazonaws.com/devops-training-app:a1b2c3d4e5f6

- 123456789012: example AWS account ID
- ap-south-1: AWS region
- devops-training-app: ECR repository
- a1b2c3d4e5f6: example commit-based image tag

Your actual account ID and tag will be different.
4. Authenticate Docker to Amazon ECR
Run:
aws ecr get-login-password --region $AWS_REGION | \
docker login --username AWS --password-stdin \
$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com


Expected output:
Login Succeeded


This uses your AWS CLI credentials to authenticate Docker. The identity needs permission to obtain an ECR authorization token.
5. Push the immutable image to ECR
Push your image:
docker push $ECR_URI:$IMAGE_TAG


Check that it exists in ECR:
aws ecr describe-images \
  --repository-name $ECR_REPOSITORY \
  --region $AWS_REGION \
  --query 'imageDetails[*].[imageTags,imageDigest,imagePushedAt]'


You should see your image tag, digest, and push timestamp.
6. Pull the same image from ECR
To simulate a deployment machine or another environment, pull the image using its exact tag:
docker pull $ECR_URI:$IMAGE_TAG


Verify the downloaded image:
docker image inspect $ECR_URI:$IMAGE_TAG \
  --format '{{json .RepoDigests}}'


For an even stronger check, compare the registry digest with the local image digest:
aws ecr describe-images \
  --repository-name $ECR_REPOSITORY \
  --image-ids imageTag=$IMAGE_TAG \
  --region $AWS_REGION \
  --query 'imageDetails[0].imageDigest' \
  --output text

docker image inspect $ECR_URI:$IMAGE_TAG \
  --format '{{.Id}}'


The registry's imageDigest and local image Id are different types of identifiers, so do not expect them to match. Compare the ECR digest with the local image's RepoDigests entry instead.
7. Test that immutability works
Try pushing a different image using the same tag:
docker tag ubuntu:24.04 $ECR_URI:$IMAGE_TAG
docker push $ECR_URI:$IMAGE_TAG


This is a deliberate test. The push should fail because the tag already exists in an immutable repository. It might fail after uploading some image layers, but ECR will reject overwriting the existing tag.
Do not use this test tag for production deployment.
To publish a new version, use a new unique tag, such as a new Git commit SHA:
export IMAGE_TAG=$(git rev-parse --short=12 HEAD)
docker build -t $ECR_URI:$IMAGE_TAG .
docker push $ECR_URI:$IMAGE_TAG


If the commit hasn't changed, the tag will remain the same and the immutable repository will reject the overwrite. Never disable immutability merely to force an overwrite.
8. Connect this to GitHub Actions
Your deployment pipeline should use a unique tag and pass the resulting image URI to ECS. The core build-and-push step can look like this:
- name: Build and push immutable image
  env:
    REGISTRY: ${{ steps.ecr.outputs.registry }}
    IMAGE_TAG: ${{ github.sha }}
  run: |
    docker build -t "$REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" .
    docker push "$REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"
    echo "IMAGE=$REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> "$GITHUB_ENV"


Use the exact image URI in your ECS task definition. For production deployments, deploying by image digest provides an even stronger guarantee that ECS uses the precise image that was verified.
9. Practical assignment checklist
ECR practical tracker
0/8 complete



[ ] Create or configure the ECR repository as IMMUTABLE
[ ] Build a Docker image using a unique commit tag
[ ] Authenticate to ECR and push the image
[ ] Verify the image tag and digest in ECR
[ ] Pull the exact image tag from ECR
[ ] Compare the registry digest with the local RepoDigests entry
[ ] Verify that overwriting an existing tag is rejected
[ ] Document the image URI, digest, and deployment version